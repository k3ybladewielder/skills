# Static Application Security Testing - SAST (Static Security Testing Subskill)

## Overview
**Static Application Security Testing (SAST)** analyzes source code or binaries statically to uncover known security flaws (CWEs, OWASP Top 10) before any code is executed or deployed.

Unlike functional tests, SAST scans for risk patterns:
- SQL / NoSQL / Shell Command Injection.
- Insecure deserialization (`pickle`, `yaml.load`).
- Outdated cryptography (MD5, SHA1, predictable PRNGs).
- Hardcoded secrets, API keys, and tokens.
- Cross-Site Scripting (XSS) and prototype pollution.

---

## Recommended Tools by Ecosystem

| Language | Tools | Primary Focus |
|----------|-------|---------------|
| **Python** | `Bandit`, `Semgrep` | Injection, eval, unsafe subprocess, crypto |
| **TypeScript / JS** | `ESLint Security`, `Semgrep` | Prototype pollution, regex DoS, eval |
| **Java** | `SpotBugs` + `FindSecBugs`, `Semgrep` | SQLi, XML injection, path traversal |
| **Multi-Language** | `Semgrep`, `SonarQube`, `Trivy` | Broad enterprise security scanning |

---

## Configuration and Execution

### 1. Bandit (Python)

```bash
# Run security analysis filtering for medium/high severity
bandit -r src/ --severity-level medium -f json -o bandit-report.json
```

```toml
# pyproject.toml
[tool.bandit]
exclude_dirs = ["tests", "migrations"]
skips = ["B101"] # Ignore assert in tests if necessary
```

### 2. Semgrep (Multi-Language)

```bash
# Run community-maintained security rulesets
semgrep --config=auto src/ --error
```

```yaml
# .semgrep.yml
rules:
  - id: raw-sql-format
    pattern: $DB.execute(f"... {$VAR} ...")
    message: "SQL Injection risk via string interpolation. Use parameterized queries."
    languages: [python]
    severity: ERROR
```

### 3. ESLint Security (TypeScript / Node.js)

```javascript
// eslint.config.mjs
import pluginSecurity from "eslint-plugin-security";

export default [
  pluginSecurity.configs.recommended,
  {
    rules: {
      "security/detect-object-injection": "warn",
      "security/detect-non-literal-fs-filename": "error",
      "security/detect-unsafe-regex": "error"
    }
  }
];
```

---

## CI Thresholds
- **High or Critical Severity Vulnerabilities**: **Zero tolerance** (immediate pipeline failure).
- **Hardcoded Secrets**: Preventive pre-commit/pre-push scanning with `gitleaks`.
