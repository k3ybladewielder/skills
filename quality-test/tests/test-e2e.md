# End-to-End Testing (E2E Testing Subskill)

## Overview
Validates that the system as a cohesive whole satisfies complete end-to-end user journeys, crossing all architectural integration tiers (UI $\to$ API Gateway $\to$ Services $\to$ Database & Messaging).

## Testing Pyramid and Target Proportions

```
          /\          E2E: 5% to 10% (few, slower, high business value)
         /  \
        /    \        Integration: 20% to 30% (APIs, persistence, messaging)
       /      \
      /        \      Unit: 60% to 70% (fast, isolated, exhaustive)
     /__________\
```

---

## Implementation Patterns

### 1. API-first E2E (Recommended Robust Baseline)
Significantly more resilient and less prone to UI churn than purely browser-driven assertions.

```python
# tests/e2e/test_order_journey.py
import pytest
from httpx import AsyncClient

@pytest.mark.e2e
async def test_complete_order_journey(client: AsyncClient):
    # 1. Authentication and user registration
    signup = await client.post("/api/v1/auth/signup", json={
        "email": "user.e2e@example.com", "password": "StrongPassword123!"
    })
    assert signup.status_code == 201
    auth_token = signup.json()["token"]
    headers = {"Authorization": f"Bearer {auth_token}"}
    
    # 2. Add product to shopping cart
    cart_res = await client.post("/api/v1/cart/items", json={
        "product_id": "prod_456", "quantity": 1
    }, headers=headers)
    assert cart_res.status_code == 200
    
    # 3. Checkout and payment processing
    checkout = await client.post("/api/v1/checkout", json={
        "payment_method": "credit_card"
    }, headers=headers)
    assert checkout.status_code == 201
    order_id = checkout.json()["order_id"]
    
    # 4. Status confirmation and persistence verification
    order = await client.get(f"/api/v1/orders/{order_id}", headers=headers)
    assert order.status_code == 200
    assert order.json()["status"] == "confirmed"
```

### 2. UI E2E with Playwright (TypeScript)

```typescript
// tests/e2e/login-dashboard.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Dashboard User Journey', () => {
  test('Successful authentication and navigation', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('E-mail').fill('alysson@example.com');
    await page.getByLabel('Password').fill('Secret123!');
    await page.getByRole('button', { name: /sign in/i }).click();
    
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.getByTestId('welcome-card')).toBeVisible();
  });
});
```

---

## Best Practices and Rules
- **Restricted Volume**: Maintain between 10 and 30 critical end-to-end journeys (core happy paths and blocking failures).
- **State Isolation**: Each test provisions its own fixture data tagged with unique identifiers (`uuid`) and performs teardown.
- **Flakiness Management**: Retry at most 2 times in CI. Persistent intermittent failures must be treated as actual software bugs, not ignored.
- **Smoke Test Differentiation**: Maintain a concise post-deploy verification subset capable of completing in under 3 minutes.
