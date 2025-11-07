# Design Patterns - Class Diagrams

A comprehensive visual guide to common Design Patterns with UML class diagrams.

---

## Table of Contents

1. [Creational Patterns](#creational-patterns)
   - [Abstract Factory Pattern](#abstract-factory-pattern)
   - [Factory Method Pattern](#factory-method-pattern)
   - [Prototype Pattern](#prototype-pattern)

2. [Structural Patterns](#structural-patterns)
   - [Adapter Pattern](#adapter-pattern)
   - [Decorator Pattern](#decorator-pattern)

---

## Creational Patterns

Creational patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

### Abstract Factory Pattern

**Intent:** Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

**Use When:**
- A system should be independent of how its products are created
- A system should be configured with one of multiple families of products
- A family of related product objects is designed to be used together

**Key Benefits:**
- Isolates concrete classes
- Makes exchanging product families easy
- Promotes consistency among products

![Abstract Factory Pattern](./AbstractFactoryPattern.svg)

**Example Use Case:** Creating UI components (buttons, checkboxes) for different operating systems (Windows, Mac) where each OS has its own factory.

---

### Factory Method Pattern

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

**Use When:**
- A class can't anticipate the class of objects it must create
- A class wants its subclasses to specify the objects it creates
- Classes delegate responsibility to one of several helper subclasses

**Key Benefits:**
- Eliminates the need to bind application-specific classes
- Provides hooks for subclasses
- Connects parallel class hierarchies

![Factory Method Pattern](./FactoryMethodPattern.svg)

**Example Use Case:** Document creation system where different factories create different document types (Word, PDF) but all share the same opening mechanism.

---

### Prototype Pattern

**Intent:** Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

**Use When:**
- Classes to instantiate are specified at run-time
- Avoiding building a class hierarchy of factories
- Instances of a class can have one of only a few different combinations of state

**Key Benefits:**
- Adding and removing products at run-time
- Specifying new objects by varying values
- Reduced subclassing
- Configuring an application with classes dynamically

![Prototype Pattern](./PrototypePattern.svg)

**Example Use Case:** Creating copies of graphic shapes (circles, rectangles) where each clone maintains the same properties but can be modified independently.

---

## Structural Patterns

Structural patterns deal with object composition, creating relationships between objects to form larger structures.

### Adapter Pattern

**Intent:** Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Use When:**
- You want to use an existing class with an incompatible interface
- You want to create a reusable class that cooperates with unrelated classes
- You need to use several existing subclasses but it's impractical to adapt their interface by subclassing

**Key Benefits:**
- Increases reusability of existing code
- Introduces only one object (adapter)
- No need to modify existing code

![Adapter Pattern](./AdapterPattern.svg)

**Example Use Case:** Media player that can play different audio formats (MP3, VLC, MP4) by adapting advanced media player capabilities to a simple player interface.

---

### Decorator Pattern

**Intent:** Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Use When:**
- You want to add responsibilities to individual objects dynamically and transparently
- You want to add responsibilities that can be withdrawn
- Extension by subclassing is impractical

**Key Benefits:**
- More flexibility than static inheritance
- Avoids feature-laden classes high up in the hierarchy
- Responsibilities can be added and removed at run-time
- Functionality can be composed from simple pieces

![Decorator Pattern](./DecoratorPattern.svg)

**Example Use Case:** Coffee ordering system where you start with a simple coffee and dynamically add ingredients (milk, sugar) with each addition modifying cost and description.

---

## Pattern Comparison

### Creational Patterns Comparison

| Pattern | Purpose | Key Difference |
|---------|---------|----------------|
| **Abstract Factory** | Create families of related objects | Creates multiple types of objects that work together |
| **Factory Method** | Create a single object | Defers object creation to subclasses |
| **Prototype** | Create objects by cloning | Creates new objects by copying existing ones |

### Structural Patterns Comparison

| Pattern | Purpose | Key Difference |
|---------|---------|----------------|
| **Adapter** | Make incompatible interfaces work together | Converts one interface to another |
| **Decorator** | Add responsibilities to objects | Wraps objects to add new behavior |

---

## Legend

### UML Notation Guide

- **Solid line with hollow triangle (━━━▷)**: Inheritance (extends)
- **Dashed line with hollow triangle (┈┈┈▷)**: Implementation (implements)
- **Solid line with diamond (◆───)**: Composition (strong ownership)
- **Solid line with hollow diamond (◇───)**: Aggregation (weak ownership)
- **Dashed line with arrow (┈┈┈→)**: Dependency (uses)
- **«creates»**: Factory/creation relationship

### Class Stereotypes

- **«interface»**: Interface definition
- **«abstract»**: Abstract class
- **Italic text**: Abstract methods
- **+ public**: Public visibility
- **- private**: Private visibility
- **# protected**: Protected visibility

---

## Additional Resources

### Books
- "Design Patterns: Elements of Reusable Object-Oriented Software" by Gang of Four
- "Head First Design Patterns" by Eric Freeman & Elisabeth Robson

### Online Resources
- [Refactoring.Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [SourceMaking - Design Patterns](https://sourcemaking.com/design_patterns)

---

## Notes

- All diagrams are created using draw.io format (.dio files)
- SVG exports maintain full quality and can be zoomed without loss
- Each pattern includes practical Java examples
- Patterns follow standard UML notation

---

*Last Updated: November 7, 2025*
