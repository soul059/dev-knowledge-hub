# Chapter 4.1: Real-Time Concepts

## Introduction

Real-time systems are computing systems that must respond to events within strict timing constraints. Unlike general-purpose systems where performance is measured by throughput, real-time systems are evaluated by their **predictability** and ability to meet **deadlines**.

---

## What is Real-Time?

Real-time does NOT mean "fast" — it means **predictable and deterministic**.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REAL-TIME DEFINITION                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A real-time system is one in which the correctness of the          │
│  computation depends on:                                             │
│                                                                      │
│    1. LOGICAL CORRECTNESS - The right answer                        │
│    2. TEMPORAL CORRECTNESS - At the right time                      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │    Result = f(Input) + f(Time)                            │     │
│  │                                                            │     │
│  │    Both correctness and timeliness matter!                │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Hard vs Soft Real-Time

```
┌─────────────────────────────────────────────────────────────────────┐
│                REAL-TIME SYSTEM CLASSIFICATION                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    HARD REAL-TIME                             │   │
│  │                                                               │   │
│  │   Value                                                       │   │
│  │     │                                                         │   │
│  │  100├─────────────────┐                                       │   │
│  │     │                 │                                       │   │
│  │     │                 │                                       │   │
│  │   0 ├─────────────────┴─────────────────────▶ Time           │   │
│  │     │                 │                                       │   │
│  │ -∞  │                 └─ Deadline (CATASTROPHIC if missed)   │   │
│  │                                                               │   │
│  │   Examples: Airbag deployment, pacemaker, flight control     │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    SOFT REAL-TIME                             │   │
│  │                                                               │   │
│  │   Value                                                       │   │
│  │     │                                                         │   │
│  │  100├──────────────────┐                                      │   │
│  │     │                   ╲                                     │   │
│  │   50│                    ╲                                    │   │
│  │     │                     ╲                                   │   │
│  │   0 ├─────────────────────────────────────▶ Time             │   │
│  │     │                  │                                      │   │
│  │                        └─ Deadline (degraded value if late)  │   │
│  │                                                               │   │
│  │   Examples: Video streaming, audio playback, gaming          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    FIRM REAL-TIME                             │   │
│  │                                                               │   │
│  │   Value                                                       │   │
│  │     │                                                         │   │
│  │  100├─────────────────┐                                       │   │
│  │     │                 │                                       │   │
│  │     │                 │                                       │   │
│  │   0 ├─────────────────┴─────────────────────▶ Time           │   │
│  │                       │                                       │   │
│  │                       └─ Deadline (worthless if late,        │   │
│  │                                   but not dangerous)          │   │
│  │                                                               │   │
│  │   Examples: Financial trading, sensor sampling               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Timing Parameters

### Key Timing Metrics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIMING PARAMETERS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Event                      Task Completes                          │
│     │                             │                                  │
│     ▼                             ▼                                  │
│  ───┬─────────────────────────────┬───────────────────────▶ Time    │
│     │                             │               │                  │
│     │◄───── Response Time ───────►│               │                  │
│     │                             │               │                  │
│     │                             │◄── Slack ────►│                  │
│     │                                             │                  │
│     │◄──────────── Deadline ─────────────────────►│                  │
│                                                                      │
│                                                                      │
│  DETAILED BREAKDOWN:                                                 │
│                                                                      │
│   Event  Scheduler   Task      Task                                  │
│  Occurs  Notified   Starts   Completes    Deadline                   │
│     │       │          │         │           │                       │
│     ▼       ▼          ▼         ▼           ▼                       │
│  ───┬───────┬──────────┬─────────┬───────────┬──────▶ Time          │
│     │       │          │         │           │                       │
│     │◄─────►│          │         │           │                       │
│     Interrupt          │         │           │                       │
│     Latency            │         │           │                       │
│             │◄────────►│         │           │                       │
│             Scheduling │         │           │                       │
│             Latency    │         │           │                       │
│                        │◄───────►│           │                       │
│                        Execution │           │                       │
│                        Time      │           │                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Mathematical Definitions

**Response Time (R)**:
$$R = L_{interrupt} + L_{scheduling} + C_{execution}$$

Where:
- $L_{interrupt}$ = Interrupt latency (time to recognize interrupt)
- $L_{scheduling}$ = Scheduling latency (context switch time)
- $C_{execution}$ = Worst-case execution time (WCET)

**Deadline Satisfaction**:
A task meets its deadline if:
$$R ≤ D$$

Where $D$ is the deadline.

---

## Task Model

### Periodic Tasks

Periodic tasks execute at regular intervals.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PERIODIC TASK MODEL                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Task Parameters:                                                    │
│    • Period (T): Time between successive releases                   │
│    • Deadline (D): Relative deadline from release                   │
│    • Execution Time (C): Worst-case execution time                  │
│                                                                      │
│   Release       Complete    Release       Complete                   │
│      │  ┌──────┐   │           │  ┌──────┐   │                      │
│      ▼  │      │   ▼           ▼  │      │   ▼                      │
│  ────┬──┴──────┴───┬───────────┬──┴──────┴───┬───────────▶ Time     │
│      │             │           │             │                       │
│      │◄───── D ───►│           │◄───── D ───►│                       │
│      │                         │                                     │
│      │◄────────── T ──────────►│                                     │
│                                                                      │
│  Utilization: U = C / T                                             │
│                                                                      │
│  Example:                                                            │
│    Period T = 100ms                                                  │
│    WCET C = 20ms                                                     │
│    Deadline D = 50ms                                                 │
│    Utilization U = 20/100 = 0.20 (20%)                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Aperiodic and Sporadic Tasks

```
┌─────────────────────────────────────────────────────────────────────┐
│                    APERIODIC vs SPORADIC                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  APERIODIC TASKS:                                                    │
│    • Arrive at random times                                         │
│    • No minimum inter-arrival time                                  │
│    • Usually soft real-time                                         │
│                                                                      │
│    ┌─┐          ┌─┐    ┌─┐              ┌─┐                          │
│    │ │          │ │    │ │              │ │                          │
│  ──┴─┴──────────┴─┴────┴─┴──────────────┴─┴───────▶ Time            │
│    Random arrival times                                              │
│                                                                      │
│                                                                      │
│  SPORADIC TASKS:                                                     │
│    • Minimum inter-arrival time (MIT)                               │
│    • Can be analyzed for schedulability                             │
│    • Usually hard real-time                                         │
│                                                                      │
│    ┌─┐              ┌─┐              ┌─┐                             │
│    │ │              │ │              │ │                             │
│  ──┴─┴──────────────┴─┴──────────────┴─┴───────▶ Time               │
│        │◄── MIT ───►│◄─── MIT ──────►│                               │
│                                                                      │
│    MIT = Minimum time between arrivals                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Jitter

