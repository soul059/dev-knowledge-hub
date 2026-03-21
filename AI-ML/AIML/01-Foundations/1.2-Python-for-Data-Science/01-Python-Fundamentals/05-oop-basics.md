# Chapter 5: Object-Oriented Programming Basics

> **Overview:** OOP organizes code around objects that bundle data (attributes) and behavior (methods). This chapter covers classes, inheritance, polymorphism, encapsulation, dunder methods, dataclasses, and OOP patterns commonly used in machine learning.

---

## 1. Classes and Objects

```python
class Dog:
    species = "Canis familiaris"   # class attribute (shared)

    def __init__(self, name, age):
        self.name = name           # instance attribute
        self.age = age

    def bark(self):
        return f"{self.name} says Woof!"

rex = Dog("Rex", 5)
print(rex.bark())         # Rex says Woof!
print(rex.species)        # Canis familiaris
print(Dog.species)        # Canis familiaris
```

---

## 2. Instance, Class, and Static Methods

```python
class MathUtils:
    pi = 3.14159

    def __init__(self, value):
        self.value = value

    # Instance method — access instance via self
    def double(self):
        return self.value * 2

    # Class method — access class via cls
    @classmethod
    def circle_area(cls, radius):
        return cls.pi * radius ** 2

    # Static method — no access to instance or class
    @staticmethod
    def add(a, b):
        return a + b

m = MathUtils(10)
print(m.double())                  # 20
print(MathUtils.circle_area(5))    # 78.53975
print(MathUtils.add(3, 7))         # 10
```

---

## 3. Inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        raise NotImplementedError("Subclass must implement speak()")

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

animals = [Cat("Whiskers"), Dog("Buddy")]
for a in animals:
    print(a.speak())
# Whiskers says Meow!
# Buddy says Woof!
```

### Multiple inheritance and MRO
```python
class A:
    def greet(self):
        return "A"

class B(A):
    def greet(self):
        return "B"

class C(A):
    def greet(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.greet())       # B  (follows MRO: D → B → C → A)
print(D.__mro__)        # shows resolution order
```

---

## 4. Polymorphism

```python
class Rectangle:
    def __init__(self, w, h):
        self.w, self.h = w, h
    def area(self):
        return self.w * self.h

class Circle:
    def __init__(self, r):
        self.r = r
    def area(self):
        return 3.14159 * self.r ** 2

# Same interface, different implementations
shapes = [Rectangle(3, 4), Circle(5)]
for s in shapes:
    print(f"{type(s).__name__}: area = {s.area():.2f}")
# Rectangle: area = 12.00
# Circle: area = 78.54
```

---

## 5. Encapsulation

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance        # convention: "protected"
        self.__id = 12345              # name-mangled: "private"

    @property
    def balance(self):
        """Read-only property."""
        return self._balance

    def deposit(self, amount):
        if amount > 0:
            self._balance += amount

    def withdraw(self, amount):
        if 0 < amount <= self._balance:
            self._balance -= amount

acc = BankAccount(1000)
acc.deposit(500)
print(acc.balance)      # 1500  (via property)
# acc.__id → AttributeError (name-mangled to _BankAccount__id)
```

---

## 6. Dunder (Magic) Methods

Dunder methods let your objects work with Python's built-in operations.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"

    def __len__(self):
        return 2

    def __getitem__(self, index):
        return (self.x, self.y)[index]

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __abs__(self):
        return (self.x**2 + self.y**2) ** 0.5

v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(repr(v1))       # Vector(3, 4)
print(str(v1))         # (3, 4)
print(len(v1))         # 2
print(v1[0])           # 3
print(v1 + v2)         # (4, 6)
print(v1 == Vector(3,4))  # True
print(abs(v1))         # 5.0
```

### Common dunder methods reference

| Method         | Triggered by           | Purpose                    |
|----------------|------------------------|----------------------------|
| `__init__`     | `MyClass()`            | Constructor                |
| `__repr__`     | `repr(obj)`            | Developer-facing string    |
| `__str__`      | `str(obj)`, `print()`  | User-facing string         |
| `__len__`      | `len(obj)`             | Length                     |
| `__getitem__`  | `obj[key]`             | Indexing / subscript       |
| `__setitem__`  | `obj[key] = val`       | Assign by index            |
| `__contains__` | `x in obj`             | Membership test            |
| `__iter__`     | `for x in obj`         | Make iterable              |
| `__add__`      | `obj + other`          | Addition operator          |
| `__eq__`       | `obj == other`         | Equality check             |
| `__lt__`       | `obj < other`          | Less than comparison       |
| `__call__`     | `obj()`                | Make callable              |

---

## 7. Dataclasses (Python 3.7+)

Dataclasses reduce boilerplate for classes that mainly hold data.

```python
from dataclasses import dataclass, field

