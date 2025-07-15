---
layout: default
title: Data Export
nav_order: 6
---

# 📤 Data Export System

**Document:** Data Export API Specification  
**Version:** 1.0.0  
**Last Updated:** July 15, 2025  

---

## 🎯 Export System Overview

### **Current Flutter App Implementation**
The app currently supports basic chat history export with this functionality:

```dart
// Current export implementation in chat_screen.dart
Future<void> _exportChatHistory() async {
  String exportData = '';
  
  for (var session in chatSessions) {
    exportData += 'Chat: ${session['title']}\n';
    exportData += 'Created: ${session['id']}\n\n';
    
    for (var message in session['messages']) {
      String role = message['role'] == 'user' ? 'You' : 'SpeechBot';
      exportData += '$role: ${message['content']}\n\n';
    }
    exportData += '${'-' * 50}\n\n';
  }
  
  // Save as text file
  await _saveExportedData(exportData, 'chat_history.txt');
}
```

### **Target API Implementation**
- **Multiple Formats**: JSON, Markdown, PDF, HTML, CSV, XML
- **Selective Export**: Choose specific chats, date ranges, or message types
- **Batch Processing**: Handle large datasets with background processing
- **Secure Downloads**: Temporary, authenticated download URLs
- **Export Templates**: Customizable output formats and layouts
- **Scheduled Exports**: Automated backup functionality

---

## 📤 Export Endpoints

### **POST /export/create**
Create a new export job for user data.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "export_type": "chat_history", // Options: chat_history, user_data, preferences, full_backup
  "format": "markdown", // Options: json, markdown, pdf, html, csv, xml, txt
  "filters": {
    "chat_ids": ["chat_1684761234567", "chat_1684675834567"], // Optional: specific chats
    "folder_ids": ["folder_work"], // Optional: specific folders
    "date_range": {
      "start": "2025-07-01T00:00:00Z",
      "end": "2025-07-15T23:59:59Z"
    },
    "include_metadata": true,
    "include_attachments": false,
    "message_types": ["user", "bot"], // Optional: filter by role
    "tags": ["important", "work"] // Optional: filter by tags
  },
  "options": {
    "include_timestamps": true,
    "include_message_ids": false,
    "group_by_chat": true,
    "sort_order": "chronological", // Options: chronological, reverse_chronological, chat_title
    "compression": "zip", // Options: none, zip, gzip
    "password_protect": false,
    "custom_template": null // Optional: custom template ID
  },
  "delivery": {
    "method": "download", // Options: download, email, webhook
    "email": "user@example.com", // Required if method is email
    "webhook_url": "https://example.com/webhook", // Required if method is webhook
    "notification_preferences": {
      "email_on_completion": true,
      "push_notification": true
    }
  }
}
```

**Success Response (202 Accepted):**
```json
{
  "success": true,
  "message": "Export job created successfully",
  "data": {
    "export_job": {
      "id": "export_1684761234567",
      "export_type": "chat_history",
      "format": "markdown",
      "status": "queued",
      "created_at": "2025-07-15T16:00:00Z",
      "estimated_completion": "2025-07-15T16:05:00Z",
      "progress": {
        "percentage": 0,
        "current_step": "Initializing export",
        "total_steps": 5
      },
      "filters": {
        "chat_count": 15,
        "estimated_messages": 347,
        "date_range": {
          "start": "2025-07-01T00:00:00Z",
          "end": "2025-07-15T23:59:59Z"
        }
      },
      "options": {
        "include_timestamps": true,
        "include_message_ids": false,
        "group_by_chat": true,
        "sort_order": "chronological",
        "compression": "zip"
      }
    }
  }
}
```

**Error Responses:**
```json
// 400 Bad Request - Invalid export parameters
{
  "success": false,
  "error": "INVALID_EXPORT_PARAMETERS",
  "message": "Invalid export parameters provided",
  "details": {
    "format": ["Unsupported format 'docx'"],
    "date_range.start": ["Start date cannot be in the future"]
  }
}

