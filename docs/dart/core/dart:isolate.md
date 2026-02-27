# dart:isolate 详解与实战

## 内容简介

`dart:isolate` 是 Dart 核心库之一，提供了 Dart 的并发编程模型。与大多数语言中的线程不同，Dart 使用 Isolate（隔离体）来实现并发——每个 Isolate 拥有独立的内存空间和事件循环，通过消息传递进行通信，从而避免了传统多线程编程中的数据竞争和死锁问题。

全书共分为6章：

- **第1章 核心概念**：详细介绍 Isolate 的并发模型、与线程的区别
- **第2章 Isolate 类**：深入讲解 Isolate 的创建、控制和生命周期管理
- **第3章 端口通信**：全面介绍 SendPort 和 ReceivePort 的使用
- **第4章 简单并发模式**：介绍 Isolate.run 和 compute 等简化 API
- **第5章 高级通信模式**：介绍双向通信、长连接 Isolate 等高级用法
- **第6章 实战应用**：提供图像处理、大数据处理等实际应用案例

---

## 第1章 核心概念

### 1.1 什么是 Isolate

Isolate 是 Dart 实现并发的基本单位。每个 Dart 程序至少运行在一个 Isolate 中，通常称为主 Isolate（Main Isolate）。与操作系统线程不同，Isolate 具有以下特点：

1. **独立内存空间**：每个 Isolate 有自己的堆内存，不与其他 Isolate 共享
2. **独立事件循环**：每个 Isolate 有自己的事件循环（Event Loop）
3. **消息传递通信**：Isolate 之间通过 SendPort/ReceivePort 传递消息，而非共享内存
4. **无锁编程模型**：由于不共享内存，天然避免了数据竞争，无需锁机制

```dart
import 'dart:isolate';

void main() {
  // 获取当前 Isolate 的信息
  print('当前 Isolate: ${Isolate.current.debugName}');
  print('当前 Isolate ID: ${Service.getIsolateID(Isolate.current)}');
}
```

**Flutter 框架小知识**

**Isolate 与线程的区别**

| 特性     | Isolate                  | 线程（Thread） |
| -------- | ------------------------ | -------------- |
| 内存模型 | 独立内存，不共享         | 共享进程内存   |
| 通信方式 | 消息传递                 | 共享变量、锁   |
| 数据竞争 | 不可能发生               | 需要锁保护     |
| 创建开销 | 较大（约 30KB 基础开销） | 较小           |
| 适用场景 | CPU 密集型任务           | I/O 密集型任务 |

### 1.2 为什么需要 Isolate

在 Flutter 中，所有 UI 代码都运行在主 Isolate 中。主 Isolate 同时负责：

- 绘制 UI（60/120 FPS）
- 响应用户交互
- 执行异步任务

当执行 CPU 密集型任务时（如大量计算、图像处理、JSON 解析），会阻塞主 Isolate 的事件循环，导致 UI 卡顿：

```dart
// 阻塞主 Isolate 的示例
void main() {
  print('开始计算...');

  // 这会阻塞 UI 约 3 秒
  final result = heavyComputation();

  print('计算完成: $result');
}

int heavyComputation() {
  var sum = 0;
  for (var i = 0; i < 1000000000; i++) {
    sum += i;
  }
  return sum;
}
```

**解决方案：使用 Isolate**

```dart
import 'dart:isolate';

void main() async {
  print('开始计算...');

  // 在 Isolate 中执行，不阻塞主线程
  final result = await Isolate.run(() => heavyComputation());

  print('计算完成: $result');
}

int heavyComputation() {
  var sum = 0;
  for (var i = 0; i < 1000000000; i++) {
    sum += i;
  }
  return sum;
}
```

### 1.3 Isolate 的架构

```
┌─────────────────────────────────────────────────────────────┐
│                         主 Isolate                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Event Loop                          │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │  │
│  │  │ Microtask│→│ Future  │→│ Future  │→│  Timer  │  │  │
│  │  │ Queue   │  │  1      │  │  2      │  │         │  │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Heap Memory                         │  │
│  │  [对象1] [对象2] [对象3] ... [对象N]                   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ SendPort/ReceivePort 消息传递
                              │
┌─────────────────────────────────────────────────────────────┐
│                        子 Isolate                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Event Loop                          │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐               │  │
│  │  │ Microtask│→│ Future  │→│  计算任务 │               │  │
│  │  │ Queue   │  │         │  │         │               │  │
│  │  └─────────┘  └─────────┘  └─────────┘               │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Heap Memory                         │  │
│  │  [独立的对象空间，与主 Isolate 不共享]                  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 1.4 可发送的对象

Isolate 之间通过消息传递通信，只有特定类型的对象可以被发送：

```dart
import 'dart:isolate';
import 'dart:typed_data';

