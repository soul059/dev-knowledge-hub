# Chapter 2.6: Inter-Process Communication (IPC)

## Overview

Processes frequently need to communicate with each other. Inter-Process Communication (IPC) provides mechanisms that allow processes to exchange data and synchronize their actions.

---

## Why Do Processes Need to Communicate?

### Reasons for IPC

| Reason | Description | Example |
|--------|-------------|---------|
| **Information Sharing** | Multiple processes need same data | Shared database |
| **Computation Speedup** | Divide task among processes | Parallel processing |
| **Modularity** | Divide system into separate processes | Microservices |
| **Convenience** | Users work on many tasks at once | Copy-paste between apps |

### Process Types by Communication Need

```
┌─────────────────────────────────────────────────────────────────┐
│                     PROCESS TYPES                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Independent Processes:                                          │
│  • Cannot affect or be affected by other processes              │
│  • No data sharing                                              │
│  • Example: Two unrelated programs                              │
│                                                                 │
│  Cooperating Processes:                                          │
│  • Can affect or be affected by other processes                 │
│  • Share data or communicate                                    │
│  • Example: Producer-Consumer, Client-Server                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## IPC Models

There are two fundamental models for IPC:

```
┌────────────────────────────────────────────────────────────────────┐
│                         IPC MODELS                                  │
├───────────────────────────────┬────────────────────────────────────┤
│        SHARED MEMORY          │          MESSAGE PASSING           │
├───────────────────────────────┼────────────────────────────────────┤
│                               │                                    │
│   Process A      Process B    │    Process A        Process B     │
│   ┌───────┐      ┌───────┐   │    ┌───────┐        ┌───────┐     │
│   │       │      │       │   │    │       │        │       │     │
│   │   ────┼──────┼────   │   │    │   ────┼──send──┼────   │     │
│   │       │      │       │   │    │       │   M    │       │     │
│   └───────┘      └───────┘   │    │   ←───┼─receive┼────   │     │
│        ↓    ↓    ↓           │    │       │        │       │     │
│       ┌─────────────┐        │    └───────┘        └───────┘     │
│       │   Shared    │        │           ↓    ↓    ↓              │
│       │   Memory    │        │         ┌───────────────┐          │
│       │   Region    │        │         │    Kernel     │          │
│       └─────────────┘        │         │  (Message     │          │
│                               │         │   Queue)      │          │
│  Faster, but needs           │         └───────────────┘          │
│  synchronization             │                                    │
│                               │  Slower, but safer                │
│                               │  (kernel manages)                 │
└───────────────────────────────┴────────────────────────────────────┘
```

---

## 1. Shared Memory

### Concept

A region of memory is shared between processes. Processes can exchange information by reading and writing to this shared region.

### How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                      SHARED MEMORY MODEL                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Process A's Address Space        Process B's Address Space      │
│  ┌─────────────────────┐         ┌─────────────────────┐        │
│  │       Stack         │         │       Stack         │        │
│  ├─────────────────────┤         ├─────────────────────┤        │
│  │       Heap          │         │       Heap          │        │
│  ├─────────────────────┤         ├─────────────────────┤        │
│  │  Shared Memory ─────┼─────────┼──→Shared Memory     │        │
│  │  Region             │         │   Region            │        │
│  ├─────────────────────┤         ├─────────────────────┤        │
│  │       Data          │         │       Data          │        │
│  ├─────────────────────┤         ├─────────────────────┤        │
│  │       Code          │         │       Code          │        │
│  └─────────────────────┘         └─────────────────────┘        │
│                                                                  │
│           Both processes see SAME physical memory                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Steps to Use Shared Memory

```
1. Create shared memory segment
   shmget() → returns segment ID

2. Attach segment to address space
   shmat() → returns pointer to shared memory

3. Use shared memory (read/write)
   Direct memory access via pointer

4. Detach when done
   shmdt() → detach from address space

5. Destroy segment (when no longer needed)
   shmctl() → remove segment
```

### Code Example (POSIX)

```c
// Process A - Writer
#include <sys/shm.h>
#include <stdio.h>
#include <string.h>

