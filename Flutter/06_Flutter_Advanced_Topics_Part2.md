# Flutter Advanced Topics Guide - Part 2

## Table of Contents
1. [Platform Integration](#platform-integration)
2. [Performance Optimization](#performance-optimization)
3. [Error Handling and Debugging](#error-handling-and-debugging)
4. [Testing](#testing)

## Platform Integration

### Platform Channels
```dart
import 'package:flutter/services.dart';

class PlatformService {
  static const MethodChannel _channel = MethodChannel('com.example.platform');
  
  // Call native method
  static Future<String?> getNativeDeviceInfo() async {
    try {
      final String? result = await _channel.invokeMethod('getDeviceInfo');
      return result;
    } on PlatformException catch (e) {
      print('Failed to get device info: ${e.message}');
      return null;
    }
  }
  
  // Call native method with parameters
  static Future<bool> saveToNativeStorage(String key, String value) async {
    try {
      final bool result = await _channel.invokeMethod('saveToStorage', {
        'key': key,
        'value': value,
      });
      return result;
    } on PlatformException catch (e) {
      print('Failed to save to native storage: ${e.message}');
      return false;
    }
  }
  
  // Listen to native events
  static const EventChannel _eventChannel = 
      EventChannel('com.example.platform/events');
  
  static Stream<dynamic> get nativeEventStream {
    return _eventChannel.receiveBroadcastStream();
  }
}

// Usage example
class PlatformIntegrationScreen extends StatefulWidget {
  @override
  _PlatformIntegrationScreenState createState() => _PlatformIntegrationScreenState();
}

class _PlatformIntegrationScreenState extends State<PlatformIntegrationScreen> {
  String _deviceInfo = 'Unknown';
  String _eventData = 'No events';
  late StreamSubscription _eventSubscription;
  
  @override
  void initState() {
    super.initState();
    _getDeviceInfo();
    _listenToNativeEvents();
  }
  
  void _getDeviceInfo() async {
    final deviceInfo = await PlatformService.getNativeDeviceInfo();
    setState(() {
      _deviceInfo = deviceInfo ?? 'Failed to get device info';
    });
  }
  
  void _listenToNativeEvents() {
    _eventSubscription = PlatformService.nativeEventStream.listen(
      (event) {
        setState(() {
          _eventData = event.toString();
        });
      },
      onError: (error) {
        print('Error listening to native events: $error');
      },
    );
  }
  
  @override
  void dispose() {
    _eventSubscription.cancel();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Platform Integration')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Device Info:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            Text(_deviceInfo),
            SizedBox(height: 20),
            Text(
              'Native Events:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            Text(_eventData),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () async {
                final success = await PlatformService.saveToNativeStorage(
                  'test_key',
                  'test_value',
                );
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text(success ? 'Saved successfully' : 'Save failed'),
                  ),
                );
              },
              child: Text('Save to Native Storage'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Native Android Integration (Kotlin)
```kotlin
// MainActivity.kt
class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.example.platform"
    private val EVENT_CHANNEL = "com.example.platform/events"
    
    override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler {
            call, result ->
            when (call.method) {
                "getDeviceInfo" -> {
                    val deviceInfo = getDeviceInfo()
                    result.success(deviceInfo)
                }
                "saveToStorage" -> {
                    val key = call.argument<String>("key")
                    val value = call.argument<String>("value")
                    val success = saveToStorage(key, value)
                    result.success(success)
                }
                else -> {
                    result.notImplemented()
                }
            }
        }
        
        EventChannel(flutterEngine.dartExecutor.binaryMessenger, EVENT_CHANNEL).setStreamHandler(
            object : EventChannel.StreamHandler {
                override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
                    // Start sending events
                    startEventTimer(events)
                }
                
                override fun onCancel(arguments: Any?) {
                    // Stop sending events
                    stopEventTimer()
                }
            }
        )
    }
    
    private fun getDeviceInfo(): String {
        return "Android ${Build.VERSION.RELEASE}, Model: ${Build.MODEL}"
    }
    
    private fun saveToStorage(key: String?, value: String?): Boolean {
        return try {
            val sharedPref = getSharedPreferences("flutter_prefs", Context.MODE_PRIVATE)
            with(sharedPref.edit()) {
                putString(key, value)
                apply()
            }
            true
        } catch (e: Exception) {
            false
        }
    }
    
    private var eventTimer: Timer? = null
    
    private fun startEventTimer(events: EventChannel.EventSink?) {
        eventTimer = Timer()
        eventTimer?.scheduleAtFixedRate(object : TimerTask() {
            override fun run() {
                events?.success("Event at ${System.currentTimeMillis()}")
            }
        }, 0, 1000)
    }
    
    private fun stopEventTimer() {
        eventTimer?.cancel()
        eventTimer = null
    }
}
```

### Native iOS Integration (Swift)
```swift
// AppDelegate.swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    let platformChannel = FlutterMethodChannel(name: "com.example.platform",
                                              binaryMessenger: controller.binaryMessenger)
    
    platformChannel.setMethodCallHandler({
      (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
      
      switch call.method {
      case "getDeviceInfo":
        let deviceInfo = self.getDeviceInfo()
        result(deviceInfo)
      case "saveToStorage":
        let args = call.arguments as? [String: Any]
        let key = args?["key"] as? String
        let value = args?["value"] as? String
        let success = self.saveToStorage(key: key, value: value)
        result(success)
      default:
        result(FlutterMethodNotImplemented)
      }
    })
    
    let eventChannel = FlutterEventChannel(name: "com.example.platform/events",
                                          binaryMessenger: controller.binaryMessenger)
    eventChannel.setStreamHandler(EventStreamHandler())
    
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
  
  private func getDeviceInfo() -> String {
    let device = UIDevice.current
    return "iOS \(device.systemVersion), Model: \(device.model)"
  }
  
  private func saveToStorage(key: String?, value: String?) -> Bool {
    guard let key = key, let value = value else { return false }
    UserDefaults.standard.set(value, forKey: key)
    return true
  }
}