void main() async {
  // ✅ 可以发送的对象
  await Isolate.run(() {
    // 基本类型
    int a = 42;
    double b = 3.14;
    String c = 'Hello';
    bool d = true;
    Null e = null;

    // 集合类型（元素也必须可发送）
    List<int> list = [1, 2, 3];
    Map<String, int> map = {'a': 1, 'b': 2};
    Set<String> set = {'x', 'y'};

    // 类型化数据
    Uint8List typedList = Uint8List(10);
    ByteData byteData = ByteData(10);

    // SendPort
    // SendPort sendPort = ...;

    // 自定义对象（需要满足条件）
    // 必须是顶层函数或静态方法创建的
  });

  // ❌ 不能发送的对象
  // - 闭包（捕获外部变量）
  // - 包含不可发送对象的集合
  // - 打开的文件或网络连接
  // - 与 UI 相关的对象（BuildContext、Widget 等）
}
```

**可发送对象清单：**

| 类型                    | 可发送 | 说明                     |
| ----------------------- | ------ | ------------------------ |
| `null`                  | ✅     | 空值                     |
| `bool`                  | ✅     | 布尔值                   |
| `int`                   | ✅     | 整数（Web 平台有限制）   |
| `double`                | ✅     | 浮点数                   |
| `String`                | ✅     | 字符串                   |
| `List`                  | ✅     | 列表（元素也必须可发送） |
| `Map`                   | ✅     | 映射（键值都必须可发送） |
| `Set`                   | ✅     | 集合（元素也必须可发送） |
| `Uint8List` 等          | ✅     | 类型化数据列表           |
| `ByteData`              | ✅     | 字节数据                 |
| `SendPort`              | ✅     | 发送端口                 |
| `Capability`            | ✅     | 能力令牌                 |
| `TransferableTypedData` | ✅     | 可转移的类型化数据       |
| 函数                    | ⚠️     | 顶层函数或静态方法       |
| 闭包                    | ❌     | 捕获外部变量的函数       |
| 对象实例                | ⚠️     | 必须是简单对象           |

**Dart Tips 语法小贴士**

**为什么闭包不能跨 Isolate 发送？**

闭包（Closure）会隐式捕获外部变量，这些变量可能包含不可发送的对象。Dart 无法安全地将闭包及其捕获的环境序列化传递到另一个 Isolate。因此，Isolate 的入口函数必须是：

1. **顶层函数**（定义在任何类之外的函数）
2. **静态方法**（使用 `static` 修饰的方法）
3. **没有捕获外部变量的函数**

```dart
// ✅ 正确的做法：顶层函数
void isolateEntry(SendPort sendPort) {
  sendPort.send('Hello');
}

// ✅ 正确的做法：静态方法
class Worker {
  static void isolateEntry(SendPort sendPort) {
    sendPort.send('Hello');
  }
}

// ❌ 错误的做法：闭包捕获外部变量
void main() {
  final message = 'Hello';  // 外部变量

  // 这个闭包捕获了 message，不能发送到 Isolate
  await Isolate.spawn(
    (sendPort) => sendPort.send(message),  // ❌ 错误！
    receivePort.sendPort,
  );
}
```

---

## 第2章 Isolate 类

### 2.1 Isolate 类的核心方法

#### 2.1.1 Isolate.spawn

`Isolate.spawn` 是最基础的 Isolate 创建方法，用于创建与当前 Isolate 共享代码的新 Isolate。

```dart
import 'dart:isolate';

void main() async {
  // 创建 ReceivePort 用于接收消息
  final receivePort = ReceivePort();

  // 创建新 Isolate
  final isolate = await Isolate.spawn(
    isolateEntry,                    // 入口函数（必须是顶层函数或静态方法）
    receivePort.sendPort,            // 传递给新 Isolate 的消息
    debugName: 'Worker Isolate',     // 调试名称（可选）
    paused: false,                   // 是否暂停启动（可选）
    errorsAreFatal: true,            // 错误是否致命（可选）
    onExit: null,                    // 退出时通知的端口（可选）
    onError: null,                   // 错误时通知的端口（可选）
  );

  // 监听消息
  receivePort.listen((message) {
    print('收到消息: $message');
    receivePort.close();
    isolate.kill();  // 关闭 Isolate
  });
}

// 入口函数（必须是顶层函数）
void isolateEntry(SendPort sendPort) {
  sendPort.send('Hello from Isolate!');
}
```

**Isolate.spawn 参数说明：**

| 参数             | 类型               | 说明                              |
| ---------------- | ------------------ | --------------------------------- |
| `entryPoint`     | `void Function(T)` | 新 Isolate 的入口函数             |
| `message`        | `T`                | 传递给入口函数的消息              |
| `debugName`      | `String?`          | 调试名称，在日志中显示            |
| `paused`         | `bool`             | 是否暂停启动，默认 `false`        |
| `errorsAreFatal` | `bool`             | 错误是否终止 Isolate，默认 `true` |
| `onExit`         | `SendPort?`        | Isolate 退出时发送消息的端口      |
| `onError`        | `SendPort?`        | Isolate 出错时发送消息的端口      |

#### 2.1.2 Isolate.spawnUri

`Isolate.spawnUri` 用于从外部 URI 加载代码并创建 Isolate。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();

  // 从外部 Dart 文件创建 Isolate
  final isolate = await Isolate.spawnUri(
    Uri.file('worker.dart'),         // Dart 文件路径
    ['arg1', 'arg2'],                // 命令行参数
    receivePort.sendPort,            // 初始消息
    debugName: 'External Isolate',
  );

  receivePort.listen((message) {
    print('收到: $message');
    receivePort.close();
    isolate.kill();
  });
}
```

#### 2.1.3 Isolate.run

`Isolate.run` 是 Dart 2.19+ 提供的简化 API，用于执行一次性计算任务。

