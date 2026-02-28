# file_picker 10.3.10 详解与实战

## 第1章 认识 file_picker

### 1.1 什么是 file_picker

`file_picker` 是 Flutter 生态中最流行、功能最全面的文件选择器插件，由 Miguel Ruivo 开发和维护。它允许开发者使用原生文件浏览器让用户选择单个或多个文件，支持扩展名过滤、目录选择、文件保存等多种功能。

与其他 Flutter 文件选择库相比，`file_picker` 的核心特点在于：

| 特性       | file_picker          | file_selector        | 其他库 |
| ---------- | -------------------- | -------------------- | ------ |
| 维护活跃度 | ⭐⭐⭐⭐⭐ 极高      | ⭐⭐⭐⭐ 高          | 各异   |
| 功能丰富度 | ⭐⭐⭐⭐⭐ 非常全面  | ⭐⭐⭐⭐ 较全面      | 各异   |
| 平台支持   | Android/iOS/Web/桌面 | Android/iOS/Web/桌面 | 各异   |
| 易用性     | ⭐⭐⭐⭐⭐ 简单直观  | ⭐⭐⭐⭐ 较简单      | 各异   |
| 社区活跃度 | 极高                 | 高                   | 各异   |
| 文档完善度 | ⭐⭐⭐⭐⭐ 非常完善  | ⭐⭐⭐⭐ 较完善      | 各异   |

Flutter 框架小知识

**原生文件选择器的优势**

`file_picker` 使用各平台的原生文件选择器：

| 平台    | 原生选择器                     | 特点               |
| ------- | ------------------------------ | ------------------ |
| Android | Storage Access Framework       | 支持云存储集成     |
| iOS     | UIDocumentPickerViewController | 与 iCloud 深度集成 |
| Web     | HTML5 File API                 | 浏览器原生支持     |
| Windows | Common Item Dialog             | 现代 Windows 风格  |
| macOS   | NSOpenPanel/NSSavePanel        | 原生 macOS 体验    |
| Linux   | GTK File Chooser               | GNOME/KDE 兼容     |

使用原生选择器的好处是用户可以获得熟悉的操作体验，同时自动支持云存储服务（如 Google Drive、iCloud、Dropbox 等）。

### 1.2 file_picker 10.3.10 的新特性

版本 10.3.10 是 `file_picker` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **单文件选择**：选择单个文件
- **多文件选择**：同时选择多个文件
- **文件类型过滤**：按扩展名、MIME 类型过滤
- **目录选择**：选择文件夹路径
- **文件保存对话框**：保存文件时选择位置和名称
- **云文件支持**：可直接选择 Google Drive、iCloud 等云存储中的文件
- **内存加载**：将文件数据直接加载到内存（Uint8List）
- **压缩支持**：选择图片时可自动压缩
- **读取流支持**：大文件可使用流式读取
- **XFile 兼容**：返回结果可轻松转换为 XFile

#### 1.2.2 平台支持

| API                           | Android | iOS | Linux | macOS | Windows | Web |
| ----------------------------- | ------- | --- | ----- | ----- | ------- | --- |
| `pickFiles()`                 | ✅      | ✅  | ✅    | ✅    | ✅      | ✅  |
| `getDirectoryPath()`          | ✅      | ✅  | ✅    | ✅    | ✅      | ❌  |
| `saveFile()`                  | ✅      | ✅  | ✅    | ✅    | ✅      | ✅  |
| `clearTemporaryFiles()`       | ✅      | ✅  | ❌    | ❌    | ❌      | ❌  |
| `pickFileAndDirectoryPaths()` | ❌      | ❌  | ❌    | ✅    | ❌      | ❌  |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  file_picker: ^10.3.10
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加存储权限：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 读取存储权限 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    <!-- Android 13+ 需要更细粒度的权限 -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO"/>
    <uses-permission android:name="android.permission.READ_MEDIA_AUDIO"/>

    <application ...>
    </application>
</manifest>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加：

```xml
<dict>
    <!-- 其他配置... -->

    <!-- 相册访问权限 -->
    <key>NSPhotoLibraryUsageDescription</key>
    <string>此应用需要访问相册来选择文件</string>
</dict>
```

**Web 配置**：

无需额外配置，但需要在 `web/index.html` 中确保支持：

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- 其他配置... -->
  </head>
  <body>
    <script src="flutter_bootstrap.js" async></script>
  </body>
