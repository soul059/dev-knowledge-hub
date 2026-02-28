# Chapter 2.3 — Pattern Matching Applications

> **Unit 2: Pattern Matching** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Pattern matching is not just an academic exercise — it underpins many critical
real-world systems. This chapter surveys the applications to show why
mastering these algorithms is so valuable.

---

## 1. Text Editors & IDEs

```
  ┌──────────────────────────────────────────────┐
  │  Find & Replace in VS Code / Vim / Notepad++ │
  │                                              │
  │  Ctrl+F → "search pattern"                   │
  │                                              │
  │  ┌────────────────────────────────┐          │
  │  │ Line 42: ... the pattern ...  │ ← hit 1  │
  │  │ Line 87: ... the pattern ...  │ ← hit 2  │
  │  │ Line 153:... the pattern ...  │ ← hit 3  │
  │  └────────────────────────────────┘          │
  │                                              │
  │  Algorithm: KMP or Boyer-Moore               │
  │  Feature:   Incremental search (highlight    │
  │             as you type)                     │
  └──────────────────────────────────────────────┘
```

---

## 2. Search Engines

```
  User Query: "string algorithm"

  ┌──────────────────────────────────────────────┐
  │ Google / Bing Pipeline:                      │
  │                                              │
  │  1. Inverted Index Lookup                    │
  │     "string" → [doc1, doc5, doc8, ...]       │
  │     "algorithm" → [doc2, doc5, doc9, ...]    │
  │                                              │
  │  2. Intersection → [doc5, ...]               │
  │                                              │
  │  3. Snippet Extraction                       │
  │     Find "string algorithm" in doc5 text     │
  │     → Pattern matching to highlight snippet  │
  │                                              │
  │  4. Ranking (PageRank, ML models)            │
  └──────────────────────────────────────────────┘
```

---

## 3. Bioinformatics / DNA Analysis

```
  DNA Sequence:    T = "ATCGATCGATCGTACGATCG..."  (billions of chars)
  Gene Pattern:    P = "ATCGTACG"                 

  Applications:
  ─────────────
  • Gene finding:         Locate gene markers in genome
  • Mutation detection:   Approximate pattern matching
  • Sequence alignment:   Find homologous sequences
  • Protein matching:     Pattern search in amino acid chains

  ┌──────────────────────────────────────────────┐
  │  Scale: Human genome ≈ 3.2 × 10⁹ bases      │
  │  Pattern: Gene sequence ≈ 100 - 10,000 bases │
  │                                              │
  │  Brute force: 3.2 × 10¹³ comparisons ✗      │
  │  KMP:         3.2 × 10⁹  comparisons ✓      │
  │  Suffix tree: Construction + O(m) query ✓    │
  └──────────────────────────────────────────────┘
```

---

## 4. Plagiarism Detection

```
  ┌──────────────────────────────────────────────┐
  │  Document A          Document B              │
  │  ┌──────────┐       ┌──────────┐            │
  │  │ ... text │       │ ... text │            │
  │  │ ████████ │ ←──── │ ████████ │  matching  │
  │  │ ... more │       │ ... more │  segments  │
  │  │ ████████ │ ←──── │ ████████ │            │
  │  └──────────┘       └──────────┘            │
  │                                              │
  │  Approach: Rabin-Karp with rolling hashes    │
  │  - Hash every k-word window in both docs     │
  │  - Find matching hashes → potential plagiarism│
  │  - Verify actual text matches                │
  └──────────────────────────────────────────────┘
```

---

## 5. Network Security / Intrusion Detection

```
  Network Packet Stream:
  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │ . │ . │ . │ M │ A │ L │ W │ . │ . │  ← payload
  └───┴───┴───┴───┴───┴───┴───┴───┴───┘

  Signature Database:
  ┌────────────────────────────┐
  │ Pattern 1: "MALW"         │
  │ Pattern 2: "EXPLOIT"      │
  │ Pattern 3: "SHELLCODE"    │
  │ ...                       │
  │ Pattern N: "ROOTKIT"      │
  └────────────────────────────┘

  Algorithm: Aho-Corasick (multi-pattern matching)
  - Build automaton from all signatures
  - Stream packets through automaton in O(n)
  - Detect any matching signature in real-time
```

---

## 6. Compiler Design

