# RustPython: a Python implementation in Rust

Rust is a relatively new programming language aimed as a safe competitor of C.
There are already attempts to write extension modules in rust and load them into CPython. A whole new approach would be to re-implement the Python language in rust.

This is what RustPython is about. The aim of the project is to create a Python interpreter written entirely in Rust. Until now we used many of the language features available in rust, such as vectors, hashmaps, iterators.

To implement standard library modules, we could just wrap existing rust crates. For example, this is how the `json` module is implemented.

## Why

* Rust is safer than C
  * In general Rust allows you to focus on the actual implementation of the library
* Learn Rust
* Learn Python intrnals
* Create a Python implementation which is more memory-safe.

## Internals

* Rust Crates
* Lexer, Parser, and AST(Abstract Syntax Tree)
* Compiler
* VM (Virtual Machine)
* Import System
* Built-in Objects

### Overall Design

The overall design of RustPython follows the CPython strategy.

![GitHub Logo](/img/rustpython.png)

Stripped-off Dependancy Tree would look like this:

    rustpython (RustPython interactive shell)
    ├── rustpython-parser (lexer, parser and AST)
    ├── rustpython-compiler (compiler)
    │   ├── rustpython-bytecode (bytecode)
    │   └── rustpython-parser (lexer, parser and AST)
    └── rustpython-vm (VM, compiler and built-in functions)
        ├── rustpython-bytecode (bytecode)
        ├── rustpython-compiler (compiler)
        └── rustpython-parser (lexer, parser and AST)