</html>
```

Dart Tips 语法小贴士

**权限处理最佳实践**

在使用 `file_picker` 之前，建议先检查并请求权限：

```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> checkStoragePermission() async {
  if (Platform.isAndroid) {
    // Android 13+ 使用新权限
    final sdkInt = await _getSdkInt();
    if (sdkInt >= 33) {
      final images = await Permission.photos.request();
      return images.isGranted;
    }
  }

  final status = await Permission.storage.request();
  return status.isGranted;
}

Future<int> _getSdkInt() async {
  // 获取 Android SDK 版本
  return 33; // 简化示例
}
```

---

## 第2章 FilePicker 类详解

### 2.1 类概述

`FilePicker` 是 `file_picker` 插件的核心类，它是一个抽象类，通过静态属性 `platform` 提供平台实现。

```dart
abstract class FilePicker {
  static FilePicker get platform => FilePickerPlatform.instance;

  // 主要方法
  Future<FilePickerResult?> pickFiles({...});
  Future<String?> getDirectoryPath({...});
  Future<String?> saveFile({...});
  Future<bool?> clearTemporaryFiles();
  Future<List<String>?> pickFileAndDirectoryPaths({...});
}
```

### 2.2 主要方法详解

#### 2.2.1 pickFiles() 方法

`pickFiles()` 是最常用的方法，用于打开文件选择器让用户选择文件。

```dart
Future<FilePickerResult?> pickFiles({
  String? dialogTitle,                    // 对话框标题
  String? initialDirectory,               // 初始目录
  FileType type = FileType.any,           // 文件类型过滤
  List<String>? allowedExtensions,        // 允许的扩展名列表
  Function(FilePickerStatus)? onFileLoading, // 加载状态回调
  bool allowCompression = false,          // 是否允许压缩
  int compressionQuality = 0,             // 压缩质量 (0-100)
  bool allowMultiple = false,             // 是否允许多选
  bool withData = false,                  // 是否加载文件数据到内存
  bool withReadStream = false,            // 是否返回读取流
  bool lockParentWindow = false,          // 是否锁定父窗口（桌面端）
  bool readSequential = false,            // 是否顺序读取
})
```

**参数详解**：

| 参数                 | 类型            | 必填 | 默认值         | 说明                   |
| -------------------- | --------------- | ---- | -------------- | ---------------------- |
| `dialogTitle`        | `String?`       | 否   | `null`         | 文件选择对话框的标题   |
| `initialDirectory`   | `String?`       | 否   | `null`         | 对话框打开时的初始目录 |
| `type`               | `FileType`      | 否   | `FileType.any` | 文件类型过滤器         |
| `allowedExtensions`  | `List<String>?` | 否   | `null`         | 自定义允许的扩展名     |
| `onFileLoading`      | `Function?`     | 否   | `null`         | 文件加载状态回调       |
| `allowCompression`   | `bool`          | 否   | `false`        | 图片是否允许压缩       |
| `compressionQuality` | `int`           | 否   | `0`            | 压缩质量，0-100        |
| `allowMultiple`      | `bool`          | 否   | `false`        | 是否允许多文件选择     |
| `withData`           | `bool`          | 否   | `false`        | 是否将文件加载到内存   |
| `withReadStream`     | `bool`          | 否   | `false`        | 是否返回文件读取流     |
| `lockParentWindow`   | `bool`          | 否   | `false`        | 桌面端是否锁定父窗口   |
| `readSequential`     | `bool`          | 否   | `false`        | 是否顺序读取多文件     |

**使用示例**：

```dart
// 基本用法 - 选择单个文件
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  print('选择的文件: ${result.files.single.path}');
} else {
  print('用户取消了选择');
}
```

#### 2.2.2 getDirectoryPath() 方法

用于选择目录路径。

```dart
Future<String?> getDirectoryPath({
  String? dialogTitle,          // 对话框标题
  bool lockParentWindow = false, // 是否锁定父窗口
  String? initialDirectory,     // 初始目录
})
```

**使用示例**：

```dart
String? selectedDirectory = await FilePicker.platform.getDirectoryPath(
  dialogTitle: '选择保存目录',
  initialDirectory: '/Users/username/Documents',
);

