# path 1.9.1 详解与实战

## 内容简介

`path` 是 Dart 官方提供的跨平台路径操作库，用于处理不同操作系统（Windows、POSIX、Web）的路径字符串。它提供了路径拼接、分解、规范化等常用操作，是文件系统编程的基础工具库。path 1.9.1 支持 Dart 3.4+，完全兼容空安全，可在 Dart VM 和浏览器环境中使用。

全书共分为5章：

- **第1章 入门与基础**：详细介绍 path 库的安装配置、核心概念
- **第2章 核心函数详解**：深入讲解路径拼接、分解、判断等核心函数
- **第3章 路径规范化与转换**：介绍路径规范化、相对/绝对路径转换
- **第4章 高级用法**：Context 类、Style 枚举、PathMap/PathSet 等高级功能
- **第5章 实战应用**：文件管理、路径工具类、跨平台路径处理等实际案例

---

## 第1章 入门与基础

### 1.1 什么是 path 库

`path` 是一个基于字符串的路径操作库，提供了跨平台的路径处理功能。与其他语言的路径库不同，path 将路径视为普通字符串而非对象，这使得它更加轻量且易于使用。

**设计哲学**：为什么 path 使用字符串而非对象？

1. **简洁性**：字符串是最自然的表示方式
2. **互操作性**：所有 API 都可以直接接受字符串路径
3. **无类型耦合**：不强制依赖特定路径类型

```dart
import 'package:path/path.dart' as p;

void main() {
  // 简单的路径拼接
  final path = p.join('directory', 'file.txt');
  print(path);  // directory/file.txt (POSIX) 或 directory\file.txt (Windows)
}
```

**Flutter 框架小知识**

**path 库的平台支持**

| 平台                    | 路径风格 | 分隔符     |
| ----------------------- | -------- | ---------- |
| Linux/macOS/iOS/Android | POSIX    | `/`        |
| Windows                 | Windows  | `\` 或 `/` |
| Web (浏览器)            | URL      | `/`        |

path 库会自动检测当前平台并使用相应的路径风格，也可以通过 `Context` 显式指定。

### 1.2 环境配置与安装

#### 1.2.1 添加依赖

在 `pubspec.yaml` 文件中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  path: ^1.9.1
```

#### 1.2.2 获取依赖

```bash
flutter pub get
```

或快捷命令：

```bash
flutter pub add path
```

#### 1.2.3 导入库

```dart
// 推荐方式：使用前缀
import 'package:path/path.dart' as p;

// 或使用特定名称
import 'package:path/path.dart' as path;
```

**Dart Tips 语法小贴士**

**为什么要使用前缀导入？**

使用 `as p` 或 `as path` 前缀导入可以避免命名冲突，并使代码意图更清晰：

```dart
// ✅ 推荐：使用前缀
import 'package:path/path.dart' as p;
final fileName = p.basename(filePath);

// ❌ 不推荐：不使用前缀
import 'package:path/path.dart';
final fileName = basename(filePath);  // 可能与其他库的函数冲突
```

### 1.3 核心概念

#### 1.3.1 路径风格（Style）

path 库支持三种路径风格：

```dart
import 'package:path/path.dart' as p;

void styleExample() {
  // POSIX 风格（Linux/macOS）
  print(p.posix.join('home', 'user', 'file.txt'));
  // 输出: home/user/file.txt

  // Windows 风格
  print(p.windows.join('C:', 'Users', 'file.txt'));
  // 输出: C:\Users\file.txt

  // URL 风格（Web）
  print(p.url.join('https://example.com', 'path', 'file.txt'));
  // 输出: https://example.com/path/file.txt
}
```

#### 1.3.2 绝对路径与相对路径

```dart
import 'package:path/path.dart' as p;

void pathTypeExample() {
  // POSIX 绝对路径
  print(p.isAbsolute('/home/user/file.txt'));  // true
  print(p.isRelative('/home/user/file.txt'));  // false

  // POSIX 相对路径
  print(p.isAbsolute('home/user/file.txt'));   // false
  print(p.isRelative('home/user/file.txt'));   // true

  // Windows 绝对路径
  print(p.windows.isAbsolute(r'C:\Users\file.txt'));  // true
  print(p.windows.isAbsolute(r'\Users\file.txt'));    // true (根相对)

  // Windows 相对路径
  print(p.windows.isAbsolute(r'Users\file.txt'));     // false
}
```

#### 1.3.3 路径分隔符

```dart
import 'package:path/path.dart' as p;

void separatorExample() {
  // 获取当前平台的分隔符
  print('分隔符: "${p.separator}"');
  // Linux/macOS: "/"
  // Windows: "\"
  // Web: "/"
}
```

---

## 第2章 核心函数详解

### 2.1 路径拼接

#### 2.1.1 join 函数

`join` 函数将多个路径部分拼接成一个完整路径，使用当前平台的分隔符。

```dart
import 'package:path/path.dart' as p;

void joinExample() {
  // 基本拼接
  final path1 = p.join('home', 'user', 'documents', 'file.txt');
  print(path1);  // home/user/documents/file.txt

  // 自动处理多余的分隔符
  final path2 = p.join('home/', '/user/', 'documents/', '/file.txt');
  print(path2);  // home/user/documents/file.txt

  // 处理空字符串
  final path3 = p.join('home', '', 'user');
  print(path3);  // home/user

  // 支持最多 16 个参数
  final path4 = p.join('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j');
  print(path4);  // a/b/c/d/e/f/g/h/i/j
}
```

