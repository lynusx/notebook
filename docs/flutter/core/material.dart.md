# Flutter Material Dart 完整参考手册

## package:flutter/material.dart

> **版本**: Flutter 3.41.0  
> **适用范围**: Android、Web、桌面端 Material Design 应用  
> **官方文档**: [api.flutter.dev/flutter/material/material-library.html](https://api.flutter.dev/flutter/material/material-library.html)

---

## 第一章：库概述

`flutter/material.dart` 是 Flutter 框架中最核心的库之一，它实现了 Google 的 **Material Design** 设计规范。这个库包含了构建现代、美观、响应式用户界面所需的一切组件。

### 1.1 库的主要组成部分

| 类别 | 说明 | 主要类 |
|------|------|--------|
| **应用结构** | 应用骨架和页面布局 | MaterialApp, Scaffold, AppBar |
| **布局 Widget** | 排列和对齐子元素 | Container, Row, Column, Stack |
| **文本显示** | 文本渲染和样式 | Text, RichText, DefaultTextStyle |
| **按钮** | 交互式按钮组件 | ElevatedButton, TextButton, IconButton |
| **输入控件** | 表单和文本输入 | TextField, Form, TextFormField |
| **列表和滚动** | 可滚动内容区域 | ListView, GridView, SingleChildScrollView |
| **对话框** | 模态对话框和底部面板 | Dialog, AlertDialog, BottomSheet |
| **导航** | 页面切换和路由 | Navigator, Route, MaterialPageRoute |
| **主题** | 颜色和样式系统 | Theme, ThemeData, ColorScheme |
| **动画** | 过渡和动画效果 | AnimatedContainer, Hero, FadeTransition |
| **反馈** | 视觉和触觉反馈 | InkWell, Ripple, SnackBar |

### 1.2 导入方式

```dart
import 'package:flutter/material.dart';
```

> **补充知识**：`material.dart` 实际上导入了多个子库，包括 `widgets.dart`、`animation.dart`、`painting.dart` 等，因此你不需要单独导入这些基础库。

---

## 第二章：应用结构组件

### 2.1 MaterialApp

`MaterialApp` 是 Material Design 应用的根 Widget，它配置了整个应用的主题、路由、本地化等全局属性。

#### 完整构造函数

```dart
const MaterialApp({
  super.key,
  super.navigatorKey,
  super.scaffoldMessengerKey,
  super.home,
  Map<String, WidgetBuilder> this.routes = const <String, WidgetBuilder>{},
  super.initialRoute,
  this.onGenerateRoute,
  this.onGenerateInitialRoutes,
  this.onUnknownRoute,
  List<NavigatorObserver> this.navigatorObservers = const <NavigatorObserver>[],
  this.builder,
  this.title = '',
  this.onGenerateTitle,
  this.color,
  super.theme,
  super.darkTheme,
  super.highContrastTheme,
  super.highContrastDarkTheme,
  super.themeMode = ThemeMode.system,
  super.themeAnimationDuration = kThemeAnimationDuration,
  super.themeAnimationCurve = Curves.linear,
  this.locale,
  this.localizationsDelegates,
  this.localeListResolutionCallback,
  this.localeResolutionCallback,
  this.supportedLocales = const <Locale>[Locale('en', 'US')],
  this.debugShowMaterialGrid = false,
  this.showPerformanceOverlay = false,
  this.checkerboardRasterCacheImages = false,
  this.checkerboardOffscreenLayers = false,
  this.showSemanticsDebugger = false,
  this.debugShowCheckedModeBanner = true,
  this.shortcuts,
  this.actions,
  this.restorationScopeId,
  this.scrollBehavior,
  @Deprecated('Use theme instead')
  this.useInheritedMediaQuery = false,
  this.themeAnimationStyle,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `home` | Widget? | null | 应用的默认路由页面，当 initialRoute 未指定时使用 |
| `routes` | Map<String, WidgetBuilder> | {} | 命名路由映射表，键为路由名，值为构建函数 |
| `initialRoute` | String? | null | 应用启动时显示的初始路由名称 |
| `onGenerateRoute` | RouteFactory? | null | 动态路由生成回调，当 routes 中找不到匹配路由时调用 |
| `onUnknownRoute` | RouteFactory? | null | 未知路由处理回调，当 onGenerateRoute 也返回 null 时调用 |
| `navigatorObservers` | List<NavigatorObserver> | [] | 导航观察器列表，用于监听页面切换事件 |
| `builder` | TransitionBuilder? | null | 应用构建器，可用于包裹整个应用，常用于添加全局 Overlay |
| `title` | String | '' | 应用标题，用于任务管理器和系统界面 |
| `onGenerateTitle` | GenerateAppTitle? | null | 动态生成标题的回调，可接收 BuildContext 参数 |
| `color` | Color? | null | 应用的主色调，影响系统界面（如 Android 状态栏） |
| `theme` | ThemeData? | null | 应用的亮色主题数据 |
| `darkTheme` | ThemeData? | null | 应用的暗色主题数据 |
| `themeMode` | ThemeMode | ThemeMode.system | 主题模式：system、light、dark |
| `themeAnimationDuration` | Duration | kThemeAnimationDuration | 主题切换动画持续时间（默认 200ms） |
| `themeAnimationCurve` | Curve | Curves.linear | 主题切换动画曲线 |
| `locale` | Locale? | null | 应用的当前语言区域 |
| `localizationsDelegates` | List<LocalizationsDelegate>? | null | 本地化委托列表 |
| `supportedLocales` | List<Locale> | [Locale('en', 'US')] | 支持的语言区域列表 |
| `debugShowMaterialGrid` | bool | false | 是否显示 Material 网格调试线 |
| `showPerformanceOverlay` | bool | false | 是否显示性能监控覆盖层 |
| `showSemanticsDebugger` | bool | false | 是否显示语义调试器 |
| `debugShowCheckedModeBanner` | bool | true | 是否显示调试模式横幅 |
| `shortcuts` | Map<ShortcutActivator, Intent>? | null | 全局键盘快捷键映射 |
| `actions` | Map<Type, Action<Intent>>? | null | 全局动作映射 |
| `restorationScopeId` | String? | null | 状态恢复作用域 ID |
| `scrollBehavior` | ScrollBehavior? | null | 滚动行为配置 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // 应用标识
      key: const Key('my_app'),
      
      // 标题配置
      title: 'Material App Demo',
      onGenerateTitle: (context) => '动态标题 - ${Localizations.localeOf(context)}',
      
      // 主题配置
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.dark,
        ),
        useMaterial3: true,
      ),
      themeMode: ThemeMode.system,
      themeAnimationDuration: const Duration(milliseconds: 300),
      themeAnimationCurve: Curves.easeInOut,
      
      // 路由配置
      initialRoute: '/',
      routes: {
        '/': (context) => const HomePage(),
        '/profile': (context) => const ProfilePage(),
        '/settings': (context) => const SettingsPage(),
      },
      onGenerateRoute: (settings) {
        // 动态路由处理
        if (settings.name?.startsWith('/user/') ?? false) {
          final userId = settings.name!.split('/').last;
          return MaterialPageRoute(
            builder: (context) => UserDetailPage(userId: userId),
          );
        }
        return null;
      },
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundPage(),
        );
      },
      
      // 本地化配置
      locale: const Locale('zh', 'CN'),
      supportedLocales: const [
        Locale('en', 'US'),
        Locale('zh', 'CN'),
        Locale('ja', 'JP'),
      ],
      localizationsDelegates: const [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      localeResolutionCallback: (locale, supportedLocales) {
        for (var supportedLocale in supportedLocales) {
          if (supportedLocale.languageCode == locale?.languageCode) {
            return supportedLocale;
          }
        }
        return const Locale('en', 'US');
      },
      
      // 调试配置
      debugShowMaterialGrid: false,
      showPerformanceOverlay: false,
      debugShowCheckedModeBanner: true,
      
      // 主页
      home: const HomePage(),
    );
  }
}

// 页面定义
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('首页')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => Navigator.pushNamed(context, '/profile'),
              child: const Text('个人资料'),
            ),
            ElevatedButton(
              onPressed: () => Navigator.pushNamed(context, '/user/123'),
              child: const Text('用户详情 (动态路由)'),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('个人资料')),
      body: const Center(child: Text('个人资料页面')),
    );
  }
}

class SettingsPage extends StatelessWidget {
  const SettingsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('设置')),
      body: const Center(child: Text('设置页面')),
    );
  }
}

class UserDetailPage extends StatelessWidget {
  final String userId;
  const UserDetailPage({super.key, required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('用户 $userId')),
      body: Center(child: Text('用户 ID: $userId')),
    );
  }
}

class NotFoundPage extends StatelessWidget {
  const NotFoundPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('页面未找到')),
      body: const Center(child: Text('404 - 页面不存在')),
    );
  }
}
```

> **补充知识**：`MaterialApp` 内部创建了一个 `WidgetsApp`，并添加了 Material 特有的功能，如 Ink 效果、Material 主题等。如果你需要更底层的控制，可以直接使用 `WidgetsApp`。

---

### 2.2 Scaffold

`Scaffold` 实现了 Material Design 的基本页面布局结构，提供了抽屉、底部导航栏、悬浮按钮等标准组件的插槽。

#### 完整构造函数

```dart
const Scaffold({
  super.key,
  this.appBar,
  this.body,
  this.floatingActionButton,
  this.floatingActionButtonLocation,
  this.floatingActionButtonAnimator,
  this.persistentFooterButtons,
  this.persistentFooterAlignment = AlignmentDirectional.centerEnd,
  this.drawer,
  this.onDrawerChanged,
  this.endDrawer,
  this.onEndDrawerChanged,
  this.bottomNavigationBar,
  this.bottomSheet,
  this.backgroundColor,
  this.resizeToAvoidBottomInset,
  this.primary = true,
  this.drawerDragStartBehavior = DragStartBehavior.start,
  this.extendBody = false,
  this.extendBodyBehindAppBar = false,
  this.drawerScrimColor,
  this.drawerEdgeDragWidth,
  this.drawerEnableOpenDragGesture = true,
  this.endDrawerEnableOpenDragGesture = true,
  this.restorationId,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `appBar` | PreferredSizeWidget? | null | 顶部应用栏，通常为 AppBar |
| `body` | Widget? | null | 页面的主要内容区域 |
| `floatingActionButton` | Widget? | null | 悬浮操作按钮（FAB） |
| `floatingActionButtonLocation` | FloatingActionButtonLocation? | null | FAB 的位置 |
| `floatingActionButtonAnimator` | FloatingActionButtonAnimator? | null | FAB 位置变化动画器 |
| `persistentFooterButtons` | List<Widget>? | null | 底部固定按钮列表 |
| `persistentFooterAlignment` | AlignmentDirectional | AlignmentDirectional.centerEnd | 底部按钮对齐方式 |
| `drawer` | Widget? | null | 左侧抽屉菜单 |
| `onDrawerChanged` | DrawerCallback? | null | 抽屉状态变化回调 |
| `endDrawer` | Widget? | null | 右侧抽屉菜单 |
| `onEndDrawerChanged` | DrawerCallback? | null | 右侧抽屉状态变化回调 |
| `bottomNavigationBar` | Widget? | null | 底部导航栏 |
| `bottomSheet` | Widget? | null | 底部面板 |
| `backgroundColor` | Color? | null | 背景颜色，默认使用主题背景色 |
| `resizeToAvoidBottomInset` | bool? | null | 是否自动调整大小避免键盘遮挡 |
| `primary` | bool | true | 是否将内容放置在状态栏下方 |
| `drawerDragStartBehavior` | DragStartBehavior | DragStartBehavior.start | 抽屉拖动开始行为 |
| `extendBody` | bool | false | 是否将 body 延伸到 bottomNavigationBar 下方 |
| `extendBodyBehindAppBar` | bool | false | 是否将 body 延伸到 AppBar 后方 |
| `drawerScrimColor` | Color? | null | 抽屉打开时的遮罩颜色 |
| `drawerEdgeDragWidth` | double? | null | 抽屉边缘拖动触发宽度 |
| `drawerEnableOpenDragGesture` | bool | true | 是否启用左侧抽屉拖动打开 |
| `endDrawerEnableOpenDragGesture` | bool | true | 是否启用右侧抽屉拖动打开 |
| `restorationId` | String? | null | 状态恢复 ID |

#### FloatingActionButtonLocation 枚举值

| 值 | 说明 |
|----|------|
| `FloatingActionButtonLocation.startTop` | 左上角 |
| `FloatingActionButtonLocation.miniStartTop` | 左上角（迷你版） |
| `FloatingActionButtonLocation.centerTop` | 顶部居中 |
| `FloatingActionButtonLocation.endTop` | 右上角 |
| `FloatingActionButtonLocation.startFloat` | 左下角浮动 |
| `FloatingActionButtonLocation.miniStartFloat` | 左下角浮动（迷你版） |
| `FloatingActionButtonLocation.centerFloat` | 底部居中浮动 |
| `FloatingActionButtonLocation.endFloat` | 右下角浮动（默认） |
| `FloatingActionButtonLocation.miniEndFloat` | 右下角浮动（迷你版） |
| `FloatingActionButtonLocation.startDocked` | 左下角停靠 |
| `FloatingActionButtonLocation.miniStartDocked` | 左下角停靠（迷你版） |
| `FloatingActionButtonLocation.centerDocked` | 底部居中停靠 |
| `FloatingActionButtonLocation.endDocked` | 右下角停靠 |
| `FloatingActionButtonLocation.miniEndDocked` | 右下角停靠（迷你版） |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Scaffold Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ScaffoldDemoPage(),
    );
  }
}

class ScaffoldDemoPage extends StatefulWidget {
  const ScaffoldDemoPage({super.key});

  @override
  State<ScaffoldDemoPage> createState() => _ScaffoldDemoPageState();
}

class _ScaffoldDemoPageState extends State<ScaffoldDemoPage> {
  int _selectedIndex = 0;
  int _fabCounter = 0;
  bool _isExtended = false;

  void _onItemTapped(int index) {
    setState(() {
      _selectedIndex = index;
    });
  }

  void _incrementCounter() {
    setState(() {
      _fabCounter++;
    });
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('计数: $_fabCounter')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 应用栏
      appBar: AppBar(
        title: const Text('Scaffold 完整演示'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {},
          ),
          IconButton(
            icon: const Icon(Icons.more_vert),
            onPressed: () {},
          ),
        ],
      ),

      // 左侧抽屉
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            DrawerHeader(
              decoration: BoxDecoration(
                color: Theme.of(context).colorScheme.primary,
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  CircleAvatar(
                    radius: 30,
                    backgroundColor: Colors.white,
                    child: Icon(Icons.person, size: 35),
                  ),
                  SizedBox(height: 10),
                  Text(
                    '用户名',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                  Text(
                    'user@example.com',
                    style: TextStyle(color: Colors.white70),
                  ),
                ],
              ),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('首页'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('设置'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
          ],
        ),
      ),
      onDrawerChanged: (isOpen) {
        debugPrint('抽屉状态: ${isOpen ? "打开" : "关闭"}');
      },

      // 右侧抽屉
      endDrawer: Drawer(
        child: ListView(
          children: const [
            DrawerHeader(
              child: Text('通知'),
            ),
            ListTile(
              leading: Icon(Icons.notifications),
              title: Text('暂无新通知'),
            ),
          ],
        ),
      ),

      // 主体内容
      body: IndexedStack(
        index: _selectedIndex,
        children: [
          // 首页
          Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text('首页内容', style: TextStyle(fontSize: 24)),
                const SizedBox(height: 20),
                Text('FAB 点击次数: $_fabCounter', style: const TextStyle(fontSize: 18)),
              ],
            ),
          ),
          // 发现页
          const Center(
            child: Text('发现页内容', style: TextStyle(fontSize: 24)),
          ),
          // 我的
          const Center(
            child: Text('我的页面', style: TextStyle(fontSize: 24)),
          ),
        ],
      ),

      // 悬浮操作按钮
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _incrementCounter,
        icon: const Icon(Icons.add),
        label: Text('添加 ($_fabCounter)'),
        isExtended: _isExtended,
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
      floatingActionButtonAnimator: FloatingActionButtonAnimator.scaling,

      // 底部导航栏
      bottomNavigationBar: BottomNavigationBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: '首页',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.explore),
            label: '发现',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: '我的',
          ),
        ],
        currentIndex: _selectedIndex,
        selectedItemColor: Theme.of(context).colorScheme.primary,
        onTap: _onItemTapped,
      ),

      // 背景色
      backgroundColor: Colors.grey[100],

      // 自动调整大小避免键盘遮挡
      resizeToAvoidBottomInset: true,

      // 延伸 body 到 AppBar 后方
      extendBodyBehindAppBar: false,

      // 延伸 body 到底部导航栏下方
      extendBody: false,

      // 抽屉遮罩颜色
      drawerScrimColor: Colors.black54,

      // 启用抽屉手势
      drawerEnableOpenDragGesture: true,
      endDrawerEnableOpenDragGesture: true,
    );
  }
}
```

> **补充知识**：`Scaffold` 使用 `ScaffoldMessenger` 来显示 `SnackBar` 和 `MaterialBanner`。通过 `ScaffoldMessenger.of(context)` 可以在页面任何地方显示这些提示组件。

---

### 2.3 AppBar

`AppBar` 是 Material Design 应用栏的实现，显示在页面顶部，包含标题、操作按钮和导航元素。

#### 完整构造函数

```dart
AppBar({
  super.key,
  this.leading,
  this.automaticallyImplyLeading = true,
  this.title,
  this.actions,
  this.flexibleSpace,
  this.bottom,
  this.elevation,
  this.scrolledUnderElevation,
  this.notificationPredicate = defaultScrollNotificationPredicate,
  this.shadowColor,
  this.surfaceTintColor,
  this.shape,
  this.backgroundColor,
  this.foregroundColor,
  this.iconTheme,
  this.actionsIconTheme,
  this.primary = true,
  this.centerTitle,
  this.excludeHeaderSemantics = false,
  this.titleSpacing,
  this.toolbarOpacity = 1.0,
  this.bottomOpacity = 1.0,
  this.toolbarHeight,
  this.leadingWidth,
  this.toolbarTextStyle,
  this.titleTextStyle,
  this.systemOverlayStyle,
  this.forceMaterialTransparency = false,
  this.clipBehavior,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `leading` | Widget? | null | 左侧导航按钮，通常为返回按钮或菜单按钮 |
| `automaticallyImplyLeading` | bool | true | 是否自动显示返回按钮（当可返回时） |
| `title` | Widget? | null | 标题 Widget |
| `actions` | List<Widget>? | null | 右侧操作按钮列表 |
| `flexibleSpace` | Widget? | null | 可扩展空间，用于实现折叠效果 |
| `bottom` | PreferredSizeWidget? | null | 底部 Widget，通常为 TabBar |
| `elevation` | double? | null | 阴影高度 |
| `scrolledUnderElevation` | double? | null | 滚动到下方时的阴影高度 |
| `notificationPredicate` | ScrollNotificationPredicate | defaultScrollNotificationPredicate | 滚动通知过滤条件 |
| `shadowColor` | Color? | null | 阴影颜色 |
| `surfaceTintColor` | Color? | null | 表面着色颜色（Material 3） |
| `shape` | ShapeBorder? | null | 形状边框 |
| `backgroundColor` | Color? | null | 背景颜色 |
| `foregroundColor` | Color? | null | 前景颜色（影响图标和文字） |
| `iconTheme` | IconThemeData? | null | 图标主题 |
| `actionsIconTheme` | IconThemeData? | null | 操作按钮图标主题 |
| `primary` | bool | true | 是否为主要应用栏 |
| `centerTitle` | bool? | null | 标题是否居中 |
| `excludeHeaderSemantics` | bool | false | 是否排除标题语义 |
| `titleSpacing` | double? | null | 标题间距 |
| `toolbarOpacity` | double | 1.0 | 工具栏不透明度 |
| `bottomOpacity` | double | 1.0 | 底部不透明度 |
| `toolbarHeight` | double? | null | 工具栏高度（默认 56） |
| `leadingWidth` | double? | null |  leading 宽度（默认 56） |
| `toolbarTextStyle` | TextStyle? | null | 工具栏文字样式 |
| `titleTextStyle` | TextStyle? | null | 标题文字样式 |
| `systemOverlayStyle` | SystemUiOverlayStyle? | null | 系统状态栏样式 |
| `forceMaterialTransparency` | bool | false | 是否强制透明（Material 3） |
| `clipBehavior` | Clip? | null | 裁剪行为 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'AppBar Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const AppBarDemoPage(),
    );
  }
}

class AppBarDemoPage extends StatefulWidget {
  const AppBarDemoPage({super.key});

  @override
  State<AppBarDemoPage> createState() => _AppBarDemoPageState();
}

class _AppBarDemoPageState extends State<AppBarDemoPage>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;
  bool _isSearching = false;
  final TextEditingController _searchController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    _searchController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: _isSearching ? _buildSearchAppBar() : _buildNormalAppBar(),
        body: TabBarView(
          controller: _tabController,
          children: [
            _buildTabContent('首页'),
            _buildTabContent('热门'),
            _buildTabContent('关注'),
          ],
        ),
      ),
    );
  }

  AppBar _buildNormalAppBar() {
    return AppBar(
      // 左侧按钮
      leading: IconButton(
        icon: const Icon(Icons.menu),
        onPressed: () {
          Scaffold.of(context).openDrawer();
        },
      ),
      automaticallyImplyLeading: true,

      // 标题
      title: const Text(
        'AppBar 演示',
        style: TextStyle(fontWeight: FontWeight.bold),
      ),
      centerTitle: true,

      // 右侧操作按钮
      actions: [
        IconButton(
          icon: const Icon(Icons.search),
          onPressed: () {
            setState(() {
              _isSearching = true;
            });
          },
        ),
        IconButton(
          icon: const Icon(Icons.notifications),
          onPressed: () {},
        ),
        PopupMenuButton<String>(
          onSelected: (value) {
            debugPrint('选中: $value');
          },
          itemBuilder: (context) => [
            const PopupMenuItem(value: 'settings', child: Text('设置')),
            const PopupMenuItem(value: 'about', child: Text('关于')),
            const PopupMenuItem(value: 'logout', child: Text('退出')),
          ],
        ),
      ],

      // 底部 TabBar
      bottom: TabBar(
        controller: _tabController,
        tabs: const [
          Tab(icon: Icon(Icons.home), text: '首页'),
          Tab(icon: Icon(Icons.trending_up), text: '热门'),
          Tab(icon: Icon(Icons.favorite), text: '关注'),
        ],
        indicatorColor: Colors.white,
        labelColor: Colors.white,
        unselectedLabelColor: Colors.white70,
      ),

      // 样式配置
      elevation: 4,
      shadowColor: Colors.black,
      backgroundColor: Theme.of(context).colorScheme.primary,
      foregroundColor: Colors.white,

      // 状态栏样式
      systemOverlayStyle: const SystemUiOverlayStyle(
        statusBarColor: Colors.transparent,
        statusBarIconBrightness: Brightness.light,
        statusBarBrightness: Brightness.dark,
      ),

      // 高度配置
      toolbarHeight: 56,
      titleSpacing: NavigationToolbar.kMiddleSpacing,
    );
  }

  AppBar _buildSearchAppBar() {
    return AppBar(
      leading: IconButton(
        icon: const Icon(Icons.arrow_back),
        onPressed: () {
          setState(() {
            _isSearching = false;
            _searchController.clear();
          });
        },
      ),
      title: TextField(
        controller: _searchController,
        autofocus: true,
        decoration: const InputDecoration(
          hintText: '搜索...',
          border: InputBorder.none,
          hintStyle: TextStyle(color: Colors.white70),
        ),
        style: const TextStyle(color: Colors.white),
        onSubmitted: (value) {
          debugPrint('搜索: $value');
        },
      ),
      actions: [
        IconButton(
          icon: const Icon(Icons.clear),
          onPressed: () {
            _searchController.clear();
          },
        ),
      ],
      backgroundColor: Theme.of(context).colorScheme.primary,
      foregroundColor: Colors.white,
    );
  }

  Widget _buildTabContent(String title) {
    return Center(
      child: Text(
        title,
        style: const TextStyle(fontSize: 24),
      ),
    );
  }
}

// SliverAppBar 示例
class SliverAppBarDemo extends StatelessWidget {
  const SliverAppBarDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 200,
            floating: false,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('可折叠 AppBar'),
              background: Image.network(
                'https://picsum.photos/400/200',
                fit: BoxFit.cover,
              ),
            ),
          ),
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                title: Text('Item $index'),
              ),
              childCount: 50,
            ),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：在 Material 3 中，`AppBar` 的默认样式有所变化，使用 `surfaceTintColor` 来实现滚动时的颜色变化效果。可以通过 `ThemeData.useMaterial3` 控制是否启用 Material 3 样式。

---

## 第三章：布局 Widget

### 3.1 Container

`Container` 是最常用的布局 Widget 之一，它结合了绘制、定位和尺寸调整功能，可以设置边距、内边距、背景、边框、阴影等样式。

#### 完整构造函数

```dart
Container({
  super.key,
  this.alignment,
  this.padding,
  this.color,
  this.decoration,
  this.foregroundDecoration,
  double? width,
  double? height,
  BoxConstraints? constraints,
  this.margin,
  this.transform,
  this.transformAlignment,
  this.child,
  this.clipBehavior = Clip.none,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `alignment` | AlignmentGeometry? | null | 子元素对齐方式 |
| `padding` | EdgeInsetsGeometry? | null | 内边距 |
| `color` | Color? | null | 背景颜色（不能与 decoration.color 同时使用） |
| `decoration` | Decoration? | null | 背景装饰（BoxDecoration、ShapeDecoration 等） |
| `foregroundDecoration` | Decoration? | null | 前景装饰，绘制在子元素上方 |
| `width` | double? | null | 宽度 |
| `height` | double? | null | 高度 |
| `constraints` | BoxConstraints? | null | 尺寸约束 |
| `margin` | EdgeInsetsGeometry? | null | 外边距 |
| `transform` | Matrix4? | null | 变换矩阵 |
| `transformAlignment` | AlignmentGeometry? | null | 变换对齐点 |
| `child` | Widget? | null | 子元素 |
| `clipBehavior` | Clip | Clip.none | 裁剪行为 |

#### Alignment 常用值

| 值 | 说明 |
|----|------|
| `Alignment.topLeft` | 左上角 |
| `Alignment.topCenter` | 顶部居中 |
| `Alignment.topRight` | 右上角 |
| `Alignment.centerLeft` | 左侧居中 |
| `Alignment.center` | 正中心 |
| `Alignment.centerRight` | 右侧居中 |
| `Alignment.bottomLeft` | 左下角 |
| `Alignment.bottomCenter` | 底部居中 |
| `Alignment.bottomRight` | 右下角 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Container Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ContainerDemoPage(),
    );
  }
}

