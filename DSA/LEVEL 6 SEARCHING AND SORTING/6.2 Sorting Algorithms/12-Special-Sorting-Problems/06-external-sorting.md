# Unit 12: Special Sorting Problems — External Sorting

[← Previous: K-Way Merge](05-k-way-merge.md) | [Back to README →](../README.md)

---

## Overview

**External Sorting** handles datasets too large to fit in RAM. When you have 100 GB of data but only 1 GB of RAM, you need to sort using disk I/O efficiently. The classic solution is **External Merge Sort**.

---

## The Problem

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Data: 100 GB on disk                                      │
  │  RAM:  1 GB available                                       │
  │                                                              │
  │  Can't load everything into memory!                        │
  │  Disk reads/writes are ~100,000x slower than RAM access    │
  │                                                              │
  │  Goal: Sort the data while minimizing disk I/O             │
  └──────────────────────────────────────────────────────────────┘
  
  Key insight: Disk access is sequential-friendly.
  Reading 1 MB sequentially ≈ reading 1 byte randomly.
  → Design for SEQUENTIAL access, minimize RANDOM seeks.
```

---

## External Merge Sort: Two Phases

```
  ┌─────────────────────────────────────────────────────────────┐
  │  PHASE 1: CREATE SORTED RUNS                               │
  │                                                             │
  │  Read M elements at a time (M fits in RAM)                 │
  │  Sort them in memory                                        │
  │  Write sorted "run" back to disk                           │
  │                                                             │
  │  Result: N/M sorted runs on disk                           │
  ├─────────────────────────────────────────────────────────────┤
  │  PHASE 2: MERGE RUNS                                       │
  │                                                             │
  │  Use K-way merge to combine sorted runs                    │
  │  K = number of runs that fit in memory simultaneously     │
  │  (K = M / buffer_size - 1, reserve 1 buffer for output)   │
  │                                                             │
  │  If more runs than K: multi-pass merge                     │
  └─────────────────────────────────────────────────────────────┘
```

---

## Visual Walkthrough

```
  DATA: 100 GB on disk,  RAM: 1 GB
  
  PHASE 1 — Create Sorted Runs:
  ┌─────────────────────────────────────────────────────┐
  │  Read 1 GB chunk → Sort in RAM → Write to disk    │
  │  Repeat 100 times                                   │
  │                                                     │
  │  Result: 100 sorted runs, each 1 GB                │
  │                                                     │
  │  Run 0: [sorted 1 GB]                              │
  │  Run 1: [sorted 1 GB]                              │
  │  ...                                                │
  │  Run 99: [sorted 1 GB]                             │
  └─────────────────────────────────────────────────────┘
  
  PHASE 2 — K-Way Merge:
  ┌─────────────────────────────────────────────────────┐
  │  RAM: split into K input buffers + 1 output buffer │
  │                                                     │
  │  With 1 GB RAM, 10 MB buffers:                     │
  │  K = 99 input buffers + 1 output buffer            │
  │  = 100 × 10 MB = 1 GB ✓                           │
  │                                                     │
  │  Read 10 MB from each of 99 runs into buffers     │
  │  K-way merge using min-heap                        │
  │  When output buffer full → write to disk           │
  │  When input buffer empty → read next 10 MB block   │
  │                                                     │
  │  Since K=99 ≥ 100... wait, need 100 runs!         │
  │  Two passes needed if K < number of runs           │
  └─────────────────────────────────────────────────────┘
```

### Multi-Pass Merge

```
  If 100 runs but K = 10 (can merge 10 at a time):
  
  Pass 1: Merge groups of 10 runs:
  Runs 0-9   → Merged Run A  (10 GB)
  Runs 10-19 → Merged Run B  (10 GB)
  ...
  Runs 90-99 → Merged Run J  (10 GB)
  → 10 merged runs
  
  Pass 2: Merge all 10 merged runs:
  Runs A-J → Final sorted output (100 GB)
  
  Total passes: ⌈log_K(R)⌉ where R = initial number of runs
  With K=10, R=100: ⌈log₁₀(100)⌉ = 2 passes
```

---

## Pseudocode

```
  EXTERNAL_MERGE_SORT(file, memory_size):
    
    // Phase 1: Create sorted runs
    runs = []
    while data remaining in file:
        chunk = read(file, memory_size)
        sort(chunk)                    // any in-memory sort
        run = write_to_temp_file(chunk)
        runs.append(run)
    
    // Phase 2: K-way merge
    K = memory_size / buffer_size - 1
    
    while len(runs) > 1:
        new_runs = []
        for i in range(0, len(runs), K):
            group = runs[i : i+K]
            merged = k_way_merge(group, buffer_size)
            new_runs.append(merged)
        runs = new_runs
    
    return runs[0]  // final sorted file
```

---

## Java Implementation (Simplified)

```java
import java.io.*;
import java.util.*;

public class ExternalSort {
    
    static final int CHUNK_SIZE = 1_000_000; // elements per run
    
    public static void externalSort(String inputFile, String outputFile) 
            throws IOException {
        
        // Phase 1: Create sorted runs
        List<String> runFiles = createSortedRuns(inputFile);
        
        // Phase 2: K-way merge
        mergeRuns(runFiles, outputFile);
        
        // Cleanup temp files
        for (String f : runFiles) new File(f).delete();
    }
    
