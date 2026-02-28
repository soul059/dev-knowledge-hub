# Chapter 10.5 — Applications in Bioinformatics

> **Unit 10: Advanced String Concepts** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Bioinformatics is one of the most impactful application domains for
string algorithms. DNA, RNA, and protein sequences are essentially
strings over small alphabets, making them perfect targets for
pattern matching, alignment, indexing, and compression techniques.

---

## 1. Biological Strings

```
  DNA:   Alphabet Σ = {A, C, G, T}    (4 characters)
  RNA:   Alphabet Σ = {A, C, G, U}    (T → U in transcription)
  Protein: Alphabet Σ = 20 amino acids (single-letter codes)

  Example DNA sequence:
  5'─ A T G C C G A T T A C G G A T ─3'
  3'─ T A C G G C T A A T G C C T A ─5'  (complement)

  Key property: Complementary base pairing
    A ↔ T  (or A ↔ U in RNA)
    C ↔ G

  Reverse complement of "ATGCCG":
    Complement:         T A C G G C
    Reverse complement: C G G C A T
```

---

## 2. Sequence Alignment

```
  Problem: Given two biological sequences, find the best
  alignment to identify similarities (evolutionary relationship).

  ┌─── Global Alignment (Needleman-Wunsch) ────────────┐
  │  Align entire sequences end-to-end                  │
  │                                                     │
  │  S1: A T C G T A C                                 │
  │  S2: A T - G A A C                                 │
  │      | |   | . | |                                  │
  │  | = match, . = mismatch, - = gap (indel)          │
  │                                                     │
  │  Scoring: match = +1, mismatch = -1, gap = -2      │
  │  Algorithm: DP, O(nm) time and space                │
  └─────────────────────────────────────────────────────┘

  ┌─── Local Alignment (Smith-Waterman) ───────────────┐
  │  Find best matching subsequences                    │
  │  Key change: dp[i][j] = max(0, ...)                │
  │  Allows alignment to start/end anywhere             │
  │                                                     │
  │  S1: xxxATCGTACxxx                                  │
  │  S2: yyyATCGAACyyy                                  │
  │         ||||.||                                     │
  │  Best local: ATCG-AC                                │
  └─────────────────────────────────────────────────────┘

  Scoring Matrix (for protein):
  ┌────────────────────────────┐
  │  BLOSUM62 / PAM matrices   │
  │  20×20 substitution scores │
  │  Based on observed         │
  │  evolutionary mutations    │
  └────────────────────────────┘
```

### Needleman-Wunsch Pseudocode

```python
def global_align(S1, S2, match=1, mismatch=-1, gap=-2):
    n, m = len(S1), len(S2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    
    # Base cases: gap penalties
    for i in range(n + 1):
        dp[i][0] = i * gap
    for j in range(m + 1):
        dp[0][j] = j * gap
    
    # Fill DP table
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            score = match if S1[i-1] == S2[j-1] else mismatch
            dp[i][j] = max(
                dp[i-1][j-1] + score,   # match/mismatch
                dp[i-1][j] + gap,        # gap in S2
                dp[i][j-1] + gap         # gap in S1
            )
    
    # Traceback to get alignment
    align1, align2 = [], []
    i, j = n, m
    while i > 0 or j > 0:
        if i > 0 and j > 0:
            score = match if S1[i-1] == S2[j-1] else mismatch
            if dp[i][j] == dp[i-1][j-1] + score:
                align1.append(S1[i-1])
                align2.append(S2[j-1])
                i -= 1; j -= 1; continue
        if i > 0 and dp[i][j] == dp[i-1][j] + gap:
            align1.append(S1[i-1])
            align2.append('-')
            i -= 1
        else:
            align1.append('-')
            align2.append(S2[j-1])
            j -= 1
    
    return ''.join(reversed(align1)), ''.join(reversed(align2))
```

---

## 3. BLAST — Fast Approximate Search

```
  Problem: Search a query sequence against a massive database
  (millions of sequences). Smith-Waterman is too slow!

  BLAST Strategy (heuristic):
  ┌────────────────────────────────────────────────────────┐
  │                                                        │
  │  Step 1: SEEDING                                       │
  │  ────────────────                                      │
  │  Break query into k-mers (words of length k ≈ 11)     │
  │  Find exact matches in DB using hash table / index     │
  │                                                        │
  │  Query: ATCGTACGATCG                                   │
  │  11-mers: ATCGTACGATC, TCGTACGATCG                    │
  │                                                        │
  │  Step 2: EXTENSION                                     │
  │  ────────────────                                      │
  │  Extend seed matches in both directions                │
  │  using ungapped alignment                              │
  │                                                        │
  │  ←── extend ── SEED ── extend ──→                      │
  │                                                        │
  │  Step 3: EVALUATION                                    │
  │  ────────────────                                      │
  │  Perform gapped alignment on high-scoring segments     │
  │  Report statistically significant matches (E-value)    │
  │                                                        │
  └────────────────────────────────────────────────────────┘

  String algorithms used:
  • Hash tables for k-mer lookup
  • Suffix arrays for database indexing (MegaBLAST)
  • Seed-and-extend paradigm (heuristic, not guaranteed optimal)
```

