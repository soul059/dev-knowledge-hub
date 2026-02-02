# Chapter 1.5: System Calls

## Overview

System calls provide the interface between a running program and the operating system. They are the primary mechanism by which user programs request services from the kernel.

---

## What is a System Call?

### Definition
A **System Call** is a programmatic way in which a computer program requests a service from the kernel of the operating system. It provides a controlled entry point into the kernel.

### The User-Kernel Boundary

```
┌─────────────────────────────────────────────────────┐
│                    USER MODE                         │
│                                                     │
│    ┌───────────────────────────────────────────┐   │
│    │           Application Program              │   │
│    │                                           │   │
│    │   printf("Hello");  // Library call       │   │
│    │         │                                 │   │
│    │         ↓                                 │   │
│    │   write() in C library                    │   │
│    │         │                                 │   │
│    └─────────┼─────────────────────────────────┘   │
│              │                                      │
│              ↓ SYSTEM CALL (Trap/Interrupt)        │
├──────────────┼──────────────────────────────────────┤
│              │            KERNEL MODE               │
│              ↓                                      │
│    ┌───────────────────────────────────────────┐   │
│    │         System Call Handler                │   │
│    │                │                          │   │
│    │                ↓                          │   │
│    │         write() system call               │   │
│    │                │                          │   │
│    │                ↓                          │   │
│    │         Device Driver                     │   │
│    └───────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Why System Calls?

| Reason | Explanation |
|--------|-------------|
| **Protection** | Prevent user programs from damaging system |
| **Abstraction** | Hide hardware complexity from applications |
| **Resource Management** | Controlled access to shared resources |
| **Security** | Enforce access permissions |

---

## How System Calls Work

### Step-by-Step Process

```
1. Application calls library function
         ↓
2. Library prepares system call parameters
         ↓
3. Library executes trap instruction (software interrupt)
         ↓
4. CPU switches from User Mode to Kernel Mode
         ↓
5. System call handler identifies the call (by number)
         ↓
6. Appropriate kernel function executes
         ↓
7. Result returned, CPU switches back to User Mode
         ↓
8. Control returns to application
```

### Detailed Mechanism

```
┌───────────────────────────────────────────────────────────┐
│  User Program                                              │
│  ┌─────────────────┐                                      │
│  │ read(fd, buf, n)│ ← Library function call              │
│  └────────┬────────┘                                      │
│           ↓                                               │
│  ┌─────────────────┐                                      │
│  │ Load syscall #  │ ← Put system call number in register │
│  │ Load parameters │ ← Put parameters in registers/stack  │
│  │ INT 0x80 / TRAP │ ← Execute trap instruction          │
│  └────────┬────────┘                                      │
├───────────┼───────────────────────────────────────────────┤
│ KERNEL    ↓                                               │
│  ┌─────────────────┐                                      │
│  │ Save context    │ ← Save user program state           │
│  │ Identify call # │ ← Look up in system call table      │
│  │ sys_read()      │ ← Execute kernel function           │
│  │ Restore context │ ← Restore user program state        │
│  │ Return to user  │ ← Switch back to user mode          │
│  └─────────────────┘                                      │
└───────────────────────────────────────────────────────────┘
```

---

## Types of System Calls

System calls can be grouped into five major categories:

```
                    SYSTEM CALLS
                         │
    ┌────────────────────┼────────────────────┐
    │          │         │         │          │
    ↓          ↓         ↓         ↓          ↓
