# Term Evaluation and Normalization Subskill (`bend/tests/eval`)

## Overview
This subskill guides runtime term reduction, normal-form computation, and operational semantics verification.

## Key Concepts
- **Pure Functional Reduction**: Expressions evaluate deterministically to Weak Head Normal Form (WHNF) or full normal form.
- **Strict vs Lazy Evaluation**: Arguments to primitive operators (`U32.add`, `F32.mul`) evaluate strictly, while algebraic constructors evaluate purely on demand.
- **Confluence and Determinism**: Pure Bend computations produce identical results regardless of parallel thread scheduling.

## Example Pattern from `bend/tests/eval`
```python
import Base

def fib(+n: U32) -> U32:
  match n:
    case 0: 0
    case 1: 1
    case _:
      a b = fib(U32.sub(n, 1)) fib(U32.sub(n, 2))
      U32.add(a, b)

def main() -> U32:
  fib(10)
```

## Agent Checklist
- Verify that definitions are terminating and pure.
- Check that branch conditions evaluate to concrete integers/data before pattern matching.
