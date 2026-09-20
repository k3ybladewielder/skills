# Memory Paging and Mutable Arrays Subskill (`bend/tests/page`)

## Overview
This subskill guides single-owner array memory management, page allocation, and in-place updates (`a[i] <- v`).

## Key Concepts
- **Single-Owner Arrays**: Arrays are linear types that permit $O(1)$ in-place updates without garbage collector pauses.
- **Array Syntax**:
  - Creation: `[default_val : Type*N]` (fixed size $N$) or `[default_val : Type^D]` (power-of-two capacity $2^D$).
  - Reading: `val = arr[idx]`
  - In-place Write: `arr[idx] <- new_val`

## Example Pattern from `bend/tests/page`
```python
import Base

def update_scores(scores: [U32*4], idx: U32, new_score: U32) -> [U32*4]:
  scores[idx] <- new_score
```

## Agent Checklist
- Always maintain single-ownership of mutable array instances.
- Re-bind updated arrays to preserve linearity (`arr = arr[i] <- val`).
