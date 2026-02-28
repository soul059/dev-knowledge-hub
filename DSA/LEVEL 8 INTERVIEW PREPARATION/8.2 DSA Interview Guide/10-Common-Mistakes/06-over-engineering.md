# Chapter 10.6: Over-Engineering

```
╔══════════════════════════════════════════════════════════╗
║          OVER-ENGINEERING                                ║
║   When doing too much hurts more than doing too little   ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Over-engineering in interviews means building a more complex solution than needed: unnecessary abstractions, premature optimization, excessive error handling, or designing for scenarios that don't exist. **Simplicity wins in interviews.**

---

## What Over-Engineering Looks Like

```
EXAMPLE: Two Sum (find two numbers that add to target)

OVER-ENGINEERED:
  class TwoSumSolver:
      def __init__(self, strategy='hash'):
          self.strategy = strategy
          self.logger = Logger()
          
      def solve(self, nums, target):
          if self.strategy == 'hash':
              return self._hash_approach(nums, target)
          elif self.strategy == 'sort':
              return self._sort_approach(nums, target)
          
      def _hash_approach(self, nums, target):
          self.logger.log("Starting hash approach")
          seen = {}
          for i, num in enumerate(nums):
              # ...20 more lines...

CORRECT (Interview-appropriate):
  def two_sum(nums, target):
      seen = {}
      for i, num in enumerate(nums):
          comp = target - num
          if comp in seen:
              return [seen[comp], i]
          seen[num] = i
      return []

  DIFFERENCE: 5 lines vs 25 lines
  RESULT: Same. But one took 5 min, the other 20 min.
```

---

## Types of Over-Engineering

```
TYPE 1: UNNECESSARY ABSTRACTION
  ✗ Creating classes when a function suffices
  ✗ Building inheritance hierarchies
  ✗ Creating factory patterns
  ✓ Use the simplest structure that works

TYPE 2: PREMATURE OPTIMIZATION
  ✗ Optimizing before having a working solution
  ✗ Using complex data structures for marginal gains
  ✗ Hand-rolling algorithms that built-ins handle
  ✓ Get working code first, then optimize if asked

TYPE 3: EXCESSIVE ERROR HANDLING
  ✗ try/except around every operation
  ✗ Type checking every parameter
  ✗ Handling scenarios the problem doesn't mention
  ✓ Handle what the interviewer specifies, mention rest

TYPE 4: FUTURE-PROOFING
  ✗ "What if we need to support strings later?"
  ✗ Making solution generic for unseen inputs
  ✗ Adding configuration parameters
  ✓ Solve the exact problem given

TYPE 5: OVER-DECOMPOSITION
  ✗ 10 helper functions for a 20-line problem
  ✗ Each helper is 2-3 lines
  ✗ Hard to follow the flow
  ✓ Extract helpers only when logic is reused or complex
```

---

## How to Know If You're Over-Engineering

```
RED FLAGS:
  ☐ Your solution is 3x longer than typical solutions
  ☐ You're spending time on features not in the problem
  ☐ You're creating classes when functions work fine
  ☐ You're optimizing before code works
  ☐ You can't explain why a piece of code is needed
  ☐ You're handling error cases the problem doesn't mention

SELF-CHECK:
  "If I removed this code, would the solution still
   solve the exact problem given?"
  If yes → it's over-engineered.
```

---

## The Right Level of Complexity

```
INTERVIEW SWEET SPOT:

  TOO SIMPLE          │    JUST RIGHT         │  OVER-ENGINEERED
  ────────────────────┼───────────────────────┼──────────────────
  No function name     │ Clear function sig    │ Class hierarchy
  Single-letter vars   │ Descriptive names     │ Enterprise naming
  No error handling    │ Key edge case guard   │ Try/except chain
  No comments          │ Brief inline comments │ Docstrings + types
  No testing           │ Trace key examples    │ Unit test framework

  AIM FOR THE MIDDLE COLUMN.
```

---

## What Interviewers Actually Want

```
INTERVIEWER PRIORITIES:
  1. Correct solution to the problem given
  2. Clean, readable code
  3. Clear communication of approach
  4. Awareness of complexity
  5. Test coverage of key cases

THEY DO NOT WANT:
  ✗ Production-grade error handling
  ✗ Extensive documentation
  ✗ Design pattern demonstrations
  ✗ Generic/reusable frameworks
  ✗ Performance micro-optimizations
```

---

## Summary Table

| Over-Engineering Type | Example | Fix |
|----------------------|---------|-----|
| Unnecessary abstraction | Classes for simple problems | Use functions |
| Premature optimization | Skip brute force → complex algo | Brute force first |
| Excessive error handling | Try/except everything | Guard only key edges |
| Future-proofing | Generalize beyond scope | Solve exact problem |
| Over-decomposition | 10 tiny helper functions | Inline unless complex |

---

## Quick Revision Questions

1. **What is over-engineering in an interview context?**
   - Building more complexity than needed: unnecessary classes, excessive error handling, premature optimization, or solving problems that weren't asked

2. **How do you know if you're over-engineering?**
   - If you can remove code and the solution still solves the exact problem given, that code is over-engineered

3. **Should you use classes in interview solutions?**
   - Almost never — functions are sufficient for 95% of interview problems. Only use classes if the problem explicitly requires it (like design problems)

4. **What's premature optimization?**
   - Trying to build the optimal solution before having a working brute force. Always get working code first, then optimize if asked

5. **How much error handling is appropriate?**
   - Handle key edge cases the problem mentions (empty input, not found). Verbally mention other considerations without coding them

6. **What level of complexity do interviewers actually want?**
   - Clean, correct, readable code with descriptive names, key edge case guards, and brief comments — the middle ground between too simple and over-engineered

---

[← Previous: Not Testing Solution](05-not-testing-solution.md) | [Next: Unit 11 — Preparation Checklist →](../11-Day-of-Interview/01-preparation-checklist.md)

---
[← Back to README](../README.md)
