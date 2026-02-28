# Unit 10: FPGA and ASIC Design

## 📋 Unit Overview

**FPGA and ASIC** represent the two main implementation paths for digital designs. This unit compares these technologies, covering FPGA architecture, ASIC design flow, and the trade-offs in choosing between them.

---

## 🎯 Unit Learning Objectives

After completing this unit, you will be able to:
- Understand FPGA architecture and programming
- Follow the complete ASIC design flow
- Compare FPGA vs ASIC for different applications
- Make informed technology selection decisions

---

## 📊 FPGA vs ASIC Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IMPLEMENTATION PATHS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                        Digital Design                                │
│                            │                                         │
│              ┌─────────────┴─────────────┐                          │
│              │                           │                           │
│              ▼                           ▼                           │
│   ┌─────────────────────┐     ┌─────────────────────┐               │
│   │        FPGA         │     │        ASIC         │               │
│   │ (Field Programmable │     │ (Application        │               │
│   │  Gate Array)        │     │  Specific IC)       │               │
│   └─────────────────────┘     └─────────────────────┘               │
│              │                           │                           │
│              ▼                           ▼                           │
│   ┌─────────────────────┐     ┌─────────────────────┐               │
│   │ • Buy off-the-shelf │     │ • Custom fabricated │               │
│   │ • Program in field  │     │ • Fixed after fab   │               │
│   │ • Fast time-to-mkt  │     │ • Months to produce │               │
│   │ • Higher unit cost  │     │ • Lower unit cost   │               │
│   │ • Lower performance │     │ • Best performance  │               │
│   │ • More power/area   │     │ • Optimized P & A   │               │
│   └─────────────────────┘     └─────────────────────┘               │
│                                                                      │
│                                                                      │
│   When to choose:                                                   │
│                                                                      │
│   FPGA best for:              ASIC best for:                        │
│   • Prototyping               • High volume (>100K units)          │
│   • Low volume (<10K)         • Maximum performance                │
│   • Reconfigurable systems    • Lowest power                       │
│   • Fast time-to-market       • Lowest unit cost                   │
│   • Algorithm development     • Analog/mixed-signal               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📈 Cost Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COST vs VOLUME                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Total                                                              │
│   Cost ($)                                                          │
│       ↑                                                             │
│       │                                                             │
│       │  ASIC                                                       │
│       │  Total                        ╱                             │
│       │   ╱                         ╱                               │
│       │  ╱                        ╱                                 │
│       │ ╱                       ╱                                   │
│       │╱                      ╱  FPGA                               │
│       ├──────────────────────●─────────────────────                 │
│       │               Crossover                                     │
│       │               point                                         │
│       │              (~50-100K                                      │
│       │               units)                                        │
│       └────────────────────────────────────────────→ Volume         │
│                                                                      │
│   ASIC:                            FPGA:                            │
│   High NRE ($1M-$100M)             Low/no NRE                       │
│   Low unit cost ($1-$10)           Higher unit ($10-$1000)          │
│                                                                      │
│   NRE = Non-Recurring Engineering (design, masks, tooling)         │
│                                                                      │
│   Break-even: $\frac{NRE_{ASIC}}{Cost_{FPGA} - Cost_{ASIC}}$       │
│                                                                      │
│   Example:                                                          │
│   NRE_ASIC = $5M, Cost_FPGA = $50, Cost_ASIC = $5                 │
│   Break-even = $5M / ($50-$5) = 111,111 units                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter Index

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 10.1 | [FPGA Architecture](01-fpga-architecture.md) | LUTs, CLBs, routing, block RAM, DSP |
| 10.2 | [FPGA Design Flow](02-fpga-design-flow.md) | Synthesis, P&R, timing closure |
| 10.3 | [ASIC Design Flow](03-asic-design-flow.md) | RTL to GDSII, verification, signoff |
| 10.4 | [FPGA vs ASIC Comparison](04-fpga-asic-comparison.md) | Trade-offs, selection criteria |

---

## 🔑 Key Terminology

| Term | Definition |
|------|------------|
| LUT | Look-Up Table - basic logic element in FPGA |
| CLB | Configurable Logic Block - FPGA logic cluster |
| NRE | Non-Recurring Engineering costs |
| RTL | Register Transfer Level description |
| Netlist | Gate-level representation of design |
| GDSII | Graphic Database System II - mask layout format |

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [Unit 9: Testing](../09-Testing-Verification/README.md) | [Course Home](../README.md) | [FPGA Architecture →](01-fpga-architecture.md) |
