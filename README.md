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

## Components

* Rust Crates
* Lexer, Parser, and AST(Abstract Syntax Tree)
* Compiler
* VM (Virtual Machine)
* Import System
* Built-in Objects

## Overall Design

The overall design of RustPython follows the CPython strategy.

![GitHub Logo](/img/rustpython.png)

Stripped-off Dependancy Tree would look like this:

    rustpython (RustPython interactive shell)
    ├── rustpython-parser (lexer, parser and AST)
    ├── rustpython-compiler (compiler)
    │   ├── rustpython-bytecode (bytecode)
    │   └── rustpython-parser (lexer, parser and AST `lalrpop`)
    └── rustpython-vm (VM, compiler and built-in functions)
        ├── rustpython-bytecode (bytecode)
        ├── rustpython-compiler (compiler)
        └── rustpython-parser (lexer, parser and AST)

## Lexing, Parsing and AST

* A hand coded lexer to deal with indent and dedent of Python
  * Task: Convert Python source into tokens

* The parser is generated with [lalrpop](https://github.com/lalrpop/lalrpop)
  * Task: Convert tokens into an AST

* The AST (abstract syntax tree) nodes are Rust structs and enums

## Compiler and Bytecode

* The compiler turns Python syntax (AST) into bytecode
* CPython bytecode is not stable and varies wildly between versions
* Example bytecode

```python
import dis

  def f(a, b):
    return a + b + 3

if __name__ == "__main__":
  dis.dis(f.__code__)
```

```

3       0  LOAD_FAST          0 (a)
        2  LOAD_FAST          1 (b)
        4  BINARY_ADD
        6  LOAD_CONST         1 (3)
        8  BINARY_ADD
        10 RETURN_VALUE

```

* Idea: standardize this bytecode between Python implementations..?
