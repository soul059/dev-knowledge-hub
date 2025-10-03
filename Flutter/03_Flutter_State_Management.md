# Flutter State Management Guide

## Table of Contents
1. [Introduction to State Management](#introduction-to-state-management)
2. [setState (Built-in)](#setstate-built-in)
3. [InheritedWidget](#inheritedwidget)
4. [Provider Package](#provider-package)
5. [Bloc Pattern](#bloc-pattern)
6. [Riverpod](#riverpod)
7. [GetX](#getx)
8. [Choosing the Right Solution](#choosing-the-right-solution)

## Introduction to State Management

State management is one of the most important concepts in Flutter development. It refers to how you manage and share data across your application. Different approaches work better for different scenarios.

### Types of State
- **Ephemeral State**: Local widget state (e.g., current page in PageView)
- **App State**: Shared across multiple screens (e.g., user login status, shopping cart)

### Key Concepts
- **State**: Data that can change over time
- **Rebuilding**: When widgets update their UI based on state changes
- **Lifting State Up**: Moving state to a common ancestor widget

## setState (Built-in)

The simplest form of state management, perfect for local widget state.

### Basic Counter Example
```dart
class CounterApp extends StatefulWidget {
  @override
  _CounterAppState createState() => _CounterAppState();
}

class _CounterAppState extends State<CounterApp> {
  int _counter = 0;
  
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
  
  void _decrementCounter() {
    setState(() {
      _counter--;
    });
  }
  
  void _resetCounter() {
    setState(() {
      _counter = 0;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('setState Counter'),
        backgroundColor: Colors.blue,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              _counter >= 0 ? Icons.trending_up : Icons.trending_down,
              size: 64,
              color: _counter >= 0 ? Colors.green : Colors.red,
            ),
            SizedBox(height: 20),
            Text(
              'Counter Value:',
              style: TextStyle(fontSize: 18),
            ),
            Text(
              '$_counter',
              style: TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
                color: _counter >= 0 ? Colors.green : Colors.red,
              ),
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                FloatingActionButton(
                  onPressed: _decrementCounter,
                  child: Icon(Icons.remove),
                  backgroundColor: Colors.red,
                  heroTag: "decrement",
                ),
                FloatingActionButton(
                  onPressed: _resetCounter,
                  child: Icon(Icons.refresh),
                  backgroundColor: Colors.grey,
                  heroTag: "reset",
                ),
                FloatingActionButton(
                  onPressed: _incrementCounter,
                  child: Icon(Icons.add),
                  backgroundColor: Colors.green,
                  heroTag: "increment",
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

### Form Validation with setState
```dart
class FormExample extends StatefulWidget {
  @override
  _FormExampleState createState() => _FormExampleState();
}

class _FormExampleState extends State<FormExample> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  
  bool _isLoading = false;
  bool _agreeToTerms = false;
  String _selectedCountry = 'USA';
  
  void _submitForm() async {
    if (!_formKey.currentState!.validate() || !_agreeToTerms) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Please fill all fields and agree to terms'),
          backgroundColor: Colors.red,
        ),
      );
      return;
    }
    
    setState(() {
      _isLoading = true;
    });
    
    // Simulate API call
    await Future.delayed(Duration(seconds: 2));
    
    setState(() {
      _isLoading = false;
    });
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Form submitted successfully!'),
        backgroundColor: Colors.green,
      ),
    );
    
    // Clear form
    _nameController.clear();
    _emailController.clear();
    setState(() {
      _agreeToTerms = false;
      _selectedCountry = 'USA';
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('setState Form Example')),
      body: Form(
        key: _formKey,
        child: SingleChildScrollView(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: 'Name',
                  border: OutlineInputBorder(),
                ),
                validator: (value) => value?.isEmpty == true ? 'Required' : null,
              ),
              SizedBox(height: 16),
              
              TextFormField(
                controller: _emailController,
                decoration: InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value?.isEmpty == true) return 'Required';
                  if (!value!.contains('@')) return 'Invalid email';
                  return null;
                },
              ),
              SizedBox(height: 16),
              
              DropdownButtonFormField<String>(
                value: _selectedCountry,
                decoration: InputDecoration(
                  labelText: 'Country',
                  border: OutlineInputBorder(),
                ),
                items: ['USA', 'Canada', 'UK', 'India']
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
              
              CheckboxListTile(
                title: Text('I agree to the terms and conditions'),
                value: _agreeToTerms,
                onChanged: (value) {
                  setState(() {
                    _agreeToTerms = value!;
                  });
                },
              ),
              SizedBox(height: 24),
              
              SizedBox(
                width: double.infinity,
                height: 50,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _submitForm,
                  child: _isLoading
                      ? CircularProgressIndicator(color: Colors.white)
                      : Text('Submit'),
                ),
              ),
            ],
          ),
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

### When to Use setState
- ✅ Local widget state
- ✅ Simple forms and UI interactions
- ✅ Prototype development
- ❌ Sharing state between screens
- ❌ Complex state logic
- ❌ Large applications

## InheritedWidget

A lower-level widget that allows data to be passed down the widget tree efficiently.

### Basic InheritedWidget Example
```dart
// Data model
class UserData {
  final String name;
  final String email;
  final bool isLoggedIn;
  
  UserData({
    required this.name,
    required this.email,
    required this.isLoggedIn,
  });
  
  UserData copyWith({
    String? name,
    String? email,
    bool? isLoggedIn,
  }) {
    return UserData(
      name: name ?? this.name,
      email: email ?? this.email,
      isLoggedIn: isLoggedIn ?? this.isLoggedIn,
    );
  }
}

// InheritedWidget
class UserInheritedWidget extends InheritedWidget {
  final UserData userData;
  final Function(UserData) updateUser;
  
  UserInheritedWidget({
    Key? key,
    required this.userData,
    required this.updateUser,
    required Widget child,
  }) : super(key: key, child: child);
  
  static UserInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<UserInheritedWidget>();
  }
  
  @override
  bool updateShouldNotify(UserInheritedWidget oldWidget) {
    return userData != oldWidget.userData;
  }
}

// Main App with InheritedWidget
class InheritedWidgetExample extends StatefulWidget {
  @override
  _InheritedWidgetExampleState createState() => _InheritedWidgetExampleState();
}

class _InheritedWidgetExampleState extends State<InheritedWidgetExample> {
  UserData _userData = UserData(
    name: 'Guest',
    email: '',
    isLoggedIn: false,
  );
  
  void _updateUser(UserData newUserData) {
    setState(() {
      _userData = newUserData;
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return UserInheritedWidget(
      userData: _userData,
      updateUser: _updateUser,
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('InheritedWidget Example')),
          body: Column(
            children: [
              UserProfile(),
              LoginForm(),
            ],
          ),
        ),
      ),
    );
  }
}

// Child widget that consumes the InheritedWidget
class UserProfile extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userWidget = UserInheritedWidget.of(context);
    final userData = userWidget!.userData;
    
    return Container(
      padding: EdgeInsets.all(16),
      margin: EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: userData.isLoggedIn ? Colors.green[100] : Colors.grey[100],
        borderRadius: BorderRadius.circular(8),
        border: Border.all(
          color: userData.isLoggedIn ? Colors.green : Colors.grey,
        ),
      ),
      child: Column(
        children: [
          Icon(
            userData.isLoggedIn ? Icons.person : Icons.person_outline,
            size: 48,
            color: userData.isLoggedIn ? Colors.green : Colors.grey,
          ),
          SizedBox(height: 8),
          Text(
            'Name: ${userData.name}',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          if (userData.isLoggedIn)
            Text('Email: ${userData.email}'),
          SizedBox(height: 8),
          Text(
            'Status: ${userData.isLoggedIn ? "Logged In" : "Guest"}',
            style: TextStyle(
              color: userData.isLoggedIn ? Colors.green : Colors.grey,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }
}

// Another child widget
class LoginForm extends StatefulWidget {
  @override
  _LoginFormState createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    final userWidget = UserInheritedWidget.of(context);
    final userData = userWidget!.userData;
    
    if (userData.isLoggedIn) {
      return Padding(
        padding: EdgeInsets.all(16),
        child: ElevatedButton(
          onPressed: () {
            userWidget.updateUser(UserData(
              name: 'Guest',
              email: '',
              isLoggedIn: false,
            ));
          },
          child: Text('Logout'),
          style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
        ),
      );
    }
    
    return Padding(
      padding: EdgeInsets.all(16),
      child: Column(
        children: [
          TextField(
            controller: _nameController,
            decoration: InputDecoration(
              labelText: 'Name',
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 12),
          TextField(
            controller: _emailController,
            decoration: InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
            ),
          ),
          SizedBox(height: 16),
          ElevatedButton(
            onPressed: () {
              if (_nameController.text.isNotEmpty && 
                  _emailController.text.isNotEmpty) {
                userWidget.updateUser(UserData(
                  name: _nameController.text,
                  email: _emailController.text,
                  isLoggedIn: true,
                ));
                _nameController.clear();
                _emailController.clear();
              }
            },
            child: Text('Login'),
          ),
        ],
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

## Provider Package

Provider is the recommended state management solution by the Flutter team for most applications.

### Setup
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.0.0
```

### Basic Provider Example
```dart
import 'package:provider/provider.dart';

// Model class
class Counter extends ChangeNotifier {
  int _count = 0;
  
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
  
  void decrement() {
    _count--;
    notifyListeners();
  }
  
  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// Main App
class ProviderExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => Counter(),
      child: MaterialApp(
        home: CounterScreen(),
      ),
    );
  }
}

// Consumer Widget
class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Provider Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Count:',
              style: TextStyle(fontSize: 24),
            ),
            Consumer<Counter>(
              builder: (context, counter, child) {
                return Text(
                  '${counter.count}',
                  style: TextStyle(
                    fontSize: 48,
                    fontWeight: FontWeight.bold,
                    color: counter.count >= 0 ? Colors.blue : Colors.red,
                  ),
                );
              },
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                FloatingActionButton(
                  onPressed: () => context.read<Counter>().decrement(),
                  child: Icon(Icons.remove),
                  heroTag: "dec",
                ),
                FloatingActionButton(
                  onPressed: () => context.read<Counter>().reset(),
                  child: Icon(Icons.refresh),
                  heroTag: "reset",
                ),
                FloatingActionButton(
                  onPressed: () => context.read<Counter>().increment(),
                  child: Icon(Icons.add),
                  heroTag: "inc",
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

### Advanced Provider Example - Shopping Cart
```dart
// Product model
class Product {
  final String id;
  final String name;
  final double price;
  final String image;
  
  Product({
    required this.id,
    required this.name,
    required this.price,
    required this.image,
  });
}

// Cart item model
class CartItem {
  final Product product;
  int quantity;
  
  CartItem({
    required this.product,
    this.quantity = 1,
  });
  
  double get totalPrice => product.price * quantity;
}

// Shopping Cart Provider
class ShoppingCart extends ChangeNotifier {
  final List<CartItem> _items = [];
  
  List<CartItem> get items => List.unmodifiable(_items);
  
  int get itemCount => _items.fold(0, (sum, item) => sum + item.quantity);
  
  double get totalAmount => _items.fold(0, (sum, item) => sum + item.totalPrice);
  
  void addItem(Product product) {
    final existingIndex = _items.indexWhere(
      (item) => item.product.id == product.id,
    );
    
    if (existingIndex >= 0) {
      _items[existingIndex].quantity++;
    } else {
      _items.add(CartItem(product: product));
    }
    
    notifyListeners();
  }
  
  void removeItem(String productId) {
    _items.removeWhere((item) => item.product.id == productId);
    notifyListeners();
  }
  
  void updateQuantity(String productId, int quantity) {
    final index = _items.indexWhere((item) => item.product.id == productId);
    if (index >= 0) {
      if (quantity <= 0) {
        _items.removeAt(index);
      } else {
        _items[index].quantity = quantity;
      }
      notifyListeners();
    }
  }
  
  void clearCart() {
    _items.clear();
    notifyListeners();
  }
}

// Products Provider
class ProductsProvider extends ChangeNotifier {
  final List<Product> _products = [
    Product(id: '1', name: 'Laptop', price: 999.99, image: '💻'),
    Product(id: '2', name: 'Phone', price: 599.99, image: '📱'),
    Product(id: '3', name: 'Headphones', price: 199.99, image: '🎧'),
    Product(id: '4', name: 'Watch', price: 299.99, image: '⌚'),
  ];
  
  List<Product> get products => List.unmodifiable(_products);
}

// Main App with Multiple Providers
class ShoppingApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => ProductsProvider()),
        ChangeNotifierProvider(create: (context) => ShoppingCart()),
      ],
      child: MaterialApp(
        title: 'Shopping App',
        home: ProductListScreen(),
      ),
    );
  }
}

