# path_provider 2.1.5 详解与实战

## 内容简介

`path_provider` 是 Flutter 官方提供的跨平台插件，用于获取设备上常用的文件系统路径。它抽象了不同平台（iOS、Android、macOS、Windows、Linux）的文件系统差异，为开发者提供统一的 API 来访问临时目录、应用文档目录、下载目录等常用位置。

全书共分为5章：

- **第1章 入门与基础**：详细介绍 path_provider 的安装配置、核心概念
- **第2章 核心 API 详解**：深入讲解各个目录获取方法及其平台差异
- **第3章 平台特定目录**：介绍各平台特有的目录获取方法
- **第4章 最佳实践**：缓存策略、错误处理、性能优化
- **第5章 实战应用**：文件管理器、日志系统、缓存管理等实际案例

---

## 第1章 入门与基础

### 1.1 什么是 path_provider

`path_provider` 是 Flutter 团队维护的官方插件，用于解决跨平台文件路径获取的问题。在移动和桌面应用开发中，我们经常需要：

1. **存储用户数据**：保存用户设置、文档、图片等
2. **缓存数据**：临时存储网络请求结果、图片缓存等
3. **下载文件**：保存用户下载的内容
4. **日志记录**：存储应用运行日志

不同平台的文件系统结构差异很大：

| 平台    | 应用私有目录                     | 临时目录  | 公共存储                   |
| ------- | -------------------------------- | --------- | -------------------------- |
| iOS     | `Documents/`                     | `tmp/`    | 受限                       |
| Android | `/data/data/<package>/`          | `cache/`  | `/sdcard/`                 |
| macOS   | `~/Library/Application Support/` | `tmp/`    | `~/Downloads/`             |
| Windows | `%APPDATA%/`                     | `%TEMP%/` | `%USERPROFILE%/Downloads/` |
| Linux   | `~/.local/share/`                | `/tmp/`   | `~/Downloads/`             |

`path_provider` 通过统一的 API 屏蔽了这些差异。

**Flutter 框架小知识**

**path_provider 与 dart:io 的关系**

`path_provider` 只负责**获取路径**，不负责文件操作。获取到路径后，需要使用 `dart:io` 库中的 `File` 和 `Directory` 类进行实际的文件读写：

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void example() async {
  // 1. path_provider 获取路径
  final directory = await getApplicationDocumentsDirectory();
  final path = directory.path;

  // 2. dart:io 进行文件操作
  final file = File('$path/my_file.txt');
  await file.writeAsString('Hello, World!');
}
```

### 1.2 环境配置与安装

#### 1.2.1 添加依赖

在 `pubspec.yaml` 文件中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  path_provider: ^2.1.5
```

#### 1.2.2 获取依赖

运行以下命令安装依赖：

```bash
flutter pub get
```

或者使用快捷命令：

```bash
flutter pub add path_provider
```

#### 1.2.3 导入库

```dart
import 'package:path_provider/path_provider.dart';
```

### 1.3 支持的平台

path_provider 2.1.5 支持以下平台：

| 平台    | 最低版本    | 支持程度    |
| ------- | ----------- | ----------- |
| Android | API 16+     | ✅ 完整支持 |
| iOS     | 12.0+       | ✅ 完整支持 |
| macOS   | 10.14+      | ✅ 完整支持 |
| Windows | Windows 10+ | ✅ 完整支持 |
| Linux   | 任意版本    | ✅ 完整支持 |
| Web     | -           | ❌ 不支持   |

**Dart Tips 语法小贴士**

**为什么 Web 不支持 path_provider？**

Web 应用运行在浏览器沙箱中，无法直接访问设备的文件系统。浏览器提供了 IndexedDB 和 LocalStorage 等替代方案。如果需要在 Web 上存储数据，可以使用：

- `shared_preferences`：存储简单的键值对
- `hive`：高性能的本地数据库
- `idb_shim`：IndexedDB 的 Dart 封装

### 1.4 核心概念

#### 1.4.1 目录类型

path_provider 提供了多种目录类型，适用于不同的使用场景：

```
┌─────────────────────────────────────────────────────────────────┐
│                        目录类型分类                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📂 临时目录 (Temporary)                                         │
│     └── 用途：缓存数据、临时文件                                  │
│     └── 生命周期：系统可随时清理                                  │
│     └── API：getTemporaryDirectory()                             │
│                                                                 │
│  📁 应用文档目录 (Application Documents)                         │
│     └── 用途：用户生成的持久化数据                                │
│     └── 生命周期：随应用存在，用户可见                            │
│     └── API：getApplicationDocumentsDirectory()                  │
│                                                                 │
│  📱 应用支持目录 (Application Support)                           │
│     └── 用途：应用运行所需的内部文件                              │
│     └── 生命周期：随应用存在，用户不可见                          │
│     └── API：getApplicationSupportDirectory()                    │
│                                                                 │
│  📚 应用库目录 (Application Library)                             │
│     └── 用途：缓存的可再生数据（iOS/macOS 特有）                  │
│     └── API：getApplicationLibraryDirectory()                    │
│                                                                 │
│  🌐 外部存储目录 (External Storage)                              │
│     └── 用途：可被其他应用访问的公共文件                          │
│     └── 生命周期：用户手动管理                                    │
│     └── API：getExternalStorageDirectory() (Android 特有)        │
│                                                                 │
│  📥 下载目录 (Downloads)                                         │
│     └── 用途：用户下载的文件                                      │
│     └── API：getDownloadsDirectory()                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 第2章 核心 API 详解

### 2.1 getTemporaryDirectory

获取应用的临时目录，用于存储临时文件和缓存数据。

#### 2.1.1 基本用法

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void temporaryDirectoryExample() async {
  // 获取临时目录
  final Directory tempDir = await getTemporaryDirectory();

  print('临时目录路径: ${tempDir.path}');
  // Android: /data/user/0/<package>/cache
  // iOS: /var/mobile/Containers/Data/Application/<UUID>/tmp
  // macOS: /var/folders/.../T/
  // Windows: C:\Users\<username>\AppData\Local\Temp\
  // Linux: /tmp/
}
```

