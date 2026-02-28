# Chapter 4: Classic Heap Problems

## 📋 Chapter Overview
Essential heap interview problems: merge K sorted lists, task scheduler, reorganize string, and meeting rooms.

---

## 📝 Problem 1: Merge K Sorted Lists (LeetCode 23)

**Problem:** Merge k sorted linked lists into one sorted list.

```
function mergeKLists(lists):
    minHeap = []
    
    // Add head of each list
    for i = 0 to k-1:
        if lists[i] not null:
            heap.push((lists[i].val, i, lists[i]))
    
    dummy = new ListNode(0)
    current = dummy
    
    while heap not empty:
        (val, idx, node) = heap.poll()
        current.next = node
        current = current.next
        
        if node.next not null:
            heap.push((node.next.val, idx, node.next))
    
    return dummy.next
```

### Trace

```
  lists: [1→4→5], [1→3→4], [2→6]
  
  Init heap: [(1,0,node), (1,1,node), (2,2,node)]
  
  Poll (1,0): output 1. Push (4,0). heap=[(1,1),(2,2),(4,0)]
  Poll (1,1): output 1. Push (3,1). heap=[(2,2),(3,1),(4,0)]
  Poll (2,2): output 2. Push (6,2). heap=[(3,1),(4,0),(6,2)]
  Poll (3,1): output 3. Push (4,1). heap=[(4,0),(4,1),(6,2)]
  Poll (4,0): output 4. Push (5,0). heap=[(4,1),(5,0),(6,2)]
  Poll (4,1): output 4. No next.    heap=[(5,0),(6,2)]
  Poll (5,0): output 5. No next.    heap=[(6,2)]
  Poll (6,2): output 6. No next.    heap=[]
  
  Result: 1→1→2→3→4→4→5→6
```

**Complexity:** O(N log k) where N = total nodes, k = number of lists

---

## 📝 Problem 2: Ugly Number II (LeetCode 264)

**Problem:** Find nth ugly number (factors only 2, 3, 5). Sequence: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12...

### Approach: Three Pointers (Optimal)

```
function nthUglyNumber(n):
    dp = array of size n
    dp[0] = 1
    i2 = 0, i3 = 0, i5 = 0
    
    for i = 1 to n-1:
        next2 = dp[i2] * 2
        next3 = dp[i3] * 3
        next5 = dp[i5] * 5
        dp[i] = min(next2, next3, next5)
        
        if dp[i] == next2: i2++
        if dp[i] == next3: i3++  // no else — handle duplicates
        if dp[i] == next5: i5++
    
    return dp[n-1]
```

### Trace

```
  i=1: next2=2, next3=3, next5=5. dp[1]=2. i2=1
  i=2: next2=4, next3=3, next5=5. dp[2]=3. i3=1
  i=3: next2=4, next3=6, next5=5. dp[3]=4. i2=2
  i=4: next2=6, next3=6, next5=5. dp[4]=5. i5=1
  i=5: next2=6, next3=6, next5=10. dp[5]=6. i2=3, i3=2
  ...
```

### Approach: Min Heap

```
function nthUglyNumberHeap(n):
    minHeap = [1]
    seen = {1}
    
    for i = 0 to n-1:
        val = heap.poll()
        if i == n-1: return val
        for factor in [2, 3, 5]:
            next = val * factor
            if next not in seen:
                seen.add(next)
                heap.push(next)
```

---

## 📝 Problem 3: Find K Pairs with Smallest Sums (LeetCode 373)

**Problem:** Two sorted arrays. Find k pairs (u,v) with smallest sums u+v.

```
function kSmallestPairs(nums1, nums2, k):
    minHeap = []
    result = []
    
    // Start with first element of nums1 paired with all of nums2[0]
    for i = 0 to min(k, len(nums1)) - 1:
        heap.push((nums1[i] + nums2[0], i, 0))
    
    while k > 0 and heap not empty:
        (sum, i, j) = heap.poll()
        result.add([nums1[i], nums2[j]])
        k--
        
        if j + 1 < len(nums2):
            heap.push((nums1[i] + nums2[j+1], i, j+1))
    
    return result
```

