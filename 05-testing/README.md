# 05 — Testing

> Strategies and patterns for building confidence that your code works correctly — now and after every future change.

Testing is not just about catching bugs. It's about:
- **Confidence** to refactor without fear
- **Documentation** of what the code is supposed to do
- **Design pressure** — code that is hard to test is usually hard to understand and change

## Topics

| # | Topic | Core Idea |
|---|-------|-----------|
| 1 | [Testing Fundamentals](./01-testing-fundamentals.md) | The testing pyramid and types of test doubles |
| 2 | [Unit Testing](./02-unit-testing.md) | Test small, fast, in isolation |
| 3 | [Integration Testing](./03-integration-testing.md) | Test how components work together |
| 4 | [End-to-End Testing](./04-e2e-testing.md) | Test the full user journey |
| 5 | [TDD](./05-tdd.md) | Write tests before code |
| 6 | [BDD](./06-bdd.md) | Write tests in the language of the business |

## The Testing Pyramid

```
              ╱─────────────╲
             ╱  End-to-End   ╲   ← Few, slow, expensive, high confidence
            ╱─────────────────╲
           ╱   Integration     ╲  ← Some, medium speed
          ╱─────────────────────╲
         ╱      Unit Tests       ╲ ← Many, fast, cheap, isolated
        ╱─────────────────────────╲
```

Invest most in unit tests, some in integration tests, few in E2E tests.
