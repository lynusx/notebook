# dart:developer 详解与实战

## 内容简介

`dart:developer` 是 Dart 核心库之一，提供了与 Dart 开发者工具（如 Dart DevTools）集成的功能。它允许开发者在代码中设置断点、记录日志、发送自定义事件、注册服务协议扩展，以及向性能时间线添加信息。这些功能对于调试、性能分析和构建自定义开发者工具至关重要。

全书共分为6章：

- **第1章 入门与基础**：详细介绍 dart:developer 的作用、适用场景
- **第2章 调试与检查**：深入讲解 debugger()、inspect() 等调试函数
- **第3章 日志记录**：详细介绍 log() 函数及其高级用法
- **第4章 服务协议扩展**：介绍 postEvent()、registerExtension() 和 Service 类
- **第5章 性能时间线**：详细介绍 Timeline、TimelineTask 和 Flow 类
- **第6章 实战应用**：提供自定义日志系统、性能监控等实际案例

---

## 第1章 入门与基础

### 1.1 什么是 dart:developer

`dart:developer` 是 Dart 提供的开发者工具支持库，它架起了应用程序与 Dart 开发者工具（Dart DevTools）之间的桥梁。通过这个库，开发者可以：

1. **调试控制**：在代码中设置条件断点
2. **对象检查**：将对象引用发送到调试器进行查看
3. **日志记录**：发送结构化日志到 DevTools
4. **自定义事件**：向 DevTools 发送自定义事件
5. **服务扩展**：注册自定义的服务协议扩展
6. **性能分析**：向时间线添加性能事件

**Flutter 框架小知识**

**dart:developer 与 dart:io 的区别**

| 特性       | dart:developer   | dart:io        |
| ---------- | ---------------- | -------------- |
| 主要用途   | 开发者工具集成   | 系统 I/O 操作  |
| 目标用户   | 开发者/调试工具  | 应用程序       |
| 运行时依赖 | 需要 VM 服务协议 | 独立运行       |
| 生产环境   | 部分功能可用     | 完全可用       |
| 典型场景   | 调试、性能分析   | 文件、网络操作 |

### 1.2 环境配置

`dart:developer` 是 Dart 核心库，无需额外依赖，直接导入即可使用：

```dart
import 'dart:developer' as dev;
```

**Dart Tips 语法小贴士**

**为什么使用 `as dev` 前缀？**

使用前缀导入可以避免命名冲突，并使代码意图更清晰：

```dart
// ✅ 推荐：使用前缀
import 'dart:developer' as dev;
dev.log('调试信息');

// ❌ 不推荐：不使用前缀
import 'dart:developer';
log('调试信息');  // 可能与自定义 log 函数冲突
```

### 1.3 库结构概览

```
dart:developer
│
├── 调试与检查
│   ├── debugger()          # 触发断点
│   ├── inspect()           # 检查对象
│   └── log()               # 记录日志
│
├── 服务协议
│   ├── Service             # 服务协议访问类
│   ├── ServiceProtocolInfo # 服务协议信息类
│   ├── ServiceExtensionResponse  # 扩展响应类
│   ├── registerExtension() # 注册服务扩展
│   └── postEvent()         # 发送自定义事件
│
├── 性能时间线
│   ├── Timeline            # 时间线操作类
│   ├── TimelineTask        # 异步任务类
│   ├── Flow                # 流程事件类
│   ├── UserTag             # CPU 分析标签
│   └── TimelineRecorder    # 时间线记录器枚举
│
├── 原生运行时
│   ├── NativeRuntime       # 原生运行时功能
│   └── reachabilityBarrier # 可达性屏障属性
│
└── 其他
    ├── extensionStreamHasListener  # 扩展流监听器状态
    ├── getCurrentTag()     # 获取当前用户标签
    ├── addHttpClientProfilingData()  # HTTP 性能分析
    └── getHttpClientProfilingData()  # 获取 HTTP 分析数据
```

### 1.4 使用场景

```dart
import 'dart:developer' as dev;

void main() {
  // 1. 日志记录
  dev.log('应用启动');

  // 2. 条件断点
  final value = calculateValue();
  dev.debugger(when: value < 0, message: '发现负值');

  // 3. 对象检查
  final user = User(name: 'Alice', age: 30);
  dev.inspect(user);

  // 4. 性能分析
  dev.Timeline.timeSync('数据处理', () {
    processData();
  });

  // 5. 发送自定义事件
  dev.postEvent('custom:app_initialized', {
    'timestamp': DateTime.now().toIso8601String(),
    'version': '1.0.0',
  });
}
```

---

## 第2章 调试与检查

### 2.1 debugger 函数

`debugger` 函数用于在代码中设置条件断点。当条件满足时，程序会暂停执行，就像遇到了断点一样。

#### 2.1.1 基本用法

```dart
import 'dart:developer' as dev;

void debuggerExample() {
  var counter = 0;

  for (var i = 0; i < 100; i++) {
    counter++;

    // 当 counter 等于 50 时触发断点
    dev.debugger(when: counter == 50, message: 'Counter reached 50');
  }
}
```

