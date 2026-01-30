# Chapter 7: Intermediate Code Generation

## 7.1 Introduction

**Intermediate Code Generation** is the phase that transforms the syntax tree (from parsing and semantic analysis) into an intermediate representation (IR) that is:
- Easier to generate from source
- Easier to translate to target machine code
- Machine independent (portable)

### Position in Compiler Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Semantic   │────▶│ Intermediate │────▶│    Code      │
│   Analyzer   │     │   Code Gen   │     │ Optimization │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │
    Syntax Tree          IR (TAC)           Optimized IR
```

### Why Intermediate Code?

```
Without IR:                 With IR:
n source languages          n source languages
m target machines              │
n × m compilers needed         ▼
                           Single IR
                               │
                           m targets
                           n + m components needed
```

**Benefits:**
1. **Portability**: One front-end, multiple back-ends
2. **Optimization**: Machine-independent optimizations on IR
3. **Modularity**: Easier compiler construction and maintenance
4. **Retargeting**: Easier to add new target machines

---

## 7.2 Intermediate Representations

### 7.2.1 Abstract Syntax Tree (AST)

A condensed parse tree with only essential structure.

**Example:** For `a = b + c * d`

```
       =
      / \
     a   +
        / \
       b   *
          / \
         c   d
```

**Characteristics:**
- Preserves program structure
- Operators are internal nodes
- Operands are leaves
- Easy to construct during parsing

### 7.2.2 Directed Acyclic Graph (DAG)

Like AST, but shares common subexpressions.

**Example:** For `a = b + c * d` and `e = c * d`

```
         =              =
        / \            / \
       a   +          e   │
          / \             │
         b   *  ◄─────────┘
            / \
           c   d
```

**Benefits:**
- Detects common subexpressions
- Saves memory
- Enables optimization

### 7.2.3 Three-Address Code (TAC)

A sequence of instructions, each with at most three operands.

**General form:**
```
x = y op z
```

Where:
- `x` is the result
- `y`, `z` are operands
- `op` is an operator

**Example:** For `a = b + c * d`
```
t1 = c * d
t2 = b + t1
a = t2
```

---

## 7.3 Three-Address Code in Detail

### Types of TAC Instructions

| Category | Forms | Example |
|----------|-------|---------|
| **Assignment** | x = y op z | t1 = a + b |
| | x = op y | t1 = -a |
| | x = y | t1 = a |
| **Indexed Assignment** | x = y[i] | t1 = a[i] |
| | x[i] = y | a[i] = t1 |
| **Pointer** | x = &y | t1 = &a |
| | x = *y | t1 = *p |
| | *x = y | *p = t1 |
| **Jumps** | goto L | goto L1 |
| | if x goto L | if t1 goto L1 |
| | if x relop y goto L | if a < b goto L1 |
| **Function** | param x | param a |
| | call p, n | call func, 3 |
| | return x | return t1 |

### Temporaries

TAC uses **temporary variables** (t1, t2, ...) to hold intermediate results.

**Key Insight**: Each temporary is assigned exactly once (can be optimized to registers).

---

## 7.4 Representations of TAC

### 7.4.1 Quadruples

A quadruple has four fields: **(op, arg1, arg2, result)**

**Example:** `a = b + c * d`

| Index | Op | Arg1 | Arg2 | Result |
|-------|-----|------|------|--------|
| 0 | * | c | d | t1 |
| 1 | + | b | t1 | t2 |
| 2 | = | t2 | - | a |

**Characteristics:**
- Easy to reorder instructions
- Each instruction is self-contained
- Requires more space (temporary names stored)

### 7.4.2 Triples

A triple has three fields: **(op, arg1, arg2)**, result is the instruction number.

**Example:** `a = b + c * d`

| Index | Op | Arg1 | Arg2 |
|-------|-----|------|------|
| 0 | * | c | d |
| 1 | + | b | (0) |
| 2 | = | a | (1) |

**Note**: (0) and (1) refer to results of instructions 0 and 1.

**Characteristics:**
- More compact (no result field)
- Harder to reorder (references break)
- References are implicit

### 7.4.3 Indirect Triples

Uses a separate list of pointers to triples.

**Example:** `a = b + c * d`

**Statement list:**
| Index | Pointer |
|-------|---------|
| 0 | (35) |
| 1 | (36) |
| 2 | (37) |

**Triple storage:**
| Address | Op | Arg1 | Arg2 |
|---------|-----|------|------|
| 35 | * | c | d |
| 36 | + | b | (35) |
| 37 | = | a | (36) |

**Benefits:**
- Allows reordering without breaking references
- Just reorder the pointer list

### Comparison of Representations

| Aspect | Quadruples | Triples | Indirect Triples |
|--------|-----------|---------|------------------|
| Space | Most | Less | Medium |
| Reordering | Easy | Difficult | Easy |
| Optimization | Easy | Difficult | Easy |

---

## 7.5 Translation of Expressions

### Simple Expressions

**Grammar with Semantic Actions:**

```
E → E₁ + T     { E.place = newtemp();
                emit(E.place, '=', E₁.place, '+', T.place); }