```dart
import 'dart:isolate';
import 'dart:convert';

Future<List<Photo>> parsePhotos(String jsonString) async {
  // 在 Isolate 中解析 JSON
  final photos = await Isolate.run<List<Photo>>(() {
    final List<dynamic> data = jsonDecode(jsonString);
    return data.map((item) => Photo.fromJson(item)).toList();
  });

  return photos;
}

class Photo {
  final String url;
  final String title;

  Photo({required this.url, required this.title});

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      url: json['url'],
      title: json['title'],
    );
  }
}
```

**Isolate.run 的特点：**

- 自动创建和销毁 Isolate
- 返回计算结果
- 异常自动传播到调用者
- 适合一次性计算任务

### 2.2 Isolate 实例方法

#### 2.2.1 kill

终止 Isolate 的执行。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final isolate = await Isolate.spawn(worker, receivePort.sendPort);

  // 5 秒后终止 Isolate
  await Future.delayed(Duration(seconds: 5));

  // 终止 Isolate
  isolate.kill(
    priority: Isolate.immediate,  // 立即终止
  );

  print('Isolate 已终止');
}

void worker(SendPort sendPort) {
  var count = 0;
  while (true) {
    sendPort.send('Count: ${count++}');
    sleep(Duration(seconds: 1));
  }
}
```

**priority 参数：**

| 值                        | 说明                       |
| ------------------------- | -------------------------- |
| `Isolate.immediate`       | 立即终止，不执行清理       |
| `Isolate.beforeNextEvent` | 在下一个事件循环迭代前终止 |

#### 2.2.2 pause 和 resume

暂停和恢复 Isolate 的执行。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final isolate = await Isolate.spawn(worker, receivePort.sendPort);

  // 暂停 Isolate
  final capability = isolate.pause();
  print('Isolate 已暂停');

  await Future.delayed(Duration(seconds: 3));

  // 恢复 Isolate
  isolate.resume(capability);
  print('Isolate 已恢复');
}

void worker(SendPort sendPort) {
  var count = 0;
  Timer.periodic(Duration(seconds: 1), (timer) {
    sendPort.send('Count: ${count++}');
  });
}
```

#### 2.2.3 addErrorListener

添加错误监听器，接收 Isolate 中的未捕获异常。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final errorPort = ReceivePort();

  final isolate = await Isolate.spawn(
    errorProneWorker,
    receivePort.sendPort,
    onError: errorPort.sendPort,  // 错误通知端口
  );

  // 监听错误
  errorPort.listen((error) {
    print('Isolate 错误: $error');
  });

  receivePort.listen((message) {
    print('收到: $message');
  });
}

void errorProneWorker(SendPort sendPort) {
  sendPort.send('开始工作');

  // 故意抛出异常
  throw Exception('出错了！');
}
```

#### 2.2.4 addOnExitListener

添加退出监听器，在 Isolate 退出时收到通知。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final exitPort = ReceivePort();

  final isolate = await Isolate.spawn(
    worker,
    receivePort.sendPort,
    onExit: exitPort.sendPort,  // 退出通知端口
  );

  // 监听退出
  exitPort.listen((message) {
    print('Isolate 已退出: $message');
    exitPort.close();
  });

  receivePort.listen((message) {
    print('收到: $message');
    receivePort.close();
    isolate.kill();  // 正常退出
  });
}

void worker(SendPort sendPort) {
  sendPort.send('Hello');
  // 工作完成后自动退出
}
```

### 2.3 Isolate 类属性

#### 2.3.1 Isolate.current

获取当前 Isolate 实例。

```dart
import 'dart:isolate';

void main() {
  final current = Isolate.current;

  print('当前 Isolate: ${current.debugName}');
  print('暂停能力: ${current.pauseCapability}');
  print('终止能力: ${current.terminateCapability}');
}
```

#### 2.3.2 debugName

获取 Isolate 的调试名称。

```dart
import 'dart:isolate';

void main() async {
  final isolate = await Isolate.spawn(
    worker,
    null,
    debugName: 'ImageProcessor',
  );

  print('Isolate 名称: ${isolate.debugName}');  // ImageProcessor
}

void worker(_) {}
```

### 2.4 Isolate.exit

`Isolate.exit` 用于立即终止当前 Isolate，并可选择性地发送最后一条消息。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();

  await Isolate.spawn(worker, receivePort.sendPort);

  final result = await receivePort.first;
  print('结果: $result');  // 结果: 5050
}

void worker(SendPort sendPort) {
  var sum = 0;
  for (var i = 1; i <= 100; i++) {
    sum += i;
  }

  // 发送结果并立即退出
  Isolate.exit(sendPort, sum);
}
```

**Isolate.exit 的优势：**

- 比正常返回更高效
- 立即释放 Isolate 资源
- 避免执行 finally 块中的清理代码

---

## 第3章 端口通信

### 3.1 ReceivePort 类

`ReceivePort` 是 Isolate 接收消息的端口，它是一个非广播的 Stream。

#### 3.1.1 创建 ReceivePort

```dart
import 'dart:isolate';

void main() {
  // 方式1：默认构造函数
  final receivePort1 = ReceivePort();

  // 方式2：带调试名称
  final receivePort2 = ReceivePort('MainPort');

  // 方式3：从 RawReceivePort 创建
  final rawPort = RawReceivePort();
  final receivePort3 = ReceivePort.fromRawReceivePort(rawPort);
}
```

#### 3.1.2 常用属性

1）**sendPort**

获取与 ReceivePort 关联的 SendPort。

```dart
import 'dart:isolate';

