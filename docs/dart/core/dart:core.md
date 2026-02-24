# Dart 核心库 dart:core 详解

`dart:core` 是 Dart 语言的核心库，它定义了 Dart 语言最基本的类和功能。这个库会自动导入到每一个 Dart 程序中，无需显式使用 `import` 语句。`dart:core` 库包含了 Dart 语言的基础类型（如 `int`、`double`、`String`、`bool`）、集合类型（如 `List`、`Map`、`Set`）、日期时间处理、URI 处理、正则表达式、错误处理等核心功能。

本文档将详细介绍 `dart:core` 库中定义的各类核心类及其属性和方法。需要注意的是，`num`、`int`、`double`、`String`、`bool`、`List`、`Map`、`Set` 等基本数据类型已在《Dart 数据类型详解》中详细介绍，本文档将不再重复。本文档重点介绍 `DateTime`、`Duration`、`Uri`、`RegExp`、`Comparable`、`Comparator` 等实用类，帮助开发者全面掌握 Dart 核心库的使用。

## 第1章 Object 类

`Object` 是 Dart 中所有类的根类，每个类都直接或间接地继承自 `Object`。`Object` 类定义了一些所有对象共有的基本方法。

### 1.1 基本方法

#### 1.1.1 toString()

`toString()` 方法返回对象的字符串表示。默认实现返回 `"Instance of 'ClassName'"`，但通常应该在子类中重写以提供更有意义的表示。

```dart
class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  @override
  String toString() => 'Person(name: $name, age: $age)';
}

void main() {
  var person = Person('Alice', 25);
  print(person.toString());  // 输出: Person(name: Alice, age: 25)
  print(person);             // 输出: Person(name: Alice, age: 25)（隐式调用 toString）
}
```

#### 1.1.2 == 运算符

`==` 运算符用于比较两个对象是否相等。默认实现比较对象的标识（identity），即两个引用是否指向同一个对象。通常应该在值类中重写以比较对象的内容。

```dart
class Point {
  final int x;
  final int y;
  
  const Point(this.x, this.y);
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    if (other is! Point) return false;
    return x == other.x && y == other.y;
  }
  
  @override
  int get hashCode => Object.hash(x, y);
}

void main() {
  var p1 = Point(1, 2);
  var p2 = Point(1, 2);
  var p3 = Point(2, 3);
  
  print(p1 == p2);  // 输出: true（值相等）
  print(p1 == p3);  // 输出: false
  print(identical(p1, p2));  // 输出: false（不同对象）
}
```

#### 1.1.3 hashCode

`hashCode` 是一个整数，用于支持基于哈希的集合（如 `Set` 和 `Map`）。如果重写了 `==` 运算符，必须同时重写 `hashCode`，确保相等的对象具有相同的哈希码。

```dart
class Product {
  final String id;
  final String name;
  
  Product(this.id, this.name);
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    if (other is! Product) return false;
    return id == other.id;  // 只比较 id
  }
  
  @override
  int get hashCode => id.hashCode;  // 与 equals 一致
}

void main() {
  var product1 = Product('P001', 'iPhone');
  var product2 = Product('P001', 'iPhone 15');  // 名称不同但 id 相同
  
  print(product1 == product2);  // 输出: true
  
  var set = {product1};
  print(set.contains(product2));  // 输出: true（hashCode 一致）
}
```

#### 1.1.4 runtimeType

`runtimeType` 返回对象的运行时类型。

```dart
void main() {
  var name = 'Alice';
  var age = 25;
  var price = 19.99;
  
  print(name.runtimeType);   // 输出: String
  print(age.runtimeType);    // 输出: int
  print(price.runtimeType);  // 输出: double
  
  // 类型检查
  if (name.runtimeType == String) {
    print('name is a String');
  }
  
  // 注意：通常使用 is 运算符进行类型检查
  if (name is String) {
    print('name is a String (using is)');
  }
}
```

#### 1.1.5 noSuchMethod()

`noSuchMethod()` 在调用对象不存在的方法时被调用。可以用于实现动态代理或模拟对象。

```dart
class DynamicProxy {
  @override
  dynamic noSuchMethod(Invocation invocation) {
    print('Called: ${invocation.memberName}');
    print('Arguments: ${invocation.positionalArguments}');
    return null;
  }
}

void main() {
  dynamic proxy = DynamicProxy();
  proxy.someMethod(1, 2, 3);  // 不会报错，调用 noSuchMethod
}
```

## 第2章 DateTime 类

`DateTime` 类用于表示日期和时间。它支持从 Unix 纪元（1970年1月1日）到大约公元275,000年的时间范围。

### 2.1 创建 DateTime 对象

#### 2.1.1 构造函数

```dart
void main() {
  // 1. 当前时间
  var now = DateTime.now();
  print(now);  // 输出: 2024-01-15 10:30:45.123456
  
  // 2. 指定日期时间（本地时间）
  var specific = DateTime(2024, 1, 15, 10, 30, 0);
  print(specific);  // 输出: 2024-01-15 10:30:00.000
  
  // 3. 从 Unix 时间戳创建（毫秒）
  var fromMillis = DateTime.fromMillisecondsSinceEpoch(1705314000000);
  print(fromMillis);
  
  // 4. 从 Unix 时间戳创建（微秒）
  var fromMicros = DateTime.fromMicrosecondsSinceEpoch(1705314000000000);
  print(fromMicros);
  
  // 5. UTC 时间
  var utc = DateTime.utc(2024, 1, 15, 10, 30, 0);
  print(utc);  // 输出: 2024-01-15 10:30:00.000Z
  
  // 6. 解析 ISO 8601 字符串
  var parsed = DateTime.parse('2024-01-15T10:30:00.000Z');
  print(parsed);
  
  // 7. 尝试解析（可能返回 null）
  var tryParsed = DateTime.tryParse('invalid');
  print(tryParsed);  // 输出: null
}
```

### 2.2 DateTime 的属性

#### 2.2.1 日期组件

