# file_selector 1.1.0 详解与实战

## 第1章 认识 file_selector

### 1.1 什么是 file_selector

`file_selector` 是 Flutter 官方团队（Google）维护的跨平台文件选择插件，提供打开文件、保存文件和选择目录的功能。它采用与平台原生文件对话框一致的用户界面，为用户提供熟悉的操作体验。

与其他 Flutter 文件选择库相比，`file_selector` 的核心特点在于：

| 特性       | file_selector     | file_picker   | 其他库 |
| ---------- | ----------------- | ------------- | ------ |
| 官方维护   | ✅ Flutter 团队   | 社区维护      | 各异   |
| API 设计   | 简洁统一          | 功能丰富      | 各异   |
| 类型系统   | XTypeGroup 强类型 | FileType 枚举 | 各异   |
| XFile 集成 | 原生支持          | 兼容支持      | 各异   |
| 桌面端支持 | ⭐⭐⭐⭐⭐ 优秀   | ⭐⭐⭐⭐ 良好 | 各异   |
| Web 支持   | ✅                | ✅            | 各异   |

Flutter 框架小知识

**file_selector 与 file_picker 的选择**

| 场景         | 推荐库        | 理由               |
| ------------ | ------------- | ------------------ |
| 需要官方支持 | file_selector | Flutter 团队维护   |
| 需要丰富功能 | file_picker   | 功能更全面         |
| 桌面端应用   | file_selector | 原生对话框体验更好 |
| 移动端应用   | file_picker   | 移动端优化更好     |
| 跨平台统一   | file_selector | API 更一致         |

两个库可以共存于同一个项目，根据具体场景选择使用。

### 1.2 file_selector 1.1.0 的新特性

版本 1.1.0 是 `file_selector` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **打开单个文件**：使用原生对话框选择单个文件
- **打开多个文件**：同时选择多个文件
- **保存文件**：打开保存对话框，指定文件名和位置
- **选择目录**：选择文件夹路径
- **类型过滤**：使用 XTypeGroup 进行精确的文件类型过滤
- **XFile 集成**：返回结果直接为 XFile 对象
- **跨平台一致**：统一的 API 接口

#### 1.2.2 平台支持

| 功能         | Android | iOS | Linux | macOS | Windows | Web |
| ------------ | ------- | --- | ----- | ----- | ------- | --- |
| 打开单个文件 | ✅      | ✅  | ✅    | ✅    | ✅      | ✅  |
| 打开多个文件 | ✅      | ✅  | ✅    | ✅    | ✅      | ✅  |
| 保存文件     | ❌      | ❌  | ✅    | ✅    | ✅      | ❌  |
| 选择目录     | ✅      | ❌  | ✅    | ✅    | ✅      | ❌  |

#### 1.2.3 类型过滤支持

| 过滤器类型               | Android | iOS | Linux | macOS | Web  | Windows |
| ------------------------ | ------- | --- | ----- | ----- | ---- | ------- |
| `extensions`             | ✅      | ❌  | ✅    | ✅    | ✅   | ✅      |
| `mimeTypes`              | ✅      | ❌  | ✅    | ✅    | ✅\* | ❌      |
| `uniformTypeIdentifiers` | ❌      | ✅  | ❌    | ✅    | ❌   | ❌      |
| `webWildCards`           | ❌      | ❌  | ❌    | ❌    | ✅   | ❌      |

\*macOS 11 (Big Sur) 以下版本不支持 `mimeTypes`

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  file_selector: ^1.1.0
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**macOS 配置**：

在 `macos/Runner/DebugProfile.entitlements` 和 `macos/Runner/Release.entitlements` 中添加：

```xml
<!-- 只读访问 -->
<dict>
  <key>com.apple.security.files.user-selected.read-only</key>
  <true/>
</dict>

<!-- 或读写访问 -->
<dict>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
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

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

  <application ...>
  </application>
</manifest>
```

Dart Tips 语法小贴士

**XFile 与 cross_file 包**

`file_selector` 返回的 `XFile` 来自 `cross_file` 包，它是一个跨平台的文件抽象：

```dart
import 'package:cross_file/cross_file.dart';

// XFile 提供统一的文件操作接口
final XFile file = await openFile(...);

// 读取文件内容
final String content = await file.readAsString();
final Uint8List bytes = await file.readAsBytes();
final Stream<String> lines = file.openRead().transform(utf8.decoder);

// 获取文件信息
final String name = file.name;
final String? mimeType = file.mimeType;
final int length = await file.length();
```

---

## 第2章 顶层函数详解

### 2.1 openFile() 函数

`openFile()` 用于打开文件选择对话框，让用户选择单个文件。

```dart
Future<XFile?> openFile({
  List<XTypeGroup>? acceptedTypeGroups,
  String? initialDirectory,
  String? confirmButtonText,
  String? suggestedName,
})
```

**参数详解**：

| 参数                 | 类型                | 必填 | 默认值 | 说明                         |
| -------------------- | ------------------- | ---- | ------ | ---------------------------- |
| `acceptedTypeGroups` | `List<XTypeGroup>?` | 否   | `null` | 接受的文件类型组             |
| `initialDirectory`   | `String?`           | 否   | `null` | 对话框打开的初始目录         |
| `confirmButtonText`  | `String?`           | 否   | `null` | 确认按钮文本（部分平台支持） |
| `suggestedName`      | `String?`           | 否   | `null` | 建议的文件名                 |

**返回值**：

- 用户选择文件：返回 `XFile` 对象
- 用户取消：返回 `null`

**使用示例**：

