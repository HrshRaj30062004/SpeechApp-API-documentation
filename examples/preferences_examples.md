---
layout: default
title: Preferences Examples
parent: Examples
nav_order: 3
---

# ⚙️ User Preferences API Examples

**Document:** User Preferences Management Examples  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Flutter Preferences Implementation Examples

### **Preferences Provider Example**
```dart
// lib/providers/preferences_provider.dart
class PreferencesProvider extends ChangeNotifier {
  UserPreferences? _preferences;
  bool _isLoading = false;
  bool _isSyncing = false;
  String? _error;
  Timer? _syncTimer;

  UserPreferences? get preferences => _preferences;
  bool get isLoading => _isLoading;
  bool get isSyncing => _isSyncing;
  String? get error => _error;

  // Quick access getters
  bool get isDarkMode => _preferences?.themeDarkMode ?? false;
  double get fontSize => _preferences?.chatFontSize ?? 16.0;
  bool get animationsEnabled => _preferences?.chatTextAnimations ?? true;
  bool get notificationsEnabled => _preferences?.notificationsEnabled ?? true;
  String get chatPersona => _preferences?.chatPersona ?? 'Friendly';

  Future<void> initialize() async {
    _isLoading = true;
    notifyListeners();

    try {
      // Load local preferences first
      await _loadLocalPreferences();
      
      // Then sync with server
      await _syncWithServer();
      
      // Start periodic sync
      _startPeriodicSync();
      
    } catch (e) {
      _error = 'Failed to initialize preferences';
      print('Error initializing preferences: $e');
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  Future<void> _loadLocalPreferences() async {
    try {
      final prefs = await SharedPreferences.getInstance();
      
      _preferences = UserPreferences(
        themeDarkMode: prefs.getBool('theme_dark_mode') ?? false,
        chatTextAnimations: prefs.getBool('chat_text_animations') ?? true,
        chatFontSize: prefs.getDouble('chat_font_size') ?? 16.0,
        chatPersona: prefs.getString('chat_persona') ?? 'Friendly',
        notificationsEnabled: prefs.getBool('notifications_enabled') ?? true,
        notificationSound: prefs.getBool('notification_sound') ?? true,
        notificationVibration: prefs.getBool('notification_vibration') ?? true,
        autoExportEnabled: prefs.getBool('auto_export_enabled') ?? false,
        exportFormat: prefs.getString('export_format') ?? 'markdown',
        onboardingCompleted: prefs.getBool('onboarding_completed') ?? false,
        language: prefs.getString('language') ?? 'en',
        timezone: prefs.getString('timezone') ?? DateTime.now().timeZoneName,
      );
      
      notifyListeners();
    } catch (e) {
      print('Error loading local preferences: $e');
    }
  }

  Future<void> _syncWithServer() async {
    if (!await _isAuthenticated()) return;

    try {
      _isSyncing = true;
      notifyListeners();

      final serverPrefs = await PreferencesService.getPreferences();
      
      if (serverPrefs != null) {
        // Merge server preferences with local ones
        _preferences = _mergePreferences(_preferences, serverPrefs);
        await _saveLocalPreferences();
        notifyListeners();
      }
    } catch (e) {
      print('Error syncing with server: $e');
      // Continue with local preferences if sync fails
    } finally {
      _isSyncing = false;
      notifyListeners();
    }
  }

  UserPreferences _mergePreferences(UserPreferences? local, UserPreferences server) {
    if (local == null) return server;
    
    // Server preferences take precedence for most settings
    // But preserve local-only settings like onboarding status
    return UserPreferences(
      themeDarkMode: server.themeDarkMode,
      chatTextAnimations: server.chatTextAnimations,
      chatFontSize: server.chatFontSize,
      chatPersona: server.chatPersona,
      notificationsEnabled: server.notificationsEnabled,
      notificationSound: server.notificationSound,
      notificationVibration: server.notificationVibration,
      autoExportEnabled: server.autoExportEnabled,
      exportFormat: server.exportFormat,
      onboardingCompleted: local.onboardingCompleted, // Keep local status
      language: server.language,
      timezone: server.timezone,
    );
  }

  Future<void> updateTheme(bool darkMode) async {
    await _updatePreference('themeDarkMode', darkMode);
  }

  Future<void> updateFontSize(double fontSize) async {
    await _updatePreference('chatFontSize', fontSize);
  }

  Future<void> updateAnimations(bool enabled) async {
    await _updatePreference('chatTextAnimations', enabled);
  }

  Future<void> updatePersona(String persona) async {
    await _updatePreference('chatPersona', persona);
  }

  Future<void> updateNotifications(bool enabled) async {
    await _updatePreference('notificationsEnabled', enabled);
  }

  Future<void> updateNotificationSound(bool enabled) async {
    await _updatePreference('notificationSound', enabled);
  }

  Future<void> updateNotificationVibration(bool enabled) async {
    await _updatePreference('notificationVibration', enabled);
  }

  Future<void> updateAutoExport(bool enabled) async {
    await _updatePreference('autoExportEnabled', enabled);
  }

  Future<void> updateExportFormat(String format) async {
    await _updatePreference('exportFormat', format);
  }

  Future<void> updateLanguage(String language) async {
    await _updatePreference('language', language);
  }

  Future<void> markOnboardingCompleted() async {
    await _updatePreference('onboardingCompleted', true);
  }

  Future<void> _updatePreference(String key, dynamic value) async {
    if (_preferences == null) return;

    try {
      // Update local preferences immediately
      _preferences = _preferences!.copyWith(key, value);
      await _saveLocalPreferences();
      notifyListeners();

      // Sync with server in background
      if (await _isAuthenticated()) {
        _syncPreferenceWithServer(key, value);
      }
    } catch (e) {
      _error = 'Failed to update preference';
      print('Error updating preference $key: $e');
      notifyListeners();
    }
  }

  Future<void> _syncPreferenceWithServer(String key, dynamic value) async {
    try {
      await PreferencesService.updatePreference(key, value);
    } catch (e) {
      print('Error syncing preference $key with server: $e');
      // Don't show error to user, will sync later
    }
  }

  Future<void> _saveLocalPreferences() async {
    if (_preferences == null) return;

    try {
      final prefs = await SharedPreferences.getInstance();
      
      await prefs.setBool('theme_dark_mode', _preferences!.themeDarkMode);
      await prefs.setBool('chat_text_animations', _preferences!.chatTextAnimations);
      await prefs.setDouble('chat_font_size', _preferences!.chatFontSize);
      await prefs.setString('chat_persona', _preferences!.chatPersona);
      await prefs.setBool('notifications_enabled', _preferences!.notificationsEnabled);
      await prefs.setBool('notification_sound', _preferences!.notificationSound);
      await prefs.setBool('notification_vibration', _preferences!.notificationVibration);
      await prefs.setBool('auto_export_enabled', _preferences!.autoExportEnabled);
      await prefs.setString('export_format', _preferences!.exportFormat);
      await prefs.setBool('onboarding_completed', _preferences!.onboardingCompleted);
      await prefs.setString('language', _preferences!.language);
      await prefs.setString('timezone', _preferences!.timezone);
    } catch (e) {
      print('Error saving local preferences: $e');
    }
  }

  void _startPeriodicSync() {
    _syncTimer = Timer.periodic(Duration(minutes: 5), (timer) async {
      if (await _isAuthenticated()) {
        await _syncWithServer();
      }
    });
  }

  Future<bool> _isAuthenticated() async {
    try {
      final token = await SecureStorageService.getAccessToken();
      return token != null;
    } catch (e) {
      return false;
    }
  }

  Future<void> resetToDefaults() async {
    try {
      _preferences = UserPreferences.defaults();
      await _saveLocalPreferences();
      notifyListeners();

      // Sync reset with server
      if (await _isAuthenticated()) {
        await PreferencesService.resetPreferences();
      }
    } catch (e) {
      _error = 'Failed to reset preferences';
      print('Error resetting preferences: $e');
      notifyListeners();
    }
  }

  void clearError() {
    _error = null;
    notifyListeners();
  }

  void clear() {
    _syncTimer?.cancel();
    _preferences = null;
    _error = null;
    notifyListeners();
  }

  @override
  void dispose() {
    _syncTimer?.cancel();
    super.dispose();
  }
}
```

