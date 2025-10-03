# Flutter Advanced Topics Guide

## Table of Contents
1. [Animations](#animations)
2. [Custom Widgets and Painters](#custom-widgets-and-painters)
3. [Platform Integration](#platform-integration)
4. [Performance Optimization](#performance-optimization)

## Animations

### Basic Animations
```dart
import 'package:flutter/material.dart';

// AnimationController with SingleTickerProviderStateMixin
class BasicAnimationScreen extends StatefulWidget {
  @override
  _BasicAnimationScreenState createState() => _BasicAnimationScreenState();
}

class _BasicAnimationScreenState extends State<BasicAnimationScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  late Animation<Color?> _colorAnimation;
  late Animation<Offset> _slideAnimation;
  
  @override
  void initState() {
    super.initState();
    
    // Create animation controller
    _controller = AnimationController(
      duration: Duration(seconds: 2),
      vsync: this,
    );
    
    // Create different types of animations
    _animation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.bounceInOut,
    ));
    
    _colorAnimation = ColorTween(
      begin: Colors.blue,
      end: Colors.red,
    ).animate(_controller);
    
    _slideAnimation = Tween<Offset>(
      begin: Offset(-1.0, 0.0),
      end: Offset.zero,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.elasticOut,
    ));
    
    // Start animation
    _controller.forward();
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Basic Animations')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Fade and scale animation
            AnimatedBuilder(
              animation: _animation,
              builder: (context, child) {
                return Opacity(
                  opacity: _animation.value,
                  child: Transform.scale(
                    scale: _animation.value,
                    child: Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: _colorAnimation.value,
                        borderRadius: BorderRadius.circular(50),
                      ),
                    ),
                  ),
                );
              },
            ),
            
            SizedBox(height: 50),
            
            // Slide animation
            SlideTransition(
              position: _slideAnimation,
              child: Card(
                child: Padding(
                  padding: EdgeInsets.all(20),
                  child: Text(
                    'Sliding Text',
                    style: TextStyle(fontSize: 18),
                  ),
                ),
              ),
            ),
            
            SizedBox(height: 50),
            
            // Control buttons
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => _controller.forward(),
                  child: Text('Forward'),
                ),
                SizedBox(width: 10),
                ElevatedButton(
                  onPressed: () => _controller.reverse(),
                  child: Text('Reverse'),
                ),
                SizedBox(width: 10),
                ElevatedButton(
                  onPressed: () => _controller.repeat(),
                  child: Text('Repeat'),
                ),
                SizedBox(width: 10),
                ElevatedButton(
                  onPressed: () => _controller.stop(),
                  child: Text('Stop'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### Implicit Animations
```dart
class ImplicitAnimationsScreen extends StatefulWidget {
  @override
  _ImplicitAnimationsScreenState createState() => _ImplicitAnimationsScreenState();
}

class _ImplicitAnimationsScreenState extends State<ImplicitAnimationsScreen> {
  bool _isExpanded = false;
  double _opacity = 1.0;
  Color _color = Colors.blue;
  EdgeInsets _padding = EdgeInsets.all(8.0);
  BorderRadius _borderRadius = BorderRadius.circular(8.0);
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Implicit Animations')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // AnimatedContainer
            GestureDetector(
              onTap: () {
                setState(() {
                  _isExpanded = !_isExpanded;
                  _color = _isExpanded ? Colors.red : Colors.blue;
                  _borderRadius = _isExpanded 
                      ? BorderRadius.circular(50.0)
                      : BorderRadius.circular(8.0);
                });
              },
              child: AnimatedContainer(
                duration: Duration(milliseconds: 500),
                curve: Curves.easeInOut,
                width: _isExpanded ? 200 : 100,
                height: _isExpanded ? 200 : 100,
                decoration: BoxDecoration(
                  color: _color,
                  borderRadius: _borderRadius,
                ),
                child: Center(
                  child: Text(
                    'Tap me!',
                    style: TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
            
            SizedBox(height: 30),
            
            // AnimatedOpacity
            AnimatedOpacity(
              opacity: _opacity,
              duration: Duration(milliseconds: 500),
              child: Container(
                width: 150,
                height: 150,
                color: Colors.green,
                child: Center(
                  child: Text(
                    'Opacity\nAnimation',
                    textAlign: TextAlign.center,
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
            
            SizedBox(height: 20),
            
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _opacity = _opacity == 1.0 ? 0.0 : 1.0;
                });
              },
              child: Text('Toggle Opacity'),
            ),
            
            SizedBox(height: 30),
            
            // AnimatedPadding
            AnimatedPadding(
              duration: Duration(milliseconds: 500),
              padding: _padding,
              child: Container(
                color: Colors.orange,
                child: Text(
                  'Animated Padding',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 16,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
            
            SizedBox(height: 20),
            
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _padding = _padding == EdgeInsets.all(8.0)
                      ? EdgeInsets.all(40.0)
                      : EdgeInsets.all(8.0);
                });
              },
              child: Text('Toggle Padding'),
            ),
            
            SizedBox(height: 30),
            
            // AnimatedSwitcher
            AnimatedSwitcher(
              duration: Duration(milliseconds: 500),
              transitionBuilder: (Widget child, Animation<double> animation) {
                return ScaleTransition(scale: animation, child: child);
              },
              child: _isExpanded
                  ? Icon(
                      Icons.star,
                      key: ValueKey('star'),
                      size: 50,
                      color: Colors.amber,
                    )
                  : Icon(
                      Icons.favorite,
                      key: ValueKey('heart'),
                      size: 50,
                      color: Colors.red,
                    ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Complex Animations - Staggered Animation
```dart
class StaggeredAnimationScreen extends StatefulWidget {
  @override
  _StaggeredAnimationScreenState createState() => _StaggeredAnimationScreenState();
}

class _StaggeredAnimationScreenState extends State<StaggeredAnimationScreen>
    with TickerProviderStateMixin {
  late AnimationController _controller;
  
  late Animation<double> _containerGrow;
  late Animation<EdgeInsets> _containerPadding;
  late Animation<BorderRadius?> _containerBorderRadius;
  late Animation<Color?> _containerColor;
  late Animation<double> _titleOpacity;
  late Animation<double> _textOpacity;
  late Animation<double> _buttonOpacity;
  
  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: Duration(seconds: 3),
      vsync: this,
    );
    
    // Create staggered animations with different intervals
    _containerGrow = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.0, 0.4, curve: Curves.ease),
    ));
    
    _containerPadding = EdgeInsetsTween(
      begin: EdgeInsets.all(0),
      end: EdgeInsets.all(20),
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.2, 0.6, curve: Curves.ease),
    ));
    
    _containerBorderRadius = BorderRadiusTween(
      begin: BorderRadius.circular(0),
      end: BorderRadius.circular(20),
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.3, 0.7, curve: Curves.ease),
    ));
    
    _containerColor = ColorTween(
      begin: Colors.transparent,
      end: Colors.blue.shade100,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.4, 0.8, curve: Curves.ease),
    ));
    
    _titleOpacity = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.5, 0.7, curve: Curves.ease),
    ));
    
    _textOpacity = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.6, 0.8, curve: Curves.ease),
    ));
    
    _buttonOpacity = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(0.7, 1.0, curve: Curves.ease),
    ));
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Staggered Animation')),
      body: Center(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Transform.scale(
              scale: _containerGrow.value,
              child: Container(
                width: 300,
                padding: _containerPadding.value,
                decoration: BoxDecoration(
                  color: _containerColor.value,
                  borderRadius: _containerBorderRadius.value,
                  border: Border.all(color: Colors.blue, width: 2),
                ),
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Opacity(
                      opacity: _titleOpacity.value,
                      child: Text(
                        'Staggered Animation',
                        style: TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                          color: Colors.blue.shade700,
                        ),
                      ),
                    ),
                    SizedBox(height: 10),
                    Opacity(
                      opacity: _textOpacity.value,
                      child: Text(
                        'This is an example of a staggered animation where different elements animate at different times.',
                        textAlign: TextAlign.center,
                        style: TextStyle(fontSize: 16),
                      ),
                    ),
                    SizedBox(height: 20),
                    Opacity(
                      opacity: _buttonOpacity.value,
                      child: ElevatedButton(
                        onPressed: () {
                          if (_controller.isCompleted) {
                            _controller.reverse();
                          } else {
                            _controller.forward();
                          }
                        },
                        child: Text('Animate'),
                      ),
                    ),
                  ],
                ),
              ),
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          if (_controller.isCompleted) {
            _controller.reverse();
          } else {
            _controller.forward();
          }
        },
        child: Icon(Icons.play_arrow),
      ),
    );
  }
}
```

## Custom Widgets and Painters

### Custom Painter
```dart
class CustomPainterScreen extends StatefulWidget {
  @override
  _CustomPainterScreenState createState() => _CustomPainterScreenState();
}

class _CustomPainterScreenState extends State<CustomPainterScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(seconds: 2),
      vsync: this,
    )..repeat();
    
    _animation = Tween<double>(
      begin: 0.0,
      end: 1.0,
    ).animate(_controller);
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Custom Painter')),
      body: Column(
        children: [
          // Animated Circle Progress
          Expanded(
            child: Center(
              child: AnimatedBuilder(
                animation: _animation,
                builder: (context, child) {
                  return Container(
                    width: 200,
                    height: 200,
                    child: CustomPaint(
                      painter: CircleProgressPainter(_animation.value),
                    ),
                  );
                },
              ),
            ),
          ),
          
          // Static Chart
          Expanded(
            child: Container(
              margin: EdgeInsets.all(20),
              child: CustomPaint(
                painter: BarChartPainter([
                  ChartData('Jan', 30),
                  ChartData('Feb', 45),
                  ChartData('Mar', 60),
                  ChartData('Apr', 35),
                  ChartData('May', 70),
                ]),
                child: Container(),
              ),
            ),
          ),
          
          // Star Shape
          Expanded(
            child: Center(
              child: Container(
                width: 150,
                height: 150,
                child: CustomPaint(
                  painter: StarPainter(),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// Circle Progress Painter
class CircleProgressPainter extends CustomPainter {
  final double progress;
  
  CircleProgressPainter(this.progress);
  
  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 10;
    
    // Background circle
    final backgroundPaint = Paint()
      ..color = Colors.grey.shade300
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8.0;
    
    canvas.drawCircle(center, radius, backgroundPaint);
    
    // Progress arc
    final progressPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8.0
      ..strokeCap = StrokeCap.round;
    
    final sweepAngle = 2 * math.pi * progress;
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2, // Start from top
      sweepAngle,
      false,
      progressPaint,
    );
    
    // Progress text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(progress * 100).toInt()}%',
        style: TextStyle(
          color: Colors.black,
          fontSize: 24,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );
  }
  
  @override
  bool shouldRepaint(CircleProgressPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}

// Bar Chart Painter
class ChartData {
  final String label;
  final double value;
  
  ChartData(this.label, this.value);
}

class BarChartPainter extends CustomPainter {
  final List<ChartData> data;
  
  BarChartPainter(this.data);
  
  @override
  void paint(Canvas canvas, Size size) {
    final barWidth = size.width / (data.length * 2);
    final maxValue = data.map((d) => d.value).reduce(math.max);
    
    for (int i = 0; i < data.length; i++) {
      final barHeight = (data[i].value / maxValue) * (size.height - 50);
      final x = i * barWidth * 2 + barWidth / 2;
      final y = size.height - barHeight - 30;
      
      // Draw bar
      final barPaint = Paint()
        ..color = Colors.blue.withOpacity(0.7)
        ..style = PaintingStyle.fill;
      
      canvas.drawRect(
        Rect.fromLTWH(x, y, barWidth, barHeight),
        barPaint,
      );
      
      // Draw value text
      final valuePainter = TextPainter(
        text: TextSpan(
          text: data[i].value.toInt().toString(),
          style: TextStyle(
            color: Colors.black,
            fontSize: 12,
            fontWeight: FontWeight.bold,
          ),
        ),
        textDirection: TextDirection.ltr,
      );
      valuePainter.layout();
      valuePainter.paint(
        canvas,
        Offset(x + barWidth / 2 - valuePainter.width / 2, y - 20),
      );
      
      // Draw label
      final labelPainter = TextPainter(
        text: TextSpan(
          text: data[i].label,
          style: TextStyle(
            color: Colors.black,
            fontSize: 12,
          ),
        ),
        textDirection: TextDirection.ltr,
      );
      labelPainter.layout();
      labelPainter.paint(
        canvas,
        Offset(
          x + barWidth / 2 - labelPainter.width / 2,
          size.height - 25,
        ),
      );
    }
  }
  
  @override
  bool shouldRepaint(BarChartPainter oldDelegate) {
    return oldDelegate.data != data;
  }
}

// Star Painter
class StarPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.amber
      ..style = PaintingStyle.fill;
    
    final path = Path();
    final center = Offset(size.width / 2, size.height / 2);
    final outerRadius = size.width / 2 - 10;
    final innerRadius = outerRadius / 2;
    
    // Create 5-pointed star
    for (int i = 0; i < 10; i++) {
      final angle = (i * math.pi) / 5;
      final radius = i.isEven ? outerRadius : innerRadius;
      final x = center.dx + radius * math.cos(angle - math.pi / 2);
      final y = center.dy + radius * math.sin(angle - math.pi / 2);
      
      if (i == 0) {
        path.moveTo(x, y);
      } else {
        path.lineTo(x, y);
      }
    }
    path.close();
    
    canvas.drawPath(path, paint);
    
    // Add border
    final borderPaint = Paint()
      ..color = Colors.orange
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2.0;
    
    canvas.drawPath(path, borderPaint);
  }
  
  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

### Custom Widgets
```dart
// Custom Button Widget
class CustomButton extends StatefulWidget {
  final String text;
  final VoidCallback? onPressed;
  final Color? color;
  final Color? textColor;
  final double? width;
  final double? height;
  final bool isLoading;
  
  const CustomButton({
    Key? key,
    required this.text,
    this.onPressed,
    this.color,
    this.textColor,
    this.width,
    this.height,
    this.isLoading = false,
  }) : super(key: key);
  
  @override
  _CustomButtonState createState() => _CustomButtonState();
}

class _CustomButtonState extends State<CustomButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  bool _isPressed = false;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 100),
      vsync: this,
    );
    _scaleAnimation = Tween<double>(
      begin: 1.0,
      end: 0.95,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    ));
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) {
        setState(() => _isPressed = true);
        _controller.forward();
      },
      onTapUp: (_) {
        setState(() => _isPressed = false);
        _controller.reverse();
        if (widget.onPressed != null && !widget.isLoading) {
          widget.onPressed!();
        }
      },
      onTapCancel: () {
        setState(() => _isPressed = false);
        _controller.reverse();
      },
      child: AnimatedBuilder(
        animation: _scaleAnimation,
        builder: (context, child) {
          return Transform.scale(
            scale: _scaleAnimation.value,
            child: Container(
              width: widget.width ?? 200,
              height: widget.height ?? 50,
              decoration: BoxDecoration(
                color: widget.color ?? Theme.of(context).primaryColor,
                borderRadius: BorderRadius.circular(25),
                boxShadow: _isPressed
                    ? []
                    : [
                        BoxShadow(
                          color: Colors.black.withOpacity(0.2),
                          offset: Offset(0, 4),
                          blurRadius: 8,
                        ),
                      ],
              ),
              child: Center(
                child: widget.isLoading
                    ? SizedBox(
                        width: 20,
                        height: 20,
                        child: CircularProgressIndicator(
                          strokeWidth: 2,
                          valueColor: AlwaysStoppedAnimation<Color>(
                            widget.textColor ?? Colors.white,
                          ),
                        ),
                      )
                    : Text(
                        widget.text,
                        style: TextStyle(
                          color: widget.textColor ?? Colors.white,
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Custom Input Field
class CustomTextField extends StatefulWidget {
  final String? hintText;
  final String? labelText;
  final TextEditingController? controller;
  final String? Function(String?)? validator;
  final bool obscureText;
  final TextInputType? keyboardType;
  final Widget? prefixIcon;
  final Widget? suffixIcon;
  final bool enabled;
  final int? maxLines;
  final Function(String)? onChanged;
  
  const CustomTextField({
    Key? key,
    this.hintText,
    this.labelText,
    this.controller,
    this.validator,
    this.obscureText = false,
    this.keyboardType,
    this.prefixIcon,
    this.suffixIcon,
    this.enabled = true,
    this.maxLines = 1,
    this.onChanged,
  }) : super(key: key);
  
  @override
  _CustomTextFieldState createState() => _CustomTextFieldState();
}

class _CustomTextFieldState extends State<CustomTextField>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Color?> _borderColorAnimation;
  late FocusNode _focusNode;
  bool _isFocused = false;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 200),
      vsync: this,
    );
    _borderColorAnimation = ColorTween(
      begin: Colors.grey.shade300,
      end: Theme.of(context).primaryColor,
    ).animate(_controller);
    
    _focusNode = FocusNode();
    _focusNode.addListener(_onFocusChange);
  }
  
  void _onFocusChange() {
    setState(() {
      _isFocused = _focusNode.hasFocus;
    });
    
    if (_isFocused) {
      _controller.forward();
    } else {
      _controller.reverse();
    }
  }
  
  @override
  void dispose() {
    _controller.dispose();
    _focusNode.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _borderColorAnimation,
      builder: (context, child) {
        return Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (widget.labelText != null)
              Padding(
                padding: EdgeInsets.only(bottom: 8),
                child: Text(
                  widget.labelText!,
                  style: TextStyle(
                    fontSize: 14,
                    fontWeight: FontWeight.w500,
                    color: _isFocused
                        ? Theme.of(context).primaryColor
                        : Colors.grey.shade600,
                  ),
                ),
              ),
            Container(
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(12),
                border: Border.all(
                  color: _borderColorAnimation.value!,
                  width: _isFocused ? 2 : 1,
                ),
              ),
              child: TextFormField(
                controller: widget.controller,
                focusNode: _focusNode,
                validator: widget.validator,
                obscureText: widget.obscureText,
                keyboardType: widget.keyboardType,
                enabled: widget.enabled,
                maxLines: widget.maxLines,
                onChanged: widget.onChanged,
                decoration: InputDecoration(
                  hintText: widget.hintText,
                  prefixIcon: widget.prefixIcon,
                  suffixIcon: widget.suffixIcon,
                  border: InputBorder.none,
                  contentPadding: EdgeInsets.symmetric(
                    horizontal: 16,
                    vertical: 12,
                  ),
                ),
              ),
            ),
          ],
        );
      },
    );
  }
}

// Custom Card Widget
class CustomCard extends StatelessWidget {
  final Widget child;
  final EdgeInsets? padding;
  final EdgeInsets? margin;
  final Color? color;
  final double? elevation;
  final VoidCallback? onTap;
  final BorderRadius? borderRadius;
  
  const CustomCard({
    Key? key,
    required this.child,
    this.padding,
    this.margin,
    this.color,
    this.elevation,
    this.onTap,
    this.borderRadius,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      margin: margin ?? EdgeInsets.all(8),
      child: Material(
        color: color ?? Colors.white,
        elevation: elevation ?? 4,
        borderRadius: borderRadius ?? BorderRadius.circular(12),
        child: InkWell(
          onTap: onTap,
          borderRadius: borderRadius ?? BorderRadius.circular(12),
          child: Padding(
            padding: padding ?? EdgeInsets.all(16),
            child: child,
          ),
        ),
      ),
    );
  }
}
```

This guide covers advanced Flutter animations, custom painters for drawing graphics, and reusable custom widgets with enhanced functionality and animations.