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

Flutter is Google's open-source UI toolkit for building natively compiled applications for mobile, web, and desktop from a single codebase. It uses the Dart programming language and provides a rich set of pre-designed widgets.

### Key Features:
- **Single Codebase**: Write once, run everywhere (iOS, Android, Web, Desktop)
- **Fast Development**: Hot reload for instant UI updates
- **Native Performance**: Compiled to native machine code
- **Rich UI**: Extensive widget library for beautiful UIs
- **Open Source**: Free and backed by Google

## Flutter Setup and Installation

### Prerequisites
- Operating System: Windows 10/11, macOS, or Linux
- Disk Space: At least 2.8 GB (excluding tools for IDEs)
- Git for Windows (if on Windows)

### Installation Steps

#### 1. Download Flutter SDK
```bash
# Download from https://flutter.dev/docs/get-started/install
# Extract to desired location (e.g., C:\flutter on Windows)
```

#### 2. Update PATH Environment Variable
```bash
# Windows
# Add C:\flutter\bin to your PATH

# macOS/Linux
export PATH="$PATH:`pwd`/flutter/bin"
```

#### 3. Verify Installation
```bash
flutter doctor
```

### IDE Setup
```bash
# Install VS Code extensions
# - Flutter
# - Dart

# Or use Android Studio with Flutter plugin
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

Flutter follows a reactive programming model with the following key concepts:

### Widget Tree
```dart
// Everything in Flutter is a Widget
// Widgets form a tree structure

MaterialApp
├── Scaffold
    ├── AppBar
    └── Body
        ├── Column
            ├── Text
            ├── Button
            └── Container
```

### Widget Types

#### Stateless Widgets
```dart
class WelcomeScreen extends StatelessWidget {
  final String userName;
  
  const WelcomeScreen({Key? key, required this.userName}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Welcome'),
      ),
      body: Center(
        child: Text('Hello, $userName!'),
      ),
    );
  }
}
```

#### Stateful Widgets
```dart
class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;
  
  void _incrementCounter() {
    setState(() {
      _count++;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Counter App'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Count:'),
            Text(
              '$_count',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

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

### Widget Hierarchy
```dart
// Basic widget structure
Widget build(BuildContext context) {
  return MaterialApp(        // Root widget
    home: Scaffold(          // Screen structure
      appBar: AppBar(        // Top app bar
        title: Text('Title'), // App bar content
      ),
      body: Container(       // Main content area
        child: Column(       // Vertical layout
          children: [        // List of child widgets
            Text('Hello'),   // Text widget
            Button(),        // Custom button widget
          ],
        ),
      ),
    ),
  );
}
```

### Common Widget Properties
```dart
// Most widgets accept these common properties:
Widget myWidget = Container(
  key: Key('unique_key'),           // Unique identifier
  child: Text('Content'),           // Single child widget
  // children: [Widget1(), Widget2()], // Multiple children (for some widgets)
  
  // Styling properties vary by widget type
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.symmetric(horizontal: 20),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
  ),
);
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