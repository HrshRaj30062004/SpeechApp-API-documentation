# ⚙️ User Preferences Management

**Document:** User Preferences API Specification  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Preferences Overview

### **Current Flutter App Implementation**
The app currently manages user preferences locally using SharedPreferences:

```dart
// Current preference structure in chat_screen.dart
bool isDarkMode = true;
bool textAnimations = true;
double fontSize = 16.0;
String persona = 'Friendly';
bool hasUsedAppBefore = false;
```

### **Target API Implementation**
- **Cloud Synchronization**: Preferences synced across all user devices
- **Versioned Updates**: Track preference changes over time
- **Bulk Operations**: Update multiple preferences in single request
- **Real-time Sync**: Immediate updates across active sessions
- **Conflict Resolution**: Handle simultaneous updates from multiple devices

---

## 🔧 User Preferences Endpoints

### **GET /preferences**
Retrieve current user preferences.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "preferences": {
      "appearance": {
        "theme_dark_mode": true,
        "chat_font_size": 16.0,
        "chat_text_animations": true,
        "high_contrast_mode": false,
        "reduce_motion": false
      },
      "chat": {
        "persona": "Friendly",
        "auto_scroll": true,
        "show_timestamps": false,
        "typewriter_speed": "normal",
        "message_grouping": true
      },
      "notifications": {
        "push_enabled": true,
        "email_enabled": false,
        "sound_enabled": true,
        "vibration_enabled": true,
        "do_not_disturb": {
          "enabled": false,
          "start_time": "22:00",
          "end_time": "08:00"
        }
      },
      "privacy": {
        "analytics_enabled": true,
        "crash_reporting": true,
        "data_collection": "minimal"
      },
      "export": {
        "default_format": "markdown",
        "include_timestamps": true,
        "include_metadata": false,
        "compression_enabled": true
      },
      "onboarding": {
        "completed": true,
        "tour_shown": true,
        "feature_highlights_seen": ["search", "export", "rename"]
      }
    },
    "metadata": {
      "version": 3,
      "last_updated": "2025-07-15T10:30:00Z",
      "updated_from": "android_app",
      "sync_status": "synchronized"
    }
  }
}
```

---

### **PUT /preferences**
Update user preferences (full replace).

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "preferences": {
    "appearance": {
      "theme_dark_mode": false,
      "chat_font_size": 18.0,
      "chat_text_animations": false,
      "high_contrast_mode": false,
      "reduce_motion": true
    },
    "chat": {
      "persona": "Professional",
      "auto_scroll": true,
      "show_timestamps": true,
      "typewriter_speed": "fast",
      "message_grouping": false
    },
    "notifications": {
      "push_enabled": false,
      "email_enabled": true,
      "sound_enabled": false,
      "vibration_enabled": false,
      "do_not_disturb": {
        "enabled": true,
        "start_time": "23:00",
        "end_time": "07:00"
      }
    }
  },
  "metadata": {
    "updated_from": "ios_app",
    "client_version": "1.0.0"
  }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Preferences updated successfully",
  "data": {
    "preferences": {
      // Updated preferences object
    },
    "metadata": {
      "version": 4,
      "last_updated": "2025-07-15T10:35:00Z",
      "updated_from": "ios_app",
      "sync_status": "synchronized"
    }
  }
}
```

**Error Responses:**
```json
// 400 Bad Request - Invalid preference values
{
  "success": false,
  "error": "INVALID_PREFERENCES",
  "message": "Invalid preference values provided",
  "details": {
    "appearance.chat_font_size": ["Must be between 12.0 and 24.0"],
    "chat.persona": ["Must be one of: Friendly, Professional, Casual, Formal"]
  }
}

// 409 Conflict - Version mismatch
{
  "success": false,
  "error": "PREFERENCES_CONFLICT",
  "message": "Preferences have been updated by another device",
  "details": {
    "current_version": 5,
    "provided_version": 3,
    "last_updated": "2025-07-15T10:33:00Z",
    "updated_from": "web_app"
  }
}
```

---

