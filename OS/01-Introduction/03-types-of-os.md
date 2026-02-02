# Chapter 1.3: Types of Operating Systems

## Overview

Operating systems can be classified based on various criteria such as processing method, user interface, and application domain. Understanding these types helps in choosing the right OS for specific requirements.

---

## Classification of Operating Systems

```
                        OPERATING SYSTEMS
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   By Processing         By Users              By Purpose
        │                     │                     │
   ┌────┴────┐           ┌────┴────┐          ┌────┴────┐
   │ Batch   │           │ Single  │          │ General │
   │ Multi-  │           │ Multi-  │          │ Special │
   │ program │           │ User    │          │ Purpose │
   │ Time-   │           └─────────┘          └─────────┘
   │ sharing │
   │ Real-   │
   │ time    │
   └─────────┘
```

---

## 1. Batch Operating System

### Concept
Jobs with similar requirements are grouped together (batched) and executed as a group. There is no direct interaction between user and computer during execution.

### Working Principle

```
┌─────────────────────────────────────────────────────┐
│                    BATCH PROCESSING                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│   Job 1 ─┐                                          │
│   Job 2 ─┼─→ [Batch Queue] ─→ [CPU] ─→ Output      │
│   Job 3 ─┘                                          │
│                                                     │
│   User submits jobs → Operator creates batch →     │
│   Computer processes batch → Results returned       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Characteristics
- Jobs are collected and processed in groups
- No user interaction during execution
- Uses FCFS (First Come First Serve) scheduling
- Operator groups similar jobs together

### Advantages
| Advantage | Explanation |
|-----------|-------------|
| Reduced Idle Time | CPU is always busy processing jobs |
| Efficient Resource Use | Similar jobs share resources |
| Less Manual Intervention | Automated job processing |

### Disadvantages
| Disadvantage | Explanation |
|--------------|-------------|
| No Interactivity | Users cannot interact with running jobs |
| Long Turnaround | May wait long for job completion |
| Debugging Difficulty | Hard to fix errors in running jobs |

### Example Use Cases
- Payroll processing
- Bank statement generation
- Billing systems

---

## 2. Multiprogramming Operating System

### Concept
Multiple programs are loaded into main memory simultaneously. When one program waits for I/O, CPU switches to another program, maximizing CPU utilization.

### Why Multiprogramming?

**Problem with Single Programming:**
```
Time →
┌────────────────────────────────────────┐
│  CPU   │████│    │████│    │████│     │
├────────┼────┼────┼────┼────┼────┼─────┤
│  I/O   │    │████│    │████│    │████ │
└────────────────────────────────────────┘
        CPU idle during I/O!
```

**Solution with Multiprogramming:**
```
Time →
┌────────────────────────────────────────┐
│  CPU   │ P1 │ P2 │ P1 │ P2 │ P1 │ P2  │
├────────┼────┼────┼────┼────┼────┼─────┤
│ P1 I/O │    │████│    │████│    │████ │
├────────┼────┼────┼────┼────┼────┼─────┤
│ P2 I/O │████│    │████│    │████│     │
└────────────────────────────────────────┘
        CPU always busy!
```

### Memory Organization

```
┌───────────────────────┐
│    Operating System   │ ← Always resident
├───────────────────────┤
│      Program 1        │
├───────────────────────┤
│      Program 2        │
├───────────────────────┤
│      Program 3        │
├───────────────────────┤
│    Free Memory        │
└───────────────────────┘
```

### Key Points
- Multiple programs in memory simultaneously
- CPU switches when current program waits for I/O
- Improves CPU utilization significantly
- Requires memory protection between programs

### Degree of Multiprogramming
The number of programs in memory at any time. Higher degree = Better CPU utilization (up to a point).

---

## 3. Multitasking / Time-Sharing Operating System

### Concept
An extension of multiprogramming where CPU switches between tasks so rapidly that users can interact with each program while it's running. Each user gets a small time slice (quantum).

### Time Quantum

```
    Time Quantum = 100ms (example)
    
    ┌────┬────┬────┬────┬────┬────┬────┬────┐
    │ P1 │ P2 │ P3 │ P1 │ P2 │ P3 │ P1 │...│
    └────┴────┴────┴────┴────┴────┴────┴────┘
    0   100  200  300  400  500  600  700 ms
    
    Each process gets 100ms before switching