class ContainerDemoPage extends StatelessWidget {
  const ContainerDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Container 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础 Container
            _buildSectionTitle('1. 基础样式'),
            Container(
              width: double.infinity,
              height: 100,
              color: Colors.blue,
              alignment: Alignment.center,
              child: const Text(
                '基础 Container',
                style: TextStyle(color: Colors.white, fontSize: 18),
              ),
            ),
            const SizedBox(height: 20),

            // 带圆角和阴影
            _buildSectionTitle('2. 圆角和阴影'),
            Container(
              width: double.infinity,
              height: 120,
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.2),
                    blurRadius: 10,
                    offset: const Offset(0, 4),
                  ),
                ],
              ),
              child: const Center(child: Text('圆角 + 阴影')),
            ),
            const SizedBox(height: 20),

            // 边框和渐变
            _buildSectionTitle('3. 边框和渐变'),
            Container(
              width: double.infinity,
              height: 120,
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Colors.purple, Colors.blue],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(12),
                border: Border.all(
                  color: Colors.white,
                  width: 3,
                ),
              ),
              child: const Center(
                child: Text(
                  '渐变背景 + 边框',
                  style: TextStyle(color: Colors.white, fontSize: 16),
                ),
              ),
            ),
            const SizedBox(height: 20),

            // 内边距和外边距
            _buildSectionTitle('4. 内边距和外边距'),
            Container(
              color: Colors.grey[200],
              padding: const EdgeInsets.all(16),
              child: Container(
                margin: const EdgeInsets.all(8),
                padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
                decoration: BoxDecoration(
                  color: Colors.green,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Text(
                  'margin: 8, padding: 24x16',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
            const SizedBox(height: 20),

            // 变换效果
            _buildSectionTitle('5. 变换效果'),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.orange,
                  transform: Matrix4.rotationZ(0.3),
                  alignment: Alignment.center,
                  child: const Text('旋转', style: TextStyle(color: Colors.white)),
                ),
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.red,
                  transform: Matrix4.identity()..scale(1.2),
                  alignment: Alignment.center,
                  child: const Text('缩放', style: TextStyle(color: Colors.white)),
                ),
                Container(
                  width: 80,
                  height: 80,
                  color: Colors.teal,
                  transform: Matrix4.translationValues(0, -10, 0),
                  alignment: Alignment.center,
                  child: const Text('平移', style: TextStyle(color: Colors.white)),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // 形状装饰
            _buildSectionTitle('6. 形状装饰'),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                Container(
                  width: 100,
                  height: 100,
                  decoration: const BoxDecoration(
                    color: Colors.pink,
                    shape: BoxShape.circle,
                  ),
                  child: const Icon(Icons.favorite, color: Colors.white, size: 40),
                ),
                Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: Colors.indigo,
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: const Icon(Icons.star, color: Colors.white, size: 40),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // 约束
            _buildSectionTitle('7. 尺寸约束'),
            Container(
              constraints: const BoxConstraints(
                minWidth: 100,
                maxWidth: 300,
                minHeight: 50,
                maxHeight: 150,
              ),
              color: Colors.amber,
              child: const Padding(
                padding: EdgeInsets.all(16),
                child: Text('受约束的 Container，内容决定实际大小'),
              ),
            ),
            const SizedBox(height: 20),

            // 裁剪
            _buildSectionTitle('8. 裁剪行为'),
            Container(
              width: 150,
              height: 150,
              clipBehavior: Clip.antiAlias,
              decoration: const BoxDecoration(
                shape: BoxShape.circle,
              ),
              child: Image.network(
                'https://picsum.photos/150/150',
                fit: BoxFit.cover,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

> **补充知识**：`Container` 的 `color` 属性实际上是 `decoration` 的快捷方式。当你同时设置 `color` 和 `decoration` 时，会抛出异常。如果需要复杂的装饰效果，应该只使用 `decoration`。

---

### 3.2 Row

`Row` 是水平排列子元素的弹性布局 Widget，它沿着水平主轴排列子元素。

#### 完整构造函数

```dart
const Row({
  super.key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline? textBaseline,
  List<Widget> children = const <Widget>[],
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `mainAxisAlignment` | MainAxisAlignment | MainAxisAlignment.start | 主轴（水平）对齐方式 |
| `mainAxisSize` | MainAxisSize | MainAxisSize.max | 主轴尺寸：max 占满可用空间，min 包裹内容 |
| `crossAxisAlignment` | CrossAxisAlignment | CrossAxisAlignment.center | 交叉轴（垂直）对齐方式 |
| `textDirection` | TextDirection? | null | 文本方向，影响子元素排列顺序 |
| `verticalDirection` | VerticalDirection | VerticalDirection.down | 垂直方向 |
| `textBaseline` | TextBaseline? | null | 文本基线，用于基线对齐 |
| `children` | List<Widget> | [] | 子元素列表 |

#### MainAxisAlignment 枚举值

| 值 | 说明 |
|----|------|
| `MainAxisAlignment.start` | 从起点开始对齐 |
| `MainAxisAlignment.end` | 从终点对齐 |
| `MainAxisAlignment.center` | 居中对齐 |
| `MainAxisAlignment.spaceBetween` | 两端对齐，中间均匀分布 |
| `MainAxisAlignment.spaceAround` | 均匀分布，两端留半间距 |
| `MainAxisAlignment.spaceEvenly` | 完全均匀分布 |

#### CrossAxisAlignment 枚举值

| 值 | 说明 |
|----|------|
| `CrossAxisAlignment.start` | 交叉轴起点对齐 |
| `CrossAxisAlignment.end` | 交叉轴终点对齐 |
| `CrossAxisAlignment.center` | 交叉轴居中对齐 |
| `CrossAxisAlignment.stretch` | 拉伸填满交叉轴 |
| `CrossAxisAlignment.baseline` | 基线对齐（需要 textBaseline） |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Row Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const RowDemoPage(),
    );
  }
}

class RowDemoPage extends StatelessWidget {
  const RowDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Row 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础 Row
            _buildSectionTitle('1. 基础 Row'),
            Container(
              color: Colors.grey[200],
              child: Row(
                children: [
                  _buildBox(Colors.red, 'A'),
                  _buildBox(Colors.green, 'B'),
                  _buildBox(Colors.blue, 'C'),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // MainAxisAlignment 对比
            _buildSectionTitle('2. MainAxisAlignment 对比'),
            ...MainAxisAlignment.values.map((alignment) {
              return Padding(
                padding: const EdgeInsets.only(bottom: 12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(alignment.toString().split('.').last),
                    Container(
                      color: Colors.grey[200],
                      child: Row(
                        mainAxisAlignment: alignment,
                        children: [
                          _buildSmallBox(Colors.red),
                          _buildSmallBox(Colors.green),
                          _buildSmallBox(Colors.blue),
                        ],
                      ),
                    ),
                  ],
                ),
              );
            }),
            const SizedBox(height: 20),

            // CrossAxisAlignment 对比
            _buildSectionTitle('3. CrossAxisAlignment 对比'),
            ...CrossAxisAlignment.values.where((a) => a != CrossAxisAlignment.baseline).map((alignment) {
              return Padding(
                padding: const EdgeInsets.only(bottom: 12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(alignment.toString().split('.').last),
                    Container(
                      height: 80,
                      color: Colors.grey[200],
                      child: Row(
                        crossAxisAlignment: alignment,
                        children: [
                          _buildBox(Colors.red, 'A', height: 30),
                          _buildBox(Colors.green, 'B', height: 60),
                          _buildBox(Colors.blue, 'C', height: 45),
                        ],
                      ),
                    ),
                  ],
                ),
              );
            }),
            const SizedBox(height: 20),

            // MainAxisSize 对比
            _buildSectionTitle('4. MainAxisSize 对比'),
            Column(
              children: [
                Container(
                  width: double.infinity,
                  color: Colors.grey[300],
                  child: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      _buildBox(Colors.orange, 'min'),
                      _buildBox(Colors.purple, 'size'),
                    ],
                  ),
                ),
                const SizedBox(height: 8),
                Container(
                  width: double.infinity,
                  color: Colors.grey[300],
                  child: Row(
                    mainAxisSize: MainAxisSize.max,
                    children: [
                      _buildBox(Colors.orange, 'max'),
                      _buildBox(Colors.purple, 'size'),
                    ],
                  ),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // Expanded 使用
            _buildSectionTitle('5. Expanded 弹性布局'),
            Container(
              color: Colors.grey[200],
              child: Row(
                children: [
                  Expanded(
                    flex: 1,
                    child: Container(
                      height: 50,
                      color: Colors.red,
                      child: const Center(child: Text('1x', style: TextStyle(color: Colors.white))),
                    ),
                  ),
                  Expanded(
                    flex: 2,
                    child: Container(
                      height: 50,
                      color: Colors.green,
                      child: const Center(child: Text('2x', style: TextStyle(color: Colors.white))),
                    ),
                  ),
                  Expanded(
                    flex: 3,
                    child: Container(
                      height: 50,
                      color: Colors.blue,
                      child: const Center(child: Text('3x', style: TextStyle(color: Colors.white))),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // Flexible 使用
            _buildSectionTitle('6. Flexible 弹性布局'),
            Container(
              color: Colors.grey[200],
              child: Row(
                children: [
                  Flexible(
                    flex: 1,
                    child: Container(
                      height: 50,
                      color: Colors.teal,
                      child: const Center(child: Text('Flexible 1', style: TextStyle(color: Colors.white))),
                    ),
                  ),
                  Flexible(
                    flex: 1,
                    child: Container(
                      height: 50,
                      color: Colors.indigo,
                      child: const Center(child: Text('Flexible 2', style: TextStyle(color: Colors.white))),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // Spacer 使用
            _buildSectionTitle('7. Spacer 间距'),
            Container(
              color: Colors.grey[200],
              child: Row(
                children: [
                  _buildBox(Colors.red, '左'),
                  const Spacer(flex: 2),
                  _buildBox(Colors.green, '中'),
                  const Spacer(flex: 1),
                  _buildBox(Colors.blue, '右'),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 溢出处理
            _buildSectionTitle('8. 溢出处理'),
            Container(
              color: Colors.grey[200],
              child: Row(
                children: [
                  _buildBox(Colors.red, 'A', width: 100),
                  _buildBox(Colors.green, 'B', width: 100),
                  _buildBox(Colors.blue, 'C', width: 100),
                  _buildBox(Colors.orange, 'D', width: 100),
                  _buildBox(Colors.purple, 'E', width: 100),
                ],
              ),
            ),
            const Text('⚠️ 溢出警告：内容超出可用空间', style: TextStyle(color: Colors.red)),
            const SizedBox(height: 12),
            Container(
              color: Colors.grey[200],
              child: SingleChildScrollView(
                scrollDirection: Axis.horizontal,
                child: Row(
                  children: [
                    _buildBox(Colors.red, 'A', width: 100),
                    _buildBox(Colors.green, 'B', width: 100),
                    _buildBox(Colors.blue, 'C', width: 100),
                    _buildBox(Colors.orange, 'D', width: 100),
                    _buildBox(Colors.purple, 'E', width: 100),
                  ],
                ),
              ),
            ),
            const Text('✅ 使用 SingleChildScrollView 解决溢出'),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }

  Widget _buildBox(Color color, String text, {double? width, double? height}) {
    return Container(
      width: width ?? 60,
      height: height ?? 60,
      color: color,
      child: Center(
        child: Text(
          text,
          style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }

  Widget _buildSmallBox(Color color) {
    return Container(
      width: 40,
      height: 40,
      color: color,
    );
  }
}
```

> **补充知识**：`Row` 和 `Column` 都继承自 `Flex`，只是主轴方向不同。`Row` 的主轴是水平的，`Column` 的主轴是垂直的。理解这一点有助于更好地掌握弹性布局。

---

### 3.3 Column

`Column` 是垂直排列子元素的弹性布局 Widget，它沿着垂直主轴排列子元素。

#### 完整构造函数

```dart
const Column({
  super.key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection? textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline? textBaseline,
  List<Widget> children = const <Widget>[],
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `mainAxisAlignment` | MainAxisAlignment | MainAxisAlignment.start | 主轴（垂直）对齐方式 |
| `mainAxisSize` | MainAxisSize | MainAxisSize.max | 主轴尺寸 |
| `crossAxisAlignment` | CrossAxisAlignment | CrossAxisAlignment.center | 交叉轴（水平）对齐方式 |
| `textDirection` | TextDirection? | null | 文本方向 |
| `verticalDirection` | VerticalDirection | VerticalDirection.down | 垂直方向（down/up） |
| `textBaseline` | TextBaseline? | null | 文本基线 |
| `children` | List<Widget> | [] | 子元素列表 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Column Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ColumnDemoPage(),
    );
  }
}

class ColumnDemoPage extends StatelessWidget {
  const ColumnDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Column 完整演示')),
      body: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // 左侧：各种对齐方式
          Expanded(
            child: SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _buildSectionTitle('1. MainAxisAlignment'),
                  ...MainAxisAlignment.values.map((alignment) {
                    return Padding(
                      padding: const EdgeInsets.only(bottom: 12),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(alignment.toString().split('.').last),
                          Container(
                            height: 120,
                            color: Colors.grey[200],
                            child: Column(
                              mainAxisAlignment: alignment,
                              children: [
                                _buildSmallBox(Colors.red),
                                _buildSmallBox(Colors.green),
                                _buildSmallBox(Colors.blue),
                              ],
                            ),
                          ),
                        ],
                      ),
                    );
                  }),
                ],
              ),
            ),
          ),
          // 右侧：其他特性
          Expanded(
            child: SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _buildSectionTitle('2. CrossAxisAlignment'),
                  ...CrossAxisAlignment.values.where((a) => a != CrossAxisAlignment.baseline).map((alignment) {
                    return Padding(
                      padding: const EdgeInsets.only(bottom: 12),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(alignment.toString().split('.').last),
                          Container(
                            width: double.infinity,
                            color: Colors.grey[200],
                            child: Column(
                              crossAxisAlignment: alignment,
                              children: [
                                _buildWideBox(Colors.red, width: 50),
                                _buildWideBox(Colors.green, width: 100),
                                _buildWideBox(Colors.blue, width: 75),
                              ],
                            ),
                          ),
                        ],
                      ),
                    );
                  }),
                  const SizedBox(height: 20),
                  _buildSectionTitle('3. Expanded'),
                  Container(
                    height: 200,
                    color: Colors.grey[200],
                    child: Column(
                      children: [
                        Expanded(
                          flex: 1,
                          child: Container(
                            color: Colors.red,
                            child: const Center(child: Text('1x', style: TextStyle(color: Colors.white))),
                          ),
                        ),
                        Expanded(
                          flex: 2,
                          child: Container(
                            color: Colors.green,
                            child: const Center(child: Text('2x', style: TextStyle(color: Colors.white))),
                          ),
                        ),
                        Expanded(
                          flex: 1,
                          child: Container(
                            color: Colors.blue,
                            child: const Center(child: Text('1x', style: TextStyle(color: Colors.white))),
                          ),
                        ),
                      ],
                    ),
                  ),
                  const SizedBox(height: 20),
                  _buildSectionTitle('4. VerticalDirection'),
                  Row(
                    children: [
                      Expanded(
                        child: Column(
                          children: [
                            const Text('down'),
                            Container(
                              height: 100,
                              color: Colors.grey[200],
                              child: const Column(
                                verticalDirection: VerticalDirection.down,
                                children: [
                                  Text('1', style: TextStyle(fontSize: 24)),
                                  Text('2', style: TextStyle(fontSize: 24)),
                                  Text('3', style: TextStyle(fontSize: 24)),
                                ],
                              ),
                            ),
                          ],
                        ),
                      ),
                      Expanded(
                        child: Column(
                          children: [
                            const Text('up'),
                            Container(
                              height: 100,
                              color: Colors.grey[200],
                              child: const Column(
                                verticalDirection: VerticalDirection.up,
                                children: [
                                  Text('1', style: TextStyle(fontSize: 24)),
                                  Text('2', style: TextStyle(fontSize: 24)),
                                  Text('3', style: TextStyle(fontSize: 24)),
                                ],
                              ),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }

  Widget _buildSmallBox(Color color) {
    return Container(
      width: 40,
      height: 40,
      color: color,
    );
  }

  Widget _buildWideBox(Color color, {double? width}) {
    return Container(
      width: width ?? 60,
      height: 30,
      color: color,
    );
  }
}
```

> **补充知识**：`Column` 在内容超出屏幕高度时会导致溢出错误。解决方案包括：使用 `Expanded` 或 `Flexible` 包装子元素、使用 `SingleChildScrollView` 包裹，或者使用 `ListView` 替代。

---

### 3.4 Stack

`Stack` 允许子元素相互堆叠，后添加的子元素会覆盖前面的子元素。常用于实现重叠布局、定位元素等场景。

#### 完整构造函数

```dart
const Stack({
  super.key,
  this.alignment = AlignmentDirectional.topStart,
  this.textDirection,
  this.fit = StackFit.loose,
  this.clipBehavior = Clip.hardEdge,
  List<Widget> children = const <Widget>[],
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `alignment` | AlignmentGeometry | AlignmentDirectional.topStart | 非定位子元素的对齐方式 |
| `textDirection` | TextDirection? | null | 文本方向，影响 alignment |
| `fit` | StackFit | StackFit.loose | 非定位子元素的尺寸适应方式 |
| `clipBehavior` | Clip | Clip.hardEdge | 超出 Stack 边界的裁剪行为 |
| `children` | List<Widget> | [] | 子元素列表 |

#### StackFit 枚举值

| 值 | 说明 |
|----|------|
| `StackFit.loose` | 子元素使用自身尺寸约束 |
| `StackFit.expand` | 子元素扩展填满 Stack |
| `StackFit.passthrough` | 将 Stack 的约束传递给子元素 |

#### Positioned 构造函数

```dart
const Positioned({
  super.key,
  this.left,
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  required super.child,
});

const Positioned.fill({
  super.key,
  this.left = 0.0,
  this.top = 0.0,
  this.right = 0.0,
  this.bottom = 0.0,
  required super.child,
});

const Positioned.directional({
  super.key,
  required TextDirection textDirection,
  this.start,
  this.top,
  this.end,
  this.bottom,
  this.width,
  this.height,
  required super.child,
});

const Positioned.fromRect({
  super.key,
  required Rect rect,
  required super.child,
});

const Positioned.fromRelativeRect({
  super.key,
  required RelativeRect rect,
  required super.child,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Stack Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const StackDemoPage(),
    );
  }
}

class StackDemoPage extends StatelessWidget {
  const StackDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Stack 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础 Stack
            _buildSectionTitle('1. 基础堆叠'),
            Container(
              width: 200,
              height: 200,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  Container(
                    width: 180,
                    height: 180,
                    color: Colors.red,
                  ),
                  Container(
                    width: 140,
                    height: 140,
                    color: Colors.green,
                  ),
                  Container(
                    width: 100,
                    height: 100,
                    color: Colors.blue,
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // Positioned 定位
            _buildSectionTitle('2. Positioned 定位'),
            Container(
              width: double.infinity,
              height: 200,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  // 左上角
                  Positioned(
                    left: 10,
                    top: 10,
                    child: _buildPositionedBox(Colors.red, '左上'),
                  ),
                  // 右上角
                  Positioned(
                    right: 10,
                    top: 10,
                    child: _buildPositionedBox(Colors.green, '右上'),
                  ),
                  // 左下角
                  Positioned(
                    left: 10,
                    bottom: 10,
                    child: _buildPositionedBox(Colors.blue, '左下'),
                  ),
                  // 右下角
                  Positioned(
                    right: 10,
                    bottom: 10,
                    child: _buildPositionedBox(Colors.orange, '右下'),
                  ),
                  // 居中
                  Positioned(
                    left: 0,
                    right: 0,
                    top: 0,
                    bottom: 0,
                    child: Center(
                      child: _buildPositionedBox(Colors.purple, '居中'),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // Positioned.fill
            _buildSectionTitle('3. Positioned.fill'),
            Container(
              width: double.infinity,
              height: 150,
              color: Colors.grey[200],
              child: Stack(
                children: [
                  Container(color: Colors.blue[100]),
                  Positioned.fill(
                    left: 20,
                    top: 20,
                    right: 20,
                    bottom: 20,
                    child: Container(
                      color: Colors.blue,
                      child: const Center(
                        child: Text(
                          'fill 带边距',
                          style: TextStyle(color: Colors.white),
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 图片叠加文字
            _buildSectionTitle('4. 图片叠加文字'),
            Container(
              width: double.infinity,
              height: 200,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(12),
              ),
              clipBehavior: Clip.antiAlias,
              child: Stack(
                fit: StackFit.expand,
                children: [
                  Image.network(
                    'https://picsum.photos/400/200',
                    fit: BoxFit.cover,
                  ),
                  // 渐变遮罩
                  Positioned.fill(
                    child: Container(
                      decoration: BoxDecoration(
                        gradient: LinearGradient(
                          begin: Alignment.topCenter,
                          end: Alignment.bottomCenter,
                          colors: [
                            Colors.transparent,
                            Colors.black.withOpacity(0.7),
                          ],
                        ),
                      ),
                    ),
                  ),
                  // 标题
                  const Positioned(
                    left: 16,
                    right: 16,
                    bottom: 16,
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          '美丽的风景',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          '拍摄于 2024 年',
                          style: TextStyle(
                            color: Colors.white70,
                            fontSize: 14,
                          ),
                        ),
                      ],
                    ),
                  ),
                  // 收藏按钮
                  const Positioned(
                    right: 16,
                    top: 16,
                    child: Icon(
                      Icons.favorite,
                      color: Colors.red,
                      size: 32,
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 对齐方式
            _buildSectionTitle('5. 对齐方式'),
            Row(
              children: [
                Expanded(
                  child: Column(
                    children: [
                      const Text('topStart'),
                      Container(
                        height: 100,
                        color: Colors.grey[200],
                        child: Stack(
                          alignment: AlignmentDirectional.topStart,
                          children: [
                            _buildSmallBox(Colors.red),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
                Expanded(
                  child: Column(
                    children: [
                      const Text('center'),
                      Container(
                        height: 100,
                        color: Colors.grey[200],
                        child: Stack(
                          alignment: Alignment.center,
                          children: [
                            _buildSmallBox(Colors.green),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
                Expanded(
                  child: Column(
                    children: [
                      const Text('bottomEnd'),
                      Container(
                        height: 100,
                        color: Colors.grey[200],
                        child: Stack(
                          alignment: AlignmentDirectional.bottomEnd,
                          children: [
                            _buildSmallBox(Colors.blue),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // 徽章效果
            _buildSectionTitle('6. 徽章效果'),
            Center(
              child: Stack(
                clipBehavior: Clip.none,
                children: [
                  Container(
                    width: 80,
                    height: 80,
                    decoration: BoxDecoration(
                      color: Colors.blue,
                      borderRadius: BorderRadius.circular(12),
                    ),
                    child: const Icon(Icons.shopping_cart, color: Colors.white, size: 40),
                  ),
                  Positioned(
                    right: -8,
                    top: -8,
                    child: Container(
                      padding: const EdgeInsets.all(6),
                      decoration: const BoxDecoration(
                        color: Colors.red,
                        shape: BoxShape.circle,
                      ),
                      child: const Text(
                        '3',
                        style: TextStyle(
                          color: Colors.white,
                          fontSize: 12,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // IndexedStack
            _buildSectionTitle('7. IndexedStack（只显示一个子元素）'),
            const IndexedStackDemo(),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }

  Widget _buildPositionedBox(Color color, String text) {
    return Container(
      width: 60,
      height: 40,
      color: color,
      child: Center(
        child: Text(
          text,
          style: const TextStyle(color: Colors.white, fontSize: 12),
        ),
      ),
    );
  }

  Widget _buildSmallBox(Color color) {
    return Container(
      width: 50,
      height: 50,
      color: color,
    );
  }
}

// IndexedStack 演示
class IndexedStackDemo extends StatefulWidget {
  const IndexedStackDemo({super.key});

  @override
  State<IndexedStackDemo> createState() => _IndexedStackDemoState();
}

class _IndexedStackDemoState extends State<IndexedStackDemo> {
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => setState(() => _index = 0),
              child: const Text('红'),
            ),
            const SizedBox(width: 8),
            ElevatedButton(
              onPressed: () => setState(() => _index = 1),
              child: const Text('绿'),
            ),
            const SizedBox(width: 8),
            ElevatedButton(
              onPressed: () => setState(() => _index = 2),
              child: const Text('蓝'),
            ),
          ],
        ),
        const SizedBox(height: 10),
        Container(
          width: 150,
          height: 150,
          color: Colors.grey[200],
          child: IndexedStack(
            index: _index,
            children: [
              Container(
                color: Colors.red,
                child: const Center(child: Text('红色页面', style: TextStyle(color: Colors.white))),
              ),
              Container(
                color: Colors.green,
                child: const Center(child: Text('绿色页面', style: TextStyle(color: Colors.white))),
              ),
              Container(
                color: Colors.blue,
                child: const Center(child: Text('蓝色页面', style: TextStyle(color: Colors.white))),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

> **补充知识**：`IndexedStack` 是 `Stack` 的子类，它只显示指定索引的子元素，但会保留所有子元素的状态。这在实现底部导航栏切换页面时非常有用。

---

## 第四章：文本组件

### 4.1 Text

`Text` 用于显示简单文本，是 Flutter 中最基础的文本显示 Widget。

#### 完整构造函数

```dart
const Text(
  String this.data, {
  super.key,
  this.style,
  this.strutStyle,
  this.textAlign,
  this.textDirection,
  this.locale,
  this.softWrap,
  this.overflow,
  @Deprecated('Use textScaler instead')
  this.textScaleFactor,
  this.textScaler,
  this.maxLines,
  this.semanticsLabel,
  this.textWidthBasis,
  this.textHeightBehavior,
  this.selectionColor,
});

const Text.rich(
  InlineSpan this.textSpan, {
  super.key,
  this.style,
  this.strutStyle,
  this.textAlign,
  this.textDirection,
  this.locale,
  this.softWrap,
  this.overflow,
  @Deprecated('Use textScaler instead')
  this.textScaleFactor,
  this.textScaler,
  this.maxLines,
  this.semanticsLabel,
  this.textWidthBasis,
  this.textHeightBehavior,
  this.selectionColor,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `data` | String | 必填 | 要显示的文本内容 |
| `textSpan` | InlineSpan | 必填（Text.rich） | 富文本片段 |
| `style` | TextStyle? | null | 文本样式 |
| `strutStyle` | StrutStyle? | null | 支柱样式，影响行高 |
| `textAlign` | TextAlign? | null | 文本对齐方式 |
| `textDirection` | TextDirection? | null | 文本方向 |
| `locale` | Locale? | null | 语言区域 |
| `softWrap` | bool? | null | 是否自动换行 |
| `overflow` | TextOverflow? | null | 溢出处理方式 |
| `textScaler` | TextScaler? | null | 文本缩放器 |
| `maxLines` | int? | null | 最大行数 |
| `semanticsLabel` | String? | null | 语义标签 |
| `textWidthBasis` | TextWidthBasis? | null | 文本宽度基准 |
| `textHeightBehavior` | TextHeightBehavior? | null | 行高行为 |
| `selectionColor` | Color? | null | 选中颜色 |

#### TextAlign 枚举值

| 值 | 说明 |
|----|------|
| `TextAlign.left` | 左对齐 |
| `TextAlign.right` | 右对齐 |
| `TextAlign.center` | 居中对齐 |
| `TextAlign.justify` | 两端对齐 |
| `TextAlign.start` | 起始方向对齐 |
| `TextAlign.end` | 结束方向对齐 |

#### TextOverflow 枚举值

| 值 | 说明 |
|----|------|
| `TextOverflow.clip` | 直接裁剪 |
| `TextOverflow.fade` | 渐变淡出 |
| `TextOverflow.ellipsis` | 显示省略号 |
| `TextOverflow.visible` | 溢出可见 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Text Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const TextDemoPage(),
    );
  }
}

class TextDemoPage extends StatelessWidget {
  const TextDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Text 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础文本
            _buildSectionTitle('1. 基础文本'),
            const Text('这是一段普通文本'),
            const SizedBox(height: 20),

            // 文本样式
            _buildSectionTitle('2. 文本样式'),
            const Text(
              '自定义样式的文本',
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
                letterSpacing: 2,
                wordSpacing: 4,
                decoration: TextDecoration.underline,
                decorationColor: Colors.red,
                decorationStyle: TextDecorationStyle.wavy,
              ),
            ),
            const SizedBox(height: 20),

            // 对齐方式
            _buildSectionTitle('3. 对齐方式'),
            Container(
              width: double.infinity,
              color: Colors.grey[200],
              child: const Column(
                children: [
                  Text('左对齐', textAlign: TextAlign.left),
                  Text('居中对齐', textAlign: TextAlign.center),
                  Text('右对齐', textAlign: TextAlign.right),
                  Text(
                    '两端对齐：这是一段很长的文本，用于演示两端对齐的效果',
                    textAlign: TextAlign.justify,
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 溢出处理
            _buildSectionTitle('4. 溢出处理'),
            Container(
              width: 200,
              color: Colors.grey[200],
              child: const Column(
                children: [
                  Text(
                    'clip: 这是一段很长的文本，超出部分会被直接裁剪',
                    overflow: TextOverflow.clip,
                    maxLines: 1,
                  ),
                  SizedBox(height: 8),
                  Text(
                    'ellipsis: 这是一段很长的文本，超出部分会显示省略号',
                    overflow: TextOverflow.ellipsis,
                    maxLines: 1,
                  ),
                  SizedBox(height: 8),
                  Text(
                    'fade: 这是一段很长的文本，超出部分会渐变淡出',
                    overflow: TextOverflow.fade,
                    maxLines: 1,
                    softWrap: false,
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 富文本
            _buildSectionTitle('5. 富文本 (Text.rich)'),
            Text.rich(
              TextSpan(
                text: '普通文本 ',
                style: const TextStyle(fontSize: 16),
                children: [
                  const TextSpan(
                    text: '粗体 ',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const TextSpan(
                    text: '斜体 ',
                    style: TextStyle(fontStyle: FontStyle.italic),
                  ),
                  TextSpan(
                    text: '带颜色的 ',
                    style: TextStyle(color: Colors.red[700]),
                  ),
                  const WidgetSpan(
                    child: Icon(Icons.star, size: 16, color: Colors.orange),
                  ),
                  const TextSpan(text: ' 和图标'),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 可点击文本
            _buildSectionTitle('6. 可点击文本'),
            GestureDetector(
              onTap: () {
                debugPrint('文本被点击');
              },
              child: const Text(
                '点击这段文本',
                style: TextStyle(
                  color: Colors.blue,
                  decoration: TextDecoration.underline,
                ),
              ),
            ),
            const SizedBox(height: 20),

            // 行数限制
            _buildSectionTitle('7. 行数限制'),
            Container(
              width: double.infinity,
              color: Colors.grey[200],
              child: const Text(
                '这是一段很长的文本。这是一段很长的文本。这是一段很长的文本。'
                '这是一段很长的文本。这是一段很长的文本。这是一段很长的文本。'
                '这是一段很长的文本。这是一段很长的文本。这是一段很长的文本。',
                maxLines: 3,
                overflow: TextOverflow.ellipsis,
              ),
            ),
            const SizedBox(height: 20),

            // 字体族
            _buildSectionTitle('8. 字体族'),
            const Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '默认字体',
                  style: TextStyle(fontSize: 18),
                ),
                Text(
                  'Roboto 字体',
                  style: TextStyle(fontSize: 18, fontFamily: 'Roboto'),
                ),
                Text(
                  '等宽字体',
                  style: TextStyle(fontSize: 18, fontFamily: 'monospace'),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // 文本阴影
            _buildSectionTitle('9. 文本阴影'),
            Text(
              '带阴影的文本',
              style: TextStyle(
                fontSize: 32,
                fontWeight: FontWeight.bold,
                color: Colors.white,
                shadows: [
                  Shadow(
                    color: Colors.black.withOpacity(0.5),
                    offset: const Offset(2, 2),
                    blurRadius: 4,
                  ),
                  const Shadow(
                    color: Colors.blue,
                    offset: Offset(-1, -1),
                    blurRadius: 2,
                  ),
                ],
              ),
            ),
            const SizedBox(height: 20),

            // 渐变文本
            _buildSectionTitle('10. 渐变文本'),
            ShaderMask(
              shaderCallback: (bounds) {
                return const LinearGradient(
                  colors: [Colors.purple, Colors.blue, Colors.green],
                ).createShader(bounds);
              },
              child: const Text(
                '渐变颜色文本',
                style: TextStyle(
                  fontSize: 32,
                  fontWeight: FontWeight.bold,
                  color: Colors.white,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

> **补充知识**：Flutter 的文本渲染引擎 Skia 提供了高质量的文本渲染。通过 `TextPainter` 类可以实现更复杂的文本布局和测量需求。

---

## 第五章：按钮组件

### 5.1 ElevatedButton

`ElevatedButton` 是 Material Design 3 中的凸起按钮，具有阴影效果，用于强调主要操作。

#### 完整构造函数

```dart
const ElevatedButton({
  super.key,
  required super.onPressed,
  super.onLongPress,
  super.onHover,
  super.onFocusChange,
  super.style,
  super.focusNode,
  super.autofocus = false,
  super.clipBehavior = Clip.none,
  super.statesController,
  required super.child,
});

factory ElevatedButton.icon({
  Key? key,
  required VoidCallback? onPressed,
  VoidCallback? onLongPress,
  ValueChanged<bool>? onHover,
  ValueChanged<bool>? onFocusChange,
  ButtonStyle? style,
  FocusNode? focusNode,
  bool? autofocus,
  Clip? clipBehavior,
  MaterialStatesController? statesController,
  required Widget icon,
  required Widget label,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `onPressed` | VoidCallback? | 必填 | 点击回调，null 时按钮禁用 |
| `onLongPress` | VoidCallback? | null | 长按回调 |
| `onHover` | ValueChanged<bool>? | null | 悬停状态变化回调 |
| `onFocusChange` | ValueChanged<bool>? | null | 焦点状态变化回调 |
| `style` | ButtonStyle? | null | 按钮样式 |
| `focusNode` | FocusNode? | null | 焦点节点 |
| `autofocus` | bool | false | 是否自动获取焦点 |
| `clipBehavior` | Clip | Clip.none | 裁剪行为 |
| `statesController` | MaterialStatesController? | null | 状态控制器 |
| `child` | Widget | 必填 | 子元素 |

#### ButtonStyle 常用属性

```dart
ButtonStyle(
  foregroundColor: MaterialStateProperty.all<Color>(Colors.white),
  backgroundColor: MaterialStateProperty.all<Color>(Colors.blue),
  overlayColor: MaterialStateProperty.all<Color>(Colors.blue.withOpacity(0.2)),
  elevation: MaterialStateProperty.all<double>(4),
  padding: MaterialStateProperty.all<EdgeInsets>(EdgeInsets.symmetric(horizontal: 16, vertical: 8)),
  minimumSize: MaterialStateProperty.all<Size>(Size(88, 36)),
  maximumSize: MaterialStateProperty.all<Size>(Size.infinite),
  shape: MaterialStateProperty.all<OutlinedBorder>(RoundedRectangleBorder(borderRadius: BorderRadius.circular(8))),
  side: MaterialStateProperty.all<BorderSide>(BorderSide(color: Colors.blue)),
  textStyle: MaterialStateProperty.all<TextStyle>(TextStyle(fontSize: 16)),
  alignment: Alignment.center,
  animationDuration: Duration(milliseconds: 200),
  enableFeedback: true,
  splashFactory: InkRipple.splashFactory,
)
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ElevatedButton Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ElevatedButtonDemoPage(),
    );
  }
}

class ElevatedButtonDemoPage extends StatefulWidget {
  const ElevatedButtonDemoPage({super.key});

  @override
  State<ElevatedButtonDemoPage> createState() => _ElevatedButtonDemoPageState();
}

class _ElevatedButtonDemoPageState extends State<ElevatedButtonDemoPage> {
  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ElevatedButton 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础按钮
            _buildSectionTitle('1. 基础按钮'),
            ElevatedButton(
              onPressed: () {
                debugPrint('ElevatedButton 被点击');
              },
              child: const Text('点击我'),
            ),
            const SizedBox(height: 20),

            // 禁用状态
            _buildSectionTitle('2. 禁用状态'),
            const ElevatedButton(
              onPressed: null,
              child: Text('禁用按钮'),
            ),
            const SizedBox(height: 20),

            // 图标按钮
            _buildSectionTitle('3. 图标按钮'),
            ElevatedButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.add),
              label: const Text('添加'),
            ),
            const SizedBox(height: 20),

            // 自定义样式
            _buildSectionTitle('4. 自定义样式'),
            ElevatedButton(
              onPressed: () {},
              style: ElevatedButton.styleFrom(
                foregroundColor: Colors.white,
                backgroundColor: Colors.purple,
                elevation: 8,
                padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(30),
                ),
                shadowColor: Colors.purple.withOpacity(0.5),
              ),
              child: const Text('自定义样式'),
            ),
            const SizedBox(height: 20),

            // 状态响应样式
            _buildSectionTitle('5. 状态响应样式'),
            ElevatedButton(
              onPressed: () {},
              style: ButtonStyle(
                backgroundColor: MaterialStateProperty.resolveWith<Color>(
                  (states) {
                    if (states.contains(MaterialState.pressed)) {
                      return Colors.blue[700]!;
                    }
                    if (states.contains(MaterialState.hovered)) {
                      return Colors.blue[600]!;
                    }
                    if (states.contains(MaterialState.disabled)) {
                      return Colors.grey;
                    }
                    return Colors.blue;
                  },
                ),
                foregroundColor: MaterialStateProperty.all(Colors.white),
                elevation: MaterialStateProperty.resolveWith<double>(
                  (states) {
                    if (states.contains(MaterialState.pressed)) {
                      return 8;
                    }
                    return 4;
                  },
                ),
              ),
              child: const Text('状态响应'),
            ),
            const SizedBox(height: 20),

            // 加载按钮
            _buildSectionTitle('6. 加载按钮'),
            ElevatedButton(
              onPressed: _isLoading
                  ? null
                  : () {
                      setState(() {
                        _isLoading = true;
                      });
                      Future.delayed(const Duration(seconds: 2), () {
                        setState(() {
                          _isLoading = false;
                        });
                      });
                    },
              child: _isLoading
                  ? const SizedBox(
                      width: 20,
                      height: 20,
                      child: CircularProgressIndicator(
                        strokeWidth: 2,
                        valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
                      ),
                    )
                  : const Text('提交'),
            ),
            const SizedBox(height: 20),

            // 按钮尺寸
            _buildSectionTitle('7. 按钮尺寸'),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    minimumSize: const Size(64, 32),
                  ),
                  child: const Text('小'),
                ),
                ElevatedButton(
                  onPressed: () {},
                  child: const Text('中（默认）'),
                ),
                ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(horizontal: 48, vertical: 20),
                  ),
                  child: const Text('大'),
                ),
              ],
            ),
            const SizedBox(height: 20),

            // 渐变背景按钮
            _buildSectionTitle('8. 渐变背景按钮'),
            Container(
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Colors.orange, Colors.red],
                ),
                borderRadius: BorderRadius.circular(8),
              ),
              child: ElevatedButton(
                onPressed: () {},
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.transparent,
                  shadowColor: Colors.transparent,
                  foregroundColor: Colors.white,
                ),
                child: const Text('渐变按钮'),
              ),
            ),
            const SizedBox(height: 20),

            // 圆角和形状
            _buildSectionTitle('9. 圆角和形状'),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    shape: const StadiumBorder(),
                  ),
                  child: const Text('胶囊形'),
                ),
                ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    shape: const CircleBorder(),
                    padding: const EdgeInsets.all(20),
                  ),
                  child: const Icon(Icons.add),
                ),
                ElevatedButton(
                  onPressed: () {},
                  style: ElevatedButton.styleFrom(
                    shape: const BeveledRectangleBorder(
                      borderRadius: BorderRadius.all(Radius.circular(8)),
                    ),
                  ),
                  child: const Text('斜角矩形'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

---

### 5.2 TextButton

`TextButton` 是文本按钮，没有背景和阴影，用于次要操作或导航。

#### 完整构造函数

```dart
const TextButton({
  super.key,
  required super.onPressed,
  super.onLongPress,
  super.onHover,
  super.onFocusChange,
  super.style,
  super.focusNode,
  super.autofocus = false,
  super.clipBehavior = Clip.none,
  super.statesController,
  required super.child,
});

factory TextButton.icon({
  Key? key,
  required VoidCallback? onPressed,
  VoidCallback? onLongPress,
  ValueChanged<bool>? onHover,
  ValueChanged<bool>? onFocusChange,
  ButtonStyle? style,
  FocusNode? focusNode,
  bool? autofocus,
  Clip? clipBehavior,
  MaterialStatesController? statesController,
  required Widget icon,
  required Widget label,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class TextButtonDemoPage extends StatelessWidget {
  const TextButtonDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextButton 演示')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础文本按钮
            TextButton(
              onPressed: () {},
              child: const Text('文本按钮'),
            ),
            const SizedBox(height: 16),

            // 图标文本按钮
            TextButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.arrow_back),
              label: const Text('返回'),
            ),
            const SizedBox(height: 16),

            // 自定义样式
            TextButton(
              onPressed: () {},
              style: TextButton.styleFrom(
                foregroundColor: Colors.red,
                padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
              ),
              child: const Text('自定义颜色'),
            ),
            const SizedBox(height: 16),

            // 对话框按钮组
            Row(
              mainAxisAlignment: MainAxisAlignment.end,
              children: [
                TextButton(
                  onPressed: () {},
                  child: const Text('取消'),
                ),
                const SizedBox(width: 8),
                TextButton(
                  onPressed: () {},
                  child: const Text('确定'),
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

### 5.3 OutlinedButton

`OutlinedButton` 是带边框的按钮，用于中等重要性的操作。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class OutlinedButtonDemoPage extends StatelessWidget {
  const OutlinedButtonDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('OutlinedButton 演示')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础边框按钮
            OutlinedButton(
              onPressed: () {},
              child: const Text('边框按钮'),
            ),
            const SizedBox(height: 16),

            // 图标边框按钮
            OutlinedButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.settings),
              label: const Text('设置'),
            ),
            const SizedBox(height: 16),

            // 自定义边框样式
            OutlinedButton(
              onPressed: () {},
              style: OutlinedButton.styleFrom(
                foregroundColor: Colors.green,
                side: const BorderSide(color: Colors.green, width: 2),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(20),
                ),
              ),
              child: const Text('自定义边框'),
            ),
            const SizedBox(height: 16),

            // 虚线边框（使用自定义绘制）
            OutlinedButton(
              onPressed: () {},
              style: ButtonStyle(
                side: MaterialStateProperty.all(
                  const BorderSide(color: Colors.purple),
                ),
                shape: MaterialStateProperty.all(
                  RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
              ),
              child: const Text('紫色边框'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 5.4 IconButton

`IconButton` 是图标按钮，通常用于 AppBar 的操作按钮。

#### 完整构造函数

```dart
const IconButton({
  super.key,
  this.iconSize,
  this.visualDensity,
  this.padding = const EdgeInsets.all(8.0),
  this.alignment = Alignment.center,
  this.splashRadius,
  this.color,
  this.focusColor,
  this.hoverColor,
  this.highlightColor,
  this.splashColor,
  this.disabledColor,
  required this.onPressed,
  this.mouseCursor = SystemMouseCursors.click,
  this.focusNode,
  this.autofocus = false,
  this.tooltip,
  this.enableFeedback = true,
  this.constraints,
  this.style,
  this.isSelected,
  this.selectedIcon,
  required this.icon,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class IconButtonDemoPage extends StatefulWidget {
  const IconButtonDemoPage({super.key});

  @override
  State<IconButtonDemoPage> createState() => _IconButtonDemoPageState();
}

class _IconButtonDemoPageState extends State<IconButtonDemoPage> {
  bool _isFavorite = false;
  bool _isSelected = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('IconButton 演示')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础图标按钮
            Row(
              children: [
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.menu),
                ),
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.search),
                ),
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.more_vert),
                ),
              ],
            ),
            const SizedBox(height: 16),

            // 带提示的图标按钮
            IconButton(
              onPressed: () {},
              icon: const Icon(Icons.delete),
              tooltip: '删除',
              color: Colors.red,
            ),
            const SizedBox(height: 16),

            // 可切换图标按钮
            IconButton(
              onPressed: () {
                setState(() {
                  _isFavorite = !_isFavorite;
                });
              },
              icon: Icon(
                _isFavorite ? Icons.favorite : Icons.favorite_border,
                color: _isFavorite ? Colors.red : null,
              ),
              iconSize: 32,
            ),
            const SizedBox(height: 16),

            // 选中状态图标按钮
            IconButton(
              isSelected: _isSelected,
              onPressed: () {
                setState(() {
                  _isSelected = !_isSelected;
                });
              },
              icon: const Icon(Icons.bookmark_border),
              selectedIcon: const Icon(Icons.bookmark),
              color: Colors.blue,
            ),
            const SizedBox(height: 16),

            // 自定义样式
            IconButton(
              onPressed: () {},
              icon: const Icon(Icons.add_circle),
              iconSize: 48,
              color: Colors.green,
              splashColor: Colors.green.withOpacity(0.3),
              highlightColor: Colors.green.withOpacity(0.2),
            ),
            const SizedBox(height: 16),

            // 样式化图标按钮
            IconButton(
              onPressed: () {},
              icon: const Icon(Icons.edit),
              style: IconButton.styleFrom(
                backgroundColor: Colors.blue,
                foregroundColor: Colors.white,
                padding: const EdgeInsets.all(12),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

> **补充知识**：从 Flutter 3.0 开始，按钮 API 进行了统一。`ElevatedButton`、`TextButton`、`OutlinedButton` 都使用 `ButtonStyle` 进行样式设置，取代了旧的 `RaisedButton`、`FlatButton`、`OutlineButton`。

---

## 第六章：输入组件

### 6.1 TextField

`TextField` 是 Material Design 的文本输入框，支持各种输入类型和样式。

#### 完整构造函数

```dart
const TextField({
  super.key,
  this.controller,
  this.focusNode,
  this.undoController,
  this.decoration = const InputDecoration(),
  TextInputType? keyboardType,
  this.textInputAction,
  this.textCapitalization = TextCapitalization.none,
  this.style,
  this.strutStyle,
  this.textAlign = TextAlign.start,
  this.textAlignVertical,
  this.textDirection,
  this.readOnly = false,
  this.showCursor,
  this.autofocus = false,
  this.obscuringCharacter = '•',
  this.obscureText = false,
  this.autocorrect = true,
  SmartDashesType? smartDashesType,
  SmartQuotesType? smartQuotesType,
  this.enableSuggestions = true,
  this.maxLines = 1,
  this.minLines,
  this.expands = false,
  this.maxLength,
  this.maxLengthEnforcement,
  this.onChanged,
  this.onEditingComplete,
  this.onSubmitted,
  this.onAppPrivateCommand,
  this.inputFormatters,
  this.enabled,
  this.cursorWidth = 2.0,
  this.cursorHeight,
  this.cursorRadius,
  this.cursorOpacityAnimates,
  this.cursorColor,
  this.cursorErrorColor,
  this.selectionHeightStyle = ui.BoxHeightStyle.tight,
  this.selectionWidthStyle = ui.BoxWidthStyle.tight,
  this.keyboardAppearance,
  this.scrollPadding = const EdgeInsets.all(20.0),
  this.dragStartBehavior = DragStartBehavior.start,
  bool? enableInteractiveSelection,
  this.selectionControls,
  this.onTap,
  this.onTapAlwaysCalled = false,
  this.onTapOutside,
  this.mouseCursor,
  this.buildCounter,
  this.scrollController,
  this.scrollPhysics,
  this.autofillHints = const <String>[],
  this.contentInsertionConfiguration,
  this.clipBehavior = Clip.hardEdge,
  this.restorationId,
  this.scribbleEnabled = true,
  this.enableIMEPersonalizedLearning = true,
  this.contextMenuBuilder = _defaultContextMenuBuilder,
  this.canRequestFocus = true,
  this.spellCheckConfiguration,
  this.magnifierConfiguration,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `controller` | TextEditingController? | null | 文本控制器 |
| `focusNode` | FocusNode? | null | 焦点节点 |
| `decoration` | InputDecoration | InputDecoration() | 输入框装饰 |
| `keyboardType` | TextInputType? | null | 键盘类型 |
| `textInputAction` | TextInputAction? | null | 键盘动作按钮 |
| `textCapitalization` | TextCapitalization | TextCapitalization.none | 自动大写 |
| `style` | TextStyle? | null | 文本样式 |
| `textAlign` | TextAlign | TextAlign.start | 文本对齐 |
| `readOnly` | bool | false | 是否只读 |
| `autofocus` | bool | false | 是否自动聚焦 |
| `obscureText` | bool | false | 是否隐藏文本（密码） |
| `autocorrect` | bool | true | 是否自动纠正 |
| `maxLines` | int? | 1 | 最大行数 |
| `minLines` | int? | null | 最小行数 |
| `maxLength` | int? | null | 最大长度 |
| `onChanged` | ValueChanged<String>? | null | 内容变化回调 |
| `onSubmitted` | ValueChanged<String>? | null | 提交回调 |
| `inputFormatters` | List<TextInputFormatter>? | null | 输入格式化器 |
| `enabled` | bool? | null | 是否启用 |
| `cursorColor` | Color? | null | 光标颜色 |

#### TextInputType 枚举值

| 值 | 说明 |
|----|------|
| `TextInputType.text` | 普通文本 |
| `TextInputType.multiline` | 多行文本 |
| `TextInputType.number` | 数字 |
| `TextInputType.phone` | 电话号码 |
| `TextInputType.datetime` | 日期时间 |
| `TextInputType.emailAddress` | 邮箱地址 |
| `TextInputType.url` | URL |
| `TextInputType.visiblePassword` | 可见密码 |
| `TextInputType.name` | 人名 |
| `TextInputType.streetAddress` | 街道地址 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TextField Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const TextFieldDemoPage(),
    );
  }
}

class TextFieldDemoPage extends StatefulWidget {
  const TextFieldDemoPage({super.key});

  @override
  State<TextFieldDemoPage> createState() => _TextFieldDemoPageState();
}

class _TextFieldDemoPageState extends State<TextFieldDemoPage> {
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  final TextEditingController _phoneController = TextEditingController();
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _bioController = TextEditingController();
  final TextEditingController _amountController = TextEditingController();
  
  bool _obscurePassword = true;
  String _inputValue = '';

  @override
  void dispose() {
    _nameController.dispose();
    _passwordController.dispose();
    _phoneController.dispose();
    _emailController.dispose();
    _bioController.dispose();
    _amountController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TextField 完整演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础输入框
            _buildSectionTitle('1. 基础输入框'),
            TextField(
              onChanged: (value) {
                setState(() {
                  _inputValue = value;
                });
              },
              decoration: const InputDecoration(
                hintText: '请输入内容',
              ),
            ),
            Text('输入内容: $_inputValue'),
            const SizedBox(height: 20),

            // 带标签的输入框
            _buildSectionTitle('2. 带标签的输入框'),
            TextField(
              controller: _nameController,
              decoration: const InputDecoration(
                labelText: '用户名',
                hintText: '请输入用户名',
                prefixIcon: Icon(Icons.person),
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),

            // 密码输入框
            _buildSectionTitle('3. 密码输入框'),
            TextField(
              controller: _passwordController,
              obscureText: _obscurePassword,
              decoration: InputDecoration(
                labelText: '密码',
                hintText: '请输入密码',
                prefixIcon: const Icon(Icons.lock),
                suffixIcon: IconButton(
                  icon: Icon(
                    _obscurePassword ? Icons.visibility_off : Icons.visibility,
                  ),
                  onPressed: () {
                    setState(() {
                      _obscurePassword = !_obscurePassword;
                    });
                  },
                ),
                border: const OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),

            // 手机号输入框
            _buildSectionTitle('4. 手机号输入框'),
            TextField(
              controller: _phoneController,
              keyboardType: TextInputType.phone,
              inputFormatters: [
                FilteringTextInputFormatter.digitsOnly,
                LengthLimitingTextInputFormatter(11),
              ],
              decoration: const InputDecoration(
                labelText: '手机号',
                hintText: '请输入11位手机号',
                prefixIcon: Icon(Icons.phone),
                prefixText: '+86 ',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),

            // 邮箱输入框
            _buildSectionTitle('5. 邮箱输入框'),
            TextField(
              controller: _emailController,
              keyboardType: TextInputType.emailAddress,
              decoration: const InputDecoration(
                labelText: '邮箱',
                hintText: 'example@email.com',
                prefixIcon: Icon(Icons.email),
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),

            // 多行文本输入
            _buildSectionTitle('6. 多行文本输入'),
            TextField(
              controller: _bioController,
              maxLines: 5,
              minLines: 3,
              maxLength: 200,
              decoration: const InputDecoration(
                labelText: '个人简介',
                hintText: '请简单介绍自己...',
                border: OutlineInputBorder(),
                alignLabelWithHint: true,
              ),
            ),
            const SizedBox(height: 20),

            // 金额输入框
            _buildSectionTitle('7. 金额输入框'),
            TextField(
              controller: _amountController,
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              inputFormatters: [
                FilteringTextInputFormatter.allow(RegExp(r'^\d*\.?\d{0,2}')),
              ],
              decoration: const InputDecoration(
                labelText: '金额',
                prefixIcon: Icon(Icons.attach_money),
                prefixText: '¥ ',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),

            // 样式化输入框
            _buildSectionTitle('8. 样式化输入框'),
            TextField(
              decoration: InputDecoration(
                labelText: '自定义样式',
                filled: true,
                fillColor: Colors.blue[50],
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  borderSide: BorderSide.none,
                ),
                enabledBorder: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  borderSide: BorderSide(color: Colors.blue[200]!),
                ),
                focusedBorder: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  borderSide: const BorderSide(color: Colors.blue, width: 2),
                ),
                errorBorder: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                  borderSide: const BorderSide(color: Colors.red),
                ),
                prefixIcon: const Icon(Icons.style),
              ),
            ),
            const SizedBox(height: 20),

            // 搜索输入框
            _buildSectionTitle('9. 搜索输入框'),
            TextField(
              textInputAction: TextInputAction.search,
              onSubmitted: (value) {
                debugPrint('搜索: $value');
              },
              decoration: InputDecoration(
                hintText: '搜索...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: IconButton(
                  icon: const Icon(Icons.clear),
                  onPressed: () {},
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(30),
                ),
                contentPadding: const EdgeInsets.symmetric(vertical: 0),
              ),
            ),
            const SizedBox(height: 20),

            // 提交按钮
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  debugPrint('用户名: ${_nameController.text}');
                  debugPrint('密码: ${_passwordController.text}');
                  debugPrint('手机号: ${_phoneController.text}');
                  debugPrint('邮箱: ${_emailController.text}');
                  debugPrint('简介: ${_bioController.text}');
                },
                child: const Text('提交'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
      ),
    );
  }
}
```

> **补充知识**：`TextField` 和 `TextFormField` 的区别在于后者集成了表单验证功能。如果你需要表单验证，应该使用 `TextFormField`。

---

## 第七章：列表和滚动组件

### 7.1 ListView

`ListView` 是最常用的滚动列表组件，支持垂直和水平方向的滚动。

#### 完整构造函数

```dart
ListView({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  this.itemExtent,
  this.prototypeItem,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  List<Widget> children = const <Widget>[],
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});

ListView.builder({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  this.itemExtent,
  this.prototypeItem,
  required IndexedWidgetBuilder itemBuilder,
  ChildIndexGetter? findChildIndexCallback,
  int? itemCount,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});

ListView.separated({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  required IndexedWidgetBuilder itemBuilder,
  required IndexedWidgetBuilder separatorBuilder,
  ChildIndexGetter? findChildIndexCallback,
  required int itemCount,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});
```

#### 核心属性详解

| 属性名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `scrollDirection` | Axis | Axis.vertical | 滚动方向 |
| `reverse` | bool | false | 是否反向滚动 |
| `controller` | ScrollController? | null | 滚动控制器 |
| `primary` | bool? | null | 是否使用主滚动控制器 |
| `physics` | ScrollPhysics? | null | 滚动物理效果 |
| `shrinkWrap` | bool | false | 是否根据内容调整高度 |
| `padding` | EdgeInsetsGeometry? | null | 内边距 |
| `itemExtent` | double? | null | 固定子元素高度/宽度 |
| `children` | List<Widget> | [] | 子元素列表 |
| `itemBuilder` | IndexedWidgetBuilder | 必填（builder） | 子元素构建器 |
| `separatorBuilder` | IndexedWidgetBuilder | 必填（separated） | 分隔符构建器 |
| `itemCount` | int? | null | 子元素数量 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ListView Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ListViewDemoPage(),
    );
  }
}

class ListViewDemoPage extends StatelessWidget {
  const ListViewDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 4,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('ListView 完整演示'),
          bottom: const TabBar(
            tabs: [
              Tab(text: '基础'),
              Tab(text: 'Builder'),
              Tab(text: '分隔线'),
              Tab(text: '水平'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            BasicListView(),
            BuilderListView(),
            SeparatedListView(),
            HorizontalListView(),
          ],
        ),
      ),
    );
  }
}

// 基础 ListView
class BasicListView extends StatelessWidget {
  const BasicListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(16),
      children: const [
        ListTile(
          leading: Icon(Icons.person),
          title: Text('张三'),
          subtitle: Text('软件工程师'),
          trailing: Icon(Icons.arrow_forward_ios),
        ),
        Divider(),
        ListTile(
          leading: Icon(Icons.person),
          title: Text('李四'),
          subtitle: Text('产品经理'),
          trailing: Icon(Icons.arrow_forward_ios),
        ),
        Divider(),
        ListTile(
          leading: Icon(Icons.person),
          title: Text('王五'),
          subtitle: Text('UI设计师'),
          trailing: Icon(Icons.arrow_forward_ios),
        ),
        Divider(),
        Card(
          child: ListTile(
            leading: Icon(Icons.notifications),
            title: Text('通知'),
            trailing: Badge(
              child: Icon(Icons.chevron_right),
            ),
          ),
        ),
        Card(
          child: ListTile(
            leading: Icon(Icons.settings),
            title: Text('设置'),
            trailing: Icon(Icons.chevron_right),
          ),
        ),
      ],
    );
  }
}

// Builder ListView
class BuilderListView extends StatefulWidget {
  const BuilderListView({super.key});

  @override
  State<BuilderListView> createState() => _BuilderListViewState();
}

class _BuilderListViewState extends State<BuilderListView> {
  final List<Map<String, String>> _items = List.generate(
    50,
    (index) => {
      'title': 'Item ${index + 1}',
      'subtitle': '这是第 ${index + 1} 项的描述',
    },
  );

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: _items.length,
      itemBuilder: (context, index) {
        return Card(
          margin: const EdgeInsets.only(bottom: 8),
          child: ListTile(
            leading: CircleAvatar(
              child: Text('${index + 1}'),
            ),
            title: Text(_items[index]['title']!),
            subtitle: Text(_items[index]['subtitle']!),
            trailing: const Icon(Icons.arrow_forward_ios, size: 16),
            onTap: () {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('点击了 ${_items[index]['title']}')),
              );
            },
          ),
        );
      },
    );
  }
}

// 带分隔线的 ListView
class SeparatedListView extends StatelessWidget {
  const SeparatedListView({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.separated(
      padding: const EdgeInsets.all(16),
      itemCount: 30,
      separatorBuilder: (context, index) => const Divider(height: 1),
      itemBuilder: (context, index) {
        return ListTile(
          leading: Container(
            width: 48,
            height: 48,
            decoration: BoxDecoration(
              color: Colors.primaries[index % Colors.primaries.length],
              borderRadius: BorderRadius.circular(8),
            ),
            child: const Icon(Icons.image, color: Colors.white),
          ),
          title: Text('商品 ${index + 1}'),
          subtitle: Text('¥${(index + 1) * 10}.00'),
          trailing: ElevatedButton(
            onPressed: () {},
            child: const Text('购买'),
          ),
        );
      },
    );
  }
}

// 水平 ListView
class HorizontalListView extends StatelessWidget {
  const HorizontalListView({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 水平滚动列表
        SizedBox(
          height: 120,
          child: ListView.builder(
            scrollDirection: Axis.horizontal,
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            itemCount: 10,
            itemBuilder: (context, index) {
              return Container(
                width: 100,
                margin: const EdgeInsets.only(right: 12),
                decoration: BoxDecoration(
                  color: Colors.primaries[index % Colors.primaries.length],
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Center(
                  child: Text(
                    '分类 ${index + 1}',
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              );
            },
          ),
        ),
        // 垂直列表
        Expanded(
          child: ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: 20,
            itemBuilder: (context, index) {
              return Card(
                margin: const EdgeInsets.only(bottom: 12),
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Row(
                    children: [
                      Container(
                        width: 80,
                        height: 80,
                        color: Colors.grey[300],
                        child: const Icon(Icons.image),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              '商品标题 ${index + 1}',
                              style: const TextStyle(
                                fontSize: 16,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            const SizedBox(height: 4),
                            Text(
                              '商品描述信息',
                              style: TextStyle(
                                fontSize: 14,
                                color: Colors.grey[600],
                              ),
                            ),
                            const SizedBox(height: 8),
                            Text(
                              '¥99.00',
                              style: TextStyle(
                                fontSize: 18,
                                fontWeight: FontWeight.bold,
                                color: Colors.red[700],
                              ),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              );
            },
          ),
        ),
      ],
    );
  }
}
```

---

### 7.2 GridView

`GridView` 用于显示网格布局，支持多种布局模式。

#### 完整构造函数

```dart
GridView({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  required this.gridDelegate,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  List<Widget> children = const <Widget>[],
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});

GridView.builder({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  required this.gridDelegate,
  required IndexedWidgetBuilder itemBuilder,
  ChildIndexGetter? findChildIndexCallback,
  int? itemCount,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});

GridView.count({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  required int crossAxisCount,
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  List<Widget> children = const <Widget>[],
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});

GridView.extent({
  super.key,
  super.scrollDirection,
  super.reverse,
  super.controller,
  super.primary,
  super.physics,
  super.shrinkWrap,
  super.padding,
  required double maxCrossAxisExtent,
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  bool addSemanticIndexes = true,
  super.cacheExtent,
  List<Widget> children = const <Widget>[],
  super.semanticChildCount,
  super.dragStartBehavior,
  super.keyboardDismissBehavior,
  super.restorationId,
  super.clipBehavior,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class GridViewDemoPage extends StatelessWidget {
  const GridViewDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('GridView 演示'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'Count'),
              Tab(text: 'Builder'),
              Tab(text: 'Extent'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            GridViewCount(),
            GridViewBuilder(),
            GridViewExtent(),
          ],
        ),
      ),
    );
  }
}

// GridView.count
class GridViewCount extends StatelessWidget {
  const GridViewCount({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.count(
      padding: const EdgeInsets.all(16),
      crossAxisCount: 2,
      mainAxisSpacing: 16,
      crossAxisSpacing: 16,
      childAspectRatio: 0.75,
      children: List.generate(
        20,
        (index) => Card(
          clipBehavior: Clip.antiAlias,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Expanded(
                child: Container(
                  color: Colors.primaries[index % Colors.primaries.length],
                  child: const Center(
                    child: Icon(Icons.image, size: 48, color: Colors.white),
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      '商品 ${index + 1}',
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '¥${(index + 1) * 10}',
                      style: TextStyle(
                        color: Colors.red[700],
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// GridView.builder
class GridViewBuilder extends StatelessWidget {
  const GridViewBuilder({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      padding: const EdgeInsets.all(16),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 3,
        mainAxisSpacing: 8,
        crossAxisSpacing: 8,
      ),
      itemCount: 50,
      itemBuilder: (context, index) {
        return Container(
          decoration: BoxDecoration(
            color: Colors.primaries[index % Colors.primaries.length],
            borderRadius: BorderRadius.circular(8),
          ),
          child: Center(
            child: Text(
              '${index + 1}',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        );
      },
    );
  }
}

// GridView.extent
class GridViewExtent extends StatelessWidget {
  const GridViewExtent({super.key});

  @override
  Widget build(BuildContext context) {
    return GridView.extent(
      padding: const EdgeInsets.all(16),
      maxCrossAxisExtent: 150,
      mainAxisSpacing: 16,
      crossAxisSpacing: 16,
      children: List.generate(
        30,
        (index) => Container(
          decoration: BoxDecoration(
            color: Colors.primaries[index % Colors.primaries.length],
            borderRadius: BorderRadius.circular(12),
          ),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.widgets, color: Colors.white, size: 40),
              const SizedBox(height: 8),
              Text(
                'Item ${index + 1}',
                style: const TextStyle(color: Colors.white),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

> **补充知识**：`SliverGridDelegateWithFixedCrossAxisCount` 用于固定列数的网格，`SliverGridDelegateWithMaxCrossAxisExtent` 用于根据最大宽度自适应列数的网格。

---

## 第八章：对话框和面板

### 8.1 AlertDialog

`AlertDialog` 是 Material Design 的警告对话框，用于需要用户确认或选择的重要信息。

#### 完整构造函数

```dart
const AlertDialog({
  super.key,
  this.icon,
  this.iconPadding,
  this.iconColor,
  this.title,
  this.titlePadding,
  this.titleTextStyle,
  this.content,
  this.contentPadding,
  this.contentTextStyle,
  this.actions,
  this.actionsPadding,
  this.actionsAlignment,
  this.actionsOverflowAlignment,
  this.actionsOverflowDirection,
  this.actionsOverflowButtonSpacing,
  this.buttonPadding,
  this.backgroundColor,
  this.elevation,
  this.shadowColor,
  this.surfaceTintColor,
  this.shape,
  this.alignment,
  this.scrollable = false,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class AlertDialogDemoPage extends StatelessWidget {
  const AlertDialogDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AlertDialog 演示')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => _showBasicDialog(context),
              child: const Text('基础对话框'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _showConfirmDialog(context),
              child: const Text('确认对话框'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _showCustomDialog(context),
              child: const Text('自定义对话框'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _showListDialog(context),
              child: const Text('列表对话框'),
            ),
          ],
        ),
      ),
    );
  }

  void _showBasicDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('提示'),
        content: const Text('这是一个基础的警告对话框。'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('确定'),
          ),
        ],
      ),
    );
  }

  void _showConfirmDialog(BuildContext context) async {
    final result = await showDialog<bool>(
      context: context,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        title: const Text('确认删除'),
        content: const Text('确定要删除这条记录吗？此操作不可撤销。'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('取消'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            style: TextButton.styleFrom(foregroundColor: Colors.red),
            child: const Text('删除'),
          ),
        ],
      ),
    );
    
    if (result == true) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('已删除')),
      );
    }
  }

  void _showCustomDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        icon: const Icon(Icons.info, color: Colors.blue, size: 48),
        title: const Text('自定义对话框'),
        content: const Text('这是一个带图标的自定义样式对话框。'),
        backgroundColor: Colors.blue[50],
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(20),
        ),
        actionsAlignment: MainAxisAlignment.center,
        actions: [
          ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('知道了'),
          ),
        ],
      ),
    );
  }

  void _showListDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('选择颜色'),
        content: SizedBox(
          width: double.maxFinite,
          child: ListView.builder(
            shrinkWrap: true,
            itemCount: Colors.primaries.length,
            itemBuilder: (context, index) {
              return ListTile(
                leading: CircleAvatar(
                  backgroundColor: Colors.primaries[index],
                ),
                title: Text('颜色 ${index + 1}'),
                onTap: () {
                  Navigator.pop(context, index);
                },
              );
            },
          ),
        ),
      ),
    );
  }
}
```

---

### 8.2 BottomSheet

`BottomSheet` 是从屏幕底部滑出的面板，用于显示补充内容或操作。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class BottomSheetDemoPage extends StatelessWidget {
  const BottomSheetDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('BottomSheet 演示')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => _showModalBottomSheet(context),
              child: const Text('模态底部面板'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _showPersistentBottomSheet(context),
              child: const Text('持久底部面板'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => _showDraggableSheet(context),
              child: const Text('可拖动面板'),
            ),
          ],
        ),
      ),
    );
  }

  void _showModalBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => Container(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Container(
              width: 40,
              height: 4,
              decoration: BoxDecoration(
                color: Colors.grey[300],
                borderRadius: BorderRadius.circular(2),
              ),
            ),
            const SizedBox(height: 20),
            const Text(
              '选择操作',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 20),
            ListTile(
              leading: const Icon(Icons.camera_alt),
              title: const Text('拍照'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.photo_library),
              title: const Text('从相册选择'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.delete, color: Colors.red),
              title: const Text('删除', style: TextStyle(color: Colors.red)),
              onTap: () => Navigator.pop(context),
            ),
            const SizedBox(height: 20),
          ],
        ),
      ),
    );
  }

  void _showPersistentBottomSheet(BuildContext context) {
    Scaffold.of(context).showBottomSheet(
      (context) => Container(
        height: 200,
        color: Colors.amber,
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text('持久底部面板'),
              ElevatedButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('关闭'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _showDraggableSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.5,
        minChildSize: 0.25,
        maxChildSize: 0.9,
        expand: false,
        builder: (context, scrollController) {
          return Container(
            decoration: const BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
            ),
            child: ListView.builder(
              controller: scrollController,
              itemCount: 50,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text('Item ${index + 1}'),
                );
              },
            ),
          );
        },
      ),
    );
  }
}
```

> **补充知识**：`showModalBottomSheet` 显示模态面板（有遮罩，点击外部可关闭），`showBottomSheet` 显示持久面板（无遮罩）。`DraggableScrollableSheet` 可以创建可拖动的底部面板。

---

## 第九章：主题和样式

### 9.1 Theme

`Theme` 用于在 Widget 树中定义和应用主题数据，子 Widget 可以通过 `Theme.of(context)` 获取主题。

#### 完整构造函数

```dart
const Theme({
  super.key,
  required this.data,
  required this.child,
});

ThemeData({
  // 颜色
  Brightness? brightness,
  Color? primaryColor,
  Color? canvasColor,
  Color? cardColor,
  Color? dividerColor,
  Color? focusColor,
  Color? hoverColor,
  Color? highlightColor,
  Color? splashColor,
  InteractiveInkFeatureFactory? splashFactory,
  Color? shadowColor,
  Color? scaffoldBackgroundColor,
  
  // ColorScheme
  ColorScheme? colorScheme,
  
  // 文本主题
  TextTheme? textTheme,
  TextTheme? primaryTextTheme,
  
  // 图标主题
  IconThemeData? iconTheme,
  IconThemeData? primaryIconTheme,
  
  // 组件主题
  AppBarTheme? appBarTheme,
  BottomNavigationBarThemeData? bottomNavigationBarTheme,
  BottomSheetThemeData? bottomSheetTheme,
  ButtonThemeData? buttonTheme,
  CardTheme? cardTheme,
  CheckboxThemeData? checkboxTheme,
  ChipThemeData? chipTheme,
  DataTableThemeData? dataTableTheme,
  DatePickerThemeData? datePickerTheme,
  DialogTheme? dialogTheme,
  DividerThemeData? dividerTheme,
  DrawerThemeData? drawerTheme,
  DropdownMenuThemeData? dropdownMenuTheme,
  ElevatedButtonThemeData? elevatedButtonTheme,
  ExpansionTileThemeData? expansionTileTheme,
  FilledButtonThemeData? filledButtonTheme,
  FloatingActionButtonThemeData? floatingActionButtonTheme,
  IconButtonThemeData? iconButtonTheme,
  InputDecorationTheme? inputDecorationTheme,
  ListTileThemeData? listTileTheme,
  MenuBarThemeData? menuBarTheme,
  MenuButtonThemeData? menuButtonTheme,
  MenuThemeData? menuTheme,
  NavigationBarThemeData? navigationBarTheme,
  NavigationDrawerThemeData? navigationDrawerTheme,
  NavigationRailThemeData? navigationRailTheme,
  OutlinedButtonThemeData? outlinedButtonTheme,
  PopupMenuThemeData? popupMenuTheme,
  ProgressIndicatorThemeData? progressIndicatorTheme,
  RadioThemeData? radioTheme,
  SearchBarThemeData? searchBarTheme,
  SearchViewThemeData? searchViewTheme,
  SegmentedButtonThemeData? segmentedButtonTheme,
  SliderThemeData? sliderTheme,
  SnackBarThemeData? snackBarTheme,
  SwitchThemeData? switchTheme,
  TabBarTheme? tabBarTheme,
  TextButtonThemeData? textButtonTheme,
  TextSelectionThemeData? textSelectionTheme,
  TimePickerThemeData? timePickerTheme,
  ToggleButtonsThemeData? toggleButtonsTheme,
  TooltipThemeData? tooltipTheme,
  
  // 其他
  bool? applyElevationOverlayColor,
  NoDefaultCupertinoThemeData? cupertinoOverrideTheme,
  MaterialTapTargetSize? materialTapTargetSize,
  PageTransitionsTheme? pageTransitionsTheme,
  bool? useMaterial3,
  VisualDensity? visualDensity,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Theme Demo',
      theme: ThemeData(
        // 使用 Material 3
        useMaterial3: true,
        
        // 颜色方案
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.light,
        ),
        
        // 应用栏主题
        appBarTheme: const AppBarTheme(
          centerTitle: true,
          elevation: 0,
        ),
        
        // 卡片主题
        cardTheme: CardTheme(
          elevation: 4,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
        ),
        
        //  elevated 按钮主题
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(8),
            ),
          ),
        ),
        
        // 输入框主题
        inputDecorationTheme: InputDecorationTheme(
          filled: true,
          fillColor: Colors.grey[100],
          border: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide.none,
          ),
          enabledBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: BorderSide(color: Colors.grey[300]!),
          ),
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(12),
            borderSide: const BorderSide(color: Colors.deepPurple, width: 2),
          ),
        ),
        
        // 文本主题
        textTheme: const TextTheme(
          displayLarge: TextStyle(fontSize: 96, fontWeight: FontWeight.w300),
          displayMedium: TextStyle(fontSize: 60, fontWeight: FontWeight.w300),
          displaySmall: TextStyle(fontSize: 48, fontWeight: FontWeight.w400),
          headlineLarge: TextStyle(fontSize: 40, fontWeight: FontWeight.w400),
          headlineMedium: TextStyle(fontSize: 34, fontWeight: FontWeight.w400),
          headlineSmall: TextStyle(fontSize: 24, fontWeight: FontWeight.w400),
          titleLarge: TextStyle(fontSize: 20, fontWeight: FontWeight.w500),
          titleMedium: TextStyle(fontSize: 16, fontWeight: FontWeight.w500),
          titleSmall: TextStyle(fontSize: 14, fontWeight: FontWeight.w500),
          bodyLarge: TextStyle(fontSize: 16, fontWeight: FontWeight.w400),
          bodyMedium: TextStyle(fontSize: 14, fontWeight: FontWeight.w400),
          bodySmall: TextStyle(fontSize: 12, fontWeight: FontWeight.w400),
          labelLarge: TextStyle(fontSize: 14, fontWeight: FontWeight.w500),
          labelMedium: TextStyle(fontSize: 12, fontWeight: FontWeight.w500),
          labelSmall: TextStyle(fontSize: 11, fontWeight: FontWeight.w500),
        ),
        
        // 列表瓦片主题
        listTileTheme: const ListTileThemeData(
          contentPadding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        ),
        
        // SnackBar 主题
        snackBarTheme: SnackBarThemeData(
          behavior: SnackBarBehavior.floating,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(8),
          ),
        ),
      ),
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.dark,
        ),
      ),
      themeMode: ThemeMode.system,
      home: const ThemeDemoPage(),
    );
  }
}

class ThemeDemoPage extends StatelessWidget {
  const ThemeDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final colorScheme = theme.colorScheme;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Theme 演示'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 颜色方案展示
            _buildSectionTitle('ColorScheme'),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                _buildColorChip('Primary', colorScheme.primary),
                _buildColorChip('Secondary', colorScheme.secondary),
                _buildColorChip('Tertiary', colorScheme.tertiary),
                _buildColorChip('Error', colorScheme.error),
                _buildColorChip('Surface', colorScheme.surface),
                _buildColorChip('Background', colorScheme.background),
              ],
            ),
            const SizedBox(height: 24),
            
            // 文本主题展示
            _buildSectionTitle('TextTheme'),
            Text('Display Large', style: theme.textTheme.displayLarge),
            Text('Display Medium', style: theme.textTheme.displayMedium),
            Text('Headline Large', style: theme.textTheme.headlineLarge),
            Text('Headline Medium', style: theme.textTheme.headlineMedium),
            Text('Title Large', style: theme.textTheme.titleLarge),
            Text('Title Medium', style: theme.textTheme.titleMedium),
            Text('Body Large', style: theme.textTheme.bodyLarge),
            Text('Body Medium', style: theme.textTheme.bodyMedium),
            const SizedBox(height: 24),
            
            // 按钮展示
            _buildSectionTitle('Buttons'),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () {},
                  child: const Text('Elevated'),
                ),
                FilledButton(
                  onPressed: () {},
                  child: const Text('Filled'),
                ),
                OutlinedButton(
                  onPressed: () {},
                  child: const Text('Outlined'),
                ),
                TextButton(
                  onPressed: () {},
                  child: const Text('Text'),
                ),
              ],
            ),
            const SizedBox(height: 24),
            
            // 卡片展示
            _buildSectionTitle('Card'),
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      '卡片标题',
                      style: theme.textTheme.titleLarge,
                    ),
                    const SizedBox(height: 8),
                    Text(
                      '这是卡片的内容描述，展示了 Card 组件的样式。',
                      style: theme.textTheme.bodyMedium,
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            
            // 输入框展示
            _buildSectionTitle('TextField'),
            TextField(
              decoration: const InputDecoration(
                labelText: '用户名',
                hintText: '请输入用户名',
                prefixIcon: Icon(Icons.person),
              ),
            ),
            const SizedBox(height: 24),
            
            // 列表瓦片展示
            _buildSectionTitle('ListTile'),
            Card(
              child: Column(
                children: [
                  ListTile(
                    leading: const Icon(Icons.person),
                    title: const Text('个人资料'),
                    subtitle: const Text('查看和编辑个人信息'),
                    trailing: const Icon(Icons.arrow_forward_ios, size: 16),
                    onTap: () {},
                  ),
                  const Divider(height: 1),
                  ListTile(
                    leading: const Icon(Icons.settings),
                    title: const Text('设置'),
                    subtitle: const Text('应用设置和偏好'),
                    trailing: const Icon(Icons.arrow_forward_ios, size: 16),
                    onTap: () {},
                  ),
                ],
              ),
            ),
            const SizedBox(height: 24),
            
            // Chip 展示
            _buildSectionTitle('Chips'),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                Chip(
                  avatar: const Icon(Icons.check_circle, size: 18),
                  label: const Text('选中'),
                ),
                ActionChip(
                  avatar: const Icon(Icons.add, size: 18),
                  label: const Text('添加'),
                  onPressed: () {},
                ),
                FilterChip(
                  label: const Text('筛选'),
                  selected: true,
                  onSelected: (value) {},
                ),
                ChoiceChip(
                  label: const Text('选择'),
                  selected: false,
                  onSelected: (value) {},
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 12),
      child: Text(
        title,
        style: const TextStyle(
          fontSize: 18,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }

  Widget _buildColorChip(String label, Color color) {
    return Chip(
      avatar: Container(
        width: 24,
        height: 24,
        decoration: BoxDecoration(
          color: color,
          shape: BoxShape.circle,
        ),
      ),
      label: Text(label),
    );
  }
}
```

> **补充知识**：从 Flutter 3.0 开始，推荐使用 `ColorScheme.fromSeed()` 创建基于种子颜色的完整配色方案，它会自动生成协调的主色、次色、表面色等。

---

## 第十章：动画组件

### 10.1 AnimatedContainer

`AnimatedContainer` 是带有隐式动画的 Container，当属性变化时会自动播放动画。

#### 完整构造函数

```dart
const AnimatedContainer({
  super.key,
  this.alignment,
  this.padding,
  this.color,
  this.decoration,
  this.foregroundDecoration,
  super.width,
  super.height,
  super.constraints,
  this.margin,
  this.transform,
  this.transformAlignment,
  super.child,
  super.clipBehavior = Clip.none,
  this.curve = Curves.linear,
  required this.duration,
  this.onEnd,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'AnimatedContainer Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const AnimatedContainerDemoPage(),
    );
  }
}

class AnimatedContainerDemoPage extends StatefulWidget {
  const AnimatedContainerDemoPage({super.key});

  @override
  State<AnimatedContainerDemoPage> createState() => _AnimatedContainerDemoPageState();
}

class _AnimatedContainerDemoPageState extends State<AnimatedContainerDemoPage> {
  bool _isExpanded = false;
  Color _color = Colors.blue;
  double _width = 100;
  double _height = 100;
  double _borderRadius = 8;

  void _toggleAnimation() {
    setState(() {
      _isExpanded = !_isExpanded;
      _color = _isExpanded ? Colors.red : Colors.blue;
      _width = _isExpanded ? 200 : 100;
      _height = _isExpanded ? 200 : 100;
      _borderRadius = _isExpanded ? 100 : 8;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedContainer 演示')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 基础动画
            AnimatedContainer(
              width: _width,
              height: _height,
              decoration: BoxDecoration(
                color: _color,
                borderRadius: BorderRadius.circular(_borderRadius),
                boxShadow: [
                  BoxShadow(
                    color: _color.withOpacity(0.5),
                    blurRadius: _isExpanded ? 20 : 5,
                    spreadRadius: _isExpanded ? 5 : 0,
                  ),
                ],
              ),
              duration: const Duration(milliseconds: 500),
              curve: Curves.easeInOut,
              onEnd: () {
                debugPrint('动画结束');
              },
              child: const Center(
                child: Icon(Icons.flutter_dash, color: Colors.white, size: 48),
              ),
            ),
            const SizedBox(height: 40),
            
            // 动画曲线对比
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildCurveDemo('Linear', Curves.linear),
                _buildCurveDemo('Bounce', Curves.bounceOut),
                _buildCurveDemo('Elastic', Curves.elasticOut),
              ],
            ),
            const SizedBox(height: 40),
            
            ElevatedButton(
              onPressed: _toggleAnimation,
              child: Text(_isExpanded ? '收缩' : '展开'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildCurveDemo(String label, Curve curve) {
    return Column(
      children: [
        Text(label),
        const SizedBox(height: 8),
        TweenAnimationBuilder<double>(
          tween: Tween(begin: 0, end: 1),
          duration: const Duration(seconds: 1),
          curve: curve,
          builder: (context, value, child) {
            return Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                color: Colors.primaries[Curves.values.indexOf(curve) % Colors.primaries.length],
                borderRadius: BorderRadius.circular(8),
              ),
              transform: Matrix4.translationValues(0, value * 50, 0),
              child: const Center(
                child: Icon(Icons.animation, color: Colors.white),
              ),
            );
          },
        ),
      ],
    );
  }
}
```

---

### 10.2 Hero

`Hero` 创建共享元素过渡动画，在页面切换时平滑过渡共享的 Widget。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hero Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const HeroDemoPage(),
    );
  }
}

class HeroDemoPage extends StatelessWidget {
  const HeroDemoPage({super.key});

  final List<Map<String, dynamic>> items = const [
    {'id': 1, 'color': Colors.red, 'name': '红色'},
    {'id': 2, 'color': Colors.green, 'name': '绿色'},
    {'id': 3, 'color': Colors.blue, 'name': '蓝色'},
    {'id': 4, 'color': Colors.orange, 'name': '橙色'},
    {'id': 5, 'color': Colors.purple, 'name': '紫色'},
    {'id': 6, 'color': Colors.teal, 'name': '青色'},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Hero 演示')),
      body: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 16,
          crossAxisSpacing: 16,
        ),
        itemCount: items.length,
        itemBuilder: (context, index) {
          final item = items[index];
          return GestureDetector(
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => DetailPage(item: item),
                ),
              );
            },
            child: Hero(
              tag: 'hero-${item['id']}',
              child: Container(
                decoration: BoxDecoration(
                  color: item['color'],
                  borderRadius: BorderRadius.circular(16),
                ),
                child: Center(
                  child: Text(
                    item['name'],
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
              flightShuttleBuilder: (
                flightContext,
                animation,
                flightDirection,
                fromHeroContext,
                toHeroContext,
              ) {
                return AnimatedBuilder(
                  animation: animation,
                  builder: (context, child) {
                    return Container(
                      decoration: BoxDecoration(
                        color: item['color'],
                        borderRadius: BorderRadius.circular(
                          16 + (animation.value * 100),
                        ),
                      ),
                    );
                  },
                );
              },
            ),
          );
        },
      ),
    );
  }
}

class DetailPage extends StatelessWidget {
  final Map<String, dynamic> item;

  const DetailPage({super.key, required this.item});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: item['color'],
      body: SafeArea(
        child: Column(
          children: [
            AppBar(
              backgroundColor: Colors.transparent,
              elevation: 0,
              iconTheme: const IconThemeData(color: Colors.white),
            ),
            Expanded(
              child: Hero(
                tag: 'hero-${item['id']}',
                child: Container(
                  width: double.infinity,
                  color: item['color'],
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        item['name'],
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 48,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 20),
                      const Text(
                        'Hero 动画演示',
                        style: TextStyle(
                          color: Colors.white70,
                          fontSize: 18,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

> **补充知识**：`Hero` 动画要求两个页面的 Hero Widget 具有相同的 `tag`。Flutter 会自动计算过渡动画，你也可以通过 `flightShuttleBuilder` 自定义过渡效果。

---

## 第十一章：其他重要组件

### 11.1 Card

`Card` 是 Material Design 的卡片组件，用于显示相关内容的容器。

#### 完整构造函数

```dart
const Card({
  super.key,
  this.color,
  this.shadowColor,
  this.surfaceTintColor,
  this.elevation,
  this.shape,
  this.borderOnForeground = true,
  this.margin,
  this.clipBehavior,
  this.child,
  this.semanticContainer = true,
});
```

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class CardDemoPage extends StatelessWidget {
  const CardDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Card 演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 基础卡片
            Card(
              child: ListTile(
                leading: const Icon(Icons.album),
                title: const Text('卡片标题'),
                subtitle: const Text('这是卡片的副标题内容'),
              ),
            ),
            const SizedBox(height: 16),
            
            // 带图片的卡片
            Card(
              clipBehavior: Clip.antiAlias,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
              ),
              child: Column(
                children: [
                  Image.network(
                    'https://picsum.photos/400/200',
                    fit: BoxFit.cover,
                  ),
                  const Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          '卡片标题',
                          style: TextStyle(
                            fontSize: 20,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        SizedBox(height: 8),
                        Text('这是卡片的详细描述内容。'),
                      ],
                    ),
                  ),
                  ButtonBar(
                    children: [
                      TextButton(
                        onPressed: () {},
                        child: const Text('分享'),
                      ),
                      TextButton(
                        onPressed: () {},
                        child: const Text('了解更多'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
            const SizedBox(height: 16),
            
            // 带阴影的卡片
            Card(
              elevation: 8,
              shadowColor: Colors.blue.withOpacity(0.5),
              child: const Padding(
                padding: EdgeInsets.all(16),
                child: Text('带阴影的卡片'),
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

### 11.2 Chip

`Chip` 是 Material Design 的碎片组件，用于表示属性、标签或操作。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class ChipDemoPage extends StatefulWidget {
  const ChipDemoPage({super.key});

  @override
  State<ChipDemoPage> createState() => _ChipDemoPageState();
}

class _ChipDemoPageState extends State<ChipDemoPage> {
  final List<String> _filters = ['Flutter', 'Dart'];
  String _selectedChoice = '全部';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chip 演示')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 基础 Chip
            const Text('基础 Chip', style: TextStyle(fontWeight: FontWeight.bold)),
            const Wrap(
              spacing: 8,
              children: [
                Chip(label: Text('标签1')),
                Chip(label: Text('标签2')),
                Chip(label: Text('标签3')),
              ],
            ),
            const SizedBox(height: 24),
            
            // 带图标的 Chip
            const Text('带图标的 Chip', style: TextStyle(fontWeight: FontWeight.bold)),
            const Wrap(
              spacing: 8,
              children: [
                Chip(
                  avatar: Icon(Icons.person),
                  label: Text('用户'),
                ),
                Chip(
                  avatar: CircleAvatar(
                    backgroundColor: Colors.blue,
                    child: Text('A'),
                  ),
                  label: Text('Alice'),
                ),
              ],
            ),
            const SizedBox(height: 24),
            
            // 可删除的 Chip
            const Text('可删除的 Chip', style: TextStyle(fontWeight: FontWeight.bold)),
            Wrap(
              spacing: 8,
              children: _filters.map((filter) {
                return Chip(
                  label: Text(filter),
                  onDeleted: () {
                    setState(() {
                      _filters.remove(filter);
                    });
                  },
                );
              }).toList(),
            ),
            const SizedBox(height: 24),
            
            // ActionChip
            const Text('ActionChip', style: TextStyle(fontWeight: FontWeight.bold)),
            Wrap(
              spacing: 8,
              children: [
                ActionChip(
                  avatar: const Icon(Icons.add),
                  label: const Text('添加标签'),
                  onPressed: () {
                    setState(() {
                      _filters.add('新标签 ${_filters.length + 1}');
                    });
                  },
                ),
              ],
            ),
            const SizedBox(height: 24),
            
            // FilterChip
            const Text('FilterChip', style: TextStyle(fontWeight: FontWeight.bold)),
            Wrap(
              spacing: 8,
              children: ['全部', '未完成', '已完成', '已取消'].map((filter) {
                return FilterChip(
                  label: Text(filter),
                  selected: _selectedChoice == filter,
                  onSelected: (selected) {
                    setState(() {
                      _selectedChoice = filter;
                    });
                  },
                );
              }).toList(),
            ),
            const SizedBox(height: 24),
            
            // ChoiceChip
            const Text('ChoiceChip', style: TextStyle(fontWeight: FontWeight.bold)),
            Wrap(
              spacing: 8,
              children: ['小', '中', '大'].map((size) {
                return ChoiceChip(
                  label: Text(size),
                  selected: _selectedChoice == size,
                  onSelected: (selected) {
                    setState(() {
                      _selectedChoice = size;
                    });
                  },
                );
              }).toList(),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 11.3 SnackBar

`SnackBar` 用于显示屏幕底部的简短消息，通常用于操作反馈。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class SnackBarDemoPage extends StatelessWidget {
  const SnackBarDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('SnackBar 演示')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('这是一个简单的 SnackBar')),
                );
              },
              child: const Text('显示 SnackBar'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: const Text('已删除'),
                    action: SnackBarAction(
                      label: '撤销',
                      onPressed: () {
                        ScaffoldMessenger.of(context).showSnackBar(
                          const SnackBar(content: Text('已撤销删除')),
                        );
                      },
                    ),
                  ),
                );
              },
              child: const Text('带操作的 SnackBar'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: const Text('自定义样式'),
                    backgroundColor: Colors.green,
                    behavior: SnackBarBehavior.floating,
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(10),
                    ),
                    duration: const Duration(seconds: 5),
                  ),
                );
              },
              child: const Text('自定义样式 SnackBar'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 11.4 ProgressIndicator

`ProgressIndicator` 用于显示操作进度，包括线性和圆形两种形式。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class ProgressIndicatorDemoPage extends StatefulWidget {
  const ProgressIndicatorDemoPage({super.key});

  @override
  State<ProgressIndicatorDemoPage> createState() => _ProgressIndicatorDemoPageState();
}

class _ProgressIndicatorDemoPageState extends State<ProgressIndicatorDemoPage>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  double _progress = 0;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ProgressIndicator 演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 圆形进度指示器
            const Text('CircularProgressIndicator', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                const CircularProgressIndicator(),
                CircularProgressIndicator(
                  value: _progress,
                  backgroundColor: Colors.grey[300],
                  valueColor: const AlwaysStoppedAnimation<Color>(Colors.blue),
                ),
                RotationTransition(
                  turns: _controller,
                  child: const CircularProgressIndicator(
                    strokeWidth: 8,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 32),
            
            // 线性进度指示器
            const Text('LinearProgressIndicator', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            const LinearProgressIndicator(),
            const SizedBox(height: 16),
            LinearProgressIndicator(
              value: _progress,
              backgroundColor: Colors.grey[300],
              valueColor: const AlwaysStoppedAnimation<Color>(Colors.green),
              minHeight: 10,
              borderRadius: BorderRadius.circular(5),
            ),
            const SizedBox(height: 32),
            
            // 控制按钮
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = 0;
                    });
                    _simulateProgress();
                  },
                  child: const Text('开始进度'),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      _progress = 0;
                    });
                  },
                  child: const Text('重置'),
                ),
              ],
            ),
            const SizedBox(height: 32),
            
            // 刷新指示器
            const Text('RefreshIndicator', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            SizedBox(
              height: 200,
              child: RefreshIndicator(
                onRefresh: () async {
                  await Future.delayed(const Duration(seconds: 2));
                },
                child: ListView.builder(
                  itemCount: 5,
                  itemBuilder: (context, index) {
                    return ListTile(title: Text('Item ${index + 1}'));
                  },
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _simulateProgress() async {
    for (int i = 0; i <= 100; i++) {
      await Future.delayed(const Duration(milliseconds: 50));
      if (mounted) {
        setState(() {
          _progress = i / 100;
        });
      }
    }
  }
}
```

---

### 11.5 TabBar & TabBarView

`TabBar` 和 `TabBarView` 用于创建标签页界面。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TabBar Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const TabBarDemoPage(),
    );
  }
}

class TabBarDemoPage extends StatelessWidget {
  const TabBarDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('TabBar 演示'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.home), text: '首页'),
              Tab(icon: Icon(Icons.favorite), text: '收藏'),
              Tab(icon: Icon(Icons.settings), text: '设置'),
            ],
            indicatorColor: Colors.white,
            labelColor: Colors.white,
            unselectedLabelColor: Colors.white70,
          ),
        ),
        body: const TabBarView(
          children: [
            Center(child: Text('首页内容', style: TextStyle(fontSize: 24))),
            Center(child: Text('收藏内容', style: TextStyle(fontSize: 24))),
            Center(child: Text('设置内容', style: TextStyle(fontSize: 24))),
          ],
        ),
      ),
    );
  }
}
```

---

### 11.6 BottomNavigationBar

`BottomNavigationBar` 用于显示底部导航栏。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class BottomNavDemoPage extends StatefulWidget {
  const BottomNavDemoPage({super.key});

  @override
  State<BottomNavDemoPage> createState() => _BottomNavDemoPageState();
}

class _BottomNavDemoPageState extends State<BottomNavDemoPage> {
  int _selectedIndex = 0;

  final List<Widget> _pages = const [
    Center(child: Text('首页', style: TextStyle(fontSize: 24))),
    Center(child: Text('发现', style: TextStyle(fontSize: 24))),
    Center(child: Text('消息', style: TextStyle(fontSize: 24))),
    Center(child: Text('我的', style: TextStyle(fontSize: 24))),
  ];

  void _onItemTapped(int index) {
    setState(() {
      _selectedIndex = index;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('BottomNavigationBar 演示')),
      body: _pages[_selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        items: const <BottomNavigationBarItem>[
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: '首页',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.explore),
            label: '发现',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.message),
            label: '消息',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: '我的',
          ),
        ],
        currentIndex: _selectedIndex,
        selectedItemColor: Colors.blue,
        unselectedItemColor: Colors.grey,
        showUnselectedLabels: true,
        type: BottomNavigationBarType.fixed,
        onTap: _onItemTapped,
      ),
    );
  }
}
```

---

### 11.7 Drawer

`Drawer` 用于创建侧滑抽屉菜单。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

class DrawerDemoPage extends StatelessWidget {
  const DrawerDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Drawer 演示')),
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            DrawerHeader(
              decoration: BoxDecoration(
                color: Theme.of(context).colorScheme.primary,
              ),
              child: const Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  CircleAvatar(
                    radius: 40,
                    backgroundColor: Colors.white,
                    child: Icon(Icons.person, size: 40),
                  ),
                  SizedBox(height: 16),
                  Text(
                    '用户名',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  Text(
                    'user@example.com',
                    style: TextStyle(color: Colors.white70),
                  ),
                ],
              ),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('首页'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('设置'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
            const Divider(),
            ListTile(
              leading: const Icon(Icons.logout),
              title: const Text('退出登录'),
              onTap: () {
                Navigator.pop(context);
              },
            ),
          ],
        ),
      ),
      body: const Center(
        child: Text('点击左上角菜单图标打开抽屉'),
      ),
    );
  }
}
```

---

## 第十二章：表单组件

### 12.1 Form & TextFormField

`Form` 用于管理表单状态，`TextFormField` 是带验证功能的文本输入框。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Form Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const FormDemoPage(),
    );
  }
}

class FormDemoPage extends StatefulWidget {
  const FormDemoPage({super.key});

  @override
  State<FormDemoPage> createState() => _FormDemoPageState();
}

class _FormDemoPageState extends State<FormDemoPage> {
  final _formKey = GlobalKey<FormState>();
  final _nameController = TextEditingController();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Form 演示')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          autovalidateMode: AutovalidateMode.onUserInteraction,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // 用户名
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: '用户名',
                  hintText: '请输入用户名',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入用户名';
                  }
                  if (value.length < 3) {
                    return '用户名至少3个字符';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 邮箱
              TextFormField(
                controller: _emailController,
                keyboardType: TextInputType.emailAddress,
                decoration: const InputDecoration(
                  labelText: '邮箱',
                  hintText: 'example@email.com',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入邮箱';
                  }
                  final emailRegex = RegExp(r'^[^@]+@[^@]+\.[^@]+');
                  if (!emailRegex.hasMatch(value)) {
                    return '请输入有效的邮箱地址';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              
              // 密码
              TextFormField(
                controller: _passwordController,
                obscureText: _obscurePassword,
                decoration: InputDecoration(
                  labelText: '密码',
                  hintText: '请输入密码',
                  prefixIcon: const Icon(Icons.lock),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _obscurePassword ? Icons.visibility_off : Icons.visibility,
                    ),
                    onPressed: () {
                      setState(() {
                        _obscurePassword = !_obscurePassword;
                      });
                    },
                  ),
                  border: const OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return '请输入密码';
                  }
                  if (value.length < 6) {
                    return '密码至少6个字符';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              
              // 提交按钮
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: () {
                    if (_formKey.currentState!.validate()) {
                      // 表单验证通过
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(
                          content: Text(
                            '注册成功！\n用户名: ${_nameController.text}\n邮箱: ${_emailController.text}',
                          ),
                        ),
                      );
                    }
                  },
                  child: const Text('注册'),
                ),
              ),
              const SizedBox(height: 16),
              
              // 重置按钮
              SizedBox(
                width: double.infinity,
                child: OutlinedButton(
                  onPressed: () {
                    _formKey.currentState!.reset();
                    _nameController.clear();
                    _emailController.clear();
                    _passwordController.clear();
                  },
                  child: const Text('重置'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

> **补充知识**：`Form` 使用 `GlobalKey<FormState>` 来管理表单状态。`validate()` 方法会触发表单中所有 `TextFormField` 的验证器，返回 `true` 表示所有验证通过。

---

## 第十三章：导航组件

### 13.1 Navigator

`Navigator` 是 Flutter 的路由管理器，用于页面之间的导航。

#### 核心方法

| 方法 | 说明 |
|------|------|
| `push()` | 导航到新页面 |
| `pop()` | 返回上一页 |
| `pushReplacement()` | 替换当前页面 |
| `pushAndRemoveUntil()` | 导航到新页面并移除之前的页面 |
| `popUntil()` | 返回到指定页面 |
| `canPop()` | 检查是否可以返回 |

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigator Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const NavigatorDemoPage(),
    );
  }
}

class NavigatorDemoPage extends StatelessWidget {
  const NavigatorDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Navigator 演示')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const SecondPage(),
                  ),
                );
              },
              child: const Text('Push 到新页面'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () async {
                final result = await Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ReturnValuePage(),
                  ),
                );
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('返回值: $result')),
                );
              },
              child: const Text('带返回值导航'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ReplacementPage(),
                  ),
                );
              },
              child: const Text('替换当前页面'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const NewRootPage(),
                  ),
                  (route) => false, // 移除所有之前的页面
                );
              },
              child: const Text('导航并清除历史'),
            ),
          ],
        ),
      ),
    );
  }
}

class SecondPage extends StatelessWidget {
  const SecondPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('第二页')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('返回'),
        ),
      ),
    );
  }
}

class ReturnValuePage extends StatelessWidget {
  const ReturnValuePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('返回值页面')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, '成功');
              },
              child: const Text('返回成功'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, '失败');
              },
              child: const Text('返回失败'),
            ),
          ],
        ),
      ),
    );
  }
}

