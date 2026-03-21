# Chapter 1: Data Types and Structures

> **Overview:** Python's built-in data types and structures form the backbone of every data science workflow. This chapter covers numeric types, strings, booleans, None, and the four core collection types — lists, tuples, dictionaries, and sets — along with type conversion and mutability concepts.

---

## 1. Numeric Types

### int — Arbitrary-precision integers
```python
count = 42
big = 10 ** 100          # Python handles arbitrarily large integers
binary = 0b1010          # 10 in binary
hex_val = 0xFF           # 255 in hexadecimal
```

### float — 64-bit double-precision
```python
pi = 3.14159
sci = 2.5e-3             # 0.0025
print(0.1 + 0.2)         # 0.30000000000000004 — floating-point caveat!
```

### complex
```python
z = 3 + 4j
print(z.real, z.imag)    # 3.0  4.0
print(abs(z))            # 5.0 — magnitude
```

### Useful numeric functions
```python
print(divmod(17, 5))     # (3, 2) — quotient and remainder
print(round(3.14159, 2)) # 3.14
print(abs(-7))           # 7
```

---

## 2. Strings

### Creation and common methods
```python
s = "Hello, World!"
print(s.lower())         # "hello, world!"
print(s.upper())         # "HELLO, WORLD!"
print(s.split(", "))     # ['Hello', 'World!']
print(s.replace("World", "Python"))  # "Hello, Python!"
print(s.strip())         # removes leading/trailing whitespace
print(s.find("World"))   # 7  (returns -1 if not found)
print(s.count("l"))      # 3
```

### String formatting
```python
name, score = "Alice", 95.678

# f-strings (recommended — Python 3.6+)
print(f"{name} scored {score:.2f}%")    # Alice scored 95.68%

# .format()
print("{} scored {:.2f}%".format(name, score))

# %-formatting (legacy)
print("%s scored %.2f%%" % (name, score))
```

### Slicing
```python
s = "PYTHON"
print(s[0:3])   # PYT
print(s[::-1])   # NOHTYP  — reverse
```

---

## 3. Booleans and None

```python
print(type(True))       # <class 'bool'>
print(isinstance(True, int))  # True — bool is a subclass of int

# Falsy values: 0, 0.0, "", [], {}, set(), None, False
print(bool([]))          # False
print(bool([1]))         # True

x = None
print(x is None)         # True — always use `is` to check None
```

---

## 4. Lists

Lists are **ordered, mutable** sequences that can hold mixed types.

```python
nums = [10, 20, 30, 40, 50]

# Slicing
print(nums[1:4])         # [20, 30, 40]
print(nums[::2])         # [10, 30, 50]

# Common methods
nums.append(60)          # [10, 20, 30, 40, 50, 60]
nums.insert(0, 5)        # [5, 10, 20, 30, 40, 50, 60]
nums.pop()               # removes & returns 60
nums.remove(30)          # removes first occurrence of 30
nums.sort(reverse=True)  # in-place descending sort
nums.reverse()           # in-place reverse
print(nums.index(20))    # index of first 20
print(nums.count(10))    # count occurrences of 10

# Copying (shallow)
copy1 = nums[:]
copy2 = nums.copy()
```

---

## 5. Tuples

Tuples are **ordered, immutable** sequences — ideal for fixed records.

```python
point = (3, 4)
single = (42,)           # trailing comma for single-element tuple

# Packing & unpacking
x, y = point             # x=3, y=4
a, *rest = (1, 2, 3, 4)  # a=1, rest=[2, 3, 4]

# Named tuples for readable records
from collections import namedtuple
Sample = namedtuple("Sample", ["feature", "label"])
s = Sample(feature=0.8, label=1)
print(s.feature)          # 0.8
```

---

## 6. Dictionaries

Dictionaries are **unordered (insertion-ordered since 3.7), mutable** key-value maps.

```python
student = {"name": "Alice", "age": 22, "gpa": 3.9}

# Access
print(student["name"])            # Alice
print(student.get("major", "N/A"))  # N/A — safe access

# Modify
student["major"] = "CS"
student.update({"age": 23, "gpa": 4.0})

# Iteration
for key, val in student.items():
    print(f"{key}: {val}")

# Useful methods
print(list(student.keys()))
print(list(student.values()))
student.pop("major")               # remove key
student.setdefault("honors", True)  # insert only if missing
```

### Dictionary comprehension
```python
squares = {x: x**2 for x in range(1, 6)}
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

---

## 7. Sets

Sets are **unordered, mutable** collections of **unique** elements.

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)    # Union:        {1, 2, 3, 4, 5, 6}
print(a & b)    # Intersection: {3, 4}
print(a - b)    # Difference:   {1, 2}
print(a ^ b)    # Symmetric:    {1, 2, 5, 6}

a.add(5)
a.discard(10)   # no error if missing (unlike .remove())
```

### frozenset — immutable set (can be used as dict key)
```python
fs = frozenset([1, 2, 3])
```

---

## 8. Type Conversion

```python
int("42")        # 42
float("3.14")    # 3.14
str(100)         # "100"
list("abc")      # ['a', 'b', 'c']
tuple([1, 2])    # (1, 2)
set([1, 1, 2])   # {1, 2}
dict([("a", 1)]) # {'a': 1}
bool(0)          # False
```

---

## 9. Mutable vs Immutable

| Mutable (can change in place) | Immutable (cannot change) |
|-------------------------------|---------------------------|
| `list`                        | `int`, `float`, `complex` |
| `dict`                        | `str`                     |
| `set`                         | `tuple`                   |
| `bytearray`                   | `frozenset`, `bytes`      |

```python
# Mutable gotcha — aliasing
a = [1, 2, 3]
b = a            # b points to SAME object
b.append(4)
print(a)         # [1, 2, 3, 4] — a is also modified!

# Fix: copy explicitly
b = a.copy()
```

---

## Summary Table

| Type       | Ordered | Mutable | Duplicates | Syntax Example        |
|------------|---------|---------|------------|-----------------------|
| `list`     | ✅      | ✅      | ✅         | `[1, 2, 3]`          |
| `tuple`    | ✅      | ❌      | ✅         | `(1, 2, 3)`          |
| `dict`     | ✅*     | ✅      | Keys: ❌   | `{"a": 1}`           |
| `set`      | ❌      | ✅      | ❌         | `{1, 2, 3}`          |
| `frozenset`| ❌      | ❌      | ❌         | `frozenset({1,2})`   |
| `str`      | ✅      | ❌      | ✅         | `"hello"`            |

*Insertion-ordered since Python 3.7

---

## Best Practices

1. **Use f-strings** for readable string formatting.
2. **Prefer `tuple`** for fixed-size records — faster and hashable.
3. **Use `dict.get()`** to avoid `KeyError` on missing keys.
4. **Use `set`** when you need fast membership tests or deduplication.
5. **Copy mutable objects explicitly** to avoid aliasing bugs.
6. **Use `is` (not `==`)** to check for `None`.

---

## Revision Questions

1. What is the difference between `list.append()` and `list.extend()`? Give an example.
2. Why does `0.1 + 0.2 != 0.3` in Python? How can you work around this?
3. Explain tuple packing and unpacking with a code example.
4. What happens when you use a mutable object (like a list) as a dictionary key?
5. How do `set.discard()` and `set.remove()` differ in behavior?
6. Write a dictionary comprehension that maps each word in a list to its length.

---

| [🏠 Home](../README.md) | ▶️ Next: [Control Flow →](./02-control-flow.md) |
|:---|---:|
