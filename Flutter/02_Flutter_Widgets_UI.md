# Flutter Widgets and UI Guide

## Table of Contents
1. [Introduction to Widgets](#introduction-to-widgets)
2. [Basic Widgets](#basic-widgets)
3. [Layout Widgets](#layout-widgets)
4. [Material Design Widgets](#material-design-widgets)
5. [Input Widgets](#input-widgets)
6. [Display Widgets](#display-widgets)
7. [Styling and Theming](#styling-and-theming)
8. [Custom Widgets](#custom-widgets)

## Introduction to Widgets

In Flutter, everything is a widget. Widgets are the building blocks of Flutter applications and describe what their view should look like given their current configuration and state. Understanding widgets is fundamental to mastering Flutter development.

### The Widget Philosophy

Flutter's approach to UI is fundamentally different from traditional mobile development:

1. **Declarative UI**: Instead of telling the system HOW to change the UI, you describe WHAT the UI should look like for any given state
2. **Composition**: Complex UIs are built by composing simple widgets together
3. **Immutability**: Widgets are immutable configurations that describe the UI at a specific moment in time

### Widget Types and Their Purpose

#### **StatelessWidget**: The Building Blocks
StatelessWidget is used for UI elements that never change. Once built, they remain the same until they're replaced entirely.

**Characteristics:**
- Immutable (cannot change after creation)
- No internal state to manage
- Rebuilds only when parent widget changes
- More efficient in terms of performance
- Perfect for static content

**When to use StatelessWidget:**
- Displaying static text, images, or icons
- Layout widgets that don't change
- UI components that only depend on external data
- Reusable components with fixed behavior

#### **StatefulWidget**: The Dynamic Elements
StatefulWidget is used for UI elements that can change over time based on user interaction or other factors.

**Characteristics:**
- Can maintain mutable state
- Can trigger rebuilds using setState()
- Has a lifecycle with multiple methods
- Slightly more overhead than StatelessWidget

**When to use StatefulWidget:**
- Interactive elements (buttons, forms, checkboxes)
- Animations and transitions
- Real-time data display
- User input handling

#### **InheritedWidget**: The Data Propagators
InheritedWidget is used to efficiently propagate information down the widget tree without explicitly passing data through every widget constructor.

**Use cases:**
- Theme data
- User authentication state
- App-wide configuration
- Shared data across multiple widgets

### Widget Lifecycle Deep Dive

Understanding the widget lifecycle is crucial for managing resources and building efficient apps:

#### StatelessWidget Lifecycle
```dart
class MyStatelessWidget extends StatelessWidget {
  const MyStatelessWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    // This method is called every time the widget needs to be rendered
    // It should be pure (no side effects) and fast
    print('StatelessWidget build called');
    return Container(
      child: Text('I am stateless'),
    );
  }
}
```

#### StatefulWidget Lifecycle (Detailed)
```dart
class MyStatefulWidget extends StatefulWidget {
  final String title;
  
  const MyStatefulWidget({Key? key, required this.title}) : super(key: key);
  
  @override
  _MyStatefulWidgetState createState() {
    print('createState called');
    return _MyStatefulWidgetState();
  }
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  String _message = '';
  
  @override
  void initState() {
    // Called once when the widget is inserted into the tree
    // Perfect for one-time initialization
    super.initState();
    print('initState called');
    _message = 'Widget initialized: ${widget.title}';
    
    // Good place to:
    // - Initialize controllers
    // - Start animations
    // - Subscribe to streams
    // - Set up listeners
  }
  
  @override
  void didChangeDependencies() {
    // Called when a dependency of this State object changes
    // Called after initState and whenever dependencies change
    super.didChangeDependencies();
    print('didChangeDependencies called');
    
    // Good place to:
    // - React to theme changes
    // - Respond to locale changes
    // - Handle MediaQuery changes
  }
  
  @override
  Widget build(BuildContext context) {
    // Called whenever the widget needs to be built/rebuilt
    // Should be pure and fast
    print('build called');
    
    return Container(
      padding: EdgeInsets.all(16),
      child: Column(
        children: [
          Text(widget.title), // Access widget properties
          Text(_message),     // Access state
          ElevatedButton(
            onPressed: () {
              setState(() {
                _message = 'Button pressed at ${DateTime.now()}';
              });
            },
            child: Text('Update State'),
          ),
        ],
      ),
    );
  }
  
  @override
  void didUpdateWidget(MyStatefulWidget oldWidget) {
    // Called when parent widget changes and needs to update this widget
    super.didUpdateWidget(oldWidget);
    print('didUpdateWidget called');
    
    // Compare old and new widget properties
    if (oldWidget.title != widget.title) {
      setState(() {
        _message = 'Title changed to: ${widget.title}';
      });
    }
  }
  
  @override
  void setState(VoidCallback fn) {
    // Called to notify framework that state has changed
    print('setState called');
    super.setState(fn);
    // This triggers a rebuild of the widget
  }
  
  @override
  void deactivate() {
    // Called when widget is removed from tree (temporarily)
    super.deactivate();
    print('deactivate called');
    
    // Widget might be reinserted later
    // Rarely overridden
  }
  
  @override
  void dispose() {
    // Called when widget is permanently removed from tree
    super.dispose();
    print('dispose called');
    
    // Clean up resources:
    // - Dispose controllers
    // - Cancel timers
    // - Close streams
    // - Remove listeners
  }
}
```

### Widget Tree Optimization

The widget tree is rebuilt frequently in Flutter, so optimization is important:

#### 1. Use const constructors
```dart
// Good: Const widgets are cached and reused
const Text('Static text')
const Icon(Icons.star)
const SizedBox(height: 16)

// Bad: New instances created every build
Text('Static text')
Icon(Icons.star)
SizedBox(height: 16)
```

#### 2. Extract widgets appropriately
```dart
// Good: Extract stable parts into separate widgets
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Header(), // Const widget, won't rebuild
        DynamicContent(), // Only this rebuilds when needed
      ],
    );
  }
}

class Header extends StatelessWidget {
  const Header({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Text('I never change');
  }
}
```

#### 3. Use keys wisely
```dart
// Keys help Flutter identify widgets during rebuilds
ListView(
  children: items.map((item) => 
    ListTile(
      key: ValueKey(item.id), // Unique key for each item
      title: Text(item.name),
    )
  ).toList(),
)
```

## Basic Widgets

Basic widgets are the fundamental building blocks that you'll use in almost every Flutter application. These widgets handle the most common UI needs: displaying text, images, and icons.

### Text Widget - The Foundation of Content

The Text widget is arguably the most important widget in Flutter. It displays text content and provides extensive customization options.

#### Basic Text Usage
```dart
class TextExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Text Widget Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Basic text - simplest form
            Text('Hello, Flutter!'),
            
            SizedBox(height: 16),
            
            // 2. Styled text with custom appearance
            Text(
              'This is styled text',
              style: TextStyle(
                fontSize: 24,              // Size of the text
                fontWeight: FontWeight.bold, // Make it bold
                color: Colors.blue,        // Text color
                letterSpacing: 1.2,        // Space between letters
                wordSpacing: 2.0,          // Space between words
                height: 1.5,               // Line height multiplier
                decoration: TextDecoration.underline, // Underline text
                decorationColor: Colors.red,    // Color of underline
                decorationThickness: 2.0,       // Thickness of underline
                fontStyle: FontStyle.italic,    // Make it italic
              ),
            ),
            
            SizedBox(height: 16),
            
            // 3. Text with custom font family
            Text(
              'Custom Font Text',
              style: TextStyle(
                fontSize: 20,
                fontFamily: 'Roboto',  // Custom font (must be added to pubspec.yaml)
                fontWeight: FontWeight.w300,
                color: Colors.purple,
              ),
            ),
            
            SizedBox(height: 16),
            
            // 4. Rich text with multiple styles in one widget
            RichText(
              text: TextSpan(
                // Default style for all text
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.black,
                ),
                children: [
                  TextSpan(text: 'This is '),
                  TextSpan(
                    text: 'bold text',
                    style: TextStyle(
                      fontWeight: FontWeight.bold,
                      color: Colors.red,
                    ),
                  ),
                  TextSpan(text: ' and this is '),
                  TextSpan(
                    text: 'italic text',
                    style: TextStyle(
                      fontStyle: FontStyle.italic,
                      color: Colors.blue,
                    ),
                  ),
                  TextSpan(text: ' in the same widget.'),
                ],
              ),
            ),
            
            SizedBox(height: 16),
            
            // 5. Text with overflow handling
            Container(
              width: 200, // Constrain width to demonstrate overflow
              child: Text(
                'This is a very long text that will overflow if not handled properly.',
                overflow: TextOverflow.ellipsis, // Show ... when text overflows
                maxLines: 2, // Limit to 2 lines
                style: TextStyle(fontSize: 16),
              ),
            ),
            
            SizedBox(height: 16),
            
            // 6. Selectable text (user can copy)
            SelectableText(
              'This text can be selected and copied by the user.',
              style: TextStyle(
                fontSize: 16,
                color: Colors.green,
              ),
            ),
            
            SizedBox(height: 16),
            
            // 7. Text with alignment in a container
            Container(
              width: double.infinity,
              padding: EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(
                'Centered Text',
                textAlign: TextAlign.center,
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.w500,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Image Widget - Displaying Visual Content

Images are crucial for creating engaging user interfaces. Flutter provides several ways to display images with different sources and styling options.

#### Image Sources
1. **Network Images**: Load from internet URLs
2. **Asset Images**: Include with your app bundle
3. **File Images**: Load from device storage
4. **Memory Images**: Load from memory bytes

#### Key Properties Explained

**Size Control:**
- `width` & `height`: Define image dimensions
- `fit`: How image should resize to fit available space

**BoxFit Options:**
- `BoxFit.cover`: Fills container, may crop image
- `BoxFit.contain`: Fits entirely within container
- `BoxFit.fill`: Stretches to fill container (may distort)
- `BoxFit.fitWidth`: Fits width, height may overflow
- `BoxFit.fitHeight`: Fits height, width may overflow

**Error & Loading Handling:**
- `loadingBuilder`: Show progress while loading
- `errorBuilder`: Show fallback when loading fails

```dart
class ImageExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Image Widget Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. Network image with comprehensive error handling
            Text(
              'Network Image with Loading & Error Handling:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Image.network(
              'https://picsum.photos/300/200',
              width: 300,
              height: 200,
              fit: BoxFit.cover,
              loadingBuilder: (context, child, loadingProgress) {
                // Show loading indicator while image loads
                if (loadingProgress == null) return child;
                return Container(
                  width: 300,
                  height: 200,
                  child: Center(
                    child: CircularProgressIndicator(
                      value: loadingProgress.expectedTotalBytes != null
                          ? loadingProgress.cumulativeBytesLoaded /
                              loadingProgress.expectedTotalBytes!
                          : null,
                    ),
                  ),
                );
              },
              errorBuilder: (context, error, stackTrace) {
                // Show error widget if image fails to load
                return Container(
                  width: 300,
                  height: 200,
                  color: Colors.grey[300],
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(Icons.error, size: 50, color: Colors.red),
                      Text('Failed to load image'),
                    ],
                  ),
                );
              },
            ),
            
            SizedBox(height: 20),
            
            // 2. Asset image setup instructions
            Text(
              'Asset Image Setup:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Container(
              padding: EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.blue[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.blue[200]!),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'To use asset images:',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  SizedBox(height: 8),
                  Text('1. Create assets/images/ folder in project root'),
                  Text('2. Add images to the folder'),
                  Text('3. Update pubspec.yaml:'),
                  SizedBox(height: 8),
                  Container(
                    padding: EdgeInsets.all(8),
                    color: Colors.grey[200],
                    child: Text(
                      'flutter:\n  assets:\n    - assets/images/',
                      style: TextStyle(fontFamily: 'monospace'),
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('4. Use: Image.asset(\'assets/images/photo.jpg\')'),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            // 3. Circular image with border (profile picture style)
            Text(
              'Circular Profile Image:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Container(
              width: 120,
              height: 120,
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                border: Border.all(color: Colors.blue, width: 3),
                image: DecorationImage(
                  image: NetworkImage('https://picsum.photos/seed/profile/120/120'),
                  fit: BoxFit.cover,
                ),
                // Add shadow for depth
                boxShadow: [
                  BoxShadow(
                    color: Colors.grey.withOpacity(0.5),
                    spreadRadius: 2,
                    blurRadius: 5,
                    offset: Offset(0, 3),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            // 4. Rounded rectangle image using ClipRRect
            Text(
              'Rounded Corner Image:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            ClipRRect(
              borderRadius: BorderRadius.circular(15),
              child: Image.network(
                'https://picsum.photos/seed/rounded/250/150',
                width: 250,
                height: 150,
                fit: BoxFit.cover,
              ),
            ),
            
            SizedBox(height: 20),
            
            // 5. BoxFit comparison examples
            Text(
              'BoxFit Comparison:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildBoxFitExample('cover', BoxFit.cover),
                _buildBoxFitExample('contain', BoxFit.contain),
                _buildBoxFitExample('fill', BoxFit.fill),
              ],
            ),
            
            SizedBox(height: 20),
            
            // 6. Image with overlay and gradient
            Text(
              'Image with Text Overlay:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Container(
              width: 300,
              height: 200,
              child: Stack(
                children: [
                  // Background image
                  Container(
                    width: 300,
                    height: 200,
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(12),
                      image: DecorationImage(
                        image: NetworkImage('https://picsum.photos/seed/overlay/300/200'),
                        fit: BoxFit.cover,
                      ),
                    ),
                  ),
                  // Gradient overlay
                  Container(
                    width: 300,
                    height: 200,
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(12),
                      gradient: LinearGradient(
                        begin: Alignment.bottomCenter,
                        end: Alignment.topCenter,
                        colors: [
                          Colors.black.withOpacity(0.7),
                          Colors.transparent,
                        ],
                      ),
                    ),
                  ),
                  // Text overlay
                  Positioned(
                    bottom: 16,
                    left: 16,
                    right: 16,
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          'Beautiful Landscape',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          'Perfect example of image overlay technique',
                          style: TextStyle(
                            color: Colors.white70,
                            fontSize: 14,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  // Helper method to build BoxFit examples
  Widget _buildBoxFitExample(String label, BoxFit fit) {
    return Column(
      children: [
        Container(
          width: 80,
          height: 80,
          decoration: BoxDecoration(
            border: Border.all(color: Colors.grey),
            borderRadius: BorderRadius.circular(8),
          ),
          child: ClipRRect(
            borderRadius: BorderRadius.circular(7),
            child: Image.network(
              'https://picsum.photos/seed/$label/100/200',
              fit: fit,
            ),
          ),
        ),
        SizedBox(height: 4),
        Text(label, style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold)),
      ],
    );
  }
}
```

#### Performance Tips for Images

1. **Optimize Network Images:**
   - Use appropriate image sizes for your UI
   - Consider using cached_network_image package for better caching
   - Implement loading placeholders

2. **Asset Image Best Practices:**
   - Provide multiple resolutions (1x, 2x, 3x folders)
   - Use appropriate formats (WebP for smaller sizes)
   - Compress images before adding to assets

3. **Memory Management:**
   - Large images consume significant memory
   - Use appropriate BoxFit to avoid unnecessary scaling
   - Consider using Image.memory for dynamic images

```dart
// Example: Creating resolution-aware asset images
// assets/images/2.0x/logo.png  (for high-DPI screens)
// assets/images/3.0x/logo.png  (for very high-DPI screens)
// assets/images/logo.png       (base resolution)

Image.asset('assets/images/logo.png') // Flutter automatically picks appropriate resolution
```

### Icon Widget - Visual Symbols and Actions

Icons are essential UI elements that provide visual cues and represent actions or concepts. Flutter includes a comprehensive set of Material Design icons and supports custom icons.

#### Icon Sources
1. **Material Icons**: Built-in icons from Google's Material Design
2. **Cupertino Icons**: iOS-style icons (requires cupertino_icons package)
3. **Custom Icons**: Your own icon fonts or image-based icons
4. **Font Awesome**: Popular icon library (requires font_awesome_flutter package)

#### Key Properties Explained

**Appearance:**
- `size`: Icon size in logical pixels (double)
- `color`: Icon color (can be null for default theme color)

**Accessibility:**
- `semanticLabel`: Screen reader description
- `textDirection`: For directional icons (LTR/RTL)

```dart
class IconExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Icon Widget Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Basic Material Design icons
            Text(
              'Basic Material Design Icons:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Column(
                  children: [
                    Icon(Icons.home, size: 32),
                    SizedBox(height: 4),
                    Text('home', style: TextStyle(fontSize: 12)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.yellow[700], size: 32),
                    SizedBox(height: 4),
                    Text('star', style: TextStyle(fontSize: 12)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.favorite, color: Colors.red, size: 32),
                    SizedBox(height: 4),
                    Text('favorite', style: TextStyle(fontSize: 12)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.settings, color: Colors.grey[600], size: 32),
                    SizedBox(height: 4),
                    Text('settings', style: TextStyle(fontSize: 12)),
                  ],
                ),
              ],
            ),
            
            SizedBox(height: 30),
            
            // 2. Icon sizes comparison
            Text(
              'Icon Size Variations:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 16),
                    SizedBox(height: 4),
                    Text('16px', style: TextStyle(fontSize: 10)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 24),
                    SizedBox(height: 4),
                    Text('24px', style: TextStyle(fontSize: 10)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 32),
                    SizedBox(height: 4),
                    Text('32px', style: TextStyle(fontSize: 10)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 48),
                    SizedBox(height: 4),
                    Text('48px', style: TextStyle(fontSize: 10)),
                  ],
                ),
                Column(
                  children: [
                    Icon(Icons.star, color: Colors.amber, size: 64),
                    SizedBox(height: 4),
                    Text('64px', style: TextStyle(fontSize: 10)),
                  ],
                ),
              ],
            ),
            
            SizedBox(height: 30),
            
            // 3. Icons with custom backgrounds and styling
            Text(
              'Styled Icon Containers:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Circular background
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    shape: BoxShape.circle,
                  ),
                  child: Icon(
                    Icons.notifications,
                    color: Colors.white,
                    size: 24,
                  ),
                ),
                // Rounded rectangle background
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.green,
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Icon(
                    Icons.check,
                    color: Colors.white,
                    size: 24,
                  ),
                ),
                // Gradient background
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    gradient: LinearGradient(
                      colors: [Colors.purple, Colors.pink],
                    ),
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Icon(
                    Icons.favorite,
                    color: Colors.white,
                    size: 24,
                  ),
                ),
                // Border style
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    border: Border.all(color: Colors.orange, width: 2),
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Icon(
                    Icons.star,
                    color: Colors.orange,
                    size: 24,
                  ),
                ),
              ],
            ),
            
            SizedBox(height: 30),
            
            // 4. Interactive IconButtons
            Text(
              'Interactive Icon Buttons:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildIconButton(
                  icon: Icons.thumb_up,
                  label: 'Like',
                  color: Colors.blue,
                  onPressed: () => print('Like pressed'),
                ),
                _buildIconButton(
                  icon: Icons.share,
                  label: 'Share',
                  color: Colors.green,
                  onPressed: () => print('Share pressed'),
                ),
                _buildIconButton(
                  icon: Icons.bookmark,
                  label: 'Save',
                  color: Colors.orange,
                  onPressed: () => print('Save pressed'),
                ),
                _buildIconButton(
                  icon: Icons.more_vert,
                  label: 'More',
                  color: Colors.grey[600]!,
                  onPressed: () => print('More pressed'),
                ),
              ],
            ),
            
            SizedBox(height: 30),
            
            // 5. Common icon categories
            Text(
              'Common Icon Categories:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            _buildIconCategory('Navigation', [
              Icons.home,
              Icons.menu,
              Icons.arrow_back,
              Icons.arrow_forward,
              Icons.close,
            ]),
            SizedBox(height: 16),
            _buildIconCategory('Actions', [
              Icons.add,
              Icons.edit,
              Icons.delete,
              Icons.save,
              Icons.search,
            ]),
            SizedBox(height: 16),
            _buildIconCategory('Communication', [
              Icons.call,
              Icons.message,
              Icons.email,
              Icons.notifications,
              Icons.share,
            ]),
            SizedBox(height: 16),
            _buildIconCategory('Media', [
              Icons.play_arrow,
              Icons.pause,
              Icons.stop,
              Icons.volume_up,
              Icons.camera,
            ]),
            
            SizedBox(height: 30),
            
            // 6. Icon with badge (notification count)
            Text(
              'Icon with Badge:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Center(
              child: Stack(
                children: [
                  Icon(
                    Icons.notifications,
                    size: 48,
                    color: Colors.grey[700],
                  ),
                  Positioned(
                    right: 0,
                    top: 0,
                    child: Container(
                      padding: EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                      decoration: BoxDecoration(
                        color: Colors.red,
                        borderRadius: BorderRadius.circular(10),
                      ),
                      constraints: BoxConstraints(
                        minWidth: 20,
                        minHeight: 20,
                      ),
                      child: Text(
                        '3',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 12,
                          fontWeight: FontWeight.bold,
                        ),
                        textAlign: TextAlign.center,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  // Helper method to build icon buttons with labels
  Widget _buildIconButton({
    required IconData icon,
    required String label,
    required Color color,
    required VoidCallback onPressed,
  }) {
    return Column(
      children: [
        IconButton(
          icon: Icon(icon),
          onPressed: onPressed,
          color: color,
          iconSize: 28,
          splashRadius: 24, // Control splash effect size
        ),
        Text(
          label,
          style: TextStyle(fontSize: 12, color: color),
        ),
      ],
    );
  }
  
  // Helper method to build icon categories
  Widget _buildIconCategory(String title, List<IconData> icons) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          title + ':',
          style: TextStyle(fontSize: 16, fontWeight: FontWeight.w600),
        ),
        SizedBox(height: 8),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: icons.map((icon) => Icon(icon, size: 24, color: Colors.grey[700])).toList(),
        ),
      ],
    );
  }
}
```

#### Popular Material Icons Reference

**Navigation Icons:**
- `Icons.home` - Home page
- `Icons.menu` - Hamburger menu
- `Icons.arrow_back` - Back navigation
- `Icons.close` - Close dialog/screen

**Action Icons:**
- `Icons.add` - Add new item
- `Icons.edit` - Edit content
- `Icons.delete` - Delete item
- `Icons.search` - Search functionality
- `Icons.filter_list` - Filter options

**Status Icons:**
- `Icons.check` - Success/completed
- `Icons.error` - Error state
- `Icons.warning` - Warning state
- `Icons.info` - Information

**Media Icons:**
- `Icons.play_arrow` - Play media
- `Icons.pause` - Pause media
- `Icons.volume_up` - Audio controls
- `Icons.camera` - Camera functionality

#### Custom Icons Setup

```dart
// To use custom icons:
// 1. Create icon font or use images
// 2. Add to pubspec.yaml:
// fonts:
//   - family: CustomIcons
//     fonts:
//       - asset: assets/fonts/custom_icons.ttf

