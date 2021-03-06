# RustPython: a Python implementation in Rust

_This document is based on the talk [RustPython: a Python implementation in Rust Building a Python 3 interpreter in Rust](https://www.youtube.com/watch?v=nJDY9ASuiLc) by Windel Bouwman([@windelbouwman](https://github.com/windelbouwman)) and Shing Lyu([@shinglyu](https://github.com/shinglyu)) at [FOSDEM](https://fosdem.org) and the Lightning Talk [RustPython: a Python implementation in Rust](http://lukas-prokop.at/talks/pygraz-rustpython) by Lukas Prokop([@meisterluk](https://github.com/meisterluk)) at [pyGraz](https://pygraz.org/)._

Rust is a relatively new programming language aimed as a safe competitor of C.
There are already attempts to write extension modules in rust and load them into CPython. A whole new approach would be to re-implement the Python language in rust.

This is what RustPython is about. The aim of the project is to create a Python interpreter written entirely in Rust. Until now we used many of the language features available in rust, such as vectors, hashmaps, iterators.

To implement standard library modules, we could just wrap existing rust crates. For example, this is how the `json` module is implemented.

## Why

- Rust is safer than C
  - In general Rust allows you to focus on the actual implementation of the library
- Learn Rust
- Learn Python intrnals
- Create a Python implementation which is more memory-safe.

## Components

- Rust Crates
- Lexer, Parser, and Abstract Syntax Tree (AST)
- Compiler
- Virtual Machine (VM)
- Import System
- Built-in Objects

## Overall Design

The CPython Design Strategy

![CPython](/img/overall_design.png)

The overall design of RustPython follows the CPython strategy.

![RustPython](/img/rustpython.png)

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

| Crates                                                   |                                                   |
| -------------------------------------------------------- | ------------------------------------------------- |
| [rustpython-parser](https://crates.io/crates/rustpython) | Lexer, Parser and AST                             |
| [rustpython-vm](https://crates.io/crates/rustpython-vm)  | VM, compiler and built-in functions               |
| [rustpython](https://crates.io/crates/rustpython)        | Using above crates to create an interactive shell |

## Lexing, Parsing and AST

- A hand coded lexer to deal with indent and dedent of Python
  - The lexer converts Python source into tokens
- The parser is generated with [lalrpop](https://github.com/lalrpop/lalrpop)
  - The parser converts tokens into an [AST (Abstract Syntax Tree)](https://en.wikipedia.org/wiki/Abstract_syntax_tree)
- The AST nodes are Rust structs and enums

## Compiler and Bytecode

- The compiler turns Python syntax (AST) into bytecode
- CPython bytecode is not stable and varies wildly between versions
- Example bytecode

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

- Idea: standardize this bytecode between Python implementations..?

## Virtual Machine (VM)

- A fetch and execute loop
- Fetch a bytecode
- lookup in a `match` statement
- perform the operation
  - If the bytecode is an `import` instruction invoke the whole loop again.

```rust

  match &instruction {
    bytecode::Instruction::LoadConst { ref value } => {
      let obj = self.unwrap.constant(vm, value);
      self.push_value(obj);
      Ok(None)
  }
  bytecode::Instruction::Import {
    ref name,
    ref symbol,
  } => self.import(vm, name, symbol),

```

## Object Model

- In CPython, there is reference counting
- The type `PyObjectRef` ([`rustpython_vm::pyobject::PyObjectRef`](https://rustpython.github.io/website/rustpython_vm/pyobject/type.PyObjectRef.html)) is a reference count to the actual Python object
- Use Rust `Rc`([`std::rc::Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html)) and `RefCell`([`std::cell::RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html)) to do reference counting of Python objects
- The struct `PyObject` ([`rustpython_vm::pyobject::PyObject`](https://rustpython.github.io/website/rustpython_vm/pyobject/struct.PyObject.html)) has a `type` which is a `PyObjectRef` ([`rustpython_vm::pyobject::PyObjectRef`](https://rustpython.github.io/website/rustpython_vm/pyobject/type.PyObjectRef.html)). And, each object has a dictionary in the form of a `HashMap`([`std::collections::HashMap`](https://doc.rust-lang.org/std/collections/struct.HashMap.html)), which is very dynamic and fast. So, you can easily set and get attributes at will.
- For certain Python types, we need to internally store a Rust payload; for instance
  - Python `String`([`str`](https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str)) is backed by a Rust [`String`](https://doc.rust-lang.org/std/string/struct.String.html)
  - Python `Float` is backed by a Rust [`f64`](https://doc.rust-lang.org/std/primitive.f64.html)
  - Python `Integer` is backed by Rust [`BigInt`](https://crates.io/crates/num-bigint) - arbitary integer precision!

```rust

pub type PyRef<T> = Rc<RefCell<T>>;
pub type PyObjectRef = PyRef<PyObject>;

pub struct PyObject {
  pub kind: PyObjectKind,
  pub typ: Optic <PyObjectRef>,
  pub dict: HashMap<String, PyObjectRef>,
}

pub enum PyObjectKind {
  String {
    value: String
  },
  Integer {
    value: Biglnt,
  },
  Float {
    value: f64,
  }
  Complex {
    value: Complex64,
  },
  Bytes {
    value: Vec<u8>,
  },

```

## Builtin functions

- Builtin Python functions are implemented in Rust

```rust

fn buitin_all(vm: &mut VirtualMachine, args: PyFuncArgs) -> PyResult {
  arg_check!(vm, args, required = ((iterable, None)]);
  let items = vm.extract_elements(iterable)?;

  for item in items {
    let result = objbool::boolval(vm, item)?;
    if I result {
      return Ok(vm.new_bool(false));
    }
  }

  Ok(vm.new_bool(true))
}

```

- `PyFuncArgs`([`rustpython_vm::function::PyFuncArgs`](https://rustpython.github.io/website/rustpython_vm/function/struct.PyFuncArgs.html)) - Contains all the positional arguments and keyword arguments.
- `arg_check`([`rustpython_vm::arg_check`](https://rustpython.github.io/website/rustpython_vm/macro.arg_check.html)) - Macro that checks the types of the arguments passed into the function.