```dart
void main() {
  var date = DateTime(2024, 1, 15, 10, 30, 45, 123, 456);
  
  // 年
  print(date.year);         // 输出: 2024
  
  // 月（1-12）
  print(date.month);        // 输出: 1
  
  // 日（1-31）
  print(date.day);          // 输出: 15
  
  // 时（0-23）
  print(date.hour);         // 输出: 10
  
  // 分（0-59）
  print(date.minute);       // 输出: 30
  
  // 秒（0-59）
  print(date.second);       // 输出: 45
  
  // 毫秒（0-999）
  print(date.millisecond);  // 输出: 123
  
  // 微秒（0-999）
  print(date.microsecond);  // 输出: 456
}
```

#### 2.2.2 星期相关

```dart
void main() {
  var date = DateTime(2024, 1, 15);  // 2024年1月15日是星期一
  
  // 星期几（1=星期一，7=星期日）
  print(date.weekday);  // 输出: 1
  
  // 使用常量
  print(date.weekday == DateTime.monday);  // 输出: true
  
  // 获取星期名称
  var weekdayNames = ['', '周一', '周二', '周三', '周四', '周五', '周六', '周日'];
  print(weekdayNames[date.weekday]);  // 输出: 周一
}
```

#### 2.2.3 时间戳

```dart
void main() {
  var date = DateTime(2024, 1, 15);
  
  // Unix 时间戳（毫秒）
  print(date.millisecondsSinceEpoch);
  
  // Unix 时间戳（微秒）
  print(date.microsecondsSinceEpoch);
  
  // 是否是 UTC 时间
  print(date.isUtc);  // 输出: false
  
  var utc = DateTime.utc(2024, 1, 15);
  print(utc.isUtc);   // 输出: true
}
```

### 2.3 DateTime 的方法

#### 2.3.1 时间计算

```dart
void main() {
  var date = DateTime(2024, 1, 15, 10, 0, 0);
  
  // 添加时间
  var later = date.add(Duration(hours: 2));
  print(later);  // 输出: 2024-01-15 12:00:00.000
  
  // 减去时间
  var earlier = date.subtract(Duration(days: 1));
  print(earlier);  // 输出: 2024-01-14 10:00:00.000
  
  // 计算时间差
  var diff = later.difference(date);
  print(diff.inHours);   // 输出: 2
  print(diff.inMinutes); // 输出: 120
}
```

#### 2.3.2 时间比较

```dart
void main() {
  var date1 = DateTime(2024, 1, 15);
  var date2 = DateTime(2024, 1, 16);
  var date3 = DateTime(2024, 1, 15);
  
  // 比较
  print(date1.compareTo(date2));  // 输出: -1（date1 早于 date2）
  print(date2.compareTo(date1));  // 输出: 1（date2 晚于 date1）
  print(date1.compareTo(date3));  // 输出: 0（相等）
  
  // 比较运算符
  print(date1.isBefore(date2));   // 输出: true
  print(date2.isAfter(date1));    // 输出: true
  print(date1.isAtSameMomentAs(date3));  // 输出: true
}
```

#### 2.3.3 时间转换

```dart
void main() {
  var local = DateTime(2024, 1, 15, 10, 0, 0);
  
  // 转换为 UTC
  var utc = local.toUtc();
  print(utc);  // 输出: 2024-01-15 02:00:00.000Z（假设本地是东八区）
  
  // 转换为本地时间
  var backToLocal = utc.toLocal();
  print(backToLocal);  // 输出本地时间
  
  // 只保留日期部分（时间设为 00:00:00）
  var dateOnly = DateTime(local.year, local.month, local.day);
  print(dateOnly);  // 输出: 2024-01-15 00:00:00.000
}
```

#### 2.3.4 格式化输出

```dart
void main() {
  var date = DateTime(2024, 1, 15, 10, 30, 45);
  
  // ISO 8601 格式
  print(date.toIso8601String());  // 输出: 2024-01-15T10:30:45.000
  
  // 默认字符串表示
  print(date.toString());  // 输出: 2024-01-15 10:30:45.000
  
  // 自定义格式化（使用 intl 包）
  // import 'package:intl/intl.dart';
  // var formatter = DateFormat('yyyy-MM-dd HH:mm:ss');
  // print(formatter.format(date));  // 输出: 2024-01-15 10:30:45
}
```

### 2.4 实战应用

```dart
// 获取本月第一天和最后一天
(DateTime, DateTime) getMonthRange(DateTime date) {
  var firstDay = DateTime(date.year, date.month, 1);
  var lastDay = DateTime(date.year, date.month + 1, 0);
  return (firstDay, lastDay);
}

// 计算年龄
int calculateAge(DateTime birthDate) {
  var now = DateTime.now();
  var age = now.year - birthDate.year;
  
  // 如果今年生日还没到，减一岁
  if (now.month < birthDate.month ||
      (now.month == birthDate.month && now.day < birthDate.day)) {
    age--;
  }
  
  return age;
}

// 格式化相对时间
String formatRelativeTime(DateTime date) {
  var now = DateTime.now();
  var diff = now.difference(date);
  
  if (diff.inSeconds < 60) {
    return '刚刚';
  } else if (diff.inMinutes < 60) {
    return '${diff.inMinutes}分钟前';
  } else if (diff.inHours < 24) {
    return '${diff.inHours}小时前';
  } else if (diff.inDays < 30) {
    return '${diff.inDays}天前';
  } else if (diff.inDays < 365) {
    return '${diff.inDays ~/ 30}个月前';
  } else {
    return '${diff.inDays ~/ 365}年前';
  }
}

void main() {
  var (first, last) = getMonthRange(DateTime(2024, 1, 15));
  print('本月: $first 到 $last');
  
  var birthDate = DateTime(1990, 5, 20);
  print('年龄: ${calculateAge(birthDate)}');
  
  var postTime = DateTime.now().subtract(Duration(hours: 2));
  print('发布时间: ${formatRelativeTime(postTime)}');
}
```

## 第3章 Duration 类

`Duration` 类表示一个时间跨度，用于日期时间的计算和延迟操作。

