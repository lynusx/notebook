# Dart 核心库 dart:async 详解

`dart:async` 是 Dart 语言的核心库之一，提供了异步编程的基础支持。该库定义了 `Future` 和 `Stream` 等核心类，以及 `Timer`、`Zone`、`Completer` 等辅助类和工具函数，为 Dart 的异步编程模型提供了完整的实现基础。

需要注意的是，`Future`、`Stream`、`StreamController` 等核心类的详细内容已在《Dart 异步编程详解》文档中系统介绍，本文档将不再重复。本文档将重点介绍 `dart:async` 库中的其他重要类和功能，包括 `Timer` 定时器、`Zone` 执行区域、`Completer` 手动完成器、微任务调度等高级特性，帮助开发者全面掌握 Dart 异步编程的底层机制。

## 第1章 Timer 定时器

`Timer` 类用于在指定的时间后执行一次代码，或按固定间隔重复执行代码。它是实现延迟操作和周期性任务的基础工具。

### 1.1 创建 Timer

#### 1.1.1 一次性定时器

```dart
import 'dart:async';

void main() {
  print('Start: ${DateTime.now()}');

  // 创建一次性定时器，2秒后执行
  Timer(Duration(seconds: 2), () {
    print('Timer fired: ${DateTime.now()}');
  });

  print('Timer scheduled');

  // 保持程序运行
  // 输出顺序：
  // Start: ...
  // Timer scheduled
  // (2秒后) Timer fired: ...
}
```

#### 1.1.2 周期性定时器

```dart
import 'dart:async';

void main() {
  var count = 0;

  // 创建周期性定时器，每秒执行一次
  var timer = Timer.periodic(Duration(seconds: 1), (t) {
    count++;
    print('Tick $count: ${DateTime.now()}');

    // 5次后停止
    if (count >= 5) {
      t.cancel();
      print('Timer cancelled');
    }
  });

  // 输出：
  // Tick 1: ...
  // Tick 2: ...
  // ...
  // Timer cancelled
}
```

### 1.2 Timer 的属性和方法

#### 1.2.1 isActive

`isActive` 属性用于检查定时器是否仍在活动状态。

```dart
import 'dart:async';

void main() {
  var timer = Timer(Duration(seconds: 2), () {});

  print('Is active: ${timer.isActive}');  // 输出: true

  // 取消定时器
  timer.cancel();

  print('Is active after cancel: ${timer.isActive}');  // 输出: false
}
```

#### 1.2.2 tick

`tick` 属性返回周期性定时器已经触发的次数。

```dart
import 'dart:async';

void main() {
  Timer.periodic(Duration(milliseconds: 100), (timer) {
    print('Tick count: ${timer.tick}');

    if (timer.tick >= 5) {
      timer.cancel();
    }
  });
  // 输出: Tick count: 1, 2, 3, 4, 5
}
```

#### 1.2.3 cancel()

`cancel()` 方法用于取消定时器。对于一次性定时器，如果代码已经执行，取消无效；对于周期性定时器，取消后将不再触发。

```dart
import 'dart:async';

void main() {
  // 取消一次性定时器
  var oneShot = Timer(Duration(seconds: 5), () {
    print('This will not print');
  });
  oneShot.cancel();

  // 取消周期性定时器
  var periodic = Timer.periodic(Duration(seconds: 1), (_) {
    print('Tick');
  });

  Timer(Duration(seconds: 3), () {
    periodic.cancel();
    print('Periodic timer cancelled');
  });
}
```

### 1.3 Timer 的静态方法

#### 1.3.1 run()

`Timer.run()` 安排一个回调函数在事件循环的下一轮执行，类似于 `Future.microtask()`，但优先级略低。

```dart
import 'dart:async';

void main() {
  print('1. Main start');

  Timer.run(() {
    print('4. Timer.run');
  });

  Future.microtask(() {
    print('3. Microtask');
  });

  Future(() {
    print('5. Future');
  });

  print('2. Main end');

  // 输出顺序：
  // 1. Main start
  // 2. Main end
  // 3. Microtask
  // 4. Timer.run
  // 5. Future
}
```