---

## 4. Suffix Arrays and BWT in Genomics

```
  Modern genome aligners use FM-index (BWT + suffix array):

  ┌─────────────────────────────────────────────────────┐
  │  Reference Genome (~3 billion bp for human)         │
  │                                                     │
  │  Build once:                                        │
  │    1. Suffix Array of reference                     │
  │    2. BWT of reference                              │
  │    3. FM-index (rank/count arrays)                  │
  │                                                     │
  │  For each read (short DNA fragment, ~150 bp):       │
  │    → Backward search in FM-index: O(m)              │
  │    → Find position via suffix array: O(1)           │
  │    → Allow mismatches: backtracking                 │
  └─────────────────────────────────────────────────────┘

  Tools: Bowtie, BWA, HISAT2 — all BWT-based!

  Why BWT works:
  ┌───────────────────────────────────────────┐
  │  DNA has lots of repetition               │
  │  BWT groups similar contexts → compresses │
  │  FM-index: search without decompressing   │
  │  Memory: ~1.5 GB for human genome         │
  │  (vs. ~12 GB for suffix tree)             │
  └───────────────────────────────────────────┘

  Read mapping example:
    Reference:  ...ATCGATCGATCG...
    Read:       GATCGA
    FM-index backward search:
      Start: range for "A"
      Then:  range for "GA"
      Then:  range for "CGA"
      Then:  range for "TCGA"
      Then:  range for "ATCGA"
      Then:  range for "GATCGA"
    → Position found in O(6) steps!
```

---

## 5. Motif Finding

```
  Problem: Given multiple DNA sequences, find a common
  short pattern (motif) that appears in each sequence.

  Sequences:
    S1: atcGCATGcag      ┐
    S2: tgcGCATGtaa      │ Motif: GCATG
    S3: aagGCATGccg      │ (conserved binding site)
    S4: ccaGCATGgat      ┘

  Approaches using string algorithms:

  ┌───────────────────────────────────────────┐
  │  1. Exact motif: Aho-Corasick            │
  │     Enumerate all k-mers from S1         │
  │     Search in S2, S3, S4 simultaneously  │
  │                                           │
  │  2. Approximate motif: Edit distance     │
  │     Allow d mismatches per occurrence     │
  │     Trie + branch-and-bound              │
  │                                           │
  │  3. Consensus motif: Profile/PWM         │
  │     Position Weight Matrix scoring       │
  │     Gibbs sampling, EM algorithm         │
  └───────────────────────────────────────────┘
```

---

## 6. Genome Assembly

```
  Problem: Reconstruct a genome from millions of short reads.

  Approach: de Bruijn Graph

  Reads: ATGCG, TGCGA, GCGAT, CGATA

  Step 1: Extract (k-1)-mers (k=4, so 3-mers)
  ┌──────────┬────────────────┐
  │ Read     │ 3-mers         │
  ├──────────┼────────────────┤
  │ ATGCG    │ ATG→TGC→GCG   │
  │ TGCGA    │ TGC→GCG→CGA   │
  │ GCGAT    │ GCG→CGA→GAT   │
  │ CGATA    │ CGA→GAT→ATA   │
  └──────────┴────────────────┘

  Step 2: Build de Bruijn graph (nodes = 3-mers, edges = 4-mers)

       ATG → TGC → GCG → CGA → GAT → ATA
              ↑           |
              └───────────┘

  Step 3: Find Eulerian path → reconstructed sequence

       ATGCGATA

  String algorithms involved:
  • Suffix arrays for overlap detection (OLC assembly)
  • Hash tables for k-mer counting
  • BWT for compressed k-mer indexing
  • String graph from suffix-prefix overlaps
```

---

## 7. Multiple Sequence Alignment (MSA)