┌───────┐ ┌───────┐ ┌───────┐ ┌────────┐ ┌────────┐
│Process│ │ File  │ │Device │ │ Info   │ │Communi-│
│Control│ │Manage-│ │Manage-│ │Mainten-│ │cation  │
│       │ │ ment  │ │ ment  │ │ance    │ │        │
└───────┘ └───────┘ └───────┘ └────────┘ └────────┘
```

---

## 1. Process Control System Calls

These system calls manage processes - creating, executing, and terminating them.

### Common Process Control Calls

| System Call | Purpose | OS Example |
|-------------|---------|------------|
| `fork()` | Create a new process | UNIX/Linux |
| `exec()` | Execute a program | UNIX/Linux |
| `exit()` | Terminate process | UNIX/Linux |
| `wait()` | Wait for child process | UNIX/Linux |
| `CreateProcess()` | Create new process | Windows |
| `TerminateProcess()` | Terminate process | Windows |

### Process Creation Example (Conceptual)

```
Parent Process                    Child Process
┌─────────────┐                  ┌─────────────┐
│             │                  │             │
│   fork()  ──┼──────────────────→   (Copy of  │
│     │       │                  │   parent)   │
│     │       │                  │             │
│   wait()    │                  │   exec()    │
│     │       │                  │  (new prog) │
│     │       │                  │      │      │
│     │       │                  │   exit()    │
│     ↓       │←─────────────────│             │
│  continue   │                  └─────────────┘
└─────────────┘
```

### Code Example: fork() and exec()

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();  // Create child process
    
    if (pid < 0) {
        // Fork failed
        perror("Fork failed");
        return 1;
    }
    else if (pid == 0) {
        // Child process
        printf("Child: My PID is %d\n", getpid());
        execlp("ls", "ls", "-l", NULL);  // Replace with 'ls' program
    }
    else {
        // Parent process
        printf("Parent: Created child with PID %d\n", pid);
        wait(NULL);  // Wait for child to finish
        printf("Parent: Child has finished\n");
    }
    
    return 0;
}
```

---

## 2. File Management System Calls

These system calls handle file operations - creating, reading, writing, and deleting files.

### Common File Management Calls

| System Call | Purpose | Description |
|-------------|---------|-------------|
| `open()` | Open a file | Returns file descriptor |
| `close()` | Close a file | Release file descriptor |
| `read()` | Read from file | Get data from file |
| `write()` | Write to file | Put data into file |
| `lseek()` | Reposition | Move file pointer |
| `create()` | Create file | Create new file |
| `unlink()` | Delete file | Remove file |

### File Operations Flow

```
┌──────────────────────────────────────────────────┐
│                FILE OPERATIONS                    │
│                                                  │
│   open("file.txt")  ──→ Returns fd (e.g., 3)    │
│         │                                        │
│         ↓                                        │
│   read(fd, buffer, size)  ──→ Data in buffer    │
│         │                                        │
│         ↓                                        │
│   write(fd, buffer, size) ──→ Data to file      │
│         │                                        │
│         ↓                                        │
│   close(fd)  ──→ Release resources              │
│                                                  │
└──────────────────────────────────────────────────┘
```

### File Descriptor Table

```
Process's File Descriptor Table:
┌─────┬────────────────────────┐
│ FD  │     Points To          │
├─────┼────────────────────────┤
│  0  │ Standard Input (stdin) │
│  1  │ Standard Output(stdout)│
│  2  │ Standard Error (stderr)│
│  3  │ file.txt               │
│  4  │ data.dat               │
│ ... │ ...                    │
└─────┴────────────────────────┘
```

### Code Example: File Operations

```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int fd;
    char buffer[100];
    
    // Open file for reading
    fd = open("example.txt", O_RDONLY);
    if (fd < 0) {
        perror("Cannot open file");
        return 1;
    }
    
    // Read from file
    int bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    buffer[bytes_read] = '\0';
    
    printf("Read: %s\n", buffer);
    
    // Close file
    close(fd);
    
    return 0;
}
```

---

## 3. Device Management System Calls

These system calls manage I/O devices - requesting, releasing, and controlling them.

### Common Device Management Calls

| System Call | Purpose | Description |
|-------------|---------|-------------|
| `ioctl()` | Device control | Device-specific operations |
| `read()` | Read from device | Same as file read |
| `write()` | Write to device | Same as file write |
| `request()` | Request device | Get exclusive access |
| `release()` | Release device | Give up device access |

### Device as File (UNIX Philosophy)

```
In UNIX/Linux, devices are treated as files:

/dev/sda     ← Hard disk
/dev/tty     ← Terminal
/dev/null    ← Null device (discard output)
/dev/random  ← Random number generator
/dev/usb     ← USB devices

Operations: open(), read(), write(), close()
Same interface as regular files!
```

### ioctl Example (Conceptual)