### Trace

```
  nums1 = [1,7,11], nums2 = [2,4,6], k = 3
  
  Init: heap = [(1+2,0,0)=3, (7+2,1,0)=9, (11+2,2,0)=13]
  
  Poll (3,0,0): result=[[1,2]]. Push (1+4,0,1)=5.
  heap = [(5,0,1), (9,1,0), (13,2,0)]
  
  Poll (5,0,1): result=[[1,2],[1,4]]. Push (1+6,0,2)=7.
  heap = [(7,0,2), (9,1,0), (13,2,0)]
  
  Poll (7,0,2): result=[[1,2],[1,4],[1,6]].
  
  Answer: [[1,2],[1,4],[1,6]]
```

---

## 📝 Problem 4: Reorganize String — Heap Approach (LeetCode 767)

```
function reorganizeString(s):
    freqMap = count frequencies
    maxHeap = [(freq, char) for each char]
    
    if any freq > (len(s) + 1) / 2: return ""
    
    result = []
    prev = null
    
    while maxHeap not empty:
        (freq, ch) = maxHeap.poll()
        result.add(ch)
        freq--
        
        if prev and prev.freq > 0:
            maxHeap.push(prev)
        prev = (freq, ch)
    
    return join(result)
```

**Key insight:** Always pick the most frequent character that isn't the same as the previous one. Hold back the previous character for one round.

---

## 📝 Problem 5: Smallest Range Covering K Lists (LeetCode 632)

**Problem:** K sorted lists. Find the smallest range [a,b] such that at least one element from each list is in [a,b].

```
function smallestRange(nums):
    minHeap = []
    maxVal = -INF
    
    // Add first element of each list
    for i = 0 to k-1:
        heap.push((nums[i][0], i, 0))
        maxVal = max(maxVal, nums[i][0])
    
    rangeStart = 0, rangeEnd = INF
    
    while true:
        (minVal, listIdx, elemIdx) = heap.poll()
        
        if maxVal - minVal < rangeEnd - rangeStart:
            rangeStart = minVal
            rangeEnd = maxVal
        
        if elemIdx + 1 == len(nums[listIdx]):
            break   // one list exhausted
        
        nextVal = nums[listIdx][elemIdx + 1]
        heap.push((nextVal, listIdx, elemIdx + 1))
        maxVal = max(maxVal, nextVal)
    
    return [rangeStart, rangeEnd]
```

**Complexity:** O(N log k) where N = total elements

---

## 📊 Classic Heap Problems Summary

| Problem | Heap Type | Time | Key Idea |
|---------|----------|------|----------|
| Merge K sorted lists | Min heap of size k | O(N log k) | Always extract global minimum |
| Ugly Number II | Min heap (or 3 pointers) | O(n log n) or O(n) | Generate candidates by multiplying |
| K Pairs smallest sums | Min heap | O(k log k) | BFS-like expansion in sum grid |
| Reorganize string | Max heap | O(n log 26) | Alternate most frequent chars |
| Smallest range K lists | Min heap + track max | O(N log k) | Slide window via heap |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Merge K lists | Min heap keeps track of k candidates; extract smallest |
| Ugly numbers | Three pointers avoids duplicates; O(n) |
| K smallest pairs | Expand in one dimension; BFS over sum grid |
| Reorganize | Hold-back previous character; use most frequent |
| Smallest range | Track both min (heap root) and max (variable) |

---

## ❓ Revision Questions

1. **Merge K lists: why O(N log k)?** → N total extractions, each involving heap operations on a heap of size k.
2. **Ugly Number: why three pointers not heap?** → O(n) vs O(n log n); avoids duplicate generation.
3. **K pairs: why not generate all pairs?** → That's O(m×n). Heap approach only generates up to k pairs: O(k log k).
4. **Reorganize: impossibility check?** → If any character's frequency > ceil(n/2), impossible — not enough slots to separate them.
5. **Smallest range: why track max separately?** → Heap only gives min efficiently. Max is tracked separately and updated on each insertion.

---

[← Previous: Two-Heap Pattern](03-two-heap-pattern.md) | [Next: When to Apply →](05-when-to-apply.md)
