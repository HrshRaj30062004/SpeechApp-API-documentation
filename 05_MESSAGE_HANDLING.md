# 📨 Message Handling System

**Document:** Message Handling API Specification  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Message System Overview

### **Current Flutter App Implementation**
The app currently handles messages within chat sessions with this structure:

```dart
// Current message structure in chat_screen.dart
{
  'id': 'msg_1684761234567_0',
  'role': 'user', // or 'bot'
  'content': 'Hello, how can you help me today?',
  'timestamp': '2025-07-15T10:30:00Z'
}

// Typewriter animation for bot responses
String fullBotResponse = "Full response text here...";
String displayedText = ""; // Gradually populated
```

### **Target API Implementation**
- **Real-time Messaging**: WebSocket connections for instant delivery
- **Rich Content Support**: Text, markdown, code blocks, file attachments
- **Message Threading**: Reply to specific messages and conversations
- **Advanced Features**: Message reactions, editing, deletion
- **Delivery Tracking**: Read receipts and delivery confirmations
- **Offline Support**: Queue messages when disconnected

---

## 📨 Message Endpoints

### **GET /chats/{chat_id}/messages**
Retrieve messages from a specific chat with pagination.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `limit`: Number of messages to return (default: 50, max: 200)
- `offset`: Offset for pagination (default: 0)
- `before`: Get messages before this message ID
- `after`: Get messages after this message ID
- `order`: Sort order (asc, desc) - default: desc (newest first)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "msg_1684761234567_5",
        "chat_id": "chat_1684761234567",
        "role": "bot",
        "content": "I'd be happy to help you with that analysis. Here's what I found:",
        "full_content": "I'd be happy to help you with that analysis. Here's what I found:",
        "content_type": "text",
        "timestamp": "2025-07-15T14:45:00Z",
        "status": "delivered",
        "metadata": {
          "generation_time_ms": 1247,
          "model_used": "gpt-4-turbo",
          "tokens_used": {
            "prompt": 245,
            "completion": 89,
            "total": 334
          },
          "temperature": 0.7,
          "max_tokens": 2048
        },
        "reactions": [],
        "thread_id": null,
        "reply_to": null,
        "edited_at": null,
        "deleted_at": null
      },
      {
        "id": "msg_1684761234567_4",
        "chat_id": "chat_1684761234567",
        "role": "user",
        "content": "Can you analyze this data for me?",
        "content_type": "text",
        "timestamp": "2025-07-15T14:44:30Z",
        "status": "read",
        "metadata": {
          "client_timestamp": "2025-07-15T14:44:28Z",
          "device_info": {
            "platform": "android",
            "app_version": "1.0.0"
          }
        },
        "reactions": [
          {
            "emoji": "👍",
            "user_id": "user_550e8400",
            "timestamp": "2025-07-15T14:45:30Z"
          }
        ],
        "thread_id": null,
        "reply_to": null,
        "edited_at": null,
        "deleted_at": null,
        "attachments": [
          {
            "id": "att_1684761234567",
            "filename": "data.csv",
            "content_type": "text/csv",
            "size": 15420,
            "url": "https://api.speechbot.com/v1/attachments/att_1684761234567",
            "thumbnail_url": null
          }
        ]
      }
    ],
    "pagination": {
      "total_messages": 24,
      "returned_messages": 2,
      "has_more": true,
      "next_offset": 2,
      "previous_offset": null
    },
    "chat_info": {
      "id": "chat_1684761234567",
      "title": "Data Analysis Discussion",
      "updated_at": "2025-07-15T14:45:00Z"
    }
  }
}
```

---

### **POST /chats/{chat_id}/messages**
Send a new message to a chat.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "content": "Hello! Can you help me with a coding problem?",
  "content_type": "text",
  "reply_to": null, // Optional: Message ID to reply to
  "thread_id": null, // Optional: Thread ID for threaded conversations
  "metadata": {
    "client_timestamp": "2025-07-15T15:00:00Z",
    "device_info": {
      "platform": "android",
      "app_version": "1.0.0"
    }
  },
  "attachments": [ // Optional file attachments
    {
      "filename": "code.py",
      "content_type": "text/x-python",
      "size": 1024,
      "data": "base64-encoded-file-content"
    }
  ]
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Message sent successfully",
  "data": {
    "message": {
      "id": "msg_1684761234567_6",
      "chat_id": "chat_1684761234567",
      "role": "user",
      "content": "Hello! Can you help me with a coding problem?",
      "content_type": "text",
      "timestamp": "2025-07-15T15:00:00Z",
      "status": "sent",
      "metadata": {
        "client_timestamp": "2025-07-15T15:00:00Z",
        "device_info": {
          "platform": "android",
          "app_version": "1.0.0"
        }
      },
      "reactions": [],
      "thread_id": null,
      "reply_to": null,
      "edited_at": null,
      "deleted_at": null,
      "attachments": [
        {
          "id": "att_1684761234568",
          "filename": "code.py",
          "content_type": "text/x-python",
          "size": 1024,
          "url": "https://api.speechbot.com/v1/attachments/att_1684761234568"
        }
      ]
    },
    "chat_updated": {
      "id": "chat_1684761234567",
      "updated_at": "2025-07-15T15:00:00Z",
      "message_count": 25
    }
  }
}
```

