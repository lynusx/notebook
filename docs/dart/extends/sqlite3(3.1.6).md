# sqlite3 3.1.6 详解与实战

## 内容简介

sqlite3 是一个为 Dart 和 Flutter 应用程序提供 SQLite 绑定的轻量级库，通过 `dart:ffi` 直接调用原生 SQLite 库。它提供了对 SQLite 的完整访问能力，包括预编译语句、事务处理、用户自定义函数、BLOB 操作等高级功能。

全书共分为6章：

- **第1章 入门与基础**：详细介绍 sqlite3 的安装配置、数据库打开和基础操作
- **第2章 数据库操作**：深入讲解数据库属性、配置和执行 SQL 语句
- **第3章 预编译语句**：全面介绍 PreparedStatement 的使用、参数绑定和性能优化
- **第4章 事务处理**：介绍事务控制、隔离级别和并发处理
- **第5章 数据类型与绑定**：详细讲解 SQLite 数据类型、参数绑定和 BLOB 操作
- **第6章 高级功能**：介绍用户自定义函数、数据变更监听、虚拟文件系统等

---

## 第1章 入门与基础

### 1.1 什么是 sqlite3

sqlite3 是一个 Dart 包，通过 `dart:ffi` 提供对 SQLite 数据库的直接访问。与其他 Dart SQLite 封装库（如 sqflite、drift）不同，sqlite3 提供了最接近原生 SQLite 的 API，让开发者能够完全控制数据库的每一个细节。

#### 1.1.1 sqlite3 的特点

1. **原生 FFI 绑定**：直接使用 `dart:ffi` 调用 SQLite C 库，性能最优
2. **完整的 API 覆盖**：支持 SQLite 的所有核心功能
3. **跨平台支持**：支持 Android、iOS、macOS、Windows、Linux 和 Web（WebAssembly）
4. **无额外依赖**：不依赖平台特定的插件或通道

#### 1.1.2 sqlite3 的架构

```
┌─────────────────────────────────────┐
│         应用层 (Dart 代码)           │
│  - Database 对象                     │
│  - PreparedStatement                 │
│  - 查询执行                          │
└──────────────┬──────────────────────┘
               │ dart:ffi
               ▼
┌─────────────────────────────────────┐
│         FFI 层                       │
│  - DynamicLibrary                    │
│  - Pointer 操作                      │
└──────────────┬──────────────────────┘
               │ 原生调用
               ▼
┌─────────────────────────────────────┐
│         SQLite 原生库                │
│  - sqlite3.dll/.so/.dylib            │
│  - 数据库引擎                        │
└─────────────────────────────────────┘
```

### 1.2 环境配置与安装

#### 1.2.1 添加依赖

在 `pubspec.yaml` 文件中添加以下依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  sqlite3: ^3.1.6
  path_provider: ^2.1.5
  path: ^1.9.0
```

**Flutter 框架小知识**

**sqlite3 与 sqlite3_flutter_libs 的区别**

- **sqlite3**：核心库，提供 Dart FFI 绑定
- **sqlite3_flutter_libs**：可选依赖，包含预编译的 SQLite 原生库，简化 Flutter 项目的配置

对于 Flutter 项目，推荐同时添加 `sqlite3_flutter_libs`：

```yaml
dependencies:
  sqlite3: ^3.1.6
  sqlite3_flutter_libs: ^0.5.32
```

#### 1.2.2 获取依赖

运行以下命令安装依赖：

```bash
dart pub get
```

或者使用快捷命令：

```bash
dart pub add sqlite3 sqlite3_flutter_libs path_provider path
```

### 1.3 打开数据库

#### 1.3.1 打开文件数据库

```dart
import 'package:sqlite3/sqlite3.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

Future<Database> openDatabase() async {
  // 获取应用文档目录
  final documentsDir = await getApplicationDocumentsDirectory();
  final dbPath = p.join(documentsDir.path, 'app_database.db');

  // 打开数据库
  final db = sqlite3.open(dbPath);

  return db;
}
```

#### 1.3.2 打开内存数据库

```dart
import 'package:sqlite3/sqlite3.dart';

// 打开临时内存数据库
void openInMemoryDatabase() {
  final db = sqlite3.openInMemory();

  // 使用数据库...

  // 关闭数据库
  db.dispose();
}
```

#### 1.3.3 数据库选项

```dart
import 'package:sqlite3/sqlite3.dart';

Future<Database> openDatabaseWithOptions() async {
  final documentsDir = await getApplicationDocumentsDirectory();
  final dbPath = p.join(documentsDir.path, 'app_database.db');

  // 使用选项打开数据库
  final db = sqlite3.open(
    dbPath,
    mode: OpenMode.readWriteCreate,  // 读写模式，不存在则创建
    vfs: null,                        // 虚拟文件系统
  );

  return db;
}
```

**OpenMode 可选值：**

| 值                         | 说明                           |
| -------------------------- | ------------------------------ |
| `OpenMode.readOnly`        | 只读模式                       |
| `OpenMode.readWrite`       | 读写模式，数据库必须已存在     |
| `OpenMode.readWriteCreate` | 读写模式，不存在则创建（默认） |

### 1.4 基础 CRUD 操作

#### 1.4.1 创建表

```dart
import 'package:sqlite3/sqlite3.dart';

void createTable(Database db) {
  db.execute('''
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      email TEXT UNIQUE,
      age INTEGER DEFAULT 0,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  ''');
}
```

#### 1.4.2 插入数据

```dart
import 'package:sqlite3/sqlite3.dart';

void insertUser(Database db, String name, String email, int age) {
  // 方式1：直接执行 SQL
  db.execute(
    'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
    [name, email, age],
  );

  print('插入成功，ID: ${db.lastInsertRowId}');
}

