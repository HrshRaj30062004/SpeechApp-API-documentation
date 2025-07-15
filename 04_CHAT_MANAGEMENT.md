---
layout: default
title: Chat Management
nav_order: 4
---

# 💬 Chat Management System

**Document:** Chat Management API Specification  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Chat Management Overview

### **Current Flutter App Implementation**
The app currently manages chats using SharedPreferences with this structure:

```dart
// Current chat management in chat_screen.dart
List<Map<String, dynamic>> chatSessions = [];
String currentChatId = '';

// Chat session structure
{
  'id': 'chat_1684761234567',
  'title': 'Custom Chat Title',
  'messages': [
    {
      'id': 'msg_1684761234567_0',
      'role': 'user',
      'content': 'Hello!',
      'timestamp': '2025-07-15T10:30:00Z'
    }
  ]
}
```

### **Target API Implementation**
- **Scalable Storage**: PostgreSQL backend with proper indexing
- **Real-time Sync**: WebSocket connections for live updates
- **Advanced Search**: Full-text search across chat content
- **Chat Organization**: Folders, tags, favorites, and archiving
- **Collaboration**: Shared chats and team features (future)

---

## 💬 Chat Management Endpoints

### **GET /chats**
Retrieve user's chat sessions with pagination and filtering.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `search`: Search query for chat titles and content
- `folder_id`: Filter by folder ID
- `is_favorite`: Filter by favorite status (true/false)
- `is_archived`: Include archived chats (default: false)
- `sort_by`: Sort field (created_at, updated_at, title, message_count)
- `sort_order`: Sort direction (asc, desc)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "chats": [
      {
        "id": "chat_1684761234567",
        "title": "Project Planning Discussion",
        "created_at": "2025-07-15T10:30:00Z",
        "updated_at": "2025-07-15T14:45:00Z",
        "message_count": 24,
        "last_message": {
          "role": "bot",
          "content": "I'd be happy to help you with that analysis...",
          "timestamp": "2025-07-15T14:45:00Z"
        },
        "is_favorite": false,
        "is_archived": false,
        "folder_id": "folder_work",
        "tags": ["planning", "productivity"],
        "metadata": {
          "total_characters": 15420,
          "estimated_tokens": 3855,
          "languages_detected": ["en"]
        }
      },
      {
        "id": "chat_1684675834567",
        "title": "Creative Writing Ideas",
        "created_at": "2025-07-14T08:15:00Z",
        "updated_at": "2025-07-14T16:20:00Z",
        "message_count": 12,
        "last_message": {
          "role": "user",
          "content": "Thanks for all the suggestions!",
          "timestamp": "2025-07-14T16:20:00Z"
        },
        "is_favorite": true,
        "is_archived": false,
        "folder_id": null,
        "tags": ["creative", "writing"],
        "metadata": {
          "total_characters": 8430,
          "estimated_tokens": 2107,
          "languages_detected": ["en"]
        }
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 3,
      "total_chats": 47,
      "has_next": true,
      "has_previous": false,
      "per_page": 20
    },
    "filters": {
      "active_search": null,
      "active_folder": null,
      "show_archived": false,
      "show_favorites_only": false
    }
  }
}
```

---

### **POST /chats**
Create a new chat session.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "New Chat Session", // Optional, auto-generated if not provided
  "folder_id": "folder_work", // Optional
  "tags": ["research", "ai"], // Optional
  "initial_message": { // Optional first message
    "content": "Hello! I need help with a project.",
    "metadata": {
      "client_timestamp": "2025-07-15T10:30:00Z"
    }
  }
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Chat created successfully",
  "data": {
    "chat": {
      "id": "chat_1684761234567",
      "title": "New Chat Session",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T10:30:00Z",
      "message_count": 1,
      "is_favorite": false,
      "is_archived": false,
      "folder_id": "folder_work",
      "tags": ["research", "ai"],
      "metadata": {
        "total_characters": 34,
        "estimated_tokens": 8,
        "languages_detected": ["en"]
      }
    },
    "messages": [
      {
        "id": "msg_1684761234567_0",
        "chat_id": "chat_1684761234567",
        "role": "user",
        "content": "Hello! I need help with a project.",
        "timestamp": "2025-07-15T10:30:00Z",
        "metadata": {
          "client_timestamp": "2025-07-15T10:30:00Z"
        }
      }
    ]
  }
}
```