### **Preferences Screen Example**
```dart
// lib/screens/preferences_screen.dart
class PreferencesScreen extends StatefulWidget {
  @override
  _PreferencesScreenState createState() => _PreferencesScreenState();
}

class _PreferencesScreenState extends State<PreferencesScreen> {
  bool _isResetting = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Preferences'),
        actions: [
          Consumer<PreferencesProvider>(
            builder: (context, prefsProvider, child) {
              return IconButton(
                icon: prefsProvider.isSyncing
                    ? SizedBox(
                        width: 20,
                        height: 20,
                        child: CircularProgressIndicator(strokeWidth: 2),
                      )
                    : Icon(Icons.sync),
                onPressed: prefsProvider.isSyncing
                    ? null
                    : () async {
                        await prefsProvider._syncWithServer();
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(content: Text('Preferences synced')),
                        );
                      },
                tooltip: 'Sync with server',
              );
            },
          ),
          
          PopupMenuButton<String>(
            onSelected: (value) async {
              switch (value) {
                case 'reset':
                  await _showResetDialog();
                  break;
                case 'export':
                  await _exportPreferences();
                  break;
                case 'import':
                  await _importPreferences();
                  break;
              }
            },
            itemBuilder: (context) => [
              PopupMenuItem(value: 'reset', child: Text('Reset to Defaults')),
              PopupMenuItem(value: 'export', child: Text('Export Settings')),
              PopupMenuItem(value: 'import', child: Text('Import Settings')),
            ],
          ),
        ],
      ),
      
      body: Consumer<PreferencesProvider>(
        builder: (context, prefsProvider, child) {
          if (prefsProvider.isLoading) {
            return Center(child: CircularProgressIndicator());
          }

          if (prefsProvider.error != null) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error, size: 64, color: Colors.red),
                  SizedBox(height: 16),
                  Text(prefsProvider.error!),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      prefsProvider.clearError();
                      prefsProvider.initialize();
                    },
                    child: Text('Retry'),
                  ),
                ],
              ),
            );
          }

          return ListView(
            padding: EdgeInsets.all(16),
            children: [
              _buildThemeSection(prefsProvider),
              SizedBox(height: 24),
              _buildChatSection(prefsProvider),
              SizedBox(height: 24),
              _buildNotificationSection(prefsProvider),
              SizedBox(height: 24),
              _buildExportSection(prefsProvider),
              SizedBox(height: 24),
              _buildLanguageSection(prefsProvider),
              SizedBox(height: 24),
              _buildAdvancedSection(prefsProvider),
            ],
          );
        },
      ),
    );
  }

  Widget _buildThemeSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Appearance',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Dark Mode'),
              subtitle: Text('Use dark theme throughout the app'),
              value: provider.isDarkMode,
              onChanged: (value) => provider.updateTheme(value),
              secondary: Icon(Icons.dark_mode),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildChatSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Chat Settings',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Text Animations'),
              subtitle: Text('Enable typewriter effect for bot responses'),
              value: provider.animationsEnabled,
              onChanged: (value) => provider.updateAnimations(value),
              secondary: Icon(Icons.animation),
            ),
            
            ListTile(
              title: Text('Font Size'),
              subtitle: Text('Current: ${provider.fontSize.toInt()}px'),
              leading: Icon(Icons.format_size),
              trailing: Text('${provider.fontSize.toInt()}'),
            ),
            
            Slider(
              value: provider.fontSize,
              min: 12.0,
              max: 24.0,
              divisions: 12,
              onChanged: (value) => provider.updateFontSize(value),
            ),
            
            ListTile(
              title: Text('Chat Persona'),
              subtitle: Text('Current: ${provider.chatPersona}'),
              leading: Icon(Icons.psychology),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () => _showPersonaDialog(provider),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildNotificationSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Notifications',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Enable Notifications'),
              subtitle: Text('Receive push notifications'),
              value: provider.notificationsEnabled,
              onChanged: (value) => provider.updateNotifications(value),
              secondary: Icon(Icons.notifications),
            ),
            
            if (provider.notificationsEnabled) ...[
              SwitchListTile(
                title: Text('Sound'),
                subtitle: Text('Play sound for notifications'),
                value: provider.preferences?.notificationSound ?? true,
                onChanged: (value) => provider.updateNotificationSound(value),
                secondary: Icon(Icons.volume_up),
              ),
              
              SwitchListTile(
                title: Text('Vibration'),
                subtitle: Text('Vibrate for notifications'),
                value: provider.preferences?.notificationVibration ?? true,
                onChanged: (value) => provider.updateNotificationVibration(value),
                secondary: Icon(Icons.vibration),
              ),
            ],
          ],
        ),
      ),
    );
  }

  Widget _buildExportSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Export Settings',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Auto Export'),
              subtitle: Text('Automatically export chats'),
              value: provider.preferences?.autoExportEnabled ?? false,
              onChanged: (value) => provider.updateAutoExport(value),
              secondary: Icon(Icons.file_download),
            ),
            
            ListTile(
              title: Text('Export Format'),
              subtitle: Text('Current: ${provider.preferences?.exportFormat?.toUpperCase() ?? 'MARKDOWN'}'),
              leading: Icon(Icons.description),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () => _showExportFormatDialog(provider),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildLanguageSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Language & Region',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            ListTile(
              title: Text('Language'),
              subtitle: Text(_getLanguageName(provider.preferences?.language ?? 'en')),
              leading: Icon(Icons.language),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () => _showLanguageDialog(provider),
            ),
            
            ListTile(
              title: Text('Timezone'),
              subtitle: Text(provider.preferences?.timezone ?? 'Auto'),
              leading: Icon(Icons.schedule),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () => _showTimezoneDialog(provider),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildAdvancedSection(PreferencesProvider provider) {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Advanced',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            ListTile(
              title: Text('Reset All Settings'),
              subtitle: Text('Restore default preferences'),
              leading: Icon(Icons.restore, color: Colors.red),
              trailing: _isResetting
                  ? SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    )
                  : Icon(Icons.arrow_forward_ios),
              onTap: _isResetting ? null : () => _showResetDialog(),
            ),
            
            ListTile(
              title: Text('Clear Cache'),
              subtitle: Text('Clear local app cache'),
              leading: Icon(Icons.clear_all),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () => _showClearCacheDialog(),
            ),
          ],
        ),
      ),
    );
  }

  Future<void> _showPersonaDialog(PreferencesProvider provider) async {
    final personas = ['Friendly', 'Professional', 'Casual', 'Academic', 'Creative'];
    
    final selected = await showDialog<String>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Chat Persona'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: personas.map((persona) => RadioListTile<String>(
            title: Text(persona),
            value: persona,
            groupValue: provider.chatPersona,
            onChanged: (value) => Navigator.of(context).pop(value),
          )).toList(),
        ),
      ),
    );
    
    if (selected != null) {
      await provider.updatePersona(selected);
    }
  }

  Future<void> _showExportFormatDialog(PreferencesProvider provider) async {
    final formats = {'markdown': 'Markdown', 'json': 'JSON', 'txt': 'Plain Text'};
    
    final selected = await showDialog<String>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Export Format'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: formats.entries.map((entry) => RadioListTile<String>(
            title: Text(entry.value),
            value: entry.key,
            groupValue: provider.preferences?.exportFormat,
            onChanged: (value) => Navigator.of(context).pop(value),
          )).toList(),
        ),
      ),
    );
    
    if (selected != null) {
      await provider.updateExportFormat(selected);
    }
  }

  Future<void> _showResetDialog() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Reset All Settings'),
        content: Text('This will restore all preferences to their default values. This action cannot be undone.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.of(context).pop(true),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
            child: Text('Reset'),
          ),
        ],
      ),
    );

    if (confirmed == true) {
      setState(() {
        _isResetting = true;
      });

      try {
        final provider = Provider.of<PreferencesProvider>(context, listen: false);
        await provider.resetToDefaults();
        
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Preferences reset to defaults')),
        );
      } catch (e) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Failed to reset preferences')),
        );
      } finally {
        setState(() {
          _isResetting = false;
        });
      }
    }
  }

  String _getLanguageName(String code) {
    const languages = {
      'en': 'English',
      'es': 'Español',
      'fr': 'Français',
      'de': 'Deutsch',
      'it': 'Italiano',
      'pt': 'Português',
      'zh': '中文',
      'ja': '日本語',
      'ko': '한국어',
      'ar': 'العربية',
    };
    return languages[code] ?? 'English';
  }
}
```