int main() {
    // Create shared memory segment
    int shmid = shmget(1234, 1024, IPC_CREAT | 0666);
    
    // Attach to our address space
    char *str = (char*) shmat(shmid, NULL, 0);
    
    // Write to shared memory
    strcpy(str, "Hello from Process A!");
    
    printf("Data written: %s\n", str);
    
    // Detach
    shmdt(str);
    
    return 0;
}

// Process B - Reader
int main() {
    // Get existing shared memory segment
    int shmid = shmget(1234, 1024, 0666);
    
    // Attach to our address space
    char *str = (char*) shmat(shmid, NULL, 0);
    
    // Read from shared memory
    printf("Data read: %s\n", str);
    
    // Detach
    shmdt(str);
    
    // Destroy segment (when done)
    shmctl(shmid, IPC_RMID, NULL);
    
    return 0;
}
```

### Producer-Consumer Problem (Shared Memory)

A classic IPC problem where producer creates data and consumer uses it.

```
┌──────────────────────────────────────────────────────────────────┐
│                   PRODUCER-CONSUMER PROBLEM                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   PRODUCER                        CONSUMER                       │
│   ┌────────┐                     ┌────────┐                     │
│   │ Create │                     │  Use   │                     │
│   │  Data  │                     │  Data  │                     │
│   └───┬────┘                     └────┬───┘                     │
│       │                               ↑                         │
│       ↓                               │                         │
│   ┌───┴───────────────────────────────┴───┐                     │
│   │            SHARED BUFFER              │                     │
│   │   ┌───┬───┬───┬───┬───┬───┬───┬───┐  │                     │
│   │   │ D │ D │ D │   │   │   │   │   │  │                     │
│   │   └───┴───┴───┴───┴───┴───┴───┴───┘  │                     │
│   │     ↑                       ↑         │                     │
│   │    out                      in        │                     │
│   └───────────────────────────────────────┘                     │
│                                                                  │
│   Challenges:                                                    │
│   • Don't produce when buffer full                              │
│   • Don't consume when buffer empty                             │
│   • Synchronize access to buffer                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Bounded Buffer Implementation

```c
#define BUFFER_SIZE 10

typedef struct {
    int buffer[BUFFER_SIZE];
    int in;   // Next free position
    int out;  // First full position
    int count; // Number of items
} SharedBuffer;

// Producer
void produce(SharedBuffer *sb, int item) {
    while (sb->count == BUFFER_SIZE)
        ; // Wait while buffer is full
    
    sb->buffer[sb->in] = item;
    sb->in = (sb->in + 1) % BUFFER_SIZE;
    sb->count++;
}

// Consumer
int consume(SharedBuffer *sb) {
    while (sb->count == 0)
        ; // Wait while buffer is empty
    
    int item = sb->buffer[sb->out];
    sb->out = (sb->out + 1) % BUFFER_SIZE;
    sb->count--;
    
    return item;
}
```

### Advantages and Disadvantages

| Advantages | Disadvantages |
|------------|---------------|
| Very fast (no kernel involvement after setup) | Requires synchronization |
| Efficient for large data | Complex to program correctly |
| Direct memory access | Security concerns |

---

## 2. Message Passing

### Concept

Processes communicate by explicitly sending and receiving messages. The kernel manages the message transfer.

### Basic Operations

```
┌──────────────────────────────────────────────────────────────────┐
│                    MESSAGE PASSING PRIMITIVES                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  send(destination, message)                                      │
│    → Send message to destination process                         │
│                                                                  │
│  receive(source, message)                                        │
│    → Receive message from source process                         │
│                                                                  │
│  If processes P and Q want to communicate:                       │
│  1. Establish a communication link                               │
│  2. Exchange messages via send/receive                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Communication Link Types

#### Direct vs Indirect Communication

```
DIRECT COMMUNICATION:
┌─────────┐                      ┌─────────┐
│Process P│ ─────────────────────→ │Process Q│
│         │  send(Q, message)    │         │
│         │ ←─────────────────────│         │
│         │  receive(P, message) │         │
└─────────┘                      └─────────┘
  • Processes must name each other explicitly
  • Link between exactly two processes
  • Exactly one link per pair

INDIRECT COMMUNICATION (via Mailbox/Port):
┌─────────┐      ┌─────────┐      ┌─────────┐
│Process P│──send(A,msg)──│Mailbox│←receive(A,msg)─│Process Q│
│         │               │   A   │               │         │
└─────────┘               └───────┘               └─────────┘
  • Messages sent to and received from mailbox
  • Link only if processes share mailbox
  • Multiple processes can share mailbox
