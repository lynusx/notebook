# io 包详解

> 本文基于 io: ^1.0.5 版本，深入解析这个为 Dart VM 的 `dart:io` 提供实用工具函数的官方包。io 包提供了进程管理、标准输入处理、路径复制、ANSI 转义码等实用功能，是构建命令行工具和服务器端应用的得力助手。

---

## 目录

1. [简介与安装](#1-简介与安装)
2. [ProcessManager 进程管理](#2-processmanager-进程管理)
3. [SharedStdIn 共享标准输入](#3-sharedstdin-共享标准输入)
4. [ExitCode 退出码](#4-exitcode-退出码)
5. [路径复制](#5-路径复制)
6. [可执行文件检查](#6-可执行文件检查)
7. [ANSI 转义码](#7-ansi-转义码)
8. [实战应用示例](#8-实战应用示例)
9. [附录：速查表](#9-附录速查表)

---

## 1. 简介与安装

### 1.1 什么是 io 包？

io 包是 Dart 团队官方维护的工具包，提供了对 `dart:io` 的补充功能，包括：

| 功能             | 说明                          |
| ---------------- | ----------------------------- |
| `ProcessManager` | 更高级的进程管理服务          |
| `SharedStdIn`    | 共享标准输入，支持多读取器    |
| `ExitCode`       | 预定义的退出码枚举            |
| `copyPath`       | 递归复制文件或目录            |
| `isExecutable`   | 检查文件是否可执行            |
| `ansi.dart`      | ANSI 转义码工具（颜色、样式） |

### 1.2 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  io: ^1.0.5
```

然后运行安装命令：

```bash
# Dart 项目
dart pub get

# Flutter 项目
flutter pub get
```

### 1.3 导入

```dart
// 导入整个包
import 'package:io/io.dart';

// 或按需导入特定模块
import 'package:io/ansi.dart';  // ANSI 转义码
```

---

## 2. ProcessManager 进程管理

`ProcessManager` 是一个高级服务，用于启动和管理进程，提供了比 `dart:io` 的 `Process` 类更便捷的 API。

### 2.1 创建 ProcessManager

```dart
import 'package:io/io.dart';

void main() async {
  // 创建 ProcessManager 实例
  final manager = ProcessManager();

  // 使用完毕后关闭
  await manager.close();
}
```

### 2.2 spawn 方法

`spawn` 方法启动一个新进程，默认将 stdin/stdout/stderr 转发到当前进程。

```dart
void spawnDemo() async {
  final manager = ProcessManager();

  // 运行 dart --version
  print('** 运行 dart --version');
  var spawn = await manager.spawn('dart', ['--version']);
  await spawn.exitCode;

  // 运行 dart format
  print('\n** 运行 dart format --output=none .');
  spawn = await manager.spawn('dart', ['format', '--output=none', '.']);
  await spawn.exitCode;

  // 运行 pub publish（需要交互输入）
  print('\n** 运行 dart pub publish --dry-run');
  spawn = await manager.spawn('dart', ['pub', 'publish', '--dry-run']);
  await spawn.exitCode;

  await manager.close();
}
```

### 2.3 spawnDetached 方法

`spawnDetached` 启动一个分离的进程，与当前进程独立运行。

```dart
void spawnDetachedDemo() async {
  final manager = ProcessManager();

  // 启动后台服务
  final spawn = await manager.spawnDetached(
    'long_running_server',
    ['--port', '8080'],
  );

  print('后台进程 PID: ${spawn.pid}');

  // 不等待进程结束，直接继续
  print('主程序继续执行...');

  await manager.close();
}
```

### 2.4 spawnShell 方法

`spawnShell` 在 shell 中执行命令，支持管道、重定向等 shell 特性。

```dart
void spawnShellDemo() async {
  final manager = ProcessManager();

  // 使用管道
  final spawn = await manager.spawnShell('cat file.txt | grep "error" | wc -l');
  await spawn.exitCode;

  // 使用重定向
  await manager.spawnShell('echo "Hello" > output.txt');

  await manager.close();
}
```

### 2.5 自定义环境变量和工作目录

```dart
void spawnWithOptions() async {
  final manager = ProcessManager();

  final spawn = await manager.spawn(
    'my_script',
    ['--env', 'production'],
    workingDirectory: '/path/to/project',
    environment: {
      'API_KEY': 'secret_key',
      'DEBUG': 'false',
    },
    includeParentEnvironment: true,
  );

  await spawn.exitCode;
  await manager.close();
}
```

### 2.6 处理进程输出

```dart
void handleProcessOutput() async {
  final manager = ProcessManager();

  final spawn = await manager.spawn('dart', ['analyze']);

  // 监听 stdout
  spawn.stdout.transform(utf8.decoder).listen((data) {
    print('输出: $data');
  });

  // 监听 stderr
  spawn.stderr.transform(utf8.decoder).listen((data) {
    print('错误: $data');
  });

  // 等待进程结束
  final exitCode = await spawn.exitCode;
  print('退出码: $exitCode');

  await manager.close();
}
```

### 2.7 向进程发送输入

```dart
void sendInputToProcess() async {
  final manager = ProcessManager();

  // 启动需要交互输入的程序
  final spawn = await manager.spawn('interactive_program', []);

  // 发送输入
  spawn.stdin.writeln('yes');
  spawn.stdin.writeln('Alice');
  spawn.stdin.writeln('password123');
  await spawn.stdin.close();

  // 等待进程结束
  await spawn.exitCode;
  await manager.close();
}
```

### 2.8 超时控制

```dart
void timeoutDemo() async {
  final manager = ProcessManager();

  final spawn = await manager.spawn('slow_command', []);

  // 设置超时
  final exitCode = await spawn.exitCode.timeout(
    Duration(seconds: 30),
    onTimeout: () {
      print('命令超时，终止进程');
      spawn.kill();
      return -1;
    },
  );

  print('退出码: $exitCode');
  await manager.close();
}
```

---

## 3. SharedStdIn 共享标准输入

`SharedStdIn` 是一个共享的标准输入流，允许多个读取器同时读取 stdin。

### 3.1 基本用法

```dart
void sharedStdInDemo() async {
  // 读取一行
  print('请输入你的名字:');
  final name = await sharedStdIn.nextLine();
  print('你好, $name!');

  // 读取多行
  print('\n请输入多行文本 (Ctrl+D 结束):');
  await for (final line in sharedStdIn.lines) {
    print('收到: $line');
  }
}
```

### 3.2 nextLine 方法

`nextLine` 异步读取一行输入，类似于 `stdin.readLineSync()` 的异步版本。

```dart
void nextLineDemo() async {
  print('用户名:');
  final username = await sharedStdIn.nextLine();

  print('密码:');
  // 隐藏密码输入
  stdin.echoMode = false;
  final password = await sharedStdIn.nextLine();
  stdin.echoMode = true;

  print('\n登录中...');
  // 验证用户名和密码
}
```

### 3.3 lines 属性

`lines` 返回一个 `Stream<String>`，可以监听所有输入行。

```dart
void linesDemo() async {
  print('开始接收输入 (输入 "exit" 退出):');

  await for (final line in sharedStdIn.lines) {
    if (line.toLowerCase() == 'exit') {
      print('再见!');
      break;
    }

    print('你输入了: $line');
  }
}
```

### 3.4 terminate 方法

`terminate` 关闭共享的 stdin，释放资源。

```dart
void terminateDemo() async {
  // 读取一些输入
  print('请输入一些内容:');
  final input = await sharedStdIn.nextLine();
  print('输入: $input');

  // 关闭 stdin
  await sharedStdIn.terminate();

  // 之后无法再读取
  // await sharedStdIn.nextLine();  // 会抛出异常
}
```

### 3.5 多读取器示例

```dart
void multipleReadersDemo() async {
  // 第一个读取器
  Future<void> reader1() async {
    await for (final line in sharedStdIn.lines) {
      print('读取器1: $line');
    }
  }

  // 第二个读取器（错误！不能同时有多个读取器）
  // await for (final line in sharedStdIn.lines) {
  //   print('读取器2: $line');
  // }

  // 正确做法：使用 Broadcast Stream
  final broadcast = sharedStdIn.lines.asBroadcastStream();

  broadcast.listen((line) => print('监听器1: $line'));
  broadcast.listen((line) => print('监听器2: $line'));
}
```

---

## 4. ExitCode 退出码

`ExitCode` 是一个枚举类，定义了常用的进程退出码。

### 4.1 预定义退出码

```dart
void exitCodeDemo() {
  // 成功
  print(ExitCode.success);  // 0

  // 一般错误
  print(ExitCode.general);  // 1

  // 用法错误（命令参数错误）
  print(ExitCode.usage);  // 64

  // 数据格式错误
  print(ExitCode.data);  // 65

  // 无法打开输入文件
  print(ExitCode.noInput);  // 66

  // 无法打开输出文件
  print(ExitCode.noUser);  // 67

  // 主机名错误
  print(ExitCode.noHost);  // 68

  // 服务不可用
  print(ExitCode.unavailable);  // 69

  // 内部软件错误
  print(ExitCode.software);  // 70

  // 系统错误
  print(ExitCode.osError);  // 71

  // 关键操作系统文件缺失
  print(ExitCode.osFile);  // 72

  // 无法创建输出文件
  print(ExitCode.cantCreate);  // 73

  // I/O 错误
  print(ExitCode.ioError);  // 74

  // 临时失败，建议重试
  print(ExitCode.tempFail);  // 75

  // 协议错误
  print(ExitCode.protocol);  // 76

  // 权限不足
  print(ExitCode.noPerm);  // 77

  // 配置错误
  print(ExitCode.config);  // 78
}
```

### 4.2 使用退出码

```dart
void useExitCode() async {
  final manager = ProcessManager();

  // 运行命令
  final spawn = await manager.spawn('dart', ['analyze']);
  final exitCode = await spawn.exitCode;

  // 检查退出码
  if (exitCode == ExitCode.success) {
    print('分析通过！');
  } else if (exitCode == ExitCode.general) {
    print('发现错误');
  } else {
    print('未知错误，退出码: $exitCode');
  }

  await manager.close();
}
```

### 4.3 设置程序退出码

```dart
void setExitCode() {
  // 设置程序退出码
  exitCode = ExitCode.success;  // 程序正常结束

  // 或根据条件设置
  final hasError = checkForErrors();
  exitCode = hasError ? ExitCode.general : ExitCode.success;
}

bool checkForErrors() {
  // 检查逻辑
  return false;
}
```

---

## 5. 路径复制

io 包提供了 `copyPath` 和 `copyPathSync` 函数，用于递归复制文件或目录（类似于 `cp -R`）。

### 5.1 copyPath 异步复制

```dart
void copyPathDemo() async {
  // 复制文件
  await copyPath('source.txt', 'dest.txt');
  print('文件复制完成');

  // 复制目录（递归）
  await copyPath('source_dir', 'dest_dir');
  print('目录复制完成');
}
```

### 5.2 copyPathSync 同步复制

```dart
void copyPathSyncDemo() {
  // 同步复制文件
  copyPathSync('source.txt', 'dest.txt');
  print('文件复制完成');

  // 同步复制目录
  copyPathSync('source_dir', 'dest_dir');
  print('目录复制完成');
}
```

### 5.3 复制选项

```dart
void copyWithOptions() async {
  // 复制并覆盖已存在的文件
  await copyPath('source.txt', 'dest.txt');

  // 复制目录，保留文件属性
  await copyPath('source_dir', 'dest_dir');

  // 处理错误
  try {
    await copyPath('nonexistent.txt', 'dest.txt');
  } on PathNotFoundException catch (e) {
    print('源文件不存在: ${e.path}');
  }
}
```

### 5.4 完整复制工具

```dart
import 'package:io/io.dart';
import 'package:path/path.dart' as p;

class FileCopier {
  /// 复制文件或目录，支持进度回调
  static Future<void> copy(
    String source,
    String destination, {
    void Function(String source, String dest)? onCopy,
  }) async {
    final sourceEntity = await _getEntity(source);

    if (sourceEntity is File) {
      await _copyFile(sourceEntity, destination, onCopy);
    } else if (sourceEntity is Directory) {
      await _copyDirectory(sourceEntity, destination, onCopy);
    } else {
      throw Exception('不支持的类型: $source');
    }
  }

  static Future<FileSystemEntity?> _getEntity(String path) async {
    if (await File(path).exists()) return File(path);
    if (await Directory(path).exists()) return Directory(path);
    return null;
  }

  static Future<void> _copyFile(
    File source,
    String destPath,
    void Function(String, String)? onCopy,
  ) async {
    await source.copy(destPath);
    onCopy?.call(source.path, destPath);
  }

  static Future<void> _copyDirectory(
    Directory source,
    String destPath,
    void Function(String, String)? onCopy,
  ) async {
    await copyPath(source.path, destPath);
    onCopy?.call(source.path, destPath);
  }
}

// 使用
void main() async {
  await FileCopier.copy(
    'source_dir',
    'dest_dir',
    onCopy: (src, dest) => print('复制: $src -> $dest'),
  );
}
```

---

## 6. 可执行文件检查

`isExecutable` 函数检查给定路径的文件是否可执行。

### 6.1 基本用法

```dart
void isExecutableDemo() async {
  // 检查文件是否可执行
  final executable = await isExecutable('/usr/bin/dart');
  print('dart 可执行: $executable');  // true

  // 检查文本文件
  final textFile = await isExecutable('README.md');
  print('README.md 可执行: $textFile');  // false

  // 检查不存在的文件
  final nonexistent = await isExecutable('nonexistent.exe');
  print('nonexistent.exe 可执行: $nonexistent');  // false
}
```

### 6.2 跨平台行为

```dart
void crossPlatformExecutable() async {
  final path = 'my_script';

  // Linux/macOS: 检查执行权限位
  // Windows: 检查文件扩展名 (.exe, .bat, .cmd, etc.)

  if (await isExecutable(path)) {
    // 运行可执行文件
    final manager = ProcessManager();
    await manager.spawn(path, []);
    await manager.close();
  } else {
    print('$path 不是可执行文件');
  }
}
```

### 6.3 查找可执行文件

```dart
Future<String?> findExecutable(String name) async {
  // 在 PATH 中查找可执行文件
  final pathEnv = Platform.environment['PATH'];
  if (pathEnv == null) return null;

  final paths = pathEnv.split(Platform.isWindows ? ';' : ':');

  for (final dir in paths) {
    final fullPath = p.join(dir, name);
    if (await isExecutable(fullPath)) {
      return fullPath;
    }

    // Windows: 尝试添加 .exe
    if (Platform.isWindows) {
      final withExe = '$fullPath.exe';
      if (await isExecutable(withExe)) {
        return withExe;
      }
    }
  }

  return null;
}

void main() async {
  final dartPath = await findExecutable('dart');
  print('dart 路径: $dartPath');
}
```

---

## 7. ANSI 转义码

io 包的 `ansi.dart` 模块提供了 ANSI 转义码工具，用于在终端中输出彩色文本和样式。

### 7.1 基本颜色

```dart
import 'package:io/ansi.dart';

void basicColors() {
  // 前景色
  print('${red}红色文本$resetAll');
  print('${green}绿色文本$resetAll');
  print('${yellow}黄色文本$resetAll');
  print('${blue}蓝色文本$resetAll');
  print('${magenta}洋红色文本$resetAll');
  print('${cyan}青色文本$resetAll');
  print('${white}白色文本$resetAll');

  // 背景色
  print('${backgroundRed}红色背景$resetAll');
  print('${backgroundGreen}绿色背景$resetAll');
  print('${backgroundYellow}黄色背景$resetAll');
  print('${backgroundBlue}蓝色背景$resetAll');

  // 组合使用
  print('${red}${backgroundWhite}红色文本白色背景$resetAll');
}
```

### 7.2 文本样式

```dart
void textStyles() {
  // 粗体
  print('${styleBold}粗体文本$resetBold');

  // 斜体
  print('${styleItalic}斜体文本$resetItalic');

  // 下划线
  print('${styleUnderline}下划线文本$resetUnderline');

  // 删除线
  print('${styleCrossedOut}删除线文本$resetCrossedOut');

  // 暗淡
  print('${styleDim}暗淡文本$resetDim');

  // 闪烁
  print('${styleBlink}闪烁文本$resetBlink');

  // 反转颜色
  print('${styleReverse}反转颜色$resetReverse');

  // 隐藏文本
  print('${styleHidden}隐藏文本$resetHidden');
}
```

### 7.3 256 色

```dart
void extendedColors() {
  // 使用 256 色
  print('${ansi256(196)}鲜红色$resetAll');
  print('${ansi256(82)}亮绿色$resetAll');
  print('${ansi256(33)}天蓝色$resetAll');

  // 背景 256 色
  print('${ansi256Background(214)}橙色背景$resetAll');

  // 灰度
  for (var i = 232; i <= 255; i++) {
    stdout.write('${ansi256Background(i)} $resetAll');
  }
  print('');
}
```

### 7.4 RGB 颜色

```dart
void rgbColors() {
  // RGB 前景色
  print('${ansiRgb(255, 0, 0)}纯红色$resetAll');
  print('${ansiRgb(0, 255, 0)}纯绿色$resetAll');
  print('${ansiRgb(0, 0, 255)}纯蓝色$resetAll');

  // RGB 背景色
  print('${ansiRgbBackground(255, 165, 0)}橙色背景$resetAll');

  // 渐变色示例
  for (var r = 0; r <= 255; r += 51) {
    stdout.write('${ansiRgbBackground(r, 0, 0)}  $resetAll');
  }
  print('');
}
```

### 7.5 光标控制

```dart
void cursorControl() {
  // 清屏
  print(clearScreen);

  // 光标上移
  print('${cursorUp}上一行');

  // 光标下移
  print('${cursorDown}下一行');

  // 光标左移
  print('${cursorLeft}左移');

  // 光标右移
  print('${cursorRight}右移');

  // 保存光标位置
  print(saveCursor);

  // 恢复光标位置
  print(restoreCursor);

  // 隐藏光标
  print(hideCursor);

  // 显示光标
  print(showCursor);
}
```

### 7.6 实用函数

```dart
void utilityFunctions() {
  // 带颜色的输出函数
  void printError(String message) {
    print('${red}${styleBold}错误:$resetAll $message');
  }

  void printSuccess(String message) {
    print('${green}${styleBold}成功:$resetAll $message');
  }

  void printWarning(String message) {
    print('${yellow}${styleBold}警告:$resetAll $message');
  }

  void printInfo(String message) {
    print('${blue}${styleBold}信息:$resetAll $message');
  }

  // 使用
  printError('文件未找到');
  printSuccess('操作完成');
  printWarning('磁盘空间不足');
  printInfo('正在处理...');
}
```

### 7.7 终端能力检测

```dart
void terminalCapability() {
  // 检查是否支持 ANSI
  if (stdout.supportsAnsiEscapes) {
    print('${green}终端支持 ANSI 转义码$resetAll');
  } else {
    print('终端不支持 ANSI 转义码');
  }

  // 检查是否有终端
  if (stdout.hasTerminal) {
    print('有终端连接');
    print('终端宽度: ${stdout.terminalColumns}');
    print('终端高度: ${stdout.terminalLines}');
  } else {
    print('没有终端（可能是管道或重定向）');
  }
}
```

### 7.8 进度条

```dart
void progressBar() async {
  final total = 100;

  for (var i = 0; i <= total; i++) {
    // 清行并回到行首
    stdout.write('${clearLine}\r');

    // 计算进度
    final percent = (i / total * 100).toInt();
    final filled = (i / total * 50).toInt();
    final empty = 50 - filled;

    // 绘制进度条
    final bar = '${green}${"█" * filled}${resetAll}${"░" * empty}';
    stdout.write('进度: [$bar] $percent%');

    // 模拟工作
    await Future.delayed(Duration(milliseconds: 50));
  }

  print('');  // 换行
  print('${green}完成!$resetAll');
}
```

### 7.9 表格输出

```dart
void printTable() {
  final data = [
    ['Name', 'Age', 'City'],
    ['Alice', '30', 'New York'],
    ['Bob', '25', 'London'],
    ['Charlie', '35', 'Tokyo'],
  ];

  // 表头
  print('${styleBold}${backgroundBlue}${white}${data[0][0].padRight(10)} '
      '${data[0][1].padRight(5)} ${data[0][2].padRight(10)}$resetAll');

  // 分隔线
  print('${styleDim}${"-" * 28}$resetAll');

  // 数据行
  for (var i = 1; i < data.length; i++) {
    print('${data[i][0].padRight(10)} ${data[i][1].padRight(5)} ${data[i][2].padRight(10)}');
  }
}
```

---

## 8. 实战应用示例

### 8.1 命令行工具框架

```dart
import 'package:io/io.dart';
import 'package:args/args.dart';

class CliTool {
  final String name;
  final String description;

  CliTool({required this.name, required this.description});

  Future<int> run(List<String> args) async {
    final parser = ArgParser()
      ..addFlag('help', abbr: 'h', help: '显示帮助信息')
      ..addFlag('version', abbr: 'v', help: '显示版本');

    final results = parser.parse(args);

    if (results['help'] as bool) {
      _printHelp(parser);
      return ExitCode.success;
    }

    if (results['version'] as bool) {
      print('$name v1.0.0');
      return ExitCode.success;
    }

    try {
      await _execute(results);
      return ExitCode.success;
    } catch (e) {
      print('${red}错误:$resetAll $e');
      return ExitCode.general;
    }
  }

  void _printHelp(ArgParser parser) {
    print('${styleBold}$name$resetAll - $description');
    print('');
    print('用法: $name [选项]');
    print('');
    print(parser.usage);
  }

  Future<void> _execute(ArgResults results) async {
    // 子类实现
  }
}
```

### 8.2 构建工具

```dart
import 'package:io/io.dart';

class BuildTool {
  final ProcessManager _manager = ProcessManager();

  Future<int> build() async {
    print('${blue}开始构建...$resetAll');

    // 获取依赖
    print('${yellow}获取依赖...$resetAll');
    var result = await _runCommand('dart', ['pub', 'get']);
    if (result != ExitCode.success) return result;

    // 格式化代码
    print('${yellow}格式化代码...$resetAll');
    result = await _runCommand('dart', ['format', '.']);
    if (result != ExitCode.success) return result;

    // 分析代码
    print('${yellow}分析代码...$resetAll');
    result = await _runCommand('dart', ['analyze']);
    if (result != ExitCode.success) return result;

    // 运行测试
    print('${yellow}运行测试...$resetAll');
    result = await _runCommand('dart', ['test']);
    if (result != ExitCode.success) return result;

    // 编译
    print('${yellow}编译...$resetAll');
    result = await _runCommand('dart', ['compile', 'exe', 'bin/main.dart']);
    if (result != ExitCode.success) return result;

    print('${green}构建成功!$resetAll');
    return ExitCode.success;
  }

  Future<int> _runCommand(String executable, List<String> args) async {
    final spawn = await _manager.spawn(executable, args);
    return await spawn.exitCode;
  }

  Future<void> close() async {
    await _manager.close();
  }
}
```

### 8.3 文件监控工具

```dart
import 'package:io/io.dart';

class FileWatcher {
  final String path;
  final void Function(String) onChange;

  FileWatcher({required this.path, required this.onChange});

  void start() {
    final dir = Directory(path);

    print('${blue}监控目录: $path$resetAll');
    print('${styleDim}按 Ctrl+C 停止$resetAll');

    dir.watch(recursive: true).listen((event) {
      final type = event.type;
      final path = event.path;

      final typeStr = switch (type) {
        FileSystemEvent.create => '${green}创建$resetAll',
        FileSystemEvent.modify => '${yellow}修改$resetAll',
        FileSystemEvent.delete => '${red}删除$resetAll',
        FileSystemEvent.move => '${blue}移动$resetAll',
        _ => '未知',
      };

      print('[$typeStr] $path');
      onChange(path);
    });
  }
}

// 使用
void main() {
  final watcher = FileWatcher(
    path: './lib',
    onChange: (path) {
      print('${yellow}文件变化，重新构建...$resetAll');
      // 触发重新构建
    },
  );

  watcher.start();
}
```

### 8.4 交互式 CLI

```dart
import 'package:io/io.dart';

class InteractiveCli {
  final ProcessManager _manager = ProcessManager();

  Future<void> run() async {
    print('${styleBold}${green}欢迎使用交互式 CLI$resetAll');
    print('${styleDim}输入 "help" 获取帮助，输入 "exit" 退出$resetAll');
    print('');

    while (true) {
      stdout.write('${blue}>$resetAll ');
      final input = await sharedStdIn.nextLine();

      if (input == null || input.isEmpty) continue;

      final parts = input.split(' ');
      final command = parts[0];
      final args = parts.sublist(1);

      switch (command) {
        case 'exit':
        case 'quit':
          print('${green}再见!$resetAll');
          await _manager.close();
          return;

        case 'help':
          _printHelp();
          break;

        case 'echo':
          print(args.join(' '));
          break;

        case 'pwd':
          print(Directory.current.path);
          break;

        case 'ls':
          await _ls(args);
          break;

        case 'cat':
          await _cat(args);
          break;

        case 'run':
          if (args.isNotEmpty) {
            await _run(args);
          } else {
            print('${red}用法: run <命令>$resetAll');
          }
          break;

        default:
          print('${red}未知命令: $command$resetAll');
          print('输入 "help" 获取帮助');
      }
    }
  }

  void _printHelp() {
    print('''
${styleBold}可用命令:$resetAll
  help          显示帮助
  echo <文本>   输出文本
  pwd           显示当前目录
  ls [路径]     列出目录内容
  cat <文件>    显示文件内容
  run <命令>    运行系统命令
  exit/quit     退出
''');
  }

  Future<void> _ls(List<String> args) async {
    final path = args.isNotEmpty ? args[0] : '.';
    final dir = Directory(path);

    try {
      await for (final entity in dir.list()) {
        final name = entity.path.split('/').last;
        if (entity is Directory) {
          print('${blue}$name/$resetAll');
        } else if (await isExecutable(entity.path)) {
          print('${green}$name*$resetAll');
        } else {
          print(name);
        }
      }
    } catch (e) {
      print('${red}错误: $e$resetAll');
    }
  }

  Future<void> _cat(List<String> args) async {
    if (args.isEmpty) {
      print('${red}用法: cat <文件>$resetAll');
      return;
    }

    try {
      final file = File(args[0]);
      final content = await file.readAsString();
      print(content);
    } catch (e) {
      print('${red}错误: $e$resetAll');
    }
  }

  Future<void> _run(List<String> args) async {
    final spawn = await _manager.spawn(args[0], args.sublist(1));
    await spawn.exitCode;
  }
}

void main() async {
  final cli = InteractiveCli();
  await cli.run();
}
```

### 8.5 日志记录器

```dart
import 'package:io/io.dart';

enum LogLevel { debug, info, warning, error }

class Logger {
  final LogLevel minLevel;

  Logger({this.minLevel = LogLevel.debug});

  void debug(String message) => _log(LogLevel.debug, message);
  void info(String message) => _log(LogLevel.info, message);
  void warning(String message) => _log(LogLevel.warning, message);
  void error(String message) => _log(LogLevel.error, message);

  void _log(LogLevel level, String message) {
    if (level.index < minLevel.index) return;

    final (prefix, color) = switch (level) {
      LogLevel.debug => ('DEBUG', styleDim),
      LogLevel.info => ('INFO', blue),
      LogLevel.warning => ('WARN', yellow),
      LogLevel.error => ('ERROR', red),
    };

    final timestamp = DateTime.now().toIso8601String();
    print('$color[$prefix]$resetAll $timestamp: $message');
  }
}

// 使用
void main() {
  final logger = Logger(minLevel: LogLevel.info);

  logger.debug('调试信息');  // 不会输出
  logger.info('应用启动');
  logger.warning('磁盘空间不足');
  logger.error('连接失败');
}
```

---

## 9. 附录：速查表

### 9.1 ProcessManager 方法

| 方法                              | 说明                               |
| --------------------------------- | ---------------------------------- |
| `spawn(executable, args)`         | 启动进程，转发 stdin/stdout/stderr |
| `spawnDetached(executable, args)` | 启动分离的进程                     |
| `spawnShell(command)`             | 在 shell 中执行命令                |
| `close()`                         | 关闭 ProcessManager                |

### 9.2 SharedStdIn 方法

| 方法/属性     | 说明             |
| ------------- | ---------------- |
| `nextLine`    | 异步读取一行输入 |
| `lines`       | 输入行流         |
| `terminate()` | 关闭 stdin       |

### 9.3 ExitCode 枚举

| 常量          | 值  | 说明         |
| ------------- | --- | ------------ |
| `success`     | 0   | 成功         |
| `general`     | 1   | 一般错误     |
| `usage`       | 64  | 用法错误     |
| `data`        | 65  | 数据格式错误 |
| `noInput`     | 66  | 无法打开输入 |
| `noUser`      | 67  | 用户不存在   |
| `noHost`      | 68  | 主机名错误   |
| `unavailable` | 69  | 服务不可用   |
| `software`    | 70  | 内部软件错误 |
| `osError`     | 71  | 系统错误     |
| `osFile`      | 72  | 关键文件缺失 |
| `cantCreate`  | 73  | 无法创建输出 |
| `ioError`     | 74  | I/O 错误     |
| `tempFail`    | 75  | 临时失败     |
| `protocol`    | 76  | 协议错误     |
| `noPerm`      | 77  | 权限不足     |
| `config`      | 78  | 配置错误     |

### 9.4 ANSI 转义码

| 常量                                                  | 效果          |
| ----------------------------------------------------- | ------------- |
| `red`, `green`, `blue`, etc.                          | 前景色        |
| `backgroundRed`, `backgroundGreen`, etc.              | 背景色        |
| `styleBold`, `styleItalic`, `styleUnderline`          | 文本样式      |
| `resetAll`, `resetBold`, `resetColor`                 | 重置          |
| `clearScreen`, `clearLine`                            | 清屏/清行     |
| `cursorUp`, `cursorDown`, `cursorLeft`, `cursorRight` | 光标移动      |
| `saveCursor`, `restoreCursor`                         | 保存/恢复光标 |
| `hideCursor`, `showCursor`                            | 隐藏/显示光标 |

### 9.5 常用函数

```dart
// 路径复制
await copyPath('source', 'dest');
copyPathSync('source', 'dest');

// 可执行检查
await isExecutable('path');

// 进程管理
final manager = ProcessManager();
final spawn = await manager.spawn('cmd', ['arg']);
await spawn.exitCode;
await manager.close();

// 标准输入
final line = await sharedStdIn.nextLine();
await for (final line in sharedStdIn.lines) { }
await sharedStdIn.terminate();

// ANSI
print('${red}红色${resetAll}');
print('${green}${styleBold}粗体绿色${resetAll}');
```

---

## 总结

io 包是 Dart 官方提供的 `dart:io` 补充工具包，主要提供了以下功能：

1. **ProcessManager** - 更高级的进程管理，简化了进程启动和通信

2. **SharedStdIn** - 共享标准输入，支持异步读取和流式处理

3. **ExitCode** - 预定义的退出码枚举，符合 Unix 标准

4. **copyPath** - 递归复制文件或目录，类似 `cp -R`

5. **isExecutable** - 跨平台检查文件是否可执行

6. **ANSI 转义码** - 丰富的终端颜色和样式控制

io 包特别适合构建命令行工具、构建脚本、开发工具和服务器端应用，是 Dart 生态中不可或缺的实用工具包。
