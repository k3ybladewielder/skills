# Quality Gate and Consolidated Dashboard (Quality Gate Subskill)

## Overview
Consolidates all quality metrics into an automated CI/CD pipeline and a unified 0-to-100 composite health score. Guarantees that incoming code never degrades engineering standards when merged into the main branch.

---

## Weighted Composite Score Model

```python
# scripts/quality_score.py
WEIGHTS = {
    "coverage_branch": 0.15,
    "mutation_score": 0.15,
    "cyclomatic_avg": 0.10,
    "cognitive_avg": 0.10,
    "maintainability_index": 0.10,
    "duplication": 0.10,
    "coupling_distance": 0.10,
    "typecheck_and_lint": 0.10,
    "dead_code_pct": 0.05,
    "security_vulnerabilities": 0.05,
}

def calculate_overall_quality_score(metrics: dict[str, float]) -> float:
    """Calculates weighted composite score from 0 to 100."""
    total_score = 0.0
    for key, weight in WEIGHTS.items():
        val = metrics.get(key, 0.0)
        total_score += min(max(val, 0.0), 100.0) * weight
    return round(total_score, 1)
```

---

## Consolidated CI Pipeline (GitHub Actions)

```yaml
# .github/workflows/quality-gate.yml
name: Quality Gate & Health Check

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 1. Type Checking & Linting (TypeScript)
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - name: TypeScript Type Check
        run: npx tsc --noEmit
      - name: TypeScript Linting
        run: npx eslint . --max-warnings=0

      # 2. Type Checking & Linting (Python)
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - name: Python Type Check
        run: mypy src/
      - name: Python Lint (Ruff)
        run: ruff check src/

      # 3. Automated Tests and Coverage
      - name: Tests with Branch Coverage
        run: pytest --cov=src --cov-branch --cov-fail-under=80

      # 4. Complexity and Modules
      - name: Cyclomatic Complexity
        run: |
          radon cc src/ --min C --no-assert
          if [ $? -ne 0 ]; then echo "❌ Functions with CC > 10"; exit 1; fi

      # 5. Code Duplication
      - name: Code Duplication (jscpd)
        run: npx jscpd src/ --threshold=5 --exitCode=1

      # 6. Dead Code
      - name: Dead Code Detection
        run: vulture src/ --min-confidence=90

      # 7. Static Security (SAST)
      - name: Static Security Scan (Bandit)
        run: bandit -r src/ --severity-level medium

      # 8. Layered Architecture
      - name: Architectural Dependencies Verification
        run: import-linter --config=.importlinter
```

---

## Gate Philosophy
- **Fail Fast**: Linters and Type Checkers execute in the initial 60 seconds of the job.
- **Non-Degradation Invariant**: PR code coverage must always be $\ge$ coverage on the `main` branch.
