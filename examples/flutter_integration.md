# 🔧 Flutter Integration Examples

**Document:** Complete Flutter Integration Guide  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Complete Flutter App Structure

### **Main Application Setup**
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'providers/auth_provider.dart';
import 'providers/chat_provider.dart';
import 'providers/preferences_provider.dart';
import 'providers/theme_provider.dart';
import 'services/notification_service.dart';
import 'services/websocket_service.dart';
import 'screens/splash_screen.dart';
import 'screens/login_screen.dart';
import 'screens/signup_screen.dart';
import 'screens/onboarding_screen.dart';
import 'screens/chat_screen.dart';
import 'screens/preferences_screen.dart';
import 'utils/app_routes.dart';
import 'utils/app_theme.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize services
  await NotificationService.initialize();
  
  runApp(SpeechBotApp());
}

class SpeechBotApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        ChangeNotifierProvider(create: (_) => PreferencesProvider()),
        ChangeNotifierProvider(create: (_) => ChatProvider()),
        ChangeNotifierProxyProvider<PreferencesProvider, ThemeProvider>(
          create: (context) => ThemeProvider(),
          update: (context, preferences, theme) => theme!..updateFromPreferences(preferences),
        ),
      ],
      child: Consumer<ThemeProvider>(
        builder: (context, themeProvider, child) {
          return MaterialApp(
            title: 'SpeechBot',
            debugShowCheckedModeBanner: false,
            
            // Theme configuration
            theme: AppTheme.lightTheme,
            darkTheme: AppTheme.darkTheme,
            themeMode: themeProvider.themeMode,
            
            // Navigation
            initialRoute: AppRoutes.splash,
            routes: AppRoutes.routes,
            
            // App initialization
            home: AppInitializer(),
          );
        },
      ),
    );
  }
}

class AppInitializer extends StatefulWidget {
  @override
  _AppInitializerState createState() => _AppInitializerState();
}

class _AppInitializerState extends State<AppInitializer> {
  bool _isInitialized = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _initializeApp();
  }

  Future<void> _initializeApp() async {
    try {
      // Initialize providers in sequence
      final authProvider = Provider.of<AuthProvider>(context, listen: false);
      final preferencesProvider = Provider.of<PreferencesProvider>(context, listen: false);
      final chatProvider = Provider.of<ChatProvider>(context, listen: false);

      // Initialize authentication first
      await authProvider.initialize();
      
      // Initialize preferences
      await preferencesProvider.initialize();
      
      // Initialize chat if user is authenticated
      if (authProvider.isAuthenticated) {
        await chatProvider.initialize();
        
        // Start WebSocket connection
        await WebSocketService().connect();
      }

      setState(() {
        _isInitialized = true;
      });

      // Navigate to appropriate screen
      _navigateToAppropriateScreen();

    } catch (e) {
      setState(() {
        _error = e.toString();
      });
    }
  }

  void _navigateToAppropriateScreen() {
    final authProvider = Provider.of<AuthProvider>(context, listen: false);
    final preferencesProvider = Provider.of<PreferencesProvider>(context, listen: false);

    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (authProvider.isAuthenticated) {
        if (preferencesProvider.preferences?.onboardingCompleted == true) {
          Navigator.of(context).pushReplacementNamed(AppRoutes.chat);
        } else {
          Navigator.of(context).pushReplacementNamed(AppRoutes.onboarding);
        }
      } else {
        Navigator.of(context).pushReplacementNamed(AppRoutes.welcome);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    if (_error != null) {
      return Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error, size: 64, color: Colors.red),
              SizedBox(height: 16),
              Text('Initialization failed'),
              SizedBox(height: 8),
              Text(_error!, style: TextStyle(color: Colors.grey)),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: () {
                  setState(() {
                    _error = null;
                    _isInitialized = false;
                  });
                  _initializeApp();
                },
                child: Text('Retry'),
              ),
            ],
          ),
        ),
      );
    }

    if (!_isInitialized) {
      return SplashScreen();
    }

    // This should not be reached due to navigation above
    return Container();
  }
}
```

### **HTTP Service Implementation**
```dart
// lib/services/api_service.dart
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class ApiService {
  static const String BASE_URL = 'https://api.speechbot.com/v1';
  static late Dio _dio;
  static const FlutterSecureStorage _storage = FlutterSecureStorage();

  static void initialize() {
    _dio = Dio(BaseOptions(
      baseUrl: BASE_URL,
      connectTimeout: Duration(seconds: 30),
      receiveTimeout: Duration(seconds: 30),
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    ));

    // Add interceptors
    _dio.interceptors.add(AuthInterceptor());
    _dio.interceptors.add(LoggingInterceptor());
    _dio.interceptors.add(ErrorInterceptor());
  }

  // HTTP Methods
  static Future<Response> get(String path, {Map<String, dynamic>? queryParameters}) {
    return _dio.get(path, queryParameters: queryParameters);
  }

  static Future<Response> post(String path, {dynamic data}) {
    return _dio.post(path, data: data);
  }

  static Future<Response> patch(String path, {dynamic data}) {
    return _dio.patch(path, data: data);
  }

  static Future<Response> delete(String path) {
    return _dio.delete(path);
  }

  static Future<Response> put(String path, {dynamic data}) {
    return _dio.put(path, data: data);
  }
}

