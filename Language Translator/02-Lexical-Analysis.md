# Chapter 2: Lexical Analysis

## 2.1 Introduction to Lexical Analysis

**Lexical Analysis** is the **first phase** of a compiler. It reads the source program as a stream of characters and groups them into meaningful sequences called **tokens**.

Problem: Parsing characters directly is hard and inefficient.

Solution: The lexer groups characters into tokens so the parser works on higher-level units.

### The Role of Lexical Analyzer (Scanner)

```
┌──────────────┐    ┌─────────────────┐    ┌────────────────┐
│    Source    │───▶│ Lexical Analyzer│───▶│   Syntax       │
│   Program    │    │   (Scanner)     │    │   Analyzer     │
│              │    │                 │    │   (Parser)     │
│ (characters) │◀───│                 │    │                │
└──────────────┘    └─────────────────┘    └────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Symbol Table │
                    └──────────────┘
```

### Why Separate Lexical Analysis?

1. **Simplicity**: Separating scanning from parsing simplifies both phases
2. **Efficiency**: Specialized buffering techniques for character handling
3. **Portability**: Character-set dependencies are isolated
4. **Modularity**: Easier to maintain and modify

Problem: Mixing scanning and parsing complicates both.

Solution: Separate lexical analysis for clean design and better performance.

---

## 2.2 Tokens, Patterns, and Lexemes

These three concepts are fundamental to understanding lexical analysis:

Problem: The compiler must connect raw text to language symbols.

Solution: Tokens classify text, lexemes are the actual strings, and patterns define valid lexemes.

### Token
A **token** is a pair consisting of:
- **Token name**: An abstract symbol representing a kind of lexical unit
- **Token value** (optional): Additional information about the token

**Token categories**:
| Category | Examples |
|----------|----------|
| Keywords | if, else, while, for, int, float |
| Identifiers | variable names, function names |
| Operators | +, -, *, /, =, ==, !=, <, > |
| Constants | 42, 3.14, 'a', "hello" |
| Punctuation | ; , ( ) { } [ ] |
| Special Symbols | #, @, $ (language dependent) |

### Lexeme
A **lexeme** is the actual character sequence in the source code that matches the pattern for a token.

### Pattern
A **pattern** is a rule that describes the set of lexemes that can represent a particular token. Patterns are often specified using **regular expressions**.

### Example to Understand All Three

```c
int count = 100;
```

| Lexeme | Token Type | Token Value |
|--------|------------|-------------|
| int | KEYWORD | int |
| count | IDENTIFIER | pointer to symbol table entry |
| = | ASSIGN_OP | - |
| 100 | NUMBER | 100 |
| ; | SEMICOLON | - |

**Pattern for IDENTIFIER**: A letter followed by zero or more letters or digits
```
letter (letter | digit)*
```

---

## 2.3 Attributes for Tokens

When a token has multiple possible lexemes, we need **attributes** to distinguish them.

### Example

For the statement: `x = y + 5`

| Token | Attribute |
|-------|-----------|
| ID | Pointer to symbol table entry for 'x' |
| ASSIGN_OP | = |
| ID | Pointer to symbol table entry for 'y' |
| ADD_OP | + |
| NUM | Value = 5 |

The token-attribute pair is passed to the parser:
```
<ID, ptr_to_x>
<ASSIGN_OP, =>
<ID, ptr_to_y>
<ADD_OP, +>
<NUM, 5>
```

---

## 2.4 Input Buffering

Reading source code character-by-character from disk is **slow**. Compilers use buffering techniques for efficiency.

Problem: Direct file I/O for each character is too slow.

Solution: Use buffers with sentinel characters to reduce checks and speed scanning.

### Buffer Pairs Scheme

Two buffers of the same size (e.g., 4096 bytes each) are used alternately:

```
┌─────────────────────────────────────┐
│  Buffer 1           │  Buffer 2    │
├─────────────────────┼──────────────┤
│ E = M * C * * 2 eof │ ............ │
│ ↑               ↑   │              │
│ lexeme_beginning│   │              │
│             forward │              │
└─────────────────────┴──────────────┘
```

**Two pointers**:
- **lexeme_beginning**: Points to the start of the current lexeme
- **forward**: Scans ahead to find the end of the lexeme

**Sentinel Character (eof)**:
- Placed at the end of each buffer
- Reduces the number of tests in the inner loop

### Algorithm for Advancing Forward Pointer

```
if forward is at end of first half then
    reload second half
    forward = beginning of second half
else if forward is at end of second half then
    reload first half
    forward = beginning of first half
else
    forward = forward + 1
```