// 批量插入
void insertMultipleUsers(Database db, List<Map<String, dynamic>> users) {
  final stmt = db.prepare(
    'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
  );

  try {
    for (final user in users) {
      stmt.execute([user['name'], user['email'], user['age']]);
    }
  } finally {
    stmt.dispose();
  }
}
```

#### 1.4.3 查询数据

```dart
import 'package:sqlite3/sqlite3.dart';

void queryUsers(Database db) {
  // 查询所有用户
  final result = db.select('SELECT * FROM users');

  for (final row in result) {
    print('ID: ${row['id']}, Name: ${row['name']}, Email: ${row['email']}');
  }
}

// 条件查询
void queryUserById(Database db, int id) {
  final result = db.select(
    'SELECT * FROM users WHERE id = ?',
    [id],
  );

  if (result.isNotEmpty) {
    final row = result.first;
    print('找到用户: ${row['name']}');
  }
}

// 使用命名参数
void queryUsersByAge(Database db, int minAge) {
  final result = db.select(
    'SELECT * FROM users WHERE age > :min_age',
    {'min_age': minAge},
  );

  print('找到 ${result.length} 个用户');
}
```

#### 1.4.4 更新数据

```dart
import 'package:sqlite3/sqlite3.dart';

void updateUser(Database db, int id, String newName) {
  db.execute(
    'UPDATE users SET name = ? WHERE id = ?',
    [newName, id],
  );

  print('更新了 ${db.updatedRows} 行');
}
```

#### 1.4.5 删除数据

```dart
import 'package:sqlite3/sqlite3.dart';

void deleteUser(Database db, int id) {
  db.execute(
    'DELETE FROM users WHERE id = ?',
    [id],
  );

  print('删除了 ${db.updatedRows} 行');
}
```

#### 1.4.6 关闭数据库

```dart
import 'package:sqlite3/sqlite3.dart';

void closeDatabase(Database db) {
  // 释放数据库资源
  db.dispose();
}
```

**Dart Tips 语法小贴士**

**资源释放的重要性**

sqlite3 使用原生资源（内存、文件句柄等），这些资源不会自动被 Dart 的垃圾回收器释放。因此，必须显式调用 `dispose()` 方法来释放：

- **Database.dispose()**：关闭数据库连接
- **PreparedStatement.dispose()**：释放预编译语句
- **ResultSet**：不需要手动释放，但底层资源会在迭代完成后自动释放

建议使用 `try-finally` 或 `try-catch-finally` 确保资源被正确释放：

```dart
final db = sqlite3.open(dbPath);
try {
  // 使用数据库...
} finally {
  db.dispose();
}
```

---

## 第2章 数据库操作

### 2.1 Database 类详解

`Database` 类是 sqlite3 的核心类，代表一个打开的 SQLite 数据库连接。

#### 2.1.1 常用属性

1）**lastInsertRowId**

获取最近一次 INSERT 操作插入的行的 ID。

```dart
import 'package:sqlite3/sqlite3.dart';

void demonstrateLastInsertRowId(Database db) {
  db.execute("INSERT INTO users (name) VALUES ('Alice')");
  final id = db.lastInsertRowId;
  print('新用户 ID: $id');  // 输出: 新用户 ID: 1
}
```

2）**updatedRows**

获取最近一次 INSERT、UPDATE 或 DELETE 操作影响的行数。

```dart
import 'package:sqlite3/sqlite3.dart';

void demonstrateUpdatedRows(Database db) {
  db.execute("UPDATE users SET age = 25 WHERE age < 25");
  final count = db.updatedRows;
  print('更新了 $count 行');
}
```

3）**autocommit**

检查数据库是否处于自动提交模式。

```dart
import 'package:sqlite3/sqlite3.dart';

void checkAutocommit(Database db) {
  print('自动提交模式: ${db.autocommit}');  // 默认: true
}
```

4）**userVersion**

获取或设置数据库的用户版本号，常用于数据库迁移。

```dart
import 'package:sqlite3/sqlite3.dart';

void manageUserVersion(Database db) {
  // 获取当前版本
  final currentVersion = db.userVersion;
  print('当前数据库版本: $currentVersion');

  // 设置新版本
  db.userVersion = 2;
}
```

#### 2.1.2 不常用属性

1）**handle**

获取底层的 SQLite 数据库连接句柄（Pointer<void>），用于与原生代码交互。

```dart
import 'package:sqlite3/sqlite3.dart';
import 'package:ffi/ffi.dart';

void getNativeHandle(Database db) {
  final handle = db.handle;
  print('原生句柄: $handle');
}
```

2）**config**

获取数据库配置对象，用于设置各种数据库选项。

```dart
import 'package:sqlite3/sqlite3.dart';

void configureDatabase(Database db) {
  final config = db.config;
  // 配置数据库选项...
}
```

### 2.2 执行 SQL 语句

#### 2.2.1 execute 方法

`execute` 方法用于执行不返回结果集的 SQL 语句（如 CREATE、INSERT、UPDATE、DELETE）。

```dart
import 'package:sqlite3/sqlite3.dart';

void executeExamples(Database db) {
  // 创建表
  db.execute('''
    CREATE TABLE IF NOT EXISTS products (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      price REAL NOT NULL,
      stock INTEGER DEFAULT 0
    )
  ''');

  // 插入数据（带参数）
  db.execute(
    'INSERT INTO products (name, price, stock) VALUES (?, ?, ?)',
    ['iPhone 15', 6999.00, 100],
  );

  // 更新数据
  db.execute(
    'UPDATE products SET price = ? WHERE id = ?',
    [6599.00, 1],
  );

  // 删除数据
  db.execute(
    'DELETE FROM products WHERE stock = ?',
    [0],
  );
}
```

#### 2.2.2 select 方法

`select` 方法用于执行返回结果集的 SQL 查询（SELECT）。

```dart
import 'package:sqlite3/sqlite3.dart';

