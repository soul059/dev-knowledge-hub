# Chapter 2.1: Process Concepts

## Overview

A process is one of the most fundamental concepts in operating systems. Understanding processes is essential for grasping how the OS manages program execution, resource allocation, and multitasking.

---

## What is a Process?

### Definition
A **Process** is a program in execution. It is an active entity, as opposed to a program which is a passive entity stored on disk.

### Program vs Process

| Aspect | Program | Process |
|--------|---------|---------|
| **Nature** | Passive (static) | Active (dynamic) |
| **Storage** | Stored on disk | Resides in memory |
| **Lifetime** | Permanent until deleted | Temporary (created & terminated) |
| **Resources** | None | CPU, memory, I/O, files |
| **State** | None | Has state (running, waiting, etc.) |

### Analogy
Think of the relationship between a program and a process like a **recipe and cooking**:
- **Program (Recipe)**: Written instructions stored in a book
- **Process (Cooking)**: Actually following the recipe - you're actively using ingredients (resources), following steps (program counter), and the state changes as you cook

```
Program (Passive)                    Process (Active)
┌─────────────────┐                 ┌─────────────────────────┐
│   calculator.exe │                 │  calculator.exe running │
│   (on disk)      │  ──Execute──→  │  - Using CPU            │
│   Static file    │                 │  - Using Memory         │
│                  │                 │  - Has state            │
└─────────────────┘                 └─────────────────────────┘
```

---

## Process Memory Layout

When a process is created, the operating system allocates memory organized into distinct sections:

```
┌─────────────────────────────────┐  High Address (e.g., 0xFFFF)
│            STACK                │  ↓ Grows downward
│   - Local variables             │
│   - Function parameters         │
│   - Return addresses            │
├─────────────────────────────────┤
│              ↓                  │
│                                 │
│         (Free Space)            │
│                                 │
│              ↑                  │
├─────────────────────────────────┤
│            HEAP                 │  ↑ Grows upward
│   - Dynamically allocated       │
│   - malloc(), new               │
├─────────────────────────────────┤
│            DATA                 │
│   - Global variables            │
│   - Static variables            │
│   ┌───────────────────────────┐ │
│   │ Initialized Data (.data)  │ │
│   ├───────────────────────────┤ │
│   │ Uninitialized Data (.bss) │ │
│   └───────────────────────────┘ │
├─────────────────────────────────┤
│            TEXT                 │
│   - Program code (instructions) │
│   - Read-only                   │
│   - Sharable between processes  │
└─────────────────────────────────┘  Low Address (e.g., 0x0000)
```

### Memory Sections Explained

| Section | Contents | Characteristics |
|---------|----------|-----------------|
| **Text** | Compiled program code | Read-only, fixed size |
| **Data** | Global and static variables | Initialized at compile time |
| **BSS** | Uninitialized global/static | Initialized to zero at runtime |
| **Heap** | Dynamically allocated memory | Grows upward, programmer managed |
| **Stack** | Function calls, local variables | Grows downward, auto managed |

### Code Example Mapping

```c
#include <stdio.h>
#include <stdlib.h>

int global_var = 100;        // Data section (initialized)
int uninit_var;              // BSS section (uninitialized)

int main() {                 // Text section (code)
    int local_var = 10;      // Stack
    static int static_var;   // BSS section
    
    int *ptr = malloc(sizeof(int) * 10);  // Heap
    
    // Use variables...
    
    free(ptr);               // Return heap memory
    return 0;
}
```

---

## Process States

A process transitions through various states during its lifetime. The OS tracks and manages these states.

### Five-State Process Model

```
                          ┌─────────────────────────────────────────┐
                          │                                         │
                          ↓                                         │
┌─────────┐         ┌──────────┐         ┌───────────┐         ┌────┴─────┐
│   NEW   │────────→│  READY   │←───────→│  RUNNING  │────────→│TERMINATED│
│         │ Admitted│          │Scheduler│           │  Exit   │          │
└─────────┘         └────┬─────┘ Dispatch└─────┬─────┘         └──────────┘
                         ↑                     │
                         │    I/O or Event     │
                         │     Completed       │
                         │                     ↓
                    ┌────┴─────────────────────────┐
                    │          WAITING             │
                    │   (Blocked for I/O/Event)    │
                    └──────────────────────────────┘
```

### State Descriptions

| State | Description | Example |
|-------|-------------|---------|
| **New** | Process is being created | User double-clicks application |
| **Ready** | Process waiting to be assigned to CPU | In queue, ready to run |
| **Running** | Process is being executed | Instructions being executed |
| **Waiting** | Process waiting for I/O or event | Waiting for file read |
| **Terminated** | Process has finished execution | Program exits normally |

### State Transitions