#### 2.1.2 函数签名

```dart
bool debugger({bool when = true, String? message})
```

| 参数      | 类型      | 默认值 | 说明                     |
| --------- | --------- | ------ | ------------------------ |
| `when`    | `bool`    | `true` | 是否触发断点             |
| `message` | `String?` | `null` | 断点消息，显示在调试器中 |

**返回值**：如果断点被触发，返回 `true`；否则返回 `false`。

#### 2.1.3 使用场景

```dart
import 'dart:developer' as dev;

class DebuggerExamples {
  /// 调试异常值
  void debugUnexpectedValue(int value) {
    dev.debugger(
      when: value < 0 || value > 100,
      message: 'Unexpected value: $value',
    );

    processValue(value);
  }

  /// 调试特定条件
  void debugSpecificCondition(User user) {
    dev.debugger(
      when: user.isAdmin && user.age < 18,
      message: 'Admin user is underage: ${user.name}',
    );

    grantPermissions(user);
  }

  /// 调试循环中的特定迭代
  void debugSpecificIteration(List<Item> items) {
    for (var i = 0; i < items.length; i++) {
      final item = items[i];

      dev.debugger(
        when: item.id == 'problematic_id',
        message: 'Found problematic item at index $i',
      );

      processItem(item);
    }
  }

  /// 调试异步操作
  Future<void> debugAsyncOperation() async {
    final result = await fetchData();

    dev.debugger(
      when: result.isEmpty,
      message: 'Empty result from fetchData()',
    );

    processResult(result);
  }
}
```

**Flutter 框架小知识**

**debugger() 与 IDE 断点的区别**

| 特性     | debugger()       | IDE 断点       |
| -------- | ---------------- | -------------- |
| 条件判断 | 代码中动态判断   | IDE 中静态设置 |
| 部署     | 随代码部署       | 仅开发环境     |
| 灵活性   | 可使用任意表达式 | 受 IDE 限制    |
| 团队协作 | 所有人共享       | 个人设置       |
| 生产环境 | 可保留（无影响） | 不存在         |

### 2.2 inspect 函数

`inspect` 函数用于将对象引用发送到任何连接的调试器，方便在 DevTools 中查看对象详情。

#### 2.2.1 基本用法

```dart
import 'dart:developer' as dev;

void inspectExample() {
  final user = User(
    id: '123',
    name: 'Alice',
    email: 'alice@example.com',
    createdAt: DateTime.now(),
  );

  // 将 user 对象发送到调试器
  dev.inspect(user);

  // 继续处理
  saveUser(user);
}

class User {
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });
}
```

#### 2.2.2 函数签名

```dart
Object? inspect(Object? object)
```

| 参数     | 类型      | 说明         |
| -------- | --------- | ------------ |
| `object` | `Object?` | 要检查的对象 |

**返回值**：返回传入的对象，方便链式调用。

#### 2.2.3 使用场景

```dart
import 'dart:developer' as dev;

class InspectExamples {
  /// 检查复杂对象
  void inspectComplexObject() {
    final config = AppConfig(
      apiUrl: 'https://api.example.com',
      timeout: Duration(seconds: 30),
      retryCount: 3,
      features: {
        'darkMode': true,
        'notifications': false,
        'analytics': true,
      },
    );

    // 在 DevTools 中查看完整配置
    dev.inspect(config);
  }

  /// 检查集合
  void inspectCollections() {
    final users = [
      User(id: '1', name: 'Alice'),
      User(id: '2', name: 'Bob'),
      User(id: '3', name: 'Charlie'),
    ];

    // 检查用户列表
    dev.inspect(users);

    final userMap = {
      'alice': users[0],
      'bob': users[1],
      'charlie': users[2],
    };

    // 检查用户映射
    dev.inspect(userMap);
  }

  /// 链式调用
  User createAndInspectUser(String name) {
    return dev.inspect(User(id: generateId(), name: name));
  }

  /// 检查异步结果
  Future<void> inspectAsyncResult() async {
    final result = await fetchUserData();

    // 检查结果
    dev.inspect(result);

    if (result.hasError) {
      // 检查错误详情
      dev.inspect(result.error);
    }
  }

  /// 检查状态变化
  void inspectStateChanges() {
    final state = AppState();

    state.addListener(() {
      // 每次状态变化时检查
      dev.inspect(state);
    });
  }
}
```

### 2.3 log 函数

`log` 函数用于发送日志事件，可以在 DevTools 的 Logging 视图中查看。

#### 2.3.1 基本用法

```dart
import 'dart:developer' as dev;

void logExample() {
  // 简单日志
  dev.log('这是一条普通日志');

  // 带名称的日志
  dev.log('用户登录成功', name: 'Auth');

  // 带错误信息的日志
  try {
    riskyOperation();
  } catch (e, stackTrace) {
    dev.log(
      '操作失败',
      name: 'Error',
      error: e,
      stackTrace: stackTrace,
    );
  }
}
```

#### 2.3.2 函数签名

