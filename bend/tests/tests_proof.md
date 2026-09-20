# Formal Proofs and Theorem Proving Subskill (`bend/tests/proof`)

## Overview
This subskill guides formal theorem specification, propositional equalities `{a == b : T}`, reflexivity proofs `{==}`, induction, rewrite tactics (`%e : P`), and refutations.

## Key Concepts
- **Laws (`law`)**: Specification stating invariants that must hold for all inputs.
  ```python
  law add_zero:
    for +x: Nat
    {Nat.add(x, 0n) == x : Nat}
  ```
- **Proofs (`def`)**: Exact proof implementations proving corresponding laws:
  - Base Case (Reflexivity): `{==}`
  - Step Case (Induction + Rewrite): `%proof : {goal_with_rewrite}`
- **Refutation of Impossible Branches**: Rewrite through discriminant function `disc(_)` returning `Empty` on contradictory branches.

## Example Pattern from `bend/tests/proof`
```python
import Base

law nat_add_zero:
  for +x: Nat
  {Nat.add(x, 0n) == x : Nat}

def nat_add_zero(x):
  match x:
    case 0n:
      {==}
    case 1n+p:
      %nat_add_zero(p) : {1n+Nat.add(p, 0n) == 1n+_ : Nat}
      {==}
```

## Agent Checklist
- Ensure every `law` declared in `LAWS.bend` has a matching `def` proof in `PROOF.bend`.
- Avoid annotating types on proof parameters in `def LAWS.name(x):`.
- Run `bend PROOF.bend` to ensure all terms check and zero open claims remain.
