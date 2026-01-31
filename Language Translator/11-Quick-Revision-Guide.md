# Compiler Design - Quick Revision Guide

## One-Page Summary for Each Unit

---

## Unit 1: Introduction to Compilers

### Key Definitions
- **Compiler**: Translates entire source code to target code before execution
- **Interpreter**: Executes source code line by line
- **Assembler**: Converts assembly to machine code

### Six Phases of Compilation
```
Source → [Lexical] → [Syntax] → [Semantic] → [IR Gen] → [Optimize] → [Code Gen] → Target
```

| Phase | Input | Output |
|-------|-------|--------|
| Lexical Analysis | Characters | Tokens |
| Syntax Analysis | Tokens | Parse Tree |
| Semantic Analysis | Parse Tree | Annotated Tree |
| IR Generation | Annotated Tree | IR Code |
| Optimization | IR Code | Optimized IR |
| Code Generation | Optimized IR | Target Code |

### Supporting Modules
- **Symbol Table**: Stores identifier information
- **Error Handler**: Reports and recovers from errors

---

## Unit 2: Lexical Analysis

### Token-Lexeme-Pattern
- **Token**: Category (IDENTIFIER, NUMBER)
- **Lexeme**: Actual text ("count", "123")
- **Pattern**: Rule describing valid lexemes

### Regular Expression Operators
| Op | Meaning | Example |
|----|---------|---------|
| \| | Union | a\|b = {a, b} |
| . | Concatenation | ab = {ab} |
| * | Kleene closure | a* = {ε, a, aa, ...} |
| + | Positive closure | a+ = {a, aa, ...} |
| ? | Optional | a? = {ε, a} |

### Finite Automata
| | NFA | DFA |
|---|-----|-----|
| Transitions | Multiple on same input | Exactly one |
| ε-moves | Yes | No |
| States | Fewer | May be 2^n |

### Key Conversions
1. **RE → NFA**: Thompson's Construction
2. **NFA → DFA**: Subset Construction (ε-closure, move)
3. **DFA → Minimal DFA**: Partition Refinement

---

## Unit 3: Syntax Analysis Fundamentals

### Context-Free Grammar
G = (V, T, P, S) where V=non-terminals, T=terminals, P=productions, S=start

### Derivations
- **Leftmost**: Replace leftmost non-terminal first
- **Rightmost**: Replace rightmost non-terminal first

### Grammar Issues

| Problem | Example | Solution |
|---------|---------|----------|
| Ambiguity | E → E+E \| E*E | Rewrite with precedence |
| Left Recursion | A → Aα \| β | A → βA', A' → αA' \| ε |
| Common Prefix | A → αβ \| αγ | A → αA', A' → β \| γ |

---

## Unit 4: Top-Down Parsing

### FIRST Set Rules
1. If X is terminal: FIRST(X) = {X}
2. If X → ε: Add ε to FIRST(X)
3. If X → Y₁Y₂...Yₖ: Add FIRST(Y₁) - {ε}, continue if ε ∈ FIRST(Yᵢ)

### FOLLOW Set Rules
1. Add $ to FOLLOW(Start)
2. A → αBβ: Add FIRST(β) - {ε} to FOLLOW(B)
3. A → αB or ε ∈ FIRST(β): Add FOLLOW(A) to FOLLOW(B)

### LL(1) Table Construction
For A → α:
1. Add A → α to M[A, a] for each a ∈ FIRST(α)
2. If ε ∈ FIRST(α): Add A → α to M[A, b] for each b ∈ FOLLOW(A)

### LL(1) Condition
Grammar is LL(1) if each table cell has at most one entry.

---

## Unit 5: Bottom-Up Parsing

### Key Concepts
- **Shift**: Push input symbol onto stack
- **Reduce**: Replace handle with non-terminal
- **Handle**: RHS that should be reduced next

### LR Parsing Table
- **ACTION[s, a]**: shift, reduce, accept, or error
- **GOTO[s, A]**: Next state after reducing to A

### LR Variants
| Type | Items | Power | Table Size |
|------|-------|-------|------------|
| SLR(1) | LR(0) + FOLLOW | Least | Small |
| LALR(1) | Merged LR(1) | Medium | Medium |
| CLR(1) | LR(1) | Most | Large |

### Item Forms
- **LR(0)**: A → α•β
- **LR(1)**: [A → α•β, a] (includes lookahead)

---

