# Chapter 9: Code Generation

## 9.1 Introduction to Code Generation

**Code Generation** is the final phase of compilation that transforms the optimized intermediate representation into target machine code.

Problem: The compiler must turn machine-independent IR into efficient machine-specific instructions.

Solution: Code generation maps IR to target code while respecting registers, memory, and instruction sets.

### Position in Compiler

```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│    Optimized   │────▶│     Code       │────▶│    Target      │
│       IR       │     │   Generator    │     │   Program      │
└────────────────┘     └────────────────┘     └────────────────┘
    (Three-Address        (Converts to            (Assembly or
        Code)              machine code)            Object code)
```

### Requirements for Code Generator

1. **Correctness**: Output must preserve program semantics
2. **Efficiency**: Generated code should be fast
3. **Resource Utilization**: Effective use of registers and memory
4. **Code Size**: Reasonably compact code

Problem: There are many valid machine codes for the same IR, but quality varies.

Solution: Apply instruction selection, register allocation, and good evaluation order to produce efficient code.

---

## 9.2 Issues in Code Generation

### 9.2.1 Input to Code Generator

- **Three-Address Code**: Quadruples or triples
- **Assumptions**: Type checking done, variables have storage allocated

### 9.2.2 Target Programs

| Type | Description |
|------|-------------|
| **Absolute Machine Code** | Fixed memory locations, directly executable |
| **Relocatable Machine Code** | Can be linked with other code |
| **Assembly Language** | Symbolic, needs assembler |

### 9.2.3 Memory Management

Variables and data need memory locations:
- **Global variables**: Fixed addresses
- **Local variables**: Stack-based offsets
- **Dynamic data**: Heap allocation

### 9.2.4 Instruction Selection

Choosing the right instructions affects efficiency:

Problem: Different instructions have different costs and capabilities.

Solution: Choose the most efficient instructions available on the target machine.

```
For: a = b + c

Option 1 (RISC-like):
    LOAD  R1, b
    LOAD  R2, c
    ADD   R1, R1, R2
    STORE R1, a

Option 2 (CISC with memory operands):
    MOV   a, b
    ADD   a, c
```

### 9.2.5 Register Allocation

Decisions about which values to keep in registers:
- **More registers used** → Faster code
- **Limited registers** → Need to spill to memory

Problem: Registers are limited, but accessing memory is expensive.

Solution: Keep frequently used values in registers and spill others carefully.

### 9.2.6 Evaluation Order

The order of evaluating subexpressions affects register usage:

Problem: A poor evaluation order can cause extra loads and spills.

Solution: Choose an order that minimizes register pressure.

```
For: (a + b) * (c + d)

Order 1:                    Order 2:
    t1 = a + b (needs R1)       t1 = c + d (needs R1)
    t2 = c + d (needs R2)       t2 = a + b (needs R2)
    t3 = t1 * t2 (R1 or R2)     t3 = t2 * t1 (R1 or R2)
```

Both use same registers, but other expressions might differ.

---

## 9.3 Target Machine Model

### Simplified Machine Model

For educational purposes, we use a simple machine:

Problem: Code generation depends on the target architecture.

Solution: Use a simplified model to explain key ideas like registers and addressing modes.

**Registers**: R0, R1, R2, ... (general purpose)

**Instructions**:

| Instruction | Meaning |
|-------------|---------|
| LOAD Ri, M | Ri ← contents of memory M |
| STORE Ri, M | Memory M ← Ri |
| MOV Ri, Rj | Ri ← Rj |
| ADD Ri, Rj, Rk | Ri ← Rj + Rk |
| SUB Ri, Rj, Rk | Ri ← Rj - Rk |
| MUL Ri, Rj, Rk | Ri ← Rj × Rk |
| DIV Ri, Rj, Rk | Ri ← Rj ÷ Rk |
| JUMP L | Unconditional jump to L |
| JUMPT Ri, L | Jump to L if Ri is true (non-zero) |
| JUMPF Ri, L | Jump to L if Ri is false (zero) |
| CMP Ri, Rj, Rk | Ri ← 1 if Rj < Rk, else 0 |
| CALL func | Call function |
| RET | Return from function |

### Addressing Modes

