# Chapter 4: Classic Hash Map Problems

## 📋 Chapter Overview
Advanced hash map problems: LRU cache, copy random list, 4Sum II, and consistent hashing concepts.

---

## 📝 Problem 1: LRU Cache (LeetCode 146)

**Problem:** Design a cache with get/put in O(1). Evict least recently used when capacity exceeded.

```
  Structure: HashMap + Doubly Linked List
  
  HashMap: key → node (O(1) access)
  DLL: maintains usage order (head = most recent, tail = least recent)
  
  ┌───────────────────────────────┐
  │  HEAD ↔ node3 ↔ node1 ↔ TAIL │   (head = MRU, tail = LRU)
  └───────────────────────────────┘
  
  map = {key3: node3, key1: node1}
```

### Pseudocode

```
class LRUCache:
    capacity, map = {}, head (dummy), tail (dummy)
    head.next = tail, tail.prev = head
    
    get(key):
        if key not in map: return -1
        node = map[key]
        moveToHead(node)
        return node.value
    
    put(key, value):
        if key in map:
            node = map[key]
            node.value = value
            moveToHead(node)
        else:
            node = new Node(key, value)
            map[key] = node
            addToHead(node)
            if map.size > capacity:
                lru = removeTail()
                delete map[lru.key]
    
    addToHead(node):
        node.next = head.next
        node.prev = head
        head.next.prev = node
        head.next = node
    
    removeNode(node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    moveToHead(node):
        removeNode(node)
        addToHead(node)
    
    removeTail():
        node = tail.prev
        removeNode(node)
        return node
```

### Trace

```
  capacity = 2
  
  put(1,1): map={1:n1}. DLL: HEAD↔n1↔TAIL
  put(2,2): map={1:n1,2:n2}. DLL: HEAD↔n2↔n1↔TAIL
  get(1):   move n1 to head. DLL: HEAD↔n1↔n2↔TAIL. return 1
  put(3,3): capacity exceeded. Remove tail (n2, key=2).
            map={1:n1,3:n3}. DLL: HEAD↔n3↔n1↔TAIL
  get(2):   not in map → return -1
```

---

## 📝 Problem 2: Copy List with Random Pointer (LeetCode 138)

**Problem:** Deep copy a linked list where each node has a random pointer to any node or null.

```
function copyRandomList(head):
    if head is null: return null
    
    map = {}   // original node → copy node
    
    // Pass 1: create all copy nodes
    curr = head
    while curr:
        map[curr] = new Node(curr.val)
        curr = curr.next
    
    // Pass 2: connect next and random pointers
    curr = head
    while curr:
        map[curr].next = map.get(curr.next)
        map[curr].random = map.get(curr.random)
        curr = curr.next
    
    return map[head]
```

### Trace

```
  Original: A(random→C) → B(random→A) → C(random→null)
  
  Pass 1: map = {A:A', B:B', C:C'}
  Pass 2: A'.next=B', A'.random=C'
          B'.next=C', B'.random=A'
          C'.next=null, C'.random=null
```

**Complexity:** O(n) time, O(n) space

---

## 📝 Problem 3: 4Sum II (LeetCode 454)

**Problem:** Four arrays A,B,C,D. Count tuples (i,j,k,l) where A[i]+B[j]+C[k]+D[l]=0.

```
function fourSumCount(A, B, C, D):
    sumAB = {}   // sum → count
    
    for a in A:
        for b in B:
            sumAB[a+b] = sumAB.getOrDefault(a+b, 0) + 1
    
    count = 0
    for c in C:
        for d in D:
            target = -(c + d)
            if target in sumAB:
                count += sumAB[target]
    
    return count
```

### Trace

```
  A=[1,2], B=[-2,-1], C=[-1,2], D=[0,2]
  
  sumAB: {1-2=-1:1, 1-1=0:1, 2-2=0:1, 2-1=1:1}
       = {-1:1, 0:2, 1:1}
  
  C,D pairs: (-1,0)→target=1. sumAB[1]=1. count=1
             (-1,2)→target=-1. sumAB[-1]=1. count=2
             (2,0)→target=-2. not in map
             (2,2)→target=-4. not in map
  
  Answer: 2
```

**Complexity:** O(n²) time, O(n²) space — reduced from O(n⁴)

---

## 📝 Problem 4: Encode and Decode Strings (LeetCode 271)

```
  Design an algorithm to encode a list of strings to a single string
  and decode it back.

function encode(strs):
    result = ""
    for each str:
        result += len(str) + "#" + str
    return result
    // ["hello","world"] → "5#hello5#world"

function decode(s):
    result = []
    i = 0
    while i < len(s):
        j = indexOf('#', start=i)
        length = parseInt(s[i:j])
        result.add(s[j+1 : j+1+length])
        i = j + 1 + length
    return result
```

---

## 📝 Problem 5: Brick Wall (LeetCode 554)

**Problem:** Draw a vertical line through a brick wall crossing the fewest bricks.

```
function leastBricks(wall):
    edgeCount = {}   // position → number of edges
    
    for each row in wall:
        position = 0
        // Don't count last edge (that's the wall boundary)
        for i = 0 to len(row) - 2:
            position += row[i]
            edgeCount[position] = edgeCount.getOrDefault(position, 0) + 1
    
    maxEdges = max(edgeCount.values()) or 0
    return len(wall) - maxEdges
```

**Key insight:** Line through most edges crosses fewest bricks. Track edge positions.

---

## 📊 Classic Problems Summary

| Problem | Hash Map Stores | Time | Space |
|---------|----------------|------|-------|
| LRU Cache | key → DLL node | O(1) per op | O(capacity) |
| Copy random list | original → copy node | O(n) | O(n) |
| 4Sum II | pair sums → count | O(n²) | O(n²) |
| Encode/Decode | N/A (length prefix) | O(total chars) | O(total chars) |
| Brick Wall | edge position → count | O(total bricks) | O(total bricks) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| LRU Cache | HashMap + DLL = O(1) get/put with eviction |
| Deep copy | Map original→copy, then wire pointers in second pass |
| 4Sum II | Pre-compute pair sums; reduce O(n⁴) to O(n²) |
| Encode | Length-prefix encoding avoids delimiter conflicts |
| Brick Wall | Count edges, not bricks; minimize = maximize edges |

---

## ❓ Revision Questions

1. **LRU Cache: why DLL + HashMap?** → HashMap gives O(1) access by key. DLL gives O(1) move-to-front and remove-tail for LRU eviction.
2. **Copy random list: why two passes?** → First pass creates all nodes (so random pointers have targets). Second pass wires next and random.
3. **4Sum II: why split into pairs?** → Four nested loops = O(n⁴). Two pairs: compute A+B sums, then look up -(C+D) = O(n²).
4. **Encode: why length prefix?** → Handles any character in strings including '#' or newlines; unambiguous parsing.
5. **Brick Wall: why subtract from height?** → maxEdges = lines passing through edges (not bricks). Bricks crossed = rows - edges at that position.

---

[← Previous: Hash Set Techniques](03-hashset-techniques.md) | [Next: When to Apply →](05-when-to-apply.md)
