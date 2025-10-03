# Flutter Data and Networking Guide - Part 2

## Table of Contents
1. [Database Integration](#database-integration)
2. [File Handling](#file-handling)
3. [Caching Strategies](#caching-strategies)
4. [Offline Data Management](#offline-data-management)

## Database Integration

### SQLite with Sqflite
```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static Database? _database;
  static const String _databaseName = 'app_database.db';
  static const int _databaseVersion = 1;
  
  // Tables
  static const String tableUsers = 'users';
  static const String tablePosts = 'posts';
  static const String tableComments = 'comments';
  
  // Singleton pattern
  DatabaseHelper._privateConstructor();
  static final DatabaseHelper instance = DatabaseHelper._privateConstructor();
  
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }
  
  Future<Database> _initDatabase() async {
    String path = join(await getDatabasesPath(), _databaseName);
    return await openDatabase(
      path,
      version: _databaseVersion,
      onCreate: _onCreate,
      onUpgrade: _onUpgrade,
    );
  }
  
  Future<void> _onCreate(Database db, int version) async {
    // Create users table
    await db.execute('''
      CREATE TABLE $tableUsers (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        phone TEXT,
        created_at TEXT NOT NULL,
        updated_at TEXT NOT NULL
      )
    ''');
    
    // Create posts table
    await db.execute('''
      CREATE TABLE $tablePosts (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER NOT NULL,
        title TEXT NOT NULL,
        body TEXT NOT NULL,
        tags TEXT,
        created_at TEXT NOT NULL,
        updated_at TEXT NOT NULL,
        FOREIGN KEY (user_id) REFERENCES $tableUsers (id) ON DELETE CASCADE
      )
    ''');
    
    // Create comments table
    await db.execute('''
      CREATE TABLE $tableComments (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        post_id INTEGER NOT NULL,
        user_id INTEGER NOT NULL,
        content TEXT NOT NULL,
        created_at TEXT NOT NULL,
        FOREIGN KEY (post_id) REFERENCES $tablePosts (id) ON DELETE CASCADE,
        FOREIGN KEY (user_id) REFERENCES $tableUsers (id) ON DELETE CASCADE
      )
    ''');
    
    // Create indexes
    await db.execute('CREATE INDEX idx_posts_user_id ON $tablePosts (user_id)');
    await db.execute('CREATE INDEX idx_comments_post_id ON $tableComments (post_id)');
    await db.execute('CREATE INDEX idx_users_email ON $tableUsers (email)');
  }
  
  Future<void> _onUpgrade(Database db, int oldVersion, int newVersion) async {
    // Handle database upgrades
    if (oldVersion < 2) {
      // Add new columns or tables for version 2
      // await db.execute('ALTER TABLE $tableUsers ADD COLUMN profile_image TEXT');
    }
  }
}

// User Database Operations
class UserDao {
  static final DatabaseHelper _dbHelper = DatabaseHelper.instance;
  
  // Insert user
  static Future<int> insertUser(User user) async {
    final db = await _dbHelper.database;
    final now = DateTime.now().toIso8601String();
    
    final userMap = user.toJson();
    userMap['created_at'] = now;
    userMap['updated_at'] = now;
    
    return await db.insert(
      DatabaseHelper.tableUsers,
      userMap,
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }
  
  // Get user by ID
  static Future<User?> getUserById(int id) async {
    final db = await _dbHelper.database;
    final maps = await db.query(
      DatabaseHelper.tableUsers,
      where: 'id = ?',
      whereArgs: [id],
    );
    
    if (maps.isNotEmpty) {
      return User.fromJson(maps.first);
    }
    return null;
  }
  
  // Get user by email
  static Future<User?> getUserByEmail(String email) async {
    final db = await _dbHelper.database;
    final maps = await db.query(
      DatabaseHelper.tableUsers,
      where: 'email = ?',
      whereArgs: [email],
    );
    
    if (maps.isNotEmpty) {
      return User.fromJson(maps.first);
    }
    return null;
  }
  
  // Get all users
  static Future<List<User>> getAllUsers() async {
    final db = await _dbHelper.database;
    final maps = await db.query(
      DatabaseHelper.tableUsers,
      orderBy: 'name ASC',
    );
    
    return maps.map((map) => User.fromJson(map)).toList();
  }
  
  // Update user
  static Future<int> updateUser(User user) async {
    final db = await _dbHelper.database;
    final userMap = user.toJson();
    userMap['updated_at'] = DateTime.now().toIso8601String();
    
    return await db.update(
      DatabaseHelper.tableUsers,
      userMap,
      where: 'id = ?',
      whereArgs: [user.id],
    );
  }
  
  // Delete user
  static Future<int> deleteUser(int id) async {
    final db = await _dbHelper.database;
    return await db.delete(
      DatabaseHelper.tableUsers,
      where: 'id = ?',
      whereArgs: [id],
    );
  }
  
  // Search users
  static Future<List<User>> searchUsers(String query) async {
    final db = await _dbHelper.database;
    final maps = await db.query(
      DatabaseHelper.tableUsers,
      where: 'name LIKE ? OR email LIKE ?',
      whereArgs: ['%$query%', '%$query%'],
      orderBy: 'name ASC',
    );
    
    return maps.map((map) => User.fromJson(map)).toList();
  }
}

// Post Database Operations
class PostDao {
  static final DatabaseHelper _dbHelper = DatabaseHelper.instance;
  
  // Insert post
  static Future<int> insertPost(Post post) async {
    final db = await _dbHelper.database;
    final now = DateTime.now().toIso8601String();
    
    final postMap = {
      'user_id': post.userId,
      'title': post.title,
      'body': post.body,
      'tags': post.tags?.join(','),
      'created_at': now,
      'updated_at': now,
    };
    
    return await db.insert(DatabaseHelper.tablePosts, postMap);
  }
  
  // Get post by ID with user info
  static Future<Post?> getPostById(int id) async {
    final db = await _dbHelper.database;
    final maps = await db.rawQuery('''
      SELECT p.*, u.name as user_name, u.email as user_email
      FROM ${DatabaseHelper.tablePosts} p
      LEFT JOIN ${DatabaseHelper.tableUsers} u ON p.user_id = u.id
      WHERE p.id = ?
    ''', [id]);
    
    if (maps.isNotEmpty) {
      return _mapToPost(maps.first);
    }
    return null;
  }
  
  // Get posts by user
  static Future<List<Post>> getPostsByUser(int userId) async {
    final db = await _dbHelper.database;
    final maps = await db.rawQuery('''
      SELECT p.*, u.name as user_name, u.email as user_email
      FROM ${DatabaseHelper.tablePosts} p
      LEFT JOIN ${DatabaseHelper.tableUsers} u ON p.user_id = u.id
      WHERE p.user_id = ?
      ORDER BY p.created_at DESC
    ''', [userId]);
    
    return maps.map((map) => _mapToPost(map)).toList();
  }
  
  // Get all posts with pagination
  static Future<List<Post>> getAllPosts({int limit = 20, int offset = 0}) async {
    final db = await _dbHelper.database;
    final maps = await db.rawQuery('''
      SELECT p.*, u.name as user_name, u.email as user_email
      FROM ${DatabaseHelper.tablePosts} p
      LEFT JOIN ${DatabaseHelper.tableUsers} u ON p.user_id = u.id
      ORDER BY p.created_at DESC
      LIMIT ? OFFSET ?
    ''', [limit, offset]);
    
    return maps.map((map) => _mapToPost(map)).toList();
  }
  
  // Search posts
  static Future<List<Post>> searchPosts(String query) async {
    final db = await _dbHelper.database;
    final maps = await db.rawQuery('''
      SELECT p.*, u.name as user_name, u.email as user_email
      FROM ${DatabaseHelper.tablePosts} p
      LEFT JOIN ${DatabaseHelper.tableUsers} u ON p.user_id = u.id
      WHERE p.title LIKE ? OR p.body LIKE ? OR p.tags LIKE ?
      ORDER BY p.created_at DESC
    ''', ['%$query%', '%$query%', '%$query%']);
    
    return maps.map((map) => _mapToPost(map)).toList();
  }
  
  // Update post
  static Future<int> updatePost(Post post) async {
    final db = await _dbHelper.database;
    final postMap = {
      'title': post.title,
      'body': post.body,
      'tags': post.tags?.join(','),
      'updated_at': DateTime.now().toIso8601String(),
    };
    
    return await db.update(
      DatabaseHelper.tablePosts,
      postMap,
      where: 'id = ?',
      whereArgs: [post.id],
    );
  }
  
  // Delete post
  static Future<int> deletePost(int id) async {
    final db = await _dbHelper.database;
    return await db.delete(
      DatabaseHelper.tablePosts,
      where: 'id = ?',
      whereArgs: [id],
    );
  }
  
  // Get posts count by user
  static Future<int> getPostsCountByUser(int userId) async {
    final db = await _dbHelper.database;
    final result = await db.rawQuery('''
      SELECT COUNT(*) as count
      FROM ${DatabaseHelper.tablePosts}
      WHERE user_id = ?
    ''', [userId]);
    
    return Sqflite.firstIntValue(result) ?? 0;
  }
  
  static Post _mapToPost(Map<String, dynamic> map) {
    return Post(
      id: map['id'] as int?,
      userId: map['user_id'] as int,
      title: map['title'] as String,
      body: map['body'] as String,
      tags: map['tags'] != null 
          ? (map['tags'] as String).split(',').where((tag) => tag.isNotEmpty).toList()
          : null,
      createdAt: map['created_at'] != null 
          ? DateTime.parse(map['created_at'])
          : null,
      author: map['user_name'] != null 
          ? User(
              id: map['user_id'] as int,
              name: map['user_name'] as String,
              email: map['user_email'] as String,
            )
          : null,
    );
  }
}

// Transaction example
class DatabaseTransactions {
  static final DatabaseHelper _dbHelper = DatabaseHelper.instance;
  
  // Create user with initial post
  static Future<Map<String, int>> createUserWithPost(
    User user,
    Post post,
  ) async {
    final db = await _dbHelper.database;
    Map<String, int> result = {};
    
    await db.transaction((txn) async {
      // Insert user
      final userId = await txn.insert(
        DatabaseHelper.tableUsers,
        {
          ...user.toJson(),
          'created_at': DateTime.now().toIso8601String(),
          'updated_at': DateTime.now().toIso8601String(),
        },
      );
      result['userId'] = userId;
      
      // Insert post with the new user ID
      final postId = await txn.insert(
        DatabaseHelper.tablePosts,
        {
          'user_id': userId,
          'title': post.title,
          'body': post.body,
          'tags': post.tags?.join(','),
          'created_at': DateTime.now().toIso8601String(),
          'updated_at': DateTime.now().toIso8601String(),
        },
      );
      result['postId'] = postId;
    });
    
    return result;
  }
  
  // Bulk insert posts
  static Future<void> bulkInsertPosts(List<Post> posts) async {
    final db = await _dbHelper.database;
    final batch = db.batch();
    
    for (final post in posts) {
      batch.insert(
        DatabaseHelper.tablePosts,
        {
          'user_id': post.userId,
          'title': post.title,
          'body': post.body,
          'tags': post.tags?.join(','),
          'created_at': DateTime.now().toIso8601String(),
          'updated_at': DateTime.now().toIso8601String(),
        },
      );
    }
    
    await batch.commit(noResult: true);
  }
}
```

## File Handling

### File Operations
```dart
import 'dart:io';
import 'dart:typed_data';
import 'package:path_provider/path_provider.dart';
import 'package:image_picker/image_picker.dart';

class FileService {
  // Get application directories
  static Future<Directory> getAppDocumentsDirectory() async {
    return await getApplicationDocumentsDirectory();
  }
  
  static Future<Directory> getAppSupportDirectory() async {
    return await getApplicationSupportDirectory();
  }
  
  static Future<Directory?> getExternalStorageDirectory() async {
    return await getExternalStorageDirectory();
  }
  
  static Future<Directory> getTemporaryDirectory() async {
    return await getTemporaryDirectory();
  }
  
  // Text file operations
  static Future<void> writeTextFile(String fileName, String content) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      await file.writeAsString(content);
    } catch (e) {
      throw FileException('Failed to write text file: $e');
    }
  }
  
  static Future<String?> readTextFile(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      
      if (await file.exists()) {
        return await file.readAsString();
      }
      return null;
    } catch (e) {
      throw FileException('Failed to read text file: $e');
    }
  }
  
  // JSON file operations
  static Future<void> writeJsonFile(
    String fileName,
    Map<String, dynamic> data,
  ) async {
    try {
      final jsonString = json.encode(data);
      await writeTextFile(fileName, jsonString);
    } catch (e) {
      throw FileException('Failed to write JSON file: $e');
    }
  }
  
  static Future<Map<String, dynamic>?> readJsonFile(String fileName) async {
    try {
      final jsonString = await readTextFile(fileName);
      if (jsonString != null) {
        return json.decode(jsonString) as Map<String, dynamic>;
      }
      return null;
    } catch (e) {
      throw FileException('Failed to read JSON file: $e');
    }
  }
  
  // Binary file operations
  static Future<void> writeBinaryFile(String fileName, Uint8List data) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      await file.writeAsBytes(data);
    } catch (e) {
      throw FileException('Failed to write binary file: $e');
    }
  }
  
  static Future<Uint8List?> readBinaryFile(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      
      if (await file.exists()) {
        return await file.readAsBytes();
      }
      return null;
    } catch (e) {
      throw FileException('Failed to read binary file: $e');
    }
  }
  
  // Image operations
  static Future<String?> saveImageFile(String fileName, Uint8List imageData) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final imagesDir = Directory('${directory.path}/images');
      
      if (!await imagesDir.exists()) {
        await imagesDir.create(recursive: true);
      }
      
      final file = File('${imagesDir.path}/$fileName');
      await file.writeAsBytes(imageData);
      return file.path;
    } catch (e) {
      throw FileException('Failed to save image file: $e');
    }
  }
  
  static Future<Uint8List?> loadImageFile(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/images/$fileName');
      
      if (await file.exists()) {
        return await file.readAsBytes();
      }
      return null;
    } catch (e) {
      throw FileException('Failed to load image file: $e');
    }
  }
  
  // File info operations
  static Future<bool> fileExists(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      return await file.exists();
    } catch (e) {
      return false;
    }
  }
  
  static Future<int?> getFileSize(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      
      if (await file.exists()) {
        final stat = await file.stat();
        return stat.size;
      }
      return null;
    } catch (e) {
      throw FileException('Failed to get file size: $e');
    }
  }
  
  static Future<DateTime?> getFileModifiedDate(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      
      if (await file.exists()) {
        final stat = await file.stat();
        return stat.modified;
      }
      return null;
    } catch (e) {
      throw FileException('Failed to get file modified date: $e');
    }
  }
  
  // Delete operations
  static Future<bool> deleteFile(String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final file = File('${directory.path}/$fileName');
      
      if (await file.exists()) {
        await file.delete();
        return true;
      }
      return false;
    } catch (e) {
      throw FileException('Failed to delete file: $e');
    }
  }
  
  static Future<void> clearDirectory(String dirName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final targetDir = Directory('${directory.path}/$dirName');
      
      if (await targetDir.exists()) {
        await targetDir.delete(recursive: true);
        await targetDir.create();
      }
    } catch (e) {
      throw FileException('Failed to clear directory: $e');
    }
  }
  
  // List files
  static Future<List<String>> listFiles({String? subdirectory}) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final targetDir = subdirectory != null 
          ? Directory('${directory.path}/$subdirectory')
          : directory;
      
      if (await targetDir.exists()) {
        final entities = await targetDir.list().toList();
        return entities
            .whereType<File>()
            .map((file) => file.path.split('/').last)
            .toList();
      }
      return [];
    } catch (e) {
      throw FileException('Failed to list files: $e');
    }
  }
  
  // Copy file
  static Future<String> copyFile(String sourcePath, String fileName) async {
    try {
      final directory = await getAppDocumentsDirectory();
      final sourceFile = File(sourcePath);
      final targetFile = File('${directory.path}/$fileName');
      
      await sourceFile.copy(targetFile.path);
      return targetFile.path;
    } catch (e) {
      throw FileException('Failed to copy file: $e');
    }
  }
}

// Image handling service
class ImageService {
  static final ImagePicker _picker = ImagePicker();
  
  // Pick image from gallery
  static Future<String?> pickImageFromGallery() async {
    try {
      final XFile? image = await _picker.pickImage(
        source: ImageSource.gallery,
        maxWidth: 1920,
        maxHeight: 1080,
        imageQuality: 85,
      );
      
      if (image != null) {
        return image.path;
      }
      return null;
    } catch (e) {
      throw FileException('Failed to pick image from gallery: $e');
    }
  }
  
  // Take photo with camera
  static Future<String?> takePhoto() async {
    try {
      final XFile? image = await _picker.pickImage(
        source: ImageSource.camera,
        maxWidth: 1920,
        maxHeight: 1080,
        imageQuality: 85,
      );
      
      if (image != null) {
        return image.path;
      }
      return null;
    } catch (e) {
      throw FileException('Failed to take photo: $e');
    }
  }
  
  // Save picked image to app directory
  static Future<String?> savePickedImage(String sourcePath) async {
    try {
      final fileName = 'img_${DateTime.now().millisecondsSinceEpoch}.jpg';
      final imageData = await File(sourcePath).readAsBytes();
      return await FileService.saveImageFile(fileName, imageData);
    } catch (e) {
      throw FileException('Failed to save picked image: $e');
    }
  }
  
  // Pick multiple images
  static Future<List<String>> pickMultipleImages() async {
    try {
      final List<XFile>? images = await _picker.pickMultiImage(
        maxWidth: 1920,
        maxHeight: 1080,
        imageQuality: 85,
      );
      
      if (images != null) {
        return images.map((image) => image.path).toList();
      }
      return [];
    } catch (e) {
      throw FileException('Failed to pick multiple images: $e');
    }
  }
}

// File Exception
class FileException implements Exception {
  final String message;
  
  FileException(this.message);
  
  @override
  String toString() => 'FileException: $message';
}
```

## Caching Strategies

### HTTP Response Caching
```dart
class CacheService {
  static const String _cacheDir = 'http_cache';
  static const Duration _defaultTtl = Duration(hours: 1);
  
  // Cache entry model
  static Future<void> cacheResponse(
    String key,
    String data, {
    Duration? ttl,
  }) async {
    try {
      final cacheData = {
        'data': data,
        'timestamp': DateTime.now().millisecondsSinceEpoch,
        'ttl': (ttl ?? _defaultTtl).inMilliseconds,
      };
      
      await FileService.writeJsonFile(
        '$_cacheDir/$key.json',
        cacheData,
      );
    } catch (e) {
      print('Failed to cache response: $e');
    }
  }
  
  static Future<String?> getCachedResponse(String key) async {
    try {
      final cacheData = await FileService.readJsonFile('$_cacheDir/$key.json');
      
      if (cacheData != null) {
        final timestamp = cacheData['timestamp'] as int;
        final ttl = cacheData['ttl'] as int;
        final now = DateTime.now().millisecondsSinceEpoch;
        
        if (now - timestamp < ttl) {
          return cacheData['data'] as String;
        } else {
          // Cache expired, remove it
          await _removeCacheFile(key);
        }
      }
      
      return null;
    } catch (e) {
      print('Failed to get cached response: $e');
      return null;
    }
  }
  
  static Future<void> _removeCacheFile(String key) async {
    try {
      await FileService.deleteFile('$_cacheDir/$key.json');
    } catch (e) {
      print('Failed to remove cache file: $e');
    }
  }
  
  static Future<void> clearCache() async {
    try {
      await FileService.clearDirectory(_cacheDir);
    } catch (e) {
      print('Failed to clear cache: $e');
    }
  }
  
  static Future<void> clearExpiredCache() async {
    try {
      final files = await FileService.listFiles(subdirectory: _cacheDir);
      final now = DateTime.now().millisecondsSinceEpoch;
      
      for (final fileName in files) {
        final key = fileName.replaceAll('.json', '');
        final cacheData = await FileService.readJsonFile('$_cacheDir/$fileName');
        
        if (cacheData != null) {
          final timestamp = cacheData['timestamp'] as int;
          final ttl = cacheData['ttl'] as int;
          
          if (now - timestamp >= ttl) {
            await _removeCacheFile(key);
          }
        }
      }
    } catch (e) {
      print('Failed to clear expired cache: $e');
    }
  }
}

// Enhanced API service with caching
class CachedApiService {
  static Future<List<Post>> getCachedPosts({bool forceRefresh = false}) async {
    const cacheKey = 'posts_list';
    
    // Try to get from cache first
    if (!forceRefresh) {
      final cachedData = await CacheService.getCachedResponse(cacheKey);
      if (cachedData != null) {
        try {
          final List<dynamic> jsonData = json.decode(cachedData);
          return jsonData.map((json) => Post.fromJson(json)).toList();
        } catch (e) {
          print('Failed to parse cached posts: $e');
        }
      }
    }
    
    // Fetch fresh data
    try {
      final posts = await BlogApiService.getPosts();
      
      // Cache the response
      await CacheService.cacheResponse(
        cacheKey,
        json.encode(posts.posts.map((post) => post.toJson()).toList()),
        ttl: Duration(minutes: 30),
      );
      
      return posts.posts;
    } catch (e) {
      // If network fails, try to return stale cache
      final cachedData = await CacheService.getCachedResponse(cacheKey);
      if (cachedData != null) {
        try {
          final List<dynamic> jsonData = json.decode(cachedData);
          return jsonData.map((json) => Post.fromJson(json)).toList();
        } catch (e) {
          print('Failed to parse stale cached posts: $e');
        }
      }
      rethrow;
    }
  }
}
```

## Offline Data Management

### Offline-First Architecture
```dart
class OfflineDataManager {
  static const String _syncQueueTable = 'sync_queue';
  
  // Sync queue item
  static Future<void> addToSyncQueue(
    String operation,
    String endpoint,
    Map<String, dynamic> data,
  ) async {
    try {
      final db = await DatabaseHelper.instance.database;
      await db.insert(_syncQueueTable, {
        'operation': operation, // CREATE, UPDATE, DELETE
        'endpoint': endpoint,
        'data': json.encode(data),
        'created_at': DateTime.now().toIso8601String(),
        'attempts': 0,
      });
    } catch (e) {
      print('Failed to add to sync queue: $e');
    }
  }
  
  // Process sync queue
  static Future<void> processSyncQueue() async {
    try {
      final db = await DatabaseHelper.instance.database;
      final pendingItems = await db.query(
        _syncQueueTable,
        where: 'attempts < ?',
        whereArgs: [3], // Max 3 attempts
        orderBy: 'created_at ASC',
      );
      
      for (final item in pendingItems) {
        try {
          await _processSyncItem(item);
          
          // Remove successful item
          await db.delete(
            _syncQueueTable,
            where: 'id = ?',
            whereArgs: [item['id']],
          );
        } catch (e) {
          // Increment attempts
          await db.update(
            _syncQueueTable,
            {'attempts': (item['attempts'] as int) + 1},
            where: 'id = ?',
            whereArgs: [item['id']],
          );
          print('Failed to sync item ${item['id']}: $e');
        }
      }
    } catch (e) {
      print('Failed to process sync queue: $e');
    }
  }
  
  static Future<void> _processSyncItem(Map<String, dynamic> item) async {
    final operation = item['operation'] as String;
    final endpoint = item['endpoint'] as String;
    final data = json.decode(item['data'] as String) as Map<String, dynamic>;
    
    switch (operation) {
      case 'CREATE':
        await HttpClient.post(endpoint, body: data);
        break;
      case 'UPDATE':
        await HttpClient.put(endpoint, body: data);
        break;
      case 'DELETE':
        await HttpClient.delete(endpoint);
        break;
    }
  }
  
  // Create post offline
  static Future<Post> createPostOffline(CreatePostRequest request) async {
    // Create local post
    final post = Post(
      id: DateTime.now().millisecondsSinceEpoch, // Temporary ID
      userId: request.userId ?? 0,
      title: request.title,
      body: request.body,
      tags: request.tags,
      createdAt: DateTime.now(),
    );
    
    // Save to local database
    await PostDao.insertPost(post);
    
    // Add to sync queue
    await addToSyncQueue(
      'CREATE',
      '/api/posts',
      request.toJson(),
    );
    
    return post;
  }
  
  // Check connectivity and sync
  static Future<bool> isOnline() async {
    try {
      final result = await http.get(Uri.parse('https://www.google.com'));
      return result.statusCode == 200;
    } catch (e) {
      return false;
    }
  }
  
  static Future<void> syncWhenOnline() async {
    if (await isOnline()) {
      await processSyncQueue();
      await CacheService.clearExpiredCache();
    }
  }
}
```

This completes the comprehensive data and networking guide for Flutter, covering HTTP requests, JSON parsing, database integration, file handling, caching, and offline data management strategies.