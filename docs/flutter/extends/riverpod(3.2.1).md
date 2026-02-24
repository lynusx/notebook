# Flutter Riverpod 3.2.1 完整详解与实战指南

## 前言

Riverpod 是 Flutter 生态系统中功能最强大的响应式缓存和数据绑定框架，由 Provider 包的作者 Remi Rousselet 创建。Riverpod 这个名字是 "Provider" 的字母重组（anagram），它解决了 Provider 包存在的诸多问题，提供了编译时安全、更好的测试支持和更强大的功能。

Riverpod 3.2.1 版本带来了许多激动人心的新特性，包括代码生成、自动重试、离线持久化、Mutation 等实验性功能，使得状态管理变得更加简单和强大。

本书将详细介绍 Riverpod 3.2.1 的所有核心概念、Provider 类型、API 用法，以及如何将其集成到 Flutter 应用中。通过丰富的示例代码，帮助读者深入理解 Riverpod 的设计哲学和最佳实践。

---

## 第1章 Riverpod 基础

### 1.1 什么是 Riverpod

Riverpod 是一个响应式缓存和数据绑定框架，它让处理异步代码变得轻而易举：

- **默认处理错误/加载状态**：无需手动捕获错误
- **原生支持高级场景**：如下拉刷新
- **将逻辑与 UI 分离**：更好的代码组织
- **确保代码可测试、可扩展、可重用**

#### 1.1.1 Riverpod 的核心特性

| 特性               | 说明                                       |
| ------------------ | ------------------------------------------ |
| 声明式编程         | 以类似 Stateless Widget 的方式编写业务逻辑 |
| 原生网络请求支持   | 内置对异步操作的支持                       |
| 自动加载/错误处理  | 无需手动管理状态                           |
| 编译时安全         | 在编译期捕获错误                           |
| 类型安全的查询参数 | 使用 family 修饰符                         |
| 易于测试           | 支持依赖注入和 mock                        |
| 纯 Dart 支持       | 可在服务器/CLI 中使用                      |
| 易于组合的状态     | 通过 ref.watch 组合 providers              |
| 内置下拉刷新支持   | 简化常见 UI 模式                           |

#### 1.1.2 Riverpod 与 Provider 的区别

| 特性              | Provider          | Riverpod      |
| ----------------- | ----------------- | ------------- |
| 编译时安全        | ❌ 运行时错误     | ✅ 编译时检查 |
| 依赖 Flutter      | ✅ 是             | ❌ 纯 Dart    |
| 全局可用          | ❌ 依赖 Widget 树 | ✅ 全局声明   |
| 同类型多 Provider | ❌ 不支持         | ✅ 支持       |
| 自动释放          | ❌ 手动管理       | ✅ 自动释放   |
| 代码生成          | ❌ 不支持         | ✅ 支持       |

---

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加以下依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^3.2.1

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^3.0.0
  riverpod_lint: ^3.0.0
  custom_lint: ^0.7.0
```

#### 1.2.2 启用 Riverpod Lint

在 `analysis_options.yaml` 中启用 Riverpod Lint：

```yaml
analyzer:
  plugins:
    - custom_lint
```

#### 1.2.3 基础项目结构

```
lib/
├── main.dart              # 应用入口
├── providers/             # Provider 定义
│   ├── auth_provider.dart
│   ├── user_provider.dart
│   └── counter_provider.dart
├── models/                # 数据模型
│   ├── user.dart
│   └── counter.dart
├── services/              # 服务层
│   ├── auth_service.dart
│   └── api_service.dart
└── pages/                 # 页面
    ├── home_page.dart
    └── login_page.dart
```

---

### 1.3 第一个 Riverpod 应用

#### 1.3.1 包裹 ProviderScope

在使用任何 provider 之前，必须在应用根节点包裹 `ProviderScope`：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // 包裹 ProviderScope
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Demo',
      home: HomePage(),
    );
  }
}
```

> **Flutter框架小知识**
>
> `ProviderScope` 是所有 provider 的作用域容器。它负责管理 provider 的生命周期、缓存和依赖关系。每个 Riverpod 应用必须且只能有一个 `ProviderScope`，通常放在 `runApp` 的最外层。

#### 1.3.2 创建 Provider

```dart
// 定义一个简单的 provider
final counterProvider = StateProvider<int>((ref) => 0);
```

