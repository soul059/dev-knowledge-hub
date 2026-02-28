# Trie Applications

## Overview
Tries power many critical systems in the real world — from the autocomplete in your search bar to IP routing in network infrastructure. This chapter explores the most important applications of tries with concrete examples and diagrams.

---

## Application Map

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    TRIE APPLICATIONS                             │
  ├────────────────────┬─────────────────────────────────────────────┤
  │  📱 Autocomplete   │  Type "pro" → ["program", "project", ...]  │
  │  📝 Spell Check    │  "teh" → Did you mean "the"?               │
  │  🌐 IP Routing     │  Longest prefix match for routing tables    │
  │  🔤 T9 Dictionary  │  Predictive text on old phones              │
  │  🔍 Search Engine  │  Query suggestions as you type              │
  │  📞 Phone Book     │  Find contacts by typing prefix             │
  │  🧬 Bioinformatics │  DNA sequence matching                      │
  │  📦 File Systems   │  Directory path resolution                  │
  │  🏷️ Word Games     │  Scrabble, Boggle word validation           │
  │  ⊕ XOR Problems    │  Maximum XOR pair finding                   │
  └────────────────────┴─────────────────────────────────────────────┘
```

---

## 1. Autocomplete System

The most iconic trie application. As a user types, suggest completions instantly.

```
  User types: "app"

  Trie traversal:
  
  root → a → p → p  ← (reached end of typed prefix)
                 │
       ┌─────┬──┴──┬─────┐
       ▼     ▼     ▼     ▼
      (*)   l     r     o
      "app" │     │     │
            e     o     i
            │     │     │
           (*)  a → c   n
         "apple" │      │
                 h      t
                 │      │
                (*)    (*)
            "approach" "appoint"
  
  Suggestions: ["app", "apple", "appoint", "approach"]
```

### How It Works
```
  FUNCTION autocomplete(prefix, limit=5):
      node = navigateToPrefix(prefix)  // O(P)
      IF node is null: RETURN []
      
      results = []
      priorityDFS(node, prefix, results, limit)  // collect top K
      RETURN results
```

### Real-World Enhancement
Add frequency/priority to rank suggestions:
```
  TrieNode:
      children = {}
      isEnd = false
      frequency = 0     // how popular is this word?
      
  When collecting: sort by frequency, return top K
  "app" typed → "apple" (freq: 5000) > "application" (freq: 3000) > "app" (freq: 200)
```

---

## 2. Spell Checker

Find words similar to a misspelled input by exploring nearby trie paths.

```
  User types: "speling" (misspelled)
  
  Strategy: Allow 1 edit distance (insert, delete, replace)
  
  Trie exploration:
  s → p → e → l → l → i → n → g*    ("spelling" — insert 'l')
  s → p → e → l → i → n → g*        ("speling" — exact, not found)
  s → p → e → l → i → n → k → ...   (explore substitutions)
  
  Suggestion: "spelling" (edit distance = 1)
```

### Algorithm Sketch
```
  FUNCTION spellCheck(word, maxEdits=2):
      results = []
      dfsWithEdits(root, word, 0, "", maxEdits, results)
      RETURN results
  
  FUNCTION dfsWithEdits(node, remaining, pos, current, editsLeft, results):
      IF pos == len(word) AND node.isEnd:
          results.add(current)
      IF editsLeft == 0:
          // exact match only
          normal trie traversal
      ELSE:
          // try insert, delete, replace with editsLeft - 1
```

---

## 3. IP Routing (Longest Prefix Match)

Network routers use tries to find the most specific matching route for an IP address.

```
  Routing Table:
  ┌──────────────────┬──────────────┐
  │ Prefix           │ Next Hop     │
  ├──────────────────┼──────────────┤
  │ 192.168.0.0/16   │ Gateway A    │
  │ 192.168.1.0/24   │ Gateway B    │
  │ 192.168.1.128/25 │ Gateway C    │
  │ 10.0.0.0/8       │ Gateway D    │
  └──────────────────┴──────────────┘

  Binary Trie for IP:
  
  root
  ├── 0... (10.x.x.x)
  │   └── 0001010 → 00000000... → Gateway D
  └── 1... (192.x.x.x)
      └── 10000000 → 10101000...
          ├── 00000000... → Gateway A (/16)
          └── 00000001...
              ├── 0... → Gateway B (/24)
              └── 1... → Gateway C (/25)

  Lookup: 192.168.1.200
  Match path: 1→1→0→0→0→0→0→0 → 1→0→1→0→1→0→0→0 → 0→0→0→0→0→0→0→1 → 1...
  Longest match: 192.168.1.128/25 → Gateway C ✓
```

---

## 4. Search Engine Suggestions

As you type in Google/Bing, the search bar suggests queries:

```
  User types: "how to"
  
  Trie of popular queries:
  
  "how to" → (prefix node)
      ├── "cook pasta"     (freq: 50000)
      ├── "tie a tie"      (freq: 45000)
      ├── "learn python"   (freq: 40000)
      ├── "lose weight"    (freq: 38000)
      └── "make money"     (freq: 35000)
  
  Return top 5 by frequency
