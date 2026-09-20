# Termination and Structural Recursion Subskill (`bend/tests/halt`)

## Overview
This subskill guides totality checking, well-founded structural recursion, and infinite loop prevention in Bend.

## Key Concepts
- **Structural Recursion**: Recursive calls must strictly operate on structurally smaller subterms (e.g., tail of a list `t`, predecessor `p` of `1n+p`).
- **Guarded Termination**: Every branch of a recursive function must either reach a base case or strictly decrease its argument.
- **`@unsafe` Escape Hatch**: Non-structural recursive loops (e.g., event listeners or servers) must be explicitly annotated with `@unsafe`.

## Example Pattern from `bend/tests/halt`
```python
import Base

# Structurally decreasing on Nat
def count_down(n: Nat) -> Nat:
  match n:
    case 0n: 0n
    case 1n+p: count_down(p)

# Unbounded loop explicitly marked @unsafe
@unsafe
def event_loop(+state: U32) -> U32:
  event_loop(U32.add(state, 1))
```

## Agent Checklist
- Structure all domain functions to be structurally recursive so they pass `bend check`.
- Use `@unsafe` only for intentionally infinite processes (e.g., web servers, game loops).
