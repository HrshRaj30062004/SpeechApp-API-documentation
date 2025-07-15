# 💬 Chat & Messages API Examples

**Document:** Chat and Message Management Examples  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Flutter Chat Implementation Examples

### **Chat Session Management Example**
```dart
// lib/providers/chat_provider.dart
class ChatProvider extends ChangeNotifier {
  List<ChatSession> _chatSessions = [];
  ChatSession? _currentSession;
  bool _isLoading = false;
  String? _error;
  
  List<ChatSession> get chatSessions => List.unmodifiable(_chatSessions);
  ChatSession? get currentSession => _currentSession;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadChatSessions() async {
    _isLoading = true;
    notifyListeners();

    try {
      final sessions = await ChatService.getChatSessions();
      _chatSessions = sessions;
      _error = null;
    } catch (e) {
      _error = 'Failed to load chat sessions';
      print('Error loading chat sessions: $e');
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  Future<ChatSession> createNewChat({String? title, String? initialMessage}) async {
    try {
      final session = await ChatService.createChatSession(
        title: title ?? 'New Chat',
        initialMessage: initialMessage,
      );
      
      _chatSessions.insert(0, session);
      _currentSession = session;
      notifyListeners();
      
      return session;
    } catch (e) {
      _error = 'Failed to create chat session';
      print('Error creating chat session: $e');
      rethrow;
    }
  }

  Future<void> selectChat(String sessionId) async {
    try {
      final session = _chatSessions.firstWhere((s) => s.id == sessionId);
      
      // Load full conversation if not already loaded
      if (session.messages.isEmpty) {
        final fullSession = await ChatService.getChatSession(sessionId);
        final index = _chatSessions.indexWhere((s) => s.id == sessionId);
        _chatSessions[index] = fullSession;
        _currentSession = fullSession;
      } else {
        _currentSession = session;
      }
      
      notifyListeners();
    } catch (e) {
      _error = 'Failed to load chat session';
      print('Error selecting chat: $e');
    }
  }

  Future<void> sendMessage(String content, {MessageType type = MessageType.text}) async {
    if (_currentSession == null) {
      throw Exception('No active chat session');
    }

    try {
      // Add user message immediately
      final userMessage = Message(
        id: DateTime.now().millisecondsSinceEpoch.toString(),
        content: content,
        type: type,
        sender: MessageSender.user,
        timestamp: DateTime.now(),
        status: MessageStatus.sending,
      );
      
      _currentSession!.messages.add(userMessage);
      notifyListeners();

      // Send to server
      final response = await ChatService.sendMessage(
        sessionId: _currentSession!.id,
        content: content,
        type: type,
      );

      // Update user message status
      userMessage.status = MessageStatus.sent;
      userMessage.id = response.userMessage.id;

      // Add bot response
      _currentSession!.messages.add(response.botMessage);
      
      // Update session metadata
      _currentSession!.lastMessageAt = response.botMessage.timestamp;
      _currentSession!.lastMessage = response.botMessage.content;
      
      notifyListeners();
      
    } catch (e) {
      // Mark user message as failed
      if (_currentSession!.messages.isNotEmpty) {
        _currentSession!.messages.last.status = MessageStatus.failed;
      }
      
      _error = 'Failed to send message';
      notifyListeners();
      print('Error sending message: $e');
      rethrow;
    }
  }

  Future<void> retrySendMessage(Message message) async {
    if (_currentSession == null) return;

    try {
      message.status = MessageStatus.sending;
      notifyListeners();

      final response = await ChatService.sendMessage(
        sessionId: _currentSession!.id,
        content: message.content,
        type: message.type,
      );

      message.status = MessageStatus.sent;
      message.id = response.userMessage.id;
      
      _currentSession!.messages.add(response.botMessage);
      notifyListeners();
      
    } catch (e) {
      message.status = MessageStatus.failed;
      notifyListeners();
      rethrow;
    }
  }

  Future<void> deleteChat(String sessionId) async {
    try {
      await ChatService.deleteChatSession(sessionId);
      
      _chatSessions.removeWhere((s) => s.id == sessionId);
      
      if (_currentSession?.id == sessionId) {
        _currentSession = null;
      }
      
      notifyListeners();
    } catch (e) {
      _error = 'Failed to delete chat';
      print('Error deleting chat: $e');
      rethrow;
    }
  }

  Future<void> renameChat(String sessionId, String newTitle) async {
    try {
      await ChatService.updateChatSession(sessionId, title: newTitle);
      
      final index = _chatSessions.indexWhere((s) => s.id == sessionId);
      if (index != -1) {
        _chatSessions[index].title = newTitle;
        
        if (_currentSession?.id == sessionId) {
          _currentSession!.title = newTitle;
        }
        
        notifyListeners();
      }
    } catch (e) {
      _error = 'Failed to rename chat';
      print('Error renaming chat: $e');
      rethrow;
    }
  }

  void clearError() {
    _error = null;
    notifyListeners();
  }

  void clear() {
    _chatSessions.clear();
    _currentSession = null;
    _error = null;
    notifyListeners();
  }
}
```