// 3. Use in code:
Icon(
  IconData(0xe801, fontFamily: 'CustomIcons'),
  size: 24,
  color: Colors.blue,
)

// For Font Awesome icons:
// Add dependency: font_awesome_flutter: ^10.4.0
// Import: import 'package:font_awesome_flutter/font_awesome_flutter.dart';
// Use: FaIcon(FontAwesomeIcons.github)
```

#### Accessibility Best Practices

```dart
// Always provide semantic labels for screen readers
Icon(
  Icons.star,
  semanticLabel: 'Mark as favorite',
)

// For interactive icons, use IconButton with proper tooltip
IconButton(
  icon: Icon(Icons.delete),
  onPressed: () => _deleteItem(),
  tooltip: 'Delete item', // Shows on long press
)
```

### Container Widget - The Multi-Purpose Layout Tool

Container is one of the most versatile widgets in Flutter. It combines common painting, positioning, and sizing widgets into a single convenient widget.

#### Container's Capabilities
1. **Sizing**: Control width, height, and constraints
2. **Decoration**: Add colors, gradients, borders, shadows, and shapes
3. **Positioning**: Add margins and padding
4. **Transformation**: Apply matrix transformations
5. **Clipping**: Control how child content is clipped

#### Key Properties Explained

**Size Control:**
- `width` & `height`: Explicit dimensions
- `constraints`: Min/max width and height limits

**Spacing:**
- `margin`: Space outside the container (affects layout)
- `padding`: Space inside the container (affects child positioning)

**Visual Styling:**
- `color`: Simple background color (cannot be used with decoration)
- `decoration`: Advanced styling (gradients, borders, shadows, etc.)

**Layout:**
- `alignment`: How to align the child within the container
- `transform`: Apply 2D transformations (rotation, scale, translation)

```dart
class ContainerExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Container Widget Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. Basic container with solid color
            Text(
              'Basic Containers:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.blue,
                  child: Center(
                    child: Text(
                      'Basic',
                      style: TextStyle(color: Colors.white, fontSize: 12),
                    ),
                  ),
                ),
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.red,
                  child: Center(
                    child: Icon(Icons.star, color: Colors.white),
                  ),
                ),
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.green,
                  child: Center(
                    child: Text(
                      '✓',
                      style: TextStyle(color: Colors.white, fontSize: 24),
                    ),
                  ),
                ),
              ],
            ),
            
            SizedBox(height: 24),
            
            // 2. Containers with different decorations
            Text(
              'Decorated Containers:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            
            // Gradient container
            Container(
              width: double.infinity,
              height: 100,
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.purple, Colors.blue, Colors.cyan],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(15),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black26,
                    blurRadius: 10,
                    offset: Offset(0, 5),
                  ),
                ],
              ),
              child: Center(
                child: Text(
                  'Gradient with Shadow',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
            
            SizedBox(height: 16),
            
            // Border styles
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Container(
                  width: 100,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.white,
                    border: Border.all(color: Colors.blue, width: 2),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Center(
                    child: Text('Border', style: TextStyle(color: Colors.blue)),
                  ),
                ),
                Container(
                  width: 100,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.orange[100],
                    border: Border(
                      left: BorderSide(color: Colors.orange, width: 4),
                      top: BorderSide(color: Colors.orange, width: 1),
                      right: BorderSide(color: Colors.orange, width: 1),
                      bottom: BorderSide(color: Colors.orange, width: 1),
                    ),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Center(
                    child: Text('Left Border', style: TextStyle(color: Colors.orange[800])),
                  ),
                ),
              ],
            ),
            
            SizedBox(height: 24),
            
            // 3. Margin and Padding demonstration
            Text(
              'Margin vs Padding:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Container(
              color: Colors.yellow[100], // Outer container to show margin effect
              child: Column(
                children: [
                  // Container with margin
                  Container(
                    margin: EdgeInsets.all(20),
                    padding: EdgeInsets.all(16),
                    decoration: BoxDecoration(
                      color: Colors.blue[100],
                      border: Border.all(color: Colors.blue),
                      borderRadius: BorderRadius.circular(8),
                    ),
                    child: Column(
                      children: [
                        Text(
                          'Margin: 20px (yellow space around)',
                          style: TextStyle(fontWeight: FontWeight.bold),
                        ),
                        SizedBox(height: 8),
                        Container(
                          padding: EdgeInsets.all(12),
                          decoration: BoxDecoration(
                            color: Colors.white,
                            borderRadius: BorderRadius.circular(4),
                          ),
                          child: Text('Padding: 16px (blue space inside)'),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 24),
            
            // 4. Different shapes
            Text(
              'Container Shapes:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Circle
                Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.green,
                    shape: BoxShape.circle,
                  ),
                  child: Icon(Icons.check, color: Colors.white, size: 40),
                ),
                // Rounded rectangle
                Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.orange,
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: Icon(Icons.star, color: Colors.white, size: 40),
                ),
                // Custom border radius
                Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    color: Colors.purple,
                    borderRadius: BorderRadius.only(
                      topLeft: Radius.circular(20),
                      bottomRight: Radius.circular(20),
                    ),
                  ),
                  child: Icon(Icons.favorite, color: Colors.white, size: 40),
                ),
              ],
            ),
            
            SizedBox(height: 24),
            
            // 5. Container with transformations
            Text(
              'Transformed Containers:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Rotated container
                Transform.rotate(
                  angle: 0.3, // radians
                  child: Container(
                    width: 60,
                    height: 60,
                    color: Colors.red,
                    child: Center(
                      child: Text('Rotate', style: TextStyle(color: Colors.white, fontSize: 10)),
                    ),
                  ),
                ),
                // Scaled container
                Transform.scale(
                  scale: 1.2,
                  child: Container(
                    width: 60,
                    height: 60,
                    color: Colors.blue,
                    child: Center(
                      child: Text('Scale', style: TextStyle(color: Colors.white, fontSize: 10)),
                    ),
                  ),
                ),
                // Skewed container
                Transform(
                  transform: Matrix4.skewX(0.2),
                  child: Container(
                    width: 60,
                    height: 60,
                    color: Colors.green,
                    child: Center(
                      child: Text('Skew', style: TextStyle(color: Colors.white, fontSize: 10)),
                    ),
                  ),
                ),
              ],
            ),
            
            SizedBox(height: 24),
            
            // 6. Container constraints demonstration
            Text(
              'Container Constraints:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Container(
              constraints: BoxConstraints(
                minWidth: 100,
                maxWidth: 200,
                minHeight: 50,
                maxHeight: 100,
              ),
              decoration: BoxDecoration(
                color: Colors.cyan[100],
                border: Border.all(color: Colors.cyan),
                borderRadius: BorderRadius.circular(8),
              ),
              child: Padding(
                padding: EdgeInsets.all(12),
                child: Text(
                  'This container has constraints: min 100x50, max 200x100. '
                  'It will grow to fit content but respect these limits.',
                  style: TextStyle(fontSize: 12),
                ),
              ),
            ),
            
            SizedBox(height: 24),
            
            // 7. Advanced decoration effects
            Text(
              'Advanced Effects:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            Container(
              width: double.infinity,
              height: 120,
              decoration: BoxDecoration(
                gradient: RadialGradient(
                  colors: [Colors.pink[200]!, Colors.purple[400]!],
                  center: Alignment.topLeft,
                  radius: 1.5,
                ),
                borderRadius: BorderRadius.circular(20),
                boxShadow: [
                  BoxShadow(
                    color: Colors.purple.withOpacity(0.3),
                    blurRadius: 15,
                    spreadRadius: 5,
                    offset: Offset(0, 8),
                  ),
                ],
              ),
              child: Stack(
                children: [
                  // Background pattern
                  Positioned.fill(
                    child: Container(
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(20),
                        image: DecorationImage(
                          image: NetworkImage('https://picsum.photos/seed/pattern/400/200'),
                          fit: BoxFit.cover,
                          opacity: 0.2,
                        ),
                      ),
                    ),
                  ),
                  // Overlay content
                  Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Text(
                          'Advanced Container',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          'Gradient + Shadow + Background Image',
                          style: TextStyle(
                            color: Colors.white70,
                            fontSize: 12,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Container vs Other Widgets

**When to use Container:**
- Need multiple styling features (color + padding + border)
- Want to add margins around a widget
- Need to transform or constrain a widget
- Creating simple rectangular layouts

**Alternatives for specific needs:**
- `Padding`: Only need padding (more efficient)
- `SizedBox`: Only need fixed size or spacing
- `DecoratedBox`: Only need decoration without size constraints
- `Transform`: Only need transformations

#### Common Container Patterns

```dart
// Card-like container
Container(
  margin: EdgeInsets.all(16),
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(
        color: Colors.grey.withOpacity(0.2),
        spreadRadius: 1,
        blurRadius: 6,
        offset: Offset(0, 3),
      ),
    ],
  ),
  child: Column(
    children: [
      Text('Card Title', style: TextStyle(fontWeight: FontWeight.bold)),
      Text('Card content goes here...'),
    ],
  ),
)

// Button-like container
GestureDetector(
  onTap: () => print('Container button pressed'),
  child: Container(
    padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
    decoration: BoxDecoration(
      gradient: LinearGradient(
        colors: [Colors.blue, Colors.blueAccent],
      ),
      borderRadius: BorderRadius.circular(25),
    ),
    child: Text(
      'Custom Button',
      style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
    ),
  ),
)
```

#### Performance Tips

1. **Use specific widgets when possible**: If you only need padding, use `Padding` widget instead of `Container`
2. **Avoid unnecessary containers**: Don't wrap widgets in containers unless you need the functionality
3. **Cache complex decorations**: For repeated decorations, consider storing `BoxDecoration` in a variable
4. **Be mindful of rebuilds**: Complex decorations can impact performance during animations

## Layout Widgets

### Column and Row
```dart
class ColumnRowExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Column & Row Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Column example
            Container(
              height: 200,
              color: Colors.grey[200],
              child: Column(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.red,
                  ),
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.green,
                  ),
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.blue,
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            // Row example
            Container(
              height: 100,
              color: Colors.grey[200],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.orange,
                  ),
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.purple,
                  ),
                  Container(
                    width: 50,
                    height: 50,
                    color: Colors.teal,
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            // Nested Column and Row
            Container(
              padding: EdgeInsets.all(16),
              color: Colors.grey[100],
              child: Column(
                children: [
                  Text(
                    'User Profile',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 16),
                  Row(
                    children: [
                      CircleAvatar(
                        radius: 30,
                        backgroundImage: NetworkImage(
                          'https://picsum.photos/60/60',
                        ),
                      ),
                      SizedBox(width: 16),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              'John Doe',
                              style: TextStyle(
                                fontSize: 18,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            Text(
                              'Flutter Developer',
                              style: TextStyle(
                                color: Colors.grey[600],
                              ),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Stack Widget
```dart
class StackExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Stack Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Basic stack
            Container(
              height: 200,
              child: Stack(
                children: [
                  Container(
                    width: 200,
                    height: 200,
                    color: Colors.blue,
                  ),
                  Positioned(
                    top: 20,
                    right: 20,
                    child: Container(
                      width: 50,
                      height: 50,
                      color: Colors.red,
                    ),
                  ),
                  Positioned(
                    bottom: 20,
                    left: 20,
                    child: Text(
                      'Stacked Text',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // Profile card with stack
            Card(
              child: Container(
                width: 300,
                height: 200,
                child: Stack(
                  children: [
                    // Background image
                    Container(
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(12),
                        image: DecorationImage(
                          image: NetworkImage('https://picsum.photos/300/200'),
                          fit: BoxFit.cover,
                        ),
                      ),
                    ),
                    
                    // Gradient overlay
                    Container(
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(12),
                        gradient: LinearGradient(
                          begin: Alignment.topCenter,
                          end: Alignment.bottomCenter,
                          colors: [
                            Colors.transparent,
                            Colors.black.withOpacity(0.7),
                          ],
                        ),
                      ),
                    ),
                    
                    // Profile info
                    Positioned(
                      bottom: 16,
                      left: 16,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            'John Doe',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 20,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            'Flutter Developer',
                            style: TextStyle(
                              color: Colors.white70,
                              fontSize: 14,
                            ),
                          ),
                        ],
                      ),
                    ),
                    
                    // Favorite button
                    Positioned(
                      top: 16,
                      right: 16,
                      child: Container(
                        padding: EdgeInsets.all(8),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          shape: BoxShape.circle,
                        ),
                        child: Icon(
                          Icons.favorite_border,
                          color: Colors.red,
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Flexible and Expanded
```dart
class FlexibleExpandedExamples extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Flexible & Expanded Examples')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // Expanded example
            Text(
              'Expanded Example:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              height: 100,
              child: Row(
                children: [
                  Container(
                    width: 50,
                    color: Colors.red,
                    child: Center(child: Text('Fixed')),
                  ),
                  Expanded(
                    flex: 2,
                    child: Container(
                      color: Colors.green,
                      child: Center(child: Text('Expanded 2')),
                    ),
                  ),
                  Expanded(
                    flex: 1,
                    child: Container(
                      color: Colors.blue,
                      child: Center(child: Text('Expanded 1')),
                    ),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // Flexible example
            Text(
              'Flexible Example:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              height: 100,
              child: Row(
                children: [
                  Flexible(
                    flex: 1,
                    child: Container(
                      color: Colors.orange,
                      child: Center(child: Text('Flexible 1')),
                    ),
                  ),
                  Flexible(
                    flex: 2,
                    child: Container(
                      color: Colors.purple,
                      child: Center(child: Text('Flexible 2')),
                    ),
                  ),
                  Container(
                    width: 80,
                    color: Colors.teal,
                    child: Center(child: Text('Fixed')),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 30),
            
            // Practical example: Chat message layout
            Text(
              'Chat Message Layout:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Container(
              padding: EdgeInsets.all(12),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Row(
                children: [
                  CircleAvatar(
                    radius: 20,
                    backgroundImage: NetworkImage('https://picsum.photos/40/40'),
                  ),
                  SizedBox(width: 12),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          'John Doe',
                          style: TextStyle(fontWeight: FontWeight.bold),
                        ),
                        Text(
                          'This is a sample message that might be quite long and should wrap properly within the available space.',
                        ),
                      ],
                    ),
                  ),
                  Text(
                    '2:30 PM',
                    style: TextStyle(
                      color: Colors.grey[600],
                      fontSize: 12,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Material Design Widgets

### Scaffold
```dart
class ScaffoldExamples extends StatefulWidget {
  @override
  _ScaffoldExamplesState createState() => _ScaffoldExamplesState();
}

class _ScaffoldExamplesState extends State<ScaffoldExamples> {
  int _selectedIndex = 0;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // App bar
      appBar: AppBar(
        title: Text('Scaffold Example'),
        backgroundColor: Colors.blue,
        elevation: 4,
        actions: [
          IconButton(
            icon: Icon(Icons.search),
            onPressed: () {},
          ),
          IconButton(
            icon: Icon(Icons.more_vert),
            onPressed: () {},
          ),
        ],
      ),
      
      // Drawer
      drawer: Drawer(
        child: ListView(
          children: [
            DrawerHeader(
              decoration: BoxDecoration(color: Colors.blue),
              child: Text(
                'Menu',
                style: TextStyle(color: Colors.white, fontSize: 24),
              ),
            ),
            ListTile(
              leading: Icon(Icons.home),
              title: Text('Home'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: Icon(Icons.settings),
              title: Text('Settings'),
              onTap: () => Navigator.pop(context),
            ),
          ],
        ),
      ),
      
      // Body
      body: _getPage(_selectedIndex),
      
      // Bottom navigation
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        onTap: (index) => setState(() => _selectedIndex = index),
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.business),
            label: 'Business',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.school),
            label: 'School',
          ),
        ],
      ),
      
      // Floating action button
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('FAB pressed!')),
          );
        },
        child: Icon(Icons.add),
      ),
      
      // FAB location
      floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,
    );
  }
  
  Widget _getPage(int index) {
    switch (index) {
      case 0:
        return Center(child: Text('Home Page'));
      case 1:
        return Center(child: Text('Business Page'));
      case 2:
        return Center(child: Text('School Page'));
      default:
        return Center(child: Text('Page not found'));
    }
  }
}
```

### Cards and Lists
```dart
class CardsListsExamples extends StatelessWidget {
  final List<Map<String, dynamic>> users = [
    {
      'name': 'Alice Johnson',
      'email': 'alice@example.com',
      'avatar': 'https://picsum.photos/seed/alice/50/50',
      'isOnline': true,
    },
    {
      'name': 'Bob Smith',
      'email': 'bob@example.com',
      'avatar': 'https://picsum.photos/seed/bob/50/50',
      'isOnline': false,
    },
    {
      'name': 'Carol Davis',
      'email': 'carol@example.com',
      'avatar': 'https://picsum.photos/seed/carol/50/50',
      'isOnline': true,
    },
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Cards & Lists Examples')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Simple Card
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Simple Card',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8),
                    Text('This is a simple card with some content.'),
                  ],
                ),
              ),
            ),
            
            SizedBox(height: 20),
            
            // Card with image
            Card(
              clipBehavior: Clip.antiAlias,
              child: Column(
                children: [
                  Image.network(
                    'https://picsum.photos/400/200',
                    height: 200,
                    width: double.infinity,
                    fit: BoxFit.cover,
                  ),
                  Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          'Beautiful Landscape',
                          style: TextStyle(
                            fontSize: 18,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        SizedBox(height: 8),
                        Text(
                          'A stunning view of nature captured in this beautiful photograph.',
                        ),
                        SizedBox(height: 16),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                          children: [
                            TextButton.icon(
                              icon: Icon(Icons.thumb_up),
                              label: Text('Like'),
                              onPressed: () {},
                            ),
                            TextButton.icon(
                              icon: Icon(Icons.share),
                              label: Text('Share'),
                              onPressed: () {},
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            Text(
              'User List:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            
            // ListView with cards
            ...users.map((user) => Card(
              margin: EdgeInsets.only(bottom: 8),
              child: ListTile(
                leading: Stack(
                  children: [
                    CircleAvatar(
                      backgroundImage: NetworkImage(user['avatar']),
                    ),
                    if (user['isOnline'])
                      Positioned(
                        right: 0,
                        bottom: 0,
                        child: Container(
                          width: 12,
                          height: 12,
                          decoration: BoxDecoration(
                            color: Colors.green,
                            shape: BoxShape.circle,
                            border: Border.all(color: Colors.white, width: 2),
                          ),
                        ),
                      ),
                  ],
                ),
                title: Text(user['name']),
                subtitle: Text(user['email']),
                trailing: IconButton(
                  icon: Icon(Icons.more_vert),
                  onPressed: () {},
                ),
                onTap: () {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Tapped ${user['name']}')),
                  );
                },
              ),
            )).toList(),
          ],
        ),
      ),
    );
  }
}
```

## Input Widgets

### Text Input and Forms
```dart
class InputExamples extends StatefulWidget {
  @override
  _InputExamplesState createState() => _InputExamplesState();
}

