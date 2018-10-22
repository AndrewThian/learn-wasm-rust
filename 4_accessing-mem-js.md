## Access WebAssembly Memory Directly from JavaScript ##

While js is a garbage-collected heap, WebAssembly is a linear mem space. We can use Javascript's ArrayBuffer to read and write to WebAssembly's mem space.

lesson: [8](https://egghead.io/lessons/webpack-access-webassembly-memory-directly-from-javascript)

Each WebAssembly module has linear memory. From js, we can freely read and write to it. lets write a small image library in Rust. We want to define structs for color and image in our Rust src file

```rust
// lib.rs
#[wasm_bindgen]
pub struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

#[wasm_bindgen]
pub struct Image {
    pixels: Vec<Color>,
}
```

Then, define methods we want to export

```rust
#[wasm_bindgen]
impl Image {
    pub fn new() -> Image {
        let color1 = Color {
            red: 255,
            green: 0,
            blue: 0,
        };

        let color2 = Color {
            red: 60,
            green: 70,
            blue: 90,
        };

        let pixels = vec![color1, color2];
        Image {
            pixels
        }
    }

    pub fn pixels_ptr(&self) -> *const Color {
        self.pixels.as_ptr()
    }
}
```

Then within our `app.js`, we want to import the `Image` from our web-assembly module

```js
// app.js
import { Image } from "../crate/pkg/rust_webpack"

const image = Image.new()
const pixelsPointer = image.pixels_ptr();
// basically, we want to make use of the unsigned 8 bit integer array to
// work with the memory space return from the web-assembly module.
const pixels = new Uint8Array(memory.buffer, pixelsPointer, 6);
console.log(pixels);
```

> need to read more about memory address and types within javascript and rust.

Next, we want to read the values(access the linear memory of our WebAssembly module) directly from javascript. We can use this method to draw a canvas from a game loop managaed in our Rust code.

```js
function numToHex(value) {
    const hex = value.toString(16);
    return hex.length === 1 ? `0${hex}` : hex    
}

function drawPixel(x, y, color) {
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = `#${numToHex(color[0])}${numToHex(color[1])}${numToHex(color[2])}`;
    ctx.fillRect(x, y, 100, 100);
}

const canvas = document.createElement("canvas")
document.body.appendChild(canvas)

drawPixel(0, 0, pixels.slice(0, 3))
drawPixel(100, 0, pixels.slice(3, 6))
```

However, we have to be aware that WebAssembly's mem can be accessed from Javascript but not the other way around. WebAssembly doesn't have direct access to Javascript values. However, we can work around this by storing a js value inside the WebAssembly memory then use inside Rust code.

---

## Final code ##

```rust
// ./src/lib.rs
#[wasm_bindgen]
pub struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

#[wasm_bindgen]
pub struct Image {
    pixels: Vec<Color>,
}

#[wasm_bindgen]
impl Image {
    pub fn new() -> Image {
        let color1 = Color {
            red: 255,
            green: 0,
            blue: 0,
        };

        let color2 = Color {
            red: 60,
            green: 70,
            blue: 90,
        };

        let pixels = vec![color1, color2];
        Image {
            pixels
        }
    }

    pub fn pixels_ptr(&self) -> *const Color {
        self.pixels.as_ptr()
    }
}
```

```js
// ./js/app.js
import { memory } from "../crate/pkg/rust_webpack_bg"
import { Image } from "../crate/pkg/rust_webpack"

console.log(memory)

const image = Image.new()
const pixelsPointer = image.pixels_ptr();
const pixels = new Uint8Array(memory.buffer, pixelsPointer, 6);
console.log(pixels);

function numToHex(value) {
    const hex = value.toString(16);
    return hex.length === 1 ? `0${hex}` : hex    
}

function drawPixel(x, y, color) {
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = `#${numToHex(color[0])}${numToHex(color[1])}${numToHex(color[2])}`;
    ctx.fillRect(x, y, 100, 100);
}

const canvas = document.createElement("canvas")
document.body.appendChild(canvas)

drawPixel(0, 0, pixels.slice(0, 3))
drawPixel(100, 0, pixels.slice(3, 6))
```