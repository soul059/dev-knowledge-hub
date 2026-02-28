# 9.4 Cuckoo Hashing

## Unit 9: Advanced Hashing

---

## What is Cuckoo Hashing?

An open addressing scheme that guarantees **worst-case $O(1)$ lookup** by using **two hash functions** and **displacing existing elements** (like a cuckoo bird evicting eggs from nests).

```
╔══════════════════════════════════════════════════════════════╗
║  STANDARD OPEN ADDRESSING:                                  ║
║    Insert: O(1) amortized     Lookup: O(1) average         ║
║    Problem: Long probe chains → O(n) worst-case lookup     ║
║                                                              ║
║  CUCKOO HASHING:                                            ║
║    Insert: O(1) amortized (with rehashing)                  ║
║    Lookup: O(1) WORST-CASE  (check exactly 2 positions)    ║
║    Delete: O(1) WORST-CASE  (check exactly 2 positions)    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## How It Works

Two hash tables $T_1$ and $T_2$ (or one table with two sections), each with its own hash function.

**Rule:** Element $x$ is stored at either $T_1[h_1(x)]$ or $T_2[h_2(x)]$ — and nowhere else.

```
╔══════════════════════════════════════════════════════════════╗
║  Table T₁ (h₁)         Table T₂ (h₂)                      ║
║  ┌───┬───┬───┬───┐     ┌───┬───┬───┬───┐                  ║
║  │   │ A │   │ C │     │   │   │ B │   │                  ║
║  └───┴───┴───┴───┘     └───┴───┴───┴───┘                  ║
║    0   1   2   3          0   1   2   3                     ║
║                                                              ║
║  LOOKUP(A): check T₁[h₁(A)]=T₁[1] → found! O(1)          ║
║  LOOKUP(B): check T₁[h₁(B)]=T₁[?] → empty                ║
║             check T₂[h₂(B)]=T₂[2] → found! O(1)          ║
║  LOOKUP(D): check T₁[h₁(D)]→empty, T₂[h₂(D)]→empty      ║
║             → not found! O(1)                               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Insert Algorithm (The "Cuckoo" Part)

```
FUNCTION insert(x):
    IF lookup(x): RETURN    // Already present

    FOR i = 0 TO MAX_KICKS:
        // Try to place x in T₁
        IF T₁[h₁(x)] is empty:
            T₁[h₁(x)] = x
            RETURN

        // Evict current occupant
        SWAP(x, T₁[h₁(x)])

        // Try to place evicted element in T₂
        IF T₂[h₂(x)] is empty:
            T₂[h₂(x)] = x
            RETURN

        // Evict from T₂
        SWAP(x, T₂[h₂(x)])

    // Too many kicks → cycle detected → rehash
    rehash()
    insert(x)
```

---

## Complete Trace

```
T₁ size = 4 (h₁), T₂ size = 4 (h₂)

h₁: A→1, B→1, C→3, D→1, E→0
h₂: A→0, B→2, C→1, D→3, E→1

Step 1: INSERT A
  T₁[h₁(A)] = T₁[1] empty → place A
  T₁: [_, A, _, _]    T₂: [_, _, _, _]

Step 2: INSERT B
  T₁[h₁(B)] = T₁[1] = A → EVICT A, place B
  T₁: [_, B, _, _]    T₂: [_, _, _, _]
  Now place evicted A in T₂:
  T₂[h₂(A)] = T₂[0] empty → place A
  T₁: [_, B, _, _]    T₂: [A, _, _, _]

Step 3: INSERT C
  T₁[h₁(C)] = T₁[3] empty → place C
  T₁: [_, B, _, C]    T₂: [A, _, _, _]

Step 4: INSERT D
  T₁[h₁(D)] = T₁[1] = B → EVICT B, place D
  T₁: [_, D, _, C]    T₂: [A, _, _, _]
  Place evicted B in T₂:
  T₂[h₂(B)] = T₂[2] empty → place B
  T₁: [_, D, _, C]    T₂: [A, _, B, _]

Step 5: INSERT E
  T₁[h₁(E)] = T₁[0] empty → place E
  T₁: [E, D, _, C]    T₂: [A, _, B, _]

Final state:
  T₁: [ E  | D  | __ | C  ]
       h₁:  0    1    2    3
  T₂: [ A  | __ | B  | __ ]
       h₂:  0    1    2    3

Verify all lookups are O(1):
  A: T₁[1]=D ✗, T₂[0]=A ✓ (2 checks)
  B: T₁[1]=D ✗, T₂[2]=B ✓ (2 checks)
  C: T₁[3]=C ✓ (1 check)
  D: T₁[1]=D ✓ (1 check)
  E: T₁[0]=E ✓ (1 check)
```