class ReplacementPage extends StatelessWidget {
  const ReplacementPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('替换页面'),
        automaticallyImplyLeading: false,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('这是替换后的页面，无法返回'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const NavigatorDemoPage(),
                  ),
                  (route) => false,
                );
              },
              child: const Text('回到首页'),
            ),
          ],
        ),
      ),
    );
  }
}

class NewRootPage extends StatelessWidget {
  const NewRootPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('新根页面'),
        automaticallyImplyLeading: false,
      ),
      body: const Center(
        child: Text('这是一个新的根页面，没有返回按钮'),
      ),
    );
  }
}
```

---

### 13.2 Route

`Route` 是 Flutter 中页面的抽象表示，可以自定义页面过渡动画。

#### 完整示例代码

```dart
import 'package:flutter/material.dart';

// 自定义路由动画
class FadeRoute extends PageRouteBuilder {
  final Widget page;

  FadeRoute({required this.page})
      : super(
          pageBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
          ) =>
              page,
          transitionsBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) =>
              FadeTransition(
            opacity: animation,
            child: child,
          ),
        );
}

class SlideRoute extends PageRouteBuilder {
  final Widget page;

  SlideRoute({required this.page})
      : super(
          pageBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
          ) =>
              page,
          transitionsBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) {
            const begin = Offset(1.0, 0.0);
            const end = Offset.zero;
            const curve = Curves.easeInOut;

            var tween = Tween(begin: begin, end: end).chain(
              CurveTween(curve: curve),
            );

            return SlideTransition(
              position: animation.drive(tween),
              child: child,
            );
          },
        );
}