**join 函数签名：**

```dart
String join(
  String part1,
  [String? part2,
  String? part3,
  String? part4,
  String? part5,
  String? part6,
  String? part7,
  String? part8,
  String? part9,
  String? part10,
  String? part11,
  String? part12,
  String? part13,
  String? part14,
  String? part15,
  String? part16]
)
```

#### 2.1.2 joinAll 函数

`joinAll` 函数接受一个可迭代对象，拼接其中的所有路径部分。

```dart
import 'package:path/path.dart' as p;

void joinAllExample() {
  final parts = ['home', 'user', 'documents', 'file.txt'];
  final path = p.joinAll(parts);
  print(path);  // home/user/documents/file.txt

  // 动态生成路径部分
  final dynamicParts = ['var', 'log'];
  dynamicParts.addAll(['app', '2024', 'log.txt']);
  final logPath = p.joinAll(dynamicParts);
  print(logPath);  // var/log/app/2024/log.txt
}
```

#### 2.1.3 absolute 函数

`absolute` 函数将相对路径转换为绝对路径，基于当前工作目录。

```dart
import 'package:path/path.dart' as p;

void absoluteExample() {
  // 假设当前目录是 /home/user
  final absPath = p.absolute('documents', 'file.txt');
  print(absPath);  // /home/user/documents/file.txt

  // 已经是绝对路径则保持不变
  final alreadyAbs = p.absolute('/etc/passwd');
  print(alreadyAbs);  // /etc/passwd

  // Windows 示例
  print(p.windows.absolute('Users', 'file.txt'));
  // 输出: C:\current\dir\Users\file.txt
}
```

### 2.2 路径分解

#### 2.2.1 basename 函数

`basename` 函数获取路径的最后一部分（文件名）。

```dart
import 'package:path/path.dart' as p;

void basenameExample() {
  // 获取文件名
  final fileName = p.basename('/home/user/documents/report.pdf');
  print(fileName);  // report.pdf

  // 目录路径
  final dirName = p.basename('/home/user/documents/');
  print(dirName);  // documents

  // 根目录
  final root = p.basename('/');
  print(root);  // (空字符串)

  // Windows
  print(p.windows.basename(r'C:\Users\file.txt'));  // file.txt
}
```

#### 2.2.2 basenameWithoutExtension 函数

`basenameWithoutExtension` 函数获取不带扩展名的文件名。

```dart
import 'package:path/path.dart' as p;

void basenameWithoutExtensionExample() {
  final name = p.basenameWithoutExtension('/home/user/report.pdf');
  print(name);  // report

  // 多扩展名文件
  final tarName = p.basenameWithoutExtension('archive.tar.gz');
  print(tarName);  // archive.tar

  // 无扩展名
  final noExt = p.basenameWithoutExtension('README');
  print(noExt);  // README
}
```

#### 2.2.3 dirname 函数

`dirname` 函数获取路径的目录部分（除去最后一部分）。

```dart
import 'package:path/path.dart' as p;

void dirnameExample() {
  // 获取父目录
  final parent = p.dirname('/home/user/documents/file.txt');
  print(parent);  // /home/user/documents

  // 多级父目录
  final grandparent = p.dirname(p.dirname('/home/user/documents/file.txt'));
  print(grandparent);  // /home/user

  // 根目录的父目录
  final rootParent = p.dirname('/');
  print(rootParent);  // /

  // 相对路径
  print(p.dirname('documents/file.txt'));  // documents
}
```

#### 2.2.4 extension 函数

`extension` 函数获取文件的扩展名。

```dart
import 'package:path/path.dart' as p;

void extensionExample() {
  // 获取扩展名
  final ext1 = p.extension('/home/user/file.txt');
  print(ext1);  // .txt

  // 多扩展名（level 参数）
  final ext2 = p.extension('archive.tar.gz');
  print(ext2);  // .gz

  final ext3 = p.extension('archive.tar.gz', 2);
  print(ext3);  // .tar.gz

  // 无扩展名
  final noExt = p.extension('README');
  print(noExt);  // (空字符串)

  // 隐藏文件
  final hidden = p.extension('.bashrc');
  print(hidden);  // (空字符串)
}
```

**extension 函数签名：**

```dart
String extension(String path, [int level = 1])
```

| 参数    | 说明                       |
| ------- | -------------------------- |
| `path`  | 文件路径                   |
| `level` | 获取的扩展名层级（默认 1） |

#### 2.2.5 split 函数

`split` 函数将路径分解为各个组成部分。

```dart
import 'package:path/path.dart' as p;

void splitExample() {
  // 分解路径
  final parts = p.split('/home/user/documents/file.txt');
  print(parts);  // [home, user, documents, file.txt]

  // Windows 路径
  final winParts = p.windows.split(r'C:\Users\file.txt');
  print(winParts);  // [C:\, Users, file.txt]

  // 相对路径
  final relParts = p.split('documents/file.txt');
  print(relParts);  // [documents, file.txt]

  // 重新拼接
  final original = p.joinAll(parts);
  print(original);  // home/user/documents/file.txt
}
```

#### 2.2.6 rootPrefix 函数

`rootPrefix` 函数获取路径的根前缀。

