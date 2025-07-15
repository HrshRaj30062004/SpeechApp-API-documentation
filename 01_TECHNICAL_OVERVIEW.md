---
layout: default
title: Technical Overview
nav_order: 1
---

# 🔧 Technical Implementation Overview

**Document:** Technical Architecture & Implementation Details  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Current Flutter App Analysis

### **Application Structure**
```
lib/
├── main.dart                    # App entry point with Provider setup
├── screens/
│   ├── initial_screen.dart      # App state determination
│   ├── welcome_screen.dart      # User welcome & navigation
│   ├── login_screen.dart        # User authentication
│   ├── signup_screen.dart       # User registration
│   ├── onboarding_screen.dart   # First-time user experience
│   ├── chat_screen.dart         # Main chat interface
│   ├── preferences_screen.dart  # User settings management
│   └── utils/
│       └── theme_provider.dart  # Theme management
```

### **State Management Architecture**
```dart
// Current Provider Pattern Implementation
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final themeProvider = ThemeProvider();
  await themeProvider.loadThemeFromPrefs();
  runApp(
    ChangeNotifierProvider(
      create: (_) => themeProvider,
      child: const SpeechApp(),
    ),
  );
}
```

### **Navigation Flow**
```
InitialScreen (Auth Check) 
    ↓
WelcomeScreen (First Launch)
    ↓
LoginScreen / SignupScreen (Authentication)
    ↓
OnboardingScreen (First-time Users)
    ↓
ChatScreen (Main Application)
    ↓
PreferencesScreen (Settings)
```

---

## 🗄️ Current Data Storage Analysis

### **SharedPreferences Usage**
```dart
// Current Implementation Pattern
class ChatScreen extends StatefulWidget {
  // User Preferences
  bool isDarkMode = true;
  bool textAnimations = true;
  double fontSize = 16.0;
  String persona = 'Friendly';
  
  // Chat Data
  List<Map<String, dynamic>> chatSessions = [];
  String currentChatId = '';
  
  // App State
  bool isLoggedIn = false;
  bool hasUsedAppBefore = false;
}
```

### **Data Persistence Patterns**
1. **User Preferences**: Theme, animations, font size, persona
2. **Authentication State**: Login status, user session
3. **Chat History**: Complete conversation history
4. **App State**: Onboarding completion, current chat ID

---

## 🏗️ Target API Architecture

### **RESTful API Design**
```
Base URL: https://api.speechbot.com/v1
Authentication: Bearer JWT Token
Content-Type: application/json
```

### **Microservices Approach**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Auth Service  │    │  Chat Service   │    │ Export Service  │
│                 │    │                 │    │                 │
│ - User Login    │    │ - Chat CRUD     │    │ - Data Export   │
│ - Registration  │    │ - Messages      │    │ - File Generation│
│ - JWT Tokens    │    │ - Real-time     │    │ - Download URLs │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │ PostgreSQL DB   │
                    │                 │
                    │ - Users         │
                    │ - Chats         │
                    │ - Messages      │
                    │ - Preferences   │
                    └─────────────────┘
```

---

## 💾 Database Schema Requirements

### **Users Table**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    email_verified BOOLEAN DEFAULT FALSE
);
```

### **User Preferences Table**
```sql
CREATE TABLE user_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    theme_dark_mode BOOLEAN DEFAULT TRUE,
    chat_text_animations BOOLEAN DEFAULT TRUE,
    chat_font_size DECIMAL(3,1) DEFAULT 16.0,
    chat_persona VARCHAR(50) DEFAULT 'Friendly',
    notifications_enabled BOOLEAN DEFAULT TRUE,
    onboarding_completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Chats Table**
```sql
CREATE TABLE chats (
    id VARCHAR(50) PRIMARY KEY, -- chat_timestamp format
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) DEFAULT 'New Chat',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    message_count INTEGER DEFAULT 0,
    is_favorite BOOLEAN DEFAULT FALSE,
    metadata JSONB
);
```

### **Messages Table**
```sql
CREATE TABLE messages (
    id VARCHAR(50) PRIMARY KEY,
    chat_id VARCHAR(50) REFERENCES chats(id) ON DELETE CASCADE,
    role VARCHAR(10) NOT NULL CHECK (role IN ('user', 'bot')),
    content TEXT NOT NULL,
    full_content TEXT, -- For bot responses with typewriter
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB
);
```

---

## 🔄 Frontend Integration Strategy

### **HTTP Client Setup**
```dart
// New API Service Layer
class ApiService {
  static const String baseUrl = 'https://api.speechbot.com/v1';
  static final Dio _dio = Dio();
  
  static Future<void> init() async {
    _dio.options.baseUrl = baseUrl;
    _dio.options.connectTimeout = Duration(seconds: 30);
    _dio.options.receiveTimeout = Duration(seconds: 30);
    
    // Add interceptors for auth, logging, error handling
    _dio.interceptors.add(AuthInterceptor());
    _dio.interceptors.add(LoggingInterceptor());
    _dio.interceptors.add(ErrorInterceptor());
  }
}
```

### **Authentication Service**
```dart
class AuthService {
  static Future<LoginResponse> login(String email, String password) async {
    final response = await ApiService.post('/auth/login', {
      'email': email,
      'password': password,
      'device_info': await _getDeviceInfo(),
    });
    
    // Store tokens securely
    await _storeTokens(response.data['tokens']);
    return LoginResponse.fromJson(response.data);
  }
  
