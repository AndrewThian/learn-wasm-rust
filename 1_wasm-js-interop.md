## Set up `wasm-bindgen` for easy Rust/Javascript Interop ##

web-assembly programs operate on a limited set of value types. Due to this, the functions bridging between js and rust only allow for primitive numeric types(integer and floats)

```html
<script>
    const appendStringToBody = string => {
        const text = createTextNode("appendStringToBody", string)
        document.body.appendChild(text)
    }

    const importObject = {
        env: {
            appendStringToBody: appendStringToBody
        }
    }

    WebAssembly.instantiateStreaming(fetch("utils.gc.wasm"), importObject)
</script>
```

> We're exporting/importing a js function into Rust's working environment, then calling it in our `run()` function.
>
> However, this appends numbers instead of a `Hello World` string.

```rust
extern {
    fn appendStringToBody(s: &str);
}

pub extern fn run() {
    unsafe {
        appendStringToBody("Hello World");
    }
}
```

Reasons being the accepts for only primitive numeric types. A work around would be to directly read or write to WebAssembly's memory using Javascript. It can be quite clumbersome to write and access memory directly.

That's why `wasm-bindgen` was created. The framework makes it possible to write idiomatic Rust function signatures that map to idiomatic Javascript functions automatically.

First, we want to write our javascript code in the `index.html`

```html
<script>
        const createTextNode = (functionName, value) => {
            return document.createTextNode(`~// #${functionName} result: ${value} //~`)
        }

        const appendNumberToBody = number => {
            const text = createTextNode("appendNumberToBody", number)
            document.body.appendChild(text)
        }

        const appendStringToBody = string => {
            const text = createTextNode("appendStringToBody", string)
            document.body.appendChild(text)
        }

        const importObject = {
            env: {
                appendNumberToBody: appendNumberToBody,
                appendStringToBody: appendStringToBody
            }
        }

        WebAssembly
            .instantiateStreaming(fetch("utils.gc.wasm"), importObject)
            .then(wasmModule => {
                wasmModule.instance.exports.run();
            })
    </script>
```

Then, we'll go into our Rust environment to add `wasm-bindgen` as our dependency.

```toml
[dependencies]
wasm-bindgen = "0.2"
```

Next, use the external crate declaration to link `wasm-bindgen` to the new library. After that, use the declaration to make all functions from `wasm-bindgen` preload.

> Tbh, I have no clue what I'm doing here 'cause I don't understand Rust's syntax.

```rust
#![feature(use_extern_macros)]

extern crate wasm_bindgen;

use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn appendStringToBody(s: &str);
}

#[wasm_bindgen]
pub fn run() {
    appendStringToBody("Hello World");
}
```

Then, run `wasm-pack` to build our library! It generates a module with a package.json and a `utils_bg.wasm` file.

> $ `wasm-pack build`

_**One gotcha tho!** the file includes some import features not yet implemented in our modern browsers, thus we need to use a js bundler(we'll be using webpack)._

---

### Webpack configuration ###

First we need webpack(initialize npm project and install `webpack/dev-server`)

> $ `npm init -y`
> 
> $ `npm i -D webpack webpack-dev-server webpack-cli`

Set up a simple `webpack.config.js` to configure the webpack-dev-server in the root project folder

```js
// ./webpack.config.js
const path = require("path")

module.exports = {
    entry: "./index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "index.js"
    },
    mode: "development"
};
```

Import `index.js` into the `index.html` as a script tag to import the dynamic module that's running our web-assembly module.

```html
<script src="./index.js"></script>
```

Next we need to tell our Rust `wasm-bindgen` implementation to import our javascript module and recompile the code with `$ wasm-pack build`

```rust
#[wasm_bindgen(module = "../domUtils")]
extern {
    fn appendStringToBody(s: &str);
}
```

Finally, when we run our `webpack-dev-server` we can see that our `#appendStringToBody` function is properly displaying the string we've passed into our Rust code.

> I'm so lost at this point.