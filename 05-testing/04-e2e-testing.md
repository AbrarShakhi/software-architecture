# End-to-End Testing

> Test the complete user journey through a real, running application — from the browser (or API client) through all services to the database and back.

---

## The Problem E2E Solves

Unit tests verify logic. Integration tests verify components. But neither verifies that the entire system works together as a user experiences it:

- The login form renders correctly
- Submitting triggers the right API call
- The API correctly persists to the database
- The success message appears on screen
- The confirmation email is sent

E2E tests verify this entire journey.

---

## E2E vs. Integration vs. Unit

| | Unit | Integration | E2E |
|-|------|-------------|-----|
| **What runs** | Logic in isolation | Multiple components | Full application stack |
| **Browser/UI** | No | No | Yes |
| **Database** | Fake/Mock | Real (test DB) | Real |
| **Speed** | Milliseconds | Seconds | Seconds–Minutes |
| **Flakiness** | None | Low | Can be high |

---

## Playwright — Modern E2E Testing

Playwright is the recommended tool for browser automation in Python. It runs Chromium, Firefox, and WebKit.

```python
import pytest
from playwright.sync_api import Page, expect

# Tests use a real browser navigating a running application

def test_user_can_register_and_login(page: Page, base_url: str):
    # Navigate to registration page
    page.goto(f"{base_url}/register")

    # Fill the form
    page.get_by_label("Name").fill("Alice Smith")
    page.get_by_label("Email").fill("alice@example.com")
    page.get_by_label("Password").fill("SecurePass123!")
    page.get_by_label("Confirm Password").fill("SecurePass123!")
    page.get_by_role("button", name="Create Account").click()

    # Verify redirect to login page with success message
    expect(page).to_have_url(f"{base_url}/login")
    expect(page.get_by_text("Account created successfully")).to_be_visible()

    # Log in
    page.get_by_label("Email").fill("alice@example.com")
    page.get_by_label("Password").fill("SecurePass123!")
    page.get_by_role("button", name="Sign In").click()

    # Verify dashboard loads
    expect(page).to_have_url(f"{base_url}/dashboard")
    expect(page.get_by_text("Welcome, Alice")).to_be_visible()

def test_checkout_flow(page: Page, base_url: str, logged_in_user):
    page.goto(f"{base_url}/products")

    # Add item to cart
    page.get_by_text("Widget Pro").click()
    page.get_by_role("button", name="Add to Cart").click()

    # Proceed to checkout
    page.get_by_role("link", name="Cart (1)").click()
    page.get_by_role("button", name="Proceed to Checkout").click()

    # Fill payment details
    page.get_by_label("Card Number").fill("4242424242424242")
    page.get_by_label("Expiry").fill("12/28")
    page.get_by_label("CVV").fill("123")
    page.get_by_role("button", name="Place Order").click()

    # Verify success
    expect(page.get_by_text("Order Confirmed")).to_be_visible(timeout=10000)
    expect(page.get_by_text("Order #")).to_be_visible()
```

---

## Page Object Model (POM)

Encapsulate page-specific selectors and actions in a class. Tests become readable and selectors are maintained in one place.

```python
# pages/login_page.py
from playwright.sync_api import Page, expect

class LoginPage:
    def __init__(self, page: Page, base_url: str):
        self._page = page
        self._url = f"{base_url}/login"

    def navigate(self) -> "LoginPage":
        self._page.goto(self._url)
        return self

    def fill_email(self, email: str) -> "LoginPage":
        self._page.get_by_label("Email").fill(email)
        return self

    def fill_password(self, password: str) -> "LoginPage":
        self._page.get_by_label("Password").fill(password)
        return self

    def submit(self) -> "DashboardPage":
        self._page.get_by_role("button", name="Sign In").click()
        return DashboardPage(self._page, self._url.replace("/login", ""))

    def expect_error(self, message: str) -> None:
        expect(self._page.get_by_role("alert")).to_contain_text(message)


# pages/dashboard_page.py
class DashboardPage:
    def __init__(self, page: Page, base_url: str):
        self._page = page
        self._url = f"{base_url}/dashboard"

    def expect_loaded(self) -> "DashboardPage":
        expect(self._page).to_have_url(self._url)
        return self

    def get_username_display(self) -> str:
        return self._page.get_by_test_id("username-display").inner_text()


# Test: readable, no raw selectors
def test_login_success(page: Page, base_url: str):
    dashboard = (
        LoginPage(page, base_url)
        .navigate()
        .fill_email("alice@example.com")
        .fill_password("SecurePass123!")
        .submit()
    )
    dashboard.expect_loaded()
    assert "Alice" in dashboard.get_username_display()

def test_login_with_wrong_password(page: Page, base_url: str):
    login = LoginPage(page, base_url).navigate()
    login.fill_email("alice@example.com").fill_password("wrongpass").submit()
    login.expect_error("Invalid credentials")
```

