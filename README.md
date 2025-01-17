# witc

> **Warning**
> This is an early-stage project

A compiler generates code for `*.wit` files.

### Overview

To understand this project, I will show you what's `*.wit` stand for first. The story starts by passing a string as an argument to function in the wasm instance, if you do so, you will find out that wasm has no type called `string`. You will figure out that you only need to encode the string as a pair of `i32`, which means `(i32, i32)` and one for address, one for string length. However, the address valid out of an instance will not be valid in that instance. Then you found runtime(e.g. wasmedge, wasmtime) can operate the memory of instances, write data to somewhere in the instance, and use that address, problem solved!

Quickly, your program grows, and now you manage tons of mappings.

```rust
fn foo(s: String, s2: String) -> String
// <->
fn foo(s_addr: i32, s_size: i32, s2_addr: i32, s2_size: i32) -> (i32, i32)
```

The thing is a bit out of control, not to say compound types like **structure**, **enum**, and **list**. In this sense, **wit** stands for one source code that is reusable for multi-target, and multi-direction and **witc** does code generation and manages ABI and memory operations. Thus, you can import/export types or functions from instance or runtime.

### Usage

#### Rust example

```rust
#![feature(wasm_abi)]

wasmedge_witc::wit_instance_import!("../xxx.wit");

#[no_mangle]
pub unsafe extern "wasm" fn start() -> u32 {
    let _s = exchange("Hello".to_string());
    // `exchange` is defined in xxx.wit & host
    // with wit type: `string -> string`
    return 0;
}
```

#### CLI

Conceptual command

```sh
witc instance import xxx.wit
witc runtime export xxx.wit
```

### Development

This project use GHC 9.2.5, since hls haven't supported this version offically, you can compile local hls for development.

```shell
ghcup --verbose compile hls --cabal-update --ghc 9.2.5 --git-describe-version --git-ref aeb57a8eb56964c8666d7cd05b6ba46d531de7c7 -- --ghc-options='+RTS -M2G -RTS'
```

### Why witc?

You might wonder why you need `witc` since `wit-bindgen` already exists.
Although `wit-bindgen` is good, it is currently in active development.
Additionally, the Component Model and Canonical ABI change frequently with large updates.
We create `witc` to serve as a middle project to wait for `wit-bindgen` to become stable, and at that point, we will contribute to `wit-bindgen`.
With `witc`, it increases the diversity in wit related toochain.
For these reasons, we will only support a small number of features in `witc`, ensuring that the basic demos will work.