---

### **GET /chats/{chat_id}**
Retrieve specific chat session with messages.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `include_messages`: Include message history (default: true)
- `message_limit`: Number of recent messages (default: 50, max: 200)
- `message_offset`: Offset for message pagination (default: 0)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "chat": {
      "id": "chat_1684761234567",
      "title": "Project Planning Discussion",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T14:45:00Z",
      "message_count": 24,
      "is_favorite": false,
      "is_archived": false,
      "folder_id": "folder_work",
      "tags": ["planning", "productivity"],
      "metadata": {
        "total_characters": 15420,
        "estimated_tokens": 3855,
        "languages_detected": ["en"]
      }
    },
    "messages": [
      {
        "id": "msg_1684761234567_0",
        "chat_id": "chat_1684761234567",
        "role": "user",
        "content": "Hello! I need help planning a project.",
        "timestamp": "2025-07-15T10:30:00Z",
        "metadata": {
          "client_timestamp": "2025-07-15T10:30:00Z"
        }
      },
      {
        "id": "msg_1684761234567_1",
        "chat_id": "chat_1684761234567",
        "role": "bot",
        "content": "I'd be happy to help you with project planning! What type of project are you working on?",
        "full_content": "I'd be happy to help you with project planning! What type of project are you working on?",
        "timestamp": "2025-07-15T10:30:15Z",
        "metadata": {
          "generation_time_ms": 1247,
          "model_used": "gpt-4-turbo",
          "tokens_used": 23
        }
      }
    ],
    "message_pagination": {
      "total_messages": 24,
      "returned_messages": 24,
      "has_more": false,
      "next_offset": null
    }
  }
}
```

**Error Responses:**
```json
// 404 Not Found - Chat doesn't exist
{
  "success": false,
  "error": "CHAT_NOT_FOUND",
  "message": "Chat session not found"
}

// 403 Forbidden - User doesn't own chat
{
  "success": false,
  "error": "CHAT_ACCESS_DENIED",
  "message": "You don't have permission to access this chat"
}
```

---

### **PUT /chats/{chat_id}**
Update chat metadata (title, folder, tags, etc.).

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Updated Chat Title",
  "folder_id": "folder_personal",
  "tags": ["updated", "important"],
  "is_favorite": true,
  "is_archived": false
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Chat updated successfully",
  "data": {
    "chat": {
      "id": "chat_1684761234567",
      "title": "Updated Chat Title",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T15:00:00Z",
      "message_count": 24,
      "is_favorite": true,
      "is_archived": false,
      "folder_id": "folder_personal",
      "tags": ["updated", "important"],
      "metadata": {
        "total_characters": 15420,
        "estimated_tokens": 3855,
        "languages_detected": ["en"]
      }
    }
  }
}
```

---

### **DELETE /chats/{chat_id}**
Delete a chat session and all its messages.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `confirm`: Required confirmation (must be "true")

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Chat deleted successfully",
  "data": {
    "deleted_chat_id": "chat_1684761234567",
    "deleted_messages": 24,
    "deletion_timestamp": "2025-07-15T15:30:00Z"
  }
}
```

**Error Responses:**
```json
// 400 Bad Request - Missing confirmation
{
  "success": false,
  "error": "CONFIRMATION_REQUIRED",
  "message": "Deletion confirmation required"
}
```

---

### **POST /chats/{chat_id}/duplicate**
Create a copy of an existing chat.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "title": "Copy of Project Planning", // Optional
  "include_messages": true, // Include message history
  "folder_id": "folder_work" // Optional new folder
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Chat duplicated successfully",
  "data": {
    "original_chat_id": "chat_1684761234567",
    "new_chat": {
      "id": "chat_1684761334567",
      "title": "Copy of Project Planning",
      "created_at": "2025-07-15T15:45:00Z",
      "updated_at": "2025-07-15T15:45:00Z",
      "message_count": 24,
      "is_favorite": false,
      "is_archived": false,
      "folder_id": "folder_work",
      "tags": ["copy"],
      "metadata": {
        "total_characters": 15420,
        "estimated_tokens": 3855,
        "languages_detected": ["en"],
        "copied_from": "chat_1684761234567"
      }
    }
  }
}
```

