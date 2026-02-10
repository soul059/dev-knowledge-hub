# Chapter 4.3: Flow Control

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

Flow control is a technique to manage the rate of data transmission between a fast sender and a slow receiver. This chapter covers flow control mechanisms including Stop-and-Wait, Sliding Window protocols, and their variations.

---

## 🎯 Learning Objectives

- Understand the need for flow control
- Master Stop-and-Wait protocol
- Learn Sliding Window concept
- Compare Go-Back-N and Selective Repeat ARQ

---

## 1. Need for Flow Control

### The Problem

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WHY FLOW CONTROL?                                │
│                                                                     │
│    SCENARIO: Fast sender, slow receiver                            │
│                                                                     │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  ┌──────────┐                            ┌──────────┐       │ │
│    │  │  FAST    │ Frame Frame Frame Frame   │  SLOW    │       │ │
│    │  │  SENDER  │ ═══════════════════════▶  │ RECEIVER │       │ │
│    │  │  100Mbps │ ═══════════════════════▶  │  10Mbps  │       │ │
│    │  └──────────┘ ═══════════════════════▶  └──────────┘       │ │
│    │               ═══════════════════════▶       │              │ │
│    │                                              ▼              │ │
│    │                                        ┌──────────┐         │ │
│    │                                        │ BUFFER   │         │ │
│    │                                        │ OVERFLOW!│         │ │
│    │                                        │ DATA LOST│         │ │
│    │                                        └──────────┘         │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    PROBLEMS WITHOUT FLOW CONTROL:                                   │
│    • Receiver buffer overflow                                      │
│    • Data loss                                                     │
│    • Need for retransmission                                       │
│    • Wasted bandwidth                                              │
│                                                                     │
│    SOLUTION:                                                        │
│    • Sender must wait for receiver acknowledgment                  │
│    • Match sending rate to receiving capacity                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Stop-and-Wait Protocol

### Basic Mechanism

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STOP-AND-WAIT PROTOCOL                           │
│                                                                     │
│    Simplest flow control mechanism                                 │
│    Send one frame, wait for ACK, then send next                   │
│                                                                     │
│    NORMAL OPERATION:                                                │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  SENDER                                RECEIVER              │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 0 ─────────────────────▶│                  │ │
│    │    │                                      │                  │ │
│    │    │◀──────────── ACK ────────────────────│                  │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 1 ─────────────────────▶│                  │ │
│    │    │                                      │                  │ │
│    │    │◀──────────── ACK ────────────────────│                  │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 2 ─────────────────────▶│                  │ │
│    │    │                                      │                  │ │
│    │    ▼                                      ▼                  │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    RULES:                                                           │
│    1. Sender sends ONE frame                                       │
│    2. Sender STOPS and waits for ACK                               │
│    3. Receiver sends ACK after receiving frame                     │
│    4. Sender receives ACK, sends next frame                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Stop-and-Wait with Errors

