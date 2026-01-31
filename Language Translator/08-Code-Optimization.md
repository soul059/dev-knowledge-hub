# Chapter 8: Code Optimization

## 8.1 Introduction to Code Optimization

**Code Optimization** transforms the program to improve its efficiency while preserving its meaning (semantics).

Problem: Correct code can still be slow, large, or power-hungry.

Solution: Optimization transforms code to improve performance without changing observable behavior.

### Goals of Optimization

1. **Reduce execution time** (speed)
2. **Reduce code size** (memory)
3. **Reduce power consumption** (embedded/mobile)

### The Golden Rule

**Never change the program's observable behavior!**

The optimized program must produce the same output as the original for all inputs.

### Optimization in Compiler Pipeline

```
┌────────────┐     ┌─────────────────┐     ┌────────────┐
│ Intermediate│────▶│   Optimizer     │────▶│    Code    │
│    Code    │     │                 │     │ Generator  │
└────────────┘     │ ┌─────────────┐ │     └────────────┘
                   │ │Machine Indep│ │
                   │ │ Optimization│ │
                   │ └─────────────┘ │
                   │ ┌─────────────┐ │
                   │ │Machine Dep  │ │
                   │ │ Optimization│ │
                   │ └─────────────┘ │
                   └─────────────────┘
```

### Classification of Optimizations

| Type | When Applied | Depends on Machine? |
|------|--------------|---------------------|
| **Machine-Independent** | On IR | No |
| **Machine-Dependent** | On target code | Yes |

| Scope | Description |
|-------|-------------|
| **Local** | Within a basic block |
| **Global** | Across basic blocks (within function) |
| **Inter-procedural** | Across functions |

---

## 8.2 Basic Blocks and Flow Graphs

### Basic Block

A **basic block** is a maximal sequence of consecutive instructions such that:
1. Control enters only at the first instruction
2. Control leaves only at the last instruction
3. No jumps into or out of the middle

Problem: Many optimizations need a clear boundary where control flow does not jump in or out.

Solution: Basic blocks provide safe regions for local analysis and optimization.

### Identifying Basic Blocks

**Algorithm:**
1. Find **leaders** (first instruction of each block):
   - First instruction of the program
   - Target of any jump
   - Instruction immediately after a jump

2. A basic block is a leader plus all following instructions until the next leader.

### Example

```
TAC:
1.  t1 = a + b
2.  t2 = t1 * c
3.  if t2 < d goto 7
4.  t3 = e + f
5.  t4 = t3 - t2
6.  goto 9
7.  t5 = e * f
8.  t4 = t5 + t2
9.  x = t4
```

**Leaders**: 1 (first), 4 (after jump), 7 (target), 9 (target)

**Basic Blocks:**
```
B1: [1, 2, 3]     t1 = a + b
                  t2 = t1 * c
                  if t2 < d goto B3

B2: [4, 5, 6]     t3 = e + f
                  t4 = t3 - t2
                  goto B4

B3: [7, 8]        t5 = e * f
                  t4 = t5 + t2

B4: [9]           x = t4
```

### Control Flow Graph (CFG)

A **CFG** shows the flow of control between basic blocks.

```
        ┌────┐
        │ B1 │
        └─┬──┘
          │
     ┌────┴────┐
     ▼         ▼
   ┌────┐    ┌────┐
   │ B2 │    │ B3 │
   └──┬─┘    └─┬──┘
      │        │
      └───┬────┘
          ▼
        ┌────┐
        │ B4 │
        └────┘
```

**Edges** represent possible control flow between blocks.

Problem: Optimizations across blocks need knowledge of all possible control paths.

Solution: CFG models control flow so analyses can propagate information correctly.

---

## 8.3 Local Optimization (Within Basic Blocks)

### 8.3.1 Common Subexpression Elimination (CSE)

If an expression is computed more than once and operands haven't changed, reuse the first result.

Problem: Redundant computations waste time.

Solution: Reuse previously computed values within a block.