void selectExamples(Database db) {
  // 简单查询
  final allProducts = db.select('SELECT * FROM products');

  // 条件查询（位置参数）
  final expensiveProducts = db.select(
    'SELECT * FROM products WHERE price > ?',
    [5000],
  );

  // 条件查询（命名参数）
  final productsInStock = db.select(
    'SELECT * FROM products WHERE stock > :min_stock',
    {'min_stock': 50},
  );

  // 聚合查询
  final stats = db.select('''
    SELECT
      COUNT(*) as total,
      AVG(price) as avg_price,
      MAX(price) as max_price,
      MIN(price) as min_price
    FROM products
  ''');

  // 处理结果
  for (final row in allProducts) {
    print('${row['name']}: ¥${row['price']}');
  }
}
```

### 2.3 数据库配置

#### 2.3.1 使用 PRAGMA 语句

SQLite 使用 PRAGMA 语句来设置和查询数据库配置。

```dart
import 'package:sqlite3/sqlite3.dart';

void configureWithPragma(Database db) {
  // 启用外键约束
  db.execute('PRAGMA foreign_keys = ON');

  // 设置同步模式
  db.execute('PRAGMA synchronous = NORMAL');

  // 设置日志模式
  db.execute('PRAGMA journal_mode = WAL');

  // 设置缓存大小
  db.execute('PRAGMA cache_size = 10000');

  // 查询配置
  final result = db.select('PRAGMA journal_mode');
  print('当前日志模式: ${result.first['journal_mode']}');
}
```

**常用 PRAGMA 配置：**

| PRAGMA         | 说明         | 推荐值             |
| -------------- | ------------ | ------------------ |
| `foreign_keys` | 启用外键约束 | `ON`               |
| `synchronous`  | 同步模式     | `NORMAL` 或 `FULL` |
| `journal_mode` | 日志模式     | `WAL`（写前日志）  |
| `cache_size`   | 缓存大小     | 根据内存调整       |
| `temp_store`   | 临时存储     | `MEMORY`           |
| `busy_timeout` | 忙等待超时   | 5000（毫秒）       |

#### 2.3.2 DatabaseConfig 类

```dart
import 'package:sqlite3/sqlite3.dart';

void useDatabaseConfig(Database db) {
  final config = db.config;

  // 启用外键约束
  config.setIntConfig(SQLiteConfig.FK, 1);

  // 设置最大表达式深度
  config.setIntConfig(SQLiteConfig.MAX_EXPR_DEPTH, 1000);
}
```

### 2.4 数据变更监听

#### 2.4.1 监听提交和回滚

```dart
import 'package:sqlite3/sqlite3.dart';

void listenToCommits(Database db) {
  // 监听事务提交
  db.commits.listen((_) {
    print('事务已提交');
  });

  // 监听事务回滚
  db.rollbacks.listen((_) {
    print('事务已回滚');
  });
}
```

#### 2.4.2 监听数据更新

```dart
import 'package:sqlite3/sqlite3.dart';

void listenToUpdates(Database db) {
  // 监听数据变更
  db.updates.listen((update) {
    print('表 ${update.tableName} 发生 ${update.kind} 操作');
    print('行 ID: ${update.rowId}');
  });
}
```

**SqliteUpdate 属性：**

| 属性        | 类型         | 说明                               |
| ----------- | ------------ | ---------------------------------- |
| `kind`      | `UpdateKind` | 操作类型（insert、delete、update） |
| `tableName` | `String`     | 受影响的表名                       |
| `rowId`     | `int`        | 受影响的行 ID                      |

---

## 第3章 预编译语句

### 3.1 PreparedStatement 简介

预编译语句（Prepared Statement）是 SQLite 的重要特性，它可以：

1. **提高性能**：SQL 语句只需解析一次，可多次执行
2. **防止 SQL 注入**：参数自动转义，避免安全问题
3. **减少内存使用**：复用已编译的语句

### 3.2 创建和使用预编译语句

#### 3.2.1 基础用法

```dart
import 'package:sqlite3/sqlite3.dart';

void preparedStatementExample(Database db) {
  // 准备语句
  final stmt = db.prepare('INSERT INTO users (name, email) VALUES (?, ?)');

  try {
    // 执行多次
    stmt.execute(['Alice', 'alice@example.com']);
    stmt.execute(['Bob', 'bob@example.com']);
    stmt.execute(['Charlie', 'charlie@example.com']);
  } finally {
    // 释放资源
    stmt.dispose();
  }
}
```

#### 3.2.2 使用命名参数

```dart
import 'package:sqlite3/sqlite3.dart';

void namedParametersExample(Database db) {
  // 使用命名参数
  final stmt = db.prepare('''
    INSERT INTO users (name, email, age)
    VALUES (:name, :email, :age)
  ''');

  try {
    stmt.execute({
      'name': 'David',
      'email': 'david@example.com',
      'age': 25,
    });
  } finally {
    stmt.dispose();
  }
}
```

### 3.3 参数绑定详解

#### 3.3.1 支持的参数类型

```dart
import 'package:sqlite3/sqlite3.dart';

void bindParametersExample(Database db) {
  final stmt = db.prepare('''
    INSERT INTO test_table
    (int_col, real_col, text_col, blob_col, null_col)
    VALUES (?, ?, ?, ?, ?)
  ''');

  try {
    stmt.execute([
      42,                          // int -> INTEGER
      3.14159,                     // double -> REAL
      'Hello, World!',             // String -> TEXT
      [0x00, 0x01, 0x02, 0x03],    // List<int> -> BLOB
      null,                        // null -> NULL
    ]);
  } finally {
    stmt.dispose();
  }
}
```

**支持的 Dart 类型：**

| Dart 类型           | SQLite 类型 |
| ------------------- | ----------- |
| `int`               | INTEGER     |
| `double`            | REAL        |
| `String`            | TEXT        |
| `List<int>`         | BLOB        |
| `null`              | NULL        |
| `BigInt` (Web 平台) | INTEGER     |

#### 3.3.2 位置参数

```dart
import 'package:sqlite3/sqlite3.dart';

