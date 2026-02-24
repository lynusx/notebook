# Dart 核心库 dart:io 详解

> 本文基于 Dart SDK 3.x 版本，深入解析 dart:io 核心库。该库提供了文件系统、网络通信、HTTP 服务器/客户端、进程管理、标准输入输出等 I/O 功能，是构建命令行应用、服务器端应用和 Flutter 桌面/移动应用的基石。

**重要提示**：dart:io 库不能在浏览器环境中使用，仅支持以下平台：

- 服务器端 Dart 应用
- 命令行脚本
- Flutter 移动应用（iOS/Android）
- Flutter 桌面应用（Windows/macOS/Linux）

---

## 目录

1. [文件系统操作](#1-文件系统操作)
2. [HTTP 客户端](#2-http-客户端)
3. [HTTP 服务器](#3-http-服务器)
4. [Socket 通信](#4-socket-通信)
5. [WebSocket](#5-websocket)
6. [进程管理](#6-进程管理)
7. [平台信息](#7-平台信息)
8. [实战应用示例](#8-实战应用示例)
9. [附录：速查表](#9-附录速查表)

---

## 1. 文件系统操作

### 1.1 File 类

`File` 类是 dart:io 中用于操作文件的核心类，提供了文件的创建、读取、写入、复制、移动、删除等完整功能。

#### 创建与存在性检查

```dart
import 'dart:io';

void fileBasics() async {
  // 创建 File 实例（不会立即创建物理文件）
  final file = File('data.txt');

  // 异步检查文件是否存在
  final exists = await file.exists();
  print('文件存在: $exists');

  // 同步检查（阻塞操作，谨慎使用）
  final existsSync = file.existsSync();
  print('文件存在(同步): $existsSync');

  // 获取绝对路径
  final absolutePath = file.absolute.path;
  print('绝对路径: $absolutePath');

  // 获取父目录
  final parent = file.parent;
  print('父目录: ${parent.path}');
}
```

> **Dart Tips：语法小贴士**
>
> `File` 构造函数只创建 Dart 对象，不会立即创建物理文件。只有当调用写入方法（如 `writeAsString`）时才会真正创建文件。

#### 文件读取

dart:io 提供了多种读取文件的方式，适用于不同的场景：

| 方法             | 返回类型               | 适用场景       |
| ---------------- | ---------------------- | -------------- |
| `readAsString()` | `Future<String>`       | 读取文本文件   |
| `readAsBytes()`  | `Future<Uint8List>`    | 读取二进制文件 |
| `readAsLines()`  | `Future<List<String>>` | 按行读取文本   |
| `openRead()`     | `Stream<List<int>>`    | 大文件流式读取 |

```dart
void readFileExamples() async {
  final file = File('example.txt');

  // 方式1：读取为字符串（小文件）
  try {
    final content = await file.readAsString();
    print('文件内容: $content');
  } on FileSystemException catch (e) {
    print('读取失败: ${e.message}');
  }

  // 方式2：指定编码读取
  final utf8Content = await file.readAsString(encoding: utf8);

  // 方式3：读取为字节（适合二进制文件）
  final bytes = await file.readAsBytes();
  print('文件大小: ${bytes.length} 字节');

  // 方式4：按行读取
  final lines = await file.readAsLines();
  for (var i = 0; i < lines.length; i++) {
    print('行 ${i + 1}: ${lines[i]}');
  }

  // 方式5：流式读取（大文件推荐）
  final stream = file.openRead();
  await for (final chunk in stream) {
    print('读取块: ${chunk.length} 字节');
  }
}
```

#### 文件写入

| 方法              | 模式      | 说明                     |
| ----------------- | --------- | ------------------------ |
| `writeAsString()` | 覆盖/追加 | 写入字符串               |
| `writeAsBytes()`  | 覆盖/追加 | 写入字节                 |
| `openWrite()`     | 流式      | 返回 IOSink 用于流式写入 |

```dart
void writeFileExamples() async {
  final file = File('output.txt');

  // 方式1：覆盖写入字符串
  await file.writeAsString('Hello, Dart!');

  // 方式2：追加写入
  await file.writeAsString(
    '\nNew line added',
    mode: FileMode.append,
  );

  // 方式3：写入字节
  final data = utf8.encode('Binary data');
  await file.writeAsBytes(data);

  // 方式4：流式写入（适合大量数据）
  final sink = file.openWrite(mode: FileMode.write);
  sink.writeln('Line 1');
  sink.writeln('Line 2');
  for (var i = 3; i <= 10; i++) {
    sink.writeln('Line $i');
  }
  await sink.close();  // 必须关闭！
}
```

> **Dart Tips：语法小贴士**
>
> `FileMode` 枚举值：
>
> - `FileMode.read` - 只读（默认）
> - `FileMode.write` - 写入（覆盖）
> - `FileMode.append` - 追加
> - `FileMode.writeOnly` - 只写
> - `FileMode.writeOnlyAppend` - 只追加

#### 文件操作（复制、移动、删除）

```dart
void fileOperations() async {
  final file = File('source.txt');

  // 复制文件
  final copied = await file.copy('backup.txt');
  print('复制到: ${copied.path}');

  // 重命名/移动文件
  final renamed = await file.rename('renamed.txt');
  print('重命名为: ${renamed.path}');

  // 删除文件
  await file.delete();

  // 安全删除（不存在时不报错）
  final tempFile = File('temp.txt');
  if (await tempFile.exists()) {
    await tempFile.delete();
  }

  // 获取文件元数据
  final stat = await file.stat();
  print('大小: ${stat.size}');
  print('修改时间: ${stat.modified}');
  print('访问时间: ${stat.accessed}');
  print('类型: ${stat.type}');  // file, directory, link, notFound
}
```

#### 文件元数据

```dart
void fileMetadata() async {
  final file = File('document.pdf');

  // 获取文件统计信息
  final stat = await file.stat();

  // 属性说明
  print('''
文件元数据:
  - 大小: ${stat.size} 字节
  - 模式: ${stat.mode} (权限位)
  - 修改时间: ${stat.modified}
  - 访问时间: ${stat.accessed}
  - 修改时间: ${stat.changed}
  - 类型: ${stat.type}
  ''');

  // 最后修改时间
  final lastModified = await file.lastModified();
  print('最后修改: $lastModified');

  // 文件长度
  final length = await file.length();
  print('文件长度: $length 字节');
}
```

### 1.2 Directory 类

`Directory` 类用于操作目录，提供了目录的创建、遍历、删除等功能。

#### 创建目录

```dart
void createDirectory() async {
  // 创建单级目录
  final dir1 = Directory('my_folder');
  await dir1.create();

  // 递归创建（创建所有不存在的父目录）
  final dir2 = Directory('parent/child/grandchild');
  await dir2.create(recursive: true);

  // 创建临时目录
  final tempDir = await Directory.systemTemp.createTemp('my_app_');
  print('临时目录: ${tempDir.path}');
}
```

#### 列出目录内容

```dart
void listDirectory() async {
  final dir = Directory('.');

  // 列出当前目录内容（非递归）
  await for (final entity in dir.list()) {
    final type = entity is File ? '文件' :
                 entity is Directory ? '目录' : '链接';
    print('$type: ${entity.path}');
  }

  // 递归列出所有内容
  print('\n--- 递归列出 ---');
  await for (final entity in dir.list(recursive: true)) {
    print(entity.path);
  }

  // 只列出文件
  print('\n--- 只列出文件 ---');
  await for (final entity in dir.list()) {
    if (entity is File) {
      print('文件: ${entity.path}');
    }
  }

  // 只列出目录
  print('\n--- 只列出目录 ---');
  await for (final entity in dir.list()) {
    if (entity is Directory) {
      print('目录: ${entity.path}');
    }
  }
}
```

#### 目录操作

```dart
void directoryOperations() async {
  final dir = Directory('test_dir');

  // 重命名目录
  await dir.rename('renamed_dir');

  // 删除空目录
  await dir.delete();

  // 递归删除目录及其内容
  final deepDir = Directory('deep_folder');
  await deepDir.delete(recursive: true);

  // 获取目录统计信息
  final stat = await dir.stat();
  print('目录类型: ${stat.type}');
}
```

### 1.3 Link 类

`Link` 类用于创建和管理符号链接（软链接）。

```dart
void linkOperations() async {
  // 创建符号链接
  final link = Link('shortcut.txt');
  await link.create('target.txt');

  // 获取链接目标
  final target = await link.target();
  print('链接指向: $target');

  // 更新链接目标
  await link.update('new_target.txt');

  // 删除链接
  await link.delete();

  // 检查是否为链接
  final isLink = FileSystemEntity.isLinkSync('shortcut.txt');
  print('是链接: $isLink');
}
```

### 1.4 路径操作

```dart
import 'package:path/path.dart' as p;

void pathOperations() {
  // 连接路径
  final fullPath = p.join('folder', 'subfolder', 'file.txt');
  print(fullPath);  // folder/subfolder/file.txt

  // 获取文件名
  final fileName = p.basename('/home/user/document.txt');
  print(fileName);  // document.txt

  // 获取目录名
  final dirName = p.dirname('/home/user/document.txt');
  print(dirName);  // /home/user

  // 获取扩展名
  final ext = p.extension('image.png');
  print(ext);  // .png

  // 规范化路径
  final normalized = p.normalize('/home/../etc//config.json');
  print(normalized);  // /etc/config.json

  // 判断是否为绝对路径
  print(p.isAbsolute('/home/user'));  // true
  print(p.isAbsolute('relative/path'));  // false
}
```

> **Dart Tips：语法小贴士**
>
> `path` 包（`package:path`）提供了跨平台的路径操作功能，可以正确处理 Windows 和 Unix 风格的路径分隔符。

---

## 2. HTTP 客户端

### 2.1 HttpClient 基础

`HttpClient` 是 dart:io 中用于发送 HTTP 请求的客户端类。

#### 创建和配置

```dart
void httpClientBasics() {
  // 创建 HttpClient
  final client = HttpClient();

  // 配置连接超时
  client.connectionTimeout = Duration(seconds: 30);

  // 配置空闲超时
  client.idleTimeout = Duration(seconds: 60);

  // 配置用户代理
  client.userAgent = 'MyDartApp/1.0';

  // 自动解压 gzip 响应
  client.autoUncompress = true;

  // 使用完毕后关闭
  // client.close();
}
```

#### GET 请求

```dart
void httpGet() async {
  final client = HttpClient();

  try {
    // 创建 GET 请求
    final request = await client.getUrl(
      Uri.parse('https://api.example.com/users'),
    );

    // 添加请求头
    request.headers.set('Accept', 'application/json');
    request.headers.set('Authorization', 'Bearer token123');

    // 发送请求并获取响应
    final response = await request.close();

    print('状态码: ${response.statusCode}');
    print('响应头: ${response.headers}');

    // 读取响应体
    final body = await utf8.decoder.bind(response).join();
    print('响应体: $body');

  } finally {
    client.close();
  }
}
```

#### POST 请求

```dart
void httpPost() async {
  final client = HttpClient();

  try {
    final request = await client.postUrl(
      Uri.parse('https://api.example.com/users'),
    );

    // 设置请求头
    request.headers.contentType = ContentType.json;
    request.headers.set('Authorization', 'Bearer token123');

    // 写入请求体
    final body = jsonEncode({
      'name': 'Alice',
      'email': 'alice@example.com',
    });
    request.write(body);

    final response = await request.close();
    final responseBody = await utf8.decoder.bind(response).join();

    print('创建成功: $responseBody');

  } finally {
    client.close();
  }
}
```

#### 其他 HTTP 方法

```dart
void httpMethods() async {
  final client = HttpClient();
  final url = Uri.parse('https://api.example.com/users/1');

  // PUT 请求
  final putRequest = await client.putUrl(url);
  putRequest.headers.contentType = ContentType.json;
  putRequest.write(jsonEncode({'name': 'Updated Name'}));
  final putResponse = await putRequest.close();

  // DELETE 请求
  final deleteRequest = await client.deleteUrl(url);
  final deleteResponse = await deleteRequest.close();

  // PATCH 请求
  final patchRequest = await client.patchUrl(url);
  patchRequest.headers.contentType = ContentType.json;
  patchRequest.write(jsonEncode({'email': 'new@example.com'}));
  final patchResponse = await patchRequest.close();

  // HEAD 请求
  final headRequest = await client.headUrl(url);
  final headResponse = await headRequest.close();
  print('Content-Length: ${headResponse.headers.contentLength}');

  client.close();
}
```

### 2.2 处理 HTTP 响应

#### 响应头处理

```dart
void handleResponseHeaders(HttpClientResponse response) {
  // 获取状态码
  final statusCode = response.statusCode;
  print('状态: $statusCode ${response.reasonPhrase}');

  // 获取所有头
  response.headers.forEach((name, values) {
    print('$name: ${values.join(", ")}');
  });

  // 获取特定头
  final contentType = response.headers.contentType;
  print('Content-Type: $contentType');

  final contentLength = response.headers.contentLength;
  print('Content-Length: $contentLength');

  // 获取 Cookie
  for (final cookie in response.cookies) {
    print('Cookie: ${cookie.name}=${cookie.value}');
  }
}
```

#### 处理重定向

```dart
void handleRedirects() async {
  final client = HttpClient();

  // 配置重定向行为
  final request = await client.getUrl(
    Uri.parse('https://bit.ly/short_url'),
  );

  // 启用/禁用自动重定向
  request.followRedirects = true;

  // 设置最大重定向次数
  request.maxRedirects = 5;

  final response = await request.close();

  // 查看重定向历史
  for (final redirect in response.redirects) {
    print('重定向: ${redirect.method} ${redirect.location}');
  }

  print('最终 URL: ${response.redirects.lastOrNull?.location ?? request.uri}');

  client.close();
}
```

#### 流式处理响应

```dart
void streamResponse() async {
  final client = HttpClient();

  try {
    final request = await client.getUrl(
      Uri.parse('https://api.example.com/large-data'),
    );
    final response = await request.close();

    // 流式处理响应（适合大文件）
    var totalBytes = 0;
    await for (final chunk in response) {
      totalBytes += chunk.length;
      print('收到块: ${chunk.length} 字节，总计: $totalBytes');
    }

    print('下载完成，共 $totalBytes 字节');

  } finally {
    client.close();
  }
}
```

### 2.3 上传文件

```dart
void uploadFile() async {
  final client = HttpClient();

  try {
    final request = await client.postUrl(
      Uri.parse('https://api.example.com/upload'),
    );

    // 设置 multipart content type
    request.headers.set(
      HttpHeaders.contentTypeHeader,
      'multipart/form-data; boundary=----DartFormBoundary',
    );

    // 读取文件
    final file = File('image.png');
    final fileBytes = await file.readAsBytes();

    // 构建 multipart body
    final boundary = '------DartFormBoundary';
    final body = BytesBuilder();

    // 添加字段
    body.add(utf8.encode('$boundary\r\n'));
    body.add(utf8.encode('Content-Disposition: form-data; name="description"\r\n\r\n'));
    body.add(utf8.encode('My image\r\n'));

    // 添加文件
    body.add(utf8.encode('$boundary\r\n'));
    body.add(utf8.encode('Content-Disposition: form-data; name="file"; filename="image.png"\r\n'));
    body.add(utf8.encode('Content-Type: image/png\r\n\r\n'));
    body.add(fileBytes);
    body.add(utf8.encode('\r\n'));

    // 结束 boundary
    body.add(utf8.encode('$boundary--\r\n'));

    // 发送
    request.add(body.toBytes());

    final response = await request.close();
    final responseBody = await utf8.decoder.bind(response).join();
    print('上传结果: $responseBody');

  } finally {
    client.close();
  }
}
```

### 2.4 下载文件（带进度回调）

```dart
Future<void> downloadFile(
  String url,
  String savePath, {
  void Function(int received, int total)? onProgress,
}) async {
  final client = HttpClient();

  try {
    final request = await client.getUrl(Uri.parse(url));
    final response = await request.close();

    final contentLength = response.headers.contentLength;
    var receivedBytes = 0;

    final file = File(savePath);
    final sink = file.openWrite();

    await for (final chunk in response) {
      sink.add(chunk);
      receivedBytes += chunk.length;

      if (onProgress != null && contentLength > 0) {
        onProgress(receivedBytes, contentLength);
      }
    }

    await sink.close();
    print('下载完成: $savePath');

  } finally {
    client.close();
  }
}

// 使用
void main() async {
  await downloadFile(
    'https://example.com/file.zip',
    'downloads/file.zip',
    onProgress: (received, total) {
      final percent = (received / total * 100).toStringAsFixed(1);
      print('下载进度: $percent% ($received / $total)');
    },
  );
}
```

---

## 3. HTTP 服务器

### 3.1 创建 HTTP 服务器

```dart
import 'dart:io';

void main() async {
  // 创建 HTTP 服务器
  final server = await HttpServer.bind(
    InternetAddress.loopbackIPv4,  // 127.0.0.1
    8080,                          // 端口
  );

  print('服务器运行在 http://${server.address.host}:${server.port}');

  // 监听请求
  await for (final request in server) {
    handleRequest(request);
  }
}

void handleRequest(HttpRequest request) {
  final response = request.response;

  // 设置状态码
  response.statusCode = HttpStatus.ok;

  // 设置响应头
  response.headers.contentType = ContentType.text;

  // 写入响应内容
  response.write('Hello, World!');

  // 关闭响应
  response.close();
}
```

#### HTTPS 服务器

```dart
Future<void> startHttpsServer() async {
  // 创建安全上下文
  final context = SecurityContext()
    ..useCertificateChain('server.crt')
    ..usePrivateKey('server.key');

  // 创建 HTTPS 服务器
  final server = await HttpServer.bindSecure(
    InternetAddress.anyIPv4,
    443,
    context,
  );

  print('HTTPS 服务器运行在 https://localhost:443');

  await for (final request in server) {
    request.response
      ..write('Secure connection!')
      ..close();
  }
}
```

### 3.2 路由处理

```dart
void handleRequest(HttpRequest request) async {
  final response = request.response;
  final path = request.uri.path;
  final method = request.method;

  print('$method ${request.uri}');

  // 路由分发
  switch (path) {
    case '/':
      _handleHome(request, response);

    case '/api/users':
      switch (method) {
        case 'GET':
          await _handleGetUsers(request, response);
        case 'POST':
          await _handleCreateUser(request, response);
        default:
          _sendError(response, HttpStatus.methodNotAllowed, 'Method Not Allowed');
      }

    case '/api/users/':
      if (path.startsWith('/api/users/')) {
        final id = path.split('/').last;
        await _handleGetUser(request, response, id);
      }

    case '/health':
      response
        ..statusCode = HttpStatus.ok
        ..write('{"status": "healthy"}')
        ..close();

    default:
      _sendError(response, HttpStatus.notFound, 'Not Found');
  }
}

void _handleHome(HttpRequest request, HttpResponse response) {
  response
    ..headers.contentType = ContentType.html
    ..write('<h1>Welcome</h1>')
    ..close();
}

Future<void> _handleGetUsers(HttpRequest request, HttpResponse response) async {
  response
    ..headers.contentType = ContentType.json
    ..write('[{"id": "1", "name": "Alice"}]')
    ..close();
}

Future<void> _handleCreateUser(HttpRequest request, HttpResponse response) async {
  final body = await utf8.decoder.bind(request).join();
  final data = jsonDecode(body);

  // 创建用户逻辑...

  response
    ..statusCode = HttpStatus.created
    ..headers.contentType = ContentType.json
    ..write(jsonEncode({'id': '2', ...data}))
    ..close();
}

void _sendError(HttpResponse response, int code, String message) {
  response
    ..statusCode = code
    ..write(message)
    ..close();
}
```

### 3.3 静态文件服务

```dart
Future<void> serveStaticFile(HttpRequest request, String rootPath) async {
  final uri = request.uri;
  var path = uri.path == '/' ? '/index.html' : uri.path;

  // 安全处理：防止目录遍历攻击
  if (path.contains('..')) {
    request.response
      ..statusCode = HttpStatus.forbidden
      ..write('Forbidden')
      ..close();
    return;
  }

  final filePath = '$rootPath$path';
  final file = File(filePath);

  if (await file.exists()) {
    // 根据扩展名设置 Content-Type
    final ext = filePath.split('.').last.toLowerCase();
    final contentType = _getContentType(ext);

    request.response
      ..statusCode = HttpStatus.ok
      ..headers.contentType = contentType
      ..headers.set(HttpHeaders.contentLengthHeader, await file.length());

    // 流式发送文件
    await request.response.addStream(file.openRead());
    await request.response.close();

  } else {
    request.response
      ..statusCode = HttpStatus.notFound
      ..write('File Not Found')
      ..close();
  }
}

ContentType _getContentType(String ext) {
  return switch (ext) {
    'html' || 'htm' => ContentType.html,
    'css' => ContentType('text', 'css'),
    'js' => ContentType('application', 'javascript'),
    'json' => ContentType.json,
    'png' => ContentType('image', 'png'),
    'jpg' || 'jpeg' => ContentType('image', 'jpeg'),
    'gif' => ContentType('image', 'gif'),
    'svg' => ContentType('image', 'svg+xml'),
    'pdf' => ContentType('application', 'pdf'),
    'zip' => ContentType('application', 'zip'),
    _ => ContentType.text,
  };
}
```

### 3.4 中间件模式

```dart
typedef Middleware = Future<void> Function(HttpRequest, HttpResponse, Future<void> Function());

class HttpServerWithMiddleware {
  final List<Middleware> _middlewares = [];
  final Map<String, Function> _routes = {};

  void use(Middleware middleware) {
    _middlewares.add(middleware);
  }

  void get(String path, Function handler) {
    _routes['GET:$path'] = handler;
  }

  void post(String path, Function handler) {
    _routes['POST:$path'] = handler;
  }

  Future<void> start(int port) async {
    final server = await HttpServer.bind(InternetAddress.loopbackIPv4, port);
    print('服务器运行在 http://localhost:$port');

    await for (final request in server) {
      await _handleRequest(request);
    }
  }

  Future<void> _handleRequest(HttpRequest request) async {
    final response = request.response;
    final key = '${request.method}:${request.uri.path}';
    final handler = _routes[key];

    // 执行中间件链
    var index = 0;
    Future<void> next() async {
      if (index < _middlewares.length) {
        final middleware = _middlewares[index++];
        await middleware(request, response, next);
      } else if (handler != null) {
        await handler(request, response);
      } else {
        response
          ..statusCode = HttpStatus.notFound
          ..write('Not Found')
          ..close();
      }
    }

    await next();
  }
}

// 使用中间件
void main() async {
  final app = HttpServerWithMiddleware();

  // 日志中间件
  app.use((request, response, next) async {
    final start = DateTime.now();
    await next();
    final duration = DateTime.now().difference(start);
    print('${request.method} ${request.uri.path} - ${duration.inMilliseconds}ms');
  });

  // CORS 中间件
  app.use((request, response, next) async {
    response.headers.set('Access-Control-Allow-Origin', '*');
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    await next();
  });

  // 路由
  app.get('/', (request, response) {
    response.write('Hello, World!');
    response.close();
  });

  await app.start(8080);
}
```

---

## 4. Socket 通信

### 4.1 TCP Socket（客户端）

```dart
void tcpClient() async {
  // 连接到服务器
  final socket = await Socket.connect('localhost', 8080);

  print('已连接到 ${socket.remoteAddress}:${socket.remotePort}');

  // 发送数据
  socket.write('Hello, Server!');

  // 监听响应
  socket.listen(
    (data) {
      print('收到: ${String.fromCharCodes(data)}');
    },
    onError: (error) {
      print('错误: $error');
    },
    onDone: () {
      print('连接已关闭');
    },
  );

  // 关闭连接
  await Future.delayed(Duration(seconds: 5));
  await socket.close();
}
```

### 4.2 TCP ServerSocket（服务器）

```dart
void tcpServer() async {
  // 创建服务器
  final server = await ServerSocket.bind('localhost', 8080);

  print('TCP 服务器运行在 ${server.address}:${server.port}');

  // 监听连接
  await for (final socket in server) {
    handleClient(socket);
  }
}

void handleClient(Socket socket) {
  print('客户端连接: ${socket.remoteAddress}:${socket.remotePort}');

  socket.listen(
    (data) {
      final message = String.fromCharCodes(data);
      print('收到: $message');

      // 发送响应
      socket.write('Echo: $message');
    },
    onError: (error) {
      print('客户端错误: $error');
    },
    onDone: () {
      print('客户端断开连接');
    },
  );
}
```

### 4.3 UDP Socket

```dart
void udpSocket() async {
  // 创建 UDP Socket
  final socket = await RawDatagramSocket.bind(InternetAddress.anyIPv4, 8080);

  print('UDP Socket 绑定到 ${socket.address}:${socket.port}');

  // 发送数据
  final data = utf8.encode('Hello UDP!');
  socket.send(data, InternetAddress('localhost'), 9090);

  // 接收数据
  await for (final event in socket) {
    if (event == RawSocketEvent.read) {
      final datagram = socket.receive();
      if (datagram != null) {
        final message = utf8.decode(datagram.data);
        print('收到来自 ${datagram.address}:${datagram.port}: $message');
      }
    }
  }
}
```

---

## 5. WebSocket

### 5.1 WebSocket 客户端

```dart
void websocketClient() async {
  // 连接 WebSocket 服务器
  final ws = await WebSocket.connect('wss://echo.websocket.org');

  print('WebSocket 已连接');

  // 发送消息
  ws.add('Hello, WebSocket!');

  // 发送二进制数据
  ws.add([1, 2, 3, 4, 5]);

  // 监听消息
  await for (final message in ws) {
    if (message is String) {
      print('收到文本: $message');
    } else if (message is List<int>) {
      print('收到二进制: $message');
    }
  }

  // 关闭连接
  await ws.close();
}
```

### 5.2 WebSocket 服务器

```dart
void websocketServer() async {
  // 创建 HTTP 服务器
  final server = await HttpServer.bind('localhost', 8080);

  print('WebSocket 服务器运行在 ws://localhost:8080');

  await for (final request in server) {
    // 升级 HTTP 到 WebSocket
    final ws = await WebSocketTransformer.upgrade(request);

    print('客户端已连接');

    // 处理消息
    ws.listen(
      (message) {
        print('收到: $message');
        ws.add('Echo: $message');
      },
      onError: (error) => print('错误: $error'),
      onDone: () => print('客户端断开'),
    );
  }
}
```

### 5.3 WebSocket 广播服务器

```dart
class WebSocketBroadcastServer {
  final Set<WebSocket> _clients = {};
  HttpServer? _server;

  Future<void> start(String host, int port) async {
    _server = await HttpServer.bind(host, port);
    print('服务器运行在 ws://$host:$port');

    await for (final request in _server!) {
      final ws = await WebSocketTransformer.upgrade(request);
      _clients.add(ws);

      print('客户端连接，当前 ${_clients.length} 个客户端');

      ws.listen(
        (message) => _broadcast(message, ws),
        onDone: () {
          _clients.remove(ws);
          print('客户端断开，剩余 ${_clients.length} 个');
        },
      );
    }
  }

  void _broadcast(dynamic message, WebSocket sender) {
    for (final client in _clients) {
      if (client != sender) {
        client.add(message);
      }
    }
  }

  Future<void> stop() async {
    await _server?.close();
    for (final client in _clients) {
      await client.close();
    }
    _clients.clear();
  }
}
```

---

## 6. 进程管理

### 6.1 运行命令

#### Process.run（等待完成）

```dart
void runCommand() async {
  // 运行命令并等待完成
  final result = await Process.run('ls', ['-la']);

  print('退出码: ${result.exitCode}');
  print('标准输出:\n${result.stdout}');
  print('标准错误:\n${result.stderr}');
}
```

#### Process.start（流式输出）

```dart
void startCommand() async {
  // 启动进程
  final process = await Process.start('ping', ['-c', '4', 'google.com']);

  // 实时读取输出
  process.stdout.transform(utf8.decoder).listen(print);
  process.stderr.transform(utf8.decoder).listen(print);

  // 等待进程结束
  final exitCode = await process.exitCode;
  print('进程退出码: $exitCode');
}
```

### 6.2 进程控制

```dart
void processControl() async {
  final process = await Process.start('long_running_task', []);

  // 向进程发送输入
  process.stdin.writeln('input data');
  await process.stdin.close();

  // 杀死进程
  await Future.delayed(Duration(seconds: 5));
  process.kill(ProcessSignal.sigterm);  // 优雅终止
  // process.kill(ProcessSignal.sigkill);  // 强制终止

  // 检查进程是否还在运行
  print('进程已杀死: ${process.kill()}');
}
```

### 6.3 运行 Shell 命令

```dart
void runShellCommand() async {
  // Linux/macOS
  if (!Platform.isWindows) {
    final result = await Process.run('sh', ['-c', 'echo \$HOME && ls -la']);
    print(result.stdout);
  }

  // Windows
  if (Platform.isWindows) {
    final result = await Process.run('cmd', ['/c', 'echo %USERNAME% && dir']);
    print(result.stdout);
  }
}
```

---

## 7. 平台信息

### 7.1 操作系统信息

```dart
void platformInfo() {
  // 操作系统类型
  print('操作系统: ${Platform.operatingSystem}');  // linux, macos, windows, android, ios

  // 操作系统版本
  print('版本: ${Platform.operatingSystemVersion}');

  // 处理器数量
  print('处理器数量: ${Platform.numberOfProcessors}');

  // 路径分隔符
  print('路径分隔符: ${Platform.pathSeparator}');  // / 或 \

  // 换行符
  print('换行符: ${Platform.lineTerminator}');  // \n 或 \r\n

  // 本地语言环境
  print('语言环境: ${Platform.localeName}');
}
```

### 7.2 路径信息

```dart
void pathInfo() {
  // 当前脚本路径
  print('脚本路径: ${Platform.script}');

  // 可执行文件路径
  print('可执行文件: ${Platform.resolvedExecutable}');

  // 包配置文件路径
  print('包配置: ${Platform.packageConfig}');

  // 当前工作目录
  print('当前目录: ${Directory.current.path}');

  // 系统临时目录
  print('临时目录: ${Directory.systemTemp.path}');
}
```

### 7.3 环境变量

```dart
void environmentVariables() {
  // 获取所有环境变量
  final env = Platform.environment;
  env.forEach((key, value) {
    print('$key=$value');
  });

  // 获取特定环境变量
  final path = Platform.environment['PATH'];
  final home = Platform.environment['HOME'] ??
               Platform.environment['USERPROFILE'];

  print('PATH: $path');
  print('HOME: $home');

  // 检查环境变量是否存在
  if (Platform.environment.containsKey('API_KEY')) {
    print('API_KEY 已设置');
  }
}
```

---

## 8. 实战应用示例

### 8.1 文件同步工具（带 MD5 校验）

```dart
import 'dart:convert';
import 'dart:io';
import 'package:crypto/crypto.dart';

class FileSyncTool {
  /// 同步两个目录，只复制修改过的文件
  static Future<void> syncDirectories(
    String sourceDir,
    String targetDir, {
    void Function(String action, String file)? onProgress,
  }) async {
    final source = Directory(sourceDir);

    if (!await source.exists()) {
      throw Exception('源目录不存在: $sourceDir');
    }

    await for (final entity in source.list(recursive: true)) {
      if (entity is File) {
        final relativePath = entity.path.substring(sourceDir.length);
        final targetPath = '$targetDir$relativePath';

        final targetFile = File(targetPath);

        // 检查目标文件是否存在且一致
        if (await targetFile.exists()) {
          final sourceHash = await _calculateMd5(entity);
          final targetHash = await _calculateMd5(targetFile);

          if (sourceHash == targetHash) {
            onProgress?.call('跳过', relativePath);
            continue;
          }
        }

        // 确保目标目录存在
        await Directory(File(targetPath).parent.path).create(recursive: true);

        // 复制文件
        await entity.copy(targetPath);
        onProgress?.call('复制', relativePath);
      }
    }

    onProgress?.call('完成', '');
  }

  static Future<String> _calculateMd5(File file) async {
    final bytes = await file.readAsBytes();
    return md5.convert(bytes).toString();
  }
}

// 使用
void main() async {
  await FileSyncTool.syncDirectories(
    './source',
    './target',
    onProgress: (action, file) => print('[$action] $file'),
  );
}
```

### 8.2 HTTP 代理服务器

```dart
import 'dart:io';

class SimpleProxyServer {
  final String targetHost;
  final int targetPort;
  final int listenPort;
  HttpServer? _server;

  SimpleProxyServer({
    required this.targetHost,
    required this.targetPort,
    this.listenPort = 8888,
  });

  Future<void> start() async {
    _server = await HttpServer.bind(InternetAddress.loopbackIPv4, listenPort);
    print('代理服务器运行在 http://localhost:$listenPort');
    print('目标: http://$targetHost:$targetPort');

    await for (final request in _server!) {
      _handleRequest(request);
    }
  }

  Future<void> _handleRequest(HttpRequest request) async {
    final client = HttpClient();

    try {
      // 创建到目标的请求
      final proxyRequest = await client.open(
        request.method,
        targetHost,
        targetPort,
        request.uri.path,
      );

      // 复制请求头
      request.headers.forEach((name, values) {
        for (final value in values) {
          proxyRequest.headers.add(name, value);
        }
      });

      // 转发请求体
      await proxyRequest.addStream(request);

      // 获取响应
      final proxyResponse = await proxyRequest.close();

      // 复制响应状态码
      request.response.statusCode = proxyResponse.statusCode;

      // 复制响应头
      proxyResponse.headers.forEach((name, values) {
        for (final value in values) {
          request.response.headers.add(name, value);
        }
      });

      // 转发响应体
      await request.response.addStream(proxyResponse);
      await request.response.close();

      print('${request.method} ${request.uri.path} -> ${proxyResponse.statusCode}');

    } catch (e) {
      print('错误: $e');
      request.response
        ..statusCode = HttpStatus.badGateway
        ..write('Proxy Error')
        ..close();
    } finally {
      client.close();
    }
  }

  Future<void> stop() async {
    await _server?.close();
  }
}

// 使用
void main() async {
  final proxy = SimpleProxyServer(
    targetHost: 'jsonplaceholder.typicode.com',
    targetPort: 80,
    listenPort: 8888,
  );
  await proxy.start();
}
```

### 8.3 简单的 CLI 下载工具（多线程下载）

```dart
import 'dart:io';
import 'dart:math';

class MultiThreadDownloader {
  static const int chunkSize = 1024 * 1024;  // 1MB

  static Future<void> download(
    String url,
    String savePath, {
    int threads = 4,
    void Function(double progress)? onProgress,
  }) async {
    final client = HttpClient();

    try {
      // 获取文件大小
      final headRequest = await client.headUrl(Uri.parse(url));
      final headResponse = await headRequest.close();
      final totalSize = headResponse.headers.contentLength;

      if (totalSize <= 0) {
        throw Exception('无法获取文件大小');
      }

      print('文件大小: ${(totalSize / 1024 / 1024).toStringAsFixed(2)} MB');

      // 创建临时文件
      final tempDir = await Directory.systemTemp.createTemp('download_');
      final chunkFiles = <File>[];

      // 计算每个线程的下载范围
      final chunkLength = (totalSize / threads).ceil();
      final futures = <Future<void>>[];

      for (var i = 0; i < threads; i++) {
        final start = i * chunkLength;
        final end = min((i + 1) * chunkLength - 1, totalSize - 1);

        final chunkFile = File('${tempDir.path}/chunk_$i');
        chunkFiles.add(chunkFile);

        futures.add(_downloadChunk(url, start, end, chunkFile));
      }

      // 等待所有线程完成
      await Future.wait(futures);

      // 合并文件
      final output = File(savePath).openWrite();
      for (final chunkFile in chunkFiles) {
        await output.addStream(chunkFile.openRead());
        await chunkFile.delete();
      }
      await output.close();
      await tempDir.delete();

      print('下载完成: $savePath');

    } finally {
      client.close();
    }
  }

  static Future<void> _downloadChunk(
    String url,
    int start,
    int end,
    File saveFile,
  ) async {
    final client = HttpClient();

    try {
      final request = await client.getUrl(Uri.parse(url));
      request.headers.set('Range', 'bytes=$start-$end');

      final response = await request.close();
      await response.pipe(saveFile.openWrite());

    } finally {
      client.close();
    }
  }
}

// 使用
void main() async {
  await MultiThreadDownloader.download(
    'https://example.com/large-file.zip',
    'downloads/file.zip',
    threads: 4,
    onProgress: (progress) => print('进度: ${(progress * 100).toStringAsFixed(1)}%'),
  );
}
```

---

## 9. 附录：速查表

### 9.1 常用类对照表

| 功能         | 主要类             | 用途             |
| ------------ | ------------------ | ---------------- |
| 文件操作     | `File`             | 读写文件         |
| 目录操作     | `Directory`        | 创建/列出目录    |
| 符号链接     | `Link`             | 创建软链接       |
| 文件系统实体 | `FileSystemEntity` | 通用文件系统操作 |
| HTTP 服务器  | `HttpServer`       | 创建 Web 服务器  |
| HTTP 客户端  | `HttpClient`       | 发送 HTTP 请求   |
| HTTP 请求    | `HttpRequest`      | 服务器端请求对象 |
| HTTP 响应    | `HttpResponse`     | 服务器端响应对象 |
| TCP Socket   | `Socket`           | TCP 通信         |
| TCP 服务器   | `ServerSocket`     | TCP 服务器       |
| WebSocket    | `WebSocket`        | WebSocket 通信   |
| 进程         | `Process`          | 运行外部命令     |
| 平台信息     | `Platform`         | 获取系统信息     |

### 9.2 HTTP 状态码常量

| 常量                             | 值  | 含义           |
| -------------------------------- | --- | -------------- |
| `HttpStatus.ok`                  | 200 | 成功           |
| `HttpStatus.created`             | 201 | 已创建         |
| `HttpStatus.accepted`            | 202 | 已接受         |
| `HttpStatus.noContent`           | 204 | 无内容         |
| `HttpStatus.movedPermanently`    | 301 | 永久重定向     |
| `HttpStatus.found`               | 302 | 临时重定向     |
| `HttpStatus.notModified`         | 304 | 未修改         |
| `HttpStatus.badRequest`          | 400 | 错误请求       |
| `HttpStatus.unauthorized`        | 401 | 未授权         |
| `HttpStatus.forbidden`           | 403 | 禁止访问       |
| `HttpStatus.notFound`            | 404 | 未找到         |
| `HttpStatus.methodNotAllowed`    | 405 | 方法不允许     |
| `HttpStatus.internalServerError` | 500 | 服务器内部错误 |
| `HttpStatus.badGateway`          | 502 | 网关错误       |
| `HttpStatus.serviceUnavailable`  | 503 | 服务不可用     |

### 9.3 文件模式

| 模式                       | 说明         |
| -------------------------- | ------------ |
| `FileMode.read`            | 只读         |
| `FileMode.write`           | 写入（覆盖） |
| `FileMode.append`          | 追加         |
| `FileMode.writeOnly`       | 只写         |
| `FileMode.writeOnlyAppend` | 只追加       |

### 9.4 常用命令速查

```dart
// 文件操作
await File('path').readAsString();
await File('path').writeAsString('content');
await File('path').copy('newPath');
await File('path').rename('newPath');
await File('path').delete();
await File('path').exists();

// 目录操作
await Directory('path').create(recursive: true);
await Directory('path').list(recursive: true).forEach(print);
await Directory('path').delete(recursive: true);

// HTTP 服务器
final server = await HttpServer.bind('localhost', 8080);
await for (final request in server) {
  request.response.write('Hello');
  request.response.close();
}

// HTTP 客户端
final client = HttpClient();
final request = await client.getUrl(Uri.parse('https://example.com'));
final response = await request.close();
final body = await utf8.decoder.bind(response).join();

// 进程
final result = await Process.run('ls', ['-la']);
print(result.stdout);

// 平台信息
print(Platform.operatingSystem);
print(Platform.environment['PATH']);
```

---

## 总结

dart:io 是 Dart 生态系统中最重要的核心库之一，它提供了：

1. **文件系统操作**：`File`、`Directory`、`Link` 类提供了完整的文件系统访问能力

2. **HTTP 通信**：`HttpServer` 和 `HttpClient` 支持构建 Web 服务和客户端

3. **Socket 通信**：`Socket` 和 `ServerSocket` 支持 TCP/UDP 网络编程

4. **WebSocket**：支持实时双向通信

5. **进程管理**：`Process` 类支持运行外部命令

6. **平台信息**：`Platform` 类提供系统和环境信息

掌握 dart:io，你将能够构建功能完整的命令行工具、Web 服务器、网络应用和系统管理脚本。
ment['PATH']);

```

---

## 总结

dart:io 是 Dart 生态系统中最重要的核心库之一，它提供了：

1. **文件系统操作**：`File`、`Directory`、`Link` 类提供了完整的文件系统访问能力

2. **标准 I/O**：`stdin`、`stdout`、`stderr` 支持命令行交互

3. **HTTP 通信**：`HttpServer` 和 `HttpClient` 支持构建 Web 服务和客户端

4. **Socket 通信**：`Socket` 和 `ServerSocket` 支持 TCP/UDP 网络编程

5. **WebSocket**：支持实时双向通信

6. **进程管理**：`Process` 类支持运行外部命令

7. **平台信息**：`Platform` 类提供系统和环境信息

掌握 dart:io，你将能够构建功能完整的命令行工具、Web 服务器、网络应用和系统管理脚本。
```
