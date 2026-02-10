# Chapter 1.3: Network Topologies

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

Network topology refers to the arrangement of nodes and connections in a network. This chapter explores various physical and logical topologies, their characteristics, advantages, and disadvantages. Choosing the right topology is crucial for network performance, reliability, and cost.

---

## 🎯 Learning Objectives

- Understand the difference between physical and logical topology
- Identify and describe various network topologies
- Analyze advantages and disadvantages of each topology
- Choose appropriate topology for specific scenarios

---

## 1. What is Network Topology?

### Definition

**Network topology** defines the structure of the network, including how nodes are arranged and connected. It determines how data flows between network nodes.

### Types of Topology

```
┌─────────────────────────────────────────────────────────────┐
│                    NETWORK TOPOLOGY                          │
├────────────────────────┬────────────────────────────────────┤
│   PHYSICAL TOPOLOGY    │      LOGICAL TOPOLOGY              │
│                        │                                     │
│  • Actual layout of    │  • How data flows in              │
│    cables and devices  │    the network                     │
│  • Physical arrangement│  • May differ from                 │
│    we can see          │    physical layout                 │
│                        │                                     │
│  Example: Star cabling │  Example: Ring data flow          │
│  with ring protocol    │  over star physical layout        │
└────────────────────────┴────────────────────────────────────┘
```

### Real-World Analogy

Think of road networks in a city:
- **Physical Topology** = The actual road layout
- **Logical Topology** = The traffic flow patterns and rules

---

## 2. Bus Topology

### Structure

All devices connected to a single central cable (the "bus" or "backbone").

```
┌─────────────────────────────────────────────────────────────────────┐
│                          BUS TOPOLOGY                               │
│                                                                     │
│    Terminator                                         Terminator    │
│       ║                                                   ║         │
│       ╠═══════════════════════════════════════════════════╣         │
│       ║           ║           ║           ║               ║         │
│       ║       ┌───┴───┐   ┌───┴───┐   ┌───┴───┐          ║         │
│              │  PC 1  │   │  PC 2  │   │  PC 3  │                   │
│              └────────┘   └────────┘   └────────┘                   │
│                                                                     │
│                    CENTRAL CABLE (BUS/BACKBONE)                     │
└─────────────────────────────────────────────────────────────────────┘
```

### How It Works