class EventStreamHandler: NSObject, FlutterStreamHandler {
  private var timer: Timer?
  
  func onListen(withArguments arguments: Any?, eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
      events("Event at \(Date().timeIntervalSince1970)")
    }
    return nil
  }
  
  func onCancel(withArguments arguments: Any?) -> FlutterError? {
    timer?.invalidate()
    timer = nil
    return nil
  }
}
```

## Performance Optimization

### Widget Optimization
```dart
class PerformanceOptimizedWidget extends StatefulWidget {
  @override
  _PerformanceOptimizedWidgetState createState() => _PerformanceOptimizedWidgetState();
}

class _PerformanceOptimizedWidgetState extends State<PerformanceOptimizedWidget> {
  List<Item> items = List.generate(1000, (index) => Item(id: index, name: 'Item $index'));
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Performance Optimization')),
      body: Column(
        children: [
          // Use const constructors for static widgets
          const OptimizedHeader(),
          
          // Use ListView.builder for large lists
          Expanded(
            child: ListView.builder(
              itemCount: items.length,
              itemBuilder: (context, index) {
                return OptimizedListItem(
                  key: ValueKey(items[index].id), // Use keys for list items
                  item: items[index],
                  onTap: () => _onItemTap(items[index]),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
  
  void _onItemTap(Item item) {
    // Handle item tap
    print('Tapped on ${item.name}');
  }
}

// Use const constructor for static widgets
class OptimizedHeader extends StatelessWidget {
  const OptimizedHeader({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      color: Colors.blue,
      child: Text(
        'Optimized List',
        style: TextStyle(
          color: Colors.white,
          fontSize: 18,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

// Separate widget for list items to prevent unnecessary rebuilds
class OptimizedListItem extends StatelessWidget {
  final Item item;
  final VoidCallback onTap;
  
  const OptimizedListItem({
    Key? key,
    required this.item,
    required this.onTap,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: CircleAvatar(
        child: Text(item.id.toString()),
      ),
      title: Text(item.name),
      onTap: onTap,
    );
  }
}

class Item {
  final int id;
  final String name;
  
  Item({required this.id, required this.name});
}
```

### Memory Management
```dart
class MemoryManagementExample extends StatefulWidget {
  @override
  _MemoryManagementExampleState createState() => _MemoryManagementExampleState();
}

class _MemoryManagementExampleState extends State<MemoryManagementExample> {
  late StreamSubscription _subscription;
  Timer? _timer;
  AnimationController? _animationController;
  
  @override
  void initState() {
    super.initState();
    
    // Example of proper subscription management
    _subscription = Stream.periodic(Duration(seconds: 1))
        .listen((event) {
      // Handle stream data
    });
    
    // Example of timer management
    _timer = Timer.periodic(Duration(seconds: 5), (timer) {
      // Periodic task
    });
  }
  
  @override
  void dispose() {
    // Always dispose of resources to prevent memory leaks
    _subscription.cancel();
    _timer?.cancel();
    _animationController?.dispose();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Memory Management')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Use RepaintBoundary for expensive widgets
            RepaintBoundary(
              child: ExpensiveWidget(),
            ),
            
            SizedBox(height: 20),
            
            // Use AutomaticKeepAliveClientMixin for tab views
            Text('Memory management best practices applied'),
          ],
        ),
      ),
    );
  }
}

class ExpensiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Expensive drawing operations
    return Container(
      width: 200,
      height: 200,
      child: CustomPaint(
        painter: ExpensivePainter(),
      ),
    );
  }
}

class ExpensivePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    // Complex drawing operations
    final paint = Paint()..color = Colors.blue;
    
    for (int i = 0; i < 100; i++) {
      canvas.drawCircle(
        Offset(
          Random().nextDouble() * size.width,
          Random().nextDouble() * size.height,
        ),
        5,
        paint,
      );
    }
  }
  
  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

### Lazy Loading and Pagination
```dart
class LazyLoadingExample extends StatefulWidget {
  @override
  _LazyLoadingExampleState createState() => _LazyLoadingExampleState();
}

class _LazyLoadingExampleState extends State<LazyLoadingExample> {
  List<String> items = [];
  bool isLoading = false;
  bool hasMore = true;
  ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _loadMoreItems();
    _scrollController.addListener(_onScroll);
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels == 
        _scrollController.position.maxScrollExtent) {
      _loadMoreItems();
    }
  }
  
  Future<void> _loadMoreItems() async {
    if (isLoading || !hasMore) return;
    
    setState(() {
      isLoading = true;
    });
    
    // Simulate API call
    await Future.delayed(Duration(seconds: 1));
    
    setState(() {
      final startIndex = items.length;
      final newItems = List.generate(
        20,
        (index) => 'Item ${startIndex + index}',
      );
      items.addAll(newItems);
      isLoading = false;
      
      // Simulate end of data
      if (items.length >= 100) {
        hasMore = false;
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Lazy Loading')),
      body: ListView.builder(
        controller: _scrollController,
        itemCount: items.length + (hasMore ? 1 : 0),
        itemBuilder: (context, index) {
          if (index == items.length) {
            return Padding(
              padding: EdgeInsets.all(16),
              child: Center(
                child: CircularProgressIndicator(),
              ),
            );
          }
          
          return ListTile(
            title: Text(items[index]),
          );
        },
      ),
    );
  }
}
```

## Error Handling and Debugging

### Global Error Handling
```dart
// main.dart
void main() {
  // Catch Flutter framework errors
  FlutterError.onError = (FlutterErrorDetails details) {
    FlutterError.presentError(details);
    // Log to crash reporting service
    logError(details.exception, details.stack);
  };
  
  // Catch other errors
  PlatformDispatcher.instance.onError = (error, stack) {
    logError(error, stack);
    return true;
  };
  
  runZonedGuarded<Future<void>>(() async {
    runApp(MyApp());
  }, (error, stack) {
    logError(error, stack);
  });
}

void logError(dynamic error, StackTrace? stack) {
  print('Error: $error');
  print('Stack trace: $stack');
  // Send to crash reporting service like Firebase Crashlytics
}

class ErrorHandler {
  static void handleError(dynamic error, StackTrace? stackTrace) {
    // Log error
    debugPrint('Error occurred: $error');
    debugPrint('Stack trace: $stackTrace');
    
    // Report to analytics/crash reporting
    // FirebaseCrashlytics.instance.recordError(error, stackTrace);
  }
  
  static Future<T?> tryAsync<T>(Future<T> Function() operation) async {
    try {
      return await operation();
    } catch (error, stackTrace) {
      handleError(error, stackTrace);
      return null;
    }
  }
  
  static T? trySync<T>(T Function() operation) {
    try {
      return operation();
    } catch (error, stackTrace) {
      handleError(error, stackTrace);
      return null;
    }
  }
}
```

### Custom Error Widgets
```dart
class ErrorWidget extends StatelessWidget {
  final String message;
  final VoidCallback? onRetry;
  
  const ErrorWidget({
    Key? key,
    required this.message,
    this.onRetry,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.error_outline,
              size: 64,
              color: Colors.red,
            ),
            SizedBox(height: 16),
            Text(
              'Something went wrong',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 8),
            Text(
              message,
              textAlign: TextAlign.center,
              style: TextStyle(color: Colors.grey[600]),
            ),
            if (onRetry != null) ...[
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: onRetry,
                child: Text('Retry'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}

// Usage with FutureBuilder
class ApiDataScreen extends StatefulWidget {
  @override
  _ApiDataScreenState createState() => _ApiDataScreenState();
}

class _ApiDataScreenState extends State<ApiDataScreen> {
  Future<List<Post>>? _future;
  
  @override
  void initState() {
    super.initState();
    _future = _fetchData();
  }
  
  Future<List<Post>> _fetchData() async {
    // Simulate API call that might fail
    await Future.delayed(Duration(seconds: 2));
    if (Random().nextBool()) {
      throw Exception('Network error occurred');
    }
    return List.generate(10, (index) => Post(id: index, title: 'Post $index'));
  }
  
  void _retry() {
    setState(() {
      _future = _fetchData();
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('API Data')),
      body: FutureBuilder<List<Post>>(
        future: _future,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (snapshot.hasError) {
            return ErrorWidget(
              message: snapshot.error.toString(),
              onRetry: _retry,
            );
          }
          
          if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return Center(child: Text('No data available'));
          }
          
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              final post = snapshot.data![index];
              return ListTile(
                title: Text(post.title),
              );
            },
          );
        },
      ),
    );
  }
}
```

## Testing

### Unit Tests
```dart
// test/models/user_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/models/user.dart';

void main() {
  group('User Model Tests', () {
    test('should create user from JSON', () {
      // Arrange
      final json = {
        'id': 1,
        'name': 'John Doe',
        'email': 'john@example.com',
      };
      
      // Act
      final user = User.fromJson(json);
      
      // Assert
      expect(user.id, 1);
      expect(user.name, 'John Doe');
      expect(user.email, 'john@example.com');
    });
    
    test('should convert user to JSON', () {
      // Arrange
      final user = User(
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
      );
      
      // Act
      final json = user.toJson();
      
      // Assert
      expect(json['id'], 1);
      expect(json['name'], 'John Doe');
      expect(json['email'], 'john@example.com');
    });
    
    test('should validate email format', () {
      // Arrange
      final user = User(
        id: 1,
        name: 'John Doe',
        email: 'invalid-email',
      );
      
      // Act & Assert
      expect(user.isEmailValid, false);
    });
  });
}
```

### Widget Tests
```dart
// test/widgets/custom_button_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/widgets/custom_button.dart';

void main() {
  group('CustomButton Widget Tests', () {
    testWidgets('should display button text', (WidgetTester tester) async {
      // Arrange
      const buttonText = 'Test Button';
      
      // Act
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomButton(
              text: buttonText,
              onPressed: () {},
            ),
          ),
        ),
      );
      
      // Assert
      expect(find.text(buttonText), findsOneWidget);
    });
    
    testWidgets('should call onPressed when tapped', (WidgetTester tester) async {
      // Arrange
      bool wasPressed = false;
      
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomButton(
              text: 'Test Button',
              onPressed: () {
                wasPressed = true;
              },
            ),
          ),
        ),
      );
      
      // Act
      await tester.tap(find.byType(CustomButton));
      await tester.pumpAndSettle();
      
      // Assert
      expect(wasPressed, true);
    });
    
    testWidgets('should show loading indicator when isLoading is true', 
        (WidgetTester tester) async {
      // Arrange & Act
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomButton(
              text: 'Test Button',
              isLoading: true,
              onPressed: () {},
            ),
          ),
        ),
      );
      
      // Assert
      expect(find.byType(CircularProgressIndicator), findsOneWidget);
      expect(find.text('Test Button'), findsNothing);
    });
  });
}
```

### Integration Tests
```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:your_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  group('App Integration Tests', () {
    testWidgets('complete user flow test', (WidgetTester tester) async {
      // Start the app
      app.main();
      await tester.pumpAndSettle();
      
      // Verify home screen
      expect(find.text('Welcome'), findsOneWidget);
      
      // Navigate to login
      await tester.tap(find.byIcon(Icons.login));
      await tester.pumpAndSettle();
      
      // Fill login form
      await tester.enterText(find.byKey(Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(Key('password_field')), 'password123');
      
      // Submit login
      await tester.tap(find.byKey(Key('login_button')));
      await tester.pumpAndSettle();
      
      // Verify successful login
      expect(find.text('Dashboard'), findsOneWidget);
    });
    
    testWidgets('network error handling test', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle();
      
      // Navigate to a screen that makes network requests
      await tester.tap(find.text('Load Data'));
      await tester.pumpAndSettle();
      
      // Wait for potential network error
      await tester.pump(Duration(seconds: 5));
      
      // Check if error handling works
      if (find.text('Retry').evaluate().isNotEmpty) {
        await tester.tap(find.text('Retry'));
        await tester.pumpAndSettle();
      }
    });
  });
}
```

### Testing Best Practices
```dart
// test/test_helpers.dart
class TestHelpers {
  // Create a wrapped widget for testing
  static Widget createTestWidget(Widget child) {
    return MaterialApp(
      home: Scaffold(
        body: child,
      ),
    );
  }
  
  // Mock data generators
  static User createMockUser({
    int id = 1,
    String name = 'Test User',
    String email = 'test@example.com',
  }) {
    return User(id: id, name: name, email: email);
  }
  
  static List<Post> createMockPosts(int count) {
    return List.generate(
      count,
      (index) => Post(
        id: index,
        userId: 1,
        title: 'Test Post $index',
        body: 'Test content for post $index',
      ),
    );
  }
}

// Custom matchers
Matcher hasErrorDialog() {
  return findsOneWidget.having(
    (finder) => finder.evaluate().first.widget,
    'widget',
    isA<AlertDialog>(),
  );
}
```

This comprehensive guide covers platform integration with native code, performance optimization techniques, proper error handling, and comprehensive testing strategies for Flutter applications.