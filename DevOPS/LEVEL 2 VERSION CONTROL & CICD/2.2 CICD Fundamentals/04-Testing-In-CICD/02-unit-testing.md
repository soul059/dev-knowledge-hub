# Chapter 4.2: Unit Testing in CI/CD

[← Previous: Test Pyramid](01-test-pyramid.md) | [Back to README](../README.md) | [Next: Integration Testing →](03-integration-testing.md)

---

## Overview

**Unit tests** verify the smallest pieces of code (functions, methods, classes) in isolation. They form the foundation of the test pyramid, providing the fastest feedback and catching logic errors early.

---

## Unit Testing Flow in CI

```
  ┌──────────────┐    ┌───────────────┐    ┌───────────────┐
  │  Code Push   │───►│  Install Deps │───►│  Run Unit     │
  │              │    │               │    │  Tests        │
  └──────────────┘    └───────────────┘    └──────┬────────┘
                                                  │
                                           ┌──────▼──────┐
                                    ┌──────┤  All Pass?  ├─────┐
                                    │ YES  └─────────────┘ NO  │
                                    ▼                          ▼
                             ┌──────────┐              ┌──────────┐
                             │ Generate │              │ Fail     │
                             │ Coverage │              │ Build    │
                             │ Report   │              │ ✕        │
                             └──────────┘              └──────────┘
```

---

## Frameworks by Language

| Language | Frameworks | Runner | Coverage |
|---|---|---|---|
| **JavaScript** | Jest, Vitest, Mocha | jest, vitest | istanbul / c8 |
| **Python** | pytest, unittest | pytest | coverage.py |
| **Java** | JUnit 5, TestNG | Maven Surefire | JaCoCo |
| **Go** | testing (stdlib) | `go test` | `go test -cover` |
| **C#/.NET** | xUnit, NUnit, MSTest | `dotnet test` | Coverlet |
| **Rust** | cargo test (built-in) | `cargo test` | cargo-tarpaulin |

---

## Code Examples

### JavaScript (Jest)

```javascript
// src/calculator.js
function add(a, b) {
  return a + b;
}

function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

module.exports = { add, divide };

// __tests__/calculator.test.js
const { add, divide } = require('../src/calculator');

describe('Calculator', () => {
  describe('add', () => {
    test('adds two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });
    test('handles negative numbers', () => {
      expect(add(-1, -1)).toBe(-2);
    });
  });

  describe('divide', () => {
    test('divides evenly', () => {
      expect(divide(10, 2)).toBe(5);
    });
    test('throws on division by zero', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
    });
  });
});
```

### Python (pytest)

```python
# src/calculator.py
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

# tests/test_calculator.py
import pytest
from src.calculator import add, divide

class TestAdd:
    def test_positive_numbers(self):
        assert add(2, 3) == 5

    def test_negative_numbers(self):
        assert add(-1, -1) == -2

class TestDivide:
    def test_even_division(self):
        assert divide(10, 2) == 5.0

    def test_division_by_zero(self):
        with pytest.raises(ValueError, match="Division by zero"):
            divide(10, 0)
```

### Java (JUnit 5)

```java
// src/main/java/com/example/Calculator.java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double divide(int a, int b) {
        if (b == 0) throw new ArithmeticException("Division by zero");
        return (double) a / b;
    }
}

// src/test/java/com/example/CalculatorTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    private final Calculator calc = new Calculator();

    @Test void addsPositiveNumbers() {
        assertEquals(5, calc.add(2, 3));
    }

    @Test void throwsOnDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }
}
```

---

## CI Configuration — Unit Tests

```yaml
# GitHub Actions
name: Unit Tests
on: [push, pull_request]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]     # Test across versions
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - run: npm ci
      - run: npm test -- --coverage --ci

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage/lcov.info
          fail_ci_if_error: true
```

---

## Code Coverage