```
PC 1 sends data to PC 3:

    ║═══════════════════════════════════════════════════════║
    ║     ║           ║           ║           ║             ║
        ┌─┴─┐     ┌───┴───┐   ┌───┴───┐   ┌───┴───┐
        │PC1│ ──► │ Data  │   │ Data  │   │ Data  │
        │SRC│     │Travels│   │Passes │   │ PC3   │ ◄── Accepts
        └───┘     │Through│   │ By    │   │ DEST  │     (destination)
                  └───────┘   └───────┘   └───────┘
                     ▲           ▲
                     │           │
                  Ignores     Ignores
               (not for me) (not for me)
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Cable** | Single coaxial cable |
| **Terminators** | Required at both ends to prevent signal reflection |
| **Signal** | Broadcast to all nodes |
| **Addressing** | Each node has unique address |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Easy to install | Single point of failure (cable) |
| Low cost (less cable) | Limited cable length |
| Simple architecture | Performance degrades with more nodes |
| Works well for small networks | Difficult to troubleshoot |
| No specialized hardware needed | Collisions can occur |

### Use Cases
- Small temporary networks
- Laboratory setups
- Legacy Ethernet (10BASE2, 10BASE5)

---

## 3. Star Topology

### Structure

All devices connected to a central device (hub or switch).

```
┌─────────────────────────────────────────────────────────────────────┐
│                          STAR TOPOLOGY                              │
│                                                                     │
│              ┌────────┐                    ┌────────┐               │
│              │  PC 1  │                    │  PC 2  │               │
│              └───┬────┘                    └────┬───┘               │
│                  │                              │                   │
│                  │      ┌──────────────┐        │                   │
│                  └──────┤    SWITCH    ├────────┘                   │
│                         │   (Central)  │                            │
│                  ┌──────┤              ├────────┐                   │
│                  │      └──────────────┘        │                   │
│                  │                              │                   │
│              ┌───┴────┐                    ┌────┴───┐               │
│              │  PC 3  │                    │  PC 4  │               │
│              └────────┘                    └────────┘               │
│                                                                     │
│                     All traffic through center                      │
└─────────────────────────────────────────────────────────────────────┘
```

### How It Works

```
PC 1 sends data to PC 3:

              ┌────────┐                    ┌────────┐
              │  PC 1  │                    │  PC 2  │
              │  SRC   │                    │(Ignores)
              └───┬────┘                    └────────┘
                  │ ①
                  │ Send to Switch
                  ▼
            ┌──────────────┐
            │    SWITCH    │
            │  ② Routes to │
            │  destination │
            └──────┬───────┘
                   │ ③
                   │ Deliver
                   ▼
              ┌────────┐
              │  PC 3  │
              │  DEST  │
              └────────┘
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Central Device** | Hub or Switch (switch preferred) |
| **Connections** | Point-to-point from each node to center |
| **Data Flow** | Through central device |
| **Scalability** | Easy to add new nodes |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Easy to install and modify | Central device is single point of failure |
| One cable failure affects only one node | More cabling required than bus |
| Easy to troubleshoot and isolate problems | Cost of central device (switch) |
| Better performance (with switch) | Limited by switch port count |
| Centralized management | Cable runs can be long |

### Hub vs Switch in Star Topology

```
WITH HUB (Broadcast):                 WITH SWITCH (Intelligent):
┌────┐ ┌────┐ ┌────┐ ┌────┐          ┌────┐ ┌────┐ ┌────┐ ┌────┐
│PC1 │ │PC2 │ │PC3 │ │PC4 │          │PC1 │ │PC2 │ │PC3 │ │PC4 │
└─┬──┘ └─┬──┘ └─┬──┘ └─┬──┘          └─┬──┘ └─┬──┘ └─┬──┘ └─┬──┘
  │  ┌───┴─────┴───┐   │                │      │      │      │
  └──┤  HUB sends  ├───┘                └──────┼──────┼──────┘
     │  to ALL     │                           │      │
     └─────────────┘                    ┌──────┴──────┘
                                        │ SWITCH sends
     Wasteful - all                     │ only to DEST
     receive data                       ▼
                                   Efficient - direct path
```

### Use Cases
- Modern office networks
- Home networks
- Most common topology today

---

## 4. Ring Topology

### Structure

Each device connected to exactly two other devices, forming a circular path.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          RING TOPOLOGY                              │
│                                                                     │
│                           ┌────────┐                                │
│                           │  PC 1  │                                │
│                           └───┬────┘                                │
│                        ┌──────┘└──────┐                             │
│                        │              │                             │
│                        ▼              ▼                             │
│                   ┌────────┐    ┌────────┐                          │
│                   │  PC 4  │    │  PC 2  │                          │
│                   └───┬────┘    └────┬───┘                          │
│                        │              │                             │
│                        │              │                             │
│                        ▼              ▼                             │
│                        └──────┐┌──────┘                             │
│                           ┌───┴┴───┐                                │
│                           │  PC 3  │                                │
│                           └────────┘                                │
│                                                                     │
│              Data travels in ONE direction (unidirectional)         │
│              or BOTH directions (bidirectional)                     │
└─────────────────────────────────────────────────────────────────────┘
```

### How It Works (Token Ring)

```
Token passing mechanism:

Step 1: Token circulates               Step 2: PC 2 captures token
                                                and attaches data
    ┌─PC1─┐                                 ┌─PC1─┐
   ╱       ╲                               ╱       ╲
