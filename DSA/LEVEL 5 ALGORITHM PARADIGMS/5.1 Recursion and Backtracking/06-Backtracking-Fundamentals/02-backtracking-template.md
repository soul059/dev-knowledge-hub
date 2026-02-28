# Chapter 2: The Backtracking Template

## Overview
Every backtracking problem follows the **same fundamental structure**. Mastering this template lets you solve a huge variety of problems by simply customizing the **state, choices, constraints, and goal**.

---

## The Universal Template

```
FUNCTION backtrack(state):
    IF isGoal(state):
        processSolution(state)
        RETURN                          ← or RETURN TRUE for single solution
    
    candidates = generateCandidates(state)
    
    FOR each candidate in candidates:
        IF isValid(candidate, state):   ← CONSTRAINT CHECK (pruning)
            applyCandidate(state)       ← MAKE the choice
            backtrack(state)            ← RECURSE
            removeCandidate(state)      ← UNDO the choice
```

---

## The 5 Components to Customize

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. STATE                                            │
│     What represents the current partial solution?    │
│     (array, string, set, grid, path, etc.)           │
│                                                      │
│  2. GOAL CONDITION                                   │
│     When is a solution complete?                     │
│     (filled all positions, reached target, etc.)     │
│                                                      │
│  3. CANDIDATES                                       │
│     What are the possible next choices?              │
│     (unused elements, valid moves, digits 1-9, etc.) │
│                                                      │
│  4. CONSTRAINTS                                      │
│     What makes a choice valid or invalid?            │
│     (no conflicts, within bounds, not visited, etc.) │
│                                                      │
│  5. MAKE/UNDO                                        │
│     How to apply and reverse a choice?               │
│     (add/remove, mark/unmark, place/remove, etc.)    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Template Variations

### Variation 1: Find ALL Solutions
```
FUNCTION backtrackAll(state, results):
    IF isGoal(state):
        results.add(copy(state))       ← IMPORTANT: add a COPY
        RETURN
    
    FOR each candidate in getCandidates(state):
        IF isValid(candidate, state):
            apply(candidate, state)
            backtrackAll(state, results)
            remove(candidate, state)
    
    RETURN results
```

### Variation 2: Find FIRST Solution
```
FUNCTION backtrackFirst(state):
    IF isGoal(state):
        RETURN TRUE                    ← Signal: solution found
    
    FOR each candidate in getCandidates(state):
        IF isValid(candidate, state):
            apply(candidate, state)
            IF backtrackFirst(state):  ← Stop if found
                RETURN TRUE
            remove(candidate, state)
    
    RETURN FALSE                       ← No solution in this branch
```

### Variation 3: Count Solutions
```
FUNCTION backtrackCount(state):
    IF isGoal(state):
        RETURN 1
    
    count = 0
    FOR each candidate in getCandidates(state):
        IF isValid(candidate, state):
            apply(candidate, state)
            count += backtrackCount(state)
            remove(candidate, state)
    
    RETURN count
```

### Variation 4: Find BEST Solution
```
FUNCTION backtrackBest(state, bestSoFar):
    IF isGoal(state):
        IF isBetter(state, bestSoFar):
            bestSoFar = copy(state)
        RETURN
    
    IF cannotBeatBest(state, bestSoFar):  ← BOUND check
        RETURN                             ← Branch & Bound pruning
    
    FOR each candidate in getCandidates(state):
        IF isValid(candidate, state):
            apply(candidate, state)
            backtrackBest(state, bestSoFar)
            remove(candidate, state)
```

---

## Applying the Template: Example

### Problem: Generate Subsets of {1, 2, 3}

```
Customizing the 5 components:

1. STATE:      current subset (list)
2. GOAL:       processed all elements (index == n)
3. CANDIDATES: include current element or exclude it
4. CONSTRAINTS: none (all subsets valid)
5. MAKE/UNDO:  add to list / remove from list

FUNCTION subsets(nums, index, current, results):
    IF index == len(nums):               ← GOAL
        results.add(copy(current))
        RETURN
    
    // Choice 1: INCLUDE nums[index]
    current.add(nums[index])             ← MAKE
    subsets(nums, index + 1, current, results)  ← RECURSE
    current.removeLast()                 ← UNDO
    
    // Choice 2: EXCLUDE nums[index]
    subsets(nums, index + 1, current, results)  ← RECURSE

Trace:
                      []
                    /     \
              [1]           []
             /   \        /    \
         [1,2]  [1]    [2]     []
         / \    / \    / \    / \
    [1,2,3][1,2][1,3][1] [2,3][2] [3] []

Results: [1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]  ✓
```

---

## Common State Representations

```
┌────────────────┬──────────────────────────────────┐
│ Problem        │ State Representation              │
├────────────────┼──────────────────────────────────┤
│ Subsets        │ List of chosen elements           │
│ Permutations   │ Array + swap positions            │
│ N-Queens       │ Board + column/diagonal sets      │
│ Sudoku         │ 9×9 grid                          │
│ Maze           │ Current position + visited set    │
│ Word Search    │ Grid position + visited cells     │
│ Combinations   │ List + start index                │
│ Parentheses    │ String + open/close counts        │
└────────────────┴──────────────────────────────────┘
```

---

## Template Checklist

```
Before coding, answer these questions:

□ What is my STATE? (data structure for partial solution)
□ What is my GOAL? (when to stop and record)
□ What are my CANDIDATES? (choices at each step)
□ What are my CONSTRAINTS? (when to prune)
□ How do I MAKE a choice? (modify state)
□ How do I UNDO a choice? (restore state exactly)
□ Do I need ALL solutions, FIRST, COUNT, or BEST?
□ Am I making a COPY when recording solutions?
```

---

## Summary Table

| Variation | Returns | Stops Early? | Use Case |
|-----------|---------|-------------|----------|
| Find All | List of solutions | No | Enumerate all valid configs |
| Find First | Boolean | Yes (on find) | Sudoku, puzzle solving |
| Count | Integer | No | Count valid arrangements |
| Find Best | Best solution | Yes (bound) | Optimization problems |

---

## Quick Revision Questions

1. **What are the 5 components** you must customize in the template?
2. **Why must you add a COPY** of the state to results (not the state itself)?
3. **How does Find First** differ from Find All in template structure?
4. **What is the Branch and Bound** variation used for?
5. **Apply the template** to the problem of generating all permutations.
6. **What happens** if you forget the UNDO step?

---

[← Previous: What Is Backtracking](01-what-is-backtracking.md) | [Next: State Space Tree →](03-state-space-tree.md)

[← Back to README](../README.md)
