# Flood Fill

[← Previous: Number of Islands](02-number-of-islands.md) | [Back to TOC](../README.md) | [Next: Surrounded Regions →](04-surrounded-regions.md)

---

## Problem

Given a 2D grid (image), a starting pixel (r, c), and a new color, "flood fill" all connected pixels of the same original color with the new color (like the paint bucket tool).

## Approach

```
FUNCTION floodFill(image, sr, sc, newColor):
    origColor = image[sr][sc]
    
    IF origColor == newColor:        // ★ IMPORTANT edge case!
        RETURN image                 // Already the right color, avoid infinite loop
    
    DFS_Paint(image, sr, sc, origColor, newColor)
    RETURN image

FUNCTION DFS_Paint(image, r, c, origColor, newColor):
    IF r < 0 OR r >= rows OR c < 0 OR c >= cols:
        RETURN
    IF image[r][c] != origColor:
        RETURN
    
    image[r][c] = newColor           // paint this pixel
    
    DFS_Paint(image, r-1, c, origColor, newColor)
    DFS_Paint(image, r+1, c, origColor, newColor)
    DFS_Paint(image, r, c-1, origColor, newColor)
    DFS_Paint(image, r, c+1, origColor, newColor)
```

## Trace

```
    image (start at (1,1), newColor = 3):
    
    Before:              After:
    ┌───┬───┬───┐        ┌───┬───┬───┐
    │ 1 │ 1 │ 1 │        │ 3 │ 3 │ 3 │
    ├───┼───┼───┤        ├───┼───┼───┤
    │ 1 │ 1 │ 0 │   →    │ 3 │ 3 │ 0 │
    ├───┼───┼───┤        ├───┼───┼───┤
    │ 1 │ 0 │ 1 │        │ 3 │ 0 │ 1 │
    └───┴───┴───┘        └───┴───┴───┘
    
    origColor = 1, all connected 1s → 3
    The 0s act as barriers.
    The isolated 1 at (2,2) is NOT connected.

    ★ Edge case: if origColor == newColor, skip!
      Otherwise DFS never stops (already painted 
      pixels still match origColor).
```

---

[← Previous: Number of Islands](02-number-of-islands.md) | [Back to TOC](../README.md) | [Next: Surrounded Regions →](04-surrounded-regions.md)
