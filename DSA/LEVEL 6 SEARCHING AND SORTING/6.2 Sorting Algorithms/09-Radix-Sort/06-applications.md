# Unit 9: Radix Sort — Applications

[← Previous: String Sorting](05-string-sorting.md) | [Next: Unit 10 - Bucket Sort →](../10-Bucket-Sort/01-algorithm-description.md)

---

## Overview

Radix Sort powers many real-world systems where data has fixed structure — IP addresses, dates, phone numbers, and more. This chapter explores practical applications.

---

## 1. Sorting Large Datasets of Integers

```
  Problem: Sort 100 million 32-bit integers
  
  Quick Sort: 100M × log₂(100M) ≈ 100M × 27 ≈ 2.7 billion ops
  Radix Sort (base 256): 4 passes × 100M ≈ 400 million ops
  
  Radix Sort is ~6x fewer operations!
  
  In practice, the speedup may be 2-3x due to cache effects,
  but for truly large datasets, Radix Sort wins.
```

---

## 2. Sorting IP Addresses

```
  IP addresses: 4 bytes (32 bits)
  Perfect for byte-level radix sort!
  
  IPs: [192.168.1.1, 10.0.0.1, 172.16.0.5, 10.0.0.2]
  
  Represent as 32-bit integers:
  192.168.1.1  → 3232235777
  10.0.0.1     → 167772161
  172.16.0.5   → 2886729733
  10.0.0.2     → 167772162
  
  4 passes of byte-level radix sort → sorted!
  
  Result: [10.0.0.1, 10.0.0.2, 172.16.0.5, 192.168.1.1]
```

---

## 3. Sorting Dates

```
  Dates: YYYY-MM-DD format
  
  LSD Radix Sort on dates:
  Pass 1: Sort by day    (DD: 01-31, k=31)
  Pass 2: Sort by month  (MM: 01-12, k=12)
  Pass 3: Sort by year   (YYYY: range varies, k=range)
  
  Since each pass is stable, dates end up in 
  chronological order after 3 passes!
  
  Input:  [2024-03-15, 2023-12-01, 2024-01-20, 2023-03-15]
  
  After day sort:   [2023-12-01, 2024-03-15, 2023-03-15, 2024-01-20]
  After month sort: [2024-01-20, 2024-03-15, 2023-03-15, 2023-12-01]
  After year sort:  [2023-03-15, 2023-12-01, 2024-01-20, 2024-03-15]
  
  ✓ Chronologically sorted!
```

---

## 4. Suffix Arrays (Used in String Algorithms)

```
  Suffix array: sorted array of all suffixes of a string
  Used in: string search, bioinformatics, data compression
  
  String: "banana"
  Suffixes:
    0: "banana"
    1: "anana"
    2: "nana"
    3: "ana"
    4: "na"
    5: "a"
  
  Suffix array (sorted): [5, 3, 1, 0, 4, 2]
    "a", "ana", "anana", "banana", "na", "nana"
  
  MSD Radix Sort can build suffix arrays efficiently:
  O(n) time for fixed alphabet (with advanced techniques like SA-IS)
```

---

## 5. Card Sorting (Classic Example)

```
  Sort a deck of 52 cards by suit then rank:
  
  ┌──────────────────────────────────────────────────────┐
  │  Think of each card as a 2-digit number:            │
  │  - Digit 1 (less significant): Rank (1-13)         │
  │  - Digit 2 (more significant): Suit (1-4)          │
  │                                                      │
  │  LSD Radix Sort:                                    │
  │  Pass 1: Sort by rank → all cards ordered by rank   │
  │  Pass 2: Sort by suit → within each suit, rank      │
  │           order preserved by stability!              │
  │                                                      │
  │  Result: ♣A,♣2,...,♣K, ♦A,...,♦K, ♥A,...,♥K, ♠A,...,♠K │
  └──────────────────────────────────────────────────────┘
```

---

## 6. Parallel and GPU Sorting

```
  Radix Sort is highly parallelizable:
  
  ┌──────────────────────────────────────────────────────┐
  │  Each pass (Counting Sort) can be parallelized:     │
  │                                                      │
  │  1. Counting phase: Each thread counts its chunk    │
  │     Thread 1: count elements 0..n/4                 │
  │     Thread 2: count elements n/4..n/2               │
  │     Thread 3: count elements n/2..3n/4              │
  │     Thread 4: count elements 3n/4..n                │
  │                                                      │
  │  2. Merge counts → prefix sum                      │
  │                                                      │
  │  3. Placement phase: Each thread places its chunk   │
  │                                                      │
  │  GPU implementations of Radix Sort are among the    │
  │  fastest sorting algorithms for large datasets!     │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Database Indexing

```
  Databases often sort records by multi-column keys:
  
  Table: Employees
  ┌────────────┬──────────┬───────┐
  │ Department │ Name     │ Age   │
  ├────────────┼──────────┼───────┤
  │ Sales      │ Alice    │ 30    │
  │ HR         │ Bob      │ 25    │
  │ Sales      │ Carol    │ 28    │
  │ HR         │ Dave     │ 32    │
  └────────────┴──────────┴───────┘
  
  ORDER BY Department, Age:
  
  LSD approach (Radix-like):
  1. Sort by Age (stable)
  2. Sort by Department (stable)
  
  Result maintains Age order within each Department!
```

---

## When Radix Sort Is Used in Practice

```
  ✓ Used in:
  ┌──────────────────────────────────────────────────────────┐
  │  • GPU parallel sorting (CUB library)                  │
  │  • Network packet sorting by IP                        │
  │  • Suffix array construction (bioinformatics)          │
  │  • Fixed-width integer sorting (large datasets)        │
  │  • Phone number / SSN sorting                          │
  │  • Sort-merge joins in databases                       │
  │  • Computer graphics (sorting by depth/Z-buffer)       │
  └──────────────────────────────────────────────────────────┘
  
  ✗ NOT commonly used for:
  ┌──────────────────────────────────────────────────────────┐
  │  • General-purpose standard library sort               │
  │  • Floating-point numbers (complex bit representation) │
  │  • Small datasets (overhead not worth it)              │
  │  • Unknown data types                                   │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Application | Data Type | Why Radix Works |
|---|---|---|
| Large integer arrays | 32/64-bit ints | Fixed digits, O(n) |
| IP addresses | 4 bytes | Natural byte decomposition |
| Dates | 3 components | Multi-key LSD approach |
| Card sorting | 2 keys | Classic LSD example |
| Suffix arrays | Strings | MSD for lexicographic order |
| GPU sorting | Any fixed-width | Highly parallelizable |
| Database keys | Multi-column | LSD = multi-key sort |

---

## Quick Revision Questions

1. **Why is Radix Sort popular for GPU implementations?**
2. **How would you sort a list of dates using radix sort?**
3. **Why can sorting IP addresses be treated as 4-pass radix sort?**
4. **What kind of data makes radix sort impractical?**
5. **How does stability enable multi-column database sorting?**

---

[← Previous: String Sorting](05-string-sorting.md) | [Next: Unit 10 - Bucket Sort →](../10-Bucket-Sort/01-algorithm-description.md)
