# Chapter 3.6: Sorting and Searching Algorithms

## 📚 Chapter Overview

Sorting and searching are fundamental algorithms in computer science. This chapter covers implementation of common sorting algorithms (Bubble Sort, Selection Sort) and searching techniques (Linear Search, Binary Search) in 8085 assembly language.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Implement Bubble Sort for ascending/descending order
- Implement Selection Sort algorithm
- Perform Linear Search in an array
- Implement Binary Search for sorted arrays
- Understand algorithm complexity and trade-offs

---

## 1. Sorting Fundamentals

### 1.1 Sorting Concepts

```
SORTING TERMINOLOGY:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ASCENDING ORDER:  Smallest → Largest                          │
│   Example: 12, 25, 38, 45, 89                                   │
│                                                                  │
│   DESCENDING ORDER: Largest → Smallest                          │
│   Example: 89, 45, 38, 25, 12                                   │
│                                                                  │
│   KEY OPERATIONS:                                                │
│   • Compare: Determine if A > B, A < B, or A = B                │
│   • Swap: Exchange positions of two elements                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

SWAP OPERATION:

; Exchange M[HL] with M[DE]
        MOV A, M        ; A = M[HL]
        MOV B, A        ; Save in B
        LDAX D          ; A = M[DE]
        MOV M, A        ; M[HL] = M[DE]
        MOV A, B        ; A = original M[HL]
        STAX D          ; M[DE] = original M[HL]
```

---

## 2. Bubble Sort

### 2.1 Algorithm Concept

```
BUBBLE SORT CONCEPT:

Compare adjacent elements, swap if out of order.
Largest element "bubbles up" to end after each pass.

EXAMPLE - Ascending Order:
───────────────────────────────────────────────────────────────────

Initial:    [45] [12] [89] [25] [38]

Pass 1:
  Compare 45,12: swap   [12] [45] [89] [25] [38]
  Compare 45,89: ok     [12] [45] [89] [25] [38]
  Compare 89,25: swap   [12] [45] [25] [89] [38]
  Compare 89,38: swap   [12] [45] [25] [38] [89] ◄ 89 in place

Pass 2:
  Compare 12,45: ok     [12] [45] [25] [38] [89]
  Compare 45,25: swap   [12] [25] [45] [38] [89]
  Compare 45,38: swap   [12] [25] [38] [45] [89] ◄ 45 in place

Pass 3:
  Compare 12,25: ok     [12] [25] [38] [45] [89]
  Compare 25,38: ok     [12] [25] [38] [45] [89] ◄ 38 in place

Pass 4:
  Compare 12,25: ok     [12] [25] [38] [45] [89] ◄ 25 in place

Result:     [12] [25] [38] [45] [89]


COMPLEXITY:
───────────────────────────────────────────────────────────────────

For n elements:
• Passes: n-1
• Comparisons per pass: n-1, n-2, ..., 1
• Total comparisons: n(n-1)/2 = O(n²)
• Best case: O(n) with early termination (already sorted)
```

### 2.2 Bubble Sort - Ascending Order

```
PROGRAM: Bubble Sort (Ascending Order)

; Input:  Array at 2000H, count at 2010H
; Output: Array sorted in ascending order at same location

        ORG 0100H
        
        LDA 2010H       ; Get count (n)
        DCR A           ; Outer loop count = n-1
        MOV E, A        ; E = outer loop counter
        
OUTER:  MOV A, E        ; Inner loop count = E
        MOV D, A        ; D = inner loop counter
        LXI H, 2000H    ; Point to start of array
        
INNER:  MOV A, M        ; Get current element
        INX H           ; Point to next element
        CMP M           ; Compare with next
        JC NOSWAP       ; If current < next, no swap needed
        JZ NOSWAP       ; If equal, no swap needed
        
        ; Swap required (current > next)
        MOV B, M        ; B = next element
        MOV M, A        ; Store current in next position
        DCX H           ; Point back to current position
        MOV M, B        ; Store next (smaller) in current
        INX H           ; Point forward again
        
NOSWAP: DCR D           ; Decrement inner counter
        JNZ INNER       ; Continue inner loop
        
        DCR E           ; Decrement outer counter
        JNZ OUTER       ; Continue outer loop
        
        HLT

; TRACE for [45, 12, 89]:
; E=2, Pass 1:
;   D=2: Compare 45,12: 45>12, swap → [12,45,89]
;   D=1: Compare 45,89: 45<89, no swap
; E=1, Pass 2:
;   D=1: Compare 12,45: 12<45, no swap
; Result: [12, 45, 89]
```