void positionalParameters(Database db) {
  // 使用 ? 作为占位符
  final stmt = db.prepare('SELECT * FROM users WHERE age > ? AND name LIKE ?');

  try {
    final result = stmt.select([18, '%Alice%']);

    for (final row in result) {
      print('${row['name']}: ${row['age']}');
    }
  } finally {
    stmt.dispose();
  }
}
```

#### 3.3.3 命名参数

```dart
import 'package:sqlite3/sqlite3.dart';

void namedParameters(Database db) {
  // 使用 :name 作为占位符
  final stmt = db.prepare('''
    SELECT * FROM users
    WHERE age > :min_age
    AND name LIKE :name_pattern
  ''');

  try {
    final result = stmt.select({
      'min_age': 18,
      'name_pattern': '%Alice%',
    });

    for (final row in result) {
      print('${row['name']}: ${row['age']}');
    }
  } finally {
    stmt.dispose();
  }
}
```

### 3.4 预编译语句的高级用法

#### 3.4.1 复用语句

```dart
import 'package:sqlite3/sqlite3.dart';

void reuseStatement(Database db, List<Map<String, dynamic>> users) {
  final stmt = db.prepare('INSERT INTO users (name, email) VALUES (?, ?)');

  try {
    // 批量插入
    for (final user in users) {
      stmt.execute([user['name'], user['email']]);
    }
  } finally {
    stmt.dispose();
  }
}
```

#### 3.4.2 读取结果集

```dart
import 'package:sqlite3/sqlite3.dart';

void readResultSet(Database db) {
  final stmt = db.prepare('SELECT * FROM users WHERE age > ?');

  try {
    final result = stmt.select([18]);

    // 遍历结果
    for (final row in result) {
      // 通过列名访问
      print('Name: ${row['name']}');

      // 通过列索引访问
      print('First column: ${row.columnAt(0)}');

      // 获取所有列名
      print('Columns: ${row.keys}');

      // 获取所有值
      print('Values: ${row.values}');
    }
  } finally {
    stmt.dispose();
  }
}
```

**Row 类的方法：**

| 方法                      | 说明             |
| ------------------------- | ---------------- |
| `row['column_name']`      | 通过列名获取值   |
| `row.columnAt(index)`     | 通过列索引获取值 |
| `row.keys`                | 获取所有列名     |
| `row.values`              | 获取所有值       |
| `row.length`              | 获取列数         |
| `row.containsKey('name')` | 检查是否包含某列 |

### 3.5 性能优化

#### 3.5.1 批量插入优化

```dart
import 'package:sqlite3/sqlite3.dart';

void optimizedBatchInsert(Database db, List<Map<String, dynamic>> users) {
  // 使用事务包裹批量插入
  db.execute('BEGIN TRANSACTION');

  final stmt = db.prepare('INSERT INTO users (name, email) VALUES (?, ?)');

  try {
    for (final user in users) {
      stmt.execute([user['name'], user['email']]);
    }

    db.execute('COMMIT');
  } catch (e) {
    db.execute('ROLLBACK');
    rethrow;
  } finally {
    stmt.dispose();
  }
}
```

#### 3.5.2 使用 WAL 模式

```dart
import 'package:sqlite3/sqlite3.dart';

void enableWALMode(Database db) {
  // 启用 WAL 模式，提高并发性能
  db.execute('PRAGMA journal_mode = WAL');

  // 设置 WAL 自动检查点阈值
  db.execute('PRAGMA wal_autocheckpoint = 1000');
}
```

---

## 第4章 事务处理

### 4.1 事务基础

SQLite 事务遵循 ACID 原则：

- **原子性（Atomicity）**：事务中的所有操作要么全部成功，要么全部失败
- **一致性（Consistency）**：事务执行前后，数据库保持一致性状态
- **隔离性（Isolation）**：并发事务之间相互隔离
- **持久性（Durability）**：已提交的事务对数据库的修改是永久的

### 4.2 基本事务控制

#### 4.2.1 显式事务

```dart
import 'package:sqlite3/sqlite3.dart';

void explicitTransaction(Database db) {
  try {
    // 开始事务
    db.execute('BEGIN TRANSACTION');

    // 执行多个操作
    db.execute(
      'INSERT INTO accounts (user_id, balance) VALUES (?, ?)',
      [1, 1000],
    );

    db.execute(
      'INSERT INTO accounts (user_id, balance) VALUES (?, ?)',
      [2, 500],
    );

    // 提交事务
    db.execute('COMMIT');
    print('事务提交成功');
  } catch (e) {
    // 回滚事务
    db.execute('ROLLBACK');
    print('事务回滚: $e');
  }
}
```

#### 4.2.2 事务封装函数

```dart
import 'package:sqlite3/sqlite3.dart';

Future<T> runInTransaction<T>(
  Database db,
  Future<T> Function() operation,
) async {
  db.execute('BEGIN TRANSACTION');

  try {
    final result = await operation();
    db.execute('COMMIT');
    return result;
  } catch (e) {
    db.execute('ROLLBACK');
    rethrow;
  }
}

