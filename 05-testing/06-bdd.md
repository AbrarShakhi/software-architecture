# Behavior-Driven Development (BDD)

> Write tests in plain language that describes system behavior from the user's perspective — using a shared vocabulary between developers, testers, and business stakeholders.

---

## The Problem BDD Solves

TDD tests describe behavior, but in code. Only developers can read them. When product owners or QA engineers want to understand what the system does, they need to read Python — a barrier that leads to:

- Requirements being "lost in translation" between business and engineering
- Tests that pass but don't actually verify the right thing
- No shared language between roles

BDD solves this by expressing tests in near-English structured prose (Gherkin) that any stakeholder can read and write.

---

## Gherkin — The BDD Language

Gherkin is a structured language for writing test scenarios. Every scenario has three sections:

```gherkin
Given  [some initial context / precondition]
When   [an action is performed]
Then   [an expected outcome]
```

These map to **Arrange → Act → Assert** from unit testing, but in human-readable form.

```gherkin
Feature: User Registration

  Scenario: Successful registration with valid data
    Given no account exists with email "alice@example.com"
    When a user registers with name "Alice" and email "alice@example.com" and password "SecurePass1!"
    Then the account is created successfully
    And a welcome email is sent to "alice@example.com"

  Scenario: Registration fails with duplicate email
    Given an account already exists with email "alice@example.com"
    When a user tries to register with email "alice@example.com"
    Then registration is rejected with error "Email already in use"
    And no email is sent
```

---

## BDD Tool: pytest-bdd

`pytest-bdd` lets you write Gherkin feature files and connect them to Python step definitions.

### Install

```bash
pip install pytest-bdd
```

---

## Full Example: E-Commerce Shopping Cart

### Feature File

```gherkin
# tests/features/shopping_cart.feature

Feature: Shopping Cart

  Background:
    Given the user is logged in as "alice@example.com"

  Scenario: Add item to empty cart
    Given the cart is empty
    When the user adds 1 "Widget Pro" at $29.99
    Then the cart contains 1 item
    And the cart total is $29.99

  Scenario: Apply bulk discount over $100
    Given the cart is empty
    When the user adds 2 "Widget Pro" at $29.99
    And the user adds 1 "Deluxe Pack" at $59.99
    Then the cart total before discount is $119.97
    And a 10% discount is applied
    And the cart total is $107.97

  Scenario: Remove item from cart
    Given the cart contains 1 "Widget Pro" at $29.99
    When the user removes "Widget Pro" from the cart
    Then the cart is empty
    And the cart total is $0.00

  Scenario Outline: Tax applied by country
    Given the cart contains 1 "Widget Pro" at $100.00
    When the user sets their country to "<country>"
    Then the tax amount is $<tax>

    Examples:
      | country | tax   |
      | US      | 8.00  |
      | UK      | 20.00 |
      | DE      | 19.00 |
      | JP      | 0.00  |
```

### Step Definitions