**Error Responses:**
```json
// 400 Bad Request - Invalid content
{
  "success": false,
  "error": "INVALID_MESSAGE_CONTENT",
  "message": "Message content cannot be empty",
  "details": {
    "content": ["Content is required"]
  }
}

// 413 Payload Too Large - Attachment too large
{
  "success": false,
  "error": "ATTACHMENT_TOO_LARGE",
  "message": "Attachment size exceeds maximum limit",
  "details": {
    "max_size": 10485760,
    "provided_size": 15728640
  }
}
```

---

### **PUT /chats/{chat_id}/messages/{message_id}**
Edit an existing message (user messages only).

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "content": "Updated message content here",
  "metadata": {
    "edit_reason": "Fixed typo",
    "client_timestamp": "2025-07-15T15:05:00Z"
  }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Message updated successfully",
  "data": {
    "message": {
      "id": "msg_1684761234567_6",
      "chat_id": "chat_1684761234567",
      "role": "user",
      "content": "Updated message content here",
      "content_type": "text",
      "timestamp": "2025-07-15T15:00:00Z",
      "status": "edited",
      "edited_at": "2025-07-15T15:05:00Z",
      "edit_history": [
        {
          "content": "Hello! Can you help me with a coding problem?",
          "edited_at": "2025-07-15T15:05:00Z",
          "edit_reason": "Fixed typo"
        }
      ]
    }
  }
}
```

**Error Responses:**
```json
// 403 Forbidden - Cannot edit bot messages
{
  "success": false,
  "error": "EDIT_NOT_ALLOWED",
  "message": "Bot messages cannot be edited"
}

// 409 Conflict - Message too old to edit
{
  "success": false,
  "error": "EDIT_TIME_EXPIRED",
  "message": "Messages can only be edited within 24 hours",
  "details": {
    "message_age_hours": 48,
    "max_edit_hours": 24
  }
}
```

---

### **DELETE /chats/{chat_id}/messages/{message_id}**
Delete a message (soft delete with option for hard delete).

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `hard_delete`: Permanently delete (default: false)
- `delete_for`: Who to delete for (me, everyone) - default: me

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Message deleted successfully",
  "data": {
    "message_id": "msg_1684761234567_6",
    "deletion_type": "soft",
    "deleted_for": "me",
    "deleted_at": "2025-07-15T15:10:00Z",
    "can_restore": true
  }
}
```

---

### **POST /chats/{chat_id}/messages/{message_id}/reactions**
Add a reaction to a message.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "emoji": "👍",
  "action": "add" // or "remove"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Reaction added successfully",
  "data": {
    "message_id": "msg_1684761234567_4",
    "reaction": {
      "emoji": "👍",
      "user_id": "user_550e8400",
      "timestamp": "2025-07-15T15:15:00Z"
    },
    "total_reactions": [
      {
        "emoji": "👍",
        "count": 1,
        "users": ["user_550e8400"]
      }
    ]
  }
}
```

---

## 🔄 Real-time Messaging (WebSocket)

### **WebSocket Connection**
```javascript
// WebSocket endpoint for real-time messaging
wss://api.speechbot.com/v1/chats/{chat_id}/ws?token={jwt_token}
```

### **WebSocket Message Types**

#### **Outgoing Messages (Client → Server)**

**Send Message:**
```json
{
  "type": "send_message",
  "data": {
    "content": "Hello, this is a real-time message!",
    "content_type": "text",
    "client_message_id": "temp_msg_12345",
    "metadata": {
      "client_timestamp": "2025-07-15T15:20:00Z"
    }
  }
}
```

**Typing Indicator:**
```json
{
  "type": "typing",
  "data": {
    "is_typing": true
  }
}
```

**Mark as Read:**
```json
{
  "type": "mark_read",
  "data": {
    "message_id": "msg_1684761234567_5"
  }
}
```

#### **Incoming Messages (Server → Client)**

**New Message:**
```json
{
  "type": "new_message",
  "data": {
    "message": {
      "id": "msg_1684761234567_7",
      "chat_id": "chat_1684761234567",
      "role": "bot",
      "content": "Here's my response to your question...",
      "content_type": "text",
      "timestamp": "2025-07-15T15:20:15Z",
      "status": "delivered"
    }
  }
}
```

**Message Status Update:**
```json
{
  "type": "message_status",
  "data": {
    "message_id": "msg_1684761234567_6",
    "status": "read",
    "timestamp": "2025-07-15T15:21:00Z"
  }
}
```

**Typing Indicator:**
```json
{
  "type": "typing_indicator",
  "data": {
    "user_id": "bot",
    "is_typing": true,
    "typing_text": "SpeechBot is thinking..."
  }
}
```

**Bot Response Streaming:**
```json
{
  "type": "bot_response_chunk",
  "data": {
    "message_id": "msg_1684761234567_7",
    "chunk": "Here's my ",
    "chunk_index": 0,
    "is_complete": false
  }
}