```dart
void log(
  String message, {
  DateTime? time,
  int? sequenceNumber,
  int level = 0,
  String name = '',
  Zone? zone,
  Object? error,
  StackTrace? stackTrace,
})
```

| 参数             | 类型          | 默认值 | 说明                                   |
| ---------------- | ------------- | ------ | -------------------------------------- |
| `message`        | `String`      | 必填   | 日志消息                               |
| `time`           | `DateTime?`   | `null` | 日志时间                               |
| `sequenceNumber` | `int?`        | `null` | 序列号                                 |
| `level`          | `int`         | `0`    | 日志级别（0=INFO, 1=WARNING, 2=ERROR） |
| `name`           | `String`      | `''`   | 日志名称/类别                          |
| `zone`           | `Zone?`       | `null` | 区域信息                               |
| `error`          | `Object?`     | `null` | 关联的错误对象                         |
| `stackTrace`     | `StackTrace?` | `null` | 关联的堆栈跟踪                         |

#### 2.3.3 日志级别

```dart
import 'dart:developer' as dev;

class LogLevel {
  static const int fine = 0;      // 详细信息
  static const int info = 0;      // 一般信息
  static const int warning = 1;   // 警告
  static const int error = 2;     // 错误
  static const int severe = 2;    // 严重错误
}

void logLevelsExample() {
  // 详细信息
  dev.log('进入函数 processData', level: LogLevel.fine);

  // 一般信息
  dev.log('数据处理完成', level: LogLevel.info);

  // 警告
  dev.log('缓存未命中', level: LogLevel.warning);

  // 错误
  dev.log('数据库连接失败', level: LogLevel.error);
}
```

#### 2.3.4 高级用法

```dart
import 'dart:developer' as dev;
import 'dart:async';

class AdvancedLogging {
  static int _sequenceNumber = 0;

  /// 结构化日志
  void structuredLog(String event, Map<String, dynamic> data) {
    dev.log(
      event,
      name: 'Analytics',
      time: DateTime.now(),
      sequenceNumber: _sequenceNumber++,
    );
  }

  /// 性能日志
  void performanceLog(String operation, int durationMs) {
    dev.log(
      '$operation took ${durationMs}ms',
      name: 'Performance',
      level: durationMs > 1000 ? 1 : 0,
    );
  }

  /// 网络请求日志
  void networkLog({
    required String method,
    required String url,
    required int statusCode,
    required int durationMs,
    Object? error,
  }) {
    dev.log(
      '$method $url - $statusCode (${durationMs}ms)',
      name: 'Network',
      level: statusCode >= 400 ? 2 : 0,
      error: error,
    );
  }

  /// 带区域的日志
  void zonedLog(String message) {
    final zone = Zone.current;
    dev.log(
      message,
      name: 'Zoned',
      zone: zone,
    );
  }

  /// 完整的错误日志
  void errorLog(
    String message,
    Object error,
    StackTrace stackTrace, {
    Map<String, dynamic>? context,
  }) {
    dev.log(
      message,
      name: 'Error',
      level: 2,
      time: DateTime.now(),
      sequenceNumber: _sequenceNumber++,
      error: error,
      stackTrace: stackTrace,
    );

    // 同时记录上下文信息
    if (context != null) {
      dev.log(
        'Context: $context',
        name: 'ErrorContext',
        level: 2,
      );
    }
  }
}
```

---

## 第3章 服务协议扩展

### 3.1 Service 类

`Service` 类提供对 Dart VM 服务协议的访问，允许开发者控制和查询服务协议 Web 服务器。

#### 3.1.1 获取服务信息

```dart
import 'dart:developer' as dev;

void serviceInfoExample() async {
  // 获取服务协议信息
  final info = await dev.Service.getInfo();

  print('服务协议版本: ${info.serverUri}');
  print('服务地址: ${info.serverUri}');

  // 获取当前 Isolate 的 ID
  final isolateId = dev.Service.getIsolateId(Isolate.current);
  print('当前 Isolate ID: $isolateId');
}
```

#### 3.1.2 控制 Web 服务器

```dart
import 'dart:developer' as dev;

void controlWebServerExample() async {
  // 启用服务协议 Web 服务器
  final info = await dev.Service.controlWebServer(enable: true);
  print('Web 服务器地址: ${info.serverUri}');

  // 静默模式启用（不输出到控制台）
  final silentInfo = await dev.Service.controlWebServer(
    enable: true,
    silenceOutput: true,
  );

  // 禁用 Web 服务器
  await dev.Service.controlWebServer(enable: false);
}
```

#### 3.1.3 获取 Isolate 和 Object ID

```dart
import 'dart:developer' as dev;
import 'dart:isolate';

void getIdsExample() {
  // 获取当前 Isolate ID
  final currentIsolateId = dev.Service.getIsolateId(Isolate.current);
  print('当前 Isolate ID: $currentIsolateId');

  // 获取特定 Isolate ID
  final isolate = Isolate.current;
  final isolateId = dev.Service.getIsolateID(isolate);
  print('Isolate ID: $isolateId');

  // 获取对象 ID
  final user = User(name: 'Alice', age: 30);
  final objectId = dev.Service.getObjectId(user);
  print('User 对象 ID: $objectId');
}
```