### 2.3 Bubble Sort - Descending Order

```
PROGRAM: Bubble Sort (Descending Order)

; Only change: swap condition
; For descending, swap if current < next

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A           ; Outer loop count
        MOV E, A
        
OUTER:  MOV A, E
        MOV D, A
        LXI H, 2000H
        
INNER:  MOV A, M        ; Get current element
        INX H           ; Point to next
        CMP M           ; Compare with next
        JNC NOSWAP      ; If current ≥ next, no swap (for descending)
        
        ; Swap required (current < next)
        MOV B, M        ; B = next element
        MOV M, A        ; Store current in next position
        DCX H           ; Point back
        MOV M, B        ; Store next (larger) in current
        INX H           ; Point forward
        
NOSWAP: DCR D
        JNZ INNER
        
        DCR E
        JNZ OUTER
        
        HLT

; The only difference is JNC instead of JC in comparison
; JNC: Jump if A ≥ M[HL] (no carry), meaning no swap needed
```

### 2.4 Optimized Bubble Sort with Flag

```
PROGRAM: Bubble Sort with early termination

; If no swaps occur in a pass, array is sorted

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A
        MOV E, A        ; E = max passes
        
OUTER:  MOV A, E
        MOV D, A        ; D = inner loop counter
        LXI H, 2000H
        MVI C, 00H      ; C = swap flag (0 = no swap)
        
INNER:  MOV A, M
        INX H
        CMP M
        JC NOSWAP
        JZ NOSWAP
        
        ; Swap
        MOV B, M
        MOV M, A
        DCX H
        MOV M, B
        INX H
        MVI C, 01H      ; Set swap flag
        
NOSWAP: DCR D
        JNZ INNER
        
        MOV A, C        ; Check swap flag
        ORA A
        JZ DONE         ; No swaps = sorted, exit early
        
        DCR E
        JNZ OUTER
        
DONE:   HLT

; If array is already sorted, terminates after first pass
; Best case: O(n), Worst case: O(n²)
```

---

## 3. Selection Sort

### 3.1 Algorithm Concept

```
SELECTION SORT CONCEPT:

Find minimum element, place at beginning.
Repeat for remaining unsorted portion.

EXAMPLE - Ascending Order:
───────────────────────────────────────────────────────────────────

Initial:    [45] [12] [89] [25] [38]
            ─────────────────────────
            unsorted

Pass 1: Find minimum in entire array (12)
        Swap with first position
            [12] [45] [89] [25] [38]
             ↑
           sorted

Pass 2: Find minimum in remaining (25)
        Swap with second position
            [12] [25] [89] [45] [38]
             ────────
              sorted

Pass 3: Find minimum in remaining (38)
        Swap with third position
            [12] [25] [38] [45] [89]
             ────────────
                sorted

Pass 4: Find minimum in remaining (45)
        Already in correct position
            [12] [25] [38] [45] [89]
             ─────────────────
                  sorted

Result:     [12] [25] [38] [45] [89]


COMPLEXITY:
───────────────────────────────────────────────────────────────────

• Comparisons: n(n-1)/2 = O(n²) always
• Swaps: n-1 (always, one per pass)
• More efficient than Bubble Sort when swaps are expensive
```

### 3.2 Selection Sort Implementation

