# Monadic State and Stateful Threading Subskill (`bend/tests/state`)

## Overview
This subskill guides pure state threading, mutable cell abstractions, and stateful monads in Bend.

## Key Concepts
- **Pure State Threading**: Passing state explicitly as a linear tuple `(state, result)`.
- **State Monad (`State<S, A>`)**: Encapsulates state transitions via `do` blocks and bind chaining.
- **Deterministic Concurrency**: Multi-state threads synchronize purely without race conditions.

## Example Pattern from `bend/tests/state`
```python
import Base

def counter_step(+current: U32, inc_by: U32) -> (U32 & U32):
  new_val = U32.add(current, inc_by)
  (new_val, new_val)
```

## Agent Checklist
- Thread state linearly through functional steps without global side effects.