class _InputExamplesState extends State<InputExamples> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  
  bool _obscurePassword = true;
  String _selectedCountry = 'USA';
  bool _agreeToTerms = false;
  double _ageSlider = 25;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Input Examples')),
      body: Form(
        key: _formKey,
        child: SingleChildScrollView(
          padding: EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Text Input
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: 'Full Name',
                  hintText: 'Enter your full name',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your name';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              // Email Input
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: InputDecoration(
                  labelText: 'Email',
                  hintText: 'Enter your email address',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your email';
                  }
                  if (!value.contains('@')) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              // Password Input
              TextFormField(
                controller: _passwordController,
                obscureText: _obscurePassword,
                decoration: InputDecoration(
                  labelText: 'Password',
                  hintText: 'Enter your password',
                  prefixIcon: Icon(Icons.lock),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _obscurePassword 
                        ? Icons.visibility 
                        : Icons.visibility_off,
                    ),
                    onPressed: () {
                      setState(() {
                        _obscurePassword = !_obscurePassword;
                      });
                    },
                  ),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter your password';
                  }
                  if (value.length < 6) {
                    return 'Password must be at least 6 characters';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              // Dropdown
              DropdownButtonFormField<String>(
                value: _selectedCountry,
                decoration: InputDecoration(
                  labelText: 'Country',
                  border: OutlineInputBorder(),
                ),
                items: ['USA', 'Canada', 'UK', 'Australia', 'India']
                    .map((country) => DropdownMenuItem(
                          value: country,
                          child: Text(country),
                        ))
                    .toList(),
                onChanged: (value) {
                  setState(() {
                    _selectedCountry = value!;
                  });
                },
              ),
              
              SizedBox(height: 16),
              
              // Slider
              Text('Age: ${_ageSlider.round()}'),
              Slider(
                value: _ageSlider,
                min: 18,
                max: 80,
                divisions: 62,
                label: _ageSlider.round().toString(),
                onChanged: (value) {
                  setState(() {
                    _ageSlider = value;
                  });
                },
              ),
              
              SizedBox(height: 16),
              
              // Checkbox
              CheckboxListTile(
                title: Text('I agree to the terms and conditions'),
                value: _agreeToTerms,
                onChanged: (value) {
                  setState(() {
                    _agreeToTerms = value!;
                  });
                },
                controlAffinity: ListTileControlAffinity.leading,
              ),
              
              SizedBox(height: 24),
              
              // Submit Button
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _agreeToTerms ? _submitForm : null,
                  child: Text('Submit'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Form submitted successfully!'),
          backgroundColor: Colors.green,
        ),
      );
    }
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