### 3.1 创建 Duration

```dart
void main() {
  // 各种时间单位
  var microseconds = Duration(microseconds: 1000);
  var milliseconds = Duration(milliseconds: 500);
  var seconds = Duration(seconds: 30);
  var minutes = Duration(minutes: 5);
  var hours = Duration(hours: 2);
  var days = Duration(days: 7);
  
  // 组合
  var mixed = Duration(days: 1, hours: 2, minutes: 30);
  print(mixed);  // 输出: 26:30:00.000000
  
  // 零时长
  var zero = Duration.zero;
  print(zero);  // 输出: 0:00:00.000000
}
```

### 3.2 Duration 的属性

```dart
void main() {
  var duration = Duration(days: 1, hours: 2, minutes: 30, seconds: 45);
  
  // 总天数（可能包含小数）
  print(duration.inDays);         // 输出: 1
  
  // 总小时数
  print(duration.inHours);        // 输出: 26
  
  // 总分钟数
  print(duration.inMinutes);      // 输出: 1590
  
  // 总秒数
  print(duration.inSeconds);      // 输出: 95445
  
  // 总毫秒数
  print(duration.inMilliseconds); // 输出: 95445000
  
  // 总微秒数
  print(duration.inMicroseconds); // 输出: 95445000000
}
```

### 3.3 Duration 的方法

#### 3.3.1 算术运算

```dart
void main() {
  var d1 = Duration(hours: 2);
  var d2 = Duration(minutes: 30);
  
  // 加法
  var sum = d1 + d2;
  print(sum);  // 输出: 2:30:00.000000
  
  // 减法
  var diff = d1 - d2;
  print(diff);  // 输出: 1:30:00.000000
  
  // 乘法
  var multiplied = d1 * 2;
  print(multiplied);  // 输出: 4:00:00.000000
  
  // 除法
  var divided = d1 ~/ 2;  // 整除
  print(divided);  // 输出: 1:00:00.000000
  
  // 取反
  var negated = -d1;
  print(negated);  // 输出: -2:00:00.000000
}
```

#### 3.3.2 比较

```dart
void main() {
  var d1 = Duration(hours: 1);
  var d2 = Duration(minutes: 90);
  var d3 = Duration(hours: 1);
  
  print(d1.compareTo(d2));  // 输出: -1（d1 < d2）
  print(d2.compareTo(d1));  // 输出: 1（d2 > d1）
  print(d1.compareTo(d3));  // 输出: 0（相等）
  
  print(d1 < d2);   // 输出: true
  print(d1 > d2);   // 输出: false
  print(d1 == d3);  // 输出: true
}
```

#### 3.3.3 绝对值

```dart
void main() {
  var negative = Duration(hours: -2);
  
  print(negative.abs());  // 输出: 2:00:00.000000
  
  // 检查是否为负
  print(negative.isNegative);  // 输出: true
}
```

### 3.4 实战应用

```dart
// 格式化时长显示
String formatDuration(Duration duration) {
  var hours = duration.inHours;
  var minutes = duration.inMinutes.remainder(60);
  var seconds = duration.inSeconds.remainder(60);
  
  if (hours > 0) {
    return '${hours.toString().padLeft(2, '0')}:${minutes.toString().padLeft(2, '0')}:${seconds.toString().padLeft(2, '0')}';
  } else {
    return '${minutes.toString().padLeft(2, '0')}:${seconds.toString().padLeft(2, '0')}';
  }
}

// 倒计时
Stream<Duration> countdown(Duration total) async* {
  var remaining = total;
  while (remaining > Duration.zero) {
    yield remaining;
    await Future.delayed(Duration(seconds: 1));
    remaining -= Duration(seconds: 1);
  }
  yield Duration.zero;
}

void main() async {
  print(formatDuration(Duration(minutes: 5, seconds: 30)));  // 输出: 05:30
  print(formatDuration(Duration(hours: 1, minutes: 30)));    // 输出: 01:30:00
  
  // 倒计时示例
  // await for (var remaining in countdown(Duration(minutes: 1))) {
  //   print(formatDuration(remaining));
  // }
}
```

## 第4章 Uri 类

`Uri` 类用于解析、构建和操作统一资源标识符（URI）。它支持 HTTP、HTTPS、FTP、文件等各种 URI 方案。

### 4.1 创建 Uri 对象

#### 4.1.1 解析 URI 字符串

```dart
void main() {
  // 解析完整 URI
  var uri1 = Uri.parse('https://www.example.com:8080/path/to/resource?key=value#fragment');
  print(uri1.scheme);    // 输出: https
  print(uri1.host);      // 输出: www.example.com
  print(uri1.port);      // 输出: 8080
  print(uri1.path);      // 输出: /path/to/resource
  print(uri1.query);     // 输出: key=value
  print(uri1.fragment);  // 输出: fragment
  
  // 尝试解析（可能返回 null）
  var uri2 = Uri.tryParse('not a valid uri');
  print(uri2);  // 输出: null
  
  // 解析文件 URI
  var fileUri = Uri.parse('file:///Users/alice/documents/file.txt');
  print(fileUri);  // 输出: file:///Users/alice/documents/file.txt
}
```

#### 4.1.2 使用构造函数构建

```dart
void main() {
  // 使用构造函数
  var uri = Uri(
    scheme: 'https',
    host: 'api.example.com',
    port: 443,
    path: '/v1/users',
    queryParameters: {
      'page': '1',
      'limit': '10',
    },
  );
  print(uri);  // 输出: https://api.example.com/v1/users?page=1&limit=10
  
  // HTTP URL 快捷构造
  var httpUri = Uri.http('api.example.com', '/users', {'id': '123'});
  print(httpUri);  // 输出: http://api.example.com/users?id=123
  
  // HTTPS URL 快捷构造
  var httpsUri = Uri.https('api.example.com', '/users', {'id': '123'});
  print(httpsUri);  // 输出: https://api.example.com/users?id=123
  
  // 文件 URI
  var fileUri = Uri.file('/Users/alice/documents/file.txt');
  print(fileUri);  // 输出: file:///Users/alice/documents/file.txt
  
  // 目录 URI（末尾带斜杠）
  var dirUri = Uri.directory('/Users/alice/documents/');
  print(dirUri);  // 输出: file:///Users/alice/documents/
}
```

