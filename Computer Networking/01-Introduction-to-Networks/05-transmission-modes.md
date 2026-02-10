# Chapter 1.5: Transmission Modes

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

Transmission mode defines the direction of signal flow between two connected devices. This chapter explores the three types of transmission modes: simplex, half-duplex, and full-duplex, along with their characteristics and applications.

---

## 🎯 Learning Objectives

- Understand the three transmission modes
- Identify real-world applications of each mode
- Compare advantages and disadvantages
- Choose appropriate mode for different scenarios

---

## 1. Introduction to Transmission Modes

### Definition

**Transmission mode** (also called communication mode) refers to the direction of data flow between two devices linked by a communication channel.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TRANSMISSION MODES                              │
│                                                                     │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐           │
│  │  SIMPLEX    │     │ HALF-DUPLEX │     │ FULL-DUPLEX │           │
│  │             │     │             │     │             │           │
│  │  A ──────► B│     │  A ◄─────► B│     │  A ◄══════► B│          │
│  │             │     │  (one at    │     │  (both at   │           │
│  │ (one-way)   │     │   a time)   │     │   same time)│           │
│  └─────────────┘     └─────────────┘     └─────────────┘           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Simplex Mode

### Definition

In **simplex mode**, communication is unidirectional - data flows in only ONE direction. One device can only transmit, and the other can only receive.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SIMPLEX MODE                                 │
│                                                                     │
│     SENDER                                    RECEIVER              │
│   (Transmitter)                              (Only receives)        │
│                                                                     │
│   ┌─────────┐                                ┌─────────┐           │
│   │         │                                │         │           │
│   │ Device  │ ══════════════════════════════►│ Device  │           │
│   │    A    │          Data Flow             │    B    │           │
│   │         │         ──────────►            │         │           │
│   └─────────┘                                └─────────┘           │
│                                                                     │
│       ✓ Can send                                ✓ Can receive       │
│       ✗ Cannot receive                          ✗ Cannot send       │
│                                                                     │
│   Uses FULL bandwidth in ONE direction                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

```
KEYBOARD TO COMPUTER:

┌──────────┐               ┌────────────────┐
│          │   Keystrokes  │                │
│ KEYBOARD │ ─────────────►│    COMPUTER    │
│          │   (one-way)   │                │
└──────────┘               └────────────────┘
  Can't receive             Can't send back
  display info              to keyboard

TV BROADCAST:

┌──────────────┐            ┌─────────────────┐
│              │   Signal   │                 │
│ TV Station   │ ──────────►│  Television     │
│ (Transmitter)│            │   (Receiver)    │
└──────────────┘            └─────────────────┘
  Broadcasts                 Only receives
  content                    and displays

FIRE ALARM SYSTEM:

┌──────────────┐            ┌─────────────────┐
│              │   Alert    │                 │
│ Smoke Sensor │ ──────────►│  Alarm Panel    │
│              │            │                 │
└──────────────┘            └─────────────────┘
  Sends alerts              Receives alerts
  only                      and activates alarm
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Direction** | Unidirectional |
| **Channel Usage** | Full capacity in one direction |
| **Roles** | Fixed sender and receiver |
| **Cost** | Low (simple design) |
| **Bandwidth Utilization** | Efficient for one-way data |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Simple implementation | No feedback/acknowledgment |
| Low cost | Cannot confirm delivery |
| Full bandwidth utilization | Limited applications |
| No collision issues | Receiver cannot respond |

---

## 3. Half-Duplex Mode

### Definition

In **half-duplex mode**, communication is bidirectional but NOT simultaneous. Both devices can transmit and receive, but only one can transmit at a time.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      HALF-DUPLEX MODE                               │
│                                                                     │
│   ┌─────────┐                                ┌─────────┐           │
│   │         │ ═══════════════════════════════│         │           │
│   │ Device  │◄───────────────────────────────│ Device  │           │
│   │    A    │                                │    B    │           │
│   │         │─────────────────────────────► │         │           │
│   └─────────┘ ═══════════════════════════════└─────────┘           │
│                                                                     │
│   Both can send and receive, but NOT at the same time              │
│                                                                     │
│   TIME 1: A sends to B        TIME 2: B sends to A                 │
│   ┌───────────────────┐       ┌───────────────────┐                │
│   │ A ────────────► B │       │ A ◄──────────── B │                │
│   └───────────────────┘       └───────────────────┘                │
│   A: Transmit, B: Receive     A: Receive, B: Transmit              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Walkie-Talkie Analogy

```
┌─────────────────────────────────────────────────────────────────────┐
│                   WALKIE-TALKIE EXAMPLE                             │
│                                                                     │
│     Person A                              Person B                  │
│   ┌─────────┐                           ┌─────────┐                │
│   │ 📱      │                           │      📱 │                │
│   │ Walkie- │                           │ Walkie- │                │
│   │ Talkie  │                           │ Talkie  │                │
│   └─────────┘                           └─────────┘                │
│                                                                     │
│   SCENARIO 1: A speaks, B listens                                  │
│   ┌──────────────────────────────────────────────────┐             │
│   │ "Hello, this is Alpha, over"  ───────────────►   │             │
│   │  A: PTT pressed (speaking)     B: listening      │             │
│   └──────────────────────────────────────────────────┘             │
│                                                                     │
│   SCENARIO 2: B responds, A listens                                │
│   ┌──────────────────────────────────────────────────┐             │
│   │   ◄───────────  "Roger, Alpha. This is Bravo"    │             │
│   │  A: listening    B: PTT pressed (speaking)       │             │
│   └──────────────────────────────────────────────────┘             │
│                                                                     │
│   PTT = Push-To-Talk (you must release to listen)                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

