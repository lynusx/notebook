# Talker 库详解

## 目录

1. [概述](#概述)
2. [核心概念](#核心概念)
3. [安装与配置](#安装与配置)
4. [Talker 类详解](#talker-类详解)
5. [TalkerSettings 配置](#talkersettings-配置)
6. [日志级别 (LogLevel)](#日志级别-loglevel)
7. [TalkerData 数据模型](#talkerdata-数据模型)
8. [TalkerLog 日志类](#talkerlog-日志类)
9. [TalkerError 与 TalkerException](#talkererror-与-talkerexception)
10. [TalkerLogger 独立日志器](#talkerlogger-独立日志器)
11. [TalkerObserver 观察者](#talkerobserver-观察者)
12. [TalkerFilter 过滤器](#talkerfilter-过滤器)
13. [格式化器 (Formatter)](#格式化器-formatter)
14. [TalkerHistory 历史记录](#talkerhistory-历史记录)
15. [实战应用示例](#实战应用示例)
16. [附录](#附录)

---

## 概述

Talker 是一个功能强大的 Dart/Flutter 日志和错误处理库，提供了一套完整的日志记录、错误捕获、异常处理和监控解决方案。

### 主要特性

| 特性                | 描述                                                      |
| ------------------- | --------------------------------------------------------- |
| 📝 **多级别日志**   | 支持 debug、info、warning、error、critical 等多种日志级别 |
| 🎨 **彩色输出**     | 控制台彩色日志输出，支持自定义颜色                        |
| 📊 **日志历史**     | 自动保存日志历史，支持查看和导出                          |
| 🔍 **过滤搜索**     | 支持按关键词和类型过滤日志                                |
| 🎯 **异常处理**     | 统一的异常和错误捕获机制                                  |
| 👁️ **观察者模式**   | 支持观察者模式，可集成第三方监控服务                      |
| 🧩 **高度可定制**   | 支持自定义日志格式、颜色、标题等                          |
| 📱 **Flutter 支持** | 提供 Flutter 专属扩展，支持 UI 日志展示                   |

### 包生态系统

| 包名                     | 版本    | 用途                       |
| ------------------------ | ------- | -------------------------- |
| `talker`                 | ^5.1.13 | 核心日志和错误处理功能     |
| `talker_flutter`         | ^5.1.13 | Flutter 扩展，包含 UI 组件 |
| `talker_logger`          | ^5.1.13 | 独立的日志记录器           |
| `talker_dio_logger`      | ^5.1.13 | Dio HTTP 请求日志          |
| `talker_bloc_logger`     | ^5.1.13 | BLoC 状态管理日志          |
| `talker_riverpod_logger` | ^5.1.13 | Riverpod 状态管理日志      |
| `talker_http_logger`     | ^5.1.13 | HTTP 包日志                |
| `talker_chopper_logger`  | ^5.1.13 | Chopper HTTP 日志          |
| `talker_grpc_logger`     | ^5.1.13 | gRPC 调用日志              |

---

## 核心概念

### 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                         Talker                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  TalkerLog  │  │ TalkerError │  │ TalkerException     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│                           │                                 │
│              ┌────────────┼────────────┐                    │
│              ▼            ▼            ▼                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   TalkerData                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                 │
│        ┌──────────────────┼──────────────────┐              │
│        ▼                  ▼                  ▼              │
│  ┌──────────┐      ┌──────────┐      ┌──────────────┐       │
│  │  Stream  │      │ History  │      │   Observer   │       │
│  └──────────┘      └──────────┘      └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件关系

```dart
// Talker 是核心入口类
final talker = Talker(
  settings: TalkerSettings(...),  // 全局配置
  logger: TalkerLogger(...),       // 日志记录器
  observer: MyObserver(),          // 观察者
  filter: MyFilter(),              // 过滤器
  history: MyHistory(),            // 历史记录
  errorHandler: MyErrorHandler(),  // 错误处理器
);
```

---

## 安装与配置

### 添加依赖

```yaml
# pubspec.yaml
dependencies:
  talker: ^5.1.13

  # Flutter 项目额外添加
  talker_flutter: ^5.1.13
```

### 基础初始化

```dart
import 'package:talker/talker.dart';

// 最简单的初始化
final talker = Talker();

// 完整配置初始化
final talker = Talker(
  settings: TalkerSettings(
    enabled: true,
    useHistory: true,
    maxHistoryItems: 1000,
    useConsoleLogs: true,
  ),
  logger: TalkerLogger(),
);
```

### Flutter 项目初始化

```dart
import 'package:talker_flutter/talker_flutter.dart';

void main() {
  // 使用 TalkerFlutter.init() 获取优化后的实例
  final talker = TalkerFlutter.init();

  runApp(MyApp(talker: talker));
}
```

---

## Talker 类详解

`Talker` 是整个库的核心类，提供了日志记录、错误处理和异常捕获的统一接口。

### 构造函数

```dart
Talker({
  TalkerLogger? logger,        // 日志记录器
  TalkerObserver? observer,    // 观察者
  TalkerSettings? settings,    // 全局设置
  TalkerFilter? filter,        // 过滤器
  TalkerErrorHandler? errorHandler,  // 错误处理器
  TalkerHistory? history,      // 历史记录
})
```

### 核心属性

| 属性       | 类型                 | 描述                     |
| ---------- | -------------------- | ------------------------ |
| `settings` | `TalkerSettings`     | 全局配置设置，可读写     |
| `history`  | `List<TalkerData>`   | 日志历史记录列表，只读   |
| `stream`   | `Stream<TalkerData>` | 日志数据流，用于实时监听 |
| `filter`   | `TalkerFilter`       | 日志过滤器，只读         |

```dart
// 访问历史记录
final logs = talker.history;
for (final log in logs) {
  print(log.message);
}

// 监听日志流
talker.stream.listen((data) {
  print('New log: ${data.message}');
});
```

### 日志方法

#### 基础日志方法

```dart
// Debug 级别日志 - 用于开发调试
talker.debug('调试信息');
talker.debug('用户点击了按钮', exception: e, stackTrace: st);

// Info 级别日志 - 一般信息
talker.info('应用启动成功');
talker.info('用户登录', exception: null, stackTrace: null);

// Warning 级别日志 - 警告信息
talker.warning('内存使用率超过 80%');

// Error 级别日志 - 错误信息
talker.error('网络请求失败', exception: dioError, stackTrace: stackTrace);

// Critical 级别日志 - 严重错误
talker.critical('数据库连接失败，应用即将崩溃');

// Verbose 级别日志 - 详细信息
talker.verbose('详细的调试信息，包含大量数据');
```

#### 通用日志方法

```dart
// 使用 log 方法自定义级别
talker.log(
  '自定义日志消息',
  logLevel: LogLevel.info,
  exception: exception,
  stackTrace: stackTrace,
  pen: AnsiPen()..blue(),  // 自定义颜色
);

// 使用自定义日志对象
talker.logCustom(
  MyCustomLog('自定义日志内容'),
);

// 类型化日志（与 logCustom 相同）
talker.logTyped(
  TalkerLog('类型化日志'),
);
```

### 异常处理方法

```dart
// 处理异常（自动区分 Exception 和 Error）
try {
  riskyOperation();
} catch (e, st) {
  talker.handle(e, st, '执行风险操作时出错');
}

// 处理特定异常
try {
  await apiClient.fetchData();
} on DioException catch (e, st) {
  talker.handle(e, st, 'API 请求失败');
} on FormatException catch (e, st) {
  talker.handle(e, st, '数据格式错误');
}
```

### 配置方法

```dart
// 动态配置 Talker
talker.configure(
  settings: TalkerSettings(
    enabled: false,  // 临时禁用
  ),
);

// 启用/禁用
talker.disable();  // 禁用所有日志
talker.enable();   // 启用日志

// 清空历史
talker.cleanHistory();
```

---

## TalkerSettings 配置

`TalkerSettings` 用于配置 Talker 的全局行为。

### 构造函数

```dart
const TalkerSettings({
  bool enabled = true,                    // 是否启用
  bool useHistory = true,                 // 是否使用历史记录
  bool useConsoleLogs = true,             // 是否输出到控制台
  int maxHistoryItems = 1000,             // 最大历史记录数
  Map<String, String>? titles,            // 自定义标题
  Map<String, AnsiPen>? colors,           // 自定义颜色
  TimeFormat timeFormat = TimeFormat.timeAndSeconds,  // 时间格式
})
```

### 属性详解

| 属性              | 类型                   | 默认值           | 描述                 |
| ----------------- | ---------------------- | ---------------- | -------------------- |
| `enabled`         | `bool`                 | `true`           | 主开关，控制所有功能 |
| `useHistory`      | `bool`                 | `true`           | 是否保存日志到历史   |
| `useConsoleLogs`  | `bool`                 | `true`           | 是否输出到控制台     |
| `maxHistoryItems` | `int`                  | `1000`           | 历史记录最大数量     |
| `titles`          | `Map<String, String>`  | `null`           | 自定义日志标题       |
| `colors`          | `Map<String, AnsiPen>` | `null`           | 自定义日志颜色       |
| `timeFormat`      | `TimeFormat`           | `timeAndSeconds` | 时间显示格式         |

### 配置示例

```dart
final settings = TalkerSettings(
  enabled: true,
  useHistory: true,
  maxHistoryItems: 500,
  useConsoleLogs: true,

  // 自定义标题
  titles: {
    'info': 'INFO',
    'error': 'ERROR',
    'debug': 'DEBUG',
    'my_custom_log': 'CUSTOM',
  },

  // 自定义颜色
  colors: {
    'info': AnsiPen()..blue(),
    'error': AnsiPen()..red(),
    'debug': AnsiPen()..gray(),
    'my_custom_log': AnsiPen()..xterm(208),  // 橙色
  },

  timeFormat: TimeFormat.timeAndSeconds,
);

final talker = Talker(settings: settings);
```

### 时间格式选项

```dart
enum TimeFormat {
  none,           // 不显示时间
  timeOnly,       // 仅时间 (HH:mm)
  timeAndSeconds, // 时间+秒 (HH:mm:ss)
  dateAndTime,    // 日期+时间 (yyyy-MM-dd HH:mm:ss)
}
```

### 动态修改配置

```dart
// 修改配置
talker.settings = talker.settings.copyWith(
  enabled: false,
  maxHistoryItems: 2000,
);

// 注册自定义键（用于过滤）
talker.settings.registerKeys(['network', 'database', 'ui']);
```

---

## 日志级别 (LogLevel)

`LogLevel` 枚举定义了日志的优先级级别。

### 日志级别定义

```dart
enum LogLevel {
  verbose,   // 最详细的信息
  debug,     // 调试信息
  info,      // 一般信息
  warning,   // 警告信息
  error,     // 错误信息
  critical,  // 严重错误
}
```

### 优先级顺序

```
verbose < debug < info < warning < error < critical
```

### 使用示例

```dart
// 设置日志级别过滤
final logger = TalkerLogger(
  settings: TalkerLoggerSettings(
    level: LogLevel.warning,  // 只显示 warning 及以上级别
  ),
);

// 不同级别的日志
talker.verbose('最详细的信息，用于深度调试');
talker.debug('调试信息');
talker.info('一般信息');
talker.warning('警告信息');
talker.error('错误信息');
talker.critical('严重错误');
```

---

## TalkerData 数据模型

`TalkerData` 是所有日志数据的基类，定义了通用的数据结构。

### 类层次结构

```
TalkerData (抽象基类)
    ├── TalkerLog      (日志)
    ├── TalkerError    (错误)
    └── TalkerException (异常)
```

### 属性详解

| 属性         | 类型          | 描述         |
| ------------ | ------------- | ------------ |
| `message`    | `String?`     | 日志消息内容 |
| `logLevel`   | `LogLevel?`   | 日志级别     |
| `exception`  | `Object?`     | 异常对象     |
| `error`      | `Error?`      | 错误对象     |
| `stackTrace` | `StackTrace?` | 堆栈跟踪     |
| `title`      | `String`      | 日志标题     |
| `time`       | `DateTime?`   | 日志时间     |
| `pen`        | `AnsiPen?`    | 控制台颜色   |
| `key`        | `String?`     | 日志类型键   |

### 扩展方法

```dart
// FieldsToDisplay 扩展提供格式化显示
final data = TalkerLog('测试消息');

print(data.displayMessage);     // 格式化的消息
print(data.displayError);       // 格式化的错误
print(data.displayException);   // 格式化的异常
print(data.displayStackTrace);  // 格式化的堆栈
print(data.displayTime);        // 格式化的时间
print(data.displayTitleWithTime); // 带时间的标题
```

---

## TalkerLog 日志类

`TalkerLog` 是标准的日志数据类，继承自 `TalkerData`。

### 构造函数

```dart
TalkerLog(
  String? message, {
  String? key,           // 日志类型键
  String? title = 'log', // 标题
  Object? exception,     // 异常
  Error? error,          // 错误
  StackTrace? stackTrace,// 堆栈
  DateTime? time,        // 时间
  AnsiPen? pen,          // 颜色
  LogLevel? logLevel,    // 级别
})
```

### 创建自定义日志类型

```dart
// 定义自定义日志类
class NetworkLog extends TalkerLog {
  final int statusCode;
  final String url;
  final String method;

  NetworkLog({
    required this.statusCode,
    required this.url,
    required this.method,
    String? message,
  }) : super(message ?? '$method $url - $statusCode');

  @override
  String get title => 'NETWORK';

  @override
  String? get key => 'network_log';

  @override
  AnsiPen get pen {
    if (statusCode >= 200 && statusCode < 300) {
      return AnsiPen()..green();
    } else if (statusCode >= 400) {
      return AnsiPen()..red();
    }
    return AnsiPen()..yellow();
  }

  @override
  LogLevel? get logLevel {
    if (statusCode >= 500) return LogLevel.error;
    if (statusCode >= 400) return LogLevel.warning;
    return LogLevel.info;
  }
}

// 使用自定义日志
talker.logCustom(NetworkLog(
  statusCode: 200,
  url: '/api/users',
  method: 'GET',
));
```

---

## TalkerError 与 TalkerException

这两个类用于封装 Dart 的 `Error` 和 `Exception`。

### TalkerError

```dart
// 用于封装 Dart Error
try {
  // 某些可能抛出 Error 的代码
} catch (e, st) {
  if (e is Error) {
    talker.handle(e, st, '发生错误');
  }
}

// TalkerError 属性
final error = TalkerError(
  '错误消息',
  error: error,           // Error 对象
  stackTrace: stackTrace, // 堆栈
  time: DateTime.now(),   // 时间
);
```

### TalkerException

```dart
// 用于封装 Dart Exception
try {
  // 某些可能抛出 Exception 的代码
} catch (e, st) {
  if (e is Exception) {
    talker.handle(e, st, '发生异常');
  }
}

// TalkerException 属性
final exception = TalkerException(
  '异常消息',
  exception: exception,   // Exception 对象
  stackTrace: stackTrace, // 堆栈
  time: DateTime.now(),   // 时间
);
```

### 错误处理最佳实践

```dart
Future<void> performOperation() async {
  try {
    await riskyAsyncOperation();
  } on FormatException catch (e, st) {
    // 特定异常处理
    talker.handle(e, st, '数据格式错误');
    rethrow;
  } on TimeoutException catch (e, st) {
    talker.handle(e, st, '操作超时');
    // 重试逻辑
  } catch (e, st) {
    // 通用异常处理
    talker.handle(e, st, '未知错误');
    rethrow;
  } finally {
    talker.debug('操作结束');
  }
}
```

---

## TalkerLogger 独立日志器

`TalkerLogger` 是一个独立的日志记录器，可以单独使用。

### 构造函数

```dart
TalkerLogger({
  TalkerLoggerSettings? settings,  // 日志器设置
  LoggerFormatter formatter = const ExtendedLoggerFormatter(),  // 格式化器
  LoggerFilter? filter,            // 过滤器
  LoggerOutput? output,            // 输出函数
})
```

### 基础使用

```dart
// 创建独立日志器
final logger = TalkerLogger();

// 不同级别的日志
logger.debug('调试信息');
logger.info('一般信息');
logger.warning('警告信息');
logger.error('错误信息');
logger.critical('严重错误');
logger.verbose('详细信息');

// 自定义日志
logger.log(
  '自定义日志',
  level: LogLevel.info,
  pen: AnsiPen()..purple(),
);
```

### 日志器设置

```dart
final logger = TalkerLogger(
  settings: TalkerLoggerSettings(
    level: LogLevel.debug,  // 最低日志级别
    colors: {
      LogLevel.debug: AnsiPen()..gray(),
      LogLevel.info: AnsiPen()..blue(),
      LogLevel.warning: AnsiPen()..yellow(),
      LogLevel.error: AnsiPen()..red(),
      LogLevel.critical: AnsiPen()..xterm(196),
    },
  ),
);
```

### 自定义格式化器

```dart
// 创建自定义格式化器
class JsonFormatter implements LoggerFormatter {
  @override
  String fmt(LogDetails details, TalkerLoggerSettings settings) {
    final data = {
      'message': details.message?.toString(),
      'level': details.level?.name,
      'time': DateTime.now().toIso8601String(),
    };
    return jsonEncode(data);
  }
}

// 使用自定义格式化器
final logger = TalkerLogger(
  formatter: JsonFormatter(),
);
```

### 自定义输出

```dart
// 自定义输出函数 - 写入文件
final logger = TalkerLogger(
  output: (String message) async {
    final file = File('app.log');
    await file.writeAsString('$message\n', mode: FileMode.append);
  },
);
```

---

## TalkerObserver 观察者

`TalkerObserver` 用于监听 Talker 的所有事件，可集成第三方监控服务。

### 基础实现

```dart
abstract class TalkerObserver {
  const TalkerObserver();

  // 当产生日志时调用
  void onLog(TalkerData log) {}

  // 当发生错误时调用
  void onError(TalkerError err) {}

  // 当发生异常时调用
  void onException(TalkerException err) {}
}
```

### 自定义观察者

```dart
// 控制台观察者
class ConsoleObserver extends TalkerObserver {
  @override
  void onLog(TalkerData log) {
    print('[OBSERVER] Log: ${log.message}');
  }

  @override
  void onError(TalkerError err) {
    print('[OBSERVER] Error: ${err.error}');
  }

  @override
  void onException(TalkerException err) {
    print('[OBSERVER] Exception: ${err.exception}');
  }
}

// 分析服务观察者
class AnalyticsObserver extends TalkerObserver {
  final AnalyticsService service;

  AnalyticsObserver(this.service);

  @override
  void onLog(TalkerData log) {
    // 发送日志到分析服务
    service.trackEvent('log', {
      'message': log.message,
      'level': log.logLevel?.name,
    });
  }

  @override
  void onError(TalkerError err) {
    service.trackError('error', {
      'error': err.error.toString(),
      'stackTrace': err.stackTrace.toString(),
    });
  }

  @override
  void onException(TalkerException err) {
    service.trackError('exception', {
      'exception': err.exception.toString(),
      'stackTrace': err.stackTrace.toString(),
    });
  }
}

// 使用观察者
final talker = Talker(
  observer: ConsoleObserver(),
);
```

### Firebase Crashlytics 集成

```dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

class CrashlyticsObserver extends TalkerObserver {
  @override
  void onError(TalkerError err) {
    FirebaseCrashlytics.instance.recordError(
      err.error,
      err.stackTrace,
      reason: err.message,
      fatal: false,
    );
  }

  @override
  void onException(TalkerException err) {
    FirebaseCrashlytics.instance.recordError(
      err.exception,
      err.stackTrace,
      reason: err.message,
      fatal: false,
    );
  }
}

final talker = Talker(
  observer: CrashlyticsObserver(),
);
```

### Sentry 集成

```dart
import 'package:sentry_flutter/sentry_flutter.dart';

class SentryObserver extends TalkerObserver {
  @override
  void onError(TalkerError err) {
    Sentry.captureException(
      err.error,
      stackTrace: err.stackTrace,
      hint: Hint.withMap({'message': err.message}),
    );
  }

  @override
  void onException(TalkerException err) {
    Sentry.captureException(
      err.exception,
      stackTrace: err.stackTrace,
      hint: Hint.withMap({'message': err.message}),
    );
  }
}
```

---

## TalkerFilter 过滤器

`TalkerFilter` 用于过滤日志，控制哪些日志被显示或保存。

### 基础过滤器

```dart
// 使用内置的日志级别过滤器
final filter = LogLevelFilter(
  LogLevel.warning,  // 只显示 warning 及以上级别
);

final talker = Talker(
  filter: filter,
);
```

### 自定义过滤器

```dart
// 关键词过滤器
class KeywordFilter extends TalkerFilter {
  final List<String> keywords;

  KeywordFilter(this.keywords);

  @override
  bool shouldDisplay(TalkerData data) {
    final message = data.message?.toLowerCase() ?? '';
    return keywords.any((keyword) =>
      message.contains(keyword.toLowerCase()));
  }
}

// 类型过滤器
class TypeFilter extends TalkerFilter {
  final List<String> allowedKeys;

  TypeFilter(this.allowedKeys);

  @override
  bool shouldDisplay(TalkerData data) {
    return allowedKeys.contains(data.key);
  }
}

// 组合过滤器
class CombinedFilter extends TalkerFilter {
  final LogLevel minLevel;
  final List<String>? allowedKeys;

  CombinedFilter({
    required this.minLevel,
    this.allowedKeys,
  });

  @override
  bool shouldDisplay(TalkerData data) {
    // 级别过滤
    final level = data.logLevel;
    if (level != null && level.index < minLevel.index) {
      return false;
    }

    // 类型过滤
    if (allowedKeys != null && !allowedKeys!.contains(data.key)) {
      return false;
    }

    return true;
  }
}

// 使用自定义过滤器
final talker = Talker(
  filter: CombinedFilter(
    minLevel: LogLevel.info,
    allowedKeys: ['network', 'database'],
  ),
);
```

---

## 格式化器 (Formatter)

格式化器控制日志的输出格式。

### 内置格式化器

```dart
// 扩展格式化器（默认）- 包含时间、标题、消息
ExtendedLoggerFormatter()

// 彩色格式化器 - 只输出彩色消息
ColoredLoggerFormatter()
```

### 自定义格式化器

```dart
// CSV 格式化器
class CsvFormatter implements LoggerFormatter {
  @override
  String fmt(LogDetails details, TalkerLoggerSettings settings) {
    final time = DateTime.now().toIso8601String();
    final level = details.level?.name ?? 'unknown';
    final message = details.message?.toString().replaceAll(',', '\\,') ?? '';
    return '$time,$level,"$message"';
  }
}

// Markdown 格式化器
class MarkdownFormatter implements LoggerFormatter {
  @override
  String fmt(LogDetails details, TalkerLoggerSettings settings) {
    final time = DateTime.now().toString();
    final level = details.level?.name.toUpperCase() ?? 'LOG';
    final message = details.message?.toString() ?? '';

    return '''
## $level - $time
$message
---
''';
  }
}

// HTML 格式化器
class HtmlFormatter implements LoggerFormatter {
  @override
  String fmt(LogDetails details, TalkerLoggerSettings settings) {
    final level = details.level?.name ?? 'log';
    final message = details.message?.toString() ?? '';

    String color;
    switch (details.level) {
      case LogLevel.error:
        color = 'red';
        break;
      case LogLevel.warning:
        color = 'orange';
        break;
      case LogLevel.info:
        color = 'blue';
        break;
      default:
        color = 'black';
    }

    return '<div style="color: $color;">[$level] $message</div>';
  }
}

// 使用自定义格式化器
final logger = TalkerLogger(
  formatter: MarkdownFormatter(),
);
```

---

## TalkerHistory 历史记录

`TalkerHistory` 用于管理日志历史记录。

### 默认历史实现

```dart
// 使用默认历史实现
final talker = Talker(
  history: DefaultTalkerHistory(
    maxItems: 1000,  // 最大记录数
  ),
);

// 访问历史
final history = talker.history;
print('历史记录数: ${history.length}');

// 清空历史
talker.cleanHistory();
```

### 自定义历史实现

```dart
// 持久化历史 - 保存到文件
class FileHistory implements TalkerHistory {
  final String filePath;
  final List<TalkerData> _history = [];

  FileHistory(this.filePath);

  @override
  List<TalkerData> get history => List.unmodifiable(_history);

  @override
  void add(TalkerData data) {
    _history.add(data);
    _persist();
  }

  @override
  void clear() {
    _history.clear();
    _persist();
  }

  Future<void> _persist() async {
    final file = File(filePath);
    final lines = _history.map((d) => jsonEncode({
      'message': d.message,
      'time': d.time?.toIso8601String(),
      'level': d.logLevel?.name,
    })).toList();
    await file.writeAsString(lines.join('\n'));
  }
}

// 内存限制历史
class LimitedMemoryHistory implements TalkerHistory {
  final int maxItems;
  final List<TalkerData> _history = [];

  LimitedMemoryHistory({required this.maxItems});

  @override
  List<TalkerData> get history => List.unmodifiable(_history);

  @override
  void add(TalkerData data) {
    _history.add(data);
    if (_history.length > maxItems) {
      _history.removeAt(0);
    }
  }

  @override
  void clear() => _history.clear();
}
```

---

## 实战应用示例

### 示例 1：完整的应用日志系统

```dart
import 'package:talker/talker.dart';
import 'package:talker_flutter/talker_flutter.dart';

class AppLogger {
  static late final Talker _talker;

  static void initialize() {
    _talker = TalkerFlutter.init(
      settings: TalkerSettings(
        enabled: true,
        useHistory: true,
        maxHistoryItems: 2000,
        useConsoleLogs: true,
        titles: {
          'info': 'ℹ️ INFO',
          'error': '❌ ERROR',
          'debug': '🐛 DEBUG',
          'warning': '⚠️ WARNING',
          'critical': '🔴 CRITICAL',
        },
        colors: {
          'info': AnsiPen()..blue(),
          'error': AnsiPen()..red(),
          'debug': AnsiPen()..gray(),
          'warning': AnsiPen()..yellow(),
          'critical': AnsiPen()..xterm(196),
        },
      ),
    );
  }

  static Talker get instance => _talker;

  // 快捷方法
  static void debug(String msg, {Object? exception, StackTrace? stackTrace}) {
    _talker.debug(msg, exception, stackTrace);
  }

  static void info(String msg, {Object? exception, StackTrace? stackTrace}) {
    _talker.info(msg, exception, stackTrace);
  }

  static void warning(String msg, {Object? exception, StackTrace? stackTrace}) {
    _talker.warning(msg, exception, stackTrace);
  }

  static void error(String msg, {Object? exception, StackTrace? stackTrace}) {
    _talker.error(msg, exception, stackTrace);
  }

  static void critical(String msg, {Object? exception, StackTrace? stackTrace}) {
    _talker.critical(msg, exception, stackTrace);
  }

  static void handle(Object exception, StackTrace? stackTrace, {String? msg}) {
    _talker.handle(exception, stackTrace, msg);
  }

  static List<TalkerData> get history => _talker.history;

  static void clearHistory() => _talker.cleanHistory();
}

// 使用
void main() {
  AppLogger.initialize();

  AppLogger.info('应用启动');
  AppLogger.debug('初始化完成');

  try {
    riskyOperation();
  } catch (e, st) {
    AppLogger.handle(e, st, msg: '操作失败');
  }
}
```

### 示例 2：网络请求日志

```dart
import 'package:dio/dio.dart';
import 'package:talker_dio_logger/talker_dio_logger.dart';

class ApiClient {
  late final Dio _dio;
  late final Talker _talker;

  ApiClient(this._talker) {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 30),
      receiveTimeout: Duration(seconds: 30),
    ));

    // 添加 Talker Dio 日志拦截器
    _dio.interceptors.add(
      TalkerDioLogger(
        talker: _talker,
        settings: TalkerDioLoggerSettings(
          printRequestHeaders: true,
          printResponseHeaders: true,
          printResponseMessage: true,
          printRequestData: true,
          printResponseData: true,
          // 自定义颜色
          requestPen: AnsiPen()..blue(),
          responsePen: AnsiPen()..green(),
          errorPen: AnsiPen()..red(),
        ),
      ),
    );
  }

  Future<Response> get(String path, {Map<String, dynamic>? query}) async {
    _talker.info('GET 请求: $path');
    try {
      final response = await _dio.get(path, queryParameters: query);
      _talker.info('GET 成功: $path - ${response.statusCode}');
      return response;
    } catch (e, st) {
      _talker.handle(e, st, 'GET 请求失败: $path');
      rethrow;
    }
  }

  Future<Response> post(String path, {dynamic data}) async {
    _talker.info('POST 请求: $path');
    try {
      final response = await _dio.post(path, data: data);
      _talker.info('POST 成功: $path - ${response.statusCode}');
      return response;
    } catch (e, st) {
      _talker.handle(e, st, 'POST 请求失败: $path');
      rethrow;
    }
  }
}
```

### 示例 3：BLoC 状态管理日志

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:talker_bloc_logger/talker_bloc_logger.dart';

class MyApp extends StatelessWidget {
  final Talker talker;

  MyApp({required this.talker});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider(
          create: (_) => AuthBloc(),
        ),
        BlocProvider(
          create: (_) => UserBloc(),
        ),
      ],
      child: BlocObserver(
        observer: TalkerBlocObserver(
          talker: talker,
          settings: TalkerBlocLoggerSettings(
            printEvents: true,
            printTransitions: true,
            printChanges: true,
            printCreations: true,
            printClosings: true,
            // 截断长状态输出
            truncateStateData: true,
            maxStateLength: 500,
          ),
        ),
        child: MaterialApp(
          title: 'My App',
          home: HomeScreen(),
        ),
      ),
    );
  }
}
```

### 示例 4：Riverpod 状态管理日志

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:talker_riverpod_logger/talker_riverpod_logger.dart';

void main() {
  final talker = TalkerFlutter.init();

  runApp(
    ProviderScope(
      observers: [
        TalkerRiverpodObserver(
          talker: talker,
          settings: TalkerRiverpodLoggerSettings(
            enabled: true,
            printProviderAdded: true,
            printProviderUpdated: true,
            printProviderDisposed: true,
            printProviderFailed: true,
            // 截断状态数据
            printStateFullData: false,
            // 过滤特定 Provider
            eventFilter: (provider) {
              return provider.name?.contains('Auth') ?? false;
            },
          ),
        ),
      ],
      child: MyApp(),
    ),
  );
}
```

### 示例 5：Flutter UI 日志查看器

```dart
import 'package:talker_flutter/talker_flutter.dart';

class LogViewerScreen extends StatelessWidget {
  final Talker talker;

  LogViewerScreen({required this.talker});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('日志查看器'),
        actions: [
          IconButton(
            icon: Icon(Icons.delete),
            onPressed: () => talker.cleanHistory(),
          ),
          IconButton(
            icon: Icon(Icons.share),
            onPressed: () => _shareLogs(context),
          ),
        ],
      ),
      body: TalkerScreen(
        talker: talker,
        theme: TalkerScreenTheme(
          backgroundColor: Colors.black,
          textColor: Colors.white,
        ),
      ),
    );
  }

  void _shareLogs(BuildContext context) async {
    final logs = talker.history
        .map((d) => '${d.displayTime} ${d.title}: ${d.message}')
        .join('\n');

    // 使用 share_plus 分享日志
    await Share.share(logs, subject: '应用日志');
  }
}
```

### 示例 6：错误边界包装器

```dart
import 'package:talker_flutter/talker_flutter.dart';

class ErrorBoundary extends StatelessWidget {
  final Widget child;
  final Talker talker;

  ErrorBoundary({required this.child, required this.talker});

  @override
  Widget build(BuildContext context) {
    return TalkerWrapper(
      talker: talker,
      options: TalkerWrapperOptions(
        // 启用错误弹窗
        enableErrorAlerts: true,
        // 自定义错误弹窗
        errorAlertBuilder: (context, data) {
          return AlertDialog(
            title: Row(
              children: [
                Icon(Icons.error, color: Colors.red),
                SizedBox(width: 8),
                Text('发生错误'),
              ],
            ),
            content: Text(data.message ?? '未知错误'),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: Text('确定'),
              ),
              TextButton(
                onPressed: () {
                  // 查看详细错误
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => TalkerScreen(talker: talker),
                    ),
                  );
                },
                child: Text('查看详情'),
              ),
            ],
          );
        },
      ),
      child: child,
    );
  }
}
```

### 示例 7：路由观察器

```dart
import 'package:talker_flutter/talker_flutter.dart';

class MyApp extends StatelessWidget {
  final Talker talker;
  final _router = GoRouter(
    routes: [...],
    observers: [
      // GoRouter 使用方式
      TalkerRouteObserver(talker),
    ],
  );

  MyApp({required this.talker});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
      // 或者使用 Navigator
      // navigatorObservers: [TalkerRouteObserver(talker)],
    );
  }
}

// 使用 auto_route
@MaterialAutoRouter(
  routes: [...],
)
class $AppRouter {}

// 配置
final appRouter = AppRouter(
  navigatorObservers: () => [TalkerRouteObserver(talker)],
);
```

### 示例 8：日志文件导出

```dart
import 'package:path_provider/path_provider.dart';
import 'package:share_plus/share_plus.dart';

class LogExporter {
  final Talker talker;

  LogExporter(this.talker);

  /// 导出日志为文本文件
  Future<String> exportToFile() async {
    final directory = await getApplicationDocumentsDirectory();
    final file = File('${directory.path}/logs.txt');

    final buffer = StringBuffer();
    buffer.writeln('=== 应用日志导出 ===');
    buffer.writeln('导出时间: ${DateTime.now()}');
    buffer.writeln('');

    for (final log in talker.history) {
      buffer.writeln('[${log.displayTime}] ${log.title}: ${log.message}');
      if (log.error != null) {
        buffer.writeln('Error: ${log.error}');
      }
      if (log.exception != null) {
        buffer.writeln('Exception: ${log.exception}');
      }
      if (log.stackTrace != null) {
        buffer.writeln('StackTrace: ${log.stackTrace}');
      }
      buffer.writeln('---');
    }

    await file.writeAsString(buffer.toString());
    return file.path;
  }

  /// 分享日志文件
  Future<void> shareLogs() async {
    final path = await exportToFile();
    await Share.shareXFiles([XFile(path)], subject: '应用日志');
  }

  /// 导出为 JSON
  Future<String> exportToJson() async {
    final directory = await getApplicationDocumentsDirectory();
    final file = File('${directory.path}/logs.json');

    final logs = talker.history.map((log) => {
      'time': log.time?.toIso8601String(),
      'title': log.title,
      'message': log.message,
      'level': log.logLevel?.name,
      'key': log.key,
      'error': log.error?.toString(),
      'exception': log.exception?.toString(),
      'stackTrace': log.stackTrace?.toString(),
    }).toList();

    await file.writeAsString(jsonEncode({
      'exportTime': DateTime.now().toIso8601String(),
      'logs': logs,
    }));

    return file.path;
  }
}
```

---

## 附录

### A. 版本兼容性

| 版本   | Dart SDK | Flutter  | 说明           |
| ------ | -------- | -------- | -------------- |
| 5.1.13 | >=3.0.0  | >=3.10.0 | 当前最新稳定版 |
| 4.x    | >=2.17.0 | >=3.0.0  | 旧版本         |

### B. 完整配置示例

```dart
import 'package:talker/talker.dart';
import 'package:talker_flutter/talker_flutter.dart';

void main() {
  final talker = TalkerFlutter.init(
    settings: TalkerSettings(
      enabled: true,
      useHistory: true,
      maxHistoryItems: 1000,
      useConsoleLogs: true,
      timeFormat: TimeFormat.timeAndSeconds,
      titles: {
        'info': 'INFO',
        'error': 'ERROR',
        'debug': 'DEBUG',
        'warning': 'WARN',
        'verbose': 'VERBOSE',
        'critical': 'CRITICAL',
      },
      colors: {
        'info': AnsiPen()..blue(),
        'error': AnsiPen()..red(),
        'debug': AnsiPen()..gray(),
        'warning': AnsiPen()..yellow(),
        'verbose': AnsiPen()..cyan(),
        'critical': AnsiPen()..xterm(196),
      },
    ),
    logger: TalkerLogger(
      settings: TalkerLoggerSettings(
        level: LogLevel.debug,
      ),
      formatter: ExtendedLoggerFormatter(),
    ),
    observer: MyCustomObserver(),
    filter: LogLevelFilter(LogLevel.debug),
  );

  runApp(MyApp(talker: talker));
}
```

### C. 最佳实践

1. **使用单例模式管理 Talker 实例**

   ```dart
   class Log {
     static final Talker _instance = TalkerFlutter.init();
     static Talker get I => _instance;
   }
   ```

2. **区分开发和生产环境**

   ```dart
   final talker = Talker(
     settings: TalkerSettings(
       enabled: kDebugMode,  // 只在调试模式启用
       useConsoleLogs: kDebugMode,
     ),
   );
   ```

3. **集成错误监控服务**
   - 使用 `TalkerObserver` 集成 Crashlytics、Sentry 等

4. **定期清理历史记录**

   ```dart
   // 在应用启动时清理旧日志
   talker.cleanHistory();
   ```

5. **敏感信息过滤**
   ```dart
   class SensitiveDataFilter extends TalkerFilter {
     @override
     bool shouldDisplay(TalkerData data) {
       final message = data.message?.toLowerCase() ?? '';
       // 过滤包含敏感信息的关键词
       return !message.contains('password') &&
              !message.contains('token') &&
              !message.contains('secret');
     }
   }
   ```

### D. 常见问题

**Q: iOS 控制台不显示彩色日志？**
A: 使用 `TalkerFlutter.init()` 初始化，它会自动适配平台。

**Q: 如何禁用特定类型的日志？**
A: 使用过滤器或设置 `enabled: false` 临时禁用。

**Q: 日志太长被截断？**
A: 这是平台限制，使用 `TalkerScreen` 查看完整日志。

**Q: 如何在 release 模式保留日志？**
A: 设置 `enabled: true` 并配置历史记录，但不要输出到控制台。

### E. 参考资源

- [Pub 包页面](https://pub.dev/packages/talker)
- [GitHub 仓库](https://github.com/Frezyx/talker)
- [API 文档](https://pub.dev/documentation/talker/latest/)
- [在线演示](https://frezyx.github.io/talker/)