// Product List Screen
class ProductListScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Products'),
        actions: [
          Consumer<ShoppingCart>(
            builder: (context, cart, child) {
              return Stack(
                children: [
                  IconButton(
                    icon: Icon(Icons.shopping_cart),
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => CartScreen()),
                      );
                    },
                  ),
                  if (cart.itemCount > 0)
                    Positioned(
                      right: 8,
                      top: 8,
                      child: Container(
                        padding: EdgeInsets.all(2),
                        decoration: BoxDecoration(
                          color: Colors.red,
                          borderRadius: BorderRadius.circular(10),
                        ),
                        constraints: BoxConstraints(
                          minWidth: 16,
                          minHeight: 16,
                        ),
                        child: Text(
                          '${cart.itemCount}',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                          ),
                          textAlign: TextAlign.center,
                        ),
                      ),
                    ),
                ],
              );
            },
          ),
        ],
      ),
      body: Consumer<ProductsProvider>(
        builder: (context, productsProvider, child) {
          return ListView.builder(
            itemCount: productsProvider.products.length,
            itemBuilder: (context, index) {
              final product = productsProvider.products[index];
              return Card(
                margin: EdgeInsets.all(8),
                child: ListTile(
                  leading: Text(
                    product.image,
                    style: TextStyle(fontSize: 32),
                  ),
                  title: Text(product.name),
                  subtitle: Text('\$${product.price.toStringAsFixed(2)}'),
                  trailing: Consumer<ShoppingCart>(
                    builder: (context, cart, child) {
                      return ElevatedButton(
                        onPressed: () {
                          cart.addItem(product);
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text('${product.name} added to cart'),
                              duration: Duration(seconds: 1),
                            ),
                          );
                        },
                        child: Text('Add to Cart'),
                      );
                    },
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

// Cart Screen
class CartScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Cart'),
        actions: [
          Consumer<ShoppingCart>(
            builder: (context, cart, child) {
              return cart.items.isNotEmpty
                  ? IconButton(
                      icon: Icon(Icons.clear_all),
                      onPressed: () {
                        cart.clearCart();
                      },
                    )
                  : Container();
            },
          ),
        ],
      ),
      body: Consumer<ShoppingCart>(
        builder: (context, cart, child) {
          if (cart.items.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.shopping_cart_outlined, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text(
                    'Your cart is empty',
                    style: TextStyle(fontSize: 18, color: Colors.grey),
                  ),
                ],
              ),
            );
          }
          
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cart.items.length,
                  itemBuilder: (context, index) {
                    final cartItem = cart.items[index];
                    return Card(
                      margin: EdgeInsets.all(8),
                      child: ListTile(
                        leading: Text(
                          cartItem.product.image,
                          style: TextStyle(fontSize: 32),
                        ),
                        title: Text(cartItem.product.name),
                        subtitle: Text(
                          '\$${cartItem.product.price.toStringAsFixed(2)} each',
                        ),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              icon: Icon(Icons.remove),
                              onPressed: () {
                                cart.updateQuantity(
                                  cartItem.product.id,
                                  cartItem.quantity - 1,
                                );
                              },
                            ),
                            Text('${cartItem.quantity}'),
                            IconButton(
                              icon: Icon(Icons.add),
                              onPressed: () {
                                cart.updateQuantity(
                                  cartItem.product.id,
                                  cartItem.quantity + 1,
                                );
                              },
                            ),
                            IconButton(
                              icon: Icon(Icons.delete, color: Colors.red),
                              onPressed: () {
                                cart.removeItem(cartItem.product.id);
                              },
                            ),
                          ],
                        ),
                      ),
                    );
                  },
                ),
              ),
              Container(
                padding: EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.grey[100],
                  border: Border(top: BorderSide(color: Colors.grey[300]!)),
                ),
                child: Column(
                  children: [
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text(
                          'Total Items: ${cart.itemCount}',
                          style: TextStyle(fontSize: 16),
                        ),
                        Text(
                          'Total: \$${cart.totalAmount.toStringAsFixed(2)}',
                          style: TextStyle(
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                    SizedBox(height: 16),
                    SizedBox(
                      width: double.infinity,
                      child: ElevatedButton(
                        onPressed: () {
                          // Implement checkout logic
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text('Checkout functionality would go here'),
                            ),
                          );
                        },
                        child: Text('Checkout'),
                        style: ElevatedButton.styleFrom(
                          padding: EdgeInsets.symmetric(vertical: 16),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

## Bloc Pattern

BLoC (Business Logic Component) pattern separates business logic from UI components using streams.

### Setup
```yaml
dependencies:
  flutter_bloc: ^8.0.0
  equatable: ^2.0.0
```

### Basic BLoC Example
```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// Events
abstract class CounterEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class CounterIncrement extends CounterEvent {}
class CounterDecrement extends CounterEvent {}
class CounterReset extends CounterEvent {}

// States
abstract class CounterState extends Equatable {
  @override
  List<Object> get props => [];
}

class CounterInitial extends CounterState {}

class CounterValue extends CounterState {
  final int value;
  
  CounterValue(this.value);
  
  @override
  List<Object> get props => [value];
}

// BLoC
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    on<CounterIncrement>((event, emit) {
      final currentState = state;
      if (currentState is CounterValue) {
        emit(CounterValue(currentState.value + 1));
      } else {
        emit(CounterValue(1));
      }
    });
    
    on<CounterDecrement>((event, emit) {
      final currentState = state;
      if (currentState is CounterValue) {
        emit(CounterValue(currentState.value - 1));
      } else {
        emit(CounterValue(-1));
      }
    });
    
    on<CounterReset>((event, emit) {
      emit(CounterValue(0));
    });
  }
}

// UI
class BlocCounterApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CounterBloc(),
      child: MaterialApp(
        home: CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('BLoC Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            BlocBuilder<CounterBloc, CounterState>(
              builder: (context, state) {
                int value = 0;
                if (state is CounterValue) {
                  value = state.value;
                }
                
                return Column(
                  children: [
                    Text(
                      'Counter Value:',
                      style: TextStyle(fontSize: 24),
                    ),
                    Text(
                      '$value',
                      style: TextStyle(
                        fontSize: 48,
                        fontWeight: FontWeight.bold,
                        color: value >= 0 ? Colors.blue : Colors.red,
                      ),
                    ),
                  ],
                );
              },
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                FloatingActionButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(CounterDecrement());
                  },
                  child: Icon(Icons.remove),
                  heroTag: "dec",
                ),
                FloatingActionButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(CounterReset());
                  },
                  child: Icon(Icons.refresh),
                  heroTag: "reset",
                ),
                FloatingActionButton(
                  onPressed: () {
                    context.read<CounterBloc>().add(CounterIncrement());
                  },
                  child: Icon(Icons.add),
                  heroTag: "inc",
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

### Advanced BLoC Example - Todo App
```dart
// Todo Model
class Todo extends Equatable {
  final String id;
  final String title;
  final bool isCompleted;
  final DateTime createdAt;
  
  Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
    required this.createdAt,
  });
  
  Todo copyWith({
    String? id,
    String? title,
    bool? isCompleted,
    DateTime? createdAt,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
    );
  }
  
  @override
  List<Object> get props => [id, title, isCompleted, createdAt];
}

// Todo Events
abstract class TodoEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class LoadTodos extends TodoEvent {}

class AddTodo extends TodoEvent {
  final String title;
  
  AddTodo(this.title);
  
  @override
  List<Object> get props => [title];
}

class UpdateTodo extends TodoEvent {
  final Todo todo;
  
  UpdateTodo(this.todo);
  
  @override
  List<Object> get props => [todo];
}

class DeleteTodo extends TodoEvent {
  final String id;
  
  DeleteTodo(this.id);
  
  @override
  List<Object> get props => [id];
}

class ToggleTodo extends TodoEvent {
  final String id;
  
  ToggleTodo(this.id);
  
  @override
  List<Object> get props => [id];
}

// Todo States
abstract class TodoState extends Equatable {
  @override
  List<Object> get props => [];
}

class TodoInitial extends TodoState {}

class TodoLoading extends TodoState {}

class TodoLoaded extends TodoState {
  final List<Todo> todos;
  
  TodoLoaded(this.todos);
  
  @override
  List<Object> get props => [todos];
}

class TodoError extends TodoState {
  final String message;
  
  TodoError(this.message);
  
  @override
  List<Object> get props => [message];
}

// Todo Repository
class TodoRepository {
  final List<Todo> _todos = [];
  
  Future<List<Todo>> getAllTodos() async {
    await Future.delayed(Duration(milliseconds: 500)); // Simulate network delay
    return List.from(_todos);
  }
  
  Future<void> addTodo(Todo todo) async {
    await Future.delayed(Duration(milliseconds: 300));
    _todos.add(todo);
  }
  
  Future<void> updateTodo(Todo todo) async {
    await Future.delayed(Duration(milliseconds: 300));
    final index = _todos.indexWhere((t) => t.id == todo.id);
    if (index >= 0) {
      _todos[index] = todo;
    }
  }
  
  Future<void> deleteTodo(String id) async {
    await Future.delayed(Duration(milliseconds: 300));
    _todos.removeWhere((t) => t.id == id);
  }
}

// Todo BLoC
class TodoBloc extends Bloc<TodoEvent, TodoState> {
  final TodoRepository repository;
  
  TodoBloc({required this.repository}) : super(TodoInitial()) {
    on<LoadTodos>(_onLoadTodos);
    on<AddTodo>(_onAddTodo);
    on<UpdateTodo>(_onUpdateTodo);
    on<DeleteTodo>(_onDeleteTodo);
    on<ToggleTodo>(_onToggleTodo);
  }
  
  Future<void> _onLoadTodos(LoadTodos event, Emitter<TodoState> emit) async {
    emit(TodoLoading());
    try {
      final todos = await repository.getAllTodos();
      emit(TodoLoaded(todos));
    } catch (e) {
      emit(TodoError('Failed to load todos'));
    }
  }
  
  Future<void> _onAddTodo(AddTodo event, Emitter<TodoState> emit) async {
    if (state is TodoLoaded) {
      final currentTodos = (state as TodoLoaded).todos;
      final newTodo = Todo(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        title: event.title,
        createdAt: DateTime.now(),
      );
      
      try {
        await repository.addTodo(newTodo);
        emit(TodoLoaded([...currentTodos, newTodo]));
      } catch (e) {
        emit(TodoError('Failed to add todo'));
      }
    }
  }
  
  Future<void> _onUpdateTodo(UpdateTodo event, Emitter<TodoState> emit) async {
    if (state is TodoLoaded) {
      final currentTodos = (state as TodoLoaded).todos;
      try {
        await repository.updateTodo(event.todo);
        final updatedTodos = currentTodos.map((todo) {
          return todo.id == event.todo.id ? event.todo : todo;
        }).toList();
        emit(TodoLoaded(updatedTodos));
      } catch (e) {
        emit(TodoError('Failed to update todo'));
      }
    }
  }
  
  Future<void> _onDeleteTodo(DeleteTodo event, Emitter<TodoState> emit) async {
    if (state is TodoLoaded) {
      final currentTodos = (state as TodoLoaded).todos;
      try {
        await repository.deleteTodo(event.id);
        final updatedTodos = currentTodos.where((todo) => todo.id != event.id).toList();
        emit(TodoLoaded(updatedTodos));
      } catch (e) {
        emit(TodoError('Failed to delete todo'));
      }
    }
  }
  
  Future<void> _onToggleTodo(ToggleTodo event, Emitter<TodoState> emit) async {
    if (state is TodoLoaded) {
      final currentTodos = (state as TodoLoaded).todos;
      final todoIndex = currentTodos.indexWhere((todo) => todo.id == event.id);
      
      if (todoIndex >= 0) {
        final todo = currentTodos[todoIndex];
        final updatedTodo = todo.copyWith(isCompleted: !todo.isCompleted);
        add(UpdateTodo(updatedTodo));
      }
    }
  }
}

// Todo App UI
class TodoApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RepositoryProvider(
      create: (context) => TodoRepository(),
      child: BlocProvider(
        create: (context) => TodoBloc(
          repository: context.read<TodoRepository>(),
        )..add(LoadTodos()),
        child: MaterialApp(
          home: TodoScreen(),
        ),
      ),
    );
  }
}

class TodoScreen extends StatelessWidget {
  final _titleController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('BLoC Todo App'),
        backgroundColor: Colors.blue,
      ),
      body: Column(
        children: [
          // Add Todo Section
          Container(
            padding: EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.grey[100],
              border: Border(bottom: BorderSide(color: Colors.grey[300]!)),
            ),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _titleController,
                    decoration: InputDecoration(
                      hintText: 'Enter todo title',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (value) => _addTodo(context, value),
                  ),
                ),
                SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () => _addTodo(context, _titleController.text),
                  child: Text('Add'),
                ),
              ],
            ),
          ),
          
          // Todo List
          Expanded(
            child: BlocBuilder<TodoBloc, TodoState>(
              builder: (context, state) {
                if (state is TodoLoading) {
                  return Center(child: CircularProgressIndicator());
                }
                
                if (state is TodoError) {
                  return Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(Icons.error_outline, size: 64, color: Colors.red),
                        SizedBox(height: 16),
                        Text(
                          state.message,
                          style: TextStyle(color: Colors.red, fontSize: 16),
                        ),
                        SizedBox(height: 16),
                        ElevatedButton(
                          onPressed: () {
                            context.read<TodoBloc>().add(LoadTodos());
                          },
                          child: Text('Retry'),
                        ),
                      ],
                    ),
                  );
                }
                
                if (state is TodoLoaded) {
                  final todos = state.todos;
                  
                  if (todos.isEmpty) {
                    return Center(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Icon(Icons.check_circle_outline, 
                               size: 64, color: Colors.grey),
                          SizedBox(height: 16),
                          Text(
                            'No todos yet!',
                            style: TextStyle(fontSize: 18, color: Colors.grey),
                          ),
                          Text(
                            'Add your first todo above',
                            style: TextStyle(color: Colors.grey),
                          ),
                        ],
                      ),
                    );
                  }
                  
                  return ListView.builder(
                    itemCount: todos.length,
                    itemBuilder: (context, index) {
                      final todo = todos[index];
                      return Card(
                        margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                        child: ListTile(
                          leading: Checkbox(
                            value: todo.isCompleted,
                            onChanged: (value) {
                              context.read<TodoBloc>().add(ToggleTodo(todo.id));
                            },
                          ),
                          title: Text(
                            todo.title,
                            style: TextStyle(
                              decoration: todo.isCompleted 
                                  ? TextDecoration.lineThrough 
                                  : null,
                              color: todo.isCompleted 
                                  ? Colors.grey 
                                  : null,
                            ),
                          ),
                          subtitle: Text(
                            'Created: ${todo.createdAt.day}/${todo.createdAt.month}/${todo.createdAt.year}',
                            style: TextStyle(fontSize: 12, color: Colors.grey),
                          ),
                          trailing: IconButton(
                            icon: Icon(Icons.delete, color: Colors.red),
                            onPressed: () {
                              context.read<TodoBloc>().add(DeleteTodo(todo.id));
                            },
                          ),
                        ),
                      );
                    },
                  );
                }
                
                return Center(child: Text('Unknown state'));
              },
            ),
          ),
        ],
      ),
    );
  }
  
  void _addTodo(BuildContext context, String title) {
    if (title.trim().isNotEmpty) {
      context.read<TodoBloc>().add(AddTodo(title.trim()));
      _titleController.clear();
    }
  }
}
```

## Riverpod

Riverpod is a reactive caching and data-binding framework that's a complete rewrite of Provider.

### Setup
```yaml
dependencies:
  flutter_riverpod: ^2.0.0
