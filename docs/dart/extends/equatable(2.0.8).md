# Equatable 库详解

> 本文基于 equatable: ^2.0.8 版本，深入解析这个简化 Dart/Flutter 对象相等性比较的必备库。通过 Equatable，你可以告别繁琐的 `==` 和 `hashCode` 重写，只需一行代码即可实现基于值的相等性判断。

---

## 目录

1. [简介与安装](#1-简介与安装)
2. [Equatable 基类](#2-equatable-基类)
3. [props 属性详解](#3-props-属性详解)
4. [stringify 与 toString](#4-stringify-与-tostring)
5. [EquatableConfig 全局配置](#5-equatableconfig-全局配置)
6. [EquatableMixin](#6-equatablemixin)
7. [实用工具函数](#7-实用工具函数)
8. [实战应用示例](#8-实战应用示例)
9. [最佳实践与注意事项](#9-最佳实践与注意事项)
10. [附录：速查表](#10-附录速查表)

---

## 1. 简介与安装

### 1.1 什么是 Equatable？

Equatable 是一个 Dart 包，旨在**简化基于值的相等性比较**。在 Dart 中，默认的对象相等性比较使用的是**引用相等**（即比较两个对象是否指向内存中的同一位置），而非**值相等**（比较对象的内容是否相同）。

```dart
// 默认 Dart 行为：引用相等
class Person {
  final String name;
  const Person(this.name);
}

void main() {
  final bob1 = Person('Bob');
  final bob2 = Person('Bob');
  print(bob1 == bob2);  // false！尽管内容相同，但引用不同
}
```

Equatable 通过自动重写 `==` 运算符和 `hashCode` 方法，让你只需声明哪些属性参与相等性判断即可：

```dart
import 'package:equatable/equatable.dart';

class Person extends Equatable {
  final String name;
  const Person(this.name);

  @override
  List<Object> get props => [name];
}

void main() {
  final bob1 = Person('Bob');
  final bob2 = Person('Bob');
  print(bob1 == bob2);  // true！基于值的相等性
}
```

### 1.2 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  equatable: ^2.0.8
```

然后运行安装命令：

```bash
# Dart 项目
dart pub get

# Flutter 项目
flutter pub get
```

### 1.3 为什么需要 Equatable？

| 场景                | 不使用 Equatable              | 使用 Equatable      |
| ------------------- | ----------------------------- | ------------------- |
| 简单类（1-2个属性） | 需要写 10+ 行样板代码         | 只需 1 行 props     |
| 复杂类（10+个属性） | 需要写 50+ 行样板代码         | 只需 1 行 props     |
| 维护成本            | 新增属性需修改 == 和 hashCode | 只需在 props 中添加 |
| 错误风险            | 容易遗漏属性或逻辑错误        | 自动处理，零错误    |
| 集合支持            | 需手动处理 List/Map 深度比较  | 自动深度比较        |

---

## 2. Equatable 基类

### 2.1 基本用法

`Equatable` 是一个抽象基类，你需要继承它并重写 `props` 属性：

```dart
import 'package:equatable/equatable.dart';

class User extends Equatable {
  final String id;
  final String name;
  final int age;

  const User({
    required this.id,
    required this.name,
    required this.age,
  });

  @override
  List<Object> get props => [id, name, age];
}

void main() {
  final user1 = User(id: '1', name: 'Alice', age: 30);
  final user2 = User(id: '1', name: 'Alice', age: 30);
  final user3 = User(id: '2', name: 'Bob', age: 25);

  print(user1 == user2);  // true（所有属性相同）
  print(user1 == user3);  // false（属性不同）
  print(user1.hashCode == user2.hashCode);  // true（哈希码一致）
}
```

### 2.2 内部实现原理

Equatable 自动为你实现了以下逻辑：

```dart
// 这是 Equatable 自动生成的代码（伪代码）
@override
bool operator ==(Object other) {
  return identical(this, other) ||
      other is User &&
          runtimeType == other.runtimeType &&
          equals(props, other.props);  // 深度比较 props
}

@override
int get hashCode => runtimeType.hashCode ^ mapPropsToHashCode(props);
```

**比较逻辑**：

1. 首先检查是否是同一对象（`identical`）→ 返回 true
2. 检查运行时类型是否相同 → 不同则返回 false
3. 深度比较 `props` 列表中的每个属性 → 全部相同则返回 true

### 2.3 支持 const 构造函数

Equatable 完全支持 `const` 构造函数：

```dart
class Point extends Equatable {
  final double x;
  final double y;

  const Point(this.x, this.y);

  @override
  List<Object> get props => [x, y];
}

// 使用 const 构造函数
const origin = Point(0, 0);
const point1 = Point(1, 2);
const point2 = Point(1, 2);

print(point1 == point2);  // true
print(identical(point1, point2));  // true（const 对象会被复用）
```

### 2.4 支持可空属性

Equatable 支持可空（nullable）属性：

```dart
class Product extends Equatable {
  final String name;
  final double price;
  final String? description;  // 可空属性
  final DateTime? expiredAt;  // 可空属性

  const Product({
    required this.name,
    required this.price,
    this.description,
    this.expiredAt,
  });

  @override
  List<Object?> get props => [name, price, description, expiredAt];
}

void main() {
  final p1 = Product(name: 'Apple', price: 5.0, description: 'Fresh');
  final p2 = Product(name: 'Apple', price: 5.0, description: 'Fresh');
  final p3 = Product(name: 'Apple', price: 5.0, description: null);

  print(p1 == p2);  // true
  print(p1 == p3);  // false（description 不同）
}
```

---

## 3. props 属性详解

### 3.1 props 的基本概念

`props` 是 Equatable 的核心属性，它是一个列表，包含所有参与相等性比较的字段。

```dart
@override
List<Object?> get props => [field1, field2, field3, ...];
```

**重要规则**：

- 列表中的顺序**不重要**，但通常按声明顺序排列
- 所有参与相等性判断的字段都应该包含在内
- 不包含在 `props` 中的字段不会影响相等性判断

### 3.2 选择性比较字段

你可以通过控制 `props` 的内容来决定哪些字段参与相等性比较：

```dart
class Article extends Equatable {
  final String id;
  final String title;
  final String content;
  final DateTime createdAt;
  final int viewCount;  // 浏览次数（频繁变化）

  const Article({
    required this.id,
    required this.title,
    required this.content,
    required this.createdAt,
    this.viewCount = 0,
  });

  // 只比较核心字段，viewCount 不参与相等性判断
  @override
  List<Object?> get props => [id, title, content, createdAt];
}

void main() {
  final article1 = Article(
    id: '1',
    title: 'Hello',
    content: 'World',
    createdAt: DateTime(2024, 1, 1),
    viewCount: 100,
  );

  final article2 = Article(
    id: '1',
    title: 'Hello',
    content: 'World',
    createdAt: DateTime(2024, 1, 1),
    viewCount: 200,  // 不同，但不影响相等性
  );

  print(article1 == article2);  // true（viewCount 不参与比较）
}
```

### 3.3 集合类型的深度比较

Equatable 自动处理集合类型的**深度比较**：

```dart
class Order extends Equatable {
  final String orderId;
  final List<String> items;      // List 深度比较
  final Map<String, int> quantities;  // Map 深度比较
  final Set<String> tags;        // Set 深度比较

  const Order({
    required this.orderId,
    required this.items,
    required this.quantities,
    required this.tags,
  });

  @override
  List<Object?> get props => [orderId, items, quantities, tags];
}

void main() {
  final order1 = Order(
    orderId: 'ORD001',
    items: ['Apple', 'Banana'],
    quantities: {'Apple': 2, 'Banana': 3},
    tags: {'urgent', 'vip'},
  );

  final order2 = Order(
    orderId: 'ORD001',
    items: ['Apple', 'Banana'],
    quantities: {'Apple': 2, 'Banana': 3},
    tags: {'vip', 'urgent'},  // Set 顺序无关
  );

  print(order1 == order2);  // true（集合内容相同）
}
```

### 3.4 嵌套 Equatable 对象

Equatable 支持嵌套的 Equatable 对象：

```dart
class Address extends Equatable {
  final String city;
  final String street;
  final String zipCode;

  const Address(this.city, this.street, this.zipCode);

  @override
  List<Object?> get props => [city, street, zipCode];
}

class Customer extends Equatable {
  final String name;
  final Address address;  // 嵌套的 Equatable 对象

  const Customer(this.name, this.address);

  @override
  List<Object?> get props => [name, address];
}

void main() {
  final customer1 = Customer(
    'Alice',
    Address('Beijing', 'Main St', '100000'),
  );

  final customer2 = Customer(
    'Alice',
    Address('Beijing', 'Main St', '100000'),
  );

  print(customer1 == customer2);  // true（嵌套对象也会深度比较）
}
```

### 3.5 继承中的 props

在继承关系中，子类需要合并父类的 props：

```dart
class Animal extends Equatable {
  final String name;
  final int age;

  const Animal(this.name, this.age);

  @override
  List<Object?> get props => [name, age];
}

class Dog extends Animal {
  final String breed;

  const Dog(String name, int age, this.breed) : super(name, age);

  @override
  List<Object?> get props => [...super.props, breed];
}

void main() {
  final dog1 = Dog('Buddy', 3, 'Golden Retriever');
  final dog2 = Dog('Buddy', 3, 'Golden Retriever');
  final dog3 = Dog('Buddy', 3, 'Labrador');

  print(dog1 == dog2);  // true
  print(dog1 == dog3);  // false（breed 不同）
}
```

---

## 4. stringify 与 toString

### 4.1 启用 stringify

Equatable 可以自动生成包含所有 props 的 `toString()` 实现：

```dart
class User extends Equatable {
  final String name;
  final int age;

  const User(this.name, this.age);

  @override
  List<Object?> get props => [name, age];

  @override
  bool get stringify => true;  // 启用 toString 增强
}

void main() {
  final user = User('Alice', 30);
  print(user.toString());  // User(Alice, 30)
}
```

### 4.2 默认行为对比

| stringify 值    | toString() 输出   |
| --------------- | ----------------- |
| `false`（默认） | `User`            |
| `true`          | `User(Alice, 30)` |

### 4.3 自定义 toString

如果你需要更复杂的 `toString()` 格式，可以手动重写：

```dart
class User extends Equatable {
  final String firstName;
  final String lastName;
  final int age;

  const User(this.firstName, this.lastName, this.age);

  @override
  List<Object?> get props => [firstName, lastName, age];

  // 自定义 toString，不使用 stringify
  @override
  String toString() {
    return 'User{fullName: $firstName $lastName, age: $age}';
  }
}

void main() {
  final user = User('Alice', 'Smith', 30);
  print(user.toString());  // User{fullName: Alice Smith, age: 30}
}
```

---

## 5. EquatableConfig 全局配置

### 5.1 全局启用 stringify

你可以通过 `EquatableConfig` 为所有 Equatable 类统一配置 `stringify`：

```dart
import 'package:equatable/equatable.dart';

void main() {
  // 全局启用 stringify
  EquatableConfig.stringify = true;

  final user = User('Alice', 30);
  print(user.toString());  // User(Alice, 30)

  final product = Product('iPhone', 999);
  print(product.toString());  // Product(iPhone, 999)
}

class User extends Equatable {
  final String name;
  final int age;
  const User(this.name, this.age);
  @override
  List<Object?> get props => [name, age];
  // 不需要单独设置 stringify
}

class Product extends Equatable {
  final String name;
  final double price;
  const Product(this.name, this.price);
  @override
  List<Object?> get props => [name, price];
  // 不需要单独设置 stringify
}
```

### 5.2 局部配置优先

类级别的 `stringify` 设置会覆盖全局配置：

```dart
void main() {
  EquatableConfig.stringify = true;  // 全局启用

  final user = User('Alice', 30);
  print(user.toString());  // User(Alice, 30) - 遵循全局配置

  final product = Product('iPhone', 999);
  print(product.toString());  // Product - 局部配置覆盖全局
}

class User extends Equatable {
  final String name;
  final int age;
  const User(this.name, this.age);
  @override
  List<Object?> get props => [name, age];
  // 不设置 stringify，使用全局配置
}

class Product extends Equatable {
  final String name;
  final double price;
  const Product(this.name, this.price);
  @override
  List<Object?> get props => [name, price];

  @override
  bool get stringify => false;  // 局部覆盖全局
}
```

### 5.3 默认行为

`EquatableConfig.stringify` 的默认值：

- **Debug 模式**：`true`
- **Release 模式**：`false`

```dart
void main() {
  // 在 Debug 模式下，以下代码默认输出详细信息
  final user = User('Alice', 30);
  print(user.toString());  // User(Alice, 30) - Debug 模式

  // 在 Release 模式下，默认只输出类名
  // print(user.toString());  // User - Release 模式
}
```

---

## 6. EquatableMixin

### 6.1 为什么需要 Mixin？

Dart 不支持多继承，如果你的类已经继承了其他类，就无法再继承 `Equatable`。这时可以使用 `EquatableMixin`：

```dart
import 'package:equatable/equatable.dart';

// 假设你有一个基类
class BaseModel {
  final String id;
  final DateTime createdAt;

  BaseModel(this.id, this.createdAt);
}

// 使用 EquatableMixin 而不是继承 Equatable
class User extends BaseModel with EquatableMixin {
  final String name;
  final int age;

  User(
    String id,
    DateTime createdAt,
    this.name,
    this.age,
  ) : super(id, createdAt);

  @override
  List<Object?> get props => [id, createdAt, name, age];
}

void main() {
  final user1 = User('1', DateTime(2024, 1, 1), 'Alice', 30);
  final user2 = User('1', DateTime(2024, 1, 1), 'Alice', 30);

  print(user1 == user2);  // true
}
```

### 6.2 Mixin 的完整功能

`EquatableMixin` 提供了与 `Equatable` 基类完全相同的功能：

```dart
class Settings extends ChangeNotifier with EquatableMixin {
  final String theme;
  final bool notificationsEnabled;
  final int fontSize;

  Settings({
    required this.theme,
    required this.notificationsEnabled,
    required this.fontSize,
  });

  @override
  List<Object?> get props => [theme, notificationsEnabled, fontSize];

  @override
  bool get stringify => true;
}
```

### 6.3 扩展现有类

Mixin 特别适合扩展现有类，如 `DateTime`：

```dart
class EquatableDateTime extends DateTime with EquatableMixin {
  EquatableDateTime(
    int year, [
    int month = 1,
    int day = 1,
    int hour = 0,
    int minute = 0,
    int second = 0,
    int millisecond = 0,
    int microsecond = 0,
  ]) : super(year, month, day, hour, minute, second, millisecond, microsecond);

  @override
  List<Object> get props => [
    year, month, day, hour, minute, second, millisecond, microsecond
  ];
}

void main() {
  final dt1 = EquatableDateTime(2024, 1, 15, 10, 30);
  final dt2 = EquatableDateTime(2024, 1, 15, 10, 30);
  final dt3 = EquatableDateTime(2024, 1, 15, 10, 31);

  print(dt1 == dt2);  // true
  print(dt1 == dt3);  // false
}
```

### 6.4 Mixin 的继承

使用 Mixin 的类也可以被继承，子类需要合并父类的 props：

```dart
class EquatableDateTime extends DateTime with EquatableMixin {
  EquatableDateTime(int year) : super(year);

  @override
  List<Object> get props => [year, month, day];
}

class TimestampedDateTime extends EquatableDateTime {
  final int timestamp;

  TimestampedDateTime(int year, this.timestamp) : super(year);

  @override
  List<Object> get props => [...super.props, timestamp];
}

void main() {
  final t1 = TimestampedDateTime(2024, 1705315200);
  final t2 = TimestampedDateTime(2024, 1705315200);

  print(t1 == t2);  // true
}
```

---

## 7. 实用工具函数

Equatable 包还提供了一些独立的工具函数，用于手动比较对象。

### 7.1 equals 函数

`equals` 函数用于深度比较两个对象：

```dart
import 'package:equatable/equatable.dart';

void main() {
  final list1 = [1, 2, 3];
  final list2 = [1, 2, 3];
  final list3 = [1, 2, 4];

  print(equals(list1, list2));  // true
  print(equals(list1, list3));  // false

  // 比较 Map
  final map1 = {'a': 1, 'b': 2};
  final map2 = {'a': 1, 'b': 2};
  print(equals(map1, map2));  // true

  // 比较嵌套集合
  final nested1 = {'data': [1, 2, {'x': 3}]};
  final nested2 = {'data': [1, 2, {'x': 3}]};
  print(equals(nested1, nested2));  // true
}
```

### 7.2 hash 函数

`hash` 函数用于计算一个或多个对象的组合哈希码：

```dart
void main() {
  // 单个对象
  final hash1 = hash('hello');
  print(hash1);  // 计算 'hello' 的哈希码

  // 多个对象
  final combinedHash = hash('Alice', 30, true);
  print(combinedHash);  // 组合哈希码

  // 与 Equatable 内部使用的 mapPropsToHashCode 类似
  final props = ['Alice', 30, true];
  final propsHash = mapPropsToHashCode(props);
  print(propsHash);
}
```

### 7.3 与 Equatable 配合使用的实用函数

```dart
class CustomComparator {
  // 检查两个列表是否包含相同的 Equatable 对象
  static bool listsEqual<T extends Equatable>(List<T> a, List<T> b) {
    if (a.length != b.length) return false;
    for (var i = 0; i < a.length; i++) {
      if (a[i] != b[i]) return false;
    }
    return true;
  }

  // 在列表中查找匹配的 Equatable 对象
  static T? findInList<T extends Equatable>(List<T> list, T target) {
    try {
      return list.firstWhere((item) => item == target);
    } catch (e) {
      return null;
    }
  }

  // 从列表中移除重复的 Equatable 对象
  static List<T> removeDuplicates<T extends Equatable>(List<T> list) {
    final result = <T>[];
    for (var item in list) {
      if (!result.any((existing) => existing == item)) {
        result.add(item);
      }
    }
    return result;
  }
}

class User extends Equatable {
  final String id;
  final String name;
  const User(this.id, this.name);
  @override
  List<Object?> get props => [id, name];
}

void main() {
  final users = [
    User('1', 'Alice'),
    User('2', 'Bob'),
    User('1', 'Alice'),  // 重复
    User('3', 'Charlie'),
  ];

  final unique = CustomComparator.removeDuplicates(users);
  print(unique);  // [User, User, User]（去重后）
}
```

---

## 8. 实战应用示例

### 8.1 Flutter BLoC 状态管理

Equatable 在 BLoC 模式中是必不可少的，用于高效比较状态：

```dart
import 'package:equatable/equatable.dart';

// 状态基类
abstract class CounterState extends Equatable {
  const CounterState();
}

// 初始状态
class CounterInitial extends CounterState {
  const CounterInitial();
  @override
  List<Object?> get props => [];
}

// 加载中状态
class CounterLoading extends CounterState {
  const CounterLoading();
  @override
  List<Object?> get props => [];
}

// 加载成功状态
class CounterLoaded extends CounterState {
  final int count;
  final DateTime updatedAt;

  const CounterLoaded(this.count, this.updatedAt);

  @override
  List<Object?> get props => [count, updatedAt];

  @override
  bool get stringify => true;
}

// 错误状态
class CounterError extends CounterState {
  final String message;

  const CounterError(this.message);

  @override
  List<Object?> get props => [message];
}

// BLoC 中使用
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterInitial()) {
    on<IncrementCounter>((event, emit) {
      if (state is CounterLoaded) {
        final current = state as CounterLoaded;
        emit(CounterLoaded(current.count + 1, DateTime.now()));
      }
    });
  }
}

// 由于使用了 Equatable，BLoC 会自动优化：
// - 如果新状态与当前状态相等，不会触发重建
// - 避免不必要的 UI 刷新
```

### 8.2 数据模型层

```dart
// 用户模型
class UserModel extends Equatable {
  final String id;
  final String email;
  final String displayName;
  final String? avatarUrl;
  final List<String> roles;
  final Map<String, dynamic> metadata;
  final DateTime createdAt;
  final DateTime? lastLoginAt;

  const UserModel({
    required this.id,
    required this.email,
    required this.displayName,
    this.avatarUrl,
    required this.roles,
    required this.metadata,
    required this.createdAt,
    this.lastLoginAt,
  });

  @override
  List<Object?> get props => [
    id, email, displayName, avatarUrl,
    roles, metadata, createdAt, lastLoginAt,
  ];

  // 复制方法（用于不可变更新）
  UserModel copyWith({
    String? id,
    String? email,
    String? displayName,
    String? avatarUrl,
    List<String>? roles,
    Map<String, dynamic>? metadata,
    DateTime? createdAt,
    DateTime? lastLoginAt,
  }) {
    return UserModel(
      id: id ?? this.id,
      email: email ?? this.email,
      displayName: displayName ?? this.displayName,
      avatarUrl: avatarUrl ?? this.avatarUrl,
      roles: roles ?? this.roles,
      metadata: metadata ?? this.metadata,
      createdAt: createdAt ?? this.createdAt,
      lastLoginAt: lastLoginAt ?? this.lastLoginAt,
    );
  }

  // JSON 序列化
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      email: json['email'] as String,
      displayName: json['displayName'] as String,
      avatarUrl: json['avatarUrl'] as String?,
      roles: List<String>.from(json['roles'] as List),
      metadata: json['metadata'] as Map<String, dynamic>,
      createdAt: DateTime.parse(json['createdAt'] as String),
      lastLoginAt: json['lastLoginAt'] != null
          ? DateTime.parse(json['lastLoginAt'] as String)
          : null,
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'email': email,
      'displayName': displayName,
      'avatarUrl': avatarUrl,
      'roles': roles,
      'metadata': metadata,
      'createdAt': createdAt.toIso8601String(),
      'lastLoginAt': lastLoginAt?.toIso8601String(),
    };
  }
}
```

### 8.3 API 响应模型

```dart
// 分页响应
class PaginatedResponse<T> extends Equatable {
  final List<T> data;
  final int total;
  final int page;
  final int perPage;
  final bool hasMore;

  const PaginatedResponse({
    required this.data,
    required this.total,
    required this.page,
    required this.perPage,
    required this.hasMore,
  });

  @override
  List<Object?> get props => [data, total, page, perPage, hasMore];

  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(dynamic) fromJsonT,
  ) {
    return PaginatedResponse(
      data: (json['data'] as List).map(fromJsonT).toList(),
      total: json['total'] as int,
      page: json['page'] as int,
      perPage: json['perPage'] as int,
      hasMore: json['hasMore'] as bool,
    );
  }
}

// API 错误响应
class ApiError extends Equatable {
  final String code;
  final String message;
  final Map<String, dynamic>? details;

  const ApiError({
    required this.code,
    required this.message,
    this.details,
  });

  @override
  List<Object?> get props => [code, message, details];

  factory ApiError.fromJson(Map<String, dynamic> json) {
    return ApiError(
      code: json['code'] as String,
      message: json['message'] as String,
      details: json['details'] as Map<String, dynamic>?,
    );
  }
}
```

### 8.4 缓存键生成

```dart
// 使用 Equatable 生成缓存键
class CacheKey extends Equatable {
  final String endpoint;
  final Map<String, dynamic> params;

  const CacheKey(this.endpoint, this.params);

  @override
  List<Object?> get props => [endpoint, params];

  String get keyString => '$endpoint:${params.toString()}';
}

class ApiCache {
  final _cache = <String, dynamic>{};

  void set<T>(CacheKey key, T value) {
    _cache[key.keyString] = value;
  }

  T? get<T>(CacheKey key) {
    return _cache[key.keyString] as T?;
  }

  bool has(CacheKey key) {
    return _cache.containsKey(key.keyString);
  }
}

void main() {
  final cache = ApiCache();

  final key1 = CacheKey('/users', {'page': 1, 'limit': 10});
  final key2 = CacheKey('/users', {'page': 1, 'limit': 10});

  cache.set(key1, ['user1', 'user2']);

  print(cache.has(key2));  // true（key1 == key2）
  print(cache.get(key2));  // [user1, user2]
}
```

### 8.5 表单状态管理

```dart
class FormState extends Equatable {
  final Map<String, String> values;
  final Map<String, String?> errors;
  final Set<String> touchedFields;
  final bool isSubmitting;
  final bool isValid;

  const FormState({
    this.values = const {},
    this.errors = const {},
    this.touchedFields = const {},
    this.isSubmitting = false,
    this.isValid = false,
  });

  @override
  List<Object?> get props => [
    values, errors, touchedFields, isSubmitting, isValid,
  ];

  FormState copyWith({
    Map<String, String>? values,
    Map<String, String?>? errors,
    Set<String>? touchedFields,
    bool? isSubmitting,
    bool? isValid,
  }) {
    return FormState(
      values: values ?? this.values,
      errors: errors ?? this.errors,
      touchedFields: touchedFields ?? this.touchedFields,
      isSubmitting: isSubmitting ?? this.isSubmitting,
      isValid: isValid ?? this.isValid,
    );
  }

  String? getFieldError(String field) {
    if (!touchedFields.contains(field)) return null;
    return errors[field];
  }

  String getFieldValue(String field) => values[field] ?? '';
}
```

---

## 9. 最佳实践与注意事项

### 9.1 使用 final 字段

Equatable 设计用于**不可变对象**，所有字段都应该是 `final`：

```dart
// ✅ 正确
class User extends Equatable {
  final String name;
  final int age;
  const User(this.name, this.age);
  @override
  List<Object?> get props => [name, age];
}

// ❌ 错误 - 可变字段可能导致哈希表问题
class User extends Equatable {
  String name;  // 不是 final！
  int age;      // 不是 final！
  User(this.name, this.age);
  @override
  List<Object?> get props => [name, age];
}
```

### 9.2 包含所有相关字段

确保 `props` 包含所有影响相等性判断的字段：

```dart
// ✅ 正确 - 所有字段都参与比较
class Product extends Equatable {
  final String id;
  final String name;
  final double price;
  const Product(this.id, this.name, this.price);
  @override
  List<Object?> get props => [id, name, price];
}

// ❌ 错误 - price 被遗漏
class Product extends Equatable {
  final String id;
  final String name;
  final double price;
  const Product(this.id, this.name, this.price);
  @override
  List<Object?> get props => [id, name];  // price 被遗漏！
}
```

### 9.3 避免在 props 中使用计算属性

不要在 `props` 中使用计算属性，应该在构造函数中计算并存储：

```dart
// ❌ 错误 - 使用计算属性
class Rectangle extends Equatable {
  final double width;
  final double height;
  Rectangle(this.width, this.height);
  double get area => width * height;  // 计算属性
  @override
  List<Object?> get props => [width, height, area];  // 不要这样做！
}

// ✅ 正确 - 存储计算结果
class Rectangle extends Equatable {
  final double width;
  final double height;
  final double area;
  Rectangle(this.width, this.height) : area = width * height;
  @override
  List<Object?> get props => [width, height, area];
}
```

### 9.4 集合类型的注意事项

确保集合字段也是不可变的：

```dart
// ✅ 正确 - 使用不可变集合
class Config extends Equatable {
  final List<String> features;
  final Map<String, dynamic> settings;
  const Config({
    required this.features,
    required this.settings,
  });
  @override
  List<Object?> get props => [features, settings];
}

// 创建时使用 const 或 unmodifiable
const config = Config(
  features: const ['feature1', 'feature2'],
  settings: const {'key': 'value'},
);

// ❌ 错误 - 可变集合可能导致问题
class Config extends Equatable {
  List<String> features;  // 不是 final，且可能可变
  Map<String, dynamic> settings;
  Config(this.features, this.settings);
  @override
  List<Object?> get props => [features, settings];
}
```

### 9.5 继承时的最佳实践

```dart
// 基类
abstract class Entity extends Equatable {
  final String id;
  final DateTime createdAt;
  const Entity(this.id, this.createdAt);

  @override
  List<Object?> get props => [id, createdAt];
}

// 子类
class User extends Entity {
  final String name;
  final String email;

  const User(
    String id,
    DateTime createdAt,
    this.name,
    this.email,
  ) : super(id, createdAt);

  @override
  List<Object?> get props => [...super.props, name, email];
}

// 孙类
class Admin extends User {
  final List<String> permissions;

  const Admin(
    String id,
    DateTime createdAt,
    String name,
    String email,
    this.permissions,
  ) : super(id, createdAt, name, email);

  @override
  List<Object?> get props => [...super.props, permissions];
}
```

### 9.6 性能优化建议

```dart
// 对于频繁比较的类，缓存 props
class LargeModel extends Equatable {
  final String field1;
  final String field2;
  // ... 很多字段
  final String field50;

  const LargeModel(
    this.field1,
    this.field2,
    // ...
    this.field50,
  );

  // 缓存 props 避免每次创建新列表
  @override
  List<Object?> get props => [
    field1, field2, /* ... */ field50,
  ];
}

// 如果只有部分字段经常变化，考虑分离
class User extends Equatable {
  final String id;           // 不变
  final String name;         // 偶尔变
  final DateTime lastSeen;   // 经常变

  const User(this.id, this.name, this.lastSeen);

  // 方案1：全部比较
  @override
  List<Object?> get props => [id, name, lastSeen];

  // 方案2：只比较核心字段（如果 lastSeen 不影响业务相等性）
  // @override
  // List<Object?> get props => [id, name];
}
```

### 9.7 常见错误与解决方案

| 错误                | 原因               | 解决方案                           |
| ------------------- | ------------------ | ---------------------------------- |
| `==` 始终返回 false | 忘记重写 props     | 确保重写 `List<Object?> get props` |
| 哈希表查找失败      | hashCode 不一致    | 确保所有字段都是 final             |
| 集合比较失败        | 使用 identity 比较 | Equatable 自动深度比较集合         |
| 子类比较失败        | 没有合并父类 props | 使用 `[...super.props, ...]`       |
| toString 输出不完整 | stringify 未启用   | 设置 `bool get stringify => true`  |

---

## 10. 附录：速查表

### 10.1 API 对照表

| 组件              | 类型   | 用途                      | 使用场景           |
| ----------------- | ------ | ------------------------- | ------------------ |
| `Equatable`       | 抽象类 | 基类，提供 == 和 hashCode | 类没有继承其他类时 |
| `EquatableMixin`  | Mixin  | 提供 == 和 hashCode       | 类已继承其他类时   |
| `props`           | 属性   | 定义参与相等性比较的字段  | 必须重写           |
| `stringify`       | 属性   | 启用增强的 toString       | 可选，默认 false   |
| `EquatableConfig` | 配置类 | 全局配置 stringify        | 应用启动时配置     |
| `equals()`        | 函数   | 深度比较两个对象          | 手动比较时使用     |
| `hash()`          | 函数   | 计算组合哈希码            | 手动计算哈希时使用 |

### 10.2 代码模板

**基础模板**：

```dart
class ClassName extends Equatable {
  final Type field1;
  final Type field2;

  const ClassName(this.field1, this.field2);

  @override
  List<Object?> get props => [field1, field2];
}
```

**完整模板**：

```dart
class ClassName extends Equatable {
  final String id;
  final String name;
  final List<String> tags;
  final DateTime createdAt;

  const ClassName({
    required this.id,
    required this.name,
    required this.tags,
    required this.createdAt,
  });

  @override
  List<Object?> get props => [id, name, tags, createdAt];

  @override
  bool get stringify => true;

  ClassName copyWith({
    String? id,
    String? name,
    List<String>? tags,
    DateTime? createdAt,
  }) {
    return ClassName(
      id: id ?? this.id,
      name: name ?? this.name,
      tags: tags ?? this.tags,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}
```

**Mixin 模板**：

```dart
class ClassName extends BaseClass with EquatableMixin {
  final Type field1;
  final Type field2;

  ClassName(this.field1, this.field2);

  @override
  List<Object?> get props => [field1, field2];
}
```

### 10.3 与手动实现对比

| 特性         | 手动实现         | Equatable                      |
| ------------ | ---------------- | ------------------------------ |
| 代码量       | 15-50+ 行        | 3-5 行                         |
| 维护成本     | 高（需同步修改） | 低（只需改 props）             |
| 集合深度比较 | 需手动实现       | 自动支持                       |
| 嵌套对象比较 | 需递归实现       | 自动支持（如果也是 Equatable） |
| 错误风险     | 高               | 低                             |
| 性能         | 手动优化         | 已优化                         |

---

## 总结

Equatable 是 Dart/Flutter 开发中不可或缺的工具，它极大地简化了对象相等性的实现：

1. **简化代码**：只需重写 `props`，无需编写繁琐的 `==` 和 `hashCode`

2. **深度比较**：自动支持 List、Map、Set 等集合类型的深度比较

3. **嵌套支持**：嵌套的 Equatable 对象也能正确比较

4. **灵活配置**：支持 `const` 构造函数、可空属性、Mixin 模式

5. **调试友好**：可选的 `stringify` 功能让日志输出更清晰

6. **状态管理必备**：与 BLoC、Provider 等状态管理方案完美配合

通过本文的详细讲解，你应该能够熟练地在项目中使用 Equatable，编写出更简洁、更可靠的 Dart/Flutter 代码。
