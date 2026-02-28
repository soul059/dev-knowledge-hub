# Chapter 10.4 — Aho-Corasick Algorithm

> **Unit 10: Advanced String Concepts** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Aho-Corasick** is the gold standard for **multi-pattern matching** —
finding all occurrences of multiple patterns simultaneously in a text.
It builds an automaton combining a trie with failure links (like KMP)
to achieve O(n + m + z) time, where z is the number of matches.

---

## 1. Problem: Multi-Pattern Matching

```
  Given: Text T and a set of patterns {P₁, P₂, ..., Pₖ}
  Find:  All occurrences of any pattern in T.

  Example:
    T = "ahishers"
    Patterns = {"he", "she", "his", "hers"}

  Matches:
    "his" at position 1
    "he"  at position 4  (inside "shers")
    "she" at position 3
    "hers" at position 4

  Naive: Search each pattern separately → O(n × k × max_m)
  Aho-Corasick: O(n + total_pattern_length + matches) — linear!
```

---

## 2. Three Construction Steps

```
  Step 1: Build a TRIE from all patterns
  ─────────────────────────────────────────
        (root)
       /    \
      h      s
      |      |
      e*     h
     / \     |
    r   i    e*
    |   |
    s*  s*

  * marks end of a pattern

  Step 2: Add FAILURE LINKS (like KMP failure function)
  ─────────────────────────────────────────────────────
  For each node, failure link → longest proper suffix
  that is also a prefix of some pattern.

  ┌────────────────────────────────────────────────────┐
  │  Failure link of node X:                           │
  │  → Where to go when text doesn't match any child   │
  │  → Points to longest suffix of current path that   │
  │    is a prefix in the trie                         │
  └────────────────────────────────────────────────────┘

  Example failure links:
    "she" → after reading "sh":
      failure of 'h' (under 's') → 'h' (under root)
      Because "h" is suffix of "sh" and prefix in trie.

    "she" → after reading "she":
      failure of 'e' (under 'sh') → 'e' (under 'h')
      Because "he" is suffix of "she" and exists in trie.

  Step 3: Add OUTPUT LINKS (dictionary suffix links)
  ─────────────────────────────────────────────────────
  Output link → nearest node reachable via failure links
  that marks end of a pattern.

  Example: When at 'e' in "she":
    'e' itself is end of "she" ✓
    failure → 'e' (in "he") is end of "he" → output link!
    → Report BOTH "she" and "he".
```

---

## 3. Failure Link Construction (BFS)

```
  Build failure links level by level (BFS from root):

  Depth 0 (root):   failure → root
  Depth 1 (h, s):   failure → root (single chars)
  Depth 2:
    'e' under 'h':  failure → root (no suffix "e" as prefix)
    'h' under 's':  failure → 'h' under root! ("h" is prefix)
    'i' under 'h':  failure → root
  Depth 3:
    'r' under "he": failure → root
    's' under "hi": failure → 's' under root
    'e' under "sh": failure → 'e' under "h"! ("he" match!)
  Depth 4:
    's' under "her": failure → 's' under root

  Algorithm: BFS
  for each node u at depth d:
      for each child (char c, node v) of u:
          f = failure[u]
          while f ≠ root AND c not in f.children:
              f = failure[f]
          failure[v] = f.children[c] if c in f.children else root
```

---

## 4. Search Phase

```
  Process text character by character, follow automaton:

  T = "ahishers"

  State trace:
  ┌────┬──────┬──────────┬────────────────┐
  │ Ch │ Node │ Matches  │ Notes          │
  ├────┼──────┼──────────┼────────────────┤
  │ a  │ root │          │ no 'a' child   │
  │ h  │ h    │          │ follow 'h'     │
  │ i  │ hi   │          │ follow 'i'     │
  │ s  │ his  │ "his"✓   │ his is end     │
  │    │ →s   │          │ failure→s      │
  │ h  │ sh   │          │ follow 'h'     │
  │ e  │ she  │ "she"✓   │ she is end     │
  │    │      │ "he"✓    │ output link!   │
  │ r  │ →her │          │ failure→he→r   │
  │ s  │ hers │ "hers"✓  │ hers is end    │
  └────┴──────┴──────────┴────────────────┘

  Total matches: "his"@1, "she"@3, "he"@4, "hers"@4
```

---

## 5. Implementation

