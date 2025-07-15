---
layout: default
title: Authentication Examples
parent: Examples
nav_order: 1
---

# 🔐 Authentication Examples

**Document:** Authentication API Usage Examples  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Flutter Authentication Implementation Examples

### **User Registration Example**
```dart
// lib/screens/signup_screen.dart
class SignupScreen extends StatefulWidget {
  @override
  _SignupScreenState createState() => _SignupScreenState();
}

class _SignupScreenState extends State<SignupScreen> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  String? _errorMessage;

  Future<void> _handleSignup() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() {
      _isLoading = true;
      _errorMessage = null;
    });

    try {
      final authProvider = Provider.of<AuthProvider>(context, listen: false);
      
      await authProvider.register(
        name: _nameController.text.trim(),
        email: _emailController.text.trim(),
        password: _passwordController.text,
      );

      // Navigate to onboarding or main app
      Navigator.of(context).pushReplacementNamed('/onboarding');
      
    } catch (e) {
      setState(() {
        _errorMessage = e is AuthException 
            ? e.userFriendlyMessage 
            : 'Registration failed. Please try again.';
      });
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Create Account')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: InputDecoration(
                  labelText: 'Full Name',
                  prefixIcon: Icon(Icons.person),
                ),
                validator: (value) {
                  if (value?.isEmpty ?? true) {
                    return 'Name is required';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: InputDecoration(
                  labelText: 'Email',
                  prefixIcon: Icon(Icons.email),
                ),
                validator: (value) {
                  if (value?.isEmpty ?? true) {
                    return 'Email is required';
                  }
                  if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value!)) {
                    return 'Please enter a valid email';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              TextFormField(
                controller: _passwordController,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: 'Password',
                  prefixIcon: Icon(Icons.lock),
                ),
                validator: (value) {
                  if (value?.isEmpty ?? true) {
                    return 'Password is required';
                  }
                  if (value!.length < 8) {
                    return 'Password must be at least 8 characters';
                  }
                  return null;
                },
              ),
              
              if (_errorMessage != null) ...[
                SizedBox(height: 16),
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.red.shade50,
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.red.shade200),
                  ),
                  child: Row(
                    children: [
                      Icon(Icons.error, color: Colors.red),
                      SizedBox(width: 8),
                      Expanded(child: Text(_errorMessage!, style: TextStyle(color: Colors.red))),
                    ],
                  ),
                ),
              ],
              
              SizedBox(height: 24),
              
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _handleSignup,
                  child: _isLoading
                      ? CircularProgressIndicator(color: Colors.white)
                      : Text('Create Account'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### **User Login Example**
```dart
// lib/screens/login_screen.dart
class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  bool _rememberMe = true;
  String? _errorMessage;

  Future<void> _handleLogin() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() {
      _isLoading = true;
      _errorMessage = null;
    });

    try {
      final authProvider = Provider.of<AuthProvider>(context, listen: false);
      
      await authProvider.login(
        email: _emailController.text.trim(),
        password: _passwordController.text,
        rememberMe: _rememberMe,
      );

      // Navigate based on user state
      final user = authProvider.user;
      if (user?.preferences?.onboardingCompleted == false) {
        Navigator.of(context).pushReplacementNamed('/onboarding');
      } else {
        Navigator.of(context).pushReplacementNamed('/chat');
      }
      
    } catch (e) {
      setState(() {
        if (e is AuthException) {
          switch (e.code) {
            case 'INVALID_CREDENTIALS':
              _errorMessage = 'Invalid email or password';
              break;
            case 'ACCOUNT_LOCKED':
              _errorMessage = 'Account temporarily locked. Please try again later.';
              break;
            case 'NETWORK_ERROR':
              _errorMessage = 'Network connection failed. Please check your internet connection.';
              break;
            default:
              _errorMessage = e.userFriendlyMessage;
          }
        } else {
          _errorMessage = 'Login failed. Please try again.';
        }
      });
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Sign In')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text(
                'Welcome Back!',
                style: Theme.of(context).textTheme.headlineMedium,
              ),
              
              SizedBox(height: 32),
              
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: InputDecoration(
                  labelText: 'Email',
                  prefixIcon: Icon(Icons.email),
                ),
                validator: (value) {
                  if (value?.isEmpty ?? true) {
                    return 'Email is required';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              TextFormField(
                controller: _passwordController,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: 'Password',
                  prefixIcon: Icon(Icons.lock),
                ),
                validator: (value) {
                  if (value?.isEmpty ?? true) {
                    return 'Password is required';
                  }
                  return null;
                },
              ),
              
              SizedBox(height: 16),
              
              Row(
                children: [
                  Checkbox(
                    value: _rememberMe,
                    onChanged: (value) {
                      setState(() {
                        _rememberMe = value ?? true;
                      });
                    },
                  ),
                  Text('Remember me'),
                  Spacer(),
                  TextButton(
                    onPressed: () {
                      Navigator.of(context).pushNamed('/forgot-password');
                    },
                    child: Text('Forgot Password?'),
                  ),
                ],
              ),
              
              if (_errorMessage != null) ...[
                SizedBox(height: 16),
                Container(
                  padding: EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.red.shade50,
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.red.shade200),
                  ),
                  child: Row(
                    children: [
                      Icon(Icons.error, color: Colors.red),
                      SizedBox(width: 8),
                      Expanded(child: Text(_errorMessage!, style: TextStyle(color: Colors.red))),
                    ],
                  ),
                ),
              ],
              
              SizedBox(height: 24),
              
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _handleLogin,
                  child: _isLoading
                      ? CircularProgressIndicator(color: Colors.white)
                      : Text('Sign In'),
                ),
              ),
              
              SizedBox(height: 16),
              
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text("Don't have an account? "),
                  TextButton(
                    onPressed: () {
                      Navigator.of(context).pushNamed('/signup');
                    },
                    child: Text('Sign Up'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### **Authentication Provider Example**
```dart
// lib/providers/auth_provider.dart
class AuthProvider extends ChangeNotifier {
  User? _user;
  bool _isAuthenticated = false;
  bool _isLoading = true;
  AuthException? _error;

  User? get user => _user;
  bool get isAuthenticated => _isAuthenticated;
  bool get isLoading => _isLoading;
  AuthException? get error => _error;

  Future<void> initialize() async {
    _isLoading = true;
    notifyListeners();

    try {
      final isLoggedIn = await AuthService.isLoggedIn();
      if (isLoggedIn) {
        _user = await AuthService.getCurrentUser();
        _isAuthenticated = true;
      }
    } catch (e) {
      print('Auth initialization error: $e');
      await AuthService.clearAuthData();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  Future<void> register({
    required String name,
    required String email,
    required String password,
  }) async {
    try {
      _error = null;
      
      final result = await AuthService.register(name, email, password);
      _user = result.user;
      _isAuthenticated = true;
      
      notifyListeners();
    } catch (e) {
      _error = e is AuthException ? e : AuthException.fromError(e);
      notifyListeners();
      rethrow;
    }
  }

  Future<void> login({
    required String email,
    required String password,
    bool rememberMe = true,
  }) async {
    try {
      _error = null;
      
      final result = await AuthService.login(email, password, rememberMe: rememberMe);
      _user = result.user;
      _isAuthenticated = true;
      
      // Initialize user preferences
      await Provider.of<PreferencesProvider>(Get.context!, listen: false).initialize();
      
      notifyListeners();
    } catch (e) {
      _error = e is AuthException ? e : AuthException.fromError(e);
      notifyListeners();
      rethrow;
    }
  }

  Future<void> logout() async {
    try {
      await AuthService.logout();
    } catch (e) {
      print('Logout error: $e');
    } finally {
      _user = null;
      _isAuthenticated = false;
      _error = null;
      
      // Clear other providers
      Provider.of<ChatProvider>(Get.context!, listen: false).clear();
      Provider.of<PreferencesProvider>(Get.context!, listen: false).clear();
      
      notifyListeners();
    }
  }

  Future<void> refreshToken() async {
    try {
      final success = await AuthService.refreshToken();
      if (!success) {
        await logout();
      }
    } catch (e) {
      print('Token refresh error: $e');
      await logout();
    }
  }

  void clearError() {
    _error = null;
    notifyListeners();
  }
}
```

### **Secure Token Storage Example**
```dart
// lib/services/secure_storage_service.dart
class SecureStorageService {
  static const FlutterSecureStorage _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: IOSAccessibility.first_unlock_this_device,
    ),
  );

  static Future<void> storeTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    try {
      await _storage.write(key: 'access_token', value: accessToken);
      await _storage.write(key: 'refresh_token', value: refreshToken);
      await _storage.write(
        key: 'token_timestamp', 
        value: DateTime.now().millisecondsSinceEpoch.toString(),
      );
    } catch (e) {
      throw StorageException('Failed to store authentication tokens: $e');
    }
  }

  static Future<String?> getAccessToken() async {
    try {
      return await _storage.read(key: 'access_token');
    } catch (e) {
      print('Error reading access token: $e');
      return null;
    }
  }

  static Future<String?> getRefreshToken() async {
    try {
      return await _storage.read(key: 'refresh_token');
    } catch (e) {
      print('Error reading refresh token: $e');
      return null;
    }
  }

  static Future<bool> hasValidTokens() async {
    final accessToken = await getAccessToken();
    final refreshToken = await getRefreshToken();
    final timestamp = await _storage.read(key: 'token_timestamp');

    if (accessToken == null || refreshToken == null || timestamp == null) {
      return false;
    }

    // Check if tokens are not too old (30 days)
    final tokenTime = DateTime.fromMillisecondsSinceEpoch(int.parse(timestamp));
    final thirtyDaysAgo = DateTime.now().subtract(Duration(days: 30));

    return tokenTime.isAfter(thirtyDaysAgo);
  }

  static Future<void> clearTokens() async {
    try {
      await _storage.delete(key: 'access_token');
      await _storage.delete(key: 'refresh_token');
      await _storage.delete(key: 'token_timestamp');
    } catch (e) {
      print('Error clearing tokens: $e');
    }
  }

  static Future<void> clearAllSecureData() async {
    try {
      await _storage.deleteAll();
    } catch (e) {
      print('Error clearing all secure data: $e');
    }
  }
}
```

---

## 🌐 HTTP Request Examples

### **Registration Request**
```bash
curl -X POST https://api.speechbot.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john.doe@example.com",
    "password": "SecurePassword123!",
    "device_info": {
      "device_id": "android_device_123",
      "platform": "android",
      "app_version": "1.0.0",
      "os_version": "Android 12"
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Account created successfully",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "email_verified": false,
      "role": "user",
      "created_at": "2025-07-15T10:30:00Z"
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 3600
    }
  }
}
```

### **Login Request**
```bash
curl -X POST https://api.speechbot.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePassword123!",
    "device_info": {
      "device_id": "android_device_123",
      "platform": "android",
      "app_version": "1.0.0",
      "os_version": "Android 12"
    },
    "remember_me": true
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "email_verified": true,
      "role": "user",
      "last_login": "2025-07-15T10:30:00Z",
      "preferences": {
        "theme_dark_mode": true,
        "chat_text_animations": true,
        "chat_font_size": 16.0,
        "chat_persona": "Friendly",
        "notifications_enabled": true,
        "onboarding_completed": true
      }
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 3600
    }
  }
}
```

### **Token Refresh Request**
```bash
curl -X POST https://api.speechbot.com/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 3600
    }
  }
}
```

### **Get Current User Request**
```bash
curl -X GET https://api.speechbot.com/v1/auth/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "email_verified": true,
      "role": "user",
      "created_at": "2025-07-15T10:30:00Z",
      "last_login": "2025-07-15T10:30:00Z",
      "preferences": {
        "theme_dark_mode": true,
        "chat_text_animations": true,
        "chat_font_size": 16.0,
        "chat_persona": "Friendly",
        "notifications_enabled": true,
        "onboarding_completed": true
      }
    }
  }
}
```

---

## 🔒 Error Handling Examples

### **Authentication Error Handling**
```dart
// lib/services/auth_service.dart
class AuthService {
  static Future<LoginResult> login(String email, String password, {bool rememberMe = true}) async {
    try {
      final response = await ApiService.post('/auth/login', data: {
        'email': email,
        'password': password,
        'device_info': await _getDeviceInfo(),
        'remember_me': rememberMe,
      });
      
      return LoginResult.fromJson(response.data);
    } on DioError catch (e) {
      if (e.response != null) {
        final data = e.response!.data;
        switch (e.response!.statusCode) {
          case 400:
            throw AuthException(
              code: data['error'] ?? 'VALIDATION_ERROR',
              message: data['message'] ?? 'Invalid input data',
              details: data['details'],
            );
          case 401:
            throw AuthException(
              code: data['error'] ?? 'INVALID_CREDENTIALS',
              message: data['message'] ?? 'Invalid email or password',
            );
          case 423:
            throw AuthException(
              code: data['error'] ?? 'ACCOUNT_LOCKED',
              message: data['message'] ?? 'Account temporarily locked',
              details: data['details'],
            );
          case 429:
            throw AuthException(
              code: data['error'] ?? 'RATE_LIMIT_EXCEEDED',
              message: data['message'] ?? 'Too many login attempts',
              details: data['details'],
            );
          default:
            throw AuthException(
              code: 'UNKNOWN_ERROR',
              message: 'An unexpected error occurred',
            );
        }
      } else {
        // Network error
        throw AuthException(
          code: 'NETWORK_ERROR',
          message: 'Network connection failed',
        );
      }
    }
  }
}
```

### **Auto-Retry with Exponential Backoff**
```dart
// lib/services/retry_service.dart
class RetryService {
  static Future<T> executeWithRetry<T>(
    Future<T> Function() operation, {
    int maxRetries = 3,
    Duration initialDelay = const Duration(seconds: 1),
    double backoffMultiplier = 2.0,
    bool Function(dynamic error)? shouldRetry,
  }) async {
    int attempt = 0;
    Duration delay = initialDelay;

    while (attempt < maxRetries) {
      try {
        return await operation();
      } catch (error) {
        attempt++;
        
        // Check if we should retry this error
        if (shouldRetry != null && !shouldRetry(error)) {
          rethrow;
        }
        
        // Don't retry on last attempt
        if (attempt >= maxRetries) {
          rethrow;
        }
        
        // Wait before retrying
        await Future.delayed(delay);
        delay = Duration(milliseconds: (delay.inMilliseconds * backoffMultiplier).round());
      }
    }

    throw Exception('Max retries exceeded');
  }
}

// Usage example
Future<LoginResult> loginWithRetry(String email, String password) async {
  return await RetryService.executeWithRetry(
    () => AuthService.login(email, password),
    maxRetries: 3,
    shouldRetry: (error) {
      if (error is AuthException) {
        // Retry on network errors and server errors, but not on client errors
        return error.code == 'NETWORK_ERROR' || error.statusCode >= 500;
      }
      return false;
    },
  );
}
```

---

**These examples provide complete, production-ready authentication implementation patterns that can be directly integrated into the SpeechBot Flutter application.**