**Before:**
```
t1 = a + b
t2 = c * d
t3 = a + b    ← Same as t1
t4 = t2 + t3
```

**After:**
```
t1 = a + b
t2 = c * d
t3 = t1       ← Reuse t1
t4 = t2 + t3
```

### 8.3.2 Copy Propagation

After `x = y`, replace subsequent uses of `x` with `y` (until `x` or `y` is redefined).

Problem: Temporary copies add unnecessary indirections.

Solution: Replace copies with original values when safe.

**Before:**
```
t1 = a + b
t2 = t1
t3 = t2 * c
t4 = t2 + d
```

**After:**
```
t1 = a + b
t2 = t1
t3 = t1 * c   ← Use t1 instead of t2
t4 = t1 + d   ← Use t1 instead of t2
```

### 8.3.3 Dead Code Elimination

Remove code whose result is never used.

Problem: Some computations have no effect on program output.

Solution: Remove them to reduce time and size.

**Before:**
```
t1 = a + b
t2 = c * d    ← t2 never used
t3 = t1 + e
x = t3
```

**After:**
```
t1 = a + b
t3 = t1 + e
x = t3
```

### 8.3.4 Constant Folding

Evaluate constant expressions at compile time.

Problem: Runtime evaluation of constant expressions is wasteful.

Solution: Precompute constants during compilation.

**Before:**
```
t1 = 3 + 5
t2 = t1 * 2
x = t2
```

**After:**
```
t1 = 8
t2 = 16
x = 16
```

### 8.3.5 Constant Propagation

Replace variables with their constant values when known.

Problem: Values known at compile time are still treated as variables.

Solution: Replace them with constants to enable further simplification.

**Before:**
```
x = 5
y = x + 3
z = y * 2
```

**After:**
```
x = 5
y = 8       ← 5 + 3
z = 16      ← 8 * 2
```

### 8.3.6 Algebraic Simplification

Use mathematical identities to simplify.

Problem: Expressions often contain neutral elements or redundant operations.

Solution: Apply algebraic identities to simplify expressions.

| Original | Simplified |
|----------|------------|
| x + 0 | x |
| x * 1 | x |
| x * 0 | 0 |
| x / 1 | x |
| x - 0 | x |
| x - x | 0 |
| x ** 2 | x * x |

### 8.3.7 Strength Reduction

Replace expensive operations with cheaper ones.

Problem: Some operations are more expensive than equivalent alternatives.

Solution: Replace with cheaper operations like shifts or additions.

| Original | Reduced |
|----------|---------|
| x * 2 | x + x or x << 1 |
| x * 4 | x << 2 |
| x / 2 | x >> 1 |
| x * 15 | (x << 4) - x |
| x ** 2 | x * x |

---

## 8.4 DAG Representation for Optimization

A **DAG** (Directed Acyclic Graph) represents expressions within a basic block, sharing common subexpressions.

Problem: Local redundancies are hard to detect from linear code alone.

Solution: DAG makes shared subexpressions explicit and enables reuse.

### Building a DAG

For each instruction `x = y op z`:
1. Find or create node for y
2. Find or create node for z
3. Check if node for `op` with children y, z exists
   - If yes, attach x as label to existing node
   - If no, create new node for op, attach x

### Example

**Code:**
```
t1 = a + b
t2 = a + b
t3 = t1 * t2
t4 = t3 + a
```

**DAG:**
```
           +  [t4]
          / \
    *  [t3]  a
       / \
      /   \
     + [t1, t2]
    / \
   a   b
```

**Observation**: `a + b` computed only once (t1 and t2 share node).

### Code Generation from DAG

Generate code visiting nodes in topological order:
```
t1 = a + b
t3 = t1 * t1    // t2 = t1, so use t1
t4 = t3 + a
```

---

## 8.5 Loop Optimization

Loops execute many times, so optimizing them has high impact.

Problem: Small inefficiencies inside loops multiply into large costs.

Solution: Move invariant code out, reduce expensive operations, and improve iteration efficiency.

### 8.5.1 Identifying Loops

