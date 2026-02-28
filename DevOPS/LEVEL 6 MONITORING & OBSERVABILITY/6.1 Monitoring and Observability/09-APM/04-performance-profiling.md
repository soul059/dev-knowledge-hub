# Chapter 4: Performance Profiling

[← Previous: Synthetic Monitoring](03-synthetic-monitoring.md) | [Back to README](../README.md) | [Next: Real User Monitoring →](05-real-user-monitoring.md)

---

## Overview

Performance profiling identifies exactly where your application spends time and memory. While traces show you which service is slow, profiles show you which function, line of code, or allocation is the bottleneck. This chapter covers continuous profiling tools and flame graph analysis.

---

## 4.1 What is Profiling?

```
┌──────────────────────────────────────────────────────────┐
│        OBSERVABILITY DEPTH                               │
│                                                          │
│  Metrics → "Something is slow"                          │
│     │                                                    │
│     ▼                                                    │
│  Traces → "This service/request is slow"                │
│     │                                                    │
│     ▼                                                    │
│  Profiles → "This FUNCTION is slow"                     │
│     │         "This LINE allocates too much memory"     │
│     │                                                    │
│     ▼                                                    │
│  Code Fix → "Changed algorithm from O(n²) to O(n log n)"│
│                                                          │
│  Profiling = The deepest level of performance analysis  │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Profile Types

| Profile Type | What It Measures | When to Use |
|-------------|------------------|-------------|
| CPU Profile | Time spent in each function | App uses too much CPU |
| Memory (Heap) | Active allocations by function | Memory growing / OOM kills |
| Allocation | All allocations (including freed) | GC pressure, high alloc rate |
| Goroutine/Thread | Concurrency, blocked goroutines | Deadlocks, thread exhaustion |
| Mutex/Lock | Time waiting on locks | Lock contention |
| Wall Clock | Real elapsed time (incl. waiting) | I/O bound applications |
| Off-CPU | Time blocked (I/O, sleep, locks) | Latency not from CPU |

---

## 4.3 Flame Graphs

```
┌──────────────────────────────────────────────────────────┐
│  FLAME GRAPH - HOW TO READ                               │
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │ main()                                     100%   │  │
│  ├─────────────────────────────┬─────────────────────┤  │
│  │ handleRequest()      60%   │ processQueue() 40%  │  │
│  ├──────────────┬──────────────┤─────────┬───────────┤  │
│  │ parseJSON()  │ queryDB()   │ decode()│ send()    │  │
│  │ 15%          │ 45%         │ 10%     │ 30%       │  │
│  ├──────────────┤──────┬──────┤─────────┤───────────┤  │
│  │              │exec()│wait()│         │ marshal() │  │
│  │              │ 15%  │ 30%  │         │ 25%       │  │
│  └──────────────┴──────┴──────┴─────────┴───────────┘  │
│                                                          │
│  Reading the graph:                                      │
│  • X-axis: NOT time! Width = proportion of samples      │
│  • Y-axis: Stack depth (root at bottom)                 │
│  • Wide bar = function consuming most resources         │
│  • Color: Usually random (or by package/module)         │
│                                                          │
│  Insight: queryDB().wait() takes 30% → optimize query   │
│           send().marshal() takes 25% → optimize serial. │
└──────────────────────────────────────────────────────────┘
```

---

## 4.4 Continuous Profiling with Grafana Pyroscope

```
┌──────────────────────────────────────────────────────────┐
│  CONTINUOUS PROFILING ARCHITECTURE                       │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Service A│  │ Service B│  │ Service C│              │
│  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │              │
│  │ │Pyro  │ │  │ │Pyro  │ │  │ │pprof │ │              │
│  │ │SDK   │ │  │ │SDK   │ │  │ │endpt │ │              │
│  │ └──┬───┘ │  │ └──┬───┘ │  │ └──┬───┘ │              │
│  └────┼─────┘  └────┼─────┘  └────┼─────┘              │
│       └──────────────┼─────────────┘                     │
│                      ▼                                   │
│       ┌──────────────────────────┐                      │
│       │    GRAFANA PYROSCOPE     │                      │
│       │                          │                      │
│       │  Stores profiles over    │                      │
│       │  time with labels        │                      │
│       │  (service, env, version) │                      │
│       │                          │                      │
│       │  Query: Compare profiles │                      │
│       │  between versions,       │                      │
│       │  time ranges, endpoints  │                      │
│       └──────────────────────────┘                      │
│                      │                                   │
│                      ▼                                   │
│       ┌──────────────────────────┐                      │
│       │       GRAFANA UI         │                      │
│       │  Flame graphs, diff view │                      │
│       │  Trace → Profile linking │                      │
│       └──────────────────────────┘                      │
└──────────────────────────────────────────────────────────┘
```

### Setup (Python)

```python
# pip install pyroscope-io
import pyroscope