{
  "type": "bot_response_chunk",
  "data": {
    "message_id": "msg_1684761234567_7",
    "chunk": "response to your question...",
    "chunk_index": 1,
    "is_complete": true
  }
}
```

**Error Message:**
```json
{
  "type": "error",
  "data": {
    "error_code": "MESSAGE_SEND_FAILED",
    "message": "Failed to send message",
    "client_message_id": "temp_msg_12345"
  }
}
```

---

## 📱 Flutter WebSocket Integration

### **WebSocket Service**
```dart
class MessageWebSocketService {
  WebSocketChannel? _channel;
  StreamController<MessageEvent> _messageController = StreamController.broadcast();
  String? _currentChatId;
  
  Stream<MessageEvent> get messageStream => _messageController.stream;
  
  Future<void> connect(String chatId) async {
    await disconnect();
    _currentChatId = chatId;
    
    final token = await AuthService.getAccessToken();
    final uri = Uri.parse('wss://api.speechbot.com/v1/chats/$chatId/ws?token=$token');
    
    try {
      _channel = WebSocketChannel.connect(uri);
      
      _channel!.stream.listen(
        (message) => _handleIncomingMessage(jsonDecode(message)),
        onError: (error) => _handleWebSocketError(error),
        onDone: () => _handleWebSocketClosed(),
      );
      
      _sendMessage({
        'type': 'connection_established',
        'data': {
          'chat_id': chatId,
          'timestamp': DateTime.now().toIso8601String(),
        }
      });
    } catch (e) {
      throw WebSocketException('Failed to connect: $e');
    }
  }
  
  Future<void> disconnect() async {
    await _channel?.sink.close();
    _channel = null;
    _currentChatId = null;
  }
  
  void sendMessage(String content, {String? clientMessageId}) {
    if (_channel == null) throw WebSocketException('Not connected');
    
    _sendMessage({
      'type': 'send_message',
      'data': {
        'content': content,
        'content_type': 'text',
        'client_message_id': clientMessageId ?? _generateClientMessageId(),
        'metadata': {
          'client_timestamp': DateTime.now().toIso8601String(),
          'device_info': {
            'platform': Platform.operatingSystem,
            'app_version': '1.0.0',
          }
        }
      }
    });
  }
  
  void sendTypingIndicator(bool isTyping) {
    if (_channel == null) return;
    
    _sendMessage({
      'type': 'typing',
      'data': {
        'is_typing': isTyping,
      }
    });
  }
  
  void markMessageAsRead(String messageId) {
    if (_channel == null) return;
    
    _sendMessage({
      'type': 'mark_read',
      'data': {
        'message_id': messageId,
      }
    });
  }
  
  void _sendMessage(Map<String, dynamic> message) {
    _channel?.sink.add(jsonEncode(message));
  }
  
