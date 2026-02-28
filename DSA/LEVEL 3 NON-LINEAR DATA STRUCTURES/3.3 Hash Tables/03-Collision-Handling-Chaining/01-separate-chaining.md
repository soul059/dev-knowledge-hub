# 3.1 Separate Chaining

## Unit 3: Collision Handling - Chaining

---

## The Collision Problem

When two different keys hash to the same index, we have a **collision**. Separate chaining is the most intuitive solution: store all colliding elements in a linked list at that index.

```
╔══════════════════════════════════════════════════════════════╗
║                 SEPARATE CHAINING                           ║
║                                                             ║
║   Hash function: h(k) = k mod 7                            ║
║   Keys: 10, 22, 31, 4, 15, 28, 17, 88, 59                ║
║                                                             ║
║   h(10)=3  h(22)=1  h(31)=3  h(4)=4  h(15)=1             ║
║   h(28)=0  h(17)=3  h(88)=4  h(59)=3                     ║
║                                                             ║
║   Index                                                     ║
║   ┌───┐                                                    ║
║   │ 0 │──► [28] ──► null                                   ║
║   ├───┤                                                    ║
║   │ 1 │──► [22] ──► [15] ──► null                         ║
║   ├───┤                                                    ║
║   │ 2 │──► null                                            ║
║   ├───┤                                                    ║
║   │ 3 │──► [10] ──► [31] ──► [17] ──► [59] ──► null      ║
║   ├───┤                                                    ║
║   │ 4 │──► [4] ──► [88] ──► null                          ║
║   ├───┤                                                    ║
║   │ 5 │──► null                                            ║
║   ├───┤                                                    ║
║   │ 6 │──► null                                            ║
║   └───┘                                                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## How Separate Chaining Works

```
╔═══════════════════════════════════════════════════════════╗
║              CHAINING OPERATIONS                         ║
║                                                          ║
║   INSERT(key, value):                                    ║
║   1. Compute index = h(key)                             ║
║   2. Add (key, value) to beginning of list at index     ║
║   → O(1)                                                ║
║                                                          ║
║   SEARCH(key):                                           ║
║   1. Compute index = h(key)                             ║
║   2. Traverse list at index, compare keys               ║
║   → O(chain length) = O(n/m) average                   ║
║                                                          ║
║   DELETE(key):                                           ║
║   1. Compute index = h(key)                             ║
║   2. Search list at index, remove node                  ║
║   → O(chain length) = O(n/m) average                   ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Trace: Insert Operations

```
Table size m = 5, h(k) = k mod 5
Operations: insert(7), insert(12), insert(22), insert(17), insert(2)

Step 1: insert(7)
  h(7) = 7 % 5 = 2
  ┌───┐
  │ 0 │──► null
  │ 1 │──► null
  │ 2 │──► [7] ──► null
  │ 3 │──► null
  │ 4 │──► null
  └───┘

Step 2: insert(12)
  h(12) = 12 % 5 = 2    ◄── collision with 7!
  ┌───┐
  │ 0 │──► null
  │ 1 │──► null
  │ 2 │──► [12] ──► [7] ──► null     (prepend to list)
  │ 3 │──► null
  │ 4 │──► null
  └───┘

Step 3: insert(22)
  h(22) = 22 % 5 = 2    ◄── another collision!
  ┌───┐
  │ 0 │──► null
  │ 1 │──► null
  │ 2 │──► [22] ──► [12] ──► [7] ──► null
  │ 3 │──► null
  │ 4 │──► null
  └───┘

Step 4: insert(17)
  h(17) = 17 % 5 = 2    ◄── yet another collision!
  ┌───┐
  │ 0 │──► null
  │ 1 │──► null
  │ 2 │──► [17] ──► [22] ──► [12] ──► [7] ──► null
  │ 3 │──► null
  │ 4 │──► null
  └───┘

Step 5: insert(2)
  h(2) = 2 % 5 = 2
  ┌───┐
  │ 0 │──► null
  │ 1 │──► null
  │ 2 │──► [2] ──► [17] ──► [22] ──► [12] ──► [7] ──► null
  │ 3 │──► null
  │ 4 │──► null
  └───┘

Worst case: all 5 keys hashed to same index → chain of length 5!
Search for key 7 requires traversing entire chain.
```

---

## Step-by-Step Trace: Search Operation

```
Using table from above, search(12):

Step 1: h(12) = 12 % 5 = 2
Step 2: Go to index 2
Step 3: Check [2]  → 2 ≠ 12, move next
Step 4: Check [17] → 17 ≠ 12, move next
Step 5: Check [22] → 22 ≠ 12, move next
Step 6: Check [12] → 12 = 12, FOUND! ✓

Comparisons needed: 4

search(99):
Step 1: h(99) = 99 % 5 = 4
Step 2: Go to index 4
Step 3: List at index 4 is null → NOT FOUND ✗

Comparisons needed: 0 (empty bucket = instant miss)
```

---

## Pseudocode

```
CLASS ChainedHashTable:
    FUNCTION init(capacity):
        this.capacity = capacity
        this.size = 0
        this.table = new Array(capacity)
        FOR i = 0 TO capacity - 1:
            this.table[i] = new LinkedList()
    
    FUNCTION hash(key):
        RETURN key.hashCode() MOD this.capacity
    
    FUNCTION insert(key, value):
        index = this.hash(key)
        // Check if key already exists (update value)
        FOR each node in this.table[index]:
            IF node.key == key:
                node.value = value
                RETURN
        // Key not found, prepend new entry
        this.table[index].prepend(key, value)
        this.size = this.size + 1
    
    FUNCTION search(key):
        index = this.hash(key)
        FOR each node in this.table[index]:
            IF node.key == key:
                RETURN node.value
        RETURN null    // Not found
    
    FUNCTION delete(key):
        index = this.hash(key)
        this.table[index].remove(key)
        this.size = this.size - 1
```

---

## Advantages and Disadvantages

| Advantages | Disadvantages |
|-----------|---------------|
| Simple to implement | Extra memory for pointers |
| Never "full" (can always add more) | Poor cache locality |
| Deletion is straightforward | Linked list overhead |
| Performance degrades gradually | Chain traversal can be slow |
| Works well with high load factors | Pointer chasing in memory |

---

## Quick Revision Question

**Q: In a hash table of size 10 using separate chaining, you insert keys 5, 15, 25, 35, and 45 using h(k) = k mod 10. What does the table look like? What is the search cost for key 45?**

<details>
<summary>Click to reveal answer</summary>

All keys hash to index 5: h(5)=5, h(15)=5, h(25)=5, h(35)=5, h(45)=5

```
Index 5: [45] → [35] → [25] → [15] → [5] → null
```

All other indices are empty. To search for key 45:
- Go to index 5
- Check first node: 45 = 45 → **Found in 1 comparison!** (since it was inserted last and prepended)

To search for key 5:
- Must traverse the entire chain: 5 comparisons (worst case in this chain)

Average search in this chain: $(1+2+3+4+5)/5 = 3$ comparisons

</details>

---

| [← Unit 2: String Hashing](../02-Hash-Functions/06-string-hashing.md) | [Next: Linked List Implementation →](02-linked-list-implementation.md) |
|:---|---:|
