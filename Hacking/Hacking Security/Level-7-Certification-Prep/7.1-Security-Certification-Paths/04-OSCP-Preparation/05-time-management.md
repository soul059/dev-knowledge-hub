# Unit 4: OSCP Preparation — Topic 5: Time Management

## Overview

The OSCP exam gives you approximately 24 hours to compromise machines worth enough points to pass. Effective **time management** is critical — many technically capable candidates fail due to poor time allocation, not lack of skill. This topic covers exam time strategies.

---

## 1. Exam Time Breakdown

```
24-HOUR EXAM TIMELINE:

  ┌──────────────────────────────────────┐
  │  Hour 0-1:   Initial Scans          │
  │              All machines Nmap       │
  │              Quick wins check        │
  │                                      │
  │  Hour 1-6:   First Attack Phase     │
  │              Target easiest machines │
  │              Get first 20-40 points  │
  │                                      │
  │  Hour 6-8:   BREAK (mandatory)      │
  │              Eat, rest, review       │
  │              Reassess strategy       │
  │                                      │
  │  Hour 8-16:  Main Attack Phase      │
  │              Target remaining        │
  │              AD set if not started   │
  │              Aim for 60-70 points    │
  │                                      │
  │  Hour 16-18: Second Break           │
  │              Sleep 1-2 hours         │
  │              Review progress         │
  │                                      │
  │  Hour 18-23: Final Push             │
  │              Attack remaining        │
  │              Verify all proofs       │
  │              Fill documentation gaps │
  │                                      │
  │  Hour 23-24: Wrap Up                │
  │              Final screenshots       │
  │              Verify all proofs       │
  │              Start report prep       │
  └──────────────────────────────────────┘

POINTS MILESTONES:
  Hour 4:  20 points minimum
  Hour 8:  40 points target
  Hour 12: 60 points target
  Hour 16: 70+ points (passing)
  Hour 24: Maximum points
```

---

## 2. Machine Prioritization

```
ATTACK ORDER STRATEGY:

OPTION A: EASY FIRST (Recommended)
  1. Scan all machines simultaneously
  2. Start with machine that looks easiest
  3. Get quick wins for confidence
  4. Build momentum
  5. Tackle AD set when warmed up

OPTION B: AD SET FIRST
  1. Scan all machines
  2. Start AD set immediately
  3. 40 points is biggest chunk
  4. All-or-nothing (risky but efficient)
  5. Do standalones after

OPTION C: HIGHEST VALUE FIRST
  1. Scan all machines
  2. Start with AD set (40 pts)
  3. Then standalones by perceived difficulty
  4. Maximize points per hour

RECOMMENDED APPROACH:
  1. Start all Nmap scans simultaneously
  2. While scans run, review results as they come
  3. Identify the "low-hanging fruit"
  4. Attack easiest standalone first (20 pts)
  5. Move to second standalone (40 pts total)
  6. Tackle AD set (up to 80 pts)
  7. Final standalone (up to 100 pts)

TIME LIMITS PER MACHINE:
  Standalone: 3-4 hours max
  AD Set: 6-8 hours max
  
  If stuck for 2 hours with no progress:
  → Move to different machine
  → Return with fresh perspective later
```

---

## 3. Managing Energy and Focus

```
PHYSICAL AND MENTAL PREPARATION:

BEFORE EXAM:
  → Get good sleep (2 nights before)
  → Start exam when you're most alert
  → Common start times: 8AM-10AM
  → Have exam area prepared
  → Stock food and drinks

DURING EXAM:
  Food Strategy:
  → Pre-prepare meals (don't cook)
  → Easy snacks: nuts, fruit, energy bars
  → Stay hydrated (water, not just caffeine)
  → Avoid sugar crashes
  → Heavy meals = sleepy (eat light)

  Break Schedule:
  → Every 2-3 hours: 5-10 min stand/stretch
  → Hour 6-8: 30-60 min meal break
  → Hour 14-16: Power nap (set alarm!)
  → Don't skip breaks — they help
  
  Mental Strategies:
  → Stay calm when stuck
  → Don't panic if behind schedule
  → Fresh eyes after break often find solutions
  → "Try Harder" doesn't mean don't rest
  → Celebrate small victories (each flag)
  
  Caffeine Management:
  → Don't overdo early
  → Space out coffee/tea
  → Last caffeine by hour 16
  → If sleeping, you need actual rest

WHEN TO SLEEP:
  → You have enough points (70+): Sleep well
  → You're close (50-60 pts): 2-hour power nap
  → You're behind (<40 pts): 1-hour minimum
  → NEVER skip sleep entirely — it hurts performance
```

