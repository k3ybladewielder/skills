# Base Library Subskill (`bend/tests/base`)

## Overview
This subskill guides development and testing of Bend's built-in standard library (`Base`), verifying core algebraic types, foundational arithmetic, and fundamental operations.

## Key Concepts
- **Standard Types**: `Bool` (`True{}`, `False{}`), `Nat` (`0n`, `1n+p`), `U32`, `F32`, `String`, `List<T>` (`Con{h, t}`, `Nil{}`).
- **Built-in Functions**:
  - `U32`: `U32.add`, `U32.sub`, `U32.mul`, `U32.div`, `U32.mod`, `U32.is_eq`, `U32.is_lt`.
  - `F32`: `F32.add`, `F32.sub`, `F32.mul`, `F32.div`, `F32.abs`, `F32.is_eq`, `F32.is_lt`.
  - `Nat`: Peano arithmetic, induction steps `1n+p`.
- **String and List Operations**: `List.map`, `List.fold`, string concatenation (`++`), list sugar `[a, b, c]`.

## Example from `bend/tests/base`
```python
import Base

def test_arithmetic(+x: U32, +y: U32) -> U32:
  sum = U32.add(x, y)
  prod = U32.mul(sum, 2)
  U32.div(prod, 2)

def test_list_ops(xs: List<U32>) -> U32:
  match xs:
    case Nil{}: 0
    case Con{h, t}: U32.add(h, test_list_ops(t))
```

## Agent Checklist
- Always import `Base`.
- Avoid redefining types already present in `Base` (such as `Bool` or `List`).
- Prefer typed `Base` module calls (`U32.add`, `F32.mul`) over bare infix operators.
