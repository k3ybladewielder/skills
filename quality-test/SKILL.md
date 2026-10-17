---
name: quality-testing
description: Master guide and objective code quality metrics index. Covers coverage, mutation, regression, E2E, architecture, dependencies, coupling, Martin metrics, connascence, module size, cyclomatic and cognitive complexity, hotspots, duplication, maintainability index, performance, property-based tests, dead code, SAST, linting, type-checking (TypeScript and Python), formal mechanical verification (Bend), and CI quality gates.
keywords: ["tests", "quality", "coverage", "mutation", "regression", "e2e", "architecture", "coupling", "connascence", "complexity", "cyclomatic", "metrics", "dependencies", "performance", "property-based", "dead-code", "sast", "lint", "type-checking", "typescript", "tsc", "mypy", "formal-verification", "bend", "quality-gate"]
---

# Quality Testing — Objective Measurement of Code Quality

## Philosophy

This skill guides the creation of tests and metrics that yield **objective, verifiable numbers** regarding the health of a codebase. The goal is not blindly pursuing 100%, but creating continuous visibility, establishing dependable baselines, detecting degradation before deployment, and driving data-informed refactoring.

Each quality dimension must be:
- **Automatable**: Collected in CI/locally without manual intervention.
- **Trackable**: Comparable across sprints and commits (trend > absolute).
- **Actionable**: When an indicator degrades, root cause and remedy are immediate.
- **Contextual**: Thresholds vary by architectural layer and domain criticality.

---

## Complete Index of Quality Subskills

### 1. Formal Verification
- **[Formal Code Verification](./tests/test-formal.md)**: Mechanical mathematical certainty of correctness by construction. Guides property declarations in `LAWS.bend`, demonstrations in `PROOF.bend`, and verification via the **[bend](/.agents/skills/bend/SKILL.md)** compiler.

### 2. Static Analysis, Linters & Typing
- **[Linting and Type Checking](./tests/test-lint-typecheck.md)**: Strict validation of type contracts and code standards in TypeScript (`tsc --noEmit`, ESLint Flat Config, Biome) and Python (`mypy --strict`, `ruff`).
- **[Static Application Security Testing (SAST)](./tests/test-sast-security.md)**: Scanning for known vulnerabilities (OWASP Top 10, CWEs) with Bandit, Semgrep, SpotBugs, and ESLint Security.
- **[Dead Code Detection](./tests/test-dead-code.md)**: Identifying and eliminating orphan functions, variables, and exports with Vulture, autoflake, ts-prune, and knip.
- **[Code Duplication (DRY)](./tests/test-duplication.md)**: Detecting code clones and duplicated snippets with `jscpd`, `pylint --enable=duplicate-code`, and `PMD CPD`.

### 3. Complexity, Hotspots & Maintainability
- **[Cyclomatic Complexity](./tests/test-cyclomatic-complexity.md)**: Quantifying McCabe control flow paths, correlating with required test cases, and refactoring using guard clauses and polymorphism.
- **[Cognitive Complexity](./tests/test-cognitive-complexity.md)**: Measuring code readability with nesting penalties via SonarQube and flake8 plugins.
- **[Module Size and Granularity](./tests/test-module-size.md)**: Thresholds for lines per file, lines per function, parameters, and classes to preserve the Single Responsibility Principle.
- **[Maintainability Index (MI)](./tests/test-maintainability-index.md)**: Composite score from 0 to 100 based on Halstead Volume, cyclomatic complexity, and LOC via Radon.
- **[Code Churn and Hotspots](./tests/test-churn-hotspots.md)**: Intersecting Git revision history with static complexity to prioritize refactoring where bugs actually occur.