### **Chat Screen Widget Example**
```dart
// lib/screens/chat_screen.dart (Enhanced version)
class ChatScreen extends StatefulWidget {
  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final TextEditingController _messageController = TextEditingController();
  final ScrollController _scrollController = ScrollController();
  bool _isComposing = false;

  @override
  void initState() {
    super.initState();
    _initializeChat();
  }

  Future<void> _initializeChat() async {
    final chatProvider = Provider.of<ChatProvider>(context, listen: false);
    
    if (chatProvider.chatSessions.isEmpty) {
      await chatProvider.loadChatSessions();
    }
    
    // Create new chat if none exists
    if (chatProvider.currentSession == null && chatProvider.chatSessions.isNotEmpty) {
      await chatProvider.selectChat(chatProvider.chatSessions.first.id);
    } else if (chatProvider.chatSessions.isEmpty) {
      await chatProvider.createNewChat();
    }
  }

  Future<void> _sendMessage() async {
    final content = _messageController.text.trim();
    if (content.isEmpty) return;

    setState(() {
      _isComposing = false;
    });
    
    _messageController.clear();
    
    try {
      final chatProvider = Provider.of<ChatProvider>(context, listen: false);
      await chatProvider.sendMessage(content);
      
      // Scroll to bottom after sending
      WidgetsBinding.instance.addPostFrameCallback((_) {
        _scrollToBottom();
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Failed to send message: ${e.toString()}'),
          action: SnackBarAction(
            label: 'Retry',
            onPressed: () async {
              _messageController.text = content;
              await _sendMessage();
            },
          ),
        ),
      );
    }
  }

  void _scrollToBottom() {
    if (_scrollController.hasClients) {
      _scrollController.animateTo(
        _scrollController.position.maxScrollExtent,
        duration: Duration(milliseconds: 300),
        curve: Curves.easeOut,
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Consumer<ChatProvider>(
          builder: (context, chatProvider, child) {
            final session = chatProvider.currentSession;
            return Text(session?.title ?? 'SpeechBot');
          },
        ),
        actions: [
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: () async {
              final chatProvider = Provider.of<ChatProvider>(context, listen: false);
              await chatProvider.loadChatSessions();
            },
          ),
          PopupMenuButton<String>(
            onSelected: (value) async {
              final chatProvider = Provider.of<ChatProvider>(context, listen: false);
              switch (value) {
                case 'new_chat':
                  await chatProvider.createNewChat();
                  break;
                case 'export':
                  await _exportCurrentChat();
                  break;
                case 'clear_all':
                  await _showClearAllDialog();
                  break;
              }
            },
            itemBuilder: (context) => [
              PopupMenuItem(value: 'new_chat', child: Text('New Chat')),
              PopupMenuItem(value: 'export', child: Text('Export Chat')),
              PopupMenuItem(value: 'clear_all', child: Text('Clear All Chats')),
            ],
          ),
        ],
      ),
      
      drawer: ChatHistoryDrawer(),
      
      body: Consumer<ChatProvider>(
        builder: (context, chatProvider, child) {
          if (chatProvider.isLoading) {
            return Center(child: CircularProgressIndicator());
          }

          if (chatProvider.error != null) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error, size: 64, color: Colors.red),
                  SizedBox(height: 16),
                  Text(chatProvider.error!),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () async {
                      chatProvider.clearError();
                      await chatProvider.loadChatSessions();
                    },
                    child: Text('Retry'),
                  ),
                ],
              ),
            );
          }

          final session = chatProvider.currentSession;
          if (session == null) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.chat_bubble_outline, size: 64),
                  SizedBox(height: 16),
                  Text('No chat session selected'),
                  SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () async {
                      await chatProvider.createNewChat();
                    },
                    child: Text('Start New Chat'),
                  ),
                ],
              ),
            );
          }

          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  controller: _scrollController,
                  padding: EdgeInsets.all(16),
                  itemCount: session.messages.length,
                  itemBuilder: (context, index) {
                    final message = session.messages[index];
                    return MessageBubble(
                      message: message,
                      onRetry: message.status == MessageStatus.failed
                          ? () => chatProvider.retrySendMessage(message)
                          : null,
                    );
                  },
                ),
              ),
              
              MessageInputField(
                controller: _messageController,
                isComposing: _isComposing,
                onChanged: (text) {
                  setState(() {
                    _isComposing = text.trim().isNotEmpty;
                  });
                },
                onSend: _sendMessage,
              ),
            ],
          );
        },
      ),
    );
  }

  Future<void> _exportCurrentChat() async {
    final chatProvider = Provider.of<ChatProvider>(context, listen: false);
    final session = chatProvider.currentSession;
    
    if (session == null) return;

    try {
      await ExportService.exportChat(session);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Chat exported successfully')),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Failed to export chat')),
      );
    }
  }

  Future<void> _showClearAllDialog() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Clear All Chats'),
        content: Text('This will permanently delete all your chat history. This action cannot be undone.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.of(context).pop(true),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
            child: Text('Delete All'),
          ),
        ],
      ),
    );

    if (confirmed == true) {
      final chatProvider = Provider.of<ChatProvider>(context, listen: false);
      // Implementation for clearing all chats
      // This would call a batch delete API
    }
  }
}
```

