# 7.6 Cycle Detection

## Unit 7: Hash Set Problems

---

## The Pattern

Use a **HashSet** to detect cycles by tracking visited states. If you visit the same state twice, a cycle exists.

```
╔══════════════════════════════════════════════════════════════╗
║              CYCLE DETECTION WITH HASHSET                   ║
║                                                              ║
║  visited = new HashSet()                                    ║
║                                                              ║
║  WHILE state is valid:                                      ║
║      IF state IN visited → CYCLE DETECTED                  ║
║      visited.add(state)                                     ║
║      state = nextState(state)                               ║
║                                                              ║
║  → No cycle (reached terminal state)                       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Problem 1: Linked List Cycle

> Given a linked list, determine if it has a cycle.

### HashSet Approach

```
FUNCTION hasCycle(head):
    visited = new HashSet()

    current = head
    WHILE current != null:
        IF visited.contains(current):
            RETURN true       // Cycle found
        visited.add(current)
        current = current.next

    RETURN false              // Reached end, no cycle
```

### Trace

```
List: 3 → 2 → 0 → -4 → (back to 2)

Step 1: current=3, visited={}, not found → add → visited={3}
Step 2: current=2, visited={3}, not found → add → visited={3,2}
Step 3: current=0, visited={3,2}, not found → add → visited={3,2,0}
Step 4: current=-4, visited={3,2,0}, not found → add → visited={3,2,0,-4}
Step 5: current=2, visited={3,2,0,-4}, FOUND! → return true ✓

Note: We store NODE REFERENCES, not values.
      Two nodes can have the same value but be different objects.
```

### Floyd's Cycle Detection (Two Pointers)

The optimal $O(1)$ space solution:

```
FUNCTION hasCycle(head):
    slow = head
    fast = head

    WHILE fast != null AND fast.next != null:
        slow = slow.next
        fast = fast.next.next

        IF slow == fast:
            RETURN true

    RETURN false
```

```
╔══════════════════════════════════════════════════════════════╗
║  3 → 2 → 0 → -4                                           ║
║      ↑          │                                           ║
║      └──────────┘                                           ║
║                                                              ║
║  Step   slow   fast                                         ║
║   0      3      3                                           ║
║   1      2      0     (slow+1, fast+2)                     ║
║   2      0     -4→2   (fast wraps around)                  ║
║   3     -4      0                                           ║
║   4      2     -4→2   → slow==fast? No wait...             ║
║                                                              ║
║  Let me re-trace with positions:                            ║
║  Nodes by position: [3₀, 2₁, 0₂, -4₃] cycle at 1         ║
║                                                              ║
║  Step   slow    fast                                        ║
║   0     pos 0   pos 0                                       ║
║   1     pos 1   pos 2                                       ║
║   2     pos 2   pos 2 (went 3→1→2 via cycle)               ║
║         slow == fast → CYCLE DETECTED ✓                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Problem 2: Happy Number

> A number is "happy" if replacing it by the sum of squares of its digits eventually reaches 1. If it loops endlessly, it's not happy.

```
Example: 19
  1² + 9² = 82
  8² + 2² = 68
  6² + 8² = 100
  1² + 0² + 0² = 1    → Happy! ✓

Example: 2
  2² = 4
  4² = 16
  1² + 6² = 37
  3² + 7² = 58
  5² + 8² = 89
  8² + 9² = 145
  1² + 4² + 5² = 42
  4² + 2² = 20
  2² + 0² = 4    ← CYCLE (we saw 4 before)! → Not happy ✗
```

### HashSet Solution

```
FUNCTION isHappy(n):
    seen = new HashSet()

    WHILE n != 1:
        IF seen.contains(n):
            RETURN false    // Cycle detected
        seen.add(n)
        n = sumOfSquares(n)

    RETURN true

FUNCTION sumOfSquares(n):
    sum = 0
    WHILE n > 0:
        digit = n % 10
        sum += digit * digit
        n = n / 10
    RETURN sum
```

### Floyd's Approach (O(1) Space)

```
FUNCTION isHappy(n):
    slow = n
    fast = sumOfSquares(n)

    WHILE fast != 1 AND slow != fast:
        slow = sumOfSquares(slow)
        fast = sumOfSquares(sumOfSquares(fast))

    RETURN fast == 1
```

### Full Trace for n = 19

```
seen = {}

n=19: not 1, not in seen → add → seen={19}
  1²+9² = 82

n=82: not 1, not in seen → add → seen={19,82}
  8²+2² = 68

n=68: not 1, not in seen → add → seen={19,82,68}
  6²+8² = 100

n=100: not 1, not in seen → add → seen={19,82,68,100}
  1²+0²+0² = 1

n=1: n==1 → return true ✓
```

---

## Problem 3: Find the Duplicate Number

