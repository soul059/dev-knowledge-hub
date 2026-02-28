# Unit 3: Computer Arithmetic

## 📋 Unit Overview

Computer Arithmetic deals with the algorithms and hardware implementations for performing arithmetic operations on binary numbers. This unit covers integer addition, subtraction, multiplication (including Booth's algorithm), division, and floating-point arithmetic operations.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:

- Implement binary addition and subtraction with overflow detection
- Understand hardware adder circuits (ripple carry, carry lookahead)
- Master multiplication algorithms (shift-and-add, Booth's algorithm)
- Implement division algorithms (restoring and non-restoring)
- Perform floating-point arithmetic operations
- Design arithmetic circuits and understand their trade-offs

---

## 📚 Chapters in This Unit

| Chapter | Topic | Description |
|---------|-------|-------------|
| 3.1 | [Integer Addition and Subtraction](01-integer-addition-subtraction.md) | Binary addition, hardware adders, overflow detection |
| 3.2 | [Multiplication Algorithms](02-multiplication-algorithms.md) | Shift-and-add, array multiplier |
| 3.3 | [Booth's Multiplication Algorithm](03-booths-algorithm.md) | Signed multiplication using Booth's encoding |
| 3.4 | [Division Algorithms](04-division-algorithms.md) | Restoring and non-restoring division |
| 3.5 | [Floating-Point Arithmetic](05-floating-point-arithmetic.md) | FP addition, subtraction, multiplication, division |

---

## 🔑 Key Concepts Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      COMPUTER ARITHMETIC                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐       │
│   │  ADDITION/      │   │  MULTIPLICATION │   │   DIVISION      │       │
│   │  SUBTRACTION    │   │                 │   │                 │       │
│   ├─────────────────┤   ├─────────────────┤   ├─────────────────┤       │
│   │ • Half Adder    │   │ • Shift-Add     │   │ • Restoring     │       │
│   │ • Full Adder    │   │ • Array Mult    │   │ • Non-restoring │       │
│   │ • Ripple Carry  │   │ • Booth's Algo  │   │ • SRT Division  │       │
│   │ • Carry Lookahead│  │ • Wallace Tree  │   │                 │       │
│   │ • Overflow Det  │   │ • Dadda Tree    │   │                 │       │
│   └─────────────────┘   └─────────────────┘   └─────────────────┘       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │              FLOATING-POINT ARITHMETIC                       │       │
│   ├─────────────────────────────────────────────────────────────┤       │
│   │  • Alignment of exponents    • Normalization               │       │
│   │  • Mantissa operations       • Rounding                     │       │
│   │  • Exception handling        • Guard/Round/Sticky bits     │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Key Formulas

| Operation | Formula/Algorithm |
|-----------|-------------------|
| **Half Adder** | Sum = A ⊕ B, Carry = A · B |
| **Full Adder** | Sum = A ⊕ B ⊕ Cᵢₙ, Cₒᵤₜ = AB + Cᵢₙ(A ⊕ B) |
| **2's Complement** | -X = X̄ + 1 |
| **Overflow (Addition)** | V = Cₙ ⊕ Cₙ₋₁ |
| **Carry Lookahead** | Cᵢ = Gᵢ₋₁ + Pᵢ₋₁·Cᵢ₋₁ |
| **Booth Encoding** | +1, 0, -1 based on bit pairs |

---

## 📊 Complexity Comparison

| Algorithm | Time Complexity | Hardware Complexity |
|-----------|-----------------|---------------------|
| Ripple Carry Adder | O(n) | O(n) gates |
| Carry Lookahead | O(log n) | O(n log n) gates |
| Shift-Add Multiply | O(n) cycles | O(n) |
| Booth's Algorithm | O(n/2) iterations | O(n) |
| Array Multiplier | O(1) (parallel) | O(n²) gates |

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 2: Data Representation](../02-Data-Representation/README.md) | [Course Home](../README.md) | [Unit 4: Register Transfer](../04-Register-Transfer-Microoperations/README.md) |

---

[← Unit 2](../02-Data-Representation/README.md) | [Course Home](../README.md) | [Unit 4 →](../04-Register-Transfer-Microoperations/README.md)
