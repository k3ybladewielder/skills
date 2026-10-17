# Dead Code Detection (Dead Code Detection Subskill)

## Overview
Identifies and eliminates sections of code that are never reached or executed at runtime:
- Functions and methods declared but never invoked.
- Orphan imports and local variables that are never read.
- Unreachable branches (`if False:`, code after unconditional `raise`/`return`).
- Classes and types unexported or never instantiated.

Dead code clutters developer cognition, falsely inflates maintenance overhead, and expands the attack surface for potential vulnerabilities.

---

## Tools by Ecosystem

### 1. Python (`vulture` and `autoflake`)

```bash
# Vulture: deep static analysis with confidence score (0-100%)
vulture src/ --min-confidence=80

# Autoflake: immediate removal of unused imports and variables
autoflake --check --remove-all-unused-imports -r src/
```

Whitelist file for dynamic methods or framework reflection:
```python
# .vulture_whitelist.py
# Methods accessed dynamically via reflection by Django/FastAPI
MyModel.DoesNotExist
MyView.get_context_data
```

### 2. TypeScript / JavaScript (`ts-prune` and `knip`)

```bash
# ts-prune: finds exports never imported by any other file
npx ts-prune --project tsconfig.json

# knip: detects unused files, orphan dependencies, and dead exports
npx knip
```

### 3. Java (`PMD`)

Native PMD rules:
- `UnusedPrivateMethod`
- `UnusedLocalVariable`
- `UnusedFormalParameter`
- `UnusedPrivateField`

---

## CI Gate Thresholds
- **New code in PRs**: **0 occurrences of dead code**.
- **Legacy codebases**: Establish a baseline and reduce by 10% per sprint until reaching zero.
