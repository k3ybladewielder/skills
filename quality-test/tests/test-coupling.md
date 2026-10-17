# Coupling (Coupling Subskill)

## Overview
Measures the degree of interdependence and connectivity across system modules. High coupling implies that refactoring or patching one module triggers cascading failures and unexpected regressions in unrelated parts of the codebase.

## Coupling Types (from Worst to Best)

| Type | Severity | Diagnostic | Solution |
|------|----------|------------|----------|
| **Content Coupling** | 🔴 Critical | One module directly accesses/modifies another's private internal state | Strict encapsulation |
| **Common Coupling** | 🔴 High | Modules share mutable global state | Dependency injection / Immutability |
| **Control Coupling** | 🟡 Medium | Module A dictates B's execution flow via control flags/booleans | Polymorphism or inversion of control |
| **Stamp Coupling** | 🟡 Medium | Module consumes a 30-field entity but only needs 1 field | Pass only the required parameter |
| **Data Coupling** | 🟢 Low | Strict communication through atomic primitive parameters | Ideal standard pattern |
| **Message Coupling** | 🟢 Minimal | Communication via decoupled asynchronous event buses | Decoupled event architectures |

---

## Mathematical Metrics: Ca and Ce

- **Afferent Coupling ($C_a$)**: Modules that depend on this module (incoming dependencies). High $C_a$ modules must be stable and rigorously validated.
- **Efferent Coupling ($C_e$)**: Modules on which this module depends (outgoing dependencies). High $C_e$ modules are fragile and vulnerable to external modifications.

### Automation Script via AST (Python)

```python
# scripts/measure_coupling.py
import ast
from pathlib import Path
from collections import defaultdict

def extract_import_graph(base_dir: Path) -> dict[str, set[str]]:
    graph = defaultdict(set)
    for file in base_dir.rglob("*.py"):
        if "tests" in file.parts:
            continue
        mod_name = str(file.relative_to(base_dir)).replace("/", ".").removesuffix(".py")
        try:
            tree = ast.parse(file.read_text())
        except SyntaxError:
            continue
        for node in ast.walk(tree):
            if isinstance(node, ast.Import):
                for alias in node.names:
                    graph[mod_name].add(alias.name)
            elif isinstance(node, ast.ImportFrom) and node.module:
                graph[mod_name].add(node.module)
    return graph

def calculate_metrics(graph: dict[str, set[str]]):
    all_mods = set(graph.keys())
    metrics = {}
    for mod in all_mods:
        ce = len(graph.get(mod, set()) & all_mods)
        ca = sum(1 for deps in graph.values() if mod in deps)
        metrics[mod] = {"Ca": ca, "Ce": ce}
    return metrics
```

## Engineering Thresholds
- $C_e \le 7$: Healthy.
- $C_e > 12$: Overburdened module warning (refactoring / SRP needed).
- Coupling cycles: **Zero tolerance**.
