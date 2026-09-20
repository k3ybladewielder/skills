# IO Monad and Concurrency Subskill (`bend/tests/io`)

## Overview
This subskill guides side-effect encapsulation via the `IO` monad, asynchronous tasks, filesystem operations, and networking.

## Key Concepts
- **Pure Effect Encapsulation**: Effects are values of type `IO(T)` executed only inside the runtime IO driver.
- **Do-Notation for IO**:
  ```python
  do IO<Unit>:
    IO.print("Message")
    return Unit{}
  ```
- **Concurrency & Sockets**: Multi-threaded TCP servers, HTTP clients, and file streaming without global state races.

## Example Pattern from `bend/tests/io`
```python
import Base

def greet_user(name: String) -> IO(Unit):
  do IO<Unit>:
    IO.print("Hello, ")
    IO.print(name)
    return Unit{}

def main() -> IO(Unit):
  greet_user("Bend Developer")
```

## Agent Checklist
- Enclose all effectful calls (print, read, socket) inside `do IO<T>:` blocks.
- Keep business logic pure and separated from IO handling.
