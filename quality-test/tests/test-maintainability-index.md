# Maintainability Index (Maintainability Index Subskill)

## Overview
The **Maintainability Index (MI)** is a composite metric originally formulated by the Software Engineering Institute (SEI) and adopted by Microsoft. It consolidates Halstead Volume, cyclomatic complexity, and lines of code into a single continuous score ranging from 0 to 100.

---

## Standard Formula

$$MI = \max\left(0, \frac{171 - 5.2 \times \ln(V) - 0.23 \times CC - 16.2 \times \ln(LOC)}{171} \times 100\right)$$

Where:
- $V$: Halstead Volume (count of unique and total operators and operands).
- $CC$: McCabe average Cyclomatic Complexity.
- $LOC$: Source Lines of Code.

---

## Rating Tiers

| Maintainability Index (MI) | Visual Grade | Maintainability Ease |
|----------------------------|--------------|----------------------|
| **85 to 100** | 🟢 Grade A | Excellent maintainability; clear, modular code |
| **65 to 84** | 🟡 Grade B | Moderate maintainability; acceptable for mid-size bases |
| **0 to 64** | 🔴 Grade C | Poor maintainability; highly prone to regression |

---

## Practical Measurement with Radon (Python)

```bash
# Overall report of maintainability index
radon mi src/ --show

# Display only files with suboptimal maintainability (Grade B or C)
radon mi src/ --min B

# Fail in CI if any file registers MI < 65 (Grade C)
radon mi src/ --max B --no-assert
```

## How to Improve a File's MI
1. **Reduce LOC**: Decompose files exceeding 300 lines into specialized modules.
2. **Reduce CC**: Flatten decision trees and eliminate nested conditionals.
3. **Reduce Halstead Volume $V$**: Avoid redundant intermediate variables and deeply chained property accesses.