TOKEN→      PC2                          PC4       PC2
   ╲       ╱                               ╲  ←──  ╱
    └─PC3─┘                                 └─PC3─┘
                                           TOKEN+DATA

Step 3: Data travels to                 Step 4: PC4 receives,
        destination (PC4)                       releases token

    ┌─PC1─┐                                 ┌─PC1─┐
   ╱       ╲                               ╱       ╲
 PC4←───────PC2                          PC4       PC2
   ╲       ╱                               ╲  TOKEN ╱
    └─PC3─┘                                 └─PC3←─┘
 DATA received                             Token free again
```

### Types of Ring Topology

**Unidirectional Ring:**
```
    ┌─► PC1 ─┐
    │        ▼
   PC4      PC2
    ▲        │
    └── PC3 ◄┘

Data flows in ONE direction only
```

**Dual Ring (Bidirectional):**
```
    ┌─► PC1 ─┐        ┌─◄ PC1 ─┐
    │   │    ▼        │        ▲
   PC4 │   PC2       PC4      PC2
    ▲  │    │         │        │
    └──┼ PC3◄┘        └► PC3 ──┘
       ▼
   PRIMARY           SECONDARY
   RING              RING (Backup)
```

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Equal access for all nodes | One node failure affects entire ring |
| No collisions (token-based) | Difficult to troubleshoot |
| Predictable performance | Adding/removing nodes disrupts network |
| Performs well under heavy load | Slower than star (token waiting) |
| No central device needed | Requires more cable than bus |

### Use Cases
- FDDI (Fiber Distributed Data Interface) networks
- SONET/SDH rings in telecom
- Industrial control systems

---

## 5. Mesh Topology

### Structure

Every device connected to every other device.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          MESH TOPOLOGY                              │
│                                                                     │
│                 FULL MESH                    PARTIAL MESH           │
│                                                                     │
│            ┌────────┐                       ┌────────┐              │
│            │  PC 1  │                       │  PC 1  │              │
│            └┬──┬──┬─┘                       └┬──┬────┘              │
│             │  │  │                          │  │                   │
│      ┌──────┘  │  └──────┐            ┌─────┘  └─────┐              │
│      │         │         │            │              │              │
│      ▼         ▼         ▼            ▼              ▼              │
│ ┌────────┐┌────────┐┌────────┐   ┌────────┐    ┌────────┐          │
│ │  PC 2  ├┤  PC 3  ├┤  PC 4  │   │  PC 2  │    │  PC 3  │          │
│ └────┬───┘└───┬────┘└───┬────┘   └────┬───┘    └───┬────┘          │
│      │        │         │             │            │                │
│      └────────┼─────────┘             └────────────┘                │
│               │                                                     │
│      Every node connected           Some nodes have                 │
│      to every other node            multiple connections            │
└─────────────────────────────────────────────────────────────────────┘
```

### Full Mesh vs Partial Mesh

```
FULL MESH (4 nodes):                    PARTIAL MESH (4 nodes):
Connections = n(n-1)/2                  Fewer connections,
            = 4(3)/2 = 6                some redundancy

      PC1                                    PC1
     /│\ \                                  / | \
    / │ \ \                                /  |  \
   /  │  \ \                              /   |   \
  /   │   \ \                            /    |    \
PC2───┼───PC3                          PC2    |    PC3
  \   │   / /                            \    |    /
   \  │  / /                              \   |   /
    \ │ / /                                \  |  /
     \│//                                   \ | /
     PC4                                    PC4
     
All 6 connections present               Only some connections
```

### Number of Links Formula

For **n** nodes in full mesh:
- **Links needed** = n(n-1)/2
- **Ports per node** = n-1