---

## 2.5 Regular Expressions

**Regular expressions** are the formal notation used to specify patterns for tokens. They describe **regular languages**.

### Basic Operations

Let **r** and **s** be regular expressions denoting languages L(r) and L(s):

| Operation | Notation | Meaning |
|-----------|----------|---------|
| **Union** | r \| s | L(r) ∪ L(s) - either r or s |
| **Concatenation** | rs | L(r)L(s) - r followed by s |
| **Kleene Closure** | r* | Zero or more occurrences of r |
| **Positive Closure** | r⁺ | One or more occurrences of r |
| **Optional** | r? | Zero or one occurrence of r |

### Regular Expression Building Blocks

| Symbol | Meaning |
|--------|---------|
| ε (epsilon) | Empty string |
| a (any character) | Matches itself |
| . | Matches any single character |
| [abc] | Character class - matches a, b, or c |
| [a-z] | Range - matches any lowercase letter |
| [^abc] | Negation - matches anything except a, b, c |
| \d | Digit (equivalent to [0-9]) |
| \w | Word character [a-zA-Z0-9_] |
| \s | Whitespace |

### Examples of Regular Expressions for Tokens

#### 1. Identifiers
An identifier starts with a letter and followed by letters or digits:
```
letter = [a-zA-Z_]
digit = [0-9]
identifier = letter(letter | digit)*
```

#### 2. Integer Constants
```
digit = [0-9]
integer = digit⁺
```

#### 3. Floating Point Numbers
```
digit = [0-9]
digits = digit⁺
fraction = . digits | ε
exponent = (E (+ | - | ε) digits) | ε
float = digits fraction exponent
```

This matches: `3.14`, `2.5E10`, `1.5E-3`, `42`

#### 4. Keywords
Keywords are fixed strings:
```
keyword = if | else | while | for | int | float | return
```

#### 5. Relational Operators
```
relop = < | > | <= | >= | == | !=
```

### Regular Definitions

For convenience, we can give names to regular expressions:

```
letter → [a-zA-Z]
digit → [0-9]
digits → digit⁺
id → letter(letter | digit)*
number → digits(. digits)? (E[+-]? digits)?
```

---

## 2.6 Finite Automata

**Finite Automata** are mathematical models used to recognize patterns described by regular expressions. They are the implementation mechanism for lexical analyzers.

### Formal Definition

A Finite Automaton is a 5-tuple: **M = (Q, Σ, δ, q₀, F)**

| Component | Meaning |
|-----------|---------|
| Q | Finite set of states |
| Σ | Input alphabet (set of symbols) |
| δ | Transition function |
| q₀ | Start state (q₀ ∈ Q) |
| F | Set of accepting/final states (F ⊆ Q) |

### Types of Finite Automata

## 2.6.1 Non-deterministic Finite Automata (NFA)

In an NFA:
- A state can have **multiple transitions** on the same input symbol
- **ε-transitions** (transitions without consuming input) are allowed
- Can be in **multiple states** simultaneously

**Transition Function**: δ: Q × (Σ ∪ {ε}) → P(Q)
(P(Q) is power set of Q - set of all subsets)

**Example NFA for (a|b)*abb**:

```
        ε           a           b           b
→(0) ─────▶ (1) ─────▶ (2) ─────▶ (3) ─────▶ ((4))
  │          ↑                              
  │    a,b   │                              
  └──────────┘                              
```

**State Diagram**:
```
             a,b
              ↓
    ┌─────────────┐
    │             │
    ▼    a        b        b
→ (0) ────▶ (1) ────▶ (2) ────▶ ((3))
    ↑             │
    └─────────────┘
          a,b
```

## 2.6.2 Deterministic Finite Automata (DFA)

In a DFA:
- Each state has **exactly one transition** for each input symbol
- **No ε-transitions**
- Always in **exactly one state**

**Transition Function**: δ: Q × Σ → Q

**Example DFA for (a|b)*abb**:

```
State Transition Table:
┌───────┬───────┬───────┐
│ State │   a   │   b   │
├───────┼───────┼───────┤
│ →0    │   1   │   0   │
│   1   │   1   │   2   │
│   2   │   1   │   3   │
│  *3   │   1   │   0   │
└───────┴───────┴───────┘
(→ = start, * = accepting)
```

**State Diagram**:
```
              b
        ┌───────────┐
        │           │
        ▼     a     │     b           b
    → (0) ────▶ (1) ────▶ (2) ────▶ ((3))
        ↑     ↗         ↙ a          │
        │   a ┘    ┌────┘            │
        │          │                 │ b
        └──────────┴─────────────────┘
```