---

## API E2E Tests (Without Browser)

For APIs, E2E tests hit the running server directly with HTTP.

```python
import httpx
import pytest

BASE_URL = "http://localhost:8000"

@pytest.fixture(scope="session")
def api_client():
    return httpx.Client(base_url=BASE_URL, timeout=10.0)

@pytest.fixture
def auth_token(api_client) -> str:
    """Get a real JWT by logging in."""
    response = api_client.post("/auth/login", json={
        "email": "testuser@example.com",
        "password": "TestPass123!",
    })
    assert response.status_code == 200
    return response.json()["token"]

def test_full_order_lifecycle(api_client, auth_token):
    headers = {"Authorization": f"Bearer {auth_token}"}

    # Create order
    response = api_client.post("/api/orders", headers=headers, json={
        "product_id": 1,
        "quantity": 2,
    })
    assert response.status_code == 201
    order_id = response.json()["data"]["id"]

    # Verify order exists
    response = api_client.get(f"/api/orders/{order_id}", headers=headers)
    assert response.status_code == 200
    assert response.json()["data"]["status"] == "PENDING"

    # Cancel order
    response = api_client.delete(f"/api/orders/{order_id}", headers=headers)
    assert response.status_code == 200

    # Verify cancelled
    response = api_client.get(f"/api/orders/{order_id}", headers=headers)
    assert response.json()["data"]["status"] == "CANCELLED"
```

---

## Avoiding Flakiness

E2E tests are notorious for flakiness (tests that fail intermittently without code changes).

```python
# Flakiness Source 1: Hardcoded waits
# BAD: sleep(2) is arbitrary and brittle
page.click("#submit")
time.sleep(2)                          # breaks if server is slow
assert page.is_visible("#success")

# GOOD: Wait for element, not time
page.click("#submit")
expect(page.locator("#success")).to_be_visible(timeout=10000)  # up to 10 seconds


# Flakiness Source 2: Shared test data
# BAD: Tests share user accounts — test A's data affects test B
def test_a():
    user = User.objects.get(email="test@example.com")
    ...

def test_b():
    user = User.objects.get(email="test@example.com")  # might be modified by test A!
    ...

# GOOD: Each test creates its own isolated data
@pytest.fixture
def unique_user(api_client) -> dict:
    """Creates a unique user for this test run."""
    email = f"test_{uuid4().hex[:8]}@example.com"
    user = api_client.post("/api/users", json={"email": email, ...}).json()
    yield user
    api_client.delete(f"/api/users/{user['id']}")   # cleanup


# Flakiness Source 3: Animation/timing
# BAD: clicking before animation completes
page.click("#menu-button")
page.click("#logout")   # menu animation not done yet

# GOOD: Wait for the element to be stable
page.click("#menu-button")
logout_item = page.get_by_text("Logout")
logout_item.wait_for(state="visible")
logout_item.click()
```

---

## Running E2E Tests in CI

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on: [push, pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: Install Python dependencies
        run: pip install -r requirements.txt

      - name: Install Playwright browsers
        run: playwright install --with-deps chromium

      - name: Start application
        run: python app.py &
        env:
          DATABASE_URL: postgresql://postgres:test@localhost/test_db

      - name: Wait for app to be ready
        run: |
          until curl -f http://localhost:8000/health; do sleep 1; done

      - name: Run E2E tests
        run: pytest tests/e2e/ -v --screenshot=only-on-failure

      - name: Upload screenshots on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-screenshots
          path: test-results/
```

---

## Key Takeaways

- E2E tests verify the full user journey — login, checkout, data persistence — from the user's perspective.
- Use Playwright for browser tests; it has better debugging, auto-waiting, and multi-browser support than Selenium.
- The Page Object Model keeps selectors in one place and makes tests readable.
- Fight flakiness: use element waits not `sleep()`, create isolated test data, avoid shared accounts.
- E2E tests are slow and expensive — run fewer of them. Focus on the critical user paths (happy path + most important error scenarios).