// 429 Too Many Requests - Export rate limit
{
  "success": false,
  "error": "EXPORT_RATE_LIMIT",
  "message": "Too many export requests. Please try again later.",
  "details": {
    "retry_after": 300,
    "current_exports": 3,
    "max_concurrent_exports": 2
  }
}
```

---

### **GET /export/jobs**
Retrieve user's export job history.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `status`: Filter by status (queued, processing, completed, failed, expired)
- `export_type`: Filter by export type
- `format`: Filter by format

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "export_jobs": [
      {
        "id": "export_1684761234567",
        "export_type": "chat_history",
        "format": "markdown",
        "status": "completed",
        "created_at": "2025-07-15T16:00:00Z",
        "completed_at": "2025-07-15T16:03:42Z",
        "expires_at": "2025-07-22T16:03:42Z",
        "file_info": {
          "filename": "chat_history_2025-07-15.zip",
          "size": 2847520,
          "download_count": 1,
          "checksum": "sha256:a1b2c3d4e5f6..."
        },
        "statistics": {
          "total_chats": 15,
          "total_messages": 347,
          "processing_time_seconds": 222
        }
      },
      {
        "id": "export_1684675834567",
        "export_type": "full_backup",
        "format": "json",
        "status": "processing",
        "created_at": "2025-07-15T15:45:00Z",
        "progress": {
          "percentage": 65,
          "current_step": "Generating export file",
          "total_steps": 5,
          "estimated_completion": "2025-07-15T15:52:00Z"
        }
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 2,
      "total_jobs": 23,
      "has_next": true,
      "has_previous": false
    }
  }
}
```

---

### **GET /export/jobs/{job_id}**
Get details of a specific export job.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "export_job": {
      "id": "export_1684761234567",
      "export_type": "chat_history",
      "format": "markdown",
      "status": "completed",
      "created_at": "2025-07-15T16:00:00Z",
      "completed_at": "2025-07-15T16:03:42Z",
      "expires_at": "2025-07-22T16:03:42Z",
      "download_url": "https://api.speechbot.com/v1/export/download/export_1684761234567?token=temp_download_token",
      "file_info": {
        "filename": "chat_history_2025-07-15.zip",
        "size": 2847520,
        "download_count": 1,
        "checksum": "sha256:a1b2c3d4e5f6..."
      },
      "filters": {
        "chat_ids": ["chat_1684761234567", "chat_1684675834567"],
        "date_range": {
          "start": "2025-07-01T00:00:00Z",
          "end": "2025-07-15T23:59:59Z"
        },
        "include_metadata": true
      },
      "options": {
        "include_timestamps": true,
        "group_by_chat": true,
        "sort_order": "chronological",
        "compression": "zip"
      },
      "statistics": {
        "total_chats": 15,
        "total_messages": 347,
        "total_characters": 89420,
        "estimated_tokens": 22355,
        "processing_time_seconds": 222,
        "file_structure": {
          "folders": 3,
          "files": 16,
          "attachments": 0
        }
      }
    }
  }
}
```

**Error Responses:**
```json
// 404 Not Found - Export job not found
{
  "success": false,
  "error": "EXPORT_JOB_NOT_FOUND",
  "message": "Export job not found"
}

// 410 Gone - Export file expired
{
  "success": false,
  "error": "EXPORT_FILE_EXPIRED",
  "message": "Export file has expired and is no longer available",
  "details": {
    "expired_at": "2025-07-22T16:03:42Z",
    "can_recreate": true
  }
}
```

---

### **GET /export/download/{job_id}**
Download the exported data file.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `token`: Temporary download token (included in download_url)

**Success Response (200 OK):**
```http
Content-Type: application/zip
Content-Disposition: attachment; filename="chat_history_2025-07-15.zip"
Content-Length: 2847520
X-Checksum: sha256:a1b2c3d4e5f6...

[Binary file content]
```

**Error Responses:**
```json
// 401 Unauthorized - Invalid download token
{
  "success": false,
  "error": "INVALID_DOWNLOAD_TOKEN",
  "message": "Download token is invalid or expired"
}