```python
from collections import deque, defaultdict

class AhoCorasick:
    def __init__(self):
        self.goto = [{}]     # goto[state][char] → next state
        self.fail = [0]      # failure links
        self.output = [[]]   # output[state] = list of pattern indices
        self.state_count = 1
    
    def _new_state(self):
        self.goto.append({})
        self.fail.append(0)
        self.output.append([])
        s = self.state_count
        self.state_count += 1
        return s
    
    def add_pattern(self, pattern, index):
        """Add pattern to the automaton."""
        state = 0
        for ch in pattern:
            if ch not in self.goto[state]:
                self.goto[state][ch] = self._new_state()
            state = self.goto[state][ch]
        self.output[state].append(index)
    
    def build(self):
        """Build failure links using BFS."""
        queue = deque()
        
        # Depth-1 nodes: failure → root
        for ch, s in self.goto[0].items():
            self.fail[s] = 0
            queue.append(s)
        
        # BFS
        while queue:
            u = queue.popleft()
            for ch, v in self.goto[u].items():
                queue.append(v)
                f = self.fail[u]
                while f and ch not in self.goto[f]:
                    f = self.fail[f]
                self.fail[v] = self.goto[f].get(ch, 0)
                if self.fail[v] == v:
                    self.fail[v] = 0
                # Merge output
                self.output[v] = self.output[v] + self.output[self.fail[v]]
    
    def search(self, text, patterns):
        """Find all pattern occurrences in text."""
        results = []
        state = 0
        
        for i, ch in enumerate(text):
            while state and ch not in self.goto[state]:
                state = self.fail[state]
            state = self.goto[state].get(ch, 0)
            
            for pat_idx in self.output[state]:
                pat = patterns[pat_idx]
                results.append((i - len(pat) + 1, pat))
        
        return results

# Usage:
ac = AhoCorasick()
patterns = ["he", "she", "his", "hers"]
for i, p in enumerate(patterns):
    ac.add_pattern(p, i)
ac.build()
print(ac.search("ahishers", patterns))
# [(1, 'his'), (3, 'she'), (4, 'he'), (4, 'hers')]
```

---

## 6. Complexity Analysis

```
  Let:
    n = text length
    m = total length of all patterns (Σ|Pᵢ|)
    k = number of patterns
    z = number of matches (output)

  ┌──────────────────────�──────────────────────────────┐
  │ Phase              │ Time                          │
  ├──────────────────────┼──────────────────────────────┤
  │ Build trie         │ O(m)                          │
  │ Build failure links│ O(m × |Σ|) or O(m) amortized │
  │ Search             │ O(n + z)                      │
  │ Total              │ O(m + n + z)                  │
  └──────────────────────┴──────────────────────────────┘

  Space: O(m × |Σ|)  for the goto function
         O(m) for failure links and output lists

  Comparison:
  ┌───────────────────────┬────────────────────┐
  │ Algorithm             │ Multi-pattern time │
  ├───────────────────────┼────────────────────┤
  │ Naive (each pattern)  │ O(n × k × max_m)  │
  │ KMP (each pattern)    │ O((n + max_m) × k) │
  │ Rabin-Karp (hashing)  │ O(n × k) avg       │
  │ Aho-Corasick          │ O(n + m + z) !!    │
  └───────────────────────┴────────────────────┘
```

---

## 7. Applications

```
  ┌──────────────────────────────────────────────────────────┐
  │  1. Network intrusion detection (Snort, Suricata)       │
  │     Scan packet data against thousands of signatures     │
  │                                                          │
  │  2. Virus scanning                                       │
  │     Match file content against malware signature DB      │
  │                                                          │
  │  3. Text filtering / content moderation                  │
  │     Check against banned word lists                      │
  │                                                          │
  │  4. Bioinformatics                                       │
  │     Search for multiple DNA motifs simultaneously        │
  │                                                          │
  │  5. Competitive programming                              │
  │     Count pattern occurrences, string DP on automaton    │
  └──────────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Component | Role | Time to Build |
|-----------|------|---------------|
| Trie (goto) | Pattern prefix matching | O(m) |
| Failure links | Fall back on mismatch | O(m) |
| Output links | Report all matches at state | O(m) |
| Search | Process text through automaton | O(n + z) |
| Total | Build + Search | O(m + n + z) |

---

## ❓ Quick Revision Questions

1. **What problem does Aho-Corasick solve?**
   <details><summary>Answer</summary>Multi-pattern string matching: given a text and multiple patterns, find all occurrences of any pattern in the text in O(n + m + z) time, where z is the number of matches.</details>

2. **What is a failure link?**
   <details><summary>Answer</summary>A failure link at node X points to the node representing the longest proper suffix of X's path label that is also a prefix of some pattern in the trie. It tells the automaton where to go on mismatch, similar to KMP's failure function.</details>

3. **What are output (dictionary suffix) links?**
   <details><summary>Answer</summary>Output links merge match information from failure chain nodes. When reaching a state, all patterns ending at that state AND all patterns reachable via failure links are reported. This handles overlapping patterns.</details>

4. **How does Aho-Corasick compare to running KMP for each pattern?**
   <details><summary>Answer</summary>KMP for k patterns: O(k × (n + m_i)) ≈ O(kn). Aho-Corasick: O(n + m + z), independent of k for the search phase. Massive speedup when k is large.</details>

5. **Why are failure links built using BFS?**
   <details><summary>Answer</summary>BFS ensures we process nodes level by level (increasing depth). Computing failure[v] requires failure[parent(v)] to be already computed, which BFS guarantees since parents are at shallower depth.</details>

---

| [⬅️ Previous: Burrows-Wheeler Transform](03-burrows-wheeler-transform.md) | [Next: Applications in Bioinformatics ➡️](05-applications-in-bioinformatics.md) |
|:---|---:|