---

## Cycle Detection and Rehashing

```
╔══════════════════════════════════════════════════════════════╗
║  A cycle occurs when eviction chains loop forever:          ║
║                                                              ║
║  Insert X → evicts A from T₁                               ║
║  A goes to T₂ → evicts B from T₂                           ║
║  B goes to T₁ → evicts C from T₁                           ║
║  C goes to T₂ → evicts A from T₂                           ║
║  A goes to T₁ → evicts X from T₁                           ║
║  X goes to T₂ → evicts B from T₂  ← LOOP!                ║
║                                                              ║
║  Detection: Limit kicks to O(log n) or 6·log₂(n)           ║
║  Solution:  Choose new h₁, h₂ and rehash all elements      ║
║                                                              ║
║  Probability of rehash: O(1/n) per insert                   ║
║  → Amortized insert remains O(1)                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Load Factor Constraint

```
╔══════════════════════════════════════════════════════════════╗
║  Standard cuckoo (2 tables):                                ║
║    Maximum load factor: ~50% (combined)                     ║
║    Each table should be < 50% full                         ║
║                                                              ║
║  With stash (small overflow area):                          ║
║    Can push to ~90% load factor                             ║
║    Stash of size s → cycle probability drops to O(1/n^s)   ║
║                                                              ║
║  d-ary cuckoo (d hash functions, d tables):                 ║
║    d=3: max load ~91%                                       ║
║    d=4: max load ~97%                                       ║
║    Trade-off: lookup checks d positions instead of 2        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Comparison with Other Schemes

| Feature | Cuckoo | Linear Probing | Chaining |
|---------|:---:|:---:|:---:|
| Lookup worst-case | $O(1)$ | $O(n)$ | $O(n)$ |
| Lookup average | $O(1)$ | $O(1)$ | $O(1)$ |
| Insert amortized | $O(1)$ | $O(1)$ | $O(1)$ |
| Space overhead | ~50% empty | ~30% empty | Pointers |
| Cache friendliness | Moderate | Excellent | Poor |
| Max load factor | 50% | ~70% | Any |
| Implementation | Moderate | Simple | Simple |

---

## Real-World Usage

| System | Why Cuckoo Hashing |
|--------|-------------------|
| Network routers | Guaranteed O(1) IP lookup (latency critical) |
| Cuckoo filters | Space-efficient alternative to Bloom filters |
| Hardware hash tables | Predictable timing for FPGA/ASIC |
| Databases | Lock-free concurrent hash tables |

---

## Quick Revision Question

**Q: Why does cuckoo hashing guarantee $O(1)$ worst-case lookup, while other open addressing schemes can degrade to $O(n)$?**

<details>
<summary>Click to reveal answer</summary>

In cuckoo hashing, every element $x$ is stored at **exactly one of two positions**: $T_1[h_1(x)]$ or $T_2[h_2(x)]$. A lookup checks these two positions and nothing else — always exactly 2 memory accesses regardless of load factor, collisions, or input distribution.

In contrast, **linear probing** may need to scan a long cluster:
- If positions $i, i+1, i+2, \ldots, i+k$ are all occupied, looking up an element that hashes to $i$ requires checking all $k+1$ positions.

**Cuckoo hashing shifts the complexity to insertion:**
- Insert might trigger a chain of evictions
- But this guarantees that every element is always at one of its two designated positions
- The "hard work" happens at insert time so lookup stays fast

This is a **design trade-off**: cuckoo hashing trades insert complexity (amortized $O(1)$ with occasional rehashing) for guaranteed $O(1)$ lookup. This is valuable when:
- Reads vastly outnumber writes
- Worst-case latency matters (real-time systems, network routers)
- You need hard guarantees, not just average-case performance

</details>

---

| [← Count-Min Sketch](03-count-min-sketch.md) | [Next: Perfect Hashing →](05-perfect-hashing.md) |
|:---|---:|