```dart
import 'package:file_selector/file_selector.dart';

// 基本用法 - 选择任何文件
Future<void> pickAnyFile() async {
  final XFile? file = await openFile();

  if (file != null) {
    print('选择的文件: ${file.name}');
    print('文件路径: ${file.path}');
  } else {
    print('用户取消了选择');
  }
}

// 选择特定类型的文件
Future<void> pickImageFile() async {
  const XTypeGroup imageTypeGroup = XTypeGroup(
    label: 'images',
    extensions: ['jpg', 'jpeg', 'png', 'gif'],
    uniformTypeIdentifiers: ['public.image'],
  );

  final XFile? file = await openFile(
    acceptedTypeGroups: [imageTypeGroup],
  );

  if (file != null) {
    final Uint8List bytes = await file.readAsBytes();
    print('图片大小: ${bytes.length} bytes');
  }
}
```

### 2.2 openFiles() 函数

`openFiles()` 用于打开文件选择对话框，让用户选择多个文件。

```dart
Future<List<XFile>> openFiles({
  List<XTypeGroup>? acceptedTypeGroups,
  String? initialDirectory,
  String? confirmButtonText,
  String? suggestedName,
})
```

**返回值**：

- 始终返回 `List<XFile>`，取消时返回空列表

**使用示例**：

```dart
// 选择多个图片文件
Future<void> pickMultipleImages() async {
  const XTypeGroup imageTypeGroup = XTypeGroup(
    label: 'images',
    extensions: ['jpg', 'jpeg', 'png'],
    uniformTypeIdentifiers: ['public.jpeg', 'public.png'],
  );

  final List<XFile> files = await openFiles(
    acceptedTypeGroups: [imageTypeGroup],
  );

  print('选择了 ${files.length} 个文件');

  for (final file in files) {
    print('  - ${file.name}');
  }
}

// 选择多种类型的文件
Future<void> pickDocuments() async {
  const XTypeGroup pdfTypeGroup = XTypeGroup(
    label: 'PDFs',
    extensions: ['pdf'],
    mimeTypes: ['application/pdf'],
  );

  const XTypeGroup docTypeGroup = XTypeGroup(
    label: 'Word Documents',
    extensions: ['doc', 'docx'],
    mimeTypes: [
      'application/msword',
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    ],
  );

  final List<XFile> files = await openFiles(
    acceptedTypeGroups: [pdfTypeGroup, docTypeGroup],
  );

  // 处理选中的文件
}
```

### 2.3 getSaveLocation() 函数

`getSaveLocation()` 用于打开保存文件对话框，让用户选择保存位置和文件名。

```dart
Future<FileSaveLocation?> getSaveLocation({
  List<XTypeGroup>? acceptedTypeGroups,
  String? initialDirectory,
  String? suggestedName,
  String? confirmButtonText,
})
```

**返回值**：

- 用户确认：返回 `FileSaveLocation` 对象
- 用户取消：返回 `null`

**FileSaveLocation 属性**：

| 属性   | 类型     | 说明               |
| ------ | -------- | ------------------ |
| `path` | `String` | 用户选择的保存路径 |

**使用示例**：

```dart
// 保存文本文件
Future<void> saveTextFile() async {
  const String fileName = 'my_document.txt';

  final FileSaveLocation? result = await getSaveLocation(
    suggestedName: fileName,
    acceptedTypeGroups: [
      const XTypeGroup(
        label: 'Text Files',
        extensions: ['txt'],
        mimeTypes: ['text/plain'],
      ),
    ],
  );

  if (result == null) {
    print('用户取消了保存');
    return;
  }

  // 创建文件内容
  final String content = 'Hello, World!';
  final Uint8List bytes = Uint8List.fromList(content.codeUnits);

  // 使用 XFile 保存
  final XFile textFile = XFile.fromData(
    bytes,
    mimeType: 'text/plain',
    name: fileName,
  );

  await textFile.saveTo(result.path);
  print('文件已保存到: ${result.path}');
}

// 保存 JSON 文件
Future<void> saveJsonFile(Map<String, dynamic> data) async {
  final FileSaveLocation? result = await getSaveLocation(
    suggestedName: 'data.json',
    acceptedTypeGroups: [
      const XTypeGroup(
        label: 'JSON Files',
        extensions: ['json'],
        mimeTypes: ['application/json'],
      ),
    ],
  );

  if (result != null) {
    final jsonString = jsonEncode(data);
    final file = File(result.path);
    await file.writeAsString(jsonString);
    print('JSON 已保存');
  }
}
```

### 2.4 getDirectoryPath() 函数

`getDirectoryPath()` 用于打开目录选择对话框，让用户选择文件夹。

```dart
Future<String?> getDirectoryPath({
  String? initialDirectory,
  String? confirmButtonText,
})
```

**返回值**：

- 用户选择目录：返回目录路径字符串
- 用户取消：返回 `null`

**使用示例**：

```dart
// 选择工作目录
Future<void> pickWorkingDirectory() async {
  final String? directoryPath = await getDirectoryPath(
    confirmButtonText: '选择此文件夹',
  );

  if (directoryPath != null) {
    print('选择的目录: $directoryPath');

    // 列出目录内容
    final directory = Directory(directoryPath);
    final entities = await directory.list().toList();

    print('目录内容:');
    for (final entity in entities) {
      print('  - ${entity.path.split('/').last}');
    }
  }
}

// 选择导出目录
Future<void> selectExportDirectory() async {
  final String? directoryPath = await getDirectoryPath(
    initialDirectory: '/Users/username/Documents',
    confirmButtonText: '导出到这里',
  );

  if (directoryPath != null) {
    // 执行导出操作
    await exportFilesTo(directoryPath);
  }
}
```

---