**Flutter 框架小知识**

**Service.getIsolateId 与 Service.getIsolateID 的区别**

Dart 3.2+ 引入了 `getIsolateId` 作为 `getIsolateID` 的新版本，两者功能相同，但 `getIsolateId` 遵循 Dart 命名规范（小写缩写）。建议在新代码中使用 `getIsolateId`：

```dart
// Dart 3.2+ 推荐
final id = dev.Service.getIsolateId(Isolate.current);

// 旧版本兼容
final id = dev.Service.getIsolateID(Isolate.current);
```

### 3.2 ServiceProtocolInfo 类

`ServiceProtocolInfo` 封装了服务协议的版本号和访问 URI。

```dart
import 'dart:developer' as dev;

void serviceProtocolInfoExample() async {
  final info = await dev.Service.getInfo();

  // 服务协议版本
  print('主版本号: ${info.majorVersion}');
  print('次版本号: ${info.minorVersion}');

  // 服务访问地址
  print('服务器 URI: ${info.serverUri}');

  // 转换为字符串
  print('信息: ${info.toString()}');
}
```

### 3.3 registerExtension 函数

`registerExtension` 用于注册服务协议扩展处理程序，允许开发者创建自定义的 DevTools 扩展。

#### 3.3.1 基本用法

```dart
import 'dart:developer' as dev;
import 'dart:convert';

void registerExtensionExample() {
  // 注册自定义扩展
  dev.registerExtension('ext.myApp.getVersion', (method, parameters) async {
    final version = {
      'version': '1.0.0',
      'buildNumber': '100',
      'commitHash': 'abc123',
    };

    return dev.ServiceExtensionResponse.result(
      jsonEncode(version),
    );
  });
}
```

#### 3.3.2 函数签名

```dart
void registerExtension(
  String method,
  ServiceExtensionHandler handler,
)
```

| 参数      | 类型                      | 说明                             |
| --------- | ------------------------- | -------------------------------- |
| `method`  | `String`                  | 扩展方法名（必须以 `ext.` 开头） |
| `handler` | `ServiceExtensionHandler` | 处理请求的回调函数               |

#### 3.3.3 ServiceExtensionHandler 类型

```dart
typedef ServiceExtensionHandler = Future<ServiceExtensionResponse> Function(
  String method,
  Map<String, String> parameters,
);
```

#### 3.3.4 ServiceExtensionResponse 类

```dart
import 'dart:developer' as dev;
import 'dart:convert';

class ExtensionResponseExamples {
  /// 成功响应
  dev.ServiceExtensionResponse successResponse(dynamic data) {
    return dev.ServiceExtensionResponse.result(
      jsonEncode({'data': data}),
    );
  }

  /// 错误响应
  dev.ServiceExtensionResponse errorResponse(String message) {
    return dev.ServiceExtensionResponse.error(
      dev.ServiceExtensionResponse.invalidParams,
      message,
    );
  }

  /// 自定义错误码响应
  dev.ServiceExtensionResponse customErrorResponse(
    int errorCode,
    String message,
  ) {
    return dev.ServiceExtensionResponse.error(errorCode, message);
  }
}
```

**ServiceExtensionResponse 错误码：**

| 常量            | 值     | 说明     |
| --------------- | ------ | -------- |
| `invalidParams` | -32602 | 参数无效 |

#### 3.3.5 完整示例

```dart
import 'dart:developer' as dev;
import 'dart:convert';

class AppServiceExtensions {
  static void registerAll() {
    _registerVersionExtension();
    _registerConfigExtension();
    _registerStatsExtension();
  }

  /// 注册版本信息扩展
  static void _registerVersionExtension() {
    dev.registerExtension('ext.myApp.version', (method, params) async {
      final version = {
        'appVersion': '1.0.0',
        'dartVersion': Platform.version,
        'platform': Platform.operatingSystem,
      };

      return dev.ServiceExtensionResponse.result(jsonEncode(version));
    });
  }

  /// 注册配置扩展
  static void _registerConfigExtension() {
    dev.registerExtension('ext.myApp.config', (method, params) async {
      final action = params['action'];

      switch (action) {
        case 'get':
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'config': _getConfig()}),
          );
        case 'set':
          final key = params['key'];
          final value = params['value'];
          _setConfig(key, value);
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'success': true}),
          );
        default:
          return dev.ServiceExtensionResponse.error(
            dev.ServiceExtensionResponse.invalidParams,
            'Unknown action: $action',
          );
      }
    });
  }

  /// 注册统计信息扩展
  static void _registerStatsExtension() {
    dev.registerExtension('ext.myApp.stats', (method, params) async {
      final stats = {
        'memoryUsage': _getMemoryUsage(),
        'activeConnections': _getActiveConnections(),
        'cacheSize': _getCacheSize(),
      };

      return dev.ServiceExtensionResponse.result(jsonEncode(stats));
    });
  }

  static Map<String, dynamic> _getConfig() => {};
  static void _setConfig(String? key, String? value) {}
  static int _getMemoryUsage() => 0;
  static int _getActiveConnections() => 0;
  static int _getCacheSize() => 0;
}
```

