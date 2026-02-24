# Flutter animations 2.1.1 完整详解与实战指南

## 前言

动画是 Flutter 应用开发中不可或缺的一部分，它能够让应用更加生动、流畅，提升用户体验。Material Design 提供了一套完整的动效系统（Material Motion System），帮助用户理解和导航应用。

`animations` 是 Flutter 官方团队开发的动画包，它提供了基于 Material Motion 规范的预构建动画组件。这些组件可以直接集成到应用中，无需复杂的动画配置，即可实现专业级的过渡效果。

本书基于 `animations` 2.1.1 版本，详细介绍该包的所有组件、API 用法，以及如何将其集成到 Flutter 应用中。

---

## 第1章 animations 基础

### 1.1 什么是 Material Motion

Material Motion 是 Google Material Design 规范中的一套过渡动画系统，它定义了四种主要的过渡模式：

| 过渡模式 | 说明 | 适用场景 |
|----------|------|----------|
| **Container Transform** | 容器变换 | 卡片到详情页、列表项到详情页、FAB 到详情页 |
| **Shared Axis** | 共享轴 | 有空间或导航关系的页面切换 |
| **Fade Through** | 淡入淡出 | 没有强关联关系的页面切换 |
| **Fade** | 淡入缩放 | 对话框、菜单、Snackbar 等 |

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  animations: ^2.1.1
```

#### 1.2.2 导入包

```dart
import 'package:animations/animations.dart';
```

### 1.3 第一个动画

```dart
import 'package:flutter/material.dart';
import 'package:animations/animations.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Animations Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animations Demo')),
      body: Center(
        child: OpenContainer(
          closedBuilder: (context, openContainer) {
            return ElevatedButton(
              onPressed: openContainer,
              child: const Text('打开详情页'),
            );
          },
          openBuilder: (context, closeContainer) {
            return Scaffold(
              appBar: AppBar(
                title: const Text('详情页'),
                leading: IconButton(
                  icon: const Icon(Icons.close),
                  onPressed: closeContainer,
                ),
              ),
              body: const Center(
                child: Text('这是详情页内容'),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

> **Flutter框架小知识**
> 
> `animations` 包中的所有过渡动画都遵循 Material Design 规范，这意味着它们在不同平台上的外观和行为都是一致的。这些动画经过精心设计，能够自然地引导用户的注意力，帮助用户理解页面之间的关系。

---

## 第2章 OpenContainer（容器变换）

`OpenContainer` 是 `animations` 包中最强大的组件之一，它实现了容器变换动画，让一个容器平滑地展开填满屏幕，展示新的内容。

### 2.1 核心特性

| 特性 | 说明 |
|------|------|
| 无缝过渡 | 容器从闭合状态平滑过渡到展开状态 |
| 内容淡入淡出 | 内容在过渡过程中淡入淡出 |
| 高度可定制 | 支持自定义颜色、形状、阴影、持续时间等 |
| 支持返回值 | 可以通过回调获取返回值 |

### 2.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| closedBuilder | CloseContainerBuilder | 必填 | 闭合状态构建器 |
| openBuilder | OpenContainerBuilder<T> | 必填 | 展开状态构建器 |
| closedColor | Color | `Colors.white` | 闭合状态背景色 |
| closedElevation | double | `1.0` | 闭合状态阴影高度 |
| closedShape | ShapeBorder | `RoundedRectangleBorder(borderRadius: BorderRadius.all(Radius.circular(4.0)))` | 闭合状态形状 |
| openColor | Color | `Colors.white` | 展开状态背景色 |
| openElevation | double | `1.0` | 展开状态阴影高度 |
| openShape | ShapeBorder | `RoundedRectangleBorder()` | 展开状态形状 |
| transitionDuration | Duration | `Duration(milliseconds: 300)` | 过渡持续时间 |
| transitionType | ContainerTransitionType | `ContainerTransitionType.fade` | 过渡类型 |
| useRootNavigator | bool | `false` | 是否使用根导航器 |
| routeSettings | RouteSettings? | `null` | 路由设置 |
| onClosed | ClosedCallback<T>? | `null` | 关闭回调 |
| tappable | bool | `true` | 是否可点击 |

### 2.3 基础用法

```dart
OpenContainer(
  closedBuilder: (context, openContainer) {
    return Card(
      child: ListTile(
        leading: const Icon(Icons.person),
        title: const Text('用户详情'),
        subtitle: const Text('点击查看更多信息'),
        onTap: openContainer,
      ),
    );
  },
  openBuilder: (context, closeContainer) {
    return UserDetailPage(onClose: closeContainer);
  },
)
```

### 2.4 完整示例：卡片到详情页

```dart
class ProductCard extends StatelessWidget {
  final Product product;

  const ProductCard({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return OpenContainer(
      // 闭合状态样式
      closedColor: Colors.white,
      closedElevation: 4.0,
      closedShape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12.0),
      ),
      // 展开状态样式
      openColor: Theme.of(context).scaffoldBackgroundColor,
      openElevation: 0.0,
      openShape: const RoundedRectangleBorder(),
      // 过渡配置
      transitionDuration: const Duration(milliseconds: 500),
      transitionType: ContainerTransitionType.fadeThrough,
      // 闭合状态构建器
      closedBuilder: (context, openContainer) {
        return GestureDetector(
          onTap: openContainer,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              ClipRRect(
                borderRadius: const BorderRadius.vertical(
                  top: Radius.circular(12.0),
                ),
                child: Image.network(
                  product.imageUrl,
                  height: 150,
                  width: double.infinity,
                  fit: BoxFit.cover,
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(12.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      product.name,
                      style: Theme.of(context).textTheme.titleMedium,
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '\$${product.price}',
                      style: Theme.of(context).textTheme.titleLarge?.copyWith(
                            color: Theme.of(context).colorScheme.primary,
                            fontWeight: FontWeight.bold,
                          ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        );
      },
      // 展开状态构建器
      openBuilder: (context, closeContainer) {
        return ProductDetailPage(
          product: product,
          onClose: closeContainer,
        );
      },
    );
  }
}

class ProductDetailPage extends StatelessWidget {
  final Product product;
  final VoidCallback onClose;

  const ProductDetailPage({
    super.key,
    required this.product,
    required this.onClose,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            pinned: true,
            leading: IconButton(
              icon: const Icon(Icons.close),
              onPressed: onClose,
            ),
            flexibleSpace: FlexibleSpaceBar(
              background: Image.network(
                product.imageUrl,
                fit: BoxFit.cover,
              ),
            ),
          ),
          SliverPadding(
            padding: const EdgeInsets.all(16.0),
            sliver: SliverList(
              delegate: SliverChildListDelegate([
                Text(
                  product.name,
                  style: Theme.of(context).textTheme.headlineSmall,
                ),
                const SizedBox(height: 8),
                Text(
                  '\$${product.price}',
                  style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                        color: Theme.of(context).colorScheme.primary,
                        fontWeight: FontWeight.bold,
                      ),
                ),
                const SizedBox(height: 16),
                Text(product.description),
                const SizedBox(height: 24),
                ElevatedButton(
                  onPressed: () {
                    // 添加到购物车
                  },
                  child: const Text('添加到购物车'),
                ),
              ]),
            ),
          ),
        ],
      ),
    );
  }
}
```

### 2.5 FAB 到详情页

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('首页')),
      body: const Center(child: Text('点击 FAB 创建新项目')),
      floatingActionButton: OpenContainer(
        closedColor: Theme.of(context).colorScheme.primary,
        closedElevation: 6.0,
        closedShape: const CircleBorder(),
        openColor: Theme.of(context).scaffoldBackgroundColor,
        openElevation: 0.0,
        transitionDuration: const Duration(milliseconds: 400),
        closedBuilder: (context, openContainer) {
          return FloatingActionButton(
            onPressed: openContainer,
            elevation: 0.0,
            backgroundColor: Colors.transparent,
            child: const Icon(Icons.add),
          );
        },
        openBuilder: (context, closeContainer) {
          return CreateProjectPage(onClose: closeContainer);
        },
      ),
    );
  }
}
```

### 2.6 带返回值

```dart
OpenContainer<bool>(
  onClosed: (result) {
    if (result == true) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('操作成功')),
      );
    }
  },
  closedBuilder: (context, openContainer) {
    return ElevatedButton(
      onPressed: openContainer,
      child: const Text('打开设置'),
    );
  },
  openBuilder: (context, closeContainer) {
    return SettingsPage(onSave: () => closeContainer(true));
  },
)
```

### 2.7 ContainerTransitionType

| 类型 | 说明 |
|------|------|
| `ContainerTransitionType.fade` | 内容淡入淡出（默认） |
| `ContainerTransitionType.fadeThrough` | 先淡出再淡入，中间有短暂空白 |

---

## 第3章 SharedAxisTransition（共享轴）

`SharedAxisTransition` 实现了共享轴动画，用于有空间或导航关系的页面之间的过渡。

### 3.1 核心特性

| 特性 | 说明 |
|------|------|
| 轴向过渡 | 支持 X、Y、Z 三个轴向的过渡 |
| 空间关系 | 强化页面之间的空间或导航关系 |
| 统一变换 | 进出页面共享相同的变换动画 |

### 3.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| animation | Animation<double> | 必填 | 主动画 |
| secondaryAnimation | Animation<double> | 必填 | 次动画 |
| transitionType | SharedAxisTransitionType | 必填 | 过渡类型 |
| fillColor | Color | `Colors.transparent` | 填充颜色 |
| child | Widget | 必填 | 子组件 |

### 3.3 SharedAxisTransitionType

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| `horizontal` | 水平轴（X轴） | 左右导航，如步骤条 |
| `vertical` | 垂直轴（Y轴） | 上下导航，如表单步骤 |
| `scaled` | 缩放轴（Z轴） | 层级导航，如父页面到子页面 |

### 3.4 基础用法

```dart
PageTransitionSwitcher(
  transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
    return SharedAxisTransition(
      animation: primaryAnimation,
      secondaryAnimation: secondaryAnimation,
      transitionType: SharedAxisTransitionType.horizontal,
      child: child,
    );
  },
  child: currentPage,
)
```

### 3.5 完整示例：步骤表单

```dart
class StepForm extends StatefulWidget {
  const StepForm({super.key});