---

## 📁 Folder Management

### **GET /chats/folders**
Retrieve user's chat folders.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "folders": [
      {
        "id": "folder_work",
        "name": "Work",
        "description": "Work-related conversations",
        "color": "#2196F3",
        "icon": "work",
        "chat_count": 15,
        "created_at": "2025-07-01T09:00:00Z",
        "updated_at": "2025-07-15T14:30:00Z"
      },
      {
        "id": "folder_personal",
        "name": "Personal",
        "description": "Personal conversations and ideas",
        "color": "#4CAF50",
        "icon": "person",
        "chat_count": 8,
        "created_at": "2025-07-01T09:00:00Z",
        "updated_at": "2025-07-14T16:20:00Z"
      }
    ]
  }
}
```

### **POST /chats/folders**
Create a new folder.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "Research",
  "description": "Research and analysis conversations",
  "color": "#FF9800",
  "icon": "research"
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Folder created successfully",
  "data": {
    "folder": {
      "id": "folder_research",
      "name": "Research",
      "description": "Research and analysis conversations",
      "color": "#FF9800",
      "icon": "research",
      "chat_count": 0,
      "created_at": "2025-07-15T16:00:00Z",
      "updated_at": "2025-07-15T16:00:00Z"
    }
  }
}
```

---

## 🔍 Search & Filtering

### **GET /chats/search**
Advanced search across chats and messages.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `q`: Search query (required)
- `search_in`: Where to search (title, content, both) - default: both
- `folder_id`: Limit search to specific folder
- `date_from`: Start date filter (ISO 8601)
- `date_to`: End date filter (ISO 8601)
- `tags`: Comma-separated tag filters
- `page`: Page number (default: 1)
- `limit`: Results per page (default: 20, max: 100)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "type": "chat",
        "chat": {
          "id": "chat_1684761234567",
          "title": "Project Planning Discussion",
          "created_at": "2025-07-15T10:30:00Z",
          "message_count": 24,
          "folder_name": "Work",
          "tags": ["planning", "productivity"]
        },
        "matches": [
          {
            "field": "title",
            "snippet": "Project <mark>Planning</mark> Discussion",
            "score": 0.95
          }
        ]
      },
      {
        "type": "message",
        "chat": {
          "id": "chat_1684675834567",
          "title": "Creative Writing Ideas",
          "folder_name": null
        },
        "message": {
          "id": "msg_1684675834567_3",
          "role": "bot",
          "timestamp": "2025-07-14T09:30:00Z",
          "snippet": "Here's a <mark>planning</mark> template you can use for your story structure..."
        },
        "matches": [
          {
            "field": "content",
            "snippet": "Here's a <mark>planning</mark> template you can use...",
            "score": 0.87
          }
        ]
      }
    ],
    "search_metadata": {
      "query": "planning",
      "total_results": 12,
      "search_time_ms": 45,
      "filters_applied": {
        "search_in": "both",
        "folder_id": null,
        "date_range": null,
        "tags": null
      }
    },
    "pagination": {
      "current_page": 1,
      "total_pages": 1,
      "total_results": 12,
      "has_next": false,
      "has_previous": false
    }
  }
}
```

---

## 📱 Flutter Integration

### **Chat Service Implementation**
```dart
class ChatService {
  static Future<ChatListResponse> getChats({
    int page = 1,
    int limit = 20,
    String? search,
    String? folderId,
    bool? isFavorite,
    bool includeArchived = false,
    String sortBy = 'updated_at',
    String sortOrder = 'desc',
  }) async {
    final params = <String, dynamic>{
      'page': page,
      'limit': limit,
      'sort_by': sortBy,
      'sort_order': sortOrder,
      'is_archived': includeArchived,
    };
    
    if (search != null) params['search'] = search;
    if (folderId != null) params['folder_id'] = folderId;
    if (isFavorite != null) params['is_favorite'] = isFavorite;
    
    try {
      final response = await ApiService.get('/chats', queryParameters: params);
      return ChatListResponse.fromJson(response.data);
    } catch (e) {
      throw ChatException.fromError(e);
    }
  }
  
