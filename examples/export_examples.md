# 📊 Data Export API Examples

**Document:** Data Export and Backup Examples  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 📱 Flutter Export Implementation Examples

### **Export Service Example**
```dart
// lib/services/export_service.dart
class ExportService {
  static const String EXPORT_DIRECTORY = '/storage/emulated/0/Documents/SpeechBot';
  
  static Future<ExportResult> exportChat({
    required ChatSession session,
    ExportFormat format = ExportFormat.markdown,
    bool includeMetadata = true,
    bool includeTimestamps = true,
  }) async {
    try {
      // Ensure directory exists
      await _ensureExportDirectory();
      
      String content;
      String fileExtension;
      
      switch (format) {
        case ExportFormat.markdown:
          content = _generateMarkdown(session, includeMetadata, includeTimestamps);
          fileExtension = 'md';
          break;
        case ExportFormat.json:
          content = _generateJson(session, includeMetadata);
          fileExtension = 'json';
          break;
        case ExportFormat.plainText:
          content = _generatePlainText(session, includeTimestamps);
          fileExtension = 'txt';
          break;
        case ExportFormat.csv:
          content = _generateCsv(session, includeMetadata);
          fileExtension = 'csv';
          break;
        case ExportFormat.pdf:
          return await _generatePdf(session, includeMetadata, includeTimestamps);
      }
      
      final fileName = _generateFileName(session, fileExtension);
      final filePath = path.join(EXPORT_DIRECTORY, fileName);
      
      final file = File(filePath);
      await file.writeAsString(content, encoding: utf8);
      
      // Upload to cloud if user has backup enabled
      String? cloudUrl;
      if (await _shouldUploadToCloud()) {
        cloudUrl = await _uploadToCloud(file, session.id);
      }
      
      return ExportResult(
        success: true,
        filePath: filePath,
        fileName: fileName,
        format: format,
        fileSize: await file.length(),
        cloudUrl: cloudUrl,
        exportedAt: DateTime.now(),
      );
      
    } catch (e) {
      return ExportResult(
        success: false,
        error: 'Export failed: ${e.toString()}',
        exportedAt: DateTime.now(),
      );
    }
  }

  static Future<ExportResult> exportAllChats({
    ExportFormat format = ExportFormat.json,
    bool includeMetadata = true,
    bool compressOutput = true,
  }) async {
    try {
      final chatProvider = Get.find<ChatProvider>();
      final allSessions = await ChatService.getAllChatSessions();
      
      await _ensureExportDirectory();
      
      if (compressOutput) {
        return await _exportAsArchive(allSessions, format, includeMetadata);
      } else {
        return await _exportAsCollection(allSessions, format, includeMetadata);
      }
      
    } catch (e) {
      return ExportResult(
        success: false,
        error: 'Bulk export failed: ${e.toString()}',
        exportedAt: DateTime.now(),
      );
    }
  }

  static Future<ExportResult> exportUserData({
    bool includeChats = true,
    bool includePreferences = true,
    bool includeProfile = true,
    bool compressOutput = true,
  }) async {
    try {
      await _ensureExportDirectory();
      
      final exportData = <String, dynamic>{};
      
      if (includeProfile) {
        final authProvider = Get.find<AuthProvider>();
        exportData['profile'] = _sanitizeUserProfile(authProvider.user!);
      }
      
      if (includePreferences) {
        final prefsProvider = Get.find<PreferencesProvider>();
        exportData['preferences'] = prefsProvider.preferences!.toJson();
      }
      
      if (includeChats) {
        final allSessions = await ChatService.getAllChatSessions();
        exportData['chats'] = allSessions.map((s) => s.toJson()).toList();
      }
      
      exportData['export_metadata'] = {
        'version': '1.0',
        'exported_at': DateTime.now().toIso8601String(),
        'app_version': await _getAppVersion(),
        'export_type': 'full_user_data',
      };
      
      final fileName = 'speechbot_user_data_${DateFormat('yyyyMMdd_HHmmss').format(DateTime.now())}';
      
      if (compressOutput) {
        return await _createCompressedExport(exportData, fileName);
      } else {
        final filePath = path.join(EXPORT_DIRECTORY, '$fileName.json');
        final file = File(filePath);
        await file.writeAsString(
          JsonEncoder.withIndent('  ').convert(exportData),
          encoding: utf8,
        );
        
        return ExportResult(
          success: true,
          filePath: filePath,
          fileName: '$fileName.json',
          format: ExportFormat.json,
          fileSize: await file.length(),
          exportedAt: DateTime.now(),
        );
      }
      
    } catch (e) {
      return ExportResult(
        success: false,
        error: 'User data export failed: ${e.toString()}',
        exportedAt: DateTime.now(),
      );
    }
  }

  static String _generateMarkdown(
    ChatSession session,
    bool includeMetadata,
    bool includeTimestamps,
  ) {
    final buffer = StringBuffer();
    
    // Header
    buffer.writeln('# ${session.title}');
    buffer.writeln();
    
    if (includeMetadata) {
      buffer.writeln('**Chat Details:**');
      buffer.writeln('- Created: ${DateFormat('yyyy-MM-dd HH:mm:ss').format(session.createdAt)}');
      buffer.writeln('- Last Updated: ${DateFormat('yyyy-MM-dd HH:mm:ss').format(session.updatedAt)}');
      buffer.writeln('- Messages: ${session.messages.length}');
      buffer.writeln();
      buffer.writeln('---');
      buffer.writeln();
    }
    
    // Messages
    for (final message in session.messages) {
      final senderIcon = message.sender == MessageSender.user ? '👤' : '🤖';
      final senderName = message.sender == MessageSender.user ? 'You' : 'SpeechBot';
      
      if (includeTimestamps) {
        buffer.writeln('## $senderIcon $senderName - ${DateFormat('HH:mm:ss').format(message.timestamp)}');
      } else {
        buffer.writeln('## $senderIcon $senderName');
      }
      
      buffer.writeln();
      buffer.writeln(message.content);
      buffer.writeln();
    }
    
    if (includeMetadata) {
      buffer.writeln('---');
      buffer.writeln();
      buffer.writeln('*Exported from SpeechBot on ${DateFormat('yyyy-MM-dd HH:mm:ss').format(DateTime.now())}*');
    }
    
    return buffer.toString();
  }

  static String _generateJson(ChatSession session, bool includeMetadata) {
    final data = session.toJson();
    
    if (includeMetadata) {
      data['export_metadata'] = {
        'exported_at': DateTime.now().toIso8601String(),
        'app_version': '1.0.0', // Get from package info
        'export_format': 'json',
      };
    }
    
    return JsonEncoder.withIndent('  ').convert(data);
  }

  static String _generatePlainText(ChatSession session, bool includeTimestamps) {
    final buffer = StringBuffer();
    
    buffer.writeln('${session.title}');
    buffer.writeln('=' * session.title.length);
    buffer.writeln();
    
    for (final message in session.messages) {
      final senderName = message.sender == MessageSender.user ? 'You' : 'SpeechBot';
      
      if (includeTimestamps) {
        buffer.writeln('[$senderName - ${DateFormat('HH:mm:ss').format(message.timestamp)}]');
      } else {
        buffer.writeln('[$senderName]');
      }
      
      buffer.writeln(message.content);
      buffer.writeln();
    }
    
    return buffer.toString();
  }

  static String _generateCsv(ChatSession session, bool includeMetadata) {
    final buffer = StringBuffer();
    
    // Headers
    buffer.writeln('Timestamp,Sender,Message,Type');
    
    // Messages
    for (final message in session.messages) {
      final csvRow = [
        message.timestamp.toIso8601String(),
        message.sender.toString().split('.').last,
        '"${message.content.replaceAll('"', '""')}"', // Escape quotes
        message.type.toString().split('.').last,
      ].join(',');
      
      buffer.writeln(csvRow);
    }
    
    if (includeMetadata) {
      buffer.writeln();
      buffer.writeln('# Metadata');
      buffer.writeln('Chat Title,"${session.title}"');
      buffer.writeln('Created,${session.createdAt.toIso8601String()}');
      buffer.writeln('Updated,${session.updatedAt.toIso8601String()}');
      buffer.writeln('Message Count,${session.messages.length}');
    }
    
    return buffer.toString();
  }

  static Future<ExportResult> _generatePdf(
    ChatSession session,
    bool includeMetadata,
    bool includeTimestamps,
  ) async {
    try {
      final pdf = pw.Document();
      
      pdf.addPage(
        pw.MultiPage(
          pageFormat: PdfPageFormat.a4,
          margin: pw.EdgeInsets.all(32),
          build: (pw.Context context) {
            final widgets = <pw.Widget>[];
            
            // Title
            widgets.add(
              pw.Header(
                level: 0,
                child: pw.Text(
                  session.title,
                  style: pw.TextStyle(fontSize: 24, fontWeight: pw.FontWeight.bold),
                ),
              ),
            );
            
            if (includeMetadata) {
              widgets.add(pw.SizedBox(height: 20));
              widgets.add(
                pw.Container(
                  padding: pw.EdgeInsets.all(10),
                  decoration: pw.BoxDecoration(
                    border: pw.Border.all(color: PdfColors.grey300),
                    borderRadius: pw.BorderRadius.circular(5),
                  ),
                  child: pw.Column(
                    crossAxisAlignment: pw.CrossAxisAlignment.start,
                    children: [
                      pw.Text('Chat Details', style: pw.TextStyle(fontWeight: pw.FontWeight.bold)),
                      pw.SizedBox(height: 5),
                      pw.Text('Created: ${DateFormat('yyyy-MM-dd HH:mm:ss').format(session.createdAt)}'),
                      pw.Text('Last Updated: ${DateFormat('yyyy-MM-dd HH:mm:ss').format(session.updatedAt)}'),
                      pw.Text('Messages: ${session.messages.length}'),
                    ],
                  ),
                ),
              );
            }
            
            widgets.add(pw.SizedBox(height: 30));
            
            // Messages
            for (final message in session.messages) {
              final isUser = message.sender == MessageSender.user;
              
              widgets.add(
                pw.Container(
                  margin: pw.EdgeInsets.only(bottom: 15),
                  padding: pw.EdgeInsets.all(10),
                  decoration: pw.BoxDecoration(
                    color: isUser ? PdfColors.blue50 : PdfColors.grey100,
                    borderRadius: pw.BorderRadius.circular(8),
                  ),
                  child: pw.Column(
                    crossAxisAlignment: pw.CrossAxisAlignment.start,
                    children: [
                      pw.Row(
                        mainAxisAlignment: pw.MainAxisAlignment.spaceBetween,
                        children: [
                          pw.Text(
                            isUser ? 'You' : 'SpeechBot',
                            style: pw.TextStyle(fontWeight: pw.FontWeight.bold),
                          ),
                          if (includeTimestamps)
                            pw.Text(
                              DateFormat('HH:mm:ss').format(message.timestamp),
                              style: pw.TextStyle(fontSize: 10, color: PdfColors.grey600),
                            ),
                        ],
                      ),
                      pw.SizedBox(height: 5),
                      pw.Text(message.content),
                    ],
                  ),
                ),
              );
            }
            
            return widgets;
          },
        ),
      );
      
      final fileName = _generateFileName(session, 'pdf');
      final filePath = path.join(EXPORT_DIRECTORY, fileName);
      
      final file = File(filePath);
      await file.writeAsBytes(await pdf.save());
      
      return ExportResult(
        success: true,
        filePath: filePath,
        fileName: fileName,
        format: ExportFormat.pdf,
        fileSize: await file.length(),
        exportedAt: DateTime.now(),
      );
      
    } catch (e) {
      return ExportResult(
        success: false,
        error: 'PDF generation failed: ${e.toString()}',
        exportedAt: DateTime.now(),
      );
    }
  }

  static Future<ExportResult> _exportAsArchive(
    List<ChatSession> sessions,
    ExportFormat format,
    bool includeMetadata,
  ) async {
    final tempDir = await getTemporaryDirectory();
    final archiveDir = Directory(path.join(tempDir.path, 'speechbot_export_${DateTime.now().millisecondsSinceEpoch}'));
    await archiveDir.create(recursive: true);
    
    try {
      // Export each chat to temp directory
      for (int i = 0; i < sessions.length; i++) {
        final session = sessions[i];
        String content;
        String extension;
        
        switch (format) {
          case ExportFormat.markdown:
            content = _generateMarkdown(session, includeMetadata, true);
            extension = 'md';
            break;
          case ExportFormat.json:
            content = _generateJson(session, includeMetadata);
            extension = 'json';
            break;
          default:
            content = _generatePlainText(session, true);
            extension = 'txt';
        }
        
        final fileName = '${i + 1}_${_sanitizeFileName(session.title)}.$extension';
        final file = File(path.join(archiveDir.path, fileName));
        await file.writeAsString(content, encoding: utf8);
      }
      
      // Create archive
      final archiveName = 'speechbot_all_chats_${DateFormat('yyyyMMdd_HHmmss').format(DateTime.now())}.zip';
      final archivePath = path.join(EXPORT_DIRECTORY, archiveName);
      
      final archive = Archive();
      
      await for (final file in archiveDir.list(recursive: true)) {
        if (file is File) {
          final relativePath = path.relative(file.path, from: archiveDir.path);
          final bytes = await file.readAsBytes();
          archive.addFile(ArchiveFile(relativePath, bytes.length, bytes));
        }
      }
      
      final zipEncoder = ZipEncoder();
      final zipData = zipEncoder.encode(archive);
      
      final zipFile = File(archivePath);
      await zipFile.writeAsBytes(zipData!);
      
      // Cleanup temp directory
      await archiveDir.delete(recursive: true);
      
      return ExportResult(
        success: true,
        filePath: archivePath,
        fileName: archiveName,
        format: ExportFormat.zip,
        fileSize: await zipFile.length(),
        exportedAt: DateTime.now(),
      );
      
    } catch (e) {
      // Cleanup on error
      if (await archiveDir.exists()) {
        await archiveDir.delete(recursive: true);
      }
      
      return ExportResult(
        success: false,
        error: 'Archive creation failed: ${e.toString()}',
        exportedAt: DateTime.now(),
      );
    }
  }

  static Future<void> _ensureExportDirectory() async {
    final directory = Directory(EXPORT_DIRECTORY);
    if (!await directory.exists()) {
      await directory.create(recursive: true);
    }
  }

  static String _generateFileName(ChatSession session, String extension) {
    final sanitizedTitle = _sanitizeFileName(session.title);
    final timestamp = DateFormat('yyyyMMdd_HHmmss').format(DateTime.now());
    return '${sanitizedTitle}_$timestamp.$extension';
  }

  static String _sanitizeFileName(String fileName) {
    return fileName
        .replaceAll(RegExp(r'[^\w\s-]'), '')
        .replaceAll(RegExp(r'\s+'), '_')
        .toLowerCase();
  }

  static Future<bool> _shouldUploadToCloud() async {
    final prefsProvider = Get.find<PreferencesProvider>();
    return prefsProvider.preferences?.autoExportEnabled ?? false;
  }

  static Future<String?> _uploadToCloud(File file, String sessionId) async {
    try {
      // Implementation for cloud upload (AWS S3, Google Drive, etc.)
      final cloudResult = await CloudStorageService.uploadFile(
        file: file,
        folder: 'exports',
        metadata: {
          'session_id': sessionId,
          'exported_at': DateTime.now().toIso8601String(),
        },
      );
      
      return cloudResult.url;
    } catch (e) {
      print('Cloud upload failed: $e');
      return null;
    }
  }
}
```

