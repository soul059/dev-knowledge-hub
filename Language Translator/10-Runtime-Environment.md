# Chapter 10: Runtime Environment

## 10.1 Introduction

The **Runtime Environment** is the structure of the target computer's memory and registers that serves to manage memory and maintain the information needed to execute a program.

Problem: Programs need memory, calls, and variable access rules during execution.

Solution: The runtime environment defines how memory is organized and how the compiler manages function calls and data access.

### What Happens at Runtime?

When a compiled program runs, it needs:
1. **Memory** for code, data, and stack
2. **Mechanisms** for function calls and returns
3. **Methods** to access variables (local, global, dynamic)
4. **Systems** to allocate and free memory dynamically

### Compiler's Role

The compiler generates code that:
- Implements the runtime environment structure
- Manages activation records (stack frames)
- Handles memory allocation
- Enables proper variable access

---

## 10.2 Memory Layout of a Program

A typical program's memory is divided into several regions:

Problem: Code, global data, stack, and heap have different lifetimes and access patterns.

Solution: Separate memory regions provide predictable allocation and access for each kind of data.

```
┌─────────────────────────────────────┐ High Address
│                                     │
│              STACK                  │  ← Function activation records
│                │                    │    (grows downward)
│                ▼                    │
│                                     │
│           (free space)              │
│                                     │
│                ▲                    │
│                │                    │
│              HEAP                   │  ← Dynamic memory allocation
│                                     │    (grows upward)
├─────────────────────────────────────┤
│           STATIC DATA               │  ← Global and static variables
│         (Data Segment)              │
├─────────────────────────────────────┤
│              CODE                   │  ← Program instructions
│         (Text Segment)              │
└─────────────────────────────────────┘ Low Address
```

### Memory Regions Explained

| Region | Purpose | Allocation Time |
|--------|---------|-----------------|
| **Code** | Program instructions | Compile/Load time |
| **Static Data** | Global variables, constants | Compile/Load time |
| **Stack** | Local variables, function calls | Runtime (auto) |
| **Heap** | Dynamic allocations (malloc) | Runtime (manual) |

---

## 10.3 Storage Allocation Strategies

### 10.3.1 Static Allocation

Memory is allocated at **compile time** and remains fixed throughout execution.

Problem: Some data sizes are known at compile time and never change.

Solution: Allocate them statically to avoid runtime overhead.

**Characteristics:**
- Size must be known at compile time
- Same location for entire program run
- No runtime allocation overhead
- Used for: global variables, static locals, constants

**Example:**
```c
int globalVar = 10;          // Static allocation
static int staticVar = 20;   // Static allocation
const char* message = "Hi";  // Static (string literal)
```

**Layout:**
```
Static Data Segment:
┌──────────────────────────┐
│ globalVar:  [10]         │ Address: 1000
│ staticVar:  [20]         │ Address: 1004
│ message:    [ptr→1100]   │ Address: 1008
│ "Hi\0":     [H][i][\0]   │ Address: 1100
└──────────────────────────┘
```

**Limitations:**
- No recursion (same location used for all calls)
- No dynamic data structures

### 10.3.2 Stack Allocation

Memory is allocated on a **stack** that grows and shrinks with function calls.

Problem: Local variables and call frames need fast allocation and deallocation.

Solution: Use a stack with LIFO behavior to push/pop activation records efficiently.

**Characteristics:**
- LIFO (Last In, First Out) order
- Automatic allocation/deallocation
- Used for: local variables, parameters, return addresses

**How it works:**
1. **Call** a function → **Push** activation record
2. **Return** from function → **Pop** activation record

**Supports:**
- Recursion (each call gets new activation record)
- Nested function calls

### 10.3.3 Heap Allocation

Memory is allocated and freed **dynamically** at runtime.

Problem: Dynamic data structures need memory not known at compile time.

Solution: Use heap allocation with explicit allocation and deallocation (or garbage collection).

**Characteristics:**
- Allocated/freed in any order
- Programmer or garbage collector manages lifetime
- Used for: dynamic arrays, linked lists, objects

**Example:**
```c
int* arr = malloc(n * sizeof(int));  // Heap allocation
// ... use arr ...
free(arr);                           // Heap deallocation
```

**Challenges:**
- Fragmentation (memory holes)
- Memory leaks (forgetting to free)
- Dangling pointers (using freed memory)

