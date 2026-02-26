# Drift 2.31.0 详解与实战

## 内容简介

Drift 是一个用于 Dart 和 Flutter 应用程序的响应式持久化库，构建在 SQLite 之上。它提供了类型安全的数据库访问、流畅的查询 API、自动更新的数据流以及强大的迁移工具。本书秉承"类型安全、响应式编程"的核心设计理念，系统且全面地介绍 Drift 2.31.0 中的各种功能和用法。

全书共分为6章：
- **第1章 入门与基础**：详细介绍 Drift 的安装配置、数据库创建和表定义
- **第2章 数据表定义**：深入讲解表的各种定义方式、列类型和约束
- **第3章 查询操作**：全面介绍增删改查操作、过滤、排序和联表查询
- **第4章 响应式查询**：详细讲解数据流监听、自动更新机制
- **第5章 事务与批量操作**：介绍事务处理、批量操作和性能优化
- **第6章 数据库迁移**：深入讲解版本管理、迁移策略和最佳实践

---

## 第1章 入门与基础

### 1.1 什么是 Drift

Drift（前身为 Moor）是一个专为 Flutter 和 Dart 设计的响应式持久化库，它构建在 SQLite 之上，为开发者提供了类型安全的数据库访问方式。与传统的 SQLite 操作方式相比，Drift 具有以下核心优势：

1. **类型安全**：通过代码生成，在编译时就能发现 SQL 错误，而不是在运行时
2. **响应式查询**：任何查询都可以转换为自动更新的数据流（Stream）
3. **流畅的 Dart API**：使用 Dart 代码编写查询，无需拼接 SQL 字符串
4. **跨平台支持**：支持 Android、iOS、macOS、Windows、Linux 和 Web

#### 1.1.1 Drift 的架构

Drift 的架构可以分为三层：

1. **应用层**：开发者编写的 Dart 代码，包括表定义、查询等
2. **生成层**：通过 `drift_dev` 和 `build_runner` 生成的类型安全代码
3. **执行层**：底层的 SQLite 数据库操作

```
┌─────────────────────────────────────┐
│         应用层 (Dart 代码)           │
│  - 表定义 (Table)                   │
│  - 数据库类 (Database)              │
│  - 查询 (Query)                     │
└──────────────┬──────────────────────┘
               │ 代码生成
               ▼
┌─────────────────────────────────────┐
│         生成层 (.g.dart)             │
│  - 数据类 (Data Class)              │
│  - 伴随类 (Companion)               │
│  - 查询构建器                        │
└──────────────┬──────────────────────┘
               │ 运行时调用
               ▼
┌─────────────────────────────────────┐
│         执行层 (SQLite)              │
│  - sqlite3 包                       │
│  - 原生 SQLite 库                   │
└─────────────────────────────────────┘
```

### 1.2 环境配置与安装

#### 1.2.1 添加依赖

在 `pubspec.yaml` 文件中添加以下依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  drift: ^2.31.0
  drift_flutter: ^0.2.8
  path_provider: ^2.1.5

dev_dependencies:
  drift_dev: ^2.31.0
  build_runner: ^2.10.5
```

**Flutter 框架小知识**

**drift、drift_dev 和 drift_flutter 的区别**

- **drift**：核心库，包含运行时的所有功能
- **drift_dev**：开发依赖，用于代码生成
- **drift_flutter**：Flutter 集成库，简化数据库初始化和平台配置

#### 1.2.2 获取依赖

运行以下命令安装依赖：

```bash
dart pub get
```

或者使用快捷命令一次性添加所有依赖：

```bash
dart pub add drift drift_flutter path_provider dev:drift_dev dev:build_runner
```

### 1.3 创建第一个数据库

#### 1.3.1 定义数据表

创建一个 `database.dart` 文件，定义一个简单的待办事项表：

```dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

part 'database.g.dart';  // 生成的代码文件

// 定义 TodoItems 表
class TodoItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get content => text().nullable()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  BoolColumn get isCompleted => boolean().withDefault(const Constant(false))();
}

// 定义数据库类
@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;
}

// 数据库连接函数
QueryExecutor _openConnection() {
  return driftDatabase(
    name: 'todo_database',
    native: const DriftNativeOptions(
      databaseDirectory: getApplicationSupportDirectory,
    ),
  );
}
```

#### 1.3.2 生成代码

运行代码生成命令：

```bash
# 一次性生成
dart run build_runner build

# 监视模式（开发时使用）
dart run build_runner watch
```

**Dart Tips 语法小贴士**

**part 和 part of 指令**

在 Dart 中，`part` 指令用于将一个大文件拆分成多个小文件。Drift 使用代码生成技术，将表定义转换为完整的 Dart 类。生成的代码会放在 `database.g.dart` 文件中，通过 `part 'database.g.dart'` 指令与原文件关联。

### 1.4 基础 CRUD 操作

#### 1.4.1 插入数据 (Insert)

```dart
// 获取数据库实例
final database = AppDatabase();