### 3.4 postEvent 函数

`postEvent` 用于向 DevTools 发送自定义事件，常用于构建自定义开发者工具插件。

#### 3.4.1 基本用法

```dart
import 'dart:developer' as dev;

void postEventExample() {
  // 发送简单事件
  dev.postEvent('myApp:page_viewed', {
    'page': 'home',
    'timestamp': DateTime.now().millisecondsSinceEpoch,
  });

  // 发送复杂事件
  dev.postEvent('myApp:api_request', {
    'method': 'GET',
    'endpoint': '/users',
    'duration': 150,
    'statusCode': 200,
  });
}
```

#### 3.4.2 函数签名

```dart
void postEvent(
  String eventKind,
  Map eventData, {
  String stream = 'Extension',
})
```

| 参数        | 类型     | 默认值        | 说明         |
| ----------- | -------- | ------------- | ------------ |
| `eventKind` | `String` | 必填          | 事件类型标识 |
| `eventData` | `Map`    | 必填          | 事件数据负载 |
| `stream`    | `String` | `'Extension'` | 目标事件流   |

#### 3.4.3 检查监听器状态

```dart
import 'dart:developer' as dev;

void checkListenerExample() {
  // 检查是否有 DevTools 监听扩展事件
  if (dev.extensionStreamHasListener) {
    // 有监听器时才发送事件（避免不必要的开销）
    dev.postEvent('myApp:detailed_event', {
      'data': 'expensive data',
    });
  }
}
```

#### 3.4.4 完整示例

```dart
import 'dart:developer' as dev;

class EventLogger {
  static void logPageView(String pageName, Map<String, dynamic> params) {
    dev.postEvent('myApp:page_view', {
      'page': pageName,
      'params': params,
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  static void logUserAction(String action, {Map<String, dynamic>? metadata}) {
    dev.postEvent('myApp:user_action', {
      'action': action,
      'metadata': metadata ?? {},
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  static void logPerformance(String operation, int durationMs) {
    dev.postEvent('myApp:performance', {
      'operation': operation,
      'duration': durationMs,
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  static void logError(String error, StackTrace stackTrace) {
    dev.postEvent('myApp:error', {
      'error': error,
      'stackTrace': stackTrace.toString(),
      'timestamp': DateTime.now().toIso8601String(),
    });
  }
}
```

---

## 第4章 性能时间线

### 4.1 Timeline 类

`Timeline` 类提供静态方法向 DevTools 性能时间线添加同步和异步事件。

#### 4.1.1 timeSync 方法

`timeSync` 用于测量同步代码块的执行时间。

```dart
import 'dart:developer' as dev;

void timeSyncExample() {
  // 测量同步操作
  dev.Timeline.timeSync('数据处理', () {
    final data = fetchData();
    processData(data);
  });

  // 带参数的时间测量
  final result = dev.Timeline.timeSync<Map<String, dynamic>>(
    'API 请求',
    () => makeApiRequest(),
  );
}
```

#### 4.1.2 timeSync 函数签名

```dart
T timeSync<T>(
  String name,
  TimelineSyncFunction<T> function, {
  Map? arguments,
})
```

| 参数        | 类型                      | 说明           |
| ----------- | ------------------------- | -------------- |
| `name`      | `String`                  | 时间线事件名称 |
| `function`  | `TimelineSyncFunction<T>` | 要测量的函数   |
| `arguments` | `Map?`                    | 额外参数信息   |

#### 4.1.3 startSync / finishSync 方法

用于手动控制时间线事件的开始和结束。

```dart
import 'dart:developer' as dev;

void manualTimelineExample() {
  // 开始时间线事件
  dev.Timeline.startSync('复杂操作');

  try {
    // 执行操作
    step1();
    step2();
    step3();
  } finally {
    // 确保结束时间线事件
    dev.Timeline.finishSync();
  }
}
```

#### 4.1.4 instantSync 方法

`instantSync` 用于记录瞬时事件（没有持续时间）。

```dart
import 'dart:developer' as dev;

void instantSyncExample() {
  // 记录瞬时事件
  dev.Timeline.instantSync('缓存已清除');

  // 带参数的瞬时事件
  dev.Timeline.instantSync(
    '用户登录',
    arguments: {'userId': '12345'},
  );
}
```

#### 4.1.5 完整示例

