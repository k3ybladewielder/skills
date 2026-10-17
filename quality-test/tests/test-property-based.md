# Property-Based Testing (Property-Based Testing Subskill)

## Overview
Rather than asserting against isolated hand-crafted examples (*example-based testing*), **property-based testing** validates that **universal logical invariants** hold across hundreds or thousands of pseudo-random inputs automatically synthesized by the test framework.

When a property check fails, the framework automatically performs **shrinking** (reducing the failing input down to the minimal reproducible counterexample that breaks the rule).

---

## When to Apply
- Pure and deterministic functions.
- Serialization and deserialization operations (roundtrip: $\text{deserialize}(\text{serialize}(x)) == x$).
- Commutative ($a + b == b + a$) or associative operations.
- Idempotent operations ($f(f(x)) == f(x)$).
- Sorting algorithms, parsers, compression routines, and mathematical filters.

---

## Implementation with Hypothesis (Python)

```python
# tests/properties/test_math_and_data.py
from hypothesis import given, strategies as st
from my_app.domain.calculator import apply_discount, sort_items

@given(st.lists(st.integers()))
def test_sort_invariants(items):
    """Sorting invariants: length preserved and elements retained."""
    sorted_items = sort_items(items)
    
    assert len(sorted_items) == len(items)
    assert set(sorted_items) == set(items)
    # Verify that each element is <= the subsequent element
    for i in range(len(sorted_items) - 1):
        assert sorted_items[i] <= sorted_items[i + 1]

@given(
    price=st.floats(min_value=0.01, max_value=1_000_000.0),
    pct=st.integers(min_value=0, max_value=100)
)
def test_discount_bounds(price, pct):
    """Discounted price must never be negative or exceed the original price."""
    final_price = apply_discount(price, pct)
    assert 0.0 <= final_price <= price

@given(st.dictionaries(keys=st.text(min_size=1), values=st.integers()))
def test_json_roundtrip_property(payload):
    """Roundtrip invariant: encode followed by decode reconstructs original object."""
    import json
    encoded = json.dumps(payload)
    decoded = json.loads(encoded)
    assert decoded == payload
```

---

## TypeScript Equivalent with fast-check

```typescript
// tests/properties/sort.spec.ts
import { test, expect } from 'vitest';
import fc from 'fast-check';

test('Integer array serialization roundtrip invariant', () => {
  fc.assert(
    fc.property(fc.array(fc.integer()), (arr) => {
      const serialized = JSON.stringify(arr);
      const deserialized = JSON.parse(serialized);
      expect(deserialized).toEqual(arr);
    })
  );
});
```