// 插入单条数据
Future<int> addTodo(String title, String content) async {
  return await database.into(database.todoItems).insert(
    TodoItemsCompanion.insert(
      title: title,
      content: Value(content),  // 可选字段使用 Value 包装
    ),
  );
}

// 插入多条数据
Future<void> addMultipleTodos(List<TodoItem> items) async {
  await database.batch((batch) {
    batch.insertAll(
      database.todoItems,
      items.map((item) => TodoItemsCompanion.insert(
        title: item.title,
        content: Value(item.content),
      )),
    );
  });
}
```

#### 1.4.2 查询数据 (Select)

```dart
// 查询所有待办事项
Future<List<TodoItem>> getAllTodos() async {
  return await database.select(database.todoItems).get();
}

// 查询未完成的待办事项
Future<List<TodoItem>> getPendingTodos() async {
  return await (database.select(database.todoItems)
        ..where((t) => t.isCompleted.equals(false)))
      .get();
}

// 查询单条数据
Future<TodoItem?> getTodoById(int id) async {
  return await (database.select(database.todoItems)
        ..where((t) => t.id.equals(id)))
      .getSingleOrNull();
}
```

#### 1.4.3 更新数据 (Update)

```dart
// 更新待办事项
Future<bool> updateTodo(int id, String newTitle) async {
  return await database.update(database.todoItems).replace(
    TodoItemsCompanion(
      id: Value(id),
      title: Value(newTitle),
    ),
  );
}

// 标记完成
Future<int> markAsCompleted(int id) async {
  return await (database.update(database.todoItems)
        ..where((t) => t.id.equals(id)))
      .write(TodoItemsCompanion(isCompleted: const Value(true)));
}
```

#### 1.4.4 删除数据 (Delete)

```dart
// 删除单条数据
Future<int> deleteTodo(int id) async {
  return await (database.delete(database.todoItems)
        ..where((t) => t.id.equals(id)))
      .go();
}

// 删除所有已完成的数据
Future<int> deleteCompletedTodos() async {
  return await (database.delete(database.todoItems)
        ..where((t) => t.isCompleted.equals(true)))
      .go();
}
```

---

## 第2章 数据表定义

### 2.1 表的基本定义

在 Drift 中，每个表都通过继承 `Table` 类来定义。表中的列使用 `late final` 字段定义，列的类型由字段的初始值决定。

#### 2.1.1 常用属性

1）**tableName**

自定义表名，默认使用类名的蛇形命名法（snake_case）。

```dart
class Users extends Table {
  @override
  String get tableName => 'user_accounts';
  
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}
```

2）**primaryKey**

自定义主键，当不使用 `autoIncrement()` 时需要手动指定。

```dart
class Categories extends Table {
  TextColumn get code => text()();
  TextColumn get name => text()();
  
  @override
  Set<Column> get primaryKey => {code};
}
```

#### 2.1.2 不常用属性

1）**withoutRowId**

SQLite 默认会为每张表添加一个隐藏的 `rowid` 列。如果需要禁用此功能，可以重写此属性。

```dart
class SpecialTable extends Table {
  @override
  bool get withoutRowId => true;
  
  IntColumn get id => integer()();
}
```

2）**customConstraints**

添加自定义的表级约束。

```dart
class Orders extends Table {
  @override
  List<String> get customConstraints => [
    'CHECK (total_amount >= 0)',
  ];
  
  IntColumn get id => integer().autoIncrement()();
  RealColumn get totalAmount => real()();
}
```

### 2.2 列类型详解

Drift 支持多种列类型，每种类型都有特定的用途和配置选项。

#### 2.2.1 IntColumn（整数列）

```dart
class Products extends Table {
  // 自增主键
  IntColumn get id => integer().autoIncrement()();
  
  // 普通整数，带约束
  IntColumn get stock => integer().withDefault(const Constant(0))();
  
  // 可空整数
  IntColumn get discount => integer().nullable()();
  
  // 自定义主键（非自增）
  IntColumn get sku => integer()();
}
```

**常用方法：**

| 方法 | 说明 |
|------|------|
| `autoIncrement()` | 设置为自增主键 |
| `nullable()` | 允许存储 null 值 |
| `withDefault(value)` | 设置默认值 |
| `clientDefault(() => value)` | 设置 Dart 端默认值 |
| `customConstraint('UNIQUE')` | 添加自定义约束 |

#### 2.2.2 TextColumn（文本列）

```dart
class Articles extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 带长度限制的文本
  TextColumn get title => text().withLength(min: 1, max: 200)();
  
  // 可空的长文本
  TextColumn get content => text().nullable()();
  
  // 带默认值的文本
  TextColumn get status => text().withDefault(const Constant('draft'))();
  
  // 使用命名指定列名
  TextColumn get authorName => text().named('author_name')();
}
```

**常用方法：**

| 方法 | 说明 |
|------|------|
| `withLength(min, max)` | 设置长度限制 |
| `nullable()` | 允许 null |
| `withDefault(value)` | 设置默认值 |
| `named(name)` | 自定义列名 |
| `customConstraint(constraint)` | 自定义约束 |

#### 2.2.3 BoolColumn（布尔列）

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 布尔列，默认 false
  BoolColumn get isUrgent => boolean().withDefault(const Constant(false))();
  
  // 可空布尔列
  BoolColumn get isNotified => boolean().nullable()();
}
```

