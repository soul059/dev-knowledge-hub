# Chapter 3: Functions and Lambda Expressions

> **Overview:** Functions are the primary unit of code reuse in Python. This chapter covers function definitions, parameter types, scope rules, closures, decorators, lambda expressions, and functional programming tools — all critical for writing clean, modular data science code.

---

## 1. Defining Functions

```python
def greet(name):
    """Return a greeting string."""
    return f"Hello, {name}!"

print(greet("Alice"))  # Hello, Alice!
```

### Multiple return values (tuple unpacking)
```python
def min_max(data):
    return min(data), max(data)

lo, hi = min_max([3, 1, 4, 1, 5])
print(lo, hi)  # 1 5
```

---

## 2. Parameters and Arguments

### Positional and keyword
```python
def power(base, exp):
    return base ** exp

power(2, 10)          # positional
power(exp=10, base=2) # keyword — order doesn't matter
```

### Default values
```python
def train(epochs=10, lr=0.001):
    print(f"Training for {epochs} epochs at lr={lr}")

train()               # uses defaults
train(epochs=50)      # override one
```

> ⚠️ **Never use mutable defaults** (like `[]` or `{}`). Use `None` instead:
```python
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### *args — variable positional arguments
```python
def total(*args):
    return sum(args)

print(total(1, 2, 3, 4))  # 10
```

### **kwargs — variable keyword arguments
```python
def build_config(**kwargs):
    for key, val in kwargs.items():
        print(f"{key} = {val}")

build_config(lr=0.01, epochs=100, batch_size=32)
```

### Combined parameter order
```python
def func(pos, /, normal, *, kw_only):
    """pos: positional-only, normal: either, kw_only: keyword-only"""
    pass

# The / separator marks positional-only; * marks keyword-only.
```

---

## 3. Docstrings

```python
def normalize(data, method="min-max"):
    """
    Normalize a list of numbers.

    Parameters
    ----------
    data : list of float
        The raw data to normalize.
    method : str, optional
        Normalization method ('min-max' or 'z-score'). Default 'min-max'.

    Returns
    -------
    list of float
        The normalized data.
    """
    if method == "min-max":
        lo, hi = min(data), max(data)
        return [(x - lo) / (hi - lo) for x in data]
    elif method == "z-score":
        mean = sum(data) / len(data)
        std = (sum((x - mean)**2 for x in data) / len(data)) ** 0.5
        return [(x - mean) / std for x in data]
```

---

## 4. Scope — The LEGB Rule

Python resolves names in this order:
| Level   | Description                     |
|---------|---------------------------------|
| **L**ocal   | Inside the current function |
| **E**nclosing | In enclosing (outer) functions |
| **G**lobal  | Module-level variables      |
| **B**uilt-in | Python built-in names       |

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)       # local
    inner()

outer()
```

### `global` and `nonlocal`
```python
counter = 0
def increment():
    global counter
    counter += 1

def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1
    inner()
    print(count)  # 1
```

---

## 5. Closures

A closure is a function that remembers variables from its enclosing scope.

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor   # 'factor' is captured
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5))   # 10
print(triple(5))   # 15
```

---

## 6. Decorators

Decorators wrap a function to extend its behavior without modifying it.

```python
import time

def timer(func):
    """Decorator that times function execution."""
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_sum(n):
    return sum(range(n))

slow_sum(1_000_000)  # slow_sum took 0.0312s
```

### Stacking decorators
```python
from functools import wraps

def debug(func):
    @wraps(func)  # preserves original function metadata
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}({args}, {kwargs})")
        return func(*args, **kwargs)
    return wrapper

@debug
@timer
def compute(x):
    return x ** 2
```

---

## 7. Lambda Functions

Anonymous, single-expression functions.

```python
square = lambda x: x ** 2
print(square(5))  # 25

# Sorting with lambda
students = [("Alice", 90), ("Bob", 75), ("Charlie", 88)]
students.sort(key=lambda s: s[1], reverse=True)
print(students)  # [('Alice', 90), ('Charlie', 88), ('Bob', 75)]
```

---

## 8. map, filter, reduce

### map — apply a function to every element
```python
nums = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, nums))
print(squared)  # [1, 4, 9, 16, 25]
```

### filter — keep elements where function returns True
```python
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4]
```

### reduce — cumulative operation
```python
from functools import reduce