| Mode | Syntax | Meaning |
|------|--------|---------|
| Register | R1 | Value in R1 |
| Immediate | #5 | Constant 5 |
| Direct | M | Value at memory address M |
| Indirect | *R1 | Value at address in R1 |
| Indexed | a(R1) | Value at address a + R1 |

---

## 9.4 A Simple Code Generator

### Basic Approach

For each three-address instruction, generate target code using a template.

Problem: We need a consistent way to map IR instructions to machine instructions.

Solution: Use templates and descriptors to track where values are stored.

### Register and Address Descriptors

**Register Descriptor**: For each register, what variables are currently in it.

```
R1: {a, b}    // R1 contains both a and b (they have same value)
R2: {t1}      // R2 contains temporary t1
R3: empty
```

**Address Descriptor**: For each variable, where its value can be found.

Problem: The generator must know where every value currently resides.

Solution: Descriptors track locations in registers and memory to avoid unnecessary loads/stores.

```
a: {R1, memory}  // a is in R1 and also in memory
b: {R1}          // b is in R1 only
t1: {R2}         // t1 is only in R2
```

### Function: getReg(x op y)

Determines where to put the result of `x op y`:

1. If y is in a register that holds only y, and y is not live after this instruction, use that register.
2. If there's an empty register, use it.
3. If x is in a register that holds only x, and x is not live and not used later in the block, use it.
4. Otherwise, spill a register (save to memory) and use it.

### Code Generation Algorithm

```
For each three-address instruction I in the block:
    
    Case x = y op z:
        L = getReg(y op z)
        
        if y is not in L:
            generate: LOAD L, y'    (y' is location of y)
        
        generate: OP L, L, z'       (z' is location of z)
        
        update descriptors:
            - x is now in L only
            - if y was in L, y is no longer there
            - L contains only x

    Case x = y:
        if y is in register R:
            add x to R's descriptor
        else:
            L = getReg(y)
            generate: LOAD L, y
        
        update address descriptor for x

    Case x = op y (unary):
        Similar to binary, but one operand
```

### Example: Generating Code for Basic Block

**Three-Address Code:**
```
t1 = a - b
t2 = a * c
t3 = t1 + t2
d = t3 + t2
```

**Assume**: a, b, c are in memory; d needs to be stored.

**Code Generation:**

| Statement | Generated Code | Register Descriptor | Address Descriptor |
|-----------|----------------|--------------------|--------------------|
| | | R1: empty, R2: empty | a,b,c,d: memory |
| t1 = a - b | LOAD R1, a<br>LOAD R2, b<br>SUB R1, R1, R2 | R1: t1<br>R2: b | t1: R1 |
| t2 = a * c | LOAD R2, a<br>LOAD R3, c<br>MUL R2, R2, R3 | R1: t1<br>R2: t2<br>R3: c | t1: R1<br>t2: R2 |
| t3 = t1 + t2 | ADD R1, R1, R2 | R1: t3<br>R2: t2 | t3: R1<br>t2: R2 |
| d = t3 + t2 | ADD R1, R1, R2<br>STORE R1, d | R1: d<br>R2: t2 | d: R1, memory |

**Final Code:**
```asm
LOAD  R1, a
LOAD  R2, b
SUB   R1, R1, R2    ; t1 = a - b
LOAD  R2, a
LOAD  R3, c
MUL   R2, R2, R3    ; t2 = a * c
ADD   R1, R1, R2    ; t3 = t1 + t2
ADD   R1, R1, R2    ; d = t3 + t2
STORE R1, d
```

---

## 9.5 Register Allocation and Assignment

### The Problem

- Many variables, few registers
- Memory operations are expensive
- Goal: Keep frequently used values in registers

Problem: Many variables compete for few registers.

Solution: Use heuristics locally and graph coloring globally to assign registers.

### Local Register Allocation

Within a single basic block:

**Usage Count Heuristic:**
1. Count how many times each variable is used in the block
2. Allocate registers to variables with highest counts

**Example:**
```
Block uses: a (5 times), b (2 times), c (3 times), d (1 time)
Available registers: R1, R2

Allocation:
    R1 ← a (most used)
    R2 ← c (second most)
    b, d in memory
```

### Global Register Allocation

Across basic blocks using graph coloring (covered in optimization chapter).

### Spilling

When we need a register but all are occupied:

1. Choose a register to spill (typically least recently used)
2. If it contains a live value not in memory, store it
3. Update descriptors
4. Use the register