### NFA vs DFA Comparison

| Aspect | NFA | DFA |
|--------|-----|-----|
| Multiple transitions on same input | Yes | No |
| ε-transitions | Yes | No |
| Ease of construction | Easier | Harder |
| Pattern recognition | Multiple paths | Single path |
| States for equivalent FA | Fewer | May be more (up to 2ⁿ) |
| Implementation | Complex | Simple |
| Execution speed | Slower | Faster |

---

## 2.7 Conversion: Regular Expression to NFA

**Thompson's Construction** is the standard algorithm to convert a regular expression to an NFA.

### Basic Rules

#### Rule 1: For ε (empty string)
```
    ε
→(i)────▶((f))
```

#### Rule 2: For a single character 'a'
```
    a
→(i)────▶((f))
```

### Inductive Rules (Building larger NFA from smaller ones)

#### Rule 3: Union (r | s)
If N(r) and N(s) are NFAs for r and s:

```
           ε     ┌─────────┐
         ┌───▶   │   N(r)   │ ───┐
         │       └─────────┘    │ ε
    → (i)│                      ▼
         │       ┌─────────┐    ((f))
         └───▶   │   N(s)   │ ───┘
           ε     └─────────┘   ε
```

#### Rule 4: Concatenation (rs)
```
         ┌─────────┐     ┌─────────┐
    →    │   N(r)   │ ───▶│   N(s)   │ ───▶ ((f))
         └─────────┘     └─────────┘
```

#### Rule 5: Kleene Closure (r*)
```
                    ε
              ┌──────────────┐
              │              │
              ▼  ┌─────────┐ │
    → (i) ─────▶ │   N(r)   │─┘ ───▶ ((f))
          │ε    └─────────┘      ε  ↑
          │                         │
          └─────────────────────────┘
                    ε
```

### Complete Example: (a|b)*abb

**Step 1**: Build NFA for 'a':
```
    a
(1)───▶(2)
```

**Step 2**: Build NFA for 'b':
```
    b
(3)───▶(4)
```

**Step 3**: Build NFA for 'a|b':
```
      ε   ┌───a───┐  ε
    ┌───▶ (1)───▶(2) ──┐
(0)─┤                   ├──▶(5)
    └───▶ (3)───▶(4) ──┘
      ε   └───b───┘  ε
```

**Step 4**: Build NFA for '(a|b)*':
```
Apply Kleene closure to the above NFA
```

**Step 5**: Concatenate with 'abb':
```
Continue building NFAs for 'a', 'b', 'b' and concatenate
```

---

## 2.8 Conversion: NFA to DFA (Subset Construction)

Since NFAs can be in multiple states simultaneously, we convert NFA to DFA by treating **sets of NFA states** as **single DFA states**.

### Key Operations

| Operation | Definition |
|-----------|------------|
| **ε-closure(s)** | Set of NFA states reachable from state s on ε-transitions alone |
| **ε-closure(T)** | Union of ε-closure(s) for all states s in set T |
| **move(T, a)** | Set of NFA states reachable from any state in T on input symbol a |

### Subset Construction Algorithm

```
Input: NFA N
Output: DFA D

Initialize:
    Dstates = {ε-closure(s₀)}  // Start with ε-closure of NFA start state
    Mark all states in Dstates as "unmarked"

while there is an unmarked state T in Dstates:
    mark T
    for each input symbol a:
        U = ε-closure(move(T, a))
        if U is not in Dstates:
            add U to Dstates (unmarked)
        Dtran[T, a] = U

DFA start state: ε-closure(s₀)
DFA accepting states: Any DFA state containing an NFA accepting state
```

### Example: Convert NFA to DFA

**NFA for (a|b)*abb:**

| State | a | b | ε |
|-------|---|---|---|
| 0 | - | - | {1,7} |
| 1 | - | - | {2,4} |
| 2 | {3} | - | - |
| 3 | - | - | {6} |
| 4 | - | {5} | - |
| 5 | - | - | {6} |
| 6 | - | - | {1,7} |
| 7 | {8} | - | - |
| 8 | - | {9} | - |
| 9 | - | {10} | - |
| 10 | - | - | - |

**Step-by-step subset construction:**

**Step 1**: Compute initial state
```
ε-closure({0}) = {0, 1, 2, 4, 7} = A (Start state of DFA)
```