```
┌─────────────────────────────────────────────────────────────────────┐
│            STOP-AND-WAIT ARQ (Automatic Repeat Request)             │
│                                                                     │
│    Handles lost/damaged frames using:                              │
│    • Sequence numbers (0 and 1 alternate)                          │
│    • ACK numbers                                                   │
│    • Timeout timer                                                 │
│                                                                     │
│    SCENARIO 1: Lost Frame                                           │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  SENDER                                RECEIVER              │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 0 ───────────X          │ (Lost!)          │ │
│    │    │                                      │                  │ │
│    │    │  ⏱️ Timeout...                       │                  │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 0 (resend) ────────────▶│                  │ │
│    │    │                                      │                  │ │
│    │    │◀──────────── ACK 1 ──────────────────│                  │ │
│    │    │                                      │                  │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    SCENARIO 2: Lost ACK                                             │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  SENDER                                RECEIVER              │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 0 ─────────────────────▶│                  │ │
│    │    │                                      │                  │ │
│    │    │         X←──── ACK 1 ────────────────│ (Lost!)          │ │
│    │    │                                      │                  │ │
│    │    │  ⏱️ Timeout...                       │                  │ │
│    │    │                                      │                  │ │
│    │    │─────── Frame 0 (resend) ────────────▶│ (Duplicate!)     │ │
│    │    │                                      │ Discard, re-ACK  │ │
│    │    │◀──────────── ACK 1 ──────────────────│                  │ │
│    │    │                                      │                  │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    SEQUENCE NUMBERS:                                                │
│    • Frame 0 → expects ACK 1                                       │
│    • Frame 1 → expects ACK 0                                       │
│    • ACK n = "I've received all frames up to n-1, send frame n"   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Stop-and-Wait Efficiency

```
┌─────────────────────────────────────────────────────────────────────┐
│            STOP-AND-WAIT EFFICIENCY                                 │
│                                                                     │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  SENDER          Channel            RECEIVER                 │ │
│    │    │                                    │                    │ │
│    │    │═══════ Frame ════════════════════▶│                    │ │
│    │    │←─────────────────────────────────→│                    │ │
│    │    │    Transmission time (Tt)          │                    │ │
│    │    │                                    │                    │ │
│    │    │←───────── Propagation (Tp) ───────│                    │ │
│    │    │                                    │                    │ │
│    │    │◀════════ ACK ═════════════════════│                    │ │
│    │    │←───────── Propagation (Tp) ───────│                    │ │
│    │    │                                    │                    │ │
│    │    ←──────────────────────────────────→                     │ │
│    │              Total cycle time                                │ │
│    │              = Tt + 2Tp + Tack                              │ │
│    │              ≈ Tt + 2Tp (ACK is small)                      │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    EFFICIENCY FORMULA:                                              │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │              Tt           1                                  │ │
│    │  η = ───────────── = ─────────                              │ │
│    │       Tt + 2Tp       1 + 2a                                 │ │
│    │                                                              │ │
│    │  Where: a = Tp/Tt = propagation time / transmission time   │ │
│    │                                                              │ │
│    │  EXAMPLE:                                                    │ │
│    │  • Frame size = 1000 bits                                   │ │
│    │  • Bandwidth = 1 Mbps                                       │ │
│    │  • RTT = 20 ms                                              │ │
│    │                                                              │ │
│    │  Tt = 1000/1,000,000 = 1 ms                                 │ │
│    │  Tp = 20/2 = 10 ms                                          │ │
│    │  a = 10/1 = 10                                              │ │
│    │                                                              │ │
│    │  η = 1/(1 + 2×10) = 1/21 ≈ 4.76%                           │ │
│    │                                                              │ │
│    │  VERY INEFFICIENT for long-distance links!                  │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Sliding Window Protocol

### Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SLIDING WINDOW CONCEPT                           │
│                                                                     │
│    Allow multiple frames to be in transit simultaneously           │
│    Use "window" to track outstanding frames                        │
│                                                                     │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  SENDER WINDOW (size = 4):                                   │ │
│    │                                                              │ │
│    │  Frames: 0  1  2  3  4  5  6  7  8  9  10 11 ...            │ │
│    │         [═══════════]                                        │ │
│    │          └─ Window ─┘                                        │ │
│    │             Can send 0,1,2,3 without waiting                │ │
│    │                                                              │ │
│    │  After ACK 2 received:                                      │ │
│    │                                                              │ │
│    │  Frames: 0  1  2  3  4  5  6  7  8  9  10 11 ...            │ │
│    │               [═══════════]                                  │ │
│    │                └─ Window ─┘                                  │ │
│    │             Window "slides" forward                         │ │
│    │             Can now send 2,3,4,5                            │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    RECEIVER WINDOW:                                                 │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  Frames: 0  1  2  3  4  5  6  7  8  9  10 11 ...            │ │
│    │         [═]                    (Window size = 1 for GBN)    │ │
│    │         [═══════════]          (Window size > 1 for SR)     │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    WINDOW SIZE:                                                     │
│    • Sender window ≤ 2^m - 1 (where m = bits for sequence number) │
│    • For 3-bit sequence: max window = 7                            │
│    • Larger window = better efficiency (up to a limit)            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Sliding Window Efficiency

