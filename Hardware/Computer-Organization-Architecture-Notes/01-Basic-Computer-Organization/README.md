# Unit 1: Basic Computer Organization

## 📋 Unit Overview

This unit introduces the fundamental concepts of how computers are organized at the hardware level. Understanding computer organization is essential for comprehending how software instructions are executed by hardware.

---

## 🎯 Learning Objectives

After completing this unit, you will be able to:
- Distinguish between Von Neumann and Harvard architectures
- Explain the role of CPU, memory, and I/O subsystems
- Describe bus structures and their functions
- Understand system interconnection methods

---

## 📑 Chapters in This Unit

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 1.1 | [Von Neumann vs Harvard Architecture](01-von-neumann-harvard-architecture.md) | Stored program concept, separate vs unified memory |
| 1.2 | [CPU, Memory, I/O Organization](02-cpu-memory-io-organization.md) | Core components and their functions |
| 1.3 | [Bus Structures](03-bus-structures.md) | Address, data, control buses |
| 1.4 | [System Interconnection](04-system-interconnection.md) | Bus topologies and standards |

---

## 🔑 Key Concepts Preview

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPUTER ORGANIZATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌───────────┐     ┌───────────┐     ┌───────────────────┐    │
│   │    CPU    │◄───►│   Memory  │◄───►│  I/O Subsystem    │    │
│   │           │     │           │     │                   │    │
│   │ • Control │     │ • RAM     │     │ • Interfaces      │    │
│   │ • ALU     │     │ • ROM     │     │ • Controllers     │    │
│   │ • Regs    │     │ • Cache   │     │ • Peripherals     │    │
│   └───────────┘     └───────────┘     └───────────────────┘    │
│         │                 │                    │                │
│         └─────────────────┴────────────────────┘                │
│                           │                                      │
│                     System Bus                                   │
│              (Address + Data + Control)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Unit Summary Table

| Topic | Von Neumann | Harvard |
|-------|-------------|---------|
| Memory | Single unified | Separate instruction/data |
| Bus | Single | Dual |
| Speed | Limited by bus | Faster parallel access |
| Cost | Lower | Higher |
| Use Case | General purpose | DSP, microcontrollers |

---

## 🧭 Navigation

| Previous | Home | Next |
|----------|------|------|
| - | [Course Home](../README.md) | [Unit 2: Data Representation](../02-Data-Representation/README.md) |

---

*Start learning: [Chapter 1.1 - Von Neumann vs Harvard Architecture](01-von-neumann-harvard-architecture.md)*