#### 1.3.3 在 Widget 中使用

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 使用 ConsumerWidget 替代 StatelessWidget
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 使用 ref.watch 监听 provider
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Riverpod Counter')),
      body: Center(
        child: Text(
          'Count: $count',
          style: Theme.of(context).textTheme.headlineMedium,
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // 使用 ref.read 读取 provider 并修改状态
        onPressed: () => ref.read(counterProvider.notifier).state++,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

---

## 第2章 Provider 类型详解

Riverpod 提供了多种 Provider 类型，每种类型适用于不同的场景。

### 2.1 Provider（基础 Provider）

`Provider` 是最基础的 provider 类型，用于暴露不可变对象或计算属性。

#### 2.1.1 核心特性

| 特性     | 说明                     |
| -------- | ------------------------ |
| 返回值   | 任意类型                 |
| 可变性   | 不可变                   |
| 适用场景 | 服务类、计算属性、常量值 |

#### 2.1.2 基础用法

```dart
// 简单的值 provider
final nameProvider = Provider<String>((ref) => 'John Doe');

// 计算属性 provider
final fullNameProvider = Provider<String>((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);
  return '$firstName $lastName';
});

// 服务类 provider
final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService(baseUrl: 'https://api.example.com');
});
```

#### 2.1.3 完整示例

```dart
// 定义格式化器 provider
final dateFormatterProvider = Provider<DateFormat>((ref) {
  return DateFormat.yMMMMd();
});

// 在 Widget 中使用
class DateDisplay extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final formatter = ref.watch(dateFormatterProvider);
    return Text(formatter.format(DateTime.now()));
  }
}
```

---

### 2.2 StateProvider（状态 Provider）

`StateProvider` 用于存储简单的可变状态，如基本类型（int、String、bool、enum 等）。

#### 2.2.1 核心特性

| 特性     | 说明                                   |
| -------- | -------------------------------------- |
| 返回值   | StateController<T>                     |
| 可变性   | 可变（通过 .state）                    |
| 适用场景 | 简单状态（计数器、筛选条件、布尔标志） |

#### 2.2.2 基础用法

```dart
// 定义 StateProvider
final counterProvider = StateProvider<int>((ref) => 0);
final userNameProvider = StateProvider<String>((ref) => '');
final isDarkModeProvider = StateProvider<bool>((ref) => false);
```

#### 2.2.3 读取和修改状态

```dart
class CounterPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 监听状态变化
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Text('Count: $count'),
      ),
      floatingActionButton: Column(
        mainAxisAlignment: MainAxisAlignment.end,
        children: [
          FloatingActionButton(
            // 读取并修改状态
            onPressed: () {
              ref.read(counterProvider.notifier).state++;
            },
            child: Icon(Icons.add),
          ),
          SizedBox(height: 8),
          FloatingActionButton(
            onPressed: () {
              ref.read(counterProvider.notifier).state--;
            },
            child: Icon(Icons.remove),
          ),
        ],
      ),
    );
  }
}
```

#### 2.2.4 注意事项

```dart
// ❌ 错误：不要直接修改 StateController
final controller = ref.read(counterProvider);
controller.state++; // 这会触发警告

// ✅ 正确：使用 .notifier 获取 StateController
ref.read(counterProvider.notifier).state++;

// ✅ 正确：使用 .update 方法
ref.read(counterProvider.notifier).update((state) => state + 1);
```

> **Dart Tips语法小贴士**
>
> `ref.read()` 和 `ref.watch()` 的区别：
>
> - `ref.watch()`：监听 provider 的变化，当值变化时 Widget 会重建
> - `ref.read()`：一次性读取 provider 的值，不监听变化
>
> 在事件回调（如 onPressed）中使用 `ref.read()`，在 build 方法中使用 `ref.watch()`。

---

### 2.3 FutureProvider（异步 Provider）

`FutureProvider` 用于处理异步操作，如网络请求、文件读取等。

#### 2.3.1 核心特性

| 特性     | 说明                           |
| -------- | ------------------------------ |
| 返回值   | AsyncValue<T>                  |
| 可变性   | 不可变（自动缓存）             |
| 适用场景 | 网络请求、数据库查询、异步计算 |

#### 2.3.2 基础用法

```dart
// 定义 FutureProvider
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});
```

#### 2.3.3 在 Widget 中使用

```dart
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => ListTile(
        leading: CircleAvatar(
          backgroundImage: NetworkImage(user.avatarUrl),
        ),
        title: Text(user.name),
        subtitle: Text(user.email),
      ),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (error, stack) => Center(
        child: Text('Error: $error'),
      ),
    );
  }
}
```

#### 2.3.4 AsyncValue 详解

`AsyncValue` 是一个封装异步状态的类，有三种状态：

| 状态         | 说明         |
| ------------ | ------------ |
| AsyncData    | 数据加载成功 |
| AsyncLoading | 数据加载中   |
| AsyncError   | 数据加载出错 |

```dart
// 使用 when 方法处理所有状态
final result = ref.watch(myProvider);

return result.when(
  data: (value) => Text('Data: $value'),
  loading: () => CircularProgressIndicator(),
  error: (err, stack) => Text('Error: $err'),
);

// 或者使用 switch 表达式（Dart 3.0+）
return switch (result) {
  AsyncData(:final value) => Text('Data: $value'),
  AsyncLoading() => CircularProgressIndicator(),
  AsyncError(:final error) => Text('Error: $error'),
};
```

#### 2.3.5 带参数的 FutureProvider（Family）

```dart
final userByIdProvider = FutureProvider.family<User, int>((ref, userId) async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users/$userId'),
  );
  return User.fromJson(jsonDecode(response.body));
});

// 使用
class UserDetail extends ConsumerWidget {
  final int userId;
  UserDetail(this.userId);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userByIdProvider(userId));
    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (err, stack) => Text('Error: $err'),
    );
  }
}
```

---

### 2.4 StreamProvider（流 Provider）

`StreamProvider` 用于监听流数据，如实时数据库、WebSocket 等。

#### 2.4.1 核心特性

| 特性     | 说明                           |
| -------- | ------------------------------ |
| 返回值   | AsyncValue<T>                  |
| 可变性   | 不可变（自动监听流）           |
| 适用场景 | 实时数据、Firestore、WebSocket |

#### 2.4.2 基础用法

```dart
// 定义 StreamProvider
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return FirebaseFirestore.instance
      .collection('messages')
      .orderBy('timestamp')
      .snapshots()
      .map((snapshot) => snapshot.docs
          .map((doc) => Message.fromFirestore(doc))
          .toList());
});
```

#### 2.4.3 完整示例

```dart
class ChatRoom extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider);

    return messagesAsync.when(
      data: (messages) => ListView.builder(
        itemCount: messages.length,
        itemBuilder: (context, index) {
          final message = messages[index];
          return ListTile(
            title: Text(message.text),
            subtitle: Text(message.sender),
          );
        },
      ),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (error, stack) => Center(child: Text('Error: $error')),
    );
  }
}
```

---

### 2.5 NotifierProvider（通知器 Provider）

`NotifierProvider` 是 Riverpod 2.0+ 引入的新类型，用于管理复杂的可变状态。

#### 2.5.1 核心特性

| 特性     | 说明                             |
| -------- | -------------------------------- |
| 返回值   | Notifier<T>                      |
| 可变性   | 可变（通过方法修改）             |
| 适用场景 | 复杂状态管理、需要业务逻辑的状态 |

#### 2.5.2 基础用法

```dart
// 定义 Notifier
class Counter extends Notifier<int> {
  @override
  int build() {
    // 返回初始状态
    return 0;
  }

  // 定义修改状态的方法
  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
  void add(int value) => state += value;
}

// 定义 NotifierProvider
final counterProvider = NotifierProvider<Counter, int>(() => Counter());
```

#### 2.5.3 完整示例

```dart
// 购物车 Notifier
class Cart extends Notifier<List<Product>> {
  @override
  List<Product> build() {
    return [];
  }

  void add(Product product) {
    state = [...state, product];
  }

  void remove(Product product) {
    state = state.where((p) => p.id != product.id).toList();
  }

  void clear() {
    state = [];
  }

  double get totalPrice {
    return state.fold(0, (sum, p) => sum + p.price);
  }
}

final cartProvider = NotifierProvider<Cart, List<Product>>(() => Cart());

// 使用
class CartPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);
    final cartNotifier = ref.read(cartProvider.notifier);

    return Scaffold(
      appBar: AppBar(
        title: Text('购物车'),
        actions: [
          IconButton(
            icon: Icon(Icons.delete),
            onPressed: cartNotifier.clear,
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: cart.length,
        itemBuilder: (context, index) {
          final product = cart[index];
          return ListTile(
            title: Text(product.name),
            subtitle: Text('\$${product.price}'),
            trailing: IconButton(
              icon: Icon(Icons.remove_circle),
              onPressed: () => cartNotifier.remove(product),
            ),
          );
        },
      ),
      bottomNavigationBar: BottomAppBar(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Text(
            '总计: \$${cartNotifier.totalPrice.toStringAsFixed(2)}',
            style: Theme.of(context).textTheme.titleLarge,
          ),
        ),
      ),
    );
  }
}
```

---

### 2.6 AsyncNotifierProvider（异步通知器 Provider）

`AsyncNotifierProvider` 用于管理复杂的异步状态。

#### 2.6.1 核心特性

| 特性     | 说明                     |
| -------- | ------------------------ |
| 返回值   | AsyncNotifier<T>         |
| 可变性   | 可变（通过异步方法修改） |
| 适用场景 | 需要异步操作的复杂状态   |

#### 2.6.2 基础用法

```dart
// 定义 AsyncNotifier
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    // 异步初始化
    return await _fetchUser();
  }

  Future<User> _fetchUser() async {
    final response = await http.get(
      Uri.parse('https://api.example.com/user'),
    );
    return User.fromJson(jsonDecode(response.body));
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(_fetchUser);
  }

  Future<void> updateProfile(String name) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final response = await http.patch(
        Uri.parse('https://api.example.com/user'),
        body: jsonEncode({'name': name}),
      );
      return User.fromJson(jsonDecode(response.body));
    });
  }
}

final userNotifierProvider = AsyncNotifierProvider<UserNotifier, User>(
  () => UserNotifier(),
);
```

#### 2.6.3 AsyncValue.guard 详解

`AsyncValue.guard` 是一个便捷方法，用于安全地执行异步操作：

```dart
// 使用 AsyncValue.guard 自动处理错误
state = await AsyncValue.guard(() async {
  final result = await someAsyncOperation();
  return result;
});

// 等价于
try {
  state = const AsyncLoading();
  final result = await someAsyncOperation();
  state = AsyncData(result);
} catch (error, stackTrace) {
  state = AsyncError(error, stackTrace);
}
```

---

### 2.7 StreamNotifierProvider（流通知器 Provider）

`StreamNotifierProvider` 用于管理流状态。

#### 2.7.1 基础用法

```dart
class MessageNotifier extends StreamNotifier<List<Message>> {
  @override
  Stream<List<Message>> build() {
    return FirebaseFirestore.instance
        .collection('messages')
        .snapshots()
        .map((snapshot) => snapshot.docs
            .map((doc) => Message.fromFirestore(doc))
            .toList());
  }

  Future<void> sendMessage(String text) async {
    await FirebaseFirestore.instance.collection('messages').add({
      'text': text,
      'timestamp': FieldValue.serverTimestamp(),
    });
  }
}

final messageNotifierProvider = StreamNotifierProvider<MessageNotifier, List<Message>>(
  () => MessageNotifier(),
);
```

---

### 2.8 Provider 类型选择指南

| Provider 类型          | 适用场景                     | 返回值类型         |
| ---------------------- | ---------------------------- | ------------------ |
| Provider               | 不可变对象、服务类、计算属性 | T                  |
| StateProvider          | 简单可变状态（基本类型）     | StateController<T> |
| FutureProvider         | 异步数据获取                 | AsyncValue<T>      |
| StreamProvider         | 流数据监听                   | AsyncValue<T>      |
| NotifierProvider       | 复杂可变状态                 | Notifier<T>        |
| AsyncNotifierProvider  | 复杂异步状态                 | AsyncNotifier<T>   |
| StreamNotifierProvider | 流状态管理                   | StreamNotifier<T>  |

---

## 第3章 Provider 修饰符

修饰符（Modifiers）用于调整 provider 的行为。

### 3.1 autoDispose（自动释放）

`autoDispose` 修饰符会在 provider 不再被使用时自动释放资源。

#### 3.1.1 基础用法

```dart
// 使用 autoDispose
final userProvider = FutureProvider.autoDispose<User>((ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
});
```

#### 3.1.2 自定义释放逻辑

```dart
final timerProvider = StreamProvider.autoDispose<int>((ref) {
  final timer = Timer.periodic(Duration(seconds: 1), (t) {});

  // 当 provider 被释放时执行
  ref.onDispose(() {
    timer.cancel();
  });

  return Stream.periodic(Duration(seconds: 1), (i) => i);
});
```

#### 3.1.3 保持存活时间

```dart
final searchProvider = FutureProvider.autoDispose<List<Result>>((ref) async {
  // 保持 provider 存活 5 分钟
  final link = ref.keepAlive();
  Timer(Duration(minutes: 5), link.close);

  final query = ref.watch(searchQueryProvider);
  return await searchApi(query);
});
```

---

### 3.2 family（参数化）

`family` 修饰符允许向 provider 传递参数。

#### 3.2.1 基础用法

```dart
// 使用 family 传递参数
final userByIdProvider = FutureProvider.family<User, int>((ref, userId) async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users/$userId'),
  );
  return User.fromJson(jsonDecode(response.body));
});

// 使用
final user1 = ref.watch(userByIdProvider(1));
final user2 = ref.watch(userByIdProvider(2));
```

#### 3.2.2 多个参数

```dart
// 使用 record（Dart 3.0+）传递多个参数
final searchProvider = FutureProvider.family<List<Result>, ({String query, int page})>(
  (ref, params) async {
    return await searchApi(params.query, params.page);
  },
);

// 使用
final results = ref.watch(searchProvider((query: 'flutter', page: 1)));
```

#### 3.2.3 结合 autoDispose

```dart
final productProvider = FutureProvider.autoDispose.family<Product, String>(
  (ref, productId) async {
    return await fetchProduct(productId);
  },
);
```

---

## 第4章 Ref 对象详解

`Ref` 对象是 provider 的核心，用于与其他 provider 交互。

### 4.1 ref.watch

`ref.watch` 用于监听其他 provider 的变化。

```dart
final firstNameProvider = Provider<String>((ref) => 'John');
final lastNameProvider = Provider<String>((ref) => 'Doe');

// 监听多个 provider
final fullNameProvider = Provider<String>((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);
  return '$firstName $lastName';
});
```

### 4.2 ref.read

`ref.read` 用于一次性读取 provider 的值，不建立监听关系。

```dart
// 在事件回调中使用
onPressed: () {
  ref.read(counterProvider.notifier).state++;
}
```

### 4.3 ref.listen

`ref.listen` 用于监听 provider 变化并执行副作用。

```dart
final counterProvider = StateProvider<int>((ref) => 0);

final loggerProvider = Provider<void>((ref) {
  ref.listen<int>(counterProvider, (previous, next) {
    print('Counter changed from $previous to $next');
  });
});
```

### 4.4 ref.onDispose

`ref.onDispose` 用于注册释放回调。

```dart
final streamProvider = StreamProvider<int>((ref) {
  final streamController = StreamController<int>();

  ref.onDispose(() {
    streamController.close();
  });

  return streamController.stream;
});
```

### 4.5 ref.invalidate

`ref.invalidate` 用于使 provider 缓存失效，强制重新计算。

```dart
// 强制刷新数据
ref.invalidate(userProvider);
```

### 4.6 ref.refresh

`ref.refresh` 用于立即重新计算 provider 并返回新值。

```dart
// 立即刷新并获取新值
final newUser = await ref.refresh(userProvider.future);
```

---

## 第5章 在 Widget 中使用 Riverpod

### 5.1 ConsumerWidget

`ConsumerWidget` 是 `StatelessWidget` 的替代，提供 `WidgetRef`。

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Text('Count: $count'),
    );
  }
}
```

### 5.2 ConsumerStatefulWidget

`ConsumerStatefulWidget` 是 `StatefulWidget` 的替代。

```dart
class HomePage extends ConsumerStatefulWidget {
  @override
  ConsumerState<HomePage> createState() => _HomePageState();
}

