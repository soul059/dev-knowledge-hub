# Chapter 2.2: Process Control Block (PCB)

## Overview

The Process Control Block (PCB) is the most important data structure in an operating system. It contains all the information needed to manage a process and is the key to understanding how the OS tracks and controls processes.

---

## What is a Process Control Block?

### Definition
A **Process Control Block (PCB)**, also called a **Task Control Block**, is a data structure maintained by the operating system for every process. It contains all the information about a process that the OS needs to manage it.

### Analogy
Think of a PCB like an **employee record** in a company:
- Just as HR maintains records for each employee (name, ID, department, salary, attendance)
- The OS maintains a PCB for each process (PID, state, registers, memory info)

```
┌─────────────────────────────────────────────────────────────┐
│   HR Records (Company)          PCB (Operating System)     │
├─────────────────────────────────────────────────────────────┤
│   Employee ID                   Process ID (PID)           │
│   Name                          Process Name               │
│   Department                    Process State              │
│   Position                      Priority                   │
│   Salary Info                   CPU Registers              │
│   Attendance                    Memory Info                │
│   Projects Assigned             Files Open                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Structure of PCB

The PCB contains comprehensive information about a process:

```
┌──────────────────────────────────────────────────────────┐
│               PROCESS CONTROL BLOCK (PCB)                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  1. PROCESS IDENTIFICATION                         │  │
│  │     • Process ID (PID)                             │  │
│  │     • Parent Process ID (PPID)                     │  │
│  │     • User ID (UID)                                │  │
│  │     • Group ID (GID)                               │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  2. PROCESS STATE INFORMATION                      │  │
│  │     • Current State (New/Ready/Running/etc.)       │  │
│  │     • Priority                                     │  │
│  │     • Scheduling Information                       │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  3. CPU STATE (CONTEXT)                            │  │
│  │     • Program Counter (PC)                         │  │
│  │     • CPU Registers                                │  │
│  │     • Stack Pointer                                │  │
│  │     • PSW (Program Status Word)                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  4. MEMORY MANAGEMENT INFORMATION                  │  │
│  │     • Base and Limit Registers                     │  │
│  │     • Page Tables / Segment Tables                 │  │
│  │     • Memory Allocated                             │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  5. I/O STATUS INFORMATION                         │  │
│  │     • List of Open Files                           │  │
│  │     • I/O Devices Allocated                        │  │
│  │     • Pending I/O Requests                         │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  6. ACCOUNTING INFORMATION                         │  │
│  │     • CPU Time Used                                │  │
│  │     • Time Limits                                  │  │
│  │     • Account Numbers                              │  │
│  │     • Job/Process Numbers                          │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  7. POINTERS                                       │  │
│  │     • Pointer to Parent PCB                        │  │
│  │     • Pointer to Child PCB(s)                      │  │
│  │     • Pointer to Next PCB in Queue                 │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Detailed PCB Components

### 1. Process Identification

| Field | Description |
|-------|-------------|
| **Process ID (PID)** | Unique integer identifying the process |
| **Parent PID (PPID)** | PID of the process that created this process |
| **User ID (UID)** | Identifier of the user who owns the process |
| **Group ID (GID)** | Identifier of the group |

```
Example:
┌─────────────────────────────┐
│ PID: 1234                   │
│ PPID: 1                     │
│ UID: 1000 (user: john)     │
│ GID: 1000 (group: users)   │
└─────────────────────────────┘
```

### 2. Process State Information

| Field | Description |
|-------|-------------|
| **Process State** | Current state (New, Ready, Running, Waiting, Terminated) |
| **Priority** | Scheduling priority (higher priority = more CPU time) |
| **Scheduling Info** | Pointers to scheduling queues, scheduling parameters |
| **Event Info** | Event the process is waiting for (if blocked) |

```
Example:
┌─────────────────────────────┐
│ State: READY                │
│ Priority: 10                │
│ Time in Current State: 5ms  │
│ Waiting For: N/A            │
└─────────────────────────────┘
```

### 3. CPU State (Context/Register State)

This is critical for **context switching**. When a process is interrupted, all register values are saved here.

