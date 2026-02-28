# 9.6 Problem-Solving Framework & Summary

[← Previous: Ugly Numbers](05-ugly-numbers.md) | [Next: Unit 10 — D-ary Heap →](../10-Heap-Variations/01-d-ary-heap.md)

---

## Recognizing When to Use a Heap

```
┌─────────────────────────────────────────────┐
│         HEAP PROBLEM INDICATORS             │
├─────────────────────────────────────────────┤
│ • "Most frequent" / "least frequent"        │
│ • "Next available" / "earliest ending"      │
│ • "Schedule" / "minimize idle"              │
│ • "Greedy choice of best/worst element"     │
│ • "Generate sequence in order"              │
│ • "Merge sorted sequences"                  │
│ • "K-th element" / "top K"                  │
│ • "Sliding window" + "min/max/median"       │
└─────────────────────────────────────────────┘
```

---

## Common Strategies

| Strategy | Example Problem |
|----------|----------------|
| **Greedy + heap** | Task scheduler, reorganize string |
| **Cooldown pattern** | Place item, put in "cooldown", re-add later |
| **Event processing** | Meeting rooms, employee free time |
| **Sequence generation** | Ugly numbers, super ugly numbers |
| **Interval management** | Meeting rooms, merge intervals |

---

## The Cooldown Pattern

Both "Reorganize String" and "Task Scheduler" use the same core pattern:

```
1. Extract from heap (best available element)
2. Use the element
3. Hold it out for a cooldown period
4. Re-insert after cooldown

The heap always provides the "best available" element.
```

---

## Real-World Applications

| Application | Problem Pattern |
|-------------|----------------|
| **CPU scheduling** | Task scheduler with cooldown |
| **Operating systems** | Meeting rooms (resource allocation) |
| **Workforce optimization** | Employee free time |
| **Character encoding** | Reorganize for transmission |
| **Mathematical sequences** | Ugly numbers in number theory |
| **Rate limiting** | Cooldown between repeated requests |

---

## Unit 9 Summary Table

| Problem | Heap Type | Key Idea | Time |
|---------|-----------|----------|------|
| Reorganize String | Max-heap (by freq) | Place most frequent + cooldown | O(n log A) |
| Task Scheduler | Max-heap (by freq) | Fill cycles of n+1 slots | O(n) with formula |
| Meeting Rooms II | Min-heap (end times) | Reuse room if meeting ended | O(n log n) |
| Employee Free Time | Min-heap (K-way merge) | Merge all intervals, find gaps | O(N log K) |
| Ugly Numbers | Min-heap | Generate next by multiplying factors | O(n log n) |
| Super Ugly Numbers | Min-heap or multi-pointer | Generalized ugly with K primes | O(n log k) |

---

## Revision Questions

1. **In "Reorganize String", why do we always pick the most frequent character?**
   <details><summary>Answer</summary>
   The most frequent character is hardest to place (most constrained). By placing it first, we ensure it gets spread out as much as possible. If we placed rare characters first, the frequent one might end up adjacent to itself.
   </details>

2. **In the Task Scheduler, what determines whether idle slots are needed?**
   <details><summary>Answer</summary>
   Idle slots appear when the number of distinct tasks available is less than the cooldown period (n+1). If there are enough different tasks to fill each cycle, no idle time is needed.
   </details>

3. **Why does Meeting Rooms II use a min-heap of END times?**
   <details><summary>Answer</summary>
   We want to know which meeting ends earliest. If the earliest-ending meeting finishes before the new meeting starts, we can reuse that room. A min-heap of end times gives us the earliest end time in O(1).
   </details>

4. **How does the "cooldown" pattern generalize across reorganize string and task scheduler?**
   <details><summary>Answer</summary>
   Both require a minimum gap between repeated elements. The pattern is: (1) extract from heap, (2) use the element, (3) hold it out for a cooldown period, (4) re-insert after cooldown. The heap always provides the "best available" element.
   </details>

---

[← Previous: Ugly Numbers](05-ugly-numbers.md) | [Next: Unit 10 — D-ary Heap →](../10-Heap-Variations/01-d-ary-heap.md)
