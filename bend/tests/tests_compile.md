# Compilation and Code Generation Subskill (`bend/tests/compile`)

## Overview
This subskill guides compiler lowering, target translation, and C/CUDA code generation verification (`bend -c <file>.bend`).

## Key Concepts
- **Lowering Pipeline**: High-level typed Bend AST $\rightarrow$ Core Lambda/Interaction Net Calculus $\rightarrow$ Native C/CUDA targets.
- **Tail-Call Optimization**: Recursive tail-calls are compiled into fast native loop jumps.
- **Independent Parallel Calls**: Parallel dispatches `a b = f(x) g(y)` are compiled into concurrent execution threads.

## Example Pattern from `bend/tests/compile`
```python
import Base

def worker(+id: U32, +data: U32) -> U32:
  U32.add(id, data)

def parallel_pipeline(+val: U32) -> (U32 & U32):
  left right = worker(1, val) worker(2, val)
  (left, right)
```

## Agent Checklist
- Test native compilation with `bend -c <file>.bend`.
- Verify that parallel bindings use decoupled arguments with no cross-dependencies.
- Ensure that recursion terminates structurally so the compiler can generate efficient loop bodies.
