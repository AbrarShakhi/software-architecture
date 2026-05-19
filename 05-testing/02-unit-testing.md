# Unit Testing

> Test the smallest testable unit of code in isolation, fast and without external dependencies.

---

## What is a Unit?

A unit is typically a function or a class method. A unit test:
- Tests **one behavior** at a time
- Runs in **milliseconds** (no database, no HTTP, no file system)
- Is **fully deterministic** — same inputs, always same output
- Tests the unit **in isolation** (dependencies replaced with test doubles)

---

## The AAA Pattern

Every unit test follows three steps: **Arrange → Act → Assert**.

```python
import pytest
from decimal import Decimal

class TaxCalculator:
    def calculate(self, amount: Decimal, country: str) -> Decimal:
        rates = {"US": Decimal("0.08"), "UK": Decimal("0.20"), "DE": Decimal("0.19")}
        rate = rates.get(country, Decimal("0"))
        return (amount * rate).quantize(Decimal("0.01"))

def test_us_tax_rate():
    # ARRANGE: set up the test
    calculator = TaxCalculator()
    amount = Decimal("100.00")

    # ACT: call the unit under test
    tax = calculator.calculate(amount, country="US")

    # ASSERT: verify the expected outcome
    assert tax == Decimal("8.00")

def test_unknown_country_returns_zero_tax():
    calculator = TaxCalculator()
    tax = calculator.calculate(Decimal("100.00"), country="ZZ")
    assert tax == Decimal("0.00")

def test_tax_is_rounded_to_two_decimal_places():
    calculator = TaxCalculator()
    tax = calculator.calculate(Decimal("10.00"), country="US")  # 10 * 0.08 = 0.80
    assert tax == Decimal("0.80")
```

---

## Pytest Fixtures

Fixtures provide reusable setup and teardown for tests.

```python
import pytest
from unittest.mock import Mock, MagicMock

# Fixtures are injected by pytest via parameter names
@pytest.fixture
def mock_user_repo():
    repo = Mock()
    repo.find_by_id.return_value = {
        "id": 1,
        "name": "Alice",
        "email": "alice@example.com",
        "role": "user",
    }
    return repo

@pytest.fixture
def mock_email_service():
    return Mock()

@pytest.fixture
def user_service(mock_user_repo, mock_email_service):
    """Compose fixtures to build the service under test."""
    return UserService(repo=mock_user_repo, email=mock_email_service)

def test_get_user_returns_user_data(user_service, mock_user_repo):
    # ARRANGE: handled by fixtures above
    # ACT
    user = user_service.get_user(1)
    # ASSERT
    assert user["name"] == "Alice"
    mock_user_repo.find_by_id.assert_called_once_with(1)

def test_get_user_raises_when_not_found(user_service, mock_user_repo):
    mock_user_repo.find_by_id.return_value = None  # override fixture default
    with pytest.raises(UserNotFoundError):
        user_service.get_user(999)
```

---

## Parameterized Tests

Test the same behavior across multiple inputs without duplicating test code.

```python
import pytest

@pytest.mark.parametrize("amount, country, expected_tax", [
    ("100.00", "US", "8.00"),
    ("100.00", "UK", "20.00"),
    ("100.00", "DE", "19.00"),
    ("100.00", "JP", "0.00"),   # unknown country → 0
    ("0.00",   "US", "0.00"),   # zero amount
    ("1.11",   "US", "0.09"),   # rounding: 1.11 * 0.08 = 0.0888 → 0.09
])
def test_tax_calculation(amount, country, expected_tax):
    from decimal import Decimal
    calculator = TaxCalculator()
    assert calculator.calculate(Decimal(amount), country) == Decimal(expected_tax)
```

Each parameterized case becomes a separate test in pytest output:
```
test_tax_calculation[100.00-US-8.00] PASSED
test_tax_calculation[100.00-UK-20.00] PASSED
...
```

---

## Testing Exceptions

