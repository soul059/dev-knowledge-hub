# 9.1 Consistent Hashing

## Unit 9: Advanced Hashing

---

## The Problem

> Distribute data across $N$ servers. When a server is added or removed, minimize the number of keys that need to be moved.

```
╔══════════════════════════════════════════════════════════════╗
║  NAIVE HASHING: server = hash(key) % N                     ║
║                                                              ║
║  N=3: key→server mapping: {a→0, b→1, c→2, d→0, e→1, f→2}║
║                                                              ║
║  Add a server (N=4): ALL mappings change!                   ║
║  {a→?, b→?, c→?, d→?, e→?, f→?}                           ║
║  Almost every key must be moved → CATASTROPHIC             ║
║                                                              ║
║  CONSISTENT HASHING: Only K/N keys move on average          ║
║  (K = total keys, N = total servers)                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## The Hash Ring

Map both **servers** and **keys** onto a circular hash space $[0, 2^{32})$:

```
╔══════════════════════════════════════════════════════════════╗
║                     0                                        ║
║                  ╱     ╲                                     ║
║               ╱           ╲                                  ║
║            S₁               S₂                               ║
║           ╱                   ╲                              ║
║          │                     │                             ║
║          │      ← Keys in     │                             ║
║          │    this arc go      │                             ║
║          │     to S₂          │                             ║
║           ╲                   ╱                              ║
║            S₃               ╱                                ║
║               ╲           ╱                                  ║
║                  ╲     ╱                                     ║
║                   2^32                                       ║
║                                                              ║
║  Rule: A key is assigned to the FIRST server encountered    ║
║        when walking CLOCKWISE from the key's position.      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm

```
CLASS ConsistentHash:
    ring = new TreeMap()     // position → server
    numReplicas = 150        // virtual nodes per server

    FUNCTION addServer(server):
        FOR i = 0 TO numReplicas - 1:
            pos = hash(server + ":" + i)
            ring[pos] = server

    FUNCTION removeServer(server):
        FOR i = 0 TO numReplicas - 1:
            pos = hash(server + ":" + i)
            ring.remove(pos)

    FUNCTION getServer(key):
        IF ring.isEmpty(): RETURN null

        pos = hash(key)
        // Find first server clockwise from key position
        entry = ring.ceilingEntry(pos)
        IF entry == null:
            entry = ring.firstEntry()    // Wrap around

        RETURN entry.value
```

---

## Adding / Removing a Server

```
╔══════════════════════════════════════════════════════════════╗
║  BEFORE (3 servers):                                        ║
║                                                              ║
║           S1 (pos 100)                                      ║
║          ╱              ╲                                    ║
║        ╱                  ╲                                  ║
║      ╱    k₁(50)→S1       ╲                                ║
║     │     k₂(120)→S2       │                                ║
║     │                      S2 (pos 200)                     ║
║     │     k₃(250)→S3       │                                ║
║      ╲                    ╱                                  ║
║        ╲                ╱                                    ║
║          S3 (pos 300) ╱                                      ║
║                                                              ║
║  ADD S4 at position 150:                                    ║
║  Only keys in (100, 150] are affected!                      ║
║  k₂(120) was going to S2, now goes to S4                   ║
║  k₁ and k₃ are UNCHANGED                                   ║
║                                                              ║
║  Keys moved: only those between S1 and S4 on the ring      ║
║  Expected: K/N keys (not K)                                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Virtual Nodes

Without virtual nodes, servers may be unevenly distributed on the ring:

```
╔══════════════════════════════════════════════════════════════╗
║  WITHOUT virtual nodes (3 servers):                         ║
║                                                              ║
║     ┌─── S1 (10) ─── S2 (20) ─────────────────────┐       ║
║     └──────────────────────── S3 (350) ────────────┘       ║
║                                                              ║
║     S1 handles:  (350, 10]  → tiny range                   ║
║     S2 handles:  (10, 20]   → tiny range                   ║
║     S3 handles:  (20, 350]  → HUGE range (83% of keys!)  ║
║                                                              ║
║  WITH virtual nodes (3 per server):                         ║
║     S1: positions 10, 120, 260                              ║
║     S2: positions 20, 150, 310                              ║
║     S3: positions 70, 200, 350                              ║
║                                                              ║
║     Much more even distribution!                            ║
║     Industry standard: 100-200 virtual nodes per server     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Trace: Complete Workflow

```
Ring starts empty.

1. addServer("A"):
   Virtual nodes: hash("A:0")=50, hash("A:1")=180, hash("A:2")=300
   Ring: {50→A, 180→A, 300→A}

2. addServer("B"):
   Virtual nodes: hash("B:0")=120, hash("B:1")=240, hash("B:2")=350
   Ring: {50→A, 120→B, 180→A, 240→B, 300→A, 350→B}

3. getServer("user123"):
   hash("user123") = 130
   Ceiling of 130 in ring → 180 → server A

4. getServer("user456"):
   hash("user456") = 310
   Ceiling of 310 in ring → 350 → server B

5. removeServer("A"):
   Remove: 50, 180, 300
   Ring: {120→B, 240→B, 350→B}
   
   Keys that were on A → now go to B (nearest clockwise)
   "user123" (130): ceiling → 240 → server B
```

---

## Real-World Usage

| System | Use of Consistent Hashing |
|--------|--------------------------|
| Amazon DynamoDB | Partition data across nodes |
| Apache Cassandra | Token ring for data distribution |
| Memcached | Client-side key routing |
| Akamai CDN | Content routing to edge servers |
| Discord | Message routing to backend servers |

---

## Complexity

| Operation | Time |
|-----------|:---:|
| Add server | $O(V \log(NV))$ where $V$ = virtual nodes |
| Remove server | $O(V \log(NV))$ |
| Lookup key | $O(\log(NV))$ |
| Keys moved on change | $O(K/N)$ expected |

---

## Quick Revision Question

**Q: With naive hashing (hash % N), if you go from 3 servers to 4, what fraction of keys need to be moved? How does consistent hashing improve this?**

<details>
<summary>Click to reveal answer</summary>

**Naive hashing (hash % N):**
When $N$ changes from 3 to 4, **approximately 75% of keys** must be moved.

A key stays on the same server only if `hash(key) % 3 == hash(key) % 4`, which is true only when `hash(key) % 12 < 3` (i.e., `hash(key)` mod 12 is 0, giving the same result mod 3 and mod 4). On average, only $\frac{1}{N_{\text{new}}} = 25\%$ of keys stay put.

General formula: fraction of keys that move $\approx 1 - \frac{N_{\text{old}}}{N_{\text{new}} \cdot N_{\text{old}}}$. In practice, nearly all keys rehash.

**Consistent hashing:**
Only $\frac{K}{N_{\text{new}}} = \frac{K}{4}$ keys move — those in the arc between the new server and its predecessor. That's **25% of keys**, and they all move to the ONE new server.

- Naive: ~75% of keys move, scattered across all servers
- Consistent: ~25% of keys move, all to the new server

This difference is critical for cache systems — with naive hashing, adding a server causes a near-total cache miss storm ("thundering herd").

</details>

---

| [← Unit 8: Longest Duplicate Substring](../08-String-Hashing-Problems/06-longest-duplicate-substring.md) | [Next: Bloom Filters →](02-bloom-filters.md) |
|:---|---:|