```dart
import 'package:path/path.dart' as p;

void rootPrefixExample() {
  // POSIX 根目录
  print(p.rootPrefix('/home/user/file.txt'));  // /

  // Windows 驱动器
  print(p.windows.rootPrefix(r'C:\Users\file.txt'));  // C:\

  // Windows UNC 路径
  print(p.windows.rootPrefix(r'\\server\share\file.txt'));  // \\server\share

  // 相对路径
  print(p.rootPrefix('home/user/file.txt'));  // (空字符串)
}
```

### 2.3 路径判断

#### 2.3.1 isAbsolute 函数

`isAbsolute` 函数判断路径是否为绝对路径。

```dart
import 'package:path/path.dart' as p;

void isAbsoluteExample() {
  // POSIX
  print(p.isAbsolute('/home/user/file.txt'));  // true
  print(p.isAbsolute('home/user/file.txt'));   // false

  // Windows
  print(p.windows.isAbsolute(r'C:\file.txt'));  // true
  print(p.windows.isAbsolute(r'\file.txt'));    // true (根相对)
  print(p.windows.isAbsolute(r'file.txt'));      // false

  // URL
  print(p.url.isAbsolute('https://example.com/file.txt'));  // true
  print(p.url.isAbsolute('/path/file.txt'));                // false
}
```

#### 2.3.2 isRelative 函数

`isRelative` 函数判断路径是否为相对路径。

```dart
import 'package:path/path.dart' as p;

void isRelativeExample() {
  print(p.isRelative('documents/file.txt'));     // true
  print(p.isRelative('/home/user/file.txt'));    // false

  // 当前目录引用
  print(p.isRelative('./file.txt'));  // true
  print(p.isRelative('../file.txt')); // true
}
```

#### 2.3.3 isRootRelative 函数

`isRootRelative` 函数判断路径是否为根相对路径（仅 Windows）。

```dart
import 'package:path/path.dart' as p;

void isRootRelativeExample() {
  // Windows 根相对路径（以 \ 开头但没有驱动器）
  print(p.windows.isRootRelative(r'\Users\file.txt'));  // true
  print(p.windows.isRootRelative(r'C:\Users\file.txt')); // false

  // POSIX 没有根相对路径的概念
  print(p.isRootRelative('/home/file.txt'));  // false
}
```

#### 2.3.4 isWithin 函数

`isWithin` 函数判断一个路径是否在另一个路径之下。

```dart
import 'package:path/path.dart' as p;

void isWithinExample() {
  // 基本用法
  print(p.isWithin('/home/user', '/home/user/documents/file.txt'));  // true
  print(p.isWithin('/home/user', '/home/other/file.txt'));           // false

  // 相同路径
  print(p.isWithin('/home/user', '/home/user'));  // false

  // 相对路径
  print(p.isWithin('documents', 'documents/reports/file.txt'));  // true

  // 需要规范化
  print(p.isWithin('/home/user', '/home/user/../documents/file.txt'));  // false

  // 规范化后
  final normalized = p.normalize('/home/user/../documents/file.txt');
  print(p.isWithin('/home/user', normalized));  // 取决于规范化后的结果
}
```

---

## 第3章 路径规范化与转换

### 3.1 normalize 函数

`normalize` 函数规范化路径，处理 `.` 和 `..`，移除多余的分隔符。

```dart
import 'package:path/path.dart' as p;

void normalizeExample() {
  // 处理当前目录引用
  final path1 = p.normalize('./documents/./file.txt');
  print(path1);  // documents/file.txt

  // 处理父目录引用
  final path2 = p.normalize('/home/user/../documents/file.txt');
  print(path2);  // /home/documents/file.txt

  // 移除多余分隔符
  final path3 = p.normalize('home//user///documents/file.txt');
  print(path3);  // home/user/documents/file.txt

  // 混合情况
  final path4 = p.normalize('/home/./user/../documents/./../downloads/file.txt');
  print(path4);  // /home/downloads/file.txt

  // 超出根目录
  final path5 = p.normalize('/home/../../../file.txt');
  print(path5);  // /file.txt
}
```

### 3.2 relative 函数

`relative` 函数将绝对路径转换为相对路径。

```dart
import 'package:path/path.dart' as p;

void relativeExample() {
  // 假设当前目录是 /home/user

  // 从当前目录计算相对路径
  final rel1 = p.relative('/home/user/documents/file.txt');
  print(rel1);  // documents/file.txt

  // 指定基准目录
  final rel2 = p.relative('/home/user/documents/file.txt', from: '/home');
  print(rel2);  // user/documents/file.txt

  // 向上导航
  final rel3 = p.relative('/etc/passwd', from: '/home/user');
  print(rel3);  // ../../etc/passwd

  // 相同目录
  final rel4 = p.relative('/home/user/file.txt', from: '/home/user');
  print(rel4);  // file.txt

  // Windows 跨驱动器
  print(p.windows.relative(r'D:\file.txt', from: r'C:\Users'));
  // 输出: D:\file.txt (无法相对，返回绝对路径)
}
```

### 3.3 withoutExtension 函数

`withoutExtension` 函数移除路径的扩展名。

```dart
import 'package:path/path.dart' as p;

void withoutExtensionExample() {
  // 移除扩展名
  final path1 = p.withoutExtension('/home/user/file.txt');
  print(path1);  // /home/user/file

  // 多扩展名
  final path2 = p.withoutExtension('archive.tar.gz');
  print(path2);  // archive.tar

  // 目录路径（不受影响）
  final path3 = p.withoutExtension('/home/user.dir/file');
  print(path3);  // /home/user.dir/file

  // 无扩展名
  final path4 = p.withoutExtension('/home/user/README');
  print(path4);  // /home/user/README
}
```

