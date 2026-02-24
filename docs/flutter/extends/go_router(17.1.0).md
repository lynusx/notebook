# Flutter go_router 17.1.0 完整详解与实战指南

## 前言

Flutter 的导航系统经历了从 Navigator 1.0 到 Navigator 2.0 的重大变革。Navigator 2.0 提供了声明式的路由管理方式，但其 API 相对复杂，让许多开发者望而却步。`go_router` 是由 Flutter 团队官方维护的声明式路由包，它基于 Navigator 2.0 构建，提供了简洁的 URL 驱动 API，让导航变得简单直观。

`go_router` 17.1.0 是目前最新的稳定版本，它提供了丰富的功能：

- 声明式路由配置
- 路径和查询参数解析
- 深度链接支持
- 重定向和路由守卫
- 嵌套导航（ShellRoute）
- 类型安全路由（通过代码生成）
- 自定义转场动画
- Web 支持

本书将详细介绍 `go_router` 17.1.0 的所有核心概念、API 用法，以及如何将其集成到 Flutter 应用中。

---

## 第1章 go_router 基础

### 1.1 什么是 go_router

`go_router` 是一个声明式路由包，它使用 Flutter 的 Router API 提供基于 URL 的导航 API。主要特点包括：

| 特性       | 说明                       |
| ---------- | -------------------------- |
| 声明式配置 | 通过代码声明路由结构       |
| URL 驱动   | 使用 URL 进行导航          |
| 深度链接   | 原生支持深度链接           |
| 路径参数   | 支持 `:id` 格式的路径参数  |
| 查询参数   | 支持 `?key=value` 查询参数 |
| 重定向     | 支持基于状态的重定向       |
| 嵌套导航   | 通过 ShellRoute 实现       |
| 类型安全   | 支持代码生成类型安全路由   |

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^17.1.0

# 可选：类型安全路由代码生成
dev_dependencies:
  build_runner: ^2.4.0
  go_router_builder: ^2.7.0
```

#### 1.2.2 基础项目结构

```
lib/
├── main.dart              # 应用入口
├── router/                # 路由配置
│   ├── app_router.dart    # 路由配置
│   └── routes.dart        # 路由定义
├── pages/                 # 页面
│   ├── home_page.dart
│   ├── profile_page.dart
│   └── settings_page.dart
└── models/                # 数据模型
    └── user.dart
```

---

### 1.3 第一个 go_router 应用

#### 1.3.1 创建路由配置

```dart
// router/app_router.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../pages/home_page.dart';
import '../pages/profile_page.dart';

final GoRouter appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfilePage(),
    ),
  ],
);
```

#### 1.3.2 在应用中使用

```dart
// main.dart
import 'package:flutter/material.dart';
import 'router/app_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'GoRouter Demo',
      routerConfig: appRouter,
    );
  }
}
```

#### 1.3.3 页面实现

```dart
// pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('首页')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.go('/profile'),
          child: const Text('前往个人资料'),
        ),
      ),
    );
  }
}