if (selectedDirectory != null) {
  print('选择的目录: $selectedDirectory');
}
```

#### 2.2.3 saveFile() 方法

打开保存文件对话框，让用户选择保存位置和文件名。

```dart
Future<String?> saveFile({
  String? dialogTitle,              // 对话框标题
  String? fileName,                 // 建议的文件名
  String? initialDirectory,         // 初始目录
  FileType type = FileType.any,     // 文件类型
  List<String>? allowedExtensions,  // 允许的扩展名
  Uint8List? bytes,                 // 要保存的文件数据
  bool lockParentWindow = false,    // 是否锁定父窗口
})
```

**使用示例**：

```dart
String? outputFile = await FilePicker.platform.saveFile(
  dialogTitle: '保存文件',
  fileName: 'document.pdf',
  type: FileType.custom,
  allowedExtensions: ['pdf'],
);

if (outputFile != null) {
  print('保存路径: $outputFile');
  // 写入文件内容
  await File(outputFile).writeAsBytes(fileData);
}
```

#### 2.2.4 clearTemporaryFiles() 方法

清除插件创建的临时文件。

```dart
Future<bool?> clearTemporaryFiles()
```

**使用示例**：

```dart
bool? result = await FilePicker.platform.clearTemporaryFiles();
if (result == true) {
  print('临时文件已清除');
}
```

#### 2.2.5 pickFileAndDirectoryPaths() 方法

同时选择文件和目录（仅 macOS 支持）。

```dart
Future<List<String>?> pickFileAndDirectoryPaths({
  String? initialDirectory,
  FileType type = FileType.any,
  List<String>? allowedExtensions,
})
```

### 2.3 FileType 枚举详解

`FileType` 用于指定要选择的文件类型：

```dart
enum FileType {
  any,        // 任何文件类型
  media,      // 媒体文件（图片+视频）
  image,      // 图片文件
  video,      // 视频文件
  audio,      // 音频文件
  custom,     // 自定义扩展名
}
```

**使用示例**：

```dart
// 选择图片
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.image,
);

// 选择视频
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.video,
);

// 选择音频
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.audio,
);

// 选择媒体文件（图片+视频）
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.media,
);

// 自定义扩展名
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.custom,
  allowedExtensions: ['pdf', 'doc', 'docx'],
);
```

Flutter 框架小知识

**FileType 与 MIME 类型的映射**

| FileType | MIME 类型        | 说明         |
| -------- | ---------------- | ------------ |
| `image`  | image/\*         | 所有图片格式 |
| `video`  | video/\*         | 所有视频格式 |
| `audio`  | audio/\*         | 所有音频格式 |
| `media`  | image/_, video/_ | 图片和视频   |
| `any`    | _/_              | 所有文件     |
| `custom` | 根据扩展名       | 自定义类型   |

---

## 第3章 FilePickerResult 类详解

### 3.1 类概述

`FilePickerResult` 是 `pickFiles()` 方法的返回类型，包含用户选择的文件信息。

```dart
class FilePickerResult {
  final List<PlatformFile> files;

  const FilePickerResult(this.files);

  // 便捷属性
  PlatformFile get single => files.single;
  List<String?> get paths => files.map((file) => file.path).toList();
  List<XFile> get xFiles => files.map((file) => file.xFile).toList();
}
```

### 3.2 属性详解

#### 3.2.1 files

| 属性 | 说明                   |
| ---- | ---------------------- |
| 类型 | `List<PlatformFile>`   |
| 用途 | 包含所有选择的文件信息 |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  allowMultiple: true,
);

if (result != null) {
  for (var file in result.files) {
    print('文件名: ${file.name}');
    print('文件路径: ${file.path}');
    print('文件大小: ${file.size} bytes');
  }
}
```

#### 3.2.2 single

| 属性 | 说明                             |
| ---- | -------------------------------- |
| 类型 | `PlatformFile`                   |
| 用途 | 获取单个文件（多选时会抛出异常） |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  PlatformFile file = result.single;  // 等同于 result.files.single
  print('选择的文件: ${file.name}');
}
```

#### 3.2.3 paths

| 属性 | 说明                       |
| ---- | -------------------------- |
| 类型 | `List<String?>`            |
| 用途 | 获取所有文件的完整路径列表 |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  allowMultiple: true,
);

if (result != null) {
  List<String?> paths = result.paths;
  print('所有路径: $paths');
}
```

