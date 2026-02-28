# Unit 5: Space Complexity

[← Previous: Big Omega and Big Theta](../04-Big-Omega-and-Big-Theta/04-big-omega-and-big-theta.md) | [Back to README](../README.md) | [Next: Recurrence Relations →](../06-Recurrence-Relations/06-recurrence-relations.md)

---

## Chapter Overview

While time complexity tells us how **fast** an algorithm is, space complexity tells us how much **memory** it uses. In many real-world scenarios (embedded systems, mobile apps, large datasets), memory is a critical constraint. This unit covers every aspect of space analysis.

---

## 5.1 Auxiliary Space

### Definition

**Auxiliary space** is the **extra space** used by an algorithm beyond the input itself.

```
Total Space = Input Space + Auxiliary Space
                  │              │
                  ▼              ▼
         Memory for the     EXTRA memory the
         input data         algorithm allocates

 When we say "space complexity," we usually mean AUXILIARY space.
```

### Example: Auxiliary Space Analysis

```
ALGORITHM SumArray(A, n)
    total ← 0               ← 1 variable   (int)
    FOR i ← 0 TO n-1        ← 1 variable   (int)
        total ← total + A[i]
    RETURN total

 Memory Layout:
 ┌──────────────────────┐
 │ Input: A[0..n-1]     │ ← Input space: O(n) — not counted
 ├──────────────────────┤
 │ total: 4 bytes       │ ← Auxiliary: constant
 │ i:     4 bytes       │ ← Auxiliary: constant
 └──────────────────────┘

 Auxiliary Space: O(1) — just a few variables regardless of n
```

### More Examples

```
ALGORITHM ReverseArray(A, n)            ALGORITHM CreateReversed(A, n)
    left ← 0                               B ← new Array[n]
    right ← n-1                             FOR i ← 0 TO n-1
    WHILE left < right                          B[i] ← A[n-1-i]
        SWAP(A[left], A[right])             RETURN B
        left ← left + 1
        right ← right - 1

 Auxiliary: O(1)                        Auxiliary: O(n)
 (modifies array in-place)             (creates new array of size n)

 Memory visualization:

 O(1) version:                         O(n) version:
 ┌───────────────────┐                 ┌───────────────────┐
 │ A: [1,2,3,4,5]    │  Input          │ A: [1,2,3,4,5]    │  Input
 ├───────────────────┤                 ├───────────────────┤
 │ left  = 0         │  Extra          │ B: [5,4,3,2,1]    │  Extra ← O(n)!
 │ right = 4         │  O(1)           │ i  = 0            │
 └───────────────────┘                 └───────────────────┘
```

---

## 5.2 Input Space

### Definition

**Input space** is the memory required to store the **input** to the algorithm.

```
┌───────────────────────────────────────────────────────┐
│ Algorithm takes an array of n integers as input       │
│                                                       │
│ Input Space = n × sizeof(int) = n × 4 bytes = O(n)   │
│                                                       │
│ Algorithm takes an n×n matrix as input                │
│                                                       │
│ Input Space = n² × sizeof(int) = O(n²)                │
└───────────────────────────────────────────────────────┘
```

### Why We Usually Ignore Input Space

```
Convention in complexity analysis:

 ┌─────────────────────────────────────────────────────────── ┐
 │ When we say "Space complexity is O(1)," we mean:          │
 │                                                            │
 │   Auxiliary space is O(1)                                  │
 │   (The extra space beyond the input)                       │
 │                                                            │
 │ The input is "given" — every algorithm needs it.           │
 │ What matters is the EXTRA space the algorithm uses.        │
 │                                                            │
 │ Exception: If the question specifically asks for           │
 │ "total space complexity," include input space.             │
 └────────────────────────────────────────────────────────────┘

 Example:
   Merge Sort
   - Input space: O(n)
   - Auxiliary space: O(n) (for the merge step)
   - Total space: O(n) + O(n) = O(n)
   - Usually stated as: Space = O(n) [meaning auxiliary]
```

---

