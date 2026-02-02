# Chapter 1.2: Functions of Operating System

## Overview

The Operating System performs several critical functions to ensure smooth operation of the computer system. These functions can be categorized into major areas of responsibility.

---

## Core Functions of Operating System

```
                    ┌─────────────────────┐
                    │   OPERATING SYSTEM  │
                    │      FUNCTIONS      │
                    └─────────┬───────────┘
                              │
    ┌─────────────────────────┼─────────────────────────┐
    │           │             │             │           │
    ▼           ▼             ▼             ▼           ▼
┌───────┐ ┌─────────┐ ┌───────────┐ ┌─────────┐ ┌──────────┐
│Process│ │ Memory  │ │   File    │ │   I/O   │ │ Security │
│Mgmt   │ │  Mgmt   │ │   Mgmt    │ │   Mgmt  │ │  Mgmt    │
└───────┘ └─────────┘ └───────────┘ └─────────┘ └──────────┘
```

---

## 1. Process Management

### What is a Process?
A **process** is a program in execution. While a program is a passive entity (file on disk), a process is an active entity with a program counter and associated resources.

### OS Responsibilities in Process Management

| Function | Description |
|----------|-------------|
| **Process Creation** | Starting new processes (fork, exec) |
| **Process Termination** | Ending processes (normal/abnormal) |
| **Process Scheduling** | Deciding which process runs when |
| **Process Synchronization** | Coordinating concurrent processes |
| **Process Communication** | Enabling inter-process communication |
| **Deadlock Handling** | Preventing/resolving resource conflicts |

### Process Management Analogy
Think of the OS as a **restaurant manager**:
- Creating a process = Seating a new customer
- Scheduling = Deciding which table to serve next
- Termination = Customer leaves after meal
- Synchronization = Coordinating multiple waiters

---

## 2. Memory Management

### Why Memory Management?
- Main memory is a limited resource
- Multiple programs compete for memory
- Programs need memory to execute
- Efficient memory use improves performance

### OS Responsibilities in Memory Management

```
┌────────────────────────────────────────┐
│            MAIN MEMORY                 │
├────────────────────────────────────────┤
│  OS (Kernel)        │ Always resident  │
├─────────────────────┼──────────────────┤
│  Process 1          │ User space       │
├─────────────────────┤                  │
│  Process 2          │                  │
├─────────────────────┤                  │
│  Process 3          │                  │
├─────────────────────┼──────────────────┤
│  Free Space         │ Available        │
└────────────────────────────────────────┘
```

| Function | Description |
|----------|-------------|
| **Memory Allocation** | Allocating memory to processes |
| **Memory Deallocation** | Freeing memory when process ends |
| **Memory Tracking** | Knowing which parts are used/free |
| **Memory Protection** | Preventing unauthorized access |
| **Virtual Memory** | Using disk as extended memory |

### Key Concepts
- **Address Binding**: Mapping logical addresses to physical addresses
- **Swapping**: Moving processes between main memory and disk
- **Paging/Segmentation**: Dividing memory into manageable units

---

## 3. File Management

### What is a File?
A **file** is a collection of related information defined by its creator. It's the logical storage unit abstracted from physical storage.

### OS Responsibilities in File Management

```
┌─────────────────────────────────────┐
│           FILE SYSTEM               │
├─────────────────────────────────────┤
│  Directory Structure                │
│    ├── Users/                       │
│    │   ├── Documents/               │
│    │   │   ├── file1.txt           │
│    │   │   └── file2.pdf           │
│    │   └── Pictures/                │
│    └── System/                      │
└─────────────────────────────────────┘
```

| Function | Description |
|----------|-------------|
| **File Creation** | Creating new files |
| **File Deletion** | Removing files |
| **File Opening/Closing** | Managing file access |
| **Reading/Writing** | Data transfer operations |
| **Directory Management** | Organizing files hierarchically |
| **Access Control** | Managing permissions (read, write, execute) |
| **Backup** | Creating copies for recovery |

### File Attributes
- Name, Type, Size
- Location (path)
- Time stamps (created, modified, accessed)
- Owner and permissions

---

## 4. I/O Device Management

### Challenge
- Wide variety of I/O devices (keyboard, mouse, printer, disk, network)
- Each device has different characteristics
- Need uniform interface for programs

### OS Responsibilities in I/O Management

| Function | Description |
|----------|-------------|
| **Device Drivers** | Software interface for hardware |
| **Buffering** | Temporary storage for data transfer |
| **Caching** | Keeping frequently used data accessible |
| **Spooling** | Managing shared device access (e.g., printer) |
| **Device Allocation** | Assigning devices to processes |

### I/O System Architecture