#### 2.1.2 平台路径对照

| 平台    | 实际路径示例                                         |
| ------- | ---------------------------------------------------- |
| Android | `/data/user/0/com.example.app/cache`                 |
| iOS     | `/var/mobile/Containers/Data/Application/ABC123/tmp` |
| macOS   | `/var/folders/xy/1234567890/T/`                      |
| Windows | `C:\Users\Username\AppData\Local\Temp`               |
| Linux   | `/tmp/`                                              |

#### 2.1.3 使用场景

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class TempFileManager {
  // 创建临时文件
  static Future<File> createTempFile(String prefix) async {
    final tempDir = await getTemporaryDirectory();
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final fileName = '${prefix}_$timestamp.tmp';
    return File('${tempDir.path}/$fileName');
  }

  // 清理所有临时文件
  static Future<void> clearTempFiles() async {
    final tempDir = await getTemporaryDirectory();
    final files = await tempDir.list().toList();

    for (final file in files) {
      if (file is File && file.path.endsWith('.tmp')) {
        await file.delete();
      }
    }
  }

  // 获取临时目录大小
  static Future<int> getTempSize() async {
    final tempDir = await getTemporaryDirectory();
    var totalSize = 0;

    await for (final entity in tempDir.list(recursive: true)) {
      if (entity is File) {
        totalSize += await entity.length();
      }
    }

    return totalSize;
  }
}
```

**Flutter 框架小知识**

**临时目录的清理策略**

- **Android**：系统可能在磁盘空间不足时清理缓存目录
- **iOS**：系统可能在应用不运行时清理 tmp 目录
- **macOS/Linux**：系统定期清理 /tmp 目录（通常 3-10 天）
- **Windows**：系统磁盘清理工具可清理 Temp 目录

**最佳实践**：不要将重要数据存储在临时目录，且应用启动时可主动清理过期的临时文件。

### 2.2 getApplicationDocumentsDirectory

获取应用的文档目录，用于存储用户生成的持久化数据。

#### 2.2.1 基本用法

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void documentsDirectoryExample() async {
  // 获取应用文档目录
  final Directory appDocDir = await getApplicationDocumentsDirectory();

  print('文档目录路径: ${appDocDir.path}');
  // Android: /data/user/0/<package>/app_flutter
  // iOS: /var/mobile/Containers/Data/Application/<UUID>/Documents
  // macOS: /Users/<username>/Documents
  // Windows: C:\Users\<username>\Documents
  // Linux: /home/<username>/Documents
}
```

#### 2.2.2 平台路径对照

| 平台    | 实际路径示例                                               | 备份行为      |
| ------- | ---------------------------------------------------------- | ------------- |
| Android | `/data/user/0/com.example.app/app_flutter`                 | 不备份        |
| iOS     | `/var/mobile/Containers/Data/Application/ABC123/Documents` | iCloud 备份   |
| macOS   | `~/Documents/`                                             | iCloud 备份   |
| Windows | `C:\Users\Username\Documents`                              | OneDrive 同步 |
| Linux   | `~/Documents/`                                             | 无            |

#### 2.2.3 使用场景

```dart
import 'dart:io';
import 'dart:convert';
import 'package:path_provider/path_provider.dart';

class UserDataManager {
  // 保存用户设置
  static Future<void> saveSettings(Map<String, dynamic> settings) async {
    final appDocDir = await getApplicationDocumentsDirectory();
    final file = File('${appDocDir.path}/settings.json');
    await file.writeAsString(jsonEncode(settings));
  }

  // 读取用户设置
  static Future<Map<String, dynamic>?> loadSettings() async {
    try {
      final appDocDir = await getApplicationDocumentsDirectory();
      final file = File('${appDocDir.path}/settings.json');

      if (await file.exists()) {
        final content = await file.readAsString();
        return jsonDecode(content) as Map<String, dynamic>;
      }
    } catch (e) {
      print('读取设置失败: $e');
    }
    return null;
  }

  // 保存用户文档
  static Future<File> saveDocument(String fileName, String content) async {
    final appDocDir = await getApplicationDocumentsDirectory();
    final documentsDir = Directory('${appDocDir.path}/documents');

    if (!await documentsDir.exists()) {
      await documentsDir.create(recursive: true);
    }

    final file = File('${documentsDir.path}/$fileName');
    return await file.writeAsString(content);
  }

  // 列出所有用户文档
  static Future<List<String>> listDocuments() async {
    final appDocDir = await getApplicationDocumentsDirectory();
    final documentsDir = Directory('${appDocDir.path}/documents');

    if (!await documentsDir.exists()) {
      return [];
    }

    final files = await documentsDir
        .list()
        .where((entity) => entity is File)
        .map((entity) => entity.path.split('/').last)
        .toList();

    return files;
  }
}
```

### 2.3 getApplicationSupportDirectory

获取应用支持目录，用于存储应用运行所需的内部文件。