| Field | Description |
|-------|-------------|
| **Program Counter (PC)** | Address of next instruction to execute |
| **Stack Pointer (SP)** | Top of the current stack |
| **General Registers** | Accumulator, Index registers, etc. |
| **PSW** | Condition codes, status info, interrupt mask |

```
CPU Registers to Save:
┌─────────────────────────────────────────┐
│  PC (Program Counter): 0x004521F0      │
│  SP (Stack Pointer): 0x7FFF5000        │
│  AX: 0x00001234                        │
│  BX: 0x00005678                        │
│  CX: 0x0000ABCD                        │
│  DX: 0x0000EF01                        │
│  Flags: ZF=0, CF=1, OF=0              │
└─────────────────────────────────────────┘
```

### 4. Memory Management Information

| Field | Description |
|-------|-------------|
| **Base Register** | Starting address of process memory |
| **Limit Register** | Size of process memory |
| **Page Table Pointer** | Pointer to process's page table |
| **Segment Table Pointer** | Pointer to segment table |

```
Memory Layout:
┌─────────────────────────────────────────┐
│  Base Address: 0x00400000               │
│  Limit: 64 KB                          │
│  Page Table Base: 0x00100000           │
│  Number of Pages: 16                   │
│  Memory Allocated: 65536 bytes         │
└─────────────────────────────────────────┘
```

### 5. I/O Status Information

| Field | Description |
|-------|-------------|
| **Open File Table** | List of files opened by the process |
| **I/O Devices** | Devices allocated to the process |
| **Pending I/O** | I/O operations in progress |

```
File Descriptor Table:
┌─────┬──────────────────────────┐
│ FD  │ File                     │
├─────┼──────────────────────────┤
│  0  │ stdin (keyboard)         │
│  1  │ stdout (terminal)        │
│  2  │ stderr (terminal)        │
│  3  │ /home/user/data.txt      │
│  4  │ /tmp/log.txt             │
└─────┴──────────────────────────┘
```

### 6. Accounting Information

| Field | Description |
|-------|-------------|
| **CPU Time Used** | Total CPU time consumed |
| **Wall Clock Time** | Real time since process started |
| **Time Limits** | Maximum allowed CPU time |
| **I/O Time** | Time spent in I/O operations |

```
Accounting Data:
┌─────────────────────────────────────────┐
│  CPU Time Used: 2.5 seconds            │
│  Start Time: 14:30:25                  │
│  Elapsed Time: 45 seconds              │
│  Time Limit: 3600 seconds              │
│  Memory Usage: 15 MB                   │
│  I/O Operations: 150                   │
└─────────────────────────────────────────┘
```

### 7. Pointers

| Field | Description |
|-------|-------------|
| **Parent Pointer** | Link to parent process PCB |
| **Child Pointers** | Links to child process PCBs |
| **Queue Pointers** | Links to next/previous PCB in queues |

```
PCB Linked Structure:
     Ready Queue:
     ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
     │PCB 1│──→│PCB 2│──→│PCB 3│──→│PCB 4│──→ NULL
     └─────┘   └─────┘   └─────┘   └─────┘
```

---

## PCB in Linux (task_struct)

In Linux, the PCB is implemented as `struct task_struct`. It's a large structure containing all process information.

### Simplified task_struct

```c
// Simplified representation of Linux task_struct
struct task_struct {
    // Process State
    volatile long state;        // -1 unrunnable, 0 runnable, >0 stopped
    
    // Process Identification
    pid_t pid;                  // Process ID
    pid_t tgid;                 // Thread Group ID
    
    // Process Relationships
    struct task_struct *parent; // Parent process
    struct list_head children;  // List of children
    struct list_head sibling;   // Linkage in parent's children list
    
    // Scheduling Information
    int prio;                   // Dynamic priority
    int static_prio;            // Static priority
    unsigned int policy;        // Scheduling policy
    
    // CPU Context
    struct thread_struct thread; // CPU-specific state
    
    // Memory Management
    struct mm_struct *mm;       // Memory descriptor
    
    // File System
    struct files_struct *files; // Open file information
    
    // Credentials
    const struct cred *cred;    // Process credentials (UID, GID)
    
    // Timing Information
    u64 utime;                  // User mode CPU time
    u64 stime;                  // System mode CPU time
    
    // ... many more fields
};
```