---

## 10.4 Activation Records (Stack Frames)

An **Activation Record** (or **Stack Frame**) is a block of memory allocated on the stack for each function invocation.

Problem: Each function call needs space for locals, parameters, and return information.

Solution: Activation records store all call-specific data and enable recursion.

### Contents of Activation Record

```
┌─────────────────────────────────┐ Higher addresses
│        Actual Parameters        │  ← Arguments passed to function
│          (Arguments)            │
├─────────────────────────────────┤
│        Return Address           │  ← Where to return after call
├─────────────────────────────────┤
│     Control Link (Old FP)       │  ← Previous frame pointer
├─────────────────────────────────┤
│        Access Link              │  ← For nested functions (static link)
│      (optional)                 │
├─────────────────────────────────┤ ← Frame Pointer (FP) points here
│       Local Variables           │
│                                 │
├─────────────────────────────────┤
│      Saved Registers            │
│                                 │
├─────────────────────────────────┤
│     Temporary Values            │
│                                 │
└─────────────────────────────────┘ ← Stack Pointer (SP) points here
                                    Lower addresses
```

### Components Explained

| Component | Purpose |
|-----------|---------|
| **Actual Parameters** | Values passed to the function |
| **Return Address** | Address of instruction after the call |
| **Control Link** | Pointer to caller's activation record |
| **Access Link** | Pointer to enclosing function's frame (for nested functions) |
| **Local Variables** | Variables declared inside the function |
| **Saved Registers** | Registers that must be preserved |
| **Temporaries** | Compiler-generated temporary values |

### Frame Pointer vs Stack Pointer

**Stack Pointer (SP):**
- Points to top of stack
- Changes as values are pushed/popped
- Not reliable for accessing locals

**Frame Pointer (FP):**
- Points to fixed location in current frame
- Stays constant during function execution
- Used to access locals and parameters

**Why both?**
```
Without FP:
    LOAD R1, -8(SP)    ; But SP keeps changing!
    
With FP:
    LOAD R1, -8(FP)    ; Always accesses same local
```

---

## 10.5 Function Call and Return

### The Call Sequence

**Caller's Responsibility:**
1. Evaluate and push/store arguments
2. Save any caller-saved registers
3. Execute CALL instruction (pushes return address)

Problem: Calls must save state and pass arguments reliably across functions.

Solution: Split responsibilities between caller and callee using a standard call/return sequence.

**Callee's Responsibility:**
1. Save old frame pointer
2. Set new frame pointer
3. Allocate space for locals
4. Save callee-saved registers
5. Execute function body

### The Return Sequence

**Callee's Responsibility:**
1. Store return value (in register or memory)
2. Restore callee-saved registers
3. Deallocate local space
4. Restore old frame pointer
5. Execute RETURN instruction

**Caller's Responsibility:**
1. Retrieve return value
2. Deallocate argument space
3. Restore caller-saved registers

### Complete Example

```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main() {
    int result = factorial(3);
}
```

**Stack During Execution of factorial(3):**

```
Initial call: main calls factorial(3)

│ result: ?           │ ← main's frame
├─────────────────────┤
│ n = 3               │
│ return to main      │
│ old FP →            │ ← factorial(3)'s frame
├─────────────────────┤ ← FP

After factorial(3) calls factorial(2):

│ result: ?           │ ← main's frame
├─────────────────────┤
│ n = 3               │
│ return to main      │
│ old FP →            │ ← factorial(3)'s frame
├─────────────────────┤
│ n = 2               │
│ return to fact(3)   │
│ old FP →            │ ← factorial(2)'s frame
├─────────────────────┤ ← FP

After factorial(2) calls factorial(1):

│ result: ?           │
├─────────────────────┤
│ n = 3               │ ← factorial(3)
├─────────────────────┤
│ n = 2               │ ← factorial(2)
├─────────────────────┤
│ n = 1               │ ← factorial(1)
├─────────────────────┤ ← FP

Unwinding (factorial(1) returns 1):
    factorial(2): returns 2 * 1 = 2
    factorial(3): returns 3 * 2 = 6
    main: result = 6
```

---

## 10.6 Parameter Passing Mechanisms

### 10.6.1 Call by Value

**The most common method.**

Problem: The caller and callee must agree on how arguments are passed.