### **PATCH /preferences**
Partially update user preferences.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "updates": {
    "appearance.theme_dark_mode": false,
    "chat.persona": "Professional",
    "notifications.push_enabled": true
  },
  "metadata": {
    "updated_from": "android_app",
    "client_version": "1.0.0"
  }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Preferences updated successfully",
  "data": {
    "updated_fields": [
      "appearance.theme_dark_mode",
      "chat.persona",
      "notifications.push_enabled"
    ],
    "preferences": {
      // Complete updated preferences object
    },
    "metadata": {
      "version": 4,
      "last_updated": "2025-07-15T10:35:00Z",
      "updated_from": "android_app",
      "sync_status": "synchronized"
    }
  }
}
```

---

### **POST /preferences/reset**
Reset preferences to default values.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "reset_scope": "all", // Options: "all", "appearance", "chat", "notifications", "privacy", "export"
  "confirm": true
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Preferences reset to defaults",
  "data": {
    "preferences": {
      // Default preferences object
    },
    "metadata": {
      "version": 5,
      "last_updated": "2025-07-15T10:40:00Z",
      "updated_from": "android_app",
      "sync_status": "synchronized"
    }
  }
}
```

---

### **GET /preferences/history**
Get preference change history.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `limit`: Number of changes to return (default: 20, max: 100)
- `page`: Page number for pagination (default: 1)
- `category`: Filter by preference category (optional)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "changes": [
      {
        "id": "change_123",
        "timestamp": "2025-07-15T10:35:00Z",
        "category": "appearance",
        "changes": [
          {
            "field": "theme_dark_mode",
            "old_value": true,
            "new_value": false
          },
          {
            "field": "chat_font_size",
            "old_value": 16.0,
            "new_value": 18.0
          }
        ],
        "metadata": {
          "updated_from": "ios_app",
          "client_version": "1.0.0",
          "user_agent": "SpeechBot/1.0.0 (iOS 15.0)"
        }
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 3,
      "total_changes": 47,
      "has_next": true,
      "has_previous": false
    }
  }
}
```

---

## 📋 Preference Categories & Validation

### **Appearance Preferences**
```json
{
  "appearance": {
    "theme_dark_mode": {
      "type": "boolean",
      "default": true,
      "description": "Enable dark mode theme"
    },
    "chat_font_size": {
      "type": "number",
      "default": 16.0,
      "min": 12.0,
      "max": 24.0,
      "step": 1.0,
      "description": "Font size for chat messages"
    },
    "chat_text_animations": {
      "type": "boolean",
      "default": true,
      "description": "Enable typewriter animation for bot responses"
    },
    "high_contrast_mode": {
      "type": "boolean",
      "default": false,
      "description": "Enable high contrast mode for accessibility"
    },
    "reduce_motion": {
      "type": "boolean",
      "default": false,
      "description": "Reduce animations and motion effects"
    }
  }
}
```

### **Chat Preferences**
```json
{
  "chat": {
    "persona": {
      "type": "enum",
      "default": "Friendly",
      "options": ["Friendly", "Professional", "Casual", "Formal", "Creative"],
      "description": "AI assistant personality style"
    },
    "auto_scroll": {
      "type": "boolean",
      "default": true,
      "description": "Automatically scroll to new messages"
    },
    "show_timestamps": {
      "type": "boolean",
      "default": false,
      "description": "Show message timestamps"
    },
    "typewriter_speed": {
      "type": "enum",
      "default": "normal",
      "options": ["slow", "normal", "fast", "instant"],
      "description": "Speed of typewriter animation"
    },
    "message_grouping": {
      "type": "boolean",
      "default": true,
      "description": "Group consecutive messages from same sender"
    }
  }
}
```

### **Notification Preferences**
```json
{
  "notifications": {
    "push_enabled": {
      "type": "boolean",
      "default": true,
      "description": "Enable push notifications"
    },
    "email_enabled": {
      "type": "boolean",
      "default": false,
      "description": "Enable email notifications"
    },
    "sound_enabled": {
      "type": "boolean",
      "default": true,
      "description": "Play notification sounds"
    },
    "vibration_enabled": {
      "type": "boolean",
      "default": true,
      "description": "Enable vibration for notifications"
    },
    "do_not_disturb": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "default": false
        },
        "start_time": {
          "type": "string",
          "format": "time",
          "default": "22:00"
        },
        "end_time": {
          "type": "string",
          "format": "time",
          "default": "08:00"
        }
      }
    }
  }
}
```

---

## 📱 Flutter Integration

### **Preferences Service**
```dart
class PreferencesService {
  static const String _localKey = 'cached_preferences';
  
