# Flutter Basics Guide

## Table of Contents
1. [What is Flutter?](#what-is-flutter)
2. [Flutter Setup and Installation](#flutter-setup-and-installation)
3. [Dart Language Basics](#dart-language-basics)
4. [Flutter Architecture](#flutter-architecture)
5. [Your First Flutter App](#your-first-flutter-app)
6. [Understanding Widgets](#understanding-widgets)
7. [Hot Reload and Hot Restart](#hot-reload-and-hot-restart)

## What is Flutter?

Flutter is Google's revolutionary open-source UI toolkit designed to solve one of the biggest challenges in mobile development: creating applications that work seamlessly across multiple platforms. Instead of writing separate codebases for iOS, Android, web, and desktop applications, Flutter allows developers to write once and deploy everywhere.

### How Flutter Works
Flutter doesn't rely on platform-specific UI components. Instead, it includes its own rendering engine built in C++, which draws every pixel on the screen. This approach provides several advantages:

1. **Consistent UI**: Your app looks and behaves the same across all platforms
2. **High Performance**: Direct compilation to native machine code
3. **Complete Control**: Every pixel is under your control for precise designs

### The Flutter Ecosystem
- **Framework**: The Flutter SDK with core libraries and tools
- **Engine**: High-performance rendering engine written in C++
- **Embedder**: Platform-specific embedders for different operating systems
- **Dart Platform**: The runtime and libraries for the Dart programming language

### Key Features:
- **Single Codebase**: Write once, run everywhere (iOS, Android, Web, Desktop, Embedded)
  - Reduces development time by up to 50%
  - Easier maintenance and updates
  - Consistent user experience across platforms

- **Fast Development**: Hot reload for instant UI updates
  - See changes in milliseconds without losing app state
  - Experiment with UI designs in real-time
  - Faster iteration cycles

- **Native Performance**: Compiled to native machine code
  - No JavaScript bridge like other cross-platform solutions
  - 60fps animations and smooth scrolling
  - Fast startup times

- **Rich UI**: Extensive widget library for beautiful UIs
  - Material Design (Android-style) widgets
  - Cupertino (iOS-style) widgets
  - Highly customizable components
  - Built-in animations and transitions

- **Open Source**: Free and backed by Google
  - Large community support
  - Regular updates and improvements
  - Extensive documentation and resources

## Flutter Setup and Installation

Setting up Flutter properly is crucial for a smooth development experience. This comprehensive guide will walk you through each step.

### System Requirements

#### Windows:
- **Operating System**: Windows 10 or later (64-bit), x86-64 based
- **Disk Space**: At least 2.8 GB (excluding disk space for IDE/tools)
- **Tools**: Git for Windows, PowerShell 5.0 or newer
- **Additional**: Windows 10 SDK (for desktop development)

#### macOS:
- **Operating System**: macOS 10.14 (Mojave) or later
- **Disk Space**: At least 2.8 GB (excluding Xcode and Android Studio)
- **Tools**: bash, curl, git 2.x, mkdir, rm, unzip, which
- **Additional**: Xcode (for iOS development), CocoaPods

#### Linux:
- **Operating System**: Linux (64-bit)
- **Disk Space**: At least 2.8 GB
- **Tools**: bash, curl, file, git 2.x, mkdir, rm, unzip, which, xz-utils, zip
- **Additional**: Various libraries for desktop development

### Prerequisites
Before installing Flutter, ensure you have:
1. **Git**: Version control system for downloading Flutter
2. **Text Editor/IDE**: VS Code, Android Studio, or IntelliJ IDEA
3. **Command Line Tools**: Terminal/Command Prompt access
4. **Internet Connection**: For downloading dependencies

### Detailed Installation Steps

#### 1. Download Flutter SDK
```bash
# Visit https://flutter.dev/docs/get-started/install
# Choose your operating system
# Download the latest stable release

# Windows: Extract to C:\flutter (avoid paths with spaces)
# macOS/Linux: Extract to desired location like ~/development/flutter
```

#### 2. Update PATH Environment Variable

**Windows (via System Properties):**
```bash
# 1. Open "Environment Variables" from System Properties
# 2. Under "User variables", edit the "Path" variable
# 3. Add C:\flutter\bin to the list
# 4. Click OK to save changes
```

**Windows (via Command Prompt):**
```bash
# Temporary (current session only)
set PATH=%PATH%;C:\flutter\bin

# Permanent (add to system PATH through GUI)
```

**macOS/Linux:**
```bash
# Add to your shell profile (~/.bashrc, ~/.zshrc, etc.)
export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin"

# For example:
export PATH="$PATH:$HOME/development/flutter/bin"

# Reload your shell configuration
source ~/.bashrc  # or ~/.zshrc
```

#### 3. Verify Installation and Check Dependencies
```bash
flutter doctor
```

The `flutter doctor` command checks your environment and displays a report of the status of your Flutter installation. It will show:
- ✅ Items that are properly set up
- ⚠️  Items that have minor issues
- ❌ Items that need attention

#### 4. Install Platform-specific Tools

**For Android Development:**
1. **Android Studio**: Download and install from https://developer.android.com/studio
2. **Android SDK**: Install through Android Studio SDK Manager
3. **Android Emulator**: Create virtual devices for testing
4. **Accept Android Licenses**: Run `flutter doctor --android-licenses`

**For iOS Development (macOS only):**
1. **Xcode**: Install from Mac App Store
2. **iOS Simulator**: Included with Xcode
3. **CocoaPods**: Run `sudo gem install cocoapods`
4. **Accept Xcode License**: Run `sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer`

### IDE Setup and Configuration

#### Visual Studio Code (Recommended for beginners)
```bash
# Install Flutter extension
# 1. Open VS Code
# 2. Go to Extensions (Ctrl+Shift+X)
# 3. Search for "Flutter"
# 4. Install Flutter extension (automatically includes Dart)
# 5. Restart VS Code
```

**Essential VS Code Extensions:**
- **Flutter**: Official Flutter support
- **Dart**: Dart language support (auto-installed with Flutter)
- **Flutter Widget Snippets**: Code snippets for common widgets
- **Awesome Flutter Snippets**: Additional helpful snippets
- **Flutter Tree**: Widget tree visualization

#### Android Studio
```bash
# Install Flutter plugin
# 1. Open Android Studio
# 2. Go to File > Settings (Windows/Linux) or Preferences (macOS)
# 3. Select Plugins
# 4. Search for "Flutter"
# 5. Install Flutter plugin (includes Dart plugin)
# 6. Restart Android Studio
```

### Common Installation Issues and Solutions

**Issue 1: Flutter not recognized in command line**
```bash
# Solution: Check PATH configuration
echo $PATH  # macOS/Linux
echo %PATH%  # Windows

# Ensure Flutter bin directory is included
```

**Issue 2: Android license not accepted**
```bash
# Solution: Accept all Android licenses
flutter doctor --android-licenses
# Press 'y' for each license
```

**Issue 3: Unable to locate Android SDK**
```bash
# Solution: Set ANDROID_HOME environment variable
# Windows: Set to C:\Users\[USERNAME]\AppData\Local\Android\Sdk
# macOS: Set to ~/Library/Android/sdk
# Linux: Set to ~/Android/Sdk
```

**Issue 4: iOS development issues (macOS)**
```bash
# Solution: Install/update Xcode command line tools
xcode-select --install

# Accept Xcode license
sudo xcodebuild -license accept
```

### Verification Commands
```bash
# Check Flutter version
flutter --version

# Check Dart version
dart --version

# Run comprehensive system check
flutter doctor -v  # Verbose output for detailed information

# Check connected devices
flutter devices

# Create and run a test project
flutter create test_app
cd test_app
flutter run
```

## Dart Language Basics

Dart is the programming language used by Flutter. Here are the essential concepts:

### Variables and Data Types
```dart
// Variables
var name = 'John';
String lastName = 'Doe';
int age = 25;
double height = 5.9;
bool isStudent = true;

// Constants
const pi = 3.14159;
final currentTime = DateTime.now();

// Nullable variables
String? middleName;
int? grade;
```

### Functions
```dart
// Basic function
String greet(String name) {
  return 'Hello, $name!';
}

// Arrow function
String greetShort(String name) => 'Hello, $name!';

// Optional parameters
String greetWithTitle(String name, [String? title]) {
  return title != null ? 'Hello, $title $name!' : 'Hello, $name!';
}

// Named parameters
String greetFull({required String firstName, String? lastName}) {
  return lastName != null 
    ? 'Hello, $firstName $lastName!' 
    : 'Hello, $firstName!';
}
```

### Classes and Objects
```dart
class Person {
  String name;
  int age;
  
  // Constructor
  Person(this.name, this.age);
  
  // Named constructor
  Person.baby(this.name) : age = 0;
  
  // Method
  void introduce() {
    print('Hi, I\'m $name and I\'m $age years old.');
  }
  
  // Getter
  bool get isAdult => age >= 18;
  
  // Setter
  set personAge(int newAge) {
    if (newAge >= 0) age = newAge;
  }
}

// Usage
var person = Person('Alice', 25);
person.introduce();
print(person.isAdult);
```

### Collections
```dart
// Lists
List<String> fruits = ['apple', 'banana', 'orange'];
fruits.add('grape');
print(fruits[0]); // apple

// Maps
Map<String, int> scores = {
  'math': 95,
  'english': 87,
  'science': 92,
};
scores['history'] = 89;

// Sets
Set<String> uniqueFruits = {'apple', 'banana', 'orange'};
uniqueFruits.add('apple'); // Won't add duplicate
```

### Control Flow
```dart
// If-else
if (age >= 18) {
  print('Adult');
} else if (age >= 13) {
  print('Teenager');
} else {
  print('Child');
}

// Switch
switch (grade) {
  case 'A':
    print('Excellent');
    break;
  case 'B':
    print('Good');
    break;
  default:
    print('Keep trying');
}

// Loops
for (int i = 0; i < 5; i++) {
  print(i);
}

for (String fruit in fruits) {
  print(fruit);
}

while (age < 100) {
  age++;
}
```

## Flutter Architecture

Flutter follows a reactive programming model with a unique architecture that sets it apart from other mobile development frameworks. Understanding this architecture is crucial for building efficient Flutter applications.

### The Flutter Framework Architecture

Flutter's architecture consists of several layers, each serving a specific purpose:

#### 1. Framework Layer (Dart)
This is where you write your application code. It includes:

**Material/Cupertino Libraries:**
- Pre-built widgets following Material Design (Android) and Cupertino (iOS) guidelines
- Buttons, text fields, navigation bars, etc.

**Widgets Layer:**
- The core of Flutter's UI system
- Everything you see on screen is a widget
- Immutable descriptions of UI elements

**Rendering Layer:**
- Handles layout, painting, and compositing
- Converts widget trees into visual elements

**Animation & Gestures:**
- Built-in animation system
- Touch and gesture recognition

#### 2. Engine Layer (C++)
The Flutter engine provides:
- **Skia Graphics Library**: Hardware-accelerated 2D graphics
- **Dart Runtime**: Executes Dart code
- **Platform Channels**: Communication bridge with native code
- **Text Layout**: Advanced text rendering capabilities

#### 3. Platform Layer
- **iOS Embedder**: Integrates with iOS platform
- **Android Embedder**: Integrates with Android platform
- **Desktop Embedders**: For Windows, macOS, and Linux

### Widget Tree Architecture

Flutter apps are built using a tree structure of widgets. Understanding this concept is fundamental:

```dart
// Widget tree visualization
MaterialApp                    // Root widget
├── Scaffold                   // Screen structure
    ├── AppBar                 // Top navigation
    │   └── Text              // App title
    └── Body                   // Main content area
        └── Column            // Vertical layout
            ├── Text          // First text widget
            ├── ElevatedButton // Button widget
            └── Container     // Container widget
                └── Image     // Image widget
```

**Key Concepts:**

1. **Parent-Child Relationships**: Each widget can have one or more children
2. **Composition over Inheritance**: Build complex UIs by combining simple widgets
3. **Immutability**: Widgets are immutable; changes create new widget instances

### The Three Trees in Flutter

Flutter actually maintains three parallel trees:

#### 1. Widget Tree
- **What**: The tree you create in your code
- **Purpose**: Describes the configuration of UI elements
- **Characteristics**: Immutable, lightweight, frequently recreated

#### 2. Element Tree
- **What**: Manages the lifecycle of widgets
- **Purpose**: Maintains state and handles widget updates
- **Characteristics**: Mutable, longer-lived than widgets

#### 3. Render Object Tree
- **What**: Handles actual layout and painting
- **Purpose**: Performs the heavy lifting of rendering
- **Characteristics**: Mutable, handles size, position, and painting

### Widget Lifecycle

Understanding how widgets live and die is crucial for managing resources:

```dart
// Stateless Widget Lifecycle
class MyStatelessWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Called every time the widget needs to be built
    return Container();
  }
}

// Stateful Widget Lifecycle
class MyStatefulWidget extends StatefulWidget {
  @override
  _MyStatefulWidgetState createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  @override
  void initState() {
    // Called once when the widget is inserted into the tree
    super.initState();
    // Initialize resources, start timers, etc.
  }
  
  @override
  void didChangeDependencies() {
    // Called when dependencies change (theme, locale, etc.)
    super.didChangeDependencies();
  }
  
  @override
  Widget build(BuildContext context) {
    // Called whenever the widget needs to be built/rebuilt
    return Container();
  }
  
  @override
  void didUpdateWidget(MyStatefulWidget oldWidget) {
    // Called when the parent widget changes and needs to update this widget
    super.didUpdateWidget(oldWidget);
  }
  
  @override
  void deactivate() {
    // Called when the widget is removed from the tree temporarily
    super.deactivate();
  }
  
  @override
  void dispose() {
    // Called once when the widget is permanently removed
    // Clean up resources, cancel timers, close streams, etc.
    super.dispose();
  }
}
```

### React-like Architecture

Flutter borrows concepts from React:

1. **Declarative UI**: Describe what the UI should look like for a given state
2. **Unidirectional Data Flow**: Data flows down, events flow up
3. **Immutable Widgets**: Changes result in new widget instances
4. **Virtual DOM-like**: Element tree acts as an intermediary layer

### Widget Types

#### Stateless Widgets
```dart
// Immutable widgets that don't change over time
class WelcomeScreen extends StatelessWidget {
  final String userName;
  
  // Constructor receives data from parent
  const WelcomeScreen({
    Key? key, 
    required this.userName
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    // Build method describes the UI
    return Scaffold(
      appBar: AppBar(
        title: Text('Welcome'),
        backgroundColor: Colors.blue,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.person,
              size: 100,
              color: Colors.blue,
            ),
            SizedBox(height: 20),
            Text(
              'Hello, $userName!',
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 10),
            Text(
              'Welcome to Flutter!',
              style: TextStyle(
                fontSize: 16,
                color: Colors.grey[600],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**When to use Stateless Widgets:**
- Static content that doesn't change
- Widgets that only depend on their constructor parameters
- UI components that display data without user interaction

#### Stateful Widgets
```dart
// Widgets that can change state over time
class Counter extends StatefulWidget {
  final int initialValue; // Configuration from parent
  
  const Counter({Key? key, this.initialValue = 0}) : super(key: key);
  
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  late int _count; // Mutable state
  late String _message;
  
  @override
  void initState() {
    super.initState();
    // Initialize state using widget configuration
    _count = widget.initialValue;
    _message = 'Count is $_count';
  }
  
  void _incrementCounter() {
    setState(() {
      // Modify state inside setState
      _count++;
      _message = _count == 1 ? 'Count is $_count' : 'Count is $_count';
    });
  }
  
  void _decrementCounter() {
    setState(() {
      if (_count > 0) {
        _count--;
        _message = 'Count is $_count';
      }
    });
  }
  
  void _resetCounter() {
    setState(() {
      _count = widget.initialValue;
      _message = 'Count reset to $_count';
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter App'),
        backgroundColor: Colors.blue,
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: _resetCounter,
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.numbers,
              size: 80,
              color: Colors.blue,
            ),
            SizedBox(height: 20),
            Text(
              _message,
              style: TextStyle(fontSize: 18),
            ),
            SizedBox(height: 20),
            Text(
              '$_count',
              style: Theme.of(context).textTheme.headlineLarge?.copyWith(
                color: Colors.blue,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 40),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: _decrementCounter,
                  child: Icon(Icons.remove),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.red,
                    foregroundColor: Colors.white,
                  ),
                ),
                SizedBox(width: 20),
                ElevatedButton(
                  onPressed: _incrementCounter,
                  child: Icon(Icons.add),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.green,
                    foregroundColor: Colors.white,
                  ),
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

**When to use Stateful Widgets:**
- Interactive widgets that respond to user input
- Widgets that change appearance based on internal state
- Widgets that need to update over time (animations, timers)
- Forms and input fields

## Your First Flutter App

### Creating a New Project
```bash
flutter create my_first_app
cd my_first_app
flutter run
```

### Basic App Structure
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My First App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  String _message = 'Hello, Flutter!';
  
  void _changeMessage() {
    setState(() {
      _message = _message == 'Hello, Flutter!' 
        ? 'Welcome to Flutter Development!' 
        : 'Hello, Flutter!';
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('My First Flutter App'),
        backgroundColor: Colors.blue,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.flutter_dash,
              size: 100,
              color: Colors.blue,
            ),
            SizedBox(height: 20),
            Text(
              _message,
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
                color: Colors.blue[800],
              ),
            ),
            SizedBox(height: 30),
            ElevatedButton(
              onPressed: _changeMessage,
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.blue,
                foregroundColor: Colors.white,
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
              child: Text('Change Message'),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _changeMessage,
        child: Icon(Icons.refresh),
        backgroundColor: Colors.blue,
      ),
    );
  }
}
```

## Understanding Widgets

Widgets are the fundamental building blocks of Flutter applications. Every visual element in a Flutter app is a widget, from simple text and buttons to complex layouts and entire screens.

### The Widget Concept

Think of widgets as **blueprints** or **instructions** for what should appear on screen. They're not the actual visual elements themselves, but rather descriptions of what those elements should look like and how they should behave.

#### Key Widget Principles:

1. **Everything is a Widget**: In Flutter, literally everything you see is a widget
2. **Composition over Inheritance**: Build complex UIs by combining simple widgets
3. **Immutability**: Widgets are immutable (unchangeable) once created
4. **Declarative**: You describe what the UI should look like, not how to achieve it

### Widget Categories

#### 1. Structural Widgets
These widgets provide structure and layout to your app:

```dart
// Scaffold - Provides basic app structure
Scaffold(
  appBar: AppBar(title: Text('My App')),
  body: Container(/* main content */),
  bottomNavigationBar: BottomNavigationBar(/* navigation */),
  floatingActionButton: FloatingActionButton(/* action button */),
)

// Container - A box that can contain other widgets
Container(
  width: 200,
  height: 100,
  color: Colors.blue,
  child: Text('Inside container'),
)

// Column/Row - Arrange widgets vertically/horizontally
Column(
  children: [
    Text('First item'),
    Text('Second item'),
    Text('Third item'),
  ],
)
```

#### 2. Content Widgets
These widgets display information:

```dart
// Text - Display text content
Text(
  'Hello World',
  style: TextStyle(
    fontSize: 18,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  ),
)

// Image - Display images
Image.network('https://example.com/image.jpg')
Image.asset('assets/images/logo.png')

// Icon - Display icons
Icon(
  Icons.star,
  size: 30,
  color: Colors.yellow,
)
```

#### 3. Interactive Widgets
These widgets respond to user input:

```dart
// ElevatedButton - A button that users can press
ElevatedButton(
  onPressed: () {
    print('Button pressed!');
  },
  child: Text('Press Me'),
)

// TextField - Input field for text
TextField(
  decoration: InputDecoration(
    labelText: 'Enter your name',
    border: OutlineInputBorder(),
  ),
)

// Switch - Toggle between on/off
Switch(
  value: true,
  onChanged: (bool newValue) {
    // Handle switch change
  },
)
```

### Widget Hierarchy and Parent-Child Relationships

Widgets are organized in a tree structure where each widget can have:
- **One parent widget** (except the root)
- **Zero, one, or multiple child widgets**

```dart
// Example hierarchy
MaterialApp(                 // Root widget
  home: Scaffold(            // Child of MaterialApp
    appBar: AppBar(          // Child of Scaffold
      title: Text('Title'),  // Child of AppBar
    ),
    body: Column(            // Child of Scaffold
      children: [            // Children of Column
        Text('Item 1'),      // Child of Column
        Text('Item 2'),      // Child of Column
        Container(           // Child of Column
          child: Text('Item 3'), // Child of Container
        ),
      ],
    ),
  ),
)
```

### Common Widget Properties

Most widgets share similar properties that control their appearance and behavior:

```dart
Widget myWidget = Container(
  // Identity and keys
  key: ValueKey('unique_identifier'),
  
  // Size and positioning
  width: 200.0,
  height: 100.0,
  margin: EdgeInsets.all(16.0),
  padding: EdgeInsets.symmetric(horizontal: 20.0, vertical: 10.0),
  
  // Visual styling
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8.0),
    border: Border.all(color: Colors.black, width: 2.0),
    boxShadow: [
      BoxShadow(
        color: Colors.grey,
        offset: Offset(2, 2),
        blurRadius: 5.0,
      ),
    ],
  ),
  
  // Content
  child: Text('Hello World'),
);
```

### Widget Property Types

#### 1. Layout Properties
```dart
// Size constraints
Container(
  width: 200,           // Fixed width
  height: 100,          // Fixed height
  constraints: BoxConstraints(
    minWidth: 100,
    maxWidth: 300,
    minHeight: 50,
    maxHeight: 150,
  ),
)

// Spacing
Container(
  margin: EdgeInsets.all(16),              // Space outside
  padding: EdgeInsets.only(top: 8),        // Space inside
)

// Alignment
Container(
  alignment: Alignment.center,             // Center content
  // Other options: topLeft, bottomRight, etc.
)
```

#### 2. Visual Properties
```dart
Container(
  decoration: BoxDecoration(
    // Colors
    color: Colors.blue,
    gradient: LinearGradient(
      colors: [Colors.blue, Colors.purple],
    ),
    
    // Borders
    border: Border.all(color: Colors.black, width: 2),
    borderRadius: BorderRadius.circular(12),
    
    // Shadows
    boxShadow: [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 10,
        offset: Offset(0, 5),
      ),
    ],
    
    // Background image
    image: DecorationImage(
      image: NetworkImage('https://example.com/bg.jpg'),
      fit: BoxFit.cover,
    ),
  ),
)
```

#### 3. Behavioral Properties
```dart
GestureDetector(
  // Touch events
  onTap: () => print('Tapped'),
  onDoubleTap: () => print('Double tapped'),
  onLongPress: () => print('Long pressed'),
  
  child: Container(
    child: Text('Touch me'),
  ),
)

// Animation properties
AnimatedContainer(
  duration: Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  // ... other properties
)
```

### Best Practices for Widget Usage

#### 1. Extract Reusable Widgets
```dart
// Bad: Everything in one widget
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Container(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                CircleAvatar(/* ... */),
                Column(
                  children: [
                    Text('User Name'),
                    Text('user@example.com'),
                  ],
                ),
              ],
            ),
          ),
          // ... more complex UI
        ],
      ),
    );
  }
}