class _HomePageState extends ConsumerState<HomePage> {
  @override
  void initState() {
    super.initState();
    // 使用 ref
    ref.read(counterProvider);
  }

  @override
  Widget build(BuildContext context) {
    // 使用 ref
    final count = ref.watch(counterProvider);
    return Text('Count: $count');
  }
}
```

### 5.3 Consumer

`Consumer` 用于在 Widget 树的局部监听 provider。

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 只有这部分会重建
          Consumer(
            builder: (context, ref, child) {
              final count = ref.watch(counterProvider);
              return Text('Count: $count');
            },
          ),
          // 这部分不会重建
          Text('Static text'),
        ],
      ),
    );
  }
}
```

### 5.4 HookConsumerWidget（使用 hooks_riverpod）

```dart
class HomePage extends HookConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 使用 Flutter Hooks
    final controller = useTextController();
    final count = ref.watch(counterProvider);

    return TextField(controller: controller);
  }
}
```

---

## 第6章 代码生成（Riverpod Generator）

Riverpod 3.0+ 推荐使用代码生成来简化 provider 的创建。

### 6.1 配置代码生成

添加依赖：

```yaml
dev_dependencies:
  riverpod_generator: ^3.0.0
  build_runner: ^2.4.0
```

### 6.2 使用 @riverpod 注解

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'my_provider.g.dart';  // 生成的文件

