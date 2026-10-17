# Code Coverage Testing (Coverage Subskill)

## Overview
Measures the percentage of source code exercised by automated test suites. The goal is ensuring no critical logical paths or branches operate without automated validation.

## Coverage Types

| Type | What it covers | When to use |
|------|----------------|-------------|
| **Line Coverage** | Executed lines | Minimum baseline |
| **Branch Coverage** | Each branch (`if`/`else`, `match`) | Conditional logic and business decisions |
| **Function Coverage** | Invoked functions/methods | Verification of public APIs and interfaces |
| **Statement Coverage** | Executed statements | Standard execution baseline |
| **Condition Coverage** | Each boolean sub-expression (`A and B`) | Compound boolean expressions |
| **Path Coverage** | Every possible execution path | Critical financial workflows (high cost) |
| **MC/DC** (Modified Condition/Decision) | Each condition independently affects outcome | Safety-critical and aerospace systems |

## Recommended Thresholds by Layer

| Architectural Layer | Minimum | Recommended Target |
|---------------------|---------|--------------------|
| **Domain / Business Logic** | 90% branch | 95%+ branch |
| **Services / Use Cases** | 80% branch | 90% branch |
| **Infrastructure / Repositories** | 60% line | 75% line |
| **Controllers / Handlers / Routes** | 70% line | 80% line |
| **Pure Utilities / Helpers** | 85% branch | 95%+ branch |

---

## Configuration by Ecosystem

### Python (`pytest-cov`)

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing --cov-report=html --cov-branch"

[tool.coverage.run]
branch = true
source = ["src"]
omit = ["*/tests/*", "*/__pycache__/*", "*/migrations/*"]

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@overload",
]
```

### TypeScript / JavaScript (`vitest` or `c8`/`istanbul`)

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
      exclude: ['**/tests/**', '**/mocks/**', '**/*.d.ts', '**/migrations/**'],
    },
  },
});
```

### Java (`JaCoCo`)

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>check</id>
            <phase>verify</phase>
            <goals><goal>check</goal></goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

## Anti-Patterns and Best Practices
- **Coverage Gaming**: Tests without valid assertions (`assert`) written solely to artificially inflate coverage numbers.
- **False Security with Line Coverage**: 100% line coverage with only 40% branch coverage leaves error handlers and branches untested.
- **Boilerplate Coverage Waste**: Spending time testing trivial getters/setters or generated code rather than core business logic.
- **Track Coverage Delta in CI**: Enforce that PRs never decrease overall coverage relative to the target branch.