Jitter is the variation in task timing.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JITTER ANALYSIS                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IDEAL (No Jitter):                                                  │
│                                                                      │
│    ┌─┐         ┌─┐         ┌─┐         ┌─┐         ┌─┐              │
│    │ │         │ │         │ │         │ │         │ │              │
│  ──┴─┴─────────┴─┴─────────┴─┴─────────┴─┴─────────┴─┴──▶          │
│    |     T     |     T     |     T     |     T     |                │
│                                                                      │
│  REAL (With Jitter):                                                 │
│                                                                      │
│    ┌─┐        ┌─┐          ┌─┐        ┌─┐           ┌─┐             │
│    │ │        │ │          │ │        │ │           │ │             │
│  ──┴─┴────────┴─┴──────────┴─┴────────┴─┴───────────┴─┴──▶         │
│    |    T-j   |    T+j     |    T-j   |     T+2j    |               │
│                                                                      │
│                                                                      │
│  TYPES OF JITTER:                                                    │
│                                                                      │
│  ┌─────────────────┬────────────────────────────────────────────┐   │
│  │ Type            │ Description                                │   │
│  ├─────────────────┼────────────────────────────────────────────┤   │
│  │ Release Jitter  │ Variation in task activation time          │   │
│  │ Execution Jitter│ Variation in execution time (BCET to WCET) │   │
│  │ Output Jitter   │ Variation in when results are produced     │   │
│  └─────────────────┴────────────────────────────────────────────┘   │
│                                                                      │
│  Jitter = T_max - T_min                                             │
│                                                                      │
│  Acceptable jitter depends on application:                          │
│    • Audio: < 1ms                                                    │
│    • Video: < 16ms (60 fps)                                         │
│    • Motor control: < 100μs                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Latency Analysis