### 1.4 实战应用

#### 1.4.1 防抖（Debounce）

```dart
import 'dart:async';

class Debouncer {
  final Duration delay;
  Timer? _timer;

  Debouncer(this.delay);

  void run(void Function() action) {
    _timer?.cancel();
    _timer = Timer(delay, action);
  }

  void cancel() {
    _timer?.cancel();
  }
}

void main() {
  var debouncer = Debouncer(Duration(milliseconds: 500));

  // 模拟快速输入
  debouncer.run(() => print('Search: F'));
  debouncer.run(() => print('Search: Fl'));
  debouncer.run(() => print('Search: Flu'))
  debouncer.run(() => print('Search: Flutter'));  // 只有这个会执行
}
```

#### 1.4.2 节流（Throttle）

```dart
import 'dart:async';

class Throttler {
  final Duration interval;
  Timer? _timer;
  bool _canRun = true;

  Throttler(this.interval);

  void run(void Function() action) {
    if (_canRun) {
      _canRun = false;
      action();
      _timer = Timer(interval, () => _canRun = true);
    }
  }

  void cancel() {
    _timer?.cancel();
  }
}

void main() {
  var throttler = Throttler(Duration(seconds: 1));

  // 每秒最多执行一次
  throttler.run(() => print('Action 1'));
  throttler.run(() => print('Action 2'));  // 被忽略

  Future.delayed(Duration(seconds: 1), () {
    throttler.run(() => print('Action 3'));  // 执行
  });
}
```

#### 1.4.3 超时处理

```dart
import 'dart:async';

Future<T> withTimeout<T>(Future<T> future, Duration timeout, {T? defaultValue}) {
  var completer = Completer<T>();

  // 设置超时定时器
  var timer = Timer(timeout, () {
    if (!completer.isCompleted) {
      if (defaultValue != null) {
        completer.complete(defaultValue);
      } else {
        completer.completeError(TimeoutException('Operation timed out', timeout));
      }
    }
  });

  // 监听原 Future
  future.then((value) {
    if (!completer.isCompleted) {
      timer.cancel();
      completer.complete(value);
    }
  }).catchError((error) {
    if (!completer.isCompleted) {
      timer.cancel();
      completer.completeError(error);
    }
  });

  return completer.future;
}

void main() async {
  // 快速操作，不会超时
  var result1 = await withTimeout(
    Future.delayed(Duration(seconds: 1), () => 'Success'),
    Duration(seconds: 2),
  );
  print(result1);  // 输出: Success

  // 慢操作，使用默认值
  var result2 = await withTimeout(
    Future.delayed(Duration(seconds: 3), () => 'Too late'),
    Duration(seconds: 1),
    defaultValue: 'Default',
  );
  print(result2);  // 输出: Default
}
```

## 第2章 Completer 手动完成器

`Completer` 类用于手动控制 `Future` 的完成。当你需要将一个基于回调的 API 转换为基于 `Future` 的 API 时，`Completer` 非常有用。

### 2.1 基本用法

#### 2.1.1 创建和完成 Completer

```dart
import 'dart:async';

void main() {
  // 创建 Completer
  var completer = Completer<String>();

  // 获取 Future
  Future<String> future = completer.future;

  // 监听 Future
  future.then((value) {
    print('Completed with: $value');
  });

  // 稍后完成
  Future.delayed(Duration(seconds: 2), () {
    completer.complete('Hello from Completer!');
  });
}
```

#### 2.1.2 完成时抛出错误

```dart
import 'dart:async';

void main() {
  var completer = Completer<int>();

  completer.future
    .then((value) => print('Success: $value'))
    .catchError((error) => print('Error: $error'));

  // 完成时抛出错误
  Future.delayed(Duration(seconds: 1), () {
    completer.completeError(Exception('Something went wrong'));
  });
}
```

### 2.2 Completer 的属性

#### 2.2.1 isCompleted

`isCompleted` 属性用于检查 `Completer` 是否已经完成。