class ScaleRoute extends PageRouteBuilder {
  final Widget page;

  ScaleRoute({required this.page})
      : super(
          pageBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
          ) =>
              page,
          transitionsBuilder: (
            BuildContext context,
            Animation<double> animation,
            Animation<double> secondaryAnimation,
            Widget child,
          ) =>
              ScaleTransition(
            scale: Tween<double>(
              begin: 0.0,
              end: 1.0,
            ).animate(
              CurvedAnimation(
                parent: animation,
                curve: Curves.fastOutSlowIn,
              ),
            ),
            child: child,
          ),
        );
}

// 使用自定义路由
class CustomRouteDemoPage extends StatelessWidget {
  const CustomRouteDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('自定义路由动画')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  FadeRoute(page: const AnimationTargetPage('淡入')),
                );
              },
              child: const Text('淡入动画'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  SlideRoute(page: const AnimationTargetPage('滑动')),
                );
              },
              child: const Text('滑动动画'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  ScaleRoute(page: const AnimationTargetPage('缩放')),
                );
              },
              child: const Text('缩放动画'),
            ),
          ],
        ),
      ),
    );
  }
}

class AnimationTargetPage extends StatelessWidget {
  final String animationType;

  const AnimationTargetPage(this.animationType, {super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('$animationType 动画页面')),
      body: Center(
        child: Text(
          '这是 $animationType 动画效果',
          style: const TextStyle(fontSize: 24),
        ),
      ),
    );
  }
}
```

---

## 第十四章：工具类和常量

### 14.1 常用常量

```dart
// 文件: flutter/lib/src/material/constants.dart