E → E₁ - T     { E.place = newtemp();
                emit(E.place, '=', E₁.place, '-', T.place); }

E → T          { E.place = T.place; }

T → T₁ * F     { T.place = newtemp();
                emit(T.place, '=', T₁.place, '*', F.place); }

T → F          { T.place = F.place; }

F → ( E )      { F.place = E.place; }

F → id         { F.place = id.entry; }
```

### Helper Functions

```c
int temp_counter = 0;

char* newtemp() {
    char* temp = malloc(10);
    sprintf(temp, "t%d", temp_counter++);
    return temp;
}

void emit(char* result, char op, char* arg1, char* op_sym, char* arg2) {
    printf("%s = %s %s %s\n", result, arg1, op_sym, arg2);
}
```

### Example: Translating `a = b * c + b * c`

**Parse and Generate:**

```
F → id(b)      F.place = b
T → F          T.place = b
F → id(c)      F.place = c
T → T * F      T.place = t1; emit: t1 = b * c
E → T          E.place = t1

F → id(b)      F.place = b
T → F          T.place = b
F → id(c)      F.place = c
T → T * F      T.place = t2; emit: t2 = b * c
E → E + T      E.place = t3; emit: t3 = t1 + t2

Assignment:    emit: a = t3
```

**Generated TAC:**
```
t1 = b * c
t2 = b * c
t3 = t1 + t2
a = t3
```

---

## 7.6 Translation of Boolean Expressions

Boolean expressions are used for:
1. Computing logical values
2. Controlling program flow (if, while)

### Numerical Representation

Treat boolean as integer: **0 = false, 1 = true**

```
E → E₁ or E₂    { E.place = newtemp();
                 emit(E.place, '=', E₁.place, 'or', E₂.place); }

E → E₁ and E₂   { E.place = newtemp();
                 emit(E.place, '=', E₁.place, 'and', E₂.place); }

E → not E₁      { E.place = newtemp();
                 emit(E.place, '=', 'not', E₁.place); }

E → E₁ relop E₂ { E.place = newtemp();
                 emit('if', E₁.place, relop, E₂.place, 'goto', nextinstr+3);
                 emit(E.place, '=', '0');
                 emit('goto', nextinstr+2);
                 emit(E.place, '=', '1'); }
```

### Short-Circuit Evaluation

Evaluate only what's needed:
- `A or B`: If A is true, B is not evaluated
- `A and B`: If A is false, B is not evaluated

**Using Control Flow:**

For `a < b or c < d`:

```
    if a < b goto L_true
    goto L1
L1: if c < d goto L_true
    goto L_false
L_true: ...
L_false: ...
```

### Backpatching

**Problem**: When generating jumps, we don't always know the target yet.

**Solution**: Leave target blank, fill in later (backpatch).

**Attributes:**
- `E.truelist`: List of jumps to patch with true destination
- `E.falselist`: List of jumps to patch with false destination

**Functions:**
```
makelist(i)      - Create new list with instruction i
merge(l1, l2)    - Concatenate two lists
backpatch(l, j)  - Fill in all instructions in list l with target j
```

**Translation with Backpatching:**

```
E → E₁ or M E₂
    { backpatch(E₁.falselist, M.instr);
      E.truelist = merge(E₁.truelist, E₂.truelist);
      E.falselist = E₂.falselist; }