Solution: Define clear passing strategies such as value, reference, or value-result.

- Actual parameter is **evaluated**
- Its **value is copied** to formal parameter
- Changes to formal don't affect actual

```c
void swap(int a, int b) {  // a, b are copies
    int temp = a;
    a = b;
    b = temp;
}  // Changes lost when function returns

int main() {
    int x = 5, y = 10;
    swap(x, y);  // x and y are still 5 and 10!
}
```

**Implementation:**
```asm
; Caller
LOAD  R1, x           ; Load x's value
PUSH  R1              ; Push value onto stack
LOAD  R1, y
PUSH  R1
CALL  swap

; Callee (swap)
; a and b are on stack, changes don't affect x, y
```

### 10.6.2 Call by Reference

- Actual parameter's **address** is passed
- Formal parameter is an **alias** for actual
- Changes to formal **do affect** actual

```c
void swap(int &a, int &b) {  // References in C++
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 5, y = 10;
    swap(x, y);  // x = 10, y = 5 now!
}
```

**Implementation:**
```asm
; Caller
LEA   R1, x           ; Load address of x
PUSH  R1
LEA   R1, y           ; Load address of y
PUSH  R1
CALL  swap

; Callee (swap)
LOAD  R1, 8(FP)       ; Load address of x
LOAD  R2, 12(FP)      ; Load address of y
LOAD  R3, (R1)        ; Load value of x
LOAD  R4, (R2)        ; Load value of y
STORE R4, (R1)        ; Store y's value in x
STORE R3, (R2)        ; Store x's value in y
```

### 10.6.3 Call by Value-Result (Copy-In, Copy-Out)

- Value is **copied in** at call
- Changes are made to local copy
- Result is **copied out** at return

```
procedure swap(a, b: inout integer);
    temp := a;
    a := b;
    b := temp;
end;
```

**Different from call by reference when:**
- Same variable passed twice: `swap(x, x)`
- Global variable accessed in function

### 10.6.4 Call by Name (Lazy Evaluation)

- Argument is **not evaluated** until used
- Textual substitution of actual for formal
- Used in Algol 60, modern lazy languages

```
// Conceptually
procedure example(x: by name integer)
    y := x;        // x is evaluated here
    z := x + x;    // x evaluated twice
end;

example(a[i++]);   // i++ happens each time x is used
```

### Comparison Table

| Method | When Evaluated | What's Passed | Changes Visible? |
|--------|----------------|---------------|------------------|
| Value | At call | Copy of value | No |
| Reference | At call | Address | Yes |
| Value-Result | At call & return | Copy, then copy back | Yes (at return) |
| Name | When used | Unevaluated expression | Yes |

---

## 10.7 Access to Non-Local Variables

### The Problem

How do nested functions access variables from enclosing scopes?

Problem: Nested functions need access to variables defined in outer scopes.

Solution: Use static links or displays to reach the correct activation record.

```pascal
program Example;
var x: integer;  // Global

procedure Outer;
var y: integer;  // Outer's local
    
    procedure Inner;
    var z: integer;  // Inner's local
    begin
        z := x + y;  // Must access x and y
    end;
    
begin
    Inner;
end;

begin
    Outer;
end.
```

### Solution 1: Static (Access) Links

Each activation record contains a pointer to the activation record of the **statically enclosing** function.

```
Stack when Inner is executing:

┌─────────────────────┐
│ x = ?               │ ← Global/Static data
├─────────────────────┤
│ y = ?               │
│ return addr         │
│ control link → ─────│─┐  
│ access link → ──────│─│─┐  ← Outer's frame
├─────────────────────┤ │ │
│ z = ?               │ │ │
│ return addr         │ │ │
│ control link → ─────│─┘ │  ← points to Outer
│ access link → ──────│───┘  ← also points to Outer (lexical parent)
└─────────────────────┘     Inner's frame
```

**Accessing y from Inner:**
1. Follow access link to Outer's frame
2. Access y at known offset

**Accessing x (two levels up):**
1. Follow access link to Outer
2. Follow Outer's access link to global
3. Access x

### Solution 2: Display

A **display** is a global array where `display[i]` holds a pointer to the most recent activation record at nesting level i.

```
Display:
┌─────────────┐
│ display[0]  │ → Global level
│ display[1]  │ → Outer's frame
│ display[2]  │ → Inner's frame
└─────────────┘
```