  static Future<Map<String, String>> getHeaders() async {
    final token = await _getStoredToken();
    return {
      'Authorization': 'Bearer $token',
      'Content-Type': 'application/json',
    };
  }
}
```

### **Data Migration Strategy**
```dart
class HybridDataService {
  // Gradual migration approach
  static Future<List<Chat>> getChats() async {
    try {
      // Try API first
      final apiChats = await ChatApiService.getChats();
      await _cacheChatsLocally(apiChats);
      return apiChats;
    } catch (e) {
      // Fallback to local storage
      return await _getLocalChats();
    }
  }
  
  static Future<void> syncPendingData() async {
    final pendingChats = await _getPendingLocalChats();
    for (final chat in pendingChats) {
      try {
        await ChatApiService.syncChat(chat);
        await _markChatAsSynced(chat.id);
      } catch (e) {
        // Retry later
        await _scheduleRetry(chat.id);
      }
    }
  }
}
```

---

## 🔐 Security Implementation

### **JWT Token Management**
```dart
class TokenManager {
  static const String _tokenKey = 'jwt_token';
  static const String _refreshKey = 'refresh_token';
  
  static Future<void> storeTokens(String accessToken, String refreshToken) async {
    final storage = FlutterSecureStorage();
    await storage.write(key: _tokenKey, value: accessToken);
    await storage.write(key: _refreshKey, value: refreshToken);
  }
  
  static Future<String?> getAccessToken() async {
    final storage = FlutterSecureStorage();
    return await storage.read(key: _tokenKey);
  }
  
  static Future<bool> refreshToken() async {
    final refreshToken = await _getRefreshToken();
    if (refreshToken == null) return false;
    
    try {
      final response = await ApiService.post('/auth/refresh', {
        'refresh_token': refreshToken,
      });
      
      await storeTokens(
        response.data['access_token'],
        response.data['refresh_token'],
      );
      return true;
    } catch (e) {
      await clearTokens();
      return false;
    }
  }
}
```

---

## 📡 Real-time Features

### **WebSocket Implementation**
```dart
class ChatWebSocketService {
  late WebSocketChannel _channel;
  
  void connect(String chatId) {
    _channel = WebSocketChannel.connect(
      Uri.parse('wss://api.speechbot.com/v1/chat/$chatId/ws'),
    );
    
    _channel.stream.listen((message) {
      final data = jsonDecode(message);
      _handleRealtimeMessage(data);
    });
  }
  
  void sendMessage(String content) {
    _channel.sink.add(jsonEncode({
      'type': 'user_message',
      'content': content,
      'timestamp': DateTime.now().toIso8601String(),
    }));
  }
}
```

---

## 🎯 Performance Optimization

### **Caching Strategy**
```dart
class CacheManager {
  static final Map<String, dynamic> _memoryCache = {};
  static const Duration _cacheExpiry = Duration(minutes: 30);
  
  static Future<T?> get<T>(String key) async {
    final cached = _memoryCache[key];
    if (cached != null && !_isExpired(cached['timestamp'])) {
      return cached['data'] as T;
    }
    return null;
  }
  
  static void set<T>(String key, T data) {
    _memoryCache[key] = {
      'data': data,
      'timestamp': DateTime.now(),
    };
  }
}
```

### **Pagination Implementation**
```dart
class PaginatedChatList extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: chats.length + (hasMore ? 1 : 0),
      itemBuilder: (context, index) {
        if (index == chats.length) {
          _loadMoreChats();
          return CircularProgressIndicator();
        }
        return ChatTile(chat: chats[index]);
      },
    );
  }
  
  Future<void> _loadMoreChats() async {
    if (isLoading) return;
    setState(() => isLoading = true);
    
    final newChats = await ChatApiService.getChats(
      page: currentPage + 1,
      limit: 20,
    );
    
    setState(() {
      chats.addAll(newChats);
      currentPage++;
      hasMore = newChats.length == 20;
      isLoading = false;
    });
  }
}
```

---

## 🔄 Offline Support

### **Sync Queue Implementation**
```dart
class OfflineSyncQueue {
  static final List<SyncOperation> _queue = [];
  
  static void addOperation(SyncOperation operation) {
    _queue.add(operation);
    _saveQueueToStorage();
  }
  
  static Future<void> processQueue() async {
    if (!await _isOnline()) return;
    
    final operations = List.from(_queue);
    for (final operation in operations) {
      try {
        await operation.execute();
        _queue.remove(operation);
      } catch (e) {
        operation.retryCount++;
        if (operation.retryCount >= 3) {
          _queue.remove(operation);
        }
      }
    }
    
    await _saveQueueToStorage();
  }
}
```

---

## 📊 Error Handling & Logging

### **Centralized Error Handler**
```dart
class ErrorHandler {
  static void handleApiError(DioError error) {
    switch (error.response?.statusCode) {
      case 401:
        _handleUnauthorized();
        break;
      case 403:
        _handleForbidden();
        break;
      case 404:
        _handleNotFound();
        break;
      case 429:
        _handleRateLimit();
        break;
      default:
        _handleGenericError(error);
    }
  }
  
  static void logEvent(String event, Map<String, dynamic> data) {
    final logEntry = {
      'timestamp': DateTime.now().toIso8601String(),
      'event': event,
      'data': data,
      'app_version': '1.0.0',
    };
    
    // Send to analytics service
    AnalyticsService.track(logEntry);
  }
}
```

---

**This technical overview provides the foundation for implementing the API integration while maintaining the current Flutter app's architecture and user experience.**
