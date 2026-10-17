# Module Size and Granularity (Module Size Subskill)

## Overview
Measures physical code dimensions (lines, functions, parameters, and classes). Overly large modules and extensive functions are primary symptoms of Single Responsibility Principle (SRP) violations.

---

## Recommended Metrics and Thresholds

| Metric | Healthy Limit | Warning Range | Needs Refactoring |
|--------|---------------|---------------|-------------------|
| **Lines per File (LOC)** | $< 300$ lines | $300 - 500$ lines | $> 500$ lines |
| **Lines per Function / Method** | $< 30$ lines | $30 - 50$ lines | $> 50$ lines |
| **Lines per Class** | $< 200$ lines | $200 - 400$ lines | $> 400$ lines |
| **Functions per File** | $< 15$ | $15 - 25$ | $> 25$ |
| **Parameters per Function** | $< 4$ | $4 - 6$ | $> 6$ (use parameter object) |
| **Methods per Class** | $< 10$ | $10 - 20$ | $> 20$ |
| **Inheritance Depth** | $< 3$ | $3 - 5$ | $> 5$ (prefer composition) |
| **Classes per File** | $1 - 2$ | $3 - 4$ | $> 4$ |

---

## Automated Measurement

### 1. Via Radon (Python)

```bash
# Raw lines of code report (LOC, SLOC, comments)
radon raw src/ --json

# Alert on files exceeding 500 lines
radon raw src/ --json | python -c "
import json, sys
data = json.load(sys.stdin)
for path, metrics in data.items():
    if metrics['loc'] > 500:
        print(f'❌ {path}: {metrics[\"loc\"]} lines (limit: 500)')
"
```

### 2. Custom AST Checker Script (Python)

```python
# scripts/check_module_sizes.py
import ast
from pathlib import Path
import sys

def check_file_size(file: Path) -> list[str]:
    violations = []
    lines = file.read_text().splitlines()
    if len(lines) > 500:
        violations.append(f"{file}: {len(lines)} lines (max 500)")
        
    try:
        tree = ast.parse("\n".join(lines))
    except SyntaxError:
        return violations
        
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            func_len = node.end_lineno - node.lineno + 1
            if func_len > 50:
                violations.append(f"{file}:{node.lineno} Function '{node.name}' has {func_len} lines (max 50)")
            params = len(node.args.args) + len(node.args.kwonlyargs)
            if params > 5:
                violations.append(f"{file}:{node.lineno} Function '{node.name}' has {params} parameters (max 5)")
    return violations

if __name__ == "__main__":
    target_dir = Path(sys.argv[1] if len(sys.argv) > 1 else "src")
    all_violations = []
    for py in target_dir.rglob("*.py"):
        all_violations.extend(check_file_size(py))
    if all_violations:
        print("\n".join(all_violations))
        sys.exit(1)
```
