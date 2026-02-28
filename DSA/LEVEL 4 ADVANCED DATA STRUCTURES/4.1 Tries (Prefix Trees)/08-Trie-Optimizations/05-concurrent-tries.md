# Concurrent Tries

## Overview
Standard tries are not thread-safe. When multiple threads read and write simultaneously, data corruption occurs. This chapter covers techniques for making tries safe for concurrent access: **locking strategies**, **lock-free approaches**, and the **Ctrie** (concurrent trie).

---

## Problems with Concurrent Access

```
  Thread 1: insert("apple")         Thread 2: insert("app")
  
  Both threads walk a → p → p simultaneously.
  
  Thread 1 creates 'l' child of second 'p'.
  Thread 2 sets is_end=True on second 'p'.
  
  If interleaved incorrectly:
  - Thread 1 might overwrite Thread 2's changes
  - Children dict/array might be in inconsistent state
  - Reader might see partially constructed nodes
```

---

## Strategy 1: Global Lock (Coarse-Grained)

```python
import threading

class ThreadSafeTrie:
    def __init__(self):
        self.root = TrieNode()
        self.lock = threading.RLock()  # Reentrant lock
    
    def insert(self, word):
        with self.lock:
            node = self.root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
    
    def search(self, word):
        with self.lock:
            node = self.root
            for ch in word:
                if ch not in node.children:
                    return False
                node = node.children[ch]
            return node.is_end
```

```
  Pros: Simple, correct
  Cons: No parallelism — all threads serialize
  
  Throughput: Single-threaded equivalent
  Use when: Low contention, simplicity matters
```

---

## Strategy 2: Read-Write Lock

```python
from threading import Lock

class RWLock:
    def __init__(self):
        self.read_lock = Lock()
        self.write_lock = Lock()
        self.readers = 0
    
    def acquire_read(self):
        with self.read_lock:
            self.readers += 1
            if self.readers == 1:
                self.write_lock.acquire()
    
    def release_read(self):
        with self.read_lock:
            self.readers -= 1
            if self.readers == 0:
                self.write_lock.release()
    
    def acquire_write(self):
        self.write_lock.acquire()
    
    def release_write(self):
        self.write_lock.release()

class RWTrie:
    def __init__(self):
        self.root = TrieNode()
        self.rw_lock = RWLock()
    
    def search(self, word):
        self.rw_lock.acquire_read()
        try:
            node = self.root
            for ch in word:
                if ch not in node.children:
                    return False
                node = node.children[ch]
            return node.is_end
        finally:
            self.rw_lock.release_read()
    
    def insert(self, word):
        self.rw_lock.acquire_write()
        try:
            node = self.root
            for ch in word:
                if ch not in node.children:
                    node.children[ch] = TrieNode()
                node = node.children[ch]
            node.is_end = True
        finally:
            self.rw_lock.release_write()
```

```
  Pros: Multiple readers can proceed in parallel
  Cons: Writers still exclusive; reader starvation possible
  
  Best when: Read-heavy workloads (95%+ reads)
```

---

## Strategy 3: Fine-Grained Locking (Per-Node)

```java
class ConcurrentTrieNode {
    ConcurrentHashMap<Character, ConcurrentTrieNode> children;
    volatile boolean isEnd;
    ReadWriteLock lock;
    
    ConcurrentTrieNode() {
        children = new ConcurrentHashMap<>();
        isEnd = false;
        lock = new ReentrantReadWriteLock();
    }
}

class FineGrainedTrie {
    ConcurrentTrieNode root = new ConcurrentTrieNode();
    
    void insert(String word) {
        ConcurrentTrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char ch = word.charAt(i);
            // ConcurrentHashMap handles concurrent puts safely
            node.children.putIfAbsent(ch, new ConcurrentTrieNode());
            node = node.children.get(ch);
        }
        node.isEnd = true;  // volatile write
    }
    
    boolean search(String word) {
        ConcurrentTrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char ch = word.charAt(i);
            node = node.children.get(ch);
            if (node == null) return false;
        }
        return node.isEnd;  // volatile read
    }
}
```

```
  Using ConcurrentHashMap for children:
  - putIfAbsent is atomic
  - get is lock-free
  - Threads working on different branches don't block each other
  
  Pros: High parallelism for different key paths
  Cons: Lock overhead per node; complex delete
```

---

## Strategy 4: Lock-Free Trie (CAS-Based)

```java
import java.util.concurrent.atomic.AtomicReferenceArray;

class LockFreeTrieNode {
    AtomicReferenceArray<LockFreeTrieNode> children;
    volatile boolean isEnd;
    
    LockFreeTrieNode() {
        children = new AtomicReferenceArray<>(26);
        isEnd = false;
    }
}

class LockFreeTrie {
    LockFreeTrieNode root = new LockFreeTrieNode();
    
    void insert(String word) {
        LockFreeTrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int c = word.charAt(i) - 'a';
            
            if (node.children.get(c) == null) {
                LockFreeTrieNode newNode = new LockFreeTrieNode();
                // CAS: only one thread succeeds in setting the child
                node.children.compareAndSet(c, null, newNode);
                // If CAS fails, another thread already created it — fine
            }
            node = node.children.get(c);
        }
        node.isEnd = true;
    }
    
    boolean search(String word) {
        LockFreeTrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int c = word.charAt(i) - 'a';
            node = node.children.get(c);
            if (node == null) return false;
        }
        return node.isEnd;
    }
}
```