### 3.4 setExtension 函数

`setExtension` 函数设置或替换路径的扩展名。

```dart
import 'package:path/path.dart' as p;

void setExtensionExample() {
  // 设置扩展名
  final path1 = p.setExtension('/home/user/file', '.txt');
  print(path1);  // /home/user/file.txt

  // 替换扩展名
  final path2 = p.setExtension('/home/user/file.pdf', '.docx');
  print(path2);  // /home/user/file.docx

  // 移除扩展名
  final path3 = p.setExtension('/home/user/file.txt', '');
  print(path3);  // /home/user/file

  // 多扩展名
  final path4 = p.setExtension('archive.tar.gz', '.bz2');
  print(path4);  // archive.tar.bz2
}
```

### 3.5 URI 转换

#### 3.5.1 fromUri 函数

`fromUri` 函数将 URI 转换为文件路径。

```dart
import 'package:path/path.dart' as p;

void fromUriExample() {
  // 从 Uri 对象转换
  final uri = Uri.parse('file:///home/user/file.txt');
  final path1 = p.fromUri(uri);
  print(path1);  // /home/user/file.txt

  // 从字符串转换
  final path2 = p.fromUri('file:///C:/Users/file.txt');
  print(path2);  // /C:/Users/file.txt (POSIX)
  // Windows: C:\Users\file.txt

  // 相对 URI
  final path3 = p.fromUri('path/to/file.txt');
  print(path3);  // path/to/file.txt

  // URL 编码
  final path4 = p.fromUri('file:///home/user/my%20file.txt');
  print(path4);  // /home/user/my file.txt
}
```

#### 3.5.2 toUri 函数

`toUri` 函数将文件路径转换为 URI。

```dart
import 'package:path/path.dart' as p;

void toUriExample() {
  // 绝对路径
  final uri1 = p.toUri('/home/user/file.txt');
  print(uri1);  // file:///home/user/file.txt

  // 相对路径
  final uri2 = p.toUri('documents/file.txt');
  print(uri2);  // documents/file.txt

  // Windows 路径
  print(p.windows.toUri(r'C:\Users\file.txt'));
  // 输出: file:///C:/Users/file.txt

  // 包含空格的 path
  final uri3 = p.toUri('/home/user/my file.txt');
  print(uri3);  // file:///home/user/my%20file.txt
}
```

#### 3.5.3 prettyUri 函数

`prettyUri` 函数返回 URI 的简洁可读表示。

```dart
import 'package:path/path.dart' as p;

void prettyUriExample() {
  // 文件 URI
  print(p.prettyUri('file:///home/user/file.txt'));
  // 输出: /home/user/file.txt

  // package URI
  print(p.prettyUri('package:path/path.dart'));
  // 输出: package:path/path.dart

  // HTTP URI
  print(p.prettyUri('https://example.com/path/file.txt'));
  // 输出: https://example.com/path/file.txt
}
```

---

## 第4章 高级用法

### 4.1 Context 类

`Context` 类允许创建特定平台的路径操作上下文，用于处理不同平台的路径。

```dart
import 'package:path/path.dart' as p;

void contextExample() {
  // 创建 Windows 上下文
  final windows = p.Context(style: p.Style.windows);
  print(windows.join('C:', 'Users', 'file.txt'));
  // 输出: C:\Users\file.txt

  // 创建 POSIX 上下文
  final posix = p.Context(style: p.Style.posix);
  print(posix.join('home', 'user', 'file.txt'));
  // 输出: home/user/file.txt

  // 创建 URL 上下文
  final url = p.Context(style: p.Style.url);
  print(url.join('https://example.com', 'path', 'file.txt'));
  // 输出: https://example.com/path/file.txt

  // 使用特定上下文的所有函数
  print(windows.basename(r'C:\Users\file.txt'));  // file.txt
  print(windows.dirname(r'C:\Users\file.txt'));   // C:\Users
  print(windows.extension(r'C:\Users\file.txt')); // .txt
}
```

**Context 构造函数：**

```dart
Context({
  Style? style,           // 路径风格
  String? current,        // 当前工作目录
})
```

#### 4.1.1 使用特定上下文处理路径

```dart
import 'package:path/path.dart' as p;

void contextAdvancedExample() {
  // 在 Linux 上处理 Windows 路径
  final windows = p.Context(style: p.Style.windows);

  // 拼接 Windows 路径
  final winPath = windows.join('C:', 'Program Files', 'App', 'config.ini');
  print(winPath);  // C:\Program Files\App\config.ini

  // 分解 Windows 路径
  final parts = windows.split(winPath);
  print(parts);  // [C:\, Program Files, App, config.ini]

  // 判断是否为绝对路径
  print(windows.isAbsolute(r'C:\file.txt'));  // true
  print(windows.isAbsolute(r'file.txt'));      // false

  // 规范化 Windows 路径
  final normalized = windows.normalize(r'C:\Users\..\Program Files\.\App');
  print(normalized);  // C:\Program Files\App
}
```

### 4.2 Style 枚举

`Style` 枚举定义了三种路径风格：

