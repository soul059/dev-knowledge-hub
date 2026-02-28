# Chapter 1: Stock Span Problem

[← Previous: Daily Temperatures](../05-Monotonic-Stack/06-daily-temperatures.md) | [Next: Max Area Histogram →](02-max-area-histogram.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

The **Stock Span** problem is a classic application of the monotonic stack. Given daily stock prices, the **span** of a day is the number of **consecutive days** just before (and including) that day where the price was **less than or equal** to the current day's price.

---

## Problem Statement

```
Given: prices = [100, 80, 60, 70, 60, 75, 85]

Day 0: price=100, no previous days                → span = 1
Day 1: price=80,  80 < 100 → stop                 → span = 1
Day 2: price=60,  60 < 80 → stop                  → span = 1
Day 3: price=70,  70 > 60 → continue, 70 < 80     → span = 2
Day 4: price=60,  60 < 70 → stop                  → span = 1
Day 5: price=75,  75 > 60, 75 > 70, 75 > 60, 
                  75 < 80 → stop                   → span = 4
Day 6: price=85,  85 > 75, 85 > 60, 85 > 70, 
                  85 > 60, 85 > 80, 85 < 100       → span = 6

Output: [1, 1, 1, 2, 1, 4, 6]
```

---

## Key Insight

```
┌──────────────────────────────────────────────────────────┐
│  Stock Span = Finding PREVIOUS GREATER ELEMENT           │
│                                                          │
│  span[i] = i - index_of_previous_greater_element         │
│                                                          │
│  If no previous greater element exists:                  │
│    span[i] = i + 1 (all days from start)                 │
│                                                          │
│  Stack Type: Monotonically DECREASING                    │
│  (we track elements with no smaller element resolved)    │
└──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION stockSpan(prices):
    n ← length(prices)
    span ← array of size n
    stack ← empty stack    // stores indices, decreasing by price
    
    FOR i = 0 TO n-1:
        // Pop all days with price ≤ current price
        WHILE stack NOT empty AND prices[stack.top()] <= prices[i]:
            stack.pop()
        
        // Span = distance from previous greater price
        IF stack is empty:
            span[i] ← i + 1       // No greater price before
        ELSE:
            span[i] ← i - stack.top()
        
        stack.push(i)
    
    RETURN span
```

---

## Detailed Trace

### Input: prices = [100, 80, 60, 70, 60, 75, 85]

```
═══════════════════════════════════════════
  Day 0: price = 100
═══════════════════════════════════════════
  Stack empty → span[0] = 0 + 1 = 1
  Push 0
  Stack: [0]    Prices: [100]

═══════════════════════════════════════════
  Day 1: price = 80
═══════════════════════════════════════════
  80 ≤ 100? NO → don't pop
  Stack top = 0 → span[1] = 1 - 0 = 1
  Push 1
  Stack: [0, 1]    Prices: [100, 80]

═══════════════════════════════════════════
  Day 2: price = 60
═══════════════════════════════════════════
  60 ≤ 80? NO → don't pop
  Stack top = 1 → span[2] = 2 - 1 = 1
  Push 2
  Stack: [0, 1, 2]    Prices: [100, 80, 60]

  ┌─────┐
  │  60 │ idx 2
  │  80 │ idx 1
  │ 100 │ idx 0
  └─────┘  Decreasing ✓

═══════════════════════════════════════════
  Day 3: price = 70
═══════════════════════════════════════════
  70 ≤ 60? YES → Pop 2 (price 60)
  70 ≤ 80? NO → stop
  Stack top = 1 → span[3] = 3 - 1 = 2
  Push 3
  Stack: [0, 1, 3]    Prices: [100, 80, 70]

═══════════════════════════════════════════
  Day 4: price = 60
═══════════════════════════════════════════
  60 ≤ 70? NO → don't pop
  Stack top = 3 → span[4] = 4 - 3 = 1
  Push 4
  Stack: [0, 1, 3, 4]    Prices: [100, 80, 70, 60]

═══════════════════════════════════════════
  Day 5: price = 75
═══════════════════════════════════════════
  75 ≤ 60? YES → Pop 4 (price 60)
  75 ≤ 70? YES → Pop 3 (price 70)
  75 ≤ 80? NO → stop
  Stack top = 1 → span[5] = 5 - 1 = 4
  Push 5
  Stack: [0, 1, 5]    Prices: [100, 80, 75]

═══════════════════════════════════════════
  Day 6: price = 85
═══════════════════════════════════════════
  85 ≤ 75? YES → Pop 5 (price 75)
  85 ≤ 80? YES → Pop 1 (price 80)
  85 ≤ 100? NO → stop
  Stack top = 0 → span[6] = 6 - 0 = 6
  Push 6
  Stack: [0, 6]    Prices: [100, 85]

═══════════════════════════════════════════
  RESULT: [1, 1, 1, 2, 1, 4, 6]  ✓
═══════════════════════════════════════════
```

---

## Visual: Price Chart with Spans

```
Price
100 │ █
 85 │ │                         █ ←── span=6 (looks back 6 days)
 80 │ │ █                       │
 75 │ │ │           █           │
 70 │ │ │     █     │           │
 60 │ │ │  █  │  █  │           │
    └─┴─┴──┴──┴──┴──┴───────────┴──
      0  1  2  3  4  5           6

Day 3 (price 70): span=2 because 70 ≥ 60 (day 2)
Day 5 (price 75): span=4 because 75 ≥ 60, 70, 60 (days 4,3,2)
Day 6 (price 85): span=6 because 85 ≥ 75, 60, 70, 60, 80 (days 5-1)
```

---

## Online Stock Span (Streaming Variant)

Process prices one at a time (don't have full array upfront).

```
CLASS StockSpanner:
    stack ← empty stack    // stores (price, span) pairs
    
    FUNCTION next(price):
        span ← 1
        
        WHILE stack NOT empty AND stack.top().price <= price:
            span ← span + stack.pop().span
        
        stack.push((price, span))
        RETURN span
```

### Trace:

```
next(100): span=1, push(100,1)   Stack: [(100,1)]
next(80):  span=1, push(80,1)    Stack: [(100,1),(80,1)]
next(60):  span=1, push(60,1)    Stack: [(100,1),(80,1),(60,1)]
next(70):  pop(60,1)→span=1+1=2
           push(70,2)             Stack: [(100,1),(80,1),(70,2)]
next(60):  span=1, push(60,1)    Stack: [(100,1),(80,1),(70,2),(60,1)]
next(75):  pop(60,1)→span=1+1=2
           pop(70,2)→span=2+2=4
           push(75,4)             Stack: [(100,1),(80,1),(75,4)]
next(85):  pop(75,4)→span=1+4=5
           pop(80,1)→span=5+1=6
           push(85,6)             Stack: [(100,1),(85,6)]

Results: [1, 1, 1, 2, 1, 4, 6] ✓
```

---

## Brute Force Comparison

```
Brute Force:
  FOR each day i:
    count ← 1
    j ← i - 1
    WHILE j ≥ 0 AND prices[j] ≤ prices[i]:
      count++
      j--
    span[i] ← count
  
  Time: O(n²) worst case
  
Stack approach: O(n) — each element pushed/popped at most once
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| **Brute Force** | O(n²) | O(1) |
| **Stack (batch)** | O(n) | O(n) |
| **Stack (online)** | O(1) amortized per call | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Consecutive days with price ≤ today |
| **Type** | Previous Greater Element |
| **Stack** | Monotonically decreasing |
| **Span Formula** | `i - stack.top()` (or `i+1` if stack empty) |
| **Time** | O(n) |
| **Space** | O(n) |

---

## Quick Revision Questions

1. **What does the stock span represent?**
   > The number of consecutive days before and including the current day where the price was ≤ the current day's price.

2. **How does stock span relate to monotonic stacks?**
   > It's a Previous Greater Element problem; the span is the distance from the previous day with a strictly greater price.

3. **Why do we use a decreasing stack for this problem?**
   > We need to find the Previous Greater Element. A decreasing stack pops elements ≤ current, leaving the first greater element as the stack top.

4. **How does the online variant differ from the batch variant?**
   > Instead of storing indices, it stores (price, span) pairs. The accumulated span of popped elements is added to the current day's span.

5. **For prices = [30, 20, 25, 28, 27, 29], what are the spans?**
   > [1, 1, 2, 3, 1, 5]. Day 5 (29): consecutive 29≥27, 29≥28, 29≥25, 29≥20, but 29<30.

---

[← Previous: Daily Temperatures](../05-Monotonic-Stack/06-daily-temperatures.md) | [Next: Max Area Histogram →](02-max-area-histogram.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