#### 3.2.4 xFiles

| 属性 | 说明                              |
| ---- | --------------------------------- |
| 类型 | `List<XFile>`                     |
| 用途 | 获取 XFile 列表，便于与其他库集成 |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  // 转换为 XFile，可与 image_picker 等库共用
  List<XFile> xFiles = result.xFiles;
}
```

---

## 第4章 PlatformFile 类详解

### 4.1 类概述

`PlatformFile` 是 `file_picker` 中表示单个文件的核心类，包含文件的完整信息。

```dart
class PlatformFile {
  final String? path;           // 文件路径
  final String name;            // 文件名
  final Uint8List? bytes;       // 文件数据（内存中）
  final int size;               // 文件大小（字节）
  final Stream<List<int>>? readStream;  // 读取流
  final String? extension;      // 文件扩展名
  final XFile _xFile;           // 内部 XFile 对象
}
```

### 4.2 属性详解

#### 4.2.1 path

| 属性 | 说明            |
| ---- | --------------- |
| 类型 | `String?`       |
| 用途 | 文件的完整路径  |
| 注意 | Web 平台为 null |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  String? path = result.files.single.path;
  if (path != null) {
    File file = File(path);
    // 使用 File 对象进行操作
  }
}
```

#### 4.2.2 name

| 属性 | 说明                 |
| ---- | -------------------- |
| 类型 | `String`             |
| 用途 | 文件名（包含扩展名） |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  String fileName = result.files.single.name;
  print('文件名: $fileName');  // 例如: document.pdf
}
```

#### 4.2.3 bytes

| 属性 | 说明                      |
| ---- | ------------------------- |
| 类型 | `Uint8List?`              |
| 用途 | 文件的字节数据            |
| 条件 | 需要设置 `withData: true` |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  withData: true,  // 必须设置为 true
);

if (result != null) {
  Uint8List? bytes = result.files.single.bytes;
  if (bytes != null) {
    print('文件大小: ${bytes.length} bytes');
    // 可以直接上传到服务器
    // await uploadToServer(bytes);
  }
}
```

#### 4.2.4 size

| 属性 | 说明             |
| ---- | ---------------- |
| 类型 | `int`            |
| 用途 | 文件大小（字节） |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  int size = result.files.single.size;

  // 格式化为可读格式
  String formattedSize = _formatFileSize(size);
  print('文件大小: $formattedSize');
}

String _formatFileSize(int bytes) {
  if (bytes < 1024) return '$bytes B';
  if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
  if (bytes < 1024 * 1024 * 1024) {
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }
  return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(2)} GB';
}
```

#### 4.2.5 extension

| 属性 | 说明                 |
| ---- | -------------------- |
| 类型 | `String?`            |
| 用途 | 文件扩展名（不含点） |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  String? ext = result.files.single.extension;
  print('扩展名: $ext');  // 例如: pdf
}
```

#### 4.2.6 readStream

| 属性 | 说明                            |
| ---- | ------------------------------- |
| 类型 | `Stream<List<int>>?`            |
| 用途 | 文件读取流                      |
| 条件 | 需要设置 `withReadStream: true` |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  withReadStream: true,  // 必须设置为 true
);

if (result != null) {
  Stream<List<int>>? stream = result.files.single.readStream;
  if (stream != null) {
    // 分块读取大文件
    await for (var chunk in stream) {
      print('读取到 ${chunk.length} 字节');
    }
  }
}
```

### 4.3 xFile 属性

| 属性 | 说明                              |
| ---- | --------------------------------- |
| 类型 | `XFile`                           |
| 用途 | 获取 XFile 对象，便于与其他库集成 |

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles();

if (result != null) {
  XFile xFile = result.files.single.xFile;

  // 使用 XFile 的方法
  String mimeType = await xFile.mimeType;
  Uint8List bytes = await xFile.readAsBytes();
  String string = await xFile.readAsString();
}
```

Dart Tips 语法小贴士

**PlatformFile 与 File 的转换**

```dart
// PlatformFile 转 File（非 Web 平台）
PlatformFile platformFile = result.files.single;
File file = File(platformFile.path!);

// 使用 File 的完整 API
int fileSize = await file.length();
DateTime modified = await file.lastModified();
bool exists = await file.exists();

// Web 平台使用 bytes
if (kIsWeb) {
  Uint8List bytes = platformFile.bytes!;
  // 直接处理字节数据
}
```