```c
#include <sys/ioctl.h>

// Get terminal window size
struct winsize ws;
ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws);
printf("Terminal size: %d rows, %d cols\n", ws.ws_row, ws.ws_col);
```

---

## 4. Information Maintenance System Calls

These system calls transfer information between user program and operating system.

### Common Information Calls

| System Call | Purpose | Description |
|-------------|---------|-------------|
| `getpid()` | Get process ID | Returns current process ID |
| `getuid()` | Get user ID | Returns user identifier |
| `time()` | Get time | Current system time |
| `gettimeofday()` | Get precise time | Time with microseconds |
| `uname()` | System info | OS name, version, etc. |
| `sysinfo()` | System statistics | Memory, uptime, etc. |

### System Information Flow

```
┌─────────────────────────────────────────────────┐
│             INFORMATION SYSTEM CALLS             │
│                                                 │
│    getpid()  ──→  1234 (Process ID)            │
│    getuid()  ──→  1000 (User ID)               │
│    time()    ──→  1706620800 (Unix timestamp)  │
│    uname()   ──→  Linux 5.15.0 x86_64          │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Code Example

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/utsname.h>
#include <time.h>

int main() {
    // Get process and user IDs
    printf("Process ID: %d\n", getpid());
    printf("Parent Process ID: %d\n", getppid());
    printf("User ID: %d\n", getuid());
    
    // Get system information
    struct utsname sysinfo;
    uname(&sysinfo);
    printf("System: %s %s\n", sysinfo.sysname, sysinfo.release);
    
    // Get time
    time_t now = time(NULL);
    printf("Current time: %s", ctime(&now));
    
    return 0;
}
```

---

## 5. Communication System Calls

These system calls enable communication between processes.

### Two Models of IPC

```
┌─────────────────────────────────────────────────────────┐
│         MESSAGE PASSING MODEL                            │
│                                                         │
│   Process A          Kernel           Process B         │
│   ┌───────┐        ┌───────┐        ┌───────┐          │
│   │       │──send──│Mailbox│──recv──│       │          │
│   │       │        │       │        │       │          │
│   └───────┘        └───────┘        └───────┘          │
│                                                         │
├─────────────────────────────────────────────────────────┤
│         SHARED MEMORY MODEL                              │
│                                                         │
│   Process A                         Process B           │
│   ┌───────┐                        ┌───────┐           │
│   │       │────┐              ┌────│       │           │
│   │       │    │   Shared     │    │       │           │
│   └───────┘    └───│Memory│───┘    └───────┘           │
│                    └──────┘                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Common Communication Calls

| System Call | Purpose | Model |
|-------------|---------|-------|
| `pipe()` | Create pipe | Message Passing |
| `shmget()` | Get shared memory | Shared Memory |
| `shmat()` | Attach shared memory | Shared Memory |
| `msgget()` | Create message queue | Message Passing |
| `msgsnd()` | Send message | Message Passing |
| `msgrcv()` | Receive message | Message Passing |
| `socket()` | Create socket | Network Communication |

### Pipe Example (Conceptual)

```
Parent creates pipe:
                    ┌─────────────────┐
                    │      PIPE       │
                    │  ┌───────────┐  │
                    │  │ Read  End │  │ ← Child reads from here
                    │  │     ↑     │  │
                    │  │   Data    │  │
                    │  │     ↑     │  │
                    │  │Write End │  │ ← Parent writes here
                    │  └───────────┘  │
                    └─────────────────┘
```

### Code Example: Pipe Communication

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];  // pipefd[0] = read end, pipefd[1] = write end
    char message[] = "Hello from parent!";
    char buffer[100];
    
    // Create pipe
    if (pipe(pipefd) == -1) {
        perror("pipe");
        return 1;
    }
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process - read from pipe
        close(pipefd[1]);  // Close write end
        read(pipefd[0], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);
        close(pipefd[0]);
    } else {
        // Parent process - write to pipe
        close(pipefd[0]);  // Close read end
        write(pipefd[1], message, strlen(message) + 1);
        close(pipefd[1]);
        wait(NULL);
    }
    
    return 0;
}
```

---

## System Call Table

### Linux System Call Table (Partial)

