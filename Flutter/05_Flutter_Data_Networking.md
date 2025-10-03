# Flutter Data and Networking Guide

## Table of Contents
1. [HTTP Requests](#http-requests)
2. [JSON Parsing](#json-parsing)
3. [REST API Integration](#rest-api-integration)
4. [Local Storage](#local-storage)
5. [Database Integration](#database-integration)
6. [File Handling](#file-handling)
7. [Caching Strategies](#caching-strategies)

## HTTP Requests

Flutter uses the `http` package for making network requests.

### Setup
```yaml
dependencies:
  http: ^1.1.0
  shared_preferences: ^2.2.0
  sqflite: ^2.3.0
  path: ^1.8.3
```

### Basic HTTP Operations
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';
  
  // GET Request
  static Future<List<Post>> getPosts() async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/posts'),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      );
      
      if (response.statusCode == 200) {
        final List<dynamic> jsonData = json.decode(response.body);
        return jsonData.map((json) => Post.fromJson(json)).toList();
      } else {
        throw Exception('Failed to load posts: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  // POST Request
  static Future<Post> createPost(Post post) async {
    try {
      final response = await http.post(
        Uri.parse('$baseUrl/posts'),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: json.encode(post.toJson()),
      );
      
      if (response.statusCode == 201) {
        return Post.fromJson(json.decode(response.body));
      } else {
        throw Exception('Failed to create post: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  // PUT Request
  static Future<Post> updatePost(int id, Post post) async {
    try {
      final response = await http.put(
        Uri.parse('$baseUrl/posts/$id'),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
        body: json.encode(post.toJson()),
      );
      
      if (response.statusCode == 200) {
        return Post.fromJson(json.decode(response.body));
      } else {
        throw Exception('Failed to update post: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  // DELETE Request
  static Future<bool> deletePost(int id) async {
    try {
      final response = await http.delete(
        Uri.parse('$baseUrl/posts/$id'),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      );
      
      return response.statusCode == 200;
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  // GET with Query Parameters
  static Future<List<Post>> getPostsByUser(int userId) async {
    try {
      final response = await http.get(
        Uri.parse('$baseUrl/posts').replace(
          queryParameters: {'userId': userId.toString()},
        ),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      );
      
      if (response.statusCode == 200) {
        final List<dynamic> jsonData = json.decode(response.body);
        return jsonData.map((json) => Post.fromJson(json)).toList();
      } else {
        throw Exception('Failed to load user posts: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Network error: $e');
    }
  }
  
  // POST with File Upload (multipart)
  static Future<String> uploadImage(String filePath) async {
    try {
      var request = http.MultipartRequest(
        'POST',
        Uri.parse('$baseUrl/upload'),
      );
      
      request.headers.addAll({
        'Content-Type': 'multipart/form-data',
        'Accept': 'application/json',
      });
      
      request.files.add(
        await http.MultipartFile.fromPath('image', filePath),
      );
      
      request.fields['description'] = 'Uploaded image';
      
      final response = await request.send();
      
      if (response.statusCode == 200) {
        final responseData = await response.stream.bytesToString();
        final jsonData = json.decode(responseData);
        return jsonData['url'];
      } else {
        throw Exception('Failed to upload image: ${response.statusCode}');
      }
    } catch (e) {
      throw Exception('Upload error: $e');
    }
  }
}
```

### HTTP Client with Interceptors
```dart
class HttpClient {
  static final http.Client _client = http.Client();
  static String? _authToken;
  
  static void setAuthToken(String token) {
    _authToken = token;
  }
  
  static Map<String, String> _getHeaders() {
    final headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    };
    
    if (_authToken != null) {
      headers['Authorization'] = 'Bearer $_authToken';
    }
    
    return headers;
  }
  
  static Future<http.Response> get(String url) async {
    print('GET: $url');
    final response = await _client.get(
      Uri.parse(url),
      headers: _getHeaders(),
    );
    _logResponse(response);
    return response;
  }
  
  static Future<http.Response> post(String url, {Map<String, dynamic>? body}) async {
    print('POST: $url');
    print('Body: ${json.encode(body)}');
    
    final response = await _client.post(
      Uri.parse(url),
      headers: _getHeaders(),
      body: body != null ? json.encode(body) : null,
    );
    _logResponse(response);
    return response;
  }
  
  static Future<http.Response> put(String url, {Map<String, dynamic>? body}) async {
    print('PUT: $url');
    print('Body: ${json.encode(body)}');
    
    final response = await _client.put(
      Uri.parse(url),
      headers: _getHeaders(),
      body: body != null ? json.encode(body) : null,
    );
    _logResponse(response);
    return response;
  }
  
  static Future<http.Response> delete(String url) async {
    print('DELETE: $url');
    final response = await _client.delete(
      Uri.parse(url),
      headers: _getHeaders(),
    );
    _logResponse(response);
    return response;
  }
  
  static void _logResponse(http.Response response) {
    print('Response Status: ${response.statusCode}');
    print('Response Body: ${response.body.length > 200 ? '${response.body.substring(0, 200)}...' : response.body}');
  }
  
  static void dispose() {
    _client.close();
  }
}
```

## JSON Parsing

### Model Classes with JSON Serialization
```dart
class Post {
  final int? id;
  final int userId;
  final String title;
  final String body;
  final DateTime? createdAt;
  final List<String>? tags;
  final User? author;
  
  Post({
    this.id,
    required this.userId,
    required this.title,
    required this.body,
    this.createdAt,
    this.tags,
    this.author,
  });
  
  // From JSON
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int?,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
      createdAt: json['createdAt'] != null 
          ? DateTime.parse(json['createdAt']) 
          : null,
      tags: json['tags'] != null 
          ? List<String>.from(json['tags']) 
          : null,
      author: json['author'] != null 
          ? User.fromJson(json['author']) 
          : null,
    );
  }
  
  // To JSON
  Map<String, dynamic> toJson() {
    return {
      if (id != null) 'id': id,
      'userId': userId,
      'title': title,
      'body': body,
      if (createdAt != null) 'createdAt': createdAt!.toIso8601String(),
      if (tags != null) 'tags': tags,
      if (author != null) 'author': author!.toJson(),
    };
  }
  
  // Copy with
  Post copyWith({
    int? id,
    int? userId,
    String? title,
    String? body,
    DateTime? createdAt,
    List<String>? tags,
    User? author,
  }) {
    return Post(
      id: id ?? this.id,
      userId: userId ?? this.userId,
      title: title ?? this.title,
      body: body ?? this.body,
      createdAt: createdAt ?? this.createdAt,
      tags: tags ?? this.tags,
      author: author ?? this.author,
    );
  }
  
  @override
  String toString() {
    return 'Post{id: $id, userId: $userId, title: $title, body: $body}';
  }
}

class User {
  final int id;
  final String name;
  final String email;
  final String? phone;
  final Address? address;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    this.phone,
    this.address,
  });
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      phone: json['phone'] as String?,
      address: json['address'] != null 
          ? Address.fromJson(json['address']) 
          : null,
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      if (phone != null) 'phone': phone,
      if (address != null) 'address': address!.toJson(),
    };
  }
}

class Address {
  final String street;
  final String city;
  final String zipcode;
  
  Address({
    required this.street,
    required this.city,
    required this.zipcode,
  });
  
  factory Address.fromJson(Map<String, dynamic> json) {
    return Address(
      street: json['street'] as String,
      city: json['city'] as String,
      zipcode: json['zipcode'] as String,
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'street': street,
      'city': city,
      'zipcode': zipcode,
    };
  }
}
```

### JSON Parsing Utilities
```dart
class JsonParser {
  // Safe parsing with error handling
  static T? safeParse<T>(
    Map<String, dynamic> json,
    String key,
    T Function(dynamic) parser,
  ) {
    try {
      final value = json[key];
      if (value == null) return null;
      return parser(value);
    } catch (e) {
      print('Error parsing $key: $e');
      return null;
    }
  }
  
  // Parse list with error handling
  static List<T> parseList<T>(
    dynamic json,
    T Function(Map<String, dynamic>) itemParser,
  ) {
    if (json == null) return [];
    
    try {
      if (json is List) {
        return json
            .whereType<Map<String, dynamic>>()
            .map(itemParser)
            .toList();
      }
    } catch (e) {
      print('Error parsing list: $e');
    }
    
    return [];
  }
  
  // Parse date with multiple formats
  static DateTime? parseDate(dynamic value) {
    if (value == null) return null;
    
    try {
      if (value is String) {
        // Try ISO format first
        try {
          return DateTime.parse(value);
        } catch (_) {
          // Try other common formats
          final formats = [
            'dd/MM/yyyy',
            'MM/dd/yyyy',
            'yyyy-MM-dd HH:mm:ss',
          ];
          
          for (final format in formats) {
            try {
              // You would use intl package's DateFormat here
              // return DateFormat(format).parse(value);
              return DateTime.parse(value); // Simplified
            } catch (_) {}
          }
        }
      } else if (value is int) {
        // Unix timestamp
        return DateTime.fromMillisecondsSinceEpoch(value * 1000);
      }
    } catch (e) {
      print('Error parsing date: $e');
    }
    
    return null;
  }
  
  // Validate JSON structure
  static bool validateJsonStructure(
    Map<String, dynamic> json,
    List<String> requiredFields,
  ) {
    for (final field in requiredFields) {
      if (!json.containsKey(field) || json[field] == null) {
        print('Missing required field: $field');
        return false;
      }
    }
    return true;
  }
}
```

## REST API Integration

### Complete API Service Example
```dart
class BlogApiService {
  static const String _baseUrl = 'https://api.example.com/v1';
  static String? _authToken;
  
  // Authentication
  static Future<AuthResponse> login(String email, String password) async {
    try {
      final response = await HttpClient.post(
        '$_baseUrl/auth/login',
        body: {
          'email': email,
          'password': password,
        },
      );
      
      if (response.statusCode == 200) {
        final authResponse = AuthResponse.fromJson(json.decode(response.body));
        _authToken = authResponse.token;
        HttpClient.setAuthToken(_authToken!);
        return authResponse;
      } else {
        throw ApiException(
          'Login failed',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      throw ApiException('Network error during login', 0, {'error': e.toString()});
    }
  }
  
  // Get posts with pagination
  static Future<PostsResponse> getPosts({
    int page = 1,
    int limit = 10,
    String? search,
    String? category,
  }) async {
    try {
      final queryParams = {
        'page': page.toString(),
        'limit': limit.toString(),
        if (search != null && search.isNotEmpty) 'search': search,
        if (category != null && category.isNotEmpty) 'category': category,
      };
      
      final uri = Uri.parse('$_baseUrl/posts').replace(
        queryParameters: queryParams,
      );
      
      final response = await HttpClient.get(uri.toString());
      
      if (response.statusCode == 200) {
        return PostsResponse.fromJson(json.decode(response.body));
      } else {
        throw ApiException(
          'Failed to load posts',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Get single post
  static Future<Post> getPost(int id) async {
    try {
      final response = await HttpClient.get('$_baseUrl/posts/$id');
      
      if (response.statusCode == 200) {
        return Post.fromJson(json.decode(response.body));
      } else if (response.statusCode == 404) {
        throw ApiException('Post not found', 404, {'error': 'Post not found'});
      } else {
        throw ApiException(
          'Failed to load post',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      if (e is ApiException) rethrow;
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Create post
  static Future<Post> createPost(CreatePostRequest request) async {
    try {
      final response = await HttpClient.post(
        '$_baseUrl/posts',
        body: request.toJson(),
      );
      
      if (response.statusCode == 201) {
        return Post.fromJson(json.decode(response.body));
      } else {
        throw ApiException(
          'Failed to create post',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      if (e is ApiException) rethrow;
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Update post
  static Future<Post> updatePost(int id, UpdatePostRequest request) async {
    try {
      final response = await HttpClient.put(
        '$_baseUrl/posts/$id',
        body: request.toJson(),
      );
      
      if (response.statusCode == 200) {
        return Post.fromJson(json.decode(response.body));
      } else {
        throw ApiException(
          'Failed to update post',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      if (e is ApiException) rethrow;
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Delete post
  static Future<void> deletePost(int id) async {
    try {
      final response = await HttpClient.delete('$_baseUrl/posts/$id');
      
      if (response.statusCode != 200 && response.statusCode != 204) {
        throw ApiException(
          'Failed to delete post',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      if (e is ApiException) rethrow;
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Get user profile
  static Future<User> getProfile() async {
    try {
      final response = await HttpClient.get('$_baseUrl/profile');
      
      if (response.statusCode == 200) {
        return User.fromJson(json.decode(response.body));
      } else {
        throw ApiException(
          'Failed to load profile',
          response.statusCode,
          json.decode(response.body),
        );
      }
    } catch (e) {
      if (e is ApiException) rethrow;
      throw ApiException('Network error', 0, {'error': e.toString()});
    }
  }
  
  // Logout
  static Future<void> logout() async {
    try {
      await HttpClient.post('$_baseUrl/auth/logout');
    } finally {
      _authToken = null;
      HttpClient.setAuthToken('');
    }
  }
}

// API Response Models
class AuthResponse {
  final String token;
  final User user;
  final DateTime expiresAt;
  
  AuthResponse({
    required this.token,
    required this.user,
    required this.expiresAt,
  });
  
  factory AuthResponse.fromJson(Map<String, dynamic> json) {
    return AuthResponse(
      token: json['token'] as String,
      user: User.fromJson(json['user']),
      expiresAt: DateTime.parse(json['expiresAt']),
    );
  }
}

class PostsResponse {
  final List<Post> posts;
  final int totalCount;
  final int currentPage;
  final int totalPages;
  final bool hasNext;
  final bool hasPrevious;
  
  PostsResponse({
    required this.posts,
    required this.totalCount,
    required this.currentPage,
    required this.totalPages,
    required this.hasNext,
    required this.hasPrevious,
  });
  
  factory PostsResponse.fromJson(Map<String, dynamic> json) {
    return PostsResponse(
      posts: JsonParser.parseList(json['data'], Post.fromJson),
      totalCount: json['totalCount'] as int,
      currentPage: json['currentPage'] as int,
      totalPages: json['totalPages'] as int,
      hasNext: json['hasNext'] as bool,
      hasPrevious: json['hasPrevious'] as bool,
    );
  }
}

class CreatePostRequest {
  final String title;
  final String body;
  final List<String>? tags;
  final String? category;
  
  CreatePostRequest({
    required this.title,
    required this.body,
    this.tags,
    this.category,
  });
  
  Map<String, dynamic> toJson() {
    return {
      'title': title,
      'body': body,
      if (tags != null) 'tags': tags,
      if (category != null) 'category': category,
    };
  }
}

class UpdatePostRequest {
  final String? title;
  final String? body;
  final List<String>? tags;
  final String? category;
  
  UpdatePostRequest({
    this.title,
    this.body,
    this.tags,
    this.category,
  });
  
  Map<String, dynamic> toJson() {
    return {
      if (title != null) 'title': title,
      if (body != null) 'body': body,
      if (tags != null) 'tags': tags,
      if (category != null) 'category': category,
    };
  }
}

// Custom Exception
class ApiException implements Exception {
  final String message;
  final int statusCode;
  final Map<String, dynamic> data;
  
  ApiException(this.message, this.statusCode, this.data);
  
  @override
  String toString() {
    return 'ApiException: $message (Status: $statusCode)';
  }
  
  String get userFriendlyMessage {
    switch (statusCode) {
      case 400:
        return 'Invalid request. Please check your input.';
      case 401:
        return 'Please log in to continue.';
      case 403:
        return 'You don\'t have permission to perform this action.';
      case 404:
        return 'The requested resource was not found.';
      case 500:
        return 'Server error. Please try again later.';
      default:
        return message;
    }
  }
}
```

## Local Storage

### SharedPreferences for Simple Data
```dart
class PreferencesService {
  static SharedPreferences? _prefs;
  
  static Future<void> init() async {
    _prefs = await SharedPreferences.getInstance();
  }
  
  // String operations
  static Future<void> setString(String key, String value) async {
    await _prefs?.setString(key, value);
  }
  
  static String? getString(String key) {
    return _prefs?.getString(key);
  }
  
  // Int operations
  static Future<void> setInt(String key, int value) async {
    await _prefs?.setInt(key, value);
  }
  
  static int? getInt(String key) {
    return _prefs?.getInt(key);
  }
  
  // Bool operations
  static Future<void> setBool(String key, bool value) async {
    await _prefs?.setBool(key, value);
  }
  
  static bool getBool(String key, {bool defaultValue = false}) {
    return _prefs?.getBool(key) ?? defaultValue;
  }
  
  // Double operations
  static Future<void> setDouble(String key, double value) async {
    await _prefs?.setDouble(key, value);
  }
  
  static double? getDouble(String key) {
    return _prefs?.getDouble(key);
  }
  
  // List operations
  static Future<void> setStringList(String key, List<String> value) async {
    await _prefs?.setStringList(key, value);
  }
  
  static List<String>? getStringList(String key) {
    return _prefs?.getStringList(key);
  }
  
  // Object operations (using JSON)
  static Future<void> setObject(String key, Map<String, dynamic> value) async {
    await _prefs?.setString(key, json.encode(value));
  }
  
  static Map<String, dynamic>? getObject(String key) {
    final jsonString = _prefs?.getString(key);
    if (jsonString != null) {
      try {
        return json.decode(jsonString) as Map<String, dynamic>;
      } catch (e) {
        print('Error decoding object: $e');
      }
    }
    return null;
  }
  
  // Remove operations
  static Future<void> remove(String key) async {
    await _prefs?.remove(key);
  }
  
  static Future<void> clear() async {
    await _prefs?.clear();
  }
  
  // Check if key exists
  static bool containsKey(String key) {
    return _prefs?.containsKey(key) ?? false;
  }
  
  // Get all keys
  static Set<String> getKeys() {
    return _prefs?.getKeys() ?? {};
  }
}

// User preferences manager
class UserPreferences {
  static const String _keyTheme = 'theme_mode';
  static const String _keyLanguage = 'language';
  static const String _keyNotifications = 'notifications_enabled';
  static const String _keyUser = 'user_data';
  static const String _keyOnboarding = 'onboarding_completed';
  
  // Theme
  static Future<void> setThemeMode(String themeMode) async {
    await PreferencesService.setString(_keyTheme, themeMode);
  }
  
  static String getThemeMode() {
    return PreferencesService.getString(_keyTheme) ?? 'system';
  }
  
  // Language
  static Future<void> setLanguage(String language) async {
    await PreferencesService.setString(_keyLanguage, language);
  }
  
  static String getLanguage() {
    return PreferencesService.getString(_keyLanguage) ?? 'en';
  }
  
  // Notifications
  static Future<void> setNotificationsEnabled(bool enabled) async {
    await PreferencesService.setBool(_keyNotifications, enabled);
  }
  
  static bool getNotificationsEnabled() {
    return PreferencesService.getBool(_keyNotifications, defaultValue: true);
  }
  
  // User data
  static Future<void> setUser(User user) async {
    await PreferencesService.setObject(_keyUser, user.toJson());
  }
  
  static User? getUser() {
    final userData = PreferencesService.getObject(_keyUser);
    if (userData != null) {
      try {
        return User.fromJson(userData);
      } catch (e) {
        print('Error parsing user data: $e');
      }
    }
    return null;
  }
  
  static Future<void> clearUser() async {
    await PreferencesService.remove(_keyUser);
  }
  
  // Onboarding
  static Future<void> setOnboardingCompleted(bool completed) async {
    await PreferencesService.setBool(_keyOnboarding, completed);
  }
  
  static bool isOnboardingCompleted() {
    return PreferencesService.getBool(_keyOnboarding);
  }
  
  // Clear all user data
  static Future<void> clearAllUserData() async {
    await PreferencesService.remove(_keyUser);
    await PreferencesService.remove(_keyTheme);
    await PreferencesService.remove(_keyLanguage);
    await PreferencesService.remove(_keyNotifications);
  }
}
```

This data and networking guide covers the essential concepts for working with APIs and local storage in Flutter. The remaining sections (Database Integration, File Handling, and Caching Strategies) would be covered in a separate file to keep this manageable.