```dart
import 'dart:async';

void main() {
  var completer = Completer<String>();

  print('Is completed: ${completer.isCompleted}');  // 输出: false

  completer.complete('Done');

  print('Is completed: ${completer.isCompleted}');  // 输出: true

  // 重复完成会报错
  // completer.complete('Again');  // 错误: Future already completed
}
```

### 2.3 实战应用

#### 2.3.1 将回调 API 转换为 Future

```dart
import 'dart:async';

// 假设这是一个基于回调的第三方库
class CallbackApi {
  void fetchData(String id, void Function(String) onSuccess, void Function(Exception) onError) {
    Future.delayed(Duration(seconds: 1), () {
      if (id.isNotEmpty) {
        onSuccess('Data for $id');
      } else {
        onError(Exception('Invalid ID'));
      }
    });
  }
}

// 包装为 Future API
Future<String> fetchDataAsync(String id) {
  var completer = Completer<String>();
  var api = CallbackApi();

  api.fetchData(
    id,
    (data) => completer.complete(data),
    (error) => completer.completeError(error),
  );

  return completer.future;
}

void main() async {
  try {
    var data = await fetchDataAsync('123');
    print('Received: $data');
  } catch (e) {
    print('Error: $e');
  }
}
```

#### 2.3.2 一次性事件通知

```dart
import 'dart:async';

class InitializationManager {
  final Completer<void> _initCompleter = Completer<void>();
  bool _initialized = false;

  Future<void> get initialization => _initCompleter.future;
  bool get isInitialized => _initialized;

  void initialize() async {
    if (_initialized) return;

    // 执行初始化
    await Future.delayed(Duration(seconds: 2));
    _initialized = true;

    // 通知所有等待者
    _initCompleter.complete();
  }
}

void main() async {
  var manager = InitializationManager();

  // 多个地方等待初始化完成
  Future.wait([
    manager.initialization.then((_) => print('Component 1 ready')),
    manager.initialization.then((_) => print('Component 2 ready')),
    manager.initialization.then((_) => print('Component 3 ready')),
  ]);

  // 执行初始化
  manager.initialize();
}
```

#### 2.3.3 实现异步缓存

```dart
import 'dart:async';

class AsyncCache<K, V> {
  final Map<K, V> _cache = {};
  final Map<K, Completer<V>> _pending = {};
  final Future<V> Function(K) _loader;

  AsyncCache(this._loader);

  Future<V> get(K key) async {
    // 检查缓存
    if (_cache.containsKey(key)) {
      return _cache[key]!;
    }

    // 检查是否有正在进行的请求
    if (_pending.containsKey(key)) {
      return _pending[key]!.future;
    }

    // 创建新的请求
    var completer = Completer<V>();
    _pending[key] = completer;

    try {
      var value = await _loader(key);
      _cache[key] = value;
      completer.complete(value);
    } catch (e) {
      completer.completeError(e);
    } finally {
      _pending.remove(key);
    }

    return completer.future;
  }

  void invalidate(K key) {
    _cache.remove(key);
  }

  void clear() {
    _cache.clear();
  }
}

void main() async {
  var fetchCount = 0;

  var cache = AsyncCache<String, String>((key) async {
    fetchCount++;
    await Future.delayed(Duration(seconds: 1));
    return 'Data for $key';
  });

  // 并发请求相同 key，只执行一次
  var results = await Future.wait([
    cache.get('A'),
    cache.get('A'),
    cache.get('A'),
  ]);

  print('Fetch count: $fetchCount');  // 输出: 1
  print('Results: $results');  // 输出: [Data for A, Data for A, Data for A]
}
```

## 第3章 Zone 执行区域

`Zone` 是 Dart 中用于管理异步代码执行环境的机制。每个 Zone 都有自己的错误处理、计时器和异步操作上下文，可以实现代码的隔离和增强。

### 3.1 Zone 的基本概念

#### 3.1.1 什么是 Zone

Zone 是一个异步操作的执行环境。当你运行异步代码时，它总是在某个 Zone 中执行。默认情况下，代码在根 Zone（`Zone.root`）中运行。