```
1. WALKIE-TALKIE / CB RADIO:
   ┌────────┐         ┌────────┐
   │Radio A │◄───────►│Radio B │    Push button to talk
   └────────┘         └────────┘    Release to listen

2. POLICE/EMERGENCY RADIO:
   ┌─────────┐        ┌─────────┐
   │Dispatch │◄──────►│Patrol   │   One speaks at a time
   └─────────┘        └─────────┘

3. LEGACY ETHERNET HUB (10BASE-T with CSMA/CD):
   ┌────────┐        ┌────────┐
   │ PC 1   │◄──────►│ PC 2   │    Collision detection
   └────────┘        └────────┘    required
```

### Time Division in Half-Duplex

```
TIME ──────────────────────────────────────────────────────────►

    ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
    │  A → B      │   │  B → A      │   │  A → B      │
    │ (A sends)   │   │ (B sends)   │   │ (A sends)   │
    └─────────────┘   └─────────────┘   └─────────────┘
    
    ├── Turn 1 ───┼──── Turn 2 ───┼──── Turn 3 ───►

    Channel alternates between devices
    Total capacity is shared (not simultaneous)
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Direction** | Bidirectional (alternating) |
| **Channel Usage** | Shared, one at a time |
| **Roles** | Dynamic (can switch) |
| **Cost** | Moderate |
| **Turnaround Time** | Delay when switching direction |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Two-way communication | Only one direction at a time |
| Simpler than full-duplex | Reduced throughput |
| Lower cost than full-duplex | Turnaround time delay |
| Feedback possible | Collisions possible |

---

## 4. Full-Duplex Mode

### Definition

In **full-duplex mode**, both devices can transmit and receive simultaneously. Data flows in both directions at the same time.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      FULL-DUPLEX MODE                               │
│                                                                     │
│   ┌─────────┐                                ┌─────────┐           │
│   │         │ ═══════════════════════════════│         │           │
│   │ Device  │═══════════════════════════════►│ Device  │           │
│   │    A    │◄═══════════════════════════════│    B    │           │
│   │         │ ═══════════════════════════════│         │           │
│   └─────────┘                                └─────────┘           │
│                                                                     │
│   Both devices transmit AND receive SIMULTANEOUSLY                  │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                                                              │  │
│   │     A ════════════════════════════════════════► B           │  │
│   │     A ◄════════════════════════════════════════ B           │  │
│   │         (Both streams active at same time)                   │  │
│   │                                                              │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   Uses TWO separate channels or frequency bands                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Telephone Conversation Analogy

```
┌─────────────────────────────────────────────────────────────────────┐
│                   TELEPHONE CONVERSATION                            │
│                                                                     │
│     Person A                              Person B                  │
│   ┌─────────┐                           ┌─────────┐                │
│   │   ☏     │                           │     ☏   │                │
│   │  Phone  │                           │  Phone  │                │
│   └─────────┘                           └─────────┘                │
│                                                                     │
│   ┌──────────────────────────────────────────────────┐             │
│   │                                                   │             │
│   │    "How are you?"  ────────────────────────►     │             │
│   │    ◄────────────────────────── "I'm fine!"       │             │
│   │                                                   │             │
│   │    Both speaking and listening at same time      │             │
│   │    (though polite people take turns!)            │             │
│   │                                                   │             │
│   └──────────────────────────────────────────────────┘             │
│                                                                     │
│   Technical implementation: Separate TX and RX channels             │
└─────────────────────────────────────────────────────────────────────┘
```

### How Full-Duplex Works

```
METHOD 1: SEPARATE PHYSICAL CHANNELS
┌────────────────────────────────────────────────────────────────────┐
│                                                                     │
│    Device A                                        Device B         │
│   ┌────────┐     ─────── TX Wire ──────►        ┌────────┐        │
│   │        │     ◄────── RX Wire ───────        │        │        │
│   │   TX ──┼──────────────────────────────────► │── RX   │        │
│   │   RX ◄─┼──────────────────────────────────◄ │── TX   │        │
│   │        │                                    │        │        │
│   └────────┘                                    └────────┘        │
│                                                                     │
│   Example: Modern Ethernet (separate TX/RX pairs)                  │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘

METHOD 2: FREQUENCY DIVISION
┌────────────────────────────────────────────────────────────────────┐
│                                                                     │
│    Same physical medium, different frequencies                      │
│                                                                     │
│    ┌────────────────────────────────────────────────────────────┐  │
│    │                                                             │  │
│    │    A → B uses Frequency f1  ════════════════►              │  │
│    │    A ← B uses Frequency f2  ◄════════════════              │  │
│    │                                                             │  │
│    └────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   Example: DSL (uploads and downloads use different frequencies)   │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

```
1. TELEPHONE SYSTEM:
   ┌────────┐ ◄═══════════════► ┌────────┐
   │Phone A │  Simultaneous     │Phone B │
   └────────┘  conversation     └────────┘

2. MODERN ETHERNET (Twisted Pair):
   ┌────────┐        ┌────────┐
   │ PC     │◄══════►│Switch  │    Separate wire pairs for
   └────────┘        └────────┘    TX and RX

3. VIDEO CONFERENCING:
   ┌────────────┐    ┌────────────┐
   │ Camera A   │◄══►│ Camera B   │   Simultaneous audio
   │   + Mic    │    │   + Mic    │   and video both ways
   └────────────┘    └────────────┘

4. FIBER OPTIC LINKS:
   ┌────────┐        ┌────────┐
   │Server  │◄══════►│Router  │    Two fibers or
   └────────┘        └────────┘    wavelength division
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Direction** | Bidirectional (simultaneous) |
| **Channel Usage** | Two channels or shared with division |
| **Roles** | Both TX and RX simultaneously |
| **Cost** | Higher |
| **Throughput** | Double compared to half-duplex |

### Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Simultaneous two-way communication | More complex/expensive |
| Higher throughput | Requires more bandwidth or separation |
| No turnaround time | More sophisticated hardware |
| Instant feedback | Higher power consumption |
| Better for real-time applications | Echo cancellation may be needed |

---

## 5. Comparison of Transmission Modes

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                   TRANSMISSION MODE COMPARISON                      │
│                                                                     │
│  SIMPLEX:                                                           │
│  ┌───────┐ ════════════════════════════════════► ┌───────┐         │
│  │   A   │           One-way only                 │   B   │         │
│  └───────┘                                        └───────┘         │
│  Capacity: 100% → one direction                                     │
│                                                                     │
│  HALF-DUPLEX:                                                       │
│  ┌───────┐ ◄════════════════════════════════════► ┌───────┐        │
│  │   A   │        Two-way, alternating            │   B   │        │
│  └───────┘                                        └───────┘        │
│  Capacity: 100% → or ← (but not both)                              │
│                                                                     │
│  FULL-DUPLEX:                                                       │
│  ┌───────┐ ◄═══════════════════════════════════► ┌───────┐         │
│  │   A   │ ════════════════════════════════════► │   B   │         │
│  └───────┘        Two-way, simultaneous           └───────┘         │
│  Capacity: 100% → AND 100% ← (200% total)                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Bandwidth Utilization

```
SIMPLEX (100 Mbps link):
├══════════════════════════════════════════════════════════════►
                    100 Mbps in one direction only

HALF-DUPLEX (100 Mbps link):
├══════════════════════════════►  (when A sends)
◄══════════════════════════════┤  (when B sends)
                    100 Mbps total, shared

FULL-DUPLEX (100 Mbps link):
├══════════════════════════════════════════════════════════════►  100 Mbps
◄══════════════════════════════════════════════════════════════┤  100 Mbps
                    200 Mbps effective (100 each way)