  static Future<ChatDetailResponse> getChat(
    String chatId, {
    bool includeMessages = true,
    int messageLimit = 50,
    int messageOffset = 0,
  }) async {
    final params = {
      'include_messages': includeMessages,
      'message_limit': messageLimit,
      'message_offset': messageOffset,
    };
    
    try {
      final response = await ApiService.get('/chats/$chatId', queryParameters: params);
      return ChatDetailResponse.fromJson(response.data);
    } catch (e) {
      throw ChatException.fromError(e);
    }
  }
  
  static Future<Chat> createChat({
    String? title,
    String? folderId,
    List<String>? tags,
    Message? initialMessage,
  }) async {
    final data = <String, dynamic>{};
    
    if (title != null) data['title'] = title;
    if (folderId != null) data['folder_id'] = folderId;
    if (tags != null) data['tags'] = tags;
    if (initialMessage != null) {
      data['initial_message'] = {
        'content': initialMessage.content,
        'metadata': {
          'client_timestamp': initialMessage.timestamp.toIso8601String(),
        },
      };
    }
    
    try {
      final response = await ApiService.post('/chats', data: data);
      return Chat.fromJson(response.data['data']['chat']);
    } catch (e) {
      throw ChatException.fromError(e);
    }
  }
  
  static Future<Chat> updateChat(
    String chatId, {
    String? title,
    String? folderId,
    List<String>? tags,
    bool? isFavorite,
    bool? isArchived,
  }) async {
    final data = <String, dynamic>{};
    
    if (title != null) data['title'] = title;
    if (folderId != null) data['folder_id'] = folderId;
    if (tags != null) data['tags'] = tags;
    if (isFavorite != null) data['is_favorite'] = isFavorite;
    if (isArchived != null) data['is_archived'] = isArchived;
    
    try {
      final response = await ApiService.put('/chats/$chatId', data: data);
      return Chat.fromJson(response.data['data']['chat']);
    } catch (e) {
      throw ChatException.fromError(e);
    }
  }
  
  static Future<void> deleteChat(String chatId) async {
    try {
      await ApiService.delete('/chats/$chatId', queryParameters: {
        'confirm': 'true',
      });
    } catch (e) {
      throw ChatException.fromError(e);
    }
  }
}
```

### **Chat Provider for State Management**
```dart
class ChatProvider extends ChangeNotifier {
  List<Chat> _chats = [];
  Chat? _currentChat;
  bool _isLoading = true;
  String? _error;
  String? _searchQuery;
  String? _selectedFolderId;
  
  List<Chat> get chats => _searchQuery?.isEmpty ?? true
      ? _chats
      : _chats.where((chat) => 
          chat.title.toLowerCase().contains(_searchQuery!.toLowerCase())
        ).toList();
  
  Chat? get currentChat => _currentChat;
  bool get isLoading => _isLoading;
  String? get error => _error;
  String? get searchQuery => _searchQuery;
  
