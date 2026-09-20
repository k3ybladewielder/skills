# Register Allocation and Variable Liveness Subskill (`bend/tests/reg`)

## Overview
This subskill guides backend register allocation, variable lifecycle management, and stackless frame generation.

## Key Concepts
- **Linear Binder Lifecycles**: Each bound variable has an exact lifetime ending upon consumption.
- **Register Reuse**: Once a linear value is consumed, its underlying register/slot is immediately reused.
- **Affine Copy Nodes**: Duplicated variables (`+x`) create lightweight fan-out nodes in the interaction net runtime.

## Example Pattern from `bend/tests/reg`
```python
import Base

def register_reuse_chain(+val: U32) -> U32:
  step1 = U32.add(val, 1)
  step2 = U32.mul(step1, 2)
  step3 = U32.sub(step2, 3)
  step3
```

## Agent Checklist
- Structure linear computational pipelines to minimize peak concurrent variable live-ranges.
