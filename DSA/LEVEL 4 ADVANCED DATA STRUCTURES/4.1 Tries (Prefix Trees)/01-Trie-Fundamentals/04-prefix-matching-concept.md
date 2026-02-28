# Prefix Matching Concept

## Overview
The defining superpower of a trie is **prefix matching** — the ability to find all strings that start with a given prefix in O(P) time (where P is the prefix length), regardless of how many total words are stored. This chapter explains the concept, the algorithm, and traces through detailed examples.

---

## What is Prefix Matching?

> Given a set of words and a prefix string, find all words that start with that prefix.

```
  Dictionary: ["the", "their", "them", "there", "thermal", "thin", "this"]
  
  Query: prefix = "the"
  Result: ["the", "their", "them", "there", "thermal"]
  
  Query: prefix = "thi"
  Result: ["thin", "this"]
  
  Query: prefix = "xyz"
  Result: [] (no words start with "xyz")
```

---

## How Prefix Matching Works in a Trie

### Phase 1: Navigate to the Prefix End

Follow the prefix character by character from the root. If any character is missing, the prefix doesn't exist.

### Phase 2: Collect All Words Below

From the prefix end node, do a DFS to gather every word in the subtree.

```
  Trie: ["the", "their", "them", "there", "thermal", "thin"]

                    (root)
                      │
                      t
                      │
                      h
                    /   \
                   e     i
                 / | \     \
                *  i  m*    n*
                   │
                   r
                  / \
                 *    m
                 │    │
                 e    a
                      │
                      l*

  Query: prefix = "the"
  
  ┌──────────────────────────────────────────────────────────┐
  │ PHASE 1: Navigate to prefix end                          │
  │                                                          │
  │ Step 1: root → 't' ✓ (found child 't')                  │
  │ Step 2:  't' → 'h' ✓ (found child 'h')                  │
  │ Step 3:  'h' → 'e' ✓ (found child 'e')                  │
  │                                                          │
  │ ✅ Reached prefix end at node 'e'                        │
  ├──────────────────────────────────────────────────────────┤
  │ PHASE 2: Collect all words from 'e' subtree              │
  │                                                          │
  │ DFS from 'e':                                            │
  │   'e' → isEnd=true → collect "the" ✓                     │
  │   'e' → 'i' → 'r':                                      │
  │     'r' → isEnd=true → collect "their" ✓                 │
  │     'r' → 'e' (no end)                                   │
  │     'r' → 'm' → 'a' → 'l' → isEnd=true                 │
  │                   → collect "thermal" ✓                   │
  │   'e' → 'm' → isEnd=true → collect "them" ✓             │
  │                                                          │
  │ Result: ["the", "their", "thermal", "them"]              │
  └──────────────────────────────────────────────────────────┘
```

---

## The Algorithm

```
FUNCTION findAllWithPrefix(prefix):
    node = root
    
    // Phase 1: Navigate to prefix end
    FOR each char c in prefix:
        IF c NOT in node.children:
            RETURN empty list    // prefix doesn't exist
        node = node.children[c]
    
    // Phase 2: Collect all words from this subtree
    results = []
    collectWords(node, prefix, results)
    RETURN results

FUNCTION collectWords(node, currentWord, results):
    IF node.isEndOfWord:
        results.add(currentWord)
    FOR each (char, childNode) in node.children:
        collectWords(childNode, currentWord + char, results)
```

---

## Step-by-Step Trace

### Example: Words = ["car", "card", "care", "carp", "cat"]

```
  Build the trie:

           (root)
             │
             c
             │
             a
            / \
           r    t*
          /|\
         d* e* p*
```

### Query: prefix = "car"

```
  Phase 1: Navigate
  ┌────────┬──────────┬─────────┐
  │ Step   │ Char     │ Result  │
  ├────────┼──────────┼─────────┤
  │ 1      │ 'c'      │ Found ✓ │
  │ 2      │ 'a'      │ Found ✓ │
  │ 3      │ 'r'      │ Found ✓ │
  └────────┴──────────┴─────────┘
  → Arrived at node 'r'

  Phase 2: DFS from 'r'
  ┌──────────────────────────────────────────┐
  │ Visit 'r': isEnd=false, skip            │
  │ Visit 'r' → 'd': isEnd=true            │
  │   → Add "card" ✓                        │
  │ Visit 'r' → 'e': isEnd=true            │
  │   → Add "care" ✓                        │
  │ Visit 'r' → 'p': isEnd=true            │
  │   → Add "carp" ✓                        │
  └──────────────────────────────────────────┘

  Result: ["card", "care", "carp"]
```

### Query: prefix = "ca"

```
  Phase 1: Navigate to 'a' node
  Phase 2: DFS from 'a' subtree
  → "car" (if r were marked), "card", "care", "carp", "cat"
  
  Result: ["card", "care", "carp", "cat"]
```

### Query: prefix = "co"

```
  Phase 1: root → 'c' ✓ → 'o' ✗
  Node 'c' has no child 'o'
  
  Result: [] (empty — no words start with "co")
```

---

## Why This is Powerful