## 第3章 XTypeGroup 类详解

### 3.1 类概述

`XTypeGroup` 是 `file_selector` 中用于定义文件类型的核心类。它允许你使用多种方式指定要筛选的文件类型，确保跨平台兼容性。

```dart
class XTypeGroup {
  const XTypeGroup({
    this.label,
    this.extensions,
    this.mimeTypes,
    this.uniformTypeIdentifiers,
    this.webWildCards,
  });

  final String? label;
  final List<String>? extensions;
  final List<String>? mimeTypes;
  final List<String>? uniformTypeIdentifiers;
  final List<String>? webWildCards;
}
```

### 3.2 属性详解

#### 3.2.1 label

| 属性 | 说明                          |
| ---- | ----------------------------- |
| 类型 | `String?`                     |
| 用途 | 文件类型的描述标签            |
| 示例 | `'images'`、`'PDF Documents'` |

`label` 在某些平台的文件对话框中显示，帮助用户理解可选择的文件类型。

```dart
const XTypeGroup images = XTypeGroup(
  label: 'Image Files',  // 对话框中显示 "Image Files"
  extensions: ['jpg', 'png'],
);
```

#### 3.2.2 extensions

| 属性 | 说明                     |
| ---- | ------------------------ |
| 类型 | `List<String>?`          |
| 用途 | 文件扩展名列表（不含点） |
| 示例 | `['jpg', 'jpeg', 'png']` |

`extensions` 是最常用的过滤方式，在大多数平台上都受支持。

```dart
// 图片文件
const XTypeGroup images = XTypeGroup(
  label: 'images',
  extensions: ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp'],
);

// 文档文件
const XTypeGroup documents = XTypeGroup(
  label: 'documents',
  extensions: ['pdf', 'doc', 'docx', 'txt', 'rtf'],
);

// 代码文件
const XTypeGroup codeFiles = XTypeGroup(
  label: 'code',
  extensions: ['dart', 'js', 'ts', 'html', 'css', 'json'],
);
```

#### 3.2.3 mimeTypes

| 属性 | 说明                          |
| ---- | ----------------------------- |
| 类型 | `List<String>?`               |
| 用途 | MIME 类型列表                 |
| 示例 | `['image/jpeg', 'image/png']` |

`mimeTypes` 提供了更精确的文件类型识别，在 Android、Linux、macOS 11+ 和 Web 上受支持。

```dart
const XTypeGroup images = XTypeGroup(
  label: 'images',
  extensions: ['jpg', 'png'],
  mimeTypes: [
    'image/jpeg',
    'image/png',
    'image/gif',
    'image/webp',
  ],
);

const XTypeGroup videos = XTypeGroup(
  label: 'videos',
  extensions: ['mp4', 'mov', 'avi'],
  mimeTypes: [
    'video/mp4',
    'video/quicktime',
    'video/x-msvideo',
  ],
);
```

#### 3.2.4 uniformTypeIdentifiers

| 属性 | 说明                                    |
| ---- | --------------------------------------- |
| 类型 | `List<String>?`                         |
| 用途 | Apple 统一类型标识符 (UTI)              |
| 示例 | `['public.image', 'public.plain-text']` |

`uniformTypeIdentifiers` 是 Apple 平台的原生类型系统，在 iOS 和 macOS 上提供最精确的类型匹配。

```dart
const XTypeGroup images = XTypeGroup(
  label: 'images',
  extensions: ['jpg', 'png'],
  uniformTypeIdentifiers: [
    'public.image',        // 所有图片
    'public.jpeg',         // JPEG 图片
    'public.png',          // PNG 图片
  ],
);

const XTypeGroup pdfs = XTypeGroup(
  label: 'pdfs',
  extensions: ['pdf'],
  uniformTypeIdentifiers: ['com.adobe.pdf'],
);
```

常用 UTI 参考：

| 文件类型 | UTI                  |
| -------- | -------------------- |
| 所有图片 | `public.image`       |
| JPEG     | `public.jpeg`        |
| PNG      | `public.png`         |
| GIF      | `com.compuserve.gif` |
| PDF      | `com.adobe.pdf`      |
| 纯文本   | `public.plain-text`  |
| JSON     | `public.json`        |
| 视频     | `public.movie`       |
| 音频     | `public.audio`       |

#### 3.2.5 webWildCards

| 属性 | 说明                   |
| ---- | ---------------------- |
| 类型 | `List<String>?`        |
| 用途 | Web 平台的通配符模式   |
| 示例 | `['image/*', '*.pdf']` |

`webWildCards` 仅在 Web 平台上使用，提供与 HTML `<input type="file">` 的 `accept` 属性相同的语法。

```dart
const XTypeGroup images = XTypeGroup(
  label: 'images',
  extensions: ['jpg', 'png'],
  webWildCards: ['image/*'],  // 接受所有图片类型
);

const XTypeGroup documents = XTypeGroup(
  label: 'documents',
  extensions: ['pdf', 'doc'],
  webWildCards: ['.pdf', '.doc', '.docx'],
);
```

### 3.3 跨平台类型配置最佳实践

为了确保应用在所有平台上都能正常工作，建议为每个 `XTypeGroup` 配置所有相关的类型标识符：

```dart
// 推荐的完整配置方式
const XTypeGroup images = XTypeGroup(
  label: 'Images',
  extensions: ['jpg', 'jpeg', 'png', 'gif', 'webp'],
  mimeTypes: [
    'image/jpeg',
    'image/png',
    'image/gif',
    'image/webp',
  ],
  uniformTypeIdentifiers: [
    'public.image',
    'public.jpeg',
    'public.png',
    'com.compuserve.gif',
  ],
  webWildCards: ['image/*'],
);

// 使用
final XFile? file = await openFile(
  acceptedTypeGroups: [images],
);
```

