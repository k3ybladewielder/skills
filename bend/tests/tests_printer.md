# Pretty-Printing and Desugaring Subskill (`bend/tests/printer`)

## Overview
This subskill guides term formatting, desugaring verification, and round-trip code printing.

## Key Concepts
- **Syntax Desugaring**:
  - `[a, b, c]` $\rightarrow$ `Con{a, Con{b, Con{c, Nil{}}}}`
  - `a && b` $\rightarrow$ boolean match
  - `do IO<T>:` $\rightarrow$ monadic `bind` and `pure` chains
- **Pretty-Printing Formats**: Human-readable canonical format ensuring lossless round-trip parsing.

## Example Pattern from `bend/tests/printer`
```python
import Base

# Sugared list literal:
def make_list() -> List<U32>:
  [1, 2, 3]

# Equivalent desugared form:
def make_list_desugared() -> List<U32>:
  Con{1, Con{2, Con{3, Nil{}}}}
```

## Agent Checklist
- Understand how syntactic sugar desugars to underlying constructors and functions.
- Format code cleanly following 2-space indentation conventions.