---

## PCB and Process Queues

The OS maintains various queues of PCBs for scheduling:

```
                    ┌───────────────────────────────────────┐
                    │           PROCESS QUEUES              │
                    └───────────────────────────────────────┘

JOB QUEUE (All processes in system):
┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
│PCB 1│──→│PCB 2│──→│PCB 3│──→│PCB 4│──→│PCB 5│──→ NULL
└─────┘   └─────┘   └─────┘   └─────┘   └─────┘


READY QUEUE (Processes ready to run):
┌─────┐   ┌─────┐   ┌─────┐
│PCB 2│──→│PCB 4│──→│PCB 5│──→ NULL
└─────┘   └─────┘   └─────┘


DEVICE QUEUES (Processes waiting for I/O):

Disk Queue:       ┌─────┐   ┌─────┐
                  │PCB 1│──→│PCB 3│──→ NULL
                  └─────┘   └─────┘

Keyboard Queue:   ┌─────┐
                  │PCB 6│──→ NULL
                  └─────┘

Printer Queue:    ┌─────┐   ┌─────┐   ┌─────┐
                  │PCB 7│──→│PCB 8│──→│PCB 9│──→ NULL
                  └─────┘   └─────┘   └─────┘
```

---

## Context Switch

### What is Context Switching?

A **context switch** occurs when the OS saves the state of one process and loads the state of another. The PCB is central to this operation.

### Context Switch Process

```
┌────────────────────────────────────────────────────────────────┐
│                     CONTEXT SWITCH                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Process P0 (Running)              Process P1 (Ready)          │
│       │                                  ▲                     │
│       │ 1. Interrupt/System Call         │                     │
│       ▼                                  │                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    OPERATING SYSTEM                      │   │
│  │                                                         │   │
│  │  2. Save state of P0 to PCB0                           │   │
│  │     ┌──────────────┐                                    │   │
│  │     │    PCB 0     │ ← Save PC, registers, state       │   │
│  │     │  State:      │                                    │   │
│  │     │  Waiting/    │                                    │   │
│  │     │  Ready       │                                    │   │
│  │     └──────────────┘                                    │   │
│  │                                                         │   │
│  │  3. Reload state from PCB1                              │   │
│  │     ┌──────────────┐                                    │   │
│  │     │    PCB 1     │ ← Load PC, registers, state       │   │
│  │     │  State:      │                                    │   │
│  │     │  Running     │                                    │   │
│  │     └──────────────┘                                    │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│       │                                  │                     │
│       │ 4. Return to P1                  │                     │
│       ▼                                  │                     │
│  Process P0 (Waiting)              Process P1 (Running)        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Context Switch Timeline

```
Time ──────────────────────────────────────────────────────────→

Process P0:  ████████████│         │░░░░░░░░░░░░│████████████
                         │ Context │            │
Process P1:  ░░░░░░░░░░░░│ Switch  │████████████│░░░░░░░░░░░░
                         │ Overhead│
                         
████ = Running    ░░░░ = Not Running/Waiting