// 标准 Material 设计尺寸
const double kToolbarHeight = 56.0;

// 应用栏扩展高度
const double kExpandedAppBarHeight = 128.0;

// 标准边距
const EdgeInsets kMaterialEdgePadding = EdgeInsets.all(16.0);

// 标准列表项高度
const double kMinInteractiveDimension = 48.0;
const double kMinInteractiveDimensionCupertino = 44.0;

// 标准图标尺寸
const double kDefaultIconSize = 24.0;

// 标准按钮高度
const double kDefaultButtonHeight = 36.0;

// 标准边框圆角
const BorderRadius kDefaultBorderRadius = BorderRadius.all(Radius.circular(4.0));
```

### 14.2 常用工具类

#### Colors

```dart
// Material 颜色
Colors.red        // 红色
Colors.pink       // 粉色
Colors.purple     // 紫色
Colors.deepPurple // 深紫色
Colors.indigo     // 靛蓝色
Colors.blue       // 蓝色
Colors.lightBlue  // 浅蓝色
Colors.cyan       // 青色
Colors.teal       // 蓝绿色
Colors.green      // 绿色
Colors.lightGreen // 浅绿色
Colors.lime       // 柠檬色
Colors.yellow     // 黄色
Colors.amber      // 琥珀色
Colors.orange     // 橙色
Colors.deepOrange // 深橙色
Colors.brown      // 棕色
Colors.grey       // 灰色
Colors.blueGrey   // 蓝灰色