// 同步 provider
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello, World!';
}

// 异步 provider
@riverpod
Future<User> fetchUser(FetchUserRef ref) async {
  final response = await http.get(Uri.parse('https://api.example.com/user'));
  return User.fromJson(jsonDecode(response.body));
}

// 带参数的 provider
@riverpod
Future<User> fetchUserById(FetchUserByIdRef ref, {required int userId}) async {
  final response = await http.get(
    Uri.parse('https://api.example.com/users/$userId'),
  );
  return User.fromJson(jsonDecode(response.body));
}
```

### 6.3 生成 Notifier

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
}

// 使用
final count = ref.watch(counterProvider);
ref.read(counterProvider.notifier).increment();
```

### 6.4 生成 AsyncNotifier

```dart
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User> build() async {
    return await _fetchUser();
  }

  Future<User> _fetchUser() async {
    final response = await http.get(
      Uri.parse('https://api.example.com/user'),
    );
    return User.fromJson(jsonDecode(response.body));
  }

  Future<void> updateName(String name) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final response = await http.patch(
        Uri.parse('https://api.example.com/user'),
        body: jsonEncode({'name': name}),
      );
      return User.fromJson(jsonDecode(response.body));
    });
  }
}
```

### 6.5 运行代码生成

```bash
# 一次性生成
dart run build_runner build

# 监视模式（开发时使用）
dart run build_runner watch

# 删除生成的文件
dart run build_runner build --delete-conflicting-outputs
```