```
  ┌─────────────────────────────────────────────────────────────┐
  │  COMPARISON: 1 million words, find all starting with "pre" │
  ├─────────────────────┬───────────────────────────────────────┤
  │  BRUTE FORCE        │  Check every word:                    │
  │  (Array/HashSet)    │  for w in words:                      │
  │                     │      if w.startsWith("pre") → add    │
  │                     │  Time: O(1,000,000 × L)               │
  │                     │  Touches: ALL words                   │
  ├─────────────────────┼───────────────────────────────────────┤
  │  TRIE               │  Navigate: root → p → r → e          │
  │                     │  Then DFS subtree                     │
  │                     │  Time: O(3 + K)   [K = matching]     │
  │                     │  Touches: ONLY matching words         │
  └─────────────────────┴───────────────────────────────────────┘
```

### Complexity Table

| Operation                         | Trie        | Hash Table | Array       |
|-----------------------------------|:-----------:|:----------:|:-----------:|
| Find all words with prefix "pre"  | **O(P + K)**| O(N × L)   | O(N × L)   |
| Check if any word has prefix      | **O(P)**    | O(N × L)   | O(N × L)   |
| Count words with prefix           | **O(P)**    | O(N × L)   | O(N × L)   |

> P = prefix length, K = total characters in matching words, N = total words, L = avg word length

---

## Prefix Exists vs. Word Exists

These are fundamentally different queries:

```
  Trie: ["apple", "app", "application"]

              (root)
                │
                a
                │
                p
                │
                p*          ← "app" ends here
                │
                l
               / \
              e*   i        ← "apple" ends at 'e'
                   │
                   c → a → t → i → o → n*

  ┌─────────────────────────────────────────────────────────┐
  │ Query              │ Type       │ Check      │ Result   │
  ├────────────────────┼────────────┼────────────┼──────────┤
  │ search("app")      │ Word exist │ isEnd?     │ true ✓   │
  │ search("appl")     │ Word exist │ isEnd?     │ false ✗  │
  │ startsWith("app")  │ Prefix     │ Path exist │ true ✓   │
  │ startsWith("appl") │ Prefix     │ Path exist │ true ✓   │
  │ startsWith("appx") │ Prefix     │ Path exist │ false ✗  │
  └─────────────────────────────────────────────────────────┘
```

Key difference:
- `search`: navigates path AND checks `isEndOfWord`
- `startsWith`: ONLY checks if path exists (no end check)

---

## Prefix Count with prefixCount Field

To instantly answer "How many words start with this prefix?", maintain a `prefixCount` at each node:

```
  Insert "apple", "app", "application", "apt"

      (root)
        │
        a  [pCnt=4]      ← 4 words pass through 'a'
        │
        p  [pCnt=4]      ← 4 words pass through 'p' 
        │
        p* [pCnt=3]      ← 3 words pass through second 'p'
       / \
      l    t* [pCnt=1]   ← 1 word passes here ("apt")
      │
      e* [pCnt=2]        ← 2 words pass here
      │
      ... (application continues)

  countWordsStartingWith("ap")  → prefixCount at 'p' = 4
  countWordsStartingWith("app") → prefixCount at second 'p' = 3
  countWordsStartingWith("apt") → prefixCount at 't' = 1
```

---

## Real-World Analogy

```
  Think of a DICTIONARY organized by prefix:

  ┌─────────────────────────────────────────────┐
  │  A Section                                   │
  │  ├── AB Section                              │
  │  │   ├── ABA: abacus, abandon               │
  │  │   ├── ABB: abbey, abbreviate              │
  │  │   └── ABS: abs, absent, absorb            │
  │  ├── AC Section                              │
  │  │   ├── ACC: accept, account                │
  │  │   └── ACT: act, action, active            │
  │  └── AD Section                              │
  │      └── ADD: add, address                   │
  └─────────────────────────────────────────────┘

  Finding all words starting with "AB":
  → Go to A section → go to AB section → list all words
  → No need to look at AC, AD, B, C, ... sections!
  
  This is EXACTLY how a trie works!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Prefix matching | Navigate to prefix end, then collect subtree |
| Phase 1 | Follow prefix characters from root: O(P) |
| Phase 2 | DFS to collect all matching words: O(K) |
| Total time | O(P + K) where K = chars in results |
| Prefix exists | Only check if path exists (no isEnd check) |
| Word exists | Check path exists AND isEndOfWord = true |
| vs brute force | Trie skips non-matching words entirely |

---

## Quick Revision Questions

1. **What are the two phases of prefix matching in a trie?**
   > Phase 1: Navigate to the prefix end node. Phase 2: DFS from that node to collect all words in the subtree.

2. **What happens if we search for prefix "xyz" and 'x' has no child 'y'?**
   > We return an empty list immediately in Phase 1 — the prefix doesn't exist in the trie.

3. **What's the time complexity of finding all words with prefix of length P?**
   > O(P + K), where K is the total characters in all matching words. P for navigation, K for collection.

4. **How does `startsWith("app")` differ from `search("app")`?**
   > `startsWith` only checks if the path a→p→p exists. `search` additionally requires isEndOfWord = true at the last node.

5. **How can you instantly count words with a given prefix without collecting them?**
   > Maintain a `prefixCount` field at each node, incremented during insertion. Navigate to prefix end and return `prefixCount`.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Trie Node Structure](03-trie-node-structure.md) | [Next: Trie Applications ▶](05-trie-applications.md) |
