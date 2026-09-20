# Language Specifications and RFC Proposals Subskill (`bend/tests/rfc`)

## Overview
This subskill guides alignment with accepted Bend language specifications, RFCs, and syntax evolutions.

## Key Concepts
- **Bend 2 Paradigm**: Strict separation between data modeling (`is Data`), functions, proofs, and IO effects.
- **Removed Concepts from Bend 1 / HVM**: Do NOT use deprecated keywords (`run-rs`, `run-c`, `fold`, `bend`, `fork`, `u24`, `i24`, `open`, `if`, `switch`).
- **Standard Modern Idioms**: Direct assignment (`x = v`), `match` statements, `do` blocks, and explicit Base operators.

## Agent Checklist
- Always adhere to modern Bend 2 specifications from `~/.bend/`.
- Never generate legacy Bend 1 constructs.