// 使用示例
Future<void> transferMoney(
  Database db,
  int fromId,
  int toId,
  double amount,
) async {
  await runInTransaction(db, () async {
    // 检查余额
    final result = db.select(
      'SELECT balance FROM accounts WHERE id = ?',
      [fromId],
    );

    if (result.isEmpty) {
      throw Exception('账户不存在');
    }

    final balance = result.first['balance'] as double;
    if (balance < amount) {
      throw Exception('余额不足');
    }

    // 扣款
    db.execute(
      'UPDATE accounts SET balance = balance - ? WHERE id = ?',
      [amount, fromId],
    );

    // 收款
    db.execute(
      'UPDATE accounts SET balance = balance + ? WHERE id = ?',
      [amount, toId],
    );

    // 记录交易
    db.execute(
      'INSERT INTO transactions (from_id, to_id, amount) VALUES (?, ?, ?)',
      [fromId, toId, amount],
    );
  });
}
```

### 4.3 事务类型

#### 4.3.1 DEFERRED 事务（默认）

```dart
import 'package:sqlite3/sqlite3.dart';

void deferredTransaction(Database db) {
  // DEFERRED：事务在实际访问数据库时才开始
  db.execute('BEGIN DEFERRED');

  // 此时事务尚未真正开始
  // 只有在执行第一个查询或修改时，事务才开始

  db.execute('SELECT * FROM users');  // 事务从这里开始

  db.execute('COMMIT');
}
```

#### 4.3.2 IMMEDIATE 事务

```dart
import 'package:sqlite3/sqlite3.dart';

void immediateTransaction(Database db) {
  // IMMEDIATE：立即获取写锁，阻止其他写入
  db.execute('BEGIN IMMEDIATE');

  // 事务立即开始，其他写入操作会被阻塞

  db.execute('UPDATE users SET age = age + 1');

  db.execute('COMMIT');
}
```

#### 4.3.3 EXCLUSIVE 事务

```dart
import 'package:sqlite3/sqlite3.dart';

void exclusiveTransaction(Database db) {
  // EXCLUSIVE：独占模式，阻止其他读写操作
  db.execute('BEGIN EXCLUSIVE');

  // 其他连接的所有操作都会被阻塞

  db.execute('DELETE FROM users WHERE age < 18');

  db.execute('COMMIT');
}
```

**事务类型对比：**

| 类型      | 开始时机 | 读锁   | 写锁   |
| --------- | -------- | ------ | ------ |
| DEFERRED  | 首次访问 | 首次读 | 首次写 |
| IMMEDIATE | 立即     | 立即   | 立即   |
| EXCLUSIVE | 立即     | 独占   | 独占   |

### 4.4 保存点

保存点（Savepoint）允许在事务中设置回滚点。

```dart
import 'package:sqlite3/sqlite3.dart';

void savepointExample(Database db) {
  db.execute('BEGIN TRANSACTION');

  try {
    // 操作1
    db.execute('INSERT INTO users (name) VALUES (?)', ['User1']);

    // 创建保存点
    db.execute('SAVEPOINT sp1');

    // 操作2（可能失败）
    db.execute('INSERT INTO users (name) VALUES (?)', ['User2']);

    // 如果操作2需要回滚
    if (someCondition) {
      db.execute('ROLLBACK TO SAVEPOINT sp1');
      print('回滚到保存点 sp1');
    }

    // 释放保存点
    db.execute('RELEASE SAVEPOINT sp1');

    // 操作3
    db.execute('INSERT INTO users (name) VALUES (?)', ['User3']);

    db.execute('COMMIT');
  } catch (e) {
    db.execute('ROLLBACK');
    rethrow;
  }
}
```

### 4.5 并发控制

#### 4.5.1 忙等待处理

```dart
import 'package:sqlite3/sqlite3.dart';

void setupBusyTimeout(Database db) {
  // 设置忙等待超时（毫秒）
  db.execute('PRAGMA busy_timeout = 5000');

  // 或使用配置
  db.config.setIntConfig(SQLiteConfig.BUSY, 5000);
}

// 自定义忙处理器
void setupBusyHandler(Database db) {
  db.busyHandler = (int count) {
    print('数据库忙，重试次数: $count');
    // 返回 true 继续等待，false 立即返回错误
    return count < 10;
  };
}
```

---

## 第5章 数据类型与绑定

### 5.1 SQLite 数据类型

SQLite 使用动态类型系统，每个值都有以下五种存储类之一：

| 存储类  | 说明                       |
| ------- | -------------------------- |
| NULL    | 空值                       |
| INTEGER | 有符号整数（1-8 字节）     |
| REAL    | 浮点数（8 字节 IEEE）      |
| TEXT    | 文本字符串（UTF-8/UTF-16） |
| BLOB    | 二进制数据                 |

### 5.2 类型亲和性

SQLite 根据列声明的类型推断亲和性：

```dart
import 'package:sqlite3/sqlite3.dart';

void typeAffinityExample(Database db) {
  db.execute('''
    CREATE TABLE type_test (
      int_col INTEGER,    -- INTEGER 亲和性
      real_col REAL,      -- REAL 亲和性
      text_col TEXT,      -- TEXT 亲和性
      numeric_col NUMERIC,-- NUMERIC 亲和性
      blob_col BLOB       -- BLOB 亲和性（无转换）
    )
  ''');

  // 插入不同类型的值
  db.execute('''
    INSERT INTO type_test VALUES (?, ?, ?, ?, ?)
  ''', [
    42,                    // INTEGER
    3.14,                  // REAL
    'Hello',               // TEXT
    '123',                 // NUMERIC -> 转换为 123
    [0x00, 0x01, 0x02],    // BLOB
  ]);
}
```

### 5.3 BLOB 操作

#### 5.3.1 存储二进制数据

```dart
import 'package:sqlite3/sqlite3.dart';
import 'dart:io';

