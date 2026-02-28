# Word Ladder

[← Previous: Single Source Shortest Path](02-single-source-shortest-path.md) | [Back to TOC](../README.md) | [Next: Minimum Moves Problems →](04-minimum-moves-problems.md)

---

## Problem

Given a beginWord, endWord, and dictionary, find the shortest transformation sequence where each step changes exactly one letter, and each intermediate word must be in the dictionary.

```
    beginWord = "hit"
    endWord = "cog"
    wordList = ["hot","dot","dog","lot","log","cog"]
    
    hit → hot → dot → dog → cog  (length = 5)
```

## Modeling as a Graph

```
    Vertex = each word
    Edge = words that differ by exactly 1 letter
    
    "hit" ── "hot" ── "dot" ── "dog" ── "cog"
                  │          │
                  └── "lot" ── "log" ─┘
    
    Shortest transformation = shortest path = BFS!
```

## Optimization: Wildcard Pattern

```
    Instead of comparing every pair O(N² × L):
    
    Group words by patterns:
    "hot" → "*ot", "h*t", "ho*"
    "dot" → "*ot", "d*t", "do*"
    "lot" → "*ot", "l*t", "lo*"
    
    Pattern "*ot" → ["hot", "dot", "lot"]
    All these words are neighbors!
    
    Build adjacency via pattern HashMap → O(N × L)
```

## Algorithm

```
FUNCTION wordLadder(beginWord, endWord, wordList):
    IF endWord NOT in wordList: RETURN 0
    
    // Build pattern → words mapping
    patterns = HashMap()
    allWords = wordList + [beginWord]
    FOR each word in allWords:
        FOR i = 0 to word.length - 1:
            pattern = word[0..i-1] + "*" + word[i+1..end]
            patterns[pattern].append(word)
    
    // BFS
    visited = {beginWord}
    queue = [beginWord]
    level = 1
    
    WHILE queue not empty:
        size = queue.size()
        FOR i = 1 to size:
            word = queue.dequeue()
            
            FOR j = 0 to word.length - 1:
                pattern = word[0..j-1] + "*" + word[j+1..end]
                
                FOR each neighbor in patterns[pattern]:
                    IF neighbor == endWord: RETURN level + 1
                    IF neighbor NOT in visited:
                        visited.add(neighbor)
                        queue.enqueue(neighbor)
        
        level += 1
    
    RETURN 0    // no path found
```

---

[← Previous: Single Source Shortest Path](02-single-source-shortest-path.md) | [Back to TOC](../README.md) | [Next: Minimum Moves Problems →](04-minimum-moves-problems.md)
