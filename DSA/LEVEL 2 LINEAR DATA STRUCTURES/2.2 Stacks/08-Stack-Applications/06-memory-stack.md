# Chapter 6: Memory Stack & Stack in System Architecture

[← Previous: Backtracking](05-backtracking.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

The **memory stack** is one of the most fundamental concepts in computer architecture. Every running program uses a stack segment in memory for function calls, local variables, and control flow. Understanding how the hardware stack works connects the abstract stack data structure to real system behavior.

---

## Memory Layout of a Process

```
┌──────────────────────────────────────────────────────────┐
│                PROCESS MEMORY LAYOUT                     │
│                                                          │
│  High Address (e.g., 0xFFFF...)                          │
│  ┌────────────────────────────┐                         │
│  │    COMMAND LINE ARGS       │                         │
│  │    ENVIRONMENT VARS        │                         │
│  ├────────────────────────────┤                         │
│  │                            │                         │
│  │         STACK              │  ← Grows DOWNWARD ↓     │
│  │    (function frames)       │                         │
│  │                            │                         │
│  │          ↓ SP              │  ← Stack Pointer        │
│  │                            │                         │
│  │     ┈┈┈ free ┈┈┈           │                         │
│  │                            │                         │
│  │          ↑ brk             │  ← Program Break        │
│  │                            │                         │
│  │         HEAP               │  ← Grows UPWARD ↑       │
│  │    (dynamic allocation)    │                         │
│  │                            │                         │
│  ├────────────────────────────┤                         │
│  │    BSS (uninitialized)     │                         │
│  ├────────────────────────────┤                         │
│  │    DATA (initialized)      │                         │
│  ├────────────────────────────┤                         │
│  │    TEXT (code/instructions) │                         │
│  └────────────────────────────┘                         │
│  Low Address (e.g., 0x0000...)                           │
└──────────────────────────────────────────────────────────┘
```

---

## Anatomy of a Stack Frame

```
When function foo(int a, int b) is called:

  High Address
  ┌─────────────────────────┐
  │ Argument b              │  ← Pushed by caller
  │ Argument a              │
  ├─────────────────────────┤
  │ Return Address          │  ← Pushed by CALL instruction
  ├─────────────────────────┤  ← Frame Pointer (EBP/RBP)
  │ Saved Frame Pointer     │  ← Old EBP
  ├─────────────────────────┤
  │ Local variable 1        │  
  │ Local variable 2        │
  │ ...                     │
  ├─────────────────────────┤
  │ Saved registers         │
  ├─────────────────────────┤  ← Stack Pointer (ESP/RSP)
  │ (space for called fns)  │
  └─────────────────────────┘
  Low Address

  Two key registers:
  SP (Stack Pointer): Points to TOP of stack
  FP (Frame Pointer): Points to BASE of current frame
  
  FP stays fixed during function → stable reference for locals
  SP moves as data is pushed/popped
```

---

## Stack Operations at Hardware Level

```
PUSH operation (e.g., PUSH EAX):
  1. SP ← SP - 4          (decrement by word size, stack grows DOWN)
  2. Memory[SP] ← EAX     (store value at new top)

POP operation (e.g., POP EBX):  
  1. EBX ← Memory[SP]     (read value from top)
  2. SP ← SP + 4          (increment, "shrinking" the stack)

CALL function_address:
  1. PUSH return_address   (save where to come back)
  2. JMP function_address  (transfer control)

RET (return):
  1. POP return_address    (retrieve saved address)
  2. JMP return_address    (go back to caller)
```

---

## Example: Assembly-Level Trace

```
C Code:
  int add(int x, int y) {
      int sum = x + y;
      return sum;
  }
  
  int main() {
      int result = add(3, 5);
  }

Assembly-like execution:

=== main() is called ===
  SP: 1000 (high address)
  
  PUSH 5          ; push argument y      SP: 996  [5]
  PUSH 3          ; push argument x      SP: 992  [3, 5]
  CALL add        ; push return addr     SP: 988  [ret, 3, 5]

=== enter add() ===
  PUSH EBP        ; save old frame ptr   SP: 984  [oldFP, ret, 3, 5]
  MOV EBP, ESP    ; set new frame ptr    FP: 984
  SUB ESP, 4      ; space for 'sum'      SP: 980

  Stack:
  Addr  │ Value
  ──────┼────────────
  1000  │ ...
  996   │ 5 (arg y)
  992   │ 3 (arg x)
  988   │ return addr
  984   │ old FP        ← FP points here
  980   │ sum (local)   ← SP points here

  ; Execute: sum = x + y
  MOV EAX, [FP+8]   ; EAX = x = 3
  ADD EAX, [FP+12]  ; EAX = 3 + 5 = 8
  MOV [FP-4], EAX   ; sum = 8

=== return from add() ===
  MOV EAX, [FP-4]  ; return value in EAX = 8
  MOV ESP, EBP     ; restore SP           SP: 984
  POP EBP          ; restore old FP       SP: 988
  RET              ; pop return addr, jump SP: 992
  
  ; Caller cleans up arguments
  ADD ESP, 8       ; remove 2 args         SP: 1000

  result = EAX = 8
```

---

## Stack vs Heap

```
┌──────────────────┬────────────────────┬──────────────────┐
│ Aspect           │ Stack              │ Heap             │
├──────────────────┼────────────────────┼──────────────────┤
│ Allocation       │ Automatic          │ Manual/GC        │
│ Speed            │ Very fast (SP adj) │ Slower (search)  │
│ Size             │ Limited (1-8 MB)   │ Large (GBs)      │
│ Growth           │ Downward           │ Upward           │
│ Access pattern   │ LIFO only          │ Random access    │
│ Fragmentation    │ None               │ Can fragment     │
│ Lifetime         │ Until fn returns   │ Until freed/GC'd │
│ Thread safety    │ Per-thread         │ Shared (needs    │
│                  │ (no sharing)       │  synchronization)│
│ Stores           │ Locals, params,    │ Dynamic objects, │
│                  │ return addresses   │ arrays, trees    │
│ Overflow         │ Stack overflow     │ Out of memory    │
└──────────────────┴────────────────────┴──────────────────┘
```

---

## Stack in Multi-Threading

```
┌──────────────────────────────────────────────────────────┐
│  Each THREAD has its own stack!                          │
│                                                          │
│  Thread 1          Thread 2          Thread 3            │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐        │
│  │ Stack 1  │     │ Stack 2  │     │ Stack 3  │        │
│  │          │     │          │     │          │        │
│  │ foo()    │     │ bar()    │     │ baz()    │        │
│  │ main()   │     │ worker() │     │ handler()│        │
│  └──────────┘     └──────────┘     └──────────┘        │
│                                                          │
│         ┌──────────────────────────┐                    │
│         │   SHARED HEAP MEMORY    │                    │
│         │   (all threads access)  │                    │
│         └──────────────────────────┘                    │
│                                                          │
│  Stacks are PRIVATE → no race conditions on locals      │
│  Heap is SHARED → needs locks/synchronization           │
└──────────────────────────────────────────────────────────┘
```

---

## Security: Stack-Based Attacks

```
BUFFER OVERFLOW ATTACK:

  Normal stack frame:          Exploited:
  ┌───────────────┐           ┌───────────────┐
  │ return addr   │           │ MALICIOUS addr │ ← overwritten!
  ├───────────────┤           ├───────────────┤
  │ saved EBP     │           │ AAAAAAAAAAAAA  │ ← overwritten!
  ├───────────────┤           ├───────────────┤
  │ buffer[20]    │           │ AAAAAAA...AAAA │ ← overflow!
  │ char buf[8]   │           │ AAAAAAAAAAAAA  │
  └───────────────┘           └───────────────┘

  If buffer overflows, it can overwrite the return 
  address, redirecting execution to attacker's code!

  Defenses:
  1. Stack canaries - random values between locals and return addr
  2. ASLR - randomize stack addresses
  3. DEP/NX bit - mark stack as non-executable
  4. Safe functions - bounds-checked string operations
```

---

## Stacks in Different Architectures

```
┌───────────────────┬──────────────────────────────────┐
│ Architecture      │ Stack Details                    │
├───────────────────┼──────────────────────────────────┤
│ x86 (32-bit)      │ ESP/EBP, grows down, 4B words   │
│ x86-64 (64-bit)   │ RSP/RBP, grows down, 8B words   │
│ ARM               │ SP (R13), grows down             │
│ JVM               │ Operand stack per method frame   │
│ Python            │ Frame objects in linked list      │
│ WebAssembly       │ Structured stack machine          │
└───────────────────┴──────────────────────────────────┘
```

---

## Virtual Machine Stacks (JVM Example)

```
The JVM uses an OPERAND STACK for expression evaluation:

  Java: int c = a + b * 2;
  
  Bytecode:
    ILOAD a         ; push a          stack: [a]
    ILOAD b         ; push b          stack: [a, b]
    ICONST_2        ; push 2          stack: [a, b, 2]
    IMUL            ; pop 2,b; push b*2  stack: [a, b*2]
    IADD            ; pop b*2,a; push a+b*2  stack: [a+b*2]
    ISTORE c        ; pop into c      stack: []

  This is essentially postfix evaluation!
  a b 2 * + → a + (b * 2)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Memory Stack** | Region of process memory for function calls |
| **Grows** | Downward (high → low address) |
| **Contains** | Return addresses, parameters, locals, saved registers |
| **Registers** | SP (Stack Pointer), FP (Frame Pointer) |
| **Per-thread** | Each thread has its own stack |
| **Size limit** | Typically 1-8 MB |
| **Security risk** | Buffer overflow can overwrite return addresses |
| **JVM** | Operand stack for expression evaluation (postfix) |

---

## Quick Revision Questions

1. **Why does the stack grow downward in most architectures?**
   > Historical convention. The stack starts at high addresses and grows toward lower addresses, while the heap grows from low addresses upward — maximizing the gap between them.

2. **What are the two key registers for stack management?**
   > Stack Pointer (SP/ESP/RSP) points to the current top. Frame Pointer (FP/EBP/RBP) points to the base of the current frame for stable local variable access.

3. **Why does each thread need its own stack?**
   > Each thread executes its own function calls independently. Sharing a stack would corrupt return addresses and local variables.

4. **What is a stack canary?**
   > A random value placed between local variables and the return address. If a buffer overflow corrupts it, the program detects the attack before returning.

5. **How does the JVM's operand stack relate to postfix notation?**
   > JVM bytecode evaluates expressions using a stack, which is essentially postfix evaluation — operands are pushed, operators pop operands and push results.

---

[← Previous: Backtracking](05-backtracking.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)