```

### Comprehensive Comparison Table

| Feature | Simplex | Half-Duplex | Full-Duplex |
|---------|---------|-------------|-------------|
| **Direction** | One-way | Two-way (alternating) | Two-way (simultaneous) |
| **Channels** | 1 | 1 (shared) | 2 (or divided) |
| **Transmission** | A→B only | A→B or B→A | A↔B |
| **Bandwidth** | Full, one direction | Shared | Full, both directions |
| **Complexity** | Simple | Moderate | Complex |
| **Cost** | Low | Medium | High |
| **Delay** | None | Turnaround time | None |
| **Example** | TV broadcast | Walkie-talkie | Telephone |

---

## 6. Practical Applications

### When to Use Each Mode

```
┌─────────────────────────────────────────────────────────────────────┐
│                    APPLICATION GUIDE                                │
│                                                                     │
│  SIMPLEX - Use when:                                                │
│  ├─ One-way information flow is sufficient                         │
│  ├─ No response expected from receiver                             │
│  ├─ Broadcasting to many receivers                                 │
│  └─ Examples: TV, Radio, Fire alarms, PA systems                   │
│                                                                     │
│  HALF-DUPLEX - Use when:                                            │
│  ├─ Two-way communication needed but not simultaneous              │
│  ├─ Cost/complexity must be minimized                              │
│  ├─ Turn-taking is acceptable                                      │
│  └─ Examples: Walkie-talkies, Legacy Ethernet, CB radio            │
│                                                                     │
│  FULL-DUPLEX - Use when:                                            │
│  ├─ Real-time bidirectional communication required                 │
│  ├─ High throughput needed                                         │
│  ├─ No delays acceptable                                           │
│  └─ Examples: Telephone, Video calls, Modern Ethernet              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Networking Context

```
ETHERNET EVOLUTION:

Legacy Ethernet (Hub-based):        Modern Ethernet (Switch-based):
┌────────────────────────────┐      ┌────────────────────────────┐
│                            │      │                            │
│   PC ────┐                 │      │   PC ──────► Switch        │
│          │                 │      │      ◄──────               │
│          ▼                 │      │                            │
│   ┌──────────┐             │      │   Full-duplex              │
│   │   HUB    │             │      │   Each direction has       │
│   └──────────┘             │      │   dedicated bandwidth      │
│                            │      │                            │
│   Half-duplex              │      │   100 Mbps TX + 100 Mbps RX│
│   Shared medium            │      │   = 200 Mbps effective     │
│   CSMA/CD required         │      │                            │
│                            │      │   No collisions            │
└────────────────────────────┘      └────────────────────────────┘
```

---

## 📊 Summary Table

| Mode | Direction | Simultaneity | Example | Best For |
|------|-----------|--------------|---------|----------|
| **Simplex** | One-way | N/A | TV broadcast, keyboard | Streaming, sensors |
| **Half-Duplex** | Two-way | Alternating | Walkie-talkie | Cost-sensitive |
| **Full-Duplex** | Two-way | Simultaneous | Telephone | Real-time communication |

---

## ❓ Quick Revision Questions

1. **Q:** Define simplex mode and give two examples.
   <details>
   <summary>Answer</summary>
   Simplex mode is unidirectional communication where data flows in only one direction. The sender can only transmit, and the receiver can only receive. Examples: Television broadcast, keyboard input to computer, smoke detector to alarm panel.
   </details>

2. **Q:** Why can't two people speak at the same time on a walkie-talkie?
   <details>
   <summary>Answer</summary>
   Walkie-talkies use half-duplex communication. The same frequency channel is used for both transmission and reception, so only one person can transmit at a time. When you press the Push-To-Talk (PTT) button, your device switches from receive mode to transmit mode.
   </details>

3. **Q:** What is the main advantage of full-duplex over half-duplex?
   <details>
   <summary>Answer</summary>
   Full-duplex allows simultaneous bidirectional communication, effectively doubling the throughput compared to half-duplex. There's no turnaround time or waiting for the other party to finish, enabling real-time interactive communication.
   </details>

4. **Q:** How does modern Ethernet achieve full-duplex communication?
   <details>
   <summary>Answer</summary>
   Modern Ethernet uses separate wire pairs for transmission and reception (in twisted pair cables), or separate fibers (in fiber optic). This physical separation allows data to flow in both directions simultaneously without collision.
   </details>

5. **Q:** Compare bandwidth utilization in simplex and full-duplex modes for a 100 Mbps link.
   <details>
   <summary>Answer</summary>
   - Simplex: 100 Mbps in one direction only
   - Full-Duplex: 100 Mbps in each direction simultaneously, for an effective throughput of 200 Mbps (though still 100 Mbps per direction)
   </details>

6. **Q:** Which transmission mode would you choose for a public announcement system in a building? Why?
   <details>
   <summary>Answer</summary>
   Simplex mode would be appropriate because announcements flow in one direction only - from the announcer to the listeners. There's no need for listeners to respond through the system, and simplex is simpler and more cost-effective for this application.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Network Devices](04-network-devices.md) | [Table of Contents](../README.md) | [OSI Model →](../02-Network-Models/01-osi-model.md) |

---

*© 2026 Computer Networking Study Notes. For educational purposes.*
