# Performance Testing and Benchmarks (Performance Testing Subskill)

## Overview
Ensures that software retains acceptable execution duration, latency, and memory consumption characteristics across development cycles, preventing silent performance regressions prior to production release.

---

## Types of Performance Tests

| Type | Objective | Primary Focus |
|------|-----------|---------------|
| **Micro-Benchmark** | Measures execution runtime of isolated functions/algorithms | $O(N)$ vs $O(N^2)$, memory allocation |
| **Performance Budget** | Asserts that endpoints and queries stay below fixed upper bounds | E.g.: `endpoint < 150ms`, `query < 30ms` |
| **Load Testing** | Assesses system behavior under anticipated sustained traffic | Throughput (RPS), $p95$ and $p99$ percentiles |
| **Stress Testing** | Identifies breaking points and system degradation boundaries | Graceful failure under severe overload |
| **Soak Testing** | Validates stability under prolonged moderate load (hours/days) | Memory leak and connection leak detection |

---

## Implementation with pytest-benchmark (Python)

```python
# tests/benchmarks/test_serialization_perf.py
import pytest
from my_app.services.serializer import fast_json_dump

@pytest.mark.benchmark(group="serialization")
def test_serialize_10k_records(benchmark):
    dataset = [{"id": i, "val": f"item_{i}"} for i in range(10_000)]
    
    # benchmark repeatedly executes the target with rigorous statistical measurement
    result = benchmark(fast_json_dump, dataset)
    
    assert len(result) > 0
    # Performance Budget: mean duration must not exceed 15 milliseconds
    assert benchmark.stats["mean"] < 0.015, f"Regression: {benchmark.stats['mean']:.4f}s > 15ms"
```

```bash
# Execute benchmarks and persist history for regression comparison
pytest tests/benchmarks/ --benchmark-autosave --benchmark-compare
```

---

## Declarative Load Testing with k6 (JavaScript / TypeScript)

```javascript
// tests/perf/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },  // Ramp-up to 50 VUs
    { duration: '1m', target: 50 },   // Steady-state load
    { duration: '20s', target: 0 },   // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'], // 95% of requests must resolve under 200ms
    http_req_failed: ['rate<0.01'],    // Error rate must remain under 1%
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api/v1/health');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(0.1);
}
```

```bash
k6 run tests/perf/load-test.js
```