---

## 第5章 实战应用

### 5.1 单文件选择

```dart
import 'dart:io';
import 'package:file_picker/file_picker.dart';
import 'package:flutter/material.dart';

class SingleFilePickerDemo extends StatefulWidget {
  @override
  SingleFilePickerDemoState createState() => SingleFilePickerDemoState();
}

class SingleFilePickerDemoState extends State<SingleFilePickerDemo> {
  File? _selectedFile;
  String? _fileName;
  int? _fileSize;

  Future<void> _pickFile() async {
    try {
      FilePickerResult? result = await FilePicker.platform.pickFiles(
        type: FileType.any,
        dialogTitle: '选择文件',
      );

      if (result != null) {
        setState(() {
          _fileName = result.files.single.name;
          _fileSize = result.files.single.size;

          // Web 平台 path 为 null
          if (result.files.single.path != null) {
            _selectedFile = File(result.files.single.path!);
          }
        });

        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('已选择: $_fileName')),
        );
      } else {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('用户取消了选择')),
        );
      }
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('选择文件失败: $e')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('单文件选择')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton.icon(
              onPressed: _pickFile,
              icon: const Icon(Icons.file_open),
              label: const Text('选择文件'),
            ),
            const SizedBox(height: 20),
            if (_fileName != null) ...[
              const Icon(Icons.insert_drive_file, size: 64, color: Colors.blue),
              const SizedBox(height: 10),
              Text('文件名: $_fileName'),
              if (_fileSize != null)
                Text('大小: ${_formatFileSize(_fileSize!)}'),
            ],
          ],
        ),
      ),
    );
  }

  String _formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }
}
```

### 5.2 多文件选择

```dart
class MultipleFilePickerDemo extends StatefulWidget {
  @override
  MultipleFilePickerDemoState createState() => MultipleFilePickerDemoState();
}

class MultipleFilePickerDemoState extends State<MultipleFilePickerDemo> {
  List<PlatformFile> _selectedFiles = [];

  Future<void> _pickMultipleFiles() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      allowMultiple: true,
      type: FileType.custom,
      allowedExtensions: ['jpg', 'jpeg', 'png', 'pdf', 'doc', 'docx'],
    );

    if (result != null) {
      setState(() {
        _selectedFiles = result.files;
      });
    }
  }

  void _removeFile(int index) {
    setState(() {
      _selectedFiles.removeAt(index);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('多文件选择')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: ElevatedButton.icon(
              onPressed: _pickMultipleFiles,
              icon: const Icon(Icons.file_copy),
              label: const Text('选择多个文件'),
            ),
          ),
          Expanded(
            child: _selectedFiles.isEmpty
                ? const Center(child: Text('未选择文件'))
                : ListView.builder(
                    itemCount: _selectedFiles.length,
                    itemBuilder: (context, index) {
                      final file = _selectedFiles[index];
                      return ListTile(
                        leading: _getFileIcon(file.extension),
                        title: Text(file.name),
                        subtitle: Text(_formatFileSize(file.size)),
                        trailing: IconButton(
                          icon: const Icon(Icons.delete, color: Colors.red),
                          onPressed: () => _removeFile(index),
                        ),
                      );
                    },
                  ),
          ),
          if (_selectedFiles.isNotEmpty)
            Padding(
              padding: const EdgeInsets.all(16),
              child: Text(
                '共 ${_selectedFiles.length} 个文件，'
                '总计 ${_formatTotalSize()}',
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
        ],
      ),
    );
  }

  Widget _getFileIcon(String? extension) {
    IconData iconData;
    Color color;

    switch (extension?.toLowerCase()) {
      case 'jpg':
      case 'jpeg':
      case 'png':
      case 'gif':
        iconData = Icons.image;
        color = Colors.blue;
        break;
      case 'pdf':
        iconData = Icons.picture_as_pdf;
        color = Colors.red;
        break;
      case 'doc':
      case 'docx':
        iconData = Icons.description;
        color = Colors.blue;
        break;
      default:
        iconData = Icons.insert_drive_file;
        color = Colors.grey;
    }

    return Icon(iconData, color: color);
  }

  String _formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }

  String _formatTotalSize() {
    int total = _selectedFiles.fold(0, (sum, file) => sum + file.size);
    return _formatFileSize(total);
  }
}
```