```dart
import 'dart:developer' as dev;

class TimelineExamples {
  /// 测量数据处理性能
  void processWithTimeline(List<Data> items) {
    dev.Timeline.timeSync('批量处理', () {
      for (var i = 0; i < items.length; i++) {
        dev.Timeline.timeSync('处理单项', () {
          processItem(items[i]);
        }, arguments: {'index': i});
      }
    }, arguments: {'count': items.length});
  }

  /// 测量异步操作（使用 Future）
  Future<void> asyncOperationWithTimeline() async {
    dev.Timeline.startSync('异步操作');

    try {
      await step1();
      await step2();
      await step3();
    } finally {
      dev.Timeline.finishSync();
    }
  }

  /// 记录关键里程碑
  void recordMilestones() {
    dev.Timeline.instantSync('应用启动');

    initializePlugins();
    dev.Timeline.instantSync('插件初始化完成');

    loadUserData();
    dev.Timeline.instantSync('用户数据加载完成');

    setupUI();
    dev.Timeline.instantSync('UI 设置完成');
  }

  /// 嵌套时间线事件
  void nestedTimelineExample() {
    dev.Timeline.timeSync('外层操作', () {
      dev.Timeline.timeSync('内层操作 1', () {
        doSomething();
      });

      dev.Timeline.timeSync('内层操作 2', () {
        doSomethingElse();
      });
    });
  }
}
```

### 4.2 TimelineTask 类

`TimelineTask` 用于表示时间线上的异步任务，支持跨 Isolate 传递。

#### 4.2.1 基本用法

```dart
import 'dart:developer' as dev;

void timelineTaskExample() async {
  // 创建异步任务
  final task = dev.TimelineTask(filterKey: 'network_request');

  // 开始任务
  task.start('API 请求');

  try {
    // 执行异步操作
    final response = await http.get(Uri.parse('https://api.example.com'));

    // 记录中间步骤
    task.instant('收到响应');

    // 解析数据
    final data = jsonDecode(response.body);
    task.instant('解析完成');

    return data;
  } finally {
    // 完成任务
    task.finish();
  }
}
```

#### 4.2.2 跨 Isolate 传递

```dart
import 'dart:developer' as dev;
import 'dart:isolate';

void crossIsolateTimelineExample() async {
  // 在主 Isolate 创建任务
  final task = dev.TimelineTask(filterKey: 'background_processing');
  task.start('后台处理');

  // 获取任务 ID
  final taskId = task.pass();

  // 在子 Isolate 中继续任务
  await Isolate.run(() {
    // 使用 taskId 重建任务
    final subTask = dev.TimelineTask(
      filterKey: 'background_processing',
      parent: taskId,
    );

    subTask.start('子任务处理');

    // 执行处理...
    processData();

    subTask.finish();
  });

  // 完成主任务
  task.finish();
}
```

#### 4.2.3 TimelineTask 方法

| 方法                                     | 说明                             |
| ---------------------------------------- | -------------------------------- |
| `start(String name, {Map? arguments})`   | 开始任务                         |
| `instant(String name, {Map? arguments})` | 记录瞬时事件                     |
| `pass()`                                 | 获取任务 ID，用于跨 Isolate 传递 |
| `finish()`                               | 完成任务                         |

### 4.3 Flow 类

`Flow` 类用于表示异步流程事件，帮助追踪异步操作的因果关系。

#### 4.3.1 基本用法

```dart
import 'dart:developer' as dev;

void flowExample() async {
  // 开始流程
  final flow = dev.Flow.begin(
    id: 1,
    filterKey: 'async_operation',
  );

  try {
    // 执行异步操作
    await operation1();

    // 流程步骤
    dev.Flow.step(flow.id, '步骤 1 完成');

    await operation2();

    dev.Flow.step(flow.id, '步骤 2 完成');

    await operation3();
  } finally {
    // 结束流程
    dev.Flow.end(flow.id);
  }
}
```

#### 4.3.2 Flow 方法

| 方法                                       | 说明         |
| ------------------------------------------ | ------------ |
| `Flow.begin({int? id, String? filterKey})` | 开始新流程   |
| `Flow.step(int id, String name)`           | 记录流程步骤 |
| `Flow.end(int id)`                         | 结束流程     |

### 4.4 TimelineRecorder 枚举

`TimelineRecorder` 定义了 VM 支持的时间线记录器类型。

```dart
import 'dart:developer' as dev;

void timelineRecorderExample() {
  // 可用的记录器类型
  print(dev.TimelineRecorder.endless);
  print(dev.TimelineRecorder.ring);
  print(dev.TimelineRecorder.startup);
}
```

| 值        | 说明             |
| --------- | ---------------- |
| `endless` | 无限记录器       |
| `ring`    | 环形缓冲区记录器 |
| `startup` | 启动记录器       |

### 4.5 TimelineStream 枚举

`TimelineStream` 定义了可以单独启用的事件流。

```dart
import 'dart:developer' as dev;

void timelineStreamExample() {
  // 可用的流
  print(dev.TimelineStream.all);
  print(dev.TimelineStream.api);
  print(dev.TimelineStream.compiler);
  print(dev.TimelineStream.compilerVerbose);
  print(dev.TimelineStream.dart);
  print(dev.TimelineStream.debugger);
  print(dev.TimelineStream.embedder);
  print(dev.TimelineStream.gc);
  print(dev.TimelineStream.isolate);
  print(dev.TimelineStream.vm);
}
```

---

## 第5章 其他功能

### 5.1 UserTag 类

`UserTag` 用于在 DevTools CPU 分析器中对样本进行分组。