| 值              | 说明                      |
| --------------- | ------------------------- |
| `Style.posix`   | POSIX 风格（Linux/macOS） |
| `Style.windows` | Windows 风格              |
| `Style.url`     | URL 风格                  |

```dart
import 'package:path/path.dart' as p;

void styleEnumExample() {
  // 获取当前平台的风格
  print('当前风格: ${p.style}');

  // 使用特定风格
  final posix = p.Context(style: p.Style.posix);
  final windows = p.Context(style: p.Style.windows);
  final url = p.Context(style: p.Style.url);

  // 比较不同风格的分隔符
  print('POSIX 分隔符: ${posix.separator}');      // /
  print('Windows 分隔符: ${windows.separator}');  // \
  print('URL 分隔符: ${url.separator}');          // /
}
```

### 4.3 预定义上下文

path 库提供了几个预定义的上下文：

```dart
import 'package:path/path.dart' as p;

void predefinedContextsExample() {
  // 当前平台上下文
  print(p.context.join('a', 'b'));  // 使用当前平台风格

  // POSIX 上下文
  print(p.posix.join('home', 'user'));  // home/user

  // Windows 上下文
  print(p.windows.join('C:', 'Users'));  // C:\Users

  // URL 上下文
  print(p.url.join('https://example.com', 'path'));  // https://example.com/path

  // 当前工作目录
  print('当前目录: ${p.current}');
}
```

### 4.4 PathMap 类

`PathMap` 是一个使用路径相等性作为键的 Map。

```dart
import 'package:path/path.dart' as p;

void pathMapExample() {
  // 创建 PathMap
  final map = p.PathMap<String>();

  // 添加条目
  map['/home/user/file.txt'] = 'content1';
  map['/home/user/../user/file.txt'] = 'content2';  // 相同路径

  // 路径会被规范化后比较
  print(map.length);  // 1 (因为两个键规范化后相同)

  // 使用
  map['/a/b/c.txt'] = 'file A';
  map['/a/b/./c.txt'] = 'file B';  // 覆盖前一个

  print(map['/a/b/c.txt']);  // file B

  // 遍历
  map.forEach((path, value) {
    print('$path: $value');
  });
}
```

### 4.5 PathSet 类

`PathSet` 是一个使用路径相等性的 Set。

```dart
import 'package:path/path.dart' as p;

void pathSetExample() {
  // 创建 PathSet
  final set = p.PathSet();

  // 添加路径
  set.add('/home/user/file.txt');
  set.add('/home/user/../user/file.txt');  // 规范化后相同，不会添加
  set.add('/home/other/file.txt');

  print(set.length);  // 2

  // 检查包含
  print(set.contains('/home/user/./file.txt'));  // true

  // 遍历
  for (final path in set) {
    print(path);
  }
}
```

### 4.6 equals 和 hash 函数

`equals` 和 `hash` 函数提供路径相等性比较和哈希值计算。

```dart
import 'package:path/path.dart' as p;

void equalsAndHashExample() {
  // 路径相等性比较（规范化后比较）
  print(p.equals('/home/user/file.txt', '/home/user/../user/file.txt'));  // true
  print(p.equals('/home/user/file.txt', '/home/other/file.txt'));         // false

  // 哈希值
  final hash1 = p.hash('/home/user/file.txt');
  final hash2 = p.hash('/home/user/../user/file.txt');
  print(hash1 == hash2);  // true

  // 用于自定义数据结构
  final map = <String, String>{};
  map[p.canonicalize('/home/user/file.txt')] = 'value';
}
```

### 4.7 canonicalize 函数

`canonicalize` 函数返回路径的规范形式，用于相等性比较。

```dart
import 'package:path/path.dart' as p;

void canonicalizeExample() {
  // 规范化路径
  final canonical1 = p.canonicalize('/home/user/../documents/./file.txt');
  print(canonical1);  // /home/documents/file.txt

  // 用于比较
  final path1 = '/home/user/file.txt';
  final path2 = '/home/./user/../user/file.txt';

  print(p.canonicalize(path1) == p.canonicalize(path2));  // true

  // Windows（不区分大小写）
  print(p.windows.canonicalize(r'C:\Users\File.TXT'));
  // 输出: c:\users\file.txt (小写)
}
```

---

## 第5章 实战应用

### 5.1 文件路径工具类

