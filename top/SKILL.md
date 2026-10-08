---
name: theorem-oriented-programming
description: "Applies the Curry–Howard isomorphism to software development: turns requirements into theorems, types into propositions, and implementations into verifiable proofs. Use when designing, implementing, reviewing, or debugging code that must be correct by construction."
---

# Theorem-Oriented Programming

## Objective

Use this skill to treat every program as a theorem and all code as the proof of that theorem. Development should start from an explicit specification, express it in the type system, and build an implementation whose correctness can be verified by the compiler, a formal prover, or clearly delimited evidence.

**Central rule:** do not write or modify functional code before formulating the theorem, contract, or property that the code must prove. When the language cannot express the entire property, record what was encoded in the type and which obligations remain outside the verifier; do not present partial evidence as a complete proof.

## When to use

Activate this skill:

- when the user requests theorem-oriented development, type-driven programming, or mentions the Curry–Howard isomorphism;
- when designing, implementing, refactoring, reviewing, or debugging code whose contract, invariants, or business rules need to be explicit;
- in critical, high-reliability, or formally verifiable systems;
- when using ADTs, dependent types, strongly typed languages, theorem provers, or effect systems;
- when modeling failures, side effects, and external boundaries with `Option`/`Maybe`, `Either`/`Result`, monads, or effect systems;
- in Haskell, Idris, Agda, Coq, Lean, Rust, and similar languages, as well as strict TypeScript — always distinguishing what the compiler actually verifies from what requires additional validation.

## Common pitfalls

- **Confusing compilation with a complete proof:** a successful type check proves only the properties represented by the type system. It does not automatically demonstrate business rules, termination, security, availability, correctness of external data, or concurrent behavior.
- **Using weak or “stringly typed” types:** representing concepts such as `Email`, `Money`, `UserId`, or machine states only with `String`, `Int`, `Object`, or `Any` produces weak propositions. Prefer domain types, wrappers, ADTs, and validated constructors.
- **Using escape hatches as proof:** `unsafe`, forced coercions such as `as`, `@ts-ignore`, reflection that bypasses the verifier, and equivalents hide obligations. Isolate any unavoidable assumption and declare it unproved.
- **Leaving failures implicit:** `null`, undeclared exceptions, and `unwrap` make the contract smaller than the signature suggests. Model possible results in the type and handle each case explicitly.
- **Treating effects as a mathematical error:** effects do not need to be eliminated, but they must appear in the contract or be isolated at a boundary. Use monads or effect systems when the language provides them; otherwise, use explicit interfaces, adapters, and I/O contracts.
- **Confusing totality with the absence of ongoing systems:** a mathematical function must have defined domain and behavior for all expected cases. Services, event loops, and streams may run indefinitely; model them as productive/coinductive behavior or as effects with progress guarantees, rather than falsely claiming that they terminate.
- **Producing vacuously true proofs:** an empty type, an impossible precondition, or an unreachable branch invented merely to satisfy the compiler does not solve a real requirement. Verify that the premises are reachable and that valid inputs exist.
- **Confusing tests with universal proof:** tests, including property-based tests, provide executable evidence about generated cases. When the property can be formalized, keep the static/formal proof separate from that evidence.

## Foundations: the Curry–Howard isomorphism

The Curry–Howard isomorphism establishes a correspondence between logic and computation:

| Logic | Type theory and programming |
| --- | --- |
| Proposition `P` | Type `P` |
| Proof of `P` | Term/value of type `P` |
| `A → B` (implication) | Function that receives `A` and produces `B` |
| `A ∧ B` (conjunction) | Product, tuple, or record `(A, B)` |
| `A ∨ B` (disjunction) | Sum, discriminated union, or variant type |
| `True` | Unit type `Unit` and its only value |
| `False` | Empty type `Empty`/`Void`, with no values |
| `∀ x. P(x)` (for all) | Dependent or polymorphic function that works for any `x` |
| `∃ x. P(x)` (there exists) | Dependent pair containing a witness `x` and a proof of `P(x)` |
| `x = y` | Equality/identity type with a proof that `x` and `y` are equal |
| Introduction rule | Constructor of a type or creation of a function/value |
| Elimination rule | Pattern matching, function application, or eliminator |
| Induction | Structural recursion over an inductive type |
| Proof normalization/reduction | Evaluation and reduction of the program |