```dart
import 'dart:async';

void main() {
  // 获取当前 Zone
  print('Current zone: ${Zone.current}');
  print('Is root zone: ${Zone.current == Zone.root}');

  // 在 Zone 中运行代码
  runZoned(() {
    print('Inside zone: ${Zone.current}');
    print('Is root zone: ${Zone.current == Zone.root}');
  });
}
```

#### 3.1.2 创建 Zone

```dart
import 'dart:async';

void main() {
  // 使用 runZoned 创建新 Zone
  runZoned(() {
    print('Running in new zone');

    // 异步操作也在这个 Zone 中
    Future(() {
      print('Future in zone: ${Zone.current}');
    });
  });
}
```

### 3.2 Zone 的错误处理

#### 3.2.1 Zone 中的错误捕获

```dart
import 'dart:async';

void main() {
  runZoned(() {
    // 这个错误会被 Zone 捕获
    throw Exception('Error in zone');
  }, zoneSpecification: ZoneSpecification(
    handleUncaughtError: (self, parent, zone, error, stackTrace) {
      print('Caught error in zone: $error');
      print('Stack trace: $stackTrace');
    },
  ));

  print('Program continues after zone error');
}
```

#### 3.2.2 异步错误捕获

```dart
import 'dart:async';

void main() {
  runZoned(() async {
    // 异步错误也会被捕获
    await Future.delayed(Duration(seconds: 1));
    throw Exception('Async error in zone');
  }, zoneSpecification: ZoneSpecification(
    handleUncaughtError: (self, parent, zone, error, stackTrace) {
      print('Caught async error: $error');
    },
  ));

  // 保持程序运行
  Future.delayed(Duration(seconds: 2), () {
    print('Program continues');
  });
}
```

### 3.3 Zone 的变量

#### 3.3.1 ZoneValues

`ZoneValues` 允许在 Zone 中存储和访问变量，实现上下文传递。

```dart
import 'dart:async';

void main() {
  // 创建带变量的 Zone
  runZoned(() {
    // 读取 Zone 变量
    var requestId = Zone.current[#requestId];
    var userId = Zone.current[#userId];

    print('Request ID: $requestId');
    print('User ID: $userId');

    // 异步操作也能访问
    Future(() {
      print('In future - Request ID: ${Zone.current[#requestId]}');
    });
  }, zoneValues: {
    #requestId: 'req-12345',
    #userId: 'user-67890',
  });
}
```

#### 3.3.2 实战：请求上下文传递

```dart
import 'dart:async';

class RequestContext {
  final String requestId;
  final String? userId;
  final DateTime startTime;

  RequestContext({
    required this.requestId,
    this.userId,
  }) : startTime = DateTime.now();

  Duration get elapsed => DateTime.now().difference(startTime);
}

void runInRequestContext(RequestContext context, void Function() body) {
  runZoned(body, zoneValues: {
    #requestContext: context,
  });
}

RequestContext? get currentContext => Zone.current[#requestContext] as RequestContext?;

void log(String message) {
  var context = currentContext;
  if (context != null) {
    print('[${context.requestId}] (${context.elapsed.inMilliseconds}ms) $message');
  } else {
    print(message);
  }
}

void main() {
  runInRequestContext(
    RequestContext(requestId: 'req-001', userId: 'user-123'),
    () async {
      log('Processing started');

      await Future.delayed(Duration(milliseconds: 100));
      log('Step 1 completed');

      await Future.delayed(Duration(milliseconds: 200));
      log('Step 2 completed');

      log('Processing finished');
    },
  );
}
```

### 3.4 Zone 的拦截和增强

#### 3.4.1 拦截异步操作