**Flutter 框架小知识**

**布尔列的存储方式**

SQLite 本身没有布尔类型，Drift 会将布尔值存储为整数：1 表示 true，0 表示 false。这种转换对开发者是透明的，你可以直接在 Dart 中使用 bool 类型。

#### 2.2.4 DateTimeColumn（日期时间列）

```dart
class Events extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 创建时间，默认当前时间
  DateTimeColumn get createdAt => 
      dateTime().withDefault(currentDateAndTime)();
  
  // 事件时间，可空
  DateTimeColumn get eventTime => dateTime().nullable()();
  
  // 使用特定日期作为默认值
  DateTimeColumn get scheduledDate => 
      dateTime().withDefault(Constant(DateTime(2024, 1, 1)))();
}
```

**存储格式：**

默认情况下，Drift 将 `DateTime` 存储为 Unix 时间戳（整数）。如果需要存储为 ISO-8601 字符串格式，可以在 `build.yaml` 中配置：

```yaml
targets:
  $default:
    builders:
      drift_dev:
        options:
          store_date_time_values_as_text: true
```

#### 2.2.5 RealColumn（浮点列）

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 价格，带约束
  RealColumn get price => real().withDefault(const Constant(0.0))();
  
  // 重量，可空
  RealColumn get weight => real().nullable()();
}
```

#### 2.2.6 BlobColumn（二进制列）

```dart
class Attachments extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get fileName => text()();
  
  // 存储二进制数据
  BlobColumn get fileData => blob()();
  
  // 可空的二进制数据
  BlobColumn get thumbnail => blob().nullable()();
}
```

### 2.3 列约束与修饰

#### 2.3.1 主键约束

```dart
// 方式1：使用 autoIncrement()（推荐）
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
}

// 方式2：自定义单列主键
class Categories extends Table {
  TextColumn get code => text()();
  
  @override
  Set<Column> get primaryKey => {code};
}

// 方式3：复合主键
class OrderItems extends Table {
  IntColumn get orderId => integer()();
  IntColumn get productId => integer()();
  IntColumn get quantity => integer()();
  
  @override
  Set<Column> get primaryKey => {orderId, productId};
}
```

#### 2.3.2 唯一约束

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  
  // 邮箱必须唯一
  TextColumn get email => text().unique()();
  
  // 手机号也必须唯一
  TextColumn get phone => text().unique()();
}
```

#### 2.3.3 引用约束（外键）

```dart
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  
  // 外键引用
  IntColumn get categoryId => 
      integer().references(Categories, #id)();
}
```

**引用约束选项：**

```dart
IntColumn get categoryId => integer().references(
  Categories,           // 引用的表
  #id,                  // 引用的列
  onUpdate: KeyAction.cascade,  // 更新时级联
  onDelete: KeyAction.setNull,  // 删除时设为 null
)();
```

**KeyAction 可选值：**

- `KeyAction.noAction`：不执行任何操作（默认）
- `KeyAction.restrict`：阻止操作
- `KeyAction.cascade`：级联操作
- `KeyAction.setNull`：设为 null
- `KeyAction.setDefault`：设为默认值

### 2.4 使用 Mixin 复用列定义

当多个表需要相同的列时，可以使用 Mixin 来复用代码：

```dart
// 定义可复用的列
 mixin TimestampMixin on Table {
  DateTimeColumn get createdAt => 
      dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => 
      dateTime().withDefault(currentDateAndTime)();
}

 mixin SoftDeleteMixin on Table {
  BoolColumn get isDeleted => 
      boolean().withDefault(const Constant(false))();
}

// 使用 Mixin
class Posts extends Table with TimestampMixin, SoftDeleteMixin {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  TextColumn get content => text()();
}

class Comments extends Table with TimestampMixin, SoftDeleteMixin {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get postId => integer().references(Posts, #id)();
  TextColumn get content => text()();
}
```

---

## 第3章 查询操作

### 3.1 查询构建器

Drift 提供了强大的查询构建器，支持类型安全的查询编写。

#### 3.1.1 简单查询

```dart
// 查询所有记录
Future<List<TodoItem>> getAllTodos() async {
  return await database.select(database.todoItems).get();
}

// 查询并转换为流
Stream<List<TodoItem>> watchAllTodos() {
  return database.select(database.todoItems).watch();
}

// 查询单条记录
Future<TodoItem?> getTodo(int id) async {
  return await (database.select(database.todoItems)
        ..where((t) => t.id.equals(id)))
      .getSingleOrNull();
}
```

#### 3.1.2 条件过滤 (where)