```
Before spilling R2:
    R2 contains x
    
If x is live:
    STORE R2, x_memory_location

Now R2 is available
```

---

## 9.6 Code Generation for Control Structures

### 9.6.1 If-Then-Else

**Three-Address Code:**
```
    if a < b goto L1
    goto L2
L1: [then-part]
    goto L3
L2: [else-part]
L3: ...
```

**Generated Code:**
```asm
    LOAD  R1, a
    LOAD  R2, b
    CMP   R3, R1, R2      ; R3 = 1 if a < b
    JUMPT R3, L1
    JUMP  L2
L1: ; then-part code
    JUMP  L3
L2: ; else-part code
L3: ; continue

Problem: Control flow requires correct branching and label management.

Solution: Translate control structures into conditional and unconditional jumps.
```

**Optimization** - Combine comparison and jump:
```asm
    LOAD  R1, a
    LOAD  R2, b
    BLT   R1, R2, L1      ; Branch if Less Than
    ; else-part code
    JUMP  L3
L1: ; then-part code
L3: ; continue
```

### 9.6.2 While Loop

**Three-Address Code:**
```
L1: if a < b goto L2
    goto L3
L2: [body]
    goto L1
L3: ...
```

**Generated Code:**
```asm
L1: LOAD  R1, a
    LOAD  R2, b
    BGE   R1, R2, L3      ; Branch if Greater or Equal (exit)
    ; body code
    JUMP  L1
L3: ; after loop
```

### 9.6.3 For Loop

```c
for (i = 0; i < n; i++) {
    body;
}
```

**Generated Code:**
```asm
    LOAD  R1, #0          ; i = 0
    STORE R1, i
L1: LOAD  R1, i
    LOAD  R2, n
    BGE   R1, R2, L2      ; if i >= n, exit
    
    ; body code
    
    LOAD  R1, i           ; i++
    ADD   R1, R1, #1
    STORE R1, i
    JUMP  L1
L2: ; after loop
```

---

## 9.7 Code Generation for Function Calls

### Calling Conventions

A **calling convention** defines:
- How arguments are passed (registers, stack)
- Who saves registers (caller or callee)
- How return values are passed
- Stack frame layout

Problem: Callers and callees must agree on how to pass arguments and return values.

Solution: Use a calling convention with a standard stack frame and register rules.

### Typical Stack Frame

```
Higher addresses
    ┌──────────────────────┐
    │   Arguments from     │
    │   caller (if any     │
    │   on stack)          │
    ├──────────────────────┤
    │   Return address     │  ← Pushed by CALL
    ├──────────────────────┤
    │   Old frame pointer  │  ← Saved by callee
    ├──────────────────────┤ ← Frame Pointer (FP)
    │   Local variables    │
    │                      │
    ├──────────────────────┤
    │   Saved registers    │
    │                      │
    ├──────────────────────┤ ← Stack Pointer (SP)
Lower addresses
```

### Function Call Sequence

**Caller (before call):**
```asm
    ; Evaluate and push/store arguments
    LOAD  R1, arg1
    LOAD  R2, arg2
    ; If using stack:
    PUSH  R1
    PUSH  R2
    ; Or if using registers, just load them
    
    CALL  function_name
    
    ; After return:
    ; Result is in designated register (e.g., R0)
    ; Clean up arguments if caller's responsibility
```

**Callee (function entry):**
```asm
function_name:
    ; Prologue
    PUSH  FP              ; Save old frame pointer
    MOV   FP, SP          ; Set new frame pointer
    SUB   SP, SP, #locals ; Allocate local variables
    PUSH  R1              ; Save callee-saved registers
    PUSH  R2
    
    ; Function body
    ; Access arguments via FP+offset
    ; Access locals via FP-offset
    
    ; Epilogue
    POP   R2              ; Restore saved registers
    POP   R1
    MOV   SP, FP          ; Deallocate locals
    POP   FP              ; Restore old frame pointer
    RET                   ; Return to caller
```

### Example: Complete Function

**Source:**
```c
int add(int a, int b) {
    int sum = a + b;
    return sum;
}

int main() {
    int x = add(3, 5);
}
```