### **Export Screen Widget Example**
```dart
// lib/screens/export_screen.dart
class ExportScreen extends StatefulWidget {
  final ChatSession? specificSession;
  
  const ExportScreen({Key? key, this.specificSession}) : super(key: key);

  @override
  _ExportScreenState createState() => _ExportScreenState();
}

class _ExportScreenState extends State<ExportScreen> {
  ExportFormat _selectedFormat = ExportFormat.markdown;
  bool _includeMetadata = true;
  bool _includeTimestamps = true;
  bool _compressOutput = true;
  bool _uploadToCloud = false;
  bool _isExporting = false;
  String _exportType = 'single'; // single, all, user_data

  @override
  void initState() {
    super.initState();
    
    if (widget.specificSession != null) {
      _exportType = 'single';
    }
    
    _loadPreferences();
  }

  Future<void> _loadPreferences() async {
    final prefsProvider = Provider.of<PreferencesProvider>(context, listen: false);
    setState(() {
      _uploadToCloud = prefsProvider.preferences?.autoExportEnabled ?? false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Export Data'),
        actions: [
          IconButton(
            icon: Icon(Icons.help_outline),
            onPressed: () => _showHelpDialog(),
          ),
        ],
      ),
      
      body: ListView(
        padding: EdgeInsets.all(16),
        children: [
          if (widget.specificSession == null) ...[
            _buildExportTypeSection(),
            SizedBox(height: 24),
          ],
          
          _buildFormatSection(),
          SizedBox(height: 24),
          
          _buildOptionsSection(),
          SizedBox(height: 24),
          
          _buildCloudSection(),
          SizedBox(height: 32),
          
          _buildExportButton(),
          
          if (_isExporting) ...[
            SizedBox(height: 24),
            _buildProgressIndicator(),
          ],
        ],
      ),
    );
  }

  Widget _buildExportTypeSection() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Export Type',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            RadioListTile<String>(
              title: Text('Current Chat'),
              subtitle: Text('Export only the current conversation'),
              value: 'single',
              groupValue: _exportType,
              onChanged: (value) => setState(() => _exportType = value!),
            ),
            
            RadioListTile<String>(
              title: Text('All Chats'),
              subtitle: Text('Export all chat conversations'),
              value: 'all',
              groupValue: _exportType,
              onChanged: (value) => setState(() => _exportType = value!),
            ),
            
            RadioListTile<String>(
              title: Text('Complete User Data'),
              subtitle: Text('Export chats, preferences, and profile'),
              value: 'user_data',
              groupValue: _exportType,
              onChanged: (value) => setState(() => _exportType = value!),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildFormatSection() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Export Format',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            Wrap(
              spacing: 8,
              children: ExportFormat.values.map((format) {
                final isSelected = _selectedFormat == format;
                return FilterChip(
                  label: Text(_getFormatName(format)),
                  selected: isSelected,
                  onSelected: (selected) {
                    if (selected) {
                      setState(() => _selectedFormat = format);
                    }
                  },
                );
              }).toList(),
            ),
            
            SizedBox(height: 12),
            
            Text(
              _getFormatDescription(_selectedFormat),
              style: Theme.of(context).textTheme.bodyMedium?.copyWith(
                color: Theme.of(context).textTheme.bodySmall?.color,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildOptionsSection() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'Export Options',
              style: Theme.of(context).textTheme.titleLarge,
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Include Metadata'),
              subtitle: Text('Add chat creation dates, message counts, etc.'),
              value: _includeMetadata,
              onChanged: (value) => setState(() => _includeMetadata = value),
            ),
            
            SwitchListTile(
              title: Text('Include Timestamps'),
              subtitle: Text('Add message timestamps to export'),
              value: _includeTimestamps,
              onChanged: (value) => setState(() => _includeTimestamps = value),
            ),
            
            if (_exportType == 'all' || _exportType == 'user_data')
              SwitchListTile(
                title: Text('Compress Output'),
                subtitle: Text('Create a ZIP archive for multiple files'),
                value: _compressOutput,
                onChanged: (value) => setState(() => _compressOutput = value),
              ),
          ],
        ),
      ),
    );
  }

  Widget _buildCloudSection() {
    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Icon(Icons.cloud_upload, color: Theme.of(context).primaryColor),
                SizedBox(width: 8),
                Text(
                  'Cloud Backup',
                  style: Theme.of(context).textTheme.titleLarge,
                ),
              ],
            ),
            SizedBox(height: 16),
            
            SwitchListTile(
              title: Text('Upload to Cloud'),
              subtitle: Text('Automatically backup exported files'),
              value: _uploadToCloud,
              onChanged: (value) => setState(() => _uploadToCloud = value),
            ),
            
            if (_uploadToCloud) ...[
              Padding(
                padding: EdgeInsets.only(left: 16, top: 8),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Files will be securely stored in your cloud account.',
                      style: Theme.of(context).textTheme.bodySmall,
                    ),
                    SizedBox(height: 4),
                    Text(
                      'You can manage these files in the cloud storage section.',
                      style: Theme.of(context).textTheme.bodySmall,
                    ),
                  ],
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }

  Widget _buildExportButton() {
    return SizedBox(
      width: double.infinity,
      height: 50,
      child: ElevatedButton.icon(
        onPressed: _isExporting ? null : _startExport,
        icon: _isExporting 
            ? SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  color: Colors.white,
                ),
              )
            : Icon(Icons.file_download),
        label: Text(_isExporting ? 'Exporting...' : 'Start Export'),
        style: ElevatedButton.styleFrom(
          backgroundColor: Theme.of(context).primaryColor,
          foregroundColor: Colors.white,
        ),
      ),
    );
  }

  Widget _buildProgressIndicator() {
    return Column(
      children: [
        LinearProgressIndicator(),
        SizedBox(height: 8),
        Text(
          'Preparing your export...',
          style: Theme.of(context).textTheme.bodyMedium,
        ),
      ],
    );
  }

  Future<void> _startExport() async {
    setState(() => _isExporting = true);

    try {
      ExportResult result;

      switch (_exportType) {
        case 'single':
          final session = widget.specificSession ?? 
              Provider.of<ChatProvider>(context, listen: false).currentSession;
          
          if (session == null) {
            throw Exception('No chat session to export');
          }
          
          result = await ExportService.exportChat(
            session: session,
            format: _selectedFormat,
            includeMetadata: _includeMetadata,
            includeTimestamps: _includeTimestamps,
          );
          break;
          
        case 'all':
          result = await ExportService.exportAllChats(
            format: _selectedFormat,
            includeMetadata: _includeMetadata,
            compressOutput: _compressOutput,
          );
          break;
          
        case 'user_data':
          result = await ExportService.exportUserData(
            includeChats: true,
            includePreferences: true,
            includeProfile: true,
            compressOutput: _compressOutput,
          );
          break;
          
        default:
          throw Exception('Invalid export type');
      }

      if (result.success) {
        _showSuccessDialog(result);
      } else {
        _showErrorDialog(result.error ?? 'Export failed');
      }

    } catch (e) {
      _showErrorDialog(e.toString());
    } finally {
      setState(() => _isExporting = false);
    }
  }

  void _showSuccessDialog(ExportResult result) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Row(
          children: [
            Icon(Icons.check_circle, color: Colors.green),
            SizedBox(width: 8),
            Text('Export Successful'),
          ],
        ),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Your data has been exported successfully.'),
            SizedBox(height: 16),
            
            _buildResultDetail('File Name', result.fileName),
            _buildResultDetail('File Size', _formatFileSize(result.fileSize)),
            _buildResultDetail('Format', _getFormatName(result.format)),
            
            if (result.cloudUrl != null)
              _buildResultDetail('Cloud URL', 'Available in cloud storage'),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('Close'),
          ),
          
          ElevatedButton(
            onPressed: () {
              Navigator.of(context).pop();
              _shareFile(result.filePath!);
            },
            child: Text('Share'),
          ),
        ],
      ),
    );
  }

  Widget _buildResultDetail(String label, String value) {
    return Padding(
      padding: EdgeInsets.only(bottom: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 80,
            child: Text(
              '$label:',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(child: Text(value)),
        ],
      ),
    );
  }

  void _showErrorDialog(String error) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Row(
          children: [
            Icon(Icons.error, color: Colors.red),
            SizedBox(width: 8),
            Text('Export Failed'),
          ],
        ),
        content: Text(error),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('Close'),
          ),
          
          ElevatedButton(
            onPressed: () {
              Navigator.of(context).pop();
              _startExport(); // Retry
            },
            child: Text('Retry'),
          ),
        ],
      ),
    );
  }

  String _getFormatName(ExportFormat format) {
    switch (format) {
      case ExportFormat.markdown:
        return 'Markdown';
      case ExportFormat.json:
        return 'JSON';
      case ExportFormat.plainText:
        return 'Plain Text';
      case ExportFormat.csv:
        return 'CSV';
      case ExportFormat.pdf:
        return 'PDF';
      case ExportFormat.zip:
        return 'ZIP Archive';
      default:
        return 'Unknown';
    }
  }

  String _getFormatDescription(ExportFormat format) {
    switch (format) {
      case ExportFormat.markdown:
        return 'Formatted text with headers and styling. Great for documentation.';
      case ExportFormat.json:
        return 'Structured data format. Perfect for importing into other apps.';
      case ExportFormat.plainText:
        return 'Simple text format. Compatible with all text editors.';
      case ExportFormat.csv:
        return 'Spreadsheet format. Easy to analyze in Excel or Google Sheets.';
      case ExportFormat.pdf:
        return 'Portable document format. Professional appearance for sharing.';
      default:
        return 'Select a format to see its description.';
    }
  }

  String _formatFileSize(int? bytes) {
    if (bytes == null) return 'Unknown';
    
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    if (bytes < 1024 * 1024 * 1024) return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
    return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(1)} GB';
  }

  Future<void> _shareFile(String filePath) async {
    try {
      await Share.shareXFiles([XFile(filePath)]);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Failed to share file: $e')),
      );
    }
  }
}
```