**Dominators**: Node d **dominates** node n if every path from entry to n goes through d.

**Back Edge**: An edge from n to d where d dominates n.

**Natural Loop**: For a back edge n → d, the loop consists of d plus all nodes that can reach n without going through d.

### 8.5.2 Code Motion (Loop Invariant Code Motion)

Move computations that don't change within the loop to outside (before) the loop.

**Before:**
```c
for (i = 0; i < n; i++) {
    x = y + z;      // y and z don't change in loop
    a[i] = x + i;
}
```

**After:**
```c
x = y + z;          // Moved outside
for (i = 0; i < n; i++) {
    a[i] = x + i;
}
```

**Conditions for moving `x = y op z`:**
- Statement dominates all loop exits
- x is not defined elsewhere in loop
- y and z are loop invariant
- No use of x in loop before this definition

### 8.5.3 Induction Variable Elimination

An **induction variable** changes by a constant amount each iteration.

**Primary induction variable**: Loop counter (i)
**Secondary induction variable**: Linear function of loop counter (j = 4*i + 2)

**Before:**
```c
for (i = 0; i < n; i++) {
    j = i * 4;
    a[j] = 0;
}
```

**After (strength reduction):**
```c
j = 0;
for (i = 0; i < n; i++) {
    a[j] = 0;
    j = j + 4;      // Replace multiplication with addition
}
```

**After (eliminating i if only used as loop counter):**
```c
j = 0;
while (j < n * 4) {
    a[j] = 0;
    j = j + 4;
}
```

### 8.5.4 Loop Unrolling

Reduce loop overhead by replicating the body.

**Before:**
```c
for (i = 0; i < 100; i++) {
    a[i] = b[i] + c[i];
}
```

**After (unroll by 4):**
```c
for (i = 0; i < 100; i += 4) {
    a[i] = b[i] + c[i];
    a[i+1] = b[i+1] + c[i+1];
    a[i+2] = b[i+2] + c[i+2];
    a[i+3] = b[i+3] + c[i+3];
}
```

**Benefits:**
- Fewer loop overhead instructions
- More opportunities for instruction scheduling
- Better cache utilization

### 8.5.5 Loop Fusion

Combine adjacent loops with same bounds.

**Before:**
```c
for (i = 0; i < n; i++)
    a[i] = b[i] + 1;

for (i = 0; i < n; i++)
    c[i] = a[i] * 2;
```

**After:**
```c
for (i = 0; i < n; i++) {
    a[i] = b[i] + 1;
    c[i] = a[i] * 2;
}
```

---

## 8.6 Global Optimization (Data Flow Analysis)

### 8.6.1 Data Flow Analysis Framework

Data flow analysis propagates information about the program along control flow edges.

Problem: Optimizations across blocks need global knowledge about definitions and uses.

Solution: Data flow analysis computes facts that enable safe global transformations.

**Components:**
- **Domain**: Set of possible values at each point
- **Transfer function**: How each statement affects values
- **Meet operator**: How to combine values at merge points
- **Direction**: Forward or backward

### 8.6.2 Reaching Definitions

A definition **reaches** a point if there's a path from the definition to that point without the variable being redefined.

**Use**: Common subexpression elimination, constant propagation.

**Equations:**
```
out[B] = gen[B] ∪ (in[B] - kill[B])
in[B] = ∪ out[P] for all predecessors P of B
```

Where:
- gen[B] = definitions generated in block B
- kill[B] = definitions killed (overwritten) by block B

### Example: Reaching Definitions

```
B1: d1: x = 5
    d2: y = 3

B2: d3: x = y + 2
    d4: z = x

B3: d5: y = x * 2
```

```
gen[B1] = {d1, d2}    kill[B1] = {d3, d5}
gen[B2] = {d3, d4}    kill[B2] = {d1}
gen[B3] = {d5}        kill[B3] = {d2}

Iterative solution:
Initially: in[B] = out[B] = ∅

Iteration 1:
in[B1] = ∅
out[B1] = {d1, d2}
in[B2] = out[B1] = {d1, d2}
out[B2] = {d3, d4} ∪ ({d1, d2} - {d1}) = {d2, d3, d4}
...
```

