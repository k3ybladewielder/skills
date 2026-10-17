# Mutation Testing (Mutation Testing Subskill)

## Overview
Measures the **true effectiveness** of an automated test suite. While code coverage only identifies executed lines, mutation testing injects deliberate syntactic defects (mutants) to verify that at least one test fails. If a mutant survives, an unasserted blind spot exists in the test suite.

## Common Mutation Operators

| Category | Applied Mutation | Before Example | After Example |
|----------|------------------|----------------|---------------|
| **Relational** | `>` to `>=`, `<` to `<=` | `if x > 0:` | `if x >= 0:` |
| **Arithmetic** | `+` to `-`, `*` to `/` | `a + b` | `a - b` |
| **Logical** | `and` to `or`, flag inversion | `a and b` | `a or b` |
| **Boundary** | Off-by-one in loops and slices | `range(n)` | `range(n - 1)` |
| **Negation** | Full boolean inversion | `if is_valid:` | `if not is_valid:` |
| **Return Value** | Modified return expression | `return price` | `return 0` or `return None` |
| **Statement Deletion** | Omission of side-effect call | `items.append(x)` | (removed) |
| **Constant** | Literal altered | `timeout = 30` | `timeout = 0` |

## Metric: Mutation Score
$$\text{Mutation Score} = \frac{\text{Killed Mutants}}{\text{Total Mutants} - \text{Equivalent Mutants}} \times 100$$

- **Thresholds**:
  - Critical logic / financial calculations / domain core: $\ge 70\%$
  - General application logic: $\ge 50\%$
- **Equivalent Mutants**: Mutations altering syntax without changing observable behavior (false positives).

---

## Tools and Configuration

### Python (`mutmut`)

```bash
# Execute mutation testing scoped strictly to domain core
mutmut run --paths-to-mutate=src/domain/ --tests-dir=tests/unit/

# Inspect results and surviving mutants
mutmut results
mutmut show <mutant_id>
mutmut html
```

```toml
# pyproject.toml
[tool.mutmut]
paths_to_mutate = "src/domain/"
tests_dir = "tests/unit/"
runner = "python -m pytest -x --tb=short"
```

### TypeScript / JavaScript (`Stryker Mutator`)

```json
// stryker.conf.json
{
  "$schema": "https://raw.githubusercontent.com/stryker-mutator/stryker-js/master/packages/api/schema/stryker-core.json",
  "packageManager": "npm",
  "reporters": ["html", "clear-text", "progress"],
  "testRunner": "vitest",
  "coverageAnalysis": "perTest",
  "mutate": ["src/core/**/*.ts", "!src/**/*.spec.ts"],
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}
```

```bash
npx stryker run
```

### Java (`PITest`)

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven-plugin</artifactId>
    <version>1.16.1</version>
    <configuration>
        <targetClasses>
            <param>com.company.core.domain.*</param>
        </targetClasses>
        <targetTests>
            <param>com.company.core.domain.*Test</param>
        </targetTests>
        <mutationThreshold>70</mutationThreshold>
        <coverageThreshold>80</coverageThreshold>
    </configuration>
</plugin>
```

---

## Cost-Effective Execution Strategy
- **High Computational Overhead**: $O(\text{mutants} \times \text{suite\_duration})$. Never run across the entire codebase on every commit.
- **Scope Restriction**: Limit strictly to pure business rules modules (domain/core).
- **Incremental PR Gates**: Run only on changed core files in PRs; trigger full runs on scheduled nightly/weekly pipelines.