```dart
// 等于
Future<List<TodoItem>> getCompletedTodos() async {
  return await (database.select(database.todoItems)
        ..where((t) => t.isCompleted.equals(true)))
      .get();
}

// 不等于
Future<List<TodoItem>> getIncompleteTodos() async {
  return await (database.select(database.todoItems)
        ..where((t) => t.isCompleted.equals(false)))
      .get();
}

// 大于/小于
Future<List<TodoItem>> getRecentTodos(DateTime since) async {
  return await (database.select(database.todoItems)
        ..where((t) => t.createdAt.isBiggerThanValue(since)))
      .get();
}

// 包含（模糊查询）
Future<List<TodoItem>> searchTodos(String keyword) async {
  return await (database.select(database.todoItems)
        ..where((t) => t.title.contains(keyword)))
      .get();
}

// 组合条件
Future<List<TodoItem>> getFilteredTodos(String keyword, bool completed) async {
  return await (database.select(database.todoItems)
        ..where((t) => 
          t.title.contains(keyword) & 
          t.isCompleted.equals(completed)))
      .get();
}
```

**条件操作符：**

| 操作符 | 说明 | 示例 |
|--------|------|------|
| `equals(value)` | 等于 | `t.id.equals(1)` |
| `equalsValue(value)` | 等于（值类型） | `t.id.equalsValue(1)` |
| `isBiggerThan(value)` | 大于 | `t.age.isBiggerThan(18)` |
| `isSmallerThan(value)` | 小于 | `t.age.isSmallerThan(65)` |
| `contains(value)` | 包含 | `t.name.contains('John')` |
| `startsWith(value)` | 以...开头 | `t.name.startsWith('A')` |
| `endsWith(value)` | 以...结尾 | `t.name.endsWith('son')` |
| `like(pattern)` | 模糊匹配 | `t.name.like('%John%')` |
| `isIn(values)` | 在列表中 | `t.status.isIn(['active', 'pending'])` |
| `isNull()` | 为 null | `t.deletedAt.isNull()` |
| `a & b` | 逻辑与 | `t.a.equals(1) & t.b.equals(2)` |
| `a \| b` | 逻辑或 | `t.a.equals(1) \| t.b.equals(2)` |
| `a.not()` | 逻辑非 | `t.isDeleted.equals(true).not()` |

#### 3.1.3 排序 (orderBy)

```dart
// 单列排序
Future<List<TodoItem>> getTodosSortedByDate() async {
  return await (database.select(database.todoItems)
        ..orderBy([(t) => OrderingTerm.desc(t.createdAt)]))
      .get();
}

// 多列排序
Future<List<TodoItem>> getTodosSorted() async {
  return await (database.select(database.todoItems)
        ..orderBy([
          (t) => OrderingTerm.desc(t.isCompleted),
          (t) => OrderingTerm.asc(t.createdAt),
        ]))
      .get();
}
```

**OrderingMode：**

- `OrderingMode.asc`：升序（从小到大）
- `OrderingMode.desc`：降序（从大到小）

#### 3.1.4 分页 (limit)

```dart
// 限制返回数量
Future<List<TodoItem>> getTop10Todos() async {
  return await (database.select(database.todoItems)
        ..limit(10))
      .get();
}

// 分页查询
Future<List<TodoItem>> getTodosPage(int page, int pageSize) async {
  return await (database.select(database.todoItems)
        ..limit(pageSize, offset: page * pageSize))
      .get();
}
```

### 3.2 联表查询 (Join)

Drift 支持多种类型的联表查询。

#### 3.2.1 内连接 (Inner Join)

```dart
// 定义关联表
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
}

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get categoryId => integer().references(Categories, #id)();
}

// 内连接查询
Future<List<ProductWithCategory>> getProductsWithCategories() async {
  final query = database.select(database.products).join([
    innerJoin(
      database.categories,
      database.categories.id.equalsExp(database.products.categoryId),
    ),
  ]);

  return await query.map((row) {
    return ProductWithCategory(
      product: row.readTable(database.products),
      category: row.readTable(database.categories),
    );
  }).get();
}

class ProductWithCategory {
  final Product product;
  final Category category;

  ProductWithCategory({required this.product, required this.category});
}
```

#### 3.2.2 左外连接 (Left Outer Join)

```dart
// 左外连接（保留左表所有记录）
Future<List<ProductWithCategory>> getAllProducts() async {
  final query = database.select(database.products).join([
    leftOuterJoin(
      database.categories,
      database.categories.id.equalsExp(database.products.categoryId),
    ),
  ]);

  return await query.map((row) {
    return ProductWithCategory(
      product: row.readTable(database.products),
      category: row.readTableOrNull(database.categories),  // 可能为 null
    );
  }).get();
}
```

#### 3.2.3 多表连接

```dart
// 三表连接
class Orders extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get userId => integer().references(Users, #id)();
  DateTimeColumn get orderDate => dateTime()();
}

class OrderDetails extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get orderId => integer().references(Orders, #id)();
  IntColumn get productId => integer().references(Products, #id)();
  IntColumn get quantity => integer()();
}

Future<List<OrderDetailInfo>> getOrderDetails(int orderId) async {
  final query = database.select(database.orderDetails).join([
    innerJoin(
      database.orders,
      database.orders.id.equalsExp(database.orderDetails.orderId),
    ),
    innerJoin(
      database.products,
      database.products.id.equalsExp(database.orderDetails.productId),
    ),
  ])
    ..where(database.orderDetails.orderId.equals(orderId));

  return await query.map((row) {
    return OrderDetailInfo(
      orderDetail: row.readTable(database.orderDetails),
      order: row.readTable(database.orders),
      product: row.readTable(database.products),
    );
  }).get();
}
```

