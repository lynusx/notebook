# Dart 3.0+ 新特性详解

## 目录

1. [概述](#概述)
2. [记录类型 (Records)](#记录类型-records)
3. [模式匹配 (Patterns)](#模式匹配-patterns)
4. [类修饰符 (Class Modifiers)](#类修饰符-class-modifiers)
5. [密封类 (Sealed Classes)](#密封类-sealed-classes)
6. [Switch 表达式](#switch-表达式)
7. [穷尽性检查 (Exhaustive Checking)](#穷尽性检查-exhaustive-checking)
8. [扩展方法 (Extension Methods)](#扩展方法-extension-methods)
9. [扩展类型 (Extension Types)](#扩展类型-extension-types)
10. [实战应用示例](#实战应用示例)
11. [附录](#附录)

---

## 概述

Dart 3.0 于 2023 年 5 月发布，带来了语言历史上最重大的更新之一。这些新特性彻底改变了 Dart 的编程范式，使其更加现代化、表达力更强。

### Dart 3.0+ 主要新特性

| 特性                    | 版本 | 描述                           |
| ----------------------- | ---- | ------------------------------ |
| 记录类型 (Records)      | 3.0  | 匿名复合数据类型，支持多值返回 |
| 模式匹配 (Patterns)     | 3.0  | 解构和匹配值的强大语法         |
| 类修饰符                | 3.0  | 控制类的继承和实现能力         |
| 密封类 (Sealed Classes) | 3.0  | 限制子类化的类层次结构         |
| Switch 表达式           | 3.0  | 表达式形式的 Switch 语句       |
| 穷尽性检查              | 3.0  | 编译时验证所有情况都被处理     |
| 扩展方法                | 2.7+ | 为现有类型添加新方法           |
| 扩展类型                | 3.3  | 零开销的类型包装               |

---

## 记录类型 (Records)

记录是一种匿名的、不可变的聚合类型，允许将多个值组合在一起，而无需定义专门的类。

### 基本语法

```dart
// 定义记录
var record = ('first', a: 2, b: true, 'last');

// 记录类型注解
(String, int) positionRecord = ('Dart', 3);
({String name, int version}) namedRecord = (name: 'Dart', version: 3);
```

### 位置字段与命名字段

```dart
void main() {
  // 纯位置字段记录
  var position = ('Dart', 3, true);
  print(position.$1); // Dart
  print(position.$2); // 3
  print(position.$3); // true

  // 纯命名字段记录
  var named = (language: 'Dart', version: 3);
  print(named.language); // Dart
  print(named.version);  // 3

  // 混合记录（位置字段 + 命名字段）
  var mixed = ('Dart', 3, stable: true);
  print(mixed.$1);      // Dart
  print(mixed.$2);      // 3
  print(mixed.stable);  // true
}
```

### 记录作为返回值

```dart
// 多值返回 - 替代返回对象或 Map
(String, int, bool) getLanguageInfo() {
  return ('Dart', 3, true);
}

({String name, int version, bool stable}) getLanguageInfoNamed() {
  return (name: 'Dart', version: 3, stable: true);
}

void main() {
  var info = getLanguageInfo();
  print('${info.$1} ${info.$2}, stable: ${info.$3}');

  var infoNamed = getLanguageInfoNamed();
  print('${infoNamed.name} ${infoNamed.version}, stable: ${infoNamed.stable}');
}
```

### 记录相等性

```dart
void main() {
  // 记录相等性基于结构，而非身份
  var record1 = (1, 2);
  var record2 = (1, 2);
  var record3 = (1, 3);

  print(record1 == record2); // true
  print(record1 == record3); // false

  // 命名字段顺序不影响相等性
  var a = (x: 1, y: 2);
  var b = (y: 2, x: 1);
  print(a == b); // true
}
```

### 记录类型注解

```dart
// 函数参数中的记录类型
void processPoint((double, double) point) {
  print('x: ${point.$1}, y: ${point.$2}');
}

// 变量声明
(String, {int age, String name}) user = ('Developer', age: 25, name: 'Alice');

// 泛型中的记录
List<(String, int)> scores = [
  ('Alice', 100),
  ('Bob', 85),
  ('Charlie', 90),
];
```

---

## 模式匹配 (Patterns)

模式是 Dart 3.0 最核心的特性之一，提供了一种声明式的方式来解构和匹配值。

### 模式类型概览

| 模式类型 | 语法示例           | 用途         |
| -------- | ------------------ | ------------ |
| 通配符   | `_`                | 忽略值       |
| 变量绑定 | `var x`            | 捕获值到变量 |
| 常量     | `1`, `'hello'`     | 匹配特定值   |
| 记录     | `(x, y)`           | 解构记录     |
| 列表     | `[a, b, c]`        | 解构列表     |
| 映射     | `{'key': value}`   | 解构映射     |
| 对象     | `Point(x: var px)` | 解构对象     |

### 解构赋值

```dart
void main() {
  // 记录解构
  var (name, age) = ('Alice', 25);
  print('$name is $age years old');

  // 列表解构
  var [first, second, third] = [1, 2, 3];
  print('$first, $second, $third');

  // 映射解构
  var {'name': userName, 'age': userAge} = {
    'name': 'Bob',
    'age': 30,
  };

  // 对象解构
  var point = Point(3, 4);
  var Point(x: xCoord, y: yCoord) = point;
  print('Point at ($xCoord, $yCoord)');
}

class Point {
  final double x;
  final double y;
  Point(this.x, this.y);
}
```

### 变量声明模式

```dart
void main() {
  // 使用模式声明多个变量
  var (a, b, c) = (1, 2, 3);

  // 带有类型注解
  final (String name, int age) = ('Alice', 25);

  // 嵌套解构
  var ((x1, y1), (x2, y2)) = ((0, 0), (100, 100));
  print('Line from ($x1, $y1) to ($x2, $y2)');
}
```

### 赋值模式

```dart
void main() {
  var a = 0;
  var b = 0;

  // 交换变量值
  (a, b) = (b, a);

  // 从记录中提取并赋值
  (a, b) = (1, 2);

  // 忽略某些值
  (a, _) = (10, 20);
  (_, b) = (100, 200);
}
```

### JSON 解构

```dart
void main() {
  var json = {
    'user': {
      'name': 'Alice',
      'age': 25,
      'address': {
        'city': 'Beijing',
        'zip': '100000',
      }
    }
  };

  // 嵌套 JSON 解构
  if (json
      case {
        'user': {
          'name': String name,
          'age': int age,
          'address': {
            'city': String city,
          }
        }
      }) {
    print('$name, $age years old, lives in $city');
  }
}
```

---

## 类修饰符 (Class Modifiers)

Dart 3.0 引入了类修饰符，用于控制类的继承、实现和混入能力。

### 修饰符概览

| 修饰符      | 允许 extends | 允许 implements | 允许 mixin | 允许构造 |
| ----------- | ------------ | --------------- | ---------- | -------- |
| (无)        | ✓            | ✓               | ✓          | ✓        |
| `abstract`  | ✓            | ✓               | ✓          | ✗        |
| `base`      | ✓            | ✗               | ✓          | ✓        |
| `interface` | ✗            | ✓               | ✗          | ✓        |
| `final`     | ✗            | ✗               | ✗          | ✓        |
| `sealed`    | ✗            | ✗               | ✗          | ✗        |
| `mixin`     | ✗            | ✓               | ✓          | -        |

### base 修饰符

```dart
// base 类只能被继承，不能被实现
base class Animal {
  String name;
  Animal(this.name);

  void makeSound() => print('Some sound');
}

// 可以继承
class Dog extends Animal {
  Dog(super.name);

  @override
  void makeSound() => print('Woof!');
}

// 错误：不能实现 base 类
// class Robot implements Animal { } // 编译错误！
```

### interface 修饰符

```dart
// interface 类只能被实现，不能被继承
interface class Logger {
  void log(String message) {
    print('[LOG] $message');
  }
}

// 可以实现
class FileLogger implements Logger {
  @override
  void log(String message) {
    // 写入文件
  }
}

// 错误：不能继承 interface 类
// class MyLogger extends Logger { } // 编译错误！
```

### final 修饰符

```dart
// final 类完全封闭，不能被继承或实现
final class Configuration {
  final String apiUrl;
  final int timeout;

  Configuration({required this.apiUrl, required this.timeout});
}

// 错误：不能继承 final 类
// class ExtendedConfig extends Configuration { } // 编译错误！

// 错误：不能实现 final 类
// class MockConfig implements Configuration { } // 编译错误！
```

### abstract 修饰符

```dart
// abstract 类不能实例化，只能被继承或实现
abstract class Shape {
  double get area;
  double get perimeter;

  void draw();
}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);

  @override
  double get area => 3.14 * radius * radius;

  @override
  double get perimeter => 2 * 3.14 * radius;

  @override
  void draw() => print('Drawing circle');
}
```

### mixin 修饰符

```dart
// mixin 类可以作为 mixin 使用，也可以被实现
mixin class Serializable {
  Map<String, dynamic> toJson();

  String serialize() => toJson().toString();
}

// 作为 mixin 使用
class User with Serializable {
  String name;
  User(this.name);

  @override
  Map<String, dynamic> toJson() => {'name': name};
}

// 也可以被实现
class CustomSerializable implements Serializable {
  @override
  Map<String, dynamic> toJson() => {};

  @override
  String serialize() => 'custom';
}
```

### 组合修饰符

```dart
// abstract base：抽象且只能被继承
abstract base class Repository {
  Future<List<Data>> fetchAll();
}

// abstract interface：抽象且只能被实现
abstract interface class Cache {
  Future<void> set(String key, dynamic value);
  Future<dynamic> get(String key);
}

// base mixin：只能被混入到继承链中
base mixin Logging {
  void log(String message) => print(message);
}
```

---

## 密封类 (Sealed Classes)

密封类是 Dart 3.0 最重要的特性之一，用于创建受限的类层次结构，配合模式匹配实现穷尽性检查。

### 基本概念

```dart
// 密封类定义
sealed class Shape {}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);
}

class Square extends Shape {
  final double side;
  Square(this.side);
}

class Rectangle extends Shape {
  final double width;
  final double height;
  Rectangle(this.width, this.height);
}

// 错误：密封类不能在库外部被扩展
// class Triangle extends Shape { } // 如果在另一个文件中，编译错误！
```

### 穷尽性检查

```dart
// 使用密封类进行穷尽性检查
double calculateArea(Shape shape) => switch (shape) {
  Circle(radius: var r) => 3.14 * r * r,
  Square(side: var s) => s * s,
  Rectangle(width: var w, height: var h) => w * h,
};
// 注意：如果添加了新的 Shape 子类但没有更新 switch，编译器会报错！
```

### 密封类与模式匹配

```dart
sealed class Result<T> {}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Error<T> extends Result<T> {
  final String message;
  final int code;
  Error(this.message, this.code);
}

class Loading<T> extends Result<T> {}

// 穷尽性处理所有状态
String getStatusMessage<T>(Result<T> result) => switch (result) {
  Success(data: var data) => 'Success: $data',
  Error(message: var msg, code: var code) => 'Error $code: $msg',
  Loading() => 'Loading...',
};
```

### 密封类最佳实践

```dart
// 状态管理 - 使用密封类定义 UI 状态
sealed class UserState {}

class UserInitial extends UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}

class UserError extends UserState {
  final String message;
  UserError(this.message);
}

// 在 BLoC/Provider 中使用
class UserBloc {
  Stream<UserState> get state => ...;

  void handleState(UserState state) {
    switch (state) {
      case UserInitial():
        print('Initial state');
      case UserLoading():
        print('Loading...');
      case UserLoaded(user: var user):
        print('User loaded: ${user.name}');
      case UserError(message: var msg):
        print('Error: $msg');
    }
  }
}
```

---

## Switch 表达式

Dart 3.0 引入了 Switch 表达式，提供了一种更简洁、更强大的方式来处理条件分支。

### 基本语法

```dart
// 传统 switch 语句
String describeNumber(int n) {
  switch (n) {
    case 0:
      return 'zero';
    case 1:
      return 'one';
    default:
      return 'other';
  }
}

// Switch 表达式
String describeNumber(int n) => switch (n) {
  0 => 'zero',
  1 => 'one',
  _ => 'other',
};
```

### 与模式匹配结合

```dart
// 记录模式匹配
String describePoint((double, double) point) => switch (point) {
  (0, 0) => 'origin',
  (var x, 0) => 'on x-axis at $x',
  (0, var y) => 'on y-axis at $y',
  (var x, var y) => 'at ($x, $y)',
};

// 列表模式匹配
String describeList(List<int> list) => switch (list) {
  [] => 'empty',
  [var single] => 'single: $single',
  [var first, var second] => 'two items: $first, $second',
  [var first, ..., var last] => 'many items, first: $first, last: $last',
};
```

### Guard 子句

```dart
// 使用 when 子句添加条件
String classifyNumber(int n) => switch (n) {
  var x when x < 0 => 'negative',
  var x when x == 0 => 'zero',
  var x when x > 0 && x < 10 => 'small positive',
  var x when x >= 10 && x < 100 => 'medium positive',
  _ => 'large positive',
};

// 结合对象模式
String describeShape(Shape shape) => switch (shape) {
  Circle(radius: var r) when r <= 0 => 'invalid circle',
  Circle(radius: var r) => 'circle with radius $r',
  Square(side: var s) when s <= 0 => 'invalid square',
  Square(side: var s) => 'square with side $s',
  Rectangle(width: var w, height: var h) when w <= 0 || h <= 0 => 'invalid rectangle',
  Rectangle(width: var w, height: var h) => 'rectangle ${w}x$h',
};
```

### 在表达式中使用

```dart
// 作为函数返回值
double calculate(Operation op, double a, double b) => switch (op) {
  Operation.add => a + b,
  Operation.subtract => a - b,
  Operation.multiply => a * b,
  Operation.divide => b != 0 ? a / b : throw ArgumentError('Division by zero'),
};

enum Operation { add, subtract, multiply, divide }

// 在赋值中使用
var status = switch (httpResponse.statusCode) {
  200 => 'OK',
  404 => 'Not Found',
  500 => 'Server Error',
  var code => 'Unknown status: $code',
};
```

---

## 穷尽性检查 (Exhaustive Checking)

穷尽性检查是 Dart 3.0 配合密封类和模式匹配引入的重要特性，确保所有可能的情况都被处理。

### 基本示例

```dart
sealed class Color {}

class Red extends Color {}
class Green extends Color {}
class Blue extends Color {}

// 穷尽性检查确保所有子类都被处理
String colorName(Color color) => switch (color) {
  Red() => 'red',
  Green() => 'green',
  Blue() => 'blue',
};
// 如果添加 Yellow 类但没有更新 switch，编译器会报错！
```

### 布尔穷尽性

```dart
// 布尔值也是穷尽性的
String boolName(bool value) => switch (value) {
  true => 'yes',
  false => 'no',
};
```

### 可空类型穷尽性

```dart
// 可空类型需要处理 null
String? maybeName(String? name) => switch (name) {
  null => 'anonymous',
  var n => n,
};
```

### 枚举穷尽性

```dart
enum Direction { north, south, east, west }

// 枚举天然支持穷尽性检查
String directionName(Direction dir) => switch (dir) {
  Direction.north => 'North',
  Direction.south => 'South',
  Direction.east => 'East',
  Direction.west => 'West',
};
```

### 穷尽性检查与 if-case

```dart
sealed class Shape {}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);
}

class Rectangle extends Shape {
  final double width;
  final double height;
  Rectangle(this.width, this.height);
}

// 使用 if-case 进行穷尽性检查
double calculateArea(Shape shape) {
  if (shape case Circle(radius: var r)) {
    return 3.14 * r * r;
  }
  if (shape case Rectangle(width: var w, height: var h)) {
    return w * h;
  }
  // 编译错误！没有穷尽所有情况
  throw UnimplementedError();
}

// 使用 switch 表达式确保穷尽性
double calculateAreaSafe(Shape shape) => switch (shape) {
  Circle(radius: var r) => 3.14 * r * r,
  Rectangle(width: var w, height: var h) => w * h,
};
```

---

## 扩展方法 (Extension Methods)

扩展方法允许为现有类型添加新方法，而无需继承或修改原始类型。

### 基本语法

```dart
// 为 String 添加扩展方法
extension StringExtension on String {
  // 反转字符串
  String get reversed => split('').reversed.join('');

  // 检查是否为有效邮箱
  bool get isValidEmail => contains('@') && contains('.');

  // 首字母大写
  String get capitalize =>
      isEmpty ? this : '${this[0].toUpperCase()}${substring(1)}';
}

void main() {
  print('hello'.reversed);      // olleh
  print('test@email.com'.isValidEmail); // true
  print('dart'.capitalize);     // Dart
}
```

### 为泛型类型添加扩展

```dart
// 为 List<T> 添加扩展
extension ListExtension<T> on List<T> {
  // 获取第二个元素
  T? get secondOrNull => length > 1 ? this[1] : null;

  // 安全获取元素
  T? getOrNull(int index) =>
      index >= 0 && index < length ? this[index] : null;

  // 分割列表
  List<List<T>> chunk(int size) {
    var chunks = <List<T>>[];
    for (var i = 0; i < length; i += size) {
      chunks.add(sublist(i, (i + size).clamp(0, length)));
    }
    return chunks;
  }
}

void main() {
  var numbers = [1, 2, 3, 4, 5];
  print(numbers.secondOrNull);  // 2
  print(numbers.getOrNull(10)); // null
  print(numbers.chunk(2));      // [[1, 2], [3, 4], [5]]
}
```

### 为可空类型添加扩展

```dart
// 为可空类型添加扩展
extension NullableExtension<T> on T? {
  // 安全映射
  R? map<R>(R Function(T) transform) =>
      this == null ? null : transform(this as T);

  // 提供默认值
  T orElse(T defaultValue) => this ?? defaultValue;

  // 条件执行
  void let(void Function(T) action) {
    if (this != null) action(this as T);
  }
}

void main() {
  String? name = 'Alice';
  print(name.map((n) => n.toUpperCase())); // ALICE

  String? nullName;
  print(nullName.orElse('Unknown')); // Unknown

  name.let((n) => print('Hello, $n!')); // Hello, Alice!
}
```

### 扩展运算符

```dart
extension IntExtension on int {
  // 范围操作符
  Iterable<int> get toZero sync* {
    for (var i = this; i >= 0; i--) {
      yield i;
    }
  }

  // 重复操作
  void times(void Function(int) action) {
    for (var i = 0; i < this; i++) {
      action(i);
    }
  }

  // 延迟操作
  Future<void> get seconds => Future.delayed(Duration(seconds: this));
}

void main() {
  // 倒计时
  for (var i in 5.toZero) {
    print(i);
  }

  // 重复执行
  3.times((i) => print('Iteration $i'));

  // 延迟
  await 2.seconds;
  print('After 2 seconds');
}
```

### 扩展的可见性控制

```dart
// 命名扩展 - 控制可见性和解决冲突
extension DateTimeFormatting on DateTime {
  String get formatted => '$year-$month-$day';
}

// 私有扩展（仅在当前库可见）
extension _PrivateExtension on String {
  String get secret => '***$this***';
}

// 使用 show/hide 解决冲突
import 'package:my_package/extensions.dart' show DateTimeFormatting;
import 'package:other_package/extensions.dart' hide StringExtension;
```

---

## 扩展类型 (Extension Types)

Dart 3.3 引入了扩展类型，提供零开销的类型包装，用于类型安全和 API 封装。

### 基本概念

```dart
// 定义扩展类型
extension type Meters(double value) {
  // 可以添加方法和 getter
  Meters operator +(Meters other) => Meters(value + other.value);

  double get toKilometers => value / 1000;

  String get display => '${value}m';
}

// 另一个扩展类型
extension type Kilometers(double value) {
  double get toMeters => value * 1000;

  String get display => '${value}km';
}

void main() {
  var distance1 = Meters(1000);
  var distance2 = Meters(500);

  var total = distance1 + distance2;
  print(total.display); // 1500.0m
  print(total.toKilometers); // 1.5

  // 类型安全 - 不能混用
  // var mixed = distance1 + Kilometers(1); // 编译错误！
}
```

### 与扩展方法的区别

```dart
// 扩展方法 - 运行时仍然是原始类型
extension StringUtils on String {
  bool get isValidEmail => contains('@');
}

// 扩展类型 - 编译时类型检查
extension type Email(String value) {
  bool get isValid => value.contains('@');
}

void main() {
  String str = 'test@email.com';
  Email email = Email('test@email.com');

  // 扩展方法：原始类型可用
  print(str.isValidEmail); // true

  // 扩展类型：类型安全
  print(email.isValid); // true

  // Email 不能直接赋值给 String
  // String s = email; // 编译错误！
  // 需要使用 .value
  String s = email.value;
}
```

### 实际应用场景

```dart
// ID 类型 - 避免混淆不同类型的 ID
extension type UserId(int value) {}
extension type ProductId(int value) {}
extension type OrderId(String value) {}

void fetchUser(UserId id) { }
void fetchProduct(ProductId id) { }

void main() {
  var userId = UserId(123);
  var productId = ProductId(456);

  fetchUser(userId);      // ✓
  // fetchUser(productId); // ✗ 编译错误！
}
```

### 带有接口实现的扩展类型

```dart
// 扩展类型可以实现接口
extension type Meters(double value) implements Comparable<Meters> {
  @override
  int compareTo(Meters other) => value.compareTo(other.value);

  Meters operator +(Meters other) => Meters(value + other.value);
  Meters operator -(Meters other) => Meters(value - other.value);
}

void main() {
  var distances = [Meters(100), Meters(50), Meters(200)];
  distances.sort();
  print(distances.map((d) => d.value)); // (50, 100, 200)
}
```

---

## 实战应用示例

### 示例 1：API 响应处理

```dart
// 定义密封类表示 API 响应状态
sealed class ApiResponse<T> {}

class ApiSuccess<T> extends ApiResponse<T> {
  final T data;
  final int statusCode;
  ApiSuccess(this.data, this.statusCode);
}

class ApiError<T> extends ApiResponse<T> {
  final String message;
  final int statusCode;
  final Map<String, dynamic>? details;
  ApiError(this.message, this.statusCode, {this.details});
}

class ApiLoading<T> extends ApiResponse<T> {}

// 处理响应的函数
String handleResponse<T>(ApiResponse<T> response) => switch (response) {
  ApiSuccess(data: var data, statusCode: var code) =>
    'Success ($code): $data',
  ApiError(message: var msg, statusCode: var code, details: var d) =>
    'Error $code: $msg${d != null ? ' - $d' : ''}',
  ApiLoading() => 'Loading...',
};

// 使用记录类型返回多个值
(ApiResponse<User>, Duration) fetchUserWithTiming(String id) {
  final stopwatch = Stopwatch()..start();

  try {
    final user = apiClient.getUser(id);
    stopwatch.stop();
    return (ApiSuccess(user, 200), stopwatch.elapsed);
  } catch (e) {
    stopwatch.stop();
    return (ApiError(e.toString(), 500), stopwatch.elapsed);
  }
}
```

### 示例 2：状态管理

```dart
// 使用密封类定义应用状态
sealed class AppState {}

class AppInitial extends AppState {}

class AppAuthenticated extends AppState {
  final User user;
  final List<Permission> permissions;
  AppAuthenticated(this.user, this.permissions);
}

class AppUnauthenticated extends AppState {
  final String? reason;
  AppUnauthenticated({this.reason});
}

class AppLoading extends AppState {
  final String message;
  AppLoading(this.message);
}

// 状态处理器
Widget buildStateWidget(AppState state) => switch (state) {
  AppInitial() => const SplashScreen(),
  AppAuthenticated(user: var u, permissions: var p) =>
    HomeScreen(user: u, permissions: p),
  AppUnauthenticated(reason: var r) =>
    LoginScreen(errorMessage: r),
  AppLoading(message: var m) =>
    LoadingScreen(message: m),
};
```

### 示例 3：JSON 解析

```dart
// 使用模式匹配解析 JSON
User parseUser(dynamic json) {
  if (json
      case {
        'id': int id,
        'name': String name,
        'email': String email,
        'address': {
          'street': String street,
          'city': String city,
        },
        'tags': List<String> tags,
      }) {
    return User(
      id: id,
      name: name,
      email: email,
      address: Address(street: street, city: city),
      tags: tags,
    );
  }
  throw FormatException('Invalid user JSON: $json');
}

// 使用扩展类型增强类型安全
extension type UserId(int value) {}
extension type Email(String value) {
  bool get isValid => value.contains('@') && value.contains('.');
}

class User {
  final UserId id;
  final String name;
  final Email email;

  User({
    required this.id,
    required this.name,
    required this.email,
  });
}
```

### 示例 4：表单验证

```dart
// 定义验证结果密封类
sealed class ValidationResult {
  bool get isValid => this is ValidationSuccess;
}

class ValidationSuccess extends ValidationResult {
  final String message;
  ValidationSuccess([this.message = 'Valid']);
}

class ValidationError extends ValidationResult {
  final String field;
  final String message;
  ValidationError(this.field, this.message);
}

// 验证器扩展
extension StringValidation on String {
  ValidationResult validateEmail() => switch (this) {
    var s when s.isEmpty =>
      ValidationError('email', 'Email is required'),
    var s when !s.contains('@') =>
      ValidationError('email', 'Invalid email format'),
    _ => ValidationSuccess(),
  };

  ValidationResult validatePassword() => switch (this) {
    var s when s.length < 8 =>
      ValidationError('password', 'Password must be at least 8 characters'),
    var s when !s.contains(RegExp(r'[A-Z]')) =>
      ValidationError('password', 'Password must contain uppercase letter'),
    var s when !s.contains(RegExp(r'[0-9]')) =>
      ValidationError('password', 'Password must contain number'),
    _ => ValidationSuccess(),
  };
}

// 表单验证
List<ValidationResult> validateForm(Map<String, String> form) => [
  form['email']?.validateEmail() ?? ValidationError('email', 'Required'),
  form['password']?.validatePassword() ?? ValidationError('password', 'Required'),
];
```

### 示例 5：集合处理

```dart
// 使用扩展方法增强集合操作
extension IterableExtension<T> on Iterable<T> {
  // 分组
  Map<K, List<T>> groupBy<K>(K Function(T) keySelector) {
    final map = <K, List<T>>{};
    for (var item in this) {
      final key = keySelector(item);
      map.putIfAbsent(key, () => []).add(item);
    }
    return map;
  }

  // 去重
  Iterable<T> distinct() => toSet();

  // 安全查找
  T? firstWhereOrNull(bool Function(T) test) {
    for (var item in this) {
      if (test(item)) return item;
    }
    return null;
  }
}

// 使用模式匹配处理集合
void processUsers(List<Map<String, dynamic>> users) {
  for (var user in users) {
    if (user
        case {
          'name': String name,
          'age': int age,
          'roles': List<String> roles,
        }) {
      final roleStr = switch (roles) {
        [] => 'no roles',
        [var single] => 'role: $single',
        [var first, var second] => 'roles: $first, $second',
        [var first, ..., var last] => 'roles: $first...$last',
      };
      print('$name ($age years old) has $roleStr');
    }
  }
}
```

---

## 附录

### A. 版本兼容性

| 特性          | 最低版本 | 说明            |
| ------------- | -------- | --------------- |
| 扩展方法      | 2.7.0    | 稳定可用        |
| 记录类型      | 3.0.0    | Dart 3 核心特性 |
| 模式匹配      | 3.0.0    | Dart 3 核心特性 |
| 类修饰符      | 3.0.0    | Dart 3 核心特性 |
| 密封类        | 3.0.0    | Dart 3 核心特性 |
| Switch 表达式 | 3.0.0    | Dart 3 核心特性 |
| 扩展类型      | 3.3.0    | 较新特性        |

### B. 迁移指南

#### 从旧代码迁移到 Dart 3

```dart
// 旧代码：使用类表示简单数据
class Point {
  final double x;
  final double y;
  Point(this.x, this.y);
}

// 新代码：使用记录类型
(double, double) createPoint() => (3.0, 4.0);

// 旧代码：Switch 语句
String describe(int value) {
  switch (value) {
    case 1:
      return 'one';
    case 2:
      return 'two';
    default:
      return 'other';
  }
}

// 新代码：Switch 表达式
String describe(int value) => switch (value) {
  1 => 'one',
  2 => 'two',
  _ => 'other',
};

// 旧代码：使用继承表示受限类型
abstract class Shape {}
class Circle extends Shape {}
class Square extends Shape {}

// 新代码：使用密封类
sealed class Shape {}
class Circle extends Shape {}
class Square extends Shape {}
```

### C. 最佳实践总结

1. **优先使用记录类型**：对于简单的数据组合，优先使用记录类型而非定义类
2. **密封类用于状态管理**：使用密封类定义 UI 状态和 API 响应状态
3. **模式匹配简化代码**：使用模式匹配替代复杂的 if-else 链
4. **穷尽性检查保证安全**：利用编译器的穷尽性检查确保所有情况都被处理
5. **扩展方法增强可读性**：为常用类型添加扩展方法提高代码可读性
6. **扩展类型保证类型安全**：使用扩展类型避免类型混淆

### D. 常见陷阱

```dart
// 陷阱 1：记录相等性
var r1 = (1, 2);
var r2 = (1, 2);
print(identical(r1, r2)); // false - 记录是值类型

// 陷阱 2：模式匹配中的变量遮蔽
var x = 10;
if ((x, 20) case (var x, var y)) {
  // 这里的 x 是新的变量，遮蔽了外部的 x
  print(x); // 10（来自模式匹配）
}
print(x); // 10（外部 x 未改变）

// 陷阱 3：忘记穷尽性检查
sealed class Status {}
class Success extends Status {}
class Error extends Status {}

// 错误：没有穷尽所有情况
String getMessage(Status status) {
  switch (status) {
    case Success():
      return 'Success';
    // 缺少 Error 的处理！
  }
}

// 正确：使用 switch 表达式确保穷尽性
String getMessageSafe(Status status) => switch (status) {
  Success() => 'Success',
  Error() => 'Error',
};
```

### E. 参考资源

- [Dart 3 发布说明](https://dart.dev/guides/whats-new#dart-3)
- [模式匹配文档](https://dart.dev/language/patterns)
- [记录类型文档](https://dart.dev/language/records)
- [类修饰符文档](https://dart.dev/language/class-modifiers)
- [扩展方法文档](https://dart.dev/language/extension-methods)
- [扩展类型文档](https://dart.dev/language/extension-types)