## 5.3 Stack Space (Recursion)

### How Recursion Uses Memory

Every recursive call adds a **stack frame** to the call stack. Each frame stores:
- Local variables
- Parameters
- Return address

```
ALGORITHM Factorial(n)
    IF n ≤ 1 THEN RETURN 1
    RETURN n × Factorial(n-1)

Call Stack for Factorial(5):

┌─────────────────────┐
│ Factorial(1) n=1    │  ← top of stack (most recent call)
├─────────────────────┤
│ Factorial(2) n=2    │
├─────────────────────┤
│ Factorial(3) n=3    │
├─────────────────────┤
│ Factorial(4) n=4    │
├─────────────────────┤
│ Factorial(5) n=5    │  ← bottom of stack (first call)
└─────────────────────┘

 Maximum depth: 5 frames for n=5 → n frames for n
 Space: O(n)
```

### Recursion Space — Step by Step

```
Factorial(4) execution:

Step 1: Call Factorial(4)
  Stack: [Fact(4)]                    depth = 1

Step 2: Call Factorial(3)
  Stack: [Fact(4), Fact(3)]           depth = 2

Step 3: Call Factorial(2)
  Stack: [Fact(4), Fact(3), Fact(2)]  depth = 3

Step 4: Call Factorial(1) → returns 1
  Stack: [Fact(4), Fact(3), Fact(2), Fact(1)]  depth = 4 ← MAX

Step 5: Fact(1) returns, frame popped
  Stack: [Fact(4), Fact(3), Fact(2)]  depth = 3

Step 6: Fact(2) returns 2×1=2, frame popped
  Stack: [Fact(4), Fact(3)]           depth = 2

Step 7: Fact(3) returns 3×2=6, frame popped
  Stack: [Fact(4)]                    depth = 1

Step 8: Fact(4) returns 4×6=24, frame popped
  Stack: []                           depth = 0

 Space = maximum stack depth = n = O(n)
```

### Binary Recursion — Space Analysis

```
ALGORITHM Fibonacci(n)
    IF n ≤ 1 THEN RETURN n
    RETURN Fibonacci(n-1) + Fibonacci(n-2)

Call tree for Fibonacci(4):

                    fib(4)
                   /      \
               fib(3)      fib(2)
              /    \       /    \
          fib(2)  fib(1) fib(1) fib(0)
          /  \
      fib(1) fib(0)

 Time: O(2ⁿ) — many calls
 
 But SPACE is NOT O(2ⁿ)!
 
 Space depends on MAX STACK DEPTH, not total calls.
 
 At any point, only ONE branch is active:
 
 Deepest path: fib(4) → fib(3) → fib(2) → fib(1)
 Max depth: 4 = n

 Stack at deepest point:
 ┌────────┐
 │ fib(1) │
 ├────────┤
 │ fib(2) │
 ├────────┤
 │ fib(3) │
 ├────────┤
 │ fib(4) │
 └────────┘

 Space: O(n) — NOT O(2ⁿ)!

 KEY INSIGHT: Space = max depth of recursion tree
              Time = total number of nodes in tree
```

### Binary Search — Recursive Space

```
ALGORITHM BinarySearch(A, lo, hi, target)
    IF lo > hi THEN RETURN -1
    mid ← (lo + hi) / 2
    IF A[mid] = target THEN RETURN mid
    IF A[mid] < target
        RETURN BinarySearch(A, mid+1, hi, target)
    ELSE
        RETURN BinarySearch(A, lo, mid-1, target)

 Each call reduces problem by half: n → n/2 → n/4 → ... → 1
 Max depth = log₂(n)
 Space: O(log n)

 Stack for n=16, searching in right half each time:
 ┌──────────────────────────┐
 │ BS(A, 8, 8, target)     │  depth 4
 ├──────────────────────────┤
 │ BS(A, 8, 9, target)     │  depth 3
 ├──────────────────────────┤
 │ BS(A, 8, 11, target)    │  depth 2
 ├──────────────────────────┤
 │ BS(A, 8, 15, target)    │  depth 1
 ├──────────────────────────┤
 │ BS(A, 0, 15, target)    │  depth 0
 └──────────────────────────┘
 Max depth: log₂(16) = 4 →  O(log n)
```