#### 4.1.3 从组件构建

```dart
void main() {
  // 使用 Uri.dataFromString 创建 data URI
  var dataUri = Uri.dataFromString(
    '<h1>Hello World</h1>',
    mimeType: 'text/html',
    encoding: utf8,
  );
  print(dataUri);  // 输出: data:text/html;charset=utf-8,%3Ch1%3EHello%20World%3C/h1%3E
  
  // 使用 Uri.dataFromBytes 创建 data URI
  var bytes = utf8.encode('Hello');
  var bytesUri = Uri.dataFromBytes(bytes, mimeType: 'text/plain');
  print(bytesUri);
}
```

### 4.2 Uri 的属性

```dart
void main() {
  var uri = Uri.parse('https://user:pass@example.com:8080/path/to/file.txt?a=1&b=2#section');
  
  // 方案
  print(uri.scheme);      // 输出: https
  
  // 用户信息
  print(uri.userInfo);    // 输出: user:pass
  
  // 主机
  print(uri.host);        // 输出: example.com
  
  // 端口
  print(uri.port);        // 输出: 8080
  print(uri.hasPort);     // 输出: true
  
  // 路径
  print(uri.path);        // 输出: /path/to/file.txt
  print(uri.pathSegments);  // 输出: [path, to, file.txt]
  
  // 查询字符串
  print(uri.query);       // 输出: a=1&b=2
  print(uri.hasQuery);    // 输出: true
  
  // 查询参数（Map）
  print(uri.queryParameters);  // 输出: {a: 1, b: 2}
  
  // 片段
  print(uri.fragment);    // 输出: section
  print(uri.hasFragment); // 输出: true
  
  // 授权部分（userInfo@host:port）
  print(uri.authority);   // 输出: user:pass@example.com:8080
  
  // 完整字符串
  print(uri.toString());
}
```

### 4.3 Uri 的方法

#### 4.3.1 替换组件

```dart
void main() {
  var uri = Uri.parse('https://example.com/path?key=value');
  
  // 替换路径
  var newPath = uri.replace(path: '/new/path');
  print(newPath);  // 输出: https://example.com/new/path?key=value
  
  // 替换查询参数
  var newQuery = uri.replace(queryParameters: {'foo': 'bar'});
  print(newQuery);  // 输出: https://example.com/path?foo=bar
  
  // 替换多个组件
  var newUri = uri.replace(
    scheme: 'http',
    host: 'api.example.com',
    path: '/v2/resource',
  );
  print(newUri);  // 输出: http://api.example.com/v2/resource?key=value
  
  // 移除查询参数
  var noQuery = uri.replace(query: '');
  print(noQuery);  // 输出: https://example.com/path
}
```

#### 4.3.2 路径操作

```dart
void main() {
  var base = Uri.parse('https://example.com/path/');
  
  // 解析相对 URI
  var resolved = base.resolve('sub/resource');
  print(resolved);  // 输出: https://example.com/path/sub/resource
  
  // 解析绝对 URI（返回原 URI）
  var absolute = base.resolve('https://other.com/page');
  print(absolute);  // 输出: https://other.com/page
  
  // 规范化路径
  var messy = Uri.parse('https://example.com/a/../b/./c');
  var normalized = messy.normalizePath();
  print(normalized);  // 输出: https://example.com/b/c
  
  // 移除片段
  var withFragment = Uri.parse('https://example.com/page#section');
  var withoutFragment = withFragment.removeFragment();
  print(withoutFragment);  // 输出: https://example.com/page
}
```

#### 4.3.3 编码和解码

```dart
void main() {
  // 编码组件
  var encoded = Uri.encodeComponent('hello world!@#');
  print(encoded);  // 输出: hello%20world!%40%23
  
  // 解码组件
  var decoded = Uri.decodeComponent('hello%20world');
  print(decoded);  // 输出: hello world
  
  // 编码查询参数键/值
  var encodedKey = Uri.encodeQueryComponent('key name');
  var encodedValue = Uri.encodeQueryComponent('value=value');
  print('$encodedKey=$encodedValue');  // 输出: key%20name=value%3Dvalue
  
  // 完整路径编码
  var path = '/path with spaces/file.txt';
  var encodedPath = Uri.encodeFull(path);
  print(encodedPath);  // 输出: /path%20with%20spaces/file.txt
}
```

### 4.4 实战应用

```dart
// 构建 API URL
class ApiUrlBuilder {
  final String baseUrl;
  final String version;
  
  ApiUrlBuilder({required this.baseUrl, this.version = 'v1'});
  
  Uri build(String endpoint, {Map<String, dynamic>? params}) {
    return Uri.https(
      baseUrl,
      '/$version/$endpoint',
      params?.map((k, v) => MapEntry(k, v.toString())),
    );
  }
}

// 解析 deep link
Map<String, String>? parseDeepLink(String url) {
  var uri = Uri.tryParse(url);
  if (uri == null) return null;
  
  if (uri.scheme == 'myapp' && uri.host == 'product') {
    return {
      'type': 'product',
      'id': uri.pathSegments.first,
      ...uri.queryParameters,
    };
  }
  
  return null;
}

// URL 签名（简化示例）
Uri signUrl(Uri uri, String secret) {
  var timestamp = DateTime.now().millisecondsSinceEpoch ~/ 1000;
  var params = Map<String, String>.from(uri.queryParameters);
  params['timestamp'] = timestamp.toString();
  
  // 实际应用中应使用 HMAC 等算法
  params['signature'] = '${uri.path}$timestamp$secret'.hashCode.toString();
  
  return uri.replace(queryParameters: params);
}

void main() {
  // API URL 构建
  var api = ApiUrlBuilder(baseUrl: 'api.example.com', version: 'v2');
  var usersUrl = api.build('users', params: {'page': 1, 'limit': 10});
  print(usersUrl);  // 输出: https://api.example.com/v2/users?page=1&limit=10
  
  // Deep link 解析
  var deepLink = parseDeepLink('myapp://product/123?ref=homepage');
  print(deepLink);  // 输出: {type: product, id: 123, ref: homepage}
  
  // URL 签名
  var signed = signUrl(Uri.https('api.example.com', '/data'), 'secret_key');
  print(signed);
}
```

