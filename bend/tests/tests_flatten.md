# Pattern Flattening and Decision Trees Subskill (`bend/tests/flatten`)

## Overview
This subskill guides pattern matching compilation, decision tree flattening, and case analysis exhaustiveness.

## Key Concepts
- **Exhaustive Matching**: All constructors of an algebraic data type must be covered or handled by a fallback case (`case _:`).
- **Nested Pattern Flattening**: Deeply nested patterns (e.g., `case Con{Con{x, _}, _}:`) are compiled into flat single-level decision trees.
- **No Unreachable Branches**: Overlapping patterns are checked for redundancy and reachability.

## Example Pattern from `bend/tests/flatten`
```python
import Base

type Option<T> is Data:
  Some{val: T}
  None{}

def unwrap_or(opt: Option<U32>, default_val: U32) -> U32:
  match opt:
    case Some{v}: v
    case None{}: default_val
```

## Agent Checklist
- Always provide exhaustive `match` arms for every constructor or include `case _:`.
- Deconstruct arguments positionally rather than using named record fields.