// 预定义颜色
Colors.black      // 黑色
Colors.white      // 白色
Colors.transparent // 透明

// 带透明度的黑色
Colors.black12    // 12% 透明度
Colors.black26    // 26% 透明度
Colors.black38    // 38% 透明度
Colors.black45    // 45% 透明度
Colors.black54    // 54% 透明度
Colors.black87    // 87% 透明度

// 带透明度的白色
Colors.white10    // 10% 透明度
Colors.white12    // 12% 透明度
Colors.white24    // 24% 透明度
Colors.white30    // 30% 透明度
Colors.white54    // 54% 透明度
Colors.white70    // 70% 透明度
```

#### Icons

```dart
// 常用图标
Icons.add           // 添加
Icons.remove        // 移除
Icons.close         // 关闭
Icons.check         // 勾选
Icons.clear         // 清除
Icons.edit          // 编辑
Icons.delete        // 删除
Icons.search        // 搜索
Icons.home          // 首页
Icons.settings      // 设置
Icons.person        // 个人
Icons.favorite      // 收藏
Icons.star          // 星星
Icons.menu          // 菜单
Icons.arrow_back    // 返回
Icons.arrow_forward // 前进
Icons.more_vert     // 更多（垂直）
Icons.more_horiz    // 更多（水平）
Icons.notifications // 通知
Icons.share         // 分享
Icons.refresh       // 刷新
Icons.camera        // 相机
Icons.image         // 图片
Icons.location_on   // 位置
Icons.phone         // 电话
Icons.email         // 邮件
Icons.lock          // 锁定
Icons.visibility    // 可见
Icons.visibility_off // 不可见
Icons.brightness_4  // 亮度
Icons.dark_mode     // 暗色模式
Icons.light_mode    // 亮色模式
```

#### EdgeInsets

```dart
// 常用边距
EdgeInsets.zero                          // 零边距
EdgeInsets.all(16)                       // 四边相同
EdgeInsets.symmetric(horizontal: 16)     // 水平方向
EdgeInsets.symmetric(vertical: 8)        // 垂直方向
EdgeInsets.only(left: 16)                // 仅左边
EdgeInsets.only(right: 16)               // 仅右边
EdgeInsets.only(top: 8)                  // 仅顶部
EdgeInsets.only(bottom: 8)               // 仅底部
EdgeInsets.only(left: 16, right: 16)     // 左右
EdgeInsets.only(top: 8, bottom: 8)       // 上下
EdgeInsets.fromLTRB(16, 8, 16, 8)        // 左上右下

