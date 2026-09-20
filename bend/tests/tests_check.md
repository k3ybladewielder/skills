# Type Checking and Linearity Subskill (`bend/tests/check`)

## Overview
This subskill guides validation against Bend's static type checker, affine linearity verifier, and kind consistency rules (`bend check`).

## Key Concepts
- **Linearity and Affine Usage**: Variables may be consumed at most once by default. Prefix with `+` for variables used multiple times (which must belong to kind `Data`).
- **Kind Rules**:
  - `Data` = `Kind(&2)`: copiable / duplicable types.
  - `Type` = `Kind(&1)`: non-copiable / single-use types (e.g., closures, linear channels).
- **Match Scrutinee Rule**: `match` must inspect a parameter or directly destructured field. Never match on expressions, computed results, or local variables.

## Example Pattern from `bend/tests/check`
```python
import Base

type Point is Data:
  PointItem{x: F32, y: F32}

# Valid affine reuse: +p is marked reusable
def scale_point(+p: Point, factor: F32) -> Point:
  match p:
    case PointItem{x, y}:
      new_x = F32.mul(x, factor)
      new_y = F32.mul(y, factor)
      PointItem{new_x, new_y}
```

## Agent Checklist
- Check all duplicate variable uses for `+` annotations.
- Keep match scrutinees strictly bound to formal parameters or immediate constructor extractions.
- Run `bend check <file>.bend` to verify type and kind consistency before executing.