| Transition | From → To | Cause |
|------------|-----------|-------|
| Admitted | New → Ready | OS accepts process |
| Dispatch | Ready → Running | Scheduler selects process |
| Interrupt | Running → Ready | Time quantum expires |
| I/O Wait | Running → Waiting | Process requests I/O |
| I/O Complete | Waiting → Ready | I/O operation finishes |
| Exit | Running → Terminated | Process completes/killed |

### Seven-State Process Model

```
┌───────────────────────────────────────────────────────────────────┐
│                          MAIN MEMORY                               │
│                                                                   │
│    ┌─────────┐      ┌───────────┐      ┌───────────────┐         │
│    │  READY  │◄────►│  RUNNING  │─────►│  TERMINATED   │         │
│    └────┬────┘      └─────┬─────┘      └───────────────┘         │
│         │                 │                                       │
│         │                 │                                       │
│         │                 ↓                                       │
│         │           ┌───────────┐                                │
│         │           │  BLOCKED  │                                │
│         │           └─────┬─────┘                                │
│         │                 │                                       │
├─────────┼─────────────────┼───────────────────────────────────────┤
│         │    Swap Out     │    Swap Out                          │
│         ↓                 ↓                                       │
│    ┌────────────┐   ┌────────────────┐                           │
│    │   READY    │   │    BLOCKED     │      SECONDARY STORAGE    │
│    │  SUSPEND   │   │    SUSPEND     │      (Swapped Out)        │
│    └────────────┘   └────────────────┘                           │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### Suspended States

| State | Description | When Used |
|-------|-------------|-----------|
| **Ready/Suspend** | Ready but swapped to disk | Memory pressure |
| **Blocked/Suspend** | Blocked and swapped to disk | Long I/O wait + memory pressure |

---

## Process Creation

### Reasons for Process Creation

1. **System Initialization**: Boot-time processes (daemons, services)
2. **User Request**: User starts an application
3. **Batch Job**: Scheduled job starts
4. **Process Spawning**: Running process creates child process

### Parent-Child Relationship

```
                    ┌─────────────┐
                    │   INIT      │  PID = 1 (First process)
                    │  (Parent)   │
                    └──────┬──────┘
                           │ Creates
           ┌───────────────┼───────────────┐
           ↓               ↓               ↓
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │   Process A  │ │   Process B  │ │   Process C  │
    │   PID = 100  │ │   PID = 200  │ │   PID = 300  │
    └──────┬───────┘ └──────────────┘ └──────────────┘
           │ Creates
    ┌──────┴──────┐
    ↓             ↓
┌────────┐   ┌────────┐
│Child A1│   │Child A2│
│PID=101 │   │PID=102 │
└────────┘   └────────┘
```

### Process Creation in UNIX (fork)

```
Before fork():                 After fork():
┌─────────────────┐            ┌─────────────────┐
│  Parent Process │            │  Parent Process │
│    PID = 100    │            │    PID = 100    │
│                 │            │  fork() = 101   │ (returns child PID)
└─────────────────┘            └─────────────────┘
                                        │
                                        │ Creates copy
                                        ↓
                               ┌─────────────────┐
                               │  Child Process  │
                               │    PID = 101    │
                               │  fork() = 0     │ (returns 0 in child)
                               └─────────────────┘
```

### Resource Sharing Options

| Option | Description |
|--------|-------------|
| **All resources shared** | Parent and child share everything |
| **Subset shared** | Child shares some parent resources |
| **No sharing** | Child gets own copies of resources |

### Execution Options

| Option | Description |
|--------|-------------|
| **Concurrent** | Parent and child execute simultaneously |
| **Sequential** | Parent waits for child to finish |

---

## Process Termination

### Normal Termination
- Process executes last statement
- Calls `exit()` system call
- Returns status to parent via `wait()`

### Abnormal Termination
- **Kill by parent**: Parent terminates child (abort)
- **Kill by OS**: Resource limits exceeded, errors
- **Kill by user**: User sends kill signal

### Reasons for Process Termination

| Reason | Description |
|--------|-------------|
| Normal Exit | Task completed successfully |
| Error Exit | Process detects error, exits |
| Fatal Error | Divide by zero, invalid memory access |
| Killed | Another process terminates it |

### Cascading Termination

When a parent terminates, what happens to children?

```
Option 1: Cascading Termination
Parent terminates → All children terminated

Option 2: Orphan Adoption
Parent terminates → Children adopted by init process

          ┌───────────┐
          │   INIT    │ (PID=1) adopts orphans
          └─────┬─────┘
                │
        ┌───────┴───────┐
        ↓               ↓
   ┌────────┐      ┌────────┐
   │Orphan 1│      │Orphan 2│
   └────────┘      └────────┘
