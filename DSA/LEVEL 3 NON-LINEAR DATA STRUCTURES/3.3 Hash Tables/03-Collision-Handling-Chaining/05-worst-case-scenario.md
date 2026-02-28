# 3.5 Worst Case Scenario

## Unit 3: Collision Handling - Chaining

---

## When Hashing Goes Wrong

The worst case for chaining occurs when **all keys hash to the same bucket**, creating a single chain of length $n$. This degrades the hash table to a linked list.

```
╔═══════════════════════════════════════════════════════════════╗
║              WORST CASE: EVERYTHING IN ONE BUCKET            ║
║                                                              ║
║   n = 8 keys, m = 8 buckets, ALL hash to index 0           ║
║                                                              ║
║   ┌───┐                                                     ║
║   │ 0 │──► [K1]──►[K2]──►[K3]──►[K4]──►[K5]──►[K6]──►... ║
║   ├───┤                                                     ║
║   │ 1 │──► null                                             ║
║   ├───┤                                                     ║
║   │ 2 │──► null                                             ║
║   ├───┤                                                     ║
║   │ 3 │──► null                                             ║
║   ├───┤         7 of 8 buckets completely wasted!           ║
║   │ 4 │──► null                                             ║
║   ├───┤                                                     ║
║   │ 5 │──► null                                             ║
║   ├───┤                                                     ║
║   │ 6 │──► null                                             ║
║   ├───┤                                                     ║
║   │ 7 │──► null                                             ║
║   └───┘                                                     ║
║                                                              ║
║   Search = O(n)   ←── Same as unsorted linked list!         ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## How Worst Case Happens

```
╔═══════════════════════════════════════════════════════════════╗
║              CAUSES OF WORST CASE                            ║
║                                                              ║
║   1. BAD HASH FUNCTION                                       ║
║      h(k) = 0 for all k     → everything to bucket 0       ║
║      h(k) = k mod 2         → only 2 buckets used          ║
║                                                              ║
║   2. ADVERSARIAL INPUT                                       ║
║      Attacker knows h(k) = k mod 7                          ║
║      Sends keys: 7, 14, 21, 28, 35, ... → all to bucket 0  ║
║                                                              ║
║   3. PATHOLOGICAL DATA PATTERNS                              ║
║      Sequential IDs with bad table size:                     ║
║      h(k) = k mod 10, keys: 10, 20, 30, 40, ...            ║
║      All land in bucket 0!                                   ║
║                                                              ║
║   4. HASH FLOODING ATTACK (HashDoS)                          ║
║      Send specially crafted HTTP parameters that             ║
║      all hash to same bucket → server DoS                   ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Worst Case Complexity

| Operation | Average Case | **Worst Case** |
|-----------|:------------:|:--------------:|
| Insert | O(1) | **O(n)** (if checking for duplicates) |
| Search | O(1 + α) | **O(n)** |
| Delete | O(1 + α) | **O(n)** |

```
Worst case search in chain of length n:

Chain: [K1] → [K2] → [K3] → ... → [Kn] → null

Searching for Kn (last element):
  Compare K1: ✗ (1)
  Compare K2: ✗ (2)
  Compare K3: ✗ (3)
  ...
  Compare Kn: ✓ (n)
  
Total: n comparisons = O(n)

Searching for non-existent key:
  Must traverse ALL n nodes + reach null
  Total: n comparisons = O(n)
```

---

## Mitigations

```
╔═══════════════════════════════════════════════════════════════╗
║              PREVENTING WORST CASE                           ║
║                                                              ║
║   1. UNIVERSAL HASHING                                       ║
║      Random hash function → adversary can't predict         ║
║      Expected O(1) for ANY input set                        ║
║                                                              ║
║   2. BALANCED BST CHAINS (Java 8+)                          ║
║      When chain length > 8 → convert to Red-Black tree     ║
║      Worst case: O(log n) instead of O(n)                   ║
║                                                              ║
║      Before (long chain):                                    ║
║      [a]→[b]→[c]→[d]→[e]→[f]→[g]→[h]→[i]                ║
║                                                              ║
║      After (balanced tree):                                  ║
║                  [e]                                         ║
║                /     \                                       ║
║             [c]       [g]                                    ║
║            / \       / \                                     ║
║          [b] [d]   [f] [h]                                  ║
║         /               \                                    ║
║       [a]                [i]                                 ║
║                                                              ║
║   3. HASH RANDOMIZATION                                     ║
║      Python 3.3+: randomize hash seed per process           ║
║      Prevents cross-request attack patterns                  ║
║                                                              ║
║   4. LOAD FACTOR MANAGEMENT                                 ║
║      Keep α low → resize aggressively                       ║
║      Shorter chains even in worst case                      ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## The Birthday Problem and Expected Maximum Chain

Even with a perfect hash function, some chains will be longer than average:

```
Expected maximum chain length with n keys, m = n buckets:

E[max chain] = Θ(log n / log log n)

n = 100:    max chain ≈ 4-5
n = 1000:   max chain ≈ 5-6
n = 10000:  max chain ≈ 6-7
n = 1000000: max chain ≈ 7-8

Even in the ideal case, some chains will be longer!
But still O(log n / log log n) = much better than O(n).
```

---

## Quick Revision Question

**Q: Java 8 converts HashMap chains to balanced trees when chain length exceeds 8. What advantage does this provide? What is the new worst-case time complexity?**

<details>
<summary>Click to reveal answer</summary>

**Advantage**: When many keys collide at the same bucket (due to a poor hash function or adversarial input), a linked list chain requires O(n) time for search. By converting to a balanced Red-Black tree, the worst-case search in that bucket becomes **O(log n)**.

**New complexities**:
- Search worst case: **O(log n)** instead of O(n)
- Insert worst case: **O(log n)** instead of O(n) (tree insertion)
- This caps the damage from hash collisions, making hash table DoS attacks significantly less effective.

The trade-off is increased memory per node (tree nodes need left, right, parent, color pointers vs. just a next pointer) and the complexity of tree operations. That's why Java only converts when chains exceed 8 elements and converts back to a list when they shrink below 6.

</details>

---

| [← Previous: Average Case Analysis](04-average-case-analysis.md) | [Next: Implementation Details →](06-implementation-details.md) |
|:---|---:|
