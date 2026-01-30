# Chapter 1: Introduction to Compilers

## 1.1 What is a Language Translator?

A **language translator** is a software that converts a program written in one language (source language) to another language (target language).

### The Fundamental Problem

Humans prefer to write programs in **high-level languages** (like C, Java, Python) because they are:
- Closer to human thinking
- Easier to read and maintain
- Platform independent

But computers only understand **machine language** (binary code: 0s and 1s).

**The Bridge**: Language translators bridge this gap by converting human-readable code into machine-executable code.

---

## 1.2 Types of Language Translators

### 1.2.1 Compiler

A **compiler** translates the **entire** source program into target code **before** execution.

```
┌─────────────┐    ┌──────────┐    ┌─────────────┐
│   Source    │───▶│ Compiler │───▶│   Target    │
│   Program   │    │          │    │   Program   │
└─────────────┘    └──────────┘    └─────────────┘
     (C code)                        (Machine code)
```

**Characteristics:**
- Translation happens once; execution can happen many times
- Errors are reported after analyzing the entire program
- Faster execution (no translation overhead during runtime)
- Examples: GCC (C/C++), javac (Java)

### 1.2.2 Interpreter

An **interpreter** translates and executes the source program **line by line**.

```
┌─────────────┐    ┌─────────────┐    ┌──────────┐
│   Source    │───▶│ Interpreter │───▶│  Output  │
│   Program   │    │             │    │          │
└─────────────┘    └─────────────┘    └──────────┘
```

**Characteristics:**
- No intermediate object code is generated
- Errors are reported immediately when encountered
- Slower execution (translation happens every time)
- Easier debugging
- Examples: Python interpreter, JavaScript engines

### 1.2.3 Assembler

An **assembler** translates assembly language programs into machine code.

```
┌─────────────┐    ┌───────────┐    ┌─────────────┐
│  Assembly   │───▶│ Assembler │───▶│   Machine   │
│   Program   │    │           │    │    Code     │
└─────────────┘    └───────────┘    └─────────────┘
```

**Assembly Language** uses mnemonics (like ADD, MOV, SUB) instead of binary codes.

### 1.2.4 Comparison Table

| Aspect | Compiler | Interpreter | Assembler |
|--------|----------|-------------|-----------|
| **Input** | High-level language | High-level language | Assembly language |
| **Output** | Object/Machine code | Immediate execution | Machine code |
| **Translation** | Entire program at once | Line by line | Entire program |
| **Speed** | Fast execution | Slow execution | Fast execution |
| **Memory** | Requires more memory | Requires less memory | Less memory |
| **Error Detection** | After complete analysis | Immediately | After complete analysis |
| **Debugging** | Difficult | Easy | Difficult |

---

## 1.3 Other Important Translators

### 1.3.1 Preprocessor
- Processes source code **before** compilation
- Handles directives like `#include`, `#define` in C
- Performs macro expansion and file inclusion

### 1.3.2 Linker
- Combines multiple object files into a single executable
- Resolves external references between modules
- Links library functions

### 1.3.3 Loader
- Loads the executable program into memory
- Assigns memory addresses
- Starts program execution

### 1.3.4 Cross-Compiler
- Runs on one platform but generates code for a **different** platform
- Example: Compiling ARM code on an x86 machine

---

## 1.4 Structure of a Compiler

A compiler is logically divided into two major parts:

```
┌─────────────────────────────────────────────────────────────┐
│                        COMPILER                              │
│  ┌─────────────────────────┐  ┌─────────────────────────┐   │
│  │     ANALYSIS PHASE      │  │    SYNTHESIS PHASE      │   │
│  │     (Front End)         │  │     (Back End)          │   │
│  │                         │  │                         │   │
│  │  • Lexical Analysis     │  │  • Intermediate Code    │   │
│  │  • Syntax Analysis      │  │    Generation           │   │
│  │  • Semantic Analysis    │  │  • Code Optimization    │   │
│  │                         │  │  • Code Generation      │   │
│  └─────────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Analysis Phase (Front End)
- **Understands** the source program
- **Breaks down** the source into constituent parts
- **Creates** an intermediate representation
- **Checks** for errors (lexical, syntactic, semantic)

### Synthesis Phase (Back End)
- **Constructs** the target program from intermediate representation
- **Optimizes** the code for efficiency
- **Generates** machine-specific code

---

## 1.5 Phases of a Compiler

A compiler operates in **six phases**, each performing a specific task:

```
                    ┌─────────────────┐
                    │  Source Program │
                    └────────┬────────┘
                             ▼
                    ┌─────────────────┐
                    │ Lexical Analysis│ ──────┐
                    │    (Scanner)    │       │
                    └────────┬────────┘       │
                             ▼                │
                    ┌─────────────────┐       │
                    │ Syntax Analysis │       │
                    │    (Parser)     │       │    ┌──────────────┐
                    └────────┬────────┘       ├───▶│ Symbol Table │
                             ▼                │    │   Manager    │
                    ┌─────────────────┐       │    └──────────────┘
                    │Semantic Analysis│       │           │
                    └────────┬────────┘       │    ┌──────────────┐
                             ▼                ├───▶│    Error     │
                    ┌─────────────────┐       │    │   Handler    │
                    │ Intermediate    │       │    └──────────────┘
                    │ Code Generation │ ──────┤
                    └────────┬────────┘       │
                             ▼                │
                    ┌─────────────────┐       │
                    │ Code            │ ──────┤
                    │ Optimization    │       │
                    └────────┬────────┘       │
                             ▼                │
                    ┌─────────────────┐       │
                    │ Code Generation │ ──────┘
                    └────────┬────────┘
                             ▼
                    ┌─────────────────┐
                    │  Target Program │
                    └─────────────────┘
```

### Phase 1: Lexical Analysis (Scanning)

**Purpose**: Read the source program character by character and group them into meaningful sequences called **tokens**.

**Input**: Stream of characters
**Output**: Stream of tokens

**Example**:
```c
position = initial + rate * 60
```

**Tokens produced**:
| Token Type | Token Value |
|------------|-------------|
| IDENTIFIER | position |
| ASSIGN_OP | = |
| IDENTIFIER | initial |
| ADD_OP | + |
| IDENTIFIER | rate |
| MUL_OP | * |
| NUMBER | 60 |

### Phase 2: Syntax Analysis (Parsing)

**Purpose**: Check if the token sequence follows the grammatical rules of the language and create a hierarchical structure called **parse tree**.

**Input**: Stream of tokens
**Output**: Parse tree / Syntax tree

**Example Parse Tree**:
```
              =
            /   \
      position   +
                / \
          initial  *
                  / \
               rate  60
```

### Phase 3: Semantic Analysis

**Purpose**: Check the meaning of the program - type checking, scope resolution, and other semantic rules.

**Tasks**:
- Type checking (can't add string to integer)
- Scope resolution (is this variable declared?)
- Array bounds checking
- Coercion (automatic type conversion)

**Example**:
```
- If 'rate' is float and 60 is integer
- Semantic analyzer converts 60 to 60.0 (type coercion)
```

### Phase 4: Intermediate Code Generation

**Purpose**: Generate an intermediate representation that is:
- Easy to produce from the syntax tree
- Easy to translate into target code
- Machine independent

**Common IR forms**:
- Three-address code
- Abstract Syntax Tree (AST)
- Directed Acyclic Graph (DAG)

**Example (Three-Address Code)**:
```
t1 = intToFloat(60)
t2 = rate * t1
t3 = initial + t2
position = t3
```

### Phase 5: Code Optimization

**Purpose**: Improve the intermediate code to produce better (faster, smaller) target code.

**Types**:
- **Machine-independent**: Applied on intermediate code
- **Machine-dependent**: Applied on target code

**Example**:
```
Before: t1 = intToFloat(60)
        t2 = rate * t1