| Nodes | Links Required | Ports per Node |
|-------|---------------|----------------|
| 2 | 1 | 1 |
| 4 | 6 | 3 |
| 5 | 10 | 4 |
| 10 | 45 | 9 |
| 100 | 4,950 | 99 |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Highly reliable and fault-tolerant | Very expensive (cables and ports) |
| Multiple paths prevent bottlenecks | Complex installation and management |
| Privacy - dedicated links | Not scalable for large networks |
| Easy fault identification | Difficult to maintain |
| Excellent for critical systems | Impractical for general use |

### Use Cases
- Core network backbone
- Military and critical infrastructure
- Financial trading systems
- Internet backbone (partial mesh)

---

## 6. Tree (Hierarchical) Topology

### Structure

Combination of star topologies connected via a bus backbone.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TREE TOPOLOGY                               │
│                                                                     │
│                           ┌───────┐                                 │
│                           │ ROOT  │◄── Central/Root Device          │
│                           │SWITCH │                                 │
│                           └───┬───┘                                 │
│                               │                                     │
│               ┌───────────────┼───────────────┐                     │
│               │               │               │                     │
│           ┌───┴───┐       ┌───┴───┐       ┌───┴───┐                │
│           │Switch │       │Switch │       │Switch │◄── Second Level │
│           │   A   │       │   B   │       │   C   │                │
│           └───┬───┘       └───┬───┘       └───┬───┘                │
│               │               │               │                     │
│         ┌─────┼─────┐   ┌─────┼─────┐   ┌─────┼─────┐              │
│         │     │     │   │     │     │   │     │     │              │
│        PC1   PC2   PC3 PC4   PC5   PC6 PC7   PC8   PC9             │
│                                                                     │
│              Hierarchical structure - like an inverted tree         │
└─────────────────────────────────────────────────────────────────────┘
```

### Characteristics

```
HIERARCHY LEVELS:

Level 0 (Root):     [Core Switch]      ◄── Main backbone
                         │
Level 1:           [Distribution]       ◄── Department/Floor level
                    /    |    \
Level 2:       [Access][Access][Access] ◄── Workgroup level
                 /|\     |      /|\
Level 3:       PCs    PCs     PCs       ◄── End devices
```

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Hierarchical management | Root failure affects entire network |
| Easy to expand | More cabling than star |
| Point-to-point wiring | Dependent on backbone cable |
| Supported by most hardware | Can become complex |
| Easy to troubleshoot sections | Higher cost than star |

### Use Cases
- Large corporate networks
- Multi-floor building networks
- Enterprise environments
- School/University networks

---

## 7. Hybrid Topology

### Definition

A **hybrid topology** combines two or more different topology types.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        HYBRID TOPOLOGY                              │
│                   (Star-Bus Combination)                            │
│                                                                     │
│  ┌─────────── STAR NETWORK A ──────────┐                           │
│  │                                      │                           │
│  │    PC──Switch──PC                    │                           │
│  │        │                             │                           │
│  │       PC                             │                           │
│  └────────┬─────────────────────────────┘                           │
│           │                                                         │
│    ═══════╪═════════════════════════════════════════════            │
│           │            BUS BACKBONE            │                    │
│    ═══════╪═════════════════════════════════════════════            │
│           │                                    │                    │
│  ┌────────┴─────────────────────────────┐     │                    │
│  │                                      │     │                    │
│  │    PC──Switch──PC                    │     │                    │
│  │        │                             │     │                    │
│  │       PC                             │   ┌─┴───────────┐        │
│  │                                      │   │   Server    │        │
│  └─────────── STAR NETWORK B ──────────┘   │   Cluster   │        │
│                                             │  (Mesh)     │        │
│                                             └─────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

### Common Hybrid Combinations

| Combination | Description | Use Case |
|-------------|-------------|----------|
| **Star-Bus** | Stars connected via bus | Office buildings |
| **Star-Ring** | Stars in ring formation | FDDI networks |
| **Star-Mesh** | Core mesh with star edges | Data centers |
| **Tree-Star** | Hierarchical with stars | Enterprise networks |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Flexible and scalable | Complex design |
| Fault tolerant | Expensive to implement |
| Best features of multiple topologies | Difficult to manage |
| Suitable for large networks | Requires careful planning |

---

## 8. Topology Comparison

### Quick Reference Table

| Topology | Cable Length | Cost | Reliability | Scalability | Complexity |
|----------|-------------|------|-------------|-------------|------------|
| **Bus** | Shortest | Low | Low | Poor | Simple |
| **Star** | Moderate | Medium | Medium | Good | Simple |
| **Ring** | Moderate | Medium | Medium | Poor | Medium |
| **Mesh** | Longest | High | High | Poor | Complex |
| **Tree** | Moderate | Medium | Medium | Good | Medium |
| **Hybrid** | Varies | High | High | Excellent | Complex |

### Decision Matrix

```
When to use each topology:

                    SMALL      MEDIUM      LARGE      CRITICAL
                   NETWORK    NETWORK    NETWORK     SYSTEMS
                      │          │          │           │
