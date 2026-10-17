# Cyclomatic Complexity (Cyclomatic Complexity Subskill)

## Overview
Measures the number of linearly independent paths through the control flow of a routine. Proposed by Thomas J. McCabe in 1976. Each conditional statement (`if`, `elif`, `for`, `while`, `and`, `or`, `except`, `case`) introduces an additional branch.

---

## Formula and Relationship with Testing

$$CC = E - N + 2P$$

In practice for a single routine ($P=1$):
$$CC = 1 + (\text{number of boolean / conditional decision points})$$

> **Rule of Thumb**: The value of $CC$ establishes the theoretical minimum number of test cases required to achieve 100% branch coverage. A function with $CC = 12$ demands at least 12 distinct test scenarios.

---

## Risk Thresholds

| Complexity Score (CC) | Risk Level | Recommended Action |
|-----------------------|------------|--------------------|
| **1 to 5** | 🟢 Low | Simple, highly testable code |
| **6 to 10** | 🟡 Moderate | Acceptable; monitor against further expansion |
| **11 to 20** | 🟠 High | Difficult to test; refactoring strongly advised |
| **21 to 50** | 🔴 Very High | Bug-prone code; refactoring mandatory |
| **> 50** | ⚫ Critical | Untestable and incomprehensible; rewrite immediately |

---

## Tools and Configuration

### Python (`radon` and `ruff`)

```bash
# Report functions exceeding lower complexity thresholds
radon cc src/ --min B --show-complexity

# Fail if functions grade C or worse (CC > 10)
radon cc src/ --min C --no-assert
```

```toml
# pyproject.toml with Ruff
[tool.ruff.lint]
select = ["C901"] # McCabe cyclomatic complexity

[tool.ruff.lint.mccabe]
max-complexity = 10
```

### TypeScript / JavaScript (`eslint`)

```javascript
// eslint.config.mjs
export default [
  {
    rules: {
      "complexity": ["error", { "max": 10 }]
    }
  }
];
```

### Java (`PMD`)

```xml
<rule ref="category/java/design.xml/CyclomaticComplexity">
    <properties>
        <property name="methodReportLevel" value="10"/>
    </properties>
</rule>
```

---

## Refactoring Patterns to Reduce CC
1. **Guard Clauses**: Replace nested blocks (`if/else/if`) with early returns.
2. **Polymorphism / Strategy**: Replace large `switch/case` structures with polymorphic strategies or dispatch dictionaries.
3. **Extract Function**: Break autonomous logic blocks into pure, focused functions with $CC \le 3$.