Flutter 框架小知识

**平台特定的类型处理**

由于不同平台使用不同的类型系统，`file_selector` 会在运行时选择最合适的类型标识符：

```dart
// 平台自动选择
final XTypeGroup group = XTypeGroup(
  extensions: ['pdf'],           // Android, Linux, Windows, Web, macOS
  mimeTypes: ['application/pdf'], // Android, Linux, macOS 11+, Web
  uniformTypeIdentifiers: ['com.adobe.pdf'],  // iOS, macOS
);

// 不需要条件判断，file_selector 会自动处理
```

---

## 第4章 XFile 类详解

### 4.1 类概述

`XFile` 是 `cross_file` 包提供的跨平台文件抽象类。`file_selector` 的所有文件操作都返回 `XFile` 对象，提供统一的文件访问接口。

```dart
class XFile {
  // 从路径创建
  XFile(
    String path, {
    String? mimeType,
    String? name,
    int? length,
    DateTime? lastModified,
    Uint8List? bytes,
  });

  // 从内存数据创建
  XFile.fromData(
    Uint8List bytes, {
    String? mimeType,
    String? name,
    int? length,
    DateTime? lastModified,
    String? path,
  });

  // 从字节流创建
  XFile.fromStream(
    Stream<List<int>> stream,
    String path, {
    String? mimeType,
    String? name,
    int? length,
    DateTime? lastModified,
  });
}
```

### 4.2 属性详解

#### 4.2.1 path

| 属性 | 说明           |
| ---- | -------------- |
| 类型 | `String`       |
| 用途 | 文件的完整路径 |

```dart
final XFile? file = await openFile();

if (file != null) {
  print('文件路径: ${file.path}');
  // 例如: /Users/username/Documents/file.txt
}
```

#### 4.2.2 name

| 属性 | 说明                 |
| ---- | -------------------- |
| 类型 | `String`             |
| 用途 | 文件名（包含扩展名） |

```dart
final XFile? file = await openFile();

if (file != null) {
  print('文件名: ${file.name}');
  // 例如: document.pdf
}
```

#### 4.2.3 mimeType

| 属性 | 说明             |
| ---- | ---------------- |
| 类型 | `String?`        |
| 用途 | 文件的 MIME 类型 |

```dart
final XFile? file = await openFile();

if (file != null) {
  final String? mime = file.mimeType;
  print('MIME 类型: $mime');
  // 例如: application/pdf, image/jpeg
}
```

### 4.3 方法详解

#### 4.3.1 readAsBytes()

异步读取文件的完整内容为字节数组。

```dart
Future<Uint8List> readAsBytes()
```

使用示例：

```dart
final XFile? file = await openFile();

if (file != null) {
  final Uint8List bytes = await file.readAsBytes();
  print('文件大小: ${bytes.length} bytes');

  // 可以上传到服务器
  // await uploadFile(bytes);
}
```

#### 4.3.2 readAsString()

异步读取文件的完整内容为字符串。

```dart
Future<String> readAsString([Encoding encoding = utf8])
```

使用示例：

```dart
final XFile? file = await openFile(
  acceptedTypeGroups: [
    const XTypeGroup(
      label: 'text',
      extensions: ['txt', 'json', 'md'],
    ),
  ],
);

if (file != null) {
  final String content = await file.readAsString();
  print('文件内容:\n$content');
}
```

#### 4.3.3 readAsLines()

异步读取文件的完整内容为字符串列表（按行分割）。

```dart
Future<List<String>> readAsLines([Encoding encoding = utf8])
```

使用示例：

```dart
final XFile? file = await openFile(
  acceptedTypeGroups: [
    const XTypeGroup(
      label: 'text',
      extensions: ['txt', 'log', 'csv'],
    ),
  ],
);

if (file != null) {
  final List<String> lines = await file.readAsLines();
  print('文件共有 ${lines.length} 行');

  for (int i = 0; i < min(5, lines.length); i++) {
    print('  第${i + 1}行: ${lines[i]}');
  }
}
```

#### 4.3.4 openRead()

返回一个读取文件内容的流，适合处理大文件。

```dart
Stream<List<int>> openRead([int? start, int? end])
```

使用示例：

```dart
final XFile? file = await openFile();

if (file != null) {
  // 分块读取大文件
  final stream = file.openRead();
  int totalBytes = 0;

  await for (final chunk in stream) {
    totalBytes += chunk.length;
    print('读取了 ${chunk.length} 字节，总计 $totalBytes');
  }

  print('文件读取完成，总大小: $totalBytes bytes');
}
```

#### 4.3.5 length()

获取文件的字节大小。

```dart
Future<int> length()
```

使用示例：

```dart
final XFile? file = await openFile();

if (file != null) {
  final int size = await file.length();
  print('文件大小: ${_formatFileSize(size)}');
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

#### 4.3.6 lastModified()

获取文件的最后修改时间。

```dart
Future<DateTime> lastModified()
```

使用示例：

```dart
final XFile? file = await openFile();

if (file != null) {
  final DateTime modified = await file.lastModified();
  print('最后修改: ${modified.toLocal()}');
}
```

#### 4.3.7 saveTo()

将 `XFile`（从内存创建的）保存到指定路径。

```dart
Future<void> saveTo(String path)
```

使用示例：

```dart
// 创建内存中的文件
final Uint8List data = Uint8List.fromList('Hello, World!'.codeUnits);
final XFile file = XFile.fromData(
  data,
  name: 'hello.txt',
  mimeType: 'text/plain',
);