### Tail Recursion Optimization

```
Some languages optimize tail recursion to use O(1) space:

NON-tail recursive:                  TAIL recursive:
ALGORITHM Fact(n)                    ALGORITHM Fact(n, acc=1)
    IF n ≤ 1 RETURN 1                   IF n ≤ 1 RETURN acc
    RETURN n × Fact(n-1)  ← needs      RETURN Fact(n-1, n×acc)  ← nothing
            to multiply AFTER return           after return
    Space: O(n)                          Space: O(1) with optimization

 The tail recursive version CAN be optimized by the compiler
 to reuse the same stack frame (not all languages do this).
 
 Languages with tail call optimization: Scheme, Haskell, Scala
 Languages WITHOUT: Python, Java, JavaScript (mostly)
```

---

## 5.4 In-Place Algorithms

### Definition

An **in-place algorithm** uses only a **constant amount of extra space** — O(1) auxiliary space. It works by modifying the input directly.

```
┌─────────────────────────────────────────────────────────┐
│           In-Place vs Not In-Place                      │
├────────────────────────┬────────────────────────────────┤
│ In-Place (O(1) aux)    │ Not In-Place (O(n)+ aux)      │
├────────────────────────┼────────────────────────────────┤
│ Bubble Sort            │ Merge Sort (needs O(n) array)  │
│ Insertion Sort         │ Counting Sort (needs O(k))     │
│ Selection Sort         │ Radix Sort (needs O(n+k))      │
│ Heap Sort              │ Creating a copy of input       │
│ Quick Sort*            │ Building a hash table          │
│ Reversing array        │ Building a new sorted array    │
│ Dutch National Flag    │ BFS (needs O(n) queue)         │
└────────────────────────┴────────────────────────────────┘

 * Quick Sort uses O(log n) for recursion stack — often
   considered "in-place" though not strictly O(1).
```

### Example: In-Place vs Not In-Place Reversal

```
IN-PLACE: Swap from both ends
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │   Original
└───┴───┴───┴───┴───┘
  ↕               ↕      Swap A[0], A[4]
┌───┬───┬───┬───┬───┐
│ 5 │ 2 │ 3 │ 4 │ 1 │
└───┴───┴───┴───┴───┘
      ↕       ↕          Swap A[1], A[3]
┌───┬───┬───┬───┬───┐
│ 5 │ 4 │ 3 │ 2 │ 1 │   Done! Extra space: O(1) — just temp variable
└───┴───┴───┴───┴───┘


NOT IN-PLACE: Create new array
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │   Original A
└───┴───┴───┴───┴───┘

┌───┬───┬───┬───┬───┐
│ 5 │ 4 │ 3 │ 2 │ 1 │   New array B   Extra space: O(n)
└───┴───┴───┴───┴───┘
```

### When to Use In-Place

```
✓ Use in-place when:
  • Memory is limited (embedded systems, mobile)
  • Input can be safely modified
  • The problem allows it

✗ Avoid in-place when:
  • Original data must be preserved
  • In-place version is much more complex
  • Immutability is required (functional programming)
```

---

## 5.5 Space-Time Trade-offs

### The Fundamental Trade-off

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│           SPACE ◄─────────────────────► TIME                │
│           (Memory)                     (Speed)              │
│                                                             │
│    Use MORE space to get FASTER execution                   │
│    Use LESS space but accept SLOWER execution               │
│                                                             │
│    You often CAN'T optimize both simultaneously!            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Example 1: Two Sum Problem

