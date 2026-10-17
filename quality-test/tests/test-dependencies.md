# Dependency Analysis and Architecture (Dependency Analysis Subskill)

## Overview
Analyzes the integrity and directional correctness of dependencies across internal modules (layered architecture and bounded contexts) alongside the security and health of third-party libraries.

## Dependency Metrics

| Metric | Concept | Recommended Threshold |
|--------|---------|-----------------------|
| **Fan-in** | Number of modules depending on this module | High in domain/core, low in controllers |
| **Fan-out** | Number of modules this module imports/depends on | High indicates excessive coupling |
| **Dependency Depth** | Maximum transitive dependency chain depth | $< 5$ levels |
| **Circular Dependencies** | Dependency graph cycles ($A \to B \to A$) | **0 (zero tolerance)** |

---

## Internal Layer Validation

### Python (`import-linter`)

```ini
# .importlinter
[importlinter]
root_package = my_app

[importlinter:contract:layers]
name = Clean Layered Architecture
type = layers
layers =
    my_app.presentation
    my_app.application
    my_app.domain
    my_app.infrastructure
```

```bash
import-linter --config=.importlinter
```

### TypeScript / JavaScript (`eslint-plugin-boundaries` or `dependency-cruiser`)

```javascript
// .dependency-cruiser.js
module.exports = {
  forbidden: [
    {
      name: 'no-circular',
      severity: 'error',
      from: {},
      to: { circular: true }
    },
    {
      name: 'domain-cannot-import-infrastructure',
      severity: 'error',
      from: { path: '^src/domain' },
      to: { path: '^src/infrastructure' }
    }
  ]
};
```

```bash
npx depcruise --config .dependency-cruiser.js src/
```

### Java (`ArchUnit`)

```java
@AnalyzeClasses(packages = "com.company.app")
class ArchitectureRulesTest {
    @ArchTest
    static final ArchRule no_circular_dependencies = slices()
        .matching("com.company.app.(*)..")
        .should().beFreeOfCycles();

    @ArchTest
    static final ArchRule domain_does_not_depend_on_controllers = noClasses()
        .that().resideInAPackage("..domain..")
        .should().dependOnClassesThat().resideInAPackage("..controllers..");
}
```

---

## External Dependency Security Auditing

```bash
# Python
pip-audit --requirement=requirements.txt --output=json

# Node.js / TypeScript
npm audit --audit-level=high

# License Auditing
npx license-checker --summary
```