// 保存到指定路径
await file.saveTo('/path/to/save/hello.txt');
```

Dart Tips 语法小贴士

**XFile 与 File 的互操作**

```dart
import 'dart:io';

// XFile 转 File
final XFile xFile = await openFile() ?? XFile('');
final File file = File(xFile.path);

// 现在可以使用 dart:io File 的所有功能
final stat = await file.stat();
print('文件大小: ${stat.size}');
print('修改时间: ${stat.modified}');
print('访问权限: ${stat.mode}');

// File 转 XFile
final File ioFile = File('/path/to/file.txt');
final XFile newXFile = XFile(ioFile.path);
```

---

## 第5章 实战应用

### 5.1 单文件选择器

```dart
import 'package:file_selector/file_selector.dart';
import 'package:flutter/material.dart';

class SingleFileSelector extends StatefulWidget {
  @override
  SingleFileSelectorState createState() => SingleFileSelectorState();
}

class SingleFileSelectorState extends State<SingleFileSelector> {
  XFile? _selectedFile;
  String? _fileContent;
  bool _isLoading = false;

  Future<void> _pickTextFile() async {
    setState(() => _isLoading = true);

    try {
      const XTypeGroup textTypeGroup = XTypeGroup(
        label: 'Text Files',
        extensions: ['txt', 'md', 'json', 'yaml', 'dart'],
        mimeTypes: ['text/plain', 'application/json'],
      );

      final XFile? file = await openFile(
        acceptedTypeGroups: [textTypeGroup],
        confirmButtonText: '选择',
      );

      if (file != null) {
        final String content = await file.readAsString();

        setState(() {
          _selectedFile = file;
          _fileContent = content;
        });

        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('已选择: ${file.name}')),
        );
      }
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('读取文件失败: $e')),
      );
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('单文件选择')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            ElevatedButton.icon(
              onPressed: _isLoading ? null : _pickTextFile,
              icon: _isLoading
                  ? const SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    )
                  : const Icon(Icons.file_open),
              label: Text(_isLoading ? '读取中...' : '选择文本文件'),
            ),
            const SizedBox(height: 20),
            if (_selectedFile != null) ...[
              Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        '文件信息',
                        style: Theme.of(context).textTheme.titleMedium,
                      ),
                      const Divider(),
                      _buildInfoRow('文件名', _selectedFile!.name),
                      _buildInfoRow('路径', _selectedFile!.path),
                      FutureBuilder<int>(
                        future: _selectedFile!.length(),
                        builder: (context, snapshot) {
                          if (snapshot.hasData) {
                            return _buildInfoRow(
                              '大小',
                              _formatFileSize(snapshot.data!),
                            );
                          }
                          return const SizedBox.shrink();
                        },
                      ),
                    ],
                  ),
                ),
              ),
              const SizedBox(height: 20),
              if (_fileContent != null) ...[
                Text(
                  '文件内容预览:',
                  style: Theme.of(context).textTheme.titleMedium,
                ),
                const SizedBox(height: 8),
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.grey.shade100,
                    borderRadius: BorderRadius.circular(8),
                    border: Border.all(color: Colors.grey.shade300),
                  ),
                  child: Text(
                    _fileContent!.length > 500
                        ? '${_fileContent!.substring(0, 500)}...'
                        : _fileContent!,
                    style: const TextStyle(fontFamily: 'monospace'),
                  ),
                ),
              ],
            ],
          ],
        ),
      ),
    );
  }

  Widget _buildInfoRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 60,
            child: Text(
              '$label:',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(
            child: Text(
              value,
              style: TextStyle(color: Colors.grey.shade700),
              overflow: TextOverflow.ellipsis,
            ),
          ),
        ],
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

### 5.2 多文件选择器