#### 5.1.1 基本用法

```dart
import 'dart:developer' as dev;

void userTagExample() {
  // 创建用户标签
  final tag = dev.UserTag('数据处理');

  // 使标签生效
  tag.makeCurrent();

  // 执行需要分析的操作
  processData();

  // 获取当前标签
  final currentTag = dev.getCurrentTag();
  print('当前标签: ${currentTag.label}');
}
```

#### 5.1.2 使用场景

```dart
import 'dart:developer' as dev;

class UserTagExamples {
  /// 标记不同操作阶段
  void tagOperations() {
    final initTag = dev.UserTag('初始化');
    initTag.makeCurrent();
    initializeApp();

    final loadTag = dev.UserTag('数据加载');
    loadTag.makeCurrent();
    loadData();

    final renderTag = dev.UserTag('渲染');
    renderTag.makeCurrent();
    renderUI();
  }

  /// 标记不同功能模块
  void tagModules() {
    // 网络模块
    final networkTag = dev.UserTag('网络请求');
    networkTag.makeCurrent();
    makeNetworkRequests();

    // 数据库模块
    final dbTag = dev.UserTag('数据库操作');
    dbTag.makeCurrent();
    performDatabaseOperations();

    // 缓存模块
    final cacheTag = dev.UserTag('缓存操作');
    cacheTag.makeCurrent();
    performCacheOperations();
  }
}
```

### 5.2 NativeRuntime 类

`NativeRuntime` 提供原生运行时上可用的功能。

```dart
import 'dart:developer' as dev;

void nativeRuntimeExample() {
  // 获取可达性屏障状态
  final barrier = dev.reachabilityBarrier;
  print('可达性屏障: $barrier');
}
```

### 5.3 HTTP 性能分析

`addHttpClientProfilingData` 和 `getHttpClientProfilingData` 用于 HTTP 性能分析。

```dart
import 'dart:developer' as dev;

void httpProfilingExample() {
  // 添加 HTTP 请求分析数据
  dev.addHttpClientProfilingData({
    'url': 'https://api.example.com/users',
    'method': 'GET',
    'duration': 150,
    'statusCode': 200,
  });

  // 获取所有 HTTP 分析数据
  final data = dev.getHttpClientProfilingData();
  for (final request in data) {
    print('${request['method']} ${request['url']}: ${request['duration']}ms');
  }
}
```

---

## 第6章 实战应用

### 6.1 自定义日志系统

```dart
import 'dart:developer' as dev;

class DeveloperLog {
  static const String _defaultName = 'App';

  /// 详细日志
  static void fine(String message, {String name = _defaultName}) {
    dev.log(message, name: name, level: 0);
  }

  /// 信息日志
  static void info(String message, {String name = _defaultName}) {
    dev.log(message, name: name, level: 0);
  }

  /// 警告日志
  static void warning(String message, {String name = _defaultName}) {
    dev.log(message, name: name, level: 1);
  }

  /// 错误日志
  static void error(
    String message, {
    String name = _defaultName,
    Object? error,
    StackTrace? stackTrace,
  }) {
    dev.log(
      message,
      name: name,
      level: 2,
      error: error,
      stackTrace: stackTrace,
    );
  }

  /// 性能日志
  static void performance(String operation, int durationMs) {
    dev.log(
      '$operation took ${durationMs}ms',
      name: 'Performance',
      level: durationMs > 100 ? 1 : 0,
    );
  }

  /// 网络日志
  static void network({
    required String method,
    required String url,
    required int statusCode,
    required int durationMs,
  }) {
    dev.log(
      '$method $url - $statusCode (${durationMs}ms)',
      name: 'Network',
      level: statusCode >= 400 ? 2 : 0,
    );
  }
}
```

### 6.2 性能监控工具

```dart
import 'dart:developer' as dev;

class PerformanceMonitor {
  static final Map<String, Stopwatch> _timers = {};

  /// 开始计时
  static void start(String operation) {
    _timers[operation] = Stopwatch()..start();
    dev.Timeline.startSync(operation);
  }

  /// 结束计时
  static int stop(String operation) {
    final stopwatch = _timers.remove(operation);
    if (stopwatch != null) {
      stopwatch.stop();
      final duration = stopwatch.elapsedMilliseconds;

      dev.Timeline.finishSync();

      // 发送性能事件
      dev.postEvent('perf:operation_complete', {
        'operation': operation,
        'duration': duration,
      });

      return duration;
    }
    return 0;
  }

  /// 测量同步操作
  static T measure<T>(String operation, T Function() fn) {
    return dev.Timeline.timeSync(operation, fn);
  }

  /// 测量异步操作
  static Future<T> measureAsync<T>(
    String operation,
    Future<T> Function() fn,
  ) async {
    final task = dev.TimelineTask(filterKey: operation);
    task.start(operation);

    try {
      return await fn();
    } finally {
      task.finish();
    }
  }
}
```

### 6.3 调试工具类

