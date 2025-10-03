# Flutter Project Structure and Best Practices Guide

## Table of Contents
1. [Project Architecture](#project-architecture)
2. [Folder Structure](#folder-structure)
3. [Code Organization](#code-organization)
4. [Design Patterns](#design-patterns)
5. [Best Practices](#best-practices)

## Project Architecture

### Clean Architecture Structure
```
lib/
├── core/
│   ├── constants/
│   │   ├── app_constants.dart
│   │   ├── api_constants.dart
│   │   └── string_constants.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   ├── failures.dart
│   │   └── error_handler.dart
│   ├── network/
│   │   ├── api_client.dart
│   │   ├── network_info.dart
│   │   └── interceptors.dart
│   ├── utils/
│   │   ├── formatters.dart
│   │   ├── validators.dart
│   │   └── helpers.dart
│   └── services/
│       ├── storage_service.dart
│       ├── notification_service.dart
│       └── location_service.dart
├── data/
│   ├── datasources/
│   │   ├── local/
│   │   │   ├── user_local_datasource.dart
│   │   │   └── cache_manager.dart
│   │   └── remote/
│   │       ├── user_remote_datasource.dart
│   │       └── api_endpoints.dart
│   ├── models/
│   │   ├── user_model.dart
│   │   ├── post_model.dart
│   │   └── response_model.dart
│   └── repositories/
│       ├── user_repository_impl.dart
│       └── post_repository_impl.dart
├── domain/
│   ├── entities/
│   │   ├── user.dart
│   │   └── post.dart
│   ├── repositories/
│   │   ├── user_repository.dart
│   │   └── post_repository.dart
│   └── usecases/
│       ├── get_user_profile.dart
│       ├── update_user_profile.dart
│       └── get_posts.dart
├── presentation/
│   ├── bloc/
│   │   ├── user/
│   │   │   ├── user_bloc.dart
│   │   │   ├── user_event.dart
│   │   │   └── user_state.dart
│   │   └── posts/
│   │       ├── posts_bloc.dart
│   │       ├── posts_event.dart
│   │       └── posts_state.dart
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── login_page.dart
│   │   │   └── register_page.dart
│   │   ├── home/
│   │   │   ├── home_page.dart
│   │   │   └── dashboard_page.dart
│   │   └── profile/
│   │       ├── profile_page.dart
│   │       └── edit_profile_page.dart
│   ├── widgets/
│   │   ├── common/
│   │   │   ├── custom_button.dart
│   │   │   ├── custom_text_field.dart
│   │   │   ├── loading_widget.dart
│   │   │   └── error_widget.dart
│   │   ├── user/
│   │   │   ├── user_avatar.dart
│   │   │   └── user_card.dart
│   │   └── post/
│   │       ├── post_card.dart
│   │       └── post_list.dart
│   └── theme/
│       ├── app_theme.dart
│       ├── colors.dart
│       ├── text_styles.dart
│       └── dimensions.dart
└── main.dart
```

### Core Constants
```dart
// core/constants/app_constants.dart
class AppConstants {
  static const String appName = 'Flutter App';
  static const String appVersion = '1.0.0';
  static const Duration connectionTimeout = Duration(seconds: 30);
  static const Duration receiveTimeout = Duration(seconds: 30);
  static const int itemsPerPage = 20;
  static const int maxRetryAttempts = 3;
}

// core/constants/api_constants.dart
class ApiConstants {
  static const String baseUrl = 'https://api.example.com';
  static const String apiVersion = '/v1';
  
  // Endpoints
  static const String login = '$apiVersion/auth/login';
  static const String register = '$apiVersion/auth/register';
  static const String users = '$apiVersion/users';
  static const String posts = '$apiVersion/posts';
  static const String refreshToken = '$apiVersion/auth/refresh';
  
  // Headers
  static const String authHeader = 'Authorization';
  static const String contentType = 'Content-Type';
  static const String accept = 'Accept';
  static const String applicationJson = 'application/json';
}

// core/constants/string_constants.dart
class StringConstants {
  // Common
  static const String loading = 'Loading...';
  static const String error = 'Error';
  static const String success = 'Success';
  static const String retry = 'Retry';
  static const String cancel = 'Cancel';
  static const String save = 'Save';
  static const String delete = 'Delete';
  
  // Auth
  static const String login = 'Login';
  static const String register = 'Register';
  static const String logout = 'Logout';
  static const String email = 'Email';
  static const String password = 'Password';
  static const String confirmPassword = 'Confirm Password';
  
  // Validation
  static const String fieldRequired = 'This field is required';
  static const String invalidEmail = 'Please enter a valid email';
  static const String passwordTooShort = 'Password must be at least 6 characters';
  static const String passwordsDoNotMatch = 'Passwords do not match';
}
```

### Error Handling
```dart
// core/errors/failures.dart
abstract class Failure {
  final String message;
  final int? statusCode;
  
  const Failure({required this.message, this.statusCode});
}

class ServerFailure extends Failure {
  const ServerFailure({required String message, int? statusCode})
      : super(message: message, statusCode: statusCode);
}

class NetworkFailure extends Failure {
  const NetworkFailure({required String message})
      : super(message: message);
}

class CacheFailure extends Failure {
  const CacheFailure({required String message})
      : super(message: message);
}

class ValidationFailure extends Failure {
  const ValidationFailure({required String message})
      : super(message: message);
}

// core/errors/exceptions.dart
class ServerException implements Exception {
  final String message;
  final int? statusCode;
  
  ServerException({required this.message, this.statusCode});
}

class NetworkException implements Exception {
  final String message;
  
  NetworkException({required this.message});
}

class CacheException implements Exception {
  final String message;
  
  CacheException({required this.message});
}

// core/errors/error_handler.dart
class ErrorHandler {
  static Failure handleException(Exception exception) {
    if (exception is ServerException) {
      return ServerFailure(
        message: exception.message,
        statusCode: exception.statusCode,
      );
    } else if (exception is NetworkException) {
      return NetworkFailure(message: exception.message);
    } else if (exception is CacheException) {
      return CacheFailure(message: exception.message);
    } else {
      return ServerFailure(message: 'Unexpected error occurred');
    }
  }
  
  static String getErrorMessage(Failure failure) {
    switch (failure.runtimeType) {
      case ServerFailure:
        return failure.message;
      case NetworkFailure:
        return 'Please check your internet connection';
      case CacheFailure:
        return 'Failed to load cached data';
      case ValidationFailure:
        return failure.message;
      default:
        return 'Something went wrong';
    }
  }
}
```

## Folder Structure

### Feature-Based Structure
```dart
// Alternative feature-based structure
lib/
├── core/
├── features/
│   ├── authentication/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── bloc/
│   │       ├── pages/
│   │       └── widgets/
│   ├── user_profile/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── posts/
│       ├── data/
│       ├── domain/
│       └── presentation/
├── shared/
│   ├── widgets/
│   ├── utils/
│   └── theme/
└── main.dart
```

### Shared Components
```dart
// shared/widgets/base_widget.dart
abstract class BaseWidget extends StatelessWidget {
  const BaseWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      child: buildContent(context),
    );
  }
  
  Widget buildContent(BuildContext context);
}

// shared/widgets/loading_overlay.dart
class LoadingOverlay extends StatelessWidget {
  final bool isLoading;
  final Widget child;
  final String? message;
  
  const LoadingOverlay({
    Key? key,
    required this.isLoading,
    required this.child,
    this.message,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        child,
        if (isLoading)
          Container(
            color: Colors.black.withOpacity(0.5),
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  if (message != null) ...[
                    SizedBox(height: 16),
                    Text(
                      message!,
                      style: TextStyle(color: Colors.white),
                    ),
                  ],
                ],
              ),
            ),
          ),
      ],
    );
  }
}

// shared/utils/app_router.dart
class AppRouter {
  static const String splash = '/';
  static const String login = '/login';
  static const String register = '/register';
  static const String home = '/home';
  static const String profile = '/profile';
  static const String editProfile = '/edit-profile';
  static const String posts = '/posts';
  static const String postDetail = '/post-detail';
  
  static Route<dynamic> generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case splash:
        return MaterialPageRoute(builder: (_) => SplashPage());
      case login:
        return MaterialPageRoute(builder: (_) => LoginPage());
      case register:
        return MaterialPageRoute(builder: (_) => RegisterPage());
      case home:
        return MaterialPageRoute(builder: (_) => HomePage());
      case profile:
        return MaterialPageRoute(builder: (_) => ProfilePage());
      case editProfile:
        return MaterialPageRoute(builder: (_) => EditProfilePage());
      case posts:
        return MaterialPageRoute(builder: (_) => PostsPage());
      case postDetail:
        final post = settings.arguments as Post;
        return MaterialPageRoute(
          builder: (_) => PostDetailPage(post: post),
        );
      default:
        return MaterialPageRoute(
          builder: (_) => Scaffold(
            body: Center(
              child: Text('Route ${settings.name} not found'),
            ),
          ),
        );
    }
  }
}
```

## Code Organization

### Repository Pattern
```dart
// domain/repositories/user_repository.dart
abstract class UserRepository {
  Future<Either<Failure, User>> getUserProfile(int userId);
  Future<Either<Failure, User>> updateUserProfile(User user);
  Future<Either<Failure, List<User>>> getUsers();
  Future<Either<Failure, void>> deleteUser(int userId);
}

// data/repositories/user_repository_impl.dart
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;
  final UserLocalDataSource localDataSource;
  final NetworkInfo networkInfo;
  
  UserRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });
  
  @override
  Future<Either<Failure, User>> getUserProfile(int userId) async {
    try {
      if (await networkInfo.isConnected) {
        final userModel = await remoteDataSource.getUserProfile(userId);
        await localDataSource.cacheUser(userModel);
        return Right(userModel.toEntity());
      } else {
        final cachedUser = await localDataSource.getCachedUser(userId);
        return Right(cachedUser.toEntity());
      }
    } on ServerException catch (e) {
      return Left(ServerFailure(message: e.message));
    } on CacheException catch (e) {
      return Left(CacheFailure(message: e.message));
    } catch (e) {
      return Left(ServerFailure(message: 'Unexpected error occurred'));
    }
  }
  
  @override
  Future<Either<Failure, User>> updateUserProfile(User user) async {
    try {
      final userModel = UserModel.fromEntity(user);
      final updatedUser = await remoteDataSource.updateUserProfile(userModel);
      await localDataSource.cacheUser(updatedUser);
      return Right(updatedUser.toEntity());
    } on ServerException catch (e) {
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      return Left(ServerFailure(message: 'Failed to update profile'));
    }
  }
  
  // ... other methods
}
```

### Use Cases
```dart
// domain/usecases/get_user_profile.dart
class GetUserProfile {
  final UserRepository repository;
  
  GetUserProfile(this.repository);
  
  Future<Either<Failure, User>> call(int userId) async {
    return await repository.getUserProfile(userId);
  }
}

// domain/usecases/usecase.dart
abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

class NoParams {}

// domain/usecases/get_posts.dart
class GetPosts implements UseCase<List<Post>, GetPostsParams> {
  final PostRepository repository;
  
  GetPosts(this.repository);
  
  @override
  Future<Either<Failure, List<Post>>> call(GetPostsParams params) async {
    return await repository.getPosts(
      page: params.page,
      limit: params.limit,
      userId: params.userId,
    );
  }
}

class GetPostsParams {
  final int page;
  final int limit;
  final int? userId;
  
  GetPostsParams({
    required this.page,
    required this.limit,
    this.userId,
  });
}
```

### State Management with BLoC
```dart
// presentation/bloc/user/user_event.dart
abstract class UserEvent {}

class LoadUserProfile extends UserEvent {
  final int userId;
  
  LoadUserProfile(this.userId);
}

class UpdateUserProfile extends UserEvent {
  final User user;
  
  UpdateUserProfile(this.user);
}

class RefreshUserProfile extends UserEvent {}

// presentation/bloc/user/user_state.dart
abstract class UserState {}

class UserInitial extends UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  
  UserLoaded(this.user);
}

class UserError extends UserState {
  final String message;
  
  UserError(this.message);
}

// presentation/bloc/user/user_bloc.dart
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUserProfile getUserProfile;
  final UpdateUserProfile updateUserProfile;
  
  UserBloc({
    required this.getUserProfile,
    required this.updateUserProfile,
  }) : super(UserInitial()) {
    on<LoadUserProfile>(_onLoadUserProfile);
    on<UpdateUserProfile>(_onUpdateUserProfile);
    on<RefreshUserProfile>(_onRefreshUserProfile);
  }
  
  Future<void> _onLoadUserProfile(
    LoadUserProfile event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    
    final result = await getUserProfile(event.userId);
    
    result.fold(
      (failure) => emit(UserError(ErrorHandler.getErrorMessage(failure))),
      (user) => emit(UserLoaded(user)),
    );
  }
  
  Future<void> _onUpdateUserProfile(
    UpdateUserProfile event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    
    final result = await updateUserProfile(event.user);
    
    result.fold(
      (failure) => emit(UserError(ErrorHandler.getErrorMessage(failure))),
      (user) => emit(UserLoaded(user)),
    );
  }
  
  Future<void> _onRefreshUserProfile(
    RefreshUserProfile event,
    Emitter<UserState> emit,
  ) async {
    if (state is UserLoaded) {
      final currentUser = (state as UserLoaded).user;
      add(LoadUserProfile(currentUser.id));
    }
  }
}
```

## Design Patterns

### Singleton Pattern
```dart
// core/services/storage_service.dart
class StorageService {
  static StorageService? _instance;
  late SharedPreferences _prefs;
  
  StorageService._();
  
  static StorageService get instance {
    _instance ??= StorageService._();
    return _instance!;
  }
  
  Future<void> init() async {
    _prefs = await SharedPreferences.getInstance();
  }
  
  Future<void> setString(String key, String value) async {
    await _prefs.setString(key, value);
  }
  
  String? getString(String key) {
    return _prefs.getString(key);
  }
  
  Future<void> setBool(String key, bool value) async {
    await _prefs.setBool(key, value);
  }
  
  bool? getBool(String key) {
    return _prefs.getBool(key);
  }
  
  Future<void> remove(String key) async {
    await _prefs.remove(key);
  }
  
  Future<void> clear() async {
    await _prefs.clear();
  }
}
```

### Factory Pattern
```dart
// data/models/response_model.dart
abstract class ResponseModel<T> {
  final bool success;
  final String? message;
  final T? data;
  final String? error;
  
  ResponseModel({
    required this.success,
    this.message,
    this.data,
    this.error,
  });
  
  factory ResponseModel.fromJson(
    Map<String, dynamic> json,
    T Function(dynamic) fromJsonT,
  ) {
    return _ResponseModelImpl<T>(
      success: json['success'] ?? false,
      message: json['message'],
      data: json['data'] != null ? fromJsonT(json['data']) : null,
      error: json['error'],
    );
  }
}

class _ResponseModelImpl<T> extends ResponseModel<T> {
  _ResponseModelImpl({
    required bool success,
    String? message,
    T? data,
    String? error,
  }) : super(
          success: success,
          message: message,
          data: data,
          error: error,
        );
}

// Usage
final response = ResponseModel<User>.fromJson(
  jsonData,
  (data) => User.fromJson(data),
);
```

### Observer Pattern
```dart
// core/services/notification_service.dart
class NotificationService {
  static final NotificationService _instance = NotificationService._();
  factory NotificationService() => _instance;
  NotificationService._();
  
  final List<NotificationObserver> _observers = [];
  
  void addObserver(NotificationObserver observer) {
    _observers.add(observer);
  }
  
  void removeObserver(NotificationObserver observer) {
    _observers.remove(observer);
  }
  
  void notify(String message, NotificationType type) {
    for (final observer in _observers) {
      observer.onNotification(message, type);
    }
  }
}

abstract class NotificationObserver {
  void onNotification(String message, NotificationType type);
}

enum NotificationType {
  info,
  success,
  warning,
  error,
}

// Implementation
class HomePage extends StatefulWidget implements NotificationObserver {
  @override
  _HomePageState createState() => _HomePageState();
  
  @override
  void onNotification(String message, NotificationType type) {
    // Handle notification
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message)),
    );
  }
}
```

## Best Practices

### Dependency Injection
```dart
// core/di/injection_container.dart
final sl = GetIt.instance;

Future<void> init() async {
  //! Features - Authentication
  // Bloc
  sl.registerFactory(
    () => AuthBloc(
      signIn: sl(),
      signUp: sl(),
      signOut: sl(),
    ),
  );
  
  // Use cases
  sl.registerLazySingleton(() => SignIn(sl()));
  sl.registerLazySingleton(() => SignUp(sl()));
  sl.registerLazySingleton(() => SignOut(sl()));
  
  // Repository
  sl.registerLazySingleton<AuthRepository>(
    () => AuthRepositoryImpl(
      remoteDataSource: sl(),
      localDataSource: sl(),
      networkInfo: sl(),
    ),
  );
  
  // Data sources
  sl.registerLazySingleton<AuthRemoteDataSource>(
    () => AuthRemoteDataSourceImpl(client: sl()),
  );
  
  sl.registerLazySingleton<AuthLocalDataSource>(
    () => AuthLocalDataSourceImpl(sharedPreferences: sl()),
  );
  
  //! Core
  sl.registerLazySingleton<NetworkInfo>(
    () => NetworkInfoImpl(connectionChecker: sl()),
  );
  
  //! External
  final sharedPreferences = await SharedPreferences.getInstance();
  sl.registerLazySingleton(() => sharedPreferences);
  
  sl.registerLazySingleton(() => http.Client());
  
  sl.registerLazySingleton(() => InternetConnectionChecker());
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await init();
  runApp(MyApp());
}
```

### Code Style Guidelines
```dart
// 1. Use descriptive naming
class UserProfileUpdateRequest {
  final String firstName;
  final String lastName;
  final String email;
  
  // Good: Descriptive constructor
  UserProfileUpdateRequest.updateProfile({
    required this.firstName,
    required this.lastName,
    required this.email,
  });
}

// 2. Use const constructors when possible
class AppColors {
  static const Color primaryColor = Color(0xFF2196F3);
  static const Color secondaryColor = Color(0xFFFFC107);
  static const Color errorColor = Color(0xFFF44336);
  static const Color successColor = Color(0xFF4CAF50);
}

// 3. Handle null safety properly
class UserService {
  User? _currentUser;
  
  User get currentUser {
    final user = _currentUser;
    if (user == null) {
      throw StateError('No user is currently logged in');
    }
    return user;
  }
  
  bool get isLoggedIn => _currentUser != null;
}

// 4. Use extension methods for utility functions
extension StringExtensions on String {
  bool get isValidEmail {
    return RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(this);
  }
  
  String get capitalizeFirst {
    if (isEmpty) return this;
    return this[0].toUpperCase() + substring(1);
  }
}

extension DateTimeExtensions on DateTime {
  String get timeAgo {
    final now = DateTime.now();
    final difference = now.difference(this);
    
    if (difference.inDays > 0) {
      return '${difference.inDays} days ago';
    } else if (difference.inHours > 0) {
      return '${difference.inHours} hours ago';
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes} minutes ago';
    } else {
      return 'Just now';
    }
  }
}

// 5. Use sealed classes for state management
@freezed
class AuthState with _$AuthState {
  const factory AuthState.initial() = _Initial;
  const factory AuthState.loading() = _Loading;
  const factory AuthState.authenticated(User user) = _Authenticated;
  const factory AuthState.unauthenticated() = _Unauthenticated;
  const factory AuthState.error(String message) = _Error;
}
```

### Performance Tips
```dart
// 1. Use const widgets
class PerformantWidget extends StatelessWidget {
  const PerformantWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: const [
        Text('Static text'), // const by default
        SizedBox(height: 16), // const by default
        Icon(Icons.star), // const by default
      ],
    );
  }
}

// 2. Use RepaintBoundary for expensive widgets
class ExpensiveChart extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      child: CustomPaint(
        painter: ChartPainter(),
        size: Size(300, 200),
      ),
    );
  }
}

// 3. Use AutomaticKeepAliveClientMixin for tabs
class TabContent extends StatefulWidget {
  @override
  _TabContentState createState() => _TabContentState();
}

class _TabContentState extends State<TabContent>
    with AutomaticKeepAliveClientMixin {
  
  @override
  bool get wantKeepAlive => true;
  
  @override
  Widget build(BuildContext context) {
    super.build(context); // Required for AutomaticKeepAliveClientMixin
    return ExpensiveWidget();
  }
}
```

This comprehensive guide covers project architecture, folder organization, code patterns, and best practices for building maintainable and scalable Flutter applications.