---

## 第7章 高级特性

### 7.1 Provider 覆盖（Overrides）

用于测试或提供不同的实现。

```dart
void main() {
  runApp(
    ProviderScope(
      overrides: [
        // 覆盖 provider
        apiServiceProvider.overrideWithValue(MockApiService()),
      ],
      child: MyApp(),
    ),
  );
}
```

### 7.2 Scoped Provider

```dart
// 定义 scoped provider
final scopedProvider = Provider<String>((ref) => throw UnimplementedError());

// 在特定作用域提供值
ProviderScope(
  overrides: [
    scopedProvider.overrideWithValue('Scoped Value'),
  ],
  child: MyWidget(),
)
```

### 7.3 ProviderObserver

用于监听所有 provider 的变化。

```dart
class Logger extends ProviderObserver {
  @override
  void didAddProvider(
    ProviderBase<Object?> provider,
    Object? value,
    ProviderContainer container,
  ) {
    print('Provider $provider was initialized with $value');
  }

  @override
  void didDisposeProvider(
    ProviderBase<Object?> provider,
    ProviderContainer container,
  ) {
    print('Provider $provider was disposed');
  }

  @override
  void didUpdateProvider(
    ProviderBase<Object?> provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) {
    print('Provider $provider updated from $previousValue to $newValue');
  }
}

void main() {
  runApp(
    ProviderScope(
      observers: [Logger()],
      child: MyApp(),
    ),
  );
}
```

