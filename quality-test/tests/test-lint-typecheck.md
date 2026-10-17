# Linting and Static Type Checking (Lint & Type-Checking Subskill)

## Overview
Ensures stylistic consistency, engineering best practices, early detection of asynchronous bugs, and mathematical integrity of the type system without executing the software.

---

## Tool Equivalence Table

| Purpose | TypeScript / JavaScript | Python | Java |
|---------|-------------------------|--------|------|
| **Standard Linter** | `eslint` with `@typescript-eslint` | `ruff check` (or `flake8`/`pylint`) | `Checkstyle`, `PMD` |
| **Ultra-fast Linter** | `biome` | `ruff check` | — |
| **Type Checker** | `tsc --noEmit` / `vue-tsc` | `mypy --strict`, `pyright` | Native compiler (`javac`) |
| **Code Formatter** | `prettier` or `biome format` | `ruff format` (or `black`) | `Spotless`, `google-java-format` |

---

## Configuration: TypeScript

### 1. `tsconfig.json` (Strict Type Checking)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noEmit": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

Execution:
```bash
npx tsc --noEmit
```

### 2. `eslint.config.mjs` (ESLint 9+ Flat Config)

```javascript
import js from "@eslint/js";
import tsParser from "@typescript-eslint/parser";
import tsPlugin from "@typescript-eslint/eslint-plugin";

export default [
  js.configs.recommended,
  {
    files: ["**/*.ts", "**/*.tsx"],
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        project: "./tsconfig.json",
      },
    },
    plugins: {
      "@typescript-eslint": tsPlugin,
    },
    rules: {
      ...tsPlugin.configs["recommended-type-checked"].rules,
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/await-thenable": "error",
      "@typescript-eslint/no-misused-promises": "error",
      "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
      "@typescript-eslint/consistent-type-imports": "error",
    },
  },
];
```

Execution:
```bash
npx eslint . --max-warnings=0
```

### 3. Alternative: Biome (`biome.json`)

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": { "noUnusedVariables": "error" },
      "suspicious": { "noExplicitAny": "error" }
    }
  }
}
```

Execution:
```bash
npx @biomejs/biome check src/
```

---

## Configuration: Python Equivalent

### 1. `mypy` (`pyproject.toml`)

```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
disallow_untyped_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
```

Execution:
```bash
mypy src/
```

### 2. `ruff` (`pyproject.toml`)

```toml
[tool.ruff]
target-version = "py311"
line-length = 100

[tool.ruff.lint]
select = [
    "E", "W", "F", "I", "B", "C4", "UP", "ARG", "PTH", "ASYNC", "S"
]
```

Execution:
```bash
ruff check src/
```

---

## CI Gates
- **Type Checking Errors**: **0 tolerated**.
- **Lint Warnings**: **0 warnings** (`--max-warnings=0`).
