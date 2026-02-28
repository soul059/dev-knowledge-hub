# 5.4 Custom Comparators

[← Previous: Update Priority](03-update-priority.md) | [Next: Standard Library Usage →](05-standard-library-usage.md)

---

## Why Custom Comparators?

Real-world priorities are often **complex** — not just a single number. Examples:
- Hospital ER: Severity first, then arrival time
- OS Scheduling: Priority class, then deadline
- Gaming: Threat level, then distance

A **custom comparator** tells the heap how to order elements.

---

## Multi-Level Priority (Tie-Breaking)

When two elements share the same primary priority, a **secondary key** breaks the tie.

```
Hospital ER Triage Example:

Patient     Severity    Arrival
─────────────────────────────────
Alice       CRITICAL    10:05
Bob         MODERATE    10:01
Carol       CRITICAL    10:02
Dave        LOW         10:00
Eve         MODERATE    10:03

Ordering (severity first, then arrival):
1. Carol    CRITICAL    10:02    ← earliest critical
2. Alice    CRITICAL    10:05
3. Bob      MODERATE    10:01    ← earliest moderate
4. Eve      MODERATE    10:03
5. Dave     LOW         10:00
```

### Pseudocode

```
COMPARE_ER(patient_a, patient_b):
    severity_order = {CRITICAL: 0, MODERATE: 1, LOW: 2}
    
    IF severity_order[a.severity] != severity_order[b.severity]:
        RETURN severity_order[a.severity] < severity_order[b.severity]
    
    // Same severity → earlier arrival wins
    RETURN a.arrival_time < b.arrival_time
```

---

## Python Example: `__lt__` Override

```python
import heapq

class Patient:
    def __init__(self, name, severity, arrival):
        self.name = name
        self.severity = severity   # 0 = critical, 1 = moderate, 2 = low
        self.arrival = arrival
    
    def __lt__(self, other):
        if self.severity != other.severity:
            return self.severity < other.severity
        return self.arrival < other.arrival

# Usage
pq = []
heapq.heappush(pq, Patient("Alice", 0, 10.05))
heapq.heappush(pq, Patient("Bob",   1, 10.01))
heapq.heappush(pq, Patient("Carol", 0, 10.02))

# heapq.heappop(pq).name → "Carol" (critical, earlier)
```

---

## Java Example: `Comparator`

```java
PriorityQueue<Patient> pq = new PriorityQueue<>(
    Comparator.comparingInt(Patient::getSeverity)
              .thenComparingDouble(Patient::getArrival)
);
```

---

## Tuple Trick (Python)

Python compares tuples element by element — ideal for multi-level priority:

```python
import heapq

pq = []
# (severity, arrival, name)
heapq.heappush(pq, (0, 10.05, "Alice"))     # critical
heapq.heappush(pq, (1, 10.01, "Bob"))       # moderate
heapq.heappush(pq, (0, 10.02, "Carol"))     # critical

heapq.heappop(pq)   # → (0, 10.02, "Carol")
heapq.heappop(pq)   # → (0, 10.05, "Alice")
heapq.heappop(pq)   # → (1, 10.01, "Bob")
```

> **Warning**: If non-comparable objects can tie on all previous tuple fields, add a unique tie-breaker (e.g., insertion counter):
> ```python
> heapq.heappush(pq, (priority, counter, item))
> ```

---

## Inverting a Min-Heap to Max-Heap

Most standard libraries provide only a min-heap. To get max behavior:

```
Strategy: Negate the key

Insert 5 → push -5
Insert 3 → push -3
Insert 7 → push -7

Min-heap stores: [-7, -3, -5]
pop() returns -7 → negate → 7  (the max!)
```

```python
# Python max-heap via negation
heapq.heappush(pq, -value)
max_val = -heapq.heappop(pq)
```

---

## Quick Check

1. **Why do we need a unique tie-breaker in a tuple-based PQ?**
   <details><summary>Answer</summary>
   If two entries have equal priority fields and the next element in the tuple is not comparable (e.g., a custom object), Python will raise a TypeError. A unique counter (e.g., insertion order) as the middle element ensures comparison never reaches the uncomparable object.
   </details>

---

[← Previous: Update Priority](03-update-priority.md) | [Next: Standard Library Usage →](05-standard-library-usage.md)