### 7.4 依赖注入

```dart
// 定义接口
abstract class AuthRepository {
  Future<User> signIn(String email, String password);
  Future<void> signOut();
}

// 实现类
class FirebaseAuthRepository implements AuthRepository {
  @override
  Future<User> signIn(String email, String password) async {
    // 实现
  }

  @override
  Future<void> signOut() async {
    // 实现
  }
}

// Provider
final authRepositoryProvider = Provider<AuthRepository>((ref) {
  return FirebaseAuthRepository();
});

// 在测试中使用 mock
class MockAuthRepository implements AuthRepository {
  @override
  Future<User> signIn(String email, String password) async {
    return User(id: '1', email: email);
  }

  @override
  Future<void> signOut() async {}
}

void main() {
  test('auth test', () async {
    final container = ProviderContainer(
      overrides: [
        authRepositoryProvider.overrideWithValue(MockAuthRepository()),
      ],
    );

    final auth = container.read(authRepositoryProvider);
    final user = await auth.signIn('test@test.com', 'password');
    expect(user.email, 'test@test.com');
  });
}
```

---

## 第8章 测试

### 8.1 测试 Provider

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  test('counter increments', () {
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // 读取初始值
    expect(container.read(counterProvider), 0);

    // 修改状态
    container.read(counterProvider.notifier).state++;

    // 验证新值
    expect(container.read(counterProvider), 1);
  });
}
```

### 8.2 测试异步 Provider

```dart
test('fetch user', () async {
  final container = ProviderContainer(
    overrides: [
      apiClientProvider.overrideWithValue(MockApiClient()),
    ],
  );
  addTearDown(container.dispose);

  // 监听 provider
  final listener = Listener<AsyncValue<User>>();
  container.listen(
    userProvider,
    listener,
    fireImmediately: true,
  );

  // 验证加载状态
  verify(listener(null, const AsyncLoading()));

  // 等待异步完成
  await container.read(userProvider.future);

  // 验证数据
  final user = container.read(userProvider).value;
  expect(user?.name, 'John Doe');
});
```

### 8.3 Widget 测试

```dart
testWidgets('counter widget test', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      child: MaterialApp(home: CounterPage()),
    ),
  );

  // 验证初始值
  expect(find.text('Count: 0'), findsOneWidget);

  // 点击按钮
  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();

  // 验证新值
  expect(find.text('Count: 1'), findsOneWidget);
});
```

---

## 第9章 实战案例

### 9.1 待办事项应用

```dart
// 模型
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  Todo copyWith({String? id, String? title, bool? isCompleted}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}

// Filter 枚举
enum TodoFilter { all, active, completed }

// 代码生成
part 'todo_provider.g.dart';

// Filter provider
@riverpod
class TodoFilterNotifier extends _$TodoFilterNotifier {
  @override
  TodoFilter build() => TodoFilter.all;

  void setFilter(TodoFilter filter) => state = filter;
}

// Todos provider
@riverpod
class TodosNotifier extends _$TodosNotifier {
  @override
  List<Todo> build() => [];

  void add(String title) {
    state = [
      ...state,
      Todo(
        id: DateTime.now().toString(),
        title: title,
      ),
    ];
  }

  void toggle(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(isCompleted: !todo.isCompleted);
      }
      return todo;
    }).toList();
  }

  void remove(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}

// 过滤后的 todos
@riverpod
List<Todo> filteredTodos(FilteredTodosRef ref) {
  final todos = ref.watch(todosNotifierProvider);
  final filter = ref.watch(todoFilterNotifierProvider);

  switch (filter) {
    case TodoFilter.active:
      return todos.where((t) => !t.isCompleted).toList();
    case TodoFilter.completed:
      return todos.where((t) => t.isCompleted).toList();
    case TodoFilter.all:
    default:
      return todos;
  }
}