### 8.6.3 Live Variable Analysis

A variable is **live** at a point if its value may be used before being redefined.

**Use**: Dead code elimination, register allocation.

**Direction**: Backward (from uses to definitions)

**Equations:**
```
in[B] = use[B] ∪ (out[B] - def[B])
out[B] = ∪ in[S] for all successors S of B
```

Where:
- use[B] = variables used in B before any definition
- def[B] = variables defined in B

### 8.6.4 Available Expressions

An expression is **available** at a point if it has been computed on all paths to that point and not invalidated.

**Use**: Common subexpression elimination.

**Direction**: Forward

**Equations:**
```
out[B] = gen[B] ∪ (in[B] - kill[B])
in[B] = ∩ out[P] for all predecessors P of B
```

Note: Uses intersection (∩) because expression must be available on ALL paths.

### 8.6.5 Very Busy Expressions

An expression is **very busy** at a point if it will be evaluated on all paths from that point before any operand is modified.

**Use**: Code hoisting (move computation earlier).

**Direction**: Backward

---

## 8.7 Peephole Optimization

**Peephole optimization** examines a small window (peephole) of instructions and applies local transformations.

Problem: Target code often contains small inefficiencies that are hard to detect globally.

Solution: Peephole optimization replaces local instruction patterns with better ones.

### Common Peephole Optimizations

#### 1. Redundant Load/Store Elimination

```
Before:                 After:
MOV R1, a               MOV R1, a
MOV a, R1   ← remove    
```

#### 2. Algebraic Simplification

```
Before:                 After:
ADD R1, 0               (remove)
MUL R1, 1               (remove)
MUL R1, 2               SHL R1, 1
```

#### 3. Unreachable Code

```
Before:                 After:
    goto L2                 goto L2
L1: MOV R1, a   ← remove
    ADD R1, b   ← remove
L2: ...                 L2: ...
```

#### 4. Jump Optimization

**Jump to jump:**
```
Before:                 After:
    goto L1                 goto L2
    ...
L1: goto L2
```

**Jump over jump:**
```
Before:                     After:
    if a < b goto L1            if a >= b goto L2
    goto L2
L1: ...                     L1: ...
```

**Jumps to return:**
```
Before:                 After:
    goto L1                 return
    ...
L1: return
```

---

## 8.8 Register Allocation

Efficiently assigning program variables to limited CPU registers.

Problem: Registers are limited and memory is slow.

Solution: Allocate registers to the most useful variables and spill others when needed.

### 8.8.1 The Problem

- Programs use many variables
- CPUs have few registers (8-32 typically)
- Memory access is slow compared to register access

### 8.8.2 Graph Coloring Approach

1. **Build interference graph**:
   - Node for each variable
   - Edge between variables that are live at the same time

2. **Color the graph**:
   - Each color represents a register
   - Connected nodes must have different colors
   - k colors for k registers

3. **Spill if needed**:
   - If can't color with k colors, spill some variables to memory

### Example

```
Variables: a, b, c, d, e
Live ranges:
- a: [1-5]
- b: [2-4]
- c: [3-6]
- d: [5-7]
- e: [1-3]
```

**Interference Graph:**
```
    a ─── b
    │╲   ╱│
    │ ╲ ╱ │
    │  ╳  │
    │ ╱ ╲ │
    e ─── c
          │
          d
```