---

## 🌐 HTTP API Examples

### **Request Export**
```bash
curl -X POST https://api.speechbot.com/v1/exports \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "export_type": "chat_session",
    "target_id": "chat_550e8400-e29b-41d4-a716-446655440000",
    "format": "json",
    "options": {
      "include_metadata": true,
      "include_timestamps": true,
      "compression": false
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Export request created successfully",
  "data": {
    "export_id": "export_550e8400-e29b-41d4-a716-446655440000",
    "status": "processing",
    "estimated_completion": "2025-07-15T10:35:00Z",
    "download_url": null,
    "created_at": "2025-07-15T10:30:00Z"
  }
}
```

### **Check Export Status**
```bash
curl -X GET https://api.speechbot.com/v1/exports/export_550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "export_id": "export_550e8400-e29b-41d4-a716-446655440000",
    "status": "completed",
    "download_url": "https://api.speechbot.com/v1/exports/export_550e8400-e29b-41d4-a716-446655440000/download",
    "file_size": 15768,
    "format": "json",
    "expires_at": "2025-07-22T10:30:00Z",
    "created_at": "2025-07-15T10:30:00Z",
    "completed_at": "2025-07-15T10:32:15Z"
  }
}
```

### **Download Export**
```bash
curl -X GET https://api.speechbot.com/v1/exports/export_550e8400-e29b-41d4-a716-446655440000/download \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  --output chat_export.json
```

### **List User Exports**
```bash
curl -X GET "https://api.speechbot.com/v1/exports?page=1&limit=20&status=completed" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "success": true,
  "data": {
    "exports": [
      {
        "export_id": "export_550e8400-e29b-41d4-a716-446655440000",
        "export_type": "chat_session",
        "target_id": "chat_550e8400-e29b-41d4-a716-446655440000",
        "status": "completed",
        "format": "json",
        "file_size": 15768,
        "download_url": "https://api.speechbot.com/v1/exports/export_550e8400-e29b-41d4-a716-446655440000/download",
        "expires_at": "2025-07-22T10:30:00Z",
        "created_at": "2025-07-15T10:30:00Z",
        "completed_at": "2025-07-15T10:32:15Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 1,
      "total_count": 5,
      "has_next": false,
      "has_previous": false
    }
  }
}
```

---

**These examples provide comprehensive data export functionality with multiple formats, cloud backup, and robust error handling for the SpeechBot application.**
