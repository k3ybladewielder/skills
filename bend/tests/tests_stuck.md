# Stuck Terms and Deadlock Diagnostics Subskill (`bend/tests/stuck`)

## Overview
This subskill guides diagnosing terms that fail to reduce to normal form, detecting pattern match incompleteness, and resolving runtime deadlocks.

## Key Concepts
- **Stuck Term Causes**:
  - Missing pattern match case for a constructor.
  - Linear resource deadlock (two branches waiting on mutually locked affine references).
  - Calling undefined or unannotated recursive cycles.
- **Diagnostic Output**: The Bend runtime prints the exact stuck redex and call-site stack trace.

## Agent Checklist
- Verify that every `match` is exhaustive across all ADT variants.
- Ensure linear channels and affine resources are consumed in consistent topological order.