```python
# tests/step_defs/test_shopping_cart.py
from decimal import Decimal
import pytest
from pytest_bdd import given, when, then, parsers, scenarios

from myapp.cart import ShoppingCart, Item
from myapp.auth import get_user_by_email
from myapp.tax import TaxCalculator

# Bind all scenarios in the feature file
scenarios("../features/shopping_cart.feature")


# ─── Fixtures ───────────────────────────────────────────────────────────────

@pytest.fixture
def cart():
    return ShoppingCart()

@pytest.fixture
def current_user():
    return {}


# ─── Given steps ────────────────────────────────────────────────────────────

@given(parsers.parse('the user is logged in as "{email}"'))
def logged_in_user(current_user, email):
    current_user["email"] = email

@given("the cart is empty")
def empty_cart(cart):
    assert len(cart.items()) == 0

@given(parsers.parse('the cart contains 1 "{name}" at ${price:f}'))
def cart_with_item(cart, name, price):
    cart.add(Item(name=name, price=Decimal(str(price))))


# ─── When steps ─────────────────────────────────────────────────────────────

@when(parsers.parse('the user adds {qty:d} "{name}" at ${price:f}'))
def add_item_to_cart(cart, qty, name, price):
    cart.add(Item(name=name, price=Decimal(str(price))), quantity=qty)

@when(parsers.parse('the user removes "{name}" from the cart'))
def remove_item_from_cart(cart, name):
    cart.remove(name)

@when(parsers.parse('the user sets their country to "{country}"'))
def set_country(cart, country):
    cart.set_country(country)


# ─── Then steps ─────────────────────────────────────────────────────────────

@then(parsers.parse("the cart contains {count:d} item"))
@then(parsers.parse("the cart contains {count:d} items"))
def cart_item_count(cart, count):
    assert len(cart.items()) == count

@then("the cart is empty")
def cart_is_empty(cart):
    assert len(cart.items()) == 0

@then(parsers.parse("the cart total is ${expected:f}"))
def cart_total(cart, expected):
    assert cart.total() == Decimal(str(expected)).quantize(Decimal("0.01"))

@then(parsers.parse("the cart total before discount is ${expected:f}"))
def cart_subtotal(cart, expected):
    assert cart.subtotal() == Decimal(str(expected)).quantize(Decimal("0.01"))

@then("a 10% discount is applied")
def discount_applied(cart):
    assert cart.discount_percent() == Decimal("10")

@then(parsers.parse("the tax amount is ${expected:f}"))
def tax_amount(cart, expected):
    assert cart.tax() == Decimal(str(expected)).quantize(Decimal("0.01"))
```

---

## Example: Authentication Feature

```gherkin
# tests/features/authentication.feature

Feature: User Authentication

  Scenario: Successful login
    Given a user exists with email "bob@example.com" and password "Pass1234!"
    When the user submits login with email "bob@example.com" and password "Pass1234!"
    Then the user receives a valid JWT token
    And the token expires in 24 hours

  Scenario: Login fails with wrong password
    Given a user exists with email "bob@example.com" and password "Pass1234!"
    When the user submits login with email "bob@example.com" and password "wrongpass"
    Then login is rejected with status 401
    And the response contains error "Invalid credentials"

  Scenario: Login fails for non-existent user
    Given no user exists with email "ghost@example.com"
    When the user submits login with email "ghost@example.com" and password "any"
    Then login is rejected with status 401

  Scenario: Token is rejected after logout
    Given the user is logged in with a valid token
    When the user logs out
    Then the token is invalidated
    And subsequent requests with the old token return 401
```

```python
# tests/step_defs/test_authentication.py
import pytest
from datetime import datetime, timedelta
from pytest_bdd import given, when, then, parsers, scenarios

scenarios("../features/authentication.feature")


@pytest.fixture
def response():
    return {}

@pytest.fixture
def user_store():
    return {}


@given(parsers.parse('a user exists with email "{email}" and password "{password}"'))
def existing_user(user_store, email, password):
    from myapp.auth import UserService
    svc = UserService()
    user_store["user"] = svc.create_user(email=email, password=password)

@given(parsers.parse('no user exists with email "{email}"'))
def no_such_user(email):
    pass   # no setup needed; the user simply doesn't exist

@when(parsers.parse('the user submits login with email "{email}" and password "{password}"'))
def submit_login(response, email, password):
    from myapp.auth import AuthService
    svc = AuthService()
    response.update(svc.login(email=email, password=password))

@then("the user receives a valid JWT token")
def has_jwt(response):
    assert "token" in response
    assert len(response["token"]) > 20

@then("the token expires in 24 hours")
def token_expiry(response):
    from myapp.auth import decode_token
    payload = decode_token(response["token"])
    expected_expiry = datetime.utcnow() + timedelta(hours=24)
    assert abs((payload["exp"] - expected_expiry).total_seconds()) < 60

@then(parsers.parse("login is rejected with status {status:d}"))
def login_rejected(response, status):
    assert response.get("status") == status

@then(parsers.parse('the response contains error "{message}"'))
def error_message(response, message):
    assert message in response.get("error", "")
```