product = reduce(lambda a, b: a * b, nums)
print(product)  # 120  (1*2*3*4*5)
```

> 💡 **Prefer list comprehensions** over `map`/`filter` for readability in most cases.

---

## 9. Higher-Order Functions

Functions that accept or return other functions.

```python
def apply_pipeline(data, *transforms):
    """Apply a series of transformations to data."""
    for func in transforms:
        data = func(data)
    return data

result = apply_pipeline(
    [1, -2, 3, -4, 5],
    lambda d: [abs(x) for x in d],     # step 1: absolute values
    lambda d: sorted(d),                # step 2: sort
    lambda d: d[-3:]                    # step 3: top 3
)
print(result)  # [3, 4, 5]
```

---

## 10. Data Science Use Cases

### Feature scaling function
```python
def scale_features(data, scaler="minmax"):
    if scaler == "minmax":
        lo, hi = min(data), max(data)
        return [(x - lo) / (hi - lo) for x in data]
    elif scaler == "standard":
        mu = sum(data) / len(data)
        sigma = (sum((x - mu)**2 for x in data) / len(data)) ** 0.5
        return [(x - mu) / sigma for x in data]

raw = [10, 20, 30, 40, 50]
print(scale_features(raw))  # [0.0, 0.25, 0.5, 0.75, 1.0]
```

### Configurable training loop with **kwargs
```python
def train_model(X, y, **hyperparams):
    lr = hyperparams.get("lr", 0.01)
    epochs = hyperparams.get("epochs", 100)
    verbose = hyperparams.get("verbose", False)
    for epoch in range(epochs):
        loss = sum((xi - yi)**2 for xi, yi in zip(X, y)) / len(X)
        if verbose and epoch % 10 == 0:
            print(f"Epoch {epoch}: loss={loss:.4f}")
```

---

## Summary Table

| Concept            | Syntax / Tool                     | Key Point                          |
|--------------------|-----------------------------------|------------------------------------|
| Basic function     | `def func(a, b): return a + b`    | Reusable named block               |
| Default params     | `def f(x, y=10):`                 | Avoid mutable defaults             |
| `*args`            | `def f(*args):`                   | Variable positional arguments      |
| `**kwargs`         | `def f(**kwargs):`                | Variable keyword arguments         |
| Scope              | LEGB rule                         | Local → Enclosing → Global → Built-in |
| Closure            | Inner function captures outer var | State without classes              |
| Decorator          | `@decorator`                      | Wraps function behavior            |
| Lambda             | `lambda x: x + 1`                | Anonymous single-expression func   |
| `map()`            | `map(func, iterable)`            | Transform every element            |
| `filter()`         | `filter(func, iterable)`         | Keep matching elements             |
| `reduce()`         | `reduce(func, iterable)`         | Cumulative aggregation             |

---

## Best Practices

1. **Write docstrings** for every non-trivial function (NumPy or Google style).
2. **Keep functions small** — each should do one thing well.
3. **Avoid mutable default arguments** — use `None` sentinel pattern.
4. **Use `@wraps`** from `functools` when writing decorators.
5. **Prefer comprehensions** over `map`/`filter` for clarity.
6. **Use type hints** to document expected types: `def add(a: int, b: int) -> int:`.

---

## Revision Questions

1. What is the LEGB rule? Explain each level with an example.
2. Why should you avoid using a list `[]` as a default parameter value?
3. Write a decorator that logs the arguments and return value of any function.
4. What is the difference between `*args` and `**kwargs`? Show both in one function.
5. Explain closures with an example. When are they useful?
6. Rewrite `list(map(lambda x: x**2, nums))` using a list comprehension.

---

| [◀️ Previous: Control Flow](./02-control-flow.md) | [🏠 Home](../README.md) | [Next: List Comprehensions ▶️](./04-list-comprehensions.md) |
|:---|:---:|---:|