  @override
  State<StepForm> createState() => _StepFormState();
}

class _StepFormState extends State<StepForm> {
  int _currentStep = 0;

  final List<Widget> _steps = [
    const PersonalInfoStep(),
    const ContactInfoStep(),
    const ReviewStep(),
  ];

  void _nextStep() {
    if (_currentStep < _steps.length - 1) {
      setState(() => _currentStep++);
    }
  }

  void _previousStep() {
    if (_currentStep > 0) {
      setState(() => _currentStep--);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('步骤 ${_currentStep + 1}/${_steps.length}'),
      ),
      body: Column(
        children: [
          LinearProgressIndicator(
            value: (_currentStep + 1) / _steps.length,
          ),
          Expanded(
            child: PageTransitionSwitcher(
              duration: const Duration(milliseconds: 300),
              reverse: _currentStep < (_currentStep > 0 ? _currentStep - 1 : 0),
              transitionBuilder: (
                child,
                primaryAnimation,
                secondaryAnimation,
              ) {
                return SharedAxisTransition(
                  animation: primaryAnimation,
                  secondaryAnimation: secondaryAnimation,
                  transitionType: SharedAxisTransitionType.horizontal,
                  child: child,
                );
              },
              child: _steps[_currentStep],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                if (_currentStep > 0)
                  ElevatedButton(
                    onPressed: _previousStep,
                    child: const Text('上一步'),
                  )
                else
                  const SizedBox(),
                ElevatedButton(
                  onPressed: _nextStep,
                  child: Text(
                    _currentStep < _steps.length - 1 ? '下一步' : '完成',
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 3.6 完整示例：底部导航栏切换

```dart
class MainScreen extends StatefulWidget {
  const MainScreen({super.key});

  @override
  State<MainScreen> createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _selectedIndex = 0;

  final List<Widget> _pages = [
    const HomePage(),
    const SearchPage(),
    const ProfilePage(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageTransitionSwitcher(
        duration: const Duration(milliseconds: 300),
        transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
          return SharedAxisTransition(
            animation: primaryAnimation,
            secondaryAnimation: secondaryAnimation,
            transitionType: SharedAxisTransitionType.scaled,
            child: child,
          );
        },
        child: _pages[_selectedIndex],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (index) {
          setState(() => _selectedIndex = index);
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: '首页',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: '搜索',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outlined),
            selectedIcon: Icon(Icons.person),
            label: '我的',
          ),
        ],
      ),
    );
  }
}
```

---

## 第4章 FadeThroughTransition（淡入淡出）

`FadeThroughTransition` 实现了淡入淡出动画，用于没有强关联关系的页面之间的过渡。

### 4.1 核心特性

| 特性 | 说明 |
|------|------|
| 淡出再淡入 | 先淡出当前页面，再淡入新页面 |
| 缩放效果 | 新页面淡入时伴随轻微缩放 |
| 适合无关页面 | 适用于底部导航栏切换等场景 |

### 4.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| animation | Animation<double> | 必填 | 主动画 |
| secondaryAnimation | Animation<double> | 必填 | 次动画 |
| fillColor | Color | `Colors.transparent` | 填充颜色 |
| child | Widget | 必填 | 子组件 |

### 4.3 基础用法

```dart
PageTransitionSwitcher(
  transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
    return FadeThroughTransition(
      animation: primaryAnimation,
      secondaryAnimation: secondaryAnimation,
      child: child,
    );
  },
  child: currentPage,
)
```

### 4.4 完整示例：底部导航栏

```dart
class Dashboard extends StatefulWidget {
  const Dashboard({super.key});

  @override
  State<Dashboard> createState() => _DashboardState();
}

class _DashboardState extends State<Dashboard> {
  int _currentIndex = 0;

  final List<Widget> _pages = [
    const DashboardHome(),
    const DashboardStats(),
    const DashboardSettings(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('仪表盘')),
      body: PageTransitionSwitcher(
        duration: const Duration(milliseconds: 300),
        transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
          return FadeThroughTransition(
            animation: primaryAnimation,
            secondaryAnimation: secondaryAnimation,
            child: child,
          );
        },
        child: _pages[_currentIndex],
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.dashboard),
            label: '概览',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.analytics),
            label: '统计',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.settings),
            label: '设置',
          ),
        ],
      ),
    );
  }
}
```

### 4.5 作为页面转场动画

```dart
MaterialApp(
  theme: ThemeData(
    pageTransitionsTheme: const PageTransitionsTheme(
      builders: {
        TargetPlatform.android: FadeThroughPageTransitionsBuilder(),
        TargetPlatform.iOS: FadeThroughPageTransitionsBuilder(),
      },
    ),
  ),
  home: const HomePage(),
)
```

---

## 第5章 FadeScaleTransition（淡入缩放）

`FadeScaleTransition` 实现了淡入缩放动画，主要用于对话框、菜单、Snackbar 等组件的显示和隐藏。

### 5.1 核心特性

| 特性 | 说明 |
|------|------|
| 淡入缩放 | 进入时从 80% 缩放到 100% 并淡入 |
| 淡出 | 退出时直接淡出 |
| 适合弹窗 | 适用于对话框、菜单等 |

### 5.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| animation | Animation<double> | 必填 | 动画 |
| child | Widget | 必填 | 子组件 |

### 5.3 基础用法

```dart
FadeScaleTransition(
  animation: _animationController,
  child: MyWidget(),
)
```

### 5.4 完整示例：带动画的对话框

```dart
class AnimatedDialog extends StatefulWidget {
  const AnimatedDialog({super.key});

  @override
  State<AnimatedDialog> createState() => _AnimatedDialogState();
}

class _AnimatedDialogState extends State<AnimatedDialog>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 200),
      vsync: this,
    );
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeScaleTransition(
      animation: _controller,
      child: AlertDialog(
        title: const Text('确认删除'),
        content: const Text('确定要删除此项目吗？'),
        actions: [
          TextButton(
            onPressed: () {
              _controller.reverse().then((_) => Navigator.pop(context));
            },
            child: const Text('取消'),
          ),
          TextButton(
            onPressed: () {
              _controller.reverse().then((_) => Navigator.pop(context, true));
            },
            child: const Text('删除'),
          ),
        ],
      ),
    );
  }
}
```

### 5.5 使用 showModal

`animations` 包提供了 `showModal` 函数，可以直接显示带 FadeScaleTransition 的模态框。

```dart
void _showConfirmDialog() async {
  final result = await showModal<bool>(
    context: context,
    configuration: const FadeScaleTransitionConfiguration(),
    builder: (context) {
      return AlertDialog(
        title: const Text('确认操作'),
        content: const Text('确定要执行此操作吗？'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('取消'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('确认'),
          ),
        ],
      );
    },
  );

  if (result == true) {
    // 用户确认
  }
}
```

### 5.6 FadeScaleTransitionConfiguration

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| barrierColor | Color | `Colors.black54` | 遮罩颜色 |
| barrierDismissible | bool | `true` | 点击遮罩是否关闭 |
| transitionDuration | Duration | `Duration(milliseconds: 200)` | 过渡持续时间 |
| reverseTransitionDuration | Duration | `Duration(milliseconds: 100)` | 反向过渡持续时间 |

---

## 第6章 PageTransitionSwitcher（页面过渡切换器）

`PageTransitionSwitcher` 是一个通用的页面过渡切换组件，当子组件变化时，它会使用指定的过渡动画进行切换。

### 6.1 核心特性

| 特性 | 说明 |
|------|------|
| 通用过渡 | 支持任意类型的过渡动画 |
| 自动管理 | 自动处理新旧页面的进出动画 |
| 灵活配置 | 可自定义过渡时长、布局等 |

### 6.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| child | Widget? | `null` | 当前显示的子组件 |
| duration | Duration | `Duration(milliseconds: 300)` | 过渡持续时间 |
| reverse | bool | `false` | 是否反向播放 |
| transitionBuilder | PageTransitionSwitcherTransitionBuilder | 必填 | 过渡构建器 |
| layoutBuilder | PageTransitionSwitcherLayoutBuilder | `defaultLayoutBuilder` | 布局构建器 |

### 6.3 基础用法

```dart
PageTransitionSwitcher(
  duration: const Duration(milliseconds: 300),
  transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
    return FadeTransition(
      opacity: primaryAnimation,
      child: child,
    );
  },
  child: currentWidget,
)
```

### 6.4 完整示例：标签页切换

```dart
class TabSwitcher extends StatefulWidget {
  const TabSwitcher({super.key});

  @override
  State<TabSwitcher> createState() => _TabSwitcherState();
}

class _TabSwitcherState extends State<TabSwitcher> {
  int _selectedTab = 0;

  final List<TabData> _tabs = [
    TabData(icon: Icons.home, label: '首页', color: Colors.blue),
    TabData(icon: Icons.favorite, label: '收藏', color: Colors.red),
    TabData(icon: Icons.settings, label: '设置', color: Colors.green),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('标签页切换')),
      body: Column(
        children: [
          Expanded(
            child: PageTransitionSwitcher(
              duration: const Duration(milliseconds: 400),
              reverse: false,
              transitionBuilder: (
                child,
                primaryAnimation,
                secondaryAnimation,
              ) {
                return SlideTransition(
                  position: Tween<Offset>(
                    begin: const Offset(1.0, 0.0),
                    end: Offset.zero,
                  ).animate(CurvedAnimation(
                    parent: primaryAnimation,
                    curve: Curves.easeInOut,
                  )),
                  child: FadeTransition(
                    opacity: primaryAnimation,
                    child: child,
                  ),
                );
              },
              child: Container(
                key: ValueKey<int>(_selectedTab),
                color: _tabs[_selectedTab].color.withOpacity(0.1),
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        _tabs[_selectedTab].icon,
                        size: 100,
                        color: _tabs[_selectedTab].color,
                      ),
                      const SizedBox(height: 16),
                      Text(
                        _tabs[_selectedTab].label,
                        style: TextStyle(
                          fontSize: 24,
                          color: _tabs[_selectedTab].color,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
          Container(
            decoration: BoxDecoration(
              color: Colors.white,
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.1),
                  blurRadius: 8,
                  offset: const Offset(0, -4),
                ),
              ],
            ),
            child: SafeArea(
              child: Padding(
                padding: const EdgeInsets.all(8.0),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: List.generate(_tabs.length, (index) {
                    return IconButton(
                      icon: Icon(_tabs[index].icon),
                      color: _selectedTab == index
                          ? _tabs[index].color
                          : Colors.grey,
                      onPressed: () => setState(() => _selectedTab = index),
                    );
                  }),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class TabData {
  final IconData icon;
  final String label;
  final Color color;

  TabData({required this.icon, required this.label, required this.color});
}
```

### 6.5 自定义布局构建器

```dart
PageTransitionSwitcher(
  layoutBuilder: (entries) {
    return Stack(
      alignment: Alignment.center,
      children: entries,
    );
  },
  transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
    return ScaleTransition(
      scale: primaryAnimation,
      child: child,
    );
  },
  child: currentWidget,
)
```

---

## 第7章 实战案例

### 7.1 电商应用：商品列表到详情

```dart
class ProductListPage extends StatelessWidget {
  final List<Product> products;

  const ProductListPage({super.key, required this.products});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('商品列表')),
      body: GridView.builder(
        padding: const EdgeInsets.all(16.0),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
          crossAxisSpacing: 16.0,
          mainAxisSpacing: 16.0,
        ),
        itemCount: products.length,
        itemBuilder: (context, index) {
          return ProductCard(product: products[index]);
        },
      ),
    );
  }
}

class ProductCard extends StatelessWidget {
  final Product product;

  const ProductCard({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return OpenContainer(
      closedColor: Colors.white,
      closedElevation: 2.0,
      closedShape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12.0),
      ),
      openColor: Theme.of(context).scaffoldBackgroundColor,
      openElevation: 0.0,
      transitionDuration: const Duration(milliseconds: 500),
      closedBuilder: (context, openContainer) {
        return GestureDetector(
          onTap: openContainer,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Expanded(
                child: ClipRRect(
                  borderRadius: const BorderRadius.vertical(
                    top: Radius.circular(12.0),
                  ),
                  child: Image.network(
                    product.imageUrl,
                    fit: BoxFit.cover,
                    width: double.infinity,
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(12.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      product.name,
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                      style: Theme.of(context).textTheme.titleSmall,
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '\$${product.price}',
                      style: Theme.of(context).textTheme.titleMedium?.copyWith(
                            color: Theme.of(context).colorScheme.primary,
                            fontWeight: FontWeight.bold,
                          ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        );
      },
      openBuilder: (context, closeContainer) {
        return ProductDetailPage(
          product: product,
          onClose: closeContainer,
        );
      },
    );
  }
}
```

### 7.2 社交应用：底部导航切换

```dart
class SocialApp extends StatefulWidget {
  const SocialApp({super.key});

  @override
  State<SocialApp> createState() => _SocialAppState();
}

class _SocialAppState extends State<SocialApp> {
  int _currentIndex = 0;

  final List<NavigationDestination> _destinations = const [
    NavigationDestination(
      icon: Icon(Icons.home_outlined),
      selectedIcon: Icon(Icons.home),
      label: '首页',
    ),
    NavigationDestination(
      icon: Icon(Icons.search_outlined),
      selectedIcon: Icon(Icons.search),
      label: '发现',
    ),
    NavigationDestination(
      icon: Icon(Icons.add_box_outlined),
      selectedIcon: Icon(Icons.add_box),
      label: '发布',
    ),
    NavigationDestination(
      icon: Icon(Icons.notifications_outlined),
      selectedIcon: Icon(Icons.notifications),
      label: '消息',
    ),
    NavigationDestination(
      icon: Icon(Icons.person_outlined),
      selectedIcon: Icon(Icons.person),
      label: '我的',
    ),
  ];

  final List<Widget> _pages = const [
    FeedPage(),
    ExplorePage(),
    CreatePostPage(),
    NotificationsPage(),
    ProfilePage(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageTransitionSwitcher(
        duration: const Duration(milliseconds: 300),
        transitionBuilder: (child, primaryAnimation, secondaryAnimation) {
          return FadeThroughTransition(
            animation: primaryAnimation,
            secondaryAnimation: secondaryAnimation,
            child: child,
          );
        },
        child: _pages[_currentIndex],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) {
          setState(() => _currentIndex = index);
        },
        destinations: _destinations,
      ),
    );
  }
}
```

### 7.3 设置向导：步骤切换

```dart
class OnboardingWizard extends StatefulWidget {
  const OnboardingWizard({super.key});

  @override
  State<OnboardingWizard> createState() => _OnboardingWizardState();
}

class _OnboardingWizardState extends State<OnboardingWizard> {
  int _currentStep = 0;
  final int _totalSteps = 4;

  final List<OnboardingStep> _steps = [
    OnboardingStep(
      title: '欢迎使用',
      description: '这是应用的介绍页面',
      icon: Icons.waving_hand,
      color: Colors.blue,
    ),
    OnboardingStep(
      title: '功能介绍',
      description: '了解应用的核心功能',
      icon: Icons.lightbulb,
      color: Colors.orange,
    ),
    OnboardingStep(
      title: '个性化设置',
      description: '根据您的喜好定制应用',
      icon: Icons.tune,
      color: Colors.purple,
    ),
    OnboardingStep(
      title: '开始使用',
      description: '一切准备就绪！',
      icon: Icons.rocket_launch,
      color: Colors.green,
    ),
  ];

  void _nextStep() {
    if (_currentStep < _totalSteps - 1) {
      setState(() => _currentStep++);
    } else {
      _finishOnboarding();
    }
  }

  void _previousStep() {
    if (_currentStep > 0) {
      setState(() => _currentStep--);
    }
  }

  void _finishOnboarding() {
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => const HomePage()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            // 进度指示器
            LinearProgressIndicator(
              value: (_currentStep + 1) / _totalSteps,
              backgroundColor: Colors.grey[200],
              valueColor: AlwaysStoppedAnimation<Color>(
                _steps[_currentStep].color,
              ),
            ),
            // 跳过按钮
            Align(
              alignment: Alignment.topRight,
              child: TextButton(
                onPressed: _finishOnboarding,
                child: const Text('跳过'),
              ),
            ),
            // 步骤内容
            Expanded(
              child: PageTransitionSwitcher(
                duration: const Duration(milliseconds: 400),
                reverse: _currentStep < (_currentStep > 0 ? _currentStep - 1 : 0),
                transitionBuilder: (
                  child,
                  primaryAnimation,
                  secondaryAnimation,
                ) {
                  return SharedAxisTransition(
                    animation: primaryAnimation,
                    secondaryAnimation: secondaryAnimation,
                    transitionType: SharedAxisTransitionType.horizontal,
                    child: child,
                  );
                },
                child: Padding(
                  key: ValueKey<int>(_currentStep),
                  padding: const EdgeInsets.all(32.0),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        _steps[_currentStep].icon,
                        size: 120,
                        color: _steps[_currentStep].color,
                      ),
                      const SizedBox(height: 32),
                      Text(
                        _steps[_currentStep].title,
                        style: Theme.of(context).textTheme.headlineMedium,
                        textAlign: TextAlign.center,
                      ),
                      const SizedBox(height: 16),
                      Text(
                        _steps[_currentStep].description,
                        style: Theme.of(context).textTheme.bodyLarge,
                        textAlign: TextAlign.center,
                      ),
                    ],
                  ),
                ),
              ),
            ),
            // 导航按钮
            Padding(
              padding: const EdgeInsets.all(32.0),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  if (_currentStep > 0)
                    TextButton(
                      onPressed: _previousStep,
                      child: const Text('上一步'),
                    )
                  else
                    const SizedBox(width: 80),
                  ElevatedButton(
                    onPressed: _nextStep,
                    style: ElevatedButton.styleFrom(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 32,
                        vertical: 16,
                      ),
                    ),
                    child: Text(
                      _currentStep < _totalSteps - 1 ? '下一步' : '开始',
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class OnboardingStep {
  final String title;
  final String description;
  final IconData icon;
  final Color color;

  OnboardingStep({
    required this.title,
    required this.description,
    required this.icon,
    required this.color,
  });
}
```

### 7.4 邮件应用：列表项展开

```dart
class EmailListPage extends StatefulWidget {
  const EmailListPage({super.key});

  @override
  State<EmailListPage> createState() => _EmailListPageState();
}

class _EmailListPageState extends State<EmailListPage> {
  final List<Email> _emails = [
    Email(
      sender: '张三',
      subject: '项目进度汇报',
      preview: '这是本周的项目进度汇报...',
      time: '10:30',
      avatar: 'Z',
    ),
    Email(
      sender: '李四',
      subject: '会议邀请',
      preview: '邀请您参加明天的项目会议...',
      time: '09:15',
      avatar: 'L',
    ),
    // 更多邮件...
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('收件箱')),
      body: ListView.builder(
        itemCount: _emails.length,
        itemBuilder: (context, index) {
          return OpenContainer(
            closedColor: Colors.transparent,
            closedElevation: 0,
            openColor: Theme.of(context).scaffoldBackgroundColor,
            openElevation: 0,
            transitionDuration: const Duration(milliseconds: 400),
            closedBuilder: (context, openContainer) {
              return ListTile(
                leading: CircleAvatar(
                  child: Text(_emails[index].avatar),
                ),
                title: Text(_emails[index].sender),
                subtitle: Text(_emails[index].preview),
                trailing: Text(_emails[index].time),
                onTap: openContainer,
              );
            },
            openBuilder: (context, closeContainer) {
              return EmailDetailPage(
                email: _emails[index],
                onClose: closeContainer,
              );
            },
          );
        },
      ),
      floatingActionButton: OpenContainer(
        closedColor: Theme.of(context).colorScheme.primary,
        closedShape: const CircleBorder(),
        openColor: Theme.of(context).scaffoldBackgroundColor,
        transitionDuration: const Duration(milliseconds: 300),
        closedBuilder: (context, openContainer) {
          return FloatingActionButton(
            onPressed: openContainer,
            elevation: 0,
            backgroundColor: Colors.transparent,
            child: const Icon(Icons.edit),
          );
        },
        openBuilder: (context, closeContainer) {
          return ComposeEmailPage(onClose: closeContainer);
        },
      ),
    );
  }
}
```

---

## 第8章 最佳实践

### 8.1 选择合适的过渡动画

| 场景 | 推荐动画 | 说明 |
|------|----------|------|
| 卡片到详情页 | `OpenContainer` | 容器变换，视觉连接强 |
| 底部导航切换 | `FadeThroughTransition` | 页面关系弱，淡入淡出自然 |
| 步骤表单 | `SharedAxisTransition.horizontal` | 水平导航关系 |
| 上下滚动内容 | `SharedAxisTransition.vertical` | 垂直导航关系 |
| 层级导航 | `SharedAxisTransition.scaled` | 深度感 |
| 对话框/菜单 | `FadeScaleTransition` | 弹窗效果 |

### 8.2 性能优化

```dart
// 使用 const 构造函数
const OpenContainer(
  // ...
)

// 避免在动画中重建复杂 Widget
PageTransitionSwitcher(
  child: RepaintBoundary(
    child: currentPage,
  ),
)

// 控制动画时长
OpenContainer(
  transitionDuration: const Duration(milliseconds: 300), // 不要过长
)
```

### 8.3 无障碍支持

```dart
OpenContainer(
  closedBuilder: (context, openContainer) {
    return Semantics(
      button: true,
      label: '打开详情页',
      child: GestureDetector(
        onTap: openContainer,
        child: const Card(
          child: ListTile(
            title: Text('标题'),
          ),
        ),
      ),
    );
  },
  // ...
)
```

### 8.4 项目结构建议

```
lib/
├── main.dart
├── pages/
│   ├── home_page.dart
│   ├── product_list_page.dart
│   └── product_detail_page.dart
├── widgets/
│   ├── transitions/
│   │   ├── fade_through_page.dart
│   │   └── shared_axis_page.dart
│   └── product_card.dart
└── models/
    └── product.dart
```

### 8.5 DO 和 DON'T

#### ✅ DO

- 根据页面关系选择合适的过渡动画
- 保持过渡时长一致（通常 200-500ms）
- 使用 `PageTransitionSwitcher` 管理页面切换
- 为复杂 Widget 添加 `RepaintBoundary`
- 测试动画在不同设备上的性能

#### ❌ DON'T

- 不要过度使用动画
- 不要在动画中执行耗时操作
- 不要让过渡时长过长（超过 500ms）
- 不要在同一个页面混合多种过渡风格
- 不要忽略无障碍支持

---

## 附录：animations API 速查表

### 组件列表

| 组件名 | 用途 |
|--------|------|
| `OpenContainer` | 容器变换动画 |
| `SharedAxisTransition` | 共享轴过渡 |
| `FadeThroughTransition` | 淡入淡出过渡 |
| `FadeScaleTransition` | 淡入缩放过渡 |
| `PageTransitionSwitcher` | 页面过渡切换器 |

### 过渡类型

| 类型 | 说明 |
|------|------|
| `ContainerTransitionType.fade` | 内容淡入淡出 |
| `ContainerTransitionType.fadeThrough` | 先淡出再淡入 |
| `SharedAxisTransitionType.horizontal` | 水平轴 |
| `SharedAxisTransitionType.vertical` | 垂直轴 |
| `SharedAxisTransitionType.scaled` | 缩放轴 |

### PageTransitionsBuilder

| Builder | 用途 |
|---------|------|
| `FadeThroughPageTransitionsBuilder` | 全局淡入淡出过渡 |
| `SharedAxisPageTransitionsBuilder` | 全局共享轴过渡 |

---

## 结语

`animations` 2.1.1 是 Flutter 官方提供的 Material Motion 动画包，它让实现专业级的过渡动画变得简单。通过本书的学习，你应该已经掌握了：

1. Material Motion 的四种过渡模式
2. `OpenContainer` 的容器变换动画
3. `SharedAxisTransition` 的共享轴过渡
4. `FadeThroughTransition` 的淡入淡出过渡
5. `FadeScaleTransition` 的淡入缩放过渡
6. `PageTransitionSwitcher` 的通用页面切换

希望本书能够帮助你在 Flutter 开发中更好地使用 `animations` 包，为用户带来流畅、自然的动画体验。

---

**版本信息**

- animations 版本：2.1.1
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