void main() {
  final receivePort = ReceivePort();
  final sendPort = receivePort.sendPort;

  print('SendPort: $sendPort');
}
```

2）**Stream 属性**

ReceivePort 继承自 Stream，可以使用所有 Stream 的方法。

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();

  // 监听第一个消息
  final first = await receivePort.first;
  print('第一个消息: $first');

  // 转换为广播 Stream
  final broadcast = receivePort.asBroadcastStream();

  // 转换为 Future
  final future = receivePort.toList();
}
```

#### 3.1.3 监听消息

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();

  // 方式1：使用 listen
  receivePort.listen(
    (message) {
      print('收到: $message');
    },
    onError: (error) {
      print('错误: $error');
    },
    onDone: () {
      print('端口关闭');
    },
  );

  // 方式2：使用 await for
  await for (final message in receivePort) {
    print('收到: $message');
    if (message == 'done') break;
  }

  // 方式3：使用 first
  final message = await receivePort.first;
  print('收到: $message');
}
```

#### 3.1.4 关闭 ReceivePort

```dart
import 'dart:isolate';

void main() {
  final receivePort = ReceivePort();

  // 使用完成后关闭
  receivePort.close();

  // 关闭后不能再发送消息
  // receivePort.listen(...);  // 会抛出异常
}
```

### 3.2 SendPort 类

`SendPort` 用于向 ReceivePort 发送消息。

#### 3.2.1 send 方法

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final sendPort = receivePort.sendPort;

  // 发送基本类型
  sendPort.send('Hello');
  sendPort.send(42);
  sendPort.send(3.14);
  sendPort.send(true);
  sendPort.send(null);

  // 发送集合
  sendPort.send([1, 2, 3]);
  sendPort.send({'a': 1, 'b': 2});

  // 发送类型化数据
  sendPort.send(Uint8List.fromList([1, 2, 3]));

  // 发送 SendPort（用于双向通信）
  final replyPort = ReceivePort();
  sendPort.send(replyPort.sendPort);
}
```

#### 3.2.2 SendPort 的特性

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();
  final sendPort1 = receivePort.sendPort;
  final sendPort2 = receivePort.sendPort;

  // 同一个 ReceivePort 的 SendPort 相等
  print(sendPort1 == sendPort2);  // true

  // SendPort 的 hashCode 相同
  print(sendPort1.hashCode == sendPort2.hashCode);  // true
}
```

### 3.3 RawReceivePort

`RawReceivePort` 是 ReceivePort 的低级版本，提供更直接的控制。

```dart
import 'dart:isolate';

void main() {
  // 创建 RawReceivePort
  final rawPort = RawReceivePort();

  // 设置消息处理器
  rawPort.handler = (message) {
    print('收到: $message');

    // 手动关闭
    if (message == 'done') {
      rawPort.close();
    }
  };

  // 获取 SendPort
  final sendPort = rawPort.sendPort;
  sendPort.send('Hello');
  sendPort.send('done');
}
```

**RawReceivePort 与 ReceivePort 的区别：**

| 特性       | RawReceivePort | ReceivePort               |
| ---------- | -------------- | ------------------------- |
| Stream API | ❌ 不支持      | ✅ 支持                   |
| 消息缓冲   | 手动处理       | 自动缓冲                  |
| 多监听器   | ❌ 不支持      | ✅ 通过 asBroadcastStream |
| 性能       | 更高           | 较低                      |
| 使用复杂度 | 较高           | 较低                      |

### 3.4 单向通信示例

```dart
import 'dart:isolate';

// 主 Isolate 向子 Isolate 发送任务
void main() async {
  final receivePort = ReceivePort();

  // 创建子 Isolate
  final isolate = await Isolate.spawn(
    worker,
    receivePort.sendPort,
  );

  // 接收结果
  final result = await receivePort.first;
  print('计算结果: $result');  // 计算结果: 45

  receivePort.close();
  isolate.kill();
}

// 子 Isolate 入口
void worker(SendPort sendPort) {
  // 计算 0+1+2+...+9
  var sum = 0;
  for (var i = 0; i < 10; i++) {
    sum += i;
  }

  // 发送结果
  sendPort.send(sum);
}
```

---

## 第4章 简单并发模式

### 4.1 Isolate.run

`Isolate.run` 是最简单的并发方式，适合一次性计算任务。

#### 4.1.1 基本用法

```dart
import 'dart:isolate';
import 'dart:math';

Future<int> calculatePrimes(int max) async {
  return await Isolate.run(() {
    var count = 0;
    for (var i = 2; i <= max; i++) {
      if (isPrime(i)) count++;
    }
    return count;
  });
}

bool isPrime(int n) {
  if (n < 2) return false;
  for (var i = 2; i <= sqrt(n); i++) {
    if (n % i == 0) return false;
  }
  return true;
}

void main() async {
  print('开始计算...');

  final count = await calculatePrimes(1000000);
  print('1-100万之间的素数个数: $count');
}
```

#### 4.1.2 异常处理

```dart
import 'dart:isolate';

Future<int> safeDivide(int a, int b) async {
  try {
    return await Isolate.run(() {
      if (b == 0) throw Exception('除数不能为零');
      return a ~/ b;
    });
  } catch (e) {
    print('捕获异常: $e');
    return 0;
  }
}

