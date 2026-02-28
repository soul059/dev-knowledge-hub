# Chapter 5: Classic Greedy Problems

## 📋 Chapter Overview
In-depth walkthroughs of classic greedy problems: Huffman coding, job scheduling, partition labels, and reorganize string.

---

## 📝 Problem 1: Huffman Coding

**Problem:** Build optimal prefix-free encoding. More frequent characters get shorter codes.

```
  Greedy: always merge the two least-frequent nodes.
  
function huffman(chars, freq):
    minHeap = build min-heap from (freq, char) pairs
    
    while heap.size > 1:
        left = heap.poll()    // smallest
        right = heap.poll()   // second smallest
        merged = new Node(freq = left.freq + right.freq,
                         left = left, right = right)
        heap.add(merged)
    
    return heap.poll()  // root of Huffman tree
```

### Trace

```
  chars: a(5), b(9), c(12), d(13), e(16), f(45)
  
  Merge a(5)+b(9) = ab(14)
  Heap: [c(12), d(13), ab(14), e(16), f(45)]
  
  Merge c(12)+d(13) = cd(25)
  Heap: [ab(14), e(16), cd(25), f(45)]
  
  Merge ab(14)+e(16) = abe(30)
  Heap: [cd(25), abe(30), f(45)]
  
  Merge cd(25)+abe(30) = cdabe(55)
  Heap: [f(45), cdabe(55)]
  
  Merge f(45)+cdabe(55) = root(100)
  
  Resulting codes:
  f: 0        (1 bit)   — most frequent
  c: 100      (3 bits)
  d: 101      (3 bits)
  a: 1100     (4 bits)
  b: 1101     (4 bits)  — least frequent
  e: 111      (3 bits)
```

**Complexity:** O(n log n)

---

## 📝 Problem 2: Job Sequencing with Deadlines

**Problem:** Jobs have profit and deadline. Each job takes 1 unit. Maximize total profit.

```
function jobSequencing(jobs):
    sort jobs by profit descending
    maxDeadline = max of all deadlines
    slots = array of size maxDeadline+1, filled with -1
    
    totalProfit = 0, count = 0
    for each job:
        // Find latest available slot ≤ deadline
        for t = job.deadline down to 1:
            if slots[t] == -1:
                slots[t] = job.id
                totalProfit += job.profit
                count++
                break
    
    return (count, totalProfit)
```

### Trace

```
  Jobs: J1(100,2) J2(19,1) J3(27,2) J4(25,1) J5(15,3)
  Sorted by profit: J1(100,2) J3(27,2) J4(25,1) J2(19,1) J5(15,3)
  
  Slots: [_, _, _, _]  (index 1 to 3)
  
  J1(100, deadline=2): slot 2 free → slots=[_,_,J1,_] profit=100
  J3(27, deadline=2):  slot 2 taken, slot 1 free → slots=[_,J3,J1,_] profit=127
  J4(25, deadline=1):  slot 1 taken → SKIP
  J2(19, deadline=1):  slot 1 taken → SKIP
  J5(15, deadline=3):  slot 3 free → slots=[_,J3,J1,J5] profit=142
  
  Result: 3 jobs, profit = 142
```

---

## 📝 Problem 3: Partition Labels (LeetCode 763)

**Problem:** Partition string so each letter appears in at most one part. Maximize number of parts.

```
function partitionLabels(s):
    lastIndex = map of char → last occurrence in s
    
    result = []
    start = 0, end = 0
    
    for i = 0 to len(s)-1:
        end = max(end, lastIndex[s[i]])
        if i == end:
            result.add(end - start + 1)
            start = i + 1
    
    return result
```

### Trace

```
  s = "ababcbacadefegdehijhklij"
  
  lastIndex: a=8, b=5, c=7, d=14, e=15, f=11, g=13, h=19, i=22, j=23, k=20, l=21
  
  i=0 'a': end = max(0,8) = 8
  i=1 'b': end = max(8,5) = 8
  ...
  i=8 'a': end = max(8,8) = 8.  i==end → partition size = 8-0+1 = 9
  i=9 'd': end = max(9,14) = 14
  ...
  i=15 'e': end = max(15,15) = 15. i==end → partition size = 15-9+1 = 7
  i=16 'h': end = max(16,19) = 19
  ...
  i=23 'j': end = max(23,23) = 23. i==end → partition size = 23-16+1 = 8
  
  Result: [9, 7, 8]
```