### Interrupt Latency

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERRUPT LATENCY BREAKDOWN                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  IRQ                     ISR                                         │
│  Signal                  Starts                                      │
│    │                        │                                        │
│    ▼                        ▼                                        │
│  ──┬───────────────────────┬──────────────────────────▶ Time        │
│    │                        │                                        │
│    │◄── Interrupt Latency ─►│                                        │
│                                                                      │
│                                                                      │
│  COMPONENTS:                                                         │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │                                                          │       │
│  │  L_int = T_recognition + T_priority + T_pipeline +       │       │
│  │          T_fetch + T_stacking                            │       │
│  │                                                          │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  ┌────────────────────┬───────────┬────────────────────────────┐    │
│  │ Component          │ ARM Cortex│ Description                │    │
│  ├────────────────────┼───────────┼────────────────────────────┤    │
│  │ Recognition        │ 1 cycle   │ Detect interrupt signal    │    │
│  │ Priority check     │ 1 cycle   │ Compare with current mask  │    │
│  │ Pipeline flush     │ 1-3 cycles│ Complete current instr.    │    │
│  │ Vector fetch       │ 3 cycles  │ Read ISR address           │    │
│  │ Context stacking   │ 12 cycles │ Push registers to stack    │    │
│  ├────────────────────┼───────────┼────────────────────────────┤    │
│  │ TOTAL (Cortex-M4)  │ ~12 cycles│ @100MHz = 120ns            │    │
│  └────────────────────┴───────────┴────────────────────────────┘    │
│                                                                      │
│  Note: Latency increases if:                                        │
│    • Interrupts are disabled (critical section)                     │
│    • Higher-priority ISR is running                                 │
│    • Memory wait states (Flash latency)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Worst-Case Execution Time (WCET)

WCET analysis determines the maximum time a task can take.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WCET ANALYSIS                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  EXECUTION TIME DISTRIBUTION:                                        │
│                                                                      │
│  Probability                                                         │
│       │                                                              │
│       │     ┌───────────────┐                                        │
│       │     │               │                                        │
│       │    ╱│               │                                        │
│       │   ╱ │               │                                        │
│       │  ╱  │               │                                        │
│       │ ╱   │               │   ╲                                    │
│       │╱    │               │    ╲                                   │
│       └─────┴───────────────┴─────┴──────────────▶ Execution Time   │
│             │               │     │                                  │
│           BCET            Avg   WCET                                │
│          (Best)                (Worst)                               │
│                                                                      │
│                                                                      │
│  WCET ANALYSIS METHODS:                                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Method          │ Approach      │ Pros/Cons                 │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Measurement     │ Run many      │ ✓ Simple                  │    │
│  │                 │ times, record │ ✗ May miss worst case     │    │
│  │                 │ maximum       │                           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Static Analysis │ Analyze code  │ ✓ Safe upper bound        │    │
│  │                 │ paths and     │ ✗ Pessimistic             │    │
│  │                 │ architecture  │ ✗ Complex for modern HW   │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Hybrid          │ Combine       │ ✓ Practical compromise    │    │
│  │                 │ both methods  │                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Factors Affecting WCET