**Join 类型：**

| 方法 | 说明 |
|------|------|
| `innerJoin(table, condition)` | 内连接，只返回匹配的记录 |
| `leftOuterJoin(table, condition)` | 左外连接，返回左表所有记录 |
| `rightOuterJoin(table, condition)` | 右外连接，返回右表所有记录 |
| `fullOuterJoin(table, condition)` | 全外连接，返回两个表的所有记录 |
| `crossJoin(table)` | 交叉连接，返回笛卡尔积 |

### 3.3 聚合查询

#### 3.3.1 计数

```dart
// 统计总数
Future<int> getTodoCount() async {
  final result = await database.customSelect(
    'SELECT COUNT(*) as count FROM todo_items',
  ).getSingle();
  return result.read<int>('count');
}

// 使用生成的查询
Future<int> getCompletedCount() async {
  final query = database.select(database.todoItems)
    ..where((t) => t.isCompleted.equals(true));
  return await query.get().then((list) => list.length);
}
```

#### 3.3.2 求和、平均值

```dart
// 自定义聚合查询
Future<double> getAveragePrice() async {
  final result = await database.customSelect(
    'SELECT AVG(price) as avg_price FROM products',
  ).getSingle();
  return result.read<double>('avg_price');
}

Future<double> getTotalSales() async {
  final result = await database.customSelect(
    'SELECT SUM(amount) as total FROM orders',
  ).getSingle();
  return result.read<double>('total');
}
```

### 3.4 子查询

```dart
// 使用子查询
Future<List<Product>> getProductsInActiveCategories() async {
  final activeCategoryIds = database.select(database.categories)
    ..where((c) => c.isActive.equals(true))
    ..map((c) => c.id);

  return await (database.select(database.products)
        ..where((p) => p.categoryId.isInQuery(activeCategoryIds)))
      .get();
}
```

---

## 第4章 响应式查询

### 4.1 数据流监听

Drift 的核心特性之一是响应式查询，任何查询都可以转换为自动更新的 Stream。

#### 4.1.1 基础监听

```dart
// 监听所有待办事项
Stream<List<TodoItem>> watchAllTodos() {
  return database.select(database.todoItems).watch();
}

// 在 Widget 中使用
class TodoList extends StatelessWidget {
  final AppDatabase database;

  const TodoList({required this.database});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<List<TodoItem>>(
      stream: database.select(database.todoItems).watch(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        }
        
        final todos = snapshot.data ?? [];
        return ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) {
            final todo = todos[index];
            return ListTile(
              title: Text(todo.title),
              subtitle: Text(todo.content ?? ''),
            );
          },
        );
      },
    );
  }
}
```

#### 4.1.2 条件监听

```dart
// 监听未完成的待办事项
Stream<List<TodoItem>> watchPendingTodos() {
  return (database.select(database.todoItems)
        ..where((t) => t.isCompleted.equals(false)))
      .watch();
}

// 监听特定分类的产品
Stream<List<Product>> watchProductsByCategory(int categoryId) {
  return (database.select(database.products)
        ..where((p) => p.categoryId.equals(categoryId)))
      .watch();
}
```

#### 4.1.3 单条数据监听

```dart
// 监听单条记录
Stream<TodoItem?> watchTodo(int id) {
  return (database.select(database.todoItems)
        ..where((t) => t.id.equals(id)))
      .watchSingleOrNull();
}
```

### 4.2 响应式机制原理

**Flutter 框架小知识**

**Drift 的响应式引擎是如何工作的？**

Drift 内部维护了一个 `TableUpdateQuery` 注册表。当你执行 Insert、Update 或 Delete 操作时，底层的 Executor 会标记受影响的表名。

响应式查询的生命周期：

1. **订阅建立**：当你在 UI 层调用 `watch()`，Drift 会创建一个监听器
2. **变更广播**：任何写操作完成后，Drift 都会扫描所有活跃查询
3. **自动刷新**：通过计算"涉及表交集"决定是否触发重新拉取

### 4.3 数据转换

#### 4.3.1 map 转换

```dart
// 将查询结果转换为特定格式
Stream<List<String>> watchTodoTitles() {
  return database.select(database.todoItems)
      .map((row) => row.title)
      .watch();
}

// 转换为自定义对象
Stream<List<TodoSummary>> watchTodoSummaries() {
  return database.select(database.todoItems)
      .map((row) => TodoSummary(
            id: row.id,
            title: row.title,
            isDone: row.isCompleted,
          ))
      .watch();
}
```

#### 4.3.2 异步转换