```

### Zombie Process

A process that has terminated but whose parent hasn't called `wait()` yet.

```
┌─────────────┐           ┌─────────────┐
│   Parent    │           │   Child     │
│  (Running)  │           │  (Zombie)   │
│             │           │             │
│ Not calling │           │ Terminated  │
│   wait()    │           │ but entry   │
│             │           │ remains in  │
│             │           │ process     │
│             │           │ table       │
└─────────────┘           └─────────────┘

Problem: Zombie takes up process table entry
Solution: Parent must call wait() to reap zombie
```

---

## Process Identification

### Process Identifiers (PIDs)

Every process has a unique identifier:

```
┌────────────────────────────────────────────────┐
│              PROCESS IDENTIFIERS                │
├────────────────────────────────────────────────┤
│  PID    │  Process ID (unique identifier)      │
│  PPID   │  Parent Process ID                   │
│  UID    │  User ID (who owns the process)     │
│  GID    │  Group ID                            │
└────────────────────────────────────────────────┘
```

### Viewing Processes (Linux/UNIX)

```bash
# List all processes
$ ps aux

# Process tree
$ pstree

# Real-time process monitoring
$ top
$ htop

# Get current process ID
$ echo $$

# Get parent process ID
$ echo $PPID
```

---

## Context of a Process

The **context** of a process is all the information needed to describe the current state of a process.

### What Makes Up Process Context?

```
┌───────────────────────────────────────────────────────┐
│                   PROCESS CONTEXT                      │
├───────────────────────────────────────────────────────┤
│                                                       │
│  1. CPU Registers                                     │
│     ├── Program Counter (PC)                          │
│     ├── Stack Pointer (SP)                            │
│     ├── General Purpose Registers                     │
│     └── Status Registers                              │
│                                                       │
│  2. Memory Management Info                            │
│     ├── Page Tables                                   │
│     ├── Segment Tables                                │
│     └── Memory Limits                                 │
│                                                       │
│  3. I/O Status                                        │
│     ├── Open Files                                    │
│     ├── I/O Devices Allocated                         │
│     └── Pending I/O Operations                        │
│                                                       │
│  4. Accounting Info                                   │
│     ├── CPU Time Used                                 │
│     ├── Time Limits                                   │
│     └── Process Number                                │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## Process Operations Summary

### Creation
```
fork() → Creates new process (copy of parent)
exec() → Replaces process memory with new program
```

### Termination
```
exit() → Normal termination
abort() → Abnormal termination
kill() → Send signal to terminate
```

### Waiting
```
wait() → Parent waits for child
waitpid() → Wait for specific child
```

---

## Practical Example: Process Lifecycle

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {
    printf("Parent Process: PID = %d\n", getpid());
    
    pid_t pid = fork();  // Create child process
    
    if (pid < 0) {
        // Fork failed
        fprintf(stderr, "Fork Failed\n");
        exit(1);
    }
    else if (pid == 0) {
        // Child process
        printf("Child Process: PID = %d, Parent = %d\n", 
               getpid(), getppid());
        printf("Child: Doing some work...\n");
        sleep(2);
        printf("Child: Work complete. Exiting.\n");
        exit(0);  // Child terminates
    }
    else {
        // Parent process
        printf("Parent: Created child with PID = %d\n", pid);
        printf("Parent: Waiting for child...\n");
        
        int status;
        wait(&status);  // Wait for child to finish
        
        printf("Parent: Child has terminated\n");
        
        if (WIFEXITED(status)) {
            printf("Parent: Child exited with status %d\n", 
                   WEXITSTATUS(status));
        }
    }
    
    return 0;
}
```

**Output:**
```
Parent Process: PID = 1000
Parent: Created child with PID = 1001
Parent: Waiting for child...
Child Process: PID = 1001, Parent = 1000
Child: Doing some work...
Child: Work complete. Exiting.
Parent: Child has terminated
Parent: Child exited with status 0
```

---

## Summary

| Concept | Key Points |
|---------|------------|
| **Process** | Program in execution with resources |
| **Memory Layout** | Text, Data, Heap, Stack sections |
| **States** | New, Ready, Running, Waiting, Terminated |
| **Creation** | fork() creates child, exec() loads program |
| **Termination** | exit() normal, abort()/kill for abnormal |
| **Hierarchy** | Parent-child relationship, init as root |

---

## Quick Revision Questions

1. What is the difference between a program and a process?
2. Draw and explain the five-state process model.
3. What are the different sections of process memory?
4. What is a zombie process? How can it be prevented?
5. Explain the fork() system call with an example.

---

[← Previous: System Calls](../01-Introduction/05-system-calls.md) | [Next: Process Control Block →](./02-process-control-block.md)