```
Problem: Given array A and target sum, find two numbers that add up to target.
         A = [2, 7, 11, 15], target = 9  →  Answer: indices [0, 1] (2+7=9)

Approach A: Brute Force
─────────────────────
FOR i ← 0 TO n-1
    FOR j ← i+1 TO n-1
        IF A[i] + A[j] = target
            RETURN (i, j)

 Time:  O(n²)  — check all pairs
 Space: O(1)   — no extra space

Approach B: Hash Map
─────────────────────
map ← empty hash map
FOR i ← 0 TO n-1
    complement ← target - A[i]
    IF complement IN map
        RETURN (map[complement], i)
    map[A[i]] ← i

 Time:  O(n)   — single pass
 Space: O(n)   — hash map stores up to n elements

 Trade-off:
 ┌────────────┬──────────┬──────────┐
 │ Approach   │  Time    │  Space   │
 ├────────────┼──────────┼──────────┤
 │ Brute      │  O(n²)   │  O(1)   │ ← Slow but memory-efficient
 │ Hash Map   │  O(n)    │  O(n)   │ ← Fast but uses more memory
 └────────────┴──────────┴──────────┘
```

### Example 2: Fibonacci — Multiple Approaches

```
Approach A: Recursive (No extra space for storage, but stack space)
 Time: O(2ⁿ)    Space: O(n) call stack

Approach B: Memoization (Cache results)
 Time: O(n)      Space: O(n) memo array + O(n) stack = O(n)

Approach C: Bottom-up DP with array
 Time: O(n)      Space: O(n) array

Approach D: Bottom-up with two variables
 Time: O(n)      Space: O(1)      ← BEST of both!

 Approach D pseudocode:
 ALGORITHM Fibonacci(n)
     a ← 0, b ← 1
     FOR i ← 2 TO n
         temp ← a + b
         a ← b
         b ← temp
     RETURN b

 Memory at each step:
 ┌─────┬─────┬─────┐
 │  a  │  b  │temp │    Only 3 variables, regardless of n!
 └─────┴─────┴─────┘
 i=2: a=0, b=1, temp=1 → a=1, b=1
 i=3: a=1, b=1, temp=2 → a=1, b=2
 i=4: a=1, b=2, temp=3 → a=2, b=3
 i=5: a=2, b=3, temp=5 → a=3, b=5 → fib(5) = 5 ✓
```

### Example 3: Lookup Table — Trading Space for Time

```
Problem: Convert character to uppercase repeatedly

Approach A: Compute each time
 def toUpper(c):
     if 'a' <= c <= 'z':
         return chr(ord(c) - 32)
     return c

 Time per call: O(1) with computation
 Space: O(1)

Approach B: Pre-built lookup table
 table = {'a':'A', 'b':'B', ..., 'z':'Z'}  # 26 entries

 def toUpper(c):
     return table.get(c, c)

 Time per call: O(1) — direct lookup, slightly faster
 Space: O(26) = O(1) — but uses memory for table

 For this example, both are O(1), but the table approach
 is common for more complex transformations.
```

### Classic Space-Time Trade-offs

```
┌──────────────────────┬──────────────┬─────────┬──────────────────┐
│ Technique            │ Extra Space  │ Time    │ Speed Gain       │
├──────────────────────┼──────────────┼─────────┼──────────────────┤
│ Hash table for lookup│ O(n)         │O(1) avg │ O(n) → O(1)     │
│ Memoization / DP     │ O(n) or O(n²)│O(n)     │ O(2ⁿ) → O(n)   │
│ Precomputed tables   │ O(k)         │O(1)     │ Avoids recompute│
│ Caching (LRU, etc.)  │ O(k)         │O(1) hit │ Avoid recompute │
│ Adjacency matrix     │ O(V²)        │O(1)     │ O(V) → O(1)    │
│ Suffix array/tree    │ O(n)         │O(m)     │ O(n·m) → O(m)  │
│ Prefix sums          │ O(n)         │O(1)     │ O(n) → O(1)    │
│ Bit manipulation     │ O(1)         │O(1)     │ Reduced space   │
└──────────────────────┴──────────────┴─────────┴──────────────────┘
```

### Example 4: Prefix Sum Array