```dart
import 'package:path/path.dart' as p;

class PathUtils {
  PathUtils._();  // 私有构造函数

  /// 获取文件名（含扩展名）
  static String getFileName(String filePath) {
    return p.basename(filePath);
  }

  /// 获取文件名（不含扩展名）
  static String getFileNameWithoutExt(String filePath) {
    return p.basenameWithoutExtension(filePath);
  }

  /// 获取文件扩展名
  static String getExtension(String filePath) {
    return p.extension(filePath);
  }

  /// 获取父目录路径
  static String getParentDirectory(String filePath) {
    return p.dirname(filePath);
  }

  /// 更改文件扩展名
  static String changeExtension(String filePath, String newExtension) {
    return p.setExtension(filePath, newExtension);
  }

  /// 移除文件扩展名
  static String removeExtension(String filePath) {
    return p.withoutExtension(filePath);
  }

  /// 拼接路径
  static String joinPaths(String part1, String part2, [String? part3]) {
    if (part3 != null) {
      return p.join(part1, part2, part3);
    }
    return p.join(part1, part2);
  }

  /// 拼接多个路径
  static String joinAllPaths(List<String> parts) {
    return p.joinAll(parts);
  }

  /// 规范化路径
  static String normalizePath(String filePath) {
    return p.normalize(filePath);
  }

  /// 转换为绝对路径
  static String toAbsolutePath(String filePath, [String? from]) {
    if (from != null) {
      return p.absolute(filePath);
    }
    return p.absolute(filePath);
  }

  /// 转换为相对路径
  static String toRelativePath(String filePath, {String? from}) {
    return p.relative(filePath, from: from);
  }

  /// 检查是否为绝对路径
  static bool isAbsolutePath(String filePath) {
    return p.isAbsolute(filePath);
  }

  /// 检查是否为相对路径
  static bool isRelativePath(String filePath) {
    return p.isRelative(filePath);
  }

  /// 检查路径是否在另一个路径下
  static bool isWithinDirectory(String parent, String child) {
    return p.isWithin(parent, child);
  }

  /// 生成唯一文件名
  static String generateUniqueFileName(
    String originalName, {
    String? suffix,
  }) {
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final nameWithoutExt = p.basenameWithoutExtension(originalName);
    final extension = p.extension(originalName);
    final suffixStr = suffix ?? timestamp.toString();

    return '$nameWithoutExt _$suffixStr$extension';
  }

  /// 生成带日期的文件名
  static String generateDatedFileName(String originalName) {
    final now = DateTime.now();
    final dateStr = '${now.year}${now.month.toString().padLeft(2, '0')}${now.day.toString().padLeft(2, '0')}';
    final nameWithoutExt = p.basenameWithoutExtension(originalName);
    final extension = p.extension(originalName);

    return '${nameWithoutExt}_$dateStr$extension';
  }

  /// 获取路径深度
  static int getPathDepth(String filePath) {
    final normalized = p.normalize(filePath);
    final parts = p.split(normalized);
    return parts.length;
  }

  /// 获取共同父目录
  static String? getCommonParent(String path1, String path2) {
    final parts1 = p.split(p.normalize(path1));
    final parts2 = p.split(p.normalize(path2));

    final commonParts = <String>[];
    final minLength = parts1.length < parts2.length ? parts1.length : parts2.length;

    for (var i = 0; i < minLength; i++) {
      if (parts1[i] == parts2[i]) {
        commonParts.add(parts1[i]);
      } else {
        break;
      }
    }

    if (commonParts.isEmpty) return null;
    return p.joinAll(commonParts);
  }

  /// 安全拼接路径（防止路径遍历攻击）
  static String? safeJoin(String basePath, String userInput) {
    final normalizedBase = p.normalize(basePath);
    final joined = p.join(normalizedBase, userInput);
    final normalizedJoined = p.normalize(joined);

    // 确保结果仍在 basePath 下
    if (!p.isWithin(normalizedBase, normalizedJoined) &&
        normalizedJoined != normalizedBase) {
      return null;  // 路径遍历攻击检测
    }

    return normalizedJoined;
  }
}
```

### 5.2 跨平台路径处理

```dart
import 'package:path/path.dart' as p;

class CrossPlatformPath {
  /// 将任意路径转换为当前平台的格式
  static String toCurrentPlatform(String anyPath) {
    // 先规范化
    final normalized = p.normalize(anyPath);

    // 如果当前是 Windows，将 / 转换为 \
    if (p.separator == '\\') {
      return normalized.replaceAll('/', '\\');
    }

    // 如果当前是 POSIX，将 \ 转换为 /
    return normalized.replaceAll('\\', '/');
  }

  /// 将路径转换为 POSIX 格式
  static String toPosix(String anyPath) {
    return anyPath.replaceAll('\\', '/');
  }

  /// 将路径转换为 Windows 格式
  static String toWindows(String anyPath) {
    return anyPath.replaceAll('/', '\\');
  }

  /// 在配置文件中存储平台无关的路径
  static String toPortablePath(String absolutePath) {
    final home = p.Context(style: p.Style.posix).join('~');

    // 尝试将用户目录替换为 ~
    if (absolutePath.startsWith(Platform.environment['HOME'] ?? '')) {
      return absolutePath.replaceFirst(
        Platform.environment['HOME']!,
        '~',
      );
    }

    return absolutePath;
  }

  /// 从可移植路径恢复为绝对路径
  static String fromPortablePath(String portablePath) {
    if (portablePath.startsWith('~/')) {
      final home = Platform.environment['HOME'] ??
                   Platform.environment['USERPROFILE'] ?? '';
      return p.join(home, portablePath.substring(2));
    }

    return portablePath;
  }

  /// 处理 Web 路径
  static String toWebPath(String filePath) {
    // Web 使用 URL 格式
    final url = p.url;
    final normalized = p.normalize(filePath);
    return toPosix(normalized);
  }

  /// 从 Web 路径恢复
  static String fromWebPath(String webPath) {
    return p.normalize(webPath);
  }
}
```

### 5.3 文件管理器