#### 2.3.1 基本用法

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void supportDirectoryExample() async {
  // 获取应用支持目录
  final Directory appSupportDir = await getApplicationSupportDirectory();

  print('支持目录路径: ${appSupportDir.path}');
  // Android: /data/user/0/<package>/files
  // iOS: /var/mobile/Containers/Data/Application/<UUID>/Library/Application Support
  // macOS: ~/Library/Application Support/<bundle_id>
  // Windows: C:\Users\<username>\AppData\Roaming\<company>\<app>
  // Linux: ~/.local/share/<app>
}
```

#### 2.3.2 平台路径对照

| 平台    | 实际路径示例                                    | 用户可见性 |
| ------- | ----------------------------------------------- | ---------- |
| Android | `/data/user/0/com.example.app/files`            | 不可见     |
| iOS     | `/var/mobile/.../Library/Application Support`   | 不可见     |
| macOS   | `~/Library/Application Support/com.example.app` | 不可见     |
| Windows | `%APPDATA%\CompanyName\AppName`                 | 不可见     |
| Linux   | `~/.local/share/appname`                        | 不可见     |

#### 2.3.3 使用场景

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class AppSupportManager {
  // 保存应用数据库
  static Future<String> getDatabasePath(String dbName) async {
    final appSupportDir = await getApplicationSupportDirectory();
    return '${appSupportDir.path}/$dbName.db';
  }

  // 保存应用缓存数据
  static Future<void> saveCacheData(String key, List<int> data) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final cacheDir = Directory('${appSupportDir.path}/cache');

    if (!await cacheDir.exists()) {
      await cacheDir.create(recursive: true);
    }

    final file = File('${cacheDir.path}/$key.cache');
    await file.writeAsBytes(data);
  }

  // 读取应用缓存数据
  static Future<List<int>?> loadCacheData(String key) async {
    try {
      final appSupportDir = await getApplicationSupportDirectory();
      final file = File('${appSupportDir.path}/cache/$key.cache');

      if (await file.exists()) {
        return await file.readAsBytes();
      }
    } catch (e) {
      print('读取缓存失败: $e');
    }
    return null;
  }

  // 保存应用状态
  static Future<void> saveAppState(Map<String, dynamic> state) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final file = File('${appSupportDir.path}/app_state.json');
    await file.writeAsString(jsonEncode(state));
  }
}
```

### 2.4 getDownloadsDirectory

获取下载目录，用于存储用户下载的文件。

#### 2.4.1 基本用法

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void downloadsDirectoryExample() async {
  // 获取下载目录
  final Directory? downloadsDir = await getDownloadsDirectory();

  if (downloadsDir != null) {
    print('下载目录路径: ${downloadsDir.path}');
    // Android: /storage/emulated/0/Download
    // iOS: /var/mobile/Containers/Data/Application/<UUID>/Downloads
    // macOS: ~/Downloads
    // Windows: C:\Users\<username>\Downloads
    // Linux: ~/Downloads
  }
}
```

#### 2.4.2 平台支持情况

| 平台    | 支持情况 | 说明                            |
| ------- | -------- | ------------------------------- |
| Android | ✅ 支持  | 返回外部存储的 Download 目录    |
| iOS     | ✅ 支持  | 返回应用沙盒内的 Downloads 目录 |
| macOS   | ✅ 支持  | 返回用户 Downloads 目录         |
| Windows | ✅ 支持  | 返回用户 Downloads 目录         |
| Linux   | ✅ 支持  | 返回用户 Downloads 目录         |

#### 2.4.3 使用场景

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:http/http.dart' as http;

class DownloadManager {
  // 下载文件到下载目录
  static Future<File> downloadFile(
    String url,
    String fileName, {
    void Function(int received, int total)? onProgress,
  }) async {
    final downloadsDir = await getDownloadsDirectory();
    if (downloadsDir == null) {
      throw Exception('无法获取下载目录');
    }

    final file = File('${downloadsDir.path}/$fileName');

    final response = await http.Client().send(http.Request('GET', Uri.parse(url)));
    final total = response.contentLength ?? 0;
    var received = 0;

    final sink = file.openWrite();

    await for (final chunk in response.stream) {
      sink.add(chunk);
      received += chunk.length;
      onProgress?.call(received, total);
    }

    await sink.close();
    return file;
  }

  // 列出下载的文件
  static Future<List<File>> listDownloads() async {
    final downloadsDir = await getDownloadsDirectory();
    if (downloadsDir == null) return [];

    final files = await downloadsDir
        .list()
        .where((entity) => entity is File)
        .cast<File>()
        .toList();

    return files;
  }
}
```

### 2.5 getApplicationCacheDirectory

获取应用缓存目录，用于存储应用缓存数据。

#### 2.5.1 基本用法

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void cacheDirectoryExample() async {
  // 获取应用缓存目录
  final Directory? cacheDir = await getApplicationCacheDirectory();

  if (cacheDir != null) {
    print('缓存目录路径: ${cacheDir.path}');
    // Android: /data/user/0/<package>/cache
    // iOS: /var/mobile/Containers/Data/Application/<UUID>/Library/Caches
    // macOS: ~/Library/Caches/<bundle_id>
    // Windows: C:\Users\<username>\AppData\Local\<company>\<app>\cache
    // Linux: ~/.cache/<app>
  }
}
```

#### 2.5.2 与 getTemporaryDirectory 的区别

| 特性     | getTemporaryDirectory | getApplicationCacheDirectory |
| -------- | --------------------- | ---------------------------- |
| 用途     | 系统临时文件          | 应用缓存数据                 |
| 生命周期 | 系统随时清理          | 应用管理清理                 |
| 备份     | 不备份                | 不备份                       |
| 典型用途 | 临时下载、解压        | 图片缓存、网络响应缓存       |

---

## 第3章 平台特定目录

### 3.1 Android 特有 API

#### 3.1.1 getExternalStorageDirectory

获取 Android 外部存储目录（需要存储权限）。

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void externalStorageExample() async {
  // ⚠️ 注意：Android 10+ 需要存储权限
  final Directory? externalDir = await getExternalStorageDirectory();

  if (externalDir != null) {
    print('外部存储路径: ${externalDir.path}');
    // /storage/emulated/0/Android/data/<package>/files
  }
}
```