### **Message Widget Example**
```dart
// lib/widgets/message_bubble.dart
class MessageBubble extends StatefulWidget {
  final Message message;
  final VoidCallback? onRetry;

  const MessageBubble({
    Key? key,
    required this.message,
    this.onRetry,
  }) : super(key: key);

  @override
  _MessageBubbleState createState() => _MessageBubbleState();
}

class _MessageBubbleState extends State<MessageBubble> with TickerProviderStateMixin {
  late AnimationController _fadeController;
  late AnimationController _typewriterController;
  late Animation<double> _fadeAnimation;
  
  String _displayedText = '';
  Timer? _typewriterTimer;

  @override
  void initState() {
    super.initState();
    
    _fadeController = AnimationController(
      duration: Duration(milliseconds: 300),
      vsync: this,
    );
    
    _typewriterController = AnimationController(
      duration: Duration(milliseconds: widget.message.content.length * 30),
      vsync: this,
    );
    
    _fadeAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _fadeController, curve: Curves.easeIn),
    );

    _fadeController.forward();
    
    // Start typewriter effect for bot messages
    if (widget.message.sender == MessageSender.bot) {
      _startTypewriterEffect();
    } else {
      _displayedText = widget.message.content;
    }
  }

  void _startTypewriterEffect() {
    const duration = Duration(milliseconds: 30);
    int currentIndex = 0;
    
    _typewriterTimer = Timer.periodic(duration, (timer) {
      if (currentIndex < widget.message.content.length) {
        setState(() {
          _displayedText = widget.message.content.substring(0, currentIndex + 1);
        });
        currentIndex++;
      } else {
        timer.cancel();
        _typewriterController.forward();
      }
    });
  }

  @override
  void dispose() {
    _fadeController.dispose();
    _typewriterController.dispose();
    _typewriterTimer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final isUser = widget.message.sender == MessageSender.user;
    
    return FadeTransition(
      opacity: _fadeAnimation,
      child: Container(
        margin: EdgeInsets.only(
          top: 8,
          bottom: 8,
          left: isUser ? 64 : 0,
          right: isUser ? 0 : 64,
        ),
        child: Row(
          mainAxisAlignment: isUser ? MainAxisAlignment.end : MainAxisAlignment.start,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (!isUser) ...[
              CircleAvatar(
                radius: 16,
                backgroundColor: theme.primaryColor,
                child: Icon(Icons.smart_toy, size: 18, color: Colors.white),
              ),
              SizedBox(width: 8),
            ],
            
            Flexible(
              child: Container(
                padding: EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                decoration: BoxDecoration(
                  color: isUser 
                      ? theme.primaryColor 
                      : theme.colorScheme.surface,
                  borderRadius: BorderRadius.circular(20).copyWith(
                    bottomLeft: isUser ? Radius.circular(20) : Radius.circular(4),
                    bottomRight: isUser ? Radius.circular(4) : Radius.circular(20),
                  ),
                  border: !isUser ? Border.all(
                    color: theme.dividerColor.withOpacity(0.3),
                    width: 1,
                  ) : null,
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    SelectableText(
                      widget.message.sender == MessageSender.bot 
                          ? _displayedText 
                          : widget.message.content,
                      style: TextStyle(
                        color: isUser ? Colors.white : theme.textTheme.bodyLarge?.color,
                        fontSize: 16,
                      ),
                    ),
                    
                    SizedBox(height: 8),
                    
                    Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(
                          _formatTime(widget.message.timestamp),
                          style: TextStyle(
                            color: isUser 
                                ? Colors.white.withOpacity(0.7)
                                : theme.textTheme.bodySmall?.color,
                            fontSize: 12,
                          ),
                        ),
                        
                        if (isUser) ...[
                          SizedBox(width: 4),
                          _buildStatusIcon(),
                        ],
                      ],
                    ),
                  ],
                ),
              ),
            ),
            
            if (isUser) ...[
              SizedBox(width: 8),
              CircleAvatar(
                radius: 16,
                backgroundColor: theme.colorScheme.primary,
                child: Icon(Icons.person, size: 18, color: Colors.white),
              ),
            ],
          ],
        ),
      ),
    );
  }

  Widget _buildStatusIcon() {
    switch (widget.message.status) {
      case MessageStatus.sending:
        return SizedBox(
          width: 12,
          height: 12,
          child: CircularProgressIndicator(
            strokeWidth: 2,
            valueColor: AlwaysStoppedAnimation<Color>(Colors.white.withOpacity(0.7)),
          ),
        );
      
      case MessageStatus.sent:
        return Icon(
          Icons.check,
          size: 14,
          color: Colors.white.withOpacity(0.7),
        );
      
      case MessageStatus.delivered:
        return Icon(
          Icons.done_all,
          size: 14,
          color: Colors.white.withOpacity(0.7),
        );
      
      case MessageStatus.failed:
        return GestureDetector(
          onTap: widget.onRetry,
          child: Icon(
            Icons.error_outline,
            size: 14,
            color: Colors.red.shade300,
          ),
        );
      
      default:
        return SizedBox.shrink();
    }
  }

  String _formatTime(DateTime timestamp) {
    final now = DateTime.now();
    final difference = now.difference(timestamp);

    if (difference.inDays > 0) {
      return DateFormat('MMM d, HH:mm').format(timestamp);
    } else if (difference.inHours > 0) {
      return DateFormat('HH:mm').format(timestamp);
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes}m ago';
    } else {
      return 'Just now';
    }
  }
}
```