  // Load preferences with caching
  static Future<UserPreferences> getPreferences() async {
    try {
      // Try API first
      final response = await ApiService.get('/preferences');
      final preferences = UserPreferences.fromJson(response.data['data']);
      
      // Cache locally
      await _cachePreferences(preferences);
      return preferences;
    } catch (e) {
      // Fallback to cached preferences
      return await _getCachedPreferences() ?? UserPreferences.defaults();
    }
  }
  
  // Update preferences
  static Future<UserPreferences> updatePreferences(
    Map<String, dynamic> updates
  ) async {
    try {
      // Update locally first for immediate UI response
      final currentPrefs = await _getCachedPreferences();
      final updatedPrefs = currentPrefs?.applyUpdates(updates);
      if (updatedPrefs != null) {
        await _cachePreferences(updatedPrefs);
      }
      
      // Send to API
      final response = await ApiService.patch('/preferences', data: {
        'updates': updates,
        'metadata': {
          'updated_from': 'flutter_app',
          'client_version': await _getAppVersion(),
        }
      });
      
      final apiPrefs = UserPreferences.fromJson(response.data['data']);
      await _cachePreferences(apiPrefs);
      return apiPrefs;
    } catch (e) {
      // If API fails, keep local changes and queue for retry
      await _queuePreferenceUpdate(updates);
      throw PreferencesException.fromError(e);
    }
  }
  
  // Sync pending changes
  static Future<void> syncPendingChanges() async {
    final pendingUpdates = await _getPendingUpdates();
    for (final update in pendingUpdates) {
      try {
        await updatePreferences(update.changes);
        await _markUpdateAsSynced(update.id);
      } catch (e) {
        // Retry later
        await _scheduleRetry(update);
      }
    }
  }
}
```

### **Preferences Provider**
```dart
class PreferencesProvider extends ChangeNotifier {
  UserPreferences? _preferences;
  bool _isLoading = true;
  String? _error;
  
  UserPreferences? get preferences => _preferences;
  bool get isLoading => _isLoading;
  String? get error => _error;
  
  // Convenience getters for common preferences
  bool get isDarkMode => _preferences?.appearance.themeDarkMode ?? true;
  double get fontSize => _preferences?.chat.chatFontSize ?? 16.0;
  String get persona => _preferences?.chat.persona ?? 'Friendly';
  bool get textAnimations => _preferences?.appearance.chatTextAnimations ?? true;
  
