---
layout: default
title: Authentication
nav_order: 2
---

# 🔐 Authentication & Authorization System

**Document:** Authentication API Specification  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Authentication Requirements

### **Current Flutter App State**
- **Local Storage**: Uses SharedPreferences for login status
- **Session Management**: Simple boolean `isLoggedIn` flag
- **User Data**: Minimal user information stored locally
- **Security**: No token-based authentication currently

### **Target Authentication System**
- **JWT-based Authentication** with access and refresh tokens
- **Secure Token Storage** using FlutterSecureStorage
- **Role-based Access Control** (User, Premium User, Admin)
- **Session Management** with automatic token refresh
- **Device Registration** for security monitoring

---

## 🔑 Authentication Endpoints

### **POST /auth/register**
Create a new user account.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "SecurePassword123!",
  "device_info": {
    "device_id": "unique-device-identifier",
    "platform": "android",
    "app_version": "1.0.0",
    "os_version": "Android 12"
  }
}
```

**Success Response (201 Created):**
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

**Error Responses:**
```json
// 400 Bad Request - Invalid data
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "details": {
    "email": ["Must be a valid email address"],
    "password": ["Password must be at least 8 characters long"]
  }
}

// 409 Conflict - Email already exists
{
  "success": false,
  "error": "EMAIL_EXISTS",
  "message": "An account with this email already exists"
}
```

---

### **POST /auth/login**
Authenticate user and receive JWT tokens.

**Request Body:**
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePassword123!",
  "device_info": {
    "device_id": "unique-device-identifier",
    "platform": "android",
    "app_version": "1.0.0",
    "os_version": "Android 12"
  },
  "remember_me": true
}
```

**Success Response (200 OK):**
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

**Error Responses:**
```json
// 401 Unauthorized - Invalid credentials
{
  "success": false,
  "error": "INVALID_CREDENTIALS",
  "message": "Invalid email or password"
}

// 423 Locked - Account locked
{
  "success": false,
  "error": "ACCOUNT_LOCKED",
  "message": "Account temporarily locked due to multiple failed login attempts",
  "details": {
    "unlock_time": "2025-07-15T11:00:00Z",
    "remaining_minutes": 15
  }
}
```

---

### **POST /auth/refresh**
Refresh access token using refresh token.

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Success Response (200 OK):**
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

**Error Responses:**
```json
// 401 Unauthorized - Invalid refresh token
{
  "success": false,
  "error": "INVALID_REFRESH_TOKEN",
  "message": "Refresh token is invalid or expired"
}
```

---

### **POST /auth/logout**
Logout user and invalidate tokens.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "logout_all_devices": false
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

### **POST /auth/forgot-password**
Request password reset email.

**Request Body:**
```json
{
  "email": "john.doe@example.com"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "If an account with this email exists, a password reset link has been sent"
}
```

---

### **POST /auth/reset-password**
Reset password using reset token.

**Request Body:**
```json
{
  "reset_token": "password-reset-token-from-email",
  "new_password": "NewSecurePassword123!"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset successful"
}
```

---

### **POST /auth/verify-email**
Verify email address using verification token.

**Request Body:**
```json
{
  "verification_token": "email-verification-token"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Email verified successfully",
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email_verified": true
    }
  }
}
```

---

### **GET /auth/me**
Get current user information.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Success Response (200 OK):**
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

## 🔐 JWT Token Structure

### **Access Token Payload**
```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john.doe@example.com",
  "role": "user",
  "device_id": "unique-device-identifier",
  "iat": 1642247400,
  "exp": 1642251000,
  "iss": "speechbot-api",
  "aud": "speechbot-app"
}
```

### **Refresh Token Payload**
```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "type": "refresh",
  "device_id": "unique-device-identifier",
  "iat": 1642247400,
  "exp": 1644839400,
  "iss": "speechbot-api",
  "aud": "speechbot-app"
}
```

---

## 🛡️ Security Features

### **Password Requirements**
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character
- Cannot be a common password
- Cannot contain user's email or name

### **Account Security**
- **Rate Limiting**: 5 failed login attempts per 15 minutes
- **Device Tracking**: Monitor login attempts from new devices
- **Session Management**: Automatic logout after 30 days of inactivity
- **Token Rotation**: Refresh tokens expire after 30 days
- **Password History**: Cannot reuse last 5 passwords