```
  COVERAGE REPORT
  ════════════════════════════════════════════
  File              │ Stmts │ Branch │ Funcs │ Lines │
  ──────────────────┼───────┼────────┼───────┼───────│
  src/calculator.js │ 100%  │ 100%   │ 100%  │ 100%  │
  src/validator.js  │  85%  │  70%   │  80%  │  85%  │
  src/parser.js     │  60%  │  45%   │  75%  │  60%  │ ⚠ Below threshold
  ──────────────────┼───────┼────────┼───────┼───────│
  Total             │  82%  │  72%   │  85%  │  82%  │
  ════════════════════════════════════════════

  Coverage Thresholds:
    ✓ Lines:     82% >= 80%
    ✕ Branches:  72% < 80%     ← BUILD FAILS
```

### Coverage Configuration (Jest)

```json
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    },
    "collectCoverageFrom": [
      "src/**/*.{js,ts}",
      "!src/**/*.d.ts",
      "!src/**/index.{js,ts}"
    ]
  }
}
```

---

## Best Practices for Unit Tests in CI

```
  ┌─────────────────────────────────────────────────────┐
  │            UNIT TEST BEST PRACTICES                 │
  ├─────────────────────────────────────────────────────┤
  │                                                     │
  │  F - Fast          Run in milliseconds              │
  │  I - Independent   No test depends on another       │
  │  R - Repeatable    Same result every time            │
  │  S - Self-Validating  Pass or fail, no manual check │
  │  T - Timely        Written with or before the code  │
  │                                                     │
  │  (FIRST principles)                                 │
  └─────────────────────────────────────────────────────┘
```

---

## Mocking Dependencies

```
  ┌──────────┐         ┌──────────┐
  │  Unit    │──MOCK──►│ Database │  ← Not a real DB
  │  Under   │         └──────────┘
  │  Test    │──MOCK──►┌──────────┐
  │          │         │ API Call │  ← Not a real API
  │          │──MOCK──►├──────────┤
  │          │         │ File Sys │  ← Not real files
  └──────────┘         └──────────┘

  Why mock?
  ✓ Fast — no I/O waits
  ✓ Reliable — no network failures
  ✓ Isolated — test only YOUR code
```

### Mock Example (JavaScript)

```javascript
// Mocking an API call with Jest
const userService = require('../src/userService');
const api = require('../src/api');

jest.mock('../src/api');

test('getUser fetches and formats user data', async () => {
  // Arrange
  api.fetch.mockResolvedValue({ id: 1, name: 'Alice' });

  // Act
  const user = await userService.getUser(1);

  // Assert
  expect(api.fetch).toHaveBeenCalledWith('/users/1');
  expect(user).toEqual({ id: 1, name: 'ALICE' });
});
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Tests pass locally, fail in CI | Different Node/Python version | Pin versions in CI matrix & `.nvmrc` |
| Flaky unit tests | Hidden shared state | Ensure true isolation; reset state in `beforeEach` |
| Slow unit tests (>5 min) | Too many or not truly "unit" | Profile; mock I/O; split integration tests out |
| Coverage drops sneakily | New code without tests | Enforce coverage thresholds in CI |
| Tests depend on order | Shared mutable state | Use `--randomize` flag; fix shared state issues |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Purpose** | Verify individual functions/classes in isolation |
| **Speed** | Milliseconds per test |
| **Dependencies** | All mocked/stubbed |
| **Runs** | Every commit, every PR |
| **Key metric** | Code coverage (target 80%+) |
| **FIRST** | Fast, Independent, Repeatable, Self-Validating, Timely |

---

## Quick Revision Questions

1. **What is a unit test?** *A test that verifies a single function or class in isolation from all external dependencies.*
2. **What are the FIRST principles?** *Fast, Independent, Repeatable, Self-Validating, Timely — guidelines for good unit tests.*
3. **Why mock dependencies in unit tests?** *To isolate the code under test, avoid I/O, and ensure fast, reliable, repeatable results.*
4. **What is code coverage and what's a good target?** *The percentage of code lines/branches executed by tests. A common target is 80%+.*
5. **When should unit tests run in CI?** *On every commit and pull request — they are fast enough to provide immediate feedback.*
6. **What causes unit tests to be flaky?** *Hidden shared state between tests, time-dependent logic, or reliance on external systems.*

---

[← Previous: Test Pyramid](01-test-pyramid.md) | [Back to README](../README.md) | [Next: Integration Testing →](03-integration-testing.md)