### 4. Architecture, Coupling & Code Design
- **[Layer and Dependency Analysis](./tests/test-dependencies.md)**: Preventing circular dependencies and safeguarding architectural boundaries with `import-linter`, `ArchUnit`, and `dependency-cruiser`.
- **[Afferent and Efferent Coupling](./tests/test-coupling.md)**: Analyzing coupling types (Content, Control, Stamp, Data), calculating $C_a$ and $C_e$, and extracting dependency graphs via AST.
- **[Martin Metrics (Abstraction, Instability, and Distance)](./tests/test-martin-metrics.md)**: Diagnosing Risk Zones (Zone of Pain and Zone of Uselessness) and distance $D$ from the Main Sequence.
- **[Connascence](./tests/test-connascence.md)**: Assessing coupling strength (Name, Type, Meaning, Position, Algorithm, Execution, Timing, Value, Identity) and applying strength reduction rules.

### 5. Dynamic & Executable Tests
- **[Code Coverage](./tests/test-coverage.md)**: Configuring branch coverage, layer-specific targets, and preventing coverage gaming (`pytest-cov`, `vitest`, `JaCoCo`).
- **[Mutation Testing](./tests/test-mutation.md)**: Evaluating actual test effectiveness by injecting synthetic faults via `mutmut`, `Stryker`, and `PITest`.
- **[Regression Testing](./tests/test-regression.md)**: Snapshot testing for API contracts, consumer-driven contract testing with `Pact`, and Test Impact Analysis.
- **[End-to-End (E2E) Testing](./tests/test-e2e.md)**: Balanced testing pyramid, API-first scenarios, and browser automation with `Playwright`.
- **[Property-Based Testing](./tests/test-property-based.md)**: Validating universal invariants, roundtrip tests, and failure shrinking with `Hypothesis` and `fast-check`.
- **[Performance Testing & Benchmarks](./tests/test-performance.md)**: Statistical micro-benchmarks (`pytest-benchmark`), performance budgets, and load testing with `k6`.

### 6. Governance & Continuous Integration
- **[Quality Gate & Dashboard](./tests/test-quality-gate.md)**: Weighted unified quality score (0-100) and consolidated GitHub Actions pipeline with fail-fast policies.

---

## Decision Matrix: When to Apply Each Subskill

| Project Context | Highly Recommended Subskills |
|-----------------|-------------------------------|
| **Project Kickoff / New Module** | [Lint & Typecheck](./tests/test-lint-typecheck.md), [Coverage](./tests/test-coverage.md), [Dependencies](./tests/test-dependencies.md) |
| **Prior to Complex Refactoring** | [Churn & Hotspots](./tests/test-churn-hotspots.md), [Coupling](./tests/test-coupling.md), [Regression](./tests/test-regression.md) |
| **Critical Business Rules / Invariant Calculations** | [Property-Based](./tests/test-property-based.md), [Formal Verification (Bend)](./tests/test-formal.md), [Mutation](./tests/test-mutation.md) |
| **Critical Production Bugs** | [Regression (Snapshots)](./tests/test-regression.md), [Mutation](./tests/test-mutation.md) on the affected module |
| **Preparing for Scale / High Concurrency** | [Performance](./tests/test-performance.md), [E2E](./tests/test-e2e.md), [Dependencies](./tests/test-dependencies.md) |
| **Audit and PR Review** | [Quality Gate](./tests/test-quality-gate.md), [Cyclomatic Complexity](./tests/test-cyclomatic-complexity.md), [SAST](./tests/test-sast-security.md) |

---

## Recommended Implementation Phases

```
Phase 1 (Quick Wins)       ───► Lint, Type Check, Coverage, Dead Code, CC <= 10
Phase 2 (Architecture)    ───► Coupling (Ca/Ce), Dependencies, Martin Metrics (D)
Phase 3 (Depth)           ───► Mutation on Core, Connascence, Hotspots
Phase 4 (Rigor & Scale)   ───► Benchmarks/Load, Property-Based, Formal Verification (Bend)
```

---

## Agent Instructions
1. **Consult the dedicated subskill document** before implementing tests or metrics in a specific technical area.
2. **Type Check and Linter come first**: Any static typing breakdown invalidates higher-level metric reliability.
3. **Use the Bend skill when formal mathematical proof of critical algorithms is required**.
4. **Automate everything in the CI pipeline**: Any test or metric that is not automatically executed on each PR will eventually be abandoned by the team.