After:  t2 = rate * 60.0    (Constant folding)
```

### Phase 6: Code Generation

**Purpose**: Generate the actual target machine code.

**Tasks**:
- Instruction selection
- Register allocation
- Memory management
- Instruction ordering

**Example (Assembly-like code)**:
```
LOAD  rate, R1
MUL   #60.0, R1
LOAD  initial, R2
ADD   R1, R2
STORE R2, position
```

---

## 1.6 Cousins of the Compiler

These are supporting modules that assist the compiler:

### 1.6.1 Symbol Table Manager

A **symbol table** is a data structure that stores information about identifiers in the program.

**Information Stored**:
| Identifier | Type | Scope | Memory Location | Other Attributes |
|------------|------|-------|-----------------|------------------|
| position | float | global | 1000 | - |
| rate | float | global | 1004 | - |
| initial | float | global | 1008 | - |

**Operations**:
- Insert: Add new identifier
- Lookup: Find identifier information
- Delete: Remove identifier (when out of scope)

### 1.6.2 Error Handler

**Types of Errors**:

1. **Lexical Errors**: Invalid characters, malformed tokens
   - Example: `$abc` (invalid character $)

2. **Syntax Errors**: Violation of grammar rules
   - Example: `if x > 5` (missing parentheses)

3. **Semantic Errors**: Type mismatches, undeclared variables
   - Example: `int x = "hello";`

4. **Logical Errors**: Program runs but produces wrong output
   - Not detected by compiler

**Error Recovery Strategies**:
- Panic mode: Skip input until a synchronizing token
- Phrase level: Local correction on remaining input
- Error productions: Augment grammar with error rules
- Global correction: Minimal changes to correct program

---

## 1.7 Compiler Construction Tools

These tools automate parts of compiler construction:

| Tool | Purpose | Famous Examples |
|------|---------|-----------------|
| **Scanner Generator** | Generates lexical analyzer from regex | LEX, FLEX |
| **Parser Generator** | Generates parser from grammar | YACC, Bison |
| **Syntax-Directed Translation Engine** | Produces IR from parse tree | - |
| **Code Generator Generator** | Produces code generator from rules | - |
| **Data Flow Analysis Engine** | Gathers information for optimization | - |
| **Compiler-Compiler** | Complete compiler generation | ANTLR |

---

## 1.8 Bootstrapping

**Bootstrapping** is the technique of writing a compiler in the language it compiles.

### The Chicken-and-Egg Problem

**Question**: How do you compile the first C compiler if it's written in C?

**Solution**: Use a **T-diagram** (Tombstone diagram) approach:

```
Step 1: Write a simple C compiler in Assembly
        ┌───────────────────┐
        │    C → ASM        │
        │                   │
        │       ASM         │
        └───────────────────┘
        
Step 2: Write a better C compiler in C, compile with Step 1
        ┌───────────────────┐
        │    C → ASM        │
        │                   │
        │        C          │  ← Source is in C
        └───────────────────┘
        
Step 3: Compile the C compiler with itself
        - Now you have a self-compiling compiler!
```

---

## 1.9 Passes of a Compiler

A **pass** is one complete traversal of the source program (or its representation).

### Single-Pass Compiler
- All phases work together in one pass
- Fast compilation
- Limited optimization
- Language must allow forward declaration

### Multi-Pass Compiler
- Each pass performs one or more phases
- Better optimization opportunities
- Slower compilation
- More flexible language design

```
┌────────────────────────────────────────────────────────┐
│                    MULTI-PASS COMPILER                  │
│                                                         │
│  Pass 1: Lexical + Syntax Analysis                     │
│          ↓                                              │
│  Pass 2: Semantic Analysis + IR Generation             │
│          ↓                                              │
│  Pass 3: Optimization                                   │
│          ↓                                              │
│  Pass 4: Code Generation                               │
└────────────────────────────────────────────────────────┘
```

---

## 1.10 Summary

### Key Takeaways:

1. **Compiler** translates entire source code before execution; **Interpreter** executes line by line

2. **Six Phases** of compilation:
   - Lexical Analysis → Tokens
   - Syntax Analysis → Parse Tree
   - Semantic Analysis → Annotated Tree
   - Intermediate Code → IR
   - Optimization → Improved IR
   - Code Generation → Target Code

3. **Symbol Table** and **Error Handler** support all phases

4. **Analysis Phase** understands the source; **Synthesis Phase** creates the target

5. **Bootstrapping** allows compilers to compile themselves

---

## 🔍 Practice Questions

1. Differentiate between compiler and interpreter with examples.
2. Explain the six phases of a compiler with a suitable example.
3. What is bootstrapping? How is it achieved?
4. Describe the role of symbol table in compilation.
5. What are the different types of errors detected by a compiler?

---

*Next Chapter: [Lexical Analysis](02-Lexical-Analysis.md)*