    static List<String> createSortedRuns(String inputFile) 
            throws IOException {
        List<String> runFiles = new ArrayList<>();
        BufferedReader reader = new BufferedReader(new FileReader(inputFile));
        
        int runId = 0;
        while (true) {
            List<Integer> chunk = new ArrayList<>();
            String line;
            for (int i = 0; i < CHUNK_SIZE; i++) {
                line = reader.readLine();
                if (line == null) break;
                chunk.add(Integer.parseInt(line.trim()));
            }
            if (chunk.isEmpty()) break;
            
            Collections.sort(chunk);  // Sort in memory
            
            String runFile = "run_" + runId + ".tmp";
            BufferedWriter writer = new BufferedWriter(new FileWriter(runFile));
            for (int val : chunk) {
                writer.write(val + "\n");
            }
            writer.close();
            
            runFiles.add(runFile);
            runId++;
        }
        reader.close();
        return runFiles;
    }
    
    static void mergeRuns(List<String> runFiles, String outputFile) 
            throws IOException {
        // Min-heap: (value, readerIndex)
        PriorityQueue<long[]> heap = new PriorityQueue<>(
            (a, b) -> Long.compare(a[0], b[0])
        );
        
        BufferedReader[] readers = new BufferedReader[runFiles.size()];
        for (int i = 0; i < runFiles.size(); i++) {
            readers[i] = new BufferedReader(new FileReader(runFiles.get(i)));
            String line = readers[i].readLine();
            if (line != null) {
                heap.offer(new long[]{Long.parseLong(line.trim()), i});
            }
        }
        
        BufferedWriter writer = new BufferedWriter(new FileWriter(outputFile));
        
        while (!heap.isEmpty()) {
            long[] min = heap.poll();
            writer.write(min[0] + "\n");
            
            int idx = (int) min[1];
            String line = readers[idx].readLine();
            if (line != null) {
                heap.offer(new long[]{Long.parseLong(line.trim()), idx});
            }
        }
        
        writer.close();
        for (BufferedReader r : readers) r.close();
    }
}
```

---

## Replacement Selection (Longer Runs)

```
  Standard approach: runs = RAM size M
  
  Replacement Selection: runs ≈ 2M on average!
  
  Algorithm:
  1. Fill RAM with M elements (use a min-heap)
  2. Output minimum to current run
  3. Read next element from input
     - If new element ≥ last output → add to heap
     - If new element < last output → mark for NEXT run
  4. When all heap elements are marked → start new run
  
  Expected run length: 2M (proven by Knuth)
  → Fewer runs → fewer merge passes → faster!
  
  ┌────────────────────────────────────────────────────────────┐
  │  100 GB data, 1 GB RAM:                                   │
  │  Standard: 100 runs of 1 GB                               │
  │  Replacement Selection: ~50 runs of ~2 GB                 │
  │  → May save one entire merge pass                         │
  └────────────────────────────────────────────────────────────┘
```

---

## Complexity Analysis

```
  N = total data size
  M = memory size
  B = disk block size
  
  Phase 1 (create runs):
    Read entire file: N/B disk reads
    Write sorted runs: N/B disk writes
    In-memory sort: N/M runs × O(M log M) each = O(N log M)
    Disk I/O: 2N/B
  
  Phase 2 (merge):
    Passes: ⌈log_K(N/M)⌉  where K ≈ M/B
    Each pass: read + write entire data = 2N/B
    Total I/O: 2N/B × ⌈log_K(N/M)⌉
  
  Total disk I/O: 2N/B × (1 + ⌈log_{M/B}(N/M)⌉)
  
  Example: N = 100 GB, M = 1 GB, B = 4 KB
    K = M/B = 1 GB / 4 KB = 250,000
    Runs = N/M = 100
    Passes = ⌈log₂₅₀₀₀₀(100)⌉ = 1 pass!
    
    Total I/O: 2 × 100/0.004 × 2 ≈ 100,000 block reads/writes
    With large K, most datasets need just 1-2 merge passes.
```

---

## Optimizations

```
  1. Double Buffering: 
     Read next block while processing current → hide I/O latency
  
  2. Replacement Selection:
     Produces ~2x longer runs → fewer passes
  
  3. Large Block Size:
     Sequential I/O is fast → use large buffers (1-64 MB)
  
  4. SSD Optimization:
     SSDs have lower seek penalty → can use more random I/O
     But sequential is still faster
  
  5. Compression:
     Compress runs on disk → less I/O, more CPU
     Worth it when I/O-bound (usually is)
  
  6. Parallel I/O:
     Read from multiple disks simultaneously
     Distribute runs across drives
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| When needed | Data > RAM |
| Algorithm | External Merge Sort |
| Phase 1 | Create sorted runs in memory |
| Phase 2 | K-way merge of runs |
| Key metric | Disk I/O operations |
| Merge passes | ⌈log_K(runs)⌉ |
| Optimization | Replacement selection (2x longer runs) |
| Used in | Databases, OS virtual memory, big data |

---

## Quick Revision Questions

1. **Why is merge sort (not quick sort) used for external sorting?**
2. **Calculate the number of merge passes for 1 TB data, 4 GB RAM, and 4 MB buffers.**
3. **What is replacement selection and how does it produce longer runs?**
4. **Why is sequential disk access so much faster than random access?**
5. **How does the K-way merge from the previous chapter apply here?**
6. **What is double buffering and why does it help external sorting?**

---

## Course Complete!

Congratulations on completing all 12 units of Sorting Algorithms! 🎉

Key takeaways:
- **Simple sorts** (Bubble, Selection, Insertion) — O(n²) but useful for small n
- **Efficient comparison sorts** (Merge, Quick, Heap) — O(n log n) barrier
- **Non-comparison sorts** (Counting, Radix, Bucket) — O(n) with constraints
- **Choosing wisely** — depends on data, constraints, and requirements
- **Special problems** — DNF, frequency sort, linked lists, K-way merge, external sort

---

[← Previous: K-Way Merge](05-k-way-merge.md) | [Back to README →](../README.md)