```dart
import 'dart:async';

void main() {
  runZoned(() {
    Future(() => print('Future executed'));
    Timer(Duration(milliseconds: 100), () => print('Timer executed'));
  }, zoneSpecification: ZoneSpecification(
    run: <R>(self, parent, zone, f) {
      print('Before run');
      var result = parent.run(zone, f);
      print('After run');
      return result;
    },
    runUnary: <R, T>(self, parent, zone, f, arg) {
      print('Before runUnary with $arg');
      var result = parent.runUnary(zone, f, arg);
      print('After runUnary');
      return result;
    },
    scheduleMicrotask: (self, parent, zone, f) {
      print('Microtask scheduled');
      parent.scheduleMicrotask(zone, f);
    },
    createTimer: (self, parent, zone, duration, f) {
      print('Timer created with duration: $duration');
      return parent.createTimer(zone, duration, f);
    },
  ));
}
```

#### 3.4.2 实战：性能监控 Zone

```dart
import 'dart:async';

class PerformanceZone {
  static void run(void Function() body, {void Function(String, Duration)? onReport}) {
    var timings = <String, Stopwatch>{};

    runZoned(body, zoneSpecification: ZoneSpecification(
      run: <R>(self, parent, zone, f) {
        var sw = Stopwatch()..start();
        var result = parent.run(zone, f);
        sw.stop();
        onReport?.call('run', sw.elapsed);
        return result;
      },
      runUnary: <R, T>(self, parent, zone, f, arg) {
        var sw = Stopwatch()..start();
        var result = parent.runUnary(zone, f, arg);
        sw.stop();
        onReport?.call('runUnary', sw.elapsed);
        return result;
      },
      createTimer: (self, parent, zone, duration, f) {
        var sw = Stopwatch()..start();
        return parent.createTimer(zone, duration, () {
          sw.stop();
          onReport?.call('timer($duration)', sw.elapsed);
          f();
        });
      },
    ));
  }
}

void main() {
  PerformanceZone.run(() async {
    await Future.delayed(Duration(milliseconds: 100));
    print('Work done');
  }, onReport: (operation, duration) {
    print('$operation took ${duration.inMicroseconds}μs');
  });
}
```

## 第4章 微任务调度

Dart 的事件循环处理两种类型的任务：微任务（microtasks）和事件（events）。微任务具有更高的优先级。

### 4.1 scheduleMicrotask

`scheduleMicrotask` 函数用于将一个回调函数添加到微任务队列中。

```dart
import 'dart:async';

void main() {
  print('1. Start');

  // 调度微任务
  scheduleMicrotask(() {
    print('3. Microtask 1');
  });

  scheduleMicrotask(() {
    print('4. Microtask 2');
  });

  // 调度普通 Future
  Future(() {
    print('5. Future 1');
  });

  Future(() {
    print('6. Future 2');
  });

  print('2. End');

  // 输出顺序：
  // 1. Start
  // 2. End
  // 3. Microtask 1
  // 4. Microtask 2
  // 5. Future 1
  // 6. Future 2
}
```

### 4.2 微任务与 Future 的区别

```dart
import 'dart:async';

void main() {
  print('Start');

  // 微任务会立即在当前事件循环迭代中执行
  scheduleMicrotask(() => print('Microtask'));

  // Future 会安排在下一个事件循环迭代中执行
  Future(() => print('Future'));

  // 微任务优先于 Future
  scheduleMicrotask(() => print('Microtask 2'));

  // 耗时操作
  for (var i = 0; i < 1000000; i++) {}

  print('End');

  // 输出：
  // Start
  // End
  // Microtask
  // Microtask 2
  // Future
}
```

### 4.3 实战应用

#### 4.3.1 确保异步回调在当前帧执行

```dart
import 'dart:async';

class AsyncNotifier<T> {
  final List<void Function(T)> _listeners = [];

  void addListener(void Function(T) listener) {
    _listeners.add(listener);
  }

  void notify(T value) {
    // 使用微任务确保回调在当前事件循环中执行
    scheduleMicrotask(() {
      for (var listener in _listeners) {
        listener(value);
      }
    });
  }
}

void main() {
  var notifier = AsyncNotifier<String>();

  notifier.addListener((value) {
    print('Listener 1: $value');
  });

  notifier.addListener((value) {
    print('Listener 2: $value');
  });

  print('Before notify');
  notifier.notify('Hello');
  print('After notify');

  // 输出：
  // Before notify
  // After notify
  // Listener 1: Hello
  // Listener 2: Hello
}
```