void storeBlob(Database db, String filePath) async {
  // 读取文件
  final file = File(filePath);
  final bytes = await file.readAsBytes();

  // 存储 BLOB
  db.execute(
    'INSERT INTO files (name, data) VALUES (?, ?)',
    [file.uri.pathSegments.last, bytes],
  );
}
```

#### 5.3.2 读取二进制数据

```dart
import 'package:sqlite3/sqlite3.dart';
import 'dart:io';

void readBlob(Database db, int fileId, String outputPath) async {
  final result = db.select(
    'SELECT name, data FROM files WHERE id = ?',
    [fileId],
  );

  if (result.isNotEmpty) {
    final row = result.first;
    final name = row['name'] as String;
    final data = row['data'] as List<int>;

    // 写入文件
    final file = File('$outputPath/$name');
    await file.writeAsBytes(data);

    print('文件已保存: ${file.path}');
  }
}
```

### 5.4 日期和时间处理

SQLite 没有专门的日期/时间类型，通常使用 TEXT 或 INTEGER 存储。

```dart
import 'package:sqlite3/sqlite3.dart';

void dateTimeExample(Database db) {
  // 创建表
  db.execute('''
    CREATE TABLE events (
      id INTEGER PRIMARY KEY,
      name TEXT,
      event_time DATETIME
    )
  ''');

  // 方式1：使用 ISO-8601 字符串
  db.execute(
    'INSERT INTO events (name, event_time) VALUES (?, ?)',
    ['Meeting', DateTime.now().toIso8601String()],
  );

  // 方式2：使用 Unix 时间戳
  db.execute(
    'INSERT INTO events (name, event_time) VALUES (?, ?)',
    ['Conference', DateTime.now().millisecondsSinceEpoch ~/ 1000],
  );

  // 使用 SQLite 日期函数查询
  final todayEvents = db.select('''
    SELECT * FROM events
    WHERE date(event_time) = date('now')
  ''');

  // 查询最近7天的事件
  final recentEvents = db.select('''
    SELECT * FROM events
    WHERE event_time > datetime('now', '-7 days')
    ORDER BY event_time DESC
  ''');
}
```

**SQLite 日期/时间函数：**

| 函数                          | 说明            |
| ----------------------------- | --------------- |
| `date('now')`                 | 当前日期        |
| `time('now')`                 | 当前时间        |
| `datetime('now')`             | 当前日期时间    |
| `datetime('now', '+7 days')`  | 7天后的日期时间 |
| `strftime('%Y-%m-%d', 'now')` | 格式化日期      |
| `julianday('now')`            | 儒略日          |

### 5.5 JSON 支持

SQLite 3.38.0+ 支持 JSON 函数。

```dart
import 'package:sqlite3/sqlite3.dart';

void jsonExample(Database db) {
  // 创建表存储 JSON
  db.execute('''
    CREATE TABLE json_data (
      id INTEGER PRIMARY KEY,
      data TEXT
    )
  ''');

  // 插入 JSON
  db.execute(
    'INSERT INTO json_data (data) VALUES (?)',
    ['{"name": "Alice", "age": 30, "city": "Beijing"}'],
  );

  // 查询 JSON 字段
  final names = db.select('''
    SELECT json_extract(data, '$.name') as name
    FROM json_data
  ''');

  // 查询满足条件的 JSON
  final adults = db.select('''
    SELECT * FROM json_data
    WHERE json_extract(data, '$.age') > 18
  ''');

  // 更新 JSON 字段
  db.execute('''
    UPDATE json_data
    SET data = json_set(data, '$.age', 31)
    WHERE id = 1
  ''');
}
```

**SQLite JSON 函数：**

| 函数                              | 说明                       |
| --------------------------------- | -------------------------- |
| `json(json)`                      | 验证并格式化 JSON          |
| `json_array(value1, ...)`         | 创建 JSON 数组             |
| `json_object(key1, value1, ...)`  | 创建 JSON 对象             |
| `json_extract(json, path)`        | 提取 JSON 值               |
| `json_insert(json, path, value)`  | 插入 JSON 值               |
| `json_replace(json, path, value)` | 替换 JSON 值               |
| `json_set(json, path, value)`     | 设置 JSON 值（插入或替换） |
| `json_remove(json, path)`         | 删除 JSON 值               |
| `json_type(json, path)`           | 获取 JSON 类型             |
| `json_valid(json)`                | 验证 JSON 格式             |

---

## 第6章 高级功能

### 6.1 用户自定义函数

SQLite 允许注册自定义函数，在 SQL 中使用。

#### 6.1.1 标量函数

```dart
import 'package:sqlite3/sqlite3.dart';

void registerCustomFunction(Database db) {
  // 注册自定义函数
  db.createFunction(
    functionName: 'reverse',
    argumentCount: const AllowedArgumentCount(1),
    function: (args) {
      final text = args[0] as String;
      return text.split('').reversed.join();
    },
  );

  // 使用自定义函数
  final result = db.select("SELECT reverse('Hello') as reversed");
  print(result.first['reversed']);  // 输出: olleH
}

// 注册更多函数
void registerMoreFunctions(Database db) {
  // 计算字符串长度（包含中文）
  db.createFunction(
    functionName: 'char_length',
    argumentCount: const AllowedArgumentCount(1),
    function: (args) {
      final text = args[0] as String;
      return text.length;
    },
  );

  // 格式化货币
  db.createFunction(
    functionName: 'format_currency',
    argumentCount: const AllowedArgumentCount(2),
    function: (args) {
      final amount = args[0] as double;
      final currency = args[1] as String;
      return '$currency ${amount.toStringAsFixed(2)}';
    },
  );
}
```

#### 6.1.2 聚合函数

```dart
import 'package:sqlite3/sqlite3.dart';