## 第5章 RegExp 类

`RegExp` 类用于处理正则表达式，支持模式匹配、搜索和替换操作。

### 5.1 创建 RegExp 对象

```dart
void main() {
  // 基本正则表达式
  var digits = RegExp(r'\d+');
  print(digits.hasMatch('abc123'));  // 输出: true
  
  // 区分大小写（默认）
  var caseSensitive = RegExp(r'hello');
  print(caseSensitive.hasMatch('Hello'));  // 输出: false
  
  // 不区分大小写
  var caseInsensitive = RegExp(r'hello', caseSensitive: false);
  print(caseInsensitive.hasMatch('Hello'));  // 输出: true
  
  // 多行模式
  var multiline = RegExp(r'^start', multiLine: true);
  var text = '''start of line 1
middle line
start of line 3''';
  print(multiline.allMatches(text).length);  // 输出: 2
  
  // Unicode 模式
  var unicode = RegExp(r'\p{L}+', unicode: true);
  print(unicode.hasMatch('你好'));  // 输出: true
  
  // 点号匹配所有字符（包括换行）
  var dotAll = RegExp(r'.*', dotAll: true);
}
```

### 5.2 RegExp 的方法

#### 5.2.1 匹配检查

```dart
void main() {
  var emailPattern = RegExp(r'^[\w\.-]+@[\w\.-]+\.\w+$');
  
  // 检查是否匹配
  print(emailPattern.hasMatch('test@example.com'));  // 输出: true
  print(emailPattern.hasMatch('invalid-email'));     // 输出: false
  
  // 获取第一个匹配
  var firstMatch = emailPattern.firstMatch('Contact: admin@site.com');
  print(firstMatch?.group(0));  // 输出: admin@site.com
  
  // 获取所有匹配
  var numbers = RegExp(r'\d+');
  var allMatches = numbers.allMatches('Age: 25, Score: 95');
  for (var match in allMatches) {
    print(match.group(0));  // 输出: 25, 95
  }
  
  // 字符串匹配（完全匹配）
  print(emailPattern.stringMatch('test@example.com'));  // 输出: test@example.com
}
```

#### 5.2.2 字符串操作

```dart
void main() {
  var text = 'Hello World, Hello Dart';
  
  // 替换第一个匹配
  var replacedFirst = text.replaceFirst(RegExp(r'Hello'), 'Hi');
  print(replacedFirst);  // 输出: Hi World, Hello Dart
  
  // 替换所有匹配
  var replacedAll = text.replaceAll(RegExp(r'Hello'), 'Hi');
  print(replacedAll);  // 输出: Hi World, Hi Dart
  
  // 使用函数替换
  var incremented = 'Item 1, Item 2, Item 3'.replaceAllMapped(
    RegExp(r'Item (\d+)'),
    (match) => 'Product ${int.parse(match.group(1)!) + 10}',
  );
  print(incremented);  // 输出: Product 11, Product 12, Product 13
  
  // 分割字符串
  var parts = 'apple, banana, orange'.split(RegExp(r',\s*'));
  print(parts);  // 输出: [apple, banana, orange]
}
```

### 5.3 Match 类

`Match` 对象包含匹配的详细信息。

```dart
void main() {
  var pattern = RegExp(r'(\d{4})-(\d{2})-(\d{2})');
  var match = pattern.firstMatch('Date: 2024-01-15');
  
  if (match != null) {
    // 完整匹配
    print(match.group(0));  // 输出: 2024-01-15
    
    // 捕获组
    print(match.group(1));  // 输出: 2024（年）
    print(match.group(2));  // 输出: 01（月）
    print(match.group(3));  // 输出: 15（日）
    
    // 获取所有组
    print(match.groups([0, 1, 2, 3]));  // 输出: [2024-01-15, 2024, 01, 15]
    
    // 匹配位置
    print(match.start);   // 输出: 6（匹配开始位置）
    print(match.end);     // 输出: 16（匹配结束位置）
    print(match.input);   // 输出: Date: 2024-01-15（原始字符串）
    
    // 命名捕获组（Dart 3.0+）
    var namedPattern = RegExp(r'(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})');
    var namedMatch = namedPattern.firstMatch('2024-01-15');
    print(namedMatch?.namedGroup('year'));   // 输出: 2024
    print(namedMatch?.namedGroup('month'));  // 输出: 01
  }
}
```

### 5.4 常用正则表达式模式

```dart
void main() {
  // 邮箱验证
  var email = RegExp(r'^[\w\.-]+@[\w\.-]+\.\w+$');
  
  // 手机号验证（中国大陆）
  var phone = RegExp(r'^1[3-9]\d{9}$');
  
  // URL 验证
  var url = RegExp(r'^https?://[\w\.-]+\.\w+.*$');
  
  // 身份证号（简化）
  var idCard = RegExp(r'^\d{17}[\dXx]$');
  
  // IP 地址
  var ip = RegExp(r'^(\d{1,3}\.){3}\d{1,3}$');
  
  // 中文字符
  var chinese = RegExp(r'[\u4e00-\u9fa5]+');
  print(chinese.hasMatch('你好世界'));  // 输出: true
  
  // HTML 标签
  var htmlTag = RegExp(r'<[^>]+>');
  print('<p>Hello</p>'.replaceAll(htmlTag, ''));  // 输出: Hello
}
```

## 第6章 Comparable 和 Comparator

### 6.1 Comparable 接口

`Comparable` 是一个抽象类，定义了对象之间的比较能力。实现 `Comparable` 的类可以定义自然的排序顺序。