BUS ─────────────────►│          │          │           │
                      │          │          │           │
STAR ────────────────►├─────────►│          │           │
                      │          │          │           │
RING ────────────────►├─────────►│          │           │
                      │          │          │           │
TREE ─────────────────┼─────────►├─────────►│           │
                      │          │          │           │
MESH ─────────────────┼──────────┼──────────┼──────────►│
                      │          │          │           │
HYBRID ───────────────┼─────────►├─────────►├──────────►│
```

---

## 📊 Summary Table

| Topology | Structure | Best For | Key Weakness |
|----------|-----------|----------|--------------|
| **Bus** | Single backbone cable | Small, temporary networks | Single point of failure |
| **Star** | Central device | Modern offices | Central device failure |
| **Ring** | Circular connection | Token-based networks | Node failure breaks ring |
| **Mesh** | All-to-all connections | Critical systems | Very expensive |
| **Tree** | Hierarchical stars | Large enterprises | Root device dependency |
| **Hybrid** | Mixed topologies | Complex requirements | High complexity |

---

## ❓ Quick Revision Questions

1. **Q:** What happens if the central cable fails in a bus topology?
   <details>
   <summary>Answer</summary>
   The entire network fails. All nodes lose connectivity because they all depend on the single backbone cable for communication.
   </details>

2. **Q:** Why is star topology preferred in modern networks?
   <details>
   <summary>Answer</summary>
   Star topology is preferred because: (1) A single cable failure only affects one node, (2) Easy to troubleshoot and isolate problems, (3) Easy to add/remove devices, (4) Centralized management, and (5) Good performance with switches.
   </details>

3. **Q:** How many cables are needed for a full mesh network with 5 nodes?
   <details>
   <summary>Answer</summary>
   Using formula n(n-1)/2 = 5(4)/2 = 10 cables
   </details>

4. **Q:** What is the main advantage of ring topology over bus topology?
   <details>
   <summary>Answer</summary>
   Ring topology eliminates collisions through token passing, providing predictable performance and equal access for all nodes. Unlike bus topology where collisions can occur, only the node with the token can transmit.
   </details>

5. **Q:** Explain the difference between physical and logical topology.
   <details>
   <summary>Answer</summary>
   Physical topology is the actual physical arrangement of cables and devices that you can see. Logical topology refers to how data actually flows through the network, which may differ from the physical layout. For example, Token Ring uses a physical star layout but data flows in a logical ring.
   </details>

6. **Q:** What is a hybrid topology? Give an example.
   <details>
   <summary>Answer</summary>
   A hybrid topology combines two or more different topology types into a single network. Example: A Star-Bus hybrid where multiple star networks (departments) are connected through a bus backbone cable running through the building.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Types of Networks](02-types-of-networks.md) | [Table of Contents](../README.md) | [Network Devices →](04-network-devices.md) |

---

*© 2026 Computer Networking Study Notes. For educational purposes.*