#### 3.1.2 getExternalCacheDirectories

获取外部缓存目录列表。

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void externalCacheExample() async {
  final List<Directory>? cacheDirs = await getExternalCacheDirectories();

  if (cacheDirs != null) {
    for (final dir in cacheDirs) {
      print('外部缓存路径: ${dir.path}');
    }
  }
}
```

#### 3.1.3 getExternalStorageDirectories

获取特定类型的外部存储目录。

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void externalStorageByTypeExample() async {
  // 获取图片存储目录
  final List<Directory>? pictureDirs = await getExternalStorageDirectories(
    type: StorageDirectory.pictures,
  );

  if (pictureDirs != null) {
    for (final dir in pictureDirs) {
      print('图片目录: ${dir.path}');
    }
  }

  // 获取音乐存储目录
  final List<Directory>? musicDirs = await getExternalStorageDirectories(
    type: StorageDirectory.music,
  );

  // 获取电影存储目录
  final List<Directory>? movieDirs = await getExternalStorageDirectories(
    type: StorageDirectory.movies,
  );
}
```

**StorageDirectory 枚举：**

| 值                               | 说明         |
| -------------------------------- | ------------ |
| `StorageDirectory.music`         | 音乐目录     |
| `StorageDirectory.podcasts`      | 播客目录     |
| `StorageDirectory.ringtones`     | 铃声目录     |
| `StorageDirectory.alarms`        | 闹钟目录     |
| `StorageDirectory.notifications` | 通知目录     |
| `StorageDirectory.pictures`      | 图片目录     |
| `StorageDirectory.movies`        | 电影目录     |
| `StorageDirectory.downloads`     | 下载目录     |
| `StorageDirectory.dcim`          | 相机照片目录 |
| `StorageDirectory.documents`     | 文档目录     |

### 3.2 iOS/macOS 特有 API

#### 3.2.1 getApplicationLibraryDirectory

获取应用库目录（iOS/macOS 特有）。

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

void libraryDirectoryExample() async {
  // 仅 iOS 和 macOS 支持
  final Directory? libraryDir = await getApplicationLibraryDirectory();

  if (libraryDir != null) {
    print('库目录路径: ${libraryDir.path}');
    // iOS: /var/mobile/.../Library
    // macOS: ~/Library
  }
}
```

#### 3.2.2 iOS 目录结构

```
/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/           ← getApplicationDocumentsDirectory()
│   └── 用户可见的文档
├── Library/
│   ├── Application Support/  ← getApplicationSupportDirectory()
│   │   └── 应用支持文件
│   ├── Caches/          ← getApplicationCacheDirectory()
│   │   └── 缓存数据
│   └── Preferences/     ← shared_preferences 存储位置
├── tmp/                 ← getTemporaryDirectory()
│   └── 临时文件
└── .com.apple.mobile_container_manager.metadata.plist
```

### 3.3 平台支持对照表

| API                                | Android | iOS | macOS | Windows | Linux |
| ---------------------------------- | :-----: | :-: | :---: | :-----: | :---: |
| `getTemporaryDirectory`            |   ✅    | ✅  |  ✅   |   ✅    |  ✅   |
| `getApplicationDocumentsDirectory` |   ✅    | ✅  |  ✅   |   ✅    |  ✅   |
| `getApplicationSupportDirectory`   |   ✅    | ✅  |  ✅   |   ✅    |  ✅   |
| `getApplicationCacheDirectory`     |   ✅    | ✅  |  ✅   |   ✅    |  ✅   |
| `getDownloadsDirectory`            |   ✅    | ✅  |  ✅   |   ✅    |  ✅   |
| `getApplicationLibraryDirectory`   |   ❌    | ✅  |  ✅   |   ❌    |  ❌   |
| `getExternalStorageDirectory`      |   ✅    | ❌  |  ❌   |   ❌    |  ❌   |
| `getExternalCacheDirectories`      |   ✅    | ❌  |  ❌   |   ❌    |  ❌   |
| `getExternalStorageDirectories`    |   ✅    | ❌  |  ❌   |   ❌    |  ❌   |

---

## 第4章 最佳实践

### 4.1 路径缓存策略

由于 path_provider 的方法涉及平台通道通信，频繁调用会有性能开销。建议缓存路径结果：

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class PathCache {
  static Directory? _tempDir;
  static Directory? _appDocDir;
  static Directory? _appSupportDir;
  static Directory? _downloadsDir;

  // 获取临时目录（带缓存）
  static Future<Directory> getTempDirectory() async {
    _tempDir ??= await getTemporaryDirectory();
    return _tempDir!;
  }

  // 获取应用文档目录（带缓存）
  static Future<Directory> getAppDocumentsDirectory() async {
    _appDocDir ??= await getApplicationDocumentsDirectory();
    return _appDocDir!;
  }

  // 获取应用支持目录（带缓存）
  static Future<Directory> getAppSupportDirectory() async {
    _appSupportDir ??= await getApplicationSupportDirectory();
    return _appSupportDir!;
  }

  // 获取下载目录（带缓存）
  static Future<Directory?> getDownloadsDir() async {
    _downloadsDir ??= await getDownloadsDirectory();
    return _downloadsDir;
  }

  // 清除缓存（应用重新初始化时调用）
  static void clearCache() {
    _tempDir = null;
    _appDocDir = null;
    _appSupportDir = null;
    _downloadsDir = null;
  }
}
```

### 4.2 错误处理

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:flutter/foundation.dart';