void registerAggregateFunction(Database db) {
  // 注册自定义聚合函数：计算中位数
  db.createAggregateFunction<int, double>(
    functionName: 'median',
    argumentCount: const AllowedArgumentCount(1),
    step: (context, args) {
      final value = args[0] as int;
      context ??= <int>[];
      (context as List<int>).add(value);
      return context;
    },
    final: (context) {
      if (context == null || context.isEmpty) return null;

      final sorted = List<int>.from(context)..sort();
      final middle = sorted.length ~/ 2;

      if (sorted.length % 2 == 0) {
        return (sorted[middle - 1] + sorted[middle]) / 2;
      } else {
        return sorted[middle].toDouble();
      }
    },
  );

  // 使用聚合函数
  final result = db.select('SELECT median(age) as median_age FROM users');
  print('中位数年龄: ${result.first['median_age']}');
}
```

### 6.2 虚拟文件系统（VFS）

sqlite3 支持自定义虚拟文件系统。

```dart
import 'package:sqlite3/sqlite3.dart';

void customVFSExample() {
  // 创建自定义 VFS
  final vfs = InMemoryVFS();

  // 注册 VFS
  sqlite3.registerVirtualFileSystem(vfs, makeDefault: false);

  // 使用自定义 VFS 打开数据库
  final db = sqlite3.open(
    'memory:test.db',
    vfs: 'in_memory_vfs',
  );

  // 使用数据库...

  db.dispose();
}
```

### 6.3 数据库加密

使用 SQLite3 Multiple Ciphers 扩展支持数据库加密。

```dart
import 'package:sqlite3/sqlite3.dart';

void encryptedDatabaseExample() {
  // 打开加密数据库
  final db = sqlite3.open('encrypted.db');

  // 设置加密密钥
  db.execute("PRAGMA key = 'my_secret_password'");

  // 验证密钥
  try {
    db.select('SELECT 1');
    print('密钥正确');
  } catch (e) {
    print('密钥错误或数据库未加密');
  }

  // 使用数据库...

  db.dispose();
}

// 修改密钥
void rekeyExample(Database db, String newPassword) {
  db.execute("PRAGMA rekey = '$newPassword'");
}
```

### 6.4 全文搜索（FTS5）

SQLite 支持全文搜索扩展。

```dart
import 'package:sqlite3/sqlite3.dart';

void fts5Example(Database db) {
  // 创建 FTS5 虚拟表
  db.execute('''
    CREATE VIRTUAL TABLE articles USING fts5(
      title,
      content,
      tokenize = 'porter'
    )
  ''');

  // 插入文档
  db.execute(
    'INSERT INTO articles (title, content) VALUES (?, ?)',
    [
      'Flutter 入门',
      'Flutter 是 Google 开发的 UI 工具包，用于构建跨平台应用。',
    ],
  );

  db.execute(
    'INSERT INTO articles (title, content) VALUES (?, ?)',
    [
      'Dart 语言',
      'Dart 是 Google 开发的编程语言，用于构建 Web、移动和桌面应用。',
    ],
  );

  // 全文搜索
  final results = db.select('''
    SELECT * FROM articles
    WHERE articles MATCH 'Flutter'
    ORDER BY rank
  ''');

  for (final row in results) {
    print('${row['title']}: ${row['content']}');
  }

  // 高亮搜索结果
  final highlighted = db.select('''
    SELECT
      highlight(articles, 0, '<b>', '</b>') as highlighted_title,
      highlight(articles, 1, '<b>', '</b>') as highlighted_content
    FROM articles
    WHERE articles MATCH 'Flutter'
  ''');
}
```

### 6.5 性能分析

```dart
import 'package:sqlite3/sqlite3.dart';

void profilingExample(Database db) {
  // 启用查询日志
  db.config.setIntConfig(SQLiteConfig.LOG, 1);

  // 执行查询
  final stopwatch = Stopwatch()..start();

  final result = db.select('SELECT * FROM large_table');

  stopwatch.stop();
  print('查询耗时: ${stopwatch.elapsedMilliseconds}ms');
  print('返回行数: ${result.length}');

  // 分析查询计划
  final plan = db.select('EXPLAIN QUERY PLAN SELECT * FROM users WHERE age > 18');
  for (final row in plan) {
    print('${row['detail']}');
  }
}
```

### 6.6 数据库维护

```dart
import 'package:sqlite3/sqlite3.dart';

void databaseMaintenance(Database db) {
  // 分析表
  db.execute('ANALYZE');

  // 优化数据库
  db.execute('PRAGMA optimize');

  // 清理空闲页
  db.execute('VACUUM');

  // 完整性检查
  final integrity = db.select('PRAGMA integrity_check');
  print('完整性检查结果: ${integrity.first['integrity_check']}');

  // 检查数据库大小
  final pageCount = db.select('PRAGMA page_count').first['page_count'] as int;
  final pageSize = db.select('PRAGMA page_size').first['page_size'] as int;
  final sizeInBytes = pageCount * pageSize;
  print('数据库大小: ${sizeInBytes / 1024 / 1024} MB');
}
```

---

## 附录：完整示例代码

### 示例1：用户管理系统

```dart
import 'package:sqlite3/sqlite3.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

class UserManager {
  late final Database _db;

  Future<void> init() async {
    final documentsDir = await getApplicationDocumentsDirectory();
    final dbPath = p.join(documentsDir.path, 'users.db');

    _db = sqlite3.open(dbPath);

    // 启用外键和 WAL 模式
    _db.execute('PRAGMA foreign_keys = ON');
    _db.execute('PRAGMA journal_mode = WAL');

    // 创建表
    _createTables();

    // 注册自定义函数
    _registerFunctions();
  }