```
  Source Code:  "int main() { return 0; }"

  Lexical Analysis (Tokenization):
  ─────────────────────────────────
  Token patterns (simplified):
    KEYWORD:    "int" | "return" | "if" | "while" | ...
    IDENTIFIER: [a-z][a-z0-9]*
    NUMBER:     [0-9]+
    SYMBOL:     "(" | ")" | "{" | "}" | ";"

  The lexer uses pattern matching (often via finite automata
  derived from regex) to break source into tokens:

  "int"    → KEYWORD
  "main"   → IDENTIFIER
  "("      → LPAREN
  ")"      → RPAREN
  "{"      → LBRACE
  "return" → KEYWORD
  "0"      → NUMBER
  ";"      → SEMICOLON
  "}"      → RBRACE
```

---

## 7. Data Compression

```
  LZ77 / LZ78 / LZW Algorithms:
  ──────────────────────────────
  Find repeated patterns in the data stream.
  Replace repeated occurrences with back-references.

  Input:  "abcabcabc"
          ─── ─── ───
          abc abc abc

  Compressed: "abc" + (back 3, length 6)

  Pattern matching is the core operation:
  "Where did we see this substring before?"
```

---

## 8. Spam Filtering

```
  Email Text: "Congratulations! You've won a FREE iPhone. Click here..."

  Spam Patterns:
  ┌────────────────────────────┐
  │ "FREE"                    │
  │ "Click here"              │
  │ "You've won"              │
  │ "Act now"                 │
  │ ...                       │
  └────────────────────────────┘

  Multi-pattern matching to score the email.
  Higher score → more likely spam.
```

---

## 9. Application-Algorithm Mapping

```
  ┌────────────────────────┬───────────────────────────────┐
  │  Application           │  Best Algorithm(s)            │
  ├────────────────────────┼───────────────────────────────┤
  │  Find in text editor   │  Boyer-Moore, KMP             │
  │  DNA gene search       │  Suffix tree, KMP             │
  │  Plagiarism detection  │  Rabin-Karp (rolling hash)    │
  │  Network IDS           │  Aho-Corasick                 │
  │  Compiler lexer        │  Finite Automata (regex → DFA)│
  │  Spell check           │  Trie + edit distance         │
  │  Autocomplete          │  Trie, Suffix tree            │
  │  Data compression      │  Suffix tree, LZ algorithms   │
  │  Version control diff  │  LCS (edit distance)          │
  │  Search suggestions    │  Trie with ranking            │
  └────────────────────────┴───────────────────────────────┘
```

---

## 📝 Summary Table

| Domain | Why Pattern Matching? |
|--------|----------------------|
| Text editors | Find/replace, syntax highlighting |
| Search engines | Snippet extraction, query matching |
| Bioinformatics | Gene finding, sequence alignment |
| Security | Intrusion detection, malware scanning |
| Compilers | Lexical analysis, tokenization |
| Compression | Finding repeated patterns |
| Plagiarism | Comparing document segments |

---

## ❓ Quick Revision Questions

1. **Why is Aho-Corasick preferred for network intrusion detection?**
   <details><summary>Answer</summary>It can match multiple patterns simultaneously in a single pass over the text, making it ideal for checking against a database of thousands of attack signatures.</details>

2. **How does Rabin-Karp help in plagiarism detection?**
   <details><summary>Answer</summary>Rolling hashes of k-word windows from both documents are compared. Matching hashes indicate potential plagiarized segments, which are then verified.</details>

3. **What role does pattern matching play in compilers?**
   <details><summary>Answer</summary>The lexer (lexical analyzer) uses pattern matching to break source code into tokens like keywords, identifiers, numbers, and symbols.</details>

4. **Why can't brute force handle DNA sequence searching?**
   <details><summary>Answer</summary>With genome size ~3 billion and patterns ~1000+, brute force needs ~3 trillion comparisons, which is impractical.</details>

5. **How does LZ77 compression relate to pattern matching?**
   <details><summary>Answer</summary>LZ77 searches for repeated substrings and replaces later occurrences with back-references (offset, length), requiring substring search in the look-back window.</details>

---

| [⬅️ Previous: Optimization Need](02-optimization-need.md) | [Next: Problem Variations ➡️](04-problem-variations.md) |
|:---|---:|