class SafePathProvider {
  // 安全获取临时目录
  static Future<Directory?> getTempDirectorySafe() async {
    try {
      return await getTemporaryDirectory();
    } catch (e) {
      if (kDebugMode) {
        print('获取临时目录失败: $e');
      }
      return null;
    }
  }

  // 安全获取应用文档目录
  static Future<Directory?> getAppDocumentsDirectorySafe() async {
    try {
      return await getApplicationDocumentsDirectory();
    } catch (e) {
      if (kDebugMode) {
        print('获取应用文档目录失败: $e');
      }
      return null;
    }
  }

  // 安全获取应用支持目录
  static Future<Directory?> getAppSupportDirectorySafe() async {
    try {
      return await getApplicationSupportDirectory();
    } catch (e) {
      if (kDebugMode) {
        print('获取应用支持目录失败: $e');
      }
      return null;
    }
  }

  // 安全获取下载目录
  static Future<Directory?> getDownloadsDirectorySafe() async {
    try {
      return await getDownloadsDirectory();
    } catch (e) {
      if (kDebugMode) {
        print('获取下载目录失败: $e');
      }
      return null;
    }
  }
}
```

### 4.3 目录选择指南

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

/// 目录选择决策树
///
/// 1. 数据是否需要持久化？
///    ├── 是 → 继续
///    └── 否 → 使用 getTemporaryDirectory()
///
/// 2. 数据是否由用户生成？
///    ├── 是 → 使用 getApplicationDocumentsDirectory()
///    └── 否 → 继续
///
/// 3. 数据是否需要用户可见？
///    ├── 是 → 使用 getDownloadsDirectory()
///    └── 否 → 使用 getApplicationSupportDirectory()

class DirectorySelector {
  /// 获取适合存储用户数据的目录
  static Future<Directory> getUserDataDirectory() async {
    return await getApplicationDocumentsDirectory();
  }

  /// 获取适合存储应用内部数据的目录
  static Future<Directory> getAppDataDirectory() async {
    return await getApplicationSupportDirectory();
  }

  /// 获取适合存储缓存的目录
  static Future<Directory> getCacheDirectory() async {
    return await getTemporaryDirectory();
  }

  /// 获取适合存储下载文件的目录
  static Future<Directory?> getDownloadDirectory() async {
    return await getDownloadsDirectory();
  }
}
```

### 4.4 文件路径拼接

```dart
import 'dart:io';
import 'package:path/path.dart' as path;
import 'package:path_provider/path_provider.dart';

class PathUtils {
  /// 安全拼接路径
  static String joinPaths(String base, String relative) {
    return path.join(base, relative);
  }

  /// 获取文件扩展名
  static String getExtension(String filePath) {
    return path.extension(filePath);
  }

  /// 获取文件名（不含扩展名）
  static String getFileNameWithoutExtension(String filePath) {
    return path.basenameWithoutExtension(filePath);
  }

  /// 获取文件名（含扩展名）
  static String getFileName(String filePath) {
    return path.basename(filePath);
  }

  /// 获取目录名
  static String getDirectoryName(String filePath) {
    return path.dirname(filePath);
  }

  /// 生成唯一文件名
  static String generateUniqueFileName(String originalName) {
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final extension = path.extension(originalName);
    final nameWithoutExt = path.basenameWithoutExtension(originalName);
    return '${nameWithoutExt}_$timestamp$extension';
  }
}
```