  void _handleIncomingMessage(Map<String, dynamic> data) {
    final type = data['type'] as String;
    final payload = data['data'] as Map<String, dynamic>;
    
    switch (type) {
      case 'new_message':
        _messageController.add(NewMessageEvent(
          Message.fromJson(payload['message'])
        ));
        break;
        
      case 'message_status':
        _messageController.add(MessageStatusEvent(
          messageId: payload['message_id'],
          status: payload['status'],
          timestamp: DateTime.parse(payload['timestamp']),
        ));
        break;
        
      case 'typing_indicator':
        _messageController.add(TypingIndicatorEvent(
          userId: payload['user_id'],
          isTyping: payload['is_typing'],
          typingText: payload['typing_text'],
        ));
        break;
        
      case 'bot_response_chunk':
        _messageController.add(BotResponseChunkEvent(
          messageId: payload['message_id'],
          chunk: payload['chunk'],
          chunkIndex: payload['chunk_index'],
          isComplete: payload['is_complete'],
        ));
        break;
        
      case 'error':
        _messageController.add(ErrorEvent(
          errorCode: payload['error_code'],
          message: payload['message'],
          clientMessageId: payload['client_message_id'],
        ));
        break;
    }
  }
  
  String _generateClientMessageId() {
    return 'temp_${DateTime.now().millisecondsSinceEpoch}_${Random().nextInt(1000)}';
  }
}
```

### **Message Provider with Real-time Support**
```dart
class MessageProvider extends ChangeNotifier {
  final MessageWebSocketService _webSocketService = MessageWebSocketService();
  final MessageService _messageService = MessageService();
  
  List<Message> _messages = [];
  bool _isLoading = true;
  bool _isConnected = false;
  bool _isTyping = false;
  String? _error;
  String? _currentChatId;
  Map<String, Message> _pendingMessages = {}; // Client-side pending messages
  
  List<Message> get messages => _messages;
  bool get isLoading => _isLoading;
  bool get isConnected => _isConnected;
  bool get isTyping => _isTyping;
  String? get error => _error;
  