```dart
class Student implements Comparable<Student> {
  final String name;
  final int score;
  
  Student(this.name, this.score);
  
  @override
  int compareTo(Student other) {
    // 按分数降序排列
    return other.score.compareTo(score);
  }
  
  @override
  String toString() => '$name($score)';
}

void main() {
  var students = [
    Student('Alice', 85),
    Student('Bob', 92),
    Student('Charlie', 78),
  ];
  
  students.sort();
  print(students);  // 输出: [Bob(92), Alice(85), Charlie(78)]
  
  // 使用 compareTo 直接比较
  var s1 = Student('A', 90);
  var s2 = Student('B', 85);
  print(s1.compareTo(s2));  // 输出: -1（s1 分数更高，排前面）
}
```

### 6.2 Comparator 函数

`Comparator` 是一个函数类型，用于定义自定义的比较逻辑。

```dart
typedef Comparator<T> = int Function(T a, T b);

void main() {
  var students = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 30},
    {'name': 'Charlie', 'age': 20},
  ];
  
  // 按年龄升序
  students.sort((a, b) => (a['age'] as int).compareTo(b['age'] as int));
  print(students.map((s) => s['name']).toList());  // 输出: [Charlie, Alice, Bob]
  
  // 按名字长度降序
  students.sort((a, b) => (b['name'] as String).length.compareTo((a['name'] as String).length));
  print(students.map((s) => s['name']).toList());  // 输出: [Charlie, Alice, Bob]
  
  // 多级排序
  var products = [
    {'category': 'A', 'price': 100},
    {'category': 'B', 'price': 50},
    {'category': 'A', 'price': 80},
  ];
  
  products.sort((a, b) {
    // 先按类别
    var categoryCompare = (a['category'] as String).compareTo(b['category'] as String);
    if (categoryCompare != 0) return categoryCompare;
    // 再按价格
    return (a['price'] as int).compareTo(b['price'] as int);
  });
  
  print(products);
  // 输出: [{category: A, price: 80}, {category: A, price: 100}, {category: B, price: 50}]
}
```

### 6.3 比较工具函数

```dart
// 创建比较器
Comparator<T> compareBy<T, R extends Comparable<R>>(R Function(T) selector) {
  return (a, b) => selector(a).compareTo(selector(b));
}

// 反转比较器
Comparator<T> reverse<T>(Comparator<T> comparator) {
  return (a, b) => comparator(b, a);
}

// 组合多个比较器
Comparator<T> thenBy<T>(Comparator<T> primary, Comparator<T> secondary) {
  return (a, b) {
    var result = primary(a, b);
    return result != 0 ? result : secondary(a, b);
  };
}

void main() {
  var people = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 30},
    {'name': 'Alice', 'age': 20},
  ];
  
  // 使用 compareBy
  people.sort(compareBy((p) => p['name'] as String));
  print(people.map((p) => '${p['name']}(${p['age']})').toList());
  
  // 多级排序
  people.sort(thenBy(
    compareBy((p) => p['name'] as String),
    compareBy((p) => p['age'] as int),
  ));
  print(people.map((p) => '${p['name']}(${p['age']})').toList());
  // 输出: [Alice(20), Alice(25), Bob(30)]
}
```

## 第7章 错误和异常处理

### 7.1 异常类层次结构

dart:core 中定义了多种异常类，用于处理不同类型的错误。

```dart
// 所有异常的基类
// - Exception：可恢复的异常
// - Error：程序错误，通常不应捕获

// 常见异常类型
void main() {
  // FormatException：格式错误
  try {
    int.parse('not a number');
  } on FormatException catch (e) {
    print('Format error: ${e.message}');
  }
  
  // ArgumentError：参数错误
  try {
    RangeError.checkValueInInterval(10, 0, 5, 'value');
  } on ArgumentError catch (e) {
    print('Argument error: ${e.message}');
  }
  
  // RangeError：范围错误
  try {
    var list = [1, 2, 3];
    print(list[10]);
  } on RangeError catch (e) {
    print('Range error: ${e.message}');
  }
  
  // StateError：状态错误
  try {
    var iterator = [1, 2, 3].iterator;
    iterator.current;  // 在 moveNext 之前访问 current
  } on StateError catch (e) {
    print('State error: ${e.message}');
  }
  
  // UnsupportedError：不支持的操作
  try {
    var list = const [1, 2, 3];
    list.add(4);  // 不可变列表
  } on UnsupportedError catch (e) {
    print('Unsupported: ${e.message}');
  }
}
```

### 7.2 自定义异常

```dart
// 业务异常基类
class BusinessException implements Exception {
  final String code;
  final String message;
  
  BusinessException(this.code, this.message);
  
  @override
  String toString() => '[$code] $message';
}

// 具体业务异常
class ValidationException extends BusinessException {
  final String field;
  
  ValidationException(this.field, String message)
      : super('VALIDATION_ERROR', '$field: $message');
}

class NotFoundException extends BusinessException {
  final String resource;
  final String id;
  
  NotFoundException(this.resource, this.id)
      : super('NOT_FOUND', '$resource with id=$id not found');
}

class UnauthorizedException extends BusinessException {
  UnauthorizedException() : super('UNAUTHORIZED', 'Authentication required');
}

// 使用
void validateUser(Map<String, dynamic> data) {
  if (!data.containsKey('email')) {
    throw ValidationException('email', 'Email is required');
  }
  
  if (!data.containsKey('age') || data['age'] < 18) {
    throw ValidationException('age', 'Must be at least 18 years old');
  }
}

void fetchUser(String id) {
  // 模拟查找
  throw NotFoundException('User', id);
}

void main() {
  try {
    validateUser({'age': 16});
  } on ValidationException catch (e) {
    print(e);  // 输出: [VALIDATION_ERROR] email: Email is required
  }
  
  try {
    fetchUser('123');
  } on NotFoundException catch (e) {
    print(e);  // 输出: [NOT_FOUND] User with id=123 not found
  }
}
```

### 7.3 StackTrace

`StackTrace` 用于获取调用堆栈信息。

