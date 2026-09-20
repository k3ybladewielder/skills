# Resource Grading and Quantities Subskill (`bend/tests/grade`)

## Overview
This subskill guides quantity typing (`&0`, `&1`, `&2`), affine resource tracking, and erased parameter compilation.

## Key Concepts
- **Quantities**:
  - `&0` (Erased): Value exists only at compile-time/proof-level; erased at runtime.
  - `&1` (Affine): Used at most once.
  - `&2` (Unrestricted / Data): Reusable and clonable.
- **Erased Binders (`-x: T`)**: Marked with a minus sign; cannot influence runtime execution.
- **Minimum Quantity (`a <&> b`)**: Combining quantities yields the most restrictive common upper bound.

## Example Pattern from `bend/tests/grade`
```python
import Base

# -proof is erased at runtime (&0), keeping execution zero-cost
def safe_div(num: U32, den: U32, -proof: {U32.is_eq(den, 0) == 0 : U32}) -> U32:
  U32.div(num, den)
```

## Agent Checklist
- Mark mathematical certificates and proofs as erased (`-`) to avoid runtime overhead.
- Ensure runtime code paths never depend on erased values.
