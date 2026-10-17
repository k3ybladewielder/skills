# Cognitive Complexity (Cognitive Complexity Subskill)

## Overview
Measures the mental effort required for a developer to **understand and reason about** code. While cyclomatic complexity ($CC$) counts logical paths for testing purposes, cognitive complexity evaluates human readability and heavily penalizes nested control structures (*nesting*).

---

## Difference between Cyclomatic and Cognitive Complexity

```python
# Case A: CC = 4, Cognitive = 9 (Deeply nested code, hard to read)
def process_orders(orders):
    for order in orders:                       # +1 CC, +1 Cog
        if order.is_active:                    # +1 CC, +2 Cog (nesting = 1)
            if order.is_paid:                  # +1 CC, +3 Cog (nesting = 2)
                fulfill(order)
            else:                              # +1 CC, +1 Cog
                cancel(order)

# Case B: CC = 4, Cognitive = 4 (Flat, modular code, easy to read)
def process_orders(orders):
    active_orders = [o for o in orders if o.is_active]
    paid_orders = [o for o in active_orders if o.is_paid]
    unpaid_orders = [o for o in active_orders if not o.is_paid]
    
    for order in paid_orders: fulfill(order)
    for order in unpaid_orders: cancel(order)
```

---

## Factors that Increase Cognitive Complexity
1. **Nesting Increment**: Each additional level of indentation (`if` inside `for` inside `try`) compounds cognitive weight.
2. **Linear Flow Disruptions**: Abrupt control jump operators (`break`, `continue`, mutual recursion).
3. **Mixed Boolean Sequences**: Alternating boolean operators without clear grouping (`a and b or c and not d`).

---

## Thresholds and Tools

| Cognitive Limit per Function | Diagnostic |
|------------------------------|------------|
| **$< 15$** | 🟢 Healthy and easy to maintain |
| **$\ge 15$** | 🔴 Excessive nesting; extract sub-functions |

### Measurement Tools

```bash
# Python: flake8-cognitive-complexity
pip install flake8-cognitive-complexity
flake8 --max-cognitive-complexity=15 src/

# SonarQube / SonarLint (JavaScript, TypeScript, Java, Python)
# Native automatic evaluation in editor and CI pipeline
```
