# Compile-Time Evaluation and Metaprogramming Subskill (`bend/tests/comptime`)

## Overview
This subskill guides compile-time macro expansion, template instantiation (`def t(~g: A -> B, x: A)`), and static constant folding.

## Key Concepts
- **Templates**: Prefix parameter with `~` to mark it for compile-time monomorphization and inlining.
- **Static Partial Evaluation**: Fully known terms at compile-time are reduced to normal form before code generation.
- **Dependent Type Families**: Types computed statically based on value parameters.

## Example Pattern from `bend/tests/comptime`
```python
import Base

def apply_twice(~op: U32 -> U32, x: U32) -> U32:
  op(op(x))

def inc(n: U32) -> U32:
  U32.add(n, 1)

def main() -> U32:
  apply_twice(~inc, 10)
```

## Agent Checklist
- Use `~` for template arguments to ensure compile-time inlining.
- Avoid passing runtime-dynamic terms as template binders.
- Validate that template expansions terminate at compile time.