```dart
// 异步转换
Stream<List<TodoItemWithExtra>> watchTodosWithExtra() async* {
  await for (final todos in database.select(database.todoItems).watch()) {
    final results = await Future.wait(
      todos.map((todo) async {
        final extra = await fetchExtraData(todo.id);
        return TodoItemWithExtra(todo: todo, extra: extra);
      }),
    );
    yield results;
  }
}
```

---

## 第5章 事务与批量操作

### 5.1 事务处理

事务可以确保一组数据库操作要么全部成功，要么全部失败，保持数据的一致性。

#### 5.1.1 基本事务

```dart
// 转账操作示例
Future<void> transferMoney(int fromId, int toId, double amount) async {
  await database.transaction(() async {
    // 扣除转出账户金额
    await (database.update(database.accounts)
          ..where((a) => a.id.equals(fromId)))
        .write(AccountsCompanion(
          balance: Value(amount * -1),
        ));

    // 增加转入账户金额
    await (database.update(database.accounts)
          ..where((a) => a.id.equals(toId)))
        .write(AccountsCompanion(
          balance: Value(amount),
        ));

    // 记录交易
    await database.into(database.transactions).insert(
      TransactionsCompanion.insert(
        fromAccount: fromId,
        toAccount: toId,
        amount: amount,
      ),
    );
  });
}
```

#### 5.1.2 事务回滚

```dart
Future<void> safeTransfer(int fromId, int toId, double amount) async {
  await database.transaction(() async {
    // 检查余额
    final fromAccount = await (database.select(database.accounts)
          ..where((a) => a.id.equals(fromId)))
        .getSingle();

    if (fromAccount.balance < amount) {
      // 抛出异常会触发回滚
      throw Exception('余额不足');
    }

    // 执行转账
    await (database.update(database.accounts)
          ..where((a) => a.id.equals(fromId)))
        .write(AccountsCompanion(
          balance: Value(fromAccount.balance - amount),
        ));

    // ... 其他操作
  });
}
```

**Dart Tips 语法小贴士**

**事务的隔离级别**

SQLite 默认使用 SERIALIZABLE 隔离级别，这意味着：
- 事务中的查询看到的是事务开始时的数据库快照
- 同时只能有一个写入事务
- 事务失败时会自动回滚所有操作

### 5.2 批量操作

批量操作可以显著提高大量数据操作的性能。

#### 5.2.1 批量插入

```dart
// 批量插入
Future<void> insertMultipleTodos(List<TodoItem> items) async {
  await database.batch((batch) {
    batch.insertAll(
      database.todoItems,
      items.map((item) => TodoItemsCompanion.insert(
        title: item.title,
        content: Value(item.content),
      )),
    );
  });
}
```

#### 5.2.2 批量更新

```dart
// 批量更新
Future<void> updateMultipleTodos(List<TodoItem> items) async {
  await database.batch((batch) {
    for (final item in items) {
      batch.update(
        database.todoItems,
        TodoItemsCompanion(
          id: Value(item.id),
          title: Value(item.title),
          isCompleted: Value(item.isCompleted),
        ),
        where: (t) => t.id.equals(item.id),
      );
    }
  });
}
```

#### 5.2.3 批量删除

```dart
// 批量删除
Future<void> deleteMultipleTodos(List<int> ids) async {
  await database.batch((batch) {
    for (final id in ids) {
      batch.delete(
        database.todoItems,
        where: (t) => t.id.equals(id),
      );
    }
  });
}
```

### 5.3 性能优化

#### 5.3.1 使用索引

```dart
class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  IntColumn get categoryId => integer()();
  
  @override
  List<String> get customConstraints => [
    'CREATE INDEX idx_category ON products(category_id)',
  ];
}
```

#### 5.3.2 延迟加载大数据

```dart
// 使用 LazyDatabase
LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'app.db'));
    return NativeDatabase.createInBackground(file);
  });
}
```

#### 5.3.3 避免 N+1 查询

```dart
// 不好的做法：N+1 查询
Future<List<ProductWithCategory>> badApproach() async {
  final products = await database.select(database.products).get();
  
  // 这会为每个产品都执行一次查询
  return await Future.wait(products.map((p) async {
    final category = await (database.select(database.categories)
          ..where((c) => c.id.equals(p.categoryId)))
        .getSingle();
    return ProductWithCategory(product: p, category: category);
  }));
}

// 好的做法：使用 Join
Future<List<ProductWithCategory>> goodApproach() async {
  final query = database.select(database.products).join([
    innerJoin(
      database.categories,
      database.categories.id.equalsExp(database.products.categoryId),
    ),
  ]);

  return await query.map((row) {
    return ProductWithCategory(
      product: row.readTable(database.products),
      category: row.readTable(database.categories),
    );
  }).get();
}
```

---

## 第6章 数据库迁移

### 6.1 迁移基础

当数据库结构发生变化时，需要编写迁移代码来更新现有数据库。

#### 6.1.1 基本迁移配置

