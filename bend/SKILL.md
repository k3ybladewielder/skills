---
name: bend
description: Bend is a programming language that combines Lean-like formal proofs with C-like speeds and CUDA-like parallelism. It gives humans an ambiguity-free language to communicate intents to AIs, a compiler capable of mechanically checking mathematical correctness, and a runtime that executes code fast on CPUs and GPUs.
keywords: ["formal verification", "proof checker", "bend", "parallelism", "interactive theorem prover"]
---

# Bend Skill Guide

When using Bend:
- Run `bend guide` to learn the core syntax and concepts.
- Use `LAWS.bend` to declare business rules, invariants, and specifications.
- Use `PROOF.bend` to formally prove every law declared in `LAWS.bend`.
- Run `bend check <file>.bend` and `bend PROOF.bend` before committing.
- Parallelize operations whenever compute-intensive or recursive tree reductions are involved.

## Rules
- Write laws for whatever should never break, and parallelize everything you want running fast.
- The [Common Errors](file:///.agents/skills/bend/examples/common_errors.md) reference must be updated whenever new error patterns are identified.
- All files and documentation in this skill are maintained strictly in English.

---

## Hello, World!

Bend's syntax is Python-shaped, with pure functional semantics closer to Haskell / Lean, while remaining resource-aware like Rust. A hello world is a typed definition returning an IO block:

```python
import Base

def main() -> IO(Unit):
  do IO<Unit>:
    IO.print("Hello, world!")
```

To run it:
```sh
bend hello.bend
```

Bend is a *pure language* where side-effects are encapsulated inside the [IO Monad](https://wiki.haskell.org/Introduction_to_IO).

---

## Reference Summary & Examples

### Core Language & Topics
- [Tooling](file:///.agents/skills/bend/examples/tooling.md): CLI commands (`bend`, `bend check`, `bend -c`), compilation flags, and workflows.
- [Syntax Reference](file:///.agents/skills/bend/examples/syntax_reference.md): Complete grammar reference, literal formats, AST forms, and assignment rules.
- [Types and Functions](file:///.agents/skills/bend/examples/types_and_functions.md): Data types, constructors, function signatures, and affine annotations (`+`).
- [Closures](file:///.agents/skills/bend/examples/closures.md): Anonymous functions, lambda expressions, and environment capture.
- [Recursion and Termination](file:///.agents/skills/bend/examples/recursion_and_termination.md): Structural recursion, totality checks, and `@unsafe` escape hatch.
- [Parallelism](file:///.agents/skills/bend/examples/parallelism.md): Multi-threaded execution, parallel dispatch (`a b = f(x) g(y)`), and GPU execution (`f!(x)`).
- [Arrays](file:///.agents/skills/bend/examples/arrays.md): Single-owner mutable arrays with $O(1)$ in-place updates (`a[i] <- v`).
- [Quantities](file:///.agents/skills/bend/examples/quantities.md): Variable usage tracking with `&0` (erased), `&1` (affine), and `&2` (unrestricted data).
- [Kinds](file:///.agents/skills/bend/examples/kinds.md): Type classification into `Data` (copiable) and `Type` (linear/non-copiable).
- [Templates](file:///.agents/skills/bend/examples/templates.md): Compile-time monomorphization and metaprogramming via `~` arguments.
- [Laws and Proofs](file:///.agents/skills/bend/examples/laws_and_proofs.md): Formal verification workflow using `LAWS.bend` and `PROOF.bend`.
- [IO and Concurrency](file:///.agents/skills/bend/examples/io_and_concurrency.md): Encapsulating effects, file operations, sockets, and asynchronous tasks.
- [Monads](file:///.agents/skills/bend/examples/monads.md): Monadic computation chains, `do` blocks, `Maybe`, and `Result`.
- [Apps](file:///.agents/skills/bend/examples/apps.md): Interactive 2D/3D graphical applications and game engines in pure Bend.
- [The Base Library](file:///.agents/skills/bend/examples/base_library.md): Standard prelude types (`Bool`, `Nat`, `U32`, `F32`, `List`) and core operations.
- [Modules](file:///.agents/skills/bend/examples/modules.md): Multi-file organization, relative imports, aliasing, and diamond deduplication.
- [Common Errors](file:///.agents/skills/bend/examples/common_errors.md): Comprehensive catalog of common compiler errors, anti-patterns, and idiomatic fixes.

### Test Subskills (Located in `.agents/skills/bend/tests/`)
- [Base Tests](file:///.agents/skills/bend/tests/tests_base.md): Standard library primitives, algebraic types, and built-in type operations.
- [Check Tests](file:///.agents/skills/bend/tests/tests_check.md): Type checking, kind inference, and affine linearity verification.
- [Compile Tests](file:///.agents/skills/bend/tests/tests_compile.md): Compiler pipeline, lowering targets, and native C/CUDA code generation.
- [Comptime Tests](file:///.agents/skills/bend/tests/tests_comptime.md): Compile-time evaluation, constant folding, and template macro expansion.
- [Cost Tests](file:///.agents/skills/bend/tests/tests_cost.md): Work metrics, interaction net rewrite accounting, and complexity analysis.
- [Eval Tests](file:///.agents/skills/bend/tests/tests_eval.md): Term reduction, normal-form computation, and operational evaluation.
- [Flatten Tests](file:///.agents/skills/bend/tests/tests_flatten.md): Pattern matching compilation and decision tree flattening.
- [Gfx Tests](file:///.agents/skills/bend/tests/tests_gfx.md): 2D/3D graphics rendering, pixel shaders, and display loops.
- [Grade Tests](file:///.agents/skills/bend/tests/tests_grade.md): Resource grading, quantities (`&0`, `&1`, `&2`), and erased proof parameters.
- [Halt Tests](file:///.agents/skills/bend/tests/tests_halt.md): Termination checking, structural recursion, and infinite loop prevention.
- [Import Tests](file:///.agents/skills/bend/tests/tests_import.md): Module resolution, relative paths, and cross-file proof linking.
- [IO Tests](file:///.agents/skills/bend/tests/tests_io.md): Side-effect encapsulation, concurrent TCP sockets, and file streams.
- [Page Tests](file:///.agents/skills/bend/tests/tests_page.md): Memory paging and single-owner mutable array operations.
- [Parse Tests](file:///.agents/skills/bend/tests/tests_parse.md): Lexical analysis, syntax grammar rules, and operator precedence.
- [Printer Tests](file:///.agents/skills/bend/tests/tests_printer.md): Pretty-printing, AST desugaring, and term formatting.
- [Proof Tests](file:///.agents/skills/bend/tests/tests_proof.md): Theorem proving, propositional equality, reflexivity (`{==}`), and rewrites.
- [Reg Tests](file:///.agents/skills/bend/tests/tests_reg.md): Backend register allocation and affine variable lifecycles.
- [RFC Tests](file:///.agents/skills/bend/tests/tests_rfc.md): Bend language specifications, modern idioms, and RFC evolutions.
- [Run Tests](file:///.agents/skills/bend/tests/tests_run.md): End-to-end execution on CPU and CUDA GPU backends.
- [Show Tests](file:///.agents/skills/bend/tests/tests_show.md): Interactive term inspection, hole diagnostics (`?name`), and goal debugging.
- [Spec Tests](file:///.agents/skills/bend/tests/tests_spec.md): Formal operational semantics and core language invariants.
- [State Tests](file:///.agents/skills/bend/tests/tests_state.md): Pure state threading, state monads, and mutable cell abstractions.
- [Stats Tests](file:///.agents/skills/bend/tests/tests_stats.md): Runtime profiling, node allocation metrics, and throughput counters.
- [Stuck Tests](file:///.agents/skills/bend/tests/tests_stuck.md): Stuck term diagnostics, reduction failures, and deadlock resolution.

---

## Further Reading

- `demos/`: Complete example programs (HTTP server, ray tracer, parallel sorting, 3D boss fight game).
- `bend2/base.bend`: The official Base prelude library (or run `bend base`).
- `paper/BendTT.pdf` & `paper/BendRT.pdf`: Formal type theory and runtime specifications.