  Future<void> loadChats({bool refresh = false}) async {
    try {
      if (refresh || _chats.isEmpty) {
        _isLoading = true;
        notifyListeners();
      }
      
      final response = await ChatService.getChats(
        folderId: _selectedFolderId,
      );
      
      _chats = response.chats;
      _error = null;
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<void> createNewChat({String? title, Message? initialMessage}) async {
    try {
      final chat = await ChatService.createChat(
        title: title,
        folderId: _selectedFolderId,
        initialMessage: initialMessage,
      );
      
      _chats.insert(0, chat);
      _currentChat = chat;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  Future<void> updateChatTitle(String chatId, String newTitle) async {
    try {
      final updatedChat = await ChatService.updateChat(
        chatId,
        title: newTitle,
      );
      
      final index = _chats.indexWhere((chat) => chat.id == chatId);
      if (index != -1) {
        _chats[index] = updatedChat;
        if (_currentChat?.id == chatId) {
          _currentChat = updatedChat;
        }
        notifyListeners();
      }
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  Future<void> deleteChat(String chatId) async {
    try {
      await ChatService.deleteChat(chatId);
      
      _chats.removeWhere((chat) => chat.id == chatId);
      if (_currentChat?.id == chatId) {
        _currentChat = _chats.isNotEmpty ? _chats.first : null;
      }
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  void setSearchQuery(String query) {
    _searchQuery = query;
    notifyListeners();
  }
  
  void setSelectedFolder(String? folderId) {
    _selectedFolderId = folderId;
    loadChats(refresh: true);
  }
  
  Future<void> toggleFavorite(String chatId) async {
    final chat = _chats.firstWhere((c) => c.id == chatId);
    await updateChatFavorite(chatId, !chat.isFavorite);
  }
  
  Future<void> updateChatFavorite(String chatId, bool isFavorite) async {
    try {
      final updatedChat = await ChatService.updateChat(
        chatId,
        isFavorite: isFavorite,
      );
      
      final index = _chats.indexWhere((chat) => chat.id == chatId);
      if (index != -1) {
        _chats[index] = updatedChat;
        if (_currentChat?.id == chatId) {
          _currentChat = updatedChat;
        }
        notifyListeners();
      }
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
}
```

### **Chat Model Classes**
```dart
class Chat {
  final String id;
  final String title;
  final DateTime createdAt;
  final DateTime updatedAt;
  final int messageCount;
  final Message? lastMessage;
  final bool isFavorite;
  final bool isArchived;
  final String? folderId;
  final List<String> tags;
  final ChatMetadata metadata;
  
  const Chat({
    required this.id,
    required this.title,
    required this.createdAt,
    required this.updatedAt,
    required this.messageCount,
    this.lastMessage,
    required this.isFavorite,
    required this.isArchived,
    this.folderId,
    required this.tags,
    required this.metadata,
  });
  
  factory Chat.fromJson(Map<String, dynamic> json) {
    return Chat(
      id: json['id'],
      title: json['title'],
      createdAt: DateTime.parse(json['created_at']),
      updatedAt: DateTime.parse(json['updated_at']),
      messageCount: json['message_count'],
      lastMessage: json['last_message'] != null
          ? Message.fromJson(json['last_message'])
          : null,
      isFavorite: json['is_favorite'],
      isArchived: json['is_archived'],
      folderId: json['folder_id'],
      tags: List<String>.from(json['tags'] ?? []),
      metadata: ChatMetadata.fromJson(json['metadata']),
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'title': title,
      'created_at': createdAt.toIso8601String(),
      'updated_at': updatedAt.toIso8601String(),
      'message_count': messageCount,
      'last_message': lastMessage?.toJson(),
      'is_favorite': isFavorite,
      'is_archived': isArchived,
      'folder_id': folderId,
      'tags': tags,
      'metadata': metadata.toJson(),
    };
  }
}

class ChatListResponse {
  final List<Chat> chats;
  final Pagination pagination;
  final Map<String, dynamic> filters;
  
  const ChatListResponse({
    required this.chats,
    required this.pagination,
    required this.filters,
  });
  
  factory ChatListResponse.fromJson(Map<String, dynamic> json) {
    return ChatListResponse(
      chats: (json['data']['chats'] as List)
          .map((chat) => Chat.fromJson(chat))
          .toList(),
      pagination: Pagination.fromJson(json['data']['pagination']),
      filters: json['data']['filters'],
    );
  }
}
```

---

## 🔄 Migration Strategy

### **Hybrid Approach During Transition**
```dart
class HybridChatService {
  // Gradual migration from local to API storage
  static Future<List<Chat>> getChats() async {
    try {
      // Try API first
      final apiChats = await ChatService.getChats();
      await _cacheChatsLocally(apiChats.chats);
      return apiChats.chats;
    } catch (e) {
      // Fallback to local storage
      return await _getLocalChats();
    }
  }
  
  static Future<void> syncLocalChatsToAPI() async {
    final localChats = await _getLocalChats();
    for (final chat in localChats) {
      if (!chat.metadata.isSynced) {
        try {
          await _uploadChatToAPI(chat);
          await _markChatAsSynced(chat.id);
        } catch (e) {
          // Queue for retry
          await _queueChatForSync(chat);
        }
      }
    }
  }
}
```

---

**This chat management system provides scalable, feature-rich conversation handling while maintaining compatibility with the existing Flutter app structure.**
