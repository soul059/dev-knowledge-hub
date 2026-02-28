# Chapter 7.3 — Spell Checker Concept

> **Unit 7: Trie Applications** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A spell checker combines a trie dictionary with **edit distance** to find
words similar to a misspelled input. This chapter covers the concept and
how tries make spell checking efficient.

---

## 1. Spell Checker Pipeline

```
  ┌───────────┐     ┌──────────────┐     ┌──────────────┐
  │ User types│────→│ Check if in  │────→│  If not:     │
  │ "helo"    │     │ dictionary   │     │  Find close  │
  └───────────┘     └──────────────┘     │  matches     │
                          │              └──────┬───────┘
                     In dict? ───Yes───→ OK ✓   │
                          │                     │
                         No                     ▼
                          ↓             ┌──────────────┐
                    Flag as error  ←──→│ Suggestions: │
                                       │ "hello"      │
                                       │ "help"       │
                                       │ "hero"       │
                                       └──────────────┘
```

---

## 2. Edit Distance (Levenshtein Distance)

```
  The number of single-character operations to transform
  one string into another:

  Operations:
  1. INSERT:    "hlo" → "helo"      (insert 'e')
  2. DELETE:    "hello" → "helo"    (delete 'l')
  3. REPLACE:   "helo" → "hero"    (replace 'l' with 'r')

  edit_distance("helo", "hello") = 1  (insert 'l')
  edit_distance("helo", "hero")  = 1  (replace 'l' with 'r')
  edit_distance("helo", "world") = 4  (many changes)

  Common threshold: suggest words with edit distance ≤ 2
```

---

## 3. Trie-Based Spell Checking

```
  Naive approach:
  ─────────────
  Compute edit distance between input and EVERY dictionary word.
  Time: O(D × m × n) where D = dictionary size

  Trie approach:
  ──────────────
  DFS through trie, track edit distance as we traverse.
  PRUNE branches where edit distance > threshold.
  Huge speedup in practice!

  ┌─────────────────────────────────────────────────┐
  │  Key insight: As we traverse deeper into trie,  │
  │  we can compute partial edit distance and PRUNE  │
  │  early if it exceeds the threshold.             │
  └─────────────────────────────────────────────────┘
```

---

## 4. Algorithm: Fuzzy Search in Trie

```
function FuzzySearch(node, word, currentRow, results, maxDist):
    // currentRow = last row of edit distance DP matrix

    if node.is_end:
        if currentRow[len(word)] ≤ maxDist:
            add node's word to results

    for each child (letter, childNode) of node:
        // Build next row of DP matrix
        nextRow[0] = currentRow[0] + 1   // deletion from trie word

        for i = 1 to len(word):
            insertCost  = nextRow[i-1] + 1
            deleteCost  = currentRow[i] + 1
            replaceCost = currentRow[i-1] + (0 if letter == word[i-1] else 1)
            nextRow[i] = min(insertCost, deleteCost, replaceCost)

        // PRUNE: if minimum of nextRow > maxDist, skip this branch!
        if min(nextRow) ≤ maxDist:
            FuzzySearch(childNode, word, nextRow, results, maxDist)
```

---

## 5. Example Trace

```
  Dictionary: {"hello", "help", "hero", "heap"}
  Query: "helo"  maxDist: 1

  Edit distance DP rows (building incrementally):

  Initial row (root): [0, 1, 2, 3, 4]  (for "helo")

  Branch 'h':
    [1, 0, 1, 2, 3]   min=0 ≤ 1 ✓ continue

    Branch 'e':
      [2, 1, 0, 1, 2]   min=0 ≤ 1 ✓ continue

      Branch 'l':
        [3, 2, 1, 0, 1]   min=0 ≤ 1 ✓ continue

        Branch 'l':
          [4, 3, 2, 1, 1]   "hell" not end, continue
          Branch 'o':
            [5, 4, 3, 2, 1]   "hello" is end! dist=1 ≤ 1 ✓ → SUGGEST

        Branch 'p':
          [4, 3, 2, 1, 1]   "help" is end! dist=1 ≤ 1 ✓ → SUGGEST

      Branch 'r':
        [3, 2, 1, 1, 1]   min=1 ≤ 1 ✓ continue
        Branch 'o':
          [4, 3, 2, 2, 1]   "hero" is end! dist=1 ≤ 1 ✓ → SUGGEST

      Branch 'a':
        [3, 2, 1, 1, 2]   min=1 ≤ 1 ✓ continue
        Branch 'p':
          [4, 3, 2, 2, 2]   "heap" is end! dist=2 > 1 ✗ → SKIP

  Suggestions: ["hello", "help", "hero"]
```