```
┌─────────────────────────────────────────────────────────────────────┐
│            SLIDING WINDOW EFFICIENCY                                │
│                                                                     │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  SENDER                                    RECEIVER          │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│              │ │
│    │    │═══ Frame 1 ═════════════════════════════▶│              │ │
│    │    │═══ Frame 2 ═════════════════════════════▶│              │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│              │ │
│    │    │                                          │              │ │
│    │    │◀═══════════════════════════════ ACK 0 ═══│              │ │
│    │    │═══ Frame 4 ═════════════════════════════▶│              │ │
│    │    │◀═══════════════════════════════ ACK 1 ═══│              │ │
│    │    │═══ Frame 5 ═════════════════════════════▶│              │ │
│    │    │                                          │              │ │
│    │    │  Window keeps "sliding" as ACKs arrive   │              │ │
│    │    │  Channel is continuously utilized!       │              │ │
│    │    │                                          │              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    EFFICIENCY:                                                      │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  If window size W ≥ 1 + 2a:                                 │ │
│    │      η = 1 (100% efficiency, channel always busy)           │ │
│    │                                                              │ │
│    │  If window size W < 1 + 2a:                                 │ │
│    │      η = W / (1 + 2a)                                       │ │
│    │                                                              │ │
│    │  From previous example (a = 10):                            │ │
│    │  Need W ≥ 21 for 100% efficiency                            │ │
│    │                                                              │ │
│    │  With W = 4: η = 4/21 ≈ 19% (vs 4.76% for stop-and-wait)   │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Go-Back-N ARQ

### Protocol Operation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GO-BACK-N ARQ                                    │
│                                                                     │
│    • Sender window size: N (up to 2^m - 1)                         │
│    • Receiver window size: 1                                       │
│    • Cumulative ACKs                                               │
│    • On error: retransmit frame and ALL subsequent frames          │
│                                                                     │
│    NORMAL OPERATION:                                                │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  SENDER                                    RECEIVER          │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 1 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 2 ═════════════════════════════▶│ Accept       │ │
│    │    │◀═══════════════════════════════ ACK 2 ═══│              │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 4 ═════════════════════════════▶│ Accept       │ │
│    │    │◀═══════════════════════════════ ACK 4 ═══│              │ │
│    │    │                                          │              │ │
│    │    ACK n = "I received all frames up to n"   │              │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    ERROR SCENARIO (Frame 2 lost):                                   │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  SENDER                                    RECEIVER          │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 1 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 2 ═══════X                      │ LOST!        │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│ Discard (out of order) │
│    │    │═══ Frame 4 ═════════════════════════════▶│ Discard      │ │
│    │    │◀═══════════════════════════════ ACK 1 ═══│ (NAK for 2)  │ │
│    │    │                                          │              │ │
│    │    │  Timeout or NAK received - GO BACK to 2  │              │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 2 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 4 ═════════════════════════════▶│ Accept       │ │
│    │    │◀═══════════════════════════════ ACK 4 ═══│              │ │
│    │    │                                          │              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    NOTE: Frames 3 and 4 must be retransmitted even though         │
│          they were sent correctly! (Wasteful on error)             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Go-Back-N Window Size Limit

```
┌─────────────────────────────────────────────────────────────────────┐
│            GO-BACK-N WINDOW SIZE                                    │
│                                                                     │
│    Maximum window size: W ≤ 2^m - 1                                │
│                                                                     │
│    WHY NOT 2^m?                                                     │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  Example: 2-bit sequence (m=2), try W = 4                   │ │
│    │  Sequence numbers: 0, 1, 2, 3 (then wraps to 0)             │ │
│    │                                                              │ │
│    │  SENDER                                    RECEIVER          │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 1 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 2 ═════════════════════════════▶│ Accept       │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│ Accept       │ │
│    │    │                                          │              │ │
│    │    │         X←═══ All ACKs LOST ═════════════│              │ │
│    │    │                                          │              │ │
│    │    │  Timeout... resend all                   │              │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│ ???          │ │
│    │    │                                          │              │ │
│    │    │  Receiver expects Frame 0 (next in seq)  │              │ │
│    │    │  But is this a NEW Frame 0 or duplicate? │              │ │
│    │    │  AMBIGUITY!                              │              │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    SOLUTION: W ≤ 2^m - 1 = 3 for m = 2                            │
│    Now Frame 0 resend is clearly identified as old                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Selective Repeat ARQ

### Protocol Operation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SELECTIVE REPEAT ARQ                             │
│                                                                     │
│    • Sender window size: N                                         │
│    • Receiver window size: N (can buffer out-of-order frames)      │
│    • Individual ACKs (not cumulative)                              │
│    • Only retransmit SPECIFIC lost/damaged frames                  │
│                                                                     │
│    ERROR SCENARIO (Frame 2 lost):                                   │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  SENDER                                    RECEIVER          │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 0 ═════════════════════════════▶│ Accept, ACK 0│ │
│    │    │═══ Frame 1 ═════════════════════════════▶│ Accept, ACK 1│ │
│    │    │═══ Frame 2 ═══════X                      │ LOST!        │ │
│    │    │═══ Frame 3 ═════════════════════════════▶│ BUFFER (out of order) │
│    │    │═══ Frame 4 ═════════════════════════════▶│ BUFFER       │ │
│    │    │◀═══════════════════════════════ ACK 0 ═══│              │ │
│    │    │◀═══════════════════════════════ ACK 1 ═══│              │ │
│    │    │◀═════════════════════════════ NAK 2 ═════│ (or timeout) │ │
│    │    │◀═══════════════════════════════ ACK 3 ═══│              │ │
│    │    │◀═══════════════════════════════ ACK 4 ═══│              │ │
│    │    │                                          │              │ │
│    │    │═══ Frame 2 (only!) ═════════════════════▶│ Accept       │ │
│    │    │◀═══════════════════════════════ ACK 2 ═══│              │ │
│    │    │                                          │              │ │
│    │    │  Receiver now has 0,1,2,3,4 → deliver to upper layer   │ │
│    │    │                                                         │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    ADVANTAGES:                                                      │
│    • Only lost frame is retransmitted                              │
│    • More efficient than Go-Back-N when errors occur               │
│                                                                     │
│    DISADVANTAGES:                                                   │
│    • More complex receiver (needs buffering)                       │
│    • More complex logic for reordering                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Selective Repeat Window Size

