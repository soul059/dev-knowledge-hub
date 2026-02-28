# Chapter 1: Accounts Merge (LeetCode 721)

## Overview

**Accounts Merge** is a classic Union-Find problem. Given a list of accounts where each account has a name and a list of emails, merge accounts that belong to the same person (sharing at least one email).

---

## Problem Statement

```
Input: accounts = [
    ["John", "john1@mail.com", "john2@mail.com"],
    ["John", "john3@mail.com"],
    ["John", "john1@mail.com", "john4@mail.com"],
    ["Mary", "mary@mail.com"]
]

Output: [
    ["John", "john1@mail.com", "john2@mail.com", "john4@mail.com"],
    ["John", "john3@mail.com"],
    ["Mary", "mary@mail.com"]
]

Account 0 and Account 2 share "john1@mail.com" → same person.
Account 1 has a different John (no shared email).
```

---

## Modeling with Union-Find

```
Key Insight: An EMAIL is the node, not a person.
  - Map each email to an owner account index
  - If email already has an owner → Union current account 
    with that owner
  - All emails in the same component = same person

Step-by-step:

  Account 0: john1@mail.com → owner[john1] = 0
             john2@mail.com → owner[john2] = 0
             (union(0, 0) no-op)
             
  Account 1: john3@mail.com → owner[john3] = 1

  Account 2: john1@mail.com → already owned by 0
             → Union(2, 0) ← accounts 0 and 2 merge!
             john4@mail.com → owner[john4] = 2 (root = 0)

  Account 3: mary@mail.com → owner[mary] = 3

  Components:
    {0, 2} → emails: john1, john2, john4
    {1}    → emails: john3
    {3}    → emails: mary
```

---

## Solution: Union-Find on Account Indices

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return
        if self.rank[rx] < self.rank[ry]: rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]: self.rank[rx] += 1

def accountsMerge(accounts):
    n = len(accounts)
    uf = UnionFind(n)
    
    # Map: email → first account index that owns it
    email_to_id = {}
    
    for i, account in enumerate(accounts):
        for email in account[1:]:
            if email in email_to_id:
                uf.union(i, email_to_id[email])
            else:
                email_to_id[email] = i
    
    # Group emails by root account
    from collections import defaultdict
    groups = defaultdict(set)
    for email, owner in email_to_id.items():
        root = uf.find(owner)
        groups[root].add(email)
    
    # Build result
    result = []
    for root, emails in groups.items():
        name = accounts[root][0]
        result.append([name] + sorted(emails))
    
    return result
```

---

## Trace Walkthrough

```
accounts = [
  0: ["John", "A", "B"],
  1: ["John", "C"],
  2: ["John", "A", "D"],
  3: ["Mary", "E"]
]

Processing:
  i=0: email A → email_to_id[A]=0
       email B → email_to_id[B]=0

  i=1: email C → email_to_id[C]=1

  i=2: email A → already in map, owner=0
       → Union(2, 0) → parent[2] = 0
       email D → email_to_id[D]=2

  i=3: email E → email_to_id[E]=3

email_to_id = {A:0, B:0, C:1, D:2, E:3}

Grouping by root:
  A → find(0) = 0 → groups[0].add(A)
  B → find(0) = 0 → groups[0].add(B)
  C → find(1) = 1 → groups[1].add(C)
  D → find(2) = 0 → groups[0].add(D)  ← find(2) = 0!
  E → find(3) = 3 → groups[3].add(E)

groups = {
  0: {A, B, D},
  1: {C},
  3: {E}
}

Result:
  ["John", "A", "B", "D"]
  ["John", "C"]
  ["Mary", "E"]
```

---

## Alternative: Union-Find on Emails Directly

```python
def accountsMerge(accounts):
    uf_parent = {}
    
    def find(x):
        if x not in uf_parent:
            uf_parent[x] = x
        if uf_parent[x] != x:
            uf_parent[x] = find(uf_parent[x])
        return uf_parent[x]
    
    def union(x, y):
        uf_parent[find(x)] = find(y)
    
    email_to_name = {}
    
    for account in accounts:
        name = account[0]
        first_email = account[1]
        for email in account[1:]:
            email_to_name[email] = name
            union(email, first_email)
    
    # Group by root email
    from collections import defaultdict
    groups = defaultdict(list)
    for email in email_to_name:
        groups[find(email)].append(email)
    
    return [[email_to_name[root]] + sorted(emails) 
            for root, emails in groups.items()]
```

---

## Complexity Analysis

```
n = number of accounts
k = total number of emails across all accounts

Time:
  Building email_to_id: O(k × α(n))
  Grouping: O(k × α(n))
  Sorting emails per group: O(k log k)
  Total: O(k log k) — dominated by sorting

Space:
  email_to_id map: O(k)
  Union-Find: O(n)
  Groups: O(k)
  Total: O(k)
```

---

## Edge Cases

```
1. Same email in many accounts → all merge into one
2. No shared emails → each account stays separate  
3. Transitive merging: A shares with B, B shares with C
   → all three merge
4. Duplicate emails in same account → no effect
5. Single account → return as-is (sorted)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Merge accounts sharing emails |
| DSU nodes | Account indices OR emails directly |
| Union trigger | Same email in multiple accounts |
| Key map | email → account_id |
| Output | Grouped + sorted emails with name |
| Complexity | O(k log k) time, O(k) space |

---

## Quick Revision Questions

1. **What are the "nodes" in the Union-Find for this problem?**
2. **When do you perform a Union?**
3. **How do you collect all emails for a merged account?**
4. **Why do you sort the final email lists?**
5. **What's the time complexity and what dominates it?**

---

[← Previous: 2-SAT Concept](../08-Advanced-DSU/06-two-sat-concept.md) | [Next: Sentence Similarity →](02-sentence-similarity.md)