// pages/profile_page.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('个人资料'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: () => context.pop(),
        ),
      ),
      body: const Center(
        child: Text('这是个人资料页面'),
      ),
    );
  }
}
```

> **Flutter框架小知识**
>
> `MaterialApp.router` 是 Flutter 为 Router API 提供的专用构造函数。它接受 `routerConfig` 参数，该参数需要实现 `RouterConfig` 接口。`GoRouter` 实现了这个接口，因此可以直接作为 `routerConfig` 的值。

---

## 第2章 GoRouter 核心类详解

### 2.1 GoRouter

`GoRouter` 是整个路由系统的核心类，负责管理所有路由配置。

#### 2.1.1 构造函数参数

| 参数名              | 类型                       | 默认值 | 说明               |
| ------------------- | -------------------------- | ------ | ------------------ |
| routes              | List<RouteBase>            | 必填   | 路由列表           |
| initialLocation     | String?                    | null   | 初始路由路径       |
| navigatorKey        | GlobalKey<NavigatorState>? | null   | Navigator 的 Key   |
| debugLogDiagnostics | bool                       | false  | 是否输出调试日志   |
| redirect            | GoRouterRedirect?          | null   | 全局重定向函数     |
| redirectLimit       | int                        | 5      | 最大重定向次数     |
| routerNeglect       | bool                       | false  | 是否忽略浏览器历史 |
| extraCodec          | Codec<Object?, Object?>?   | null   | extra 参数编解码器 |
| onException         | GoExceptionHandler?        | null   | 异常处理函数       |
| errorPageBuilder    | GoRouterPageBuilder?       | null   | 错误页面构建器     |
| errorBuilder        | GoRouterWidgetBuilder?     | null   | 错误 Widget 构建器 |
| restorationScopeId  | String?                    | null   | 状态恢复 ID        |
| observers           | List<NavigatorObserver>?   | null   | 导航观察者列表     |
| refreshListenable   | Listenable?                | null   | 刷新监听器         |
| requestFocus        | bool                       | true   | 是否请求焦点       |

#### 2.1.2 完整配置示例

```dart
final GoRouter appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  routes: [
    // 路由定义
  ],
  redirect: (context, state) {
    // 全局重定向逻辑
    return null;
  },
  errorBuilder: (context, state) {
    return ErrorPage(error: state.error);
  },
  observers: [
    MyNavigatorObserver(),
  ],
);
```

#### 2.1.3 常用方法

| 方法                                | 返回值     | 说明           |
| ----------------------------------- | ---------- | -------------- |
| go(String location)                 | void       | 导航到指定路径 |
| push(String location)               | Future<T?> | 推入新页面     |
| pop<T extends Object?>([T? result]) | void       | 返回上一页     |
| replace(String location)            | void       | 替换当前页面   |
| refresh()                           | void       | 刷新当前路由   |

---

### 2.2 GoRoute

`GoRoute` 定义单个路由配置。

#### 2.2.1 构造函数参数

| 参数名             | 类型                       | 默认值   | 说明             |
| ------------------ | -------------------------- | -------- | ---------------- |
| path               | String                     | 必填     | 路由路径         |
| name               | String?                    | null     | 路由名称         |
| builder            | GoRouterWidgetBuilder?     | null     | Widget 构建器    |
| pageBuilder        | GoRouterPageBuilder?       | null     | Page 构建器      |
| parentNavigatorKey | GlobalKey<NavigatorState>? | null     | 父 Navigator Key |
| redirect           | GoRouterRedirect?          | null     | 路由级重定向     |
| onExit             | ExitCallback?              | null     | 退出回调         |
| routes             | List<RouteBase>            | const [] | 子路由列表       |

#### 2.2.2 基础用法

```dart
GoRoute(
  path: '/user/:id',
  name: 'user',
  builder: (context, state) {
    final userId = state.pathParameters['id'];
    return UserPage(userId: userId!);
  },
)
```

#### 2.2.3 使用 pageBuilder 自定义转场

```dart
GoRoute(
  path: '/details',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const DetailsPage(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(opacity: animation, child: child);
      },
    );
  },
)
```

---

### 2.3 GoRouterState

`GoRouterState` 包含当前路由的状态信息。

#### 2.3.1 属性

| 属性名          | 类型                | 说明           |
| --------------- | ------------------- | -------------- |
| location        | String              | 完整 URL       |
| matchedLocation | String              | 匹配的路径     |
| name            | String?             | 路由名称       |
| path            | String              | 路由路径模式   |
| fullPath        | String              | 完整路径模式   |
| pathParameters  | Map<String, String> | 路径参数       |
| uri             | Uri                 | URI 对象       |
| pageKey         | ValueKey<String>    | 页面 Key       |
| error           | Exception?          | 错误信息       |
| extra           | Object?             | 额外传递的数据 |

#### 2.3.2 使用示例

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    // 获取路径参数
    final productId = state.pathParameters['id'];

    // 获取查询参数
    final sort = state.uri.queryParameters['sort'];

    // 获取 extra 数据
    final extraData = state.extra as Map<String, dynamic>?;

    return ProductPage(
      id: productId!,
      sort: sort,
      extraData: extraData,
    );
  },
)
```

---

## 第3章 导航方式详解

### 3.1 context.go()

`go()` 方法导航到指定路径，替换导航栈。