```python
def test_divide_raises_for_zero_denominator():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_divide_raises_with_descriptive_message():
    with pytest.raises(ValueError, match="Denominator cannot be zero"):
        divide(10, 0)

def test_invalid_email_raises_validation_error():
    with pytest.raises(ValidationError) as exc_info:
        validate_email("not-an-email")
    assert "Invalid email format" in str(exc_info.value)
```

---

## What NOT to Unit Test

```python
# DON'T test framework internals
def test_flask_route_registers():
    # Testing Flask's routing — not your code
    assert "/users" in app.url_map

# DON'T test simple getters/setters with no logic
def test_user_name_getter():
    user = User(name="Alice")
    assert user.name == "Alice"   # pure data, nothing to test

# DON'T test constructor assignment
def test_order_stores_items():
    items = ["item1", "item2"]
    order = Order(items=items)
    assert order.items == items   # this is just testing Python assignment

# DO test business logic
def test_order_cannot_be_placed_when_stock_is_zero():
    product = Product(stock=0)
    with pytest.raises(InsufficientStockError):
        product.reserve(quantity=1)

# DO test edge cases and boundaries
def test_order_discount_applied_for_orders_over_100():
    order = Order(total=Decimal("101.00"))
    assert order.apply_bulk_discount() == Decimal("90.90")

def test_order_no_discount_for_orders_exactly_100():
    order = Order(total=Decimal("100.00"))
    assert order.apply_bulk_discount() == Decimal("100.00")  # boundary: no discount AT 100
```

---

## Testing Pure Functions vs. Side Effects

**Pure functions** are the easiest to unit test:

```python
# Pure function: same input → always same output, no side effects
def format_currency(amount: Decimal, currency: str = "USD") -> str:
    symbols = {"USD": "$", "EUR": "€", "GBP": "£"}
    symbol = symbols.get(currency, currency)
    return f"{symbol}{amount:,.2f}"

def test_format_currency_usd():
    assert format_currency(Decimal("1234.56")) == "$1,234.56"

def test_format_currency_eur():
    assert format_currency(Decimal("99.99"), "EUR") == "€99.99"
```

**Functions with side effects** need mocks:

```python
from unittest.mock import patch, Mock

def test_send_welcome_email_called_on_registration():
    with patch("myapp.services.email_service.send") as mock_send:
        user_service.register("alice@example.com", "Alice")
        mock_send.assert_called_once()
        args = mock_send.call_args
        assert args[1]["to"] == "alice@example.com"

def test_current_time_used_in_timestamp():
    fixed_time = datetime(2024, 1, 15, 10, 30, 0)
    with patch("myapp.services.datetime") as mock_dt:
        mock_dt.now.return_value = fixed_time
        order = create_order(items=[])
        assert order.created_at == fixed_time
```

---

## Test Coverage

Coverage measures which lines of code are executed by tests.

```bash
# Install pytest-cov
pip install pytest-cov

# Run tests with coverage report
pytest --cov=myapp --cov-report=term-missing

# Output:
# Name                    Stmts   Miss  Cover   Missing
# myapp/services.py          45      3    93%   34, 67, 89
# myapp/models.py            30      0   100%
# TOTAL                      75      3    96%
```

**Coverage is a negative indicator, not a positive one:**
- 100% coverage does not mean 100% correctness.
- 0% coverage guarantees untested code.
- Aim for high coverage on business logic (domain + application layers).
- Don't chase 100% — infrastructure layer and glue code can have lower coverage.

---

## Key Takeaways

- Unit tests are fast, isolated, and deterministic — the foundation of your test suite.
- Follow Arrange-Act-Assert: setup, call, verify.
- Parameterize tests to cover multiple inputs without duplication.
- Test behavior (what the code does) not implementation (how it does it).
- Don't test trivial code (constructors, simple getters); do test business logic, edge cases, and exceptions.
- High coverage on domain and application logic; less important for framework glue code.
