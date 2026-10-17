# Connascence (Connascence Subskill)

## Overview
Measures the quality and strength of coupling between two or more components. If modifying one component forces a modification in another to preserve correctness, both share **Connascence**.

---

## Connascence Taxonomy

### 1. Static Connascence (Detectable at Source Code Level)

| Type | Abbr. | Description | Example | Solution / Refactoring |
|------|-------|-------------|---------|------------------------|
| **Name** | CoN | Components agree on the identifier/symbol name | Calling method `calculate_tax()` | ✅ Healthy and unavoidable |
| **Type** | CoT | Components agree on the data type | Parameter must be `int` or `User` | ✅ Desirable (Type Checker) |
| **Meaning** | CoM | Agree on the meaning of literal values | `status == 2` represents "Approved" | 🟡 Replace with `Enum` or Constant |
| **Position**| CoP | Agree on parameter positioning | `func(name, age, email)` | 🟡 Use kwargs, objects, or DTOs |
| **Algorithm**| CoA | Agree on identical internal algorithms | Identical manual hashing logic in two locations | 🔴 Extract into single reusable module |

### 2. Dynamic Connascence (Detectable only at Runtime)

| Type | Abbr. | Description | Example | Solution / Refactoring |
|------|-------|-------------|---------|------------------------|
| **Execution** | CoE | Method execution ordering matters | `parser.init()` before `parser.run()` | 🟡 Builder pattern or atomic constructor |
| **Timing** | CoTm | Execution timing/milliseconds impact behavior | Thread race conditions | 🔴 Eliminate shared mutable state |
| **Value** | CoV | Values across separate places must match | Sum of line items must match record total | 🔴 Encapsulate invariant in rich domain entity |
| **Identity** | CoI | Must refer to the exact same object instance | Two services holding pointers to same mutable singleton | 🔴 Decoupled dependency injection |

---

## Golden Rules of Refactoring
1. **Reduce Strength**: Transform dynamic connascence into static, and strong static into weak static ($CoM \to CoN$ via enum).
2. **Reduce Distance**: Connascence between methods of the same class is acceptable; across distinct microservices it is critical.
3. **Reduce Degree**: Minimize the count of components bound by the dependency.

---

## Automated Detection via AST (Python)

```python
# scripts/detect_connascence.py
import ast

class ConnascenceVisitor(ast.NodeVisitor):
    def __init__(self):
        self.findings = []
        
    def visit_Call(self, node):
        # Detect CoP: more than 3 positional arguments without keywords
        if len(node.args) > 3 and not node.keywords:
            self.findings.append({
                "line": node.lineno,
                "type": "CoP (Position)",
                "msg": f"Call with {len(node.args)} positional args. Use kwargs or DTO."
            })
        self.generic_visit(node)
        
    def visit_Compare(self, node):
        # Detect CoM: direct comparison with magic numbers
        for comp in node.comparators:
            if isinstance(comp, ast.Constant) and isinstance(comp.value, int):
                if comp.value not in (0, 1, -1):
                    self.findings.append({
                        "line": node.lineno,
                        "type": "CoM (Meaning)",
                        "msg": f"Magic number '{comp.value}' detected. Extract into Enum or constant."
                    })
        self.generic_visit(node)
```
