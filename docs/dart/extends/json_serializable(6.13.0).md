# json_serializable 库详解

> 本文基于 json_serializable: ^6.13.0 版本，深入解析这个 Dart/Flutter 中最流行的 JSON 序列化代码生成库。通过注解驱动的方式，json_serializable 能够自动生成类型安全的 `fromJson` 和 `toJson` 方法，彻底告别繁琐的手动序列化代码。

---

## 目录

1. [简介与安装](#1-简介与安装)
2. [快速入门](#2-快速入门)
3. [@JsonSerializable 注解详解](#3-jsonserializable-注解详解)
4. [@JsonKey 注解详解](#4-jsonkey-注解详解)
5. [JsonConverter 自定义转换器](#5-jsonconverter-自定义转换器)
6. [泛型支持](#6-泛型支持)
7. [嵌套对象与集合](#7-嵌套对象与集合)
8. [build.yaml 全局配置](#8-buildyaml-全局配置)
9. [实战应用示例](#9-实战应用示例)
10. [最佳实践与常见问题](#10-最佳实践与常见问题)
11. [附录：速查表](#11-附录速查表)

---

## 1. 简介与安装

### 1.1 什么是 json_serializable？

json_serializable 是一个 Dart 构建系统（build system）生成器，它通过代码生成技术，自动为标注了特定注解的 Dart 类创建 JSON 序列化和反序列化代码。

**手动序列化的痛点**：

```dart
// 手动实现 fromJson/toJson - 繁琐且容易出错
class User {
  final String name;
  final int age;
  final String email;
  final DateTime createdAt;
  final List<String> tags;

  User(this.name, this.age, this.email, this.createdAt, this.tags);

  // 手动实现 fromJson - 容易出错
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      json['name'] as String,
      json['age'] as int,
      json['email'] as String,
      DateTime.parse(json['createdAt'] as String),
      (json['tags'] as List).map((e) => e as String).toList(),
    );
  }

  // 手动实现 toJson - 容易遗漏字段
  Map<String, dynamic> toJson() {
    return {
      'name': name,
      'age': age,
      'email': email,
      'createdAt': createdAt.toIso8601String(),
      'tags': tags,
    };
  }
}
```

**使用 json_serializable 的优雅方案**：

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart';  // 生成的代码文件

@JsonSerializable()
class User {
  final String name;
  final int age;
  final String email;
  final DateTime createdAt;
  final List<String> tags;

  User(this.name, this.age, this.email, this.createdAt, this.tags);

  // 一行代码连接生成的序列化逻辑
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

### 1.2 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  json_annotation: ^4.9.0 # 运行时注解

dev_dependencies:
  build_runner: ^2.4.9 # 代码生成工具
  json_serializable: ^6.13.0 # 代码生成器
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

---

## 2. 快速入门

### 2.1 创建第一个模型类

```dart
// lib/models/user.dart
import 'package:json_annotation/json_annotation.dart';

// 必须：声明生成的 part 文件
part 'user.g.dart';

// 注解：告诉生成器为该类生成序列化代码
@JsonSerializable()
class User {
  final String name;
  final int age;
  final String email;

  // 构造函数
  User({
    required this.name,
    required this.age,
    required this.email,
  });

  // 工厂构造函数：连接生成的反序列化方法
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);

  // 方法：连接生成的序列化方法
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

### 2.2 生成代码

运行代码生成命令：

```bash
dart run build_runner build --delete-conflicting-outputs
```

生成的 `user.g.dart` 文件内容大致如下：

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'user.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

User _$UserFromJson(Map<String, dynamic> json) => User(
      name: json['name'] as String,
      age: json['age'] as int,
      email: json['email'] as String,
    );

Map<String, dynamic> _$UserToJson(User instance) => <String, dynamic>{
      'name': instance.name,
      'age': instance.age,
      'email': instance.email,
    };
```

### 2.3 使用模型类

```dart
void main() {
  // JSON 字符串
  const jsonString = '''
    {
      "name": "Alice",
      "age": 30,
      "email": "alice@example.com"
    }
  ''';

  // 解析 JSON
  final json = jsonDecode(jsonString) as Map<String, dynamic>;
  final user = User.fromJson(json);

  print(user.name);  // Alice
  print(user.age);   // 30

  // 序列化为 JSON
  final outputJson = user.toJson();
  print(jsonEncode(outputJson));
  // {"name":"Alice","age":30,"email":"alice@example.com"}
}
```

---

## 3. @JsonSerializable 注解详解

`@JsonSerializable` 是 json_serializable 的核心注解，用于控制类的序列化行为。

### 3.1 完整参数列表

```dart
@JsonSerializable(
  // 基础配置
  createFactory: true,           // 是否生成 fromJson 工厂构造函数
  createToJson: true,            // 是否生成 toJson 方法

  // 命名策略
  fieldRename: FieldRename.none, // 字段命名转换策略

  // null 值处理
  includeIfNull: true,           // 是否包含值为 null 的字段

  // 类型检查
  checked: false,                // 是否启用运行时类型检查
  disallowUnrecognizedKeys: false, // 是否禁止未识别的键

  // 嵌套对象序列化
  explicitToJson: false,         // 是否显式调用嵌套对象的 toJson

  // 泛型支持
  genericArgumentFactories: false, // 是否生成泛型参数工厂

  // 其他选项
  anyMap: false,                 // 是否使用 Map<String, dynamic> 而不是 Map
  createFieldMap: false,         // 是否生成字段映射
  createJsonKeys: false,         // 是否生成 JSON 键常量
  createJsonSchema: false,       // 是否生成 JSON Schema
  createPerFieldToJson: false,   // 是否为每个字段生成单独的 toJson 方法
  dateTimeUtc: false,            // DateTime 是否使用 UTC 格式
  ignoreUnannotated: false,      // 是否忽略未标注 @JsonKey 的字段
  constructor: '',               // 指定使用的构造函数（默认无参）
)
```

### 3.2 字段命名转换（fieldRename）

```dart
// 原始命名（默认）
@JsonSerializable(fieldRename: FieldRename.none)
class UserV1 {
  final String userName;  // JSON: "userName"
  final String emailAddress;  // JSON: "emailAddress"

  UserV1(this.userName, this.emailAddress);
  factory UserV1.fromJson(Map<String, dynamic> json) => _$UserV1FromJson(json);
  Map<String, dynamic> toJson() => _$UserV1ToJson(this);
}

// 蛇形命名（snake_case）
@JsonSerializable(fieldRename: FieldRename.snake)
class UserV2 {
  final String userName;  // JSON: "user_name"
  final String emailAddress;  // JSON: "email_address"

  UserV2(this.userName, this.emailAddress);
  factory UserV2.fromJson(Map<String, dynamic> json) => _$UserV2FromJson(json);
  Map<String, dynamic> toJson() => _$UserV2ToJson(this);
}

// 大写蛇形命名（SCREAMING_SNAKE_CASE）
@JsonSerializable(fieldRename: FieldRename.screamingSnake)
class UserV3 {
  final String userName;  // JSON: "USER_NAME"
  final String emailAddress;  // JSON: "EMAIL_ADDRESS"

  UserV3(this.userName, this.emailAddress);
  factory UserV3.fromJson(Map<String, dynamic> json) => _$UserV3FromJson(json);
  Map<String, dynamic> toJson() => _$UserV3ToJson(this);
}

// 大驼峰命名（PascalCase）
@JsonSerializable(fieldRename: FieldRename.pascal)
class UserV4 {
  final String userName;  // JSON: "UserName"
  final String emailAddress;  // JSON: "EmailAddress"

  UserV4(this.userName, this.emailAddress);
  factory UserV4.fromJson(Map<String, dynamic> json) => _$UserV4FromJson(json);
  Map<String, dynamic> toJson() => _$UserV4ToJson(this);
}

// 小驼峰命名（camelCase）
@JsonSerializable(fieldRename: FieldRename.kebab)
class UserV5 {
  final String userName;  // JSON: "user-name"
  final String emailAddress;  // JSON: "email-address"

  UserV5(this.userName, this.emailAddress);
  factory UserV5.fromJson(Map<String, dynamic> json) => _$UserV5FromJson(json);
  Map<String, dynamic> toJson() => _$UserV5ToJson(this);
}
```

### 3.3 null 值处理（includeIfNull）

```dart
// 包含 null 值（默认）
@JsonSerializable(includeIfNull: true)
class ProfileV1 {
  final String name;
  final String? bio;  // 可能为 null

  ProfileV1(this.name, this.bio);
  factory ProfileV1.fromJson(Map<String, dynamic> json) => _$ProfileV1FromJson(json);
  Map<String, dynamic> toJson() => _$ProfileV1ToJson(this);
}

// toJson 输出: {"name":"Alice","bio":null}

// 排除 null 值
@JsonSerializable(includeIfNull: false)
class ProfileV2 {
  final String name;
  final String? bio;

  ProfileV2(this.name, this.bio);
  factory ProfileV2.fromJson(Map<String, dynamic> json) => _$ProfileV2FromJson(json);
  Map<String, dynamic> toJson() => _$ProfileV2ToJson(this);
}

// toJson 输出（bio 为 null 时）: {"name":"Alice"}
// toJson 输出（bio 不为 null 时）: {"name":"Alice","bio":"Developer"}
```

### 3.4 运行时类型检查（checked）

```dart
// 启用运行时类型检查
@JsonSerializable(checked: true)
class Product {
  final String name;
  final double price;
  final int quantity;

  Product(this.name, this.price, this.quantity);
  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);
  Map<String, dynamic> toJson() => _$ProductToJson(this);
}

// 当 JSON 类型不匹配时，抛出 CheckedFromJsonException
// 例如：{"name": "Apple", "price": "not_a_number", "quantity": 10}
// 会抛出异常，指明 price 字段类型错误
```

### 3.5 禁止未识别的键（disallowUnrecognizedKeys）

```dart
// 禁止未识别的键
@JsonSerializable(disallowUnrecognizedKeys: true)
class StrictUser {
  final String name;
  final int age;

  StrictUser(this.name, this.age);
  factory StrictUser.fromJson(Map<String, dynamic> json) => _$StrictUserFromJson(json);
  Map<String, dynamic> toJson() => _$StrictUserToJson(this);
}

// 当 JSON 包含未定义的字段时抛出异常
// 例如：{"name": "Alice", "age": 30, "extraField": "value"}
// 会抛出 UnrecognizedKeysException
```

### 3.6 显式嵌套序列化（explicitToJson）

```dart
// 嵌套对象模型
@JsonSerializable()
class Address {
  final String city;
  final String street;

  Address(this.city, this.street);
  factory Address.fromJson(Map<String, dynamic> json) => _$AddressFromJson(json);
  Map<String, dynamic> toJson() => _$AddressToJson(this);
}

// 默认行为（explicitToJson: false）
@JsonSerializable()
class PersonV1 {
  final String name;
  final Address address;

  PersonV1(this.name, this.address);
  factory PersonV1.fromJson(Map<String, dynamic> json) => _$PersonV1FromJson(json);
  Map<String, dynamic> toJson() => _$PersonV1ToJson(this);
}

// toJson 输出: {"name":"Alice","address":{"city":"Beijing","street":"Main St"}}

// 显式嵌套序列化（explicitToJson: true）
@JsonSerializable(explicitToJson: true)
class PersonV2 {
  final String name;
  final Address address;

  PersonV2(this.name, this.address);
  factory PersonV2.fromJson(Map<String, dynamic> json) => _$PersonV2FromJson(json);
  Map<String, dynamic> toJson() => _$PersonV2ToJson(this);
}

// 效果相同，但确保嵌套对象的 toJson 被显式调用
// 在某些复杂场景下更可靠
```

### 3.7 控制生成的方法

```dart
// 只生成 fromJson（用于只读模型）
@JsonSerializable(createFactory: true, createToJson: false)
class ReadOnlyModel {
  final String data;
  ReadOnlyModel(this.data);
  factory ReadOnlyModel.fromJson(Map<String, dynamic> json) => _$ReadOnlyModelFromJson(json);
}

// 只生成 toJson（用于只写模型）
@JsonSerializable(createFactory: false, createToJson: true)
class WriteOnlyModel {
  final String data;
  WriteOnlyModel(this.data);
  Map<String, dynamic> toJson() => _$WriteOnlyModelToJson(this);
}
```

---

## 4. @JsonKey 注解详解

`@JsonKey` 用于自定义单个字段的序列化行为，优先级高于 `@JsonSerializable` 的设置。

### 4.1 完整参数列表

```dart
@JsonKey(
  // 字段名映射
  name: 'json_field_name',       // JSON 中的字段名

  // 默认值
  defaultValue: 'default',       // JSON 中字段缺失或为 null 时的默认值

  // 是否包含
  includeFromJson: true,         // 是否参与反序列化
  includeToJson: true,           // 是否参与序列化
  includeIfNull: true,           // null 值是否包含（覆盖类级别设置）

  // 自定义转换
  fromJson: customFromJson,      // 自定义反序列化函数
  toJson: customToJson,          // 自定义序列化函数

  // 必填字段
  required: false,               // 是否必填（JSON 必须包含该字段）
  disallowNullValue: false,      // 是否禁止 null 值

  // 忽略字段
  ignore: false,                 // 是否完全忽略该字段

  // 未知枚举值处理
  unknownEnumValue: Enum.value,  // 遇到未知枚举值时的默认值
)
```

### 4.2 字段名映射（name）

```dart
@JsonSerializable()
class ApiUser {
  @JsonKey(name: 'user_name')  // JSON 字段名为 user_name
  final String userName;

  @JsonKey(name: 'email_address')
  final String emailAddress;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  ApiUser(this.userName, this.emailAddress, this.createdAt);
  factory ApiUser.fromJson(Map<String, dynamic> json) => _$ApiUserFromJson(json);
  Map<String, dynamic> toJson() => _$ApiUserToJson(this);
}

// JSON: {"user_name": "Alice", "email_address": "alice@example.com", "created_at": "2024-01-01T00:00:00.000Z"}
```

### 4.3 默认值（defaultValue）

```dart
@JsonSerializable()
class Settings {
  @JsonKey(defaultValue: true)
  final bool notificationsEnabled;

  @JsonKey(defaultValue: 'en')
  final String language;

  @JsonKey(defaultValue: 10)
  final int itemsPerPage;

  @JsonKey(defaultValue: ['read', 'write'])
  final List<String> permissions;

  Settings(this.notificationsEnabled, this.language, this.itemsPerPage, this.permissions);
  factory Settings.fromJson(Map<String, dynamic> json) => _$SettingsFromJson(json);
  Map<String, dynamic> toJson() => _$SettingsToJson(this);
}

// 当 JSON 中缺少字段或为 null 时，使用默认值
// {"notificationsEnabled": null} -> notificationsEnabled = true
// {} -> 所有字段使用默认值
```

### 4.4 忽略字段（ignore）

```dart
@JsonSerializable()
class UserWithCache {
  final String id;
  final String name;

  @JsonKey(ignore: true)  // 不参与序列化/反序列化
  DateTime? cachedAt;

  @JsonKey(ignore: true)
  bool isLoading = false;

  UserWithCache(this.id, this.name);
  factory UserWithCache.fromJson(Map<String, dynamic> json) => _$UserWithCacheFromJson(json);
  Map<String, dynamic> toJson() => _$UserWithCacheToJson(this);
}

// toJson 输出: {"id": "123", "name": "Alice"}
// cachedAt 和 isLoading 不会被序列化
```

### 4.5 只序列化或只反序列化

```dart
@JsonSerializable()
class SecureUser {
  final String id;
  final String name;

  // 只反序列化（从 JSON 读取，但不写入）
  @JsonKey(includeToJson: false)
  final String? password;

  // 只序列化（写入 JSON，但不从 JSON 读取）
  @JsonKey(includeFromJson: false)
  final DateTime? lastLoginAt;

  SecureUser(this.id, this.name, {this.password, this.lastLoginAt});
  factory SecureUser.fromJson(Map<String, dynamic> json) => _$SecureUserFromJson(json);
  Map<String, dynamic> toJson() => _$SecureUserToJson(this);
}

// 从 JSON 读取 password，但不写入
// 写入 lastLoginAt，但不从 JSON 读取
```

### 4.6 必填字段（required）

```dart
@JsonSerializable()
class StrictProduct {
  @JsonKey(required: true)
  final String id;

  @JsonKey(required: true)
  final String name;

  final String? description;

  StrictProduct(this.id, this.name, this.description);
  factory StrictProduct.fromJson(Map<String, dynamic> json) => _$StrictProductFromJson(json);
  Map<String, dynamic> toJson() => _$StrictProductToJson(this);
}

// 如果 JSON 中缺少 id 或 name 字段，抛出 RequiredKeysMissingError
```

### 4.7 未知枚举值处理（unknownEnumValue）

```dart
enum Status { pending, active, inactive }

@JsonSerializable()
class Task {
  final String id;

  @JsonKey(unknownEnumValue: Status.pending)  // 未知枚举值默认转为 pending
  final Status status;

  Task(this.id, this.status);
  factory Task.fromJson(Map<String, dynamic> json) => _$TaskFromJson(json);
  Map<String, dynamic> toJson() => _$TaskToJson(this);
}

// JSON: {"id": "1", "status": "unknown_status"}
// 解析后: Task(id: "1", status: Status.pending)
```

---

## 5. JsonConverter 自定义转换器

当需要处理复杂类型转换时，可以实现 `JsonConverter` 接口。

### 5.1 基本用法

```dart
// 定义转换器
class DateTimeConverter implements JsonConverter<DateTime, String> {
  const DateTimeConverter();

  @override
  DateTime fromJson(String json) => DateTime.parse(json);

  @override
  String toJson(DateTime object) => object.toIso8601String();
}

// 使用转换器
@JsonSerializable()
class Event {
  final String name;

  @DateTimeConverter()
  final DateTime startTime;

  @DateTimeConverter()
  final DateTime endTime;

  Event(this.name, this.startTime, this.endTime);
  factory Event.fromJson(Map<String, dynamic> json) => _$EventFromJson(json);
  Map<String, dynamic> toJson() => _$EventToJson(this);
}
```

### 5.2 时间戳转换器

```dart
class TimestampConverter implements JsonConverter<DateTime, int> {
  const TimestampConverter();

  @override
  DateTime fromJson(int json) =>
    DateTime.fromMillisecondsSinceEpoch(json);

  @override
  int toJson(DateTime object) => object.millisecondsSinceEpoch;
}

@JsonSerializable()
class LogEntry {
  final String message;

  @TimestampConverter()
  final DateTime timestamp;

  LogEntry(this.message, this.timestamp);
  factory LogEntry.fromJson(Map<String, dynamic> json) => _$LogEntryFromJson(json);
  Map<String, dynamic> toJson() => _$LogEntryToJson(this);
}

// JSON: {"message": "User logged in", "timestamp": 1704067200000}
```

### 5.3 枚举转换器

```dart
enum Priority { low, medium, high }

class PriorityConverter implements JsonConverter<Priority, String> {
  const PriorityConverter();

  @override
  Priority fromJson(String json) {
    return Priority.values.firstWhere(
      (e) => e.name == json,
      orElse: () => Priority.medium,
    );
  }

  @override
  String toJson(Priority object) => object.name;
}

@JsonSerializable()
class Ticket {
  final String title;

  @PriorityConverter()
  final Priority priority;

  Ticket(this.title, this.priority);
  factory Ticket.fromJson(Map<String, dynamic> json) => _$TicketFromJson(json);
  Map<String, dynamic> toJson() => _$TicketToJson(this);
}

// JSON: {"title": "Bug fix", "priority": "high"}
```

### 5.4 集合类型转换器

```dart
class StringListConverter implements JsonConverter<List<String>, String> {
  const StringListConverter();

  @override
  List<String> fromJson(String json) => json.split(',');

  @override
  String toJson(List<String> object) => object.join(',');
}

@JsonSerializable()
class TagGroup {
  final String name;

  @StringListConverter()
  final List<String> tags;

  TagGroup(this.name, this.tags);
  factory TagGroup.fromJson(Map<String, dynamic> json) => _$TagGroupFromJson(json);
  Map<String, dynamic> toJson() => _$TagGroupToJson(this);
}

// JSON: {"name": "Frontend", "tags": "dart,flutter,js"}
// 解析后: TagGroup(name: "Frontend", tags: ["dart", "flutter", "js"])
```

### 5.5 通用集合转换器

```dart
class ListConverter<T> implements JsonConverter<List<T>, List<dynamic>> {
  final T Function(dynamic) fromJsonT;
  final dynamic Function(T) toJsonT;

  const ListConverter(this.fromJsonT, this.toJsonT);

  @override
  List<T> fromJson(List<dynamic> json) => json.map(fromJsonT).toList();

  @override
  List<dynamic> toJson(List<T> object) => object.map(toJsonT).toList();
}

// 使用
@JsonSerializable(genericArgumentFactories: true)
class Container<T> {
  final String name;

  @ListConverter<int>(
    (json) => json as int,
    (item) => item,
  )
  final List<int> items;

  Container(this.name, this.items);
  factory Container.fromJson(Map<String, dynamic> json, T Function(Object?) fromJsonT) =>
    _$ContainerFromJson(json, fromJsonT);
  Map<String, dynamic> toJson(Object? Function(T) toJsonT) => _$ContainerToJson(this, toJsonT);
}
```

---

## 6. 泛型支持

### 6.1 基本泛型类

```dart
@JsonSerializable(genericArgumentFactories: true)
class ApiResponse<T> {
  final bool success;
  final String? message;
  final T? data;

  ApiResponse({
    required this.success,
    this.message,
    this.data,
  });

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson(json, fromJsonT);

  Map<String, dynamic> toJson(Object? Function(T) toJsonT) =>
    _$ApiResponseToJson(this, toJsonT);
}

// 使用
final response = ApiResponse<User>.fromJson(
  json,
  (json) => User.fromJson(json as Map<String, dynamic>),
);
```

### 6.2 泛型列表

```dart
@JsonSerializable(genericArgumentFactories: true)
class PaginatedList<T> {
  final List<T> items;
  final int total;
  final int page;
  final int perPage;

  PaginatedList({
    required this.items,
    required this.total,
    required this.page,
    required this.perPage,
  });

  factory PaginatedList.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$PaginatedListFromJson(json, fromJsonT);

  Map<String, dynamic> toJson(Object? Function(T) toJsonT) =>
    _$PaginatedListToJson(this, toJsonT);
}

// 使用
final users = PaginatedList<User>.fromJson(
  json,
  (json) => User.fromJson(json as Map<String, dynamic>),
);
```

### 6.3 使用 JsonConverter 处理泛型

```dart
class GenericConverter<T> implements JsonConverter<T, Object?> {
  final T Function(Object?) fromJsonT;
  final Object? Function(T) toJsonT;

  const GenericConverter(this.fromJsonT, this.toJsonT);

  @override
  T fromJson(Object? json) => fromJsonT(json);

  @override
  Object? toJson(T object) => toJsonT(object);
}

@JsonSerializable()
class Wrapper<T> {
  final String type;

  @GenericConverter<User>(
    (json) => User.fromJson(json as Map<String, dynamic>),
    (user) => user.toJson(),
  )
  final T value;

  Wrapper(this.type, this.value);
  factory Wrapper.fromJson(Map<String, dynamic> json) => _$WrapperFromJson(json);
  Map<String, dynamic> toJson() => _$WrapperToJson(this);
}
```

---

## 7. 嵌套对象与集合

### 7.1 嵌套对象

```dart
@JsonSerializable()
class Address {
  final String city;
  final String street;
  final String zipCode;

  Address(this.city, this.street, this.zipCode);
  factory Address.fromJson(Map<String, dynamic> json) => _$AddressFromJson(json);
  Map<String, dynamic> toJson() => _$AddressToJson(this);
}

@JsonSerializable(explicitToJson: true)
class Person {
  final String name;
  final Address address;  // 嵌套对象

  Person(this.name, this.address);
  factory Person.fromJson(Map<String, dynamic> json) => _$PersonFromJson(json);
  Map<String, dynamic> toJson() => _$PersonToJson(this);
}

// JSON:
// {
//   "name": "Alice",
//   "address": {
//     "city": "Beijing",
//     "street": "Main St",
//     "zipCode": "100000"
//   }
// }
```

### 7.2 列表

```dart
@JsonSerializable(explicitToJson: true)
class Team {
  final String name;
  final List<Person> members;  // 嵌套对象列表

  Team(this.name, this.members);
  factory Team.fromJson(Map<String, dynamic> json) => _$TeamFromJson(json);
  Map<String, dynamic> toJson() => _$TeamToJson(this);
}

// JSON:
// {
//   "name": "Engineering",
//   "members": [
//     {"name": "Alice", "address": {...}},
//     {"name": "Bob", "address": {...}}
//   ]
// }
```

### 7.3 Map 类型

```dart
@JsonSerializable()
class Config {
  final String appName;
  final Map<String, dynamic> settings;  // 动态 Map
  final Map<String, int> counters;      // 类型化 Map

  Config(this.appName, this.settings, this.counters);
  factory Config.fromJson(Map<String, dynamic> json) => _$ConfigFromJson(json);
  Map<String, dynamic> toJson() => _$ConfigToJson(this);
}

// JSON:
// {
//   "appName": "MyApp",
//   "settings": {"theme": "dark", "notifications": true},
//   "counters": {"visits": 100, "clicks": 50}
// }
```

### 7.4 复杂嵌套结构

```dart
@JsonSerializable(explicitToJson: true)
class Company {
  final String name;
  final Address headquarters;
  final List<Department> departments;
  final Map<String, List<Employee>> employeesByDepartment;

  Company({
    required this.name,
    required this.headquarters,
    required this.departments,
    required this.employeesByDepartment,
  });

  factory Company.fromJson(Map<String, dynamic> json) => _$CompanyFromJson(json);
  Map<String, dynamic> toJson() => _$CompanyToJson(this);
}

@JsonSerializable(explicitToJson: true)
class Department {
  final String id;
  final String name;
  final Department? parent;  // 自引用

  Department(this.id, this.name, this.parent);
  factory Department.fromJson(Map<String, dynamic> json) => _$DepartmentFromJson(json);
  Map<String, dynamic> toJson() => _$DepartmentToJson(this);
}

@JsonSerializable()
class Employee {
  final String id;
  final String name;
  final List<String> skills;

  Employee(this.id, this.name, this.skills);
  factory Employee.fromJson(Map<String, dynamic> json) => _$EmployeeFromJson(json);
  Map<String, dynamic> toJson() => _$EmployeeToJson(this);
}
```

---

## 8. build.yaml 全局配置

### 8.1 基本配置结构

```yaml
# build.yaml

targets:
  $default:
    builders:
      json_serializable:
        options:
          # 全局配置选项
          field_rename: snake
          include_if_null: false
          explicit_to_json: true
          checked: false
          create_factory: true
          create_to_json: true
          generic_argument_factories: false
          any_map: false
          create_field_map: false
          create_json_keys: false
          create_json_schema: false
          disallow_unrecognized_keys: false
```

### 8.2 按目录配置

```yaml
# build.yaml

targets:
  $default:
    builders:
      json_serializable:
        # 全局默认配置
        options:
          field_rename: none
          include_if_null: true

        # 为特定文件/目录覆盖配置
        generate_for:
          - lib/models/*.dart
          - lib/entities/*.dart

        # 特定文件的配置
        dev_options:
          # 开发环境配置
          checked: true

        release_options:
          # 发布环境配置
          checked: false
```

### 8.3 排除生成的文件 from 代码覆盖率

```yaml
# build.yaml

targets:
  $default:
    builders:
      source_gen:combining_builder:
        options:
          preamble: |
            // coverage:ignore-file

      json_serializable:
        options:
          field_rename: snake
```

### 8.4 完整配置示例

```yaml
# build.yaml

targets:
  $default:
    builders:
      json_serializable:
        options:
          # 字段命名：蛇形命名法
          field_rename: snake

          # 不包含 null 值字段
          include_if_null: false

          # 显式调用嵌套对象的 toJson
          explicit_to_json: true

          # 启用运行时类型检查（开发环境推荐）
          checked: true

          # 生成工厂构造函数
          create_factory: true

          # 生成 toJson 方法
          create_to_json: true

          # 禁止未识别的键
          disallow_unrecognized_keys: false

          # 泛型参数工厂
          generic_argument_factories: true

          # 使用 Map<dynamic, dynamic> 而不是 Map<String, dynamic>
          any_map: false

          # 生成字段映射
          create_field_map: false

          # 生成 JSON 键常量
          create_json_keys: false

          # 生成 JSON Schema
          create_json_schema: false

          # DateTime 使用 UTC
          date_time_utc: false

          # 只处理标注了 @JsonKey 的字段
          ignore_unannotated: false
```

---

## 9. 实战应用示例

### 9.1 完整的 API 响应模型

```dart
// lib/models/api_response.dart
import 'package:json_annotation/json_annotation.dart';

part 'api_response.g.dart';

@JsonSerializable(genericArgumentFactories: true)
class ApiResponse<T> {
  final bool success;
  final int code;
  final String? message;
  final T? data;
  final Pagination? pagination;

  ApiResponse({
    required this.success,
    required this.code,
    this.message,
    this.data,
    this.pagination,
  });

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson(json, fromJsonT);

  Map<String, dynamic> toJson(Object? Function(T) toJsonT) =>
    _$ApiResponseToJson(this, toJsonT);
}

@JsonSerializable()
class Pagination {
  final int page;
  final int perPage;
  final int total;
  final int totalPages;

  Pagination({
    required this.page,
    required this.perPage,
    required this.total,
    required this.totalPages,
  });

  factory Pagination.fromJson(Map<String, dynamic> json) =>
    _$PaginationFromJson(json);
  Map<String, dynamic> toJson() => _$PaginationToJson(this);
}

// 使用示例
void fetchUsers() async {
  final response = await http.get(Uri.parse('/api/users'));
  final json = jsonDecode(response.body);

  final apiResponse = ApiResponse<List<User>>.fromJson(
    json,
    (data) => (data as List)
      .map((e) => User.fromJson(e as Map<String, dynamic>))
      .toList(),
  );

  if (apiResponse.success) {
    final users = apiResponse.data!;
    print('获取到 ${users.length} 个用户');
  }
}
```

### 9.2 用户认证系统

```dart
// lib/models/auth.dart
import 'package:json_annotation/json_annotation.dart';

part 'auth.g.dart';

// 登录请求
@JsonSerializable()
class LoginRequest {
  @JsonKey(name: 'email')
  final String email;

  @JsonKey(name: 'password')
  final String password;

  @JsonKey(name: 'remember_me', defaultValue: false)
  final bool rememberMe;

  LoginRequest({
    required this.email,
    required this.password,
    this.rememberMe = false,
  });

  factory LoginRequest.fromJson(Map<String, dynamic> json) =>
    _$LoginRequestFromJson(json);
  Map<String, dynamic> toJson() => _$LoginRequestToJson(this);
}

// 登录响应
@JsonSerializable()
class LoginResponse {
  @JsonKey(name: 'access_token')
  final String accessToken;

  @JsonKey(name: 'refresh_token')
  final String refreshToken;

  @JsonKey(name: 'expires_in')
  final int expiresIn;

  final User user;

  LoginResponse({
    required this.accessToken,
    required this.refreshToken,
    required this.expiresIn,
    required this.user,
  });

  factory LoginResponse.fromJson(Map<String, dynamic> json) =>
    _$LoginResponseFromJson(json);
  Map<String, dynamic> toJson() => _$LoginResponseToJson(this);
}

// 用户信息
@JsonSerializable()
class User {
  final String id;
  final String email;

  @JsonKey(name: 'display_name')
  final String displayName;

  @JsonKey(name: 'avatar_url')
  final String? avatarUrl;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  @JsonKey(name: 'last_login_at')
  final DateTime? lastLoginAt;

  final List<String> roles;

  @JsonKey(ignore: true)
  bool isLoading = false;

  User({
    required this.id,
    required this.email,
    required this.displayName,
    this.avatarUrl,
    required this.createdAt,
    this.lastLoginAt,
    required this.roles,
  });

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

### 9.3 电商订单系统

```dart
// lib/models/order.dart
import 'package:json_annotation/json_annotation.dart';

part 'order.g.dart';

// 订单状态枚举
enum OrderStatus {
  pending,
  confirmed,
  shipped,
  delivered,
  cancelled,
}

// 订单模型
@JsonSerializable(explicitToJson: true)
class Order {
  final String id;

  @JsonKey(name: 'user_id')
  final String userId;

  @JsonKey(name: 'order_number')
  final String orderNumber;

  @JsonKey(unknownEnumValue: OrderStatus.pending)
  final OrderStatus status;

  final List<OrderItem> items;

  @JsonKey(name: 'shipping_address')
  final Address shippingAddress;

  @JsonKey(name: 'billing_address')
  final Address billingAddress;

  @JsonKey(name: 'subtotal')
  final double subtotal;

  @JsonKey(name: 'shipping_cost')
  final double shippingCost;

  @JsonKey(name: 'tax_amount')
  final double taxAmount;

  final double total;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  @JsonKey(name: 'updated_at')
  final DateTime updatedAt;

  Order({
    required this.id,
    required this.userId,
    required this.orderNumber,
    required this.status,
    required this.items,
    required this.shippingAddress,
    required this.billingAddress,
    required this.subtotal,
    required this.shippingCost,
    required this.taxAmount,
    required this.total,
    required this.createdAt,
    required this.updatedAt,
  });

  factory Order.fromJson(Map<String, dynamic> json) => _$OrderFromJson(json);
  Map<String, dynamic> toJson() => _$OrderToJson(this);
}

// 订单项
@JsonSerializable()
class OrderItem {
  @JsonKey(name: 'product_id')
  final String productId;

  final String name;

  final String? sku;

  final int quantity;

  @JsonKey(name: 'unit_price')
  final double unitPrice;

  final double total;

  @JsonKey(name: 'image_url')
  final String? imageUrl;

  OrderItem({
    required this.productId,
    required this.name,
    this.sku,
    required this.quantity,
    required this.unitPrice,
    required this.total,
    this.imageUrl,
  });

  factory OrderItem.fromJson(Map<String, dynamic> json) =>
    _$OrderItemFromJson(json);
  Map<String, dynamic> toJson() => _$OrderItemToJson(this);
}

// 地址
@JsonSerializable()
class Address {
  final String name;
  final String phone;
  final String line1;
  final String? line2;
  final String city;
  final String state;
  @JsonKey(name: 'postal_code')
  final String postalCode;
  final String country;

  Address({
    required this.name,
    required this.phone,
    required this.line1,
    this.line2,
    required this.city,
    required this.state,
    required this.postalCode,
    required this.country,
  });

  factory Address.fromJson(Map<String, dynamic> json) => _$AddressFromJson(json);
  Map<String, dynamic> toJson() => _$AddressToJson(this);
}
```

### 9.4 带版本控制的配置

```dart
// lib/models/app_config.dart
import 'package:json_annotation/json_annotation.dart';

part 'app_config.g.dart';

@JsonSerializable(fieldRename: FieldRename.snake)
class AppConfig {
  @JsonKey(defaultValue: '1.0.0')
  final String version;

  @JsonKey(defaultValue: 'development')
  final String environment;

  final ApiConfig api;
  final FeatureFlags features;
  final ThemeConfig theme;

  AppConfig({
    required this.version,
    required this.environment,
    required this.api,
    required this.features,
    required this.theme,
  });

  factory AppConfig.fromJson(Map<String, dynamic> json) =>
    _$AppConfigFromJson(json);
  Map<String, dynamic> toJson() => _$AppConfigToJson(this);

  // 默认配置
  static AppConfig get defaultConfig => AppConfig(
    version: '1.0.0',
    environment: 'development',
    api: ApiConfig(baseUrl: 'https://api.example.com', timeout: 30),
    features: FeatureFlags(),
    theme: ThemeConfig(primaryColor: '#007AFF', darkMode: false),
  );
}

@JsonSerializable()
class ApiConfig {
  @JsonKey(defaultValue: 'https://api.example.com')
  final String baseUrl;

  @JsonKey(defaultValue: 30)
  final int timeout;

  @JsonKey(defaultValue: 3)
  final int retryCount;

  ApiConfig({
    required this.baseUrl,
    required this.timeout,
    this.retryCount = 3,
  });

  factory ApiConfig.fromJson(Map<String, dynamic> json) =>
    _$ApiConfigFromJson(json);
  Map<String, dynamic> toJson() => _$ApiConfigToJson(this);
}

@JsonSerializable()
class FeatureFlags {
  @JsonKey(defaultValue: true)
  final bool newDashboard;

  @JsonKey(defaultValue: false)
  final bool betaFeature;

  @JsonKey(defaultValue: true)
  final bool pushNotifications;

  FeatureFlags({
    this.newDashboard = true,
    this.betaFeature = false,
    this.pushNotifications = true,
  });

  factory FeatureFlags.fromJson(Map<String, dynamic> json) =>
    _$FeatureFlagsFromJson(json);
  Map<String, dynamic> toJson() => _$FeatureFlagsToJson(this);
}

@JsonSerializable()
class ThemeConfig {
  @JsonKey(defaultValue: '#007AFF')
  final String primaryColor;

  @JsonKey(defaultValue: false)
  final bool darkMode;

  @JsonKey(defaultValue: 'system')
  final String fontScale;

  ThemeConfig({
    required this.primaryColor,
    required this.darkMode,
    this.fontScale = 'system',
  });

  factory ThemeConfig.fromJson(Map<String, dynamic> json) =>
    _$ThemeConfigFromJson(json);
  Map<String, dynamic> toJson() => _$ThemeConfigToJson(this);
}
```

### 9.5 时间戳转换实战

```dart
// lib/converters/timestamp_converter.dart
import 'package:json_annotation/json_annotation.dart';

// 秒级时间戳转换器
class SecondsTimestampConverter implements JsonConverter<DateTime, int> {
  const SecondsTimestampConverter();

  @override
  DateTime fromJson(int json) =>
    DateTime.fromMillisecondsSinceEpoch(json * 1000);

  @override
  int toJson(DateTime object) =>
    object.millisecondsSinceEpoch ~/ 1000;
}

// 毫秒级时间戳转换器
class MillisecondsTimestampConverter implements JsonConverter<DateTime, int> {
  const MillisecondsTimestampConverter();

  @override
  DateTime fromJson(int json) =>
    DateTime.fromMillisecondsSinceEpoch(json);

  @override
  int toJson(DateTime object) => object.millisecondsSinceEpoch;
}

// 使用示例
@JsonSerializable()
class LogEntry {
  final String level;
  final String message;
  final Map<String, dynamic> metadata;

  @SecondsTimestampConverter()
  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  LogEntry({
    required this.level,
    required this.message,
    required this.metadata,
    required this.createdAt,
  });

  factory LogEntry.fromJson(Map<String, dynamic> json) =>
    _$LogEntryFromJson(json);
  Map<String, dynamic> toJson() => _$LogEntryToJson(this);
}

// JSON: {"level": "INFO", "message": "User logged in", "created_at": 1704067200}
```

---

## 10. 最佳实践与常见问题

### 10.1 最佳实践

```dart
// ✅ 1. 始终使用 const 构造函数（如果可能）
@JsonSerializable()
class User {
  final String name;
  const User(this.name);  // const 构造函数
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}

// ✅ 2. 使用 final 字段
@JsonSerializable()
class Product {
  final String id;      // ✅ final
  final String name;    // ✅ final
  // String? description; // ❌ 避免可变字段

  Product(this.id, this.name);
  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);
  Map<String, dynamic> toJson() => _$ProductToJson(this);
}

// ✅ 3. 为复杂类型提供 copyWith 方法
@JsonSerializable()
class Settings {
  final String theme;
  final bool notifications;

  Settings({required this.theme, required this.notifications});

  Settings copyWith({String? theme, bool? notifications}) {
    return Settings(
      theme: theme ?? this.theme,
      notifications: notifications ?? this.notifications,
    );
  }

  factory Settings.fromJson(Map<String, dynamic> json) => _$SettingsFromJson(json);
  Map<String, dynamic> toJson() => _$SettingsToJson(this);
}

// ✅ 4. 使用明确的字段名
@JsonSerializable()
class Order {
  @JsonKey(name: 'order_id')  // 明确指定 JSON 字段名
  final String orderId;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  Order(this.orderId, this.createdAt);
  factory Order.fromJson(Map<String, dynamic> json) => _$OrderFromJson(json);
  Map<String, dynamic> toJson() => _$OrderToJson(this);
}

// ✅ 5. 处理 null 值
@JsonSerializable(includeIfNull: false)
class Profile {
  final String name;
  final String? bio;  // 可空字段
  final String? website;

  Profile({required this.name, this.bio, this.website});
  factory Profile.fromJson(Map<String, dynamic> json) => _$ProfileFromJson(json);
  Map<String, dynamic> toJson() => _$ProfileToJson(this);
}
```

### 10.2 常见问题

**问题 1：生成的文件不存在**

```bash
# 错误：Target of URI doesn't exist: 'user.g.dart'

# 解决方案：运行代码生成
dart run build_runner build --delete-conflicting-outputs
```

**问题 2：类型转换错误**

```dart
// 错误：type 'int' is not a subtype of type 'String'

// 解决方案：使用 checked: true 获取详细错误信息
@JsonSerializable(checked: true)
class User {
  final String age;  // 应该是 int
  User(this.age);
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

**问题 3：嵌套对象序列化失败**

```dart
// 错误：嵌套对象没有被正确序列化

// 解决方案：启用 explicitToJson
@JsonSerializable(explicitToJson: true)
class Company {
  final Address address;
  Company(this.address);
  factory Company.fromJson(Map<String, dynamic> json) => _$CompanyFromJson(json);
  Map<String, dynamic> toJson() => _$CompanyToJson(this);
}
```

**问题 4：枚举序列化失败**

```dart
// 解决方案：使用 unknownEnumValue
enum Status { active, inactive }

@JsonSerializable()
class Item {
  @JsonKey(unknownEnumValue: Status.active)
  final Status status;
  Item(this.status);
  factory Item.fromJson(Map<String, dynamic> json) => _$ItemFromJson(json);
  Map<String, dynamic> toJson() => _$ItemToJson(this);
}
```

### 10.3 调试技巧

```dart
// 1. 使用 checked: true 获取详细的类型错误信息
@JsonSerializable(checked: true)
class DebugModel {
  final String name;
  final int age;
  DebugModel(this.name, this.age);
  factory DebugModel.fromJson(Map<String, dynamic> json) =>
    _$DebugModelFromJson(json);
}

// 2. 打印原始 JSON 进行调试
void debugJson(Map<String, dynamic> json) {
  print('Raw JSON: ${jsonEncode(json)}');
  try {
    final model = DebugModel.fromJson(json);
    print('Parsed: $model');
  } catch (e) {
    print('Parse error: $e');
  }
}

// 3. 验证生成的代码
cat lib/models/user.g.dart
```

---

## 11. 附录：速查表

### 11.1 注解参数速查表

#### @JsonSerializable 参数

| 参数                       | 类型        | 默认值 | 说明                       |
| -------------------------- | ----------- | ------ | -------------------------- |
| `createFactory`            | bool        | true   | 生成 fromJson 工厂构造函数 |
| `createToJson`             | bool        | true   | 生成 toJson 方法           |
| `fieldRename`              | FieldRename | none   | 字段命名转换策略           |
| `includeIfNull`            | bool        | true   | 是否包含 null 值字段       |
| `checked`                  | bool        | false  | 启用运行时类型检查         |
| `disallowUnrecognizedKeys` | bool        | false  | 禁止未识别的键             |
| `explicitToJson`           | bool        | false  | 显式调用嵌套对象的 toJson  |
| `genericArgumentFactories` | bool        | false  | 生成泛型参数工厂           |
| `anyMap`                   | bool        | false  | 使用 Map<dynamic, dynamic> |
| `createFieldMap`           | bool        | false  | 生成字段映射               |
| `createJsonKeys`           | bool        | false  | 生成 JSON 键常量           |
| `createJsonSchema`         | bool        | false  | 生成 JSON Schema           |
| `dateTimeUtc`              | bool        | false  | DateTime 使用 UTC          |
| `ignoreUnannotated`        | bool        | false  | 只处理标注 @JsonKey 的字段 |

#### @JsonKey 参数

| 参数                | 类型     | 说明               |
| ------------------- | -------- | ------------------ |
| `name`              | String   | JSON 中的字段名    |
| `defaultValue`      | dynamic  | 默认值             |
| `includeFromJson`   | bool     | 是否参与反序列化   |
| `includeToJson`     | bool     | 是否参与序列化     |
| `includeIfNull`     | bool     | null 值是否包含    |
| `fromJson`          | Function | 自定义反序列化函数 |
| `toJson`            | Function | 自定义序列化函数   |
| `required`          | bool     | 是否必填           |
| `disallowNullValue` | bool     | 是否禁止 null 值   |
| `ignore`            | bool     | 是否完全忽略       |
| `unknownEnumValue`  | Enum     | 未知枚举值默认值   |

### 11.2 FieldRename 枚举值

| 值               | Dart 字段 | JSON 输出 |
| ---------------- | --------- | --------- |
| `none`           | userName  | userName  |
| `snake`          | userName  | user_name |
| `screamingSnake` | userName  | USER_NAME |
| `pascal`         | userName  | UserName  |
| `kebab`          | userName  | user-name |

### 11.3 代码模板

**基础模型**：

```dart
import 'package:json_annotation/json_annotation.dart';

part '{{name}}.g.dart';

@JsonSerializable()
class {{Name}} {
  final String id;
  final String name;

  {{Name}}({required this.id, required this.name});

  factory {{Name}}.fromJson(Map<String, dynamic> json) =>
    _${{Name}}FromJson(json);
  Map<String, dynamic> toJson() => _${{Name}}ToJson(this);
}
```

**带命名策略的模型**：

```dart
@JsonSerializable(fieldRename: FieldRename.snake)
class {{Name}} {
  final String fieldName;
  {{Name}}(this.fieldName);
  factory {{Name}}.fromJson(Map<String, dynamic> json) =>
    _${{Name}}FromJson(json);
  Map<String, dynamic> toJson() => _${{Name}}ToJson(this);
}
```

**泛型模型**：

```dart
@JsonSerializable(genericArgumentFactories: true)
class ApiResponse<T> {
  final bool success;
  final T? data;
  ApiResponse({required this.success, this.data});
  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object?) fromJsonT,
  ) => _$ApiResponseFromJson(json, fromJsonT);
  Map<String, dynamic> toJson(Object? Function(T) toJsonT) =>
    _$ApiResponseToJson(this, toJsonT);
}
```

### 11.4 常用命令速查

```bash
# 安装依赖
flutter pub add json_annotation
flutter pub add --dev build_runner json_serializable

# 生成代码（一次性）
dart run build_runner build

# 生成代码并删除冲突文件
dart run build_runner build --delete-conflicting-outputs

# 监视模式（自动重新生成）
dart run build_runner watch

# 清理生成的文件
dart run build_runner clean

# 删除并重新生成
rm -rf lib/**/*.g.dart && dart run build_runner build
```

---

## 总结

json_serializable 是 Dart/Flutter 生态中最成熟、最流行的 JSON 序列化解决方案：

1. **代码生成**：通过注解自动生成序列化代码，彻底告别手写样板代码

2. **类型安全**：编译时类型检查，运行时类型验证，确保数据安全

3. **灵活配置**：丰富的注解参数，支持全局配置和局部覆盖

4. **复杂场景支持**：泛型、嵌套对象、集合、枚举、自定义转换器等一应俱全

5. **开发体验**：配合 build_runner watch 实现实时代码生成，提升开发效率

掌握 json_serializable，你将能够高效、安全地处理 Dart/Flutter 应用中的所有 JSON 序列化需求。