**Generated Code:**
```asm
add:
    ; Prologue
    PUSH  FP
    MOV   FP, SP
    SUB   SP, SP, #4      ; space for sum
    
    ; Body: sum = a + b
    LOAD  R1, 8(FP)       ; load a (first arg)
    LOAD  R2, 12(FP)      ; load b (second arg)
    ADD   R1, R1, R2
    STORE R1, -4(FP)      ; store in sum
    
    ; return sum
    LOAD  R0, -4(FP)      ; return value in R0
    
    ; Epilogue
    MOV   SP, FP
    POP   FP
    RET

main:
    PUSH  FP
    MOV   FP, SP
    SUB   SP, SP, #4      ; space for x
    
    ; Call add(3, 5)
    LOAD  R1, #3
    PUSH  R1              ; push first arg
    LOAD  R1, #5
    PUSH  R1              ; push second arg
    CALL  add
    ADD   SP, SP, #8      ; clean up args
    STORE R0, -4(FP)      ; x = result
    
    ; Epilogue
    MOV   SP, FP
    POP   FP
    RET
```

---

## 9.8 Code Generation for Arrays

### One-Dimensional Array Access

**For `a[i]` where a starts at base address and elements have size w:**

Problem: Arrays require address arithmetic before access.

Solution: Compute addresses using base + index * element_size (with shifts when possible).

```
address = base + i * w
```

**Generated Code for `x = a[i]`:**
```asm
    LOAD  R1, i
    MUL   R1, R1, #w      ; or shift if w is power of 2
    ADD   R1, R1, #base_a
    LOAD  R2, 0(R1)       ; load a[i]
    STORE R2, x
```

**Optimized (if w = 4):**
```asm
    LOAD  R1, i
    SHL   R1, R1, #2      ; i * 4
    LOAD  R2, base_a(R1)  ; indexed addressing
    STORE R2, x
```

### Two-Dimensional Array Access

**For `a[i][j]` in row-major order, with dimensions [n][m]:**

```
address = base + (i * m + j) * w
```

**Generated Code:**
```asm
    LOAD  R1, i
    MUL   R1, R1, #m      ; i * m
    LOAD  R2, j
    ADD   R1, R1, R2      ; i * m + j
    MUL   R1, R1, #w      ; (i * m + j) * w
    ADD   R1, R1, #base_a
    LOAD  R2, 0(R1)       ; load a[i][j]
```

---

## 9.9 Code Generation for Expressions

### Arithmetic Expression Trees

**Method**: Generate code by post-order traversal of expression tree.

Problem: Expressions must be evaluated with limited registers.

Solution: Use post-order traversal and algorithms like Sethi-Ullman to reduce register needs.

**For expression `(a + b) * (c - d)`:**

```
        *
       / \
      +   -
     / \ / \
    a  b c  d
```

**Post-order traversal**: a, b, +, c, d, -, *

**Generated Code:**
```asm
    LOAD  R1, a
    LOAD  R2, b
    ADD   R1, R1, R2      ; R1 = a + b
    LOAD  R2, c
    LOAD  R3, d
    SUB   R2, R2, R3      ; R2 = c - d
    MUL   R1, R1, R2      ; R1 = (a+b) * (c-d)
```

### Sethi-Ullman Algorithm

An algorithm to generate code with minimum number of registers.

**Labeling (number of registers needed):**
1. Label leaves: 1 for left child, 0 for right child (or vice versa based on instruction format)
2. Label internal nodes:
   - If children have different labels: max(left_label, right_label)
   - If children have same labels: left_label + 1

**Code Generation:**
- Evaluate the subtree needing more registers first
- This minimizes total registers needed

**Example:**
```
        *  [2]
       / \
    +[2]  -[1]
    / \   / \
 a[1] b[0] c[1] d[0]
```

Labels computed bottom-up. Root needs 2 registers.

---

## 9.10 Instruction Selection by Tree Rewriting

### The Idea

- Represent target instructions as tree patterns
- Match patterns against IR trees
- Cover the IR tree with instruction patterns

Problem: Multiple instruction sequences can implement the same IR tree.

Solution: Use pattern matching and dynamic programming to select minimal-cost instruction sets.

### Example Instruction Patterns

```
Pattern                     Instruction       Cost
───────────────────────────────────────────────────
reg → CONST                 LOAD reg, #c      1
reg → MEM(CONST)            LOAD reg, c       1
reg → MEM(reg)              LOAD reg, (reg)   1
reg → MEM(+(reg, CONST))    LOAD reg, c(reg)  1
reg → +(reg, reg)           ADD reg, reg, reg 1
reg → *(reg, reg)           MUL reg, reg, reg 1
```