```
PROGRAM: Selection Sort (Ascending Order)

; Input:  Array at 2000H, count at 2010H
; Output: Sorted array at same location

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A           ; Outer loop count = n-1
        MOV E, A        ; E = outer loop counter
        LXI H, 2000H    ; Start of unsorted portion
        
OUTER:  ; Find minimum in unsorted portion
        MOV A, M        ; Current minimum = first unsorted
        MOV B, A        ; B = current minimum value
        MOV D, H        ; Save pointer to minimum position
        MOV C, L        ; DC = minimum position
        
        PUSH H          ; Save start of unsorted
        MOV A, E        ; Inner loop count = E
        
INNER:  INX H           ; Next element
        MOV A, M        ; Get element
        CMP B           ; Compare with current minimum
        JNC NOTMIN      ; If ≥ minimum, not new minimum
        
        ; Found new minimum
        MOV B, A        ; B = new minimum value
        MOV D, H        ; DC = new minimum position
        MOV C, L
        
NOTMIN: DCR E           ; We reuse E temporarily
        JNZ INNER
        
        ; Restore outer counter and start position
        POP H           ; HL = start of unsorted
        PUSH D          ; Save minimum position
        PUSH B          ; Save minimum value
        LDA 2010H       ; Restore count
        DCR A
        ; Calculate remaining passes
        ; ... (complex, simplified version below)
        
; SIMPLIFIED VERSION:

        ORG 0100H
        
        LDA 2010H       ; n
        DCR A           ; n-1 passes
        STA 2020H       ; Store outer counter
        LXI H, 2000H    ; Start position
        SHLD 2021H      ; Store start pointer
        
OUTER:  LHLD 2021H      ; Get current start
        MOV A, M        ; First element = initial min
        MOV B, A        ; B = min value
        PUSH H          ; Save start as min position initially
        
        LDA 2020H       ; Get outer counter
        MOV C, A        ; C = inner counter
        
FIND:   INX H           ; Next element
        MOV A, M        ; Get element
        CMP B           ; Compare with min
        JNC SKIP        ; If >= min, skip
        
        MOV B, A        ; New minimum value
        POP D           ; Discard old min position
        PUSH H          ; Save new min position
        
SKIP:   DCR C
        JNZ FIND
        
        ; Swap minimum with first of unsorted
        POP D           ; DE = minimum position
        LHLD 2021H      ; HL = start of unsorted
        
        MOV A, M        ; A = first unsorted element
        MOV C, A        ; Save it
        MOV M, B        ; Place minimum at start
        MOV A, C
        STAX D          ; Place first at min position
        
        ; Move to next unsorted position
        INX H
        SHLD 2021H      ; Update start pointer
        
        LDA 2020H
        DCR A
        STA 2020H       ; Update outer counter
        JNZ OUTER
        
        HLT
```

---

## 4. Linear Search

### 4.1 Algorithm Concept

```
LINEAR SEARCH (Sequential Search):

Check each element one by one until found or end reached.

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Array: [45] [12] [89] [25] [38]                               │
│   Search for: 25                                                │
│                                                                  │
│   Step 1: Check 45 ≠ 25, continue                               │
│   Step 2: Check 12 ≠ 25, continue                               │
│   Step 3: Check 89 ≠ 25, continue                               │
│   Step 4: Check 25 = 25, FOUND at position 3                    │
│                                                                  │
│   COMPLEXITY:                                                    │
│   • Best case: O(1) - first element                             │
│   • Worst case: O(n) - last element or not found                │
│   • Average: O(n/2) = O(n)                                      │
│                                                                  │
│   WORKS ON: Any array (sorted or unsorted)                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Linear Search Implementation

```
PROGRAM: Linear Search

; Input:  Array at 2000H, count at 2010H, key at 2011H
; Output: Position at 2012H (1-based), 00H if not found

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        LDA 2011H       ; Get search key
        MOV B, A        ; B = key
        MVI D, 01H      ; D = position counter (1-based)
        
SEARCH: MOV A, M        ; Get element
        CMP B           ; Compare with key
        JZ FOUND        ; If equal, found
        
        INX H           ; Next element
        INR D           ; Increment position
        DCR C           ; Decrement counter
        JNZ SEARCH      ; Continue if not end
        
        ; Not found
        MVI D, 00H      ; Position = 0 (not found)
        
FOUND:  MOV A, D
        STA 2012H       ; Store position
        HLT

; EXAMPLE:
; Array: 45, 12, 89, 25, 38 at 2000H
; Count: 05 at 2010H
; Key: 25 at 2011H
; Result: 04 at 2012H (found at position 4)
```

### 4.3 Linear Search - Return Address

```
PROGRAM: Linear Search returning memory address

; Input:  Array at 2000H, count at 2010H, key at 2011H
; Output: Address at 2012H-2013H, 0000H if not found

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = counter
        LDA 2011H       ; Get key
        MOV B, A        ; B = key
        
