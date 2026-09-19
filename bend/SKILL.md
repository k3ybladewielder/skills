---
name: bend
description: Bend compiles to native code. On one core, it runs nearly as fast as C. The same binary also runs on sixteen cores, or on the GPU, running up to a hundred times faster than one core. Bend's type checker is a proof checker, as in Lean and Rocq. Those can take minutes on a mid-sized codebase. Bend takes a second at most, so an AI agent can check after every change.
keywords: ["formal verification", "proof checker", "bend"]
---

When using Bend:
- run `bend guide` to learn it
- use `LAWS.bend` to keep important rules
- run `bend PROOF.bend` before committing
- parallelize the code whenever possible

Rules:
- write laws for whatever should never break, and to parallelize everything you want running fast.
