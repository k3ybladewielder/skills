# Language Specification and Invariants Subskill (`bend/tests/spec`)

## Overview
This subskill guides compliance with Bend's core operational semantics, substitution lemmas, and formal language invariants.

## Key Concepts
- **Type Soundness**: Well-typed programs never get stuck on defined constructors and total functions.
- **Substitution Integrity**: Substituting equal terms preserves well-typedness and operational equivalence.
- **Affine Safety**: No linear memory cell or affine resource is ever accessed concurrently or copied illegally.

## Agent Checklist
- Adhere strictly to the operational semantics defined in `paper/BendTT.pdf` and `paper/BendRT.pdf`.
