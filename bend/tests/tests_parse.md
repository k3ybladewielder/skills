# Syntax Parsing and Grammar Rules Subskill (`bend/tests/parse`)

## Overview
This subskill guides lexical analysis, AST parsing rules, operator precedence, and literal grammar compliance in Bend 2.

## Key Concepts
- **Direct Assignment**: `x = v` (no `let` keyword).
- **Literals**:
  - `42` -> `U32`
  - `42n` -> `Nat`
  - `3.14` -> `F32`
  - `"string"` -> `String`
  - `'c'` -> `Char`
- **Operator Wrapping**: Infix operations require explicit typed wrapping `((a + b) : U32)` or direct `Base` calls `U32.add(a, b)`.

## Example Pattern from `bend/tests/parse`
```python
import Base

def parse_valid_statements(+a: U32, +b: U32) -> (U32 & U32):
  sum = U32.add(a, b)
  diff = U32.sub(a, b)
  (sum, diff)
```

## Agent Checklist
- Never use reserved keywords from other languages (like `let`, `var`, `function`).
- Keep constructor instantiations positional with braces: `Constructor{arg1, arg2}`.
