# Regression Testing (Regression Testing Subskill)

## Overview
Validates that existing functionality continues to work without regressions after code modifications. Regression is not an isolated testing tier, but a **purpose** applied across the entire testing pyramid.

## Primary Strategies

| Strategy | Ideal Scenario | Advantage / Trade-off |
|----------|----------------|-----------------------|
| **Re-run All** | Fast builds (< 10 min) | Simplicity; linear computational overhead |
| **Test Impact Analysis (TIA)** | Monorepos and extensive suites | Executes solely tests affected by the PR diff |
| **Snapshot Testing** | API schemas, JSON responses, and payloads | Immediate alerts against accidental contract modifications |
| **Contract Testing (Pact)** | Microservice architectures | Ensures interoperability without requiring remote services running |
| **Golden File Testing** | Compilers, parsers, and data pipelines | Compares produced output against verified canonical files |
| **Visual Regression** | UI components (Storybook, Playwright) | Detects accidental pixel-level CSS/layout regressions |

---

## Practical Implementations

### 1. Snapshot Testing for API Contracts (Python)

```python
# tests/snapshots/test_user_api.py
import json
from pathlib import Path

SNAPSHOT_DIR = Path(__file__).parent / "snapshots"

def test_user_response_contract(client):
    """Ensures API response schema and fields do not mutate unintentionally."""
    response = client.get("/api/v1/users/1")
    assert response.status_code == 200
    
    snapshot_file = SNAPSHOT_DIR / "user_response_schema.json"
    if snapshot_file.exists():
        expected_schema = json.loads(snapshot_file.read_text())
        assert response.json().keys() == expected_schema.keys(), (
            "Response contract modified! If intentional, update the snapshot deliberately."
        )
    else:
        snapshot_file.write_text(json.dumps(response.json(), indent=2))
```

### 2. Contract Testing with Pact (TypeScript / Python)

```python
# tests/contract/test_consumer.py
from pact import Consumer, Provider

pact = Consumer("WebFrontend").has_pact_with(Provider("OrdersBackend"))

def test_get_order_contract(orders_client):
    expected_order = {"id": "ord_123", "amount": 150.0, "status": "approved"}
    
    (pact
        .given("order ord_123 exists")
        .upon_receiving("a request for order ord_123")
        .with_request("GET", "/api/v1/orders/ord_123")
        .will_respond_with(200, body=expected_order))
    
    with pact:
        result = orders_client.get_order("ord_123")
        assert result["status"] == "approved"
```

---

## CI / CD Organization
- Separate fast regression tests from slow tests using test tags/markers (`@pytest.mark.slow`, `vitest -t`).
- **PR Gate**: Runs fast regression suite (< 5 min).
- **Nightly Pipeline**: Executes comprehensive regression, contract testing, and end-to-end simulations.