**Complexity:** O(n) time, O(1) space (26 chars)

---

## 📝 Problem 4: Reorganize String (LeetCode 767)

**Problem:** Rearrange string so no two adjacent chars are the same. Return "" if impossible.

```
function reorganizeString(s):
    count = frequency map
    maxHeap = build max-heap from (freq, char)
    
    // Impossible if any char frequency > (n+1)/2
    if any freq > (n+1)/2: return ""
    
    result = []
    prev = null   // previously placed (freq, char)
    
    while maxHeap not empty:
        curr = maxHeap.poll()    // most frequent available
        result.add(curr.char)
        curr.freq--
        
        if prev and prev.freq > 0:
            maxHeap.add(prev)    // put back previous
        prev = curr
    
    return result as string
```

### Trace

```
  s = "aab"
  count: a=2, b=1
  
  Round 1: poll a(2). result="a". prev=a(1)
  Round 2: poll b(1). result="ab". push a(1). prev=b(0)
  Round 3: poll a(1). result="aba". prev=a(0). b(0) not pushed.
  
  Result: "aba"
```

---

## 📝 Problem 5: Queue Reconstruction by Height (LeetCode 406)

**Problem:** People described by `[h, k]` — height h, k people in front who are taller or equal. Reconstruct queue.

```
function reconstructQueue(people):
    // Sort: tallest first, then by k ascending
    sort people by (-h, k)
    
    result = []
    for each person [h, k]:
        result.insert(at index k, person)
    
    return result
```

### Trace

```
  Input: [[7,0],[4,4],[7,1],[5,0],[6,1],[5,2]]
  Sorted: [7,0] [7,1] [6,1] [5,0] [5,2] [4,4]
  
  Insert [7,0] at index 0: [[7,0]]
  Insert [7,1] at index 1: [[7,0],[7,1]]
  Insert [6,1] at index 1: [[7,0],[6,1],[7,1]]
  Insert [5,0] at index 0: [[5,0],[7,0],[6,1],[7,1]]
  Insert [5,2] at index 2: [[5,0],[7,0],[5,2],[6,1],[7,1]]
  Insert [4,4] at index 4: [[5,0],[7,0],[5,2],[6,1],[4,4],[7,1]]
```

**Key insight:** Process tallest first — when inserting shorter people later, they don't affect the k-count of taller people already placed.

---

## 📊 Classic Problems Summary

| Problem | Strategy | Time | Space |
|---------|----------|------|-------|
| Huffman coding | Min-heap, merge two smallest | O(n log n) | O(n) |
| Job sequencing | Sort by profit, fill latest slot | O(n²) or O(n log n) with DSU | O(n) |
| Partition labels | Track last occurrence, extend partition | O(n) | O(1) |
| Reorganize string | Max-heap, alternate most frequent | O(n log k) | O(k) |
| Queue reconstruction | Sort by height desc, insert by k | O(n²) | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Huffman | Optimal prefix codes via repeated merging of least frequent |
| Job scheduling | Process highest value first, fill latest available slot |
| Partition labels | Greedy extend partition until all chars contained |
| Reorganize string | Always place most frequent char; hold-back technique |
| Queue by height | Insert tallest first; shorter people don't affect taller counts |

---

## ❓ Revision Questions

1. **Huffman: why merge two smallest?** → Creates longest codes for least frequent chars, minimizing total encoding length.
2. **Job sequencing: why latest available slot?** → Preserves earlier slots for jobs with tighter deadlines.
3. **Partition labels: how to know when to cut?** → When current index equals the maximum last-occurrence of all chars seen in current partition.
4. **Reorganize string: impossibility condition?** → If any character frequency > ceil(n/2), no valid arrangement exists.
5. **Queue reconstruction: why tallest first?** → Inserting shorter people later at index k doesn't disturb the k-counts of taller people already placed.

---

[← Previous: Template and Variations](04-template-and-variations.md) | [Next: When to Apply →](06-when-to-apply.md)