## 第5章 Stream 相关类深入

虽然《Dart 异步编程详解》已介绍了 Stream 的基础知识，但 `dart:async` 库还提供了一些更高级的 Stream 相关类。

### 5.1 StreamSink

`StreamSink` 是一个抽象类，定义了可以同步或异步接收数据流的接口。`StreamController` 实现了这个接口。

```dart
import 'dart:async';

void main() async {
  // StreamController 实现了 StreamSink
  var controller = StreamController<int>();
  StreamSink<int> sink = controller.sink;

  // 通过 sink 添加数据
  sink.add(1);
  sink.add(2);
  sink.add(3);

  // 关闭 sink
  await sink.close();

  // 监听流
  await for (var value in controller.stream) {
    print('Received: $value');
  }
}
```

### 5.2 EventSink

`EventSink` 是 `StreamSink` 的子类，增加了处理错误的能力。

```dart
import 'dart:async';

void main() {
  var controller = StreamController<int>();
  EventSink<int> eventSink = controller.sink;

  // 添加数据
  eventSink.add(1);
  eventSink.add(2);

  // 添加错误
  eventSink.addError(Exception('Test error'));

  // 添加更多数据
  eventSink.add(3);

  eventSink.close();

  // 监听（包含错误处理）
  controller.stream.listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done'),
  );
}
```

### 5.3 StreamTransformer

`StreamTransformer` 用于将一个 `Stream` 转换为另一个 `Stream`。

```dart
import 'dart:async';

// 创建自定义转换器
StreamTransformer<int, String> intToStringTransformer() {
  return StreamTransformer<int, String>.fromHandlers(
    handleData: (int data, EventSink<String> sink) {
      sink.add('Number: $data');
    },
    handleError: (Object error, StackTrace stackTrace, EventSink<String> sink) {
      sink.addError('Error occurred: $error', stackTrace);
    },
    handleDone: (EventSink<String> sink) {
      sink.add('Stream completed');
      sink.close();
    },
  );
}

void main() async {
  var stream = Stream.fromIterable([1, 2, 3, 4, 5]);

  // 应用转换器
  var transformed = stream.transform(intToStringTransformer());

  await for (var value in transformed) {
    print(value);
  }
  // 输出：
  // Number: 1
  // Number: 2
  // ...
  // Stream completed
}
```

### 5.4 StreamSubscription

`StreamSubscription` 表示对流的订阅，提供了控制订阅的方法。

```dart
import 'dart:async';

void main() async {
  var controller = StreamController<int>();

  // 订阅流
  StreamSubscription<int> subscription = controller.stream.listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done'),
  );

  // 暂停订阅
  subscription.pause();
  controller.add(1);  // 不会被处理
  controller.add(2);  // 不会被处理

  await Future.delayed(Duration(milliseconds: 100));

  // 恢复订阅
  subscription.resume();  // 会处理 1 和 2

  // 取消订阅
  await subscription.cancel();
  controller.add(3);  // 不会被处理

  await controller.close();
}
```

### 5.5 SynchronousStreamController

`SynchronousStreamController` 是一种特殊的 `StreamController`，它会立即（同步地）向监听器分发事件。

```dart
import 'dart:async';

void main() {
  // 普通 StreamController（异步分发）
  var asyncController = StreamController<int>();

  asyncController.stream.listen((data) {
    print('Async received: $data');
  });

  print('Before async add');
  asyncController.add(1);
  print('After async add');

  // 输出：
  // Before async add
  // After async add
  // Async received: 1

  // 同步 StreamController（同步分发）
  var syncController = StreamController<int>(sync: true);

  syncController.stream.listen((data) {
    print('Sync received: $data');
  });

  print('Before sync add');
  syncController.add(1);
  print('After sync add');

  // 输出：
  // Before sync add
  // Sync received: 1
  // After sync add
}
```

**Dart Tips 语法小贴士**

同步 StreamController 的使用注意事项

同步 `StreamController` 会立即执行监听器回调，这可能导致意外行为：