### 5.3 图片选择与压缩

```dart
class ImagePickerWithCompression extends StatefulWidget {
  @override
  ImagePickerWithCompressionState createState() =>
      ImagePickerWithCompressionState();
}

class ImagePickerWithCompressionState extends State<ImagePickerWithCompression> {
  Uint8List? _imageBytes;
  String? _imageName;
  int? _originalSize;
  int? _compressedSize;

  Future<void> _pickAndCompressImage() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      type: FileType.image,
      allowCompression: true,
      compressionQuality: 70,  // 压缩质量 70%
      withData: true,
    );

    if (result != null) {
      setState(() {
        _imageName = result.files.single.name;
        _imageBytes = result.files.single.bytes;
        _originalSize = result.files.single.size;
        _compressedSize = _imageBytes?.length;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('图片选择与压缩')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton.icon(
              onPressed: _pickAndCompressImage,
              icon: const Icon(Icons.image),
              label: const Text('选择图片'),
            ),
            const SizedBox(height: 20),
            if (_imageBytes != null) ...[
              Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Image.memory(_imageBytes!, fit: BoxFit.cover),
              ),
              const SizedBox(height: 16),
              Text('文件名: $_imageName'),
              Text('原始大小: ${_formatSize(_originalSize ?? 0)}'),
              Text('压缩后: ${_formatSize(_compressedSize ?? 0)}'),
              Text(
                '压缩率: ${_calculateCompressionRate()}%',
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  color: Colors.green,
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }

  String _formatSize(int bytes) {
    if (bytes < 1024 * 1024) {
      return '${(bytes / 1024).toStringAsFixed(2)} KB';
    }
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }

  String _calculateCompressionRate() {
    if (_originalSize == null || _compressedSize == null) return '0';
    double rate = (1 - _compressedSize! / _originalSize!) * 100;
    return rate.toStringAsFixed(1);
  }
}
```

### 5.4 文件上传（Web 支持）

```dart
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';

class FileUploadService {
  final Dio _dio = Dio();

  /// 上传单个文件
  Future<void> uploadFile(PlatformFile file, String uploadUrl) async {
    try {
      FormData formData;

      if (kIsWeb) {
        // Web 平台使用 bytes
        formData = FormData.fromMap({
          'file': MultipartFile.fromBytes(
            file.bytes!,
            filename: file.name,
          ),
        });
      } else {
        // 移动端使用文件路径
        formData = FormData.fromMap({
          'file': await MultipartFile.fromFile(
            file.path!,
            filename: file.name,
          ),
        });
      }

      Response response = await _dio.post(
        uploadUrl,
        data: formData,
        onSendProgress: (sent, total) {
          double progress = sent / total * 100;
          print('上传进度: ${progress.toStringAsFixed(2)}%');
        },
      );

      print('上传成功: ${response.data}');
    } catch (e) {
      print('上传失败: $e');
      rethrow;
    }
  }

  /// 批量上传文件
  Future<void> uploadMultipleFiles(
    List<PlatformFile> files,
    String uploadUrl,
  ) async {
    for (var file in files) {
      await uploadFile(file, uploadUrl);
    }
  }
}

// 使用示例
class FileUploadDemo extends StatefulWidget {
  @override
  FileUploadDemoState createState() => FileUploadDemoState();
}

class FileUploadDemoState extends State<FileUploadDemo> {
  final FileUploadService _uploadService = FileUploadService();
  List<PlatformFile> _selectedFiles = [];
  bool _isUploading = false;
  double _uploadProgress = 0;

  Future<void> _pickFiles() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      allowMultiple: true,
      withData: true,  // Web 平台需要
    );

    if (result != null) {
      setState(() {
        _selectedFiles = result.files;
      });
    }
  }

  Future<void> _uploadFiles() async {
    setState(() => _isUploading = true);

    try {
      for (int i = 0; i < _selectedFiles.length; i++) {
        setState(() => _uploadProgress = i / _selectedFiles.length);

        await _uploadService.uploadFile(
          _selectedFiles[i],
          'https://your-api.com/upload',
        );
      }

      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('所有文件上传成功')),
      );

      setState(() {
        _selectedFiles = [];
        _uploadProgress = 0;
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('上传失败: $e')),
      );
    } finally {
      setState(() => _isUploading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('文件上传')),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: _isUploading ? null : _pickFiles,
            child: const Text('选择文件'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: _selectedFiles.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text(_selectedFiles[index].name),
                  subtitle: Text(
                    '${_formatSize(_selectedFiles[index].size)}',
                  ),
                );
              },
            ),
          ),
          if (_isUploading) ...[
            LinearProgressIndicator(value: _uploadProgress),
            const SizedBox(height: 8),
            Text('上传中... ${(_uploadProgress * 100).toInt()}%'),
          ],
          ElevatedButton(
            onPressed: _selectedFiles.isEmpty || _isUploading
                ? null
                : _uploadFiles,
            child: const Text('开始上传'),
          ),
        ],
      ),
    );
  }

  String _formatSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }
}
```