  void _createTables() {
    _db.execute('''
      CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT UNIQUE NOT NULL,
        password_hash TEXT NOT NULL,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )
    ''');

    _db.execute('''
      CREATE TABLE IF NOT EXISTS user_profiles (
        user_id INTEGER PRIMARY KEY,
        full_name TEXT,
        avatar BLOB,
        bio TEXT,
        FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
      )
    ''');

    _db.execute('''
      CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)
    ''');
  }

  void _registerFunctions() {
    _db.createFunction(
      functionName: 'hash_password',
      argumentCount: const AllowedArgumentCount(1),
      function: (args) {
        // 简化的密码哈希（实际应使用 bcrypt 等）
        final password = args[0] as String;
        return password.hashCode.toString();
      },
    );
  }

  int createUser(String username, String email, String password) {
    return _db.lastInsertRowId;
  }

  Map<String, dynamic>? getUser(int id) {
    final result = _db.select(
      'SELECT * FROM users WHERE id = ?',
      [id],
    );

    if (result.isEmpty) return null;

    final row = result.first;
    return {
      'id': row['id'],
      'username': row['username'],
      'email': row['email'],
      'created_at': row['created_at'],
    };
  }

  List<Map<String, dynamic>> getAllUsers() {
    final result = _db.select('SELECT * FROM users ORDER BY created_at DESC');

    return result.map((row) => {
      'id': row['id'],
      'username': row['username'],
      'email': row['email'],
      'created_at': row['created_at'],
    }).toList();
  }

  bool updateUser(int id, Map<String, dynamic> updates) {
    final allowedFields = ['username', 'email'];
    final setClauses = <String>[];
    final values = <dynamic>[];

    for (final entry in updates.entries) {
      if (allowedFields.contains(entry.key)) {
        setClauses.add('${entry.key} = ?');
        values.add(entry.value);
      }
    }

    if (setClauses.isEmpty) return false;

    values.add(id);

    _db.execute(
      'UPDATE users SET ${setClauses.join(', ')}, updated_at = CURRENT_TIMESTAMP WHERE id = ?',
      values,
    );

    return _db.updatedRows > 0;
  }

  bool deleteUser(int id) {
    _db.execute('DELETE FROM users WHERE id = ?', [id]);
    return _db.updatedRows > 0;
  }

  void dispose() {
    _db.dispose();
  }
}
```

### 示例2：日志系统

```dart
import 'package:sqlite3/sqlite3.dart';
import 'dart:convert';

class Logger {
  late final Database _db;

  Logger(Database db) : _db = db {
    _initTable();
  }

  void _initTable() {
    _db.execute('''
      CREATE TABLE IF NOT EXISTS logs (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        level TEXT NOT NULL,
        message TEXT NOT NULL,
        metadata TEXT,
        created_at DATETIME DEFAULT CURRENT_TIMESTAMP
      )
    ''');

    _db.execute('''
      CREATE INDEX IF NOT EXISTS idx_logs_level ON logs(level)
    ''');

    _db.execute('''
      CREATE INDEX IF NOT EXISTS idx_logs_created_at ON logs(created_at)
    ''');
  }

  void log(String level, String message, [Map<String, dynamic>? metadata]) {
    _db.execute(
      'INSERT INTO logs (level, message, metadata) VALUES (?, ?, ?)',
      [level, message, metadata != null ? jsonEncode(metadata) : null],
    );
  }

  void debug(String message, [Map<String, dynamic>? metadata]) {
    log('DEBUG', message, metadata);
  }

  void info(String message, [Map<String, dynamic>? metadata]) {
    log('INFO', message, metadata);
  }

  void warning(String message, [Map<String, dynamic>? metadata]) {
    log('WARNING', message, metadata);
  }

  void error(String message, [Map<String, dynamic>? metadata]) {
    log('ERROR', message, metadata);
  }

  List<Map<String, dynamic>> getLogs({
    String? level,
    DateTime? startTime,
    DateTime? endTime,
    int limit = 100,
  }) {
    var query = 'SELECT * FROM logs WHERE 1=1';
    final params = <dynamic>[];

    if (level != null) {
      query += ' AND level = ?';
      params.add(level);
    }

    if (startTime != null) {
      query += ' AND created_at >= ?';
      params.add(startTime.toIso8601String());
    }

    if (endTime != null) {
      query += ' AND created_at <= ?';
      params.add(endTime.toIso8601String());
    }

    query += ' ORDER BY created_at DESC LIMIT ?';
    params.add(limit);

    final result = _db.select(query, params);

    return result.map((row) => {
      'id': row['id'],
      'level': row['level'],
      'message': row['message'],
      'metadata': row['metadata'] != null
          ? jsonDecode(row['metadata'] as String)
          : null,
      'created_at': row['created_at'],
    }).toList();
  }

  void clearOldLogs(int daysToKeep) {
    _db.execute('''
      DELETE FROM logs
      WHERE created_at < datetime('now', '-$daysToKeep days')
    ''');
  }
}
```

---

## 结语

sqlite3 3.1.6 为 Dart 和 Flutter 开发者提供了强大而灵活的 SQLite 访问能力。通过本书的学习，你应该已经掌握了：

1. **基础配置**：如何安装和配置 sqlite3
2. **数据库操作**：如何打开、配置和关闭数据库
3. **预编译语句**：如何使用 PreparedStatement 提高性能和安全性
4. **事务处理**：如何控制事务和保证数据一致性
5. **数据类型**：如何处理各种数据类型和 BLOB
6. **高级功能**：如何使用自定义函数、FTS5、加密等

sqlite3 的设计理念是提供最接近原生 SQLite 的 API，因此在使用时需要：

- 注意资源管理，及时调用 `dispose()`
- 使用参数绑定防止 SQL 注入
- 合理使用事务保证数据一致性
- 根据场景选择合适的 PRAGMA 配置
- 利用预编译语句提高性能

希望本书能帮助你更好地使用 sqlite3 构建高性能的 Flutter 应用！