```dart
class MultipleFileSelector extends StatefulWidget {
  @override
  MultipleFileSelectorState createState() => MultipleFileSelectorState();
}

class MultipleFileSelectorState extends State<MultipleFileSelector> {
  List<XFile> _selectedFiles = [];

  Future<void> _pickMultipleFiles() async {
    const XTypeGroup images = XTypeGroup(
      label: 'Images',
      extensions: ['jpg', 'jpeg', 'png', 'gif', 'webp', 'bmp'],
      mimeTypes: ['image/*'],
      uniformTypeIdentifiers: ['public.image'],
    );

    const XTypeGroup videos = XTypeGroup(
      label: 'Videos',
      extensions: ['mp4', 'mov', 'avi', 'mkv', 'webm'],
      mimeTypes: ['video/*'],
      uniformTypeIdentifiers: ['public.movie'],
    );

    final List<XFile> files = await openFiles(
      acceptedTypeGroups: [images, videos],
      confirmButtonText: '导入',
    );

    setState(() {
      _selectedFiles = files;
    });

    if (files.isNotEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('已选择 ${files.length} 个文件')),
      );
    }
  }

  void _removeFile(int index) {
    setState(() {
      _selectedFiles.removeAt(index);
    });
  }

  Future<void> _clearAll() async {
    final confirm = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('确认清除'),
        content: const Text('确定要清除所有已选文件吗？'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('取消'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('确定'),
          ),
        ],
      ),
    );

    if (confirm == true) {
      setState(() => _selectedFiles = []);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('多文件选择'),
        actions: [
          if (_selectedFiles.isNotEmpty)
            IconButton(
              icon: const Icon(Icons.clear_all),
              onPressed: _clearAll,
              tooltip: '清除全部',
            ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: ElevatedButton.icon(
              onPressed: _pickMultipleFiles,
              icon: const Icon(Icons.folder_open),
              label: const Text('选择媒体文件'),
            ),
          ),
          if (_selectedFiles.isNotEmpty)
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                children: [
                  Text(
                    '已选择 ${_selectedFiles.length} 个文件',
                    style: Theme.of(context).textTheme.titleSmall,
                  ),
                ],
              ),
            ),
          Expanded(
            child: _selectedFiles.isEmpty
                ? const Center(
                    child: Text(
                      '点击上方按钮选择文件',
                      style: TextStyle(color: Colors.grey),
                    ),
                  )
                : ListView.builder(
                    itemCount: _selectedFiles.length,
                    itemBuilder: (context, index) {
                      final file = _selectedFiles[index];
                      return ListTile(
                        leading: _getFileIcon(file.name),
                        title: Text(
                          file.name,
                          overflow: TextOverflow.ellipsis,
                        ),
                        subtitle: FutureBuilder<int>(
                          future: file.length(),
                          builder: (context, snapshot) {
                            if (snapshot.hasData) {
                              return Text(_formatFileSize(snapshot.data!));
                            }
                            return const SizedBox.shrink();
                          },
                        ),
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
              child: ElevatedButton(
                onPressed: () {
                  // 处理选中的文件
                  _processFiles();
                },
                child: const Text('开始处理'),
              ),
            ),
        ],
      ),
    );
  }

  Widget _getFileIcon(String fileName) {
    final ext = fileName.split('.').last.toLowerCase();

    IconData icon;
    Color color;

    switch (ext) {
      case 'jpg':
      case 'jpeg':
      case 'png':
      case 'gif':
      case 'webp':
      case 'bmp':
        icon = Icons.image;
        color = Colors.blue;
        break;
      case 'mp4':
      case 'mov':
      case 'avi':
      case 'mkv':
      case 'webm':
        icon = Icons.video_file;
        color = Colors.red;
        break;
      default:
        icon = Icons.insert_drive_file;
        color = Colors.grey;
    }

    return Icon(icon, color: color);
  }

  String _formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }

  void _processFiles() {
    // 实现文件处理逻辑
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('处理文件'),
        content: Text('即将处理 ${_selectedFiles.length} 个文件'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('确定'),
          ),
        ],
      ),
    );
  }
}
```

### 5.3 文件保存功能

```dart
class FileSaveDemo extends StatefulWidget {
  @override
  FileSaveDemoState createState() => FileSaveDemoState();
}

class FileSaveDemoState extends State<FileSaveDemo> {
  final TextEditingController _contentController = TextEditingController(
    text: 'Hello, Flutter!\n\nThis is a sample text file.',
  );
  final TextEditingController _fileNameController = TextEditingController(
    text: 'sample',
  );
  String? _lastSavedPath;

  Future<void> _saveTextFile() async {
    final String fileName = '${_fileNameController.text}.txt';

    final FileSaveLocation? result = await getSaveLocation(
      suggestedName: fileName,
      acceptedTypeGroups: const [
        XTypeGroup(
          label: 'Text Files',
          extensions: ['txt'],
          mimeTypes: ['text/plain'],
        ),
      ],
    );

    if (result == null) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('用户取消了保存')),
      );
      return;
    }

    try {
      final String content = _contentController.text;
      final Uint8List bytes = Uint8List.fromList(content.codeUnits);

      final XFile file = XFile.fromData(
        bytes,
        mimeType: 'text/plain',
        name: fileName,
      );

      await file.saveTo(result.path);

      setState(() => _lastSavedPath = result.path);

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('文件已保存到: ${result.path}')),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('保存失败: $e')),
      );
    }
  }

  Future<void> _saveJsonFile() async {
    final Map<String, dynamic> data = {
      'app': 'Flutter Demo',
      'version': '1.0.0',
      'createdAt': DateTime.now().toIso8601String(),
      'settings': {
        'theme': 'dark',
        'language': 'zh-CN',
      },
    };

    final FileSaveLocation? result = await getSaveLocation(
      suggestedName: 'config.json',
      acceptedTypeGroups: const [
        XTypeGroup(
          label: 'JSON Files',
          extensions: ['json'],
          mimeTypes: ['application/json'],
        ),
      ],
    );

    if (result != null) {
      final String jsonString = const JsonEncoder.withIndent('  ').convert(data);
      final file = File(result.path);
      await file.writeAsString(jsonString);

      setState(() => _lastSavedPath = result.path);

      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('JSON 文件已保存')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('文件保存')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            TextField(
              controller: _fileNameController,
              decoration: const InputDecoration(
                labelText: '文件名（不含扩展名）',
                prefixIcon: Icon(Icons.label),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _contentController,
              maxLines: 8,
              decoration: const InputDecoration(
                labelText: '文件内容',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
            ),
            const SizedBox(height: 20),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _saveTextFile,
                    icon: const Icon(Icons.save),
                    label: const Text('保存为文本文件'),
                  ),
                ),
                const SizedBox(width: 16),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _saveJsonFile,
                    icon: const Icon(Icons.data_object),
                    label: const Text('保存为 JSON'),
                  ),
                ),
              ],
            ),
            if (_lastSavedPath != null) ...[
              const SizedBox(height: 20),
              Card(
                color: Colors.green.shade50,
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Row(
                        children: [
                          Icon(Icons.check_circle, color: Colors.green),
                          SizedBox(width: 8),
                          Text(
                            '上次保存成功',
                            style: TextStyle(
                              fontWeight: FontWeight.bold,
                              color: Colors.green,
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 8),
                      Text(
                        '路径: $_lastSavedPath',
                        style: TextStyle(
                          fontSize: 12,
                          color: Colors.grey.shade700,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    _contentController.dispose();
    _fileNameController.dispose();
    super.dispose();
  }
}
```