```

### Multiprogramming vs Multitasking

| Aspect | Multiprogramming | Multitasking |
|--------|------------------|--------------|
| **Switching** | On I/O wait | On time quantum expiry |
| **Interaction** | No | Yes |
| **Response Time** | Not guaranteed | Quick response |
| **Goal** | Max CPU utilization | User responsiveness |

### Characteristics
- Quick response time for users
- Each user feels they have dedicated system
- Uses scheduling algorithms (Round Robin)
- Supports multiple users simultaneously

### Example
Modern desktop OS (Windows, Linux) are time-sharing systems where you can:
- Browse web
- Listen to music
- Write documents
- All "simultaneously" (actually time-sliced)

---

## 4. Multiprocessing Operating System

### Concept
Uses two or more CPUs (processors) within a single computer system. Multiple processors share memory and I/O devices.

### Types of Multiprocessing

#### Symmetric Multiprocessing (SMP)
```
┌──────────────────────────────────────────┐
│               MAIN MEMORY                │
├──────────────────────────────────────────┤
              │      │      │
          ┌───┴───┐┌─┴──┐┌──┴──┐
          │ CPU 1 ││CPU 2││CPU 3│
          └───────┘└────┘└─────┘
          
All CPUs are equal, share memory and workload
```

#### Asymmetric Multiprocessing (AMP)
```
┌──────────────────────────────────────────┐
│               MAIN MEMORY                │
├──────────────────────────────────────────┤
              │             │
          ┌───┴───┐     ┌───┴───┐
          │Master │────→│ Slave │
          │  CPU  │     │ CPUs  │
          └───────┘     └───────┘
          
Master CPU controls, assigns tasks to slaves
```

### Advantages
| Advantage | Explanation |
|-----------|-------------|
| Increased Throughput | More CPUs = More work done |
| Reliability | If one CPU fails, others continue |
| Economy of Scale | Share peripherals, power supply |

### Disadvantages
- Complex OS design
- Load balancing challenges
- Synchronization overhead

---

## 5. Distributed Operating System

### Concept
A collection of independent computers that appears to users as a single coherent system. Resources are distributed across multiple networked machines.

### Architecture

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  Node 1 │◄──►│  Node 2 │◄──►│  Node 3 │
│ (CPU+   │    │ (CPU+   │    │ (CPU+   │
│ Memory) │    │ Memory) │    │ Memory) │
└────┬────┘    └────┬────┘    └────┬────┘
     │              │              │
     └──────────────┴──────────────┘
                    │
            ┌───────┴───────┐
            │   NETWORK     │
            │ (LAN/WAN)     │
            └───────────────┘
```

### Characteristics
- **Transparency**: Users see single system
- **Resource Sharing**: Access resources on any node
- **Fault Tolerance**: System survives node failures
- **Scalability**: Easy to add new nodes

### Types of Distributed Systems

| Type | Description | Example |
|------|-------------|---------|
| **Client-Server** | Clients request, servers provide | Web applications |
| **Peer-to-Peer** | All nodes equal | BitTorrent |
| **Cluster** | Tightly coupled nodes | High-performance computing |
| **Cloud** | Resources over internet | AWS, Azure |

### Challenges
- Network communication delays
- Partial failures
- Security across network
- Maintaining consistency

---

## 6. Real-Time Operating System (RTOS)

### Concept
An OS that guarantees response within specified time constraints. Critical for time-sensitive applications where delayed response could be catastrophic.

### Types of Real-Time Systems

#### Hard Real-Time
- **Definition**: Deadlines MUST be met, or system fails
- **Consequence of Missing Deadline**: Catastrophic
- **Examples**: 
  - Pacemaker
  - Aircraft control systems
  - Nuclear reactor control

```
┌─────────────────────────────────────────┐
│          HARD REAL-TIME                  │
│                                         │
│    Stimulus ──→ [Process] ──→ Response  │
│                                         │
│    Time constraint: MUST be met         │
│    Failure = System failure/danger      │
└─────────────────────────────────────────┘
```

#### Soft Real-Time
- **Definition**: Deadlines should be met, but occasional misses acceptable
- **Consequence of Missing Deadline**: Degraded quality
- **Examples**:
  - Video streaming
  - Audio playback
  - Video games