### **Device Management**
```json
// GET /auth/devices - List user's active devices
{
  "success": true,
  "data": {
    "devices": [
      {
        "id": "device-1",
        "name": "Samsung Galaxy S21",
        "platform": "android",
        "last_active": "2025-07-15T10:30:00Z",
        "is_current": true,
        "location": "New York, NY"
      },
      {
        "id": "device-2",
        "name": "iPhone 13",
        "platform": "ios",
        "last_active": "2025-07-14T15:20:00Z",
        "is_current": false,
        "location": "Los Angeles, CA"
      }
    ]
  }
}

// DELETE /auth/devices/{device_id} - Logout specific device
{
  "success": true,
  "message": "Device logged out successfully"
}
```

---

## 📱 Flutter Integration

### **Authentication Service**
```dart
class AuthService {
  static const String _tokenKey = 'access_token';
  static const String _refreshKey = 'refresh_token';
  static const String _userKey = 'user_data';
  
  // Login method
  static Future<LoginResult> login(String email, String password) async {
    try {
      final deviceInfo = await _getDeviceInfo();
      final response = await dio.post('/auth/login', data: {
        'email': email,
        'password': password,
        'device_info': deviceInfo,
        'remember_me': true,
      });
      
      final result = LoginResult.fromJson(response.data);
      await _storeAuthData(result);
      return result;
    } catch (e) {
      throw AuthException.fromDioError(e);
    }
  }
  
  // Auto-login check
  static Future<bool> isLoggedIn() async {
    final token = await _getStoredToken();
    if (token == null) return false;
    
    try {
      await dio.get('/auth/me', options: Options(
        headers: {'Authorization': 'Bearer $token'}
      ));
      return true;
    } catch (e) {
      // Try refresh token
      return await _refreshToken();
    }
  }
  
  // Token refresh
  static Future<bool> _refreshToken() async {
    final refreshToken = await _getRefreshToken();
    if (refreshToken == null) return false;
    
    try {
      final response = await dio.post('/auth/refresh', data: {
        'refresh_token': refreshToken,
      });
      
      final tokens = TokenPair.fromJson(response.data['data']['tokens']);
      await _storeTokens(tokens);
      return true;
    } catch (e) {
      await clearAuthData();
      return false;
    }
  }
}
```

### **Authentication State Management**
```dart
class AuthProvider extends ChangeNotifier {
  User? _user;
  bool _isAuthenticated = false;
  bool _isLoading = true;
  
  User? get user => _user;
  bool get isAuthenticated => _isAuthenticated;
  bool get isLoading => _isLoading;
  
  Future<void> initialize() async {
    _isLoading = true;
    notifyListeners();
    
    final isLoggedIn = await AuthService.isLoggedIn();
    if (isLoggedIn) {
      _user = await AuthService.getCurrentUser();
      _isAuthenticated = true;
    }
    
    _isLoading = false;
    notifyListeners();
  }
  
  Future<void> login(String email, String password) async {
    final result = await AuthService.login(email, password);
    _user = result.user;
    _isAuthenticated = true;
    notifyListeners();
  }
  
  Future<void> logout() async {
    await AuthService.logout();
    _user = null;
    _isAuthenticated = false;
    notifyListeners();
  }
}
```

---

## 🔒 Error Handling

### **Authentication Errors**
```dart
class AuthException implements Exception {
  final String code;
  final String message;
  final Map<String, dynamic>? details;
  
  const AuthException(this.code, this.message, [this.details]);
  
  factory AuthException.fromDioError(DioError error) {
    final response = error.response;
    if (response != null) {
      final data = response.data;
      return AuthException(
        data['error'] ?? 'UNKNOWN_ERROR',
        data['message'] ?? 'An unknown error occurred',
        data['details'],
      );
    }
    
    return AuthException('NETWORK_ERROR', 'Network connection failed');
  }
  
  bool get isNetworkError => code == 'NETWORK_ERROR';
  bool get isValidationError => code == 'VALIDATION_ERROR';
  bool get isCredentialsError => code == 'INVALID_CREDENTIALS';
  bool get isAccountLocked => code == 'ACCOUNT_LOCKED';
}
```

---

**This authentication system provides secure, scalable user management while integrating seamlessly with the existing Flutter app architecture.**