E → E₁ and M E₂
    { backpatch(E₁.truelist, M.instr);
      E.falselist = merge(E₁.falselist, E₂.falselist);
      E.truelist = E₂.truelist; }

E → not E₁
    { E.truelist = E₁.falselist;
      E.falselist = E₁.truelist; }

E → ( E₁ )
    { E.truelist = E₁.truelist;
      E.falselist = E₁.falselist; }

E → id₁ relop id₂
    { E.truelist = makelist(nextinstr);
      E.falselist = makelist(nextinstr + 1);
      emit('if', id₁.place, relop, id₂.place, 'goto', _);
      emit('goto', _); }

M → ε
    { M.instr = nextinstr; }
```

---

## 7.7 Translation of Control Structures

### If-Then Statement

```
S → if E then S₁

Code structure:
    [E code - jumps to S₁ if true, to after if false]
    [S₁ code]
```

**Translation:**

```
S → if E then M S₁
    { backpatch(E.truelist, M.instr);
      S.nextlist = merge(E.falselist, S₁.nextlist); }

M → ε
    { M.instr = nextinstr; }
```

### If-Then-Else Statement

```
S → if E then S₁ else S₂

Code structure:
    [E code]
    [S₁ code]
    goto S.next
    [S₂ code]
```

**Translation:**

```
S → if E then M₁ S₁ N else M₂ S₂
    { backpatch(E.truelist, M₁.instr);
      backpatch(E.falselist, M₂.instr);
      temp = merge(S₁.nextlist, N.nextlist);
      S.nextlist = merge(temp, S₂.nextlist); }

M → ε
    { M.instr = nextinstr; }

N → ε
    { N.nextlist = makelist(nextinstr);
      emit('goto', _); }
```

### While Statement

```
S → while E do S₁

Code structure:
begin:  [E code]
        [S₁ code]
        goto begin
```

**Translation:**

```
S → while M₁ E do M₂ S₁
    { backpatch(S₁.nextlist, M₁.instr);
      backpatch(E.truelist, M₂.instr);
      S.nextlist = E.falselist;
      emit('goto', M₁.instr); }
```

### Example: While Loop

For `while (a < b) { a = a + 1; }`:

```
L1: if a < b goto L2
    goto L3
L2: t1 = a + 1
    a = t1
    goto L1
L3: ...
```

### For Statement

```
S → for (S₁; E; S₂) S₃

Equivalent to:
    S₁;
    while (E) {
        S₃;
        S₂;
    }
```

**Generated Code:**

```
    [S₁ code]
L1: [E code - if true goto L2, if false goto L3]
L2: [S₃ code]
    [S₂ code]
    goto L1
L3: ...
```

---

## 7.8 Translation of Switch-Case

```
switch (E) {
    case V₁: S₁; break;
    case V₂: S₂; break;
    ...
    default: Sₙ;
}
```

**Method 1: Sequential Testing**

```
    t = [E code]
    if t != V₁ goto L1
    [S₁ code]
    goto end
L1: if t != V₂ goto L2
    [S₂ code]
    goto end
L2: ...
Ln: [Sₙ code]  (default)
end:
```

**Method 2: Jump Table** (when values are dense)

```
    t = [E code]
    if t < min goto default
    if t > max goto default
    goto table[t - min]
    
table:
    [0]: L1
    [1]: L2
    ...

L1: [S₁ code]
    goto end
L2: [S₂ code]
    goto end
...
```

---

## 7.9 Translation of Arrays

### One-Dimensional Arrays

For array `A[i]` with base address `base` and element size `w`:

**Address Calculation:**
```
address(A[i]) = base + i * w
```

If lower bound is `low`:
```
address(A[i]) = base + (i - low) * w
              = (base - low * w) + i * w
              = c + i * w    where c = base - low * w
