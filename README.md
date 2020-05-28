# RustPython: a Python implementation in Rust

Rust is a relatively new programming language aimed as a safe competitor of C.
There are already attempts to write extension modules in rust and load them into CPython. A whole new approach would be to re-implement the Python language in rust.

This is what RustPython is about. RustPython is a relatively new project. The aim of the project is to create a Python interpreter written entirely in Rust. Until now we used many of the language features available in rust, such as vectors, hashmaps, iterators. 

To implement standard library modules, we could just wrap existing rust crates. This is what we did with the `json` module for example.

## Why

* Rust is safer than C
  * In general Rust allows you to focus on the actual implementation of the library
* Learn Rust
* Learn Python intrnals
* Create a Python implementation which is more memory safe.