// 统计
@riverpod
class TodoStats extends _$TodoStats {
  @override
  ({int total, int active, int completed}) build() {
    final todos = ref.watch(todosNotifierProvider);
    return (
      total: todos.length,
      active: todos.where((t) => !t.isCompleted).length,
      completed: todos.where((t) => t.isCompleted).length,
    );
  }
}

// UI
class TodoPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(filteredTodosProvider);
    final stats = ref.watch(todoStatsProvider);

    return Scaffold(
      appBar: AppBar(
        title: Text('Todos'),
        actions: [
          DropdownButton<TodoFilter>(
            value: ref.watch(todoFilterNotifierProvider),
            onChanged: (filter) {
              if (filter != null) {
                ref.read(todoFilterNotifierProvider.notifier).setFilter(filter);
              }
            },
            items: TodoFilter.values.map((filter) {
              return DropdownMenuItem(
                value: filter,
                child: Text(filter.name),
              );
            }).toList(),
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Text('Total: ${stats.total}, Active: ${stats.active}, Completed: ${stats.completed}'),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: todos.length,
              itemBuilder: (context, index) {
                final todo = todos[index];
                return ListTile(
                  leading: Checkbox(
                    value: todo.isCompleted,
                    onChanged: (_) {
                      ref.read(todosNotifierProvider.notifier).toggle(todo.id);
                    },
                  ),
                  title: Text(
                    todo.title,
                    style: TextStyle(
                      decoration: todo.isCompleted
                          ? TextDecoration.lineThrough
                          : null,
                    ),
                  ),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () {
                      ref.read(todosNotifierProvider.notifier).remove(todo.id);
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          showDialog(
            context: context,
            builder: (context) => AddTodoDialog(),
          );
        },
        child: Icon(Icons.add),
      ),
    );
  }
}

class AddTodoDialog extends ConsumerStatefulWidget {
  @override
  ConsumerState<AddTodoDialog> createState() => _AddTodoDialogState();
}

class _AddTodoDialogState extends ConsumerState<AddTodoDialog> {
  final _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text('Add Todo'),
      content: TextField(
        controller: _controller,
        autofocus: true,
        decoration: InputDecoration(hintText: 'What needs to be done?'),
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text('Cancel'),
        ),
        TextButton(
          onPressed: () {
            if (_controller.text.isNotEmpty) {
              ref.read(todosNotifierProvider.notifier).add(_controller.text);
              Navigator.pop(context);
            }
          },
          child: Text('Add'),
        ),
      ],
    );
  }
}
```

---

### 9.2 用户认证系统

```dart
// 用户模型
class User {
  final String id;
  final String email;
  final String name;

  User({required this.id, required this.email, required this.name});
}

// 认证状态
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  Future<User?> build() async {
    // 检查本地存储的 token
    final token = await _getStoredToken();
    if (token != null) {
      return await _fetchUser(token);
    }
    return null;
  }

  Future<void> signIn(String email, String password) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final response = await http.post(
        Uri.parse('https://api.example.com/auth/login'),
        body: jsonEncode({'email': email, 'password': password}),
      );

      final data = jsonDecode(response.body);
      await _storeToken(data['token']);

      return User(
        id: data['user']['id'],
        email: data['user']['email'],
        name: data['user']['name'],
      );
    });
  }

  Future<void> signOut() async {
    await _clearToken();
    state = const AsyncData(null);
  }

  Future<String?> _getStoredToken() async {
    // 实现
  }

  Future<void> _storeToken(String token) async {
    // 实现
  }

  Future<void> _clearToken() async {
    // 实现
  }

  Future<User> _fetchUser(String token) async {
    // 实现
  }
}

// 受保护的路由
class ProtectedRoute extends ConsumerWidget {
  final Widget child;

  ProtectedRoute({required this.child});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authNotifierProvider);

    return authState.when(
      data: (user) {
        if (user == null) {
          return LoginPage();
        }
        return child;
      },
      loading: () => Scaffold(
        body: Center(child: CircularProgressIndicator()),
      ),
      error: (_, __) => LoginPage(),
    );
  }
}
```

---

### 9.3 分页列表

```dart
@riverpod
class PaginatedItems extends _$PaginatedItems {
  @override
  Future<List<Item>> build({required int page}) async {
    return await fetchItems(page: page);
  }

  Future<void> loadMore() async {
    final currentItems = state.value ?? [];
    final nextPage = (currentItems.length ~/ 20) + 1;

    final newItems = await fetchItems(page: nextPage);
    state = AsyncData([...currentItems, ...newItems]);
  }
}

class PaginatedList extends ConsumerStatefulWidget {
  @override
  ConsumerState<PaginatedList> createState() => _PaginatedListState();
}

