# Chapter 1.1: Introduction to Operating Systems

## What is an Operating System?

An **Operating System (OS)** is a program that acts as an intermediary between a user of a computer and the computer hardware. It is the most fundamental software that runs on a computer.

### Formal Definition

> An Operating System is system software that manages computer hardware and software resources and provides common services for computer programs.

### Analogy: The Government of a Computer

Think of an Operating System as the **government** of your computer:
- Just as a government manages resources (land, water, electricity) for citizens, an OS manages hardware resources (CPU, memory, storage) for programs
- Just as a government provides services (healthcare, education), an OS provides services (file management, security)
- Just as a government enforces laws, an OS enforces access control and security policies

---

## Goals of an Operating System

### 1. Convenience (User Goal)
Make the computer system convenient to use. Users shouldn't need to know hardware details to run programs.

### 2. Efficiency (System Goal)
Use computer hardware in an efficient manner. Maximize resource utilization while minimizing waste.

### 3. Ability to Evolve
The OS should be constructed in a way that permits effective development, testing, and introduction of new system functions without interfering with service.

---

## Computer System Components

A computer system can be divided into four components:

```
┌─────────────────────────────────────────────┐
│                   USER                       │
│         (People, Machines, Others)          │
├─────────────────────────────────────────────┤
│            APPLICATION PROGRAMS              │
│    (Compilers, Games, Web Browsers, etc.)   │
├─────────────────────────────────────────────┤
│            OPERATING SYSTEM                  │
│     (Windows, Linux, macOS, Android)        │
├─────────────────────────────────────────────┤
│               HARDWARE                       │
│      (CPU, Memory, I/O Devices)             │
└─────────────────────────────────────────────┘
```

### 1. Hardware
- Provides basic computing resources
- Includes: CPU, Memory, I/O devices, Storage

### 2. Operating System
- Controls and coordinates use of hardware
- Acts as resource manager and allocator

### 3. Application Programs
- Define ways in which system resources are used
- Examples: Word processors, compilers, web browsers, games

### 4. Users
- People, machines, or other computers that use the system

---

## Views of Operating System

### 1. User View
From a user's perspective, the OS is designed for:
- **Ease of use**: Simple interface to interact with
- **Performance**: Fast response time
- **Resource utilization**: Not a primary concern for single users

### 2. System View (Resource Allocator)
The OS acts as a **resource manager**:
- CPU Time
- Memory Space
- File-storage Space
- I/O Devices

It must decide how to allocate these resources to specific programs and users to operate efficiently and fairly.

### 3. System View (Control Program)
The OS is a **control program** that:
- Manages execution of user programs
- Prevents errors and improper use of computer
- Controls I/O devices and file management

---

## Historical Evolution of Operating Systems

### Generation 1 (1945-1955): No Operating System
- **Hardware**: Vacuum tubes, plug boards
- **Programming**: Machine language
- **Operation**: Manual operation, single user
- **Problem**: Extremely slow, no automation

### Generation 2 (1955-1965): Batch Systems
- **Hardware**: Transistors
- **Programming**: Assembly language, FORTRAN
- **Operation**: Batch processing with punch cards
- **Improvement**: Reduced setup time by batching similar jobs

```
Job 1 ─┐
Job 2 ─┼─→ [Batch] ─→ [Computer] ─→ Output
Job 3 ─┘
```

### Generation 3 (1965-1980): Multiprogramming & Time-sharing
- **Hardware**: Integrated circuits
- **Key Innovation**: Multiple jobs in memory simultaneously
- **Time-sharing**: CPU switches between jobs quickly
- **Example OS**: UNIX, MULTICS

### Generation 4 (1980-Present): Personal Computers & Beyond
- **Hardware**: Microprocessors, VLSI
- **Characteristics**: User-friendly, GUI-based
- **Examples**: MS-DOS, Windows, macOS, Linux

### Generation 5 (Present & Future): Mobile & Distributed Systems
- **Trends**: Cloud computing, IoT, Mobile OS
- **Examples**: Android, iOS, Chrome OS

---

## Key Terminology

| Term | Definition |
|------|------------|
| **Kernel** | The core component of OS that manages system resources |
| **Shell** | The interface between user and kernel |
| **Process** | A program in execution |
| **Thread** | A lightweight process, unit of CPU utilization |
| **System Call** | Interface for programs to request OS services |
| **Bootstrap** | The process of loading the OS into memory |

---

## Modern Operating System Examples

### Desktop/Laptop OS
- **Windows**: Most widely used, developed by Microsoft
- **macOS**: Apple's operating system for Mac computers
- **Linux**: Open-source, various distributions (Ubuntu, Fedora, etc.)

### Mobile OS
- **Android**: Google's mobile OS (Linux-based)
- **iOS**: Apple's mobile OS

### Server OS
- **Windows Server**: Enterprise server solutions
- **Linux Distributions**: Ubuntu Server, CentOS, Red Hat Enterprise Linux

### Real-time OS
- **VxWorks**: Used in spacecraft, medical devices
- **FreeRTOS**: For microcontrollers and small systems

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Definition** | Software that manages hardware and provides services |
| **Primary Goals** | Convenience, Efficiency, Evolvability |
| **Role** | Resource allocator and control program |
| **Components Managed** | CPU, Memory, Storage, I/O Devices |
| **Evolution** | From manual operation to modern multi-user systems |

---

## Quick Revision Questions

1. What is the primary function of an operating system?
2. How does an OS act as a resource allocator?
3. Name the four components of a computer system.
4. What was the key innovation in 3rd generation operating systems?
5. Differentiate between kernel and shell.

---

[Next: Functions of Operating System →](./02-functions-of-os.md)
