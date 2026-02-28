# Chapter 1: Hash Map Fundamentals

## 📋 Chapter Overview
Hash map internals, collision handling, operations, and foundational problems: two-sum, group anagrams, and isomorphic strings.

---

## 🧠 How Hash Maps Work

```
  Hash Function: key → integer (hash code) → index in array
  
  key="apple" → hash("apple") = 539172 → 539172 % 16 = 4
  
  Bucket Array (size 16):
  Index: 0   1   2   3   4         5   ...  15
       [ ] [ ] [ ] [ ] [apple→5] [ ] ... [ ]
  
  Collision: two keys map to same index
```

### Collision Resolution

```
  1. Chaining (Linked List):
     Index 4: [apple→5] → [grape→3] → null
  
  2. Open Addressing (Linear Probing):
     Index 4: [apple→5]
     Index 5: [grape→3]   ← next available slot
```

---

## 📐 Operations & Complexity

| Operation | Average | Worst (all collisions) |
|-----------|---------|----------------------|
| Insert | O(1) | O(n) |
| Lookup | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Space | O(n) | O(n) |

> Worst case is rare with good hash functions and load factor management.

---

## 📝 Problem 1: Two Sum (LeetCode 1)

**Problem:** Find two indices whose values sum to target.

```
function twoSum(nums, target):
    map = {}   // value → index
    
    for i = 0 to n-1:
        complement = target - nums[i]
        if complement in map:
            return [map[complement], i]
        map[nums[i]] = i
    
    return []
```

### Trace

```
  nums = [2, 7, 11, 15], target = 9
  
  i=0: complement = 9-2 = 7. Not in map. map = {2:0}
  i=1: complement = 9-7 = 2. Found! map[2] = 0.
       Return [0, 1] ✓
```

**Complexity:** O(n) time, O(n) space — one pass.

---

## 📝 Problem 2: Group Anagrams (LeetCode 49)

**Problem:** Group strings that are anagrams of each other.

```
function groupAnagrams(strs):
    map = {}   // sorted string → list of anagrams
    
    for each str in strs:
        key = sort(str)    // or character count as key
        map[key].add(str)
    
    return map.values()
```

### Trace

```
  strs = ["eat","tea","tan","ate","nat","bat"]
  
  "eat" → key="aet" → map={"aet":["eat"]}
  "tea" → key="aet" → map={"aet":["eat","tea"]}
  "tan" → key="ant" → map={"aet":[...],"ant":["tan"]}
  "ate" → key="aet" → map={"aet":["eat","tea","ate"],...}
  "nat" → key="ant" → map={...,"ant":["tan","nat"]}
  "bat" → key="abt" → map={...,"abt":["bat"]}
  
  Result: [["eat","tea","ate"],["tan","nat"],["bat"]]
```

### Optimized Key: Character Count

```
  Instead of sorting O(k log k), use count array O(k):
  key = "2#1#0#...#0#1#0..." (count of each letter)
  
  For lowercase: 26-element count → convert to tuple/string as key
```

---

## 📝 Problem 3: Isomorphic Strings (LeetCode 205)

**Problem:** Two strings are isomorphic if characters can be mapped one-to-one.

```
function isIsomorphic(s, t):
    mapST = {}   // s char → t char
    mapTS = {}   // t char → s char
    
    for i = 0 to n-1:
        if s[i] in mapST:
            if mapST[s[i]] ≠ t[i]: return false
        else:
            mapST[s[i]] = t[i]
        
        if t[i] in mapTS:
            if mapTS[t[i]] ≠ s[i]: return false
        else:
            mapTS[t[i]] = s[i]
    
    return true
```

### Trace

```
  s = "egg", t = "add"
  i=0: e↔a. mapST={e:a}, mapTS={a:e}
  i=1: g↔d. mapST={e:a,g:d}, mapTS={a:e,d:g}
  i=2: g→d ✓ (mapST[g]=d=t[2]). d→g ✓ (mapTS[d]=g=s[2])
  Result: true
  
  s = "foo", t = "bar"
  i=0: f↔b. OK
  i=1: o↔a. OK
  i=2: o→a but t[2]='r' ≠ 'a'. Return false
```

---

## 📝 Problem 4: Contains Duplicate II (LeetCode 219)

**Problem:** Check if there are two distinct indices i, j where nums[i] == nums[j] and |i-j| ≤ k.

```
function containsNearbyDuplicate(nums, k):
    map = {}   // value → last seen index
    
    for i = 0 to n-1:
        if nums[i] in map and i - map[nums[i]] <= k:
            return true
        map[nums[i]] = i
    
    return false
```

**Alternative:** Use HashSet of size k as sliding window.

---

## 📊 Hash Map vs Other Structures

| Feature | Hash Map | Array | BST | Sorted Array |
|---------|----------|-------|-----|-------------|
| Lookup | O(1) avg | O(1) by index | O(log n) | O(log n) |
| Insert | O(1) avg | O(n) middle | O(log n) | O(n) |
| Ordered | ✗ | By index | ✓ | ✓ |
| Memory | Higher (hash overhead) | Compact | Pointers | Compact |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Hash map | O(1) average lookup/insert via hash function |
| Two Sum | Store complement; single-pass O(n) |
| Group Anagrams | Sort or count characters as key |
| Isomorphic | Two bidirectional maps ensure 1-to-1 mapping |
| Collision | Chaining (linked list) or open addressing (probing) |

---

## ❓ Revision Questions

1. **Hash map vs hash set?** → Map stores key-value pairs; set stores only keys (existence check).
2. **Two Sum: why hash map not brute force?** → Brute force is O(n²); hash map reduces to O(n) by complementary lookup.
3. **Group Anagrams: two key strategies?** → Sort characters (O(k log k)) or character count tuple (O(k)).
4. **Why two maps for isomorphic?** → One map ensures s→t is consistent; second ensures t→s is consistent (bijection).
5. **Load factor?** → Ratio of elements to buckets. When too high (>0.75 typically), rehash to larger array.

---

[← Previous: Heap — When to Apply](../10-Heap-Pattern/05-when-to-apply.md) | [Next: Frequency Counting →](02-frequency-counting.md)