// 429 Too Many Requests - Download rate limit
{
  "success": false,
  "error": "DOWNLOAD_RATE_LIMIT",
  "message": "Too many download attempts. Please try again later.",
  "details": {
    "retry_after": 60
  }
}
```

---

### **DELETE /export/jobs/{job_id}**
Cancel or delete an export job.

**Headers:**
```http
Authorization: Bearer <access_token>
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Export job deleted successfully",
  "data": {
    "job_id": "export_1684761234567",
    "status": "cancelled",
    "deleted_at": "2025-07-15T16:30:00Z"
  }
}
```

---

### **POST /export/jobs/{job_id}/recreate**
Recreate an expired or failed export job.

**Headers:**
```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "use_original_filters": true,
  "format_override": "pdf", // Optional: change format
  "options_override": { // Optional: update options
    "include_timestamps": false,
    "compression": "none"
  }
}
```

**Success Response (202 Accepted):**
```json
{
  "success": true,
  "message": "Export job recreated successfully",
  "data": {
    "original_job_id": "export_1684761234567",
    "new_job_id": "export_1684761334567",
    "status": "queued",
    "created_at": "2025-07-15T16:35:00Z"
  }
}
```

---

## 📋 Export Formats & Templates

### **Markdown Format Example**
```markdown
# Chat History Export
**Generated:** July 15, 2025 at 4:03 PM  
**Total Chats:** 15  
**Total Messages:** 347  
**Date Range:** July 1, 2025 - July 15, 2025  

---

## Chat: Project Planning Discussion
**Created:** July 15, 2025 at 10:30 AM  
**Messages:** 24  
**Folder:** Work  
**Tags:** planning, productivity  

### Messages

**You** *(July 15, 2025 at 10:30 AM)*  
Hello! I need help planning a project.

**SpeechBot** *(July 15, 2025 at 10:30 AM)*  
I'd be happy to help you with project planning! What type of project are you working on?

**You** *(July 15, 2025 at 10:32 AM)*  
It's a software development project for a mobile app.

---

## Chat: Creative Writing Ideas
**Created:** July 14, 2025 at 8:15 AM  
**Messages:** 12  
**Folder:** Personal  
**Tags:** creative, writing  

### Messages

**You** *(July 14, 2025 at 8:15 AM)*  
I need some inspiration for a short story.

**SpeechBot** *(July 14, 2025 at 8:15 AM)*  
I'd love to help spark your creativity! What genre or theme interests you most?

---