```dart
import 'dart:developer' as dev;
import 'dart:isolate';

class DebugUtils {
  /// 条件断点
  static void breakpoint({
    required bool condition,
    String? message,
  }) {
    dev.debugger(when: condition, message: message);
  }

  /// 检查对象
  static T inspect<T>(T object, {String? label}) {
    if (label != null) {
      dev.log('Inspecting: $label');
    }
    return dev.inspect(object) as T;
  }

  /// 获取调试信息
  static Future<Map<String, dynamic>> getDebugInfo() async {
    final serviceInfo = await dev.Service.getInfo();

    return {
      'isolateId': dev.Service.getIsolateId(Isolate.current),
      'serverUri': serviceInfo.serverUri?.toString(),
      'majorVersion': serviceInfo.majorVersion,
      'minorVersion': serviceInfo.minorVersion,
      'currentTag': dev.getCurrentTag().label,
    };
  }

  /// 发送自定义事件
  static void emitEvent(String kind, Map<String, dynamic> data) {
    if (dev.extensionStreamHasListener) {
      dev.postEvent(kind, data);
    }
  }

  /// 标记用户标签
  static void tag(String label) {
    final tag = dev.UserTag(label);
    tag.makeCurrent();
  }
}
```

### 6.4 DevTools 扩展插件

```dart
import 'dart:developer' as dev;
import 'dart:convert';

class DevToolsExtension {
  static void initialize() {
    _registerStateExtension();
    _registerNetworkExtension();
    _registerPerformanceExtension();
  }

  /// 注册状态查看扩展
  static void _registerStateExtension() {
    dev.registerExtension('ext.myApp.state', (method, params) async {
      final action = params['action'];

      switch (action) {
        case 'get':
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'state': _getAppState()}),
          );
        case 'set':
          final key = params['key'];
          final value = params['value'];
          _setAppState(key, value);
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'success': true}),
          );
        default:
          return dev.ServiceExtensionResponse.error(
            dev.ServiceExtensionResponse.invalidParams,
            'Unknown action: $action',
          );
      }
    });
  }

  /// 注册网络调试扩展
  static void _registerNetworkExtension() {
    dev.registerExtension('ext.myApp.network', (method, params) async {
      final action = params['action'];

      switch (action) {
        case 'requests':
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'requests': _getNetworkRequests()}),
          );
        case 'clear':
          _clearNetworkRequests();
          return dev.ServiceExtensionResponse.result(
            jsonEncode({'success': true}),
          );
        default:
          return dev.ServiceExtensionResponse.error(
            dev.ServiceExtensionResponse.invalidParams,
            'Unknown action: $action',
          );
      }
    });
  }

  /// 注册性能监控扩展
  static void _registerPerformanceExtension() {
    dev.registerExtension('ext.myApp.performance', (method, params) async {
      return dev.ServiceExtensionResponse.result(
        jsonEncode({
          'memory': _getMemoryUsage(),
          'cpu': _getCpuUsage(),
          'fps': _getFps(),
        }),
      );
    });
  }

  static Map<String, dynamic> _getAppState() => {};
  static void _setAppState(String? key, String? value) {}
  static List<Map<String, dynamic>> _getNetworkRequests() => [];
  static void _clearNetworkRequests() {}
  static Map<String, dynamic> _getMemoryUsage() => {};
  static Map<String, dynamic> _getCpuUsage() => {};
  static double _getFps() => 60.0;
}
```

### 6.5 应用生命周期追踪

```dart
import 'dart:developer' as dev;
import 'package:flutter/widgets.dart';

class AppLifecycleTracker extends WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    dev.Timeline.instantSync(
      'AppLifecycle: $state',
      arguments: {'state': state.toString()},
    );

    dev.postEvent('app:lifecycle_changed', {
      'state': state.toString(),
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  @override
  void didHaveMemoryPressure() {
    dev.log(
      'Memory pressure detected',
      name: 'Lifecycle',
      level: 1,
    );

    dev.postEvent('app:memory_pressure', {
      'timestamp': DateTime.now().toIso8601String(),
    });
  }

  void trackBuild(String widgetName, Duration buildTime) {
    dev.Timeline.timeSync(
      'Build: $widgetName',
      () {},
      arguments: {'duration': buildTime.inMicroseconds},
    );
  }
}
```

---

## 结语

`dart:developer` 是 Dart 开发者工具集成的核心库，通过本书的学习，你应该已经掌握了：

1. **调试功能**：debugger()、inspect()、log() 的使用
2. **服务协议**：Service 类、registerExtension()、postEvent() 的使用
3. **性能分析**：Timeline、TimelineTask、Flow 的使用
4. **其他功能**：UserTag、NativeRuntime、HTTP 性能分析
5. **实战应用**：自定义日志系统、性能监控、DevTools 扩展

在实际开发中，建议：

- **开发环境**：充分利用所有功能进行调试和性能分析
- **生产环境**：谨慎使用，避免性能开销
- **条件编译**：使用 `kDebugMode` 等常量控制调试代码
- **DevTools 配合**：结合 Dart DevTools 进行可视化调试

希望本书能帮助你更好地使用 `dart:developer` 提升开发效率！
