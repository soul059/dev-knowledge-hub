# Unit 3: 8085 Programming

## 📚 Unit Overview

This unit covers assembly language programming for the 8085 microprocessor. You'll learn program development techniques, data manipulation, arithmetic operations, and practical programming examples.

---

## 🎯 Unit Objectives

By the end of this unit, you will be able to:
- Develop 8085 assembly language programs
- Implement arithmetic and logical operations
- Write programs using different data structures
- Debug and optimize assembly code
- Handle arrays, sorting, and searching operations

---

## 📖 Chapters in This Unit

### [3.1 Assembly Language Basics](01-assembly-language-basics.md)
- Assembly language syntax and format
- Assembler directives
- Program structure and organization
- Labels, opcodes, and operands
- Machine code generation

### [3.2 Data Transfer Programs](02-data-transfer-programs.md)
- Memory to register transfers
- Block data movement
- Stack operations
- Exchange operations

### [3.3 Arithmetic Operations](03-arithmetic-operations.md)
- 8-bit and 16-bit addition/subtraction
- BCD arithmetic
- Multiplication algorithms
- Division algorithms

### [3.4 Logical and Bit Operations](04-logical-bit-operations.md)
- AND, OR, XOR operations
- Bit manipulation techniques
- Masking and testing
- Flag-based operations

### [3.5 Array and String Operations](05-array-string-operations.md)
- Array traversal and manipulation
- Finding largest/smallest elements
- Counting and summing
- String operations

### [3.6 Sorting and Searching](06-sorting-searching.md)
- Bubble sort algorithm
- Selection sort
- Linear search
- Binary search concepts

---

## 🔗 Key Concepts

| Concept | Description |
|---------|-------------|
| Assembly Directives | ORG, DB, DW, DS, EQU, END |
| Program Flow | Sequential, loop, conditional |
| Data Structures | Arrays, tables, stacks |
| Algorithm Design | Problem → Flowchart → Code |

---

## 📋 Programming Guidelines

```
8085 PROGRAM DEVELOPMENT PROCESS:

1. PROBLEM ANALYSIS
   └── Understand requirements
   └── Identify inputs/outputs
   └── Determine algorithm

2. FLOWCHART DESIGN
   └── Visual representation
   └── Logic flow
   └── Decision points

3. CODING
   └── Write assembly code
   └── Use proper syntax
   └── Add comments

4. ASSEMBLY
   └── Convert to machine code
   └── Resolve addresses
   └── Generate hex file

5. TESTING
   └── Load into system
   └── Execute step-by-step
   └── Verify results

6. DEBUGGING
   └── Find and fix errors
   └── Optimize code
   └── Document changes
```

---

## 📊 Common Programming Patterns

| Pattern | Use Case | Key Instructions |
|---------|----------|------------------|
| Counter Loop | Repeat N times | DCR r, JNZ |
| Conditional | If-then-else | CMP, JZ, JNZ |
| Data Move | Block transfer | MOV, INX, DCX |
| Accumulator Op | Arithmetic | ADD, SUB, ADC |
| Pointer Access | Array/Table | MOV r, M |

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 2: 8085 Microprocessor](../02-8085-Microprocessor/README.md) | [Course Index](../README.md) | [Unit 4: 8086 Microprocessor](../04-8086-Microprocessor/README.md) |

---

*[← Back to Course Index](../README.md)*