void main() async {
  final result1 = await safeDivide(10, 2);
  print('10 / 2 = $result1');  // 10 / 2 = 5

  final result2 = await safeDivide(10, 0);
  print('10 / 0 = $result2');  // 捕获异常: Exception: 除数不能为零
}
```

#### 4.1.3 传递参数

```dart
import 'dart:isolate';

class ImageProcessingParams {
  final List<int> pixels;
  final int width;
  final int height;
  final double brightness;

  ImageProcessingParams({
    required this.pixels,
    required this.width,
    required this.height,
    required this.brightness,
  });
}

Future<List<int>> adjustBrightness(ImageProcessingParams params) async {
  return await Isolate.run(() {
    final result = List<int>.from(params.pixels);

    for (var i = 0; i < result.length; i++) {
      result[i] = (result[i] * params.brightness).clamp(0, 255).toInt();
    }

    return result;
  });
}
```

### 4.2 compute 函数

`compute` 是 Flutter 提供的便捷函数，功能与 `Isolate.run` 类似。

```dart
import 'package:flutter/foundation.dart';

// compute 自动处理 Web 平台的兼容性
// 在 Web 上，compute 在主线程执行
// 在移动端，compute 创建 Isolate

Future<List<Photo>> parsePhotosInBackground(String jsonString) async {
  return await compute<String, List<Photo>>(
    _parsePhotos,      // 回调函数
    jsonString,        // 参数
  );
}

List<Photo> _parsePhotos(String jsonString) {
  final List<dynamic> data = jsonDecode(jsonString);
  return data.map((item) => Photo.fromJson(item)).toList();
}

class Photo {
  final String url;
  final String title;

  Photo({required this.url, required this.title});

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      url: json['url'],
      title: json['title'],
    );
  }
}
```

**compute 与 Isolate.run 的对比：**

| 特性         | compute               | Isolate.run         |
| ------------ | --------------------- | ------------------- |
| 来源         | Flutter (foundation)  | Dart (dart:isolate) |
| Web 支持     | ✅ 兼容（主线程执行） | ❌ 不支持           |
| 参数传递     | 直接传递              | 通过闭包或对象      |
| 返回值       | 直接返回              | 直接返回            |
| 推荐使用场景 | Flutter 应用          | 纯 Dart 应用        |

### 4.3 批量处理

```dart
import 'dart:isolate';
import 'dart:math';

// 并行处理大量数据
Future<List<int>> parallelProcess(
  List<int> data,
  int Function(int) processor,
) async {
  // 将数据分成 4 份
  final chunkSize = (data.length / 4).ceil();
  final chunks = <List<int>>[];

  for (var i = 0; i < data.length; i += chunkSize) {
    chunks.add(data.sublist(
      i,
      min(i + chunkSize, data.length),
    ));
  }

  // 并行处理每个分块
  final futures = chunks.map((chunk) => Isolate.run(
    () => chunk.map(processor).toList(),
  ));

  // 等待所有结果并合并
  final results = await Future.wait(futures);
  return results.expand((x) => x).toList();
}

void main() async {
  final data = List<int>.generate(1000000, (i) => i);

  final result = await parallelProcess(data, (x) => x * x);
  print('处理完成，结果数量: ${result.length}');
}
```

---

## 第5章 高级通信模式

### 5.1 双向通信

```dart
import 'dart:isolate';

void main() async {
  // 主 Isolate 的接收端口
  final mainReceivePort = ReceivePort();

  // 创建子 Isolate
  final isolate = await Isolate.spawn(
    worker,
    mainReceivePort.sendPort,
  );

  // 获取子 Isolate 的 SendPort
  final workerSendPort = await mainReceivePort.first as SendPort;

  // 创建回复端口
  final replyPort = ReceivePort();

  // 发送消息给子 Isolate
  workerSendPort.send(['Hello', replyPort.sendPort]);

  // 接收回复
  final response = await replyPort.first;
  print('收到回复: $response');  // 收到回复: Echo: Hello

  // 继续通信...
  final replyPort2 = ReceivePort();
  workerSendPort.send(['World', replyPort2.sendPort]);
  final response2 = await replyPort2.first;
  print('收到回复: $response2');  // 收到回复: Echo: World

  // 关闭
  mainReceivePort.close();
  replyPort.close();
  replyPort2.close();
  isolate.kill();
}

void worker(SendPort mainSendPort) {
  // 创建子 Isolate 的接收端口
  final workerReceivePort = ReceivePort();

  // 将子 Isolate 的 SendPort 发送给主 Isolate
  mainSendPort.send(workerReceivePort.sendPort);

  // 监听消息
  workerReceivePort.listen((message) {
    final text = message[0] as String;
    final replyTo = message[1] as SendPort;

    // 处理并回复
    replyTo.send('Echo: $text');
  });
}
```

### 5.2 长连接 Worker Isolate

```dart
import 'dart:isolate';

// 任务类型
enum TaskType { add, multiply, factorial }

class Task {
  final TaskType type;
  final List<int> args;
  final SendPort replyPort;

  Task({required this.type, required this.args, required this.replyPort});
}

class WorkerPool {
  late Isolate _isolate;
  late SendPort _sendPort;
  final _receivePort = ReceivePort();

  Future<void> init() async {
    _isolate = await Isolate.spawn(
      _workerEntry,
      _receivePort.sendPort,
    );

    _sendPort = await _receivePort.first as SendPort;
  }