```
┌─────────────────────────────────────────────────────────────────────┐
│            SELECTIVE REPEAT WINDOW SIZE                             │
│                                                                     │
│    Maximum window size: W ≤ 2^(m-1) = 2^m / 2                      │
│                                                                     │
│    WHY SMALLER THAN GO-BACK-N?                                      │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  Both sender AND receiver have windows                      │ │
│    │  Receiver can accept out-of-order frames                    │ │
│    │                                                              │ │
│    │  Example: 3-bit sequence (m=3), try W = 5                   │ │
│    │  Sequence: 0,1,2,3,4,5,6,7 (wraps to 0)                     │ │
│    │                                                              │ │
│    │  Sender sends: 0,1,2,3,4                                    │ │
│    │  All ACKs lost                                              │ │
│    │  Sender resends: 0,1,2,3,4                                  │ │
│    │                                                              │ │
│    │  Receiver was expecting: 5,6,7,0,1                          │ │
│    │  Receiver accepts 0,1 as NEW (but they're duplicates!)      │ │
│    │                                                              │ │
│    │  SOLUTION: W ≤ 4 (half of 8)                                │ │
│    │  Sender window and receiver window don't overlap            │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    WINDOW VISUALIZATION:                                            │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │                                                              │ │
│    │  Sequence: 0  1  2  3 │ 4  5  6  7                          │ │
│    │            ├─────────┤ ├─────────┤                          │ │
│    │            Sender      Receiver                              │ │
│    │            Window      Window                                │ │
│    │                                                              │ │
│    │  No overlap = no ambiguity!                                 │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Comparison of ARQ Protocols

```
┌─────────────────────────────────────────────────────────────────────┐
│            ARQ PROTOCOLS COMPARISON                                 │
│                                                                     │
│  ┌────────────────┬─────────────┬─────────────┬─────────────────┐  │
│  │ Feature        │ Stop-Wait   │ Go-Back-N   │ Selective Repeat│  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Sender Window  │ 1           │ N (≤2^m-1)  │ N (≤2^(m-1))   │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Receiver Window│ 1           │ 1           │ N              │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ ACK Type       │ Individual  │ Cumulative  │ Individual     │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Retransmission │ Single frame│ Frame + all │ Only lost frame│  │
│  │                │             │ following   │                 │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Receiver Buffer│ None        │ None        │ Required       │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Efficiency     │ Poor        │ Good        │ Best           │  │
│  │ (no errors)    │ 1/(1+2a)    │ N/(1+2a)    │ N/(1+2a)       │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Efficiency     │ Poor        │ Moderate    │ Good           │  │
│  │ (with errors)  │             │ (wastes BW) │                 │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Complexity     │ Simple      │ Moderate    │ Complex        │  │
│  ├────────────────┼─────────────┼─────────────┼─────────────────┤  │
│  │ Best Use       │ Low latency │ Moderate    │ High error rate│  │
│  │                │ links       │ error links │ or high latency│  │
│  └────────────────┴─────────────┴─────────────┴─────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Piggybacking

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PIGGYBACKING                                     │
│                                                                     │
│    Combine ACK with data frame in bidirectional communication     │
│    Reduces overhead of separate ACK frames                         │
│                                                                     │
│    WITHOUT PIGGYBACKING:                                            │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  Station A                               Station B           │ │
│    │    │                                        │                │ │
│    │    │═══════ Data Frame ════════════════════▶│                │ │
│    │    │◀═══════════ ACK ═══════════════════════│                │ │
│    │    │                                        │                │ │
│    │    │◀═════════ Data Frame ══════════════════│                │ │
│    │    │════════════ ACK ══════════════════════▶│                │ │
│    │    │                                        │                │ │
│    │    4 separate transmissions!                │                │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    WITH PIGGYBACKING:                                               │
│    ┌─────────────────────────────────────────────────────────────┐ │
│    │  Station A                               Station B           │ │
│    │    │                                        │                │ │
│    │    │═══════ Data Frame ════════════════════▶│                │ │
│    │    │                                        │                │ │
│    │    │◀═════ Data + ACK (piggybacked) ════════│                │ │
│    │    │                                        │                │ │
│    │    │═════ Data + ACK (piggybacked) ════════▶│                │ │
│    │    │                                        │                │ │
│    │    Only 3 transmissions!                    │                │ │
│    │                                                              │ │
│    └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│    PIGGYBACK FRAME:                                                 │
│    ┌───────────────────────────────────────────────────────────┐   │
│    │ Header │ SEQ # │ ACK # │ DATA │ Trailer │                 │   │
│    └───────────────────────────────────────────────────────────┘   │
│              ↑         ↑                                           │
│         This frame's   Acknowledging                               │
│         sequence #     received frame                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Protocol | Sender Window | Receiver Window | Max Window Size | On Error |
|----------|---------------|-----------------|-----------------|----------|
| **Stop-and-Wait** | 1 | 1 | 1 | Retransmit 1 |
| **Go-Back-N** | N | 1 | 2^m - 1 | Retransmit all from error |
| **Selective Repeat** | N | N | 2^(m-1) | Retransmit only lost |