```

#### Synchronous vs Asynchronous

```
┌─────────────────────────────────────────────────────────────────┐
│                   SYNCHRONIZATION OPTIONS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SYNCHRONOUS (Blocking):                                        │
│  ├── Blocking Send: Sender waits until message received        │
│  └── Blocking Receive: Receiver waits until message arrives    │
│                                                                 │
│  P: send(msg) ──────→ │ Blocks until Q receives               │
│                       │                                        │
│  Q:                   receive(msg) ← Gets message, both proceed│
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ASYNCHRONOUS (Non-Blocking):                                   │
│  ├── Non-blocking Send: Sender sends and continues             │
│  └── Non-blocking Receive: Returns message or null             │
│                                                                 │
│  P: send(msg) → Continue immediately (message queued)          │
│  Q: receive(msg) → Returns NULL if no message available        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Buffering

Where do messages wait before being received?

```
┌─────────────────────────────────────────────────────────────────┐
│                      MESSAGE BUFFERING                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Zero Capacity (No Buffering):                                  │
│  ┌─────────────────┐                                            │
│  │    No Queue     │  Sender must wait for receiver            │
│  │   (Rendezvous)  │  Both must be ready simultaneously        │
│  └─────────────────┘                                            │
│                                                                 │
│  Bounded Capacity:                                              │
│  ┌───┬───┬───┬───┬───┐                                         │
│  │msg│msg│msg│   │   │  Queue holds n messages                 │
│  └───┴───┴───┴───┴───┘  Sender waits if queue full             │
│     n = buffer size                                             │
│                                                                 │
│  Unbounded Capacity:                                            │
│  ┌───┬───┬───┬───┬───┬─ ─ ─ ─┐                                 │
│  │msg│msg│msg│msg│msg│  ...  │  Unlimited queue size           │
│  └───┴───┴───┴───┴───┴─ ─ ─ ─┘  Sender never waits             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## IPC Mechanisms in UNIX/Linux

### 1. Pipes

Unidirectional communication channel between related processes.

```
┌─────────────────────────────────────────────────────────────────┐
│                         PIPE                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Parent creates pipe, then forks                                │
│                                                                 │
│  ┌────────────┐              ┌────────────┐                    │
│  │   Parent   │              │   Child    │                    │
│  │            │              │            │                    │
│  │  write()  ─┼──────────────┼──→ read()  │                    │
│  │    fd[1]   │    PIPE      │    fd[0]   │                    │
│  │            │              │            │                    │
│  └────────────┘              └────────────┘                    │
│                                                                 │
│  Characteristics:                                               │
│  • Unidirectional (one-way)                                    │
│  • Only between related processes (parent-child)               │
│  • Data is FIFO                                                │
│  • Limited capacity (typically 64KB)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Pipe Example:**

```c
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main() {
    int fd[2];  // fd[0] = read end, fd[1] = write end
    char buffer[100];
    
    // Create pipe
    pipe(fd);
    
    if (fork() == 0) {
        // Child process - reader
        close(fd[1]);  // Close unused write end
        read(fd[0], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);
        close(fd[0]);
    } else {
        // Parent process - writer
        close(fd[0]);  // Close unused read end
        char *msg = "Hello from parent!";
        write(fd[1], msg, strlen(msg) + 1);
        close(fd[1]);
        wait(NULL);
    }
    
    return 0;
}
```

**Shell Pipes:**
```bash
$ ls -la | grep ".txt" | wc -l
#        ↑              ↑
#      Pipe 1        Pipe 2
# Output of ls goes to input of grep
# Output of grep goes to input of wc
```

### 2. Named Pipes (FIFOs)

Pipes that exist as special files in the filesystem.