```dart
void main() {
  try {
    throw Exception('Something went wrong');
  } catch (e, stackTrace) {
    print('Error: $e');
    print('Stack trace:\n$stackTrace');
  }
  
  // 获取当前堆栈
  print(StackTrace.current);
}

// 异步堆栈跟踪
Future<void> asyncOperation() async {
  await Future.delayed(Duration(milliseconds: 100));
  throw Exception('Async error');
}

void main2() async {
  try {
    await asyncOperation();
  } catch (e, stackTrace) {
    // 使用 StackTrace.current 获取完整堆栈
    print('Async error: $e');
    print(stackTrace);
  }
}
```

## 第8章 其他核心类

### 8.1 Runes

`Runes` 用于处理 Unicode 码点，特别是处理表情符号等多字节字符。

```dart
void main() {
  var emoji = '😀';
  
  // 获取 Unicode 码点
  print(emoji.runes.toList());  // 输出: [128512]
  
  // 从码点创建字符串
  var fromRunes = String.fromCharCode(128512);
  print(fromRunes);  // 输出: 😀
  
  // 从多个码点创建
  var flag = String.fromCharCodes([127468, 127463]);  // 🇬🇧
  print(flag);
  
  // 遍历字符串的码点
  var text = 'Hello 😀';
  for (var rune in text.runes) {
    print('U+${rune.toRadixString(16).toUpperCase().padLeft(4, '0')}');
  }
  // 输出:
  // U+0048
  // U+0065
  // ...
  // U+1F600
}
```

### 8.2 Symbol

`Symbol` 用于表示标识符，主要用于反射 API。

```dart
void main() {
  // 创建 Symbol
  var symbol1 = #myMethod;
  var symbol2 = Symbol('myMethod');
  
  print(symbol1 == symbol2);  // 输出: true
  
  // 私有标识符
  var private = #_privateField;
  print(private);
  
  // 操作符 Symbol
  var plus = #+;
  var equals = #==;
  print(plus);
  print(equals);
}
```

### 8.3 Type

`Type` 表示 Dart 中的类型，可以通过 `runtimeType` 获取。

```dart
void main() {
  var name = 'Alice';
  var age = 25;
  
  // 获取类型
  Type stringType = name.runtimeType;
  Type intType = age.runtimeType;
  
  print(stringType);  // 输出: String
  print(intType);     // 输出: int
  
  // 类型比较
  print(stringType == String);  // 输出: true
  print(intType == int);        // 输出: true
  
  // 泛型类型
  var list = <int>[1, 2, 3];
  print(list.runtimeType);  // 输出: List<int>
}
```

### 8.4 Function

`Function` 是所有函数类型的基类。

```dart
void main() {
  // 函数类型
  int add(int a, int b) => a + b;
  String greet(String name) => 'Hello, $name!';
  
  // 将函数赋值给 Function 类型变量
  Function fn = add;
  print(fn(1, 2));  // 输出: 3
  
  fn = greet;
  print(fn('Alice'));  // 输出: Hello, Alice!
  
  // 更具体的函数类型
  int Function(int, int) mathOp = add;
  print(mathOp(3, 4));  // 输出: 7
  
  // 检查函数类型
  print(add is Function);  // 输出: true
  print(add is int Function(int, int));  // 输出: true
}
```

## 第9章 实战应用示例

### 9.1 日期时间工具类

```dart
class DateTimeUtils {
  // 获取今天的开始时间
  static DateTime startOfDay(DateTime date) {
    return DateTime(date.year, date.month, date.day);
  }
  
  // 获取今天的结束时间
  static DateTime endOfDay(DateTime date) {
    return DateTime(date.year, date.month, date.day, 23, 59, 59, 999);
  }
  
  // 获取本周的开始时间（周一）
  static DateTime startOfWeek(DateTime date) {
    var daysSinceMonday = (date.weekday - DateTime.monday) % 7;
    return startOfDay(date.subtract(Duration(days: daysSinceMonday)));
  }
  
  // 获取本月的开始时间
  static DateTime startOfMonth(DateTime date) {
    return DateTime(date.year, date.month, 1);
  }
  
  // 获取本月的结束时间
  static DateTime endOfMonth(DateTime date) {
    var nextMonth = DateTime(date.year, date.month + 1, 1);
    return nextMonth.subtract(Duration(microseconds: 1));
  }
  
  // 格式化日期（简单实现）
  static String format(DateTime date, String pattern) {
    return pattern
      .replaceAll('yyyy', date.year.toString().padLeft(4, '0'))
      .replaceAll('MM', date.month.toString().padLeft(2, '0'))
      .replaceAll('dd', date.day.toString().padLeft(2, '0'))
      .replaceAll('HH', date.hour.toString().padLeft(2, '0'))
      .replaceAll('mm', date.minute.toString().padLeft(2, '0'))
      .replaceAll('ss', date.second.toString().padLeft(2, '0'));
  }
  
  // 检查是否是同一天
  static bool isSameDay(DateTime a, DateTime b) {
    return a.year == b.year && a.month == b.month && a.day == b.day;
  }
  
  // 检查是否是闰年
  static bool isLeapYear(int year) {
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
  }
  
  // 获取某月的天数
  static int daysInMonth(int year, int month) {
    if (month == 2) return isLeapYear(year) ? 29 : 28;
    if ([4, 6, 9, 11].contains(month)) return 30;
    return 31;
  }
}

void main() {
  var now = DateTime(2024, 1, 15, 10, 30, 0);
  
  print('Start of day: ${DateTimeUtils.startOfDay(now)}');
  print('End of day: ${DateTimeUtils.endOfDay(now)}');
  print('Start of week: ${DateTimeUtils.startOfWeek(now)}');
  print('Start of month: ${DateTimeUtils.startOfMonth(now)}');
  print('Formatted: ${DateTimeUtils.format(now, 'yyyy-MM-dd HH:mm:ss')}');
  print('Days in Feb 2024: ${DateTimeUtils.daysInMonth(2024, 2)}');
}
```

### 9.2 URI 路由匹配