### 4.5 清理策略

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class CacheCleaner {
  /// 清理临时目录
  static Future<void> clearTempDirectory() async {
    final tempDir = await getTemporaryDirectory();
    await _clearDirectory(tempDir);
  }

  /// 清理应用缓存目录
  static Future<void> clearCacheDirectory() async {
    final cacheDir = await getApplicationCacheDirectory();
    if (cacheDir != null) {
      await _clearDirectory(cacheDir);
    }
  }

  /// 清理指定目录
  static Future<void> _clearDirectory(Directory dir) async {
    if (!await dir.exists()) return;

    await for (final entity in dir.list()) {
      try {
        await entity.delete(recursive: true);
      } catch (e) {
        print('删除失败: ${entity.path}, 错误: $e');
      }
    }
  }

  /// 清理过期缓存（保留最近 N 天的文件）
  static Future<void> clearExpiredCache(int keepDays) async {
    final tempDir = await getTemporaryDirectory();
    final now = DateTime.now();
    final threshold = now.subtract(Duration(days: keepDays));

    await for (final entity in tempDir.list()) {
      if (entity is File) {
        try {
          final stat = await entity.stat();
          if (stat.modified.isBefore(threshold)) {
            await entity.delete();
          }
        } catch (e) {
          print('检查文件失败: ${entity.path}, 错误: $e');
        }
      }
    }
  }

  /// 获取目录大小
  static Future<int> getDirectorySize(Directory dir) async {
    var totalSize = 0;

    if (!await dir.exists()) return 0;

    await for (final entity in dir.list(recursive: true)) {
      if (entity is File) {
        try {
          totalSize += await entity.length();
        } catch (e) {
          // 忽略无法访问的文件
        }
      }
    }

    return totalSize;
  }

  /// 格式化文件大小
  static String formatSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
    if (bytes < 1024 * 1024 * 1024) {
      return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
    }
    return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(2)} GB';
  }
}
```

---

## 第5章 实战应用

### 5.1 文件管理器

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class FileManager {
  late final Directory _baseDir;

  FileManager._(this._baseDir);

  /// 初始化文件管理器
  static Future<FileManager> create({String? subDir}) async {
    final appDocDir = await getApplicationDocumentsDirectory();
    final basePath = subDir != null
        ? path.join(appDocDir.path, subDir)
        : appDocDir.path;

    final baseDir = Directory(basePath);
    if (!await baseDir.exists()) {
      await baseDir.create(recursive: true);
    }

    return FileManager._(baseDir);
  }

  /// 创建目录
  Future<Directory> createDirectory(String dirName) async {
    final dir = Directory(path.join(_baseDir.path, dirName));
    if (!await dir.exists()) {
      await dir.create(recursive: true);
    }
    return dir;
  }

  /// 保存文件
  Future<File> saveFile(String fileName, List<int> data) async {
    final file = File(path.join(_baseDir.path, fileName));
    return await file.writeAsBytes(data);
  }

  /// 保存文本文件
  Future<File> saveTextFile(String fileName, String content) async {
    final file = File(path.join(_baseDir.path, fileName));
    return await file.writeAsString(content);
  }

  /// 读取文件
  Future<List<int>?> readFile(String fileName) async {
    final file = File(path.join(_baseDir.path, fileName));
    if (await file.exists()) {
      return await file.readAsBytes();
    }
    return null;
  }

  /// 读取文本文件
  Future<String?> readTextFile(String fileName) async {
    final file = File(path.join(_baseDir.path, fileName));
    if (await file.exists()) {
      return await file.readAsString();
    }
    return null;
  }

  /// 删除文件
  Future<void> deleteFile(String fileName) async {
    final file = File(path.join(_baseDir.path, fileName));
    if (await file.exists()) {
      await file.delete();
    }
  }

  /// 删除目录
  Future<void> deleteDirectory(String dirName) async {
    final dir = Directory(path.join(_baseDir.path, dirName));
    if (await dir.exists()) {
      await dir.delete(recursive: true);
    }
  }

  /// 列出文件
  Future<List<File>> listFiles({String? extension}) async {
    final files = <File>[];

    await for (final entity in _baseDir.list()) {
      if (entity is File) {
        if (extension == null || entity.path.endsWith('.$extension')) {
          files.add(entity);
        }
      }
    }

    return files;
  }

  /// 列出目录
  Future<List<Directory>> listDirectories() async {
    final dirs = <Directory>[];

    await for (final entity in _baseDir.list()) {
      if (entity is Directory) {
        dirs.add(entity);
      }
    }

    return dirs;
  }

  /// 检查文件是否存在
  Future<bool> fileExists(String fileName) async {
    final file = File(path.join(_baseDir.path, fileName));
    return await file.exists();
  }

  /// 获取文件大小
  Future<int?> getFileSize(String fileName) async {
    final file = File(path.join(_baseDir.path, fileName));
    if (await file.exists()) {
      return await file.length();
    }
    return null;
  }

  /// 重命名文件
  Future<File> renameFile(String oldName, String newName) async {
    final oldFile = File(path.join(_baseDir.path, oldName));
    final newFile = File(path.join(_baseDir.path, newName));
    return await oldFile.rename(newFile.path);
  }

  /// 复制文件
  Future<File> copyFile(String sourceName, String destName) async {
    final sourceFile = File(path.join(_baseDir.path, sourceName));
    final destFile = File(path.join(_baseDir.path, destName));
    return await sourceFile.copy(destFile.path);
  }

  /// 获取完整路径
  String getFullPath(String relativePath) {
    return path.join(_baseDir.path, relativePath);
  }
}
```

### 5.2 日志系统

```dart
import 'dart:io';
import 'dart:convert';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class FileLogger {
  late final File _logFile;
  late final IOSink _sink;
  final String _logLevel;

  FileLogger._(this._logFile, this._logLevel) {
    _sink = _logFile.openWrite(mode: FileMode.append);
  }

  /// 初始化日志系统
  static Future<FileLogger> create({
    String logLevel = 'DEBUG',
    String? fileName,
  }) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final logsDir = Directory(path.join(appSupportDir.path, 'logs'));

    if (!await logsDir.exists()) {
      await logsDir.create(recursive: true);
    }

    final logFileName = fileName ?? 'app_${_formatDate(DateTime.now())}.log';
    final logFile = File(path.join(logsDir.path, logFileName));

    return FileLogger._(logFile, logLevel);
  }

  /// 记录调试日志
  void debug(String message) => _log('DEBUG', message);

  /// 记录信息日志
  void info(String message) => _log('INFO', message);

  /// 记录警告日志
  void warning(String message) => _log('WARN', message);

  /// 记录错误日志
  void error(String message, [Object? error, StackTrace? stackTrace]) {
    var fullMessage = message;
    if (error != null) {
      fullMessage += ' | Error: $error';
    }
    if (stackTrace != null) {
      fullMessage += ' | StackTrace: $stackTrace';
    }
    _log('ERROR', fullMessage);
  }

  void _log(String level, String message) {
    final timestamp = DateTime.now().toIso8601String();
    final logLine = '[$timestamp] [$level] $message\n';
    _sink.write(logLine);
  }

  /// 刷新日志到磁盘
  Future<void> flush() async {
    await _sink.flush();
  }

  /// 关闭日志系统
  Future<void> close() async {
    await _sink.close();
  }

  /// 获取日志文件路径
  String get logFilePath => _logFile.path;

  /// 获取日志文件大小
  Future<int> get logFileSize async {
    if (await _logFile.exists()) {
      return await _logFile.length();
    }
    return 0;
  }

  static String _formatDate(DateTime date) {
    return '${date.year}${date.month.toString().padLeft(2, '0')}${date.day.toString().padLeft(2, '0')}';
  }

  /// 清理旧日志文件
  static Future<void> cleanOldLogs(int keepDays) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final logsDir = Directory(path.join(appSupportDir.path, 'logs'));

    if (!await logsDir.exists()) return;

    final now = DateTime.now();
    final threshold = now.subtract(Duration(days: keepDays));

    await for (final entity in logsDir.list()) {
      if (entity is File && entity.path.endsWith('.log')) {
        try {
          final stat = await entity.stat();
          if (stat.modified.isBefore(threshold)) {
            await entity.delete();
          }
        } catch (e) {
          // 忽略删除失败的文件
        }
      }
    }
  }
}
```

