# Martin Metrics: Abstraction, Instability, and Distance (Martin Metrics Subskill)

## Overview
Metrics formulated by Robert C. Martin ("Uncle Bob") to evaluate the architectural balance between stability and abstractness across packages and subsystems.

## Fundamental Formulas

### 1. Abstractness ($A$)
$$A = \frac{\text{Abstract Classes} + \text{Interfaces}}{\text{Total Classes in Package}}$$
- $A = 0$: 100% concrete module.
- $A = 1$: 100% abstract module (pure contracts/interfaces).

### 2. Instability ($I$)
$$I = \frac{C_e}{C_a + C_e}$$
- $I = 0$: Completely **stable** (many depend on it, it depends on none).
- $I = 1$: Completely **unstable** (no one depends on it, it depends on many).

### 3. Distance from the Main Sequence ($D$)
$$D = |A + I - 1|$$
- $D = 0$: Optimal balance (located precisely on the Main Sequence).
- $D \to 1$: Architectural imbalance (maintenance hazard).

---

## Diagram: Risk Zones

```
A (Abstractness)
1.0 ┌─────────────────────────┐
    │  ZONE OF              / │
    │  USELESSNESS        /   │  (Overly abstract, no consumers)
    │                   /     │
    │                 /       │
    │               /         │
    │             /   Main    │
    │           /   Sequence  │ (Ideal: A + I ≈ 1)
    │         /               │
    │       /   ZONE OF PAIN  │
    │     /                   │  (Overly concrete, hard to modify)
    │   /                     │
0.0 └─────────────────────────┘
    0.0                     1.0
              I (Instability)
```

- **Zone of Pain** ($A \approx 0, I \approx 0$): Rigid component with heavy inbound dependents, difficult to change without wide regressions.
- **Zone of Uselessness** ($A \approx 1, I \approx 1$): Abstract, unstable package with no real dependents; accidental complexity.
- **Main Sequence** ($A + I \approx 1$): Abstractness is directly proportional to stability.

---

## Automated Measurement (Python)

```python
# scripts/measure_martin_metrics.py
from dataclasses import dataclass

@dataclass
class PackageMartinMetrics:
    name: str
    num_classes: int
    num_abstract: int
    ca: int
    ce: int
    
    @property
    def abstraction(self) -> float:
        return self.num_abstract / max(self.num_classes, 1)
        
    @property
    def instability(self) -> float:
        return self.ce / max(self.ca + self.ce, 1)
        
    @property
    def distance(self) -> float:
        return abs(self.abstraction + self.instability - 1.0)
        
    @property
    def diagnosis(self) -> str:
        if self.distance < 0.3:
            return "✅ On the Main Sequence"
        if self.abstraction < 0.3 and self.instability < 0.3:
            return "🔴 Zone of Pain (Overly rigid)"
        if self.abstraction > 0.7 and self.instability > 0.7:
            return "🟡 Zone of Uselessness (Unused abstraction)"
        return "⚠️ Outside Main Sequence"
```

## Engineering Thresholds
- $D < 0.3$: **Healthy**.
- $0.3 \le D \le 0.5$: **Warning**.
- $D > 0.5$: **Refactor** (extract interfaces to decouple or eliminate redundant abstractions).