```dart
@DriftDatabase(tables: [TodoItems, Categories])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 2;  // 升级版本号

  @override
  MigrationStrategy get migration {
    return MigrationStrategy(
      onCreate: (Migrator m) async {
        await m.createAll();
      },
      onUpgrade: (Migrator m, int from, int to) async {
        if (from < 2) {
          // 从版本1升级到版本2
          await m.addColumn(todoItems, todoItems.priority);
        }
      },
    );
  }
}
```

#### 6.1.2 常用迁移操作

```dart
// 添加列
await m.addColumn(table, table.newColumn);

// 删除列（SQLite 不直接支持，需要重建表）
await m.dropColumn(table, 'column_name');

// 创建表
await m.createTable(newTable);

// 删除表
await m.deleteTable('table_name');

// 创建索引
await m.createIndex(IndexName(table, [table.column]));

// 删除索引
await m.dropIndex('index_name');

// 重建所有视图
await m.recreateAllViews();
```

### 6.2 高级迁移策略

#### 6.2.1 分步迁移

```dart
@override
MigrationStrategy get migration {
  return MigrationStrategy(
    onCreate: (Migrator m) async {
      await m.createAll();
    },
    onUpgrade: (Migrator m, int from, int to) async {
      // 从版本1升级到版本2
      if (from < 2) {
        await m.addColumn(todoItems, todoItems.dueDate);
      }
      
      // 从版本2升级到版本3
      if (from < 3) {
        await m.addColumn(todoItems, todoItems.priority);
        await m.createTable(categories);
      }
      
      // 从版本3升级到版本4
      if (from < 4) {
        await m.addColumn(categories, categories.color);
      }
    },
  );
}
```

#### 6.2.2 数据迁移

```dart
onUpgrade: (Migrator m, int from, int to) async {
  if (from < 2) {
    // 添加新列
    await m.addColumn(users, users.fullName);
    
    // 迁移数据：将 firstName 和 lastName 合并为 fullName
    await customUpdate(
      'UPDATE users SET full_name = first_name || \' \' || last_name',
    );
    
    // 删除旧列
    await m.dropColumn(users, 'first_name');
    await m.dropColumn(users, 'last_name');
  }
},
```

### 6.3 使用 make-migrations 工具

Drift 提供了 `make-migrations` 命令来自动生成迁移代码。

#### 6.3.1 配置 build.yaml

```yaml
targets:
  $default:
    builders:
      drift_dev:
        options:
          databases:
            app_database: lib/database.dart
          schema_dir: drift_schemas/
          test_dir: test/drift/
```

#### 6.3.2 生成迁移

```bash
# 生成初始迁移
dart run drift_dev make-migrations

# 修改表结构后，升级版本号并再次运行
dart run drift_dev make-migrations
```

#### 6.3.3 使用生成的迁移代码

```dart
import 'database.steps.dart';  // 生成的文件

@DriftDatabase(tables: [TodoItems, Categories])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 3;

  @override
  MigrationStrategy get migration {
    return MigrationStrategy(
      onCreate: (Migrator m) async {
        await m.createAll();
      },
      onUpgrade: stepByStep(
        from1To2: (m, schema) async {
          await m.addColumn(schema.todoItems, schema.todoItems.dueDate);
        },
        from2To3: (m, schema) async {
          await m.createTable(schema.categories);
        },
      ),
    );
  }
}
```

### 6.4 迁移最佳实践

#### 6.4.1 外键处理

```dart
@override
MigrationStrategy get migration {
  return MigrationStrategy(
    onUpgrade: (Migrator m, int from, int to) async {
      // 禁用外键检查
      await customStatement('PRAGMA foreign_keys = OFF');

      // 执行迁移
      if (from < 2) {
        await m.addColumn(todoItems, todoItems.categoryId);
      }

      // 重新启用外键检查
      await customStatement('PRAGMA foreign_keys = ON');

      // 验证外键完整性（仅在调试模式）
      if (kDebugMode) {
        final wrongForeignKeys = await customSelect(
          'PRAGMA foreign_key_check',
        ).get();
        assert(
          wrongForeignKeys.isEmpty,
          'Foreign key violations: ${wrongForeignKeys.map((e) => e.data)}',
        );
      }
    },
  );
}
```

#### 6.4.2 测试迁移

```dart
import 'package:test/test.dart';
import 'package:drift/drift.dart';

void main() {
  group('Migration tests', () {
    test('can migrate from v1 to v2', () async {
      final database = AppDatabase.forTesting(schemaVersion: 1);
      
      // 插入测试数据
      await database.into(database.todoItems).insert(
        TodoItemsCompanion.insert(title: 'Test'),
      );
      
      // 执行迁移
      await database.migration.onUpgrade(
        database.migrator,
        1,
        2,
      );
      
      // 验证数据完整性
      final todos = await database.select(database.todoItems).get();
      expect(todos.length, 1);
      expect(todos.first.title, 'Test');
      
      await database.close();
    });
  });
}
```

---

## 附录：完整示例代码

### 示例1：待办事项应用