## Display Widgets

### ListView and GridView
```dart
class ListGridExamples extends StatelessWidget {
  final List<Map<String, dynamic>> products = List.generate(20, (index) => {
    'id': index + 1,
    'name': 'Product ${index + 1}',
    'price': '\$${(index + 1) * 10}',
    'image': 'https://picsum.photos/seed/product$index/200/200',
    'rating': (3.0 + (index % 3)).toDouble(),
  });
  
  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: Text('Lists & Grids'),
          bottom: TabBar(
            tabs: [
              Tab(text: 'ListView'),
              Tab(text: 'GridView'),
              Tab(text: 'Custom'),
            ],
          ),
        ),
        body: TabBarView(
          children: [
            _buildListView(),
            _buildGridView(),
            _buildCustomView(),
          ],
        ),
      ),
    );
  }
  
  Widget _buildListView() {
    return ListView.builder(
      padding: EdgeInsets.all(8),
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return Card(
          margin: EdgeInsets.symmetric(vertical: 4),
          child: ListTile(
            leading: ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                product['image'],
                width: 60,
                height: 60,
                fit: BoxFit.cover,
              ),
            ),
            title: Text(product['name']),
            subtitle: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product['price'],
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    color: Colors.green,
                  ),
                ),
                Row(
                  children: [
                    ...List.generate(5, (starIndex) => Icon(
                      starIndex < product['rating'] 
                        ? Icons.star 
                        : Icons.star_border,
                      color: Colors.amber,
                      size: 16,
                    )),
                    SizedBox(width: 4),
                    Text('${product['rating']}'),
                  ],
                ),
              ],
            ),
            trailing: IconButton(
              icon: Icon(Icons.add_shopping_cart),
              onPressed: () {},
            ),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Tapped ${product['name']}')),
              );
            },
          ),
        );
      },
    );
  }
  
  Widget _buildGridView() {
    return GridView.builder(
      padding: EdgeInsets.all(8),
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        childAspectRatio: 0.8,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
      ),
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return Card(
          clipBehavior: Clip.antiAlias,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Expanded(
                child: Image.network(
                  product['image'],
                  width: double.infinity,
                  fit: BoxFit.cover,
                ),
              ),
              Padding(
                padding: EdgeInsets.all(8),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      product['name'],
                      style: TextStyle(fontWeight: FontWeight.bold),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
                    SizedBox(height: 4),
                    Text(
                      product['price'],
                      style: TextStyle(
                        color: Colors.green,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    Row(
                      children: [
                        Icon(Icons.star, color: Colors.amber, size: 16),
                        Text(
                          '${product['rating']}',
                          style: TextStyle(fontSize: 12),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ],
          ),
        );
      },
    );
  }
  
  Widget _buildCustomView() {
    return SingleChildScrollView(
      child: Column(
        children: [
          // Horizontal scrolling list
          Container(
            height: 120,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              padding: EdgeInsets.symmetric(horizontal: 16),
              itemCount: 5,
              itemBuilder: (context, index) {
                return Container(
                  width: 100,
                  margin: EdgeInsets.only(right: 16),
                  decoration: BoxDecoration(
                    color: Colors.primaries[index % Colors.primaries.length],
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Center(
                    child: Text(
                      'Item $index',
                      style: TextStyle(
                        color: Colors.white,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                );
              },
            ),
          ),
          
          SizedBox(height: 16),
          
          // Staggered grid effect
          Padding(
            padding: EdgeInsets.symmetric(horizontal: 16),
            child: Column(
              children: [
                Row(
                  children: [
                    Expanded(
                      child: Container(
                        height: 200,
                        margin: EdgeInsets.only(right: 8),
                        decoration: BoxDecoration(
                          color: Colors.blue[100],
                          borderRadius: BorderRadius.circular(12),
                        ),
                        child: Center(child: Text('Tall Card')),
                      ),
                    ),
                    Expanded(
                      child: Column(
                        children: [
                          Container(
                            height: 96,
                            margin: EdgeInsets.only(bottom: 8),
                            decoration: BoxDecoration(
                              color: Colors.green[100],
                              borderRadius: BorderRadius.circular(12),
                            ),
                            child: Center(child: Text('Short Card 1')),
                          ),
                          Container(
                            height: 96,
                            decoration: BoxDecoration(
                              color: Colors.orange[100],
                              borderRadius: BorderRadius.circular(12),
                            ),
                            child: Center(child: Text('Short Card 2')),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

## Styling and Theming

### Custom Themes
```dart
class ThemeExamples extends StatefulWidget {
  @override
  _ThemeExamplesState createState() => _ThemeExamplesState();
}