SEARCH: MOV A, M        ; Get element
        CMP B           ; Compare
        JZ FOUND        ; If equal, found
        
        INX H           ; Next element
        DCR C
        JNZ SEARCH
        
        ; Not found
        LXI H, 0000H    ; Return null address
        
FOUND:  ; HL contains address (or 0000H)
        SHLD 2012H      ; Store address
        HLT
```

---

## 5. Binary Search

### 5.1 Algorithm Concept

```
BINARY SEARCH:

Requires sorted array. Divide search space in half each step.

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Sorted Array: [12] [25] [38] [45] [89]                        │
│   Indices:        0    1    2    3    4                         │
│   Search for: 45                                                │
│                                                                  │
│   Step 1:                                                        │
│     low=0, high=4, mid=2                                        │
│     Array[2] = 38                                               │
│     38 < 45, so search in upper half                            │
│     low = mid + 1 = 3                                           │
│                                                                  │
│   Step 2:                                                        │
│     low=3, high=4, mid=3                                        │
│     Array[3] = 45                                               │
│     45 = 45, FOUND at index 3                                   │
│                                                                  │
│   COMPLEXITY:                                                    │
│   • Best case: O(1) - middle element                            │
│   • Worst case: O(log₂n)                                        │
│   • Much faster than linear for large sorted arrays             │
│                                                                  │
│   REQUIRES: Sorted array                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

ALGORITHM:
───────────────────────────────────────────────────────────────────

1. Set low = 0, high = n-1
2. While low ≤ high:
   a. mid = (low + high) / 2
   b. If array[mid] = key, FOUND
   c. If array[mid] < key, low = mid + 1
   d. If array[mid] > key, high = mid - 1
3. NOT FOUND
```

### 5.2 Binary Search Implementation

```
PROGRAM: Binary Search (for sorted ascending array)

; Input:  Sorted array at 2000H, count at 2010H, key at 2011H
; Output: Position at 2012H (1-based), 00H if not found

        ORG 0100H
        
        ; Initialize
        LDA 2011H       ; Get key
        STA 2020H       ; Store key for later access
        
        XRA A           ; low = 0
        STA 2021H       ; Store low
        
        LDA 2010H       ; Get count
        DCR A           ; high = count - 1
        STA 2022H       ; Store high
        
SEARCH: ; Check if low > high
        LDA 2021H       ; Get low
        MOV B, A
        LDA 2022H       ; Get high
        CMP B           ; Compare high with low
        JC NOTFOUND     ; If high < low, not found
        
        ; Calculate mid = (low + high) / 2
        ADD B           ; A = low + high
        RRC             ; A = (low + high) / 2
        MOV C, A        ; C = mid
        
        ; Get array[mid]
        LXI H, 2000H    ; Base address
        MVI D, 00H
        MOV E, C        ; DE = mid
        DAD D           ; HL = base + mid
        
        MOV A, M        ; A = array[mid]
        MOV B, A        ; B = array[mid]
        LDA 2020H       ; A = key
        
        CMP B           ; Compare key with array[mid]
        JZ FOUND        ; If equal, found
        JC UPPER        ; If key < array[mid], search lower half
        
        ; key > array[mid], search upper half
        MOV A, C        ; mid
        INR A           ; low = mid + 1
        STA 2021H
        JMP SEARCH
        
UPPER:  ; key < array[mid], search lower half
        MOV A, C        ; mid
        DCR A           ; high = mid - 1
        STA 2022H
        JMP SEARCH
        
FOUND:  MOV A, C        ; Get mid (0-based index)
        INR A           ; Convert to 1-based position
        STA 2012H
        JMP DONE
        
NOTFOUND:
        MVI A, 00H
        STA 2012H
        
DONE:   HLT

; EXAMPLE:
; Sorted array: 12, 25, 38, 45, 89 (at 2000H-2004H)
; Count: 05 at 2010H
; Key: 45 at 2011H
; 
; Iteration 1: low=0, high=4, mid=2
;   array[2]=38, 45>38, so low=3
; Iteration 2: low=3, high=4, mid=3
;   array[3]=45, 45=45, FOUND
; Result: 04 at 2012H (position 4)
```

### 5.3 Binary Search - Detailed Trace

```
TRACE EXAMPLE:

Array: [12, 25, 38, 45, 89] at 2000H-2004H
Key: 25

Initial: low=0, high=4

Iteration 1:
  mid = (0+4)/2 = 2
  array[2] = 38
  25 < 38, so search lower half
  high = mid - 1 = 1

Iteration 2:
  low=0, high=1
  mid = (0+1)/2 = 0
  array[0] = 12
  25 > 12, so search upper half
  low = mid + 1 = 1

Iteration 3:
  low=1, high=1
  mid = (1+1)/2 = 1
  array[1] = 25
  25 = 25, FOUND at index 1 (position 2)


SEARCH FOR NON-EXISTENT ELEMENT:
───────────────────────────────────────────────────────────────────

Array: [12, 25, 38, 45, 89]
Key: 30

Iteration 1:
  low=0, high=4, mid=2
  array[2]=38, 30<38
  high = 1

Iteration 2:
  low=0, high=1, mid=0
  array[0]=12, 30>12
  low = 1

Iteration 3:
  low=1, high=1, mid=1
  array[1]=25, 30>25
  low = 2

Iteration 4:
  low=2, high=1
  low > high, NOT FOUND
```

---

## 6. Algorithm Comparison

### 6.1 Sorting Algorithm Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   SORTING ALGORITHM COMPARISON                                  │
│                                                                  │
│   ┌──────────────┬───────────┬───────────┬─────────────────┐    │
│   │  Algorithm   │   Best    │   Worst   │     Swaps       │    │
│   ├──────────────┼───────────┼───────────┼─────────────────┤    │
│   │ Bubble Sort  │   O(n)    │  O(n²)    │  O(n²) worst    │    │
│   │ (optimized)  │           │           │                 │    │
│   ├──────────────┼───────────┼───────────┼─────────────────┤    │
│   │ Bubble Sort  │  O(n²)    │  O(n²)    │  O(n²)          │    │
│   │ (basic)      │           │           │                 │    │
│   ├──────────────┼───────────┼───────────┼─────────────────┤    │
│   │ Selection    │  O(n²)    │  O(n²)    │  O(n) always    │    │
│   │ Sort         │           │           │                 │    │
│   └──────────────┴───────────┴───────────┴─────────────────┘    │
│                                                                  │
│   WHEN TO USE WHAT:                                              │
│   • Bubble Sort: Small arrays, nearly sorted data               │
│   • Selection Sort: When swaps are expensive (external storage) │
│   • Both have O(n²) complexity - not for large datasets         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Search Algorithm Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   SEARCH ALGORITHM COMPARISON                                   │
│                                                                  │
│   ┌──────────────┬───────────┬───────────┬─────────────────┐    │
│   │  Algorithm   │   Best    │   Worst   │   Requirement   │    │
│   ├──────────────┼───────────┼───────────┼─────────────────┤    │
│   │ Linear       │   O(1)    │   O(n)    │   None          │    │
│   │ Search       │           │           │                 │    │
│   ├──────────────┼───────────┼───────────┼─────────────────┤    │
│   │ Binary       │   O(1)    │ O(log n)  │   Sorted array  │    │
│   │ Search       │           │           │                 │    │
│   └──────────────┴───────────┴───────────┴─────────────────┘    │
│                                                                  │
│   COMPARISON FOR n=1000:                                         │
│   • Linear Search: up to 1000 comparisons                       │
│   • Binary Search: up to 10 comparisons (log₂1000 ≈ 10)        │
│                                                                  │
│   WHEN TO USE WHAT:                                              │
│   • Linear: Unsorted data, small arrays, single search         │
│   • Binary: Sorted data, large arrays, multiple searches       │
│   • If sorting + binary search < linear, use binary            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Complete Programs

### 7.1 Sort and Find Maximum

```
PROGRAM: Sort array and return maximum (last element)

; Input:  Array at 2000H, count at 2010H
; Output: Sorted array, maximum at 2011H

        ORG 0100H
        
        ; Bubble Sort ascending
        LDA 2010H
        DCR A
        MOV E, A
        
OUTER:  MOV A, E
        MOV D, A
        LXI H, 2000H
        