```dart
// 危险：可能导致无限递归
var controller = StreamController<int>(sync: true);

controller.stream.listen((data) {
  if (data < 10) {
    controller.add(data + 1);  // 同步执行，可能栈溢出
  }
});

// 推荐：使用异步控制器
var safeController = StreamController<int>();  // 默认异步
```

## 第6章 实战应用示例

### 6.1 异步任务队列

```dart
import 'dart:async';

class AsyncTaskQueue {
  final int maxConcurrent;
  final List<_QueuedTask> _queue = [];
  final List<Future> _running = [];

  AsyncTaskQueue({this.maxConcurrent = 3});

  Future<T> add<T>(Future<T> Function() task) {
    var completer = Completer<T>();
    _queue.add(_QueuedTask(task, completer));
    _processQueue();
    return completer.future;
  }

  void _processQueue() {
    while (_running.length < maxConcurrent && _queue.isNotEmpty) {
      var queued = _queue.removeAt(0);

      var future = queued.task().then((result) {
        queued.completer.complete(result);
      }).catchError((error) {
        queued.completer.completeError(error);
      }).whenComplete(() {
        _running.removeWhere((f) => f == future);
        _processQueue();
      });

      _running.add(future);
    }
  }
}

class _QueuedTask {
  final Future Function() task;
  final Completer completer;

  _QueuedTask(this.task, this.completer);
}

void main() async {
  var queue = AsyncTaskQueue(maxConcurrent: 2);

  // 添加任务
  var futures = <Future>[];
  for (var i = 0; i < 5; i++) {
    futures.add(queue.add(() async {
      print('Task $i started');
      await Future.delayed(Duration(seconds: 1));
      print('Task $i completed');
      return 'Result $i';
    }));
  }

  await Future.wait(futures);
  print('All tasks completed');
}
```

### 6.2 带重试的异步操作

```dart
import 'dart:async';

Future<T> retry<T>(
  Future<T> Function() operation, {
  int maxAttempts = 3,
  Duration delay = const Duration(seconds: 1),
  bool Function(Exception)? retryIf,
}) async {
  var attempts = 0;

  while (true) {
    attempts++;

    try {
      return await operation();
    } on Exception catch (e) {
      if (attempts >= maxAttempts) {
        rethrow;
      }

      if (retryIf != null && !retryIf(e)) {
        rethrow;
      }

      print('Attempt $attempts failed: $e. Retrying...');
      await Future.delayed(delay);
    }
  }
}

void main() async {
  var attempt = 0;

  try {
    var result = await retry(() async {
      attempt++;
      if (attempt < 3) {
        throw Exception('Network error');
      }
      return 'Success!';
    }, maxAttempts: 5);

    print('Result: $result');
  } catch (e) {
    print('Failed after retries: $e');
  }
}
```

### 6.3 资源池管理

```dart
import 'dart:async';

class ResourcePool<T> {
  final List<T> _available;
  final List<T> _inUse = [];
  final List<Completer<T>> _waiters = [];

  ResourcePool(List<T> resources) : _available = List.from(resources);

  Future<T> acquire() async {
    if (_available.isNotEmpty) {
      var resource = _available.removeLast();
      _inUse.add(resource);
      return resource;
    }

    // 等待资源可用
    var completer = Completer<T>();
    _waiters.add(completer);
    return completer.future;
  }

  void release(T resource) {
    _inUse.remove(resource);

    if (_waiters.isNotEmpty) {
      var waiter = _waiters.removeAt(0);
      _inUse.add(resource);
      waiter.complete(resource);
    } else {
      _available.add(resource);
    }
  }

  Future<R> withResource<R>(Future<R> Function(T) operation) async {
    var resource = await acquire();
    try {
      return await operation(resource);
    } finally {
      release(resource);
    }
  }
}

void main() async {
  // 创建数据库连接池
  var pool = ResourcePool<String>(['Conn1', 'Conn2', 'Conn3']);

  // 并发使用连接
  await Future.wait([
    pool.withResource((conn) async {
      print('Using $conn');
      await Future.delayed(Duration(seconds: 1));
      print('Released $conn');
      return 'Result 1';
    }),
    pool.withResource((conn) async {
      print('Using $conn');
      await Future.delayed(Duration(seconds: 1));
      print('Released $conn');
      return 'Result 2';
    }),
    pool.withResource((conn) async {
      print('Using $conn');
      await Future.delayed(Duration(seconds: 1));
      print('Released $conn');
      return 'Result 3';
    }),
    pool.withResource((conn) async {
      print('Using $conn');
      await Future.delayed(Duration(seconds: 1));
      print('Released $conn');
      return 'Result 4';
    }),
  ]);
}
```

