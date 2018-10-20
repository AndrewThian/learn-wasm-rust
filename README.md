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

## Lessons ##

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

### 1. Load a web-assembly function written in Rust and invoke from Javascript ###