class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    // Add authentication token to requests
    final token = await _storage.read(key: 'access_token');
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    
    handler.next(options);
  }

  @override
  void onError(DioError err, ErrorInterceptorHandler handler) async {
    // Handle token refresh on 401 errors
    if (err.response?.statusCode == 401) {
      try {
        final refreshed = await _refreshToken();
        if (refreshed) {
          // Retry the original request
          final response = await _retry(err.requestOptions);
          handler.resolve(response);
          return;
        }
      } catch (e) {
        // Refresh failed, redirect to login
        await _handleLogout();
      }
    }
    
    handler.next(err);
  }

  Future<bool> _refreshToken() async {
    try {
      final refreshToken = await _storage.read(key: 'refresh_token');
      if (refreshToken == null) return false;

      final response = await Dio().post(
        '${ApiService.BASE_URL}/auth/refresh',
        data: {'refresh_token': refreshToken},
      );

      if (response.statusCode == 200) {
        final data = response.data['data'];
        await _storage.write(key: 'access_token', data['tokens']['access_token']);
        await _storage.write(key: 'refresh_token', data['tokens']['refresh_token']);
        return true;
      }
    } catch (e) {
      print('Token refresh failed: $e');
    }
    
    return false;
  }

  Future<Response> _retry(RequestOptions options) async {
    final token = await _storage.read(key: 'access_token');
    options.headers['Authorization'] = 'Bearer $token';
    
    return await Dio().fetch(options);
  }

  Future<void> _handleLogout() async {
    await _storage.deleteAll();
    // Navigate to login screen
    // This should be handled by the AuthProvider
  }
}

class LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('🚀 ${options.method} ${options.uri}');
    if (options.data != null) {
      print('📤 Request Data: ${options.data}');
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('✅ ${response.statusCode} ${response.requestOptions.uri}');
    print('📥 Response Data: ${response.data}');
    handler.next(response);
  }

  @override
  void onError(DioError err, ErrorInterceptorHandler handler) {
    print('❌ ${err.response?.statusCode} ${err.requestOptions.uri}');
    print('📥 Error Data: ${err.response?.data}');
    handler.next(err);
  }
}

class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioError err, ErrorInterceptorHandler handler) {
    String message;
    
    switch (err.type) {
      case DioErrorType.connectionTimeout:
      case DioErrorType.sendTimeout:
      case DioErrorType.receiveTimeout:
        message = 'Connection timeout. Please check your internet connection.';
        break;
      case DioErrorType.badResponse:
        message = _handleHttpError(err.response!);
        break;
      case DioErrorType.cancel:
        message = 'Request was cancelled';
        break;
      default:
        message = 'Network error occurred. Please try again.';
    }
    
    final error = ApiException(
      message: message,
      statusCode: err.response?.statusCode,
      data: err.response?.data,
    );
    
    handler.reject(DioError(
      requestOptions: err.requestOptions,
      error: error,
      type: err.type,
      response: err.response,
    ));
  }

  String _handleHttpError(Response response) {
    switch (response.statusCode) {
      case 400:
        return response.data['message'] ?? 'Invalid request';
      case 401:
        return 'Authentication failed';
      case 403:
        return 'Access denied';
      case 404:
        return 'Resource not found';
      case 429:
        return 'Too many requests. Please try again later.';
      case 500:
        return 'Server error. Please try again later.';
      default:
        return 'An unexpected error occurred';
    }
  }
}

class ApiException implements Exception {
  final String message;
  final int? statusCode;
  final dynamic data;

  ApiException({
    required this.message,
    this.statusCode,
    this.data,
  });

