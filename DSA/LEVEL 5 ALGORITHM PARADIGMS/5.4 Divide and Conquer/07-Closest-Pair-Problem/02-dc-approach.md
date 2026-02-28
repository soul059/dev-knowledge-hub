# Chapter 2: D&C Approach to Closest Pair

## Overview

The Divide and Conquer approach to the Closest Pair problem is one of the most elegant geometric algorithms. It divides the point set by a vertical line, recursively finds closest pairs in each half, and then efficiently checks pairs that straddle the dividing line.

---

## High-Level Idea

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. DIVIDE:  Split points into Left half and Right half     │
│              by a vertical line through the median x-coord  │
│                                                             │
│  2. CONQUER: Recursively find closest pair in each half     │
│              δ_L = closest in left,  δ_R = closest in right │
│                                                             │
│  3. COMBINE: Let δ = min(δ_L, δ_R)                         │
│              Check if any cross-boundary pair < δ           │
│              (This is the HARD part!)                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## The Algorithm Step by Step

### Step 1: Pre-sort

```
Sort all points by x-coordinate: Px
Sort all points by y-coordinate: Py

(Sorting once: O(n log n) — done before recursion)
```

### Step 2: Divide

```
        Left Half              Right Half
           │
    •      │      •
  •     •  │        •
       •   │    •
    •      │         •
           │
        median x

Split Px into two halves at the median x-coordinate.
Each half has ⌊n/2⌋ and ⌈n/2⌉ points.
```

### Step 3: Conquer (Recurse)

```
δ_L ← CLOSEST_PAIR(Left Half)
δ_R ← CLOSEST_PAIR(Right Half)
δ   ← min(δ_L, δ_R)
```

### Step 4: Combine (The Key Step)

```
The closest pair overall is EITHER:
  (a) Both points in the Left half   → captured by δ_L
  (b) Both points in the Right half  → captured by δ_R
  (c) One point in Left, one in Right → need to CHECK!

       Left         Right
         │    δ   δ    │
    •    │◄──►│   │◄──►│    •
  •   •  │    │   │    │  •
     •   │    │   │    │      •
    •    │    │   │    │    •
         │    │   │    │
              │   │
         ◄────────────►
            2δ strip

Only points within distance δ of the dividing line
could possibly form a closer pair!
```

---

## Pseudocode

```
CLOSEST_PAIR(P):
    // Pre-processing (done once)
    Px ← P sorted by x-coordinate
    Py ← P sorted by y-coordinate
    return CLOSEST_PAIR_REC(Px, Py)

CLOSEST_PAIR_REC(Px, Py):
    n ← |Px|
    
    // Base case
    if n ≤ 3:
        return BRUTE_FORCE(Px)
    
    // DIVIDE
    mid ← n / 2
    midPoint ← Px[mid]
    
    // Split Px into left and right halves
    Pxl ← Px[0 .. mid-1]
    Pxr ← Px[mid .. n-1]
    
    // Split Py into left and right (maintaining y-order)
    Pyl ← points in Py with x ≤ midPoint.x
    Pyr ← points in Py with x > midPoint.x
    
    // CONQUER
    (p1, q1, δL) ← CLOSEST_PAIR_REC(Pxl, Pyl)
    (p2, q2, δR) ← CLOSEST_PAIR_REC(Pxr, Pyr)
    
    // Best from recursive calls
    δ ← min(δL, δR)
    (best_p, best_q) ← pair achieving δ
    
    // COMBINE — check strip
    strip ← [points in Py where |x - midPoint.x| < δ]
    
    (p3, q3, δS) ← CLOSEST_IN_STRIP(strip, δ)
    
    if δS < δ:
        return (p3, q3, δS)
    else:
        return (best_p, best_q, δ)

CLOSEST_IN_STRIP(strip, δ):
    // strip is sorted by y-coordinate (from Py)
    min_dist ← δ
    best ← null
    
    for i ← 0 to |strip|-1:
        for j ← i+1 to |strip|-1:
            if strip[j].y - strip[i].y ≥ δ:
                break    // No more candidates
            d ← DISTANCE(strip[i], strip[j])
            if d < min_dist:
                min_dist ← d
                best ← (strip[i], strip[j])
    
    return best, min_dist
```

---

## Complete Walkthrough