**Accessing variable at level k:**
- Use `display[k]` to get frame pointer
- Add offset to access variable

**Advantages:**
- O(1) access regardless of nesting depth
- Faster than following chain of links

**Disadvantages:**
- Display must be saved/restored on calls
- More complex to maintain

### Languages Without Nesting

Languages like C don't allow nested functions. All functions are at the same nesting level (global).

**Access to non-locals:**
- Local variables: via frame pointer
- Global variables: via fixed addresses

---

## 10.8 Heap Management

### 10.8.1 Dynamic Memory Allocation

**Allocation Functions:**
- C: `malloc()`, `calloc()`
- C++: `new`
- Java: `new` (garbage collected)

Problem: The heap must serve allocation requests of varying sizes efficiently.

Solution: Use free lists and allocation strategies (first fit, best fit, etc.).

**Deallocation Functions:**
- C: `free()`
- C++: `delete`
- Java: Automatic (garbage collection)

### 10.8.2 Heap Organization

**Free List:**
```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Free   │────▶│  Free   │────▶│  Free   │────▶ NULL
│  Block  │     │  Block  │     │  Block  │
│  100B   │     │  250B   │     │  50B    │
└─────────┘     └─────────┘     └─────────┘
```

### 10.8.3 Allocation Strategies

**First Fit:**
- Search list from beginning
- Allocate from first block that's big enough
- Fast but may cause fragmentation at beginning

**Best Fit:**
- Search entire list
- Allocate from smallest adequate block
- Less fragmentation but slower

**Worst Fit:**
- Allocate from largest block
- Idea: Leave larger remainder
- Often performs poorly

**Next Fit:**
- Like first fit, but start from last allocation point
- More even distribution

### 10.8.4 Fragmentation

**External Fragmentation:**
- Free memory exists but in non-contiguous blocks
- Can't satisfy large allocation even though total free > requested

```
Heap: [USED][FREE:50][USED][FREE:100][USED][FREE:75]

Request for 150 bytes fails even though 225 bytes are free
```

**Internal Fragmentation:**
- Allocated block is larger than requested
- Wasted space inside allocated blocks

```
Request: 100 bytes
Allocated: 128 bytes (rounded to power of 2)
Wasted: 28 bytes internal fragmentation
```

### 10.8.5 Coalescing

When a block is freed, merge it with adjacent free blocks:

```
Before freeing middle block:
[USED][FREE:50][USED:100][FREE:75]

After freeing and coalescing:
[USED][        FREE:225         ]
```

---

## 10.9 Garbage Collection

### The Problem

Manual memory management is error-prone:
- **Memory leaks**: Forgetting to free
- **Dangling pointers**: Using freed memory
- **Double free**: Freeing twice

Solution: Garbage collection automatically finds and frees unreachable objects.

### What is Garbage?

An object is **garbage** if it can never be accessed again by the program.

**Reachable objects:**
- Directly reachable from roots (stack, globals, registers)
- Reachable through pointers from other reachable objects

### 10.9.1 Reference Counting

Each object maintains a count of references to it.

```
When reference is created: count++
When reference is removed: count--
When count == 0: object is garbage, free it
```

**Example:**
```python
x = Object()      # Object.refcount = 1
y = x             # Object.refcount = 2
x = None          # Object.refcount = 1
y = None          # Object.refcount = 0 → FREE
```

**Problem: Cycles**
```
A.ref = B
B.ref = A
# Both have refcount 1
A = None
B = None
# Both still have refcount 1 (from each other)
# Memory leak!
```

### 10.9.2 Mark and Sweep

**Phase 1: Mark**
- Start from roots
- Mark all reachable objects

**Phase 2: Sweep**
- Scan all objects
- Free unmarked (unreachable) objects
- Clear marks for next collection

```
Before:
Root → [A] → [B] → [C]
            ↘
             [D] → [E]
       [F]  (unreachable)
       [G]  (unreachable)

Mark phase marks: A, B, C, D, E

Sweep phase frees: F, G
```

**Advantages:**
- Handles cycles
- Simple algorithm

**Disadvantages:**
- Pauses program during collection
- Must traverse all objects

### 10.9.3 Copying Collection

Divides heap into two semi-spaces: **From-space** and **To-space**.

