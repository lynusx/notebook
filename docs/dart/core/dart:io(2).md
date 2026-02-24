# Dart 核心库 dart:io 详解

> 本文基于 Dart SDK 3.x 版本，深入解析 dart:io 核心库。该库提供了文件系统、网络通信、HTTP 服务器/客户端、进程管理、标准输入输出等 I/O 功能，是构建命令行应用、服务器端应用和 Flutter 桌面/移动应用的基石。

**重要提示**：dart:io 库不能在浏览器环境中使用，仅支持以下平台：

- 服务器端 Dart 应用
- 命令行脚本
- Flutter 移动应用（iOS/Android）
- Flutter 桌面应用（Windows/macOS/Linux）

---

## 目录

1. [简介与导入](#1-简介与导入)
2. [文件系统操作](#2-文件系统操作)
3. [标准输入输出](#3-标准输入输出)
4. [HTTP 服务器](#4-http-服务器)
5. [HTTP 客户端](#5-http-客户端)
6. [Socket 通信](#6-socket-通信)
7. [WebSocket](#7-websocket)
8. [进程管理](#8-进程管理)
9. [平台信息](#9-平台信息)
10. [异常处理](#10-异常处理)
11. [实战应用示例](#11-实战应用示例)
12. [附录：速查表](#12-附录速查表)

---

## 1. 简介与导入

### 1.1 什么是 dart:io？

dart:io 是 Dart SDK 的核心库之一，提供了丰富的 I/O 功能，包括：

| 功能类别  | 主要类                                                    |
| --------- | --------------------------------------------------------- |
| 文件系统  | `File`, `Directory`, `Link`, `FileSystemEntity`           |
| 标准 I/O  | `stdin`, `stdout`, `stderr`                               |
| HTTP      | `HttpServer`, `HttpClient`, `HttpRequest`, `HttpResponse` |
| Socket    | `Socket`, `ServerSocket`                                  |
| WebSocket | `WebSocket`, `WebSocketTransformer`                       |
| 进程      | `Process`, `ProcessResult`                                |
| 平台      | `Platform`                                                |

### 1.2 导入库

```dart
import 'dart:io';
```

### 1.3 核心概念

dart:io 中的大多数 I/O 操作都是**异步**的，使用 `Future` 或 `Stream` 来处理：

```dart
// 异步读取文件
Future<void> readFileAsync() async {
  final file = File('data.txt');
  final contents = await file.readAsString();
  print(contents);
}

// 使用 Stream 处理大文件
void readLargeFile() {
  final file = File('large_file.txt');
  file.openRead()
    .transform(utf8.decoder)
    .transform(LineSplitter())
    .listen((line) => print(line));
}
```

---

## 2. 文件系统操作

### 2.1 File 类

`File` 类用于操作文件系统中的文件。

#### 创建和检查文件

```dart
void fileBasics() async {
  final file = File('example.txt');

  // 检查文件是否存在
  if (await file.exists()) {
    print('文件存在');
  }

  // 同步检查
  if (file.existsSync()) {
    print('文件存在（同步）');
  }

  // 获取文件信息
  final stat = await file.stat();
  print('大小: ${stat.size} 字节');
  print('修改时间: ${stat.modified}');
  print('类型: ${stat.type}');  // file, directory, link, notFound
}
```

#### 读取文件

```dart
void readFile() async {
  final file = File('data.txt');

  // 方式1：读取为字符串
  final stringContent = await file.readAsString();
  print(stringContent);

  // 方式2：读取为字符串（指定编码）
  final utf8Content = await file.readAsString(encoding: utf8);

  // 方式3：读取为字节列表
  final bytes = await file.readAsBytes();
  print('字节数: ${bytes.length}');

  // 方式4：读取为行列表
  final lines = await file.readAsLines();
  for (var line in lines) {
    print(line);
  }

  // 方式5：使用 Stream 读取大文件
  await for (final chunk in file.openRead()) {
    print('读取 ${chunk.length} 字节');
  }
}
```

#### 写入文件

```dart
void writeFile() async {
  final file = File('output.txt');

  // 方式1：写入字符串（覆盖）
  await file.writeAsString('Hello, World!');

  // 方式2：写入字符串（追加）
  await file.writeAsString('\nNew line', mode: FileMode.append);

  // 方式3：写入字节
  await file.writeAsBytes([72, 101, 108, 108, 111]);  // "Hello"

  // 方式4：使用 IOSink 流式写入
  final sink = file.openWrite();
  sink.write('Line 1\n');
  sink.write('Line 2\n');
  sink.writeln('Line 3');
  await sink.close();
}
```

#### 文件操作

```dart
void fileOperations() async {
  final file = File('old_name.txt');

  // 重命名文件
  await file.rename('new_name.txt');

  // 复制文件
  await file.copy('backup.txt');

  // 删除文件
  await file.delete();

  // 安全删除（不存在不报错）
  await file.delete(recursive: false);

  // 获取绝对路径
  final absolutePath = await file.absolute.path;
  print(absolutePath);

  // 获取父目录
  final parent = file.parent;
  print(parent.path);
}
```

### 2.2 Directory 类

`Directory` 类用于操作目录。

```dart
void directoryOperations() async {
  final dir = Directory('my_folder');

  // 创建目录
  await dir.create();

  // 递归创建（创建所有父目录）
  await dir.create(recursive: true);

  // 检查目录是否存在
  if (await dir.exists()) {
    print('目录存在');
  }

  // 列出目录内容
  await for (final entity in dir.list()) {
    print('${entity.path} - ${entity.runtimeType}');
  }

  // 递归列出
  await for (final entity in dir.list(recursive: true)) {
    print(entity.path);
  }

  // 只列出文件
  await for (final entity in dir.list()) {
    if (entity is File) {
      print('文件: ${entity.path}');
    }
  }

  // 重命名目录
  await dir.rename('new_folder');

  // 删除目录（必须为空）
  await dir.delete();

  // 递归删除
  await dir.delete(recursive: true);
}
```

### 2.3 FileSystemEntity 类

`FileSystemEntity` 是 `File`、`Directory` 和 `Link` 的基类，提供了通用的文件系统操作方法。

```dart
void fileSystemEntity() async {
  final path = 'some/path';

  // 判断路径类型
  final type = await FileSystemEntity.type(path);
  switch (type) {
    case FileSystemEntityType.file:
      print('是文件');
    case FileSystemEntityType.directory:
      print('是目录');
    case FileSystemEntityType.link:
      print('是链接');
    case FileSystemEntityType.notFound:
      print('不存在');
  }

  // 检查是否为目录
  if (await FileSystemEntity.isDirectory(path)) {
    print('是目录');
  }

  // 检查是否为文件
  if (await FileSystemEntity.isFile(path)) {
    print('是文件');
  }

  // 检查是否为链接
  if (await FileSystemEntity.isLink(path)) {
    print('是链接');
  }

  // 检查两个路径是否指向同一实体
  final same = await FileSystemEntity.identical('file1.txt', 'file2.txt');
  print('是否相同: $same');
}
```

### 2.4 Link 类

`Link` 类用于创建和管理符号链接。

```dart
void linkOperations() async {
  final link = Link('shortcut.txt');

  // 创建链接
  await link.create('target.txt');

  // 获取链接目标
  final target = await link.target();
  print('目标: $target');

  // 更新链接目标
  await link.update('new_target.txt');

  // 删除链接
  await link.delete();
}
```

### 2.5 文件系统监听

```dart
void watchFileSystem() {
  final dir = Directory('watched_folder');

  // 监听目录变化
  final subscription = dir.watch().listen((event) {
    print('事件类型: ${event.type}');
    print('路径: ${event.path}');

    switch (event.type) {
      case FileSystemEvent.create:
        print('创建了文件/目录');
      case FileSystemEvent.modify:
        print('修改了文件/目录');
      case FileSystemEvent.delete:
        print('删除了文件/目录');
      case FileSystemEvent.move:
        print('移动了文件/目录');
    }
  });

  // 取消监听
  // subscription.cancel();
}
```

---

## 3. 标准输入输出

### 3.1 stdout（标准输出）

```dart
void stdoutDemo() {
  // 写入字符串（不换行）
  stdout.write('Hello');

  // 写入字符串（换行）
  stdout.writeln('World');

  // 写入多个对象
  stdout.writeAll(['A', 'B', 'C'], ', ');  // A, B, C

  // 写入字符编码
  stdout.writeCharCode(65);  // A

  // 添加换行
  stdout.writeln();

  // 获取终端宽度
  final hasTerminal = stdout.hasTerminal;
  if (hasTerminal) {
    print('终端宽度: ${stdout.terminalColumns}');
    print('终端高度: ${stdout.terminalLines}');
  }

  // 检查是否支持 ANSI 转义码
  if (stdout.supportsAnsiEscapes) {
    stdout.write('\x1B[31m红色文本\x1B[0m');
  }
}
```

### 3.2 stderr（标准错误）

```dart
void stderrDemo() {
  // 写入错误信息
  stderr.writeln('错误: 文件未找到');

  // 与 stdout 用法相同
  stderr.write('警告: ');
  stderr.writeln('内存不足');
}
```

### 3.3 stdin（标准输入）

```dart
void stdinDemo() async {
  // 方式1：读取一行（同步）
  stdout.write('请输入你的名字: ');
  final name = stdin.readLineSync();
  print('你好, $name!');

  // 方式2：读取一行（指定编码）
  final input = stdin.readLineSync(encoding: utf8);

  // 方式3：异步读取所有行
  await stdin
    .transform(utf8.decoder)
    .transform(LineSplitter())
    .forEach((line) {
      print('收到: $line');
    });

  // 方式4：检查是否有终端输入
  if (stdin.hasTerminal) {
    print('从终端读取');
  } else {
    print('从管道读取');
  }

  // 方式5：逐字符读取（带回显控制）
  stdin.echoMode = false;  // 关闭回显（用于密码输入）
  stdin.lineMode = false;  // 逐字符模式

  await for (final data in stdin) {
    final char = String.fromCharCodes(data);
    if (char == '\n') break;
    stdout.write('*');
  }

  stdin.echoMode = true;
  stdin.lineMode = true;
}
```

### 3.4 退出码

```dart
void exitCodeDemo() {
  // 设置退出码（程序结束时返回）
  exitCode = 0;  // 成功

  // 常见退出码
  // 0 - 成功
  // 1 - 警告
  // 2 - 错误

  // 立即退出程序
  // exit(0);
}
```

---

## 4. HTTP 服务器

### 4.1 创建基本 HTTP 服务器

```dart
import 'dart:io';

void main() async {
  // 创建服务器
  final server = await HttpServer.bind(
    InternetAddress.loopbackIPv4,  // 127.0.0.1
    8080,                           // 端口
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

  // 写入响应
  response.write('Hello, World!');

  // 关闭响应
  response.close();
}
```

### 4.2 处理不同路由

```dart
void handleRequest(HttpRequest request) async {
  final response = request.response;
  final path = request.uri.path;
  final method = request.method;

  print('$method ${request.uri}');

  switch (path) {
    case '/':
      response.statusCode = HttpStatus.ok;
      response.write('首页');

    case '/api/users':
      if (method == 'GET') {
        response.headers.contentType = ContentType.json;
        response.write('{"users": []}');
      } else if (method == 'POST') {
        // 读取请求体
        final body = await utf8.decoder.bind(request).join();
        print('收到数据: $body');
        response.statusCode = HttpStatus.created;
      }

    case '/redirect':
      await response.redirect(Uri.parse('/'));
      return;

    default:
      response.statusCode = HttpStatus.notFound;
      response.write('404 Not Found');
  }

  response.close();
}
```

### 4.3 设置响应头

```dart
void handleRequest(HttpRequest request) {
  final response = request.response;

  // 设置内容类型
  response.headers.contentType = ContentType.json;
  // 或
  response.headers.set(HttpHeaders.contentTypeHeader, 'application/json');

  // 设置自定义头
  response.headers.set('X-Custom-Header', 'MyValue');

  // 添加多个值
  response.headers.add('X-Items', 'item1');
  response.headers.add('X-Items', 'item2');

  // 设置 Cookie
  response.cookies.add(Cookie('session', 'abc123')
    ..expires = DateTime.now().add(Duration(hours: 1))
    ..httpOnly = true);

  // 移除头
  response.headers.remove('X-Remove-This');

  response.write('{"message": "OK"}');
  response.close();
}
```

### 4.4 处理 POST 请求体

```dart
void handlePost(HttpRequest request) async {
  final contentType = request.headers.contentType;
  final contentLength = request.contentLength;

  print('内容类型: $contentType');
  print('内容长度: $contentLength');

  // 读取请求体
  final body = await utf8.decoder.bind(request).join();
  print('请求体: $body');

  // 解析 JSON
  try {
    final data = jsonDecode(body);
    print('解析成功: $data');
  } catch (e) {
    print('解析失败: $e');
  }

  request.response
    ..statusCode = HttpStatus.ok
    ..write('Received')
    ..close();
}
```

### 4.5 静态文件服务

```dart
Future<void> serveStaticFile(HttpRequest request, String filePath) async {
  final file = File(filePath);

  if (await file.exists()) {
    final ext = filePath.split('.').last;
    final contentType = _getContentType(ext);

    request.response
      ..headers.contentType = contentType
      ..headers.set(HttpHeaders.contentLengthHeader, await file.length())
      ..addStream(file.openRead())
      .then((_) => request.response.close());
  } else {
    request.response
      ..statusCode = HttpStatus.notFound
      ..write('File not found')
      ..close();
  }
}

ContentType _getContentType(String ext) {
  return switch (ext) {
    'html' => ContentType.html,
    'css' => ContentType('text', 'css'),
    'js' => ContentType('application', 'javascript'),
    'json' => ContentType.json,
    'png' => ContentType('image', 'png'),
    'jpg' || 'jpeg' => ContentType('image', 'jpeg'),
    _ => ContentType.text,
  };
}
```

### 4.6 HTTPS 服务器

```dart
Future<void> startHttpsServer() async {
  // 读取证书和私钥
  final context = SecurityContext()
    ..useCertificateChain('server.crt')
    ..usePrivateKey('server.key');

  final server = await HttpServer.bindSecure(
    InternetAddress.anyIPv4,
    443,
    context,
  );

  print('HTTPS 服务器运行在端口 443');

  await for (final request in server) {
    request.response
      ..write('Secure connection')
      ..close();
  }
}
```

---

## 5. HTTP 客户端

### 5.1 基本 GET 请求

```dart
void httpGet() async {
  final client = HttpClient();

  try {
    final request = await client.getUrl(Uri.parse('https://api.example.com/data'));
    final response = await request.close();

    print('状态码: ${response.statusCode}');
    print('头信息: ${response.headers}');

    final body = await utf8.decoder.bind(response).join();
    print('响应体: $body');
  } finally {
    client.close();
  }
}
```

### 5.2 POST 请求

```dart
void httpPost() async {
  final client = HttpClient();

  try {
    final request = await client.postUrl(Uri.parse('https://api.example.com/users'));

    // 设置请求头
    request.headers.contentType = ContentType.json;
    request.headers.set('Authorization', 'Bearer token123');

    // 写入请求体
    final body = jsonEncode({'name': 'Alice', 'age': 30});
    request.write(body);

    final response = await request.close();
    final responseBody = await utf8.decoder.bind(response).join();

    print('响应: $responseBody');
  } finally {
    client.close();
  }
}
```

### 5.3 处理 Cookie

```dart
void httpWithCookies() async {
  final client = HttpClient();

  // 启用 Cookie 存储
  client.cookieJar = CookieJar();

  try {
    // 第一个请求 - 服务器设置 Cookie
    final request1 = await client.getUrl(Uri.parse('https://example.com/login'));
    final response1 = await request1.close();
    await response1.drain();

    // 第二个请求 - 自动携带 Cookie
    final request2 = await client.getUrl(Uri.parse('https://example.com/profile'));
    final response2 = await request2.close();

    final body = await utf8.decoder.bind(response2).join();
    print(body);
  } finally {
    client.close();
  }
}
```

### 5.4 下载文件

```dart
Future<void> downloadFile(String url, String savePath) async {
  final client = HttpClient();

  try {
    final request = await client.getUrl(Uri.parse(url));
    final response = await request.close();

    final file = File(savePath);
    await response.pipe(file.openWrite());

    print('下载完成: $savePath');
  } finally {
    client.close();
  }
}
```

### 5.5 处理重定向

```dart
void httpWithRedirects() async {
  final client = HttpClient();
  client.autoUncompress = true;

  try {
    final request = await client.getUrl(Uri.parse('https://bit.ly/xxx'));
    request.followRedirects = true;
    request.maxRedirects = 5;

    final response = await request.close();
    print('最终 URL: ${response.redirects.last.location}');

    final body = await utf8.decoder.bind(response).join();
    print(body);
  } finally {
    client.close();
  }
}
```

### 5.6 HTTPS 请求（处理证书）

```dart
void httpsRequest() async {
  final client = HttpClient()
    ..badCertificateCallback = (X509Certificate cert, String host, int port) {
      // 验证证书，返回 true 接受，false 拒绝
      print('证书: ${cert.subject}');
      print('颁发者: ${cert.issuer}');
      print('有效期: ${cert.startValidity} - ${cert.endValidity}');

      // 生产环境应该正确验证证书！
      return host == 'trusted.example.com';
    };

  try {
    final request = await client.getUrl(Uri.parse('https://example.com'));
    final response = await request.close();

    final body = await utf8.decoder.bind(response).join();
    print(body);
  } finally {
    client.close();
  }
}
```

---

## 6. Socket 通信

### 6.1 TCP 客户端

```dart
void tcpClient() async {
  // 连接到服务器
  final socket = await Socket.connect('localhost', 8080);

  print('已连接: ${socket.remoteAddress}:${socket.remotePort}');

  // 发送数据
  socket.write('Hello, Server!');

  // 接收数据
  await for (final data in socket) {
    print('收到: ${String.fromCharCodes(data)}');
  }

  // 关闭连接
  await socket.close();
}
```

### 6.2 TCP 服务器

```dart
void tcpServer() async {
  // 创建服务器
  final server = await ServerSocket.bind('localhost', 8080);

  print('服务器运行在 ${server.address}:${server.port}');

  // 监听连接
  await for (final socket in server) {
    handleConnection(socket);
  }
}

void handleConnection(Socket socket) {
  print('客户端连接: ${socket.remoteAddress}:${socket.remotePort}');

  // 监听数据
  socket.listen(
    (data) {
      final message = String.fromCharCodes(data);
      print('收到: $message');

      // 发送响应
      socket.write('Echo: $message');
    },
    onError: (error) {
      print('错误: $error');
    },
    onDone: () {
      print('客户端断开连接');
    },
  );
}
```

### 6.3 UDP 通信

```dart
void udpSocket() async {
  // 创建 UDP socket
  final socket = await RawDatagramSocket.bind(InternetAddress.anyIPv4, 8080);

  print('UDP Socket 绑定到 ${socket.address}:${socket.port}');

  // 发送数据
  final data = utf8.encode('Hello UDP');
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

### 6.4 Unix Domain Socket

```dart
void unixSocket() async {
  // 创建 Unix Domain Socket
  final socket = await Socket.connect(
    InternetAddress('/tmp/my_socket.sock', type: InternetAddressType.unix),
    0,
  );

  socket.write('Hello via Unix Socket');

  await for (final data in socket) {
    print('收到: ${String.fromCharCodes(data)}');
  }
}
```

---

## 7. WebSocket

### 7.1 WebSocket 客户端

```dart
void websocketClient() async {
  // 连接 WebSocket
  final ws = await WebSocket.connect('wss://echo.websocket.org');

  print('WebSocket 已连接');

  // 发送消息
  ws.add('Hello, WebSocket!');

  // 发送二进制数据
  ws.add([1, 2, 3, 4, 5]);

  // 接收消息
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

### 7.2 WebSocket 服务器

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

        // 广播给所有客户端
        ws.add('Echo: $message');
      },
      onError: (error) => print('错误: $error'),
      onDone: () => print('客户端断开'),
    );
  }
}
```

### 7.3 WebSocket 广播服务器

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
          print('客户端断开，剩余 ${_clients.length} 个客户端');
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

## 8. 进程管理

### 8.1 运行外部命令

```dart
void runCommand() async {
  // 方式1：简单运行
  final result = await Process.run('ls', ['-la']);
  print('退出码: ${result.exitCode}');
  print('输出: ${result.stdout}');
  print('错误: ${result.stderr}');

  // 方式2：带环境变量和工作目录
  final result2 = await Process.run(
    'dart',
    ['--version'],
    workingDirectory: '/home/user',
    environment: {'MY_VAR': 'value'},
  );

  // 方式3：获取 Process 对象（流式输出）
  final process = await Process.start('ping', ['-c', '4', 'google.com']);

  process.stdout.transform(utf8.decoder).listen(print);
  process.stderr.transform(utf8.decoder).listen(print);

  final exitCode = await process.exitCode;
  print('退出码: $exitCode');
}
```

### 8.2 进程间通信

```dart
void processCommunication() async {
  final process = await Process.start('cat', []);

  // 向进程发送数据
  process.stdin.writeln('Hello');
  process.stdin.writeln('World');
  await process.stdin.close();

  // 读取输出
  final output = await process.stdout.transform(utf8.decoder).join();
  print('输出: $output');

  final exitCode = await process.exitCode;
  print('退出码: $exitCode');
}
```

### 8.3 杀死进程

```dart
void killProcess() async {
  final process = await Process.start('long_running_task', []);

  // 5秒后杀死进程
  await Future.delayed(Duration(seconds: 5));

  process.kill(ProcessSignal.sigterm);  // 优雅终止
  // process.kill(ProcessSignal.sigkill);  // 强制终止

  final exitCode = await process.exitCode;
  print('退出码: $exitCode');
}
```

### 8.4 守护进程

```dart
void detachProcess() async {
  final process = await Process.start(
    'my_daemon',
    [],
    mode: ProcessMode.detached,  // 分离模式
  );

  print('守护进程 PID: ${process.pid}');

  // 不等待进程结束
}
```

---

## 9. 平台信息

### 9.1 Platform 类

```dart
void platformInfo() {
  // 操作系统
  print('操作系统: ${Platform.operatingSystem}');  // linux, macos, windows, android, ios

  // 操作系统版本
  print('版本: ${Platform.operatingSystemVersion}');

  // 本地语言环境
  print('语言环境: ${Platform.localeName}');

  // 处理器架构
  print('处理器数量: ${Platform.numberOfProcessors}');

  // 路径分隔符
  print('路径分隔符: ${Platform.pathSeparator}');  // / 或 \

  // 换行符
  print('换行符: ${Platform.lineTerminator}');  // \n 或 \r\n
}
```

### 9.2 环境变量

```dart
void environmentVariables() {
  // 获取所有环境变量
  final env = Platform.environment;
  env.forEach((key, value) {
    print('$key=$value');
  });

  // 获取特定环境变量
  final path = Platform.environment['PATH'];
  final home = Platform.environment['HOME'];

  // 检查环境变量是否存在
  if (Platform.environment.containsKey('API_KEY')) {
    print('API_KEY 已设置');
  }
}
```

### 9.3 脚本路径

```dart
void scriptInfo() {
  // 当前脚本路径
  print('脚本路径: ${Platform.script}');

  // 可执行文件路径
  print('可执行文件: ${Platform.resolvedExecutable}');

  // 包配置文件路径
  print('包配置: ${Platform.packageConfig}');
}
```

### 9.4 特殊目录

```dart
void specialDirectories() {
  // 当前工作目录
  print('当前目录: ${Directory.current.path}');

  // 系统临时目录
  print('临时目录: ${Directory.systemTemp.path}');

  // 用户主目录
  final home = Platform.environment['HOME'] ??
               Platform.environment['USERPROFILE'];
  print('用户目录: $home');
}
```

---

## 10. 异常处理

### 10.1 IO 异常类型

```dart
void exceptionHandling() async {
  try {
    final file = File('nonexistent.txt');
    await file.readAsString();
  } on FileSystemException catch (e) {
    print('文件系统错误: ${e.message}');
    print('路径: ${e.path}');
    print('操作系统错误: ${e.osError}');
  }

  try {
    final socket = await Socket.connect('invalid.host', 80);
  } on SocketException catch (e) {
    print('Socket 错误: ${e.message}');
    print('地址: ${e.address}');
    print('端口: ${e.port}');
    print('操作系统错误: ${e.osError}');
  }

  try {
    final client = HttpClient();
    final request = await client.getUrl(Uri.parse('https://expired.badssl.com'));
  } on HandshakeException catch (e) {
    print('TLS 握手错误: ${e.message}');
  } on CertificateException catch (e) {
    print('证书错误: ${e.message}');
  } on TlsException catch (e) {
    print('TLS 错误: ${e.message}');
  }
}
```

### 10.2 常见错误处理

```dart
void commonErrorHandling() async {
  // 文件不存在
  try {
    await File('missing.txt').readAsString();
  } on PathNotFoundException catch (e) {
    print('文件不存在: ${e.path}');
  }

  // 权限不足
  try {
    await File('/root/secret.txt').readAsString();
  } on PathAccessException catch (e) {
    print('权限不足: ${e.path}');
  }

  // 路径已存在
  try {
    await Directory('existing_dir').create();
  } on PathExistsException catch (e) {
    print('路径已存在: ${e.path}');
  }

  // 进程错误
  try {
    await Process.run('nonexistent_command', []);
  } on ProcessException catch (e) {
    print('进程错误: ${e.message}');
    print('命令: ${e.executable}');
    print('参数: ${e.arguments}');
  }
}
```

---

## 11. 实战应用示例

### 11.1 简单的静态文件服务器

```dart
import 'dart:io';
import 'dart:typed_data';

class StaticFileServer {
  final String rootPath;
  final int port;
  HttpServer? _server;

  StaticFileServer({
    required this.rootPath,
    this.port = 8080,
  });

  Future<void> start() async {
    _server = await HttpServer.bind(InternetAddress.anyIPv4, port);
    print('服务器运行在 http://localhost:$port');
    print('根目录: $rootPath');

    await for (final request in _server!) {
      _handleRequest(request);
    }
  }

  Future<void> _handleRequest(HttpRequest request) async {
    final uri = request.uri;
    var path = uri.path == '/' ? '/index.html' : uri.path;

    // 安全处理：防止目录遍历
    if (path.contains('..')) {
      _sendError(request.response, HttpStatus.forbidden, 'Forbidden');
      return;
    }

    final filePath = '$rootPath$path';
    final file = File(filePath);

    if (await file.exists()) {
      final ext = filePath.split('.').last.toLowerCase();
      final contentType = _getContentType(ext);

      request.response
        ..statusCode = HttpStatus.ok
        ..headers.contentType = contentType
        ..headers.set(HttpHeaders.contentLengthHeader, await file.length());

      await request.response.addStream(file.openRead());
      await request.response.close();
    } else {
      _sendError(request.response, HttpStatus.notFound, 'Not Found');
    }
  }

  void _sendError(HttpResponse response, int statusCode, String message) {
    response
      ..statusCode = statusCode
      ..write(message)
      ..close();
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
      _ => ContentType.text,
    };
  }

  Future<void> stop() async {
    await _server?.close();
  }
}

// 使用
void main() async {
  final server = StaticFileServer(rootPath: './public');
  await server.start();
}
```

### 11.2 REST API 服务器

```dart
import 'dart:convert';
import 'dart:io';

class RestApiServer {
  final Map<String, Map<String, Function>> _routes = {};
  HttpServer? _server;

  void get(String path, Function handler) {
    _routes[path] ??= {};
    _routes[path]!['GET'] = handler;
  }

  void post(String path, Function handler) {
    _routes[path] ??= {};
    _routes[path]!['POST'] = handler;
  }

  void put(String path, Function handler) {
    _routes[path] ??= {};
    _routes[path]!['PUT'] = handler;
  }

  void delete(String path, Function handler) {
    _routes[path] ??= {};
    _routes[path]!['DELETE'] = handler;
  }

  Future<void> start(int port) async {
    _server = await HttpServer.bind(InternetAddress.loopbackIPv4, port);
    print('API 服务器运行在 http://localhost:$port');

    await for (final request in _server!) {
      _handleRequest(request);
    }
  }

  Future<void> _handleRequest(HttpRequest request) async {
    final path = request.uri.path;
    final method = request.method;

    final route = _routes[path];
    final handler = route?[method];

    if (handler != null) {
      try {
        final result = await handler(request);
        _sendJson(request.response, HttpStatus.ok, result);
      } catch (e) {
        _sendJson(request.response, HttpStatus.internalServerError, {
          'error': e.toString(),
        });
      }
    } else {
      _sendJson(request.response, HttpStatus.notFound, {
        'error': 'Not Found',
      });
    }
  }

  void _sendJson(HttpResponse response, int statusCode, dynamic data) {
    response
      ..statusCode = statusCode
      ..headers.contentType = ContentType.json
      ..write(jsonEncode(data))
      ..close();
  }
}

// 使用
void main() async {
  final users = <Map<String, dynamic>>[
    {'id': '1', 'name': 'Alice'},
    {'id': '2', 'name': 'Bob'},
  ];

  final api = RestApiServer();

  api.get('/api/users', (_) => users);

  api.get('/api/users/', (HttpRequest request) {
    final id = request.uri.pathSegments.last;
    return users.firstWhere(
      (u) => u['id'] == id,
      orElse: () => throw Exception('User not found'),
    );
  });

  api.post('/api/users', (HttpRequest request) async {
    final body = await utf8.decoder.bind(request).join();
    final data = jsonDecode(body);
    users.add({'id': DateTime.now().millisecondsSinceEpoch.toString(), ...data});
    return data;
  });

  await api.start(8080);
}
```

### 11.3 文件复制工具

```dart
import 'dart:io';

class FileCopier {
  /// 复制文件或目录
  static Future<void> copy(String source, String destination) async {
    final sourceEntity = await _getEntity(source);

    if (sourceEntity is File) {
      await _copyFile(sourceEntity, destination);
    } else if (sourceEntity is Directory) {
      await _copyDirectory(sourceEntity, destination);
    } else {
      throw Exception('不支持的类型: $source');
    }
  }

  static Future<FileSystemEntity?> _getEntity(String path) async {
    if (await File(path).exists()) return File(path);
    if (await Directory(path).exists()) return Directory(path);
    return null;
  }

  static Future<void> _copyFile(File source, String destPath) async {
    await source.copy(destPath);
    print('复制文件: ${source.path} -> $destPath');
  }

  static Future<void> _copyDirectory(Directory source, String destPath) async {
    final destDir = Directory(destPath);
    await destDir.create(recursive: true);

    await for (final entity in source.list()) {
      final name = entity.path.split(Platform.pathSeparator).last;
      final newPath = '$destPath${Platform.pathSeparator}$name';

      if (entity is File) {
        await _copyFile(entity, newPath);
      } else if (entity is Directory) {
        await _copyDirectory(entity, newPath);
      }
    }

    print('复制目录: ${source.path} -> $destPath');
  }
}

// 使用
void main() async {
  await FileCopier.copy('source.txt', 'dest.txt');
  await FileCopier.copy('source_dir', 'dest_dir');
}
```

### 11.4 日志文件轮转

```dart
import 'dart:io';

class RotatingFileLogger {
  final String basePath;
  final int maxSize;
  final int maxFiles;
  IOSink? _sink;
  int _currentSize = 0;

  RotatingFileLogger({
    required this.basePath,
    this.maxSize = 10 * 1024 * 1024,  // 10MB
    this.maxFiles = 5,
  });

  Future<void> init() async {
    await _rotateIfNeeded();
    _sink = File(basePath).openWrite(mode: FileMode.append);
  }

  Future<void> log(String message) async {
    final line = '${DateTime.now()}: $message\n';
    final bytes = utf8.encode(line);

    if (_currentSize + bytes.length > maxSize) {
      await _rotate();
    }

    _sink?.add(bytes);
    _currentSize += bytes.length;
  }

  Future<void> _rotateIfNeeded() async {
    final file = File(basePath);
    if (await file.exists()) {
      _currentSize = await file.length();
      if (_currentSize > maxSize) {
        await _rotate();
      }
    }
  }

  Future<void> _rotate() async {
    await _sink?.close();

    // 删除最旧的日志
    final oldestFile = '$basePath.$maxFiles';
    final oldest = File(oldestFile);
    if (await oldest.exists()) {
      await oldest.delete();
    }

    // 移动现有日志
    for (var i = maxFiles - 1; i >= 1; i--) {
      final oldFile = File('$basePath.$i');
      final newFile = File('$basePath.${i + 1}');
      if (await oldFile.exists()) {
        await oldFile.rename(newFile.path);
      }
    }

    // 移动当前日志
    final current = File(basePath);
    if (await current.exists()) {
      await current.rename('$basePath.1');
    }

    _currentSize = 0;
    _sink = File(basePath).openWrite();
  }

  Future<void> close() async {
    await _sink?.close();
  }
}

// 使用
void main() async {
  final logger = RotatingFileLogger(basePath: 'app.log');
  await logger.init();

  for (var i = 0; i < 1000; i++) {
    await logger.log('Log message $i');
  }

  await logger.close();
}
```

### 11.5 简单的代理服务器

```dart
import 'dart:io';

class SimpleProxy {
  final String targetHost;
  final int targetPort;
  final int listenPort;
  HttpServer? _server;

  SimpleProxy({
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
      // 创建到目标服务器的请求
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

      // 获取目标响应
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
  final proxy = SimpleProxy(
    targetHost: 'jsonplaceholder.typicode.com',
    targetPort: 80,
    listenPort: 8888,
  );
  await proxy.start();
}
```

---

## 12. 附录：速查表

### 12.1 常用类对照表

| 功能         | 主要类             | 用途             |
| ------------ | ------------------ | ---------------- |
| 文件操作     | `File`             | 读写文件         |
| 目录操作     | `Directory`        | 创建/列出目录    |
| 符号链接     | `Link`             | 创建软链接       |
| 文件系统实体 | `FileSystemEntity` | 通用文件系统操作 |
| 标准输入     | `stdin`            | 读取用户输入     |
| 标准输出     | `stdout`           | 输出到控制台     |
| 标准错误     | `stderr`           | 输出错误信息     |
| HTTP 服务器  | `HttpServer`       | 创建 Web 服务器  |
| HTTP 客户端  | `HttpClient`       | 发送 HTTP 请求   |
| HTTP 请求    | `HttpRequest`      | 服务器端请求对象 |
| HTTP 响应    | `HttpResponse`     | 服务器端响应对象 |
| TCP Socket   | `Socket`           | TCP 通信         |
| TCP 服务器   | `ServerSocket`     | TCP 服务器       |
| WebSocket    | `WebSocket`        | WebSocket 通信   |
| 进程         | `Process`          | 运行外部命令     |
| 平台信息     | `Platform`         | 获取系统信息     |

### 12.2 HTTP 状态码常量

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

### 12.3 文件模式

| 模式                       | 说明         |
| -------------------------- | ------------ |
| `FileMode.read`            | 只读         |
| `FileMode.write`           | 写入（覆盖） |
| `FileMode.append`          | 追加         |
| `FileMode.writeOnly`       | 只写         |
| `FileMode.writeOnlyAppend` | 只追加       |

### 12.4 常用命令速查

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

2. **标准 I/O**：`stdin`、`stdout`、`stderr` 支持命令行交互

3. **HTTP 通信**：`HttpServer` 和 `HttpClient` 支持构建 Web 服务和客户端

4. **Socket 通信**：`Socket` 和 `ServerSocket` 支持 TCP/UDP 网络编程

5. **WebSocket**：支持实时双向通信

6. **进程管理**：`Process` 类支持运行外部命令

7. **平台信息**：`Platform` 类提供系统和环境信息

掌握 dart:io，你将能够构建功能完整的命令行工具、Web 服务器、网络应用和系统管理脚本。
