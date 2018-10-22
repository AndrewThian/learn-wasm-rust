## Introduction ##

Learning repo for using how to use `rust` with web-assembly and javascript.
I'm following on the free course on [egghead.io](https://egghead.io/lessons/javascript-setup-rust-for-webassembly).

## Installation ##

**Installing Rust**

1. for OSx, we can use `brew install rustup`. If we are on any other OS, head over to [https://rustup.rs/] for more installation options

2. after installing `rustup`, run:

    > `$ rustup-init`

    which will install rust into the system. Don't forget to `source $HOME/.cargo/env` in the current shell.

3. we want to set the `nightly` toolchain as the default:

    > $ `rustup default nightly`

    we need to do this because only the `nightly` toolchain is supporting webassembly.

4. adding target:

    > $ `rustup target add wasm32-unknown-unknown`

    usually we add targets like `x68_64-apple-darwn`, specifying the target platform. Since web-assembly is not compiled on any specific platform, the values after the first dash are `unknown`.

5. installing `wasm-pack`:

    > $ `cargo install wasm-pack`

    we use `cargo`(rust's package manager) to install `wasm-pack`. This tool is the one-stop shop for building and working with Rust-generated web-assembly that we would like to interop with javascript. 

6. installing `wasm-gc`:

    > $ `cargo install wasm-gc`

    For the first few tutorials, we will be using a tool called `wasm-gc`. `wasm-gc` is a tool that removes all unneeded exports, imports, functions and etc from a web-assembly module.

7. web-server:

    > $ `cargo install https`

    we need a web server, so we installed it with cargo's http crate. It's a web server that serves static content from the current directory, we could have used any other web-server. Maybe even `webpack-dev-server`?

## Summary ##

Now that we've installed a whole bunch of stuff, I guess we're ready to go with the course itself. Summary of what we've installed:

> 1. Rust with `rustup-init`
> 
> 2. set up Rust to use a `nightly` build
> 
> 3. tell Rust to target web-assembly
>
> 4. Rust's package manager to install `wasm-pack` and `wasm-gc`(just for the first few lessons).
>
> 5. Rust's https package to serve static content

---

## Lessons ##

### 1. Load a web-assembly function written in Rust and invoke from Javascript ###

First we want to create a Rust library with the `cargo` package manager:

> $ `cargo new --lib utils`

Then we have to change the crate type to `cdylib` in Cargo.toml. This has an impact on how things are linked together during compilation.

```toml
[lib]
crate-type = ["cdylib"]
```

Now we can write some Rust code!

```rust
// [extern] keywork 
// is needed to create an interface so that this function 
// can be invoked from other languages
// -------
// [no_mangle] annotation 
// to tell the rust compiler not to mangle the name of this function
#[no_mangle]
pub extern fn add_one(x: u32) -> u32 {
    x + 1
}
```

After we've written our super fast rust implementation of a counter, lets build the application and target `wasm32-unknown-unknown`. Which will dump the built files into the `project-root/target/*` directory.

> $ `cargo build --target wasm32-unknown-unknown --release`

Next, use `wasm-gc`(which is not recommended, only for demonstration purposes) to build our wasm file

> $ `wasm-gc target/wasm32-unknown/release/utils.wasm -o utils.gc.wasm`

There are two ways of interacting with web-assembly code, `instantiateStreaming` directly or fetch and then instantiate a buffer.

1. fetching and instantiating a web-assembly module:

    ```html
    <script>
        fetch("utils.gc.wasm")
            .then(response => response.arrayBuffer())
            // WebAssembly.instantiate takes in arrayBuffer
            // https://developer.mozilla.org/en-US/docs/WebAssembly/Using_the_JavaScript_API
            .then(buffer => WebAssembly.instantiate(result))
            .then(wasmModule => {
                const result = wasmModel.instance.exports.add_one(2)
                const text = document.createTextNode(result);
                document.body.appendChild(text);
            })
    </script>
    ```

2. `InstantiateStreaming`:

    ```html
    <script>
        WebAssembly.instantiateStreaming(fetch("utils.gc.wasm"))
            .then(module => {
                const result = module.instance.exports.add_one(3);
                const text = document.createTextNode(result)
                document.body.appendChild(text);
            })
    </script>
    ```

    > This way we can stream, compile and instantiate a web-assembly module in one go. This can speed up the time to execution a lot especially on slow connections.

---

### 2. Pass a javascript function to WebAssembly and invoke from Rust ###

Some cases, it's useful to be able to invoke Javascript functions inside of Rust. Basically, we want to export a function from javascript, import it into rust and then instantiate the stream from web-assembly.

First we want to define a function in an object in our javascript

```html
<script>
    const appendNumberToBody = number => {
        const text = document.createTextNode(number)
        document.body.appendChild(text)
    }

    const importObject = {
        appendNumberToBody: appendNumberToBody
    }
</script>
```

let's write our rust code to accept the javascript object we've imported.

```rust
extern {
    fn appendNumberToBody(x: u32);
}

#[no_mangle]
pub extern fn run() {
    // wrapping in an unsafe block
    // rust compiler can't provide mem safety for external functions
    unsafe {
        appendNumberToBody(42);
    }
}
```

finally, we can instantiate the web-assembly module in our javascript on our `index.html`, the same way we've done so before.

```html
<script>
    // ... previous code obmitted for ease of reading

    WebAssembly
        .instantiateStreaming(fetch("utils.gc.wasm"), importObject)
        .then(wasmModule => {
                console.log(wasmModule)
                wasmModule.instance.exports.run()
            })
</script>
```

Next add one more, we're gonna pass in a **native** function into the module

```html
<script>
    // ... previous code obmitted for ease of reading

    const importObject = {
        env: {
            appendNumberToBody: appendNumberToBody,
            alert: alert
        }
    }

    // ... previous code obmitted for ease of reading
</script>
```

then, import the function into our rust code and import our unsafe block. Don't forget to recompile with `cargo build` and `wasm-gc`.

```rust
extern {
    fn appendNumberToBody(x: u32);
    fn alert(x: u32);
}

#[no_mangle]
pub extern fn run() {
    // wrapping in an unsafe block
    // rust compiler can't provide mem safety for external functions
    unsafe {
        appendNumberToBody(42);
        alert(4);
    }
}
```

Whew! We've just gotten through the introductory lessons for working with `Rust`, `WebAssembly` and browser javascript!

---

## Further lessons ##

Contents:

    1. [wasm-js-interop](1_wasm-js-interop.md)

    Deeper dive into `wasm-bindgen`, the backend to `wasm-pack` build/compile layer. As well as, implementing webpack to bundle and serve our wasm modules.

    2. [rust-webpack-template](2_rust-webpack-template.md)

    Using the `rust-webpack` template provided to us by the Rust/WebAssembly team and how to implement it.