  Future<void> initialize() async {
    try {
      _isLoading = true;
      _error = null;
      notifyListeners();
      
      _preferences = await PreferencesService.getPreferences();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<void> updatePreference(String key, dynamic value) async {
    try {
      _error = null;
      
      // Update locally for immediate UI response
      _preferences = _preferences?.copyWith({key: value});
      notifyListeners();
      
      // Update via API
      await PreferencesService.updatePreferences({key: value});
      
      // Refresh from server to ensure consistency
      _preferences = await PreferencesService.getPreferences();
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  Future<void> updateMultiplePreferences(Map<String, dynamic> updates) async {
    try {
      _error = null;
      
      // Update locally for immediate UI response
      _preferences = _preferences?.copyWith(updates);
      notifyListeners();
      
      // Update via API
      await PreferencesService.updatePreferences(updates);
      
      // Refresh from server
      _preferences = await PreferencesService.getPreferences();
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  Future<void> resetPreferences([String? category]) async {
    try {
      _error = null;
      
      final response = await ApiService.post('/preferences/reset', data: {
        'reset_scope': category ?? 'all',
        'confirm': true,
      });
      
      _preferences = UserPreferences.fromJson(response.data['data']);
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
}
```

### **Preferences Model**
```dart
class UserPreferences {
  final AppearancePreferences appearance;
  final ChatPreferences chat;
  final NotificationPreferences notifications;
  final PrivacyPreferences privacy;
  final ExportPreferences export;
  final OnboardingPreferences onboarding;
  final PreferencesMetadata metadata;
  
  const UserPreferences({
    required this.appearance,
    required this.chat,
    required this.notifications,
    required this.privacy,
    required this.export,
    required this.onboarding,
    required this.metadata,
  });
  
  factory UserPreferences.defaults() {
    return UserPreferences(
      appearance: AppearancePreferences.defaults(),
      chat: ChatPreferences.defaults(),
      notifications: NotificationPreferences.defaults(),
      privacy: PrivacyPreferences.defaults(),
      export: ExportPreferences.defaults(),
      onboarding: OnboardingPreferences.defaults(),
      metadata: PreferencesMetadata.defaults(),
    );
  }
  
  factory UserPreferences.fromJson(Map<String, dynamic> json) {
    return UserPreferences(
      appearance: AppearancePreferences.fromJson(json['preferences']['appearance']),
      chat: ChatPreferences.fromJson(json['preferences']['chat']),
      notifications: NotificationPreferences.fromJson(json['preferences']['notifications']),
      privacy: PrivacyPreferences.fromJson(json['preferences']['privacy']),
      export: ExportPreferences.fromJson(json['preferences']['export']),
      onboarding: OnboardingPreferences.fromJson(json['preferences']['onboarding']),
      metadata: PreferencesMetadata.fromJson(json['metadata']),
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'preferences': {
        'appearance': appearance.toJson(),
        'chat': chat.toJson(),
        'notifications': notifications.toJson(),
        'privacy': privacy.toJson(),
        'export': export.toJson(),
        'onboarding': onboarding.toJson(),
      },
      'metadata': metadata.toJson(),
    };
  }
  
  UserPreferences copyWith(Map<String, dynamic> updates) {
    // Deep copy with nested updates
    final Map<String, dynamic> currentJson = toJson();
    
    // Apply updates using dot notation
    for (final entry in updates.entries) {
      _setNestedValue(currentJson['preferences'], entry.key, entry.value);
    }
    
    return UserPreferences.fromJson({
      'preferences': currentJson['preferences'],
      'metadata': currentJson['metadata'],
    });
  }
  
  void _setNestedValue(Map<String, dynamic> map, String key, dynamic value) {
    final keys = key.split('.');
    Map<String, dynamic> current = map;
    
    for (int i = 0; i < keys.length - 1; i++) {
      current = current[keys[i]] as Map<String, dynamic>;
    }
    
    current[keys.last] = value;
  }
}

class AppearancePreferences {
  final bool themeDarkMode;
  final double chatFontSize;
  final bool chatTextAnimations;
  final bool highContrastMode;
  final bool reduceMotion;
  
  const AppearancePreferences({
    required this.themeDarkMode,
    required this.chatFontSize,
    required this.chatTextAnimations,
    required this.highContrastMode,
    required this.reduceMotion,
  });
  
  factory AppearancePreferences.defaults() {
    return const AppearancePreferences(
      themeDarkMode: true,
      chatFontSize: 16.0,
      chatTextAnimations: true,
      highContrastMode: false,
      reduceMotion: false,
    );
  }
  
  factory AppearancePreferences.fromJson(Map<String, dynamic> json) {
    return AppearancePreferences(
      themeDarkMode: json['theme_dark_mode'] ?? true,
      chatFontSize: (json['chat_font_size'] ?? 16.0).toDouble(),
      chatTextAnimations: json['chat_text_animations'] ?? true,
      highContrastMode: json['high_contrast_mode'] ?? false,
      reduceMotion: json['reduce_motion'] ?? false,
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'theme_dark_mode': themeDarkMode,
      'chat_font_size': chatFontSize,
      'chat_text_animations': chatTextAnimations,
      'high_contrast_mode': highContrastMode,
      'reduce_motion': reduceMotion,
    };
  }
}
```

---

## 🔄 Synchronization Strategy

### **Conflict Resolution**
```dart
class PreferencesSyncManager {
  // Handle conflicts when multiple devices update preferences
  static Future<UserPreferences> resolveConflict(
    UserPreferences local,
    UserPreferences remote,
    ConflictResolutionStrategy strategy,
  ) async {
    switch (strategy) {
      case ConflictResolutionStrategy.serverWins:
        return remote;
      
      case ConflictResolutionStrategy.clientWins:
        return local;
      
      case ConflictResolutionStrategy.mostRecent:
        return local.metadata.lastUpdated.isAfter(remote.metadata.lastUpdated)
            ? local
            : remote;
      
      case ConflictResolutionStrategy.fieldLevel:
        return _mergePreferences(local, remote);
    }
  }
  
  static UserPreferences _mergePreferences(
    UserPreferences local,
    UserPreferences remote,
  ) {
    // Field-level conflict resolution
    // Use most recently updated value for each field
    // Implementation depends on tracking field-level timestamps
  }
}
```

---

**This preferences system provides seamless synchronization across devices while maintaining local responsiveness and offline capability.**