| Number | Name | Description |
|--------|------|-------------|
| 0 | read | Read from file descriptor |
| 1 | write | Write to file descriptor |
| 2 | open | Open file |
| 3 | close | Close file descriptor |
| 9 | mmap | Map memory |
| 39 | getpid | Get process ID |
| 57 | fork | Create child process |
| 59 | execve | Execute program |
| 60 | exit | Terminate process |
| 61 | wait4 | Wait for process |

### System Call Invocation

```
x86_64 Linux System Call Convention:
┌────────────────────────────────────────┐
│  RAX ← System call number              │
│  RDI ← First argument                  │
│  RSI ← Second argument                 │
│  RDX ← Third argument                  │
│  R10 ← Fourth argument                 │
│  R8  ← Fifth argument                  │
│  R9  ← Sixth argument                  │
│                                        │
│  SYSCALL instruction triggers call     │
│                                        │
│  RAX ← Return value                    │
└────────────────────────────────────────┘
```

---

## System Calls vs Library Functions

### Key Differences

| Aspect | System Call | Library Function |
|--------|-------------|------------------|
| **Execution** | Kernel mode | User mode |
| **Overhead** | Higher (mode switch) | Lower |
| **Direct HW access** | Yes | No (uses system calls) |
| **Examples** | read(), write(), fork() | printf(), malloc(), strcpy() |

### Relationship

```
┌────────────────────────────────────────────────────┐
│  printf("Hello %s", name);   ← Library Function    │
│        │                                           │
│        ↓                                           │
│  write(1, buffer, len);      ← System Call        │
│        │                                           │
│        ↓                                           │
│  Kernel writes to terminal   ← Kernel Operation   │
└────────────────────────────────────────────────────┘

printf() is convenient but internally uses write() system call
```

---

## System Call Overhead

### Why System Calls are Expensive

```
Steps that add overhead:
1. Save user context
2. Switch to kernel mode
3. Validate parameters
4. Execute kernel code
5. Copy results to user space
6. Switch back to user mode
7. Restore user context

Each mode switch = 100s to 1000s of cycles
```

### Minimizing System Call Overhead

| Technique | Description |
|-----------|-------------|
| **Buffering** | Batch small I/Os into larger ones |
| **Memory Mapping** | Access files as memory (fewer calls) |
| **Batching** | Combine multiple operations |
| **Caching** | Avoid repeated system calls |

---

## Summary Table

| Category | Purpose | Examples |
|----------|---------|----------|
| Process Control | Create/manage processes | fork, exec, exit, wait |
| File Management | File operations | open, read, write, close |
| Device Management | I/O device control | ioctl, read, write |
| Information | System/process info | getpid, time, uname |
| Communication | IPC | pipe, shmget, socket |

---

## System Call Flow Diagram

```
┌──────────────────────────────────────────────────────────┐
│                    USER PROGRAM                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │  printf("Hello")                                  │    │
│  │       │                                          │    │
│  │       ↓                                          │    │
│  │  C Library (glibc)                               │    │
│  │  ┌─────────────────────────────────────────┐    │    │
│  │  │ Prepare arguments                        │    │    │
│  │  │ Load syscall number into register        │    │    │
│  │  │ Execute SYSCALL/INT 0x80                 │    │    │
│  │  └────────────────┬────────────────────────┘    │    │
│  └───────────────────┼──────────────────────────────┘    │
│                      │ Mode Switch                        │
├──────────────────────┼───────────────────────────────────┤
│  KERNEL              ↓                                    │
│  ┌──────────────────────────────────────────────────┐    │
│  │ System Call Handler                               │    │
│  │  │                                               │    │
│  │  ├─→ Validate syscall number                     │    │
│  │  ├─→ Look up in syscall table                    │    │
│  │  ├─→ Execute sys_write()                         │    │
│  │  ├─→ Write to terminal device                    │    │
│  │  └─→ Return result                               │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## Quick Revision Questions

1. What is the difference between a system call and a library function?
2. Why do we need to switch from user mode to kernel mode for system calls?
3. Name three system calls for file operations.
4. What is the purpose of fork() system call?
5. Explain the two models of inter-process communication.

---

[← Previous: OS Structure](./04-os-structure.md) | [Next: Process Concepts →](../02-Process-Management/01-process-concepts.md)