> Given an array of $n+1$ integers where each integer is in $[1, n]$, find the duplicate.

```
╔══════════════════════════════════════════════════════════════╗
║  nums = [1, 3, 4, 2, 2]                                    ║
║                                                              ║
║  HashSet approach:                                          ║
║    1 → {1}                                                  ║
║    3 → {1, 3}                                               ║
║    4 → {1, 3, 4}                                            ║
║    2 → {1, 3, 4, 2}                                         ║
║    2 → FOUND! Return 2 ✓                                   ║
║                                                              ║
║  Cycle approach (O(1) space):                               ║
║    Treat array as a linked list:                            ║
║    index → nums[index]                                      ║
║    0→1→3→2→4→2→4→2... (CYCLE at 2)                        ║
╚══════════════════════════════════════════════════════════════╝
```

### Cycle Detection for Duplicate Finding

```
FUNCTION findDuplicate(nums):
    // Phase 1: Detect cycle (find meeting point)
    slow = nums[0]
    fast = nums[0]

    DO:
        slow = nums[slow]
        fast = nums[nums[fast]]
    WHILE slow != fast

    // Phase 2: Find cycle entrance (= duplicate)
    slow = nums[0]
    WHILE slow != fast:
        slow = nums[slow]
        fast = nums[fast]

    RETURN slow
```

### Trace

```
nums = [1, 3, 4, 2, 2]
Index:  0  1  2  3  4

Implicit linked list: 0→1→3→2→4→2→4→2...

Phase 1 (detect meeting point):
  slow = nums[0] = 1, fast = nums[0] = 1
  
  Iteration 1:
    slow = nums[1] = 3
    fast = nums[nums[1]] = nums[3] = 2
  
  Iteration 2:
    slow = nums[3] = 2
    fast = nums[nums[2]] = nums[4] = 2
    
    slow == fast == 2 → meeting point found

Phase 2 (find entrance):
  slow = nums[0] = 1, fast = 2

  Iteration 1:
    slow = nums[1] = 3
    fast = nums[2] = 4

  Iteration 2:
    slow = nums[3] = 2
    fast = nums[4] = 2

    slow == fast == 2 → duplicate is 2 ✓
```

---

## HashSet vs Floyd's: When to Use Each

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  ┌─────────────────────────────┐                            ║
║  │ Need O(1) space?            │                            ║
║  │   YES → Floyd's algorithm   │                            ║
║  │   NO  → HashSet (simpler)   │                            ║
║  └─────────────────────────────┘                            ║
║                                                              ║
║  ┌─────────────────────────────┐                            ║
║  │ Need cycle START position?  │                            ║
║  │   YES → Floyd's Phase 2    │                            ║
║  │   NO  → Either works       │                            ║
║  └─────────────────────────────┘                            ║
║                                                              ║
║  ┌─────────────────────────────┐                            ║
║  │ Complex state (not numeric)?│                            ║
║  │   YES → HashSet             │                            ║
║  │   NO  → Floyd's possible   │                            ║
║  └─────────────────────────────┘                            ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

| Feature | HashSet | Floyd's |
|---------|:---:|:---:|
| Time complexity | $O(n)$ | $O(n)$ |
| Space complexity | $O(n)$ | $O(1)$ |
| Implementation | Simple | Moderate |
| Finds cycle start | Extra work | Phase 2 |
| Works with any state | Yes | Needs successor function |
| Detects cycle length | Count elements | Extra pass |

---

## Quick Revision Question

**Q: In the Happy Number problem, why are we guaranteed that the sequence either reaches 1 or enters a cycle? Why can't it grow forever?**

<details>
<summary>Click to reveal answer</summary>

The sequence **cannot grow forever** because the sum of squares of digits is always **much smaller** than the number itself for large numbers.

**Proof by bounding:**
- For a $d$-digit number, the maximum value is $10^d - 1$
- The maximum sum of digit squares is $d \times 9^2 = 81d$
- For $d \geq 4$: $81d < 10^d - 1$

**Example:** A 4-digit number (up to 9999) maps to at most $81 \times 4 = 324$.

So any number > 999 maps to a number ≤ $81 \times \lfloor\log_{10}(n)\rfloor + 81$ — which is at most a 3-digit number.

Once in the range [1, 999], there are only 999 possible states. By the **pigeonhole principle**, after at most 1000 steps, some state must repeat — creating a cycle.

Therefore the sequence either:
1. Reaches 1 (happy)
2. Enters a cycle not containing 1 (not happy)

It can never diverge to infinity.

</details>

---

| [← Contains Duplicate Variants](05-contains-duplicate.md) | [Unit 8: String Hashing →](../08-String-Hashing-Problems/01-rabin-karp-algorithm.md) |
|:---|---:|
