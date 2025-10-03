# Flutter Navigation and Routing Guide - Part 2

## Table of Contents
1. [Bottom Navigation](#bottom-navigation)
2. [Drawer Navigation](#drawer-navigation)
3. [Tab Navigation](#tab-navigation)
4. [Navigation 2.0 (Router API)](#navigation-20-router-api)
5. [Deep Linking](#deep-linking)

## Bottom Navigation

Bottom navigation provides persistent navigation between top-level views.

### Basic Bottom Navigation
```dart
class BottomNavigationExample extends StatefulWidget {
  @override
  _BottomNavigationExampleState createState() => _BottomNavigationExampleState();
}

class _BottomNavigationExampleState extends State<BottomNavigationExample> {
  int _selectedIndex = 0;
  
  final List<Widget> _pages = [
    HomeTabPage(),
    SearchTabPage(),
    FavoritesTabPage(),
    ProfileTabPage(),
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages[_selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        onTap: (index) => setState(() => _selectedIndex = index),
        type: BottomNavigationBarType.fixed,
        selectedItemColor: Colors.blue,
        unselectedItemColor: Colors.grey,
        backgroundColor: Colors.white,
        elevation: 8,
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            activeIcon: Icon(Icons.home, size: 28),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            activeIcon: Icon(Icons.search, size: 28),
            label: 'Search',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.favorite),
            activeIcon: Icon(Icons.favorite, size: 28),
            label: 'Favorites',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            activeIcon: Icon(Icons.person, size: 28),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}

// Home Tab Page
class HomeTabPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),
        backgroundColor: Colors.blue,
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Welcome back!',
              style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 20),
            
            // Featured Content
            Container(
              height: 200,
              child: ListView.builder(
                scrollDirection: Axis.horizontal,
                itemCount: 5,
                itemBuilder: (context, index) {
                  return Container(
                    width: 300,
                    margin: EdgeInsets.only(right: 16),
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(12),
                      gradient: LinearGradient(
                        colors: [
                          Colors.primaries[index % Colors.primaries.length],
                          Colors.primaries[index % Colors.primaries.length].withOpacity(0.7),
                        ],
                      ),
                    ),
                    child: Padding(
                      padding: EdgeInsets.all(16),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        mainAxisAlignment: MainAxisAlignment.end,
                        children: [
                          Text(
                            'Featured Content ${index + 1}',
                            style: TextStyle(
                              color: Colors.white,
                              fontSize: 20,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                          Text(
                            'Discover amazing content here',
                            style: TextStyle(color: Colors.white70),
                          ),
                        ],
                      ),
                    ),
                  );
                },
              ),
            ),
            
            SizedBox(height: 30),
            
            // Recent Activity
            Text(
              'Recent Activity',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 12),
            
            ...List.generate(5, (index) => Card(
              margin: EdgeInsets.only(bottom: 8),
              child: ListTile(
                leading: CircleAvatar(
                  backgroundColor: Colors.blue,
                  child: Text('${index + 1}'),
                ),
                title: Text('Activity ${index + 1}'),
                subtitle: Text('Description of activity'),
                trailing: Icon(Icons.arrow_forward_ios),
                onTap: () {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Tapped Activity ${index + 1}')),
                  );
                },
              ),
            )),
          ],
        ),
      ),
    );
  }
}

// Search Tab Page
class SearchTabPage extends StatefulWidget {
  @override
  _SearchTabPageState createState() => _SearchTabPageState();
}

class _SearchTabPageState extends State<SearchTabPage> {
  final _searchController = TextEditingController();
  List<String> _searchResults = [];
  bool _isSearching = false;
  
  final List<String> _sampleData = [
    'Flutter Development',
    'Dart Programming',
    'Mobile App Design',
    'State Management',
    'Widget Building',
    'Navigation Patterns',
    'Animation Techniques',
    'Performance Optimization',
    'Testing Strategies',
    'Deployment Guide',
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Search'),
        backgroundColor: Colors.green,
      ),
      body: Column(
        children: [
          Container(
            padding: EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.grey[100],
              border: Border(bottom: BorderSide(color: Colors.grey[300]!)),
            ),
            child: TextField(
              controller: _searchController,
              decoration: InputDecoration(
                hintText: 'Search...',
                prefixIcon: Icon(Icons.search),
                suffixIcon: _searchController.text.isNotEmpty
                    ? IconButton(
                        icon: Icon(Icons.clear),
                        onPressed: () {
                          _searchController.clear();
                          setState(() {
                            _searchResults.clear();
                            _isSearching = false;
                          });
                        },
                      )
                    : null,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(25),
                  borderSide: BorderSide.none,
                ),
                filled: true,
                fillColor: Colors.white,
              ),
              onChanged: _performSearch,
            ),
          ),
          
          Expanded(
            child: _isSearching
                ? _buildSearchResults()
                : _buildSearchSuggestions(),
          ),
        ],
      ),
    );
  }
  
  Widget _buildSearchResults() {
    if (_searchResults.isEmpty) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.search_off, size: 64, color: Colors.grey),
            SizedBox(height: 16),
            Text(
              'No results found',
              style: TextStyle(fontSize: 18, color: Colors.grey),
            ),
          ],
        ),
      );
    }
    
    return ListView.builder(
      itemCount: _searchResults.length,
      itemBuilder: (context, index) {
        return ListTile(
          leading: Icon(Icons.article, color: Colors.green),
          title: Text(_searchResults[index]),
          subtitle: Text('Search result description'),
          onTap: () {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('Selected: ${_searchResults[index]}')),
            );
          },
        );
      },
    );
  }
  
  Widget _buildSearchSuggestions() {
    return ListView(
      children: [
        Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            'Popular Searches',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
        ),
        
        ...List.generate(5, (index) => ListTile(
          leading: Icon(Icons.trending_up, color: Colors.orange),
          title: Text(_sampleData[index]),
          subtitle: Text('Trending topic'),
          onTap: () {
            _searchController.text = _sampleData[index];
            _performSearch(_sampleData[index]);
          },
        )),
        
        SizedBox(height: 20),
        
        Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            'Recent Searches',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
        ),
        
        ...List.generate(3, (index) => ListTile(
          leading: Icon(Icons.history, color: Colors.grey),
          title: Text(_sampleData[index + 5]),
          subtitle: Text('Recent search'),
          trailing: IconButton(
            icon: Icon(Icons.close, color: Colors.grey),
            onPressed: () {
              // Remove from recent searches
            },
          ),
          onTap: () {
            _searchController.text = _sampleData[index + 5];
            _performSearch(_sampleData[index + 5]);
          },
        )),
      ],
    );
  }
  
  void _performSearch(String query) {
    setState(() {
      _isSearching = query.isNotEmpty;
      if (query.isNotEmpty) {
        _searchResults = _sampleData
            .where((item) => item.toLowerCase().contains(query.toLowerCase()))
            .toList();
      } else {
        _searchResults.clear();
      }
    });
  }
  
  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }
}

// Favorites Tab Page
class FavoritesTabPage extends StatefulWidget {
  @override
  _FavoritesTabPageState createState() => _FavoritesTabPageState();
}

class _FavoritesTabPageState extends State<FavoritesTabPage> {
  List<Map<String, dynamic>> _favorites = [
    {
      'title': 'Flutter Basics',
      'subtitle': 'Learn the fundamentals',
      'icon': Icons.school,
      'color': Colors.blue,
    },
    {
      'title': 'Widget Catalog',
      'subtitle': 'Explore all widgets',
      'icon': Icons.widgets,
      'color': Colors.green,
    },
    {
      'title': 'State Management',
      'subtitle': 'Master app state',
      'icon': Icons.settings,
      'color': Colors.orange,
    },
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Favorites'),
        backgroundColor: Colors.red,
        actions: [
          IconButton(
            icon: Icon(Icons.clear_all),
            onPressed: _favorites.isNotEmpty ? _clearAllFavorites : null,
          ),
        ],
      ),
      body: _favorites.isEmpty
          ? _buildEmptyState()
          : ListView.builder(
              itemCount: _favorites.length,
              itemBuilder: (context, index) {
                final item = _favorites[index];
                return Card(
                  margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                  child: ListTile(
                    leading: Container(
                      padding: EdgeInsets.all(8),
                      decoration: BoxDecoration(
                        color: item['color'].withOpacity(0.1),
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: Icon(
                        item['icon'],
                        color: item['color'],
                      ),
                    ),
                    title: Text(item['title']),
                    subtitle: Text(item['subtitle']),
                    trailing: IconButton(
                      icon: Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _removeFavorite(index),
                    ),
                    onTap: () {
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('Opened ${item['title']}')),
                      );
                    },
                  ),
                );
              },
            ),
    );
  }
  
  Widget _buildEmptyState() {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.favorite_border, size: 80, color: Colors.grey),
          SizedBox(height: 20),
          Text(
            'No Favorites Yet',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 10),
          Text(
            'Add items to your favorites to see them here',
            style: TextStyle(color: Colors.grey[600]),
            textAlign: TextAlign.center,
          ),
          SizedBox(height: 30),
          ElevatedButton(
            onPressed: () {
              // Navigate to explore content
            },
            child: Text('Explore Content'),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
          ),
        ],
      ),
    );
  }
  
  void _removeFavorite(int index) {
    setState(() {
      _favorites.removeAt(index);
    });
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Removed from favorites'),
        action: SnackBarAction(
          label: 'Undo',
          onPressed: () {
            // Implement undo functionality
          },
        ),
      ),
    );
  }
  
  void _clearAllFavorites() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Clear All Favorites'),
        content: Text('Are you sure you want to remove all favorites?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () {
              setState(() => _favorites.clear());
              Navigator.pop(context);
            },
            child: Text('Clear All'),
            style: TextButton.styleFrom(foregroundColor: Colors.red),
          ),
        ],
      ),
    );
  }
}

// Profile Tab Page
class ProfileTabPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Profile'),
        backgroundColor: Colors.purple,
        actions: [
          IconButton(
            icon: Icon(Icons.settings),
            onPressed: () {
              Navigator.pushNamed(context, '/settings');
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        child: Column(
          children: [
            // Profile Header
            Container(
              width: double.infinity,
              padding: EdgeInsets.all(20),
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.purple, Colors.purple[300]!],
                ),
              ),
              child: Column(
                children: [
                  CircleAvatar(
                    radius: 50,
                    backgroundImage: NetworkImage('https://picsum.photos/100/100'),
                  ),
                  SizedBox(height: 16),
                  Text(
                    'John Doe',
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                      color: Colors.white,
                    ),
                  ),
                  Text(
                    'Flutter Developer',
                    style: TextStyle(fontSize: 16, color: Colors.white70),
                  ),
                ],
              ),
            ),
            
            // Profile Stats
            Container(
              padding: EdgeInsets.all(20),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _buildStatItem('Posts', '142'),
                  _buildStatItem('Followers', '1.2K'),
                  _buildStatItem('Following', '89'),
                ],
              ),
            ),
            
            // Profile Options
            _buildProfileOption(
              icon: Icons.person,
              title: 'Edit Profile',
              subtitle: 'Update your information',
              onTap: () {},
            ),
            _buildProfileOption(
              icon: Icons.notifications,
              title: 'Notifications',
              subtitle: 'Manage notifications',
              onTap: () {},
            ),
            _buildProfileOption(
              icon: Icons.security,
              title: 'Privacy & Security',
              subtitle: 'Control your privacy',
              onTap: () {},
            ),
            _buildProfileOption(
              icon: Icons.help,
              title: 'Help & Support',
              subtitle: 'Get help and support',
              onTap: () {},
            ),
            _buildProfileOption(
              icon: Icons.info,
              title: 'About',
              subtitle: 'App information',
              onTap: () {},
            ),
            
            SizedBox(height: 20),
            
            Padding(
              padding: EdgeInsets.symmetric(horizontal: 20),
              child: ElevatedButton(
                onPressed: () {
                  // Implement logout
                },
                child: Text('Logout'),
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.red,
                  minimumSize: Size(double.infinity, 50),
                ),
              ),
            ),
            
            SizedBox(height: 20),
          ],
        ),
      ),
    );
  }
  
  Widget _buildStatItem(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
        ),
        Text(
          label,
          style: TextStyle(color: Colors.grey[600]),
        ),
      ],
    );
  }
  
  Widget _buildProfileOption({
    required IconData icon,
    required String title,
    required String subtitle,
    required VoidCallback onTap,
  }) {
    return ListTile(
      leading: Container(
        padding: EdgeInsets.all(8),
        decoration: BoxDecoration(
          color: Colors.purple.withOpacity(0.1),
          borderRadius: BorderRadius.circular(8),
        ),
        child: Icon(icon, color: Colors.purple),
      ),
      title: Text(title),
      subtitle: Text(subtitle),
      trailing: Icon(Icons.arrow_forward_ios, size: 16),
      onTap: onTap,
    );
  }
}
```

## Drawer Navigation

Drawer provides a slide-out navigation menu.

### Custom Drawer Implementation
```dart
class DrawerNavigationExample extends StatefulWidget {
  @override
  _DrawerNavigationExampleState createState() => _DrawerNavigationExampleState();
}

class _DrawerNavigationExampleState extends State<DrawerNavigationExample> {
  String _currentPage = 'Home';
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_currentPage),
        backgroundColor: Colors.indigo,
      ),
      drawer: CustomDrawer(
        currentPage: _currentPage,
        onPageChanged: (page) {
          setState(() {
            _currentPage = page;
          });
          Navigator.pop(context); // Close drawer
        },
      ),
      body: _buildPageContent(),
    );
  }
  
  Widget _buildPageContent() {
    switch (_currentPage) {
      case 'Dashboard':
        return DashboardPage();
      case 'Analytics':
        return AnalyticsPage();
      case 'Reports':
        return ReportsPage();
      case 'Settings':
        return SettingsPage();
      case 'Home':
      default:
        return HomePage();
    }
  }
}

class CustomDrawer extends StatelessWidget {
  final String currentPage;
  final Function(String) onPageChanged;
  
  CustomDrawer({required this.currentPage, required this.onPageChanged});
  
  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: Column(
        children: [
          // Drawer Header
          Container(
            height: 200,
            child: DrawerHeader(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.indigo, Colors.indigo[300]!],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  CircleAvatar(
                    radius: 30,
                    backgroundImage: NetworkImage('https://picsum.photos/60/60'),
                  ),
                  SizedBox(height: 12),
                  Text(
                    'John Doe',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    'john.doe@example.com',
                    style: TextStyle(color: Colors.white70),
                  ),
                ],
              ),
            ),
          ),
          
          // Navigation Items
          Expanded(
            child: ListView(
              padding: EdgeInsets.zero,
              children: [
                _buildDrawerItem(
                  icon: Icons.home,
                  title: 'Home',
                  isSelected: currentPage == 'Home',
                  onTap: () => onPageChanged('Home'),
                ),
                _buildDrawerItem(
                  icon: Icons.dashboard,
                  title: 'Dashboard',
                  isSelected: currentPage == 'Dashboard',
                  onTap: () => onPageChanged('Dashboard'),
                ),
                _buildDrawerItem(
                  icon: Icons.analytics,
                  title: 'Analytics',
                  isSelected: currentPage == 'Analytics',
                  onTap: () => onPageChanged('Analytics'),
                ),
                _buildDrawerItem(
                  icon: Icons.report,
                  title: 'Reports',
                  isSelected: currentPage == 'Reports',
                  onTap: () => onPageChanged('Reports'),
                ),
                
                Divider(),
                
                _buildDrawerItem(
                  icon: Icons.settings,
                  title: 'Settings',
                  isSelected: currentPage == 'Settings',
                  onTap: () => onPageChanged('Settings'),
                ),
                _buildDrawerItem(
                  icon: Icons.help,
                  title: 'Help & Support',
                  onTap: () {
                    Navigator.pop(context);
                    _showHelpDialog(context);
                  },
                ),
                _buildDrawerItem(
                  icon: Icons.info,
                  title: 'About',
                  onTap: () {
                    Navigator.pop(context);
                    _showAboutDialog(context);
                  },
                ),
              ],
            ),
          ),
          
          // Footer
          Container(
            padding: EdgeInsets.all(16),
            decoration: BoxDecoration(
              border: Border(top: BorderSide(color: Colors.grey[300]!)),
            ),
            child: Row(
              children: [
                Icon(Icons.logout, color: Colors.red),
                SizedBox(width: 12),
                Text(
                  'Logout',
                  style: TextStyle(color: Colors.red, fontSize: 16),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildDrawerItem({
    required IconData icon,
    required String title,
    bool isSelected = false,
    required VoidCallback onTap,
  }) {
    return Container(
      margin: EdgeInsets.symmetric(horizontal: 8, vertical: 2),
      child: ListTile(
        leading: Icon(
          icon,
          color: isSelected ? Colors.indigo : Colors.grey[600],
        ),
        title: Text(
          title,
          style: TextStyle(
            color: isSelected ? Colors.indigo : Colors.grey[800],
            fontWeight: isSelected ? FontWeight.bold : FontWeight.normal,
          ),
        ),
        selected: isSelected,
        selectedTileColor: Colors.indigo.withOpacity(0.1),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        onTap: onTap,
      ),
    );
  }
  
  void _showHelpDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Help & Support'),
        content: Text('Contact us at support@example.com for assistance.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('OK'),
          ),
        ],
      ),
    );
  }
  
  void _showAboutDialog(BuildContext context) {
    showAboutDialog(
      context: context,
      applicationName: 'Flutter App',
      applicationVersion: '1.0.0',
      applicationIcon: Icon(Icons.flutter_dash),
      children: [
        Text('This is a sample Flutter application demonstrating navigation patterns.'),
      ],
    );
  }
}

// Sample pages for drawer navigation
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.home, size: 80, color: Colors.indigo),
          SizedBox(height: 20),
          Text(
            'Welcome to Home Page',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 10),
          Text('Open the drawer to navigate to other pages'),
        ],
      ),
    );
  }
}

class DashboardPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(16),
      child: GridView.count(
        crossAxisCount: 2,
        crossAxisSpacing: 16,
        mainAxisSpacing: 16,
        children: [
          _buildDashboardCard('Total Users', '1,234', Icons.people, Colors.blue),
          _buildDashboardCard('Revenue', '\$12,345', Icons.attach_money, Colors.green),
          _buildDashboardCard('Orders', '567', Icons.shopping_cart, Colors.orange),
          _buildDashboardCard('Growth', '+15%', Icons.trending_up, Colors.purple),
        ],
      ),
    );
  }
  
  Widget _buildDashboardCard(String title, String value, IconData icon, Color color) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(icon, size: 40, color: color),
            SizedBox(height: 12),
            Text(
              value,
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            Text(title, style: TextStyle(color: Colors.grey[600])),
          ],
        ),
      ),
    );
  }
}

class AnalyticsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.analytics, size: 80, color: Colors.blue),
          SizedBox(height: 20),
          Text(
            'Analytics Dashboard',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 10),
          Text('View your app analytics here'),
        ],
      ),
    );
  }
}

class ReportsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.report, size: 80, color: Colors.orange),
          SizedBox(height: 20),
          Text(
            'Reports Center',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 10),
          Text('Generate and view reports'),
        ],
      ),
    );
  }
}
```

## Tab Navigation

Tab navigation organizes content into different categories.

### Custom Tab Navigation
```dart
class TabNavigationExample extends StatefulWidget {
  @override
  _TabNavigationExampleState createState() => _TabNavigationExampleState();
}

class _TabNavigationExampleState extends State<TabNavigationExample>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;
  
  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 4, vsync: this);
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Tab Navigation'),
        backgroundColor: Colors.teal,
        bottom: TabBar(
          controller: _tabController,
          indicatorColor: Colors.white,
          indicatorWeight: 3,
          tabs: [
            Tab(icon: Icon(Icons.photo), text: 'Photos'),
            Tab(icon: Icon(Icons.video_library), text: 'Videos'),
            Tab(icon: Icon(Icons.music_note), text: 'Music'),
            Tab(icon: Icon(Icons.article), text: 'Documents'),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: [
          PhotosTab(),
          VideosTab(),
          MusicTab(),
          DocumentsTab(),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }
}

// Photos Tab
class PhotosTab extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      padding: EdgeInsets.all(8),
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
      ),
      itemCount: 20,
      itemBuilder: (context, index) {
        return Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(12),
            image: DecorationImage(
              image: NetworkImage('https://picsum.photos/seed/photo$index/200/200'),
              fit: BoxFit.cover,
            ),
          ),
          child: Container(
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(12),
              gradient: LinearGradient(
                begin: Alignment.topCenter,
                end: Alignment.bottomCenter,
                colors: [Colors.transparent, Colors.black.withOpacity(0.3)],
              ),
            ),
            child: Align(
              alignment: Alignment.bottomLeft,
              child: Padding(
                padding: EdgeInsets.all(8),
                child: Text(
                  'Photo ${index + 1}',
                  style: TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ),
          ),
        );
      },
    );
  }
}

// Videos Tab
class VideosTab extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: EdgeInsets.all(16),
      itemCount: 10,
      itemBuilder: (context, index) {
        return Card(
          margin: EdgeInsets.only(bottom: 16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Container(
                height: 200,
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.vertical(top: Radius.circular(12)),
                  image: DecorationImage(
                    image: NetworkImage('https://picsum.photos/seed/video$index/400/200'),
                    fit: BoxFit.cover,
                  ),
                ),
                child: Center(
                  child: Icon(
                    Icons.play_circle_filled,
                    size: 64,
                    color: Colors.white,
                  ),
                ),
              ),
              Padding(
                padding: EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Video Title ${index + 1}',
                      style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                    ),
                    SizedBox(height: 4),
                    Text(
                      'Duration: ${(index + 1) * 2}:${(index * 15) % 60}',
                      style: TextStyle(color: Colors.grey[600]),
                    ),
                    SizedBox(height: 8),
                    Row(
                      children: [
                        Icon(Icons.thumb_up, size: 16, color: Colors.blue),
                        SizedBox(width: 4),
                        Text('${(index + 1) * 10}'),
                        SizedBox(width: 16),
                        Icon(Icons.visibility, size: 16, color: Colors.grey),
                        SizedBox(width: 4),
                        Text('${(index + 1) * 100} views'),
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
}

// Music Tab
class MusicTab extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Now Playing
        Container(
          padding: EdgeInsets.all(16),
          decoration: BoxDecoration(
            gradient: LinearGradient(
              colors: [Colors.purple, Colors.purple[300]!],
            ),
          ),
          child: Row(
            children: [
              Container(
                width: 60,
                height: 60,
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(8),
                  image: DecorationImage(
                    image: NetworkImage('https://picsum.photos/60/60'),
                    fit: BoxFit.cover,
                  ),
                ),
              ),
              SizedBox(width: 12),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Currently Playing',
                      style: TextStyle(color: Colors.white70, fontSize: 12),
                    ),
                    Text(
                      'Song Title',
                      style: TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.bold),
                    ),
                    Text(
                      'Artist Name',
                      style: TextStyle(color: Colors.white70, fontSize: 14),
                    ),
                  ],
                ),
              ),
              IconButton(
                icon: Icon(Icons.play_arrow, color: Colors.white),
                onPressed: () {},
              ),
            ],
          ),
        ),
        
        // Playlist
        Expanded(
          child: ListView.builder(
            itemCount: 15,
            itemBuilder: (context, index) {
              return ListTile(
                leading: Container(
                  width: 50,
                  height: 50,
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(8),
                    image: DecorationImage(
                      image: NetworkImage('https://picsum.photos/seed/music$index/50/50'),
                      fit: BoxFit.cover,
                    ),
                  ),
                ),
                title: Text('Song ${index + 1}'),
                subtitle: Text('Artist ${index + 1}'),
                trailing: IconButton(
                  icon: Icon(Icons.more_vert),
                  onPressed: () {},
                ),
                onTap: () {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Playing Song ${index + 1}')),
                  );
                },
              );
            },
          ),
        ),
      ],
    );
  }
}

// Documents Tab
class DocumentsTab extends StatelessWidget {
  final List<Map<String, dynamic>> documents = [
    {'name': 'Resume.pdf', 'size': '245 KB', 'type': 'PDF', 'icon': Icons.picture_as_pdf, 'color': Colors.red},
    {'name': 'Presentation.pptx', 'size': '1.2 MB', 'type': 'PowerPoint', 'icon': Icons.slideshow, 'color': Colors.orange},
    {'name': 'Spreadsheet.xlsx', 'size': '567 KB', 'type': 'Excel', 'icon': Icons.grid_on, 'color': Colors.green},
    {'name': 'Document.docx', 'size': '89 KB', 'type': 'Word', 'icon': Icons.description, 'color': Colors.blue},
  ];
  
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: EdgeInsets.all(16),
      itemCount: documents.length * 3, // Repeat for demo
      itemBuilder: (context, index) {
        final doc = documents[index % documents.length];
        return Card(
          margin: EdgeInsets.only(bottom: 8),
          child: ListTile(
            leading: Container(
              padding: EdgeInsets.all(8),
              decoration: BoxDecoration(
                color: doc['color'].withOpacity(0.1),
                borderRadius: BorderRadius.circular(8),
              ),
              child: Icon(doc['icon'], color: doc['color']),
            ),
            title: Text(doc['name']),
            subtitle: Text('${doc['size']} • ${doc['type']}'),
            trailing: PopupMenuButton(
              itemBuilder: (context) => [
                PopupMenuItem(
                  child: ListTile(
                    leading: Icon(Icons.open_in_new),
                    title: Text('Open'),
                    dense: true,
                  ),
                ),
                PopupMenuItem(
                  child: ListTile(
                    leading: Icon(Icons.share),
                    title: Text('Share'),
                    dense: true,
                  ),
                ),
                PopupMenuItem(
                  child: ListTile(
                    leading: Icon(Icons.delete, color: Colors.red),
                    title: Text('Delete'),
                    dense: true,
                  ),
                ),
              ],
            ),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('Opening ${doc['name']}')),
              );
            },
          ),
        );
      },
    );
  }
}
```

This part of the navigation guide covers Bottom Navigation, Drawer Navigation, and Tab Navigation with comprehensive examples. The remaining sections (Navigation 2.0 and Deep Linking) would be covered in the next part to keep file sizes manageable.