### Optimal Instruction Selection

Use **dynamic programming** to find minimum cost covering:

1. For each node in IR tree, compute minimum cost for each possible pattern covering that node
2. Choose patterns that minimize total cost

### Example

**IR Tree for `a[i]`:**
```
      MEM
       |
       +
      / \
     a   *
        / \
       i   4
```

**Possible Coverings:**
```
Covering 1:                 Cost = 4
    LOAD R1, a              1
    LOAD R2, i              1
    MUL  R2, R2, #4         1
    ADD  R1, R1, R2         1
    LOAD R1, (R1)           (covered by previous pattern)

Covering 2:                 Cost = 3
    LOAD R1, i              1
    MUL  R1, R1, #4         1
    LOAD R1, a(R1)          1  (indexed addressing)
```

---

## 9.11 Target Code Quality

### Measuring Code Quality

| Metric | Description |
|--------|-------------|
| **Speed** | Execution time (cycles) |
| **Size** | Code size in bytes |
| **Power** | Energy consumption |

### Trade-offs

- **Speed vs Size**: Loop unrolling increases speed but also size
- **Speed vs Power**: Faster code may use more power
- **Compile Time vs Code Quality**: More optimization takes longer

Problem: Improving one metric can hurt another.

Solution: Balance speed, size, and power based on compiler goals.

### Simple Improvements

1. **Use efficient addressing modes**
2. **Prefer register operations over memory**
3. **Use immediate operands when possible**
4. **Combine operations (MAC instructions)**
5. **Avoid redundant operations**

---

## 9.12 Generating Code for Different Architectures

### RISC (Reduced Instruction Set Computer)

**Characteristics:**
- Fixed-length instructions
- Load/Store architecture
- Many registers
- Simple addressing modes

Problem: RISC and CISC provide different instruction capabilities.

Solution: Tailor instruction selection and addressing modes to the target architecture.

**Code Generation Considerations:**
- Heavy use of registers
- Explicit load/store for memory
- Simple instruction selection

### CISC (Complex Instruction Set Computer)

**Characteristics:**
- Variable-length instructions
- Memory operands in most instructions
- Fewer registers
- Many addressing modes

**Code Generation Considerations:**
- Complex instruction selection
- Memory operands reduce load/stores
- Special instructions for common patterns

### Comparison Example

**For `x = a + b`:**

**RISC (ARM-like):**
```asm
LDR  R1, [a]
LDR  R2, [b]
ADD  R3, R1, R2
STR  R3, [x]
```

**CISC (x86-like):**
```asm
MOV  EAX, [a]
ADD  EAX, [b]
MOV  [x], EAX
```

---

## 9.13 Summary

### Key Concepts:

1. **Code Generation** converts IR to target machine code

2. **Issues** include:
   - Instruction selection
   - Register allocation
   - Evaluation order
   - Addressing modes

3. **Descriptors** track register and variable locations:
   - Register descriptor: What's in each register
   - Address descriptor: Where each variable resides

4. **Register Allocation** assigns variables to limited registers
   - Local: Usage count heuristics
   - Global: Graph coloring

5. **Calling Conventions** define function call mechanics:
   - Argument passing
   - Stack frame layout
   - Register saving

6. **Instruction Selection** can be optimized using:
   - Pattern matching
   - Tree rewriting
   - Dynamic programming

Simple takeaway: Code generation turns IR into efficient machine code by choosing good instructions, registers, and evaluation orders.

---

## 🔍 Practice Questions

1. Generate target code for:
   ```
   t1 = a + b
   t2 = c - d
   t3 = t1 * t2
   e = t3
   ```
   Show register and address descriptors at each step.

2. Generate code for the function call `result = func(a, b+c, d)`.

3. Generate code for the following with 2 registers only:
   ```
   w = (a + b) * (c + d) + (e * f)
   ```

4. Explain the Sethi-Ullman algorithm with an example.

5. Compare code generation for RISC vs CISC for:
   ```
   if (a < b)
       x = y + z;
   else
       x = y - z;
   ```

---

*Next Chapter: [Runtime Environment](10-Runtime-Environment.md)*