INNER:  MOV A, M
        INX H
        CMP M
        JC NOSWAP
        JZ NOSWAP
        
        MOV B, M
        MOV M, A
        DCX H
        MOV M, B
        INX H
        
NOSWAP: DCR D
        JNZ INNER
        DCR E
        JNZ OUTER
        
        ; Maximum is at end of sorted array
        LXI H, 2000H
        LDA 2010H
        DCR A           ; index = count - 1
        MVI D, 00H
        MOV E, A
        DAD D           ; HL = base + index
        MOV A, M        ; A = maximum
        STA 2011H
        HLT
```

### 7.2 Count Occurrences in Sorted Array

```
PROGRAM: Count occurrences using binary search bounds

; For a sorted array, find first and last occurrence
; of a value, then count = last - first + 1

; Input:  Sorted array at 2000H, count at 2010H, key at 2011H
; Output: Count at 2012H

; Simplified approach: Linear scan after binary find

        ORG 0100H
        
        ; First, use linear search (simpler for 8085)
        LXI H, 2000H
        LDA 2010H
        MOV C, A        ; C = count
        LDA 2011H
        MOV B, A        ; B = key
        MVI D, 00H      ; D = occurrence count
        
SEARCH: MOV A, M
        CMP B           ; Compare with key
        JNZ SKIP
        INR D           ; Count this occurrence
SKIP:   INX H
        DCR C
        JNZ SEARCH
        
        MOV A, D
        STA 2012H       ; Store count
        HLT

; EXAMPLE:
; Array: 12, 25, 25, 25, 38, 45
; Key: 25
; Count: 3
```

---

## 📋 Summary Table

| Algorithm | Type | Complexity | Requirements | Key Steps |
|-----------|------|------------|--------------|-----------|
| Bubble Sort | Sorting | O(n²) | None | Compare adjacent, swap if needed |
| Selection Sort | Sorting | O(n²) | None | Find min, place at start |
| Linear Search | Searching | O(n) | None | Check each element |
| Binary Search | Searching | O(log n) | Sorted array | Divide in half each step |

---

## ❓ Quick Revision Questions

1. **What is the key difference between Bubble Sort and Selection Sort?**
   <details>
   <summary>Show Answer</summary>
   Bubble Sort: Compares adjacent elements and swaps many times per pass. Selection Sort: Finds minimum in unsorted portion and makes exactly one swap per pass. Selection Sort is better when swaps are expensive.
   </details>

2. **Why is Binary Search faster than Linear Search?**
   <details>
   <summary>Show Answer</summary>
   Binary Search eliminates half the search space with each comparison (O(log n)), while Linear Search checks one element at a time (O(n)). For 1000 elements: Binary needs ~10 comparisons max, Linear needs up to 1000.
   </details>

3. **What is required for Binary Search to work correctly?**
   <details>
   <summary>Show Answer</summary>
   The array must be sorted. Binary Search relies on the ordering to eliminate half the elements each step. On unsorted data, you cannot determine which half contains the target.
   </details>

4. **How can Bubble Sort be optimized for nearly-sorted arrays?**
   <details>
   <summary>Show Answer</summary>
   Use a swap flag. If no swaps occur during a pass, the array is already sorted and we can exit early. This makes best case O(n) instead of O(n²) for already sorted or nearly sorted arrays.
   </details>

5. **Write the comparison logic for ascending vs descending Bubble Sort.**
   <details>
   <summary>Show Answer</summary>
   Ascending: Swap if current > next (use JC after CMP to skip swap when current < next). Descending: Swap if current < next (use JNC after CMP to skip swap when current ≥ next).
   </details>

6. **In Binary Search, what happens when the key is not found?**
   <details>
   <summary>Show Answer</summary>
   The search space keeps halving until low > high. At that point, we know the key doesn't exist in the array. The algorithm terminates when low exceeds high, returning a "not found" indicator (typically 0 or -1).
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [3.5 Array and String Operations](05-array-string-operations.md) | [Unit 3 Index](README.md) | [Unit 4: 8086 Microprocessor](../04-8086-Microprocessor/README.md) |

---

*[← Previous: Array and String Operations](05-array-string-operations.md) | [Next: Unit 4 - 8086 Microprocessor →](../04-8086-Microprocessor/README.md)*