```dart
import 'dart:io';
import 'package:path/path.dart' as p;

class FilePathManager {
  final String _basePath;

  FilePathManager(this._basePath);

  /// 获取基础路径
  String get basePath => _basePath;

  /// 构建完整路径
  String buildPath(String relativePath) {
    return p.join(_basePath, relativePath);
  }

  /// 构建文件路径
  String buildFilePath(String directory, String fileName) {
    return p.join(_basePath, directory, fileName);
  }

  /// 获取目录下的所有文件
  Future<List<File>> listFiles(String directory, {String? extension}) async {
    final dir = Directory(buildPath(directory));
    if (!await dir.exists()) return [];

    final files = <File>[];
    await for (final entity in dir.list()) {
      if (entity is File) {
        if (extension == null || entity.path.endsWith('.$extension')) {
          files.add(entity);
        }
      }
    }

    return files;
  }

  /// 获取目录下的所有子目录
  Future<List<Directory>> listDirectories(String directory) async {
    final dir = Directory(buildPath(directory));
    if (!await dir.exists()) return [];

    final dirs = <Directory>[];
    await for (final entity in dir.list()) {
      if (entity is Directory) {
        dirs.add(entity);
      }
    }

    return dirs;
  }

  /// 创建目录结构
  Future<Directory> createDirectory(String relativePath) async {
    final path = buildPath(relativePath);
    return Directory(path).create(recursive: true);
  }

  /// 移动文件
  Future<File> moveFile(String source, String destination) async {
    final sourceFile = File(buildPath(source));
    final destPath = buildPath(destination);

    // 确保目标目录存在
    await Directory(p.dirname(destPath)).create(recursive: true);

    return sourceFile.rename(destPath);
  }

  /// 复制文件
  Future<File> copyFile(String source, String destination) async {
    final sourceFile = File(buildPath(source));
    final destPath = buildPath(destination);

    // 确保目标目录存在
    await Directory(p.dirname(destPath)).create(recursive: true);

    return sourceFile.copy(destPath);
  }

  /// 删除文件
  Future<void> deleteFile(String relativePath) async {
    final file = File(buildPath(relativePath));
    if (await file.exists()) {
      await file.delete();
    }
  }

  /// 删除目录
  Future<void> deleteDirectory(String relativePath, {bool recursive = false}) async {
    final dir = Directory(buildPath(relativePath));
    if (await dir.exists()) {
      await dir.delete(recursive: recursive);
    }
  }

  /// 检查文件是否存在
  Future<bool> fileExists(String relativePath) async {
    return File(buildPath(relativePath)).exists();
  }

  /// 检查目录是否存在
  Future<bool> directoryExists(String relativePath) async {
    return Directory(buildPath(relativePath)).exists();
  }

  /// 获取文件信息
  Future<FileStat?> getFileInfo(String relativePath) async {
    try {
      return await File(buildPath(relativePath)).stat();
    } catch (e) {
      return null;
    }
  }

  /// 搜索文件
  Future<List<File>> searchFiles(
    String pattern, {
    String? inDirectory,
  }) async {
    final searchDir = inDirectory != null
        ? Directory(buildPath(inDirectory))
        : Directory(_basePath);

    final results = <File>[];
    final regex = RegExp(pattern);

    await for (final entity in searchDir.list(recursive: true)) {
      if (entity is File) {
        final fileName = p.basename(entity.path);
        if (regex.hasMatch(fileName)) {
          results.add(entity);
        }
      }
    }

    return results;
  }

  /// 计算目录大小
  Future<int> calculateDirectorySize(String relativePath) async {
    final dir = Directory(buildPath(relativePath));
    if (!await dir.exists()) return 0;

    var totalSize = 0;
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
}
```

### 5.4 路径验证器

```dart
import 'package:path/path.dart' as p;

class PathValidator {
  PathValidator._();

  /// 验证文件路径是否合法
  static bool isValidFilePath(String path) {
    if (path.isEmpty) return false;

    // 检查是否包含非法字符
    final illegalChars = RegExp(r'[<>:"|?*\x00-\x1f]');
    if (illegalChars.hasMatch(path)) return false;

    // 检查文件名长度
    final fileName = p.basename(path);
    if (fileName.length > 255) return false;

    return true;
  }

  /// 验证是否为安全的相对路径（防止路径遍历）
  static bool isSafeRelativePath(String path, {String? basePath}) {
    if (p.isAbsolute(path)) return false;

    final normalized = p.normalize(path);

    // 检查是否包含 .. 导致的越界
    if (normalized.startsWith('..')) return false;

    // 如果提供了基础路径，验证结果是否在基础路径下
    if (basePath != null) {
      final fullPath = p.join(basePath, normalized);
      return p.isWithin(basePath, fullPath) || fullPath == basePath;
    }

    return true;
  }

  /// 验证文件扩展名
  static bool hasValidExtension(String path, List<String> allowedExtensions) {
    final ext = p.extension(path).toLowerCase();
    return allowedExtensions.contains(ext);
  }

  /// 验证路径长度
  static bool isValidPathLength(String path, {int maxLength = 260}) {
    return path.length <= maxLength;
  }

  /// 清理文件名中的非法字符
  static String sanitizeFileName(String fileName) {
    // 替换非法字符
    var sanitized = fileName
        .replaceAll(RegExp(r'[<>:"/\\|?*\x00-\x1f]'), '_')
        .trim();

    // 移除首尾空格和点
    sanitized = sanitized.replaceAll(RegExp(r'^[\s.]+|[\s.]+$'), '');

    // 限制长度
    if (sanitized.length > 255) {
      final ext = p.extension(sanitized);
      sanitized = sanitized.substring(0, 255 - ext.length) + ext;
    }

    // 如果为空，使用默认名称
    if (sanitized.isEmpty) {
      sanitized = 'unnamed';
    }

    return sanitized;
  }

  /// 清理路径中的非法字符
  static String sanitizePath(String path) {
    final parts = p.split(path);
    final sanitizedParts = parts.map(sanitizeFileName).toList();
    return p.joinAll(sanitizedParts);
  }

  /// 检查是否为图片文件
  static bool isImageFile(String path) {
    final imageExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.bmp', '.webp'];
    return hasValidExtension(path, imageExtensions);
  }

  /// 检查是否为文档文件
  static bool isDocumentFile(String path) {
    final docExtensions = ['.pdf', '.doc', '.docx', '.txt', '.rtf'];
    return hasValidExtension(path, docExtensions);
  }

  /// 检查是否为代码文件
  static bool isCodeFile(String path) {
    final codeExtensions = [
      '.dart', '.js', '.ts', '.java', '.py', '.cpp', '.c', '.h',
      '.cs', '.go', '.rs', '.swift', '.kt', '.rb', '.php'
    ];
    return hasValidExtension(path, codeExtensions);
  }
}
```