### 5.4 目录选择与批量处理

```dart
class DirectorySelector extends StatefulWidget {
  @override
  DirectorySelectorState createState() => DirectorySelectorState();
}

class DirectorySelectorState extends State<DirectorySelector> {
  String? _selectedDirectory;
  List<FileSystemEntity> _files = [];
  bool _isScanning = false;

  Future<void> _pickDirectory() async {
    final String? directoryPath = await getDirectoryPath(
      confirmButtonText: '选择此文件夹',
    );

    if (directoryPath != null) {
      setState(() {
        _selectedDirectory = directoryPath;
        _isScanning = true;
      });

      await _scanDirectory(directoryPath);

      setState(() => _isScanning = false);
    }
  }

  Future<void> _scanDirectory(String path) async {
    final directory = Directory(path);

    try {
      final entities = await directory
          .list(recursive: false, followLinks: false)
          .toList();

      setState(() {
        _files = entities.where((e) => e is File).toList();
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('扫描目录失败: $e')),
      );
    }
  }

  Future<void> _processAllFiles() async {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => const AlertDialog(
        content: Row(
          children: [
            CircularProgressIndicator(),
            SizedBox(width: 20),
            Text('正在处理文件...'),
          ],
        ),
      ),
    );

    // 模拟处理
    await Future.delayed(const Duration(seconds: 2));

    if (mounted) {
      Navigator.pop(context);

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('已处理 ${_files.length} 个文件')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('目录选择')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: ElevatedButton.icon(
              onPressed: _isScanning ? null : _pickDirectory,
              icon: _isScanning
                  ? const SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(strokeWidth: 2),
                    )
                  : const Icon(Icons.folder_open),
              label: Text(_isScanning ? '扫描中...' : '选择目录'),
            ),
          ),
          if (_selectedDirectory != null)
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(12),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        '已选择目录:',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      const SizedBox(height: 4),
                      Text(
                        _selectedDirectory!,
                        style: TextStyle(
                          color: Colors.grey.shade700,
                          fontSize: 12,
                        ),
                      ),
                      const SizedBox(height: 8),
                      Text(
                        '包含 ${_files.length} 个文件',
                        style: TextStyle(color: Colors.grey.shade600),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          Expanded(
            child: _files.isEmpty
                ? const Center(
                    child: Text(
                      '选择一个目录查看文件',
                      style: TextStyle(color: Colors.grey),
                    ),
                  )
                : ListView.builder(
                    itemCount: _files.length,
                    itemBuilder: (context, index) {
                      final file = _files[index] as File;
                      final fileName = file.path.split('/').last;

                      return ListTile(
                        leading: const Icon(Icons.insert_drive_file),
                        title: Text(
                          fileName,
                          overflow: TextOverflow.ellipsis,
                        ),
                        onTap: () {
                          // 查看文件详情
                        },
                      );
                    },
                  ),
          ),
          if (_files.isNotEmpty)
            Padding(
              padding: const EdgeInsets.all(16),
              child: ElevatedButton(
                onPressed: _processAllFiles,
                child: const Text('批量处理所有文件'),
              ),
            ),
        ],
      ),
    );
  }
}
```

Flutter 框架小知识

**桌面端的特殊处理**

桌面端（Windows、macOS、Linux）使用原生文件对话框，需要注意：

```dart
// Windows 和 Linux 支持锁定父窗口
final XFile? file = await openFile(
  // 这会阻止用户与主窗口交互直到对话框关闭
  // 在 macOS 上此参数被忽略
);

// macOS 需要配置沙盒权限
// 在 .entitlements 文件中添加:
// com.apple.security.files.user-selected.read-write

// 桌面端支持设置初始目录
final XFile? file = await openFile(
  initialDirectory: '/Users/username/Documents',
);
```

---

## 附录

### 附录A：函数速查表

| 函数                 | 返回值                      | 用途         | Web 支持 |
| -------------------- | --------------------------- | ------------ | -------- |
| `openFile()`         | `Future<XFile?>`            | 选择单个文件 | ✅       |
| `openFiles()`        | `Future<List<XFile>>`       | 选择多个文件 | ✅       |
| `getSaveLocation()`  | `Future<FileSaveLocation?>` | 获取保存位置 | ❌       |
| `getDirectoryPath()` | `Future<String?>`           | 选择目录     | ❌       |

### 附录B：XTypeGroup 属性速查表

| 属性                     | 类型            | Android | iOS | Linux | macOS | Web | Windows |
| ------------------------ | --------------- | ------- | --- | ----- | ----- | --- | ------- |
| `extensions`             | `List<String>?` | ✅      | ❌  | ✅    | ✅    | ✅  | ✅      |
| `mimeTypes`              | `List<String>?` | ✅      | ❌  | ✅    | ✅\*  | ✅  | ❌      |
| `uniformTypeIdentifiers` | `List<String>?` | ❌      | ✅  | ❌    | ✅    | ❌  | ❌      |
| `webWildCards`           | `List<String>?` | ❌      | ❌  | ❌    | ❌    | ✅  | ❌      |

\*macOS 11 (Big Sur) 以下版本不支持

### 附录C：XFile 方法速查表

| 方法             | 返回值                 | 用途         |
| ---------------- | ---------------------- | ------------ |
| `readAsBytes()`  | `Future<Uint8List>`    | 读取为字节   |
| `readAsString()` | `Future<String>`       | 读取为字符串 |
| `readAsLines()`  | `Future<List<String>>` | 按行读取     |
| `openRead()`     | `Stream<List<int>>`    | 流式读取     |
| `length()`       | `Future<int>`          | 获取文件大小 |
| `lastModified()` | `Future<DateTime>`     | 获取修改时间 |
| `saveTo()`       | `Future<void>`         | 保存到路径   |