```
┌─────────────────────────────────────────┐
│           Application Program           │
├─────────────────────────────────────────┤
│          I/O System Calls               │
├─────────────────────────────────────────┤
│       Device-Independent Layer          │
├─────────────────────────────────────────┤
│           Device Drivers                │
├──────────┬──────────┬──────────┬────────┤
│  Disk    │ Keyboard │ Printer  │Network │
│Controller│Controller│Controller│  Card  │
└──────────┴──────────┴──────────┴────────┘
```

---

## 5. Security and Protection

### Security vs Protection

| Aspect | Security | Protection |
|--------|----------|------------|
| **Focus** | External threats | Internal access control |
| **Concern** | Unauthorized access | Resource access by processes |
| **Scope** | System-wide | Process-level |

### OS Responsibilities

| Function | Description |
|----------|-------------|
| **User Authentication** | Verifying user identity (passwords, biometrics) |
| **Authorization** | Determining access permissions |
| **Encryption** | Protecting data confidentiality |
| **Auditing** | Logging system activities |
| **Firewall Management** | Controlling network access |

### Access Control Matrix

```
              │  File1  │  File2  │ Printer │
──────────────┼─────────┼─────────┼─────────┤
   User A     │  R/W    │   R     │  Print  │
──────────────┼─────────┼─────────┼─────────┤
   User B     │   R     │  R/W    │  Print  │
──────────────┼─────────┼─────────┼─────────┤
   User C     │   -     │   R     │    -    │
```

---

## 6. Secondary Storage Management

### Why Secondary Storage?
- Main memory is volatile (loses data when power off)
- Main memory is limited and expensive
- Need permanent storage for programs and data

### OS Responsibilities

| Function | Description |
|----------|-------------|
| **Free Space Management** | Tracking available storage |
| **Storage Allocation** | Allocating space to files |
| **Disk Scheduling** | Optimizing disk access order |
| **Storage Hierarchy** | Managing cache → RAM → SSD → HDD |

---

## 7. Networking (Distributed Systems)

### In Networked/Distributed Systems

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Computer │◄───►│ Computer │◄───►│ Computer │
│    A     │     │    B     │     │    C     │
└──────────┘     └──────────┘     └──────────┘
      │               │               │
      └───────────────┴───────────────┘
                      │
              ┌───────┴───────┐
              │    Network    │
              │ (Internet/LAN)│
              └───────────────┘
```

| Function | Description |
|----------|-------------|
| **Protocol Management** | TCP/IP, UDP handling |
| **Socket Management** | Network communication endpoints |
| **Remote Procedure Calls** | Executing procedures on remote systems |
| **Distributed File Systems** | Accessing files across network |

---

## 8. Command Interpretation

### Shell/Command Interpreter

The **shell** reads commands from user and executes them.

```bash
# Example: Command Line Interface
$ ls -la          # List files
$ cd /home        # Change directory  
$ mkdir newdir    # Create directory
$ rm file.txt     # Delete file
```

### Types of Interfaces

| Type | Description | Example |
|------|-------------|---------|
| **CLI** | Command Line Interface | Bash, PowerShell |
| **GUI** | Graphical User Interface | Windows Explorer |
| **Touch** | Touch-based interface | Smartphone OS |

---

## Function Summary Diagram

```
┌───────────────────────────────────────────────────────────┐
│                   OPERATING SYSTEM                         │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │   Process   │  │   Memory    │  │    File     │       │
│  │ Management  │  │ Management  │  │ Management  │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
│                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │    I/O      │  │  Security   │  │  Network    │       │
│  │ Management  │  │ Protection  │  │ Management  │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
│                                                           │
├───────────────────────────────────────────────────────────┤
│               Command Interpreter (Shell)                  │
└───────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Function | Primary Goal | Key Activities |
|----------|--------------|----------------|
| Process Management | CPU Utilization | Scheduling, Synchronization |
| Memory Management | Efficient Memory Use | Allocation, Virtual Memory |
| File Management | Data Organization | Storage, Retrieval, Protection |
| I/O Management | Device Abstraction | Drivers, Buffering, Spooling |
| Security | System Protection | Authentication, Authorization |
| Storage Management | Permanent Storage | Disk Scheduling, Allocation |
| Networking | Communication | Protocols, Resource Sharing |
| Command Interpretation | User Interface | CLI, GUI |

---

## Quick Revision Questions

1. What is the difference between a program and a process?
2. Why is memory management necessary?
3. What is the role of device drivers?
4. How does the OS ensure security?
5. What is spooling and when is it used?

---

[← Previous: Introduction](./01-introduction-definition.md) | [Next: Types of Operating Systems →](./03-types-of-os.md)