### 5.5 批量路径处理

```dart
import 'package:path/path.dart' as p;

class BatchPathProcessor {
  /// 批量规范化路径
  static List<String> normalizeAll(List<String> paths) {
    return paths.map(p.normalize).toList();
  }

  /// 批量转换为绝对路径
  static List<String> toAbsoluteAll(
    List<String> paths, {
    String? from,
  }) {
    return paths.map((path) {
      if (p.isAbsolute(path)) return path;
      if (from != null) {
        return p.normalize(p.join(from, path));
      }
      return p.absolute(path);
    }).toList();
  }

  /// 批量转换为相对路径
  static List<String> toRelativeAll(
    List<String> paths, {
    required String from,
  }) {
    return paths.map((path) => p.relative(path, from: from)).toList();
  }

  /// 批量更改扩展名
  static List<String> changeExtensionAll(
    List<String> paths,
    String newExtension,
  ) {
    return paths.map((path) => p.setExtension(path, newExtension)).toList();
  }

  /// 批量提取文件名
  static List<String> extractFileNames(List<String> paths) {
    return paths.map(p.basename).toList();
  }

  /// 批量提取目录名
  static List<String> extractDirectories(List<String> paths) {
    return paths.map(p.dirname).toList();
  }

  /// 查找重复路径（规范化后）
  static Map<String, List<String>> findDuplicates(List<String> paths) {
    final canonicalMap = <String, List<String>>{};

    for (final path in paths) {
      final canonical = p.canonicalize(path);
      canonicalMap.putIfAbsent(canonical, () => []).add(path);
    }

    // 只返回有重复的
    return Map.fromEntries(
      canonicalMap.entries.where((e) => e.value.length > 1),
    );
  }

  /// 去重路径列表（保持顺序）
  static List<String> uniquePaths(List<String> paths) {
    final seen = <String>{};
    final result = <String>[];

    for (final path in paths) {
      final canonical = p.canonicalize(path);
      if (!seen.contains(canonical)) {
        seen.add(canonical);
        result.add(path);
      }
    }

    return result;
  }

  /// 按目录分组路径
  static Map<String, List<String>> groupByDirectory(List<String> paths) {
    final groups = <String, List<String>>{};

    for (final path in paths) {
      final dir = p.dirname(path);
      groups.putIfAbsent(dir, () => []).add(path);
    }

    return groups;
  }

  /// 按扩展名分组路径
  static Map<String, List<String>> groupByExtension(List<String> paths) {
    final groups = <String, List<String>>{};

    for (final path in paths) {
      final ext = p.extension(path).toLowerCase();
      groups.putIfAbsent(ext.isEmpty ? '(no extension)' : ext, () => []).add(path);
    }

    return groups;
  }

  /// 计算路径的共同前缀
  static String? findCommonPrefix(List<String> paths) {
    if (paths.isEmpty) return null;
    if (paths.length == 1) return p.dirname(paths.first);

    final normalized = paths.map(p.normalize).toList();
    final splitPaths = normalized.map(p.split).toList();

    final firstParts = splitPaths.first;
    var commonLength = firstParts.length;

    for (var i = 1; i < splitPaths.length; i++) {
      final parts = splitPaths[i];
      var j = 0;
      while (j < commonLength && j < parts.length && firstParts[j] == parts[j]) {
        j++;
      }
      commonLength = j;
      if (commonLength == 0) break;
    }

    if (commonLength == 0) return null;
    return p.joinAll(firstParts.take(commonLength));
  }
}
```

---

## 结语

`path` 1.9.1 是 Dart 生态系统中最重要的基础库之一，通过本书的学习，你应该已经掌握了：

1. **核心概念**：路径风格、绝对/相对路径、路径分隔符
2. **核心函数**：路径拼接、分解、判断、规范化
3. **高级用法**：Context 类、Style 枚举、PathMap/PathSet
4. **实战应用**：文件路径工具类、跨平台处理、路径验证器、批量处理

在实际开发中，建议：

- **始终使用前缀导入**：`import 'package:path/path.dart' as p`
- **规范化用户输入的路径**：防止路径遍历攻击
- **使用 Context 处理跨平台路径**：确保代码在不同平台行为一致
- **利用 PathMap/PathSet 处理路径集合**：自动处理路径相等性
- **缓存规范化结果**：避免重复计算

希望本书能帮助你更好地使用 `path` 库处理各种路径操作场景！