  Future<void> initializeChat(String chatId) async {
    try {
      _currentChatId = chatId;
      _isLoading = true;
      notifyListeners();
      
      // Load message history
      final response = await _messageService.getMessages(chatId);
      _messages = response.messages;
      
      // Connect WebSocket
      await _webSocketService.connect(chatId);
      _isConnected = true;
      
      // Listen for real-time events
      _webSocketService.messageStream.listen(_handleMessageEvent);
      
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<void> sendMessage(String content) async {
    if (_currentChatId == null) return;
    
    final clientMessageId = _generateClientMessageId();
    
    // Create optimistic message
    final optimisticMessage = Message(
      id: clientMessageId,
      chatId: _currentChatId!,
      role: 'user',
      content: content,
      contentType: 'text',
      timestamp: DateTime.now(),
      status: 'sending',
      metadata: MessageMetadata(
        clientTimestamp: DateTime.now(),
      ),
    );
    
    // Add to UI immediately
    _messages.add(optimisticMessage);
    _pendingMessages[clientMessageId] = optimisticMessage;
    notifyListeners();
    
    // Send via WebSocket
    if (_isConnected) {
      _webSocketService.sendMessage(content, clientMessageId: clientMessageId);
    } else {
      // Fallback to REST API
      try {
        final sentMessage = await _messageService.sendMessage(_currentChatId!, content);
        _replacePendingMessage(clientMessageId, sentMessage);
      } catch (e) {
        _markMessageAsFailed(clientMessageId);
      }
    }
  }
  
  void startTyping() {
    if (_isConnected) {
      _webSocketService.sendTypingIndicator(true);
    }
  }
  
  void stopTyping() {
    if (_isConnected) {
      _webSocketService.sendTypingIndicator(false);
    }
  }
  
  void _handleMessageEvent(MessageEvent event) {
    switch (event.type) {
      case MessageEventType.newMessage:
        final newMessageEvent = event as NewMessageEvent;
        _addNewMessage(newMessageEvent.message);
        break;
        
      case MessageEventType.messageStatus:
        final statusEvent = event as MessageStatusEvent;
        _updateMessageStatus(statusEvent.messageId, statusEvent.status);
        break;
        
      case MessageEventType.typingIndicator:
        final typingEvent = event as TypingIndicatorEvent;
        _updateTypingIndicator(typingEvent.isTyping);
        break;
        
      case MessageEventType.botResponseChunk:
        final chunkEvent = event as BotResponseChunkEvent;
        _handleBotResponseChunk(chunkEvent);
        break;
        
      case MessageEventType.error:
        final errorEvent = event as ErrorEvent;
        _handleMessageError(errorEvent);
        break;
    }
  }
  
  void _addNewMessage(Message message) {
    // Check if this is a confirmation of a pending message
    final pendingMessage = _pendingMessages.values
        .firstWhereOrNull((m) => m.content == message.content);
    
    if (pendingMessage != null) {
      _replacePendingMessage(pendingMessage.id, message);
    } else {
      _messages.add(message);
      notifyListeners();
    }
  }
  
  void _handleBotResponseChunk(BotResponseChunkEvent event) {
    final existingMessageIndex = _messages
        .indexWhere((m) => m.id == event.messageId);
    
    if (existingMessageIndex != -1) {
      // Update existing message with new chunk
      final currentMessage = _messages[existingMessageIndex];
      final updatedContent = currentMessage.content + event.chunk;
      
      _messages[existingMessageIndex] = currentMessage.copyWith(
        content: updatedContent,
        status: event.isComplete ? 'delivered' : 'typing',
      );
    } else {
      // Create new message for streaming response
      final newMessage = Message(
        id: event.messageId,
        chatId: _currentChatId!,
        role: 'bot',
        content: event.chunk,
        contentType: 'text',
        timestamp: DateTime.now(),
        status: event.isComplete ? 'delivered' : 'typing',
      );
      
      _messages.add(newMessage);
    }
    
    notifyListeners();
  }
  
  void _replacePendingMessage(String clientMessageId, Message actualMessage) {
    final index = _messages.indexWhere((m) => m.id == clientMessageId);
    if (index != -1) {
      _messages[index] = actualMessage;
      _pendingMessages.remove(clientMessageId);
      notifyListeners();
    }
  }
  
  void _markMessageAsFailed(String clientMessageId) {
    final index = _messages.indexWhere((m) => m.id == clientMessageId);
    if (index != -1) {
      _messages[index] = _messages[index].copyWith(status: 'failed');
      notifyListeners();
    }
  }
  
  void _updateTypingIndicator(bool isTyping) {
    _isTyping = isTyping;
    notifyListeners();
  }
  
  @override
  void dispose() {
    _webSocketService.disconnect();
    super.dispose();
  }
}
```

### **Message Models**
```dart
class Message {
  final String id;
  final String chatId;
  final String role;
  final String content;
  final String contentType;
  final DateTime timestamp;
  final String status;
  final MessageMetadata? metadata;
  final List<Reaction> reactions;
  final String? threadId;
  final String? replyTo;
  final DateTime? editedAt;
  final DateTime? deletedAt;
  final List<Attachment> attachments;
  
  const Message({
    required this.id,
    required this.chatId,
    required this.role,
    required this.content,
    required this.contentType,
    required this.timestamp,
    required this.status,
    this.metadata,
    this.reactions = const [],
    this.threadId,
    this.replyTo,
    this.editedAt,
    this.deletedAt,
    this.attachments = const [],
  });
  
  factory Message.fromJson(Map<String, dynamic> json) {
    return Message(
      id: json['id'],
      chatId: json['chat_id'],
      role: json['role'],
      content: json['content'],
      contentType: json['content_type'] ?? 'text',
      timestamp: DateTime.parse(json['timestamp']),
      status: json['status'] ?? 'delivered',
      metadata: json['metadata'] != null 
          ? MessageMetadata.fromJson(json['metadata'])
          : null,
      reactions: (json['reactions'] as List? ?? [])
          .map((r) => Reaction.fromJson(r))
          .toList(),
      threadId: json['thread_id'],
      replyTo: json['reply_to'],
      editedAt: json['edited_at'] != null 
          ? DateTime.parse(json['edited_at'])
          : null,
      deletedAt: json['deleted_at'] != null 
          ? DateTime.parse(json['deleted_at'])
          : null,
      attachments: (json['attachments'] as List? ?? [])
          .map((a) => Attachment.fromJson(a))
          .toList(),
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'chat_id': chatId,
      'role': role,
      'content': content,
      'content_type': contentType,
      'timestamp': timestamp.toIso8601String(),
      'status': status,
      'metadata': metadata?.toJson(),
      'reactions': reactions.map((r) => r.toJson()).toList(),
      'thread_id': threadId,
      'reply_to': replyTo,
      'edited_at': editedAt?.toIso8601String(),
      'deleted_at': deletedAt?.toIso8601String(),
      'attachments': attachments.map((a) => a.toJson()).toList(),
    };
  }
  
  Message copyWith({
    String? id,
    String? chatId,
    String? role,
    String? content,
    String? contentType,
    DateTime? timestamp,
    String? status,
    MessageMetadata? metadata,
    List<Reaction>? reactions,
    String? threadId,
    String? replyTo,
    DateTime? editedAt,
    DateTime? deletedAt,
    List<Attachment>? attachments,
  }) {
    return Message(
      id: id ?? this.id,
      chatId: chatId ?? this.chatId,
      role: role ?? this.role,
      content: content ?? this.content,
      contentType: contentType ?? this.contentType,
      timestamp: timestamp ?? this.timestamp,
      status: status ?? this.status,
      metadata: metadata ?? this.metadata,
      reactions: reactions ?? this.reactions,
      threadId: threadId ?? this.threadId,
      replyTo: replyTo ?? this.replyTo,
      editedAt: editedAt ?? this.editedAt,
      deletedAt: deletedAt ?? this.deletedAt,
      attachments: attachments ?? this.attachments,
    );
  }
  
  bool get isUser => role == 'user';
  bool get isBot => role == 'bot';
  bool get isEdited => editedAt != null;
  bool get isDeleted => deletedAt != null;
  bool get isPending => status == 'sending' || status == 'pending';
  bool get isFailed => status == 'failed';
  bool get hasAttachments => attachments.isNotEmpty;
}

abstract class MessageEvent {
  MessageEventType get type;
}

class NewMessageEvent extends MessageEvent {
  final Message message;
  
  NewMessageEvent(this.message);
  
  @override
  MessageEventType get type => MessageEventType.newMessage;
}

class BotResponseChunkEvent extends MessageEvent {
  final String messageId;
  final String chunk;
  final int chunkIndex;
  final bool isComplete;
  
  BotResponseChunkEvent({
    required this.messageId,
    required this.chunk,
    required this.chunkIndex,
    required this.isComplete,
  });
  
  @override
  MessageEventType get type => MessageEventType.botResponseChunk;
}

enum MessageEventType {
  newMessage,
  messageStatus,
  typingIndicator,
  botResponseChunk,
  error,
}
```

---

## 🔄 Offline Message Queue

### **Offline Support Implementation**
```dart
class OfflineMessageQueue {
  static const String _queueKey = 'offline_message_queue';
  
  static Future<void> queueMessage(OfflineMessage message) async {
    final prefs = await SharedPreferences.getInstance();
    final queue = await getQueuedMessages();
    
    queue.add(message);
    
    final queueJson = queue.map((m) => m.toJson()).toList();
    await prefs.setString(_queueKey, jsonEncode(queueJson));
  }
  
  static Future<List<OfflineMessage>> getQueuedMessages() async {
    final prefs = await SharedPreferences.getInstance();
    final queueData = prefs.getString(_queueKey);
    
    if (queueData == null) return [];
    
    final queueJson = jsonDecode(queueData) as List;
    return queueJson.map((m) => OfflineMessage.fromJson(m)).toList();
  }
  
  static Future<void> processQueue() async {
    final queue = await getQueuedMessages();
    if (queue.isEmpty) return;
    
    final processedMessages = <OfflineMessage>[];
    
    for (final message in queue) {
      try {
        await MessageService.sendMessage(message.chatId, message.content);
        processedMessages.add(message);
      } catch (e) {
        // Increment retry count
        message.retryCount++;
        if (message.retryCount >= 3) {
          processedMessages.add(message); // Remove after 3 failed attempts
        }
      }
    }
    
    // Remove processed messages from queue
    final remainingQueue = queue.where((m) => !processedMessages.contains(m)).toList();
    await _saveQueue(remainingQueue);
  }
  
  static Future<void> _saveQueue(List<OfflineMessage> queue) async {
    final prefs = await SharedPreferences.getInstance();
    final queueJson = queue.map((m) => m.toJson()).toList();
    await prefs.setString(_queueKey, jsonEncode(queueJson));
  }
}

class OfflineMessage {
  final String id;
  final String chatId;
  final String content;
  final DateTime timestamp;
  int retryCount;
  
  OfflineMessage({
    required this.id,
    required this.chatId,
    required this.content,
    required this.timestamp,
    this.retryCount = 0,
  });
  
  factory OfflineMessage.fromJson(Map<String, dynamic> json) {
    return OfflineMessage(
      id: json['id'],
      chatId: json['chat_id'],
      content: json['content'],
      timestamp: DateTime.parse(json['timestamp']),
      retryCount: json['retry_count'] ?? 0,
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'chat_id': chatId,
      'content': content,
      'timestamp': timestamp.toIso8601String(),
      'retry_count': retryCount,
    };
  }
}
```

---

**This message handling system provides real-time, reliable messaging with rich features while maintaining offline capability and seamless integration with the existing Flutter app.**
