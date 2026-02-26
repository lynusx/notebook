# Hive CE 库详解

## 目录

1. [概述](#概述)
2. [核心概念](#核心概念)
3. [安装与配置](#安装与配置)
4. [Hive 初始化](#hive-初始化)
5. [Box 详解](#box-详解)
6. [类型适配器 (TypeAdapter)](#类型适配器-typeadapter)
7. [HiveObject 与 HiveField](#hiveobject-与-hivefield)
8. [自动适配器生成](#自动适配器生成)
9. [LazyBox 延迟加载](#lazybox-延迟加载)
10. [BoxCollection 集合](#boxcollection-集合)
11. [IsolatedHive 多线程支持](#isolatedhive-多线程支持)
12. [数据加密](#数据加密)
13. [实战应用示例](#实战应用示例)
14. [附录](#附录)

---

## 概述

Hive CE (Community Edition) 是 Hive v2 的精神续作，是一个轻量级、高性能的 NoSQL 键值数据库，完全使用 Dart 编写。它受到 Bitcask 的启发，专为 Dart 和 Flutter 应用设计。

### 主要特性

| 特性              | 描述                                    |
| ----------------- | --------------------------------------- |
| 🚀 **跨平台**     | 支持移动端、桌面端、浏览器（包括 WASM） |
| ⚡ **高性能**     | 比 Hive v4 快数倍，文件更小             |
| ❤️ **简单易用**   | 类似 Map 的直观 API                     |
| 🔒 **内置加密**   | 支持 AES256 加密                        |
| 🎈 **无原生依赖** | 纯 Dart 实现，无需平台特定代码          |
| 🔋 **开箱即用**   | 内置 Duration、Set 等类型的适配器       |

### Hive CE vs Hive v2 vs Hive v4

| 特性             | Hive CE (2.19+)     | Hive v2     | Hive v4     |
| ---------------- | ------------------- | ----------- | ----------- |
| 维护状态         | ✅ 活跃维护         | ⚠️ 停止维护 | ⚠️ 非社区版 |
| Flutter Web WASM | ✅ 支持             | ❌ 不支持   | ✅ 支持     |
| Isolate 支持     | ✅ IsolatedHive     | ❌ 不支持   | ✅ 支持     |
| 自动适配器生成   | ✅ GenerateAdapters | ❌ 手动     | ✅ 部分支持 |
| 最大 Type ID     | 65439               | 223         | 223         |
| Freezed 支持     | ✅ 完整支持         | ⚠️ 有限     | ⚠️ 有限     |
| Duration 适配器  | ✅ 内置             | ❌ 需自定义 | ❌ 需自定义 |
| Set 支持         | ✅ 内置             | ❌ 不支持   | ❌ 不支持   |
| DevTools 扩展    | ✅ 内置             | ❌ 无       | ❌ 无       |

### 性能对比

| 操作数    | Hive CE 时间 | Hive CE 大小 | Hive v4 时间 | Hive v4 大小 |
| --------- | ------------ | ------------ | ------------ | ------------ |
| 1,000     | 0.02s        | 0.11 MB      | 0.06s        | 1.00 MB      |
| 10,000    | 0.13s        | 1.10 MB      | 0.64s        | 5.00 MB      |
| 100,000   | 1.40s        | 10.97 MB     | 7.26s        | 30.00 MB     |
| 1,000,000 | 19.94s       | 109.67 MB    | 84.87s       | 290.00 MB    |

---

## 核心概念

### 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                        Hive CE                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    HiveInterface                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐ │  │
│  │  │    Box      │  │   LazyBox   │  │ BoxCollection │ │  │
│  │  │  (内存)      │  │  (延迟加载)  │  │   (集合)       │ │  │
│  │  └──────┬──────┘  └──────┬──────┘  └───────┬───────┘ │  │
│  │         │                │                  │         │  │
│  │         └────────────────┴──────────────────┘         │  │
│  │                          │                            │  │
│  │              ┌───────────┴───────────┐                │  │
│  │              ▼                       ▼                │  │
│  │  ┌─────────────────────┐  ┌──────────────────────┐   │  │
│  │  │    TypeAdapter      │  │    HiveCipher        │   │  │
│  │  │  (序列化/反序列化)    │  │   (AES256 加密)       │   │  │
│  │  └─────────────────────┘  └──────────────────────┘   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件             | 用途                          |
| ---------------- | ----------------------------- |
| `Hive`           | 主入口，管理所有 Box 和适配器 |
| `Box<E>`         | 数据容器，类似 Map            |
| `LazyBox<E>`     | 延迟加载的 Box，适合大数据    |
| `BoxCollection`  | 多个 Box 的集合               |
| `TypeAdapter<T>` | 自定义类型序列化器            |
| `HiveObject`     | 可持久化的对象基类            |
| `HiveField`      | 标记可持久化字段              |
| `HiveCipher`     | 加密接口                      |

---

## 安装与配置

### 添加依赖

```yaml
# pubspec.yaml
dependencies:
  hive_ce: ^2.19.3
  hive_ce_flutter: ^2.2.0 # Flutter 项目添加

dev_dependencies:
  hive_ce_generator: ^1.8.0 # 代码生成器
  build_runner: ^2.4.13 # 构建工具
```

### Dart SDK 要求

```yaml
environment:
  sdk: ^3.0.0 # Hive CE 需要 Dart 3+
```

### 安装命令

```bash
# Flutter 项目
flutter pub add hive_ce hive_ce_flutter
flutter pub add dev:hive_ce_generator dev:build_runner

# Dart 项目
dart pub add hive_ce
dart pub add dev:hive_ce_generator dev:build_runner
```

### 项目结构建议

```
lib/
├── models/
│   ├── user.dart
│   ├── user.g.dart          # 生成的适配器
│   └── adapters.dart        # 适配器注册
├── services/
│   └── hive_service.dart    # Hive 封装服务
├── main.dart
test/
└── ...
```

---

## Hive 初始化

### 基础初始化

```dart
import 'package:hive_ce/hive_ce.dart';

void main() async {
  // 初始化 Hive（非 Flutter 项目）
  Hive.init('path/to/hive/directory');

  // 打开一个 Box
  final box = await Hive.openBox('myBox');

  // 使用 Box
  box.put('name', 'David');
  final name = box.get('name');
  print('Name: $name');
}
```

### Flutter 项目初始化

```dart
import 'package:hive_ce_flutter/hive_ce_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 初始化 Hive for Flutter
  await Hive.initFlutter();

  // 注册适配器（如果使用代码生成）
  registerAdapters();

  // 打开 Box
  await Hive.openBox('settings');
  await Hive.openBox<User>('users');

  runApp(MyApp());
}
```

### 自定义存储路径

```dart
import 'package:path_provider/path_provider.dart';

Future<void> initHive() async {
  // 获取应用文档目录
  final appDocDir = await getApplicationDocumentsDirectory();

  // 初始化 Hive 到指定目录
  Hive.init('${appDocDir.path}/hive_data');

  // 或使用 Flutter 便捷方法
  await Hive.initFlutter(subDir: 'my_app_data');
}
```

### 关闭 Hive

```dart
// 关闭单个 Box
await box.close();

// 关闭所有 Box
await Hive.close();

// 删除 Box 文件
await Hive.deleteBoxFromDisk('myBox');

// 删除所有 Box
await Hive.deleteFromDisk();
```

---

## Box 详解

`Box<E>` 是 Hive 的核心数据容器，类似于 Map，但支持异步操作和类型安全。

### 打开 Box

```dart
// 打开无类型 Box（存储 dynamic）
final box = await Hive.openBox('settings');

// 打开类型化 Box
final userBox = await Hive.openBox<User>('users');

// 打开加密 Box
final encryptedBox = await Hive.openBox(
  'secrets',
  encryptionCipher: HiveAesCipher(key),
);

// 打开时设置压缩策略
final box = await Hive.openBox(
  'data',
  compactionStrategy: (entries, deletedEntries) {
    return deletedEntries > 50;
  },
);
```

### 基本 CRUD 操作

```dart
void boxOperations(Box box) {
  // ===== 创建/更新 =====

  // 存储单个值
  box.put('key1', 'value1');

  // 使用数字键（自增）
  final key = box.add('value');  // 返回自动分配的 key

  // 批量存储
  box.putAll({
    'key1': 'value1',
    'key2': 'value2',
    'key3': 'value3',
  });

  // 在指定索引处存储（仅用于数字键）
  box.putAt(0, 'newValue');

  // ===== 读取 =====

  // 获取值
  final value = box.get('key1');

  // 获取值，如果不存在返回默认值
  final value2 = box.get('key2', defaultValue: 'default');

  // 通过索引获取（仅用于数字键）
  final firstValue = box.getAt(0);

  // 获取键
  final keyAtIndex = box.keyAt(0);

  // ===== 删除 =====

  // 删除单个键
  box.delete('key1');

  // 通过索引删除
  box.deleteAt(0);

  // 批量删除
  box.deleteAll(['key1', 'key2', 'key3']);

  // 清空所有数据
  box.clear();

  // ===== 查询 =====

  // 检查键是否存在
  final exists = box.containsKey('key1');

  // 获取所有键
  final keys = box.keys;

  // 获取所有值
  final values = box.values;

  // 转换为 Map
  final map = box.toMap();

  // 获取长度
  final length = box.length;

  // 是否为空
  final isEmpty = box.isEmpty;
  final isNotEmpty = box.isNotEmpty;
}
```

### 监听变化

```dart
void watchBox(Box box) {
  // 监听所有变化
  box.watch().listen((event) {
    print('Key: ${event.key}');
    print('Value: ${event.value}');
    print('Deleted: ${event.deleted}');
  });

  // 监听特定键
  box.watch(key: 'userId').listen((event) {
    print('User ID changed: ${event.value}');
  });
}
```

### 数据持久化

```dart
void persistence(Box box) async {
  // 立即写入磁盘（通常不需要手动调用）
  await box.flush();

  // 压缩 Box（删除已标记为删除的条目）
  await box.compact();

  // 关闭 Box
  await box.close();

  // 从磁盘删除 Box
  await box.deleteFromDisk();
}
```

### Box 配置选项

```dart
final box = await Hive.openBox(
  'myBox',

  // 键比较函数
  keyComparator: (a, b) => a.toString().compareTo(b.toString()),

  // 压缩策略
  compactionStrategy: (entries, deletedEntries) {
    // 当删除的条目超过 50 时触发压缩
    return deletedEntries > 50;
  },

  // 加密
  encryptionCipher: HiveAesCipher(encryptionKey),

  // 崩溃恢复
  crashRecovery: true,

  // 存储后端偏好
  backendPreference: HiveStorageBackendPreference.native,
);
```

---

## 类型适配器 (TypeAdapter)

TypeAdapter 用于将自定义对象序列化为二进制格式存储，以及从二进制格式反序列化。

### 内置适配器

Hive CE 内置了以下类型的适配器：

| 类型       | 说明                    |
| ---------- | ----------------------- |
| `null`     | 空值                    |
| `int`      | 整数                    |
| `double`   | 浮点数                  |
| `bool`     | 布尔值                  |
| `String`   | 字符串                  |
| `List`     | 列表                    |
| `Map`      | 映射                    |
| `Set`      | 集合 (Hive CE 新增)     |
| `DateTime` | 日期时间                |
| `Duration` | 持续时间 (Hive CE 新增) |

### 手动编写适配器

```dart
import 'package:hive_ce/hive_ce.dart';

// 定义模型类
@HiveType(typeId: 1)
class User {
  @HiveField(0)
  final String id;

  @HiveField(1)
  final String name;

  @HiveField(2)
  final int age;

  @HiveField(3)
  final DateTime createdAt;

  User({
    required this.id,
    required this.name,
    required this.age,
    required this.createdAt,
  });
}

// 手动编写适配器
class UserAdapter extends TypeAdapter<User> {
  @override
  final int typeId = 1;

  @override
  User read(BinaryReader reader) {
    return User(
      id: reader.readString(),
      name: reader.readString(),
      age: reader.readInt(),
      createdAt: DateTime.parse(reader.readString()),
    );
  }

  @override
  void write(BinaryWriter writer, User obj) {
    writer.writeString(obj.id);
    writer.writeString(obj.name);
    writer.writeInt(obj.age);
    writer.writeString(obj.createdAt.toIso8601String());
  }
}

// 注册适配器
void registerAdapters() {
  Hive.registerAdapter(UserAdapter());
}
```

### BinaryReader 方法

```dart
// 读取基本类型
final intValue = reader.readInt();
final doubleValue = reader.readDouble();
final boolValue = reader.readBool();
final stringValue = reader.readString();

// 读取列表
final list = reader.readList();

// 读取 Map
final map = reader.readMap();

// 读取指定类型的列表
final intList = reader.readIntList();
final stringList = reader.readStringList();

// 读取字节
final bytes = reader.readByteList();

// 条件读取
if (reader.readBool()) {
  final value = reader.readString();
}
```

### BinaryWriter 方法

```dart
// 写入基本类型
writer.writeInt(42);
writer.writeDouble(3.14);
writer.writeBool(true);
writer.writeString('Hello');

// 写入列表
writer.writeList([1, 2, 3]);

// 写入 Map
writer.writeMap({'key': 'value'});

// 写入指定类型的列表
writer.writeIntList([1, 2, 3]);
writer.writeStringList(['a', 'b', 'c']);

// 写入字节
writer.writeByteList([0x01, 0x02, 0x03]);
```

---

## HiveObject 与 HiveField

### HiveObject

`HiveObject` 是一个便捷的混入类，为对象提供 `save()` 和 `delete()` 方法。

```dart
import 'package:hive_ce/hive_ce.dart';

@HiveType(typeId: 2)
class Task extends HiveObject {
  @HiveField(0)
  String title;

  @HiveField(1)
  bool isCompleted;

  @HiveField(2)
  DateTime? dueDate;

  Task({
    required this.title,
    this.isCompleted = false,
    this.dueDate,
  });
}

// 使用 HiveObject
void useHiveObject() async {
  final box = await Hive.openBox<Task>('tasks');

  // 创建对象
  final task = Task(title: 'Buy milk', dueDate: DateTime.now());

  // 保存到 Box（需要设置 box 属性或传递 box 参数）
  await box.add(task);  // 自动分配 key

  // 或者
  task.box = box;
  await task.save();  // 使用已设置的 box

  // 修改并保存
  task.isCompleted = true;
  await task.save();

  // 删除
  await task.delete();
}
```

### HiveField 属性

```dart
@HiveType(typeId: 3)
class Product {
  @HiveField(0)
  final String id;

  @HiveField(1)
  final String name;

  @HiveField(2, defaultValue: 0.0)  // 设置默认值
  final double price;

  @HiveField(3)
  final List<String> tags;

  // 嵌套对象
  @HiveField(4)
  final Category? category;

  Product({
    required this.id,
    required this.name,
    this.price = 0.0,
    this.tags = const [],
    this.category,
  });
}
```

### 字段迁移

```dart
@HiveType(typeId: 4)
class UserV2 {
  @HiveField(0)
  final String id;

  @HiveField(1)
  final String name;

  // 新字段，使用默认值处理旧数据
  @HiveField(2, defaultValue: 'user@example.com')
  final String email;

  // 已删除字段的占位（不要重复使用索引）
  // @HiveField(3) - 已删除，保留索引

  // 新字段使用新索引
  @HiveField(4)
  final DateTime? lastLogin;
}
```

---

## 自动适配器生成

Hive CE 提供了 `GenerateAdapters` 注解，可以自动生成所有适配器，无需手动添加 `@HiveType` 和 `@HiveField`。

### 配置生成

```dart
// lib/models/adapters.dart
import 'package:hive_ce/hive_ce.dart';
import 'user.dart';
import 'product.dart';
import 'order.dart';

// 生成所有适配器
@GenerateAdapters([
  AdapterSpec<User>(),
  AdapterSpec<Product>(),
  AdapterSpec<Order>(),
])
part 'adapters.g.dart';
```

### 运行代码生成

```bash
# 生成适配器代码
dart run build_runner build

# 监视文件变化并自动生成
dart run build_runner watch
```

### 注册生成的适配器

```dart
// lib/main.dart
import 'models/adapters.dart';

void main() async {
  await Hive.initFlutter();

  // 注册所有生成的适配器
  registerAdapters();

  runApp(MyApp());
}
```

### 自定义适配器配置

```dart
@GenerateAdapters([
  // 基本配置
  AdapterSpec<User>(),

  // 指定 Type ID
  AdapterSpec<Product>(typeId: 10),

  // 保留特定 Type ID（防止冲突）
  AdapterSpec<Order>(
    typeId: 20,
    reservedTypeIds: [21, 22, 23],  // 预留相邻的 ID
  ),
])
part 'adapters.g.dart';
```

### 为外部包生成适配器

```dart
// 为第三方包中的类生成适配器
@GenerateAdapters([
  AdapterSpec<external_package.SomeClass>(),
])
part 'adapters.g.dart';
```

---

## LazyBox 延迟加载

`LazyBox` 与 `Box` 类似，但不会将所有数据加载到内存，适合存储大量数据。

### 使用场景

- 大量数据（数千条以上）
- 内存敏感的应用
- 不需要频繁访问所有数据

### 基本用法

```dart
void lazyBoxExample() async {
  // 打开 LazyBox
  final lazyBox = await Hive.openLazyBox<LargeData>('largeData');

  // 存储数据（与 Box 相同）
  await lazyBox.put('key1', LargeData(...));
  await lazyBox.add(LargeData(...));

  // 读取数据（异步）
  final data = await lazyBox.get('key1');

  // 批量读取
  final futures = ['key1', 'key2', 'key3']
      .map((key) => lazyBox.get(key));
  final results = await Future.wait(futures);

  // 遍历（按需加载）
  for (final key in lazyBox.keys) {
    final value = await lazyBox.get(key);
    print(value);
  }

  // 关闭
  await lazyBox.close();
}
```

### LazyBox vs Box

| 特性     | Box            | LazyBox         |
| -------- | -------------- | --------------- |
| 内存占用 | 高（全部加载） | 低（按需加载）  |
| 读取速度 | 快（内存访问） | 较慢（磁盘 IO） |
| 写入速度 | 快             | 快              |
| 遍历性能 | 快             | 慢              |
| 适用场景 | 小数据量       | 大数据量        |

---

## BoxCollection 集合

`BoxCollection` 用于管理多个相关的 Box，支持事务操作。

### 基本用法

```dart
void boxCollectionExample() async {
  // 创建集合
  final collection = await BoxCollection.open(
    'myCollection',  // 集合名称
    {'users', 'orders', 'products'},  // Box 名称集合
    path: './hive_data',  // 存储路径
  );

  // 打开集合中的 Box
  final userBox = await collection.openBox<Map>('users');
  final orderBox = await collection.openBox<Map>('orders');

  // 事务操作
  await collection.transaction(() async {
    await userBox.put('user1', {'name': 'Alice'});
    await orderBox.put('order1', {'userId': 'user1', 'total': 100});
  });

  // 读取
  final user = await userBox.get('user1');
  final order = await orderBox.get('order1');

  // 关闭集合
  await collection.close();
}
```

### 事务处理

```dart
void transactionExample(BoxCollection collection) async {
  // 开始事务
  await collection.transaction(() async {
    final userBox = await collection.openBox('users');
    final accountBox = await collection.openBox('accounts');

    // 转账操作（原子性）
    await userBox.put('user1', {'balance': 900});
    await userBox.put('user2', {'balance': 1100});

    await accountBox.put('transaction1', {
      'from': 'user1',
      'to': 'user2',
      'amount': 100,
    });
  });
  // 事务中的所有操作要么全部成功，要么全部失败
}
```

---

## IsolatedHive 多线程支持

`IsolatedHive` 允许在单独的 Isolate 中运行 Hive 操作，避免阻塞 UI 线程。

### 基本用法

```dart
import 'package:hive_ce/hive_ce.dart';

void isolatedHiveExample() async {
  // 初始化 IsolatedHive
  final isolatedHive = IsolatedHive();
  await isolatedHive.init('path/to/hive');

  // 打开 Box
  final box = await isolatedHive.openBox<Map>('data');

  // 所有操作都是异步的
  await box.put('key1', {'value': 123});

  final value = await box.get('key1');
  print(value);  // {value: 123}

  // 关闭
  await isolatedHive.close();
}
```

### 性能对比

| 操作数    | Hive 时间 | IsolatedHive 时间 |
| --------- | --------- | ----------------- |
| 1,000     | 0.02s     | 0.03s             |
| 10,000    | 0.13s     | 0.25s             |
| 100,000   | 1.40s     | 2.64s             |
| 1,000,000 | 19.94s    | 41.50s            |

### 使用场景

- 大量数据操作
- 后台数据处理
- 避免 UI 卡顿

---

## 数据加密

Hive CE 支持 AES256 CBC 加密，保护敏感数据。

### 生成加密密钥

```dart
import 'dart:convert';
import 'dart:math';
import 'dart:typed_data';

// 生成随机密钥
Uint8List generateEncryptionKey() {
  final random = Random.secure();
  return Uint8List.fromList(
    List.generate(32, (_) => random.nextInt(256)),
  );
}

// 将密钥保存到安全存储（如 Keychain/Keystore）
Future<void> saveKeySecurely(Uint8List key) async {
  // 使用 flutter_secure_storage 或其他安全存储
  final storage = FlutterSecureStorage();
  await storage.write(
    key: 'hive_encryption_key',
    value: base64Encode(key),
  );
}

// 从安全存储读取密钥
Future<Uint8List?> getKeyFromSecureStorage() async {
  final storage = FlutterSecureStorage();
  final keyString = await storage.read(key: 'hive_encryption_key');
  if (keyString != null) {
    return base64Decode(keyString);
  }
  return null;
}
```

### 打开加密 Box

```dart
void encryptedBoxExample() async {
  // 获取或生成密钥
  Uint8List? encryptionKey = await getKeyFromSecureStorage();
  if (encryptionKey == null) {
    encryptionKey = generateEncryptionKey();
    await saveKeySecurely(encryptionKey);
  }

  // 创建加密 Cipher
  final cipher = HiveAesCipher(encryptionKey);

  // 打开加密 Box
  final encryptedBox = await Hive.openBox(
    'secrets',
    encryptionCipher: cipher,
  );

  // 使用与普通 Box 相同
  await encryptedBox.put('password', 'mySecretPassword');
  await encryptedBox.put('apiKey', 'sk-1234567890');

  final password = encryptedBox.get('password');
  print(password);  // mySecretPassword

  await encryptedBox.close();
}
```

### 自定义加密

```dart
// 实现自定义 Cipher
class CustomCipher implements HiveCipher {
  @override
  int calculateEncryptedLength(int inputLength) {
    // 返回加密后的长度
    return inputLength + 16;  // 示例：添加 16 字节 IV
  }

  @override
  int decrypt(
    Uint8List input,
    int inputOffset,
    int inputLength,
    Uint8List output,
    int outputOffset,
    Uint8List key,
  ) {
    // 实现解密逻辑
    // ...
    return outputLength;
  }

  @override
  int encrypt(
    Uint8List input,
    int inputOffset,
    int inputLength,
    Uint8List output,
    int outputOffset,
    Uint8List key,
  ) {
    // 实现加密逻辑
    // ...
    return outputLength;
  }
}
```

---

## 实战应用示例

### 示例 1：完整的用户管理

```dart
// models/user.dart
import 'package:hive_ce/hive_ce.dart';

part 'user.g.dart';  // 生成的代码

@HiveType(typeId: 1)
class User extends HiveObject {
  @HiveField(0)
  final String id;

  @HiveField(1)
  String name;

  @HiveField(2)
  String email;

  @HiveField(3)
  DateTime createdAt;

  @HiveField(4, defaultValue: false)
  bool isActive;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
    this.isActive = false,
  });
}

// services/user_service.dart
import 'package:hive_ce/hive_ce.dart';
import '../models/user.dart';

class UserService {
  static const String _boxName = 'users';
  Box<User>? _box;

  Future<Box<User>> get _userBox async {
    _box ??= await Hive.openBox<User>(_boxName);
    return _box!;
  }

  // 创建用户
  Future<User> createUser({
    required String name,
    required String email,
  }) async {
    final box = await _userBox;
    final user = User(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      name: name,
      email: email,
      createdAt: DateTime.now(),
    );
    await box.put(user.id, user);
    return user;
  }

  // 获取用户
  Future<User?> getUser(String id) async {
    final box = await _userBox;
    return box.get(id);
  }

  // 获取所有用户
  Future<List<User>> getAllUsers() async {
    final box = await _userBox;
    return box.values.toList();
  }

  // 更新用户
  Future<void> updateUser(User user) async {
    final box = await _userBox;
    await box.put(user.id, user);
  }

  // 删除用户
  Future<void> deleteUser(String id) async {
    final box = await _userBox;
    await box.delete(id);
  }

  // 搜索用户
  Future<List<User>> searchUsers(String query) async {
    final box = await _userBox;
    return box.values
        .where((user) =>
            user.name.toLowerCase().contains(query.toLowerCase()) ||
            user.email.toLowerCase().contains(query.toLowerCase()))
        .toList();
  }

  // 监听用户变化
  Stream<BoxEvent> watchUser(String id) async* {
    final box = await _userBox;
    yield* box.watch(key: id);
  }

  // 关闭
  Future<void> close() async {
    await _box?.close();
    _box = null;
  }
}
```

### 示例 2：设置管理

```dart
// services/settings_service.dart
import 'package:hive_ce/hive_ce.dart';

class SettingsService {
  static const String _boxName = 'settings';
  late Box<dynamic> _box;

  Future<void> init() async {
    _box = await Hive.openBox(_boxName);
  }

  // 通用获取方法
  T? get<T>(String key, {T? defaultValue}) {
    return _box.get(key, defaultValue: defaultValue) as T?;
  }

  // 通用设置方法
  Future<void> set<T>(String key, T value) async {
    await _box.put(key, value);
  }

  // 主题设置
  String get theme => get<String>('theme', defaultValue: 'system')!;
  Future<void> setTheme(String theme) => set('theme', theme);

  // 语言设置
  String get language => get<String>('language', defaultValue: 'zh')!;
  Future<void> setLanguage(String language) => set('language', language);

  // 通知设置
  bool get notificationsEnabled =>
      get<bool>('notificationsEnabled', defaultValue: true)!;
  Future<void> setNotificationsEnabled(bool enabled) =>
      set('notificationsEnabled', enabled);

  // 字体大小
  double get fontSize => get<double>('fontSize', defaultValue: 14.0)!;
  Future<void> setFontSize(double size) => set('fontSize', size);

  // 清除所有设置
  Future<void> clear() async {
    await _box.clear();
  }

  // 监听设置变化
  Stream<BoxEvent> watch<T>(String key) {
    return _box.watch(key: key);
  }
}
```

### 示例 3：缓存管理

```dart
// services/cache_service.dart
import 'package:hive_ce/hive_ce.dart';

class CacheEntry {
  final dynamic data;
  final DateTime timestamp;
  final Duration ttl;

  CacheEntry({
    required this.data,
    required this.timestamp,
    required this.ttl,
  });

  bool get isExpired =>
      DateTime.now().difference(timestamp) > ttl;
}

class CacheService {
  static const String _boxName = 'cache';
  late LazyBox<CacheEntry> _box;

  Future<void> init() async {
    _box = await Hive.openLazyBox<CacheEntry>(_boxName);
  }

  // 获取缓存
  Future<T?> get<T>(String key) async {
    final entry = await _box.get(key);
    if (entry == null) return null;

    if (entry.isExpired) {
      await _box.delete(key);
      return null;
    }

    return entry.data as T;
  }

  // 设置缓存
  Future<void> set<T>(
    String key,
    T data, {
    Duration ttl = const Duration(minutes: 5),
  }) async {
    final entry = CacheEntry(
      data: data,
      timestamp: DateTime.now(),
      ttl: ttl,
    );
    await _box.put(key, entry);
  }

  // 清除过期缓存
  Future<void> clearExpired() async {
    final keys = <dynamic>[];
    for (final key in _box.keys) {
      final entry = await _box.get(key);
      if (entry?.isExpired ?? true) {
        keys.add(key);
      }
    }
    await _box.deleteAll(keys);
  }

  // 清除所有缓存
  Future<void> clear() async {
    await _box.clear();
  }
}
```

### 示例 4：购物车实现

```dart
// models/cart_item.dart
import 'package:hive_ce/hive_ce.dart';

part 'cart_item.g.dart';

@HiveType(typeId: 2)
class CartItem extends HiveObject {
  @HiveField(0)
  final String productId;

  @HiveField(1)
  final String productName;

  @HiveField(2)
  final double price;

  @HiveField(3)
  int quantity;

  @HiveField(4)
  final String? imageUrl;

  CartItem({
    required this.productId,
    required this.productName,
    required this.price,
    this.quantity = 1,
    this.imageUrl,
  });

  double get total => price * quantity;
}

// services/cart_service.dart
import 'package:hive_ce/hive_ce.dart';
import '../models/cart_item.dart';

class CartService {
  static const String _boxName = 'cart';
  late Box<CartItem> _box;

  Future<void> init() async {
    _box = await Hive.openBox<CartItem>(_boxName);
  }

  // 获取所有商品
  List<CartItem> get items => _box.values.toList();

  // 获取商品数量
  int get itemCount => _box.length;

  // 获取总价
  double get totalPrice =>
      _box.values.fold(0, (sum, item) => sum + item.total);

  // 添加商品
  Future<void> addItem(CartItem item) async {
    final existingItem = _box.get(item.productId);
    if (existingItem != null) {
      existingItem.quantity += item.quantity;
      await existingItem.save();
    } else {
      await _box.put(item.productId, item);
    }
  }

  // 更新数量
  Future<void> updateQuantity(String productId, int quantity) async {
    final item = _box.get(productId);
    if (item != null) {
      if (quantity <= 0) {
        await item.delete();
      } else {
        item.quantity = quantity;
        await item.save();
      }
    }
  }

  // 移除商品
  Future<void> removeItem(String productId) async {
    await _box.delete(productId);
  }

  // 清空购物车
  Future<void> clear() async {
    await _box.clear();
  }

  // 监听变化
  Stream<BoxEvent> get watch => _box.watch();
}
```

### 示例 5：Flutter 集成

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:hive_ce_flutter/hive_ce_flutter.dart';
import 'models/adapters.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 初始化 Hive
  await Hive.initFlutter();

  // 注册适配器
  registerAdapters();

  // 打开需要的 Box
  await Hive.openBox('settings');
  await Hive.openBox<User>('users');

  runApp(MyApp());
}

// 使用 Provider/ Riverpod 管理 Hive
class UserProvider extends ChangeNotifier {
  final Box<User> _userBox = Hive.box<User>('users');

  List<User> get users => _userBox.values.toList();

  Future<void> addUser(User user) async {
    await _userBox.put(user.id, user);
    notifyListeners();
  }

  Future<void> deleteUser(String id) async {
    await _userBox.delete(id);
    notifyListeners();
  }

  // 监听变化
  UserProvider() {
    _userBox.watch().listen((_) => notifyListeners());
  }
}

// UI 示例
class UserListScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Users')),
      body: ValueListenableBuilder<Box<User>>(
        valueListenable: Hive.box<User>('users').listenable(),
        builder: (context, box, _) {
          final users = box.values.toList();
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return ListTile(
                title: Text(user.name),
                subtitle: Text(user.email),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () => user.delete(),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 添加用户
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

---

## 附录

### A. 版本兼容性

| 版本   | Dart SDK | 说明       |
| ------ | -------- | ---------- |
| 2.19.3 | ^3.0.0   | 当前稳定版 |
| 2.8.0+ | ^3.0.0   | 支持 WASM  |
| 2.0.0+ | ^3.0.0   | 初始版本   |

### B. 完整 API 参考

#### 核心类

| 类名                | 用途             |
| ------------------- | ---------------- |
| `Hive`              | 主入口，单例实例 |
| `HiveInterface`     | Hive 接口定义    |
| `Box<E>`            | 数据容器         |
| `LazyBox<E>`        | 延迟加载容器     |
| `BoxBase<E>`        | Box 基类         |
| `BoxCollection`     | Box 集合         |
| `BoxEvent`          | 变化事件         |
| `HiveObject`        | 可持久化对象     |
| `HiveObjectMixin`   | HiveObject 混入  |
| `HiveList<E>`       | 对象引用列表     |
| `HiveCollection<E>` | 对象集合         |
| `TypeAdapter<T>`    | 类型适配器接口   |
| `TypeRegistry`      | 适配器注册表     |
| `BinaryReader`      | 二进制读取器     |
| `BinaryWriter`      | 二进制写入器     |
| `HiveCipher`        | 加密接口         |
| `HiveAesCipher`     | AES 加密实现     |
| `IsolatedHive`      | 多线程 Hive      |
| `IsolatedBox<E>`    | 多线程 Box       |
| `IsolateNameServer` | Isolate 名称服务 |

#### 注解

| 注解                  | 用途             |
| --------------------- | ---------------- |
| `@HiveType()`         | 标记可序列化类   |
| `@HiveField()`        | 标记可序列化字段 |
| `@GenerateAdapters()` | 自动生成适配器   |
| `AdapterSpec<T>()`    | 适配器配置       |

#### 枚举

| 枚举                           | 值                    |
| ------------------------------ | --------------------- |
| `HiveStorageBackendPreference` | `native`, `indexedDb` |

### C. 最佳实践

1. **Type ID 管理**
   - 每个类使用唯一的 Type ID
   - 使用常量管理 Type ID
   - 预留一些 ID 用于未来扩展

2. **字段索引**
   - 从 0 开始连续编号
   - 删除字段后保留索引，不重复使用
   - 使用 `defaultValue` 处理旧数据

3. **Box 命名**
   - 使用小写名称
   - 使用有意义的名称
   - 避免特殊字符

4. **性能优化**
   - 大数据使用 LazyBox
   - 批量操作使用 `putAll`/`deleteAll`
   - 定期压缩 Box

5. **数据安全**
   - 敏感数据使用加密
   - 密钥安全存储
   - 定期备份

### D. 常见问题

**Q: 如何迁移从 Hive v2 到 Hive CE？**

A: 参考官方迁移指南，主要步骤：

1. 替换依赖包
2. 更新导入语句
3. 运行代码生成
4. 测试验证

**Q: Type ID 冲突怎么办？**

A: 确保每个适配器有唯一的 Type ID，使用 `reservedTypeIds` 预留相邻 ID。

**Q: 如何处理大数据？**

A: 使用 `LazyBox` 替代 `Box`，数据按需加载。

**Q: Web 平台支持如何？**

A: Hive CE 完全支持 Web，包括 WASM 编译。

**Q: 如何调试 Hive 数据？**

A: 使用 Hive CE Inspector DevTools 扩展。

### E. 参考资源

- [Pub 包页面](https://pub.dev/packages/hive_ce)
- [API 文档](https://pub.dev/documentation/hive_ce/latest/)
- [GitHub 仓库](https://github.com/IO-Design-Team/hive_ce)
- [官方文档](https://docs.hivedb.dev/)
- [迁移指南](https://github.com/IO-Design-Team/hive_ce/blob/main/hive/MIGRATION.md)