*Export completed on July 15, 2025 at 4:03 PM*  
*Total processing time: 3 minutes 42 seconds*
```

### **JSON Format Structure**
```json
{
  "export_metadata": {
    "export_id": "export_1684761234567",
    "export_type": "chat_history",
    "generated_at": "2025-07-15T16:03:42Z",
    "generated_by": "SpeechBot Export Service v1.0.0",
    "user_id": "user_550e8400",
    "filters_applied": {
      "date_range": {
        "start": "2025-07-01T00:00:00Z",
        "end": "2025-07-15T23:59:59Z"
      },
      "include_metadata": true
    },
    "statistics": {
      "total_chats": 15,
      "total_messages": 347,
      "total_characters": 89420,
      "processing_time_seconds": 222
    }
  },
  "chats": [
    {
      "id": "chat_1684761234567",
      "title": "Project Planning Discussion",
      "created_at": "2025-07-15T10:30:00Z",
      "updated_at": "2025-07-15T14:45:00Z",
      "message_count": 24,
      "folder": {
        "id": "folder_work",
        "name": "Work"
      },
      "tags": ["planning", "productivity"],
      "metadata": {
        "total_characters": 15420,
        "estimated_tokens": 3855
      },
      "messages": [
        {
          "id": "msg_1684761234567_0",
          "role": "user",
          "content": "Hello! I need help planning a project.",
          "timestamp": "2025-07-15T10:30:00Z",
          "metadata": {
            "client_timestamp": "2025-07-15T10:30:00Z"
          }
        },
        {
          "id": "msg_1684761234567_1",
          "role": "bot",
          "content": "I'd be happy to help you with project planning! What type of project are you working on?",
          "timestamp": "2025-07-15T10:30:15Z",
          "metadata": {
            "generation_time_ms": 1247,
            "model_used": "gpt-4-turbo",
            "tokens_used": 23
          }
        }
      ]
    }
  ]
}
```

### **CSV Format Structure**
```csv
chat_id,chat_title,chat_created_at,message_id,message_role,message_content,message_timestamp,folder_name,tags
chat_1684761234567,"Project Planning Discussion","2025-07-15T10:30:00Z",msg_1684761234567_0,user,"Hello! I need help planning a project.","2025-07-15T10:30:00Z",Work,"planning,productivity"
chat_1684761234567,"Project Planning Discussion","2025-07-15T10:30:00Z",msg_1684761234567_1,bot,"I'd be happy to help you with project planning! What type of project are you working on?","2025-07-15T10:30:15Z",Work,"planning,productivity"
```

---

## 📱 Flutter Integration

### **Export Service Implementation**
```dart
class ExportService {
  static Future<ExportJob> createExport({
    required ExportType exportType,
    required ExportFormat format,
    ExportFilters? filters,
    ExportOptions? options,
    ExportDelivery? delivery,
  }) async {
    final data = {
      'export_type': exportType.toString().split('.').last,
      'format': format.toString().split('.').last,
    };
    
    if (filters != null) data['filters'] = filters.toJson();
    if (options != null) data['options'] = options.toJson();
    if (delivery != null) data['delivery'] = delivery.toJson();
    
    try {
      final response = await ApiService.post('/export/create', data: data);
      return ExportJob.fromJson(response.data['data']['export_job']);
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
  
  static Future<List<ExportJob>> getExportJobs({
    int page = 1,
    int limit = 20,
    ExportStatus? status,
    ExportType? exportType,
    ExportFormat? format,
  }) async {
    final params = <String, dynamic>{
      'page': page,
      'limit': limit,
    };
    
    if (status != null) params['status'] = status.toString().split('.').last;
    if (exportType != null) params['export_type'] = exportType.toString().split('.').last;
    if (format != null) params['format'] = format.toString().split('.').last;
    
    try {
      final response = await ApiService.get('/export/jobs', queryParameters: params);
      final jobs = response.data['data']['export_jobs'] as List;
      return jobs.map((job) => ExportJob.fromJson(job)).toList();
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
  
  static Future<ExportJob> getExportJob(String jobId) async {
    try {
      final response = await ApiService.get('/export/jobs/$jobId');
      return ExportJob.fromJson(response.data['data']['export_job']);
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
  
  static Future<void> downloadExport(String jobId, String filename) async {
    try {
      final response = await ApiService.download('/export/download/$jobId');
      await _saveDownloadedFile(response.data, filename);
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
  
  static Future<void> cancelExport(String jobId) async {
    try {
      await ApiService.delete('/export/jobs/$jobId');
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
  
  static Future<ExportJob> recreateExport(
    String jobId, {
    bool useOriginalFilters = true,
    ExportFormat? formatOverride,
    ExportOptions? optionsOverride,
  }) async {
    final data = {
      'use_original_filters': useOriginalFilters,
    };
    
    if (formatOverride != null) {
      data['format_override'] = formatOverride.toString().split('.').last;
    }
    if (optionsOverride != null) {
      data['options_override'] = optionsOverride.toJson();
    }
    
    try {
      final response = await ApiService.post('/export/jobs/$jobId/recreate', data: data);
      return ExportJob.fromJson(response.data['data']);
    } catch (e) {
      throw ExportException.fromError(e);
    }
  }
}
```

### **Export Provider for State Management**
```dart
class ExportProvider extends ChangeNotifier {
  List<ExportJob> _exportJobs = [];
  bool _isLoading = false;
  String? _error;
  
  List<ExportJob> get exportJobs => _exportJobs;
  bool get isLoading => _isLoading;
  String? get error => _error;
  
  List<ExportJob> get activeExports => _exportJobs
      .where((job) => job.status == ExportStatus.queued || job.status == ExportStatus.processing)
      .toList();
  
  List<ExportJob> get completedExports => _exportJobs
      .where((job) => job.status == ExportStatus.completed)
      .toList();
  
  Future<void> loadExportJobs() async {
    try {
      _isLoading = true;
      _error = null;
      notifyListeners();
      
      _exportJobs = await ExportService.getExportJobs();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _isLoading = false;
      notifyListeners();
    }
  }
  
  Future<ExportJob> createExport({
    required ExportType exportType,
    required ExportFormat format,
    ExportFilters? filters,
    ExportOptions? options,
  }) async {
    try {
      _error = null;
      
      final exportJob = await ExportService.createExport(
        exportType: exportType,
        format: format,
        filters: filters,
        options: options,
      );
      
      _exportJobs.insert(0, exportJob);
      notifyListeners();
      
      // Start polling for updates
      _pollExportProgress(exportJob.id);
      
      return exportJob;
    } catch (e) {
      _error = e.toString();
      notifyListeners();
      rethrow;
    }
  }
  
  Future<void> downloadExport(ExportJob job) async {
    try {
      _error = null;
      
      final filename = job.fileInfo?.filename ?? 'export_${job.id}.${job.format.extension}';
      await ExportService.downloadExport(job.id, filename);
      
      // Update download count
      final updatedJob = await ExportService.getExportJob(job.id);
      final index = _exportJobs.indexWhere((j) => j.id == job.id);
      if (index != -1) {
        _exportJobs[index] = updatedJob;
        notifyListeners();
      }
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  Future<void> cancelExport(String jobId) async {
    try {
      _error = null;
      
      await ExportService.cancelExport(jobId);
      
      final index = _exportJobs.indexWhere((job) => job.id == jobId);
      if (index != -1) {
        _exportJobs[index] = _exportJobs[index].copyWith(status: ExportStatus.cancelled);
        notifyListeners();
      }
    } catch (e) {
      _error = e.toString();
      notifyListeners();
    }
  }
  
  void _pollExportProgress(String jobId) {
    Timer.periodic(Duration(seconds: 5), (timer) async {
      try {
        final updatedJob = await ExportService.getExportJob(jobId);
        
        final index = _exportJobs.indexWhere((job) => job.id == jobId);
        if (index != -1) {
          _exportJobs[index] = updatedJob;
          notifyListeners();
        }
        
        // Stop polling when job is complete or failed
        if (updatedJob.status == ExportStatus.completed ||
            updatedJob.status == ExportStatus.failed ||
            updatedJob.status == ExportStatus.cancelled) {
          timer.cancel();
        }
      } catch (e) {
        timer.cancel();
      }
    });
  }
}
```

### **Export Models**
```dart
class ExportJob {
  final String id;
  final ExportType exportType;
  final ExportFormat format;
  final ExportStatus status;
  final DateTime createdAt;
  final DateTime? completedAt;
  final DateTime? expiresAt;
  final String? downloadUrl;
  final ExportFileInfo? fileInfo;
  final ExportProgress? progress;
  final ExportFilters? filters;
  final ExportOptions? options;
  final ExportStatistics? statistics;
  
  const ExportJob({
    required this.id,
    required this.exportType,
    required this.format,
    required this.status,
    required this.createdAt,
    this.completedAt,
    this.expiresAt,
    this.downloadUrl,
    this.fileInfo,
    this.progress,
    this.filters,
    this.options,
    this.statistics,
  });
  
  factory ExportJob.fromJson(Map<String, dynamic> json) {
    return ExportJob(
      id: json['id'],
      exportType: ExportType.values.firstWhere(
        (e) => e.toString().split('.').last == json['export_type']
      ),
      format: ExportFormat.values.firstWhere(
        (e) => e.toString().split('.').last == json['format']
      ),
      status: ExportStatus.values.firstWhere(
        (e) => e.toString().split('.').last == json['status']
      ),
      createdAt: DateTime.parse(json['created_at']),
      completedAt: json['completed_at'] != null 
          ? DateTime.parse(json['completed_at'])
          : null,
      expiresAt: json['expires_at'] != null 
          ? DateTime.parse(json['expires_at'])
          : null,
      downloadUrl: json['download_url'],
      fileInfo: json['file_info'] != null 
          ? ExportFileInfo.fromJson(json['file_info'])
          : null,
      progress: json['progress'] != null 
          ? ExportProgress.fromJson(json['progress'])
          : null,
      filters: json['filters'] != null 
          ? ExportFilters.fromJson(json['filters'])
          : null,
      options: json['options'] != null 
          ? ExportOptions.fromJson(json['options'])
          : null,
      statistics: json['statistics'] != null 
          ? ExportStatistics.fromJson(json['statistics'])
          : null,
    );
  }
  
  ExportJob copyWith({
    ExportStatus? status,
    DateTime? completedAt,
    String? downloadUrl,
    ExportFileInfo? fileInfo,
    ExportProgress? progress,
    ExportStatistics? statistics,
  }) {
    return ExportJob(
      id: id,
      exportType: exportType,
      format: format,
      status: status ?? this.status,
      createdAt: createdAt,
      completedAt: completedAt ?? this.completedAt,
      expiresAt: expiresAt,
      downloadUrl: downloadUrl ?? this.downloadUrl,
      fileInfo: fileInfo ?? this.fileInfo,
      progress: progress ?? this.progress,
      filters: filters,
      options: options,
      statistics: statistics ?? this.statistics,
    );
  }
  
  bool get isActive => status == ExportStatus.queued || status == ExportStatus.processing;
  bool get isCompleted => status == ExportStatus.completed;
  bool get isDownloadable => isCompleted && downloadUrl != null && !isExpired;
  bool get isExpired => expiresAt != null && DateTime.now().isAfter(expiresAt!);
  
  String get statusDisplayName {
    switch (status) {
      case ExportStatus.queued:
        return 'Queued';
      case ExportStatus.processing:
        return 'Processing';
      case ExportStatus.completed:
        return 'Completed';
      case ExportStatus.failed:
        return 'Failed';
      case ExportStatus.cancelled:
        return 'Cancelled';
      case ExportStatus.expired:
        return 'Expired';
    }
  }
}

enum ExportType { chatHistory, userData, preferences, fullBackup }
enum ExportFormat { json, markdown, pdf, html, csv, xml, txt }
enum ExportStatus { queued, processing, completed, failed, cancelled, expired }

extension ExportFormatExtension on ExportFormat {
  String get extension {
    switch (this) {
      case ExportFormat.json:
        return 'json';
      case ExportFormat.markdown:
        return 'md';
      case ExportFormat.pdf:
        return 'pdf';
      case ExportFormat.html:
        return 'html';
      case ExportFormat.csv:
        return 'csv';
      case ExportFormat.xml:
        return 'xml';
      case ExportFormat.txt:
        return 'txt';
    }
  }
  
  String get mimeType {
    switch (this) {
      case ExportFormat.json:
        return 'application/json';
      case ExportFormat.markdown:
        return 'text/markdown';
      case ExportFormat.pdf:
        return 'application/pdf';
      case ExportFormat.html:
        return 'text/html';
      case ExportFormat.csv:
        return 'text/csv';
      case ExportFormat.xml:
        return 'application/xml';
      case ExportFormat.txt:
        return 'text/plain';
    }
  }
}

class ExportFilters {
  final List<String>? chatIds;
  final List<String>? folderIds;
  final DateRange? dateRange;
  final bool includeMetadata;
  final bool includeAttachments;
  final List<String>? messageTypes;
  final List<String>? tags;
  
  const ExportFilters({
    this.chatIds,
    this.folderIds,
    this.dateRange,
    this.includeMetadata = true,
    this.includeAttachments = false,
    this.messageTypes,
    this.tags,
  });
  
  factory ExportFilters.fromJson(Map<String, dynamic> json) {
    return ExportFilters(
      chatIds: json['chat_ids']?.cast<String>(),
      folderIds: json['folder_ids']?.cast<String>(),
      dateRange: json['date_range'] != null 
          ? DateRange.fromJson(json['date_range'])
          : null,
      includeMetadata: json['include_metadata'] ?? true,
      includeAttachments: json['include_attachments'] ?? false,
      messageTypes: json['message_types']?.cast<String>(),
      tags: json['tags']?.cast<String>(),
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      if (chatIds != null) 'chat_ids': chatIds,
      if (folderIds != null) 'folder_ids': folderIds,
      if (dateRange != null) 'date_range': dateRange!.toJson(),
      'include_metadata': includeMetadata,
      'include_attachments': includeAttachments,
      if (messageTypes != null) 'message_types': messageTypes,
      if (tags != null) 'tags': tags,
    };
  }
}
```

---

## 🔄 Migration from Local Export

### **Hybrid Export Approach**
```dart
class HybridExportService {
  // Support both local and cloud exports during migration
  static Future<void> exportChatHistory({
    required ExportFormat format,
    ExportFilters? filters,
    bool useLocalFallback = true,
  }) async {
    try {
      // Try cloud export first
      await ExportService.createExport(
        exportType: ExportType.chatHistory,
        format: format,
        filters: filters,
      );
    } catch (e) {
      if (useLocalFallback) {
        // Fallback to local export
        await _performLocalExport(format, filters);
      } else {
        rethrow;
      }
    }
  }
  
  static Future<void> _performLocalExport(
    ExportFormat format,
    ExportFilters? filters,
  ) async {
    // Use existing local export logic as fallback
    final chatSessions = await _getLocalChatSessions(filters);
    
    switch (format) {
      case ExportFormat.markdown:
        await _exportAsMarkdown(chatSessions);
        break;
      case ExportFormat.json:
        await _exportAsJson(chatSessions);
        break;
      default:
        await _exportAsText(chatSessions);
    }
  }
}
```

---

**This export system provides comprehensive data export capabilities with multiple formats, flexible filtering, and reliable delivery while maintaining compatibility with the existing Flutter app's export functionality.**