  Future<int> add(int a, int b) => _sendTask(TaskType.add, [a, b]);
  Future<int> multiply(int a, int b) => _sendTask(TaskType.multiply, [a, b]);
  Future<int> factorial(int n) => _sendTask(TaskType.factorial, [n]);

  Future<int> _sendTask(TaskType type, List<int> args) async {
    final replyPort = ReceivePort();
    _sendPort.send(Task(type: type, args: args, replyPort: replyPort.sendPort));
    return await replyPort.first as int;
  }

  void dispose() {
    _receivePort.close();
    _isolate.kill();
  }

  static void _workerEntry(SendPort mainSendPort) {
    final receivePort = ReceivePort();
    mainSendPort.send(receivePort.sendPort);

    receivePort.listen((task) {
      final result = _processTask(task as Task);
      task.replyPort.send(result);
    });
  }

  static int _processTask(Task task) {
    switch (task.type) {
      case TaskType.add:
        return task.args[0] + task.args[1];
      case TaskType.multiply:
        return task.args[0] * task.args[1];
      case TaskType.factorial:
        var result = 1;
        for (var i = 2; i <= task.args[0]; i++) {
          result *= i;
        }
        return result;
    }
  }
}

void main() async {
  final pool = WorkerPool();
  await pool.init();

  final sum = await pool.add(10, 20);
  print('10 + 20 = $sum');  // 10 + 20 = 30

  final product = await pool.multiply(5, 6);
  print('5 * 6 = $product');  // 5 * 6 = 30

  final fact = await pool.factorial(10);
  print('10! = $fact');  // 10! = 3628800

  pool.dispose();
}
```

### 5.3 进度报告

```dart
import 'dart:isolate';

void main() async {
  final receivePort = ReceivePort();

  final isolate = await Isolate.spawn(
    workerWithProgress,
    receivePort.sendPort,
  );

  receivePort.listen((message) {
    if (message is int) {
      print('进度: $message%');
    } else if (message is String) {
      print('结果: $message');
      receivePort.close();
      isolate.kill();
    }
  });
}

void workerWithProgress(SendPort sendPort) {
  final total = 100;

  for (var i = 0; i <= total; i++) {
    // 模拟耗时操作
    for (var j = 0; j < 10000000; j++) {}

    // 发送进度
    sendPort.send(i);
  }

  // 发送完成消息
  sendPort.send('Done');
}
```

### 5.4 可转移数据

`TransferableTypedData` 允许在不复制数据的情况下将类型化数据转移到另一个 Isolate。

```dart
import 'dart:isolate';
import 'dart:typed_data';

void main() async {
  // 创建大量数据
  final data = Uint8List(10 * 1024 * 1024);  // 10MB

  // 包装为可转移数据
  final transferable = TransferableTypedData.fromList([data]);

  // 转移后，原数据不再可用
  // print(data.length);  // 会抛出异常

  final result = await Isolate.run(() {
    // 在 Isolate 中解包
    final received = transferable.materialize().asUint8List();

    // 处理数据...
    return received.length;
  });

  print('数据大小: $result');  // 数据大小: 10485760
}
```

**TransferableTypedData 的使用场景：**

- 传递大型二进制数据（图像、音频、视频）
- 避免数据复制带来的性能开销
- 转移后原始数据失效，防止重复释放

### 5.5 Isolate 名称服务器

`IsolateNameServer` 用于在多个 Isolate 之间共享 SendPort。

```dart
import 'dart:isolate';
import 'dart:ui';

void main() async {
  // 注册端口
  final receivePort = ReceivePort();
  IsolateNameServer.registerPortWithName(
    receivePort.sendPort,
    'main_port',
  );

  // 在其他 Isolate 中查找端口
  final sendPort = IsolateNameServer.lookupPortByName('main_port');

  if (sendPort != null) {
    sendPort.send('Hello from another isolate!');
  }

  // 取消注册
  IsolateNameServer.removePortNameMapping('main_port');
}
```

**注意**：`IsolateNameServer` 仅在 Flutter 中可用。纯 Dart 应用可以使用 `package:isolate_name_server`。

---

## 第6章 实战应用

### 6.1 图像处理

```dart
import 'dart:isolate';
import 'dart:typed_data';
import 'dart:math';

class ImageProcessor {
  // 调整亮度
  static Future<Uint8List> adjustBrightness(
    Uint8List pixels,
    double factor,
  ) async {
    return await Isolate.run(() {
      final result = Uint8List(pixels.length);

      for (var i = 0; i < pixels.length; i++) {
        result[i] = (pixels[i] * factor).clamp(0, 255).toInt();
      }

      return result;
    });
  }

  // 应用高斯模糊
  static Future<Uint8List> gaussianBlur(
    Uint8List pixels,
    int width,
    int height,
    int radius,
  ) async {
    return await Isolate.run(() {
      final result = Uint8List(pixels.length);

      // 生成高斯核
      final kernel = _generateGaussianKernel(radius);

      for (var y = 0; y < height; y++) {
        for (var x = 0; x < width; x++) {
          var r = 0.0, g = 0.0, b = 0.0;

          for (var ky = -radius; ky <= radius; ky++) {
            for (var kx = -radius; kx <= radius; kx++) {
              final px = (x + kx).clamp(0, width - 1);
              final py = (y + ky).clamp(0, height - 1);
              final idx = (py * width + px) * 4;
              final weight = kernel[ky + radius][kx + radius];

              r += pixels[idx] * weight;
              g += pixels[idx + 1] * weight;
              b += pixels[idx + 2] * weight;
            }
          }

          final idx = (y * width + x) * 4;
          result[idx] = r.clamp(0, 255).toInt();
          result[idx + 1] = g.clamp(0, 255).toInt();
          result[idx + 2] = b.clamp(0, 255).toInt();
          result[idx + 3] = pixels[idx + 3];  // Alpha
        }
      }

      return result;
    });
  }

