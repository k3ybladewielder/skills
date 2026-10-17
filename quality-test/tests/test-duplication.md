# Code Duplication & DRY Violations (Code Duplication Subskill)

## Overview
Measures the percentage of cloned or near-duplicate code. Duplication is the primary driver of divergent bugs: a defect is resolved in one copy, while remaining unnoticed across other cloned instances.

---

## Code Duplication Types

| Type | Description | Detection Level |
|------|-------------|-----------------|
| **Type 1** | Identical character-for-character copy (ignoring whitespace) | Simple textual |
| **Type 2** | Copy with renamed variables and identifiers | Token / AST |
| **Type 3** | Copy with minor edits (lines inserted or removed) | AST / Fuzzy token |
| **Type 4** | Functionally equivalent logic expressed with entirely different syntax | Advanced semantic |

---

## Engineering Thresholds

| Duplicated Code Percentage | Status | CI Action |
|----------------------------|--------|-----------|
| **$< 3\%$** | 🟢 Excellent | Allow merge |
| **$3\% - 5\%$** | 🟡 Acceptable | Monitor trend |
| **$5\% - 10\%$** | 🟠 Investigate | PR warning |
| **$> 10\%$** | 🔴 Unacceptable | Block merge |

---

## Detection Tools

### 1. Multi-Language (`jscpd`)

Supports TypeScript, JavaScript, Python, Java, Go, Rust, C#, etc.

```bash
# Execute check enforcing a 5% tolerance threshold
npx jscpd src/ --threshold=5 --min-lines=5 --min-tokens=50 --reporters=console,html
```

Configuration file:
```json
// .jscpd.json
{
  "threshold": 5,
  "reporters": ["html", "console"],
  "ignore": ["**/node_modules/**", "**/tests/**", "**/dist/**"],
  "minTokens": 50,
  "minLines": 5
}
```

### 2. Python (`pylint`)

```bash
pylint --disable=all --enable=duplicate-code --min-similarity-lines=6 src/
```

### 3. Java (`PMD CPD`)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.21.0</version>
    <configuration>
        <minimumTokens>100</minimumTokens>
        <targetDirectory>${project.build.directory}/cpd</targetDirectory>
    </configuration>
</plugin>
```