```
  CAS (Compare-And-Swap):
  - Atomically: if children[c] == null, set to newNode
  - If another thread set it first, CAS fails → use their node
  - No locks needed!
  
  Pros: Maximum throughput, no blocking
  Cons: Delete is very hard; ABA problem; memory ordering
```

---

## Strategy 5: Ctrie (Concurrent Hash-Trie)

```
  Ctrie = Lock-free concurrent trie using:
  - Indirection Nodes (I-Nodes): mutable pointers
  - Main Nodes (C-Nodes, T-Nodes, L-Nodes): immutable
  - CAS on I-Node pointers for atomic updates
  
  Structure:
  
  root (I-Node)
    └→ C-Node [bitmap, array of branches]
         ├── (key='a', I-Node) → C-Node [...]
         └── (key='b', I-Node) → T-Node (leaf: "bat")
  
  Insert: Create new C-Node with updated branches,
          CAS the parent I-Node to point to new C-Node.
  
  Since C-Nodes are immutable:
  - Readers always see consistent snapshots
  - Writers create new versions atomically
  
  This is a production-grade approach used in Scala's standard library.
```

---

## Comparison of Strategies

| Strategy | Read Throughput | Write Throughput | Complexity | Delete Support |
|----------|:---:|:---:|:---:|:---:|
| Global Lock | Low | Low | Simple | Easy |
| RW Lock | High | Low | Medium | Easy |
| Fine-Grained | High | High | Complex | Medium |
| Lock-Free (CAS) | Highest | High | Hard | Very Hard |
| Ctrie | High | High | Very Hard | Supported |

---

## When to Use What

```
  Single-threaded: No synchronization needed
  
  Few writers, many readers: Read-Write Lock
  
  Many writers, many readers: 
    Java → ConcurrentHashMap children
    Scala → Ctrie (built-in)
    C++ → Lock-free with atomics
  
  Maximum performance:
    Lock-free CAS-based (insert-only trie)
  
  Need snapshots/iteration:
    Ctrie or Copy-on-Write
```

---

## Java ConcurrentHashMap-Based (Recommended for Production)

```java
// Simplest practical concurrent trie in Java

class ConcurrentTrie {
    private final ConcurrentHashMap<String, Boolean> prefixes = 
        new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, Boolean> words = 
        new ConcurrentHashMap<>();
    
    void insert(String word) {
        words.put(word, true);
        for (int i = 1; i <= word.length(); i++) {
            prefixes.putIfAbsent(word.substring(0, i), true);
        }
    }
    
    boolean search(String word) {
        return words.containsKey(word);
    }
    
    boolean startsWith(String prefix) {
        return prefixes.containsKey(prefix);
    }
}
```

```
  Trades memory for simplicity.
  All operations are thread-safe via ConcurrentHashMap.
  No trie structure needed — just prefix/word sets.
  
  Good enough for most production use cases.
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Goal | Safe concurrent read/write on trie |
| Simplest | Global lock (serialize all) |
| Read-optimized | RW lock (parallel reads) |
| Write-optimized | Fine-grained / CAS-based |
| Production-grade | Ctrie (immutable nodes + CAS) |
| Java practical | ConcurrentHashMap children |

---

## Quick Revision Questions

1. **Why is a standard trie not thread-safe?**
   > Concurrent modifications to shared mutable state (children dicts/arrays, is_end flags) without synchronization can cause data races: partial updates, lost writes, or reading inconsistent state.

2. **How does CAS (Compare-And-Swap) enable lock-free insertion?**
   > CAS atomically checks if a child pointer is null and sets it to a new node. If another thread already set it, CAS fails and the thread uses the existing node. No locks needed because the operation is atomic.

3. **Why is deletion hard in lock-free tries?**
   > Deletion requires removing a node while other threads may be reading it. Without locks, you need techniques like hazard pointers or epoch-based reclamation to safely free memory, which adds significant complexity.

4. **What makes the Ctrie approach special?**
   > Ctrie uses immutable internal nodes. Updates create new nodes and atomically swap pointers via CAS. Readers always see consistent snapshots without locks, and the structure supports linearizable operations.

5. **When is a global lock sufficient?**
   > When contention is low (few threads), operations are fast (short critical sections), and simplicity is more important than maximum throughput. For many interview problems and single-threaded applications, it's fine.

---

## Navigation

| | |
|:---|---:|
| [◀ Previous: Memory Optimization](04-memory-optimization.md) | [Next: Persistent Tries ▶](06-persistent-tries.md) |