### 附录D：常用 XTypeGroup 配置

```dart
// 图片文件
const XTypeGroup images = XTypeGroup(
  label: 'Images',
  extensions: ['jpg', 'jpeg', 'png', 'gif', 'webp', 'bmp', 'svg'],
  mimeTypes: ['image/*'],
  uniformTypeIdentifiers: ['public.image'],
  webWildCards: ['image/*'],
);

// 文档文件
const XTypeGroup documents = XTypeGroup(
  label: 'Documents',
  extensions: ['pdf', 'doc', 'docx', 'txt', 'rtf', 'odt'],
  mimeTypes: [
    'application/pdf',
    'application/msword',
    'text/plain',
  ],
  uniformTypeIdentifiers: [
    'com.adobe.pdf',
    'public.plain-text',
  ],
);

// 视频文件
const XTypeGroup videos = XTypeGroup(
  label: 'Videos',
  extensions: ['mp4', 'mov', 'avi', 'mkv', 'webm', 'flv'],
  mimeTypes: ['video/*'],
  uniformTypeIdentifiers: ['public.movie'],
  webWildCards: ['video/*'],
);

// 音频文件
const XTypeGroup audio = XTypeGroup(
  label: 'Audio',
  extensions: ['mp3', 'wav', 'aac', 'flac', 'ogg', 'm4a'],
  mimeTypes: ['audio/*'],
  uniformTypeIdentifiers: ['public.audio'],
  webWildCards: ['audio/*'],
);

// 代码文件
const XTypeGroup code = XTypeGroup(
  label: 'Code Files',
  extensions: ['dart', 'js', 'ts', 'html', 'css', 'json', 'xml', 'yaml'],
  mimeTypes: [
    'text/plain',
    'application/json',
    'application/javascript',
  ],
);

// 压缩文件
const XTypeGroup archives = XTypeGroup(
  label: 'Archives',
  extensions: ['zip', 'rar', '7z', 'tar', 'gz'],
  mimeTypes: [
    'application/zip',
    'application/x-rar-compressed',
  ],
);
```

### 附录E：常见问题与解决方案

#### Q1: macOS 文件对话框不显示

**问题**：在 macOS 上调用 `openFile()` 没有任何反应。

**解决方案**：

```xml
<!-- 确保 entitlements 文件配置正确 -->
<!-- macos/Runner/DebugProfile.entitlements -->
<!-- macos/Runner/Release.entitlements -->

<dict>
  <key>com.apple.security.files.user-selected.read-only</key>
  <true/>

  <!-- 如果需要写入 -->
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
```

#### Q2: Web 平台无法获取文件路径

**问题**：Web 平台 `XFile.path` 返回空或无效路径。

**解决方案**：

```dart
// Web 平台使用 bytes 而不是 path
final XFile? file = await openFile();

if (file != null) {
  // 不要依赖 path
  final Uint8List bytes = await file.readAsBytes();

  // 使用 bytes 进行后续操作
  await uploadFile(bytes);
}
```

#### Q3: 类型过滤在某些平台不生效

**问题**：设置了 `XTypeGroup` 但所有文件仍然显示。

**解决方案**：

```dart
// 确保为所有目标平台配置适当的类型标识符
const XTypeGroup group = XTypeGroup(
  label: 'Images',
  extensions: ['jpg', 'png'],        // Android, Linux, Windows, Web, macOS
  mimeTypes: ['image/jpeg', 'image/png'],  // Android, Linux, macOS 11+, Web
  uniformTypeIdentifiers: ['public.image'], // iOS, macOS
);

// 如果某个平台不支持过滤，可以在选择后手动验证
final XFile? file = await openFile(acceptedTypeGroups: [group]);

if (file != null) {
  final ext = file.name.split('.').last.toLowerCase();
  if (!['jpg', 'jpeg', 'png'].contains(ext)) {
    // 显示错误提示
    return;
  }
}
```

#### Q4: iOS 无法选择目录

**问题**：`getDirectoryPath()` 在 iOS 上返回 null。

**解决方案**：

```dart
// iOS 不支持目录选择
// 使用条件编译或运行时检查

if (Platform.isIOS) {
  // iOS 使用替代方案
  // 例如使用 file_picker 的 getDirectoryPath()
} else {
  final String? path = await getDirectoryPath();
}
```

### 附录F：参考资源

| 资源             | 链接                                                                               |
| ---------------- | ---------------------------------------------------------------------------------- |
| Pub 包页面       | https://pub.dev/packages/file_selector                                             |
| GitHub 仓库      | https://github.com/flutter/packages/tree/main/packages/file_selector/file_selector |
| API 参考         | https://pub.dev/documentation/file_selector/latest/                                |
| Flutter 官方文档 | https://docs.flutter.dev/cookbook/plugins/picture-using-camera                     |
| XFile 文档       | https://pub.dev/documentation/cross_file/latest/                                   |

### 附录G：相关插件推荐

| 插件                 | 用途                 |
| -------------------- | -------------------- |
| `file_picker`        | 功能更丰富的文件选择 |
| `cross_file`         | XFile 跨平台文件抽象 |
| `path_provider`      | 获取系统路径         |
| `permission_handler` | 权限管理             |
| `open_filex`         | 打开文件             |
| `share_plus`         | 分享文件             |
| `mime`               | MIME 类型检测        |

---

**文档版本**: 1.0  
**适用版本**: file_selector ^1.1.0  
**最后更新**: 2025年