class _ThemeExamplesState extends State<ThemeExamples> {
  bool _isDarkMode = false;
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Theme Examples',
      theme: _isDarkMode ? _darkTheme : _lightTheme,
      home: Scaffold(
        appBar: AppBar(
          title: Text('Theme Examples'),
          actions: [
            IconButton(
              icon: Icon(_isDarkMode ? Icons.light_mode : Icons.dark_mode),
              onPressed: () {
                setState(() {
                  _isDarkMode = !_isDarkMode;
                });
              },
            ),
          ],
        ),
        body: _buildThemedContent(),
      ),
    );
  }
  
  ThemeData get _lightTheme => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.deepPurple,
      brightness: Brightness.light,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: Colors.deepPurple,
      foregroundColor: Colors.white,
      elevation: 4,
    ),
    cardTheme: CardTheme(
      elevation: 4,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.deepPurple,
        foregroundColor: Colors.white,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
      ),
    ),
    textTheme: TextTheme(
      headlineLarge: TextStyle(
        fontSize: 32,
        fontWeight: FontWeight.bold,
        color: Colors.deepPurple,
      ),
      bodyLarge: TextStyle(
        fontSize: 16,
        color: Colors.grey[800],
      ),
    ),
  );
  
  ThemeData get _darkTheme => ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.deepPurple,
      brightness: Brightness.dark,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: Colors.grey[900],
      foregroundColor: Colors.white,
      elevation: 4,
    ),
    cardTheme: CardTheme(
      elevation: 4,
      color: Colors.grey[800],
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.deepPurpleAccent,
        foregroundColor: Colors.white,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
      ),
    ),
  );
  
  Widget _buildThemedContent() {
    return SingleChildScrollView(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            'Themed Components',
            style: Theme.of(context).textTheme.headlineLarge,
          ),
          
          SizedBox(height: 20),
          
          Card(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'Card with Theme',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 8),
                  Text('This card adapts to the current theme.'),
                  SizedBox(height: 16),
                  Row(
                    children: [
                      ElevatedButton(
                        onPressed: () {},
                        child: Text('Themed Button'),
                      ),
                      SizedBox(width: 12),
                      OutlinedButton(
                        onPressed: () {},
                        child: Text('Outlined'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
          
          SizedBox(height: 20),
          
          // Custom styled container
          Container(
            width: double.infinity,
            padding: EdgeInsets.all(20),
            decoration: BoxDecoration(
              gradient: LinearGradient(
                colors: [
                  Theme.of(context).colorScheme.primary.withOpacity(0.1),
                  Theme.of(context).colorScheme.secondary.withOpacity(0.1),
                ],
              ),
              borderRadius: BorderRadius.circular(12),
              border: Border.all(
                color: Theme.of(context).colorScheme.outline,
              ),
            ),
            child: Column(
              children: [
                Icon(
                  Icons.palette,
                  size: 48,
                  color: Theme.of(context).colorScheme.primary,
                ),
                SizedBox(height: 12),
                Text(
                  'Custom Themed Container',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.onSurface,
                  ),
                ),
                Text(
                  'This container uses theme colors for consistent styling.',
                  textAlign: TextAlign.center,
                  style: TextStyle(
                    color: Theme.of(context).colorScheme.onSurface.withOpacity(0.7),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

## Custom Widgets

### Reusable Custom Widgets
```dart
// Custom Button Widget
class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback? onPressed;
  final Color? backgroundColor;
  final Color? textColor;
  final IconData? icon;
  final double? width;
  final bool isLoading;
  
  const CustomButton({
    Key? key,
    required this.text,
    this.onPressed,
    this.backgroundColor,
    this.textColor,
    this.icon,
    this.width,
    this.isLoading = false,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      width: width,
      height: 50,
      child: ElevatedButton(
        onPressed: isLoading ? null : onPressed,
        style: ElevatedButton.styleFrom(
          backgroundColor: backgroundColor ?? Theme.of(context).primaryColor,
          foregroundColor: textColor ?? Colors.white,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          elevation: 2,
        ),
        child: isLoading
            ? SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  valueColor: AlwaysStoppedAnimation<Color>(
                    textColor ?? Colors.white,
                  ),
                ),
              )
            : Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  if (icon != null) ...[
                    Icon(icon),
                    SizedBox(width: 8),
                  ],
                  Text(
                    text,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
      ),
    );
  }
}

// Custom Card Widget
class CustomCard extends StatelessWidget {
  final Widget child;
  final EdgeInsetsGeometry? padding;
  final EdgeInsetsGeometry? margin;
  final Color? backgroundColor;
  final double? elevation;
  final VoidCallback? onTap;
  final BorderRadius? borderRadius;
  
  const CustomCard({
    Key? key,
    required this.child,
    this.padding,
    this.margin,
    this.backgroundColor,
    this.elevation,
    this.onTap,
    this.borderRadius,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      margin: margin ?? EdgeInsets.all(8),
      child: Material(
        color: backgroundColor ?? Theme.of(context).cardColor,
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

// Custom Input Field Widget
class CustomTextField extends StatefulWidget {
  final String label;
  final String? hint;
  final IconData? prefixIcon;
  final TextEditingController? controller;
  final String? Function(String?)? validator;
  final TextInputType? keyboardType;
  final bool obscureText;
  final int? maxLines;
  final VoidCallback? onSuffixIconPressed;
  final IconData? suffixIcon;
  
  const CustomTextField({
    Key? key,
    required this.label,
    this.hint,
    this.prefixIcon,
    this.controller,
    this.validator,
    this.keyboardType,
    this.obscureText = false,
    this.maxLines = 1,
    this.onSuffixIconPressed,
    this.suffixIcon,
  }) : super(key: key);
  
  @override
  _CustomTextFieldState createState() => _CustomTextFieldState();
}

class _CustomTextFieldState extends State<CustomTextField> {
  bool _isFocused = false;
  
  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.only(bottom: 16),
      child: Focus(
        onFocusChange: (hasFocus) {
          setState(() {
            _isFocused = hasFocus;
          });
        },
        child: TextFormField(
          controller: widget.controller,
          validator: widget.validator,
          keyboardType: widget.keyboardType,
          obscureText: widget.obscureText,
          maxLines: widget.maxLines,
          decoration: InputDecoration(
            labelText: widget.label,
            hintText: widget.hint,
            prefixIcon: widget.prefixIcon != null 
                ? Icon(
                    widget.prefixIcon,
                    color: _isFocused 
                        ? Theme.of(context).primaryColor 
                        : Colors.grey,
                  )
                : null,
            suffixIcon: widget.suffixIcon != null
                ? IconButton(
                    icon: Icon(widget.suffixIcon),
                    onPressed: widget.onSuffixIconPressed,
                  )
                : null,
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(color: Colors.grey[300]!),
            ),
            focusedBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(
                color: Theme.of(context).primaryColor,
                width: 2,
              ),
            ),
            errorBorder: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
              borderSide: BorderSide(color: Colors.red),
            ),
            filled: true,
            fillColor: _isFocused 
                ? Theme.of(context).primaryColor.withOpacity(0.05)
                : Colors.grey[50],
          ),
        ),
      ),
    );
  }
}

// Usage Example
class CustomWidgetsExample extends StatefulWidget {
  @override
  _CustomWidgetsExampleState createState() => _CustomWidgetsExampleState();
}

class _CustomWidgetsExampleState extends State<CustomWidgetsExample> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  bool _isLoading = false;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Custom Widgets Example')),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            CustomCard(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'User Registration',
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 20),
                  
                  CustomTextField(
                    label: 'Full Name',
                    hint: 'Enter your full name',
                    prefixIcon: Icons.person,
                    controller: _nameController,
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter your name';
                      }
                      return null;
                    },
                  ),
                  
                  CustomTextField(
                    label: 'Email',
                    hint: 'Enter your email address',
                    prefixIcon: Icons.email,
                    keyboardType: TextInputType.emailAddress,
                    controller: _emailController,
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'Please enter your email';
                      }
                      return null;
                    },
                  ),
                  
                  SizedBox(height: 20),
                  
                  CustomButton(
                    text: 'Register',
                    icon: Icons.person_add,
                    width: double.infinity,
                    isLoading: _isLoading,
                    onPressed: () {
                      setState(() {
                        _isLoading = true;
                      });
                      
                      // Simulate network request
                      Future.delayed(Duration(seconds: 2), () {
                        setState(() {
                          _isLoading = false;
                        });
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(content: Text('Registration successful!')),
                        );
                      });
                    },
                  ),
                ],
              ),
            ),
            
            SizedBox(height: 20),
            
            // Custom cards with different styles
            CustomCard(
              backgroundColor: Colors.blue[50],
              onTap: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Card tapped!')),
                );
              },
              child: Row(
                children: [
                  Icon(Icons.info, color: Colors.blue, size: 32),
                  SizedBox(width: 16),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          'Information',
                          style: TextStyle(
                            fontSize: 18,
                            fontWeight: FontWeight.bold,
                            color: Colors.blue[800],
                          ),
                        ),
                        Text('Tap this card to see interaction'),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }
}
```

This comprehensive guide covers the essential Flutter widgets and UI components. Practice building with these widgets to create beautiful and functional user interfaces!