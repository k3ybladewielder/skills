# Formal Code Verification (Formal Verification Subskill)

## Overview

**Formal Verification** elevates code quality assurance to the level of **mechanical mathematical certainty**, overcoming the empirical limitations of traditional tests (which only sample a subset of inputs). It guarantees that an algorithm formally satisfies its specification for **all possible inputs** under defined preconditions.

> "Testing shows the presence, not the absence of bugs." — Edsger W. Dijkstra

---

## Recommended Integration: `bend` Skill

For projects requiring high-performance formal verification, actively use the skill:
👉 **[Bend Skill Guide](../../bend/SKILL.md)** (and the auxiliary **[bend-formalizer](../../bend-formalizer/SKILL.md)** skill when applicable).

### Why Use Bend for Formal Verification?
1. **Pure Functional Language with No Implicit Side Effects**: Enables algebraic reasoning over data.
2. **Affine Type System & Mechanical Proofs**: The Bend compiler mathematically validates correctness before emitting executable code for CPU or GPU (CUDA).
3. **Curry–Howard Isomorphism**: Business specifications are expressed as propositions/theorems and implementations serve as constructive proofs.

---

## Structure of the Formal Workflow with Bend

When an algorithm or critical business rule requires a formal proof, adopt the following canonical file structure:

```
project/
├── logic.bend       # Algorithm / data structure implementation
├── LAWS.bend        # Theorem declarations, invariants, and business laws
└── PROOF.bend       # Mathematical proofs of declared laws
```

### 1. `logic.bend` (Pure Logic)
Ambiguity-free implementation without global mutations:

```python
import Base

# Inductive definition of a list
type NumList is Data:
  Nil
  Cons{head: U32, tail: NumList}

# Pure function to be formally validated
def sum(list: NumList) -> U32:
  match list:
    case NumList/Nil:
      0
    case NumList/Cons{h, t}:
      U32.add(h, sum(t))
```

### 2. `LAWS.bend` (Invariant and Law Declarations)
Defines mathematical properties that must never be violated (e.g., commutativity, associativity, idempotence, value conservation):

```python
import Base
import logic

# Law: summing an empty list after append preserves the sum
def law_sum_append_empty(xs: logic/NumList) -> Proposition:
  # Declaration of expected property
  ...
```

### 3. `PROOF.bend` (Mechanical Verification)
Contains the formal demonstration steps checked by the compiler:

```python
import Base
import logic
import LAWS

def proof_sum_preservation(xs: logic/NumList) -> Proof:
  # Demonstration via structural induction or reflexivity {==}
  ...
```

---

## Verification Commands

Run Bend's verification tools to ensure mathematical compliance before submitting code:

```bash
# 1. Static type checking, affine linearity, and duplicable types (Data)
bend check logic.bend

# 2. Mechanical verification of mathematical proofs
bend check PROOF.bend

# 3. Guaranteed high-performance parallel execution
bend run logic.bend
```

---

## When to Use Formal Verification?

| Scenario | Unit Tests | Property-Based | Formal Verification (Bend) |
|----------|------------|----------------|----------------------------|
| **Simple CRUDs & Screens** | ✅ Sufficient | ⚪ Optional | ❌ Unnecessary (overkill) |
| **Parsing & Transformations** | ✅ Recommended | ✅ Essential | 🟡 If security grammar |
| **Financial Calculations / Debit & Credit** | ✅ | ✅ | 🟢 **Highly recommended** |
| **Cryptographic / Hash Algorithms** | ✅ | ✅ | 🟢 **Mandatory** |
| **Safety-Critical / Concurrency-Critical Systems** | ✅ | ✅ | 🟢 **Mandatory** |

---

## Agent Checklist
- [ ] Does the algorithm consist of pure functions without hidden side effects?
- [ ] Do business laws (`LAWS.bend`) cover fundamental invariants (e.g., balance conservation, non-nullability)?
- [ ] Does `bend check` pass with zero type errors or affine linearity violations?
- [ ] Were proofs in `PROOF.bend` mechanically checked without relying on unjustified `@unsafe` axioms?