class _PaginatedListState extends ConsumerState<PaginatedList> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent * 0.8) {
      ref.read(paginatedItemsProvider(page: 1).notifier).loadMore();
    }
  }

  @override
  Widget build(BuildContext context) {
    final itemsAsync = ref.watch(paginatedItemsProvider(page: 1));

    return itemsAsync.when(
      data: (items) => ListView.builder(
        controller: _scrollController,
        itemCount: items.length,
        itemBuilder: (context, index) {
          return ListTile(title: Text(items[index].name));
        },
      ),
      loading: () => Center(child: CircularProgressIndicator()),
      error: (err, stack) => Center(child: Text('Error: $err')),
    );
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## 第10章 最佳实践

### 10.1 项目结构建议

```
lib/
├── main.dart
├── app.dart                    # 应用根组件
├── providers/                  # Provider 定义
│   ├── auth_provider.dart
│   ├── user_provider.dart
│   └── providers.dart          # 导出所有 providers
├── models/                     # 数据模型
│   ├── user.dart
│   ├── todo.dart
│   └── models.dart
├── services/                   # 服务层
│   ├── api_service.dart
│   ├── storage_service.dart
│   └── services.dart
├── repositories/               # 数据仓库
│   ├── auth_repository.dart
│   └── repositories.dart
├── pages/                      # 页面
│   ├── home/
│   │   ├── home_page.dart
│   │   └── widgets/
│   └── login/
│       └── login_page.dart
├── widgets/                    # 共享组件
│   ├── common/
│   └── widgets.dart
└── utils/                      # 工具函数
    └── extensions.dart
```

### 10.2 命名规范

| 类型              | 命名规范              | 示例                     |
| ----------------- | --------------------- | ------------------------ |
| Provider          | `xxxProvider`         | `userProvider`           |
| Notifier          | `XxxNotifier`         | `UserNotifier`           |
| Notifier Provider | `xxxNotifierProvider` | `userNotifierProvider`   |
| State Provider    | `xxxStateProvider`    | `counterStateProvider`   |
| Future Provider   | `xxxFutureProvider`   | `userFutureProvider`     |
| Stream Provider   | `xxxStreamProvider`   | `messagesStreamProvider` |

### 10.3 DO 和 DON'T

#### ✅ DO

- 将 providers 声明为全局变量
- 使用 `ref.watch` 在 build 方法中
- 使用 `ref.read` 在事件回调中
- 使用代码生成
- 保持 provider 的单一职责
- 使用 `autoDispose` 避免内存泄漏

#### ❌ DON'T

- 在 provider 中创建可变状态
- 在 build 方法中使用 `ref.read`
- 在 provider 中直接修改其他 provider
- 使用 `ChangeNotifierProvider`（已弃用）
- 在 provider 中进行副作用操作

---

## 附录：Riverpod API 速查表

### Provider 类型

| 类型                   | 用途         | 返回值             |
| ---------------------- | ------------ | ------------------ |
| Provider               | 不可变对象   | T                  |
| StateProvider          | 简单状态     | StateController<T> |
| FutureProvider         | 异步数据     | AsyncValue<T>      |
| StreamProvider         | 流数据       | AsyncValue<T>      |
| NotifierProvider       | 复杂状态     | Notifier<T>        |
| AsyncNotifierProvider  | 复杂异步状态 | AsyncNotifier<T>   |
| StreamNotifierProvider | 流状态       | StreamNotifier<T>  |

### Ref 方法

| 方法           | 用途               |
| -------------- | ------------------ |
| ref.watch      | 监听 provider 变化 |
| ref.read       | 一次性读取         |
| ref.listen     | 监听并执行副作用   |
| ref.onDispose  | 注册释放回调       |
| ref.invalidate | 使缓存失效         |
| ref.refresh    | 立即刷新           |
| ref.keepAlive  | 保持存活           |

### 修饰符

| 修饰符       | 用途       |
| ------------ | ---------- |
| .autoDispose | 自动释放   |
| .family      | 参数化     |
| .select      | 选择性监听 |

### AsyncValue 方法

| 方法         | 用途          |
| ------------ | ------------- |
| .when        | 处理所有状态  |
| .map         | 函数式处理    |
| .maybeWhen   | 可选处理      |
| .valueOrNull | 获取值或 null |
| .hasValue    | 是否有值      |
| .hasError    | 是否有错误    |
| .isLoading   | 是否加载中    |

---

## 结语

Riverpod 3.2.1 是 Flutter 生态中最强大的状态管理解决方案之一。通过本书的学习，你应该已经掌握了：

1. Riverpod 的核心概念和设计哲学
2. 各种 Provider 类型的使用场景
3. 代码生成简化开发流程
4. 高级特性如依赖注入、测试、Observer
5. 实战案例的开发经验

Riverpod 的设计让状态管理变得简单、安全、可测试。希望本书能够帮助你在 Flutter 开发中更好地使用 Riverpod，构建出高质量的应用程序。

---

**版本信息**

- Riverpod 版本：3.2.1
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
