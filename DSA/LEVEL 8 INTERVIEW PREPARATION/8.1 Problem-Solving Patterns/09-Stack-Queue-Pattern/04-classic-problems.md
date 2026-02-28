# Chapter 4: Classic Stack/Queue Problems

## 📋 Chapter Overview
Essential interview problems: trapping rain water, asteroid collision, decode string, and online stock span.

---

## 📝 Problem 1: Trapping Rain Water (LeetCode 42)

**Problem:** Given elevation map, compute how much water it can trap.

```
  Elevation: [0,1,0,2,1,0,1,3,2,1,2,1]
  
              3
           ___█___
      2   █ W █ W 2
   ___█_W_█_█_█_█_█___
  █_█_█_█_█_█_█_█_█_█_█
  0 1 0 2 1 0 1 3 2 1 2 1
  
  W = water. Total trapped = 6
```

### Approach 1: Two Pointers (Optimal)

```
function trap(height):
    left = 0, right = n-1
    leftMax = 0, rightMax = 0
    water = 0
    
    while left < right:
        if height[left] < height[right]:
            if height[left] >= leftMax:
                leftMax = height[left]
            else:
                water += leftMax - height[left]
            left++
        else:
            if height[right] >= rightMax:
                rightMax = height[right]
            else:
                water += rightMax - height[right]
            right--
    
    return water
```

### Trace (Two Pointers)

```
  height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
  left=0, right=11, leftMax=0, rightMax=0, water=0
  
  h[0]=0 < h[11]=1: leftMax=max(0,0)=0. water+=0.     left=1
  h[1]=1 < h[11]=1: leftMax=1.                         left=2
  Nope, 1 ≥ 1 →  else branch:
  h[11]=1: rightMax=1.                                  right=10
  h[2]=0 < h[10]=2: 0 < leftMax=1 → water+=1-0=1.     left=3. water=1
  h[3]=2 ≥ h[10]=2: h[10]=2 → rightMax=2.              right=9
  h[3]=2 ≥ h[9]=1: h[9]=1 < rightMax=2 → water+=2-1=1. right=8. water=2
  h[3]=2 ≥ h[8]=2: h[8]=2 → rightMax=2.                right=7
  h[3]=2 < h[7]=3: leftMax=2.                          left=4
  h[4]=1 < h[7]=3: water+=2-1=1.                       left=5. water=3
  h[5]=0 < h[7]=3: water+=2-0=2.                       left=6. water=5
  h[6]=1 < h[7]=3: water+=2-1=1.                       left=7. water=6
  
  left == right → stop. Answer: 6
```

### Approach 2: Monotonic Stack

```
function trapStack(height):
    stack = []
    water = 0
    
    for i = 0 to n-1:
        while stack not empty and height[i] > height[stack.top()]:
            bottom = height[stack.pop()]
            if stack is empty: break
            width = i - stack.top() - 1
            h = min(height[i], height[stack.top()]) - bottom
            water += width * h
        stack.push(i)
    
    return water
```

**Complexity:** O(n) time, O(1) space (two pointers) or O(n) space (stack)

---

## 📝 Problem 2: Asteroid Collision (LeetCode 735)

**Problem:** Positive = moving right, negative = moving left. Same size = both explode. Larger survives.

```
function asteroidCollision(asteroids):
    stack = []
    
    for each ast in asteroids:
        alive = true
        while alive and stack not empty and ast < 0 and stack.top() > 0:
            if stack.top() < -ast:
                stack.pop()          // top destroyed
            elif stack.top() == -ast:
                stack.pop()          // both destroyed
                alive = false
            else:
                alive = false        // current destroyed
        
        if alive:
            stack.push(ast)
    
    return stack
```

### Trace

```
  asteroids = [5, 10, -5]
  
  5:  push. stack=[5]
  10: push (both going right). stack=[5,10]
  -5: collision with 10. |10| > |-5| → -5 destroyed. stack=[5,10]
  
  Result: [5, 10]
  
  asteroids = [8, -8]
  8:  push. stack=[8]
  -8: |8| == |-8| → both destroyed. stack=[]
  Result: []
  
  asteroids = [10, 2, -5]
  10: push. stack=[10]
  2:  push. stack=[10,2]
  -5: collision with 2. |-5|>2 → pop 2. stack=[10]
      collision with 10. |-5|<10 → -5 destroyed. stack=[10]
  Result: [10]
```