**Step 2**: Compute transitions for A
```
move(A, a) = move({0,1,2,4,7}, a) = {3, 8}
ε-closure({3, 8}) = {1, 2, 3, 4, 6, 7, 8} = B

move(A, b) = move({0,1,2,4,7}, b) = {5}
ε-closure({5}) = {1, 2, 4, 5, 6, 7} = C
```

**Continue for B and C...**

**Final DFA Transition Table:**

| State | a | b |
|-------|---|---|
| →A | B | C |
| B | B | D |
| C | B | C |
| D | B | E |
| *E | B | C |

---

## 2.9 Minimization of DFA

Two states are **equivalent** if they produce the same output for all possible input strings. We can merge equivalent states to get a **minimal DFA**.

### Partition Refinement Algorithm

**Step 1**: Start with two partitions:
- Final states (F)
- Non-final states (Q - F)

**Step 2**: Refine partitions:
For each partition, check if all states in it have transitions to the same partition on each input.
If not, split the partition.

**Step 3**: Repeat until no more splitting is possible.

### Example

Given DFA:
```
States: {A, B, C, D, E}
Final states: {E}
```

**Initial Partition:**
```
Π₀ = {{A, B, C, D}, {E}}
```

**Iteration 1:**
Check if states in {A, B, C, D} go to same partitions:
- On 'b': A→C, B→D, C→C, D→E
- D goes to different partition, so split

```
Π₁ = {{A, B, C}, {D}, {E}}
```

**Continue until no more splits...**

**Minimal DFA** has fewer states than original.

---

## 2.10 Lexical Analyzer Generator: LEX

**LEX** is a tool that automatically generates a lexical analyzer from pattern specifications.

### Structure of a LEX Program

```lex
%{
    /* Declarations Section */
    /* C code: includes, variable declarations */
    #include <stdio.h>
    int line_count = 0;
%}

/* Definitions Section */
DIGIT   [0-9]
LETTER  [a-zA-Z]
ID      {LETTER}({LETTER}|{DIGIT})*

%%
    /* Rules Section */
    /* pattern    action */
    
{DIGIT}+        { printf("INTEGER: %s\n", yytext); }
{ID}            { printf("IDENTIFIER: %s\n", yytext); }
"if"            { printf("KEYWORD: IF\n"); }
"else"          { printf("KEYWORD: ELSE\n"); }
"+"             { printf("OPERATOR: PLUS\n"); }
"-"             { printf("OPERATOR: MINUS\n"); }
[ \t]           { /* ignore whitespace */ }
\n              { line_count++; }
.               { printf("UNKNOWN: %s\n", yytext); }

%%
    /* User Code Section */
    
int main() {
    yylex();  // Call the lexer
    printf("Lines: %d\n", line_count);
    return 0;
}

int yywrap() {
    return 1;  // No more input files
}
```

### LEX Workflow

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  LEX Source │   LEX   │   lex.yy.c  │    CC   │   a.out     │
│   (spec.l)  │ ──────▶ │  (C code)   │ ──────▶ │  (lexer)    │
└─────────────┘         └─────────────┘         └─────────────┘
```

### Important LEX Variables and Functions

| Name | Purpose |
|------|---------|
| `yytext` | Pointer to matched string |
| `yyleng` | Length of matched string |
| `yylex()` | Main function, returns next token |
| `yyin` | Input file pointer (default: stdin) |
| `yyout` | Output file pointer (default: stdout) |
| `ECHO` | Write matched string to output |

### LEX Pattern Matching Rules

1. **Longest Match**: LEX prefers the pattern that matches the longest input
2. **First Match**: If two patterns match the same length, the first one in the file is chosen
3. **Keywords before Identifiers**: List specific keywords before general identifier pattern

---

## 2.11 Implementation of Lexical Analyzer

### Simple Lexer in C

```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>

#define MAX_TOKEN_LEN 100

typedef enum {
    TOKEN_INT, TOKEN_FLOAT, TOKEN_ID, 
    TOKEN_KEYWORD, TOKEN_ASSIGN, 
    TOKEN_PLUS, TOKEN_MINUS, TOKEN_MULT, TOKEN_DIV,
    TOKEN_LPAREN, TOKEN_RPAREN,
    TOKEN_SEMICOLON, TOKEN_EOF, TOKEN_ERROR
} TokenType;

const char *keywords[] = {"if", "else", "while", "for", "int", "float", "return"};
#define NUM_KEYWORDS 7

char token_value[MAX_TOKEN_LEN];