```dart
// 基础导航
context.go('/home');

// 带路径参数
context.go('/user/123');

// 带查询参数
context.go('/search?query=flutter&page=1');

// 使用 Uri
context.go(Uri(path: '/user', queryParameters: {'id': '123'}).toString());
```

### 3.2 context.push()

`push()` 方法推入新页面到导航栈，保留当前页面。

```dart
// 推入新页面
context.push('/details');

// 等待返回结果
final result = await context.push<bool>('/confirm');
if (result == true) {
  // 用户确认
}
```

### 3.3 context.pop()

`pop()` 方法返回上一页。

```dart
// 简单返回
context.pop();

// 返回并传递结果
context.pop(true);
context.pop({'success': true, 'data': data});
```

### 3.4 context.replace()

`replace()` 方法替换当前页面。

```dart
context.replace('/new-page');
```

### 3.5 命名路由导航

```dart
// 定义命名路由
GoRoute(
  path: '/user/:id',
  name: 'user',
  builder: (context, state) => UserPage(id: state.pathParameters['id']!),
)

// 使用命名路由导航
goNamed('user', pathParameters: {'id': '123'});

// 带查询参数
pushNamed('search', queryParameters: {'q': 'flutter'});

// 带 extra 数据
pushNamed('details', extra: {'product': product});
```

#### 3.5.1 命名路由参数表

| 参数名          | 类型                 | 说明     |
| --------------- | -------------------- | -------- |
| name            | String               | 路由名称 |
| pathParameters  | Map<String, String>? | 路径参数 |
| queryParameters | Map<String, String>? | 查询参数 |
| extra           | Object?              | 额外数据 |

---

## 第4章 路径参数与查询参数

### 4.1 路径参数

路径参数使用 `:paramName` 语法定义。

```dart
GoRoute(
  path: '/user/:userId/posts/:postId',
  builder: (context, state) {
    final userId = state.pathParameters['userId'];
    final postId = state.pathParameters['postId'];
    return PostPage(userId: userId!, postId: postId!);
  },
)

// 导航
context.go('/user/123/posts/456');
```

### 4.2 可选路径参数

```dart
GoRoute(
  path: '/user/:userId/posts/:postId?',
  builder: (context, state) {
    final userId = state.pathParameters['userId'];
    final postId = state.pathParameters['postId']; // 可能为 null
    return PostsPage(userId: userId!, postId: postId);
  },
)

// 两种导航方式都有效
context.go('/user/123/posts');
context.go('/user/123/posts/456');
```

### 4.3 查询参数

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'];
    final page = int.tryParse(state.uri.queryParameters['page'] ?? '1');
    final sort = state.uri.queryParameters['sort'] ?? 'relevance';

    return SearchPage(
      query: query,
      page: page!,
      sort: sort,
    );
  },
)