---

## 🌐 HTTP API Examples

### **Create Chat Session**
```bash
curl -X POST https://api.speechbot.com/v1/chats \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New Conversation",
    "initial_message": "Hello! How can you help me today?"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Chat session created successfully",
  "data": {
    "session": {
      "id": "chat_550e8400-e29b-41d4-a716-446655440000",
      "title": "New Conversation",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T10:30:00Z",
      "last_message_at": null,
      "last_message": null,
      "message_count": 0,
      "messages": []
    }
  }
}
```

### **Send Message**
```bash
curl -X POST https://api.speechbot.com/v1/chats/chat_550e8400-e29b-41d4-a716-446655440000/messages \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "content": "What are the benefits of renewable energy?",
    "type": "text"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Message sent successfully",
  "data": {
    "user_message": {
      "id": "msg_550e8400-e29b-41d4-a716-446655440001",
      "content": "What are the benefits of renewable energy?",
      "type": "text",
      "sender": "user",
      "timestamp": "2025-07-15T10:30:15Z",
      "status": "sent"
    },
    "bot_message": {
      "id": "msg_550e8400-e29b-41d4-a716-446655440002",
      "content": "Renewable energy offers several key benefits:\n\n1. **Environmental Impact**: Significantly reduces greenhouse gas emissions and air pollution\n2. **Sustainability**: Inexhaustible energy sources like solar and wind\n3. **Economic Benefits**: Lower long-term costs and job creation\n4. **Energy Security**: Reduces dependence on fossil fuel imports\n5. **Health Benefits**: Cleaner air leads to better public health outcomes\n\nWould you like me to elaborate on any of these points?",
      "type": "text",
      "sender": "bot",
      "timestamp": "2025-07-15T10:30:18Z",
      "status": "delivered"
    }
  }
}
```