  @override
  String toString() => message;
}
```

### **State Management Integration**
```dart
// lib/providers/app_state_provider.dart
class AppStateProvider extends ChangeNotifier {
  bool _isOnline = true;
  bool _isSyncing = false;
  String? _globalError;
  Map<String, dynamic> _cache = {};

  bool get isOnline => _isOnline;
  bool get isSyncing => _isSyncing;
  String? get globalError => _globalError;

  void setOnlineStatus(bool online) {
    if (_isOnline != online) {
      _isOnline = online;
      notifyListeners();
      
      if (online) {
        _triggerSync();
      }
    }
  }

  void setSyncStatus(bool syncing) {
    if (_isSyncing != syncing) {
      _isSyncing = syncing;
      notifyListeners();
    }
  }

  void setGlobalError(String? error) {
    _globalError = error;
    notifyListeners();
  }

  void clearGlobalError() {
    _globalError = null;
    notifyListeners();
  }

  void cacheData(String key, dynamic data) {
    _cache[key] = data;
  }

  T? getCachedData<T>(String key) {
    return _cache[key] as T?;
  }

  void clearCache() {
    _cache.clear();
  }

  Future<void> _triggerSync() async {
    setSyncStatus(true);
    
    try {
      // Sync all providers
      final context = Get.context!;
      
      await Provider.of<AuthProvider>(context, listen: false).refreshToken();
      await Provider.of<PreferencesProvider>(context, listen: false).syncWithServer();
      await Provider.of<ChatProvider>(context, listen: false).syncWithServer();
      
    } catch (e) {
      print('Sync failed: $e');
    } finally {
      setSyncStatus(false);
    }
  }
}
```

### **Network Connectivity Monitoring**
```dart
// lib/services/connectivity_service.dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'dart:async';

class ConnectivityService {
  static final Connectivity _connectivity = Connectivity();
  static StreamSubscription<ConnectivityResult>? _connectivitySubscription;
  static bool _isOnline = true;

  static bool get isOnline => _isOnline;

  static Future<void> initialize() async {
    // Check initial connectivity
    final result = await _connectivity.checkConnectivity();
    _isOnline = result != ConnectivityResult.none;

    // Listen for connectivity changes
    _connectivitySubscription = _connectivity.onConnectivityChanged.listen(
      (ConnectivityResult result) {
        final wasOnline = _isOnline;
        _isOnline = result != ConnectivityResult.none;
        
        if (wasOnline != _isOnline) {
          _handleConnectivityChange(_isOnline);
        }
      },
    );
  }

  static void _handleConnectivityChange(bool isOnline) {
    print('Connectivity changed: ${isOnline ? 'Online' : 'Offline'}');
    
    // Update app state
    final appStateProvider = Get.find<AppStateProvider>();
    appStateProvider.setOnlineStatus(isOnline);
    
    if (isOnline) {
      // Reconnect WebSocket
      WebSocketService().reconnect();
      
      // Show online status
      Get.snackbar(
        'Connection Restored',
        'You are back online',
        snackPosition: SnackPosition.BOTTOM,
        backgroundColor: Colors.green,
        colorText: Colors.white,
        duration: Duration(seconds: 2),
      );
    } else {
      // Show offline status
      Get.snackbar(
        'Connection Lost',
        'You are currently offline',
        snackPosition: SnackPosition.BOTTOM,
        backgroundColor: Colors.orange,
        colorText: Colors.white,
        duration: Duration(seconds: 3),
      );
    }
  }

  static void dispose() {
    _connectivitySubscription?.cancel();
  }
}
```

### **Complete Error Handling**
```dart
// lib/utils/error_handler.dart
class ErrorHandler {
  static void handleError(dynamic error, {String? context}) {
    String message;
    String? details;
    
    if (error is ApiException) {
      message = error.message;
      details = error.data?.toString();
    } else if (error is AuthException) {
      message = error.userFriendlyMessage;
      details = error.details;
    } else if (error is WebSocketException) {
      message = 'Real-time connection failed';
      details = error.toString();
    } else if (error is StorageException) {
      message = 'Local storage error';
      details = error.toString();
    } else {
      message = 'An unexpected error occurred';
      details = error.toString();
    }
    
    // Log error
    _logError(error, context, message, details);
    
    // Show user-friendly error
    _showErrorToUser(message, details);
    
    // Report to crash analytics (if available)
    _reportToCrashlytics(error, context);
  }