### 5.3 配置管理器

```dart
import 'dart:io';
import 'dart:convert';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class ConfigManager {
  late final File _configFile;
  Map<String, dynamic> _config = {};

  ConfigManager._(this._configFile);

  /// 初始化配置管理器
  static Future<ConfigManager> create({String? configFileName}) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final fileName = configFileName ?? 'config.json';
    final configFile = File(path.join(appSupportDir.path, fileName));

    final manager = ConfigManager._(configFile);
    await manager._load();

    return manager;
  }

  /// 加载配置
  Future<void> _load() async {
    if (await _configFile.exists()) {
      try {
        final content = await _configFile.readAsString();
        _config = jsonDecode(content) as Map<String, dynamic>;
      } catch (e) {
        _config = {};
      }
    } else {
      _config = {};
    }
  }

  /// 保存配置
  Future<void> _save() async {
    final content = const JsonEncoder.withIndent('  ').convert(_config);
    await _configFile.writeAsString(content);
  }

  /// 获取配置值
  T? get<T>(String key) {
    final value = _config[key];
    if (value is T) {
      return value;
    }
    return null;
  }

  /// 获取配置值（带默认值）
  T getOrDefault<T>(String key, T defaultValue) {
    return get<T>(key) ?? defaultValue;
  }

  /// 设置配置值
  Future<void> set<T>(String key, T value) async {
    _config[key] = value;
    await _save();
  }

  /// 删除配置项
  Future<void> remove(String key) async {
    _config.remove(key);
    await _save();
  }

  /// 检查配置项是否存在
  bool has(String key) => _config.containsKey(key);

  /// 获取所有配置键
  List<String> get keys => _config.keys.toList();

  /// 清除所有配置
  Future<void> clear() async {
    _config.clear();
    await _save();
  }

  /// 批量设置配置
  Future<void> setAll(Map<String, dynamic> values) async {
    _config.addAll(values);
    await _save();
  }

  /// 导出配置
  Map<String, dynamic> export() => Map.from(_config);

  /// 导入配置
  Future<void> import(Map<String, dynamic> config) async {
    _config = Map.from(config);
    await _save();
  }

  /// 获取配置文件路径
  String get configFilePath => _configFile.path;

  /// 获取配置项数量
  int get count => _config.length;
}
```

### 5.4 图片缓存管理器

```dart
import 'dart:io';
import 'dart:typed_data';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;
import 'package:crypto/crypto.dart';
import 'dart:convert';

class ImageCacheManager {
  late final Directory _cacheDir;
  final int _maxCacheSize;  // 最大缓存大小（字节）
  final int _maxCacheAge;   // 最大缓存时间（天）

  ImageCacheManager._(this._cacheDir, this._maxCacheSize, this._maxCacheAge);

  /// 初始化图片缓存管理器
  static Future<ImageCacheManager> create({
    int maxCacheSizeMB = 100,  // 默认 100MB
    int maxCacheAgeDays = 7,   // 默认 7 天
  }) async {
    final tempDir = await getTemporaryDirectory();
    final cacheDir = Directory(path.join(tempDir.path, 'image_cache'));

    if (!await cacheDir.exists()) {
      await cacheDir.create(recursive: true);
    }

    final manager = ImageCacheManager._(
      cacheDir,
      maxCacheSizeMB * 1024 * 1024,
      maxCacheAgeDays,
    );

    // 启动时清理过期缓存
    await manager._cleanExpiredCache();

    return manager;
  }

  /// 生成缓存键
  String _generateCacheKey(String url) {
    final bytes = utf8.encode(url);
    final digest = md5.convert(bytes);
    return digest.toString();
  }

  /// 获取缓存文件路径
  String _getCacheFilePath(String cacheKey) {
    return path.join(_cacheDir.path, '$cacheKey.jpg');
  }

  /// 缓存图片
  Future<File> cacheImage(String url, Uint8List data) async {
    final cacheKey = _generateCacheKey(url);
    final filePath = _getCacheFilePath(cacheKey);
    final file = File(filePath);

    await file.writeAsBytes(data);

    // 检查缓存大小，必要时清理
    await _checkCacheSize();

    return file;
  }

  /// 获取缓存图片
  Future<File?> getCachedImage(String url) async {
    final cacheKey = _generateCacheKey(url);
    final filePath = _getCacheFilePath(cacheKey);
    final file = File(filePath);

    if (await file.exists()) {
      // 更新访问时间
      final now = DateTime.now();
      await file.setLastAccessed(now);
      return file;
    }

    return null;
  }

  /// 检查缓存是否存在
  Future<bool> isCached(String url) async {
    final cacheKey = _generateCacheKey(url);
    final filePath = _getCacheFilePath(cacheKey);
    final file = File(filePath);
    return await file.exists();
  }

  /// 删除缓存图片
  Future<void> removeCachedImage(String url) async {
    final cacheKey = _generateCacheKey(url);
    final filePath = _getCacheFilePath(cacheKey);
    final file = File(filePath);

    if (await file.exists()) {
      await file.delete();
    }
  }

  /// 清理过期缓存
  Future<void> _cleanExpiredCache() async {
    final now = DateTime.now();
    final threshold = now.subtract(Duration(days: _maxCacheAge));

    await for (final entity in _cacheDir.list()) {
      if (entity is File) {
        try {
          final stat = await entity.stat();
          if (stat.accessed.isBefore(threshold)) {
            await entity.delete();
          }
        } catch (e) {
          // 忽略错误
        }
      }
    }
  }

  /// 检查缓存大小
  Future<void> _checkCacheSize() async {
    var totalSize = 0;
    final files = <File, DateTime>[];

    await for (final entity in _cacheDir.list()) {
      if (entity is File) {
        try {
          final stat = await entity.stat();
          totalSize += stat.size;
          files.add(entity, stat.accessed);
        } catch (e) {
          // 忽略错误
        }
      }
    }

    // 如果缓存超过限制，删除最旧的文件
    if (totalSize > _maxCacheSize) {
      files.sort((a, b) => a.value.compareTo(b.value));

      for (final entry in files) {
        if (totalSize <= _maxCacheSize) break;

        try {
          final file = entry.key;
          final size = await file.length();
          await file.delete();
          totalSize -= size;
        } catch (e) {
          // 忽略错误
        }
      }
    }
  }

  /// 清空所有缓存
  Future<void> clearCache() async {
    await for (final entity in _cacheDir.list()) {
      if (entity is File) {
        try {
          await entity.delete();
        } catch (e) {
          // 忽略错误
        }
      }
    }
  }

  /// 获取缓存大小
  Future<int> getCacheSize() async {
    var totalSize = 0;

    await for (final entity in _cacheDir.list()) {
      if (entity is File) {
        try {
          totalSize += await entity.length();
        } catch (e) {
          // 忽略错误
        }
      }
    }

    return totalSize;
  }

  /// 获取缓存文件数量
  Future<int> getCacheFileCount() async {
    var count = 0;

    await for (final entity in _cacheDir.list()) {
      if (entity is File) count++;
    }

    return count;
  }
}
```