// Good: Extract reusable components
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          UserProfileCard(
            name: 'User Name',
            email: 'user@example.com',
          ),
          // ... other widgets
        ],
      ),
    );
  }
}

class UserProfileCard extends StatelessWidget {
  final String name;
  final String email;
  
  const UserProfileCard({
    Key? key,
    required this.name,
    required this.email,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      child: Row(
        children: [
          CircleAvatar(
            child: Text(name[0]),
          ),
          SizedBox(width: 12),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                name,
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
              Text(
                email,
                style: TextStyle(
                  fontSize: 14,
                  color: Colors.grey[600],
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

#### 2. Use const constructors when possible
```dart
// Good: Use const for static widgets
const Text('Static text')
const Icon(Icons.star)
const SizedBox(height: 16)

// This improves performance by reusing widget instances
```

#### 3. Choose the right widget for the job
```dart
// For simple layouts, use Column/Row
Column(children: [...])

// For scrollable content, use ListView
ListView(children: [...])

// For grid layouts, use GridView
GridView.count(crossAxisCount: 2, children: [...])

// For overlapping elements, use Stack
Stack(children: [...])
```

## Hot Reload and Hot Restart

### Hot Reload
- **What**: Updates the UI instantly while preserving app state
- **When to use**: Making UI changes, adjusting layouts, modifying styling
- **Shortcut**: `r` in terminal or `Ctrl+S` in IDE

```dart
// Example: Change this
Text('Original Text', style: TextStyle(color: Colors.blue))

// To this (Hot Reload will show changes instantly)
Text('Updated Text', style: TextStyle(color: Colors.red))
```

### Hot Restart
- **What**: Restarts the app completely, losing all state
- **When to use**: Adding new imports, changing main(), modifying app initialization
- **Shortcut**: `R` in terminal or `Ctrl+Shift+F5` in IDE

### Development Tips
```dart
// Use const constructors for better performance
const Text('Static text'); // Better than Text('Static text')

// Use meaningful variable names
Widget _buildUserProfile() {  // Clear purpose
  return Container(/*...*/);
}

// Extract widgets for reusability
Widget _buildCustomButton(String text, VoidCallback onPressed) {
  return ElevatedButton(
    onPressed: onPressed,
    child: Text(text),
  );
}
```

### Best Practices for Beginners

1. **Start Simple**: Begin with basic widgets and gradually add complexity
2. **Use Hot Reload**: Make small changes and see results immediately
3. **Read Documentation**: Flutter's documentation is excellent
4. **Practice Widget Composition**: Break complex UIs into smaller widgets
5. **Use IDE Tools**: Take advantage of Flutter-specific IDE features

### Next Steps
After mastering these basics, you'll be ready to:
- Explore different widget types
- Learn about layouts and positioning
- Understand state management
- Work with user input and navigation
- Integrate with external APIs

This foundation will prepare you for building more complex Flutter applications!