```
┌─────────────────────────────────────────────────────────────────┐
│                        NAMED PIPE (FIFO)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Process A                          Process B                   │
│  ┌─────────┐                       ┌─────────┐                 │
│  │         │                       │         │                 │
│  │ write() ├───────────────────────┤ read()  │                 │
│  │         │                       │         │                 │
│  └─────────┘                       └─────────┘                 │
│       ↓                                 ↑                       │
│       └──────────┐          ┌──────────┘                       │
│                  ↓          ↓                                   │
│              ┌──────────────────┐                               │
│              │   /tmp/myfifo    │  ← Named pipe in filesystem  │
│              └──────────────────┘                               │
│                                                                 │
│  Advantages over regular pipes:                                 │
│  • Unrelated processes can communicate                         │
│  • Persists in filesystem                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Named Pipe Example:**

```bash
# Terminal 1 - Create FIFO and read
$ mkfifo /tmp/myfifo
$ cat < /tmp/myfifo    # Blocks waiting for data

# Terminal 2 - Write to FIFO
$ echo "Hello via FIFO" > /tmp/myfifo
```

```c
// Writer process
#include <fcntl.h>
#include <unistd.h>

int main() {
    mkfifo("/tmp/myfifo", 0666);
    int fd = open("/tmp/myfifo", O_WRONLY);
    write(fd, "Hello!", 7);
    close(fd);
    return 0;
}

// Reader process
int main() {
    char buffer[100];
    int fd = open("/tmp/myfifo", O_RDONLY);
    read(fd, buffer, sizeof(buffer));
    printf("Received: %s\n", buffer);
    close(fd);
    return 0;
}
```

### 3. Message Queues

Kernel-maintained queue of messages.

```
┌─────────────────────────────────────────────────────────────────┐
│                       MESSAGE QUEUE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Process A         Kernel Message Queue          Process B      │
│  ┌─────────┐      ┌───┬───┬───┬───┬───┐       ┌─────────┐     │
│  │         │──────│msg│msg│msg│msg│   │───────│         │     │
│  │ msgsnd()│      │ 1 │ 2 │ 3 │ 4 │   │       │ msgrcv()│     │
│  │         │      └───┴───┴───┴───┴───┘       │         │     │
│  └─────────┘            Message Queue          └─────────┘     │
│                                                                 │
│  Features:                                                      │
│  • Messages have types (can select by type)                    │
│  • Asynchronous communication                                  │
│  • Persists until explicitly removed                           │
│  • Multiple readers and writers possible                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Message Queue Example:**

```c
#include <sys/msg.h>
#include <stdio.h>
#include <string.h>

struct message {
    long msg_type;
    char msg_text[100];
};

// Sender
int main() {
    int msgid = msgget(1234, IPC_CREAT | 0666);
    
    struct message msg;
    msg.msg_type = 1;
    strcpy(msg.msg_text, "Hello via message queue!");
    
    msgsnd(msgid, &msg, sizeof(msg.msg_text), 0);
    printf("Sent: %s\n", msg.msg_text);
    
    return 0;
}

// Receiver
int main() {
    int msgid = msgget(1234, 0666);
    
    struct message msg;
    msgrcv(msgid, &msg, sizeof(msg.msg_text), 1, 0);
    
    printf("Received: %s\n", msg.msg_text);
    
    msgctl(msgid, IPC_RMID, NULL);  // Remove queue
    return 0;
}
```

### 4. Signals

Asynchronous notification mechanism.

```
┌─────────────────────────────────────────────────────────────────┐
│                         SIGNALS                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Signal: A software interrupt delivered to a process           │
│                                                                 │
│  ┌─────────┐      Signal Event      ┌─────────────────────────┐│
│  │ Process │ ←────────────────────── │ Kernel / Other Process ││
│  │         │      SIGINT, SIGKILL    │                        ││
│  └─────────┘                         └─────────────────────────┘│
│       │                                                         │
│       ↓                                                         │
│  Signal Handler                                                 │
│  • Default action (terminate, ignore, stop)                    │
│  • User-defined handler                                        │
│                                                                 │
│  Common Signals:                                                │
│  ┌──────────┬─────────────────────────────────────────────────┐│
│  │ SIGINT   │ Ctrl+C - Interrupt                              ││
│  │ SIGTERM  │ Termination request                             ││
│  │ SIGKILL  │ Forceful termination (cannot be caught)         ││
│  │ SIGCHLD  │ Child process terminated                        ││
│  │ SIGUSR1  │ User-defined signal 1                           ││
│  │ SIGALRM  │ Timer expired                                   ││
│  └──────────┴─────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Signal Example:**

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void handler(int signum) {
    printf("Caught signal %d\n", signum);
}

int main() {
    // Register signal handler
    signal(SIGINT, handler);  // Handle Ctrl+C
    
    printf("Press Ctrl+C (PID: %d)\n", getpid());
    
    while(1) {
        sleep(1);
        printf("Running...\n");
    }
    
    return 0;
}
```

