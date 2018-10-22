## Using the rust-webpack template ##

While previously we've set up our own webpack project to handle rust and wasm, the Rust WebAssembly team also ships a template for it. All we have to do is npm run the template and install the deps.

lesson: [6](https://egghead.io/lessons/webpack-create-a-new-rust-webpack-project-using-the-rust-webpack-template)

> $ `npm init rust-webpack`
>
> $ `npm install`

_Please note that we've got to have `wasm-pack` already installed, if not just `cargo install wasm-pack`._

Next, start the project. Wait a while as the crate building takes a while. If there's any compilation errors with `wasm-bindgen`, just update your nightly builds

> $ `rustup update nightly`
> 
> $ `cargo update`
>
> $ `npm start`

So how does this deviate from our own implementation of wasm-js-interop? Well, the template setup(within `Cargo.toml`) defines custom Rust features like:

* console.error and panic hook
    ```
    # The `console_error_panic_hook` crate provides better debugging of panics by
    # logging them with `console.error`. This is great for development, but requires
    # all the `std::fmt` and `std::panicking` infrastructure, so isn't great for
    # code size when deploying.
    console_error_panic_hook = { version = "0.1.5", optional = true }
    ```
* some tiny allocator

    > No idea what an allocator is in this context...

    ```
    # `wee_alloc` is a tiny allocator for wasm that is only ~1K in code size
    # compared to the default allocator's ~10K. It is slower than the default
    # allocator, however.
    wee_alloc = { version = "0.4.2", optional = true }
    ```

When we go to our `lib.rs` entry file, we can see the previously mentioned features are configured. _In some way, I need to learn Rust for this..._

```rust
cfg_if! {
    // When the `console_error_panic_hook` feature is enabled, we can call the
    // `set_panic_hook` function to get better error messages if we ever panic.
    if #[cfg(feature = "console_error_panic_hook")] {
        extern crate console_error_panic_hook;
        use console_error_panic_hook::set_once as set_panic_hook;
    }
}

cfg_if! {
    // When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
    // allocator.
    if #[cfg(feature = "wee_alloc")] {
        extern crate wee_alloc;
        #[global_allocator]
        static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;
    }
}
```

---

In addition, this setup ships with wasm-pack webpack plugin that builds our library everytime our file changes.

```js
// webpack.config.js
const WasmPackPlugin = require("@wasm-tool/wasm-pack-plugin");

module.exports = {
    // ...
    plugins: [
        //...

        new WasmPackPlugin({
        crateDirectory: path.resolve(__dirname, "crate")
        }),
    ]
}
```

Let verify this works by changing our string greeting:

```rust
val.set_inner_html("Hello from Rust, WebAssembly, and Webpack!");

// to 

val.set_inner_html("Hello World");
```

Tada! Hot Reload :)

It's recommended to always use `rust-webpack` template to use best practices from the Rust/WebAssembly team.