```
Problem: Answer many range-sum queries on an array

A = [3, 1, 4, 1, 5, 9, 2, 6]

Query: Sum from index 2 to 5 = 4 + 1 + 5 + 9 = 19

Naive: For each query, loop from i to j → O(n) per query
       For Q queries: O(Q × n)

With prefix sum:
P[0] = 0
P[1] = 3
P[2] = 3+1 = 4
P[3] = 4+4 = 8
P[4] = 8+1 = 9
P[5] = 9+5 = 14
P[6] = 14+9 = 23
P[7] = 23+2 = 25
P[8] = 25+6 = 31

Sum(2,5) = P[6] - P[2] = 23 - 4 = 19 ✓   → O(1) per query!

 Build: O(n) time, O(n) space
 Query: O(1) per query
 Total for Q queries: O(n + Q) instead of O(Q × n)

 ┌───────────────┬───────────┬───────────┐
 │               │   Time    │   Space   │
 ├───────────────┼───────────┼───────────┤
 │ Naive         │ O(Q × n)  │   O(1)   │
 │ Prefix Sum    │ O(n + Q)  │   O(n)   │   ← MUCH faster
 └───────────────┴───────────┴───────────┘
```

---

## 5.6 Memory Optimization

### Common Techniques

```
┌─────────────────────────────────────────────────────────────┐
│              Memory Optimization Techniques                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. USE IN-PLACE ALGORITHMS                                 │
│     Modify input instead of creating copies                 │
│                                                             │
│  2. REDUCE DP TABLE DIMENSIONS                              │
│     If dp[i] only depends on dp[i-1], use two arrays       │
│     instead of n arrays                                     │
│                                                             │
│  3. USE BIT MANIPULATION                                    │
│     Store boolean flags in bits instead of bytes            │
│     32 booleans in one int instead of 32 bytes              │
│                                                             │
│  4. ITERATIVE OVER RECURSIVE                                │
│     Eliminate call stack overhead                            │
│     Convert recursion to iteration when possible            │
│                                                             │
│  5. STREAM PROCESSING                                       │
│     Process data one piece at a time instead of loading all │
│                                                             │
│  6. CHOOSE COMPACT DATA STRUCTURES                          │
│     Use arrays instead of linked lists (no pointer overhead)│
│     Use appropriate data types (short vs int vs long)       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Technique: DP Space Optimization

```
Problem: Maximum path sum in a grid (n×n)

Full DP table: O(n²) space
┌─────┬─────┬─────┬─────┐
│  1  │  3  │  1  │  2  │
├─────┼─────┼─────┼─────┤
│  4  │  5  │  3  │  1  │
├─────┼─────┼─────┼─────┤
│  2  │  6  │  7  │  8  │
├─────┼─────┼─────┼─────┤
│  3  │  2  │  4  │  9  │
└─────┴─────┴─────┴─────┘

Observation: Each row only depends on the previous row!

Optimized: Keep only 2 rows → O(n) space
┌─────┬─────┬─────┬─────┐
│ prev row              │  ← Previous row values
├─────┼─────┼─────┼─────┤
│ curr row              │  ← Current row being computed
└─────┴─────┴─────┴─────┘

Space reduced from O(n²) to O(n)!
```

### Technique: Converting Recursion to Iteration

```
Recursive (O(n) stack space):          Iterative (O(1) space):
──────────────────────────             ────────────────────────
ALGORITHM SumList(head)                ALGORITHM SumList(head)
    IF head = NULL                         total ← 0
        RETURN 0                           node ← head
    RETURN head.val +                      WHILE node ≠ NULL
           SumList(head.next)                  total ← total + node.val
                                               node ← node.next
Stack: O(n)                                RETURN total

                                       Stack: O(1) — no recursive calls!

When to convert:
 ✓ Linear recursion (one recursive call) → easy to convert
 ✗ Tree recursion (branching) → may need explicit stack
 ⚡ Tail recursion → simplest to convert
```

### Technique: Bit Manipulation for Space

```
Storing 8 boolean flags:

Using boolean array:          Using one byte with bits:
┌─────────────────────┐      ┌───────────────────────────┐
│ bool flags[8]       │      │ unsigned char flags = 0   │
│ = 8 bytes (64 bits) │      │ = 1 byte (8 bits)        │
└─────────────────────┘      └───────────────────────────┘
                              
  flags[i] = true             flags |= (1 << i)   // set bit i
  flags[i] = false            flags &= ~(1 << i)  // clear bit i
  check flags[i]              (flags >> i) & 1     // check bit i

 Savings: 8x less memory!
 For n flags: n bytes → n/8 bytes → O(n) but 8x smaller constant
```

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| Auxiliary Space | Extra space beyond input; main measure of space complexity |
| Input Space | Memory for the input data; usually not counted separately |
| Stack Space | Recursion creates stack frames; space = max recursion depth |
| Binary Recursion Space | O(depth), NOT O(total calls); only one path active at a time |
| In-Place Algorithm | O(1) auxiliary space; modifies input directly |
| Space-Time Trade-off | Use more space → faster execution (and vice versa) |
| Hash Table Trade-off | O(n) extra space → O(1) lookup instead of O(n) |
| Memoization | O(n) space → avoid recomputing: O(2ⁿ) → O(n) |
| Prefix Sums | O(n) space → O(1) range queries instead of O(n) |
| DP Optimization | If row depends only on previous row, keep 2 rows: O(n²) → O(n) |
| Iteration vs Recursion | Converting to iteration saves O(n) stack space |

---

## Quick Revision Questions

**Q1.** What is the difference between auxiliary space and total space complexity?

<details>
<summary>Answer</summary>
Auxiliary space is the EXTRA space beyond the input (variables, temporary arrays, stack frames). Total space = input space + auxiliary space. When people say "space complexity is O(1)," they usually mean O(1) auxiliary space. The input itself always takes O(n) for an array of n elements.
</details>

**Q2.** The recursive Fibonacci function has time O(2ⁿ). Is its space complexity also O(2ⁿ)?

<details>
<summary>Answer</summary>
No! Space is O(n). Although there are O(2ⁿ) total calls, at any given time only ONE path from root to leaf is on the call stack. The deepest path has n frames. After a branch completes, its frames are popped before the other branch starts. Space = max stack depth = O(n).
</details>

**Q3.** Explain the space-time trade-off in the Two Sum problem.

<details>
<summary>Answer</summary>
Brute force uses O(1) space but O(n²) time (checking all pairs). The hash map approach uses O(n) space to store seen elements but reduces time to O(n) (single pass with O(1) lookups). We trade extra memory for dramatically faster execution.
</details>

**Q4.** Why is Merge Sort considered "not in-place" even though it sorts the original array?

<details>
<summary>Answer</summary>
The merge step requires a temporary array of size O(n) to merge two sorted halves. Even though the result ends up in the original array, the algorithm needs O(n) auxiliary space during execution. In-place means O(1) extra space, which Merge Sort doesn't achieve.
</details>

**Q5.** How can you optimize a 2D DP table to use O(n) space instead of O(n²)?

<details>
<summary>Answer</summary>
If each row of the DP table only depends on the immediately previous row (not rows further back), you only need to keep two rows: the "previous" row and the "current" row being computed. After filling the current row, swap them and reuse. This reduces O(n²) space to O(n). In some cases, you can even use a single row with careful ordering.
</details>

**Q6.** What is the space complexity of iterative binary search vs recursive binary search?

<details>
<summary>Answer</summary>
Iterative binary search: O(1) — uses only a few variables (lo, hi, mid). Recursive binary search: O(log n) — creates log₂(n) stack frames as the search space halves at each level. The iterative version is more space-efficient, which is why it's often preferred.
</details>

---

[← Previous: Big Omega and Big Theta](../04-Big-Omega-and-Big-Theta/04-big-omega-and-big-theta.md) | [Back to README](../README.md) | [Next: Recurrence Relations →](../06-Recurrence-Relations/06-recurrence-relations.md)