Context Switch Steps:
1. Save P0's PC, registers → PCB0
2. Update P0's state to Ready/Waiting
3. Move P0's PCB to appropriate queue
4. Select P1 from Ready queue
5. Update P1's state to Running
6. Load P1's PC, registers from PCB1
7. Resume P1 execution
```

### Context Switch Overhead

Context switching is **pure overhead** - no useful work is done during the switch.

| Factor | Impact on Overhead |
|--------|-------------------|
| **Hardware Support** | More registers = more to save/restore |
| **OS Complexity** | More PCB fields = more to process |
| **Memory Speed** | Faster memory = faster context switch |
| **TLB Flush** | Page table changes require TLB flush |

**Typical Times**: 1-1000 microseconds depending on system

---

## PCB Creation

When a new process is created, a new PCB is allocated and initialized:

```
Process Creation Steps:
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  1. Allocate new PCB                                       │
│     ┌──────────────┐                                       │
│     │   New PCB    │  ← Empty PCB allocated               │
│     └──────────────┘                                       │
│                                                            │
│  2. Assign unique PID                                      │
│     ┌──────────────┐                                       │
│     │  PID: 1234   │  ← System assigns unique ID          │
│     └──────────────┘                                       │
│                                                            │
│  3. Initialize PCB fields                                  │
│     ┌──────────────────────────────────┐                  │
│     │  State: NEW                       │                  │
│     │  Priority: Default                │                  │
│     │  PC: Entry point of program       │                  │
│     │  Memory: Allocate pages          │                  │
│     │  Files: Copy from parent or init │                  │
│     └──────────────────────────────────┘                  │
│                                                            │
│  4. Link into process tree                                 │
│     Parent PCB ──→ Child PCB (new)                        │
│                                                            │
│  5. Place in Ready queue                                   │
│     Ready Queue: [...] ──→ [New PCB]                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## PCB Destruction

When a process terminates, its PCB is cleaned up:

```
Process Termination Steps:
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  1. Process calls exit() or is killed                      │
│                                                            │
│  2. Release all resources                                  │
│     • Close open files                                     │
│     • Release memory                                       │
│     • Release I/O devices                                  │
│                                                            │
│  3. Store exit status in PCB                               │
│     ┌──────────────────────────────────┐                  │
│     │  State: TERMINATED (ZOMBIE)      │                  │
│     │  Exit Status: 0                  │                  │
│     └──────────────────────────────────┘                  │
│                                                            │
│  4. Signal parent (SIGCHLD)                                │
│                                                            │
│  5. Wait for parent to call wait()                         │
│     • Parent reads exit status                             │
│     • PCB can now be freed                                 │
│                                                            │
│  6. Deallocate PCB                                         │
│     • Remove from all queues                               │
│     • Free PCB memory                                      │
│     • PID can be reused                                    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Process Table

The OS maintains a **Process Table** - an array or linked list of all PCBs in the system.

```
┌──────────────────────────────────────────────────────────────┐
│                      PROCESS TABLE                            │
├──────┬───────────────────────────────────────────────────────┤
│ Index│                      PCB                              │
├──────┼───────────────────────────────────────────────────────┤
│   0  │ [PID:1, State:Running, Priority:0, Parent:0, ...]    │
├──────┼───────────────────────────────────────────────────────┤
│   1  │ [PID:100, State:Ready, Priority:5, Parent:1, ...]    │
├──────┼───────────────────────────────────────────────────────┤
│   2  │ [PID:101, State:Waiting, Priority:10, Parent:1, ...] │
├──────┼───────────────────────────────────────────────────────┤
│   3  │ [PID:200, State:Ready, Priority:5, Parent:100, ...]  │
├──────┼───────────────────────────────────────────────────────┤
│  ... │ ...                                                   │
├──────┼───────────────────────────────────────────────────────┤
│ MAX  │ [Empty or NULL]                                       │
└──────┴───────────────────────────────────────────────────────┘

Maximum number of processes = Size of Process Table
```

---

## Summary

| Component | Purpose |
|-----------|---------|
| **Process ID** | Unique identification |
| **State** | Current process state |
| **CPU State** | Register values for context switch |
| **Memory Info** | Memory allocation details |
| **I/O Info** | Open files and devices |
| **Accounting** | Resource usage statistics |
| **Pointers** | Links to related PCBs |

### Key Points

1. **PCB is the identity card** of a process in the OS
2. **Context switch** saves/restores PCB information
3. **Process queues** are implemented as linked lists of PCBs
4. **PCB size** affects context switch overhead
5. **Process table** limits maximum number of processes

---

## Quick Revision Questions

1. What is a PCB and why is it important?
2. List the main components of a PCB.
3. How is the PCB used during a context switch?
4. What happens to the PCB when a process terminates?
5. What is the relationship between PCB and process queues?

---

[← Previous: Process Concepts](./01-process-concepts.md) | [Next: Process Scheduling →](./03-process-scheduling.md)