**Coloring with 3 colors (registers):**
- a: R1
- b: R2
- c: R3
- d: R1 (doesn't interfere with a)
- e: R3 (doesn't interfere with c)

---

## 8.9 Instruction Scheduling

Reorder instructions to reduce pipeline stalls and improve parallelism.

Problem: Dependencies can cause pipeline stalls in modern processors.

Solution: Reorder independent instructions to keep the pipeline busy.

### The Problem

Modern processors have pipelines where different stages of instruction execution overlap. Dependencies can cause stalls.

**Example:**
```
Original:               Problem:
LOAD R1, a              (LOAD takes 3 cycles)
ADD R2, R1, 1           (must wait for R1)
LOAD R3, b
ADD R4, R3, 2
```

**After scheduling:**
```
LOAD R1, a              (start loading a)
LOAD R3, b              (start loading b while waiting for a)
ADD R2, R1, 1           (a is ready now)
ADD R4, R3, 2           (b is ready now)
```

### List Scheduling Algorithm

1. Build dependency graph
2. Compute priorities (longest path to any root)
3. Schedule greedily by priority, respecting dependencies

---

## 8.10 Machine-Dependent Optimizations

### 8.10.1 Instruction Selection

Choose the best instruction for an operation based on target machine.

Problem: Different architectures offer different instruction costs and capabilities.

Solution: Select target-specific instructions that minimize cost.

**Example (x86):**
```
For: x = x + 1

Options:
    ADD x, 1        ; memory operand
    MOV R1, x       ; load, add, store
    ADD R1, 1
    MOV x, R1
    
    INC x           ; special instruction (best for x86)
```

### 8.10.2 Utilizing Special Instructions

- SIMD (Single Instruction, Multiple Data)
- Hardware multiply-accumulate
- Bit manipulation instructions
- Cache prefetch instructions

### 8.10.3 Memory Hierarchy Optimization

- **Loop tiling**: Process data in cache-sized chunks
- **Data layout**: Arrange data for cache efficiency
- **Prefetching**: Load data before it's needed

---

## 8.11 Optimization Summary by Scope

### Local (Basic Block)

| Optimization | Technique |
|--------------|-----------|
| Common Subexpression Elimination | DAG construction |
| Copy Propagation | Value numbering |
| Dead Code Elimination | Use-def analysis |
| Constant Folding/Propagation | Symbolic evaluation |
| Strength Reduction | Pattern matching |

### Global (Whole Function)

| Optimization | Analysis Needed |
|--------------|-----------------|
| Global CSE | Available expressions |
| Dead Code Elimination | Live variables |
| Constant Propagation | Reaching definitions |
| Code Motion | Dominators, loop detection |

### Loop

| Optimization | Benefit |
|--------------|---------|
| Loop Invariant Code Motion | Reduce redundant computation |
| Induction Variable Elimination | Simplify expressions |
| Loop Unrolling | Reduce overhead |
| Loop Fusion/Fission | Improve cache behavior |

---

## 8.12 Summary

### Key Concepts:

1. **Basic Blocks** are maximal straight-line code sequences
   - Control enters at beginning, exits at end

2. **Control Flow Graph** shows relationships between basic blocks

3. **Local Optimizations** work within basic blocks:
   - CSE, copy propagation, dead code elimination
   - Constant folding, algebraic simplification

4. **Data Flow Analysis** propagates information through CFG:
   - Reaching definitions, live variables, available expressions
   - Enables global optimizations

5. **Loop Optimizations** have high impact:
   - Code motion, induction variable elimination
   - Loop unrolling, fusion

6. **Register Allocation** uses graph coloring

7. **Peephole Optimization** examines small windows of code

Simple takeaway: Optimization removes redundant work and uses machine resources efficiently without changing program behavior.

---

## 🔍 Practice Questions

1. Identify basic blocks and draw CFG for:
   ```
   read x
   if x > 0 then
       y = x * 2
   else
       y = -x
   write y
   ```

2. Apply local optimizations to:
   ```
   t1 = a + b
   t2 = a + b
   t3 = t1 * t2
   t4 = a + b
   t5 = t4 + t3
   x = t5
   ```

3. Perform reaching definitions analysis for a given CFG.

4. Identify loop invariant code and move it out of:
   ```
   for i = 1 to n:
       t = a + b
       c[i] = t * i
   ```

5. Apply strength reduction to eliminate multiplication from:
   ```
   for i = 0 to n-1:
       a[i * 4] = 0
   ```

---

*Next Chapter: [Code Generation](09-Code-Generation.md)*