```

**TAC for A[i]:**
```
t1 = i * w
t2 = A[t1]    // or t2 = A + t1, then t3 = *t2
```

### Two-Dimensional Arrays

For array `A[i][j]` with dimensions [low₁..high₁, low₂..high₂]:

**Row-major order:**
```
address(A[i][j]) = base + ((i - low₁) * n₂ + (j - low₂)) * w

where n₂ = high₂ - low₂ + 1 (number of columns)
```

**Column-major order:**
```
address(A[i][j]) = base + ((j - low₂) * n₁ + (i - low₁)) * w

where n₁ = high₁ - low₁ + 1 (number of rows)
```

### Example: Array Assignment

For `x = A[i][j]` with row-major, n₂ = 10, w = 4:

```
t1 = i * 10    // i * n₂
t2 = t1 + j    // (i * n₂) + j
t3 = t2 * 4    // multiply by element size
t4 = A[t3]     // indexed access
x = t4
```

---

## 7.10 Translation of Function Calls

### Parameter Passing

**Grammar:**
```
S → call id ( Elist )
Elist → Elist , E | E
```

**Translation (using stack for parameters):**

```
Elist → Elist , E
    { append E.place to queue; }

Elist → E
    { initialize queue with E.place; }

S → call id ( Elist )
    { for each item p in queue:
          emit('param', p);
      emit('call', id.name, n); }  // n = number of params
```

### Function Call Example

For `result = func(a, b+c, d)`:

```
t1 = b + c
param a
param t1
param d
t2 = call func, 3
result = t2
```

### Function Definition

```
func:
    [function prologue - save registers, allocate locals]
    [body code]
    [function epilogue - restore, return]
```

---

## 7.11 Static Single Assignment (SSA) Form

### What is SSA?

In **SSA form**, each variable is assigned exactly once, and every use refers to exactly one definition.

**Original Code:**
```
x = 1
x = 2
y = x
```

**SSA Form:**
```
x₁ = 1
x₂ = 2
y₁ = x₂
```

### φ-Functions

When control flow merges, we need **φ-functions** to choose values:

**Original:**
```
if (cond)
    x = 1
else
    x = 2
y = x
```

**SSA Form:**
```
if (cond)
    x₁ = 1
else
    x₂ = 2
x₃ = φ(x₁, x₂)
y₁ = x₃
```

The φ-function selects x₁ if coming from then-branch, x₂ if from else-branch.

### Benefits of SSA

1. **Simpler data flow analysis**: Each definition reaches exactly one use
2. **Easier optimization**: Dead code elimination, constant propagation
3. **Better register allocation**: Interference graphs are simpler

---

## 7.12 Summary

### Key Concepts:

1. **Intermediate Representation** bridges source and target
   - Machine independent
   - Enables optimization
   - Supports multiple front-ends and back-ends

2. **Three-Address Code** uses at most 3 operands per instruction
   - Quadruples: (op, arg1, arg2, result)
   - Triples: (op, arg1, arg2) - result is instruction number
   - Indirect Triples: Pointers to triples

3. **Expression Translation** uses temporaries for intermediate values

4. **Boolean Expressions** can use:
   - Numerical representation (0/1)
   - Short-circuit with control flow

5. **Backpatching** handles forward jumps with unknown targets

6. **Control Structures** translated with proper jump management

7. **Array Access** requires address calculation based on layout

8. **SSA Form** assigns each variable exactly once

---

## 🔍 Practice Questions

1. Generate three-address code for:
   ```
   a = b * c + d / e - f
   ```

2. Show quadruple, triple, and indirect triple representations for:
   ```
   x = (a + b) * (c - d) + (a + b)
   ```

3. Translate with backpatching:
   ```
   while (a < b and c > d) {
       x = x + 1;
   }
   ```

4. Generate code for 2D array access `A[i][j]` with:
   - A is 10×20 array
   - Element size is 4 bytes
   - Row-major order
   - Lower bounds are 0

5. Explain SSA form and its advantages for optimization.

---

*Next Chapter: [Code Optimization](08-Code-Optimization.md)*