```

### Basic Riverpod Example
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Simple state provider
final counterProvider = StateProvider<int>((ref) => 0);

// Computed provider
final doubledCounterProvider = Provider<int>((ref) {
  final count = ref.watch(counterProvider);
  return count * 2;
});

// Async provider
final userProvider = FutureProvider<String>((ref) async {
  await Future.delayed(Duration(seconds: 2));
  return 'John Doe';
});

// Riverpod App
class RiverpodApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ProviderScope(
      child: MaterialApp(
        home: CounterScreen(),
      ),
    );
  }
}

class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    final doubledCount = ref.watch(doubledCounterProvider);
    final userAsync = ref.watch(userProvider);
    
    return Scaffold(
      appBar: AppBar(title: Text('Riverpod Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            userAsync.when(
              data: (user) => Text('Hello, $user!', style: TextStyle(fontSize: 18)),
              loading: () => CircularProgressIndicator(),
              error: (error, stack) => Text('Error: $error'),
            ),
            SizedBox(height: 30),
            Text('Count: $count', style: TextStyle(fontSize: 24)),
            Text('Doubled: $doubledCount', style: TextStyle(fontSize: 18)),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                FloatingActionButton(
                  onPressed: () => ref.read(counterProvider.notifier).state--,
                  child: Icon(Icons.remove),
                  heroTag: "dec",
                ),
                FloatingActionButton(
                  onPressed: () => ref.read(counterProvider.notifier).state = 0,
                  child: Icon(Icons.refresh),
                  heroTag: "reset",
                ),
                FloatingActionButton(
                  onPressed: () => ref.read(counterProvider.notifier).state++,
                  child: Icon(Icons.add),
                  heroTag: "inc",
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

## Choosing the Right Solution

### Decision Matrix

| Scenario | Recommended Solution | Why |
|----------|---------------------|-----|
| Simple UI state, forms | setState | Simplest, built-in |
| Sharing data down widget tree | InheritedWidget | Low-level, efficient |
| Medium complexity apps | Provider | Recommended by Flutter team |
| Complex business logic | BLoC | Separates business logic |
| Modern reactive approach | Riverpod | Better Developer Experience |
| Rapid prototyping | GetX | Less boilerplate |

### Guidelines

**Use setState when:**
- Managing local widget state
- Simple forms and toggles
- Prototyping
- Learning Flutter basics

**Use Provider when:**
- Need to share state across widgets
- Building production apps
- Want Flutter team's recommended approach
- Working with medium complexity

**Use BLoC when:**
- Complex business logic
- Need to separate concerns
- Working in teams
- Building large applications
- Need testability

**Use Riverpod when:**
- Want modern reactive programming
- Need compile-time safety
- Building new applications
- Want better developer experience

Each state management solution has its place. Start with setState for learning, move to Provider for most applications, and consider BLoC or Riverpod for complex scenarios.