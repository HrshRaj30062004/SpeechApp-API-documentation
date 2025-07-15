---
layout: default
title: Error Handling
nav_order: 7
---

# ❌ Error Handling & Status Codes

**Document:** Error Handling & HTTP Status Codes Reference  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Error Handling Philosophy

### **Consistent Error Response Format**
All API errors follow a standardized response structure for predictable client-side handling:

```json
{
  "success": false,
  "error": "ERROR_CODE",
  "message": "Human-readable error description",
  "details": {
    // Optional additional error information
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

### **Error Handling Principles**
- **Predictable Structure**: Consistent error format across all endpoints
- **Informative Messages**: Clear, actionable error descriptions
- **Appropriate Status Codes**: Standard HTTP status codes with semantic meaning
- **Client Recovery**: Guidance for handling and recovering from errors
- **Security Conscious**: No sensitive information exposed in error messages
- **Localization Ready**: Error codes for easy translation support

---

## 📊 HTTP Status Codes

### **2xx Success Codes**

#### **200 OK**
Request completed successfully with response body.
```json
{
  "success": true,
  "data": {
    // Response data
  }
}
```

#### **201 Created**
Resource created successfully.
```json
{
  "success": true,
  "message": "Resource created successfully",
  "data": {
    // Created resource data
  }
}
```

#### **202 Accepted**
Request accepted for asynchronous processing.
```json
{
  "success": true,
  "message": "Request accepted for processing",
  "data": {
    "job_id": "job_1684761234567",
    "status": "queued",
    "estimated_completion": "2025-07-15T16:35:00Z"
  }
}
```

#### **204 No Content**
Request completed successfully with no response body (typically for DELETE operations).

---

### **4xx Client Error Codes**

#### **400 Bad Request**
Invalid request data or parameters.

```json
{
  "success": false,
  "error": "VALIDATION_ERROR",
  "message": "Invalid request data",
  "details": {
    "email": ["Must be a valid email address"],
    "password": ["Password must be at least 8 characters long"],
    "chat_font_size": ["Must be between 12.0 and 24.0"]
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Common Error Codes:**
- `VALIDATION_ERROR`: Request validation failed
- `INVALID_FORMAT`: Unsupported format specified
- `MISSING_REQUIRED_FIELD`: Required field not provided
- `INVALID_DATE_RANGE`: Invalid date range specified
- `PAYLOAD_TOO_LARGE`: Request payload exceeds size limit

#### **401 Unauthorized**
Authentication required or invalid credentials.

```json
{
  "success": false,
  "error": "UNAUTHORIZED",
  "message": "Authentication required",
  "details": {
    "reason": "invalid_token",
    "provided_token_type": "bearer"
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Common Error Codes:**
- `UNAUTHORIZED`: No authentication provided
- `INVALID_TOKEN`: JWT token is invalid or malformed
- `TOKEN_EXPIRED`: JWT token has expired
- `INVALID_CREDENTIALS`: Username/password authentication failed
- `ACCOUNT_SUSPENDED`: User account is suspended

#### **403 Forbidden**
Access denied for authenticated user.

```json
{
  "success": false,
  "error": "ACCESS_DENIED",
  "message": "You don't have permission to access this resource",
  "details": {
    "required_permission": "chat:delete",
    "user_permissions": ["chat:read", "chat:write"]
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Common Error Codes:**
- `ACCESS_DENIED`: Insufficient permissions
- `RESOURCE_ACCESS_DENIED`: Cannot access specific resource
- `OPERATION_NOT_ALLOWED`: Operation not permitted for user
- `ACCOUNT_LOCKED`: Account temporarily locked

#### **404 Not Found**
Requested resource does not exist.

```json
{
  "success": false,
  "error": "RESOURCE_NOT_FOUND",
  "message": "The requested resource was not found",
  "details": {
    "resource_type": "chat",
    "resource_id": "chat_1684761234567"
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Common Error Codes:**
- `RESOURCE_NOT_FOUND`: Generic resource not found
- `CHAT_NOT_FOUND`: Specific chat session not found
- `MESSAGE_NOT_FOUND`: Specific message not found
- `USER_NOT_FOUND`: User account not found
- `ENDPOINT_NOT_FOUND`: API endpoint doesn't exist

#### **409 Conflict**
Request conflicts with current state of resource.

```json
{
  "success": false,
  "error": "RESOURCE_CONFLICT",
  "message": "The request conflicts with the current state of the resource",
  "details": {
    "conflict_type": "version_mismatch",
    "current_version": 5,
    "provided_version": 3,
    "last_updated_by": "another_device"
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Common Error Codes:**
- `RESOURCE_CONFLICT`: Generic resource conflict
- `EMAIL_EXISTS`: Email address already registered
- `CHAT_TITLE_EXISTS`: Chat title already exists
- `PREFERENCES_CONFLICT`: Preferences updated by another device
- `EXPORT_IN_PROGRESS`: Export already in progress

#### **413 Payload Too Large**
Request payload exceeds size limits.

```json
{
  "success": false,
  "error": "PAYLOAD_TOO_LARGE",
  "message": "Request payload exceeds maximum size limit",
  "details": {
    "max_size_bytes": 10485760,
    "provided_size_bytes": 15728640,
    "limit_type": "file_upload"
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

#### **422 Unprocessable Entity**
Request is well-formed but contains semantic errors.

```json
{
  "success": false,
  "error": "UNPROCESSABLE_ENTITY",
  "message": "The request contains semantic errors",
  "details": {
    "semantic_errors": [
      "End date must be after start date",
      "Cannot export data from the future"
    ]
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

#### **429 Too Many Requests**
Rate limit exceeded.

```json
{
  "success": false,
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Please try again later.",
  "details": {
    "limit_type": "api_requests",
    "max_requests": 100,
    "window_seconds": 3600,
    "retry_after_seconds": 1800,
    "current_usage": 100
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

**Rate Limit Headers:**
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1642251000
Retry-After: 1800
```

---

### **5xx Server Error Codes**

#### **500 Internal Server Error**
Unexpected server error.

```json
{
  "success": false,
  "error": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected server error occurred",
  "details": {
    "error_id": "err_1684761234567",
    "support_contact": "support@speechbot.com"
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

#### **502 Bad Gateway**
Upstream service error.

```json
{
  "success": false,
  "error": "BAD_GATEWAY",
  "message": "Upstream service error",
  "details": {
    "service": "ai_processing_service",
    "retry_recommended": true
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

#### **503 Service Unavailable**
Service temporarily unavailable.

```json
{
  "success": false,
  "error": "SERVICE_UNAVAILABLE",
  "message": "Service temporarily unavailable",
  "details": {
    "reason": "scheduled_maintenance",
    "estimated_duration_minutes": 30,
    "retry_after_seconds": 1800
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

#### **504 Gateway Timeout**
Request timeout from upstream service.

```json
{
  "success": false,
  "error": "GATEWAY_TIMEOUT",
  "message": "Request timeout from upstream service",
  "details": {
    "timeout_seconds": 30,
    "service": "ai_processing_service",
    "retry_recommended": true
  },
  "timestamp": "2025-07-15T16:30:00Z",
  "request_id": "req_1684761234567"
}
```

---

## 📱 Flutter Error Handling Implementation

### **Centralized Exception Handling**
```dart
class ApiException implements Exception {
  final String code;
  final String message;
  final Map<String, dynamic>? details;
  final DateTime timestamp;
  final String? requestId;
  final int statusCode;
  
  const ApiException({
    required this.code,
    required this.message,
    required this.statusCode,
    this.details,
    required this.timestamp,
    this.requestId,
  });
  
  factory ApiException.fromResponse(Response response) {
    final data = response.data;
    
    return ApiException(
      code: data['error'] ?? 'UNKNOWN_ERROR',
      message: data['message'] ?? 'An unknown error occurred',
      statusCode: response.statusCode ?? 500,
      details: data['details'],
      timestamp: data['timestamp'] != null 
          ? DateTime.parse(data['timestamp'])
          : DateTime.now(),
      requestId: data['request_id'],
    );
  }
  
  factory ApiException.fromDioError(DioError error) {
    final response = error.response;
    
    if (response != null) {
      return ApiException.fromResponse(response);
    }
    
    // Handle network errors
    switch (error.type) {
      case DioErrorType.connectTimeout:
      case DioErrorType.sendTimeout:
      case DioErrorType.receiveTimeout:
        return ApiException(
          code: 'NETWORK_TIMEOUT',
          message: 'Network request timed out',
          statusCode: 408,
          timestamp: DateTime.now(),
        );
      
      case DioErrorType.cancel:
        return ApiException(
          code: 'REQUEST_CANCELLED',
          message: 'Request was cancelled',
          statusCode: 499,
          timestamp: DateTime.now(),
        );
      
      default:
        return ApiException(
          code: 'NETWORK_ERROR',
          message: 'Network connection failed',
          statusCode: 0,
          timestamp: DateTime.now(),
        );
    }
  }
  
  bool get isNetworkError => statusCode == 0 || code == 'NETWORK_ERROR';
  bool get isAuthError => statusCode == 401;
  bool get isPermissionError => statusCode == 403;
  bool get isNotFoundError => statusCode == 404;
  bool get isConflictError => statusCode == 409;
  bool get isRateLimitError => statusCode == 429;
  bool get isServerError => statusCode >= 500;
  bool get isClientError => statusCode >= 400 && statusCode < 500;
  
  bool get isRetryable => isNetworkError || statusCode >= 500 || statusCode == 429;
  
  String get userFriendlyMessage {
    switch (code) {
      case 'NETWORK_ERROR':
      case 'NETWORK_TIMEOUT':
        return 'Network connection failed. Please check your internet connection and try again.';
      
      case 'UNAUTHORIZED':
      case 'INVALID_TOKEN':
      case 'TOKEN_EXPIRED':
        return 'Your session has expired. Please log in again.';
      
      case 'ACCESS_DENIED':
        return 'You don\'t have permission to perform this action.';
      
      case 'RESOURCE_NOT_FOUND':
        return 'The requested item could not be found.';
      
      case 'RATE_LIMIT_EXCEEDED':
        return 'Too many requests. Please wait before trying again.';
      
      case 'SERVICE_UNAVAILABLE':
        return 'Service is temporarily unavailable. Please try again later.';
      
      case 'VALIDATION_ERROR':
        return 'Please check your input and try again.';
      
      default:
        return message;
    }
  }
  
  @override
  String toString() {
    return 'ApiException($code): $message';
  }
}
```

### **Global Error Handler with Retry Logic**
```dart
class ErrorHandler {
  static const int maxRetries = 3;
  static const Duration retryDelay = Duration(seconds: 2);
  
  static Future<T> handleApiCall<T>(
    Future<T> Function() apiCall, {
    bool autoRetry = true,
    int maxRetries = ErrorHandler.maxRetries,
    Duration retryDelay = ErrorHandler.retryDelay,
  }) async {
    int attempts = 0;
    
    while (attempts < maxRetries) {
      try {
        return await apiCall();
      } catch (e) {
        attempts++;
        
        final exception = e is ApiException ? e : ApiException.fromDioError(e as DioError);
        
        // Don't retry client errors (except rate limits)
        if (!exception.isRetryable || attempts >= maxRetries) {
          await _logError(exception);
          rethrow;
        }
        
        // Wait before retrying
        await Future.delayed(retryDelay * attempts);
      }
    }
    
    throw ApiException(
      code: 'MAX_RETRIES_EXCEEDED',
      message: 'Maximum retry attempts exceeded',
      statusCode: 0,
      timestamp: DateTime.now(),
    );
  }
  
  static Future<void> _logError(ApiException exception) async {
    // Log error for analytics/debugging
    final errorData = {
      'error_code': exception.code,
      'message': exception.message,
      'status_code': exception.statusCode,
      'timestamp': exception.timestamp.toIso8601String(),
      'request_id': exception.requestId,
      'user_id': await _getCurrentUserId(),
      'app_version': await _getAppVersion(),
      'platform': Platform.operatingSystem,
    };
    
    // Send to logging service
    try {
      await AnalyticsService.logError(errorData);
    } catch (e) {
      // Don't let logging errors crash the app
      print('Failed to log error: $e');
    }
  }
  
  static void showErrorDialog(BuildContext context, ApiException exception) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Error'),
        content: Text(exception.userFriendlyMessage),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('OK'),
          ),
          if (exception.isRetryable)
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
                // Trigger retry if applicable
              },
              child: Text('Retry'),
            ),
        ],
      ),
    );
  }
  
  static void showErrorSnackBar(BuildContext context, ApiException exception) {
    final messenger = ScaffoldMessenger.of(context);
    
    messenger.showSnackBar(
      SnackBar(
        content: Text(exception.userFriendlyMessage),
        backgroundColor: exception.isServerError ? Colors.orange : Colors.red,
        action: exception.isRetryable 
            ? SnackBarAction(
                label: 'Retry',
                onPressed: () {
                  // Trigger retry if applicable
                },
              )
            : null,
        duration: Duration(seconds: exception.isServerError ? 6 : 4),
      ),
    );
  }
}
```

### **Provider Error Handling Pattern**
```dart
class BaseProvider extends ChangeNotifier {
  ApiException? _error;
  bool _isLoading = false;
  
  ApiException? get error => _error;
  bool get isLoading => _isLoading;
  bool get hasError => _error != null;
  
  void clearError() {
    _error = null;
    notifyListeners();
  }
  
  Future<T> executeWithErrorHandling<T>(
    Future<T> Function() operation, {
    bool showLoading = true,
    bool autoRetry = true,
  }) async {
    try {
      if (showLoading) {
        _isLoading = true;
        _error = null;
        notifyListeners();
      }
      
      final result = await ErrorHandler.handleApiCall(
        operation,
        autoRetry: autoRetry,
      );
      
      _error = null;
      return result;
    } catch (e) {
      _error = e is ApiException ? e : ApiException.fromDioError(e as DioError);
      rethrow;
    } finally {
      if (showLoading) {
        _isLoading = false;
        notifyListeners();
      }
    }
  }
}

// Usage in providers
class ChatProvider extends BaseProvider {
  List<Chat> _chats = [];
  
  List<Chat> get chats => _chats;
  
  Future<void> loadChats() async {
    await executeWithErrorHandling(() async {
      final response = await ChatService.getChats();
      _chats = response.chats;
      notifyListeners();
    });
  }
  
  Future<void> createChat(String title) async {
    await executeWithErrorHandling(() async {
      final chat = await ChatService.createChat(title: title);
      _chats.insert(0, chat);
      notifyListeners();
    });
  }
}
```

### **Dio Interceptor for Global Error Handling**
```dart
class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioError err, ErrorInterceptorHandler handler) {
    final exception = ApiException.fromDioError(err);
    
    // Handle specific error types globally
    switch (exception.code) {
      case 'TOKEN_EXPIRED':
      case 'INVALID_TOKEN':
        _handleAuthError(exception);
        break;
      
      case 'SERVICE_UNAVAILABLE':
        _handleServiceUnavailable(exception);
        break;
      
      case 'RATE_LIMIT_EXCEEDED':
        _handleRateLimit(exception);
        break;
    }
    
    // Continue with the error
    handler.next(err);
  }
  
  void _handleAuthError(ApiException exception) {
    // Clear auth tokens and redirect to login
    AuthService.clearTokens();
    NavigationService.navigateToLogin();
  }
  
  void _handleServiceUnavailable(ApiException exception) {
    // Show maintenance mode message
    NotificationService.showMaintenanceMessage();
  }
  
  void _handleRateLimit(ApiException exception) {
    // Implement backoff strategy
    final retryAfter = exception.details?['retry_after_seconds'] ?? 60;
    RateLimitService.setBackoff(Duration(seconds: retryAfter));
  }
}
```

---

## 🔄 Offline Error Handling

### **Network Connectivity Detection**
```dart
class ConnectivityService {
  static final Connectivity _connectivity = Connectivity();
  static StreamSubscription<ConnectivityResult>? _subscription;
  
  static bool _isOnline = true;
  static bool get isOnline => _isOnline;
  
  static Stream<bool> get onConnectivityChanged => 
      _connectivity.onConnectivityChanged.map(_mapConnectivityResult);
  
  static Future<void> initialize() async {
    final result = await _connectivity.checkConnectivity();
    _isOnline = _mapConnectivityResult(result);
    
    _subscription = _connectivity.onConnectivityChanged.listen((result) {
      final wasOnline = _isOnline;
      _isOnline = _mapConnectivityResult(result);
      
      if (!wasOnline && _isOnline) {
        _handleReconnection();
      }
    });
  }
  
  static bool _mapConnectivityResult(ConnectivityResult result) {
    return result != ConnectivityResult.none;
  }
  
  static void _handleReconnection() {
    // Retry pending operations when connection is restored
    OfflineQueueService.processQueue();
    NotificationService.showReconnectedMessage();
  }
  
  static void dispose() {
    _subscription?.cancel();
  }
}
```

### **Offline Error Handling**
```dart
class OfflineErrorHandler {
  static Future<T> handleOfflineOperation<T>(
    Future<T> Function() operation, {
    String? operationId,
    bool queueForRetry = true,
  }) async {
    if (!ConnectivityService.isOnline) {
      if (queueForRetry && operationId != null) {
        await OfflineQueueService.queueOperation(operationId, operation);
      }
      
      throw ApiException(
        code: 'OFFLINE_ERROR',
        message: 'This action requires an internet connection',
        statusCode: 0,
        timestamp: DateTime.now(),
      );
    }
    
    try {
      return await operation();
    } catch (e) {
      final exception = e is ApiException ? e : ApiException.fromDioError(e as DioError);
      
      if (exception.isNetworkError && queueForRetry && operationId != null) {
        await OfflineQueueService.queueOperation(operationId, operation);
      }
      
      rethrow;
    }
  }
}
```

---

## 📊 Error Analytics & Monitoring

### **Error Tracking Implementation**
```dart
class ErrorAnalytics {
  static Future<void> trackError(ApiException exception, {
    String? context,
    Map<String, dynamic>? additionalData,
  }) async {
    final errorEvent = {
      'error_code': exception.code,
      'error_message': exception.message,
      'status_code': exception.statusCode,
      'timestamp': exception.timestamp.toIso8601String(),
      'request_id': exception.requestId,
      'context': context,
      'user_id': await _getCurrentUserId(),
      'session_id': await _getSessionId(),
      'app_version': await _getAppVersion(),
      'platform': Platform.operatingSystem,
      'device_info': await _getDeviceInfo(),
      'network_status': ConnectivityService.isOnline ? 'online' : 'offline',
      ...?additionalData,
    };
    
    // Send to analytics service
    await AnalyticsService.track('api_error', errorEvent);
    
    // Send to error monitoring service (e.g., Sentry)
    await ErrorMonitoringService.captureException(exception, errorEvent);
  }
  
  static Future<void> trackUserAction(String action, {
    bool success = true,
    String? errorCode,
    Duration? duration,
  }) async {
    final actionEvent = {
      'action': action,
      'success': success,
      'error_code': errorCode,
      'duration_ms': duration?.inMilliseconds,
      'timestamp': DateTime.now().toIso8601String(),
      'user_id': await _getCurrentUserId(),
      'session_id': await _getSessionId(),
    };
    
    await AnalyticsService.track('user_action', actionEvent);
  }
}
```

---

**This comprehensive error handling system provides robust error management, user-friendly error messages, automatic retry logic, and detailed error tracking while maintaining excellent user experience even during network issues or service outages.**