  static List<List<double>> _generateGaussianKernel(int radius) {
    final size = radius * 2 + 1;
    final kernel = List.generate(size, (_) => List<double>.filled(size, 0));
    final sigma = radius / 3.0;
    var sum = 0.0;

    for (var y = -radius; y <= radius; y++) {
      for (var x = -radius; x <= radius; x++) {
        final value = exp(-(x * x + y * y) / (2 * sigma * sigma));
        kernel[y + radius][x + radius] = value;
        sum += value;
      }
    }

    // 归一化
    for (var y = 0; y < size; y++) {
      for (var x = 0; x < size; x++) {
        kernel[y][x] /= sum;
      }
    }

    return kernel;
  }
}
```

### 6.2 JSON 批量解析

```dart
import 'dart:isolate';
import 'dart:convert';

class BatchJsonParser {
  static Future<List<Map<String, dynamic>>> parseBatch(
    List<String> jsonStrings,
  ) async {
    // 将数据分成 4 份并行处理
    final chunkSize = (jsonStrings.length / 4).ceil();
    final chunks = <List<String>>[];

    for (var i = 0; i < jsonStrings.length; i += chunkSize) {
      chunks.add(jsonStrings.sublist(
        i,
        min(i + chunkSize, jsonStrings.length),
      ));
    }

    // 并行解析
    final futures = chunks.map((chunk) => Isolate.run(
      () => chunk.map((s) => jsonDecode(s) as Map<String, dynamic>).toList(),
    ));

    final results = await Future.wait(futures);
    return results.expand((x) => x).toList();
  }
}

void main() async {
  final jsonStrings = List<String>.generate(
    1000,
    (i) => '{"id": $i, "name": "Item $i", "value": ${i * 10}}',
  );

  final results = await BatchJsonParser.parseBatch(jsonStrings);
  print('解析完成: ${results.length} 条记录');
}
```

### 6.3 矩阵运算

```dart
import 'dart:isolate';
import 'dart:typed_data';

class ParallelMatrix {
  final int rows;
  final int cols;
  final Float64List data;

  ParallelMatrix(this.rows, this.cols) : data = Float64List(rows * cols);

  double get(int row, int col) => data[row * cols + col];
  void set(int row, int col, double value) => data[row * cols + col] = value;

  static Future<ParallelMatrix> multiply(
    ParallelMatrix a,
    ParallelMatrix b,
  ) async {
    if (a.cols != b.rows) {
      throw ArgumentError('矩阵维度不匹配');
    }

    final result = ParallelMatrix(a.rows, b.cols);

    // 并行计算每一行
    final futures = <Future<void>>[];

    for (var i = 0; i < a.rows; i++) {
      futures.add(Isolate.run(() {
        for (var j = 0; j < b.cols; j++) {
          var sum = 0.0;
          for (var k = 0; k < a.cols; k++) {
            sum += a.get(i, k) * b.get(k, j);
          }
          result.set(i, j, sum);
        }
      }));
    }

    await Future.wait(futures);
    return result;
  }
}
```

### 6.4 文件处理

```dart
import 'dart:isolate';
import 'dart:io';
import 'dart:convert';

class FileProcessor {
  // 并行处理多个文件
  static Future<List<int>> processFiles(List<String> paths) async {
    final futures = paths.map((path) => Isolate.run(() async {
      final file = File(path);
      final content = await file.readAsString();

      // 处理内容（例如：统计行数）
      return content.split('\n').length;
    }));

    return await Future.wait(futures);
  }

  // 大文件分块处理
  static Future<int> countLinesInLargeFile(String path) async {
    final file = File(path);
    final length = await file.length();

    // 分成 4 个块
    final chunkSize = length ~/ 4;
    final futures = <Future<int>>[];

    for (var i = 0; i < 4; i++) {
      final start = i * chunkSize;
      final end = (i == 3) ? length : (i + 1) * chunkSize;

      futures.add(Isolate.run(() async {
        final chunk = await file.openRead(start, end).toList();
        final content = utf8.decode(chunk.expand((x) => x).toList());
        return content.split('\n').length;
      }));
    }

    final counts = await Future.wait(futures);
    return counts.reduce((a, b) => a + b);
  }
}
```

### 6.5 流式处理

```dart
import 'dart:isolate';
import 'dart:async';

class StreamProcessor {
  static Stream<int> processStream(Stream<int> input) async* {
    final receivePort = ReceivePort();
    final isolate = await Isolate.spawn(
      _worker,
      receivePort.sendPort,
    );

    final sendPort = await receivePort.first as SendPort;

    // 发送数据到 Isolate
    await for (final value in input) {
      final replyPort = ReceivePort();
      sendPort.send([value, replyPort.sendPort]);
      yield await replyPort.first as int;
    }

    receivePort.close();
    isolate.kill();
  }

