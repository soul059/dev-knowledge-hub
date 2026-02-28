# Chapter 10.3: ASIC Design Flow

## 📋 Chapter Overview

The **ASIC design flow** transforms RTL into physical layouts ready for fabrication. This chapter covers the complete flow from specification to GDSII, including the verification and signoff stages.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the complete RTL-to-GDSII flow
- Identify the major design stages and their outputs
- Understand physical design and verification steps
- Know the signoff requirements before tapeout

---

## 10.3.1 ASIC Design Flow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ASIC DESIGN FLOW                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌────────────────────────────────────────────────────────────┐    │
│   │                    FRONT-END DESIGN                        │    │
│   │                                                            │    │
│   │   Specification ──► Architecture ──► RTL Design           │    │
│   │                                          │                 │    │
│   │                                          ▼                 │    │
│   │                               Functional Verification      │    │
│   │                              (Simulation, Formal)          │    │
│   │                                          │                 │    │
│   │                                          ▼                 │    │
│   │                                     Synthesis              │    │
│   │                              (RTL → Gate Netlist)          │    │
│   └────────────────────────────────────────────┬───────────────┘    │
│                                                │                    │
│   ┌────────────────────────────────────────────▼───────────────┐    │
│   │                    BACK-END DESIGN                         │    │
│   │                                                            │    │
│   │   Floorplanning ──► Placement ──► CTS ──► Routing          │    │
│   │        │                                       │           │    │
│   │        ▼                                       ▼           │    │
│   │   Power Planning                         DRC/LVS          │    │
│   │                                              │             │    │
│   │                                              ▼             │    │
│   │                                         Signoff            │    │
│   │                                        (Timing, Power)     │    │
│   │                                              │             │    │
│   │                                              ▼             │    │
│   │                                          GDSII             │    │
│   │                                       (Tapeout)            │    │
│   └────────────────────────────────────────────────────────────┘    │
│                                                                      │
│   Timeline: 6-18 months typical                                    │
│   Cost: $1M-$100M+ depending on node                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.3.2 Front-End Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FRONT-END STAGES                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   1. SPECIFICATION                                                  │
│   ────────────────                                                  │
│   • Feature requirements                                           │
│   • Performance targets (MHz, MIPS, throughput)                   │
│   • Power budget                                                   │
│   • Area constraints                                               │
│   • Interface definitions                                          │
│                                                                      │
│   2. ARCHITECTURE                                                   │
│   ───────────────                                                   │
│   • Block diagram                                                  │
│   • Data path design                                               │
│   • Memory architecture                                            │
│   • Interface protocols                                            │
│   • Power domains                                                  │
│                                                                      │
│   3. RTL DESIGN                                                    │
│   ─────────────                                                     │
│   • Verilog/VHDL/SystemVerilog coding                             │
│   • Design partitioning                                            │
│   • Clock domain planning                                          │
│   • Reset strategy                                                 │
│                                                                      │
│   4. FUNCTIONAL VERIFICATION                                       │
│   ──────────────────────────                                        │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                  Verification Methods                       │   │
│   │                                                             │   │
│   │   ┌────────────┐   ┌────────────┐   ┌────────────┐        │   │
│   │   │Simulation  │   │   Formal   │   │ Emulation/ │        │   │
│   │   │            │   │Verification│   │Prototyping │        │   │
│   │   ├────────────┤   ├────────────┤   ├────────────┤        │   │
│   │   │• UVM/OVM   │   │• Assertions│   │• FPGA proto│        │   │
│   │   │• Directed  │   │• Model chk │   │• Hardware  │        │   │
│   │   │• Random    │   │• Equiv chk │   │  emulator  │        │   │
│   │   │• Coverage  │   │• CDC check │   │• Speed: MHz│        │   │
│   │   │            │   │            │   │            │        │   │
│   │   │Speed: kHz  │   │ Exhaustive │   │ Real-time  │        │   │
│   │   └────────────┘   └────────────┘   └────────────┘        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│   Coverage targets: >95% functional, 100% code                     │
│                                                                      │
│   5. SYNTHESIS                                                     │
│   ───────────────                                                   │
│   RTL ──► Generic gates ──► Technology-mapped netlist             │
│                                                                      │
│   Inputs:  RTL, constraints (SDC), library (.lib)                 │
│   Output:  Gate-level netlist (.v), timing report                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.3.3 Physical Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PHYSICAL DESIGN FLOW                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   1. FLOORPLANNING                                                  │
│   ────────────────                                                  │
│   Define chip area and arrange major blocks                        │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────────────────────────┐ │   │
│   │  │   IO    │  │   IO    │  │           IO                │ │   │
│   │  │  Ring   │  │  Ring   │  │          Ring               │ │   │
│   │  ├─────────┼──┼─────────┼──┼─────────────────────────────┤ │   │
│   │  │         │  │         │  │                             │ │   │
│   │  │   CPU   │  │  Cache  │  │         Memory              │ │   │
│   │  │  Core   │  │   L1    │  │        Subsystem            │ │   │
│   │  │         │  │         │  │                             │ │   │
│   │  ├─────────┼──┼─────────┼──┼─────────────────────────────┤ │   │
│   │  │         │  │         │  │                             │ │   │
│   │  │ Periph  │  │   Bus   │  │        I/O Block            │ │   │
│   │  │         │  │         │  │                             │ │   │
│   │  └─────────┴──┴─────────┴──┴─────────────────────────────┘ │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│                                                                      │
│   2. POWER PLANNING (Power Grid)                                   │
│   ──────────────────────────────                                    │
│                                                                      │
│   VDD straps (horizontal)                                          │
│   ═══════════════════════════════════════════════                   │
│   ║     ║     ║     ║     ║     ║     ║     ║    ← VDD rings       │
│   ═══════════════════════════════════════════════                   │
│   ║     ║     ║     ║     ║     ║     ║     ║                      │
│   ═══════════════════════════════════════════════                   │
│   ║     ║     ║     ║     ║     ║     ║     ║                      │
│   ═══════════════════════════════════════════════                   │
│       VSS straps (vertical)                                         │
│                                                                      │
│   • Ring around chip edge                                          │
│   • Straps across die for distribution                            │
│   • IR drop analysis: < 5% VDD typically                          │
│                                                                      │
│                                                                      │
│   3. PLACEMENT                                                     │
│   ────────────                                                      │
│   Assign standard cells to rows                                    │
│                                                                      │
│   ┌────┬────┬────┬────┬────┬────┬────┬────┐  Standard             │
│   │Cell│Cell│Cell│Cell│Cell│Cell│Cell│Cell│  cell rows            │
│   ├────┼────┼────┼────┼────┼────┼────┼────┤                        │
│   │Cell│Cell│Cell│    │    │Cell│Cell│Cell│                        │
│   ├────┼────┼────┼────┼────┼────┼────┼────┤                        │
│   │Cell│Cell│    │Cell│Cell│Cell│    │Cell│                        │
│   └────┴────┴────┴────┴────┴────┴────┴────┘                        │
│                                                                      │
│   Optimization: timing, congestion, power                          │
│                                                                      │
│                                                                      │
│   4. CLOCK TREE SYNTHESIS (CTS)                                    │
│   ─────────────────────────────                                     │
│   Build balanced clock distribution                                │
│                                                                      │
│                   CLK_IN                                            │
│                     │                                               │
│                   ┌─┴─┐                                            │
│                   │BUF│                                            │
│                   └─┬─┘                                            │
│           ┌─────────┼─────────┐                                    │
│         ┌─┴─┐     ┌─┴─┐     ┌─┴─┐                                 │
│         │BUF│     │BUF│     │BUF│                                 │
│         └─┬─┘     └─┬─┘     └─┬─┘                                 │
│        ┌──┴──┐   ┌──┴──┐   ┌──┴──┐                                │
│        │     │   │     │   │     │                                │
│       FF    FF  FF    FF  FF    FF                                │
│                                                                      │
│   Goal: Minimize clock skew (< 100ps typical)                      │
│                                                                      │
│                                                                      │
│   5. ROUTING                                                       │
│   ──────────                                                        │
│   • Global routing (path planning)                                │
│   • Detail routing (metal layer assignment)                       │
│   • Signal integrity (crosstalk, EM)                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.3.4 Verification and Signoff

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIGNOFF VERIFICATION                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   All checks must pass before tapeout!                             │
│                                                                      │
│   1. PHYSICAL VERIFICATION                                         │
│   ────────────────────────                                          │
│                                                                      │
│   DRC (Design Rule Check):                                         │
│   • Minimum width                                                  │
│   • Minimum spacing                                                │
│   • Enclosure rules                                                │
│   • Density rules                                                  │
│   • Antenna rules                                                  │
│                                                                      │
│   LVS (Layout vs Schematic):                                       │
│   • Netlist extraction from layout                                │
│   • Compare to gate-level netlist                                 │
│   • Must match exactly (0 errors)                                 │
│                                                                      │
│   ERC (Electrical Rule Check):                                     │
│   • Floating nodes                                                 │
│   • Short circuits                                                 │
│   • Well ties                                                      │
│                                                                      │
│                                                                      │
│   2. TIMING SIGNOFF                                                │
│   ─────────────────                                                 │
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │             Multi-Corner Multi-Mode (MCMM)                   │  │
│   │                                                              │  │
│   │   Corners:                 Modes:                           │  │
│   │   • SS (slow-slow)         • Functional                     │  │
│   │   • FF (fast-fast)         • Test (scan)                    │  │
│   │   • TT (typical)           • Low power                      │  │
│   │   • SF, FS (skewed)        • Different frequencies          │  │
│   │                                                              │  │
│   │   Temperature: -40°C to 125°C                               │  │
│   │   Voltage: ±10% nominal                                     │  │
│   │                                                              │  │
│   │   All combinations must pass timing!                        │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│   OCV (On-Chip Variation):                                         │
│   • Derate paths for local variation                              │
│   • AOCV/POCV for more accuracy                                   │
│                                                                      │
│                                                                      │
│   3. POWER SIGNOFF                                                 │
│   ────────────────                                                  │
│                                                                      │
│   IR Drop Analysis:                                                │
│   • Static IR (average current)                                   │
│   • Dynamic IR (switching transients)                             │
│   • Voltage drop map                                              │
│                                                                      │
│   ┌─────────────────────────────────────────┐                       │
│   │    VDD = 1.0V            │    │    │    │                       │
│   │   ┌────────────────────┐ │Max drop     │                       │
│   │   │░░░░░░░░░░░░░░░░░░░░│ │= 50mV      │                       │
│   │   │░░░░█████░░░░░░░░░░░│◄┤(5%)        │                       │
│   │   │░░░░░░░░░░░░░░░░░░░░│ │             │                       │
│   │   └────────────────────┘ │             │                       │
│   │   ░ = low drop, █ = hot spot           │                       │
│   └─────────────────────────────────────────┘                       │
│                                                                      │
│   EM (Electromigration):                                           │
│   • Current density limits                                         │
│   • Lifetime estimation                                            │
│                                                                      │
│                                                                      │
│   4. FINAL CHECKS                                                  │
│   ───────────────                                                   │
│   • Metal fill (density balance)                                  │
│   • Dummy insertion                                                │
│   • Chip finishing (seal ring, corners)                           │
│   • GDSII output and verification                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.3.5 ASIC Design Tools

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EDA TOOL LANDSCAPE                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ Stage            │ Cadence      │ Synopsys    │ Mentor/Siemens│ │
│   ├──────────────────────────────────────────────────────────────┤  │
│   │ RTL Simulation   │ Xcelium      │ VCS         │ QuestaSim     │ │
│   │ Formal Verify    │ JasperGold   │ VC Formal   │ Questa Formal │ │
│   │ Synthesis        │ Genus        │ Design Comp │ -             │ │
│   │ P&R              │ Innovus      │ IC Compiler │ -             │ │
│   │ Static Timing    │ Tempus       │ PrimeTime   │ -             │ │
│   │ Physical Verify  │ Pegasus      │ IC Validator│ Calibre       │ │
│   │ Power Analysis   │ Voltus       │ PrimePower  │ -             │ │
│   │ Parasitic Extr   │ Quantus      │ StarRC      │ -             │ │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                                                                      │
│   Typical tool flow:                                                │
│                                                                      │
│        RTL                                                          │
│         │                                                           │
│         ▼                                                           │
│   ┌───────────┐    ┌───────────┐    ┌───────────┐                  │
│   │ Synthesis │───►│   P&R     │───►│   DRC     │                  │
│   │  (Genus)  │    │ (Innovus) │    │ (Calibre) │                  │
│   └───────────┘    └─────┬─────┘    └─────┬─────┘                  │
│                          │                │                         │
│                          ▼                ▼                         │
│                    ┌───────────┐    ┌───────────┐                  │
│                    │   STA     │    │   LVS     │                  │
│                    │(PrimeTime)│    │ (Calibre) │                  │
│                    └───────────┘    └───────────┘                  │
│                          │                │                         │
│                          └───────┬────────┘                        │
│                                  ▼                                  │
│                            ┌──────────┐                            │
│                            │  GDSII   │                            │
│                            │ (Tapeout)│                            │
│                            └──────────┘                            │
│                                                                      │
│                                                                      │
│   License costs: $100K - $1M+ per seat per year                   │
│   Server farms: 100s-1000s of CPU cores for large designs         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10.3.6 Tapeout and Manufacturing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TAPEOUT TO SILICON                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   GDSII file                                                        │
│      │                                                              │
│      ▼                                                              │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    FOUNDRY                                  │   │
│   │                                                             │   │
│   │   1. Mask generation (reticle creation)                    │   │
│   │      • One mask per layer (40-80 masks)                    │   │
│   │      • $100K-$1M per mask set                              │   │
│   │                                                             │   │
│   │   2. Wafer fabrication                                     │   │
│   │      • 300mm wafers                                        │   │
│   │      • 400-1000 process steps                              │   │
│   │      • 2-4 months manufacturing                            │   │
│   │                                                             │   │
│   │   3. Wafer test (die sort)                                 │   │
│   │                                                             │   │
│   │   4. Packaging                                             │   │
│   │      • Wire bond or flip-chip                              │   │
│   │      • BGA, QFN, etc.                                      │   │
│   │                                                             │   │
│   │   5. Final test                                            │   │
│   │                                                             │   │
│   └─────────────────────────────────────────────────────────────┘   │
│      │                                                              │
│      ▼                                                              │
│   Packaged chips ready for system integration                      │
│                                                                      │
│                                                                      │
│   Cost breakdown (advanced node):                                   │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ Item                    │ Cost (7nm example)                │  │
│   ├──────────────────────────────────────────────────────────────┤  │
│   │ Mask set                │ $5-10M                            │  │
│   │ Design (NRE)            │ $50-100M                          │  │
│   │ Wafer (300mm)           │ $10,000-15,000                    │  │
│   │ Dies per wafer          │ 100-1000 (size dependent)         │  │
│   │ Cost per good die       │ $10-150                           │  │
│   │ Packaging               │ $1-20                             │  │
│   │ Test                    │ $0.50-5                           │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│   Major foundries: TSMC, Samsung, Intel, GlobalFoundries           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Stage | Main Activity | Key Output | Duration |
|-------|---------------|------------|----------|
| Specification | Define requirements | Spec document | 1-2 months |
| RTL Design | Code the design | Verilog/VHDL | 2-6 months |
| Verification | Validate functionality | Coverage reports | 3-6 months |
| Synthesis | Map to gates | Netlist | 1-2 weeks |
| Physical Design | Layout creation | GDSII | 2-4 months |
| Signoff | Final checks | Tapeout approval | 2-4 weeks |

---

## ❓ Quick Revision Questions

1. **What are the main stages of the ASIC design flow?**

2. **What is the difference between front-end and back-end design?**

3. **What do DRC and LVS check for?**

4. **What is MCMM and why is it important for timing signoff?**

5. **Name three major EDA vendors and one tool from each.**

6. **Why is mask cost a significant factor in ASIC economics?**

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [FPGA Design Flow](02-fpga-design-flow.md) | [Unit 10 Home](README.md) | [FPGA vs ASIC Comparison →](04-fpga-asic-comparison.md) |