### **Get Chat Sessions**
```bash
curl -X GET "https://api.speechbot.com/v1/chats?page=1&limit=20&include_messages=false" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "sessions": [
      {
        "id": "chat_550e8400-e29b-41d4-a716-446655440000",
        "title": "Renewable Energy Discussion",
        "created_at": "2025-07-15T10:30:00Z",
        "updated_at": "2025-07-15T10:35:22Z",
        "last_message_at": "2025-07-15T10:35:22Z",
        "last_message": "That's a great question about solar panel efficiency...",
        "message_count": 8
      },
      {
        "id": "chat_550e8400-e29b-41d4-a716-446655440001",
        "title": "Programming Help",
        "created_at": "2025-07-14T15:20:00Z",
        "updated_at": "2025-07-14T16:45:30Z",
        "last_message_at": "2025-07-14T16:45:30Z",
        "last_message": "Here's the complete Flutter widget example...",
        "message_count": 12
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 3,
      "total_count": 45,
      "has_next": true,
      "has_previous": false
    }
  }
}
```

### **Get Chat Messages**
```bash
curl -X GET "https://api.speechbot.com/v1/chats/chat_550e8400-e29b-41d4-a716-446655440000/messages?page=1&limit=50" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "msg_550e8400-e29b-41d4-a716-446655440001",
        "content": "What are the benefits of renewable energy?",
        "type": "text",
        "sender": "user",
        "timestamp": "2025-07-15T10:30:15Z",
        "status": "sent"
      },
      {
        "id": "msg_550e8400-e29b-41d4-a716-446655440002",
        "content": "Renewable energy offers several key benefits...",
        "type": "text",
        "sender": "bot",
        "timestamp": "2025-07-15T10:30:18Z",
        "status": "delivered"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 1,
      "total_count": 8,
      "has_next": false,
      "has_previous": false
    }
  }
}
```

### **Update Chat Session**
```bash
curl -X PATCH https://api.speechbot.com/v1/chats/chat_550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Renewable Energy & Sustainability"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Chat session updated successfully",
  "data": {
    "session": {
      "id": "chat_550e8400-e29b-41d4-a716-446655440000",
      "title": "Renewable Energy & Sustainability",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T11:15:30Z",
      "last_message_at": "2025-07-15T10:35:22Z",
      "last_message": "That's a great question about solar panel efficiency...",
      "message_count": 8
    }
  }
}
```

### **Delete Chat Session**
```bash
curl -X DELETE https://api.speechbot.com/v1/chats/chat_550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "message": "Chat session deleted successfully",
  "data": {
    "deleted_session_id": "chat_550e8400-e29b-41d4-a716-446655440000",
    "deleted_at": "2025-07-15T11:20:00Z"
  }
}
```