// 导航
context.go('/search?q=flutter&page=2&sort=date');
```

### 4.4 参数验证

```dart
GoRoute(
  path: '/user/:userId',
  redirect: (context, state) {
    final userId = state.pathParameters['userId'];
    if (userId == null || int.tryParse(userId) == null) {
      return '/error/invalid-user-id';
    }
    return null;
  },
  builder: (context, state) => UserPage(id: state.pathParameters['userId']!),
)
```

---

## 第5章 嵌套路由与子路由

### 5.1 基础子路由

```dart
GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'details',  // 完整路径: /details
          builder: (context, state) => const DetailsPage(),
        ),
        GoRoute(
          path: 'settings',  // 完整路径: /settings
          builder: (context, state) => const SettingsPage(),
        ),
      ],
    ),
  ],
)
```

### 5.2 多级嵌套路由

```dart
GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'shop',
          builder: (context, state) => const ShopPage(),
          routes: [
            GoRoute(
              path: 'products',
              builder: (context, state) => const ProductsPage(),
              routes: [
                GoRoute(
                  path: ':id',
                  builder: (context, state) => ProductPage(
                    id: state.pathParameters['id']!,
                  ),
                ),
              ],
            ),
          ],
        ),
      ],
    ),
  ],
)
```

---

## 第6章 ShellRoute 嵌套导航

`ShellRoute` 用于创建嵌套导航器，常用于底部导航栏、侧边栏等场景。

### 6.1 基础 ShellRoute

```dart
GoRouter(
  routes: [
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithNavBar(child: child);
      },
      routes: [
        GoRoute(
          path: '/',
          builder: (context, state) => const HomePage(),
        ),
        GoRoute(
          path: '/search',
          builder: (context, state) => const SearchPage(),
        ),
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfilePage(),
        ),
      ],
    ),
  ],
)
```

### 6.2 带底部导航栏的 ShellRoute

```dart
class ScaffoldWithNavBar extends StatelessWidget {
  const ScaffoldWithNavBar({
    required this.child,
    super.key,
  });

  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: '首页'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: '搜索'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: '我的'),
        ],
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
      ),
    );
  }

  static int _calculateSelectedIndex(BuildContext context) {
    final String location = GoRouterState.of(context).matchedLocation;
    if (location.startsWith('/search')) return 1;
    if (location.startsWith('/profile')) return 2;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/');
        break;
      case 1:
        context.go('/search');
        break;
      case 2:
        context.go('/profile');
        break;
    }
  }
}
```

### 6.3 StatefulShellRoute（状态保持）

`StatefulShellRoute` 可以保持每个分支的导航状态。

```dart
GoRouter(
  routes: [
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/',
              builder: (context, state) => const HomePage(),
              routes: [
                GoRoute(
                  path: 'details',
                  builder: (context, state) => const HomeDetailsPage(),
                ),
              ],
            ),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/search',
              builder: (context, state) => const SearchPage(),
            ),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              builder: (context, state) => const ProfilePage(),
            ),
          ],
        ),
      ],
    ),
  ],
)

// 使用 StatefulNavigationShell
class ScaffoldWithNavBar extends StatelessWidget {
  const ScaffoldWithNavBar({
    required this.navigationShell,
    super.key,
  });

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: '首页'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: '搜索'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: '我的'),
        ],
        currentIndex: navigationShell.currentIndex,
        onTap: navigationShell.goBranch,
      ),
    );
  }
}
```

---

## 第7章 重定向与路由守卫

### 7.1 全局重定向

```dart
final GoRouter appRouter = GoRouter(
  routes: [...],
  redirect: (context, state) {
    final isLoggedIn = AuthService.instance.isLoggedIn;
    final isLoggingIn = state.matchedLocation == '/login';

    // 未登录且不在登录页，重定向到登录页
    if (!isLoggedIn && !isLoggingIn) {
      return '/login';
    }

    // 已登录但在登录页，重定向到首页
    if (isLoggedIn && isLoggingIn) {
      return '/';
    }

    return null;
  },
);
```

### 7.2 路由级重定向

```dart
GoRoute(
  path: '/admin',
  redirect: (context, state) {
    if (!AuthService.instance.isAdmin) {
      return '/unauthorized';
    }
    return null;
  },
  builder: (context, state) => const AdminPage(),
)
```

### 7.3 带状态监听的重定向

```dart
class AuthService extends ChangeNotifier {
  bool _isLoggedIn = false;
  bool get isLoggedIn => _isLoggedIn;

  void login() {
    _isLoggedIn = true;
    notifyListeners();
  }

  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

final authService = AuthService();

final GoRouter appRouter = GoRouter(
  refreshListenable: authService,
  redirect: (context, state) {
    final isLoggedIn = authService.isLoggedIn;
    final isLoggingIn = state.matchedLocation == '/login';

    if (!isLoggedIn && !isLoggingIn) {
      return '/login?from=${state.matchedLocation}';
    }

    if (isLoggedIn && isLoggingIn) {
      final from = state.uri.queryParameters['from'];
      return from ?? '/';
    }

    return null;
  },
  routes: [...],
);
```

### 7.4 登录后返回原页面

```dart
// 登录页
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final from = GoRouterState.of(context).uri.queryParameters['from'];

    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            authService.login();
            // 重定向会自动处理
          },
          child: const Text('登录'),
        ),
      ),
    );
  }
}
```

---

## 第8章 转场动画

### 8.1 默认转场

```dart
GoRoute(
  path: '/page',
  builder: (context, state) => const MyPage(),
)
```

### 8.2 自定义转场

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const MyPage(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return SlideTransition(
          position: Tween<Offset>(
            begin: const Offset(1, 0),
            end: Offset.zero,
          ).animate(animation),
          child: child,
        );
      },
    );
  },
)
```

