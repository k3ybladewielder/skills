# Code Churn and Maintenance Hotspots (Churn & Hotspots Subskill)

## Overview
Identifies where code undergoes the highest volume and frequency of edits (**code churn**) and correlates this activity with static complexity metrics. The intersection of high churn and high complexity flags the system's **hotspots** — areas statistically most prone to recurrent defects.

---

## The Hotspot Model

$$\text{File Risk} = \text{Churn Frequency} \times \text{Average Complexity (CC)}$$

- **Complex code with low churn**: Low risk (stable legacy logic; avoid unprompted refactoring).
- **Simple code with high churn**: Moderate risk (typically configuration files or route tables).
- **Complex code with high churn**: 🔴 **CRITICAL HOTSPOT**. The origin of most production incidents.

---

## Measurement and Automation via Git

### 1. Inspect Top Churn Files in the Terminal

```bash
# Top 15 most altered files over the last 6 months
git log --since="6 months ago" --name-only --pretty=format: | \
    sort | uniq -c | sort -rn | head -15
```

### 2. Automated Hotspot Detection Script (Python)

```python
# scripts/detect_hotspots.py
import subprocess
import json
from pathlib import Path

def get_git_churn(months: int = 6) -> dict[str, int]:
    cmd = ["git", "log", f"--since={months} months ago", "--name-only", "--pretty=format:"]
    res = subprocess.run(cmd, capture_output=True, text=True)
    churn = {}
    for line in res.stdout.splitlines():
        line = line.strip()
        if line and not line.startswith("tests/"):
            churn[line] = churn.get(line, 0) + 1
    return churn

def get_radon_complexity(directory: str) -> dict[str, float]:
    cmd = ["radon", "cc", directory, "--json", "--average"]
    res = subprocess.run(cmd, capture_output=True, text=True)
    try:
        data = json.loads(res.stdout)
    except json.JSONDecodeError:
        return {}
    cc_map = {}
    for file, funcs in data.items():
        if funcs:
            cc_map[file] = sum(f["complexity"] for f in funcs) / len(funcs)
    return cc_map

def find_hotspots():
    churn = get_git_churn(6)
    complexity = get_radon_complexity("src")
    hotspots = []
    
    for file, changes in churn.items():
        cc = complexity.get(file, 1.0)
        risk = changes * cc
        if risk > 40:
            hotspots.append({"file": file, "churn": changes, "cc": round(cc, 1), "risk": round(risk, 1)})
            
    hotspots.sort(key=lambda x: x["risk"], reverse=True)
    return hotspots
```

## Recommended Preventive Practices
1. Do not refactor complex code unnecessarily if it exhibits **zero churn**.
2. When reviewing a PR touching a **Hotspot**, enforce strict mutation test coverage and peer architectural review.