### Characteristics of RTOS
| Feature | Description |
|---------|-------------|
| Deterministic | Predictable response times |
| Low Latency | Minimal delay in response |
| Priority Scheduling | Critical tasks get priority |
| Minimal Interrupt Latency | Quick interrupt handling |

### Examples of RTOS
- VxWorks (spacecraft, medical devices)
- FreeRTOS (microcontrollers)
- QNX (automotive systems)
- RTLinux (industrial applications)

---

## 7. Network Operating System

### Concept
OS that provides services to computers connected over a network. Each computer maintains its own OS but can access shared resources.

### Architecture

```
┌──────────────────────────────────────────┐
│                SERVER                     │
│  ┌─────────────────────────────────────┐ │
│  │      Network Operating System        │ │
│  │   (File Server, Print Server, etc.)  │ │
│  └─────────────────────────────────────┘ │
└───────────────────┬──────────────────────┘
                    │ Network
    ┌───────────────┼───────────────┐
    │               │               │
┌───┴───┐       ┌───┴───┐       ┌───┴───┐
│Client │       │Client │       │Client │
│   1   │       │   2   │       │   3   │
└───────┘       └───────┘       └───────┘
```

### Features
- File sharing across network
- Printer sharing
- User management and authentication
- Network security

### Examples
- Windows Server
- Novell NetWare
- Linux distributions (configured as servers)

---

## 8. Mobile Operating System

### Concept
Operating systems designed specifically for mobile devices (smartphones, tablets) with touch interfaces, power efficiency, and connectivity features.

### Unique Characteristics

| Feature | Description |
|---------|-------------|
| Touch Interface | Designed for finger input |
| Power Management | Battery optimization |
| Always Connected | Cellular, WiFi, Bluetooth |
| App Ecosystem | App stores for software |
| Location Services | GPS integration |
| Sensors | Accelerometer, gyroscope |

### Popular Mobile Operating Systems

```
┌─────────────────────────────────────────┐
│  Android (Google) - Linux-based         │
│  - Open source                          │
│  - Market leader (~70% share)           │
│  - Used by: Samsung, OnePlus, Xiaomi    │
├─────────────────────────────────────────┤
│  iOS (Apple)                            │
│  - Closed source                        │
│  - Premium devices                       │
│  - Used by: iPhone, iPad                │
├─────────────────────────────────────────┤
│  Others: HarmonyOS, KaiOS               │
└─────────────────────────────────────────┘
```

---

## 9. Embedded Operating System

### Concept
OS designed for embedded systems - specialized computer systems that perform dedicated functions within larger systems.

### Characteristics
- Limited resources (memory, processing power)
- Specific, dedicated functionality
- Often real-time requirements
- High reliability requirements

### Examples

| Device | Embedded OS |
|--------|-------------|
| Smart TV | Tizen, WebOS, Android TV |
| Router | OpenWrt, DD-WRT |
| ATM Machine | Windows Embedded |
| Car Entertainment | QNX, Android Automotive |
| Washing Machine | Proprietary RTOS |

---

## Comparison Summary

| Type | Users | Interaction | Response | Use Case |
|------|-------|-------------|----------|----------|
| Batch | Single/Multi | None | Slow | Payroll, Reports |
| Multiprogramming | Multi | Limited | Medium | Server systems |
| Time-sharing | Multi | Interactive | Fast | Desktop OS |
| Real-time | Single/Multi | Interactive | Guaranteed | Medical, Industrial |
| Distributed | Multi | Network | Variable | Cloud computing |
| Mobile | Single | Touch | Fast | Smartphones |

---

## Visual Comparison

```
Response Time Requirements:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Hard Real-Time │████████████████████████████████████│ Strictest
Soft Real-Time │██████████████████████████│
Time-sharing   │████████████████│
Batch         │████│ Most relaxed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
              Microseconds    Milliseconds    Seconds/Minutes
```

---

## Quick Revision Questions

1. What is the main difference between multiprogramming and time-sharing?
2. Why do we need real-time operating systems?
3. How does a distributed OS differ from a network OS?
4. What are the two types of multiprocessing systems?
5. Name three characteristics of mobile operating systems.

---

[← Previous: Functions of OS](./02-functions-of-os.md) | [Next: Operating System Structure →](./04-os-structure.md)
