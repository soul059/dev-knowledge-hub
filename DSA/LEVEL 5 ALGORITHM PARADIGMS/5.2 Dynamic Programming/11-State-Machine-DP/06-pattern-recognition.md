# Chapter 6: State Machine DP — Pattern Recognition

## 📋 Overview
This chapter synthesizes the state machine DP pattern across all stock problems and beyond. You'll learn to **identify, model, and solve** any state machine DP problem by recognizing the pattern elements.

---

## 🧠 The Universal Framework

```
Step 1: IDENTIFY STATES
  What are the distinct "modes" or "situations"?

Step 2: DRAW TRANSITIONS
  What events move between states?
  What is the cost/value of each transition?

Step 3: DEFINE DP VARIABLES
  One DP value per (step, state) pair.

Step 4: WRITE TRANSITIONS AS RECURRENCES
  dp[state][i] = optimize over incoming transitions

Step 5: SET BASE CASES
  What is the initial state?

Step 6: DETERMINE ANSWER
  Which state(s) at the final step give the answer?
```

---

## 📊 Complete Stock Problem Family

```
┌──────────────────────────────────────────────────────────────┐
│ UNIFIED STOCK TRADING FRAMEWORK                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Variables: hold, notHold  (+ optional: sold/cooldown)       │
│                                                              │
│  For each price:                                             │
│    notHold = max(notHold, hold + price [- fee])              │
│    hold    = max(hold, [buy_source] - price [- fee])         │
│                                                              │
│  Where buy_source varies:                                    │
│  ┌────────────────┬────────────────────────────┐             │
│  │ Variant        │ buy_source                 │             │
│  ├────────────────┼────────────────────────────┤             │
│  │ 1 transaction  │ 0 (always start fresh)     │             │
│  │ Unlimited      │ notHold (current profit)   │             │
│  │ K transactions │ dp[k-1][notHold]           │             │
│  │ Cooldown       │ notHold from 2 days ago    │             │
│  │ With fee       │ notHold (subtract fee)     │             │
│  └────────────────┴────────────────────────────┘             │
│                                                              │
│  Answer: notHold at end                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 🌐 Beyond Stock Trading

### Paint House
```
States: color of current house (R, G, B)
Transition: can't use same color as previous

cost[i][c] = paintCost[i][c] + min(cost[i-1][other colors])

  ┌───┐     ┌───┐     ┌───┐
  │ R │←───→│ G │←───→│ B │
  └───┘     └───┘     └───┘
    ↑─────────────────────↑

dp[i][R] = costs[i][R] + min(dp[i-1][G], dp[i-1][B])
dp[i][G] = costs[i][G] + min(dp[i-1][R], dp[i-1][B])
dp[i][B] = costs[i][B] + min(dp[i-1][R], dp[i-1][G])
```

### Ninja Training (Activity Selection)
```
States: which activity was done today (0, 1, 2)
Transition: can't do same activity two days in row

dp[i][a] = points[i][a] + max(dp[i-1][b]) for b ≠ a

Same structure as Paint House!
```

### String Parsing/Validation
```
States: parser positions in an automaton
Transition: match next character

Example: validate number format
  States: START, SIGN, INTEGER, DOT, DECIMAL, EXP, EXP_SIGN, EXP_NUM
  
  "Valid Number" problem = simulate state machine
  dp[i][state] = can string[0..i] reach this state?
```

### Keyboard with Special Keys
```
States: normal typing, selecting (Ctrl-A), copying (Ctrl-C)
Transition: press key → change state

dp[i][state] = max characters on screen at step i
```

---

## 🔑 State Machine DP Recognition Checklist

```
You're looking at State Machine DP when:

□ Problem has distinct phases/modes
□ Rules restrict which actions follow others
□ "Can't do X right after Y" constraints
□ Sequential decisions over time/steps
□ Each step has limited choices
□ Optimal accumulated value over all steps

Key phrases:
  "after selling, must wait..."  → cooldown state
  "at most K times"             → K states
  "can't choose same as last"   → exclusion constraint
  "pay a fee for each"          → transition cost
```

---

## 🧩 Template Code

```
function stateMachineDP(input, params):
    // Initialize states
    state_values = initial values for each state
    
    for step in input:
        new_state_values = {}
        
        for each state:
            for each valid transition from state:
                target = transition.target_state
                value = state_values[state] + transition.value(step)
                new_state_values[target] = optimize(
                    new_state_values[target], value
                )
        
        state_values = new_state_values
    
    return optimize(answer_states in state_values)
```

---

## 📊 Problem-to-State Mapping

| Problem | States | Key Transition Rule |
|---------|--------|-------------------|
| Stock I | hold, notHold | Buy from 0 only |
| Stock II | hold, notHold | Buy from notHold |
| Stock III | 4 states | Sequential: buy1→sell1→buy2→sell2 |
| Stock K | 2K states | Sequential K stages |
| Cooldown | hold, sold, rest | sold → rest → hold |
| Fee | hold, notHold | Subtract fee on sell |
| Paint House | K colors | Exclude previous color |
| Ninja | K activities | Exclude previous activity |
| Keyboard | typing modes | Special key sequences |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Pattern** | Discrete states + transitions + sequential steps |
| **Framework** | dp[step][state] = optimal value |
| **Stock unified** | buy_source changes per variant |
| **Beyond stocks** | Paint, activity selection, parsing |
| **Recognition** | Phase constraints, sequential decisions |
| **Template** | Initialize → for each step → for each transition |

---

## ❓ Quick Revision Questions

1. **What are the 6 steps to solve any state machine DP?**
2. **How does buy_source unify all stock variants?**
3. **Name three non-stock problems that use state machine DP.**
4. **How do you identify state machine DP from a problem statement?**
5. **What is the general time complexity of state machine DP?**
6. **How does Paint House map to the state machine framework?**

---

[← Previous: With Transaction Fee](05-with-fee.md) | [Next Unit: Bitmask DP →](../12-Bitmask-DP/01-tsp-bitmask.md)

[← Back to README](../README.md)
