# Unit 2: Data Representation

## 📋 Unit Overview

This unit explores how different types of data are represented in computer systems. Understanding data representation is fundamental to computer architecture, as all information—numbers, characters, and instructions—must be encoded in binary format.

---

## 🎯 Learning Objectives

After completing this unit, you will be able to:
- Represent integers using fixed-point notation
- Understand floating-point representation and IEEE 754 standard
- Explain character encoding schemes (ASCII, Unicode)
- Convert between different number representations
- Identify overflow and underflow conditions

---

## 📑 Chapters in This Unit

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 2.1 | [Fixed Point Representation](01-fixed-point-representation.md) | Signed/unsigned integers, 2's complement |
| 2.2 | [Floating Point Representation](02-floating-point-representation.md) | Scientific notation, mantissa, exponent |
| 2.3 | [IEEE 754 Standard](03-ieee-754-standard.md) | Single/double precision, special values |
| 2.4 | [Character Representation](04-character-representation.md) | ASCII, Unicode, UTF-8 |

---

## 🔑 Key Concepts Preview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      DATA REPRESENTATION                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   NUMERIC DATA                                                           │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                  │   │
│   │   Fixed Point              Floating Point                        │   │
│   │   ┌───────────────┐       ┌──────────────────────────────────┐  │   │
│   │   │ Integer       │       │ Sign │ Exponent │   Mantissa     │  │   │
│   │   │ Representation│       │  1   │   8/11   │   23/52 bits   │  │   │
│   │   │               │       │ bit  │   bits   │                │  │   │
│   │   │ • Unsigned    │       └──────────────────────────────────┘  │   │
│   │   │ • Signed      │                                              │   │
│   │   │ • 2's comp    │       Value = (-1)^S × 1.M × 2^(E-bias)     │   │
│   │   └───────────────┘                                              │   │
│   │                                                                  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   CHARACTER DATA                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                  │   │
│   │   ASCII (7-bit)    Extended ASCII    Unicode (UTF-8/16/32)      │   │
│   │   128 characters   256 characters    1,114,112 code points     │   │
│   │                                                                  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Unit Summary Table

| Data Type | Representation | Range/Example |
|-----------|----------------|---------------|
| **Unsigned Integer** | Pure binary | 0 to 2ⁿ-1 |
| **Signed Integer** | 2's complement | -2ⁿ⁻¹ to 2ⁿ⁻¹-1 |
| **Float (32-bit)** | IEEE 754 Single | ±1.18×10⁻³⁸ to ±3.4×10³⁸ |
| **Double (64-bit)** | IEEE 754 Double | ±2.23×10⁻³⁰⁸ to ±1.8×10³⁰⁸ |
| **Character** | ASCII/Unicode | 'A' = 65, 'a' = 97 |

---

## 🧭 Navigation

| Previous | Home | Next |
|----------|------|------|
| [Unit 1: Basic Computer Organization](../01-Basic-Computer-Organization/README.md) | [Course Home](../README.md) | [Unit 3: Computer Arithmetic](../03-Computer-Arithmetic/README.md) |

---

*Start learning: [Chapter 2.1 - Fixed Point Representation](01-fixed-point-representation.md)*
