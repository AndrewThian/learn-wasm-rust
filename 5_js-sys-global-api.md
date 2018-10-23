## `js-sys` to invoke global APIs in JS environment ##

`js-sys` offers bindings to all the global APIs available in every JS environment as defined by the ECMAScript standard. Here we will install the `js-sys` dependency and invoke the Date(hour/minutes) from Rust and display into the browser.

Lesson: [9](https://egghead.io/lessons/webpack-use-the-js-sys-crate-to-invoke-global-apis-available-in-any-javascript-environment)

Before anything, we want a clean template from `rust-webpack`.

> $ `npm init rust-wepback`

If we face any errors, we should update our `Cargo.toml` file directly.

> $ `rustup update`
>
> $ `cargo update`

Also, if running `npm start` from `rust-webpack` template isn't working, we can generate the package directly with

> $ `wasm-pack --verbose build`

Then build again.

Next, we want to define the dependency within our `Cargo.toml` file

```toml
# ./target/Cargo.toml

[dependencies]
cfg-if = "0.1.5"
wasm-bindgen = "0.2.25"
js-sys = "0.2"
```

Then we have to define the crate dependency in the file we're using `lib.rs`

```rust
#![feature(use_extern_macros)]

#[macro_use]

// ...

extern crate js_sys;

use wasm_bindgen::prelude::*;
```

Now we'll be able to use the global `js_sys` API in our Rust functions

```rust
#[wasm_bindgen]
pub fn run() {
    let now = js_sys::Date::now();
    let now_date = js_sys::Date::new(&JsValue::from_f64(now));
    let val = document.createElement("p");
    val.set_inner_html(&format!(
        "Hello from Rust, it's {}:{}",
        now_date.get_hours(),
        now_date.get_minutes(),
    ));
    document.body().append_child(val);
}
```

Very simple to implement!

> I'm still super confused by the bindings between rust and web-assembly