```
  Problem: Align 3+ sequences simultaneously.

  DP approach: O(nᵏ) for k sequences — exponential!

  Progressive alignment (ClustalW, MUSCLE):
  ┌──────────────────────────────────────────────┐
  │                                              │
  │  1. Compute all pairwise alignments          │
  │     → Use edit distance / alignment score    │
  │                                              │
  │  2. Build guide tree from distances           │
  │     → Hierarchical clustering                │
  │                                              │
  │  3. Align sequences following tree order     │
  │     → Merge alignments bottom-up             │
  │                                              │
  │       ┌── S1                                 │
  │     ┌─┤                                      │
  │   ┌─┤ └── S2        Align S1,S2 first       │
  │   │ │                then add S3             │
  │   │ └─── S3         then add S4             │
  │   │                                          │
  │   └───── S4                                  │
  │                                              │
  └──────────────────────────────────────────────┘

  Result:
    S1: A T C G - T A C
    S2: A T C G A A C -
    S3: A - C G A T C -
    S4: A T C - A T A C
        * * * .   * . *   (* = conserved column)
```

---

## 8. Summary: String Algorithms in Bioinformatics

```
  ┌──────────────────────────────────────────────────────────┐
  │  Algorithm             → Application                     │
  ├──────────────────────────────────────────────────────────┤
  │  Edit Distance         → Sequence alignment scoring      │
  │  Needleman-Wunsch      → Global alignment                │
  │  Smith-Waterman        → Local alignment (BLAST)         │
  │  Suffix Array          → Genome indexing, LCP queries    │
  │  BWT / FM-index        → Read mapping (Bowtie, BWA)     │
  │  Aho-Corasick          → Multi-motif scanning           │
  │  KMP / Z-algorithm     → Exact pattern search in DNA    │
  │  String Hashing        → K-mer fingerprinting           │
  │  Trie                  → Dictionary of biological terms  │
  │  de Bruijn Graph       → Genome assembly                │
  │  LCS / LCSubstring     → Sequence comparison            │
  │  Suffix Tree           → Repeat finding, MUM detection  │
  └──────────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Application | Key Algorithms | Scale |
|-------------|---------------|-------|
| Pairwise alignment | DP (NW/SW) | O(nm) |
| Database search | BLAST (seed-extend) | Heuristic, fast |
| Read mapping | BWT / FM-index | O(m) per read |
| Motif finding | Aho-Corasick, k-mers | O(n + m + z) |
| Genome assembly | de Bruijn graph, suffix array | Billions of k-mers |
| Multiple alignment | Progressive DP | O(nᵏ) exact, heuristic |

---

## ❓ Quick Revision Questions

1. **What alphabets are used for DNA and protein sequences?**
   <details><summary>Answer</summary>DNA uses {A, C, G, T} (4 characters). Protein uses 20 amino acids, each represented by a single letter (e.g., A=Alanine, R=Arginine, etc.).</details>

2. **What is the difference between global and local alignment?**
   <details><summary>Answer</summary>Global alignment (Needleman-Wunsch) aligns entire sequences end-to-end. Local alignment (Smith-Waterman) finds the best matching sub-regions by allowing dp[i][j] = max(0, ...), so the alignment can start and end anywhere.</details>

3. **How does BLAST achieve fast database search?**
   <details><summary>Answer</summary>BLAST uses a seed-and-extend heuristic: (1) break query into k-mers, (2) find exact seed matches in the database using indexing, (3) extend seeds into longer alignments, (4) evaluate statistical significance. It avoids full Smith-Waterman on the entire database.</details>

4. **Why is BWT/FM-index preferred for read mapping over suffix trees?**
   <details><summary>Answer</summary>FM-index compresses the genome index to ~1.5 GB for the human genome (vs ~12 GB for suffix trees), while still supporting O(m) exact pattern matching via backward search. This is critical when mapping billions of short reads.</details>

5. **What is a de Bruijn graph and how is it used in assembly?**
   <details><summary>Answer</summary>A de Bruijn graph has (k-1)-mers as nodes and k-mers as edges. Reads are broken into k-mers, edges are added between consecutive (k-1)-mers. The genome is reconstructed by finding an Eulerian path through this graph.</details>

6. **Which string algorithms are used for motif finding?**
   <details><summary>Answer</summary>Exact motifs: Aho-Corasick (multi-pattern matching), k-mer enumeration with hashing. Approximate motifs: edit distance with trie-based branch and bound. Probabilistic motifs: Position Weight Matrices with Gibbs sampling or EM algorithm.</details>

---

| [⬅️ Previous: Aho-Corasick Algorithm](04-aho-corasick.md) | [🏠 Course Home](../README.md) |
|:---|---:|

---

> 🎉 **Congratulations!** You have completed the String Algorithms course.
> Return to the [Course Home](../README.md) to review any unit.