```dart
class RoutePattern {
  final RegExp _pattern;
  final List<String> _paramNames;
  
  RoutePattern(String pattern)
      : _paramNames = [],
        _pattern = _compilePattern(pattern, []) {
    _compilePattern(pattern, _paramNames);
  }
  
  static RegExp _compilePattern(String pattern, List<String> paramNames) {
    // 提取参数名
    var paramRegex = RegExp(r':(\w+)');
    var matches = paramRegex.allMatches(pattern);
    for (var match in matches) {
      paramNames.add(match.group(1)!);
    }
    
    // 将 :param 替换为捕获组
    var regexPattern = pattern
      .replaceAllMapped(paramRegex, (m) => r'([^/]+)')
      .replaceAll('*', r'.*');
    
    return RegExp('^$regexPattern\$');
  }
  
  MatchResult? match(String path) {
    var match = _pattern.firstMatch(path);
    if (match == null) return null;
    
    var params = <String, String>{};
    for (var i = 0; i < _paramNames.length; i++) {
      params[_paramNames[i]] = match.group(i + 1)!;
    }
    
    return MatchResult(path, params);
  }
}

class MatchResult {
  final String path;
  final Map<String, String> params;
  
  MatchResult(this.path, this.params);
}

void main() {
  var userRoute = RoutePattern('/users/:id');
  var postRoute = RoutePattern('/users/:userId/posts/:postId');
  
  var match1 = userRoute.match('/users/123');
  print(match1?.params);  // 输出: {id: 123}
  
  var match2 = postRoute.match('/users/456/posts/789');
  print(match2?.params);  // 输出: {userId: 456, postId: 789}
  
  var noMatch = userRoute.match('/products/123');
  print(noMatch);  // 输出: null
}
```

### 9.3 验证器链

```dart
class ValidationResult {
  final bool isValid;
  final List<String> errors;
  
  ValidationResult.valid() : isValid = true, errors = [];
  ValidationResult.invalid(this.errors) : isValid = false;
}

typedef Validator<T> = ValidationResult Function(T value);

class ValidatorChain<T> {
  final List<Validator<T>> _validators = [];
  
  ValidatorChain<T> add(Validator<T> validator) {
    _validators.add(validator);
    return this;
  }
  
  ValidationResult validate(T value) {
    var errors = <String>[];
    for (var validator in _validators) {
      var result = validator(value);
      if (!result.isValid) {
        errors.addAll(result.errors);
      }
    }
    return errors.isEmpty ? ValidationResult.valid() : ValidationResult.invalid(errors);
  }
}

// 预定义验证器
class Validators {
  static Validator<String> required([String message = 'This field is required']) {
    return (value) => value.isNotEmpty
        ? ValidationResult.valid()
        : ValidationResult.invalid([message]);
  }
  
  static Validator<String> minLength(int min, [String? message]) {
    return (value) => value.length >= min
        ? ValidationResult.valid()
        : ValidationResult.invalid([message ?? 'Minimum length is $min']);
  }
  
  static Validator<String> pattern(RegExp regex, String message) {
    return (value) => regex.hasMatch(value)
        ? ValidationResult.valid()
        : ValidationResult.invalid([message]);
  }
  
  static Validator<String> email([String message = 'Invalid email format']) {
    return pattern(
      RegExp(r'^[\w\.-]+@[\w\.-]+\.\w+$'),
      message,
    );
  }
  
  static Validator<int> range(int min, int max, [String? message]) {
    return (value) => value >= min && value <= max
        ? ValidationResult.valid()
        : ValidationResult.invalid([message ?? 'Must be between $min and $max']);
  }
}

void main() {
  var emailValidator = ValidatorChain<String>()
    .add(Validators.required())
    .add(Validators.email());
  
  var result1 = emailValidator.validate('test@example.com');
  print('Valid: ${result1.isValid}');  // 输出: Valid: true
  
  var result2 = emailValidator.validate('invalid-email');
  print('Valid: ${result2.isValid}, Errors: ${result2.errors}');
  // 输出: Valid: false, Errors: [Invalid email format]
  
  var passwordValidator = ValidatorChain<String>()
    .add(Validators.required())
    .add(Validators.minLength(8))
    .add(Validators.pattern(
      RegExp(r'^(?=.*[A-Za-z])(?=.*\d)'),
      'Must contain letters and numbers',
    ));
  
  var result3 = passwordValidator.validate('short');
  print('Valid: ${result3.isValid}, Errors: ${result3.errors}');
  // 输出: Valid: false, Errors: [Minimum length is 8, Must contain letters and numbers]
}
```

## 附录：dart:core 核心类速查表

| 类名 | 主要用途 | 核心方法/属性 |
|------|---------|--------------|
| Object | 所有类的根类 | toString(), ==, hashCode, runtimeType |
| DateTime | 日期时间处理 | now(), add(), subtract(), difference(), toIso8601String() |
| Duration | 时间跨度 | inDays, inHours, inMinutes, inSeconds, +, - |
| Uri | URI 处理 | parse(), http(), https(), queryParameters, resolve() |
| RegExp | 正则表达式 | hasMatch(), firstMatch(), allMatches(), replaceAll() |
| Match | 正则匹配结果 | group(), start, end, input |
| Comparable | 可比较对象接口 | compareTo() |
| Comparator | 比较函数类型 | (a, b) => int |
| Exception | 异常基类 | message |
| Error | 错误基类 | stackTrace |
| StackTrace | 堆栈跟踪 | current |
| Runes | Unicode 码点 | toList(), String.fromCharCode() |
| Symbol | 标识符符号 | #identifier |
| Type | 类型对象 | runtimeType |
| Function | 函数基类 | apply() |

---

本文档详细介绍了 Dart 核心库 `dart:core` 中除基本数据类型和集合类型之外的核心类，包括 `Object`、`DateTime`、`Duration`、`Uri`、`RegExp`、`Comparable`、`Comparator` 以及各种异常类。这些类构成了 Dart 编程的基础工具集，掌握它们对于编写高效、健壮的 Dart 程序至关重要。