  static void _worker(SendPort mainSendPort) {
    final receivePort = ReceivePort();
    mainSendPort.send(receivePort.sendPort);

    receivePort.listen((message) {
      final value = message[0] as int;
      final replyTo = message[1] as SendPort;

      // 处理数据
      final result = value * value;
      replyTo.send(result);
    });
  }
}

void main() async {
  final input = Stream.fromIterable(List<int>.generate(100, (i) => i));

  await for (final result in StreamProcessor.processStream(input)) {
    print(result);
  }
}
```

---

## 附录：完整示例代码

### 示例1：Isolate 管理器

```dart
import 'dart:isolate';
import 'dart:collection';

class IsolateManager {
  final int maxIsolates;
  final Queue<_Worker> _available = Queue();
  final Set<_Worker> _busy = {};
  int _created = 0;

  IsolateManager({this.maxIsolates = 4});

  Future<T> run<T>(Future<T> Function() computation) async {
    _Worker? worker;

    // 获取可用 Worker
    if (_available.isNotEmpty) {
      worker = _available.removeFirst();
    } else if (_created < maxIsolates) {
      worker = await _createWorker();
      _created++;
    } else {
      // 等待可用 Worker
      while (_available.isEmpty) {
        await Future.delayed(Duration(milliseconds: 10));
      }
      worker = _available.removeFirst();
    }

    _busy.add(worker);

    try {
      final result = await worker.run(computation);
      return result;
    } finally {
      _busy.remove(worker);
      _available.add(worker);
    }
  }

  Future<_Worker> _createWorker() async {
    final receivePort = ReceivePort();
    final isolate = await Isolate.spawn(
      _workerEntry,
      receivePort.sendPort,
    );

    final sendPort = await receivePort.first as SendPort;
    return _Worker(isolate, sendPort);
  }

  void dispose() {
    for (final worker in _available) {
      worker.dispose();
    }
    for (final worker in _busy) {
      worker.dispose();
    }
    _available.clear();
    _busy.clear();
  }

  static void _workerEntry(SendPort mainSendPort) {
    final receivePort = ReceivePort();
    mainSendPort.send(receivePort.sendPort);

    receivePort.listen((message) async {
      final computation = message[0] as Future<dynamic> Function();
      final replyTo = message[1] as SendPort;

      try {
        final result = await computation();
        replyTo.send(['success', result]);
      } catch (e) {
        replyTo.send(['error', e.toString()]);
      }
    });
  }
}

class _Worker {
  final Isolate isolate;
  final SendPort sendPort;

  _Worker(this.isolate, this.sendPort);

  Future<T> run<T>(Future<T> Function() computation) async {
    final replyPort = ReceivePort();
    sendPort.send([computation, replyPort.sendPort]);

    final response = await replyPort.first as List;
    replyPort.close();

    if (response[0] == 'success') {
      return response[1] as T;
    } else {
      throw Exception(response[1]);
    }
  }

  void dispose() {
    isolate.kill();
  }
}
```

### 示例2：取消令牌

```dart
import 'dart:isolate';

class CancellationToken {
  bool _isCancelled = false;
  final _listeners = <void Function()>[];

  bool get isCancelled => _isCancelled;

  void cancel() {
    _isCancelled = true;
    for (final listener in _listeners) {
      listener();
    }
  }

  void onCancel(void Function() listener) {
    _listeners.add(listener);
  }

  void throwIfCancelled() {
    if (_isCancelled) {
      throw CancellationException();
    }
  }
}

class CancellationException implements Exception {
  @override
  String toString() => 'Operation was cancelled';
}

Future<T> runWithCancellation<T>(
  Future<T> Function(CancellationToken token) computation,
  CancellationToken token,
) async {
  return await Isolate.run(() async {
    return await computation(token);
  });
}

// 使用示例
void main() async {
  final token = CancellationToken();

  // 5 秒后取消
  Future.delayed(Duration(seconds: 5), () => token.cancel());

  try {
    final result = await runWithCancellation((token) async {
      for (var i = 0; i < 100; i++) {
        token.throwIfCancelled();
        await Future.delayed(Duration(milliseconds: 100));
      }
      return 'Completed';
    }, token);

    print(result);
  } on CancellationException {
    print('Operation was cancelled');
  }
}
```

---

## 结语

`dart:isolate` 为 Dart 和 Flutter 开发者提供了一种安全、高效的并发编程模型。通过本书的学习，你应该已经掌握了：

1. **核心概念**：Isolate 的并发模型、与线程的区别、可发送的对象
2. **Isolate 类**：创建、控制和管理 Isolate 的生命周期
3. **端口通信**：使用 SendPort 和 ReceivePort 进行消息传递
4. **简单并发模式**：Isolate.run 和 compute 的使用
5. **高级通信模式**：双向通信、长连接 Worker、进度报告
6. **实战应用**：图像处理、JSON 解析、矩阵运算等

Isolate 的设计哲学是**通过消息传递实现无锁并发**，这让并发编程变得更加安全和简单。在实际开发中，建议：

- 使用 `Isolate.run` 或 `compute` 处理一次性计算任务
- 使用长连接 Worker Isolate 处理重复性任务
- 使用 `TransferableTypedData` 传递大型数据
- 注意资源管理，及时关闭 ReceivePort 和终止 Isolate
- 避免在 Isolate 中访问 UI 相关对象

希望本书能帮助你更好地使用 `dart:isolate` 构建高性能的 Dart/Flutter 应用！