### 5.5 目录选择与文件保存

```dart
class DirectoryAndSaveDemo extends StatefulWidget {
  @override
  DirectoryAndSaveDemoState createState() => DirectoryAndSaveDemoState();
}

class DirectoryAndSaveDemoState extends State<DirectoryAndSaveDemo> {
  String? _selectedDirectory;
  String? _savedFilePath;

  Future<void> _pickDirectory() async {
    String? directory = await FilePicker.platform.getDirectoryPath(
      dialogTitle: '选择工作目录',
    );

    if (directory != null) {
      setState(() => _selectedDirectory = directory);

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('已选择目录: $directory')),
      );
    }
  }

  Future<void> _saveFile() async {
    String? outputPath = await FilePicker.platform.saveFile(
      dialogTitle: '保存配置文件',
      fileName: 'config.json',
      type: FileType.custom,
      allowedExtensions: ['json'],
    );

    if (outputPath != null) {
      // 创建示例 JSON 内容
      final content = '''
{
  "app_name": "My App",
  "version": "1.0.0",
  "settings": {
    "theme": "dark",
    "language": "zh-CN"
  }
}
''';

      // 写入文件
      final file = File(outputPath);
      await file.writeAsString(content);

      setState(() => _savedFilePath = outputPath);

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('文件已保存到: $outputPath')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('目录选择与文件保存')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton.icon(
              onPressed: _pickDirectory,
              icon: const Icon(Icons.folder_open),
              label: const Text('选择目录'),
            ),
            const SizedBox(height: 16),
            if (_selectedDirectory != null)
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text('已选择: $_selectedDirectory'),
              ),
            const Divider(),
            ElevatedButton.icon(
              onPressed: _saveFile,
              icon: const Icon(Icons.save),
              label: const Text('保存文件'),
            ),
            const SizedBox(height: 16),
            if (_savedFilePath != null)
              Padding(
                padding: const EdgeInsets.all(16),
                child: Text('已保存到: $_savedFilePath'),
              ),
          ],
        ),
      ),
    );
  }
}
```

Flutter 框架小知识

**Web 平台的特殊处理**

Web 平台由于浏览器安全限制，有以下不同：

| 特性     | 移动端/桌面端 | Web 端                    |
| -------- | ------------- | ------------------------- |
| `path`   | 可用          | 始终为 null               |
| `bytes`  | 可选          | 必须设置 `withData: true` |
| 文件访问 | 完整路径      | 仅内存访问                |
| 持久化   | 直接写入      | 需要下载                  |

Web 平台保存文件的解决方案：

```dart
import 'dart:html' as html;

void downloadFileWeb(Uint8List bytes, String fileName) {
  final blob = html.Blob([bytes]);
  final url = html.Url.createObjectUrlFromBlob(blob);

  final anchor = html.AnchorElement(href: url)
    ..setAttribute('download', fileName)
    ..click();

  html.Url.revokeObjectUrl(url);
}
```

---

## 附录

### 附录A：方法速查表

| 方法                          | 返回值                      | 用途               | Web 支持 |
| ----------------------------- | --------------------------- | ------------------ | -------- |
| `pickFiles()`                 | `Future<FilePickerResult?>` | 选择文件           | ✅       |
| `getDirectoryPath()`          | `Future<String?>`           | 选择目录           | ❌       |
| `saveFile()`                  | `Future<String?>`           | 保存文件           | ✅       |
| `clearTemporaryFiles()`       | `Future<bool?>`             | 清除临时文件       | ❌       |
| `pickFileAndDirectoryPaths()` | `Future<List<String>?>`     | 同时选择文件和目录 | ❌       |