---

## 🌐 HTTP API Examples

### **Get User Preferences**
```bash
curl -X GET https://api.speechbot.com/v1/users/preferences \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "preferences": {
      "theme_dark_mode": true,
      "chat_text_animations": true,
      "chat_font_size": 16.0,
      "chat_persona": "Friendly",
      "notifications_enabled": true,
      "notification_sound": true,
      "notification_vibration": true,
      "auto_export_enabled": false,
      "export_format": "markdown",
      "language": "en",
      "timezone": "America/New_York",
      "updated_at": "2025-07-15T10:30:00Z"
    }
  }
}
```

### **Update Specific Preference**
```bash
curl -X PATCH https://api.speechbot.com/v1/users/preferences \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "theme_dark_mode": false,
    "chat_font_size": 18.0
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Preferences updated successfully",
  "data": {
    "updated_fields": ["theme_dark_mode", "chat_font_size"],
    "preferences": {
      "theme_dark_mode": false,
      "chat_text_animations": true,
      "chat_font_size": 18.0,
      "chat_persona": "Friendly",
      "notifications_enabled": true,
      "notification_sound": true,
      "notification_vibration": true,
      "auto_export_enabled": false,
      "export_format": "markdown",
      "language": "en",
      "timezone": "America/New_York",
      "updated_at": "2025-07-15T11:45:30Z"
    }
  }
}
```

