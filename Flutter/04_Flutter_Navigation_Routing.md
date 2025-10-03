# Flutter Navigation and Routing Guide

## Table of Contents
1. [Basic Navigation](#basic-navigation)
2. [Named Routes](#named-routes)
3. [Navigation 2.0 (Router API)](#navigation-20-router-api)
4. [Page Transitions](#page-transitions)
5. [Bottom Navigation](#bottom-navigation)
6. [Drawer Navigation](#drawer-navigation)
7. [Tab Navigation](#tab-navigation)
8. [Deep Linking](#deep-linking)

## Basic Navigation

Flutter's navigation system is based on a stack-like structure where you push and pop routes.

### Simple Navigation Example
```dart
// First Screen
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home Screen'),
        backgroundColor: Colors.blue,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Welcome to the Home Screen!',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 30),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => SecondScreen(message: 'Hello from Home!'),
                  ),
                );
              },
              child: Text('Go to Second Screen'),
              style: ElevatedButton.styleFrom(
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
            ),
            SizedBox(height: 15),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => ProfileScreen(),
                  ),
                );
              },
              child: Text('View Profile'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.green,
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// Second Screen with data passing
class SecondScreen extends StatelessWidget {
  final String message;
  
  SecondScreen({required this.message});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Second Screen'),
        backgroundColor: Colors.green,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.message,
              size: 64,
              color: Colors.green,
            ),
            SizedBox(height: 20),
            Text(
              'Message from previous screen:',
              style: TextStyle(fontSize: 16),
            ),
            SizedBox(height: 10),
            Container(
              padding: EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.green[50],
                borderRadius: BorderRadius.circular(8),
                border: Border.all(color: Colors.green),
              ),
              child: Text(
                message,
                style: TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                  color: Colors.green[800],
                ),
              ),
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: () {
                    Navigator.pop(context);
                  },
                  child: Text('Go Back'),
                  style: ElevatedButton.styleFrom(backgroundColor: Colors.orange),
                ),
                ElevatedButton(
                  onPressed: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => ThirdScreen(),
                      ),
                    );
                  },
                  child: Text('Next Screen'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

// Third Screen with return data
class ThirdScreen extends StatefulWidget {
  @override
  _ThirdScreenState createState() => _ThirdScreenState();
}

class _ThirdScreenState extends State<ThirdScreen> {
  final _textController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Third Screen'),
        backgroundColor: Colors.purple,
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'Enter some data to return:',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 20),
            TextField(
              controller: _textController,
              decoration: InputDecoration(
                labelText: 'Your message',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.edit),
              ),
            ),
            SizedBox(height: 30),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: () {
                    Navigator.pop(context);
                  },
                  child: Text('Cancel'),
                  style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
                ),
                ElevatedButton(
                  onPressed: () {
                    Navigator.pop(context, _textController.text);
                  },
                  child: Text('Return Data'),
                  style: ElevatedButton.styleFrom(backgroundColor: Colors.green),
                ),
                ElevatedButton(
                  onPressed: () {
                    Navigator.popUntil(context, (route) => route.isFirst);
                  },
                  child: Text('Back to Home'),
                  style: ElevatedButton.styleFrom(backgroundColor: Colors.blue),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _textController.dispose();
    super.dispose();
  }
}

// Profile Screen
class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Profile'),
        backgroundColor: Colors.indigo,
        actions: [
          IconButton(
            icon: Icon(Icons.edit),
            onPressed: () async {
              final result = await Navigator.push<String>(
                context,
                MaterialPageRoute(
                  builder: (context) => EditProfileScreen(),
                ),
              );
              
              if (result != null) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('Profile updated: $result'),
                    backgroundColor: Colors.green,
                  ),
                );
              }
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 60,
              backgroundImage: NetworkImage('https://picsum.photos/120/120'),
            ),
            SizedBox(height: 20),
            Text(
              'John Doe',
              style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
            ),
            Text(
              'Flutter Developer',
              style: TextStyle(fontSize: 16, color: Colors.grey[600]),
            ),
            SizedBox(height: 30),
            Container(
              padding: EdgeInsets.all(16),
              margin: EdgeInsets.symmetric(horizontal: 32),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(12),
              ),
              child: Column(
                children: [
                  _buildProfileItem(Icons.email, 'john.doe@example.com'),
                  SizedBox(height: 12),
                  _buildProfileItem(Icons.phone, '+1 234 567 8900'),
                  SizedBox(height: 12),
                  _buildProfileItem(Icons.location_on, 'San Francisco, CA'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildProfileItem(IconData icon, String text) {
    return Row(
      children: [
        Icon(icon, color: Colors.indigo),
        SizedBox(width: 12),
        Text(text, style: TextStyle(fontSize: 16)),
      ],
    );
  }
}

// Edit Profile Screen
class EditProfileScreen extends StatefulWidget {
  @override
  _EditProfileScreenState createState() => _EditProfileScreenState();
}

class _EditProfileScreenState extends State<EditProfileScreen> {
  final _nameController = TextEditingController(text: 'John Doe');
  final _emailController = TextEditingController(text: 'john.doe@example.com');
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Edit Profile'),
        backgroundColor: Colors.indigo,
        actions: [
          TextButton(
            onPressed: () {
              Navigator.pop(context, _nameController.text);
            },
            child: Text(
              'Save',
              style: TextStyle(color: Colors.white, fontSize: 16),
            ),
          ),
        ],
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _nameController,
              decoration: InputDecoration(
                labelText: 'Name',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.person),
              ),
            ),
            SizedBox(height: 16),
            TextField(
              controller: _emailController,
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.email),
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

## Named Routes

Named routes provide a cleaner way to navigate using string identifiers.

### Named Routes Setup
```dart
class NamedRoutesApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Named Routes Demo',
      initialRoute: '/',
      routes: {
        '/': (context) => HomeScreen(),
        '/second': (context) => SecondScreen(message: 'Default message'),
        '/third': (context) => ThirdScreen(),
        '/profile': (context) => ProfileScreen(),
        '/settings': (context) => SettingsScreen(),
        '/login': (context) => LoginScreen(),
      },
      // Handle unknown routes
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => NotFoundScreen(routeName: settings.name ?? 'Unknown'),
        );
      },
      // Generate routes dynamically
      onGenerateRoute: (settings) {
        if (settings.name == '/user') {
          final args = settings.arguments as Map<String, dynamic>?;
          return MaterialPageRoute(
            builder: (context) => UserScreen(
              userId: args?['userId'] ?? 'unknown',
              userName: args?['userName'] ?? 'Unknown User',
            ),
          );
        }
        return null;
      },
    );
  }
}

// Enhanced Home Screen with named navigation
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Named Routes Home'),
        backgroundColor: Colors.blue,
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Text(
              'Choose a destination:',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 30),
            
            _buildNavigationButton(
              context,
              'Go to Second Screen',
              '/second',
              Icons.arrow_forward,
              Colors.green,
            ),
            
            _buildNavigationButton(
              context,
              'View Profile',
              '/profile',
              Icons.person,
              Colors.blue,
            ),
            
            _buildNavigationButton(
              context,
              'Settings',
              '/settings',
              Icons.settings,
              Colors.orange,
            ),
            
            SizedBox(height: 20),
            
            ElevatedButton.icon(
              onPressed: () {
                Navigator.pushNamed(
                  context,
                  '/user',
                  arguments: {
                    'userId': '123',
                    'userName': 'Alice Johnson',
                  },
                );
              },
              icon: Icon(Icons.person_search),
              label: Text('View User (with arguments)'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.purple,
                padding: EdgeInsets.symmetric(vertical: 15),
              ),
            ),
            
            SizedBox(height: 10),
            
            ElevatedButton.icon(
              onPressed: () {
                Navigator.pushNamed(context, '/nonexistent');
              },
              icon: Icon(Icons.error),
              label: Text('Test Unknown Route'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red,
                padding: EdgeInsets.symmetric(vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildNavigationButton(
    BuildContext context,
    String text,
    String route,
    IconData icon,
    Color color,
  ) {
    return Container(
      margin: EdgeInsets.only(bottom: 12),
      child: ElevatedButton.icon(
        onPressed: () => Navigator.pushNamed(context, route),
        icon: Icon(icon),
        label: Text(text),
        style: ElevatedButton.styleFrom(
          backgroundColor: color,
          padding: EdgeInsets.symmetric(vertical: 15),
        ),
      ),
    );
  }
}

// Settings Screen
class SettingsScreen extends StatefulWidget {
  @override
  _SettingsScreenState createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  bool _darkMode = false;
  bool _notifications = true;
  double _volume = 0.5;
  String _language = 'English';
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Settings'),
        backgroundColor: Colors.orange,
      ),
      body: ListView(
        children: [
          _buildSectionHeader('Appearance'),
          SwitchListTile(
            title: Text('Dark Mode'),
            subtitle: Text('Enable dark theme'),
            value: _darkMode,
            onChanged: (value) => setState(() => _darkMode = value),
            secondary: Icon(Icons.dark_mode),
          ),
          
          _buildSectionHeader('Notifications'),
          SwitchListTile(
            title: Text('Push Notifications'),
            subtitle: Text('Receive notifications'),
            value: _notifications,
            onChanged: (value) => setState(() => _notifications = value),
            secondary: Icon(Icons.notifications),
          ),
          
          _buildSectionHeader('Audio'),
          ListTile(
            title: Text('Volume'),
            subtitle: Slider(
              value: _volume,
              onChanged: (value) => setState(() => _volume = value),
              divisions: 10,
              label: '${(_volume * 100).round()}%',
            ),
            leading: Icon(Icons.volume_up),
          ),
          
          _buildSectionHeader('Language'),
          ListTile(
            title: Text('Language'),
            subtitle: Text(_language),
            leading: Icon(Icons.language),
            trailing: Icon(Icons.arrow_forward_ios),
            onTap: () => _showLanguageDialog(),
          ),
          
          SizedBox(height: 20),
          
          Padding(
            padding: EdgeInsets.symmetric(horizontal: 16),
            child: ElevatedButton(
              onPressed: () {
                Navigator.pushNamedAndRemoveUntil(
                  context,
                  '/login',
                  (route) => false,
                );
              },
              child: Text('Logout'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red,
                padding: EdgeInsets.symmetric(vertical: 15),
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildSectionHeader(String title) {
    return Padding(
      padding: EdgeInsets.fromLTRB(16, 20, 16, 8),
      child: Text(
        title,
        style: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.bold,
          color: Colors.orange[800],
        ),
      ),
    );
  }
  
  void _showLanguageDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Select Language'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            _buildLanguageOption('English'),
            _buildLanguageOption('Spanish'),
            _buildLanguageOption('French'),
            _buildLanguageOption('German'),
          ],
        ),
      ),
    );
  }
  
  Widget _buildLanguageOption(String language) {
    return RadioListTile<String>(
      title: Text(language),
      value: language,
      groupValue: _language,
      onChanged: (value) {
        setState(() => _language = value!);
        Navigator.pop(context);
      },
    );
  }
}

// User Screen with arguments
class UserScreen extends StatelessWidget {
  final String userId;
  final String userName;
  
  UserScreen({required this.userId, required this.userName});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('User Details'),
        backgroundColor: Colors.purple,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 50,
              backgroundImage: NetworkImage('https://picsum.photos/seed/$userId/100/100'),
            ),
            SizedBox(height: 20),
            Text(
              userName,
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            Text(
              'User ID: $userId',
              style: TextStyle(fontSize: 16, color: Colors.grey[600]),
            ),
            SizedBox(height: 30),
            Container(
              padding: EdgeInsets.all(16),
              margin: EdgeInsets.symmetric(horizontal: 32),
              decoration: BoxDecoration(
                color: Colors.purple[50],
                borderRadius: BorderRadius.circular(12),
                border: Border.all(color: Colors.purple),
              ),
              child: Column(
                children: [
                  Text(
                    'This user screen was navigated to using named routes with arguments.',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 16),
                  ),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () => Navigator.pop(context),
                    child: Text('Go Back'),
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

// Login Screen
class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Login'),
        backgroundColor: Colors.indigo,
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.lock_outline,
              size: 80,
              color: Colors.indigo,
            ),
            SizedBox(height: 30),
            TextField(
              controller: _emailController,
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.email),
              ),
            ),
            SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              obscureText: true,
              decoration: InputDecoration(
                labelText: 'Password',
                border: OutlineInputBorder(),
                prefixIcon: Icon(Icons.lock),
              ),
            ),
            SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _isLoading ? null : _login,
                child: _isLoading
                    ? CircularProgressIndicator(color: Colors.white)
                    : Text('Login'),
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.indigo,
                  padding: EdgeInsets.symmetric(vertical: 15),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  void _login() async {
    setState(() => _isLoading = true);
    
    // Simulate login process
    await Future.delayed(Duration(seconds: 2));
    
    setState(() => _isLoading = false);
    
    Navigator.pushNamedAndRemoveUntil(
      context,
      '/',
      (route) => false,
    );
  }
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}

// 404 Not Found Screen
class NotFoundScreen extends StatelessWidget {
  final String routeName;
  
  NotFoundScreen({required this.routeName});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Page Not Found'),
        backgroundColor: Colors.red,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.error_outline,
              size: 100,
              color: Colors.red,
            ),
            SizedBox(height: 20),
            Text(
              '404',
              style: TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
            ),
            Text(
              'Page Not Found',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 10),
            Text(
              'Route "$routeName" does not exist',
              style: TextStyle(fontSize: 16, color: Colors.grey[600]),
            ),
            SizedBox(height: 30),
            ElevatedButton(
              onPressed: () => Navigator.pushNamedAndRemoveUntil(
                context,
                '/',
                (route) => false,
              ),
              child: Text('Go Home'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.blue,
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

## Page Transitions

Custom animations for navigation transitions.

### Custom Page Transitions
```dart
// Custom Page Route with animations
class CustomPageRoute<T> extends PageRouteBuilder<T> {
  final Widget child;
  final String transitionType;
  
  CustomPageRoute({
    required this.child,
    this.transitionType = 'slide',
  }) : super(
          pageBuilder: (context, animation, secondaryAnimation) => child,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            switch (transitionType) {
              case 'fade':
                return FadeTransition(opacity: animation, child: child);
              case 'scale':
                return ScaleTransition(scale: animation, child: child);
              case 'rotation':
                return RotationTransition(turns: animation, child: child);
              case 'slide':
              default:
                return SlideTransition(
                  position: animation.drive(
                    Tween(begin: Offset(1.0, 0.0), end: Offset.zero)
                        .chain(CurveTween(curve: Curves.easeInOut)),
                  ),
                  child: child,
                );
            }
          },
          transitionDuration: Duration(milliseconds: 500),
        );
}

// Page Transitions Demo
class PageTransitionsDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Page Transitions'),
        backgroundColor: Colors.deepPurple,
      ),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Text(
              'Choose a transition type:',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 30),
            
            _buildTransitionButton(
              context,
              'Slide Transition',
              'slide',
              Icons.swipe,
              Colors.blue,
            ),
            
            _buildTransitionButton(
              context,
              'Fade Transition',
              'fade',
              Icons.opacity,
              Colors.green,
            ),
            
            _buildTransitionButton(
              context,
              'Scale Transition',
              'scale',
              Icons.zoom_in,
              Colors.orange,
            ),
            
            _buildTransitionButton(
              context,
              'Rotation Transition',
              'rotation',
              Icons.rotate_right,
              Colors.red,
            ),
            
            SizedBox(height: 20),
            
            ElevatedButton.icon(
              onPressed: () {
                Navigator.push(
                  context,
                  _createCustomTransition(),
                );
              },
              icon: Icon(Icons.auto_awesome),
              label: Text('Custom Combined Transition'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.purple,
                padding: EdgeInsets.symmetric(vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildTransitionButton(
    BuildContext context,
    String text,
    String transitionType,
    IconData icon,
    Color color,
  ) {
    return Container(
      margin: EdgeInsets.only(bottom: 12),
      child: ElevatedButton.icon(
        onPressed: () {
          Navigator.push(
            context,
            CustomPageRoute(
              child: TransitionDemoScreen(transitionType: transitionType),
              transitionType: transitionType,
            ),
          );
        },
        icon: Icon(icon),
        label: Text(text),
        style: ElevatedButton.styleFrom(
          backgroundColor: color,
          padding: EdgeInsets.symmetric(vertical: 15),
        ),
      ),
    );
  }
  
  PageRouteBuilder _createCustomTransition() {
    return PageRouteBuilder(
      pageBuilder: (context, animation, secondaryAnimation) => 
          TransitionDemoScreen(transitionType: 'custom'),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        const begin = Offset(0.0, 1.0);
        const end = Offset.zero;
        const curve = Curves.elasticOut;
        
        var slideTween = Tween(begin: begin, end: end)
            .chain(CurveTween(curve: curve));
        var fadeAnimation = Tween(begin: 0.0, end: 1.0)
            .animate(animation);
        
        return SlideTransition(
          position: animation.drive(slideTween),
          child: FadeTransition(
            opacity: fadeAnimation,
            child: child,
          ),
        );
      },
      transitionDuration: Duration(milliseconds: 800),
    );
  }
}

// Destination screen for transitions
class TransitionDemoScreen extends StatelessWidget {
  final String transitionType;
  
  TransitionDemoScreen({required this.transitionType});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('${transitionType.toUpperCase()} Transition'),
        backgroundColor: _getColorForTransition(transitionType),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(
              width: 200,
              height: 200,
              decoration: BoxDecoration(
                color: _getColorForTransition(transitionType).withOpacity(0.2),
                borderRadius: BorderRadius.circular(100),
                border: Border.all(
                  color: _getColorForTransition(transitionType),
                  width: 3,
                ),
              ),
              child: Icon(
                _getIconForTransition(transitionType),
                size: 100,
                color: _getColorForTransition(transitionType),
              ),
            ),
            SizedBox(height: 30),
            Text(
              '${transitionType.toUpperCase()} Transition Demo',
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
                color: _getColorForTransition(transitionType),
              ),
            ),
            SizedBox(height: 20),
            Container(
              padding: EdgeInsets.all(16),
              margin: EdgeInsets.symmetric(horizontal: 32),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(12),
              ),
              child: Text(
                'This screen was navigated to using a $transitionType transition animation.',
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 16),
              ),
            ),
            SizedBox(height: 30),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: Text('Go Back'),
              style: ElevatedButton.styleFrom(
                backgroundColor: _getColorForTransition(transitionType),
                padding: EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Color _getColorForTransition(String type) {
    switch (type) {
      case 'slide':
        return Colors.blue;
      case 'fade':
        return Colors.green;
      case 'scale':
        return Colors.orange;
      case 'rotation':
        return Colors.red;
      case 'custom':
        return Colors.purple;
      default:
        return Colors.grey;
    }
  }
  
  IconData _getIconForTransition(String type) {
    switch (type) {
      case 'slide':
        return Icons.swipe;
      case 'fade':
        return Icons.opacity;
      case 'scale':
        return Icons.zoom_in;
      case 'rotation':
        return Icons.rotate_right;
      case 'custom':
        return Icons.auto_awesome;
      default:
        return Icons.help;
    }
  }
}
```

This navigation guide covers the fundamental concepts and provides practical examples. The remaining sections (Bottom Navigation, Drawer Navigation, Tab Navigation, Deep Linking, and Navigation 2.0) would be covered in a separate file to manage length.