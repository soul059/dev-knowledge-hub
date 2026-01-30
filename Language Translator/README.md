# 📚 Compiler Design - Complete Study Notes

## Course Overview

**Compiler Design** (also known as **Language Translator**) is the study of how high-level programming languages are translated into machine-executable code. This subject forms the backbone of programming language implementation.

---

## 📖 Table of Contents

### Unit 1: Foundations
1. [Introduction to Compilers](01-Introduction-to-Compilers.md)
   - What is a Compiler?
   - Translators: Compiler vs Interpreter vs Assembler
   - Phases of Compilation
   - Compiler Construction Tools

### Unit 2: Lexical Analysis
2. [Lexical Analysis](02-Lexical-Analysis.md)
   - Role of Lexical Analyzer
   - Tokens, Patterns, and Lexemes
   - Regular Expressions
   - Finite Automata (DFA & NFA)
   - LEX Tool

### Unit 3: Syntax Analysis
3. [Syntax Analysis - Part 1: Fundamentals](03-Syntax-Analysis-Fundamentals.md)
   - Context-Free Grammars
   - Parse Trees and Derivations
   - Ambiguity in Grammars

4. [Syntax Analysis - Part 2: Top-Down Parsing](04-Top-Down-Parsing.md)
   - Recursive Descent Parsing
   - LL(1) Parsing
   - FIRST and FOLLOW Sets

5. [Syntax Analysis - Part 3: Bottom-Up Parsing](05-Bottom-Up-Parsing.md)
   - Shift-Reduce Parsing
   - LR Parsing (SLR, CLR, LALR)
   - Operator Precedence Parsing

### Unit 4: Semantic Analysis
6. [Semantic Analysis](06-Semantic-Analysis.md)
   - Syntax-Directed Definitions
   - Attribute Grammars
   - Type Checking
   - Symbol Table Management

### Unit 5: Intermediate Code Generation
7. [Intermediate Code Generation](07-Intermediate-Code-Generation.md)
   - Intermediate Representations
   - Three-Address Code
   - Quadruples, Triples, and Indirect Triples
   - Translation of Expressions and Statements

### Unit 6: Code Optimization
8. [Code Optimization](08-Code-Optimization.md)
   - Machine-Independent Optimization
   - Machine-Dependent Optimization
   - Loop Optimization
   - Data Flow Analysis

### Unit 7: Code Generation
9. [Code Generation](09-Code-Generation.md)
   - Issues in Code Generation
   - Target Machine Architecture
   - Register Allocation
   - Instruction Selection

### Unit 8: Runtime Environment
10. [Runtime Environment](10-Runtime-Environment.md)
    - Storage Organization
    - Stack Allocation
    - Heap Management
    - Parameter Passing

---

## 🎯 Learning Objectives

After completing these notes, you will be able to:

1. **Understand** the complete compilation process from source to target code
2. **Design** lexical analyzers using regular expressions and finite automata
3. **Construct** parsers using various parsing techniques
4. **Implement** semantic analysis and type checking
5. **Generate** intermediate code and optimize it
6. **Produce** efficient target machine code

---

## 📝 How to Use These Notes

1. **Sequential Reading**: Follow the chapters in order for a structured understanding
2. **Concept Building**: Each chapter builds upon previous concepts
3. **Practice**: Work through the examples and try variations
4. **Code Snippets**: Understand the logic, don't just memorize code

---

## 🔑 Key Terminology Quick Reference

| Term | Meaning |
|------|---------|
| **Compiler** | Translates entire source code to target code before execution |
| **Interpreter** | Executes source code line by line |
| **Token** | Basic meaningful unit identified by lexer |
| **Parse Tree** | Hierarchical representation of grammar derivation |
| **Semantic** | Meaning-related aspects of the program |
| **IR** | Intermediate Representation |
| **Optimization** | Improving code efficiency without changing behavior |

---

*Happy Learning! 🚀*