**Algorithm:**
1. Allocate in From-space until full
2. Copy all reachable objects to To-space
3. Update all pointers
4. Swap roles of spaces

```
From-space:                    To-space:
[A][garbage][B][garbage][C]    [        empty        ]

After collection:
[        empty           ]    [A'][B'][C']
(now To-space)                 (now From-space)
```

**Advantages:**
- Automatic compaction (no fragmentation)
- Fast allocation (bump pointer)

**Disadvantages:**
- Uses half the memory
- Must copy all live objects

### 10.9.4 Generational Collection

**Observation**: Most objects die young.

**Idea**: Divide heap into generations:
- **Young generation**: Collected frequently
- **Old generation**: Collected rarely

```
Young Gen (nursery):      Old Gen (tenured):
[new objects here]        [long-lived objects]
    │                          ↑
    │ Objects that survive     │
    │ multiple collections     │
    └──────────────────────────┘
           (promoted)
```

**Benefits:**
- Most collections are fast (young gen is small)
- Long-lived objects rarely moved

---

## 10.10 Runtime Memory Layout Example

### Complete Example in C

```c
int global = 100;           // Static data

int factorial(int n) {      // Code segment
    if (n <= 1) return 1;
    return n * factorial(n-1);
}

int main() {
    int local = 5;          // Stack
    int *heap = malloc(40); // Heap
    int result = factorial(local);
    free(heap);
    return result;
}
```

**Memory Layout:**

```
┌──────────────────────────────────────────┐
│ STACK                                     │
│ ┌──────────────────────────────────────┐ │
│ │ factorial(1): n=1, return addr, FP   │ │
│ ├──────────────────────────────────────┤ │
│ │ factorial(2): n=2, return addr, FP   │ │
│ ├──────────────────────────────────────┤ │
│ │ ...                                  │ │
│ ├──────────────────────────────────────┤ │
│ │ factorial(5): n=5, return addr, FP   │ │
│ ├──────────────────────────────────────┤ │
│ │ main: local=5, heap=ptr, result=?    │ │
│ └──────────────────────────────────────┘ │
├──────────────────────────────────────────┤
│           (free space)                    │
├──────────────────────────────────────────┤
│ HEAP                                      │
│ ┌──────────────────────────────────────┐ │
│ │ [40 bytes allocated by malloc]       │ │
│ └──────────────────────────────────────┘ │
├──────────────────────────────────────────┤
│ STATIC DATA                               │
│ ┌──────────────────────────────────────┐ │
│ │ global: 100                          │ │
│ └──────────────────────────────────────┘ │
├──────────────────────────────────────────┤
│ CODE                                      │
│ ┌──────────────────────────────────────┐ │
│ │ main: ...                            │ │
│ │ factorial: ...                       │ │
│ └──────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

---

## 10.11 Summary

### Key Concepts:

1. **Memory Layout**: Code, Static Data, Stack, Heap

2. **Activation Records** contain:
   - Parameters, locals, return address
   - Control and access links
   - Saved registers, temporaries

3. **Storage Allocation Strategies**:
   - Static: Fixed at compile time
   - Stack: LIFO, automatic
   - Heap: Dynamic, manual/GC

4. **Parameter Passing**:
   - Value: Copy of value
   - Reference: Address of variable
   - Value-Result: Copy in, copy out
   - Name: Delayed evaluation

5. **Non-Local Access**:
   - Static links: Chain through enclosing scopes
   - Display: Array of frame pointers

6. **Heap Management**:
   - Free list with various fit strategies
   - Fragmentation (internal/external)
   - Coalescing adjacent free blocks

7. **Garbage Collection**:
   - Reference counting (problem with cycles)
   - Mark and sweep
   - Copying collection
   - Generational collection

Simple takeaway: The runtime environment defines how memory, calls, and data access work during program execution.

---

## 🔍 Practice Questions

1. Draw the stack showing activation records for:
   ```c
   int fib(int n) {
       if (n <= 1) return n;
       return fib(n-1) + fib(n-2);
   }
   main() { return fib(4); }
   ```

2. Explain the difference between control link and access link.

3. Compare call-by-value and call-by-reference with examples.

4. Given nested functions with 3 levels, show how static links work.

5. Compare mark-and-sweep with copying garbage collection.

6. What is fragmentation? How can it be reduced?

---

*This concludes the Compiler Design study notes. Good luck with your studies!*