---

## 🔌 WebSocket Integration Example

### **WebSocket Connection Setup**
```dart
// lib/services/websocket_service.dart
class WebSocketService {
  static const String WS_URL = 'wss://api.speechbot.com/v1/ws';
  
  IOWebSocketChannel? _channel;
  StreamSubscription? _subscription;
  Timer? _heartbeatTimer;
  
  final StreamController<WebSocketMessage> _messageController = StreamController.broadcast();
  Stream<WebSocketMessage> get messageStream => _messageController.stream;
  
  bool _isConnected = false;
  bool get isConnected => _isConnected;

  Future<void> connect() async {
    try {
      final token = await SecureStorageService.getAccessToken();
      if (token == null) throw Exception('No access token available');

      _channel = IOWebSocketChannel.connect(
        '$WS_URL?token=$token',
        protocols: ['speechbot-v1'],
      );

      _subscription = _channel!.stream.listen(
        _handleMessage,
        onError: _handleError,
        onDone: _handleDisconnection,
      );

      _isConnected = true;
      _startHeartbeat();
      
      print('WebSocket connected successfully');
    } catch (e) {
      print('WebSocket connection failed: $e');
      _isConnected = false;
      rethrow;
    }
  }

  void _handleMessage(dynamic data) {
    try {
      final Map<String, dynamic> json = jsonDecode(data);
      final message = WebSocketMessage.fromJson(json);
      
      switch (message.type) {
        case 'message_response':
          _handleMessageResponse(message);
          break;
        case 'typing_indicator':
          _handleTypingIndicator(message);
          break;
        case 'connection_ack':
          print('WebSocket connection acknowledged');
          break;
        case 'error':
          _handleServerError(message);
          break;
      }
      
      _messageController.add(message);
    } catch (e) {
      print('Error handling WebSocket message: $e');
    }
  }

  void _handleMessageResponse(WebSocketMessage message) {
    // Handle real-time message responses
    final chatProvider = Get.find<ChatProvider>();
    chatProvider.handleWebSocketMessage(message);
  }

  void _handleTypingIndicator(WebSocketMessage message) {
    // Handle typing indicators
    final data = message.data as Map<String, dynamic>;
    final sessionId = data['session_id'];
    final isTyping = data['is_typing'] as bool;
    
    // Update UI to show bot typing
    print('Bot typing in session $sessionId: $isTyping');
  }

  void sendMessage(String sessionId, String content, {MessageType type = MessageType.text}) {
    if (!_isConnected || _channel == null) {
      throw Exception('WebSocket not connected');
    }

    final message = {
      'type': 'send_message',
      'data': {
        'session_id': sessionId,
        'content': content,
        'message_type': type.toString().split('.').last,
        'timestamp': DateTime.now().toIso8601String(),
      }
    };

    _channel!.sink.add(jsonEncode(message));
  }

  void _startHeartbeat() {
    _heartbeatTimer = Timer.periodic(Duration(seconds: 30), (timer) {
      if (_isConnected && _channel != null) {
        _channel!.sink.add(jsonEncode({
          'type': 'ping',
          'timestamp': DateTime.now().toIso8601String(),
        }));
      }
    });
  }

  void _handleError(error) {
    print('WebSocket error: $error');
    _isConnected = false;
    
    // Attempt reconnection
    Future.delayed(Duration(seconds: 5), () {
      if (!_isConnected) {
        connect();
      }
    });
  }

  void _handleDisconnection() {
    print('WebSocket disconnected');
    _isConnected = false;
    _heartbeatTimer?.cancel();
    
    // Attempt reconnection
    Future.delayed(Duration(seconds: 3), () {
      if (!_isConnected) {
        connect();
      }
    });
  }

  void disconnect() {
    _isConnected = false;
    _heartbeatTimer?.cancel();
    _subscription?.cancel();
    _channel?.sink.close();
    _messageController.close();
  }
}
```

---

**These examples provide comprehensive, production-ready chat functionality that integrates seamlessly with the SpeechBot API architecture.**