// 预定义边距
const EdgeInsets kMaterialEdgePadding = EdgeInsets.all(16.0);
```

#### BorderRadius

```dart
// 常用圆角
BorderRadius.zero                              // 无圆角
BorderRadius.all(Radius.circular(8))           // 四边相同
BorderRadius.circular(8)                       // 快捷方式
BorderRadius.only(topLeft: Radius.circular(8)) // 仅左上
BorderRadius.horizontal(left: Radius.circular(8)) // 仅左侧
BorderRadius.vertical(top: Radius.circular(8))    // 仅顶部

// 特殊圆角
BorderRadius.circular(999)                     // 胶囊形
```

#### BoxShadow

```dart
// 常用阴影
BoxShadow(
  color: Colors.black.withOpacity(0.2),
  blurRadius: 10,
  offset: const Offset(0, 4),
)

// 轻微阴影
BoxShadow(
  color: Colors.black.withOpacity(0.1),
  blurRadius: 4,
  offset: const Offset(0, 2),
)

// 强烈阴影
BoxShadow(
  color: Colors.black.withOpacity(0.3),
  blurRadius: 20,
  spreadRadius: 5,
  offset: const Offset(0, 8),
)
```

#### LinearGradient

```dart
// 常用渐变
LinearGradient(
  colors: [Colors.blue, Colors.purple],
  begin: Alignment.topLeft,
  end: Alignment.bottomRight,
)

