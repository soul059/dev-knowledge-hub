# Chapter 9.6: Graphics Basics

> **Unit 9 — Practical Applications**
> Apply bit manipulation to color representation, alpha blending, pixel operations, and image processing.

---

## Overview

Computer graphics stores colors, transparency, and pixel data as packed bit fields. Bitwise operations are essential for color manipulation, blending, and efficient rendering.

---

## Color Representation (RGBA)

```
  32-bit color: 8 bits per channel

  Bit:  31      24 23      16 15       8 7        0
        ┌────────┬──────────┬──────────┬──────────┐
        │ Alpha  │   Red    │  Green   │   Blue   │
        │  0xFF  │   0xCC   │  0x88    │  0x44    │
        └────────┴──────────┴──────────┴──────────┘

  ARGB: 0xFFCC8844
    A = 255 (fully opaque)
    R = 204
    G = 136
    B = 68
```

### Channel Extraction

```
FUNCTION getAlpha(color): RETURN (color >> 24) & 0xFF
FUNCTION getRed(color):   RETURN (color >> 16) & 0xFF
FUNCTION getGreen(color): RETURN (color >> 8)  & 0xFF
FUNCTION getBlue(color):  RETURN color & 0xFF
```

### Color Construction

```
FUNCTION makeColor(a, r, g, b):
    RETURN (a << 24) | (r << 16) | (g << 8) | b
```

### Trace: Extract green from 0xFF6B8E23

```
  0xFF6B8E23 >> 8 = 0x00FF6B8E
  0x00FF6B8E & 0xFF = 0x8E = 142

  Green = 142  ✓
```

---

## Color Manipulation

### Brighten/Darken

```
FUNCTION brighten(color, amount):
    a = getAlpha(color)
    r = MIN(255, getRed(color) + amount)
    g = MIN(255, getGreen(color) + amount)
    b = MIN(255, getBlue(color) + amount)
    RETURN makeColor(a, r, g, b)
```

### Grayscale Conversion

```
FUNCTION toGrayscale(color):
    r = getRed(color)
    g = getGreen(color)
    b = getBlue(color)
    // Luminance formula (approximate with shifts)
    // 0.299R + 0.587G + 0.114B
    // ≈ (77R + 150G + 29B) >> 8
    gray = (77*r + 150*g + 29*b) >> 8
    RETURN makeColor(getAlpha(color), gray, gray, gray)
```

### Invert Colors

```
FUNCTION invert(color):
    // XOR RGB with 0xFF, keep alpha
    RETURN color ^ 0x00FFFFFF
```

### Trace: Invert 0xFF6B8E23

```
    0xFF6B8E23
  ^ 0x00FFFFFF
  ────────────
    0xFF9471DC

  R: 0x6B → 0x94 (107 → 148)
  G: 0x8E → 0x71 (142 → 113)
  B: 0x23 → 0xDC (35  → 220)
  A: 0xFF → 0xFF (unchanged)  ✓
```

---

## Alpha Blending

```
  Blend foreground over background using alpha:

  result = fg × alpha + bg × (1 - alpha)

  With integer math (alpha 0-255):
    result = (fg × a + bg × (255 - a)) / 255
```

### Fast Approximate Blend

```
FUNCTION blendChannel(fg, bg, alpha):
    // Exact: (fg * alpha + bg * (255 - alpha)) / 255
    // Fast:  (fg * alpha + bg * (255 - alpha) + 128) >> 8
    // Even faster with bit tricks:
    tmp = fg * alpha + bg * (255 - alpha) + 128
    RETURN (tmp + (tmp >> 8)) >> 8

FUNCTION alphaBlend(fg, bg):
    a = getAlpha(fg)
    r = blendChannel(getRed(fg), getRed(bg), a)
    g = blendChannel(getGreen(fg), getGreen(bg), a)
    b = blendChannel(getBlue(fg), getBlue(bg), a)
    RETURN makeColor(255, r, g, b)
```

---

## Bit-Level Pixel Operations

### Swap Red and Blue Channels

```
FUNCTION swapRedBlue(color):
    a = color & 0xFF00FF00    // keep alpha and green
    r = (color >> 16) & 0xFF  // extract red
    b = color & 0xFF          // extract blue
    RETURN a | (b << 16) | r  // blue→red position, red→blue position
```

