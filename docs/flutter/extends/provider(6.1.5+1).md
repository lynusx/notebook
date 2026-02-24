# Flutter Provider 6.1.5+1 完整参考手册

## package:provider

> **版本**: 6.1.5+1  
> **适用范围**: Flutter 状态管理  
> **官方文档**: [pub.dev/packages/provider](https://pub.dev/packages/provider)  
> **GitHub**: [github.com/rrousselGit/provider](https://github.com/rrousselGit/provider)

---

## 第一章：库概述

Provider 是 Flutter 中最流行的状态管理库之一，由 Remi Rousselet 创建。它基于 InheritedWidget 构建，提供了一种简单、可扩展的方式来管理应用状态并在 Widget 树中共享数据。

### 1.1 为什么选择 Provider

| 特性         | 说明                           |
| ------------ | ------------------------------ |
| **简单易用** | API 设计直观，学习曲线平缓     |
| **类型安全** | 编译时类型检查，避免运行时错误 |
| **可测试**   | 易于模拟和测试                 |
| **性能优化** | 只重建依赖特定状态的 Widget    |
| **灵活性**   | 支持多种状态管理方式           |
| **社区活跃** | 文档丰富，生态完善             |

### 1.2 Provider 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                    Provider 架构图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Widget Tree                           │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │              Provider (InheritedWidget)          │   │   │
│  │  │                                                 │   │   │
│  │  │  ┌─────────────────────────────────────────┐   │   │   │
│  │  │  │           Consumer/Selector              │   │   │   │
│  │  │  │                                          │   │   │   │
│  │  │  │  ┌─────────────────────────────────┐   │   │   │   │
│  │  │  │  │         Child Widgets            │   │   │   │   │
│  │  │  │  │                                  │   │   │   │   │
│  │  │  │  │  • 通过 context.watch() 监听     │   │   │   │   │
│  │  │  │  │  • 通过 context.read() 读取      │   │   │   │   │
│  │  │  │  │  • 通过 Provider.of() 访问       │   │   │   │   │
│  │  │  │  └─────────────────────────────────┘   │   │   │   │
│  │  │  └─────────────────────────────────────────┘   │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      State Object                        │   │
│  │  • ChangeNotifier                                        │   │
│  │  • ValueNotifier                                         │   │
│  │  • StateNotifier                                         │   │
│  │  • 任意对象                                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 导入方式

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.5+1
```

```dart
import 'package:provider/provider.dart';
```

### 1.4 Provider 类型概览

| Provider 类型             | 用途                | 适用场景         |
| ------------------------- | ------------------- | ---------------- |
| `Provider`                | 提供不可变对象      | 配置、主题、服务 |
| `ChangeNotifierProvider`  | 提供 ChangeNotifier | 复杂状态管理     |
| `ValueListenableProvider` | 提供 ValueNotifier  | 简单值监听       |
| `StreamProvider`          | 提供 Stream         | 实时数据流       |
| `FutureProvider`          | 提供 Future         | 异步数据加载     |
| `MultiProvider`           | 组合多个 Provider   | 大型应用         |
| `ProxyProvider`           | 依赖其他 Provider   | 派生状态         |

> **补充知识**：Provider 6.x 版本与 Flutter 3.x 完全兼容，支持新的 `context.watch()` 和 `context.read()` 扩展方法，使代码更加简洁。

---

## 第二章：Provider 基础

### 2.1 Provider

`Provider` 是最基础的 Provider，用于提供不可变对象或服务。

#### 构造函数

```dart
Provider({
  Key? key,
  required Create<T> create,
  Dispose<T>? dispose,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 参数说明

| 参数名    | 类型                 | 说明                      |
| --------- | -------------------- | ------------------------- |
| `create`  | `Create<T>`          | 创建对象的回调函数        |
| `dispose` | `Dispose<T>?`        | 释放资源的回调函数        |
| `lazy`    | `bool?`              | 是否延迟创建（默认 true） |
| `builder` | `TransitionBuilder?` | 构建子 Widget 的回调      |
| `child`   | `Widget?`            | 子 Widget                 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 配置类（不可变）
class AppConfig {
  final String appName;
  final String apiBaseUrl;
  final int apiVersion;

  const AppConfig({
    required this.appName,
    required this.apiBaseUrl,
    required this.apiVersion,
  });
}

// 服务类
class LoggerService {
  void log(String message) {
    debugPrint('[LOG] $message');
  }

  void error(String message) {
    debugPrint('[ERROR] $message');
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        // 提供配置
        Provider<AppConfig>(
          create: (_) => const AppConfig(
            appName: 'My App',
            apiBaseUrl: 'https://api.example.com',
            apiVersion: 1,
          ),
        ),
        // 提供服务
        Provider<LoggerService>(
          create: (_) => LoggerService(),
          dispose: (_, service) {
            // 清理资源
          },
        ),
      ],
      child: MaterialApp(
        title: 'Provider Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const ProviderDemoPage(),
      ),
    );
  }
}

class ProviderDemoPage extends StatelessWidget {
  const ProviderDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    // 读取 Provider 的值
    final config = context.read<AppConfig>();
    final logger = context.read<LoggerService>();

    logger.log('Building ProviderDemoPage');

    return Scaffold(
      appBar: AppBar(
        title: Text(config.appName),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('App Name: ${config.appName}'),
            Text('API Base URL: ${config.apiBaseUrl}'),
            Text('API Version: ${config.apiVersion}'),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                logger.log('Button clicked');
              },
              child: const Text('Log Message'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 2.2 ChangeNotifierProvider

`ChangeNotifierProvider` 是最常用的 Provider，用于提供 `ChangeNotifier` 对象，当状态变化时自动重建依赖的 Widget。

#### 构造函数

```dart
ChangeNotifierProvider({
  Key? key,
  required Create<T> create,
  Dispose<T>? dispose,
  bool? lazy,
  TransitionBuilder? builder,
  Widget? child,
});

// 从已有实例创建
ChangeNotifierProvider.value({
  Key? key,
  required T value,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 购物车状态管理
class CartModel extends ChangeNotifier {
  final List<CartItem> _items = [];

  List<CartItem> get items => List.unmodifiable(_items);
  int get itemCount => _items.length;
  double get totalPrice => _items.fold(
        0,
        (sum, item) => sum + item.price * item.quantity,
      );

  void addItem(String name, double price) {
    final existingIndex = _items.indexWhere((item) => item.name == name);

    if (existingIndex >= 0) {
      _items[existingIndex].quantity++;
    } else {
      _items.add(CartItem(name: name, price: price));
    }

    notifyListeners();
  }

  void removeItem(String name) {
    _items.removeWhere((item) => item.name == name);
    notifyListeners();
  }

  void updateQuantity(String name, int quantity) {
    final index = _items.indexWhere((item) => item.name == name);
    if (index >= 0) {
      if (quantity <= 0) {
        _items.removeAt(index);
      } else {
        _items[index].quantity = quantity;
      }
      notifyListeners();
    }
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}

class CartItem {
  final String name;
  final double price;
  int quantity;

  CartItem({
    required this.name,
    required this.price,
    this.quantity = 1,
  });
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CartModel(),
      child: MaterialApp(
        title: 'ChangeNotifierProvider Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const CartPage(),
      ),
    );
  }
}

class CartPage extends StatelessWidget {
  const CartPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('购物车'),
        actions: [
          // 使用 Consumer 只重建购物车图标
          Consumer<CartModel>(
            builder: (context, cart, child) {
              return Badge(
                label: Text('${cart.itemCount}'),
                child: const Icon(Icons.shopping_cart),
              );
            },
          ),
          const SizedBox(width: 16),
        ],
      ),
      body: Column(
        children: [
          // 商品列表
          Expanded(
            child: ListView(
              padding: const EdgeInsets.all(16),
              children: [
                _buildProductCard(context, 'iPhone 15', 9999),
                _buildProductCard(context, 'MacBook Pro', 14999),
                _buildProductCard(context, 'AirPods Pro', 1999),
                _buildProductCard(context, 'iPad Air', 4999),
              ],
            ),
          ),

          // 购物车摘要
          Consumer<CartModel>(
            builder: (context, cart, child) {
              return Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.grey[100],
                  borderRadius: const BorderRadius.vertical(
                    top: Radius.circular(16),
                  ),
                ),
                child: SafeArea(
                  child: Column(
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text('商品数量: ${cart.itemCount}'),
                          Text(
                            '总计: ¥${cart.totalPrice.toStringAsFixed(2)}',
                            style: const TextStyle(
                              fontSize: 18,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 16),
                      if (cart.items.isNotEmpty)
                        SizedBox(
                          width: double.infinity,
                          child: ElevatedButton(
                            onPressed: () {
                              cart.clear();
                              ScaffoldMessenger.of(context).showSnackBar(
                                const SnackBar(content: Text('购物车已清空')),
                              );
                            },
                            child: const Text('清空购物车'),
                          ),
                        ),
                    ],
                  ),
                ),
              );
            },
          ),
        ],
      ),
    );
  }

  Widget _buildProductCard(BuildContext context, String name, double price) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        title: Text(name),
        subtitle: Text('¥${price.toStringAsFixed(2)}'),
        trailing: ElevatedButton(
          onPressed: () {
            context.read<CartModel>().addItem(name, price);
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('$name 已添加到购物车')),
            );
          },
          child: const Text('添加'),
        ),
      ),
    );
  }
}
```

### 2.3 ValueListenableProvider

`ValueListenableProvider` 用于提供 `ValueListenable` 对象（如 `ValueNotifier`），当值变化时自动重建依赖的 Widget。

#### 构造函数

```dart
ValueListenableProvider({
  Key? key,
  required Create<ValueListenable<T>> create,
  Dispose<ValueListenable<T>>? dispose,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ValueListenableProvider Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const CounterPage(),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return ValueListenableProvider<ValueNotifier<int>>(
      create: (_) => ValueNotifier<int>(0),
      child: Scaffold(
        appBar: AppBar(
          title: const Text('ValueListenableProvider'),
        ),
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              CounterDisplay(),
              SizedBox(height: 24),
              CounterControls(),
            ],
          ),
        ),
      ),
    );
  }
}

class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // 监听 ValueNotifier 的值
    final counter = context.watch<ValueNotifier<int>>();

    return Text(
      '${counter.value}',
      style: const TextStyle(fontSize: 72, fontWeight: FontWeight.bold),
    );
  }
}

class CounterControls extends StatelessWidget {
  const CounterControls({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = context.read<ValueNotifier<int>>();

    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        ElevatedButton(
          onPressed: () => counter.value--,
          child: const Icon(Icons.remove),
        ),
        const SizedBox(width: 16),
        ElevatedButton(
          onPressed: () => counter.value = 0,
          child: const Text('重置'),
        ),
        const SizedBox(width: 16),
        ElevatedButton(
          onPressed: () => counter.value++,
          child: const Icon(Icons.add),
        ),
      ],
    );
  }
}
```

---

## 第三章：异步 Provider

### 3.1 FutureProvider

`FutureProvider` 用于异步加载数据，它会自动处理 Future 的完成状态，并在数据加载完成后重建依赖的 Widget。

#### 构造函数

```dart
FutureProvider({
  Key? key,
  required Create<Future<T>> create,
  T? initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  TransitionBuilder? builder,
  Widget? child,
});

// 从已有 Future 创建
FutureProvider.value({
  Key? key,
  required Future<T> value,
  T? initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 参数说明

| 参数名               | 类型                     | 说明                           |
| -------------------- | ------------------------ | ------------------------------ |
| `create`             | `Create<Future<T>>`      | 创建 Future 的回调函数         |
| `initialData`        | `T?`                     | 初始数据，在 Future 完成前使用 |
| `catchError`         | `ErrorBuilder<T>?`       | 处理错误的回调函数             |
| `updateShouldNotify` | `UpdateShouldNotify<T>?` | 自定义更新通知逻辑             |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 用户模型
class User {
  final String id;
  final String name;
  final String email;
  final String avatarUrl;

  User({
    required this.id,
    required this.name,
    required this.email,
    required this.avatarUrl,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      avatarUrl: json['avatarUrl'],
    );
  }
}

// 用户服务
class UserService {
  Future<User> fetchUser(String userId) async {
    // 模拟网络请求
    await Future.delayed(const Duration(seconds: 2));

    // 模拟随机失败
    if (userId == 'error') {
      throw Exception('用户不存在');
    }

    return User(
      id: userId,
      name: '张三',
      email: 'zhangsan@example.com',
      avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=$userId',
    );
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider<UserService>(
      create: (_) => UserService(),
      child: MaterialApp(
        title: 'FutureProvider Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const UserProfilePage(userId: 'user123'),
      ),
    );
  }
}

class UserProfilePage extends StatelessWidget {
  final String userId;

  const UserProfilePage({
    super.key,
    required this.userId,
  });

  @override
  Widget build(BuildContext context) {
    return FutureProvider<User?>(
      initialData: null,
      create: (context) => context.read<UserService>().fetchUser(userId),
      catchError: (context, error) {
        debugPrint('Error loading user: $error');
        return null;
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('用户资料'),
        ),
        body: const UserProfileContent(),
      ),
    );
  }
}

class UserProfileContent extends StatelessWidget {
  const UserProfileContent({super.key});

  @override
  Widget build(BuildContext context) {
    final user = context.watch<User?>();

    if (user == null) {
      return const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircularProgressIndicator(),
            SizedBox(height: 16),
            Text('加载中...'),
          ],
        ),
      );
    }

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircleAvatar(
            radius: 60,
            backgroundImage: NetworkImage(user.avatarUrl),
          ),
          const SizedBox(height: 24),
          Text(
            user.name,
            style: const TextStyle(
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            user.email,
            style: TextStyle(
              fontSize: 16,
              color: Colors.grey[600],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 3.2 StreamProvider

`StreamProvider` 用于监听 Stream，适用于实时数据更新场景，如 WebSocket、Firebase 实时数据库等。

#### 构造函数

```dart
StreamProvider({
  Key? key,
  required Create<Stream<T>> create,
  T? initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  TransitionBuilder? builder,
  Widget? child,
});

// 从已有 Stream 创建
StreamProvider.value({
  Key? key,
  required Stream<T> value,
  T? initialData,
  ErrorBuilder<T>? catchError,
  UpdateShouldNotify<T>? updateShouldNotify,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 完整示例代码

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 实时数据服务
class RealtimeService {
  Stream<int> getCounterStream() {
    return Stream.periodic(
      const Duration(seconds: 1),
      (count) => count,
    ).take(60);
  }

  Stream<List<String>> getMessageStream() {
    final messages = [
      '系统启动',
      '连接服务器...',
      '连接成功',
      '接收数据中...',
      '数据同步完成',
    ];

    return Stream.periodic(
      const Duration(seconds: 2),
      (index) => messages.sublist(0, (index % messages.length) + 1),
    ).take(100);
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider<RealtimeService>(
      create: (_) => RealtimeService(),
      child: MaterialApp(
        title: 'StreamProvider Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const RealtimeDashboard(),
      ),
    );
  }
}

class RealtimeDashboard extends StatelessWidget {
  const RealtimeDashboard({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        StreamProvider<int>(
          initialData: 0,
          create: (context) => context.read<RealtimeService>().getCounterStream(),
        ),
        StreamProvider<List<String>>(
          initialData: const [],
          create: (context) => context.read<RealtimeService>().getMessageStream(),
        ),
      ],
      child: Scaffold(
        appBar: AppBar(
          title: const Text('实时监控面板'),
        ),
        body: const Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              CounterCard(),
              SizedBox(height: 16),
              MessageLogCard(),
            ],
          ),
        ),
      ),
    );
  }
}

class CounterCard extends StatelessWidget {
  const CounterCard({super.key});

  @override
  Widget build(BuildContext context) {
    final counter = context.watch<int>();

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            const Text(
              '运行时间',
              style: TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(
              '$counter 秒',
              style: const TextStyle(
                fontSize: 48,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class MessageLogCard extends StatelessWidget {
  const MessageLogCard({super.key});

  @override
  Widget build(BuildContext context) {
    final messages = context.watch<List<String>>();

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '系统日志',
              style: TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Container(
              height: 200,
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: ListView.builder(
                padding: const EdgeInsets.all(8),
                itemCount: messages.length,
                itemBuilder: (context, index) {
                  return Padding(
                    padding: const EdgeInsets.symmetric(vertical: 4),
                    child: Row(
                      children: [
                        Icon(
                          Icons.circle,
                          size: 8,
                          color: index == messages.length - 1
                              ? Colors.green
                              : Colors.grey,
                        ),
                        const SizedBox(width: 8),
                        Text(
                          '[${DateTime.now().toString().substring(11, 19)}] ${messages[index]}',
                          style: TextStyle(
                            fontSize: 12,
                            color: index == messages.length - 1
                                ? Colors.black
                                : Colors.grey[600],
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 第四章：Consumer 和 Selector

### 4.1 Consumer

`Consumer` 是一个 Widget，用于监听 Provider 的变化并重建部分 UI。与直接使用 `context.watch()` 相比，Consumer 可以更精确地控制哪些 Widget 需要重建。

#### 构造函数

```dart
Consumer<T>({
  Key? key,
  required Widget Function(BuildContext context, T value, Widget? child) builder,
  Widget? child,
});
```

#### 参数说明

| 参数名    | 类型                                        | 说明                                |
| --------- | ------------------------------------------- | ----------------------------------- |
| `builder` | `Widget Function(BuildContext, T, Widget?)` | 构建 Widget 的回调函数              |
| `child`   | `Widget?`                                   | 可选的子 Widget，不会随状态变化重建 |

#### Consumer 的优势

```
┌─────────────────────────────────────────────────────────────┐
│                     Consumer 优势                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 精确控制重建范围                                        │
│     ┌─────────────────────────────────────────────────┐    │
│     │  Page (不重建)                                   │    │
│     │  ┌─────────────────────────────────────────┐   │    │
│     │  │  Consumer<Counter> (监听状态)            │   │    │
│     │  │  ┌─────────────────────────────────┐   │   │    │
│     │  │  │  Text (重建)                     │   │   │    │
│     │  │  └─────────────────────────────────┘   │   │    │
│     │  └─────────────────────────────────────────┘   │    │
│     │  ┌─────────────────────────────────────────┐   │    │
│     │  │  StaticWidget (不重建)                   │   │    │
│     │  └─────────────────────────────────────────┘   │    │
│     └─────────────────────────────────────────────────┘    │
│                                                             │
│  2. 优化性能 - child 参数                                   │
│     将不需要重建的 Widget 作为 child 传入，提高性能          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 应用状态
class AppState extends ChangeNotifier {
  int _counter = 0;
  String _username = 'User';
  ThemeMode _themeMode = ThemeMode.light;

  int get counter => _counter;
  String get username => _username;
  ThemeMode get themeMode => _themeMode;

  void increment() {
    _counter++;
    notifyListeners();
  }

  void decrement() {
    _counter--;
    notifyListeners();
  }

  void setUsername(String name) {
    _username = name;
    notifyListeners();
  }

  void toggleTheme() {
    _themeMode = _themeMode == ThemeMode.light
        ? ThemeMode.dark
        : ThemeMode.light;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => AppState(),
      child: Consumer<AppState>(
        builder: (context, state, child) {
          return MaterialApp(
            title: 'Consumer Demo',
            themeMode: state.themeMode,
            theme: ThemeData.light(),
            darkTheme: ThemeData.dark(),
            home: const HomePage(),
          );
        },
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    print('HomePage building');

    return Scaffold(
      appBar: AppBar(
        title: const Text('Consumer 演示'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 使用 Consumer 只重建计数器部分
            const CounterSection(),
            const SizedBox(height: 24),
            // 使用 Consumer 只重建用户名部分
            const UsernameSection(),
            const SizedBox(height: 24),
            // 主题切换按钮
            const ThemeToggleSection(),
            const SizedBox(height: 24),
            // 静态内容 - 不会重建
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text(
                '这部分是静态内容，不会随状态变化重建',
                style: TextStyle(fontSize: 14),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class CounterSection extends StatelessWidget {
  const CounterSection({super.key});

  @override
  Widget build(BuildContext context) {
    print('CounterSection building');

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '计数器',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 16),
            // 使用 Consumer 监听 counter 变化
            Consumer<AppState>(
              builder: (context, state, child) {
                print('Counter Consumer building');
                return Text(
                  '${state.counter}',
                  style: const TextStyle(
                    fontSize: 48,
                    fontWeight: FontWeight.bold,
                  ),
                );
              },
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                ElevatedButton(
                  onPressed: () => context.read<AppState>().decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: () => context.read<AppState>().increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

class UsernameSection extends StatelessWidget {
  const UsernameSection({super.key});

  @override
  Widget build(BuildContext context) {
    print('UsernameSection building');

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '用户名',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 16),
            Consumer<AppState>(
              builder: (context, state, child) {
                print('Username Consumer building');
                return Text(
                  '当前用户: ${state.username}',
                  style: const TextStyle(fontSize: 16),
                );
              },
            ),
            const SizedBox(height: 16),
            TextField(
              decoration: const InputDecoration(
                labelText: '输入新用户名',
                border: OutlineInputBorder(),
              ),
              onSubmitted: (value) {
                context.read<AppState>().setUsername(value);
              },
            ),
          ],
        ),
      ),
    );
  }
}

class ThemeToggleSection extends StatelessWidget {
  const ThemeToggleSection({super.key});

  @override
  Widget build(BuildContext context) {
    print('ThemeToggleSection building');

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            const Text(
              '主题模式',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            Consumer<AppState>(
              builder: (context, state, child) {
                print('Theme Consumer building');
                return Switch(
                  value: state.themeMode == ThemeMode.dark,
                  onChanged: (_) => context.read<AppState>().toggleTheme(),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Consumer 的 child 参数优化

```dart
// 未优化 - 每次重建都会创建新的 ExpensiveWidget
Consumer<AppState>(
  builder: (context, state, child) {
    return Column(
      children: [
        Text('Counter: ${state.counter}'), // 需要重建
        ExpensiveWidget(), // 不需要重建，但每次都重新创建
      ],
    );
  },
)

// 优化后 - ExpensiveWidget 只创建一次
Consumer<AppState>(
  builder: (context, state, child) {
    return Column(
      children: [
        Text('Counter: ${state.counter}'), // 需要重建
        child!, // 使用预创建的 Widget，不会重建
      ],
    );
  },
  child: const ExpensiveWidget(), // 只创建一次
)
```

### 4.2 Selector

`Selector` 是 Consumer 的高级版本，它可以选择性地监听对象的部分属性，只有当选中的属性变化时才重建 Widget，进一步优化性能。

#### 构造函数

```dart
Selector<T, S>({
  Key? key,
  required S Function(BuildContext context, T value) selector,
  required Widget Function(BuildContext context, S value, Widget? child) builder,
  bool Function(S previous, S next)? shouldRebuild,
  Widget? child,
});
```

#### 参数说明

| 参数名          | 类型                                        | 说明               |
| --------------- | ------------------------------------------- | ------------------ |
| `selector`      | `S Function(BuildContext, T)`               | 选择要监听的属性   |
| `builder`       | `Widget Function(BuildContext, S, Widget?)` | 构建 Widget 的回调 |
| `shouldRebuild` | `bool Function(S, S)?`                      | 自定义重建判断逻辑 |
| `child`         | `Widget?`                                   | 可选的子 Widget    |

#### Selector 工作原理

```
┌─────────────────────────────────────────────────────────────────┐
│                     Selector 工作原理                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AppState (包含多个属性)                                         │
│  ├── counter: int                                               │
│  ├── username: String                                           │
│  └── themeMode: ThemeMode                                       │
│                                                                 │
│  Selector<AppState, int>                                        │
│  ├── selector: (state) => state.counter  ← 只选择 counter       │
│  ├── 只有 counter 变化时才重建                                   │
│  └── username/themeMode 变化时不重建                             │
│                                                                 │
│  Selector<AppState, String>                                     │
│  ├── selector: (state) => state.username  ← 只选择 username     │
│  ├── 只有 username 变化时才重建                                  │
│  └── counter/themeMode 变化时不重建                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// 复杂应用状态
class ComplexAppState extends ChangeNotifier {
  int _counter = 0;
  String _username = 'User';
  int _age = 25;
  List<String> _tags = [];
  bool _isLoading = false;

  // Getters
  int get counter => _counter;
  String get username => _username;
  int get age => _age;
  List<String> get tags => List.unmodifiable(_tags);
  bool get isLoading => _isLoading;

  // 计数器操作
  void increment() {
    _counter++;
    notifyListeners();
  }

  // 用户信息操作
  void updateUserInfo({String? name, int? age}) {
    if (name != null) _username = name;
    if (age != null) _age = age;
    notifyListeners();
  }

  // 标签操作
  void addTag(String tag) {
    _tags.add(tag);
    notifyListeners();
  }

  void removeTag(String tag) {
    _tags.remove(tag);
    notifyListeners();
  }

  // 加载状态
  void setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => ComplexAppState(),
      child: MaterialApp(
        title: 'Selector Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const SelectorDemoPage(),
      ),
    );
  }
}

class SelectorDemoPage extends StatelessWidget {
  const SelectorDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    print('SelectorDemoPage building');

    return Scaffold(
      appBar: AppBar(
        title: const Text('Selector 性能优化演示'),
      ),
      body: const SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            // 只监听 counter
            CounterSelectorCard(),
            SizedBox(height: 16),
            // 只监听 username
            UsernameSelectorCard(),
            SizedBox(height: 16),
            // 只监听 age
            AgeSelectorCard(),
            SizedBox(height: 16),
            // 监听多个属性
            UserSummarySelectorCard(),
            SizedBox(height: 16),
            // 使用自定义 shouldRebuild
            TagsSelectorCard(),
            SizedBox(height: 16),
            // 控制面板
            ControlPanel(),
          ],
        ),
      ),
    );
  }
}

class CounterSelectorCard extends StatelessWidget {
  const CounterSelectorCard({super.key});

  @override
  Widget build(BuildContext context) {
    print('CounterSelectorCard building');

    return Card(
      color: Colors.blue[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Counter Selector',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const Text(
              '只监听 counter 变化',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            // 只选择 counter 属性
            Selector<ComplexAppState, int>(
              selector: (context, state) => state.counter,
              builder: (context, counter, child) {
                print('  Counter Selector builder called');
                return Text(
                  'Counter: $counter',
                  style: const TextStyle(
                    fontSize: 32,
                    fontWeight: FontWeight.bold,
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

class UsernameSelectorCard extends StatelessWidget {
  const UsernameSelectorCard({super.key});

  @override
  Widget build(BuildContext context) {
    print('UsernameSelectorCard building');

    return Card(
      color: Colors.green[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Username Selector',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const Text(
              '只监听 username 变化',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            Selector<ComplexAppState, String>(
              selector: (context, state) => state.username,
              builder: (context, username, child) {
                print('  Username Selector builder called');
                return Text(
                  'Username: $username',
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

class AgeSelectorCard extends StatelessWidget {
  const AgeSelectorCard({super.key});

  @override
  Widget build(BuildContext context) {
    print('AgeSelectorCard building');

    return Card(
      color: Colors.orange[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Age Selector',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const Text(
              '只监听 age 变化',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            Selector<ComplexAppState, int>(
              selector: (context, state) => state.age,
              builder: (context, age, child) {
                print('  Age Selector builder called');
                return Text(
                  'Age: $age',
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

// 监听多个属性 - 使用 Record (Dart 3.0+)
class UserSummarySelectorCard extends StatelessWidget {
  const UserSummarySelectorCard({super.key});

  @override
  Widget build(BuildContext context) {
    print('UserSummarySelectorCard building');

    return Card(
      color: Colors.purple[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Multi-Property Selector',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const Text(
              '监听 username 和 age 两个属性',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            // 使用 Record 选择多个属性
            Selector<ComplexAppState, (String, int)>(
              selector: (context, state) => (state.username, state.age),
              builder: (context, data, child) {
                print('  UserSummary Selector builder called');
                final (username, age) = data;
                return Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'User: $username',
                      style: const TextStyle(fontSize: 20),
                    ),
                    Text(
                      'Age: $age',
                      style: const TextStyle(fontSize: 20),
                    ),
                  ],
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

// 使用自定义 shouldRebuild
class TagsSelectorCard extends StatelessWidget {
  const TagsSelectorCard({super.key});

  @override
  Widget build(BuildContext context) {
    print('TagsSelectorCard building');

    return Card(
      color: Colors.red[50],
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'Tags Selector (Custom shouldRebuild)',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const Text(
              '只在标签数量变化时重建',
              style: TextStyle(fontSize: 12, color: Colors.grey),
            ),
            const SizedBox(height: 8),
            Selector<ComplexAppState, List<String>>(
              selector: (context, state) => state.tags,
              // 自定义重建逻辑：只在标签数量变化时重建
              shouldRebuild: (previous, next) {
                return previous.length != next.length;
              },
              builder: (context, tags, child) {
                print('  Tags Selector builder called');
                return Wrap(
                  spacing: 8,
                  children: tags
                      .map((tag) => Chip(
                            label: Text(tag),
                            onDeleted: () {
                              context.read<ComplexAppState>().removeTag(tag);
                            },
                          ))
                      .toList(),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

class ControlPanel extends StatelessWidget {
  const ControlPanel({super.key});

  @override
  Widget build(BuildContext context) {
    print('ControlPanel building');

    final state = context.read<ComplexAppState>();

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '控制面板',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () => state.increment(),
                  child: const Text('+ Counter'),
                ),
                ElevatedButton(
                  onPressed: () => state.updateUserInfo(name: 'User${DateTime.now().second}'),
                  child: const Text('Change Username'),
                ),
                ElevatedButton(
                  onPressed: () => state.updateUserInfo(age: 20 + DateTime.now().second),
                  child: const Text('Change Age'),
                ),
                ElevatedButton(
                  onPressed: () => state.addTag('Tag ${DateTime.now().second}'),
                  child: const Text('Add Tag'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 第五章：MultiProvider

### 5.1 MultiProvider 概述

`MultiProvider` 用于组合多个 Provider，使代码更加简洁和可维护。当应用需要多个 Provider 时，使用 MultiProvider 比嵌套多个 Provider 更清晰。

#### 嵌套 Provider vs MultiProvider

```
┌─────────────────────────────────────────────────────────────────┐
│                   Provider 对比                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  嵌套方式（不推荐）                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Provider<A>(                                             │   │
│  │   create: (_) => A(),                                   │   │
│  │   child: Provider<B>(                                   │   │
│  │     create: (_) => B(),                                 │   │
│  │     child: Provider<C>(                                 │   │
│  │       create: (_) => C(),                               │   │
│  │       child: MyApp(),                                   │   │
│  │     ),                                                  │   │
│  │   ),                                                    │   │
│  │ )                                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  MultiProvider 方式（推荐）                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ MultiProvider(                                          │   │
│  │   providers: [                                          │   │
│  │     Provider<A>(create: (_) => A()),                    │   │
│  │     Provider<B>(create: (_) => B()),                    │   │
│  │     Provider<C>(create: (_) => C()),                    │   │
│  │   ],                                                    │   │
│  │   child: MyApp(),                                       │   │
│  │ )                                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 构造函数

```dart
MultiProvider({
  Key? key,
  required List<SingleChildWidget> providers,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// ============ 各种服务和状态 ============

// 1. 应用配置服务
class AppConfig {
  final String appName;
  final String apiBaseUrl;
  final bool enableLogging;

  const AppConfig({
    required this.appName,
    required this.apiBaseUrl,
    this.enableLogging = true,
  });
}

// 2. 日志服务
class LoggerService {
  final AppConfig _config;

  LoggerService(this._config);

  void log(String message) {
    if (_config.enableLogging) {
      debugPrint('[LOG] ${DateTime.now()}: $message');
    }
  }

  void error(String message) {
    debugPrint('[ERROR] ${DateTime.now()}: $message');
  }
}

// 3. 认证状态
class AuthState extends ChangeNotifier {
  bool _isAuthenticated = false;
  String? _userId;
  String? _username;

  bool get isAuthenticated => _isAuthenticated;
  String? get userId => _userId;
  String? get username => _username;

  void login(String userId, String username) {
    _isAuthenticated = true;
    _userId = userId;
    _username = username;
    notifyListeners();
  }

  void logout() {
    _isAuthenticated = false;
    _userId = null;
    _username = null;
    notifyListeners();
  }
}

// 4. 主题状态
class ThemeState extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;
  Color _primaryColor = Colors.blue;

  ThemeMode get themeMode => _themeMode;
  Color get primaryColor => _primaryColor;

  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    notifyListeners();
  }

  void setPrimaryColor(Color color) {
    _primaryColor = color;
    notifyListeners();
  }

  ThemeData get lightTheme {
    return ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: _primaryColor,
        brightness: Brightness.light,
      ),
      useMaterial3: true,
    );
  }

  ThemeData get darkTheme {
    return ThemeData(
      colorScheme: ColorScheme.fromSeed(
        seedColor: _primaryColor,
        brightness: Brightness.dark,
      ),
      useMaterial3: true,
    );
  }
}

// 5. 购物车状态
class CartState extends ChangeNotifier {
  final List<CartItem> _items = [];

  List<CartItem> get items => List.unmodifiable(_items);
  int get itemCount => _items.length;
  double get totalPrice => _items.fold(
        0,
        (sum, item) => sum + item.price * item.quantity,
      );

  void addItem(String id, String name, double price) {
    final existingIndex = _items.indexWhere((item) => item.id == id);
    if (existingIndex >= 0) {
      _items[existingIndex].quantity++;
    } else {
      _items.add(CartItem(id: id, name: name, price: price));
    }
    notifyListeners();
  }

  void removeItem(String id) {
    _items.removeWhere((item) => item.id == id);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}

class CartItem {
  final String id;
  final String name;
  final double price;
  int quantity;

  CartItem({
    required this.id,
    required this.name,
    required this.price,
    this.quantity = 1,
  });
}

// ============ 应用入口 ============

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        // 1. 提供配置
        Provider<AppConfig>(
          create: (_) => const AppConfig(
            appName: 'MultiProvider Demo',
            apiBaseUrl: 'https://api.example.com',
            enableLogging: true,
          ),
        ),
        // 2. 日志服务（依赖 AppConfig）
        ProxyProvider<AppConfig, LoggerService>(
          update: (_, config, __) => LoggerService(config),
        ),
        // 3. 认证状态
        ChangeNotifierProvider<AuthState>(
          create: (_) => AuthState(),
        ),
        // 4. 主题状态
        ChangeNotifierProvider<ThemeState>(
          create: (_) => ThemeState(),
        ),
        // 5. 购物车状态
        ChangeNotifierProvider<CartState>(
          create: (_) => CartState(),
        ),
      ],
      child: const AppContent(),
    );
  }
}

class AppContent extends StatelessWidget {
  const AppContent({super.key});

  @override
  Widget build(BuildContext context) {
    final config = context.read<AppConfig>();
    final themeState = context.watch<ThemeState>();

    return MaterialApp(
      title: config.appName,
      themeMode: themeState.themeMode,
      theme: themeState.lightTheme,
      darkTheme: themeState.darkTheme,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final logger = context.read<LoggerService>();
    final authState = context.watch<AuthState>();

    logger.log('Building HomePage');

    return Scaffold(
      appBar: AppBar(
        title: const Text('MultiProvider Demo'),
        actions: [
          // 购物车图标
          const CartIcon(),
          // 主题切换
          IconButton(
            icon: const Icon(Icons.brightness_6),
            onPressed: () {
              final themeState = context.read<ThemeState>();
              themeState.setThemeMode(
                themeState.themeMode == ThemeMode.light
                    ? ThemeMode.dark
                    : ThemeMode.light,
              );
            },
          ),
        ],
      ),
      body: authState.isAuthenticated
          ? const AuthenticatedContent()
          : const LoginForm(),
    );
  }
}

class CartIcon extends StatelessWidget {
  const CartIcon({super.key});

  @override
  Widget build(BuildContext context) {
    return Consumer<CartState>(
      builder: (context, cart, child) {
        return Badge(
          label: Text('${cart.itemCount}'),
          child: IconButton(
            icon: const Icon(Icons.shopping_cart),
            onPressed: () {
              showModalBottomSheet(
                context: context,
                builder: (context) => const CartSheet(),
              );
            },
          ),
        );
      },
    );
  }
}

class LoginForm extends StatelessWidget {
  const LoginForm({super.key});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.lock_outline, size: 64, color: Colors.grey),
            const SizedBox(height: 24),
            const Text(
              '请先登录',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                context.read<AuthState>().login(
                      'user_${DateTime.now().millisecond}',
                      'User ${DateTime.now().second}',
                    );
              },
              child: const Text('登录'),
            ),
          ],
        ),
      ),
    );
  }
}

class AuthenticatedContent extends StatelessWidget {
  const AuthenticatedContent({super.key});

  @override
  Widget build(BuildContext context) {
    final authState = context.watch<AuthState>();
    final logger = context.read<LoggerService>();

    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 用户信息卡片
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    '欢迎, ${authState.username}!',
                    style: const TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text('User ID: ${authState.userId}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      logger.log('User logged out');
                      context.read<AuthState>().logout();
                    },
                    child: const Text('退出登录'),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(height: 24),
          // 商品列表
          const Text(
            '商品列表',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 16),
          Expanded(
            child: ListView(
              children: [
                _buildProductCard(context, '1', 'iPhone 15', 9999),
                _buildProductCard(context, '2', 'MacBook Pro', 14999),
                _buildProductCard(context, '3', 'AirPods Pro', 1999),
                _buildProductCard(context, '4', 'iPad Air', 4999),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildProductCard(
    BuildContext context,
    String id,
    String name,
    double price,
  ) {
    final logger = context.read<LoggerService>();

    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        title: Text(name),
        subtitle: Text('¥${price.toStringAsFixed(2)}'),
        trailing: ElevatedButton(
          onPressed: () {
            context.read<CartState>().addItem(id, name, price);
            logger.log('Added $name to cart');
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('$name 已添加到购物车')),
            );
          },
          child: const Text('添加'),
        ),
      ),
    );
  }
}

class CartSheet extends StatelessWidget {
  const CartSheet({super.key});

  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartState>();

    return Container(
      padding: const EdgeInsets.all(16),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text(
                '购物车',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              if (cart.items.isNotEmpty)
                TextButton(
                  onPressed: () => cart.clear(),
                  child: const Text('清空'),
                ),
            ],
          ),
          const SizedBox(height: 16),
          if (cart.items.isEmpty)
            const Center(
              child: Padding(
                padding: EdgeInsets.all(32),
                child: Text('购物车是空的'),
              ),
            )
          else
            ...cart.items.map((item) => ListTile(
                  title: Text(item.name),
                  subtitle: Text('数量: ${item.quantity}'),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Text('¥${(item.price * item.quantity).toStringAsFixed(2)}'),
                      IconButton(
                        icon: const Icon(Icons.delete),
                        onPressed: () => cart.removeItem(item.id),
                      ),
                    ],
                  ),
                )),
          if (cart.items.isNotEmpty) ...[
            const Divider(),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text(
                  '总计',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                ),
                Text(
                  '¥${cart.totalPrice.toStringAsFixed(2)}',
                  style: const TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                    color: Colors.blue,
                  ),
                ),
              ],
            ),
          ],
        ],
      ),
    );
  }
}
```

---

## 第六章：ProxyProvider

### 6.1 ProxyProvider 概述

`ProxyProvider` 用于创建依赖其他 Provider 的对象。当依赖的 Provider 发生变化时，ProxyProvider 会自动更新创建的对象。

#### ProxyProvider 类型

| 类型                            | 依赖数量 | 说明                    |
| ------------------------------- | -------- | ----------------------- |
| `ProxyProvider<T, R>`           | 1        | 依赖 1 个 Provider      |
| `ProxyProvider2<T1, T2, R>`     | 2        | 依赖 2 个 Providers     |
| `ProxyProvider3<T1, T2, T3, R>` | 3        | 依赖 3 个 Providers     |
| ...                             | ...      | 最多支持 ProxyProvider6 |

#### 构造函数

```dart
ProxyProvider<T, R>({
  Key? key,
  Create<R>? create,
  required ProxyProviderBuilder<T, R> update,
  UpdateShouldNotify<R>? updateShouldNotify,
  Dispose<R>? dispose,
  TransitionBuilder? builder,
  Widget? child,
});
```

#### 参数说明

| 参数名               | 类型                         | 说明                   |
| -------------------- | ---------------------------- | ---------------------- |
| `create`             | `Create<R>?`                 | 首次创建对象的回调     |
| `update`             | `ProxyProviderBuilder<T, R>` | 更新对象的回调（必须） |
| `updateShouldNotify` | `UpdateShouldNotify<R>?`     | 自定义更新通知逻辑     |
| `dispose`            | `Dispose<R>?`                | 释放资源的回调         |

#### ProxyProvider 工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                  ProxyProvider 工作流程                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 初始创建                                                     │
│     Provider<A> ──► ProxyProvider<A, B>                         │
│                    ├── create: (context) => B(context.read<A>())│
│                    └── 创建 B 的初始实例                        │
│                                                                 │
│  2. 依赖变化时更新                                               │
│     Provider<A> 变化 ──► ProxyProvider<A, B>                    │
│                         ├── update: (context, a, previousB) {   │
│                         │           return B(a, previousB.data); │
│                         │         }                              │
│                         └── 返回新的 B 实例                      │
│                                                                 │
│  3. 多个依赖                                                     │
│     Provider<A> ──┐                                              │
│     Provider<B> ──┼──► ProxyProvider2<A, B, C>                  │
│                    │    update: (context, a, b, previousC) {    │
│                    │      return C(a, b);                        │
│                    │    }                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

// ============ 基础服务 ============

// 1. API 配置
class ApiConfig {
  final String baseUrl;
  final String apiKey;
  final Duration timeout;

  const ApiConfig({
    required this.baseUrl,
    required this.apiKey,
    this.timeout = const Duration(seconds: 30),
  });
}

// 2. 认证令牌
class AuthToken {
  final String token;
  final DateTime expiresAt;

  AuthToken({
    required this.token,
    required this.expiresAt,
  });

  bool get isExpired => DateTime.now().isAfter(expiresAt);
}

// ============ 依赖服务 ============

// 3. HTTP 客户端（依赖 ApiConfig）
class HttpClient {
  final ApiConfig _config;

  HttpClient(this._config);

  Future<String> get(String path) async {
    await Future.delayed(const Duration(milliseconds: 500));
    return 'GET ${_config.baseUrl}$path';
  }

  Future<String> post(String path, Map<String, dynamic> data) async {
    await Future.delayed(const Duration(milliseconds: 500));
    return 'POST ${_config.baseUrl}$path with ${data.toString()}';
  }

  @override
  String toString() => 'HttpClient(${_config.baseUrl})';
}

// 4. 认证服务（依赖 ApiConfig 和 HttpClient）
class AuthService {
  final ApiConfig _config;
  final HttpClient _httpClient;
  AuthToken? _token;

  AuthService(this._config, this._httpClient);

  Future<bool> login(String username, String password) async {
    final response = await _httpClient.post('/auth/login', {
      'username': username,
      'password': password,
    });
    // 模拟登录成功
    _token = AuthToken(
      token: 'mock_token_${DateTime.now().millisecond}',
      expiresAt: DateTime.now().add(const Duration(hours: 24)),
    );
    return true;
  }

  Future<void> logout() async {
    _token = null;
  }

  bool get isAuthenticated => _token != null && !_token!.isExpired;
  String? get token => _token?.token;

  @override
  String toString() => 'AuthService(authenticated: $isAuthenticated)';
}

// 5. 用户服务（依赖 AuthService 和 HttpClient）
class UserService {
  final AuthService _authService;
  final HttpClient _httpClient;

  UserService(this._authService, this._httpClient);

  Future<Map<String, dynamic>> getUserProfile() async {
    if (!_authService.isAuthenticated) {
      throw Exception('Not authenticated');
    }
    final response = await _httpClient.get('/user/profile');
    return {
      'id': 'user_123',
      'name': '张三',
      'email': 'zhangsan@example.com',
    };
  }

  @override
  String toString() => 'UserService(auth: ${_authService.isAuthenticated})';
}

// ============ 应用入口 ============

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        // 1. 基础配置
        Provider<ApiConfig>(
          create: (_) => const ApiConfig(
            baseUrl: 'https://api.example.com',
            apiKey: 'my_api_key',
          ),
        ),
        // 2. HTTP 客户端（依赖 ApiConfig）
        ProxyProvider<ApiConfig, HttpClient>(
          create: (context) => HttpClient(context.read<ApiConfig>()),
          update: (context, config, previous) {
            // 当 ApiConfig 变化时，重新创建 HttpClient
            return HttpClient(config);
          },
        ),
        // 3. 认证服务（依赖 ApiConfig 和 HttpClient）
        ProxyProvider2<ApiConfig, HttpClient, AuthService>(
          create: (context) => AuthService(
            context.read<ApiConfig>(),
            context.read<HttpClient>(),
          ),
          update: (context, config, httpClient, previous) {
            // 当任一依赖变化时，保留认证状态
            if (previous != null) {
              return previous; // 保持现有实例
            }
            return AuthService(config, httpClient);
          },
        ),
        // 4. 用户服务（依赖 AuthService 和 HttpClient）
        ProxyProvider2<AuthService, HttpClient, UserService>(
          create: (context) => UserService(
            context.read<AuthService>(),
            context.read<HttpClient>(),
          ),
          update: (context, authService, httpClient, previous) {
            return UserService(authService, httpClient);
          },
        ),
      ],
      child: MaterialApp(
        title: 'ProxyProvider Demo',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const HomePage(),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final apiConfig = context.read<ApiConfig>();
    final httpClient = context.read<HttpClient>();
    final authService = context.read<AuthService>();
    final userService = context.read<UserService>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('ProxyProvider Demo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 服务信息卡片
            _buildServiceInfoCard(
              'ApiConfig',
              'baseUrl: ${apiConfig.baseUrl}\napiKey: ${apiConfig.apiKey}',
              Colors.blue,
            ),
            const SizedBox(height: 12),
            _buildServiceInfoCard(
              'HttpClient',
              httpClient.toString(),
              Colors.green,
            ),
            const SizedBox(height: 12),
            _buildServiceInfoCard(
              'AuthService',
              authService.toString(),
              Colors.orange,
            ),
            const SizedBox(height: 12),
            _buildServiceInfoCard(
              'UserService',
              userService.toString(),
              Colors.purple,
            ),
            const SizedBox(height: 24),
            // 操作按钮
            Row(
              children: [
                ElevatedButton(
                  onPressed: () async {
                    final success = await authService.login('user', 'pass');
                    if (success && context.mounted) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(content: Text('登录成功')),
                      );
                      // 强制重建以显示更新后的状态
                      (context as Element).markNeedsBuild();
                    }
                  },
                  child: const Text('登录'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () async {
                    await authService.logout();
                    if (context.mounted) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(content: Text('已退出登录')),
                      );
                      (context as Element).markNeedsBuild();
                    }
                  },
                  child: const Text('退出'),
                ),
                const SizedBox(width: 12),
                ElevatedButton(
                  onPressed: () async {
                    try {
                      final profile = await userService.getUserProfile();
                      if (context.mounted) {
                        showDialog(
                          context: context,
                          builder: (context) => AlertDialog(
                            title: const Text('用户资料'),
                            content: Text(profile.toString()),
                            actions: [
                              TextButton(
                                onPressed: () => Navigator.pop(context),
                                child: const Text('确定'),
                              ),
                            ],
                          ),
                        );
                      }
                    } catch (e) {
                      if (context.mounted) {
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(content: Text('错误: $e')),
                        );
                      }
                    }
                  },
                  child: const Text('获取资料'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildServiceInfoCard(String title, String content, Color color) {
    return Card(
      color: color.withOpacity(0.1),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              title,
              style: TextStyle(
                fontWeight: FontWeight.bold,
                color: color,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              content,
              style: const TextStyle(fontSize: 12),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 第七章：扩展方法

### 7.1 context.watch()

`context.watch<T>()` 用于在 Widget 的 build 方法中监听 Provider 的变化。当 Provider 的值变化时，Widget 会自动重建。

#### 使用场景

```dart
// 在 StatelessWidget 中使用
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 监听 Counter 的变化，变化时重建整个 MyWidget
    final counter = context.watch<Counter>();
    return Text('Count: ${counter.value}');
  }
}
```

#### 完整示例

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class Counter extends ChangeNotifier {
  int _value = 0;
  int get value => _value;

  void increment() {
    _value++;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => Counter(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('context.watch() Demo')),
          body: const CounterDisplay(),
          floatingActionButton: const CounterButton(),
        ),
      ),
    );
  }
}

class CounterDisplay extends StatelessWidget {
  const CounterDisplay({super.key});

  @override
  Widget build(BuildContext context) {
    // 使用 context.watch() 监听 Counter 变化
    final counter = context.watch<Counter>();
    print('CounterDisplay rebuilt');

    return Center(
      child: Text(
        'Count: ${counter.value}',
        style: const TextStyle(fontSize: 48),
      ),
    );
  }
}

class CounterButton extends StatelessWidget {
  const CounterButton({super.key});

  @override
  Widget build(BuildContext context) {
    // 这里不需要监听，所以使用 context.read()
    return FloatingActionButton(
      onPressed: () => context.read<Counter>().increment(),
      child: const Icon(Icons.add),
    );
  }
}
```

### 7.2 context.read()

`context.read<T>()` 用于在回调函数中读取 Provider 的值，不会建立监听关系。适用于事件处理（如 onPressed）。

#### 使用场景

```dart
// 在回调中使用 - 不会重建 Widget
ElevatedButton(
  onPressed: () {
    // 读取值但不监听变化
    final counter = context.read<Counter>();
    counter.increment();
  },
  child: const Text('Increment'),
)
```

#### 完整示例

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class SettingsService {
  void saveSetting(String key, dynamic value) {
    debugPrint('Saving setting: $key = $value');
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider<SettingsService>(
      create: (_) => SettingsService(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('context.read() Demo')),
          body: const SettingsPage(),
        ),
      ),
    );
  }
}

class SettingsPage extends StatelessWidget {
  const SettingsPage({super.key});

  @override
  Widget build(BuildContext context) {
    print('SettingsPage built'); // 只打印一次

    return ListView(
      children: [
        ListTile(
          title: const Text('通知'),
          trailing: Switch(
            value: true,
            // 使用 context.read() 读取服务
            onChanged: (value) {
              context.read<SettingsService>().saveSetting('notifications', value);
            },
          ),
        ),
        ListTile(
          title: const Text('深色模式'),
          trailing: Switch(
            value: false,
            onChanged: (value) {
              context.read<SettingsService>().saveSetting('darkMode', value);
            },
          ),
        ),
        ListTile(
          title: const Text('清除缓存'),
          onTap: () {
            // 读取服务并调用方法
            context.read<SettingsService>().saveSetting('clearCache', true);
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(content: Text('缓存已清除')),
            );
          },
        ),
      ],
    );
  }
}
```

### 7.3 context.select()

`context.select<T, R>(R Function(T value) selector)` 用于选择性地监听 Provider 的部分属性，只有选中的属性变化时才重建 Widget。

#### 使用场景

```dart
// 只监听 User 的 name 属性
class UserWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 只选择 name 属性，其他属性变化不会重建
    final name = context.select<User, String>((user) => user.name);
    return Text('Name: $name');
  }
}
```

#### 完整示例

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class UserProfile extends ChangeNotifier {
  String _name = '张三';
  int _age = 25;
  String _email = 'zhangsan@example.com';

  String get name => _name;
  int get age => _age;
  String get email => _email;

  void updateName(String name) {
    _name = name;
    notifyListeners();
  }

  void updateAge(int age) {
    _age = age;
    notifyListeners();
  }

  void updateEmail(String email) {
    _email = email;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => UserProfile(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('context.select() Demo')),
          body: const UserProfilePage(),
        ),
      ),
    );
  }
}

class UserProfilePage extends StatelessWidget {
  const UserProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 只监听 name 变化
          _buildNameSection(),
          const SizedBox(height: 16),
          // 只监听 age 变化
          _buildAgeSection(),
          const SizedBox(height: 16),
          // 只监听 email 变化
          _buildEmailSection(),
          const SizedBox(height: 32),
          // 控制面板
          const ControlPanel(),
        ],
      ),
    );
  }

  Widget _buildNameSection() {
    return Builder(
      builder: (context) {
        // 只选择 name 属性
        final name = context.select<UserProfile, String>((user) => user.name);
        print('Name section rebuilt: $name');

        return Card(
          color: Colors.blue[50],
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Name: $name',
              style: const TextStyle(fontSize: 20),
            ),
          ),
        );
      },
    );
  }

  Widget _buildAgeSection() {
    return Builder(
      builder: (context) {
        // 只选择 age 属性
        final age = context.select<UserProfile, int>((user) => user.age);
        print('Age section rebuilt: $age');

        return Card(
          color: Colors.green[50],
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Age: $age',
              style: const TextStyle(fontSize: 20),
            ),
          ),
        );
      },
    );
  }

  Widget _buildEmailSection() {
    return Builder(
      builder: (context) {
        // 只选择 email 属性
        final email = context.select<UserProfile, String>((user) => user.email);
        print('Email section rebuilt: $email');

        return Card(
          color: Colors.orange[50],
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              'Email: $email',
              style: const TextStyle(fontSize: 20),
            ),
          ),
        );
      },
    );
  }
}

class ControlPanel extends StatelessWidget {
  const ControlPanel({super.key});

  @override
  Widget build(BuildContext context) {
    final user = context.read<UserProfile>();

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              '控制面板',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () => user.updateName('李四${DateTime.now().second}'),
                  child: const Text('Change Name'),
                ),
                ElevatedButton(
                  onPressed: () => user.updateAge(20 + DateTime.now().second),
                  child: const Text('Change Age'),
                ),
                ElevatedButton(
                  onPressed: () => user.updateEmail('user${DateTime.now().second}@test.com'),
                  child: const Text('Change Email'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### 7.4 Provider.of()

`Provider.of<T>(context, listen: bool)` 是传统的方法，用于获取 Provider 的值。`listen: true` 等价于 `context.watch()`，`listen: false` 等价于 `context.read()`。

#### 使用方式

```dart
// 监听变化（已不推荐，使用 context.watch() 替代）
final counter = Provider.of<Counter>(context, listen: true);

// 不监听变化（已不推荐，使用 context.read() 替代）
final counter = Provider.of<Counter>(context, listen: false);
```

#### 完整示例

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class Counter extends ChangeNotifier {
  int _value = 0;
  int get value => _value;

  void increment() {
    _value++;
    notifyListeners();
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => Counter(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('Provider.of() Demo')),
          body: const CounterPage(),
        ),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context) {
    // 使用 Provider.of 监听变化
    final counter = Provider.of<Counter>(context, listen: true);

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(
            'Count: ${counter.value}',
            style: const TextStyle(fontSize: 48),
          ),
          const SizedBox(height: 24),
          ElevatedButton(
            onPressed: () {
              // 使用 Provider.of 不监听变化
              Provider.of<Counter>(context, listen: false).increment();
            },
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}
```

### 7.5 扩展方法对比

```
┌─────────────────────────────────────────────────────────────────┐
│                  扩展方法对比表                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  方法                │  监听变化  │  使用场景                    │
│  ────────────────────┼───────────┼────────────────────────────  │
│  context.watch<T>()  │  是        │  build 方法中获取状态        │
│  context.read<T>()   │  否        │  回调函数中读取状态          │
│  context.select<T,R>()│ 部分      │  只监听特定属性              │
│  Provider.of<T>()    │  可选      │  传统方式（不推荐）          │
│                                                                 │
│  推荐用法：                                                     │
│  • build 方法中使用 watch() 或 select()                         │
│  • 事件回调中使用 read()                                        │
│  • 避免使用 Provider.of()（为了代码一致性）                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 第八章：完整项目案例

### 8.1 待办事项应用

一个完整的待办事项应用，展示 Provider 的各种用法。

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const TodoApp());
}

// ============ 数据模型 ============

enum TodoPriority { low, medium, high }

class Todo {
  final String id;
  String title;
  bool isCompleted;
  TodoPriority priority;
  DateTime createdAt;

  Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
    this.priority = TodoPriority.medium,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();

  Todo copyWith({
    String? title,
    bool? isCompleted,
    TodoPriority? priority,
  }) {
    return Todo(
      id: id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
      priority: priority ?? this.priority,
      createdAt: createdAt,
    );
  }
}

// ============ 状态管理 ============

class TodoFilter extends ChangeNotifier {
  bool _showCompleted = true;
  TodoPriority? _priorityFilter;
  String _searchQuery = '';

  bool get showCompleted => _showCompleted;
  TodoPriority? get priorityFilter => _priorityFilter;
  String get searchQuery => _searchQuery;

  void toggleShowCompleted() {
    _showCompleted = !_showCompleted;
    notifyListeners();
  }

  void setPriorityFilter(TodoPriority? priority) {
    _priorityFilter = priority;
    notifyListeners();
  }

  void setSearchQuery(String query) {
    _searchQuery = query;
    notifyListeners();
  }

  void clearFilters() {
    _showCompleted = true;
    _priorityFilter = null;
    _searchQuery = '';
    notifyListeners();
  }
}

class TodoList extends ChangeNotifier {
  final List<Todo> _todos = [];

  List<Todo> get todos => List.unmodifiable(_todos);

  List<Todo> get filteredTodos {
    return _todos.where((todo) {
      // 完成状态过滤
      if (!todo.isCompleted && !_showCompleted) return false;
      // 优先级过滤
      if (_priorityFilter != null && todo.priority != _priorityFilter) {
        return false;
      }
      // 搜索过滤
      if (_searchQuery.isNotEmpty &&
          !todo.title.toLowerCase().contains(_searchQuery.toLowerCase())) {
        return false;
      }
      return true;
    }).toList();
  }

  // 过滤状态
  bool _showCompleted = true;
  TodoPriority? _priorityFilter;
  String _searchQuery = '';

  set showCompleted(bool value) {
    _showCompleted = value;
    notifyListeners();
  }

  set priorityFilter(TodoPriority? value) {
    _priorityFilter = value;
    notifyListeners();
  }

  set searchQuery(String value) {
    _searchQuery = value;
    notifyListeners();
  }

  // 统计
  int get totalCount => _todos.length;
  int get completedCount => _todos.where((t) => t.isCompleted).length;
  int get pendingCount => _todos.where((t) => !t.isCompleted).length;

  void addTodo(String title, TodoPriority priority) {
    _todos.add(Todo(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title,
      priority: priority,
    ));
    notifyListeners();
  }

  void toggleTodo(String id) {
    final index = _todos.indexWhere((t) => t.id == id);
    if (index >= 0) {
      _todos[index].isCompleted = !_todos[index].isCompleted;
      notifyListeners();
    }
  }

  void updateTodo(String id, {String? title, TodoPriority? priority}) {
    final index = _todos.indexWhere((t) => t.id == id);
    if (index >= 0) {
      if (title != null) _todos[index].title = title;
      if (priority != null) _todos[index].priority = priority;
      notifyListeners();
    }
  }

  void deleteTodo(String id) {
    _todos.removeWhere((t) => t.id == id);
    notifyListeners();
  }

  void clearCompleted() {
    _todos.removeWhere((t) => t.isCompleted);
    notifyListeners();
  }
}

// ============ 应用入口 ============

class TodoApp extends StatelessWidget {
  const TodoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => TodoList()),
        ChangeNotifierProvider(create: (_) => TodoFilter()),
      ],
      child: MaterialApp(
        title: 'Todo App',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
          useMaterial3: true,
        ),
        home: const TodoHomePage(),
      ),
    );
  }
}

class TodoHomePage extends StatelessWidget {
  const TodoHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('待办事项'),
        centerTitle: true,
        actions: [
          // 统计徽章
          const TodoStatsBadge(),
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: () {
              context.read<TodoList>().clearCompleted();
            },
            tooltip: '清除已完成',
          ),
        ],
      ),
      body: const Column(
        children: [
          // 搜索栏
          SearchBar(),
          // 过滤器
          FilterBar(),
          // 待办列表
          Expanded(child: TodoListView()),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddTodoDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddTodoDialog(BuildContext context) {
    final textController = TextEditingController();
    TodoPriority selectedPriority = TodoPriority.medium;

    showDialog(
      context: context,
      builder: (dialogContext) {
        return StatefulBuilder(
          builder: (context, setState) {
            return AlertDialog(
              title: const Text('添加待办事项'),
              content: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  TextField(
                    controller: textController,
                    decoration: const InputDecoration(
                      labelText: '标题',
                      hintText: '输入待办事项...',
                    ),
                    autofocus: true,
                  ),
                  const SizedBox(height: 16),
                  SegmentedButton<TodoPriority>(
                    segments: const [
                      ButtonSegment(
                        value: TodoPriority.low,
                        label: Text('低'),
                        icon: Icon(Icons.arrow_downward),
                      ),
                      ButtonSegment(
                        value: TodoPriority.medium,
                        label: Text('中'),
                        icon: Icon(Icons.remove),
                      ),
                      ButtonSegment(
                        value: TodoPriority.high,
                        label: Text('高'),
                        icon: Icon(Icons.arrow_upward),
                      ),
                    ],
                    selected: {selectedPriority},
                    onSelectionChanged: (Set<TodoPriority> newSelection) {
                      setState(() {
                        selectedPriority = newSelection.first;
                      });
                    },
                  ),
                ],
              ),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('取消'),
                ),
                FilledButton(
                  onPressed: () {
                    if (textController.text.isNotEmpty) {
                      context.read<TodoList>().addTodo(
                            textController.text,
                            selectedPriority,
                          );
                      Navigator.pop(context);
                    }
                  },
                  child: const Text('添加'),
                ),
              ],
            );
          },
        );
      },
    );
  }
}

class TodoStatsBadge extends StatelessWidget {
  const TodoStatsBadge({super.key});

  @override
  Widget build(BuildContext context) {
    return Selector<TodoList, (int, int)>(
      selector: (_, list) => (list.completedCount, list.pendingCount),
      builder: (context, stats, _) {
        final (completed, pending) = stats;
        return Padding(
          padding: const EdgeInsets.only(right: 16),
          child: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              Chip(
                avatar: const Icon(Icons.check_circle, size: 16),
                label: Text('$completed'),
                backgroundColor: Colors.green[100],
              ),
              const SizedBox(width: 8),
              Chip(
                avatar: const Icon(Icons.pending, size: 16),
                label: Text('$pending'),
                backgroundColor: Colors.orange[100],
              ),
            ],
          ),
        );
      },
    );
  }
}

class SearchBar extends StatelessWidget {
  const SearchBar({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: TextField(
        decoration: InputDecoration(
          hintText: '搜索待办事项...',
          prefixIcon: const Icon(Icons.search),
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          contentPadding: const EdgeInsets.symmetric(horizontal: 16),
        ),
        onChanged: (value) {
          context.read<TodoList>().searchQuery = value;
        },
      ),
    );
  }
}

class FilterBar extends StatelessWidget {
  const FilterBar({super.key});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: [
          // 显示已完成切换
          Consumer<TodoList>(
            builder: (context, list, _) {
              return FilterChip(
                label: const Text('显示已完成'),
                selected: list._showCompleted,
                onSelected: (_) => list.showCompleted = !list._showCompleted,
              );
            },
          ),
          const SizedBox(width: 8),
          // 优先级过滤
          Consumer<TodoList>(
            builder: (context, list, _) {
              return PopupMenuButton<TodoPriority?>(
                child: Chip(
                  label: Text(
                    list._priorityFilter == null
                        ? '所有优先级'
                        : _priorityToString(list._priorityFilter!),
                  ),
                ),
                itemBuilder: (context) => [
                  const PopupMenuItem(
                    value: null,
                    child: Text('所有优先级'),
                  ),
                  ...TodoPriority.values.map((p) => PopupMenuItem(
                        value: p,
                        child: Text(_priorityToString(p)),
                      )),
                ],
                onSelected: (value) {
                  list.priorityFilter = value;
                },
              );
            },
          ),
        ],
      ),
    );
  }

  String _priorityToString(TodoPriority priority) {
    switch (priority) {
      case TodoPriority.low:
        return '低优先级';
      case TodoPriority.medium:
        return '中优先级';
      case TodoPriority.high:
        return '高优先级';
    }
  }
}

class TodoListView extends StatelessWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context) {
    return Consumer<TodoList>(
      builder: (context, list, _) {
        final todos = list.filteredTodos;

        if (todos.isEmpty) {
          return const Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.inbox, size: 64, color: Colors.grey),
                SizedBox(height: 16),
                Text(
                  '没有待办事项',
                  style: TextStyle(color: Colors.grey, fontSize: 16),
                ),
              ],
            ),
          );
        }

        return ListView.builder(
          padding: const EdgeInsets.all(16),
          itemCount: todos.length,
          itemBuilder: (context, index) {
            final todo = todos[index];
            return TodoItemCard(todo: todo);
          },
        );
      },
    );
  }
}

class TodoItemCard extends StatelessWidget {
  final Todo todo;

  const TodoItemCard({super.key, required this.todo});

  @override
  Widget build(BuildContext context) {
    return Dismissible(
      key: Key(todo.id),
      direction: DismissDirection.endToStart,
      background: Container(
        alignment: Alignment.centerRight,
        padding: const EdgeInsets.only(right: 16),
        color: Colors.red,
        child: const Icon(Icons.delete, color: Colors.white),
      ),
      onDismissed: (_) {
        context.read<TodoList>().deleteTodo(todo.id);
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('待办事项已删除')),
        );
      },
      child: Card(
        margin: const EdgeInsets.only(bottom: 8),
        child: ListTile(
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (_) {
              context.read<TodoList>().toggleTodo(todo.id);
            },
          ),
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.isCompleted ? TextDecoration.lineThrough : null,
              color: todo.isCompleted ? Colors.grey : null,
            ),
          ),
          subtitle: Text(
            '创建于 ${_formatDate(todo.createdAt)}',
            style: const TextStyle(fontSize: 12),
          ),
          trailing: _buildPriorityChip(todo.priority),
          onTap: () => _showEditDialog(context, todo),
        ),
      ),
    );
  }

  Widget _buildPriorityChip(TodoPriority priority) {
    final (color, label) = switch (priority) {
      TodoPriority.low => (Colors.green, '低'),
      TodoPriority.medium => (Colors.orange, '中'),
      TodoPriority.high => (Colors.red, '高'),
    };

    return Chip(
      label: Text(
        label,
        style: const TextStyle(fontSize: 12, color: Colors.white),
      ),
      backgroundColor: color,
      padding: EdgeInsets.zero,
      materialTapTargetSize: MaterialTapTargetSize.shrinkWrap,
    );
  }

  String _formatDate(DateTime date) {
    return '${date.month}/${date.day} ${date.hour}:${date.minute.toString().padLeft(2, '0')}';
  }

  void _showEditDialog(BuildContext context, Todo todo) {
    final textController = TextEditingController(text: todo.title);
    TodoPriority selectedPriority = todo.priority;

    showDialog(
      context: context,
      builder: (dialogContext) {
        return StatefulBuilder(
          builder: (context, setState) {
            return AlertDialog(
              title: const Text('编辑待办事项'),
              content: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  TextField(
                    controller: textController,
                    decoration: const InputDecoration(labelText: '标题'),
                  ),
                  const SizedBox(height: 16),
                  SegmentedButton<TodoPriority>(
                    segments: const [
                      ButtonSegment(
                        value: TodoPriority.low,
                        label: Text('低'),
                      ),
                      ButtonSegment(
                        value: TodoPriority.medium,
                        label: Text('中'),
                      ),
                      ButtonSegment(
                        value: TodoPriority.high,
                        label: Text('高'),
                      ),
                    ],
                    selected: {selectedPriority},
                    onSelectionChanged: (Set<TodoPriority> newSelection) {
                      setState(() {
                        selectedPriority = newSelection.first;
                      });
                    },
                  ),
                ],
              ),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('取消'),
                ),
                FilledButton(
                  onPressed: () {
                    context.read<TodoList>().updateTodo(
                          todo.id,
                          title: textController.text,
                          priority: selectedPriority,
                        );
                    Navigator.pop(context);
                  },
                  child: const Text('保存'),
                ),
              ],
            );
          },
        );
      },
    );
  }
}
```

---

## 附录

### A.1 API 速查表

#### Provider 类型

| Provider                     | 用途       | 示例                                                       |
| ---------------------------- | ---------- | ---------------------------------------------------------- |
| `Provider<T>`                | 不可变对象 | `Provider<Config>(create: (_) => Config())`                |
| `ChangeNotifierProvider<T>`  | 可变状态   | `ChangeNotifierProvider(create: (_) => Counter())`         |
| `ValueListenableProvider<T>` | 值监听     | `ValueListenableProvider(create: (_) => ValueNotifier(0))` |
| `FutureProvider<T>`          | 异步数据   | `FutureProvider(create: (_) => fetchData())`               |
| `StreamProvider<T>`          | 数据流     | `StreamProvider(create: (_) => dataStream)`                |
| `MultiProvider`              | 组合多个   | `MultiProvider(providers: [...])`                          |
| `ProxyProvider<T,R>`         | 依赖其他   | `ProxyProvider(update: (_, a, __) => B(a))`                |

#### 消费方式

| 方法                    | 监听 | 使用位置   |
| ----------------------- | ---- | ---------- |
| `context.watch<T>()`    | 是   | build 方法 |
| `context.read<T>()`     | 否   | 回调函数   |
| `context.select<T,R>()` | 部分 | build 方法 |
| `Consumer<T>`           | 是   | Widget 树  |
| `Selector<T,R>`         | 部分 | Widget 树  |

### A.2 常见问题

#### Q1: Provider 找不到？

```dart
// 错误：在 Provider 作用域外访问
void main() {
  runApp(
    MyWidget(), // 这里访问 context.read<Config>() 会报错
  );
}

// 正确：在 Provider 作用域内访问
void main() {
  runApp(
    Provider(
      create: (_) => Config(),
      child: MyWidget(), // 可以访问
    ),
  );
}
```

#### Q2: 如何避免不必要的重建？

```dart
// 方法1：使用 Selector
Selector<AppState, int>(
  selector: (_, state) => state.counter,
  builder: (_, counter, __) => Text('$counter'),
)

// 方法2：使用 context.select
final counter = context.select<AppState, int>((s) => s.counter);

// 方法3：使用 Consumer 的 child
Consumer<AppState>(
  builder: (_, state, child) => Column(
    children: [Text('${state.counter}'), child!],
  ),
  child: ExpensiveWidget(), // 不会重建
)
```

#### Q3: 如何处理异步初始化？

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FutureProvider<Config?>(
      initialData: null,
      create: (_) => loadConfig(),
      child: Consumer<Config?>(
        builder: (context, config, _) {
          if (config == null) {
            return const LoadingScreen();
          }
          return MainApp(config: config);
        },
      ),
    );
  }
}
```

#### Q4: 如何测试 Provider？

```dart
// 测试代码
testWidgets('Counter increments', (tester) async {
  await tester.pumpWidget(
    ChangeNotifierProvider(
      create: (_) => Counter(),
      child: MyApp(),
    ),
  );

  // 查找并点击按钮
  await tester.tap(find.byType(FloatingActionButton));
  await tester.pump();

  // 验证结果
  expect(find.text('1'), findsOneWidget);
});
```

### A.3 最佳实践

1. **合理拆分状态**：将相关状态放在同一个 ChangeNotifier 中
2. **使用 Selector**：避免不必要的 Widget 重建
3. **在回调中使用 read()**：不要在 onPressed 等回调中使用 watch()
4. **及时释放资源**：在 Provider 的 dispose 回调中清理资源
5. **使用 MultiProvider**：避免深层嵌套
6. **类型安全**：始终指定泛型类型

### A.4 版本兼容性

| Provider 版本 | Flutter 版本 | 特性                      |
| ------------- | ------------ | ------------------------- |
| 6.x           | 3.x          | context.watch/read/select |
| 5.x           | 2.x          | Provider.of               |
| 4.x           | 1.x          | 基础功能                  |

---

**文档版本**: 1.0  
**最后更新**: 2024年  
**Provider 版本**: ^6.1.5+1
