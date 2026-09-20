# Term Inspection and Goal Diagnostics Subskill (`bend/tests/show`)

## Overview
This subskill guides interactive proof hole discovery, type inspection, and term goal debugging (`?name`, `?TODO`).

## Key Concepts
- **Hole Discovery (`?name`)**: Inserting `?name` in place of an expression causes the checker to print the expected type, context hypotheses, and free variables.
- **Incomplete Proofs (`?TODO`)**: Allows leaving open claims during development without halting the checker.
- **Type Inspection**: Using `{x : T}` to force the compiler to assert and show the evaluated type of an expression.

## Example Pattern from `bend/tests/show`
```python
import Base

def inspect_goal(n: Nat) -> Nat:
  match n:
    case 0n: 0n
    case 1n+p:
      # Inserting a hole prints the expected goal for the inductive step
      ?step_hole
```

## Agent Checklist
- Use `?name` whenever a complex type or proof goal is unclear.
- Replace all holes (`?name` / `?TODO`) before finalizing proofs.