---

## 📝 Problem 3: Decode String (LeetCode 394)

**Problem:** `k[encoded_string]` means repeat encoded_string k times. E.g., `3[a2[c]]` → `accaccacc`.

```
function decodeString(s):
    numStack = []
    strStack = []
    currentStr = ""
    currentNum = 0
    
    for each char c:
        if c is digit:
            currentNum = currentNum * 10 + (c - '0')
        elif c == '[':
            numStack.push(currentNum)
            strStack.push(currentStr)
            currentNum = 0
            currentStr = ""
        elif c == ']':
            count = numStack.pop()
            prevStr = strStack.pop()
            currentStr = prevStr + currentStr.repeat(count)
        else:
            currentStr += c
    
    return currentStr
```

### Trace

```
  s = "3[a2[c]]"
  
  '3': currentNum = 3
  '[': push(3, ""). currentNum=0, currentStr="".
       numStack=[3], strStack=[""]
  'a': currentStr = "a"
  '2': currentNum = 2
  '[': push(2, "a"). currentNum=0, currentStr="".
       numStack=[3,2], strStack=["","a"]
  'c': currentStr = "c"
  ']': count=2, prev="a". currentStr = "a" + "cc" = "acc"
       numStack=[3], strStack=[""]
  ']': count=3, prev="". currentStr = "" + "accaccacc" = "accaccacc"
  
  Result: "accaccacc"
```

---

## 📝 Problem 4: Online Stock Span (LeetCode 901)

**Problem:** For each day's stock price, find the span (consecutive days the price was ≤ today's, including today).

```
class StockSpanner:
    stack = []  // pairs (price, span)
    
    next(price):
        span = 1
        while stack not empty and stack.top().price <= price:
            span += stack.pop().span
        stack.push((price, span))
        return span
```

### Trace

```
  Prices: 100, 80, 60, 70, 60, 75, 85
  
  100: stack empty. span=1. push(100,1). → 1
  80:  80<100. span=1. push(80,1).      → 1
  60:  60<80. span=1. push(60,1).       → 1
  70:  70>60 → pop(60,1). span=1+1=2.
       70<80. push(70,2).               → 2
  60:  60<70. span=1. push(60,1).       → 1
  75:  75>60 → pop(60,1). span=1+1=2.
       75>70 → pop(70,2). span=2+2=4.
       75<80. push(75,4).               → 4
  85:  85>75 → pop(75,4). span=1+4=5.
       85>80 → pop(80,1). span=5+1=6.
       85<100. push(85,6).              → 6
```

---

## 📊 Classic Problems Summary

| Problem | Data Structure | Key Idea | Time |
|---------|---------------|----------|------|
| Trapping rain water | Two pointers / stack | Water level = min(leftMax, rightMax) - height | O(n) |
| Asteroid collision | Stack | Simulate collisions between right-moving and left-moving | O(n) |
| Decode string | Two stacks (num + str) | Save context on '[', restore on ']' | O(output len) |
| Stock span | Monotonic stack | Accumulate spans of smaller elements | O(1) amortized |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Two pointers for water | Process from both ends; water depends on shorter side |
| Stack for collisions | Process sequence; stack holds surviving elements |
| Nested decoding | Stack preserves outer context while processing inner |
| Span accumulation | Pop and absorb consecutive ≤ elements' spans |

---

## ❓ Revision Questions

1. **Trapping rain water: two approaches?** → Two pointers (O(1) space) or monotonic stack (O(n) space). Both O(n) time.
2. **Asteroid collision: when do collisions happen?** → Only when top of stack is positive (right-moving) and current is negative (left-moving).
3. **Decode string: why two stacks?** → One for repeat counts, one for partial strings before each '['. Handles nesting.
4. **Stock span: why accumulate?** → When popping a smaller element, its span represents consecutive days already counted, so we absorb them.
5. **Which problems use monotonic stack?** → Rain water (stack approach), stock span, daily temperatures, next greater element, largest rectangle.

---

[← Previous: Queue Patterns](03-queue-patterns.md) | [Next: When to Apply →](05-when-to-apply.md)