---

## 4. Dealing with Stuck Situations

```
WHEN YOU'RE STUCK:

30-MINUTE STUCK PROTOCOL:
  → Step back and re-enumerate
  → Re-read your notes
  → Try a different approach
  → Check if you missed something

1-HOUR STUCK PROTOCOL:
  → Run enumeration again with different tools
  → Try different wordlists
  → Check ALL found credentials everywhere
  → Look at the problem from a different angle
  → Read service documentation

2-HOUR STUCK PROTOCOL:
  → Take a 15-minute break
  → Move to a different machine
  → Come back later with fresh eyes
  → Review your methodology checklist

COMMON THINGS PEOPLE MISS:
  → Different web directories not checked
  → Source code comments
  → robots.txt and sitemap.xml
  → Virtual hosts
  → UDP ports
  → Version-specific exploits
  → Default credentials
  → Password reuse across services
  → Hidden parameters in forms
  → File permissions on writable dirs

MINDSET:
  → "The answer is there, I just haven't 
     found it yet"
  → Every machine IS solvable
  → Re-enumerate, don't just re-exploit
  → The vulnerability is usually in the
     enumeration data you already have
  → Check everything twice
```

---

## 5. Post-Exam Time Management

```
REPORT WRITING TIMELINE (24 hours):

HOUR 0-2: ORGANIZE
  → Sort all screenshots by machine
  → Review all notes for completeness
  → Identify any missing documentation
  → Open report template

HOUR 2-6: WRITE MACHINE SECTIONS
  → Start with best-documented machine
  → Copy/format commands and output
  → Insert screenshots
  → Write exploitation steps
  → Verify steps are reproducible

HOUR 6-8: BREAK
  → Rest your eyes and mind
  → Eat a proper meal
  → Brief review of progress

HOUR 8-14: COMPLETE ALL SECTIONS
  → Finish remaining machines
  → Write executive summary
  → Write methodology section
  → Add remediation recommendations

HOUR 14-18: REVIEW AND POLISH
  → Read entire report start to finish
  → Check all screenshots present
  → Verify step reproducibility
  → Fix formatting issues
  → Check page numbers and TOC

HOUR 18-20: FINAL REVIEW
  → Fresh read-through
  → Grammar and spelling check
  → Export to PDF
  → Verify PDF looks correct
  → Check file size

HOUR 20-22: SUBMIT
  → Upload to OffSec portal
  → Verify upload successful
  → Save confirmation
  → Keep backup copy

HOUR 22-24: BUFFER
  → For technical issues
  → Last-minute fixes
  → DO NOT USE ALL 24 HOURS
```

---

## Summary Table

| Phase | Time Allocation | Goal |
|-------|----------------|------|
| Initial Scans | Hour 0-1 | Map all targets |
| First Attacks | Hour 1-6 | 20-40 points |
| Main Phase | Hour 8-16 | 60-70 points |
| Final Push | Hour 18-23 | Maximum points |
| Report Write | Post-exam 24h | Professional report |

---

## Revision Questions

1. What is the recommended time limit per standalone machine?
2. When should you move to a different machine if stuck?
3. What is the recommended approach for break scheduling?
4. How should you prioritize machines at the start of the exam?
5. What is the optimal report writing timeline?

---

*Previous: [04-report-writing.md](04-report-writing.md) | Next: [06-practice-platforms.md](06-practice-platforms.md)*

---

*[Back to README](../README.md)*