---

## Scenario Outline — Data-Driven BDD

`Scenario Outline` runs the same scenario for each row in an `Examples` table:

```gherkin
Feature: Password Validation

  Scenario Outline: Reject invalid passwords
    When a user registers with password "<password>"
    Then registration is rejected with error "<error>"

    Examples:
      | password      | error                                  |
      | short         | at least 8 characters                  |
      | alllowercase1!| at least one uppercase letter          |
      | ALLUPPERCASE! | at least one digit                     |
      | NoSpecial123  | at least one special character         |
```

---

## Background — Shared Setup Across Scenarios

`Background` runs before every scenario in a feature file, replacing repeated `Given` steps:

```gherkin
Feature: Order Management

  Background:
    Given the user is logged in
    And the following products exist:
      | name       | price | stock |
      | Widget Pro | 29.99 | 10    |
      | Deluxe Kit | 99.99 | 3     |

  Scenario: Place a valid order
    When the user orders 1 "Widget Pro"
    Then the order status is "CONFIRMED"
    And the stock for "Widget Pro" is 9

  Scenario: Cannot order more than available stock
    When the user tries to order 11 "Widget Pro"
    Then the order is rejected with "Insufficient stock"
```

---

## BDD vs TDD

| | TDD | BDD |
|-|-----|-----|
| **Language** | Code (Python) | Gherkin (near-English) |
| **Written by** | Developers | Devs + QA + Product |
| **Focus** | Unit / function behavior | Business feature behavior |
| **Test scope** | Unit, integration | Integration, acceptance |
| **Primary value** | Code design feedback | Shared understanding |

BDD is not a replacement for TDD. Use both:
- TDD for unit-level design and fast feedback on logic
- BDD for acceptance criteria that business stakeholders define and review

---

## File Structure

```
tests/
├── features/
│   ├── authentication.feature
│   ├── shopping_cart.feature
│   └── order_management.feature
├── step_defs/
│   ├── test_authentication.py
│   ├── test_shopping_cart.py
│   └── test_order_management.py
└── conftest.py       ← shared fixtures (db, app client, etc.)
```

---

## Running BDD Tests

```bash
# Run all BDD tests
pytest tests/step_defs/

# Run with verbose feature output
pytest tests/step_defs/ -v

# Run a specific feature
pytest tests/step_defs/test_shopping_cart.py

# Generate a plain-language test report
pytest tests/step_defs/ --bdd-feature-base-dir tests/features/
```

Output:
```
PASSED  tests/step_defs/test_shopping_cart.py::test_add_item_to_empty_cart
PASSED  tests/step_defs/test_shopping_cart.py::test_apply_bulk_discount_over_100
PASSED  tests/step_defs/test_shopping_cart.py::test_remove_item_from_cart
PASSED  tests/step_defs/test_shopping_cart.py::test_tax_applied_by_country[US-8.00]
PASSED  tests/step_defs/test_shopping_cart.py::test_tax_applied_by_country[UK-20.00]
PASSED  tests/step_defs/test_shopping_cart.py::test_tax_applied_by_country[DE-19.00]
PASSED  tests/step_defs/test_shopping_cart.py::test_tax_applied_by_country[JP-0.00]
```

---

## Key Takeaways

- BDD uses Gherkin (Given-When-Then) to write tests in near-English readable by any stakeholder.
- `pytest-bdd` connects `.feature` files to Python step definitions — tests run the same as any pytest tests.
- `Scenario Outline` + `Examples` table replaces parameterized test boilerplate.
- `Background` handles shared preconditions without repeating `Given` steps in every scenario.
- BDD and TDD are complementary: TDD drives code design at the unit level; BDD captures business requirements at the acceptance level.
