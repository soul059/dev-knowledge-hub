# C Programming - Complete Reference

A comprehensive guide to systems-level programming with C, covering fundamentals to advanced topics.

## 📚 Table of Contents

- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [Prerequisites](#prerequisites)
- [Learning Path](#learning-path)

## 🎯 Overview

This C programming documentation provides in-depth coverage of the C language, from basic syntax to advanced concepts like pointers, dynamic memory allocation, and file I/O.

## 📂 Topics Covered

### Fundamentals
- **[C language.md](./C%20language.md)** - Introduction to C, history, and basic structure
- **[Data types.md](./Data%20types.md)** - Primitive types, type modifiers, and type conversion
- **[Operator.md](./Operator.md)** - Arithmetic, relational, logical, and bitwise operators

### Control Flow
- **[if-else.md](./if-else.md)** - Conditional statements and decision making
- **[loops.md](./loops.md)** - for, while, do-while loops and loop control

### Functions
- **[Function.md](./Function.md)** - Function declaration, definition, and calling conventions
- **[Recursion.md](./Recursion.md)** - Recursive functions and techniques

### Data Structures
- **[Structures.md](./Structures.md)** - User-defined data types with struct
- **[Union.md](./Union.md)** - Memory-efficient data storage

### Memory Management
- **[Pointers.md](./Pointers.md)** - Pointer basics, arithmetic, and applications
- **[Dynamic memory allocation.md](./Dynamic%20memory%20allocation.md)** - malloc, calloc, realloc, free

### File Operations
- **[File IO.md](./File%20IO.md)** - Standard file I/O operations
- **[C File IO.md](./C%20File%20IO.md)** - Low-level file operations

### Preprocessor
- **[Preprocessor directives.md](./Preprocessor%20directives.md)** - Macros, conditional compilation, and includes

## 🔧 Prerequisites

- Basic understanding of programming concepts
- A C compiler (GCC, Clang, or MSVC)
- Text editor or IDE

## 🛤️ Recommended Learning Path

```
1. C Language Basics
    ↓
2. Data Types & Operators
    ↓
3. Control Flow (if-else, loops)
    ↓
4. Functions & Recursion
    ↓
5. Pointers (Essential!)
    ↓
6. Structures & Unions
    ↓
7. Dynamic Memory Allocation
    ↓
8. File I/O
    ↓
9. Preprocessor Directives
```

## 💡 How to Use This Guide

| Use Case | Approach |
|----------|----------|
| **Learning C** | Follow the learning path sequentially |
| **Quick Reference** | Navigate directly to specific topics |
| **Interview Prep** | Focus on pointers, memory, and data structures |
| **Systems Programming** | Emphasize pointers, file I/O, and memory management |

## 🎯 Key Concepts

| Concept | Description |
|---------|-------------|
| **Pointers** | Memory addresses and indirect access |
| **Memory Management** | Manual allocation and deallocation |
| **Pass by Value/Reference** | Function argument passing |
| **Arrays & Strings** | Contiguous memory and character arrays |
| **Preprocessor** | Compile-time text substitution |

## 📝 Quick Reference

```c
// Hello World
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}

// Pointer basics
int x = 10;
int *ptr = &x;       // ptr holds address of x
printf("%d", *ptr);  // Dereference: prints 10

// Dynamic memory
int *arr = (int *)malloc(5 * sizeof(int));
free(arr);  // Always free allocated memory

// Structure
struct Person {
    char name[50];
    int age;
};

// File I/O
FILE *fp = fopen("file.txt", "r");
fclose(fp);
```

## 🛠️ Compilation

```bash
# Compile with GCC
gcc -o program program.c

# Compile with warnings
gcc -Wall -Wextra -o program program.c

# Compile and run
gcc -o program program.c && ./program
```

---

*Last Updated: April 2026*