### 8.3 常用转场动画

```dart
// 淡入
FadeTransition(opacity: animation, child: child)

// 滑动（从右到左）
SlideTransition(
  position: Tween<Offset>(
    begin: const Offset(1, 0),
    end: Offset.zero,
  ).animate(animation),
  child: child,
)

// 缩放
ScaleTransition(scale: animation, child: child)

// 旋转
RotationTransition(turns: animation, child: child)

// 组合动画
FadeTransition(
  opacity: animation,
  child: SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(0, 1),
      end: Offset.zero,
    ).animate(animation),
    child: child,
  ),
)
```

### 8.4 无转场动画

```dart
GoRoute(
  path: '/page',
  pageBuilder: (context, state) {
    return NoTransitionPage(
      key: state.pageKey,
      child: const MyPage(),
    );
  },
)
```

---

## 第9章 错误处理

### 9.1 自定义错误页面

```dart
final GoRouter appRouter = GoRouter(
  routes: [...],
  errorBuilder: (context, state) {
    return Scaffold(
      appBar: AppBar(title: const Text('页面未找到')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, size: 64, color: Colors.red),
            const SizedBox(height: 16),
            Text(
              '404',
              style: Theme.of(context).textTheme.headlineLarge,
            ),
            const SizedBox(height: 8),
            Text('页面未找到: ${state.matchedLocation}'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.go('/'),
              child: const Text('返回首页'),
            ),
          ],
        ),
      ),
    );
  },
)
```

### 9.2 使用 errorPageBuilder

```dart
final GoRouter appRouter = GoRouter(
  routes: [...],
  errorPageBuilder: (context, state) {
    return MaterialPage(
      key: state.pageKey,
      child: ErrorPage(error: state.error),
    );
  },
)
```

### 9.3 onException 处理

```dart
final GoRouter appRouter = GoRouter(
  routes: [...],
  onException: (context, state, router) {
    // 记录错误日志
    debugPrint('路由错误: ${state.error}');

    // 重定向到错误页面
    router.go('/error');
  },
)
```

---

## 第10章 深度链接

### 10.1 Android 配置

在 `AndroidManifest.xml` 中添加：

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    ...>
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https" android:host="example.com" />
    </intent-filter>

    <!-- 自定义 scheme -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
</activity>
```

### 10.2 iOS 配置

在 `Info.plist` 中添加：

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>com.example.app</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

添加 Associated Domains（通用链接）：

```xml
<key>com.apple.developer.associated-domains</key>
<array>
    <string>applinks:example.com</string>
</array>
```

### 10.3 测试深度链接

```bash
# Android
adb shell am start -a android.intent.action.VIEW \
  -c android.intent.category.BROWSABLE \
  -d "https://example.com/product/123" \
  com.example.app

# iOS
xcrun simctl openurl booted "myapp://product/123"
```

---

## 第11章 类型安全路由（代码生成）

### 11.1 配置代码生成

添加依赖：

```yaml
dev_dependencies:
  build_runner: ^2.4.0
  go_router_builder: ^2.7.0
```

### 11.2 定义类型安全路由

```dart
// routes.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part 'routes.g.dart';

@TypedGoRoute<HomeRoute>(
  path: '/',
)
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomePage();
  }
}

@TypedGoRoute<UserRoute>(
  path: '/user/:userId',
)
class UserRoute extends GoRouteData {
  const UserRoute({required this.userId});

  final String userId;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return UserPage(userId: userId);
  }
}

@TypedGoRoute<SearchRoute>(
  path: '/search',
)
class SearchRoute extends GoRouteData {
  const SearchRoute({this.query, this.page = 1});

  final String? query;
  final int page;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return SearchPage(query: query, page: page);
  }
}
```

### 11.3 使用生成的路由

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'routes.dart';

final GoRouter appRouter = GoRouter(
  routes: $appRoutes,
);

// 导航
// 类型安全！
const HomeRoute().go(context);
const UserRoute(userId: '123').push(context);
const SearchRoute(query: 'flutter', page: 2).go(context);
```

### 11.4 运行代码生成