@dataclass
class Sample:
    feature_1: float
    feature_2: float
    label: int = 0
    tags: list = field(default_factory=list)

    @property
    def vector(self):
        return [self.feature_1, self.feature_2]

s = Sample(0.5, 1.2, label=1, tags=["train"])
print(s)            # Sample(feature_1=0.5, feature_2=1.2, label=1, tags=['train'])
print(s.vector)     # [0.5, 1.2]
print(s == Sample(0.5, 1.2, label=1, tags=["train"]))  # True — auto __eq__
```

### Frozen dataclass (immutable)
```python
@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
# p.x = 5  → FrozenInstanceError
```

---

## 8. OOP Patterns in Machine Learning

### Custom Estimator (scikit-learn style)
```python
class SimpleScaler:
    """Min-max scaler following the fit/transform pattern."""

    def __init__(self):
        self.min_ = None
        self.max_ = None

    def fit(self, data):
        self.min_ = min(data)
        self.max_ = max(data)
        return self

    def transform(self, data):
        rng = self.max_ - self.min_
        return [(x - self.min_) / rng for x in data]

    def fit_transform(self, data):
        return self.fit(data).transform(data)

scaler = SimpleScaler()
raw = [10, 20, 30, 40, 50]
scaled = scaler.fit_transform(raw)
print(scaled)  # [0.0, 0.25, 0.5, 0.75, 1.0]
```

### Custom Dataset / Data Loader
```python
class Dataset:
    """Indexable dataset with len and getitem support."""

    def __init__(self, features, labels):
        self.features = features
        self.labels = labels

    def __len__(self):
        return len(self.features)

    def __getitem__(self, idx):
        return self.features[idx], self.labels[idx]

    def __iter__(self):
        for i in range(len(self)):
            yield self[i]

ds = Dataset([[1, 2], [3, 4], [5, 6]], [0, 1, 0])
print(len(ds))       # 3
print(ds[1])         # ([3, 4], 1)

for feat, label in ds:
    print(feat, "→", label)
```

---

## Summary Table

| Concept          | Keyword / Decorator      | Purpose                              |
|------------------|--------------------------|--------------------------------------|
| Class            | `class MyClass:`         | Blueprint for objects                |
| Constructor      | `__init__(self, ...)`    | Initialize instance state            |
| Instance method  | `def method(self):`      | Operates on instance data            |
| Class method     | `@classmethod`           | Operates on class-level data         |
| Static method    | `@staticmethod`          | Utility — no access to self/cls      |
| Inheritance      | `class Child(Parent):`   | Reuse + extend parent behavior       |
| Property         | `@property`              | Controlled attribute access          |
| Dataclass        | `@dataclass`             | Auto-generate __init__, __repr__, etc. |
| Dunder methods   | `__str__`, `__add__`, …  | Integrate with Python operators      |

---

## Best Practices

1. **Prefer composition over inheritance** when behaviors don't form an "is-a" hierarchy.
2. **Use `@dataclass`** for data-holding classes to reduce boilerplate.
3. **Implement `__repr__`** for every class — invaluable for debugging.
4. **Use properties** instead of getters/setters for controlled access.
5. **Follow the fit/transform pattern** for ML preprocessing classes.
6. **Keep classes focused** — each class should have a single responsibility.

---

## Revision Questions

1. What is the difference between a class attribute and an instance attribute?
2. Explain the difference between `@classmethod`, `@staticmethod`, and a regular method.
3. What is the MRO, and how does Python determine it for multiple inheritance?
4. Implement a `__repr__` and `__add__` method for a `Money` class with `amount` and `currency`.
5. When would you use a `@dataclass` over a regular class?
6. Design a simple `DataPipeline` class with `add_step()` and `run(data)` methods using OOP.

---

| [◀️ Previous: List Comprehensions](./04-list-comprehensions.md) | [🏠 Home](../README.md) | [Next: Exception Handling ▶️](./06-exception-handling.md) |
|:---|:---:|---:|