### **Reset All Preferences**
```bash
curl -X POST https://api.speechbot.com/v1/users/preferences/reset \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "message": "Preferences reset to defaults",
  "data": {
    "preferences": {
      "theme_dark_mode": false,
      "chat_text_animations": true,
      "chat_font_size": 16.0,
      "chat_persona": "Friendly",
      "notifications_enabled": true,
      "notification_sound": true,
      "notification_vibration": true,
      "auto_export_enabled": false,
      "export_format": "markdown",
      "language": "en",
      "timezone": "UTC",
      "updated_at": "2025-07-15T12:00:00Z"
    }
  }
}
```

### **Export Preferences**
```bash
curl -X GET https://api.speechbot.com/v1/users/preferences/export \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "export": {
      "version": "1.0",
      "exported_at": "2025-07-15T12:15:00Z",
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "preferences": {
        "theme_dark_mode": true,
        "chat_text_animations": true,
        "chat_font_size": 16.0,
        "chat_persona": "Friendly",
        "notifications_enabled": true,
        "notification_sound": true,
        "notification_vibration": true,
        "auto_export_enabled": false,
        "export_format": "markdown",
        "language": "en",
        "timezone": "America/New_York"
      }
    }
  }
}
```

### **Import Preferences**
```bash
curl -X POST https://api.speechbot.com/v1/users/preferences/import \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "preferences": {
      "theme_dark_mode": true,
      "chat_text_animations": false,
      "chat_font_size": 18.0,
      "chat_persona": "Professional",
      "notifications_enabled": false,
      "export_format": "json",
      "language": "es"
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Preferences imported successfully",
  "data": {
    "imported_fields": [
      "theme_dark_mode",
      "chat_text_animations", 
      "chat_font_size",
      "chat_persona",
      "notifications_enabled",
      "export_format",
      "language"
    ],
    "skipped_fields": [],
    "preferences": {
      "theme_dark_mode": true,
      "chat_text_animations": false,
      "chat_font_size": 18.0,
      "chat_persona": "Professional",
      "notifications_enabled": false,
      "notification_sound": true,
      "notification_vibration": true,
      "auto_export_enabled": false,
      "export_format": "json",
      "language": "es",
      "timezone": "America/New_York",
      "updated_at": "2025-07-15T12:30:00Z"
    }
  }
}
```