## Unit 6: Semantic Analysis

### Attribute Types
| Type | Computed From | Flow |
|------|---------------|------|
| Synthesized | Children | Bottom-up |
| Inherited | Parent/Siblings | Top-down |

### Type Checking Rules
```
int op int → int
float op float → float
int op float → float (coerce int)
```

### Symbol Table Operations
- **insert(name, type, offset)**
- **lookup(name)** → returns entry or null
- **enter_scope()** / **exit_scope()**

---

## Unit 7: Intermediate Code

### Three-Address Code Forms
| | Quadruple | Triple | Indirect Triple |
|---|-----------|--------|-----------------|
| Fields | (op, arg1, arg2, result) | (op, arg1, arg2) | Pointers to triples |
| Result | Explicit | Instruction # | Via pointer |

### Common TAC Instructions
```
x = y op z    // Binary operation
x = op y      // Unary operation  
x = y         // Copy
goto L        // Unconditional jump
if x relop y goto L  // Conditional jump
param x       // Parameter
call p, n     // Function call
return x      // Return
```

### Backpatching
- **makelist(i)**: Create list with instruction i
- **merge(l1, l2)**: Concatenate lists
- **backpatch(l, j)**: Fill all instructions in l with target j

---

## Unit 8: Code Optimization

### Local Optimizations
| Technique | Before | After |
|-----------|--------|-------|
| CSE | t1=a+b; t2=a+b | t1=a+b; t2=t1 |
| Copy Prop | t1=a; t2=t1+b | t1=a; t2=a+b |
| Dead Code | t1=a; (unused) | (remove) |
| Const Fold | t1=3*5 | t1=15 |
| Strength Red | x*2 | x<<1 |

### Data Flow Analysis
| Analysis | Direction | Meet | Use |
|----------|-----------|------|-----|
| Reaching Defs | Forward | Union | CSE, const prop |
| Live Variables | Backward | Union | Dead code elim |
| Available Expr | Forward | Intersect | Global CSE |

### Loop Optimizations
1. **Code Motion**: Move invariants outside loop
2. **Induction Variable**: Replace multiplication with addition
3. **Loop Unrolling**: Reduce overhead

---

## Unit 9: Code Generation

### Register/Address Descriptors
- **Register Desc**: Which variables are in each register
- **Address Desc**: Where each variable's value is stored

### getReg Algorithm
1. Use register with only dead value
2. Use empty register
3. Spill least recently used

### Instruction Selection Factors
- Addressing modes available
- Special instructions (MAC, SIMD)
- Register constraints

---

## Unit 10: Runtime Environment

### Memory Layout
```
High → [Stack (grows down)]
       [Free Space]
       [Heap (grows up)]
       [Static Data]
Low  → [Code]
```

### Activation Record Contents
```
┌─ Arguments ─┐
│ Return Addr │
│ Control Link│  
│ Access Link │ ← FP
│ Locals      │
│ Saved Regs  │ ← SP
└─────────────┘
```

### Parameter Passing
| Method | Passed | Changes Visible? |
|--------|--------|------------------|
| Value | Copy | No |
| Reference | Address | Yes |
| Value-Result | Copy, then back | At return |

### Garbage Collection
| Method | Handles Cycles? | Compacts? |
|--------|-----------------|-----------|
| Ref Counting | No | No |
| Mark-Sweep | Yes | No |
| Copying | Yes | Yes |

---

## Important Formulas

### Array Address Calculation
```
1D: A[i] = base + i * w
2D Row-Major: A[i][j] = base + (i * n₂ + j) * w
2D Col-Major: A[i][j] = base + (j * n₁ + i) * w
```

### NFA to DFA
```
ε-closure(s): All states reachable from s via ε
move(T, a): All states reachable from T on input a
DFA state = ε-closure(move(current_state, input))
```

---

## 🎯 Common Exam Questions

1. **Phases of compiler** - Draw diagram, explain each phase
2. **RE to NFA to DFA** - Thompson's + Subset construction
3. **FIRST and FOLLOW** - Calculate for given grammar
4. **LL(1) table** - Construct and parse input
5. **LR parsing** - Items, states, parsing table
6. **Three-address code** - Generate for given expression
7. **Optimization** - Apply techniques to given code
8. **Activation records** - Draw stack for recursive function
9. **Parameter passing** - Compare different methods
10. **Type checking** - Write semantic rules

---

*Good luck with your exam! 🍀*
