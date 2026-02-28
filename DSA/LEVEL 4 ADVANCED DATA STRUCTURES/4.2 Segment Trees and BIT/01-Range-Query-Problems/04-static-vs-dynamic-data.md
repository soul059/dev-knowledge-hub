# Static vs Dynamic Data

## Chapter Overview

Understanding whether your data is **static** (never changes) or **dynamic** (changes between queries) is the most important decision in choosing a range query approach. This chapter formalizes the distinction and its implications.

---

## Static Data

The array is built once and **never modified**. All queries operate on the same data.

```
┌─────────────────────────────────────────────────┐
│                 STATIC DATA                      │
│                                                  │
│  Build Array ──→ Query ──→ Query ──→ Query ──→ … │
│                                                  │
│  Array NEVER changes after construction          │
└─────────────────────────────────────────────────┘

Example: Historical temperature data
  temps = [72, 68, 75, 71, 80, 77, 69, 73]
  
  Q: "What was the coldest temp from day 2 to day 5?"
  Q: "What was the total rainfall from day 0 to day 3?"
  
  Data doesn't change — it's history.
```

### Best Approaches for Static Data

| Query Type | Best Structure | Query Time | Build Time |
|-----------|---------------|:----------:|:----------:|
| Range Sum | Prefix Sum | O(1) | O(n) |
| Range Min/Max | Sparse Table | O(1) | O(n log n) |
| Range GCD | Sparse Table | O(1) | O(n log n) |
| k-th element | Persistent ST | O(log n) | O(n log n) |

---

## Dynamic Data

The array is **modified between queries**. Queries must reflect the current state.

```
┌──────────────────────────────────────────────────────────┐
│                    DYNAMIC DATA                           │
│                                                          │
│  Build ──→ Query ──→ Update ──→ Query ──→ Update ──→ …  │
│                                                          │
│  Array CHANGES between queries                           │
└──────────────────────────────────────────────────────────┘

Example: Live stock portfolio
  prices = [150, 89, 200, 175, 310]
  
  Q: "Total value of stocks 1-3?"  → 89+200+175 = 464
  U: Stock 2 price changes to 220
  prices = [150, 89, 220, 175, 310]
  Q: "Total value of stocks 1-3?"  → 89+220+175 = 484  ← DIFFERENT!
```

### Best Approaches for Dynamic Data

| Query + Update Type | Best Structure | Query | Update |
|-------------------|---------------|:-----:|:------:|
| Sum + Point Update | BIT | O(log n) | O(log n) |
| Min/Max + Point Update | Segment Tree | O(log n) | O(log n) |
| Sum + Range Update | Seg Tree + Lazy | O(log n) | O(log n) |
| Min/Max + Range Update | Seg Tree + Lazy | O(log n) | O(log n) |

---

## Identifying the Pattern

```
┌─────────────────────────────────────────────────────┐
│                DECISION FLOWCHART                    │
│                                                      │
│  ┌──────────┐                                        │
│  │ Are there │──── NO ──→ Prefix Sum / Sparse Table  │
│  │ updates?  │                                       │
│  └────┬─────┘                                        │
│       │ YES                                          │
│       ▼                                              │
│  ┌──────────────┐                                    │
│  │  Point or    │── POINT ──→ BIT (for sum/xor)     │
│  │  Range       │             Seg Tree (for min/max) │
│  │  updates?    │                                    │
│  └────┬─────────┘                                    │
│       │ RANGE                                        │
│       ▼                                              │
│  Segment Tree with Lazy Propagation                  │
└─────────────────────────────────────────────────────┘
```

---

## Online vs Offline Problems

### Online
Queries arrive one at a time. You must answer each query before seeing the next.

```
Online:
  Read query → Answer → Read next query → Answer → ...
  
  Must use real-time data structures (Segment Tree, BIT)
```

### Offline
All queries are known in advance. You can reorder and preprocess them.

```
Offline:
  Read ALL queries → Sort/group them → Process efficiently
  
  Can use techniques like:
  - Mo's algorithm (sqrt decomposition of queries)
  - Sort queries by right endpoint
  - CDQ divide and conquer
```

---

## Problem Classification Examples

```
┌─────────────────────────────────────────────────────────────┐
│  Problem                        │ Static/Dynamic │ Approach │
├─────────────────────────────────┼────────────────┼──────────┤
│  Immutable range sum            │ Static         │ Prefix   │
│  Range min query, no updates    │ Static         │ Sparse T │
│  Sum query + point update       │ Dynamic        │ BIT      │
│  Min query + point update       │ Dynamic        │ Seg Tree │
│  Sum query + range update       │ Dynamic        │ Lazy ST  │
│  Range sum, given all queries   │ Offline        │ Mo's     │
│  Online min query + range add   │ Dynamic        │ Lazy ST  │
└─────────────────────────────────┴────────────────┴──────────┘
```

---

## The Spectrum

```
Simple ◄────────────────────────────────────────────► Complex

Prefix    Sparse    Sqrt     BIT     Segment    Persistent
 Sum      Table    Decomp           Tree+Lazy   Seg Tree

Static ◄──────────────────►  Dynamic  ◄─────────► Versioned
No updates                   Updates      Need history of
                             between      all versions
                             queries
```

---

## Key Insight: Identify Before You Code

Before writing any code, classify the problem:

```
Ask yourself:
  1. What QUERY do I need?       (sum? min? max? count?)
  2. Are there UPDATES?          (yes → dynamic)
  3. What TYPE of updates?       (point? range?)
  4. Are ALL queries known?      (yes → maybe offline)
  5. How large is n and Q?       (determines if O(n√n) is okay)
```

---

## Summary Table

| Concept | Static | Dynamic |
|---------|--------|---------|
| Array changes? | No | Yes |
| Best for sum | Prefix Sum O(1) | BIT/Seg Tree O(log n) |
| Best for min/max | Sparse Table O(1) | Seg Tree O(log n) |
| Update cost | N/A | O(log n) with proper structure |
| Build cost | O(n) or O(n log n) | O(n) |
| Key advantage | Faster queries | Handles modifications |
| Online required? | Usually no | Usually yes |

---

## Quick Revision Questions

1. What defines "static" vs "dynamic" data? *(Static = no updates after build; Dynamic = updates between queries)*
2. What is the best structure for static range minimum queries? *(Sparse Table — O(1) query)*
3. When do you need lazy propagation? *(When you have range updates, not just point updates)*
4. What is the difference between online and offline query processing? *(Online: answer each query immediately; Offline: see all queries first, process in optimal order)*
5. For a problem with 10⁵ sum queries and 10⁵ point updates, what structure would you use? *(BIT — simpler and sufficient for sum + point update)*

---

[← Previous: Need for Advanced Structures](03-need-for-advanced-structures.md) | [Next: Query Types →](05-query-types.md)