---

## 🔄 Real-time Sync Example

### **Preferences Sync Service**
```dart
// lib/services/preferences_sync_service.dart
class PreferencesSyncService {
  static Timer? _syncTimer;
  static bool _isSyncing = false;
  
  static void startPeriodicSync() {
    _syncTimer = Timer.periodic(Duration(minutes: 5), (timer) async {
      await syncPreferences();
    });
  }
  
  static void stopPeriodicSync() {
    _syncTimer?.cancel();
    _syncTimer = null;
  }
  
  static Future<void> syncPreferences() async {
    if (_isSyncing) return;
    
    _isSyncing = true;
    
    try {
      final prefsProvider = Get.find<PreferencesProvider>();
      final authProvider = Get.find<AuthProvider>();
      
      if (!authProvider.isAuthenticated) return;
      
      // Get local preferences timestamp
      final prefs = await SharedPreferences.getInstance();
      final localTimestamp = prefs.getString('preferences_last_sync');
      
      // Check if server has newer preferences
      final serverInfo = await PreferencesService.getPreferencesInfo();
      final serverTimestamp = serverInfo['updated_at'];
      
      if (localTimestamp == null || 
          DateTime.parse(serverTimestamp).isAfter(DateTime.parse(localTimestamp))) {
        // Server has newer preferences, download them
        final serverPrefs = await PreferencesService.getPreferences();
        await prefsProvider._mergeAndSavePreferences(serverPrefs);
      } else {
        // Local preferences are newer or same, upload them
        await PreferencesService.uploadPreferences(prefsProvider.preferences!);
      }
      
      // Update sync timestamp
      await prefs.setString('preferences_last_sync', DateTime.now().toIso8601String());
      
    } catch (e) {
      print('Preferences sync failed: $e');
      // Continue silently, will retry next cycle
    } finally {
      _isSyncing = false;
    }
  }
  
  static Future<void> forceSyncToServer() async {
    try {
      final prefsProvider = Get.find<PreferencesProvider>();
      if (prefsProvider.preferences != null) {
        await PreferencesService.uploadPreferences(prefsProvider.preferences!);
        
        final prefs = await SharedPreferences.getInstance();
        await prefs.setString('preferences_last_sync', DateTime.now().toIso8601String());
      }
    } catch (e) {
      throw Exception('Failed to sync preferences to server: $e');
    }
  }
}
```

---

**These examples provide comprehensive user preferences management with real-time syncing, local storage, and seamless cloud backup functionality.**