```

---

## 5. Dictionary / Word Validation

Used in word games (Scrabble, Boggle, Wordle) to validate words:

```
  Dictionary Trie loaded with ~270,000 English words
  
  Game: Boggle Board
  ┌───┬───┬───┬───┐
  │ G │ I │ Z │ A │
  ├───┼───┼───┼───┤
  │ R │ E │ N │ T │
  ├───┼───┼───┼───┤
  │ A │ D │ O │ S │
  ├───┼───┼───┼───┤
  │ P │ K │ L │ U │
  └───┴───┴───┴───┘
  
  Path: G→R→E→A→T → check trie → "great" ✓ valid!
  Path: G→I→Z      → check trie → "giz" prefix exists? No → PRUNE!
  
  Trie enables PRUNING: stop exploring paths that don't match any word prefix
```

---

## 6. File System / URL Routing

Tries naturally model hierarchical paths:

```
  File System Trie:
  
  root
  ├── usr/
  │   ├── local/
  │   │   ├── bin/
  │   │   └── lib/
  │   └── share/
  ├── etc/
  │   ├── nginx/
  │   └── ssh/
  └── var/
      ├── log/
      └── www/

  URL Router (web frameworks):
  
  root
  ├── api/
  │   ├── users/
  │   │   ├── {id} → getUserById()
  │   │   └── list → listUsers()
  │   └── posts/
  │       └── {id} → getPostById()
  └── static/
      └── * → serveStatic()
```

---

## 7. T9 Predictive Text

Old phone keyboards mapped digits to letters. Trie enables predictions:

```
  Phone keypad:
  2 → ABC    5 → JKL    8 → TUV
  3 → DEF    6 → MNO    9 → WXYZ
  4 → GHI    7 → PQRS

  User presses: 4-6-6-3
  
  Possible letters:    G/H/I → M/N/O → M/N/O → D/E/F
  
  Trie lookup for all combinations:
  "good" ✓  "gone" ✓  "home" ✓  "hood" ✓  "hone" ✓
  
  Show most frequent: "good", "home", "gone"
```

---

## 8. DNA Sequence Analysis

Bioinformatics uses tries over alphabet {A, C, G, T}:

```
  DNA Sequences: ["ACGT", "ACGA", "ACTT", "TGCA"]
  
  Trie (alphabet = {A, C, G, T}):
  
      (root)
      /    \
     A      T
     │      │
     C      G
     │      │
     G      C
    / \     │
   T*  A*   A*
   
  Query: Find all sequences starting with "AC"
  → Navigate to 'C' under 'A'
  → Collect: "ACGT", "ACGA", "ACTT"
  
  Each node has only 4 possible children → very memory efficient!
```

---

## 9. XOR Optimization (Competitive Programming)

Binary tries solve maximum XOR problems efficiently:

```
  Problem: Find two numbers whose XOR is maximum
  Numbers: [3, 10, 5, 25]
  
  Binary representations (5 bits):
  3  = 00011
  10 = 01010
  5  = 00101
  25 = 11001
  
  Binary Trie:
        (root)
       /     \
      0       1
     / \       \
    0   1       1
    |   |       |
    0   0       0
   / \  |       |
  1   1 1       0
  |   | |       |
  1   0 0       1
  3   5 10     25
  
  For each number, traverse trie choosing OPPOSITE bits
  → Maximizes XOR
  → Answer: 5 XOR 25 = 28
```

---

## Summary Table

| Application | Why Trie? | Key Benefit |
|-------------|-----------|-------------|
| Autocomplete | Prefix search | O(P + K) suggestions |
| Spell check | Edit distance exploration | Explore similar paths |
| IP routing | Longest prefix match | Binary trie on IP bits |
| Search suggestions | Ranked prefix results | Frequency-based ranking |
| Word games | Validation + pruning | Early termination |
| File systems | Hierarchical paths | Natural tree structure |
| T9 predictive text | Multi-character prefix | Branch on digit groups |
| DNA analysis | Small alphabet (4 chars) | Memory efficient |
| XOR optimization | Binary trie | Greedy bit selection |

---

## Quick Revision Questions

1. **How does autocomplete use a trie?**
   > Navigate to the typed prefix, then DFS the subtree to collect all completions. Rank by frequency for best results.

2. **What makes tries ideal for IP routing?**
   > IP addresses are bit strings. A binary trie enables longest prefix match in O(B) time (B = bits) — critical for routing speed.

3. **How do tries help in Boggle/Scrabble?**
   > Check if current path is a valid prefix. If not, prune entire branch early — massively reduces search space.

4. **Why are tries used in spell checkers instead of hash tables?**
   > Tries can explore similar words (1-2 edits away) by traversing nearby trie branches. Hash tables can't navigate to "similar" keys.

5. **What alphabet size does a DNA trie use?**
   > 4 characters: A, C, G, T. Each node has at most 4 children — very compact.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Prefix Matching Concept](04-prefix-matching-concept.md) | [Next: When to Use Tries ▶](06-when-to-use-tries.md) |
