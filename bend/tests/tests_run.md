# End-to-End Execution and Runtime Backends Subskill (`bend/tests/run`)

## Overview
This subskill guides executing programs on Bend's native runtime backends (CPU interpreter, native C binary, and parallel CUDA GPU).

## Key Concepts
- **Execution Commands**:
  - `bend <file>.bend`: Fast interpreter run.
  - `bend -c <file>.bend`: Compile and run as native C executable.
  - `bend -cu <file>.bend`: Compile and run with CUDA GPU acceleration.
- **Entrypoint**: `def main() -> T:` serves as the program entrypoint.

## Example Pattern from `bend/tests/run`
```python
import Base

def main() -> U32:
  U32.add(40, 2)
```

## Agent Checklist
- Test programs end-to-end with `bend <file>.bend`.
- For compute-intensive tree reductions and shaders, benchmark with `bend -cu <file>.bend`.