```bash
# 一次性生成
dart run build_runner build

# 监视模式
dart run build_runner watch
```

---

## 第12章 实战案例

### 12.1 电商应用路由结构

```dart
final GoRouter appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: 'product/:id',
          builder: (context, state) => ProductPage(
            id: state.pathParameters['id']!,
          ),
        ),
        GoRoute(
          path: 'category/:categoryId',
          builder: (context, state) => CategoryPage(
            categoryId: state.pathParameters['categoryId']!,
          ),
        ),
      ],
    ),
    GoRoute(
      path: '/cart',
      builder: (context, state) => const CartPage(),
    ),
    GoRoute(
      path: '/checkout',
      builder: (context, state) => const CheckoutPage(),
      redirect: (context, state) {
        if (!AuthService.instance.isLoggedIn) {
          return '/login?from=/checkout';
        }
        return null;
      },
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginPage(
        from: state.uri.queryParameters['from'],
      ),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfilePage(),
      redirect: (context, state) {
        if (!AuthService.instance.isLoggedIn) {
          return '/login?from=/profile';
        }
        return null;
      },
      routes: [
        GoRoute(
          path: 'orders',
          builder: (context, state) => const OrdersPage(),
        ),
        GoRoute(
          path: 'settings',
          builder: (context, state) => const SettingsPage(),
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) => ErrorPage(
    error: state.error,
  ),
);
```

### 12.2 带底部导航的完整应用

```dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();
final _shellNavigatorKey = GlobalKey<NavigatorState>();

final GoRouter appRouter = GoRouter(
  navigatorKey: _rootNavigatorKey,
  initialLocation: '/',
  routes: [
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // 首页分支
        StatefulShellBranch(
          navigatorKey: _shellNavigatorKey,
          routes: [
            GoRoute(
              path: '/',
              builder: (context, state) => const HomePage(),
              routes: [
                GoRoute(
                  path: 'details/:id',
                  builder: (context, state) => DetailsPage(
                    id: state.pathParameters['id']!,
                  ),
                ),
              ],
            ),
          ],
        ),
        // 搜索分支
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/search',
              builder: (context, state) => const SearchPage(),
            ),
          ],
        ),
        // 购物车分支
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/cart',
              builder: (context, state) => const CartPage(),
            ),
          ],
        ),
        // 我的分支
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              builder: (context, state) => const ProfilePage(),
            ),
          ],
        ),
      ],
    ),
  ],
);

class ScaffoldWithNavBar extends StatelessWidget {
  const ScaffoldWithNavBar({
    required this.navigationShell,
    super.key,
  });

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: navigationShell.goBranch,
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: '首页'),
          NavigationDestination(icon: Icon(Icons.search), label: '搜索'),
          NavigationDestination(icon: Icon(Icons.shopping_cart), label: '购物车'),
          NavigationDestination(icon: Icon(Icons.person), label: '我的'),
        ],
      ),
    );
  }
}
```

### 12.3 认证流程完整示例

```dart
class AuthService extends ChangeNotifier {
  static final AuthService _instance = AuthService._internal();
  factory AuthService() => _instance;
  AuthService._internal();

  bool _isLoggedIn = false;
  bool get isLoggedIn => _isLoggedIn;

  void login() {
    _isLoggedIn = true;
    notifyListeners();
  }

  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

final authService = AuthService();

final GoRouter appRouter = GoRouter(
  refreshListenable: authService,
  redirect: (context, state) {
    final isLoggedIn = authService.isLoggedIn;
    final isAuthRoute = state.matchedLocation == '/login' ||
                        state.matchedLocation == '/register';

    // 未登录且不在认证页面，重定向到登录
    if (!isLoggedIn && !isAuthRoute) {
      return '/login?from=${Uri.encodeComponent(state.matchedLocation)}';
    }

    // 已登录但在认证页面，重定向到首页或原页面
    if (isLoggedIn && isAuthRoute) {
      final from = state.uri.queryParameters['from'];
      return from != null && from.isNotEmpty ? from : '/';
    }

    return null;
  },
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginPage(
        redirectTo: state.uri.queryParameters['from'],
      ),
    ),
    GoRoute(
      path: '/register',
      builder: (context, state) => const RegisterPage(),
    ),
    ShellRoute(
      builder: (context, state, child) => MainScaffold(child: child),
      routes: [
        GoRoute(
          path: '/',
          builder: (context, state) => const DashboardPage(),
        ),
        GoRoute(
          path: '/settings',
          builder: (context, state) => const SettingsPage(),
        ),
      ],
    ),
  ],
);
```