int is_keyword(const char *str) {
    for (int i = 0; i < NUM_KEYWORDS; i++) {
        if (strcmp(str, keywords[i]) == 0) return 1;
    }
    return 0;
}

TokenType get_next_token(FILE *fp) {
    int c;
    int i = 0;
    
    // Skip whitespace
    while ((c = fgetc(fp)) != EOF && isspace(c));
    
    if (c == EOF) return TOKEN_EOF;
    
    // Identifier or Keyword
    if (isalpha(c) || c == '_') {
        token_value[i++] = c;
        while ((c = fgetc(fp)) != EOF && (isalnum(c) || c == '_')) {
            token_value[i++] = c;
        }
        ungetc(c, fp);  // Put back last character
        token_value[i] = '\0';
        return is_keyword(token_value) ? TOKEN_KEYWORD : TOKEN_ID;
    }
    
    // Number (integer or float)
    if (isdigit(c)) {
        token_value[i++] = c;
        while ((c = fgetc(fp)) != EOF && isdigit(c)) {
            token_value[i++] = c;
        }
        if (c == '.') {  // Float
            token_value[i++] = c;
            while ((c = fgetc(fp)) != EOF && isdigit(c)) {
                token_value[i++] = c;
            }
            ungetc(c, fp);
            token_value[i] = '\0';
            return TOKEN_FLOAT;
        }
        ungetc(c, fp);
        token_value[i] = '\0';
        return TOKEN_INT;
    }
    
    // Operators and punctuation
    token_value[0] = c;
    token_value[1] = '\0';
    
    switch (c) {
        case '+': return TOKEN_PLUS;
        case '-': return TOKEN_MINUS;
        case '*': return TOKEN_MULT;
        case '/': return TOKEN_DIV;
        case '=': return TOKEN_ASSIGN;
        case '(': return TOKEN_LPAREN;
        case ')': return TOKEN_RPAREN;
        case ';': return TOKEN_SEMICOLON;
        default:  return TOKEN_ERROR;
    }
}
```

### How the Lexer Works

1. **Read characters** from input
2. **Skip whitespace** and comments
3. **Recognize patterns**:
   - If letter → read identifier/keyword
   - If digit → read number
   - If operator → return operator token
4. **Return token** with its value

---

## 2.12 Handling Errors in Lexical Analysis

### Common Lexical Errors

1. **Illegal characters**: Characters not in the alphabet
   - Example: `$` in standard C

2. **Malformed tokens**: Tokens that don't match any pattern
   - Example: `1.2.3` (invalid number)

3. **Unterminated strings**: Missing closing quote
   - Example: `"hello` (no closing ")

4. **Unterminated comments**: Missing closing delimiter
   - Example: `/* comment` (no closing */)

### Error Recovery Strategies

1. **Delete character**: Remove the erroneous character
2. **Insert character**: Add a missing character
3. **Replace character**: Substitute correct character
4. **Transpose characters**: Swap adjacent characters

### Panic Mode Recovery

When an error is detected:
1. Skip characters until a valid token start is found
2. Report the error with line and column number
3. Continue lexical analysis

```c
void panic_mode_recovery(FILE *fp) {
    int c;
    fprintf(stderr, "Lexical Error: Illegal character, skipping...\n");
    // Skip until whitespace or valid token start
    while ((c = fgetc(fp)) != EOF) {
        if (isspace(c) || isalpha(c) || isdigit(c)) {
            ungetc(c, fp);
            break;
        }
    }
}
```

---

## 2.13 Summary

### Key Points:

1. **Lexical Analyzer** converts character stream to token stream

2. **Token** = abstract symbol, **Lexeme** = actual text, **Pattern** = rule

3. **Regular Expressions** describe token patterns formally

4. **Finite Automata** implement pattern recognition:
   - NFA: Multiple paths, ε-transitions allowed
   - DFA: Single path, deterministic

5. **Conversions**:
   - RE → NFA (Thompson's Construction)
   - NFA → DFA (Subset Construction)
   - DFA → Minimal DFA (Partition Refinement)

6. **LEX** automates lexer generation from specifications

---

## 🔍 Practice Questions

1. Write regular expressions for:
   - Valid email addresses
   - C-style comments
   - Binary numbers

2. Convert the regular expression `(a|b)*a(a|b)` to NFA using Thompson's construction.

3. Convert the following NFA to DFA using subset construction.

4. Minimize the given DFA using partition refinement.

5. Write a LEX program to count words, lines, and characters in a file.

---

*Next Chapter: [Syntax Analysis - Fundamentals](03-Syntax-Analysis-Fundamentals.md)*
