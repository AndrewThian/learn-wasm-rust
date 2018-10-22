## Debug a wasm module in Rust using console.log ##

Having some kind of debugging tool in our belt is extremely useful before writing a lot of code. In this lesson we build a println!()-style syntax using JavaScript’s console.log to be able to log out values in Rust.

lesson: [7](https://egghead.io/lessons/webpack-debug-a-webassembly-module-written-in-rust-using-console-log)

Super simple implementation, we want to define the `js_namespace` within our `lib.rs` and provide it as a macro to use in our `fn run()`

```rust
extern "C" {
    // ...

    #[wasm_bindgen(js_namespace = console)]
    fn log(msg: &str);
}

macro_rules! log {
    ($($t:tt)*) => (log(&format!($($t)*)))
}

#[wasm_bindgen]
pub fn run() {
    // ...

    log!("The {} is {}", "meaning of life", 42);

    // ...
}
```

Viola! We can print a console.log from within our Rust wasm module.