### 5. Sockets

Communication endpoints for network (or local) communication.

```
┌─────────────────────────────────────────────────────────────────┐
│                         SOCKETS                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Machine A                              Machine B               │
│  (or same machine)                      (or same machine)       │
│  ┌─────────────────┐                   ┌─────────────────┐     │
│  │     Client      │                   │     Server      │     │
│  │                 │                   │                 │     │
│  │  ┌───────────┐  │      Network      │  ┌───────────┐  │     │
│  │  │  Socket   │◄─┼───────────────────┼─►│  Socket   │  │     │
│  │  │ (Port X)  │  │   TCP/IP/UDP      │  │ (Port 80) │  │     │
│  │  └───────────┘  │                   │  └───────────┘  │     │
│  │                 │                   │                 │     │
│  └─────────────────┘                   └─────────────────┘     │
│                                                                 │
│  Types:                                                         │
│  • Stream sockets (TCP) - Reliable, connection-oriented        │
│  • Datagram sockets (UDP) - Unreliable, connectionless         │
│  • Unix domain sockets - Local IPC (faster than network)       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## IPC Mechanism Comparison

| Mechanism | Communication | Related Processes | Persistence | Use Case |
|-----------|---------------|-------------------|-------------|----------|
| **Pipe** | Unidirectional | Parent-Child | Process lifetime | Simple data flow |
| **Named Pipe** | Unidirectional | Any | Filesystem | Simple IPC, unrelated |
| **Shared Memory** | Bidirectional | Any | Explicit removal | Large data, fast |
| **Message Queue** | Bidirectional | Any | Explicit removal | Structured messages |
| **Signals** | Notification | Any | None | Events, interrupts |
| **Sockets** | Bidirectional | Any (network) | Connection | Network, client-server |

---

## Client-Server Communication

### Common Architectures

```
┌──────────────────────────────────────────────────────────────────┐
│                    CLIENT-SERVER MODELS                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Sockets                                                      │
│     ┌────────┐         Network         ┌────────┐               │
│     │ Client │ ◄─────────────────────► │ Server │               │
│     └────────┘    TCP/UDP Sockets      └────────┘               │
│                                                                  │
│  2. Remote Procedure Calls (RPC)                                 │
│     ┌────────┐    Stub │ Network │ Stub    ┌────────┐           │
│     │ Client │──→│    │         │    │←───│ Server │           │
│     │ call() │   │    ├─────────┤    │    │ func() │           │
│     └────────┘   │    │ Request │    │    └────────┘           │
│                  │    │ Response│    │                          │
│     Looks like local call, actually remote                      │
│                                                                  │
│  3. Pipes (Local)                                               │
│     ┌────────┐       Pipe         ┌────────┐                    │
│     │ Client │ ─────────────────→ │ Server │                    │
│     └────────┘                    └────────┘                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **IPC** | Mechanisms for process communication |
| **Shared Memory** | Direct memory sharing, fast but needs sync |
| **Message Passing** | Kernel-managed message exchange |
| **Pipes** | Unidirectional, parent-child communication |
| **Named Pipes** | Pipes with filesystem presence |
| **Message Queues** | Typed messages in kernel queue |
| **Signals** | Asynchronous event notification |
| **Sockets** | Network or local communication endpoints |

### When to Use What

| Situation | Recommended IPC |
|-----------|-----------------|
| Large data, same machine | Shared Memory |
| Parent-child, simple data | Pipes |
| Unrelated processes, local | Named Pipes, Message Queues |
| Network communication | Sockets |
| Event notification | Signals |
| Structured messages | Message Queues |

---

## Quick Revision Questions

1. What are the two main IPC models?
2. Explain the difference between pipes and named pipes.
3. What are the advantages of shared memory over message passing?
4. When would you use signals for IPC?
5. Describe the producer-consumer problem in the context of IPC.

---

[← Previous: Threads](./05-threads.md) | [Next: Synchronization Concepts →](../03-Process-Synchronization/01-synchronization-concepts.md)