---

## 6. Implementation

```python
def spell_check(trie_root, word, max_dist=2):
    """Find all dictionary words within max_dist edits of word."""
    results = []
    initial_row = list(range(len(word) + 1))
    
    for letter, child in trie_root.children.items():
        _search_recursive(child, letter, word, initial_row, results, max_dist, letter)
    
    return results

def _search_recursive(node, letter, word, prev_row, results, max_dist, current_word):
    n = len(word)
    current_row = [prev_row[0] + 1]
    
    for i in range(1, n + 1):
        insert_cost = current_row[i - 1] + 1
        delete_cost = prev_row[i] + 1
        replace_cost = prev_row[i - 1] + (0 if word[i - 1] == letter else 1)
        current_row.append(min(insert_cost, delete_cost, replace_cost))
    
    # If last element ≤ max_dist AND end of word → add to results
    if current_row[-1] <= max_dist and node.is_end:
        results.append((current_word, current_row[-1]))
    
    # If any element in current_row ≤ max_dist → continue searching
    if min(current_row) <= max_dist:
        for next_letter, child in node.children.items():
            _search_recursive(child, next_letter, word, current_row, 
                            results, max_dist, current_word + next_letter)
```

---

## 7. Performance

```
  Without pruning: O(D × m × n) — check every dictionary word
  With trie + pruning: Much faster in practice

  Why pruning helps:
  ┌───────────────────────────────────────────────────────┐
  │  "zzzz" with max_dist=1:                            │
  │  Most trie branches get pruned at depth 1 or 2.      │
  │  Only words starting with 'z', 'y'(replace), etc.    │
  │  are explored.                                       │
  │                                                       │
  │  Typical speedup: 10-100× over naive approach        │
  └───────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Component | Details |
|-----------|---------|
| Dictionary storage | Trie |
| Similarity metric | Edit distance (Levenshtein) |
| Key technique | DFS with incremental DP + pruning |
| Threshold | Usually max_dist = 1 or 2 |
| Pruning condition | min(current_row) > max_dist → stop |
| Time (practical) | Much better than checking all words |
| Applications | IDE spelling, search engines, autocorrect |

---

## ❓ Quick Revision Questions

1. **What are the three edit operations?**
   <details><summary>Answer</summary>Insert (add a character), delete (remove a character), and replace (change one character to another). Each counts as distance 1.</details>

2. **How does the trie-based approach prune the search space?**
   <details><summary>Answer</summary>While traversing the trie, we incrementally compute edit distance DP rows. If the minimum value in the current row exceeds max_dist, we prune that entire subtree — no word in that branch can be within the threshold.</details>

3. **What is stored in each row of the incremental DP?**
   <details><summary>Answer</summary>Row i represents the edit distance between the trie word prefix (up to current depth) and each prefix of the query word. The last element is the full edit distance.</details>

4. **Why is edit distance ≤ 2 a common threshold?**
   <details><summary>Answer</summary>Most real typos are within 1-2 edits (mistyped key, transposition, missing letter). Distance > 2 produces too many irrelevant suggestions.</details>

5. **What is the time complexity improvement from pruning?**
   <details><summary>Answer</summary>Worst case is still exponential in max_dist, but in practice, pruning eliminates most branches early, giving 10-100× speedup over checking every dictionary word.</details>

---

| [⬅️ Previous: Autocomplete](02-autocomplete.md) | [Next: Word Dictionary ➡️](04-word-dictionary.md) |
|:---|---:|