### Average Two Colors (without overflow)

```
FUNCTION averageColor(c1, c2):
    // Average each channel without overflow
    // (a+b)/2 = (a&b) + ((a^b)>>1), but per-channel:
    RETURN ((c1 & c2) + (((c1 ^ c2) & 0xFEFEFEFE) >> 1))
```

```
  This clever formula applies the bitwise average to all
  four channels simultaneously!

  (a & b):      common bits (bits set in both)
  (a ^ b):      differing bits
  (a^b) >> 1:   half of the differing bits
  & 0xFEFEFEFE: prevent bit bleeding between channels
```

---

## Image as Bit Array

```
  1-bit images (monochrome):
  Each pixel = 1 bit, 8 pixels per byte

  ┌──────────────────────────────────┐
  │  Image: 8×4 pixels (4 bytes)    │
  │                                  │
  │  Byte 0: 0xFF = ████████        │
  │  Byte 1: 0x81 = █      █        │
  │  Byte 2: 0x81 = █      █        │
  │  Byte 3: 0xFF = ████████        │
  │                                  │
  │  A rectangle using only 4 bytes!│
  └──────────────────────────────────┘
```

### Get/Set Pixel in 1-bit Image

```
FUNCTION getPixel(image[], x, y, width):
    index = y * width + x
    byte_index = index / 8
    bit_index = 7 - (index % 8)    // MSB = leftmost pixel
    RETURN (image[byte_index] >> bit_index) & 1

FUNCTION setPixel(image[], x, y, width, value):
    index = y * width + x
    byte_index = index / 8
    bit_index = 7 - (index % 8)
    IF value:
        image[byte_index] |= (1 << bit_index)
    ELSE:
        image[byte_index] &= ~(1 << bit_index)
```

---

## Color Formats Summary

```
  ┌─────────────┬──────┬────────────────────────────────┐
  │ Format      │ Bits │ Layout                         │
  ├─────────────┼──────┼────────────────────────────────┤
  │ RGB565      │ 16   │ RRRRR GGGGGG BBBBB             │
  │ ARGB1555    │ 16   │ A RRRRR GGGGG BBBBB            │
  │ RGB888      │ 24   │ RRRRRRRR GGGGGGGG BBBBBBBB     │
  │ ARGB8888    │ 32   │ AAAAAAAA RRRRRRRR GGGGGG BBBB  │
  │ Monochrome  │ 1    │ Single bit per pixel            │
  └─────────────┴──────┴────────────────────────────────┘
```

---

## Quick Revision Questions

1. **Why is color stored as a packed 32-bit integer?**
   <details><summary>Answer</summary>Packing ARGB into one integer enables efficient copy, compare, and bulk operations using single-instruction 32-bit operations. Memory alignment is natural, and SIMD can process multiple colors simultaneously.</details>

2. **How does the average-without-overflow formula work?**
   <details><summary>Answer</summary>It uses the identity (a+b)/2 = (a&b) + ((a^b)>>1). The mask 0xFEFEFEFE clears the LSB of each byte before shifting right, preventing one channel's bits from bleeding into the adjacent channel.</details>

3. **Why XOR 0x00FFFFFF instead of 0xFFFFFFFF to invert?**
   <details><summary>Answer</summary>0x00FFFFFF inverts only the RGB channels while keeping the alpha channel unchanged. Using 0xFFFFFFFF would also invert alpha, making an opaque pixel transparent — usually not desired.</details>

4. **What advantage does RGB565 have over RGB888?**
   <details><summary>Answer</summary>RGB565 uses 16 bits vs. 24 bits — 33% less memory. For embedded displays and older GPUs with limited memory bandwidth, this savings is significant. The quality loss (fewer bits per channel) is acceptable for many applications.</details>

5. **How many bytes does a 640×480 monochrome image use?**
   <details><summary>Answer</summary>640 × 480 = 307,200 pixels. At 1 bit per pixel: 307,200 / 8 = 38,400 bytes ≈ 37.5 KB. Compared to 32-bit ARGB: 307,200 × 4 = 1.2 MB — a 32× savings.</details>

---

[← Previous: Bloom Filter Bits](05-bloom-filter-bits.md) | [Back to README →](../README.md)