```dart
// database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

part 'database.g.dart';

class TodoItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get description => text().nullable()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get dueDate => dateTime().nullable()();
  BoolColumn get isCompleted => boolean().withDefault(const Constant(false))();
  IntColumn get priority => integer().withDefault(const Constant(0))();
}

@DriftDatabase(tables: [TodoItems])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(
      name: 'todo_app',
      native: const DriftNativeOptions(
        databaseDirectory: getApplicationSupportDirectory,
      ),
    );
  }

  // CRUD 操作
  Future<int> addTodo(String title, {String? description, DateTime? dueDate}) {
    return into(todoItems).insert(
      TodoItemsCompanion.insert(
        title: title,
        description: Value(description),
        dueDate: Value(dueDate),
      ),
    );
  }

  Future<List<TodoItem>> getAllTodos() => select(todoItems).get();

  Stream<List<TodoItem>> watchAllTodos() => select(todoItems).watch();

  Stream<List<TodoItem>> watchPendingTodos() {
    return (select(todoItems)
          ..where((t) => t.isCompleted.equals(false))
          ..orderBy([(t) => OrderingTerm.desc(t.priority)]))
        .watch();
  }

  Future<bool> updateTodo(TodoItem todo) => update(todoItems).replace(todo);

  Future<int> deleteTodo(int id) {
    return (delete(todoItems)..where((t) => t.id.equals(id))).go();
  }

  Future<int> markAsCompleted(int id) {
    return (update(todoItems)..where((t) => t.id.equals(id)))
        .write(const TodoItemsCompanion(isCompleted: Value(true)));
  }
}
```

### 示例2：电商应用

```dart
// shop_database.dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

part 'shop_database.g.dart';

class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
}

class Products extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get description => text().nullable()();
  RealColumn get price => real()();
  IntColumn get stock => integer().withDefault(const Constant(0))();
  IntColumn get categoryId => integer().references(Categories, #id)();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

class CartItems extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get productId => integer().references(Products, #id)();
  IntColumn get quantity => integer().withDefault(const Constant(1))();
}

@DriftDatabase(tables: [Categories, Products, CartItems])
class ShopDatabase extends _$ShopDatabase {
  ShopDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  static QueryExecutor _openConnection() {
    return driftDatabase(name: 'shop_app');
  }

  // 产品相关
  Future<List<Product>> getProductsByCategory(int categoryId) {
    return (select(products)
          ..where((p) => p.categoryId.equals(categoryId)))
        .get();
  }

  Stream<List<ProductWithCategory>> watchAllProducts() {
    final query = select(products).join([
      innerJoin(
        categories,
        categories.id.equalsExp(products.categoryId),
      ),
    ]);

    return query.map((row) {
      return ProductWithCategory(
        product: row.readTable(products),
        category: row.readTable(categories),
      );
    }).watch();
  }

  // 购物车相关
  Future<void> addToCart(int productId, {int quantity = 1}) async {
    final existing = await (select(cartItems)
          ..where((c) => c.productId.equals(productId)))
        .getSingleOrNull();

    if (existing != null) {
      await update(cartItems).replace(
        existing.copyWith(quantity: existing.quantity + quantity),
      );
    } else {
      await into(cartItems).insert(
        CartItemsCompanion.insert(productId: productId, quantity: quantity),
      );
    }
  }

  Stream<List<CartItemWithProduct>> watchCart() {
    final query = select(cartItems).join([
      innerJoin(products, products.id.equalsExp(cartItems.productId)),
    ]);

    return query.map((row) {
      return CartItemWithProduct(
        cartItem: row.readTable(cartItems),
        product: row.readTable(products),
      );
    }).watch();
  }

  Future<double> getCartTotal() async {
    final query = select(cartItems).join([
      innerJoin(products, products.id.equalsExp(cartItems.productId)),
    ]);

    final items = await query.get();
    return items.fold<double>(
      0,
      (sum, row) {
        final cartItem = row.readTable(cartItems);
        final product = row.readTable(products);
        return sum + (product.price * cartItem.quantity);
      },
    );
  }
}

class ProductWithCategory {
  final Product product;
  final Category category;

  ProductWithCategory({required this.product, required this.category});
}

class CartItemWithProduct {
  final CartItem cartItem;
  final Product product;

  CartItemWithProduct({required this.cartItem, required this.product});
}
```

---

## 结语

Drift 2.31.0 为 Flutter 开发者提供了一个强大、类型安全且易于使用的数据库解决方案。通过本书的学习，你应该已经掌握了：

1. **基础配置**：如何安装和配置 Drift
2. **表定义**：如何定义数据表和列
3. **查询操作**：如何进行增删改查和联表查询
4. **响应式编程**：如何使用 Stream 实现自动更新的 UI
5. **事务处理**：如何保证数据一致性
6. **数据库迁移**：如何安全地升级数据库结构

Drift 的设计理念是让数据库操作变得简单、安全和高效。在实际项目中，建议：

- 使用代码生成确保类型安全
- 利用响应式查询简化 UI 更新
- 合理使用事务保证数据一致性
- 编写迁移测试确保升级安全
- 使用批量操作优化性能

希望本书能帮助你更好地使用 Drift 构建高质量的 Flutter 应用！