Thus, a signature such as `A -> B` is not merely an interface: it states that, given a proof of `A`, there is a program capable of constructing a proof of `B`. A function of this type is the proof of the implication itself.

### Program-to-proof correspondence

- **Types are propositions:** choose types that express the domain rules, their preconditions, and their valid states.
- **Programs are proofs:** each constructor, pattern-matching branch, and function call must correspond to a valid derivation rule.
- **Evaluation is reduction:** executing the program reduces the proof; pure functions should preserve the specified meaning during this reduction.
- **Data is evidence:** a value of a rich type carries evidence that it satisfies that contract.
- **Inductive types are case definitions:** constructors enumerate valid forms, and eliminators require each relevant form to be handled.
- **Polymorphism represents generality:** a universal function cannot depend on details that its type does not provide; preserve this parametricity.
- **Refinement is incremental proof:** transform a broad requirement into smaller lemmas until each obligation has a verifiable implementation.

The correspondence is most direct in pure typed calculi, dependent languages, and provers such as Lean, Coq, Agda, and Idris. In conventional languages, the verifier proves only the properties that the type system actually expresses. Compilation does not automatically prove business rules, termination, absence of external failures, concurrent behavior, or the correctness of data in production.

## Mandatory principles

1. **Specification before implementation:** write the theorem, premises, conclusion, and invariants before the function body.
2. **Strong and specific types:** prefer algebraic types, `newtypes`, wrappers, refined types, units of measure, and discriminated unions over `String`, `Int`, `Any`, or objects without a contract.
3. **Invalid states unrepresentable:** construct valid values through safe constructors or smart constructors; validate untrusted inputs at the boundary.
4. **Total functions when the domain requires totality:** cover all cases and provide a termination argument. If an operation can fail, model the failure in `Option`/`Maybe`, `Either`/`Result`, or an effect type.
5. **Explicit and isolated effects:** I/O, mutability, exceptions, time, randomness, concurrency, and network access are part of the theorem. Model them in the type or isolate them in adapters with verifiable contracts.
6. **Proofs without unverified shortcuts:** do not use `unsafe`, `any`, forced coercions, `as`, `unwrap`/`expect`, `panic`, `sorry`, `admit`, `@ts-ignore`, or equivalents to hide an obligation. If a boundary requires an escape hatch, isolate it, justify it, and mark the property as an unproved assumption.
7. **Non-vacuously true proof:** verify that the premises are reachable and that the input type has valid inhabitants. An empty type, an impossible precondition, or an ignored exception does not constitute a useful solution.
8. **Separation between proof and evidence:** tests, simulations, and runtime validation provide evidence about examples and boundaries; they do not replace a universal proof when the property can be formalized.
9. **Traceability:** each relevant piece of code must be linked to a specific premise, conclusion, invariant, or lemma.

## Development process

### 1. Formulate the theorem

Convert the requirement into a verifiable statement. Record at least:

```text
Theorem:
  For every valid input ..., the program produces ...

Premises:
  - input conditions and invariants already guaranteed;
  - effects, resources, and environmental assumptions.

Conclusion:
  - result type and properties;
  - postconditions that must be true.

Edge cases and counterexamples:
  - empty, invalid, duplicate, or extreme inputs;
  - dependency failures and concurrency;
  - expected behavior for each case.
```

If the requirement is ambiguous, ask for clarification or state the assumptions adopted before implementing. Do not silently choose an interpretation that changes the theorem.

### 2. Choose a representation that carries the proof

Model the domain with the smallest type capable of excluding invalid states. Encode preconditions in constructors and invariants in types, rather than leaving them as comments. Separate raw data from validated data:

```text
RawInput -> ValidationError + ValidatedInput
```

Make later functions receive `ValidatedInput`, not an input that may still violate the contract. For alternatives, use sum types; for information that must coexist, use product types; for partial operations, use an explicit result.

### 3. Decompose into lemmas

Break the main theorem into smaller properties. For each function, write:

- the signature, which is the proposition;
- the preconditions and postconditions;
- the preserved invariants;
- the lemma or logical rule that justifies each branch;
- the termination argument, when recursion is present.

Prefer APIs whose signatures make incorrect uses impossible or produce a type error immediately.

### 4. Build the proof in code

Implement according to the structure of the proposition:

- use constructors to introduce valid values;
- use function application to prove implications;
- use records/tuples to prove conjunctions;
- use variants and pattern matching to eliminate disjunctions;
- use exhaustive pattern matching over inductive types;
- use structural recursion or a well-founded measure to prove termination;
- preserve invariants in each recursive call and each state transition;
- keep the core pure when effects are not part of the property.

Do not add an impossible branch merely to silence the compiler. If the case is truly impossible, represent that impossibility in the type and eliminate it with the appropriate proof.

### 5. Treat failures and effects as part of the theorem

A function that throws an undeclared exception or returns `null` has a smaller contract than it appears to. Prefer signatures that expose all results:

```text
parse : RawInput -> Result<ValidatedInput, ParseError>
fetch : Request -> Effect<Result<Response, NetworkError>>
```

The I/O boundary may be impure, but its protocol must be explicit. Do not confuse a function that returns an effect plan with a function that has already executed the effect. For concurrent or distributed systems, include states, messages, failures, idempotency, ordering, and progress guarantees in the specification.

### 6. Verify the proof

Run checks in the order appropriate to the project:

1. formal prover, when available;
2. type checker in strict mode and exhaustiveness checking;
3. compiler, lint, and static analysis;
4. property, invariant, and edge-case tests;
5. integration tests for effects and external boundaries;
6. example-based and regression tests.

Record exactly what each stage guarantees. If a property is not expressed by the type system, declare it a residual obligation and cover it with contracts, tests, or an appropriate formal tool.

### 7. Explain the correspondence

When delivering code, briefly explain:

- what the theorem and its premises are;
- how each type represents a proposition in the domain;
- how each constructor, function, or branch provides a proof;
- why the cases are exhaustive;
- how termination and effects were handled;
- which properties were verified automatically and which remain assumptions or test evidence.

## Adaptation by ecosystem

- **Lean, Coq, Agda, and Idris:** formalize the proposition directly; use dependent types, equality, induction, and total functions. Never use `sorry`, `admit`, or unjustified axioms to declare the task complete.
- **Haskell, OCaml, Rust, and languages with ADTs:** use sum/product types, exhaustive pattern matching, error types, modules that hide constructors, and pure functions. Document the properties the compiler does not prove.
- **TypeScript and languages with partially verifiable types:** enable strict mode, validate `unknown` at boundaries, use discriminated unions, branded types, and `Result`; do not treat `as`, `!`, `any`, or `@ts-ignore` as proof.
- **Dynamic languages:** express contracts with validators, schema types, executable invariants, and property tests. The validator proves only the instance observed at the boundary, not a universal property of the program.
- **APIs, databases, and distributed systems:** include schema, authorization, transactions, consistency, timeout, retry, idempotency, and failures in the theorem. External data is always untrusted until validated.

## How to act in common tasks

### Implement a feature

1. Restate the requirement as a theorem and make assumptions explicit.
2. Propose types and contracts before the code.
3. Identify lemmas, edge cases, and possible counterexamples.
4. Implement the proof in small steps.
5. Run available checks and describe residual obligations.

### Change existing code

1. Reconstruct the theorem that the current code is intended to satisfy.
2. Preserve properties already proved and identify those that will be affected.
3. Look for counterexamples before changing the implementation.
4. Update types, proofs, tests, and documentation together.
5. Verify that no change silently weakened the contract.

### Debug a defect

Treat the bug as a counterexample to the current theorem. Reproduce it, describe which premise or invariant failed, strengthen the representation or fix the proof, and add a check that prevents regression. Do not merely suppress the symptom with a special condition or cast.

### Review code

Ask: what proposition does this module assert? Does the type really express it? Are there invalid values, unhandled branches, undeclared effects, recursion without a termination proof, or escape hatches hiding an obligation? Is the theorem satisfiable and useful, or is it vacuously true?

## Completion criteria

Consider an implementation complete only when:

- the theorem, premises, and postconditions are documented;
- the types represent the essential rules and invalid states are rejected;
- the relevant cases are exhaustive;
- termination or continuous behavior has been justified;
- failures and effects appear in the contract or are isolated at explicit boundaries;
- static/formal checking and executable validations have been run;
- every unproved obligation has been identified, not hidden;
- the final explanation connects the code to the proof through the Curry–Howard isomorphism.

The goal is not to use mathematics as a decorative metaphor. It is to build software whose structure, types, and checks carry explicit evidence of the intended correctness, while honestly recognizing the boundary between a proved property and a merely tested property.
