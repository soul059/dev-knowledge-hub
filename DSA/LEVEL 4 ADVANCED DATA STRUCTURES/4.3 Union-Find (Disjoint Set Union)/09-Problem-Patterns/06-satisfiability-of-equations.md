# Chapter 6: Satisfiability of Equality Equations (LeetCode 990)

## Overview

Given an array of strings representing equations between lowercase variables (like `"a==b"` or `"a!=b"`), determine if it's possible to assign integers to variables satisfying all equations simultaneously.

---

## Problem Statement

```
Input: ["a==b", "b!=a"]
Output: false
Explanation: a==b implies a and b are equal, 
             but b!=a says they're not. Contradiction!

Input: ["b==a", "a==b"]
Output: true
Explanation: Both say a equals b. Consistent.

Input: ["a==b", "b==c", "a==c"]
Output: true
Explanation: All say a=b=c. Consistent.

Input: ["a==b", "b!=c", "c==a"]
Output: false  
Explanation: a==b, c==a → b==c, but b!=c. Contradiction!
```

---

## The Two-Pass Algorithm

```
Core insight: 
  "==" creates equivalence classes (transitive)
  "!=" asserts two variables are in DIFFERENT classes

Algorithm:
  Pass 1: Process all "==" equations
    → Union the two variables
    
  Pass 2: Check all "!=" equations
    → If Find(x) == Find(y), CONTRADICTION!
    
Why this order?
  We need the FULL equivalence classes before checking
  inequalities. An early != might look fine but become
  a contradiction after later == equations.
```

---

## Implementation

```python
def equationsPossible(equations):
    parent = list(range(26))  # 26 lowercase letters
    
    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    
    def union(x, y):
        rx, ry = find(x), find(y)
        if rx != ry:
            parent[rx] = ry
    
    # Pass 1: Process equalities
    for eq in equations:
        if eq[1] == '=':  # "x==y"
            x = ord(eq[0]) - ord('a')
            y = ord(eq[3]) - ord('a')
            union(x, y)
    
    # Pass 2: Check inequalities
    for eq in equations:
        if eq[1] == '!':  # "x!=y"
            x = ord(eq[0]) - ord('a')
            y = ord(eq[3]) - ord('a')
            if find(x) == find(y):
                return False
    
    return True
```

---

## Detailed Trace

```
Input: ["a==b", "b!=c", "c==a"]

Pass 1 (equalities only):
  "a==b" → union(0, 1) → parent[0] = 1
  "c==a" → union(2, 0) → find(2)=2, find(0)=1 
                        → parent[2] = 1
  
  Groups: {a, b, c} all in same group

Pass 2 (inequalities):
  "b!=c" → find(1)=1, find(2)=1
         → SAME GROUP! Contradiction!
         
Return: false ✓

---

Input: ["c==c", "b==d", "x!=z"]

Pass 1:
  "c==c" → union(2, 2) → no-op
  "b==d" → union(1, 3) → parent[1] = 3

Pass 2:
  "x!=z" → find(23)=23, find(25)=25
         → Different groups → OK

Return: true ✓
```

---

## Edge Cases

```
1. "a==a": Always true (trivially satisfied)
2. "a!=a": Always false (contradiction with reflexivity)
3. Empty input: Return true (vacuously true)
4. Only equalities: Always true
5. Only inequalities between distinct variables: Always true
   (assign different values to each)
6. Long chains: "a==b", "b==c", ..., "y==z", "a!=z" → false
```

```python
# Special case: self-inequality
for eq in equations:
    if eq[1] == '!' and eq[0] == eq[3]:
        return False  # "a!=a" is always false
```

---

## Generalized Pattern: Constraint Satisfaction with DSU

```
Template for equality/inequality constraint problems:

  def satisfiable(n, equalities, inequalities):
      uf = UnionFind(n)
      
      # Phase 1: Build equivalence classes
      for x, y in equalities:
          uf.union(x, y)
      
      # Phase 2: Check conflicts
      for x, y in inequalities:
          if uf.connected(x, y):
              return False
      
      return True

This pattern appears in:
  ✓ Satisfiability of Equality Equations (LC 990)
  ✓ Sentence Similarity II (LC 737)
  ✓ Graph coloring constraints
  ✓ Type inference in compilers
  ✓ Database equality constraints
```

---

## C++ Solution

```cpp
class Solution {
public:
    vector<int> parent;
    
    int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    
    bool equationsPossible(vector<string>& equations) {
        parent.resize(26);
        iota(parent.begin(), parent.end(), 0);
        
        // Pass 1: equalities
        for (auto& eq : equations) {
            if (eq[1] == '=') {
                int x = eq[0] - 'a', y = eq[3] - 'a';
                parent[find(x)] = find(y);
            }
        }
        
        // Pass 2: inequalities
        for (auto& eq : equations) {
            if (eq[1] == '!') {
                int x = eq[0] - 'a', y = eq[3] - 'a';
                if (find(x) == find(y)) return false;
            }
        }
        
        return true;
    }
};
```

---

## Java Solution

```java
class Solution {
    int[] parent = new int[26];
    
    int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    
    public boolean equationsPossible(String[] equations) {
        for (int i = 0; i < 26; i++) parent[i] = i;
        
        // Pass 1
        for (String eq : equations) {
            if (eq.charAt(1) == '=') {
                int x = eq.charAt(0) - 'a';
                int y = eq.charAt(3) - 'a';
                parent[find(x)] = find(y);
            }
        }
        
        // Pass 2
        for (String eq : equations) {
            if (eq.charAt(1) == '!') {
                int x = eq.charAt(0) - 'a';
                int y = eq.charAt(3) - 'a';
                if (find(x) == find(y)) return false;
            }
        }
        
        return true;
    }
}
```

---

## Complexity Analysis

```
Let E = number of equations

Time: O(E × α(26)) = O(E)
  - 26 variables at most → essentially O(1) per find/union
  - Two passes through equations → O(E)

Space: O(1)
  - parent array of size 26 (constant)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Check equation satisfiability |
| Variables | 26 lowercase letters |
| Pass 1 | Process `==` with Union |
| Pass 2 | Check `!=` for contradictions |
| Key rule | Equalities BEFORE inequalities |
| Complexity | O(E) time, O(1) space |
| Pattern | Equality/inequality constraint satisfaction |

---

## Quick Revision Questions

1. **Why must equalities be processed before inequalities?**
2. **What happens if a!=a appears in the input?**
3. **How many nodes does the Union-Find need?**
4. **Give an example of a transitive contradiction.**
5. **What other problems use this two-pass equality/inequality pattern?**

---

[← Previous: Swim in Rising Water](05-swim-in-rising-water.md)

[← Back to Main README](../README.md)

---

## 🎓 Congratulations!

You have completed the comprehensive Union-Find (Disjoint Set Union) study notes! 

### What You've Learned:
1. **Fundamentals** — What DSU is and when to use it
2. **Basic Implementation** — Find and Union operations
3. **Path Compression** — Flattening trees for speed
4. **Union by Rank/Size** — Keeping trees balanced
5. **Connected Components** — Dynamic connectivity problems
6. **Cycle Detection** — Finding cycles with DSU
7. **MST with Union-Find** — Kruskal's algorithm
8. **Advanced Techniques** — Weighted DSU, rollback, offline queries
9. **Problem Patterns** — Classic competitive programming problems

### Key Takeaway:
> **Union-Find is the go-to data structure whenever you need to dynamically merge groups and query connectivity. Master the template, recognize the patterns, and practice the problems!**