// 多色渐变
LinearGradient(
  colors: [Colors.red, Colors.orange, Colors.yellow],
  stops: [0.0, 0.5, 1.0],
)

// 垂直渐变
LinearGradient(
  colors: [Colors.transparent, Colors.black],
  begin: Alignment.topCenter,
  end: Alignment.bottomCenter,
)
```

### 14.3 常用曲线

```dart
// 线性
Curves.linear

// 加速
Curves.easeIn
Curves.easeInQuad
Curves.easeInCubic
Curves.easeInQuart
Curves.easeInExpo

// 减速
Curves.easeOut
Curves.easeOutQuad
Curves.easeOutCubic
Curves.easeOutQuart
Curves.easeOutExpo

// 先加速后减速
Curves.easeInOut
Curves.easeInOutQuad
Curves.easeInOutCubic
Curves.easeInOutQuart
Curves.easeInOutExpo

// 弹性
Curves.elasticIn
Curves.elasticOut
Curves.elasticInOut

// 弹跳
Curves.bounceIn
Curves.bounceOut
Curves.bounceInOut

// 快速开始
Curves.fastOutSlowIn

// 慢速开始和结束
Curves.slowMiddle

// 过冲
Curves.easeOutBack

// 其他
Curves.decelerate
Curves.fastLinearToSlowEaseIn
```

---

## 附录 A：Material 组件速查表

### A.1 基础组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `Container` | 通用容器 | `color`, `padding`, `margin`, `decoration` |
| `Row` | 水平布局 | `mainAxisAlignment`, `crossAxisAlignment` |
| `Column` | 垂直布局 | `mainAxisAlignment`, `crossAxisAlignment` |
| `Stack` | 堆叠布局 | `alignment`, `fit` |
| `Expanded` | 弹性填充 | `flex` |
| `Flexible` | 弹性布局 | `flex`, `fit` |
| `Padding` | 内边距 | `padding` |
| `Center` | 居中对齐 | `child` |
| `Align` | 对齐 | `alignment` |
| `SizedBox` | 固定尺寸 | `width`, `height` |

### A.2 文本组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `Text` | 显示文本 | `style`, `textAlign`, `overflow`, `maxLines` |
| `RichText` | 富文本 | `text` |
| `DefaultTextStyle` | 默认文本样式 | `style`, `child` |

### A.3 按钮组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `ElevatedButton` | 凸起按钮 | `onPressed`, `style`, `child` |
| `TextButton` | 文本按钮 | `onPressed`, `style`, `child` |
| `OutlinedButton` | 边框按钮 | `onPressed`, `style`, `child` |
| `IconButton` | 图标按钮 | `onPressed`, `icon`, `color` |
| `FloatingActionButton` | 悬浮按钮 | `onPressed`, `child` |

### A.4 输入组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `TextField` | 文本输入 | `controller`, `decoration`, `keyboardType` |
| `TextFormField` | 表单输入 | `validator`, `onSaved` |
| `Form` | 表单容器 | `key`, `child`, `autovalidateMode` |

### A.5 列表组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `ListView` | 滚动列表 | `children`, `scrollDirection` |
| `ListView.builder` | 动态列表 | `itemCount`, `itemBuilder` |
| `ListView.separated` | 带分隔线列表 | `separatorBuilder` |
| `GridView` | 网格列表 | `gridDelegate`, `children` |
| `GridView.count` | 固定列数网格 | `crossAxisCount` |
| `GridView.builder` | 动态网格 | `gridDelegate`, `itemBuilder` |

### A.6 导航组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `MaterialApp` | 应用根组件 | `home`, `routes`, `theme` |
| `Scaffold` | 页面脚手架 | `appBar`, `body`, `floatingActionButton` |
| `AppBar` | 应用栏 | `title`, `actions`, `leading` |
| `BottomNavigationBar` | 底部导航 | `items`, `currentIndex`, `onTap` |
| `Drawer` | 侧滑菜单 | `child` |
| `TabBar` | 标签栏 | `tabs`, `controller` |
| `TabBarView` | 标签内容 | `children` |

### A.7 反馈组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `SnackBar` | 底部提示 | `content`, `action`, `duration` |
| `AlertDialog` | 警告对话框 | `title`, `content`, `actions` |
| `SimpleDialog` | 简单对话框 | `title`, `children` |
| `BottomSheet` | 底部面板 | - |
| `CircularProgressIndicator` | 圆形进度 | `value`, `color` |
| `LinearProgressIndicator` | 线性进度 | `value`, `backgroundColor` |

### A.8 展示组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `Card` | 卡片 | `child`, `elevation`, `shape` |
| `Chip` | 碎片 | `label`, `avatar`, `onDeleted` |
| `Badge` | 徽章 | `child`, `label` |
| `Divider` | 分隔线 | `height`, `color` |
| `CircleAvatar` | 圆形头像 | `backgroundColor`, `child` |
| `Image` | 图片 | `image`, `fit` |
| `Icon` | 图标 | `icon`, `color`, `size` |

### A.9 动画组件

| 组件 | 用途 | 常用属性 |
|------|------|----------|
| `AnimatedContainer` | 动画容器 | `duration`, `curve` |
| `AnimatedOpacity` | 动画透明度 | `opacity`, `duration` |
| `AnimatedPositioned` | 动画定位 | `duration`, `position` |
| `AnimatedBuilder` | 动画构建器 | `animation`, `builder` |
| `Hero` | 共享元素过渡 | `tag`, `child` |
| `FadeTransition` | 淡入淡出 | `opacity`, `child` |
| `SlideTransition` | 滑动过渡 | `position`, `child` |
| `ScaleTransition` | 缩放过渡 | `scale`, `child` |

---

## 附录 B：Material 设计规范参考

### B.1 颜色系统

```dart
// Material 3 颜色角色
ColorScheme(
  primary: ...,        // 主要颜色，用于主要操作
  onPrimary: ...,      // 在 primary 上的文字/图标颜色
  primaryContainer: ..., // 主要容器颜色
  onPrimaryContainer: ..., // 在 primaryContainer 上的颜色
  
  secondary: ...,      // 次要颜色
  onSecondary: ...,
  secondaryContainer: ...,
  onSecondaryContainer: ...,
  
  tertiary: ...,       // 第三颜色
  onTertiary: ...,
  tertiaryContainer: ...,
  onTertiaryContainer: ...,
  
  error: ...,          // 错误颜色
  onError: ...,
  errorContainer: ...,
  onErrorContainer: ...,
  
  surface: ...,        // 表面颜色
  onSurface: ...,
  surfaceVariant: ..., // 表面变体
  onSurfaceVariant: ...,
  
  background: ...,     // 背景颜色
  onBackground: ...,
  
  outline: ...,        // 轮廓颜色
  outlineVariant: ..., // 轮廓变体
  
  shadow: ...,         // 阴影颜色
  scrim: ...,          // 遮罩颜色
  
  inverseSurface: ..., // 反色表面
  onInverseSurface: ...,
  inversePrimary: ..., // 反色主要
)
```

### B.2 类型比例

| 样式 | 尺寸 | 字重 | 用途 |
|------|------|------|------|
| Display Large | 57 | Regular | 最大的展示文本 |
| Display Medium | 45 | Regular | 大标题 |
| Display Small | 36 | Regular | 中等标题 |
| Headline Large | 32 | Regular | 页面标题 |
| Headline Medium | 28 | Regular | 区块标题 |
| Headline Small | 24 | Regular | 小标题 |
| Title Large | 22 | Medium | 卡片标题 |
| Title Medium | 16 | Medium | 列表标题 |
| Title Small | 14 | Medium | 小标题 |
| Body Large | 16 | Regular | 正文 |
| Body Medium | 14 | Regular | 默认正文 |
| Body Small | 12 | Regular | 辅助文本 |
| Label Large | 14 | Medium | 按钮文字 |
| Label Medium | 12 | Medium | 小标签 |
| Label Small | 11 | Medium | 最小标签 |

### B.3 间距系统

| 名称 | 值 | 用途 |
|------|-----|------|
| `space_0` | 0 | 无间距 |
| `space_1` | 4 | 最小间距 |
| `space_2` | 8 | 小间距 |
| `space_3` | 12 | 中间距 |
| `space_4` | 16 | 标准间距 |
| `space_5` | 20 | 大间距 |
| `space_6` | 24 | 更大间距 |
| `space_8` | 32 | 超大间距 |
| `space_10` | 40 | 最大间距 |
| `space_12` | 48 | 页面边距 |
| `space_16` | 64 | 区块间距 |
| `space_20` | 80 | 大区块间距 |
| `space_24` | 96 | 超大区块间距 |

### B.4 圆角系统

| 名称 | 值 | 用途 |
|------|-----|------|
| `radius_0` | 0 | 无圆角 |
| `radius_1` | 4 | 小圆角 |
| `radius_2` | 8 | 标准圆角 |
| `radius_3` | 12 | 中等圆角 |
| `radius_4` | 16 | 大圆角 |
| `radius_5` | 20 | 更大圆角 |
| `radius_6` | 24 | 超大圆角 |
| `radius_7` | 28 | 卡片圆角 |
| `radius_full` | 999 | 胶囊/圆形 |

---

## 附录 C：常见问题解答

### C.1 布局问题

**Q: Row/Column 子元素溢出怎么办？**

A: 可以使用以下方法解决：
1. 使用 `Expanded` 或 `Flexible` 包装子元素
2. 使用 `SingleChildScrollView` 包裹
3. 使用 `ListView` 替代
4. 使用 `Wrap` 替代 `Row`

```dart
// 解决方案1：使用 Expanded
Row(
  children: [
    Expanded(child: Text('长文本...')),
  ],
)

// 解决方案2：使用 ScrollView
SingleChildScrollView(
  scrollDirection: Axis.horizontal,
  child: Row(...),
)

// 解决方案3：使用 Wrap
Wrap(
  children: [...],
)
```

**Q: 如何让 Container 填满父元素？**

A: 使用 `double.infinity` 或 `Expanded`：

```dart
Container(
  width: double.infinity,
  height: double.infinity,
  child: ...,
)

// 或
Expanded(
  child: Container(...),
)
```

### C.2 样式问题

**Q: 如何设置全局主题？**

A: 在 `MaterialApp` 中设置 `theme`：

```dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
    useMaterial3: true,
  ),
)
```

**Q: 如何覆盖局部主题？**

A: 使用 `Theme` Widget：

```dart
Theme(
  data: Theme.of(context).copyWith(
    primaryColor: Colors.red,
  ),
  child: ...,
)
```

### C.3 状态管理问题

**Q: 如何管理表单状态？**

A: 使用 `Form` 和 `GlobalKey<FormState>`：

```dart
final _formKey = GlobalKey<FormState>();

Form(
  key: _formKey,
  child: ...,
)

// 验证表单
if (_formKey.currentState!.validate()) {
  // 验证通过
}
```

### C.4 性能问题

**Q: 如何优化列表性能？**

A: 
1. 使用 `ListView.builder` 而不是 `ListView`
2. 使用 `const` 构造函数
3. 使用 `RepaintBoundary` 隔离重绘
4. 使用 `AutomaticKeepAliveClientMixin` 保持状态

```dart
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return const ListTile(
      title: Text('Item'),
    );
  },
)
```

**Q: 如何减少不必要的重建？**

A:
1. 使用 `const` 构造函数
2. 使用 `StatefulWidget` 的 `shouldRebuild` 方法
3. 使用 `ValueNotifier` 和 `ValueListenableBuilder`
4. 使用状态管理库（如 Riverpod、Provider）

---

## 附录 D：版本更新说明

### Flutter 3.41 Material 更新

| 特性 | 说明 |
|------|------|
| Material 3 默认启用 | 新创建的项目默认使用 Material 3 |
| 更新的 ColorScheme | 支持动态颜色（从壁纸提取） |
| 新的组件主题 | 更多组件支持主题定制 |
| 改进的动画 | 更流畅的过渡动画 |
| 可访问性改进 | 更好的屏幕阅读器支持 |

### 迁移指南

从 Material 2 迁移到 Material 3：

```dart
// 启用 Material 3
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
  ),
)

// 更新颜色引用
// Material 2: Theme.of(context).primaryColor
// Material 3: Theme.of(context).colorScheme.primary
```

---

**文档版本**: 1.0  
**最后更新**: 2024年  
**Flutter 版本**: 3.41.0

---

> **补充知识**：`flutter/material.dart` 是 Flutter 最核心、最常用的库。掌握这个库中的组件和 API，就能够构建出符合 Material Design 规范的精美应用。建议结合官方文档和实际项目练习，深入理解每个组件的用法和最佳实践。