```
Points: A(2,3), B(12,30), C(40,50), D(5,1), E(12,10), F(3,4)

Step 1: Sort by x
  Px = [A(2,3), F(3,4), D(5,1), B(12,30), E(12,10), C(40,50)]
  Py = [D(5,1), A(2,3), F(3,4), E(12,10), B(12,30), C(40,50)]

Step 2: Divide at median (between D and B, x = 5/12 boundary)
  Left:  {A(2,3), F(3,4), D(5,1)}
  Right: {B(12,30), E(12,10), C(40,50)}

Step 3: Recurse on Left {A(2,3), F(3,4), D(5,1)}
  Base case (n=3): check all pairs
    dist(A,F) = √(1+1) = 1.41  ← minimum!
    dist(A,D) = √(9+4) = 3.61
    dist(F,D) = √(4+9) = 3.61
  δ_L = 1.41, pair = (A, F)

Step 4: Recurse on Right {B(12,30), E(12,10), C(40,50)}
  Base case (n=3): check all pairs
    dist(B,E) = √(0+400) = 20.0
    dist(B,C) = √(784+400) = 34.4
    dist(E,C) = √(784+1600) = 48.8
  δ_R = 20.0, pair = (B, E)

Step 5: Combine
  δ = min(1.41, 20.0) = 1.41
  midPoint.x = 5
  Strip: points where |x - 5| < 1.41
    → x range: [3.59, 6.41]
    → Strip contains: F(3,4)? |3-5|=2 > 1.41, NO
                      D(5,1)? |5-5|=0 < 1.41, YES
    → Strip has only 1 point → no cross pairs possible

  ANSWER: (A, F) with distance 1.41
```

---

## Visualizing the Recursion

```
                    {A,F,D,B,E,C}
                     /         \
              {A,F,D}           {B,E,C}
             /                       \
         Brute force              Brute force
         δ_L = 1.41              δ_R = 20.0
         (A, F)                  (B, E)
              \                  /
               \                /
                δ = min(1.41, 20.0) = 1.41
                Check strip: no cross pair < 1.41
                
                ANSWER: (A, F), dist = 1.41
```

---

## Critical Detail: Splitting Py

```
When we split Px into left/right, we ALSO need Py split:

Py (all points sorted by y):
  D(5,1), A(2,3), F(3,4), E(12,10), B(12,30), C(40,50)

midPoint.x = 5

Pyl (x ≤ 5, maintaining y-order):
  D(5,1), A(2,3), F(3,4)    ← still sorted by y!

Pyr (x > 5, maintaining y-order):
  E(12,10), B(12,30), C(40,50)  ← still sorted by y!

This splitting takes O(n) time (one linear scan).
We preserve y-ordering for the strip step later!
```

---

## Why This Approach Works

```
KEY INSIGHT: If closest pair crosses the dividing line,
both points must be within distance δ of the line.

Proof:
  If p is left of line and q is right of line,
  and dist(p,q) < δ, then:
  
  |p.x - line.x| ≤ dist(p,q) < δ    ✓
  |q.x - line.x| ≤ dist(p,q) < δ    ✓
  
  So both p and q are in the strip of width 2δ!

           │← δ →│← δ →│
           │     │     │
      •    │  •  │  •  │    •
           │     │     │
     Points outside the strip CANNOT be
     part of a cross-boundary closest pair.
```

---

## Initial Complexity Analysis

```
T(n) = 2T(n/2) + C(n)

where C(n) is the combine step cost:
  - Building strip: O(n) 
  - Checking strip pairs: ???

If strip check is O(n²) worst case → T(n) = O(n² ) total!
That would be no better than brute force!

BUT: the strip has a special geometric property that
limits it to O(n) or at most O(n log n).
→ See next chapter: Strip Optimization!
```

---

## Summary Table

| Step | Operation | Time |
|------|-----------|------|
| Pre-sort | Sort by x and y | O(n log n) |
| Divide | Split at median x | O(n) |
| Conquer | Two recursive calls | 2T(n/2) |
| Build strip | Filter points near line | O(n) |
| Check strip | Compare strip pairs | ??? |
| Recurrence | T(n) = 2T(n/2) + O(n) + strip | Depends on strip |

---

## Quick Revision Questions

1. **How do we divide the points into two halves?**
2. **What is δ in the combine step?**
3. **Why do we only check a strip of width 2δ?**
4. **Why is maintaining y-sorted order important?**
5. **What is the base case for the recursion?**
6. **Why can't we naively check all strip pairs?**

---

[← Previous: Problem Description](01-problem-description.md) | [Next: Strip Optimization →](03-strip-optimization.md)
