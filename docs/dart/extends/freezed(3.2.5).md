# Freezed 库详解

> 本文基于 freezed: ^3.2.5 版本，深入解析这个 Dart/Flutter 生态中最强大的代码生成库。Freezed 能够自动生成不可变数据类、联合类型（Union Types）、深度拷贝、值相等性比较和 JSON 序列化代码，让你专注于业务逻辑而非样板代码。

---

## 目录

1. [简介与安装](#1-简介与安装)
2. [快速入门](#2-快速入门)
3. [不可变数据类](#3-不可变数据类)
4. [copyWith 深度拷贝](#4-copywith-深度拷贝)
5. [联合类型（Union Types）](#5-联合类型union-types)
6. [JSON 序列化](#6-json-序列化)
7. [泛型支持](#7-泛型支持)
8. [高级特性](#8-高级特性)
9. [build.yaml 全局配置](#9-buildyaml-全局配置)
10. [实战应用示例](#10-实战应用示例)
11. [最佳实践与常见问题](#11-最佳实践与常见问题)
12. [附录：速查表](#12-附录速查表)

---

## 1. 简介与安装

### 1.1 什么是 Freezed？

Freezed 是一个 Dart 代码生成器，专门为数据类自动生成样板代码。它解决了 Dart 中定义模型类时需要编写大量重复代码的问题。

**手动实现的痛点**：

```dart
// 手动实现一个数据类需要大量样板代码
class Person {
  final String firstName;
  final String lastName;
  final int age;

  const Person({
    required this.firstName,
    required this.lastName,
    required this.age,
  });

  // copyWith - 用于不可变更新
  Person copyWith({
    String? firstName,
    String? lastName,
    int? age,
  }) {
    return Person(
      firstName: firstName ?? this.firstName,
      lastName: lastName ?? this.lastName,
      age: age ?? this.age,
    );
  }

  // 相等性比较
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is Person &&
        other.firstName == firstName &&
        other.lastName == lastName &&
        other.age == age;
  }

  @override
  int get hashCode => Object.hash(firstName, lastName, age);

  // 调试输出
  @override
  String toString() =>
    'Person(firstName: $firstName, lastName: $lastName, age: $age)';
}
```

**使用 Freezed 的优雅方案**：

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'person.freezed.dart';

@freezed
class Person with _$Person {
  const factory Person({
    required String firstName,
    required String lastName,
    required int age,
  }) = _Person;
}
```

### 1.2 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  freezed_annotation: ^3.0.0

dev_dependencies:
  build_runner: ^2.4.9
  freezed: ^3.2.5
  json_serializable: ^6.13.0 # 如果需要 JSON 序列化
```

然后运行安装命令：

```bash
# Dart 项目
dart pub get

# Flutter 项目
flutter pub get
```

### 1.3 代码生成命令

```bash
# 一次性生成代码
dart run build_runner build

# 生成代码并删除冲突文件（推荐）
dart run build_runner build --delete-conflicting-outputs

# 监视模式 - 自动检测文件变化并重新生成
dart run build_runner watch

# 监视模式并删除冲突文件
dart run build_runner watch --delete-conflicting-outputs

# 清理生成的文件
dart run build_runner clean
```

### 1.4 Freezed 能为你生成什么？

| 功能            | 说明                           |
| --------------- | ------------------------------ |
| **不可变性**    | 所有字段自动为 `final`         |
| **copyWith**    | 深度拷贝方法，支持嵌套对象     |
| **相等性**      | 自动生成 `==` 和 `hashCode`    |
| **toString**    | 调试友好的字符串表示           |
| **联合类型**    | 支持 sealed classes 和模式匹配 |
| **JSON 序列化** | 与 json_serializable 无缝集成  |

---

## 2. 快速入门

### 2.1 创建第一个 Freezed 类

```dart
// lib/models/user.dart
import 'package:freezed_annotation/freezed_annotation.dart';

// 必须：声明生成的 part 文件
part 'user.freezed.dart';

// 注解：告诉 Freezed 为该类生成代码
@freezed
class User with _$User {
  // 工厂构造函数定义
  const factory User({
    required String id,
    required String name,
    required String email,
    required int age,
  }) = _User;
}
```

### 2.2 生成代码

运行代码生成命令：

```bash
dart run build_runner build --delete-conflicting-outputs
```

生成的 `user.freezed.dart` 文件内容大致如下：

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'user.dart';

// **************************************************************************
// FreezedGenerator
// **************************************************************************

abstract class _$User {
  String get id;
  String get name;
  String get email;
  int get age;
}

class _User implements User {
  const _User({
    required this.id,
    required this.name,
    required this.email,
    required this.age,
  });

  @override
  final String id;
  @override
  final String name;
  @override
  final String email;
  @override
  final int age;

  @override
  _User copyWith({...}) => ...;

  @override
  bool operator ==(Object other) => ...;

  @override
  int get hashCode => ...;

  @override
  String toString() => ...;
}
```

### 2.3 使用 Freezed 类

```dart
void main() {
  // 创建实例
  final user = User(
    id: '1',
    name: 'Alice',
    email: 'alice@example.com',
    age: 30,
  );

  // 使用 copyWith 创建修改后的副本
  final updatedUser = user.copyWith(age: 31);

  print(user.age);        // 30（原对象不变）
  print(updatedUser.age); // 31

  // 相等性比较
  final user2 = User(
    id: '1',
    name: 'Alice',
    email: 'alice@example.com',
    age: 30,
  );

  print(user == user2);        // true（值相等）
  print(identical(user, user2)); // false（不同对象）

  // 调试输出
  print(user);
  // User(id: 1, name: Alice, email: alice@example.com, age: 30)
}
```

---

## 3. 不可变数据类

### 3.1 基本用法

```dart
@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required double price,
    String? description,
    required List<String> categories,
  }) = _Product;
}
```

### 3.2 默认值（@Default）

使用 `@Default` 注解为字段提供默认值：

```dart
@freezed
class Settings with _$Settings {
  const factory Settings({
    @Default('light') String theme,
    @Default(true) bool notifications,
    @Default(10) int itemsPerPage,
    @Default(['read']) List<String> permissions,
  }) = _Settings;
}

void main() {
  // 使用默认值
  final settings = Settings();
  print(settings.theme);           // light
  print(settings.notifications);   // true
  print(settings.itemsPerPage);    // 10

  // 覆盖默认值
  final darkSettings = Settings(theme: 'dark');
  print(darkSettings.theme);  // dark
}
```

### 3.3 位置参数

Freezed 也支持位置参数：

```dart
@freezed
class Point with _$Point {
  const factory Point(double x, double y) = _Point;
}

void main() {
  final point = Point(10.0, 20.0);
  print(point.x);  // 10.0
  print(point.y);  // 20.0
}
```

### 3.4 混合使用命名和位置参数

```dart
@freezed
class Rectangle with _$Rectangle {
  const factory Rectangle(
    double width,
    double height, {
    String? label,
    @Default(0xFF000000) int color,
  }) = _Rectangle;
}

void main() {
  final rect = Rectangle(100.0, 50.0, label: 'Box', color: 0xFFFF0000);
}
```

### 3.5 断言（Asserts）

在构造函数中添加断言：

```dart
@freezed
class ValidatedAge with _$ValidatedAge {
  const factory ValidatedAge(int value) = _ValidatedAge;

  // 私有构造函数用于断言
  const ValidatedAge._() : assert(value >= 0 && value <= 150, '年龄必须在 0-150 之间');
}

void main() {
  final validAge = ValidatedAge(25);    // OK
  final invalidAge = ValidatedAge(200); // 抛出 AssertionError
}
```

### 3.6 非常量默认值

对于无法使用 `@Default` 的非常量默认值：

```dart
@freezed
class TimestampedData with _$TimestampedData {
  // 私有构造函数初始化非常量默认值
  TimestampedData._() : createdAt = DateTime.now();

  const factory TimestampedData({
    required String data,
    DateTime? createdAt,
  }) = _TimestampedData;

  @override
  final DateTime createdAt;
}
```

### 3.7 可变类（@unfreezed）

如果你需要可变类（mutable class），可以使用 `@unfreezed`：

```dart
@unfreezed
class MutableCounter with _$MutableCounter {
  factory MutableCounter({@Default(0) int count}) = _MutableCounter;
}

void main() {
  final counter = MutableCounter();
  counter.count++;  // 可以修改！
  print(counter.count);  // 1
}
```

### 3.8 经典类风格

如果你需要更传统的 Dart 类风格：

```dart
@freezed
@JsonSerializable()
class Person with _$Person {
  const Person({
    required this.firstName,
    required this.lastName,
    required this.age,
  });

  @override
  final String firstName;
  @override
  final String lastName;
  @override
  final int age;

  factory Person.fromJson(Map<String, Object?> json) => _$PersonFromJson(json);
  Map<String, Object?> toJson() => _$PersonToJson(this);
}
```

---

## 4. copyWith 深度拷贝

### 4.1 基本用法

```dart
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required Address address,
  }) = _User;
}

@freezed
class Address with _$Address {
  const factory Address({
    required String city,
    required String street,
  }) = _Address;
}

void main() {
  final user = User(
    id: '1',
    name: 'Alice',
    address: Address(city: 'Beijing', street: 'Main St'),
  );

  // 浅拷贝 - 修改顶层字段
  final user2 = user.copyWith(name: 'Bob');

  // 深度拷贝 - 修改嵌套对象
  final user3 = user.copyWith(
    address: user.address.copyWith(city: 'Shanghai'),
  );

  print(user.address.city);   // Beijing（原对象不变）
  print(user3.address.city);  // Shanghai
}
```

### 4.2 处理 null 值

Freezed 的 `copyWith` 使用特殊的 `Object?` 类型来区分"不修改"和"设为 null"：

```dart
@freezed
class OptionalFields with _$OptionalFields {
  const factory OptionalFields({
    String? name,
    int? age,
  }) = _OptionalFields;
}

void main() {
  final data = OptionalFields(name: 'Alice', age: 30);

  // 不修改 name（保持原值）
  final data2 = data.copyWith(age: 31);
  print(data2.name);  // Alice
  print(data2.age);   // 31

  // 将 name 设为 null
  final data3 = data.copyWith(name: null);
  print(data3.name);  // null
  print(data3.age);   // 30
}
```

### 4.3 集合的可变性

默认情况下，Freezed 会将集合（List/Map/Set）转换为不可变的：

```dart
@freezed
class ImmutableCollections with _$ImmutableCollections {
  const factory ImmutableCollections({
    required List<int> items,
  }) = _ImmutableCollections;
}

void main() {
  final data = ImmutableCollections(items: [1, 2, 3]);

  // 这会抛出异常！
  // data.items.add(4);  // UnsupportedError

  // 正确做法：使用 copyWith 创建新列表
  final data2 = data.copyWith(items: [...data.items, 4]);
}
```

允许可变集合：

```dart
@Freezed(makeCollectionsUnmodifiable: false)
class MutableCollections with _$MutableCollections {
  const factory MutableCollections({
    required List<int> items,
  }) = _MutableCollections;
}

void main() {
  final data = MutableCollections(items: [1, 2, 3]);
  data.items.add(4);  // OK
}
```

---

## 5. 联合类型（Union Types）

联合类型（也称为密封类或 Sealed Classes）允许一个类具有多种互斥的状态。

### 5.1 基本用法

```dart
@freezed
sealed class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse.loading() = ApiLoading<T>;
  const factory ApiResponse.data(T data) = ApiData<T>;
  const factory ApiResponse.error(String message) = ApiError<T>;
}
```

### 5.2 使用 Dart 3 模式匹配

```dart
void handleResponse(ApiResponse<User> response) {
  // 使用 switch 表达式进行模式匹配
  final message = switch (response) {
    ApiLoading() => '加载中...',
    ApiData(data: final user) => '用户: ${user.name}',
    ApiError(message: final msg) => '错误: $msg',
  };
  print(message);
}

void main() {
  final loading = ApiResponse<User>.loading();
  final data = ApiResponse<User>.data(User(id: '1', name: 'Alice'));
  final error = ApiResponse<User>.error('网络错误');

  handleResponse(loading);  // 加载中...
  handleResponse(data);     // 用户: Alice
  handleResponse(error);    // 错误: 网络错误
}
```

### 5.3 共享属性

联合类型可以定义共享属性：

```dart
@freezed
sealed class NetworkState with _$NetworkState {
  const factory NetworkState.loading({
    required DateTime timestamp,
  }) = NetworkLoading;

  const factory NetworkState.success({
    required DateTime timestamp,
    required String data,
  }) = NetworkSuccess;

  const factory NetworkState.error({
    required DateTime timestamp,
    required String error,
  }) = NetworkError;
}

void main() {
  final state = NetworkState.success(
    timestamp: DateTime.now(),
    data: 'Hello',
  );

  // 访问共享属性
  print(state.timestamp);  // 所有状态都有 timestamp

  // 访问特定属性需要模式匹配
  if (state is NetworkSuccess) {
    print(state.data);
  }
}
```

### 5.4 使用 when/map（传统方式）

虽然 Dart 3 的模式匹配是推荐方式，但 Freezed 仍然提供 `when` 和 `map` 方法：

```dart
void handleWithWhen(ApiResponse<User> response) {
  final result = response.when(
    loading: () => '加载中',
    data: (user) => '数据: ${user.name}',
    error: (message) => '错误: $message',
  );
  print(result);
}

// maybeWhen - 只处理部分情况
void handleWithMaybeWhen(ApiResponse<User> response) {
  final result = response.maybeWhen(
    data: (user) => '数据: ${user.name}',
    orElse: () => '其他状态',
  );
  print(result);
}

// map - 访问状态对象本身
void handleWithMap(ApiResponse<User> response) {
  response.map(
    loading: (loading) => print('Loading: $loading'),
    data: (data) => print('Data: ${data.data}'),
    error: (error) => print('Error: ${error.message}'),
  );
}
```

### 5.5 联合类型的继承和接口

```dart
// 定义接口
abstract class HasMessage {
  String get message;
}

@freezed
sealed class Result<T> with _$Result<T> implements HasMessage {
  const Result._();

  const factory Result.success(T data) = Success<T>;

  const factory Result.failure({
    required String message,
    required int code,
  }) = Failure<T>;

  @override
  String get message => when(
    success: (_) => 'Success',
    failure: (message, _) => message,
  );
}
```

---

## 6. JSON 序列化

### 6.1 基本 JSON 序列化

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';  // json_serializable 生成的文件

@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required String email,
    required int age,
  }) = _User;

  // 从 JSON 反序列化
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

void main() {
  // JSON 字符串
  const jsonString = '''
    {
      "id": "1",
      "name": "Alice",
      "email": "alice@example.com",
      "age": 30
    }
  ''';

  // 解析 JSON
  final json = jsonDecode(jsonString) as Map<String, dynamic>;
  final user = User.fromJson(json);

  print(user.name);  // Alice

  // 序列化为 JSON
  final outputJson = user.toJson();
  print(jsonEncode(outputJson));
}
```

### 6.2 字段名映射（@JsonKey）

```dart
@freezed
class ApiUser with _$ApiUser {
  const factory ApiUser({
    @JsonKey(name: 'user_id') required String id,
    @JsonKey(name: 'user_name') required String name,
    @JsonKey(name: 'created_at') required DateTime createdAt,
  }) = _ApiUser;

  factory ApiUser.fromJson(Map<String, dynamic> json) => _$ApiUserFromJson(json);
}

// JSON: {"user_id": "1", "user_name": "Alice", "created_at": "2024-01-01T00:00:00Z"}
```

### 6.3 自定义序列化

```dart
@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    @JsonKey(fromJson: _priceFromJson, toJson: _priceToJson)
    required double price,
  }) = _Product;

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);

  // 自定义反序列化
  static double _priceFromJson(dynamic json) => (json as num).toDouble();

  // 自定义序列化
  static num _priceToJson(double price) => price;
}
```

### 6.4 联合类型的 JSON 序列化

```dart
@Freezed(unionKey: 'type')
class Animal with _$Animal {
  const factory Animal.dog({
    required String name,
    required String breed,
  }) = Dog;

  const factory Animal.cat({
    required String name,
    required int livesLeft,
  }) = Cat;

  factory Animal.fromJson(Map<String, dynamic> json) => _$AnimalFromJson(json);
}

void main() {
  // JSON 会自动包含 type 字段
  final dogJson = {'type': 'dog', 'name': 'Buddy', 'breed': 'Golden'};
  final animal = Animal.fromJson(dogJson);

  print(animal);  // Animal.dog(name: Buddy, breed: Golden)

  // 序列化
  print(animal.toJson());
  // {type: dog, name: Buddy, breed: Golden}
}
```

### 6.5 自定义联合类型键

```dart
@Freezed(unionKey: 'status', unionValueCase: FreezedUnionCase.pascal)
class Response with _$Response {
  const factory Response.loading() = ResponseLoading;
  const factory Response.success(String data) = ResponseSuccess;
  const factory Response.error(String message) = ResponseError;

  factory Response.fromJson(Map<String, dynamic> json) => _$ResponseFromJson(json);
}

// JSON: {"status": "Loading"}
// JSON: {"status": "Success", "data": "Hello"}
// JSON: {"status": "Error", "message": "Failed"}
```

### 6.6 嵌套对象的 JSON 序列化

```dart
@freezed
class Company with _$Company {
  const factory Company({
    required String name,
    required Address address,
    required List<Employee> employees,
  }) = _Company;

  factory Company.fromJson(Map<String, dynamic> json) => _$CompanyFromJson(json);
}

@freezed
class Address with _$Address {
  const factory Address({
    required String city,
    required String street,
  }) = _Address;

  factory Address.fromJson(Map<String, dynamic> json) => _$AddressFromJson(json);
}

@freezed
class Employee with _$Employee {
  const factory Employee({
    required String id,
    required String name,
  }) = _Employee;

  factory Employee.fromJson(Map<String, dynamic> json) => _$EmployeeFromJson(json);
}
```

---

## 7. 泛型支持

### 7.1 基本泛型类

```dart
@freezed
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({
    required bool success,
    T? data,
    String? error,
  }) = _ApiResponse<T>;

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson<T>(json, fromJsonT);
}

void main() {
  final userResponse = ApiResponse<User>(
    success: true,
    data: User(id: '1', name: 'Alice'),
  );

  final productResponse = ApiResponse<Product>(
    success: true,
    data: Product(id: '1', name: 'iPhone'),
  );
}
```

### 7.2 泛型联合类型

```dart
@freezed
sealed class Result<T> with _$Result<T> {
  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(String error) = Failure<T>;

  factory Result.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ResultFromJson<T>(json, fromJsonT);
}

void main() {
  final result = Result<User>.success(User(id: '1', name: 'Alice'));

  switch (result) {
    case Success(data: final user):
      print('User: ${user.name}');
    case Failure(error: final msg):
      print('Error: $msg');
  }
}
```

### 7.3 泛型约束

```dart
@freezed
class PaginatedList<T extends Serializable> with _$PaginatedList<T> {
  const factory PaginatedList({
    required List<T> items,
    required int total,
    required int page,
  }) = _PaginatedList<T>;

  factory PaginatedList.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$PaginatedListFromJson<T>(json, fromJsonT);
}

abstract class Serializable {
  Map<String, dynamic> toJson();
}
```

---

## 8. 高级特性

### 8.1 添加方法和 Getter

```dart
@freezed
class Rectangle with _$Rectangle {
  const Rectangle._();  // 私有构造函数用于添加方法

  const factory Rectangle({
    required double width,
    required double height,
  }) = _Rectangle;

  // 自定义 getter
  double get area => width * height;
  double get perimeter => 2 * (width + height);

  // 自定义方法
  Rectangle scale(double factor) {
    return copyWith(
      width: width * factor,
      height: height * factor,
    );
  }
}

void main() {
  final rect = Rectangle(width: 10, height: 5);
  print(rect.area);        // 50
  print(rect.perimeter);   // 30

  final scaled = rect.scale(2);
  print(scaled.area);      // 200
}
```

### 8.2 继承其他类

```dart
abstract class Entity {
  String get id;
  DateTime get createdAt;
}

@freezed
class User extends Entity with _$User {
  const User._();

  const factory User({
    required String id,
    required String name,
    required DateTime createdAt,
  }) = _User;

  @override
  String get displayName => name.toUpperCase();
}
```

### 8.3 实现接口

```dart
abstract class Printable {
  String printString();
}

@freezed
class Document with _$Document implements Printable {
  const factory Document({
    required String title,
    required String content,
  }) = _Document;

  @override
  String printString() => 'Document: $title';
}
```

### 8.4 自定义 toString

```dart
@freezed
class User with _$User {
  const User._();

  const factory User({
    required String id,
    required String name,
    required String email,
  }) = _User;

  @override
  String toString() => 'User(id: $id, name: $name)';
}

void main() {
  final user = User(id: '1', name: 'Alice', email: 'alice@example.com');
  print(user);  // User(id: 1, name: Alice)
}
```

### 8.5 自定义相等性

```dart
@freezed
class Person with _$Person {
  const Person._();

  const factory Person({
    required String id,
    required String name,
    required DateTime createdAt,  // 不参与相等性比较
  }) = _Person;

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is Person && other.id == id && other.name == name;
  }

  @override
  int get hashCode => Object.hash(id, name);
}
```

---

## 9. build.yaml 全局配置

### 9.1 基本配置结构

```yaml
# build.yaml

targets:
  $default:
    builders:
      freezed:
        options:
          # 全局配置选项
          generic_argument_factories: true

      json_serializable:
        options:
          field_rename: snake
          explicit_to_json: true
```

### 9.2 完整配置示例

```yaml
# build.yaml

targets:
  $default:
    builders:
      freezed:
        options:
          # 为泛型类生成参数工厂
          generic_argument_factories: true

          # 禁用格式化（提高性能）
          format: false

      json_serializable:
        options:
          # 字段命名转换
          field_rename: snake

          # 显式调用嵌套对象的 toJson
          explicit_to_json: true

          # 不包含 null 值
          include_if_null: false

          # 启用运行时类型检查
          checked: true
```

### 9.3 按目录配置

```yaml
# build.yaml

targets:
  $default:
    builders:
      freezed:
        generate_for:
          - lib/models/**/*.dart
          - lib/entities/**/*.dart

        options:
          generic_argument_factories: true
```

---

## 10. 实战应用示例

### 10.1 完整的 BLoC 状态管理

```dart
// lib/bloc/user_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_state.freezed.dart';

@freezed
sealed class UserState with _$UserState {
  const factory UserState.initial() = UserInitial;

  const factory UserState.loading() = UserLoading;

  const factory UserState.loaded({
    required User user,
    required List<Order> orders,
  }) = UserLoaded;

  const factory UserState.error({
    required String message,
    required int code,
  }) = UserError;
}

// lib/bloc/user_event.dart
@freezed
sealed class UserEvent with _$UserEvent {
  const factory UserEvent.fetchUser(String userId) = FetchUser;
  const factory UserEvent.updateUser(User user) = UpdateUser;
  const factory UserEvent.deleteUser() = DeleteUser;
}

// 使用
class UserBloc extends Bloc<UserEvent, UserState> {
  UserBloc() : super(const UserState.initial()) {
    on<FetchUser>(_onFetchUser);
    on<UpdateUser>(_onUpdateUser);
  }

  Future<void> _onFetchUser(FetchUser event, Emitter<UserState> emit) async {
    emit(const UserState.loading());
    try {
      final user = await _repository.getUser(event.userId);
      final orders = await _repository.getOrders(event.userId);
      emit(UserState.loaded(user: user, orders: orders));
    } catch (e) {
      emit(UserState.error(message: e.toString(), code: 500));
    }
  }

  Future<void> _onUpdateUser(UpdateUser event, Emitter<UserState> emit) async {
    final currentState = state;
    if (currentState is UserLoaded) {
      emit(currentState.copyWith(user: event.user));
    }
  }
}
```

### 10.2 API 响应模型

```dart
// lib/models/api_response.dart
@freezed
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({
    required bool success,
    required int code,
    String? message,
    T? data,
    Pagination? pagination,
  }) = _ApiResponse<T>;

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson<T>(json, fromJsonT);
}

@freezed
class Pagination with _$Pagination {
  const factory Pagination({
    required int page,
    required int perPage,
    required int total,
    required int totalPages,
  }) = _Pagination;

  factory Pagination.fromJson(Map<String, dynamic> json) =>
    _$PaginationFromJson(json);
}

// 使用
final response = ApiResponse<List<User>>.fromJson(
  json,
  (data) => (data as List)
    .map((e) => User.fromJson(e as Map<String, dynamic>))
    .toList(),
);
```

### 10.3 表单状态管理

```dart
// lib/models/form_state.dart
@freezed
class FormState with _$FormState {
  const factory FormState({
    @Default('') String email,
    @Default('') String password,
    @Default(false) bool isSubmitting,
    @Default(false) bool isValid,
    String? emailError,
    String? passwordError,
  }) = _FormState;
}

// 表单验证逻辑
extension FormStateValidation on FormState {
  FormState validate() {
    String? emailErr;
    String? passwordErr;

    if (email.isEmpty) {
      emailErr = '邮箱不能为空';
    } else if (!email.contains('@')) {
      emailErr = '请输入有效的邮箱';
    }

    if (password.length < 6) {
      passwordErr = '密码至少6位';
    }

    return copyWith(
      emailError: emailErr,
      passwordError: passwordErr,
      isValid: emailErr == null && passwordErr == null,
    );
  }
}

// 使用
class FormCubit extends Cubit<FormState> {
  FormCubit() : super(const FormState());

  void updateEmail(String email) {
    emit(state.copyWith(email: email).validate());
  }

  void updatePassword(String password) {
    emit(state.copyWith(password: password).validate());
  }

  Future<void> submit() async {
    if (!state.isValid) return;

    emit(state.copyWith(isSubmitting: true));
    // 提交逻辑...
  }
}
```

### 10.4 配置管理

```dart
// lib/models/app_config.dart
@freezed
class AppConfig with _$AppConfig {
  const factory AppConfig({
    @Default('1.0.0') String version,
    @Default('development') String environment,
    required ApiConfig api,
    required ThemeConfig theme,
    required FeatureFlags features,
  }) = _AppConfig;

  factory AppConfig.fromJson(Map<String, dynamic> json) =>
    _$AppConfigFromJson(json);

  // 默认配置
  static const AppConfig defaultConfig = AppConfig(
    api: ApiConfig(baseUrl: 'https://api.example.com', timeout: 30),
    theme: ThemeConfig(primaryColor: '#007AFF', darkMode: false),
    features: FeatureFlags(),
  );
}

@freezed
class ApiConfig with _$ApiConfig {
  const factory ApiConfig({
    @Default('https://api.example.com') String baseUrl,
    @Default(30) int timeout,
    @Default(3) int retryCount,
  }) = _ApiConfig;

  factory ApiConfig.fromJson(Map<String, dynamic> json) =>
    _$ApiConfigFromJson(json);
}

@freezed
class ThemeConfig with _$ThemeConfig {
  const factory ThemeConfig({
    @Default('#007AFF') String primaryColor,
    @Default(false) bool darkMode,
    @Default('system') String fontScale,
  }) = _ThemeConfig;

  factory ThemeConfig.fromJson(Map<String, dynamic> json) =>
    _$ThemeConfigFromJson(json);
}

@freezed
class FeatureFlags with _$FeatureFlags {
  const factory FeatureFlags({
    @Default(true) bool newDashboard,
    @Default(false) bool betaFeature,
    @Default(true) bool pushNotifications,
  }) = _FeatureFlags;

  factory FeatureFlags.fromJson(Map<String, dynamic> json) =>
    _$FeatureFlagsFromJson(json);
}
```

### 10.5 购物车状态管理

```dart
// lib/models/cart_state.dart
@freezed
class CartState with _$CartState {
  const factory CartState({
    @Default([]) List<CartItem> items,
    @Default(false) bool isLoading,
    String? error,
  }) = _CartState;

  // 计算属性
  double get totalPrice => items.fold(
    0, (sum, item) => sum + item.totalPrice,
  );

  int get totalItems => items.fold(
    0, (sum, item) => sum + item.quantity,
  );
}

@freezed
class CartItem with _$CartItem {
  const factory CartItem({
    required String productId,
    required String name,
    required double price,
    @Default(1) int quantity,
    String? imageUrl,
  }) = _CartItem;

  factory CartItem.fromJson(Map<String, dynamic> json) =>
    _$CartItemFromJson(json);

  double get totalPrice => price * quantity;
}

// 购物车逻辑
extension CartStateLogic on CartState {
  CartState addItem(CartItem newItem) {
    final existingIndex = items.indexWhere(
      (item) => item.productId == newItem.productId,
    );

    if (existingIndex >= 0) {
      // 更新数量
      final updatedItems = [...items];
      updatedItems[existingIndex] = updatedItems[existingIndex].copyWith(
        quantity: updatedItems[existingIndex].quantity + newItem.quantity,
      );
      return copyWith(items: updatedItems);
    } else {
      // 添加新项
      return copyWith(items: [...items, newItem]);
    }
  }

  CartState removeItem(String productId) {
    return copyWith(
      items: items.where((item) => item.productId != productId).toList(),
    );
  }

  CartState updateQuantity(String productId, int quantity) {
    if (quantity <= 0) return removeItem(productId);

    return copyWith(
      items: items.map((item) {
        if (item.productId == productId) {
          return item.copyWith(quantity: quantity);
        }
        return item;
      }).toList(),
    );
  }
}
```

---

## 11. 最佳实践与常见问题

### 11.1 最佳实践

```dart
// ✅ 1. 使用 const 构造函数
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
  }) = _User;
}

// 创建实例时使用 const
const user = User(id: '1', name: 'Alice');

// ✅ 2. 组织文件结构
// lib/
// └── models/
//     ├── user/
//     │   ├── user.dart
//     │   ├── user.freezed.dart
//     │   └── user.g.dart
//     └── product/
//         ├── product.dart
//         ├── product.freezed.dart
//         └── product.g.dart

// ✅ 3. 为复杂模型添加扩展方法
@freezed
class Order with _$Order {
  const factory Order({
    required String id,
    required List<OrderItem> items,
  }) = _Order;
}

extension OrderCalculations on Order {
  double get total => items.fold(0, (sum, item) => sum + item.subtotal);
  int get itemCount => items.length;
}

// ✅ 4. 使用 sealed class 进行状态管理
@freezed
sealed class AsyncValue<T> with _$AsyncValue<T> {
  const factory AsyncValue.loading() = AsyncLoading<T>;
  const factory AsyncValue.data(T value) = AsyncData<T>;
  const factory AsyncValue.error(Object error, StackTrace stackTrace) = AsyncError<T>;
}

// ✅ 5. 使用 @Default 提供合理的默认值
@freezed
class PaginationParams with _$PaginationParams {
  const factory PaginationParams({
    @Default(1) int page,
    @Default(20) int perPage,
    @Default('created_at') String sortBy,
    @Default('desc') String sortOrder,
  }) = _PaginationParams;
}
```

### 11.2 常见问题

**问题 1：生成的文件不存在**

```bash
# 错误：Target of URI doesn't exist: 'user.freezed.dart'

# 解决方案：运行代码生成
dart run build_runner build --delete-conflicting-outputs
```

**问题 2：part 语句顺序错误**

```dart
// ❌ 错误顺序
part 'user.g.dart';      // json_serializable 生成的文件
part 'user.freezed.dart'; // freezed 生成的文件

// ✅ 正确顺序
part 'user.freezed.dart'; // 必须先导入 freezed 生成的文件
part 'user.g.dart';       // 然后导入 json_serializable 生成的文件
```

**问题 3：泛型类的 JSON 序列化**

```dart
// ✅ 启用 generic_argument_factories
@Freezed(genericArgumentFactories: true)
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({
    required bool success,
    T? data,
  }) = _ApiResponse<T>;

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson<T>(json, fromJsonT);
}
```

**问题 4：集合修改问题**

```dart
// ❌ 错误：直接修改集合
final user = User(tags: ['a', 'b']);
user.tags.add('c');  // 抛出 UnsupportedError

// ✅ 正确：使用 copyWith
final updated = user.copyWith(tags: [...user.tags, 'c']);
```

### 11.3 调试技巧

```dart
// 1. 打印生成的代码
cat lib/models/user.freezed.dart

// 2. 使用 toString 调试
final user = User(id: '1', name: 'Alice');
print(user);  // User(id: 1, name: Alice)

// 3. 验证相等性
final user1 = User(id: '1', name: 'Alice');
final user2 = User(id: '1', name: 'Alice');
print(user1 == user2);  // true
print(identical(user1, user2));  // false

// 4. 检查 copyWith 行为
final original = User(id: '1', name: 'Alice');
final copy = original.copyWith(name: 'Bob');
print(original.name);  // Alice（原对象不变）
print(copy.name);      // Bob
```

---

## 12. 附录：速查表

### 12.1 注解速查表

| 注解              | 用途             | 示例                                       |
| ----------------- | ---------------- | ------------------------------------------ |
| `@freezed`        | 生成不可变数据类 | `@freezed class User with _$User`          |
| `@unfreezed`      | 生成可变数据类   | `@unfreezed class Counter with _$Counter`  |
| `@Default(value)` | 设置默认值       | `@Default(10) int count`                   |
| `@JsonKey()`      | JSON 字段配置    | `@JsonKey(name: 'user_id')`                |
| `@Freezed()`      | 类级别配置       | `@Freezed(genericArgumentFactories: true)` |

### 12.2 @Freezed 配置参数

| 参数                          | 类型             | 默认值 | 说明                 |
| ----------------------------- | ---------------- | ------ | -------------------- |
| `genericArgumentFactories`    | bool             | false  | 为泛型类生成参数工厂 |
| `unionKey`                    | String?          | null   | 联合类型的 JSON 键   |
| `unionValueCase`              | FreezedUnionCase | none   | 联合类型值的命名风格 |
| `makeCollectionsUnmodifiable` | bool             | true   | 集合是否不可变       |
| `addImplicitFinal`            | bool             | true   | 自动添加 final       |
| `copyWith`                    | bool             | true   | 生成 copyWith        |
| `equal`                       | bool             | true   | 生成 == 和 hashCode  |
| `toString`                    | bool             | true   | 生成 toString        |
| `fromJson`                    | bool             | true   | 生成 fromJson        |
| `toJson`                      | bool             | true   | 生成 toJson          |

### 12.3 代码模板

**基础数据类**：

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part '{{name}}.freezed.dart';

@freezed
class {{Name}} with _${{Name}} {
  const factory {{Name}}({
    required String id,
    required String name,
  }) = _{{Name}};
}
```

**带 JSON 序列化的数据类**：

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part '{{name}}.freezed.dart';
part '{{name}}.g.dart';

@freezed
class {{Name}} with _${{Name}} {
  const factory {{Name}}({
    required String id,
    required String name,
  }) = _{{Name}};

  factory {{Name}}.fromJson(Map<String, dynamic> json) =>
    _${{Name}}FromJson(json);
}
```

**联合类型（状态管理）**：

```dart
@freezed
sealed class {{StateName}} with _${{StateName}} {
  const factory {{StateName}}.initial() = {{StateName}}Initial;
  const factory {{StateName}}.loading() = {{StateName}}Loading;
  const factory {{StateName}}.loaded({{DataType}} data) = {{StateName}}Loaded;
  const factory {{StateName}}.error(String message) = {{StateName}}Error;
}
```

**泛型类**：

```dart
@Freezed(genericArgumentFactories: true)
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({
    required bool success,
    T? data,
  }) = _ApiResponse<T>;

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson<T>(json, fromJsonT);
}
```

### 12.4 常用命令速查

```bash
# 安装依赖
flutter pub add freezed_annotation
flutter pub add --dev build_runner freezed json_serializable

# 生成代码（一次性）
dart run build_runner build

# 生成代码并删除冲突文件
dart run build_runner build --delete-conflicting-outputs

# 监视模式（自动重新生成）
dart run build_runner watch

# 清理生成的文件
dart run build_runner clean

# 删除并重新生成
rm -rf lib/**/*.freezed.dart lib/**/*.g.dart && dart run build_runner build
```

---

## 总结

Freezed 是 Dart/Flutter 生态中最强大的代码生成库之一，它彻底改变了我们编写数据类的方式：

1. **不可变性**：自动创建不可变数据类，避免意外的状态修改

2. **copyWith**：优雅的深度拷贝机制，支持嵌套对象更新

3. **相等性**：自动实现值相等性比较，适用于集合和状态管理

4. **联合类型**：与 Dart 3 的 sealed classes 完美配合，实现类型安全的状态管理

5. **JSON 序列化**：与 json_serializable 无缝集成，自动生成序列化代码

6. **泛型支持**：完整支持泛型类和泛型联合类型

掌握 Freezed，你将能够以更少的代码、更高的安全性构建 Dart/Flutter 应用程序，专注于业务逻辑而非样板代码。