| Factor | Impact | Mitigation |
|--------|--------|------------|
| **Cache misses** | 10-100x slower memory access | Lock critical code in cache |
| **Branch prediction** | Pipeline stalls on mispredict | Use simple control flow |
| **Interrupts** | Preemption adds overhead | Disable in critical sections |
| **Memory contention** | DMA/peripherals compete | Schedule DMA carefully |
| **Dynamic allocation** | Non-deterministic malloc | Use static allocation |

---

## Schedulability Analysis

### Utilization Test

For $n$ periodic tasks, the CPU utilization is:

$$U = \sum_{i=1}^{n} \frac{C_i}{T_i}$$

Where:
- $C_i$ = Worst-case execution time of task $i$
- $T_i$ = Period of task $i$

**Necessary Condition**: $U ≤ 1$ (utilization cannot exceed 100%)

**Rate Monotonic Bound** (sufficient condition for RMS):

$$U ≤ n(2^{1/n} - 1)$$

| Tasks (n) | Utilization Bound |
|-----------|-------------------|
| 1         | 100%              |
| 2         | 82.8%             |
| 3         | 78.0%             |
| 4         | 75.7%             |
| ∞         | 69.3% (ln 2)      |

---

## Real-Time System Examples

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REAL-TIME EXAMPLES BY DOMAIN                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  AUTOMOTIVE                                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ System          │ Type   │ Deadline  │ Consequence          │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Airbag deploy   │ Hard   │ <10ms     │ Death/injury         │    │
│  │ ABS braking     │ Hard   │ <5ms      │ Loss of control      │    │
│  │ Engine control  │ Hard   │ <1ms      │ Engine damage        │    │
│  │ Infotainment    │ Soft   │ <100ms    │ User annoyance       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  MEDICAL                                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Pacemaker       │ Hard   │ <10ms     │ Cardiac arrest       │    │
│  │ Insulin pump    │ Hard   │ Minutes   │ Hypo/hyperglycemia   │    │
│  │ Patient monitor │ Firm   │ <1s       │ Missed alert         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  INDUSTRIAL                                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Robot arm       │ Hard   │ <1ms      │ Damage/injury        │    │
│  │ CNC machine     │ Hard   │ <100μs    │ Part scrap           │    │
│  │ Process control │ Firm   │ <100ms    │ Quality issue        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Definition | Key Formula |
|---------|------------|-------------|
| **Hard Real-Time** | Missing deadline = system failure | Must satisfy: R ≤ D |
| **Soft Real-Time** | Late results have degraded value | Best-effort scheduling |
| **Firm Real-Time** | Late results are worthless but not dangerous | Drop late tasks |
| **Period (T)** | Time between task releases | Fixed interval |
| **Deadline (D)** | Time by which task must complete | Often D = T |
| **WCET (C)** | Maximum execution time | Static/dynamic analysis |
| **Utilization (U)** | Fraction of CPU used | U = Σ(C/T) |
| **Jitter** | Timing variation | J = T_max - T_min |
| **Latency** | Delay from event to response | L = L_int + L_sched + C |

---

## Quick Revision Questions

1. **Define hard and soft real-time systems. Give two examples of each.**

2. **A periodic task has T=50ms, D=40ms, C=10ms. Calculate its utilization. Will it meet its deadline if it's the only task?**

3. **What are the three components of interrupt latency? How can each be minimized?**

4. **Explain the difference between WCET and average-case execution time. Why is WCET important for schedulability analysis?**

5. **Three tasks have: Task A (T=100ms, C=20ms), Task B (T=50ms, C=10ms), Task C (T=200ms, C=40ms). Calculate total utilization. Is this schedulable under Rate Monotonic scheduling?**

6. **What is jitter? Why is it critical in audio/video applications but less important in a temperature monitoring system?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 4: RTOS](README.md) | [Unit Index](README.md) | [Chapter 4.2: RTOS Architecture](02-rtos-architecture.md) |

---