  static void _logError(dynamic error, String? context, String message, String? details) {
    print('=== ERROR ===');
    print('Context: ${context ?? 'Unknown'}');
    print('Message: $message');
    print('Details: $details');
    print('Stack trace: ${StackTrace.current}');
    print('============');
  }

  static void _showErrorToUser(String message, String? details) {
    Get.snackbar(
      'Error',
      message,
      snackPosition: SnackPosition.BOTTOM,
      backgroundColor: Colors.red,
      colorText: Colors.white,
      duration: Duration(seconds: 4),
      mainButton: details != null ? TextButton(
        onPressed: () => _showErrorDetails(message, details),
        child: Text('Details', style: TextStyle(color: Colors.white)),
      ) : null,
    );
  }

  static void _showErrorDetails(String message, String details) {
    Get.dialog(
      AlertDialog(
        title: Text('Error Details'),
        content: SingleChildScrollView(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            mainAxisSize: MainAxisSize.min,
            children: [
              Text('Message:', style: TextStyle(fontWeight: FontWeight.bold)),
              Text(message),
              SizedBox(height: 16),
              Text('Details:', style: TextStyle(fontWeight: FontWeight.bold)),
              Text(details),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Get.back(),
            child: Text('Close'),
          ),
          TextButton(
            onPressed: () {
              Clipboard.setData(ClipboardData(text: '$message\n\n$details'));
              Get.back();
              Get.snackbar('Copied', 'Error details copied to clipboard');
            },
            child: Text('Copy'),
          ),
        ],
      ),
    );
  }

  static void _reportToCrashlytics(dynamic error, String? context) {
    // Implementation for crash reporting service
    // e.g., Firebase Crashlytics, Sentry, etc.
    try {
      // FirebaseCrashlytics.instance.recordError(error, stackTrace, context: context);
    } catch (e) {
      print('Failed to report crash: $e');
    }
  }
}
```

### **Loading States Management**
```dart
// lib/widgets/loading_overlay.dart
class LoadingOverlay extends StatelessWidget {
  final bool isLoading;
  final String? message;
  final Widget child;

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
            color: Colors.black.withOpacity(0.3),
            child: Center(
              child: Card(
                child: Padding(
                  padding: EdgeInsets.all(20),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      CircularProgressIndicator(),
                      if (message != null) ...[
                        SizedBox(height: 16),
                        Text(message!),
                      ],
                    ],
                  ),
                ),
              ),
            ),
          ),
      ],
    );
  }
}

// Usage in screens
class ExampleScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Consumer<ChatProvider>(
        builder: (context, chatProvider, child) {
          return LoadingOverlay(
            isLoading: chatProvider.isLoading,
            message: 'Loading chats...',
            child: YourScreenContent(),
          );
        },
      ),
    );
  }
}
```

### **App Routes Configuration**
```dart
// lib/utils/app_routes.dart
class AppRoutes {
  static const String splash = '/';
  static const String welcome = '/welcome';
  static const String login = '/login';
  static const String signup = '/signup';
  static const String forgotPassword = '/forgot-password';
  static const String onboarding = '/onboarding';
  static const String chat = '/chat';
  static const String preferences = '/preferences';
  static const String export = '/export';
  static const String profile = '/profile';

  static Map<String, WidgetBuilder> get routes {
    return {
      splash: (context) => SplashScreen(),
      welcome: (context) => WelcomeScreen(),
      login: (context) => LoginScreen(),
      signup: (context) => SignupScreen(),
      forgotPassword: (context) => ForgotPasswordScreen(),
      onboarding: (context) => OnboardingScreen(),
      chat: (context) => ChatScreen(),
      preferences: (context) => PreferencesScreen(),
      export: (context) => ExportScreen(),
      profile: (context) => ProfileScreen(),
    };
  }

  static void navigateToChat(BuildContext context) {
    Navigator.of(context).pushNamedAndRemoveUntil(
      chat,
      (route) => false,
    );
  }

  static void navigateToLogin(BuildContext context) {
    Navigator.of(context).pushNamedAndRemoveUntil(
      login,
      (route) => false,
    );
  }

  static void navigateToOnboarding(BuildContext context) {
    Navigator.of(context).pushNamedAndRemoveUntil(
      onboarding,
      (route) => false,
    );
  }
}
```

---

**This comprehensive Flutter integration provides a robust foundation for the SpeechBot application with proper state management, error handling, networking, and user experience considerations.**