---

## ❓ Quick Revision Questions

1. **Q:** What is the main problem that flow control solves?
   <details>
   <summary>Answer</summary>
   Flow control prevents a **fast sender from overwhelming a slow receiver**. Without flow control, the receiver's buffer would overflow, causing data loss and requiring retransmission. Flow control ensures the sender adjusts its transmission rate to match the receiver's processing capacity.
   </details>

2. **Q:** Calculate the efficiency of Stop-and-Wait for a link with bandwidth 10 Mbps, frame size 10,000 bits, and propagation delay 10 ms.
   <details>
   <summary>Answer</summary>
   - Tt (transmission time) = Frame size / Bandwidth = 10,000 / 10,000,000 = 1 ms
   - Tp (propagation delay) = 10 ms
   - a = Tp / Tt = 10 / 1 = 10
   - **Efficiency η = 1 / (1 + 2a) = 1 / 21 ≈ 4.76%**
   
   Only 4.76% of the channel capacity is utilized!
   </details>

3. **Q:** Why is the maximum window size for Go-Back-N = 2^m - 1?
   <details>
   <summary>Answer</summary>
   If window size equals 2^m, there's **ambiguity** when all ACKs are lost. The receiver can't distinguish between:
   - A retransmission of old frames, or
   - New frames with the same sequence numbers
   
   With window size = 2^m - 1, there's always at least one unused sequence number, making it clear whether a frame is new or a retransmission.
   </details>

4. **Q:** How does Selective Repeat handle a lost frame differently from Go-Back-N?
   <details>
   <summary>Answer</summary>
   | Go-Back-N | Selective Repeat |
   |-----------|------------------|
   | Receiver discards out-of-order frames | Receiver buffers out-of-order frames |
   | Retransmits lost frame AND all subsequent frames | Retransmits ONLY the lost frame |
   | Receiver window = 1 | Receiver window = N |
   | Wastes bandwidth on errors | More efficient on errors |
   | Simpler implementation | More complex (needs reordering) |
   </details>

5. **Q:** What is piggybacking and what advantage does it provide?
   <details>
   <summary>Answer</summary>
   **Piggybacking** is the technique of combining an acknowledgment (ACK) with a data frame when both stations have data to send to each other. Instead of sending a separate ACK frame, the acknowledgment is included in the header of the outgoing data frame.
   
   **Advantages:**
   - Reduces the number of frames transmitted
   - Improves channel efficiency
   - Reduces overhead of sending separate ACK frames
   </details>

6. **Q:** For a 3-bit sequence number field, what is the maximum window size for Selective Repeat?
   <details>
   <summary>Answer</summary>
   For Selective Repeat: **Max window size = 2^(m-1) = 2^(3-1) = 2^2 = 4**
   
   This is because both sender and receiver have windows, and the windows must not overlap to avoid ambiguity. With m = 3 bits (8 sequence numbers), using window size 4 ensures the sender's window (e.g., 0-3) and receiver's window (e.g., 4-7) don't overlap.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Error Detection and Correction](02-error-detection.md) | [Table of Contents](../README.md) | [Media Access Control →](04-media-access-control.md) |

---

*© 2025 Computer Networking Study Notes. For educational purposes.*