### 6.4 事件总线（带类型安全）

```dart
import 'dart:async';

class TypedEventBus {
  final Map<Type, StreamController<dynamic>> _controllers = {};
  final Map<Type, Map<String, StreamSubscription>> _subscriptions = {};

  StreamController<T> _getController<T>() {
    return _controllers.putIfAbsent(
      T,
      () => StreamController<T>.broadcast(),
    ) as StreamController<T>;
  }

  void emit<T>(T event) {
    _getController<T>().add(event);
  }

  String on<T>(void Function(T) handler) {
    var id = '${T.toString()}_${DateTime.now().millisecondsSinceEpoch}';
    var subscription = _getController<T>().stream.listen((event) {
      handler(event as T);
    });

    _subscriptions.putIfAbsent(T, () => {});
    _subscriptions[T]![id] = subscription;

    return id;
  }

  void off<T>(String subscriptionId) {
    var subscription = _subscriptions[T]?.remove(subscriptionId);
    subscription?.cancel();
  }

  void offAll<T>() {
    _subscriptions[T]?.values.forEach((s) => s.cancel());
    _subscriptions.remove(T);
  }

  void dispose() {
    _controllers.values.forEach((c) => c.close());
    _controllers.clear();
    _subscriptions.clear();
  }
}

// 定义事件类型
class UserLoggedIn {
  final String userId;
  final String username;
  UserLoggedIn(this.userId, this.username);
}

class OrderCreated {
  final String orderId;
  final double amount;
  OrderCreated(this.orderId, this.amount);
}

void main() {
  var bus = TypedEventBus();

  // 订阅事件
  var sub1 = bus.on<UserLoggedIn>((event) {
    print('User logged in: ${event.username}');
  });

  var sub2 = bus.on<OrderCreated>((event) {
    print('Order created: ${event.orderId} - \$${event.amount}');
  });

  // 发送事件
  bus.emit(UserLoggedIn('123', 'Alice'));
  bus.emit(OrderCreated('ORD-001', 99.99));

  // 取消订阅
  bus.off<UserLoggedIn>(sub1);

  bus.dispose();
}
```

## 附录：dart:async 核心类速查表

| 类/函数            | 主要用途        | 核心方法/属性                                 |
| ------------------ | --------------- | --------------------------------------------- |
| Timer              | 定时器          | Timer(), Timer.periodic(), cancel(), isActive |
| Completer          | 手动完成 Future | complete(), completeError(), isCompleted      |
| Zone               | 执行区域        | runZoned(), run, bindCallback                 |
| scheduleMicrotask  | 调度微任务      | scheduleMicrotask(callback)                   |
| StreamSink         | 流接收器        | add(), addError(), close()                    |
| EventSink          | 事件接收器      | add(), addError(), close()                    |
| StreamTransformer  | 流转换器        | bind(), fromHandlers()                        |
| StreamSubscription | 流订阅          | pause(), resume(), cancel()                   |
| TimeoutException   | 超时异常        | message, duration                             |

---

本文档详细介绍了 Dart 核心库 `dart:async` 中除 `Future` 和 `Stream` 基础内容之外的高级特性，包括 `Timer` 定时器、`Completer` 手动完成器、`Zone` 执行区域、微任务调度以及 `Stream` 相关的高级类。掌握这些内容将帮助开发者更深入地理解和运用 Dart 的异步编程模型，编写出更高效、更健壮的异步代码。