### 附录B：FileType 枚举速查表

| 值       | 说明                  | 适用场景   |
| -------- | --------------------- | ---------- |
| `any`    | 任何文件类型          | 通用选择   |
| `media`  | 媒体文件（图片+视频） | 多媒体应用 |
| `image`  | 图片文件              | 图片上传   |
| `video`  | 视频文件              | 视频上传   |
| `audio`  | 音频文件              | 音频上传   |
| `custom` | 自定义扩展名          | 特定格式   |

### 附录C：PlatformFile 属性速查表

| 属性         | 类型                 | Web 可用     | 说明         |
| ------------ | -------------------- | ------------ | ------------ |
| `path`       | `String?`            | ❌           | 文件完整路径 |
| `name`       | `String`             | ✅           | 文件名       |
| `bytes`      | `Uint8List?`         | ✅（需设置） | 文件字节数据 |
| `size`       | `int`                | ✅           | 文件大小     |
| `extension`  | `String?`            | ✅           | 文件扩展名   |
| `readStream` | `Stream<List<int>>?` | ❌           | 文件读取流   |
| `xFile`      | `XFile`              | ✅           | XFile 对象   |

### 附录D：常见问题与解决方案

#### Q1: 文件选择器不显示或崩溃

**问题**：调用 `pickFiles()` 后无反应或应用崩溃。

**解决方案**：

```dart
// 1. 检查权限
final status = await Permission.storage.request();
if (!status.isGranted) {
  // 引导用户开启权限
  return;
}

// 2. Android 11+ 需要特殊处理
// 在 AndroidManifest.xml 中添加:
// <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />

// 3. iOS 需要配置 Info.plist
// NSPhotoLibraryUsageDescription
```

#### Q2: Web 平台无法获取文件路径

**问题**：Web 平台 `path` 属性为 null。

**解决方案**：

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  withData: true,  // Web 平台必须设置为 true
);

if (result != null) {
  // Web 平台使用 bytes
  Uint8List? bytes = result.files.single.bytes;
  // 不要依赖 path
}
```

#### Q3: 扩展名过滤不生效

**问题**：设置了 `allowedExtensions` 但所有文件仍显示。

**解决方案**：

```dart
// 确保 type 设置为 FileType.custom
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.custom,  // 必须设置
  allowedExtensions: ['pdf', 'doc'],
);

// 某些平台（如 iOS）可能不支持扩展名过滤
// 可以在选择后进行二次验证
```

#### Q4: 大文件导致内存溢出

**问题**：选择大文件时应用卡顿或崩溃。

**解决方案**：

```dart
// 使用流式读取
FilePickerResult? result = await FilePicker.platform.pickFiles(
  withReadStream: true,  // 使用流
  withData: false,        // 不要加载到内存
);

if (result != null) {
  Stream<List<int>>? stream = result.files.single.readStream;
  if (stream != null) {
    await for (var chunk in stream) {
      // 分块处理
    }
  }
}
```

### 附录E：参考资源

| 资源        | 链接                                                                    |
| ----------- | ----------------------------------------------------------------------- |
| Pub 包页面  | https://pub.dev/packages/file_picker                                    |
| GitHub 仓库 | https://github.com/miguelpruivo/flutter_file_picker                     |
| Wiki 文档   | https://github.com/miguelpruivo/flutter_file_picker/wiki                |
| API 参考    | https://pub.dev/documentation/file_picker/latest/                       |
| 示例应用    | https://github.com/miguelpruivo/flutter_file_picker/tree/master/example |

### 附录F：相关插件推荐

| 插件                 | 用途                          |
| -------------------- | ----------------------------- |
| `permission_handler` | 权限管理                      |
| `path_provider`      | 获取系统路径                  |
| `dio`                | 网络请求和文件上传            |
| `image_picker`       | 相机和相册选择                |
| `file_selector`      | 跨平台文件选择（Google 官方） |
| `open_filex`         | 打开文件                      |
| `share_plus`         | 分享文件                      |

---

**文档版本**: 1.0  
**适用版本**: file_picker ^10.3.10  
**最后更新**: 2025年
