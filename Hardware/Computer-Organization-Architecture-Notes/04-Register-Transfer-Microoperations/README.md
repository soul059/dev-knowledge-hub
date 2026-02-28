# Unit 4: Register Transfer and Microoperations

## 📋 Unit Overview

Register Transfer and Microoperations form the foundation of computer hardware design. This unit covers how data moves between registers and the elementary operations performed on data stored in registers. Understanding these concepts is essential for designing the control unit and datapath of a computer.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:

- Understand register transfer language (RTL) notation
- Describe various types of microoperations
- Design hardware for arithmetic, logic, and shift microoperations
- Understand bus structures for register transfer
- Design arithmetic logic units (ALU)

---

## 📚 Chapters in This Unit

| Chapter | Topic | Description |
|---------|-------|-------------|
| 4.1 | [Register Transfer Language](01-register-transfer-language.md) | RTL notation, conditional transfers, bus systems |
| 4.2 | [Arithmetic Microoperations](02-arithmetic-microoperations.md) | Add, subtract, increment, decrement operations |
| 4.3 | [Logic Microoperations](03-logic-microoperations.md) | AND, OR, XOR, complement operations |
| 4.4 | [Shift Microoperations](04-shift-microoperations.md) | Logical, arithmetic, and circular shifts |

---

## 🔑 Key Concepts Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 REGISTER TRANSFER AND MICROOPERATIONS                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                    REGISTER TRANSFER                              │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │  R2 ← R1         Transfer contents of R1 to R2                   │ │
│   │  If P: R2 ← R1   Conditional transfer (when P=1)                 │ │
│   │  R2 ← R1, R3←R4  Simultaneous transfers                          │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │
│   │   ARITHMETIC    │  │     LOGIC       │  │     SHIFT       │        │
│   ├─────────────────┤  ├─────────────────┤  ├─────────────────┤        │
│   │ • Add           │  │ • AND           │  │ • Logical       │        │
│   │ • Subtract      │  │ • OR            │  │ • Arithmetic    │        │
│   │ • Increment     │  │ • XOR           │  │ • Circular      │        │
│   │ • Decrement     │  │ • NOT           │  │   (Rotate)      │        │
│   │ • Negate        │  │ • Clear/Set     │  │ • Left/Right    │        │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘        │
│                                                                          │
│                              ↓                                           │
│              ┌───────────────────────────────────────┐                  │
│              │       ARITHMETIC LOGIC UNIT (ALU)     │                  │
│              │   Combines all microoperations        │                  │
│              │   Selection via control signals       │                  │
│              └───────────────────────────────────────┘                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Key RTL Symbols

| Symbol | Description | Example |
|--------|-------------|---------|
| **Letters** | Register name | R1, AR, PC, MAR |
| **Parentheses** | Part of register | R1(0-7), R2(L) |
| **Arrow ←** | Transfer | R2 ← R1 |
| **Comma ,** | Simultaneous | R1 ← R2, R3 ← R4 |
| **Colon :** | Conditional | P: R2 ← R1 |
| **Brackets []** | Memory address | M[AR] |

---

## 📊 Microoperation Summary

| Category | Operations | Hardware |
|----------|------------|----------|
| **Arithmetic** | +, -, +1, -1, 2's comp | Adder, Subtractor |
| **Logic** | ∧, ∨, ⊕, ¬ | Logic gates |
| **Shift** | shl, shr, cil, cir, ashl, ashr | Shifter |
| **Transfer** | ← | MUX, Bus |

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 3: Computer Arithmetic](../03-Computer-Arithmetic/README.md) | [Course Home](../README.md) | [Unit 5: Basic Computer Design](../05-Basic-Computer-Design/README.md) |

---

[← Unit 3](../03-Computer-Arithmetic/README.md) | [Course Home](../README.md) | [Unit 5 →](../05-Basic-Computer-Design/README.md)
