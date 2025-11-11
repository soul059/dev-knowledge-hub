# Android Canvas and Paint Cheatsheet

A comprehensive guide to Canvas and Paint fundamentals for Android development with Java.

---

## Table of Contents

1. [Canvas and Paint Fundamentals](#canvas-and-paint-fundamentals)
2. [Drawing Basic Shapes](#drawing-basic-shapes)
3. [Path Drawing](#path-drawing)
4. [Paint Styles and Properties](#paint-styles-and-properties)
5. [Advanced Techniques](#advanced-techniques)
6. [Complete Examples](#complete-examples)

---

## Canvas and Paint Fundamentals

### Basic Setup

```java
public class CustomView extends View {
    private Paint paint;
    private Canvas canvas;
    
    public CustomView(Context context) {
        super(context);
        init();
    }
    
    private void init() {
        paint = new Paint();
        paint.setAntiAlias(true); // Smooth edges
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // Your drawing code here
    }
}
```

### Paint Object Initialization

```java
Paint paint = new Paint();

// Basic properties
paint.setColor(Color.RED);                    // Set color
paint.setStrokeWidth(5f);                     // Set line width
paint.setAntiAlias(true);                     // Enable anti-aliasing
paint.setStyle(Paint.Style.FILL);             // FILL, STROKE, or FILL_AND_STROKE
paint.setAlpha(128);                          // Set transparency (0-255)
```

### Paint Styles

| Style | Description | Usage |
|-------|-------------|-------|
| `Paint.Style.FILL` | Fill the shape | Solid shapes |
| `Paint.Style.STROKE` | Draw outline only | Borders, lines |
| `Paint.Style.FILL_AND_STROKE` | Both fill and outline | Bold shapes |

---

## Drawing Basic Shapes

### 1. Drawing Lines

```java
// Draw a single line
canvas.drawLine(startX, startY, stopX, stopY, paint);

// Example: Diagonal line
paint.setColor(Color.BLACK);
paint.setStrokeWidth(5f);
canvas.drawLine(100, 100, 500, 500, paint);

// Draw multiple lines
float[] points = {
    x1, y1, x2, y2,  // Line 1
    x3, y3, x4, y4,  // Line 2
    x5, y5, x6, y6   // Line 3
};
canvas.drawLines(points, paint);
```

### 2. Drawing Rectangles

```java
// Method 1: Using coordinates
canvas.drawRect(left, top, right, bottom, paint);

// Example: Filled rectangle
paint.setStyle(Paint.Style.FILL);
paint.setColor(Color.BLUE);
canvas.drawRect(100, 100, 400, 300, paint);

// Method 2: Using Rect object
Rect rect = new Rect(100, 100, 400, 300);
canvas.drawRect(rect, paint);

// Method 3: Using RectF (floating point)
RectF rectF = new RectF(100f, 100f, 400f, 300f);
canvas.drawRect(rectF, paint);

// Rounded rectangle
float cornerRadius = 20f;
canvas.drawRoundRect(rectF, cornerRadius, cornerRadius, paint);
```

### 3. Drawing Circles

```java
// Draw circle: (centerX, centerY, radius, paint)
canvas.drawCircle(cx, cy, radius, paint);

// Example: Filled circle
paint.setStyle(Paint.Style.FILL);
paint.setColor(Color.RED);
canvas.drawCircle(300, 300, 100, paint);

// Example: Circle with border
paint.setStyle(Paint.Style.STROKE);
paint.setStrokeWidth(5f);
paint.setColor(Color.BLACK);
canvas.drawCircle(300, 300, 100, paint);
```

### 4. Drawing Ovals/Ellipses

```java
// Using RectF to define bounding box
RectF ovalRect = new RectF(100, 200, 400, 500);
canvas.drawOval(ovalRect, paint);

// Example: Horizontal ellipse
paint.setColor(Color.GREEN);
paint.setStyle(Paint.Style.FILL);
RectF ellipse = new RectF(100, 300, 500, 400);
canvas.drawOval(ellipse, paint);
```

### 5. Drawing Arcs

```java
// drawArc(oval, startAngle, sweepAngle, useCenter, paint)
RectF arcRect = new RectF(100, 100, 400, 400);

// Pie slice (useCenter = true)
canvas.drawArc(arcRect, 0, 90, true, paint);

// Arc only (useCenter = false)
canvas.drawArc(arcRect, 0, 180, false, paint);

// Example: Semi-circle
paint.setColor(Color.YELLOW);
paint.setStyle(Paint.Style.FILL);
canvas.drawArc(new RectF(100, 100, 400, 400), 0, 180, true, paint);
```

### 6. Drawing Points

```java
// Single point
canvas.drawPoint(x, y, paint);

// Multiple points
float[] points = {x1, y1, x2, y2, x3, y3};
canvas.drawPoints(points, paint);

// Example: Dot pattern
paint.setColor(Color.BLACK);
paint.setStrokeWidth(10f);
for (int i = 0; i < 10; i++) {
    canvas.drawPoint(i * 50, 200, paint);
}
```

### 7. Drawing Text

```java
// Basic text
paint.setTextSize(50f);
paint.setColor(Color.BLACK);
canvas.drawText("Hello Android", x, y, paint);

// Text alignment
paint.setTextAlign(Paint.Align.CENTER);  // LEFT, CENTER, RIGHT
canvas.drawText("Centered", canvasWidth/2, 100, paint);

// Text with custom font
Typeface typeface = Typeface.create("sans-serif-light", Typeface.BOLD);
paint.setTypeface(typeface);
canvas.drawText("Bold Text", 100, 200, paint);
```

---

## Path Drawing

### Path Basics

```java
Path path = new Path();

// Move to starting point (without drawing)
path.moveTo(x, y);

// Draw line to point
path.lineTo(x, y);

// Close the path (connect last point to first)
path.close();

// Draw the path
canvas.drawPath(path, paint);
```

### Drawing Lines with Path

```java
// Simple line
Path path = new Path();
path.moveTo(100, 100);
path.lineTo(500, 100);
path.lineTo(500, 500);
path.lineTo(100, 500);
path.close(); // Creates a rectangle

paint.setStyle(Paint.Style.STROKE);
paint.setStrokeWidth(5f);
canvas.drawPath(path, paint);
```

### Drawing Curves

```java
// Quadratic Bezier Curve
Path path = new Path();
path.moveTo(100, 500);
path.quadTo(controlX, controlY, endX, endY);
canvas.drawPath(path, paint);

// Example: Smooth curve
Path curvePath = new Path();
curvePath.moveTo(100, 400);
curvePath.quadTo(300, 100, 500, 400);
paint.setStyle(Paint.Style.STROKE);
paint.setStrokeWidth(5f);
canvas.drawPath(curvePath, paint);

// Cubic Bezier Curve (two control points)
path.cubicTo(control1X, control1Y, control2X, control2Y, endX, endY);
```

### Drawing Shapes with Path

```java
// Triangle
Path triangle = new Path();
triangle.moveTo(300, 100);  // Top point
triangle.lineTo(100, 400);  // Bottom left
triangle.lineTo(500, 400);  // Bottom right
triangle.close();           // Connect back to start
canvas.drawPath(triangle, paint);

// Star
Path star = new Path();
star.moveTo(250, 50);      // Top
star.lineTo(300, 200);
star.lineTo(450, 200);
star.lineTo(325, 300);
star.lineTo(375, 450);
star.lineTo(250, 350);
star.lineTo(125, 450);
star.lineTo(175, 300);
star.lineTo(50, 200);
star.lineTo(200, 200);
star.close();
canvas.drawPath(star, paint);
```

### Path Operations

```java
// Add circle to path
path.addCircle(cx, cy, radius, Path.Direction.CW);

// Add rectangle to path
RectF rect = new RectF(100, 100, 400, 400);
path.addRect(rect, Path.Direction.CW);

// Add oval to path
path.addOval(rect, Path.Direction.CW);

// Add arc to path
path.addArc(rect, startAngle, sweepAngle);

// Add rounded rectangle
path.addRoundRect(rect, radiusX, radiusY, Path.Direction.CW);
```

### Complex Path Example

```java
// Drawing a heart shape
Path heartPath = new Path();
heartPath.moveTo(250, 300);

// Left curve
heartPath.cubicTo(150, 200, 100, 250, 100, 350);
heartPath.cubicTo(100, 450, 200, 500, 250, 550);

// Right curve
heartPath.cubicTo(300, 500, 400, 450, 400, 350);
heartPath.cubicTo(400, 250, 350, 200, 250, 300);

heartPath.close();

paint.setColor(Color.RED);
paint.setStyle(Paint.Style.FILL);
canvas.drawPath(heartPath, paint);
```

---

## Paint Styles and Properties

### Stroke Properties

```java
// Stroke width
paint.setStrokeWidth(10f);

// Stroke cap (end of lines)
paint.setStrokeCap(Paint.Cap.ROUND);   // ROUND, SQUARE, BUTT

// Stroke join (corners)
paint.setStrokeJoin(Paint.Join.ROUND); // ROUND, BEVEL, MITER

// Miter limit (for MITER join)
paint.setStrokeMiter(10f);
```

### Colors and Transparency

```java
// Set color by constant
paint.setColor(Color.RED);

// Set color by RGB
paint.setColor(Color.rgb(255, 100, 50));

// Set color by ARGB (with alpha)
paint.setColor(Color.argb(128, 255, 100, 50));

// Set color by hex
paint.setColor(Color.parseColor("#FF6347"));

// Set alpha separately
paint.setAlpha(128); // 0 (transparent) to 255 (opaque)
```

### Gradients

```java
// Linear Gradient
LinearGradient linearGradient = new LinearGradient(
    x0, y0,           // Start point
    x1, y1,           // End point
    Color.RED,        // Start color
    Color.BLUE,       // End color
    Shader.TileMode.CLAMP
);
paint.setShader(linearGradient);

// Radial Gradient
RadialGradient radialGradient = new RadialGradient(
    centerX, centerY, // Center
    radius,           // Radius
    Color.WHITE,      // Center color
    Color.BLACK,      // Edge color
    Shader.TileMode.CLAMP
);
paint.setShader(radialGradient);

// Multiple color gradient
int[] colors = {Color.RED, Color.YELLOW, Color.GREEN, Color.BLUE};
float[] positions = {0f, 0.33f, 0.66f, 1f};
LinearGradient multiGradient = new LinearGradient(
    0, 0, canvasWidth, 0,
    colors,
    positions,
    Shader.TileMode.CLAMP
);
```

### Effects

```java
// Shadow
paint.setShadowLayer(10f, 5f, 5f, Color.BLACK);

// Blur effect
MaskFilter blur = new BlurMaskFilter(10f, BlurMaskFilter.Blur.NORMAL);
paint.setMaskFilter(blur);

// Dash effect
PathEffect dashEffect = new DashPathEffect(new float[]{10, 5}, 0);
paint.setPathEffect(dashEffect);

// Corner effect
PathEffect cornerEffect = new CornerPathEffect(20f);
paint.setPathEffect(cornerEffect);
```

---

## Advanced Techniques

### Canvas Transformations

```java
// Save and restore canvas state
canvas.save();

// Translate (move origin)
canvas.translate(dx, dy);

// Rotate
canvas.rotate(degrees, pivotX, pivotY);

// Scale
canvas.scale(sx, sy, pivotX, pivotY);

// Skew
canvas.skew(sx, sy);

// Restore to saved state
canvas.restore();

// Example: Rotating a rectangle
canvas.save();
canvas.rotate(45, 300, 300);
canvas.drawRect(200, 200, 400, 400, paint);
canvas.restore();
```

### Clipping

```java
// Clip to rectangle
canvas.clipRect(left, top, right, bottom);

// Clip to path
Path clipPath = new Path();
clipPath.addCircle(300, 300, 200, Path.Direction.CW);
canvas.clipPath(clipPath);

// Only content inside the clip region will be drawn
canvas.drawColor(Color.BLUE);
```

### Drawing Bitmap

```java
// Load bitmap
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image);

// Draw bitmap
canvas.drawBitmap(bitmap, x, y, paint);

// Draw bitmap with destination rectangle
Rect src = new Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
Rect dst = new Rect(100, 100, 500, 500);
canvas.drawBitmap(bitmap, src, dst, paint);
```

---

## Complete Examples

### Example 1: Custom Button

```java
public class CustomButton extends View {
    private Paint paint;
    private RectF buttonRect;
    
    public CustomButton(Context context) {
        super(context);
        init();
    }
    
    private void init() {
        paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        buttonRect = new RectF();
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        
        // Button background
        buttonRect.set(50, 50, 350, 150);
        paint.setColor(Color.parseColor("#4CAF50"));
        paint.setStyle(Paint.Style.FILL);
        canvas.drawRoundRect(buttonRect, 20, 20, paint);
        
        // Button border
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeWidth(5);
        paint.setColor(Color.parseColor("#388E3C"));
        canvas.drawRoundRect(buttonRect, 20, 20, paint);
        
        // Button text
        paint.setStyle(Paint.Style.FILL);
        paint.setColor(Color.WHITE);
        paint.setTextSize(40);
        paint.setTextAlign(Paint.Align.CENTER);
        canvas.drawText("CLICK ME", 200, 110, paint);
    }
}
```

### Example 2: Progress Circle

```java
public class ProgressCircle extends View {
    private Paint backgroundPaint;
    private Paint progressPaint;
    private RectF oval;
    private float progress = 0f; // 0 to 100
    
    public ProgressCircle(Context context) {
        super(context);
        init();
    }
    
    private void init() {
        backgroundPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        backgroundPaint.setStyle(Paint.Style.STROKE);
        backgroundPaint.setStrokeWidth(20);
        backgroundPaint.setColor(Color.LTGRAY);
        
        progressPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        progressPaint.setStyle(Paint.Style.STROKE);
        progressPaint.setStrokeWidth(20);
        progressPaint.setColor(Color.BLUE);
        progressPaint.setStrokeCap(Paint.Cap.ROUND);
        
        oval = new RectF();
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        
        int centerX = getWidth() / 2;
        int centerY = getHeight() / 2;
        int radius = 150;
        
        oval.set(centerX - radius, centerY - radius,
                centerX + radius, centerY + radius);
        
        // Background circle
        canvas.drawArc(oval, 0, 360, false, backgroundPaint);
        
        // Progress arc
        float sweepAngle = (progress / 100f) * 360f;
        canvas.drawArc(oval, -90, sweepAngle, false, progressPaint);
        
        // Progress text
        Paint textPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setTextSize(60);
        textPaint.setTextAlign(Paint.Align.CENTER);
        textPaint.setColor(Color.BLACK);
        canvas.drawText((int)progress + "%", centerX, centerY + 20, textPaint);
    }
    
    public void setProgress(float progress) {
        this.progress = progress;
        invalidate(); // Redraw
    }
}
```

### Example 3: Wave Animation

```java
public class WaveView extends View {
    private Paint wavePaint;
    private Path wavePath;
    private float waveOffset = 0f;
    
    public WaveView(Context context) {
        super(context);
        init();
    }
    
    private void init() {
        wavePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        wavePaint.setStyle(Paint.Style.FILL);
        wavePaint.setColor(Color.CYAN);
        wavePath = new Path();
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        
        wavePath.reset();
        wavePath.moveTo(0, getHeight() / 2);
        
        // Create wave using sine curve
        float amplitude = 50f;
        float frequency = 0.02f;
        
        for (int x = 0; x <= getWidth(); x += 10) {
            float y = (float) (getHeight() / 2 + 
                     amplitude * Math.sin(frequency * (x + waveOffset)));
            wavePath.lineTo(x, y);
        }
        
        // Complete the path
        wavePath.lineTo(getWidth(), getHeight());
        wavePath.lineTo(0, getHeight());
        wavePath.close();
        
        canvas.drawPath(wavePath, wavePaint);
        
        // Animate
        waveOffset += 5;
        invalidate();
    }
}
```

### Example 4: Chart/Graph

```java
public class LineChart extends View {
    private Paint linePaint;
    private Paint pointPaint;
    private Paint axisPaint;
    private float[] dataPoints = {20, 45, 30, 60, 50, 75, 65};
    
    public LineChart(Context context) {
        super(context);
        init();
    }
    
    private void init() {
        linePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        linePaint.setStyle(Paint.Style.STROKE);
        linePaint.setStrokeWidth(5);
        linePaint.setColor(Color.BLUE);
        
        pointPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        pointPaint.setStyle(Paint.Style.FILL);
        pointPaint.setColor(Color.RED);
        
        axisPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        axisPaint.setStyle(Paint.Style.STROKE);
        axisPaint.setStrokeWidth(2);
        axisPaint.setColor(Color.BLACK);
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        
        int padding = 50;
        int chartWidth = getWidth() - 2 * padding;
        int chartHeight = getHeight() - 2 * padding;
        
        // Draw axes
        canvas.drawLine(padding, padding, padding, 
                       getHeight() - padding, axisPaint);
        canvas.drawLine(padding, getHeight() - padding, 
                       getWidth() - padding, getHeight() - padding, axisPaint);
        
        // Draw line chart
        Path path = new Path();
        float xStep = chartWidth / (dataPoints.length - 1);
        
        for (int i = 0; i < dataPoints.length; i++) {
            float x = padding + i * xStep;
            float y = getHeight() - padding - (dataPoints[i] / 100f * chartHeight);
            
            if (i == 0) {
                path.moveTo(x, y);
            } else {
                path.lineTo(x, y);
            }
            
            // Draw points
            canvas.drawCircle(x, y, 8, pointPaint);
        }
        
        canvas.drawPath(path, linePaint);
    }
}
```

---

## Quick Reference Table

### Common Canvas Methods

| Method | Description | Example |
|--------|-------------|---------|
| `drawColor()` | Fill entire canvas | `canvas.drawColor(Color.WHITE)` |
| `drawLine()` | Draw line | `canvas.drawLine(x1, y1, x2, y2, paint)` |
| `drawRect()` | Draw rectangle | `canvas.drawRect(l, t, r, b, paint)` |
| `drawCircle()` | Draw circle | `canvas.drawCircle(cx, cy, r, paint)` |
| `drawOval()` | Draw oval | `canvas.drawOval(rectF, paint)` |
| `drawArc()` | Draw arc | `canvas.drawArc(rectF, start, sweep, useCenter, paint)` |
| `drawPath()` | Draw path | `canvas.drawPath(path, paint)` |
| `drawText()` | Draw text | `canvas.drawText(text, x, y, paint)` |
| `drawBitmap()` | Draw image | `canvas.drawBitmap(bitmap, x, y, paint)` |

### Common Paint Methods

| Method | Description | Values |
|--------|-------------|--------|
| `setColor()` | Set color | `Color.RED`, `#RRGGBB` |
| `setStyle()` | Set style | `FILL`, `STROKE`, `FILL_AND_STROKE` |
| `setStrokeWidth()` | Set line width | `float` value in pixels |
| `setAlpha()` | Set transparency | 0-255 |
| `setTextSize()` | Set text size | `float` value in pixels |
| `setAntiAlias()` | Enable smoothing | `true`/`false` |

---

## Tips and Best Practices

### Performance Tips

1. **Reuse Paint objects** - Create once, reuse many times
```java
private Paint paint = new Paint(); // Class member
```

2. **Avoid object creation in onDraw()** - Pre-allocate objects
```java
// DON'T do this in onDraw()
Path path = new Path(); // ❌

// DO this instead
private Path path = new Path(); // ✓ Class member
```

3. **Use hardware acceleration** - Add to manifest
```xml
<application android:hardwareAccelerated="true">
```

4. **Clip drawing regions** - Only draw what's visible
```java
canvas.clipRect(visibleRect);
```

### Drawing Tips

1. **Always use anti-aliasing** for smooth edges
```java
paint.setAntiAlias(true);
```

2. **Use appropriate Paint style** for your needs
```java
paint.setStyle(Paint.Style.FILL);    // For solid shapes
paint.setStyle(Paint.Style.STROKE);  // For outlines
```

3. **Remember coordinate system**
   - Origin (0,0) is top-left
   - X increases to the right
   - Y increases downward

4. **Use save/restore for transformations**
```java
canvas.save();
// Transformation code
canvas.restore();
```

---

## Resources

### Official Documentation
- [Canvas API Reference](https://developer.android.com/reference/android/graphics/Canvas)
- [Paint API Reference](https://developer.android.com/reference/android/graphics/Paint)
- [Path API Reference](https://developer.android.com/reference/android/graphics/Path)

### Learning Resources
- [Android Graphics Overview](https://developer.android.com/guide/topics/graphics)
- [Custom Drawing Tutorial](https://developer.android.com/training/custom-views/custom-drawing)

---

*Last Updated: November 11, 2025*