---

## 第13章 最佳实践

### 13.1 路由常量定义

```dart
// constants/routes.dart
class AppRoutes {
  static const String home = '/';
  static const String login = '/login';
  static const String register = '/register';
  static const String profile = '/profile';
  static const String product = '/product';
  static const String cart = '/cart';
  static const String checkout = '/checkout';
}

// 使用
context.go(AppRoutes.home);
context.go('${AppRoutes.product}/123');
```

### 13.2 路由扩展方法

```dart
// extensions/context_extensions.dart
extension GoRouterContextExtensions on BuildContext {
  void goHome() => go('/');
  void goProduct(String id) => go('/product/$id');
  void goProfile() => go('/profile');
  void goLogin({String? from}) => go('/login?from=$from');
}

// 使用
context.goHome();
context.goProduct('123');
context.goLogin(from: '/checkout');
```

### 13.3 项目结构建议

```
lib/
├── main.dart
├── app.dart
├── router/
│   ├── router.dart           # GoRouter 配置
│   ├── routes.dart           # 路由常量
│   └── redirect.dart         # 重定向逻辑
├── pages/
│   ├── home/
│   │   ├── home_page.dart
│   │   └── widgets/
│   ├── profile/
│   │   ├── profile_page.dart
│   │   └── widgets/
│   └── login/
│       └── login_page.dart
├── widgets/
│   └── scaffold_with_nav_bar.dart
└── services/
    └── auth_service.dart
```

### 13.4 DO 和 DON'T

#### ✅ DO

- 使用常量定义路由路径
- 使用 `context.go()` 进行导航
- 使用 `context.push()` 保留导航栈
- 使用 `ShellRoute` 实现嵌套导航
- 使用 `StatefulShellRoute` 保持状态
- 使用类型安全路由（代码生成）
- 使用重定向实现路由守卫

#### ❌ DON'T

- 混合使用 Navigator 1.0 和 go_router
- 使用 `extra` 传递重要数据（不支持深度链接）
- 在 `redirect` 中执行耗时操作
- 忽略错误处理
- 在 Web 中使用 `push` 而不处理 URL

---

## 附录：go_router API 速查表

### 导航方法

| 方法                    | 说明               |
| ----------------------- | ------------------ |
| context.go(path)        | 导航到路径，替换栈 |
| context.goNamed(name)   | 使用命名路由导航   |
| context.push(path)      | 推入新页面         |
| context.pushNamed(name) | 使用命名路由推入   |
| context.pop([result])   | 返回上一页         |
| context.replace(path)   | 替换当前页面       |
| context.canPop()        | 是否可以返回       |

### GoRoute 参数

| 参数        | 说明          |
| ----------- | ------------- |
| path        | 路由路径      |
| name        | 路由名称      |
| builder     | Widget 构建器 |
| pageBuilder | Page 构建器   |
| redirect    | 重定向函数    |
| routes      | 子路由        |

### GoRouterState 属性

| 属性                | 说明       |
| ------------------- | ---------- |
| matchedLocation     | 匹配的路径 |
| pathParameters      | 路径参数   |
| uri.queryParameters | 查询参数   |
| extra               | 额外数据   |
| error               | 错误信息   |

---

## 结语

`go_router` 17.1.0 是 Flutter 官方推荐的声明式路由解决方案。通过本书的学习，你应该已经掌握了：

1. go_router 的核心概念和基础用法
2. 各种导航方式的区别和使用场景
3. 路径参数和查询参数的处理
4. 嵌套路由和 ShellRoute 的实现
5. 重定向和路由守卫的配置
6. 转场动画的自定义
7. 深度链接的配置
8. 类型安全路由的代码生成

希望本书能够帮助你在 Flutter 开发中更好地使用 go_router，构建出具有优秀导航体验的应用程序。

---

**版本信息**

- go_router 版本：17.1.0
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