### 5.5 数据库路径管理

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class DatabasePathManager {
  /// 获取 SQLite 数据库路径
  static Future<String> getDatabasePath(String dbName) async {
    final appSupportDir = await getApplicationSupportDirectory();
    final dbDir = Directory(path.join(appSupportDir.path, 'databases'));

    if (!await dbDir.exists()) {
      await dbDir.create(recursive: true);
    }

    return path.join(dbDir.path, '$dbName.db');
  }

  /// 获取 Drift 数据库路径
  static Future<String> getDriftDatabasePath(String dbName) async {
    return await getDatabasePath(dbName);
  }

  /// 获取 Hive 数据库路径
  static Future<String> getHiveDatabasePath() async {
    final appSupportDir = await getApplicationSupportDirectory();
    final hiveDir = Directory(path.join(appSupportDir.path, 'hive'));

    if (!await hiveDir.exists()) {
      await hiveDir.create(recursive: true);
    }

    return hiveDir.path;
  }

  /// 获取 ObjectBox 数据库路径
  static Future<String> getObjectBoxPath() async {
    final appSupportDir = await getApplicationSupportDirectory();
    final objectBoxDir = Directory(path.join(appSupportDir.path, 'objectbox'));

    if (!await objectBoxDir.exists()) {
      await objectBoxDir.create(recursive: true);
    }

    return objectBoxDir.path;
  }

  /// 备份数据库
  static Future<File> backupDatabase(String dbName) async {
    final dbPath = await getDatabasePath(dbName);
    final dbFile = File(dbPath);

    if (!await dbFile.exists()) {
      throw Exception('数据库不存在: $dbName');
    }

    final appDocDir = await getApplicationDocumentsDirectory();
    final backupDir = Directory(path.join(appDocDir.path, 'backups'));

    if (!await backupDir.exists()) {
      await backupDir.create(recursive: true);
    }

    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final backupPath = path.join(backupDir.path, '${dbName}_$timestamp.db');

    return await dbFile.copy(backupPath);
  }

  /// 列出数据库备份
  static Future<List<File>> listDatabaseBackups(String dbName) async {
    final appDocDir = await getApplicationDocumentsDirectory();
    final backupDir = Directory(path.join(appDocDir.path, 'backups'));

    if (!await backupDir.exists()) {
      return [];
    }

    final backups = <File>[];

    await for (final entity in backupDir.list()) {
      if (entity is File &&
          entity.path.contains(dbName) &&
          entity.path.endsWith('.db')) {
        backups.add(entity);
      }
    }

    backups.sort((a, b) => b.path.compareTo(a.path));
    return backups;
  }

  /// 删除数据库
  static Future<void> deleteDatabase(String dbName) async {
    final dbPath = await getDatabasePath(dbName);
    final dbFile = File(dbPath);

    if (await dbFile.exists()) {
      await dbFile.delete();
    }
  }
}
```

---

## 结语

`path_provider` 2.1.5 是 Flutter 开发中不可或缺的插件，通过本书的学习，你应该已经掌握了：

1. **核心概念**：path_provider 的作用、与 dart:io 的关系
2. **核心 API**：临时目录、应用文档目录、应用支持目录、下载目录等
3. **平台特定目录**：Android 外部存储、iOS 应用库目录
4. **最佳实践**：路径缓存、错误处理、目录选择指南、清理策略
5. **实战应用**：文件管理器、日志系统、配置管理器、图片缓存、数据库路径管理

在实际开发中，建议：

- **缓存路径结果**：避免频繁调用 path_provider 方法
- **正确处理错误**：平台通道可能失败，需要 try-catch
- **选择合适的目录**：根据数据类型和生命周期选择正确的目录
- **定期清理缓存**：避免缓存无限增长占用存储空间
- **使用 path 包拼接路径**：正确处理不同平台的路径分隔符

希望本书能帮助你更好地使用 `path_provider` 构建高质量的 Flutter 应用！