pyroscope.configure(
    application_name="order-service",
    server_address="http://pyroscope:4040",
    tags={
        "env": "production",
        "version": "v2.3.1",
        "region": "us-east-1"
    }
)

# Application runs normally — profiling happens in background
# with minimal overhead (~2-5% CPU)
```

### Setup (Go)

```go
package main

import (
    "github.com/grafana/pyroscope-go"
)

func main() {
    pyroscope.Start(pyroscope.Config{
        ApplicationName: "order-service",
        ServerAddress:   "http://pyroscope:4040",
        Tags:            map[string]string{
            "env": "production",
            "version": "v2.3.1",
        },
        ProfileTypes: []pyroscope.ProfileType{
            pyroscope.ProfileCPU,
            pyroscope.ProfileAllocObjects,
            pyroscope.ProfileAllocSpace,
            pyroscope.ProfileInuseObjects,
            pyroscope.ProfileInuseSpace,
            pyroscope.ProfileGoroutines,
            pyroscope.ProfileMutexCount,
            pyroscope.ProfileMutexDuration,
            pyroscope.ProfileBlockCount,
            pyroscope.ProfileBlockDuration,
        },
    })
    
    // Application code...
}
```

---

## 4.5 Ad-Hoc Profiling

### Go pprof

```bash
# Expose pprof endpoint (already included in net/http/pprof)
# import _ "net/http/pprof"

# Capture 30-second CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Capture heap profile
go tool pprof http://localhost:6060/debug/pprof/heap

# Interactive commands in pprof
(pprof) top 10          # Top 10 functions by CPU
(pprof) web             # Open flame graph in browser
(pprof) list funcName   # Source-level view of function
```

### Python cProfile

```python
import cProfile
import pstats

# Profile a function
profiler = cProfile.Profile()
profiler.enable()

# ... your code here ...
process_orders()

profiler.disable()

# Print top 20 slowest functions
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(20)
```

### Java Async Profiler

```bash
# Attach to running JVM
./asprof -d 30 -f profile.html <PID>

# CPU profiling for 30 seconds, output as flame graph
./asprof -e cpu -d 30 -f cpu_flame.html <PID>

# Allocation profiling
./asprof -e alloc -d 30 -f alloc_flame.html <PID>

# Lock profiling
./asprof -e lock -d 30 -f lock_flame.html <PID>
```

---

## 4.6 Diff Flame Graphs

```
┌──────────────────────────────────────────────────────────┐
│  DIFF FLAME GRAPH (Before vs After optimization)         │
│                                                          │
│  Red = MORE time in new version                         │
│  Blue = LESS time in new version (improvement)          │
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │ main()                                            │  │
│  ├─────────────────────────┬─────────────────────────┤  │
│  │ handleRequest()   [=]   │ processQueue()  [BLUE]  │  │
│  ├──────────┬──────────────┤──────────┬──────────────┤  │
│  │ parse()  │ queryDB()    │ decode() │ send()       │  │
│  │ [=]      │ [RED ▲20%]   │ [BLUE]   │ [BLUE ▼30%] │  │
│  └──────────┴──────────────┴──────────┴──────────────┘  │
│                                                          │
│  Insight:                                                │
│  ✓ send() improved by 30% (blue) — serialization fix   │
│  ✗ queryDB() got 20% worse (red) — investigate!        │
│                                                          │
│  Use case: Compare v2.3.0 vs v2.3.1 profiles           │
│  to validate optimizations and catch regressions        │
└──────────────────────────────────────────────────────────┘
```

---

## 4.7 Profiling Best Practices

| Practice | Details |
|----------|---------|
| Use continuous profiling | Catch regressions early, low overhead |
| Profile in production | Dev/staging profiles may not represent real workload |
| Compare across versions | Use diff flame graphs for before/after |
| Link traces to profiles | Span-level profiling for root cause |
| Start with CPU profile | Most common bottleneck type |
| Profile under load | Contention issues only appear under load |
| Low sampling rate | 100 Hz is standard, minimal overhead |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Profiler overhead too high | Sampling too frequent | Reduce to 100 Hz, use async profiler |
| Flat flame graph | Top-level function dominates | Filter or zoom into specific subtree |
| Missing function names | Binary stripped | Build with debug symbols |
| Profile doesn't match user experience | Profiling wrong time window | Use continuous profiling, align with incident time |
| Memory profile incorrect | Only showing live objects | Use allocation profiler for allocation rate |

---

## Quick Revision Questions

1. **What is the difference between CPU profiling and wall-clock profiling?**
2. **How do you read a flame graph? What does the width of a bar represent?**
3. **What is continuous profiling and why is it preferred over ad-hoc profiling?**
4. **How do diff flame graphs help identify performance regressions?**
5. **How does profiling complement tracing in debugging performance issues?**

---

[← Previous: Synthetic Monitoring](03-synthetic-monitoring.md) | [Back to README](../README.md) | [Next: Real User Monitoring →](05-real-user-monitoring.md)
