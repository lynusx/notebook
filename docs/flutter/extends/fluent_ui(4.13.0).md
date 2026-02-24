# Flutter fluent_ui 4.13.0 组件详解与实战

## 前言

Flutter 是 Google 推出的一款开源 UI 软件开发工具包，它允许开发者使用一套代码库构建跨平台应用程序。除了 Material Design 和 Cupertino 风格外，对于 Windows 平台的应用开发，Microsoft 提供了 Fluent Design System 设计语言。

`fluent_ui` 是一个非官方的 Flutter 包，实现了 Microsoft 的 Windows UI（WinUI 3）设计规范。它为开发者提供了构建具有 Windows 原生风格应用的能力，包括 Acrylic 材质效果、Reveal 聚焦效果、NavigationView 导航等 Fluent Design 特有的视觉元素。

本书基于 `fluent_ui` 4.13.0 版本，详细介绍该包中包含的全部组件，包括每个组件的属性、方法用途及意义，并提供丰富的示例代码，帮助读者深入理解和灵活运用这些组件。

---

## 第1章 基础应用组件

### 1.1 FluentApp

FluentApp 是 Fluent UI 风格应用的顶层组件，类似于 Material 中的 MaterialApp，它提供了主题、导航、本地化等 Fluent 应用所需的基础配置。

#### 1.1.1 基础用法

```dart
import 'package:fluent_ui/fluent_ui.dart';

void main() {
  runApp(FluentApp(
    title: 'Fluent App',
    home: HomePage(),
  ));
}
```

#### 1.1.2 核心属性详解

| 属性名                     | 类型                                  | 默认值                     | 用途说明             |
| -------------------------- | ------------------------------------- | -------------------------- | -------------------- |
| navigatorKey               | GlobalKey<NavigatorState>?            | null                       | 导航器的全局键       |
| home                       | Widget?                               | null                       | 应用首页             |
| routes                     | Map<String, WidgetBuilder>?           | null                       | 路由表               |
| initialRoute               | String?                               | null                       | 初始路由             |
| onGenerateRoute            | RouteFactory?                         | null                       | 路由生成器           |
| onUnknownRoute             | RouteFactory?                         | null                       | 未知路由处理器       |
| navigatorObservers         | List<NavigatorObserver>               | const []                   | 导航观察者           |
| builder                    | TransitionBuilder?                    | null                       | 应用构建器           |
| title                      | String                                | ''                         | 应用标题             |
| onGenerateTitle            | GenerateAppTitle?                     | null                       | 标题生成器           |
| color                      | Color?                                | null                       | 应用主色             |
| theme                      | FluentThemeData?                      | null                       | Fluent 主题          |
| darkTheme                  | FluentThemeData?                      | null                       | 暗色主题             |
| themeMode                  | ThemeMode?                            | null                       | 主题模式             |
| locale                     | Locale?                               | null                       | 当前语言区域         |
| localizationsDelegates     | List<LocalizationsDelegate<dynamic>>? | null                       | 本地化代理           |
| supportedLocales           | List<Locale>                          | const [Locale('en', 'US')] | 支持的语言区域       |
| showPerformanceOverlay     | bool                                  | false                      | 是否显示性能覆盖层   |
| showSemanticsDebugger      | bool                                  | false                      | 是否显示语义调试器   |
| debugShowCheckedModeBanner | bool                                  | true                       | 是否显示调试模式横幅 |
| shortcuts                  | Map<ShortcutActivator, Intent>?       | null                       | 快捷键映射           |
| actions                    | Map<Type, Action<Intent>>?            | null                       | 动作映射             |
| restorationScopeId         | String?                               | null                       | 恢复作用域 ID        |
| scrollBehavior             | ScrollBehavior?                       | null                       | 滚动行为             |

#### 1.1.3 完整示例

```dart
import 'package:fluent_ui/fluent_ui.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return FluentApp(
      title: 'Fluent UI Demo',
      theme: FluentThemeData(
        accentColor: Colors.blue,
        visualDensity: VisualDensity.standard,
      ),
      darkTheme: FluentThemeData(
        brightness: Brightness.dark,
        accentColor: Colors.blue,
      ),
      home: const NavigationPage(),
    );
  }
}
```

> **Flutter框架小知识**
>
> FluentApp 支持自动切换亮色/暗色主题。通过设置 `themeMode` 为 `ThemeMode.system`，应用会自动跟随系统的主题设置。你也可以使用 `ThemeMode.light` 或 `ThemeMode.dark` 强制使用特定主题。

---

### 1.2 FluentTheme

FluentTheme 用于为子组件提供 Fluent 主题配置，可以在应用的任意层级使用。

#### 1.2.1 核心属性

| 属性名 | 类型            | 默认值 | 用途说明         |
| ------ | --------------- | ------ | ---------------- |
| data   | FluentThemeData | -      | 主题数据（必传） |
| child  | Widget          | -      | 子组件（必传）   |

#### 1.2.2 使用示例

```dart
FluentTheme(
  data: FluentThemeData(
    accentColor: Colors.orange.toAccentColor(),
    activeColor: Colors.orange,
  ),
  child: MyWidget(),
)
```

---

### 1.3 FluentThemeData

FluentThemeData 定义了 Fluent 主题的完整配置数据。

#### 1.3.1 核心属性

| 属性名                  | 类型              | 默认值                 | 用途说明       |
| ----------------------- | ----------------- | ---------------------- | -------------- |
| brightness              | Brightness?       | null                   | 亮度           |
| visualDensity           | VisualDensity     | VisualDensity.standard | 视觉密度       |
| accentColor             | AccentColor?      | null                   | 强调色         |
| activeColor             | Color?            | null                   | 激活颜色       |
| inactiveColor           | Color?            | null                   | 未激活颜色     |
| inactiveBackgroundColor | Color?            | null                   | 未激活背景色   |
| shadowColor             | Color?            | null                   | 阴影颜色       |
| scaffoldBackgroundColor | Color?            | null                   | 脚手架背景色   |
| acrylicBackgroundColor  | Color?            | null                   | Acrylic 背景色 |
| menuColor               | Color?            | null                   | 菜单颜色       |
| cardColor               | Color?            | null                   | 卡片颜色       |
| selectionColor          | Color?            | null                   | 选中颜色       |
| inactiveColor           | Color?            | null                   | 未激活颜色     |
| borderInputColor        | Color?            | null                   | 输入框边框颜色 |
| focusTheme              | FocusThemeData?   | null                   | 焦点主题       |
| tooltipTheme            | TooltipThemeData? | null                   | 工具提示主题   |
| typography              | Typography?       | null                   | 排版           |
| iconTheme               | IconThemeData?    | null                   | 图标主题       |

#### 1.3.2 完整示例

```dart
FluentThemeData(
  brightness: Brightness.light,
  accentColor: Colors.blue.toAccentColor(),
  visualDensity: VisualDensity.comfortable,
  scaffoldBackgroundColor: Colors.white,
  cardColor: Colors.grey[10],
  focusTheme: FocusThemeData(
    glowFactor: 2.0,
    primaryBorder: BorderSide(
      color: Colors.blue,
      width: 2,
    ),
  ),
  typography: Typography(
    display: TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
    titleLarge: TextStyle(fontSize: 24, fontWeight: FontWeight.w600),
    body: TextStyle(fontSize: 14),
  ),
)
```

---

## 第2章 导航组件

### 2.1 NavigationView

NavigationView 是 Fluent UI 的核心导航组件，提供了 Windows 风格的侧边导航栏和内容区域。

#### 2.1.1 基础用法

```dart
NavigationView(
  appBar: NavigationAppBar(
    title: Text('Fluent App'),
  ),
  pane: NavigationPane(
    selected: _selectedIndex,
    onChanged: (index) => setState(() => _selectedIndex = index),
    items: [
      PaneItem(
        icon: Icon(FluentIcons.home),
        title: Text('首页'),
        body: HomePage(),
      ),
      PaneItem(
        icon: Icon(FluentIcons.settings),
        title: Text('设置'),
        body: SettingsPage(),
      ),
    ],
  ),
)
```

#### 2.1.2 核心属性

| 属性名            | 类型                               | 默认值        | 用途说明                 |
| ----------------- | ---------------------------------- | ------------- | ------------------------ |
| appBar            | NavigationAppBar?                  | null          | 应用栏                   |
| pane              | NavigationPane?                    | null          | 导航面板                 |
| content           | Widget?                            | null          | 内容区域（与 pane 互斥） |
| contentShape      | ShapeBorder?                       | null          | 内容区域形状             |
| transitionBuilder | AnimatedSwitcherTransitionBuilder? | null          | 过渡动画构建器           |
| clipBehavior      | Clip                               | Clip.hardEdge | 裁剪行为                 |

---

### 2.2 NavigationPane

NavigationPane 是 NavigationView 的导航面板，支持多种显示模式。

#### 2.2.1 核心属性

| 属性名                    | 类型                     | 默认值               | 用途说明           |
| ------------------------- | ------------------------ | -------------------- | ------------------ |
| key                       | Key?                     | null                 | 键                 |
| selected                  | int?                     | null                 | 当前选中索引       |
| onChanged                 | ValueChanged<int>?       | null                 | 选中变化回调       |
| onItemPressed             | VoidCallback?            | null                 | 项目按下回调       |
| displayMode               | PaneDisplayMode          | PaneDisplayMode.auto | 显示模式           |
| items                     | List<NavigationPaneItem> | const []             | 导航项列表         |
| footerItems               | List<NavigationPaneItem> | const []             | 底部导航项         |
| size                      | double?                  | null                 | 面板宽度           |
| header                    | Widget?                  | null                 | 头部组件           |
| autoSuggestBox            | Widget?                  | null                 | 自动建议框         |
| autoSuggestBoxReplacement | Widget?                  | null                 | 自动建议框替代组件 |
| indicator                 | Widget?                  | null                 | 指示器             |
| scrollController          | ScrollController?        | null                 | 滚动控制器         |

**PaneDisplayMode 枚举值：**

| 值      | 说明                           |
| ------- | ------------------------------ |
| top     | 顶部显示                       |
| open    | 始终展开                       |
| compact | 紧凑模式（仅图标）             |
| minimal | 最小模式（可展开）             |
| auto    | 自动模式（根据窗口大小自适应） |

#### 2.2.2 完整示例

```dart
NavigationPane(
  selected: _selectedIndex,
  onChanged: (index) => setState(() => _selectedIndex = index),
  displayMode: PaneDisplayMode.auto,
  header: Container(
    height: 40,
    padding: EdgeInsets.symmetric(horizontal: 16),
    child: Text('应用名称', style: TextStyle(fontWeight: FontWeight.bold)),
  ),
  items: [
    PaneItem(
      icon: Icon(FluentIcons.home),
      title: Text('首页'),
      infoBadge: InfoBadge(source: Text('3')),
      body: HomePage(),
    ),
    PaneItemSeparator(),
    PaneItem(
      icon: Icon(FluentIcons.document_set),
      title: Text('文档'),
      body: DocumentsPage(),
    ),
    PaneItem(
      icon: Icon(FluentIcons.picture),
      title: Text('图片'),
      body: ImagesPage(),
    ),
  ],
  footerItems: [
    PaneItem(
      icon: Icon(FluentIcons.settings),
      title: Text('设置'),
      body: SettingsPage(),
    ),
  ],
)
```

---

### 2.3 NavigationAppBar

NavigationAppBar 是 Fluent UI 的应用栏组件，通常与 NavigationView 配合使用。

#### 2.3.1 核心属性

| 属性名                    | 类型    | 默认值 | 用途说明             |
| ------------------------- | ------- | ------ | -------------------- |
| key                       | Key?    | null   | 键                   |
| leading                   | Widget? | null   | 左侧组件             |
| title                     | Widget? | null   | 标题                 |
| actions                   | Widget? | null   | 操作按钮             |
| height                    | double? | null   | 高度                 |
| backgroundColor           | Color?  | null   | 背景颜色             |
| automaticallyImplyLeading | bool    | true   | 是否自动显示返回按钮 |

#### 2.3.2 完整示例

```dart
NavigationAppBar(
  title: Text('Fluent UI 应用'),
  leading: IconButton(
    icon: Icon(FluentIcons.back),
    onPressed: () => Navigator.pop(context),
  ),
  actions: Row(
    children: [
      IconButton(
        icon: Icon(FluentIcons.search),
        onPressed: () {},
      ),
      IconButton(
        icon: Icon(FluentIcons.more),
        onPressed: () {},
      ),
    ],
  ),
)
```

---

### 2.4 PaneItem

PaneItem 是 NavigationPane 中的单个导航项。

#### 2.4.1 核心属性

| 属性名      | 类型         | 默认值 | 用途说明       |
| ----------- | ------------ | ------ | -------------- |
| key         | Key?         | null   | 键             |
| icon        | Widget       | -      | 图标（必传）   |
| title       | Widget?      | null   | 标题           |
| body        | Widget       | -      | 内容体（必传） |
| infoBadge   | Widget?      | null   | 信息徽章       |
| enabled     | bool         | true   | 是否启用       |
| mouseCursor | MouseCursor? | null   | 鼠标光标       |

---

### 2.5 TabView

TabView 是 Fluent UI 的标签页组件，支持多标签页切换和关闭操作。

#### 2.5.1 基础用法

```dart
TabView(
  tabs: _tabs,
  currentIndex: _currentIndex,
  onChanged: (index) => setState(() => _currentIndex = index),
  onNewPressed: () {
    setState(() {
      _tabs.add(Tab(
        text: Text('新标签'),
        body: Center(child: Text('新标签页内容')),
      ));
    });
  },
  onReorder: (oldIndex, newIndex) {
    setState(() {
      final tab = _tabs.removeAt(oldIndex);
      _tabs.insert(newIndex, tab);
    });
  },
)
```

#### 2.5.2 核心属性

| 属性名                | 类型                      | 默认值                           | 用途说明         |
| --------------------- | ------------------------- | -------------------------------- | ---------------- |
| tabs                  | List<Tab>                 | -                                | 标签列表（必传） |
| currentIndex          | int                       | 0                                | 当前选中索引     |
| onChanged             | ValueChanged<int>?        | null                             | 选中变化回调     |
| onReorder             | ReorderCallback?          | null                             | 重新排序回调     |
| onNewPressed          | VoidCallback?             | null                             | 新建标签回调     |
| closeButtonVisibility | CloseButtonVisibilityMode | CloseButtonVisibilityMode.always | 关闭按钮可见性   |
| showScrollButtons     | bool                      | true                             | 是否显示滚动按钮 |
| scrollButtonBuilder   | ScrollButtonBuilder?      | null                             | 滚动按钮构建器   |
| tabWidthBehavior      | TabWidthBehavior          | TabWidthBehavior.flexible        | 标签宽度行为     |
| header                | Widget?                   | null                             | 头部组件         |
| footer                | Widget?                   | null                             | 底部组件         |

**CloseButtonVisibilityMode 枚举值：**

| 值      | 说明       |
| ------- | ---------- |
| always  | 始终显示   |
| never   | 从不显示   |
| onHover | 悬停时显示 |

#### 2.5.3 完整示例

```dart
class TabViewDemo extends StatefulWidget {
  @override
  _TabViewDemoState createState() => _TabViewDemoState();
}

class _TabViewDemoState extends State<TabViewDemo> {
  int _currentIndex = 0;
  List<Tab> _tabs = [];

  @override
  void initState() {
    super.initState();
    _tabs = [
      Tab(
        text: Text('文档 1'),
        semanticLabel: '文档 1',
        icon: Icon(FluentIcons.document),
        body: Center(child: Text('文档 1 内容')),
      ),
      Tab(
        text: Text('文档 2'),
        semanticLabel: '文档 2',
        icon: Icon(FluentIcons.document),
        body: Center(child: Text('文档 2 内容')),
      ),
    ];
  }

  @override
  Widget build(BuildContext context) {
    return TabView(
      tabs: _tabs,
      currentIndex: _currentIndex,
      onChanged: (index) => setState(() => _currentIndex = index),
      onReorder: (oldIndex, newIndex) {
        setState(() {
          if (oldIndex < newIndex) {
            newIndex--;
          }
          final tab = _tabs.removeAt(oldIndex);
          _tabs.insert(newIndex, tab);
        });
      },
      closeButtonVisibility: CloseButtonVisibilityMode.onHover,
      onNewPressed: () {
        setState(() {
          final index = _tabs.length + 1;
          _tabs.add(Tab(
            text: Text('文档 $index'),
            semanticLabel: '文档 $index',
            icon: Icon(FluentIcons.document),
            body: Center(child: Text('文档 $index 内容')),
          ));
          _currentIndex = _tabs.length - 1;
        });
      },
    );
  }
}
```

---

### 2.6 BottomNavigation

BottomNavigation 是 Fluent UI 的底部导航组件，适用于移动设备。

#### 2.6.1 核心属性

| 属性名    | 类型                       | 默认值 | 用途说明           |
| --------- | -------------------------- | ------ | ------------------ |
| index     | int                        | 0      | 当前索引           |
| onChanged | ValueChanged<int>?         | null   | 索引变化回调       |
| items     | List<BottomNavigationItem> | -      | 导航项列表（必传） |

#### 2.6.2 完整示例

```dart
BottomNavigation(
  index: _currentIndex,
  onChanged: (index) => setState(() => _currentIndex = index),
  items: [
    BottomNavigationItem(
      icon: Icon(FluentIcons.home),
      title: Text('首页'),
    ),
    BottomNavigationItem(
      icon: Icon(FluentIcons.search),
      title: Text('搜索'),
    ),
    BottomNavigationItem(
      icon: Icon(FluentIcons.person),
      title: Text('我的'),
    ),
  ],
)
```

---

## 第3章 按钮组件

### 3.1 Button

Button 是 Fluent UI 的基础按钮组件，支持多种样式和状态。

#### 3.1.1 基础用法

```dart
Button(
  child: Text('点击我'),
  onPressed: () {
    print('按钮被点击');
  },
)
```

#### 3.1.2 核心属性

| 属性名       | 类型          | 默认值         | 用途说明         |
| ------------ | ------------- | -------------- | ---------------- |
| child        | Widget        | -              | 子组件（必传）   |
| onPressed    | VoidCallback? | -              | 点击回调（必传） |
| onLongPress  | VoidCallback? | null           | 长按回调         |
| style        | ButtonStyle?  | null           | 按钮样式         |
| focusNode    | FocusNode?    | null           | 焦点节点         |
| autofocus    | bool          | false          | 是否自动聚焦     |
| clipBehavior | Clip          | Clip.antiAlias | 裁剪行为         |

#### 3.1.3 样式定制

```dart
Button(
  style: ButtonStyle(
    backgroundColor: ButtonState.all(Colors.blue),
    foregroundColor: ButtonState.all(Colors.white),
    padding: ButtonState.all(EdgeInsets.symmetric(horizontal: 24, vertical: 12)),
    shape: ButtonState.all(RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(4),
    )),
  ),
  onPressed: () {},
  child: Text('自定义样式'),
)
```

#### 3.1.4 完整示例

```dart
Column(
  children: [
    // 默认按钮
    Button(
      onPressed: () {},
      child: Text('默认按钮'),
    ),
    SizedBox(height: 8),

    // 强调按钮
    FilledButton(
      onPressed: () {},
      child: Text('强调按钮'),
    ),
    SizedBox(height: 8),

    // 透明按钮
    TransparentButton(
      onPressed: () {},
      child: Text('透明按钮'),
    ),
    SizedBox(height: 8),

    // 禁用状态
    Button(
      onPressed: null,
      child: Text('禁用按钮'),
    ),
  ],
)
```

---

### 3.2 FilledButton

FilledButton 是填充样式的按钮，用于强调主要操作。

```dart
FilledButton(
  child: Text('保存'),
  onPressed: () {
    // 保存操作
  },
)
```

---

### 3.3 TransparentButton

TransparentButton 是透明背景的按钮，用于次要操作。

```dart
TransparentButton(
  child: Text('取消'),
  onPressed: () {
    // 取消操作
  },
)
```

---

### 3.4 IconButton

IconButton 是图标按钮，常用于工具栏。

#### 3.4.1 核心属性

| 属性名      | 类型          | 默认值 | 用途说明         |
| ----------- | ------------- | ------ | ---------------- |
| icon        | Widget        | -      | 图标（必传）     |
| onPressed   | VoidCallback? | -      | 点击回调（必传） |
| onLongPress | VoidCallback? | null   | 长按回调         |
| style       | ButtonStyle?  | null   | 按钮样式         |
| focusNode   | FocusNode?    | null   | 焦点节点         |
| autofocus   | bool          | false  | 是否自动聚焦     |

#### 3.4.2 使用示例

```dart
IconButton(
  icon: Icon(FluentIcons.add),
  onPressed: () {},
)
```

---

### 3.5 ToggleButton

ToggleButton 是切换按钮，用于表示开/关状态。

#### 3.5.1 核心属性

| 属性名    | 类型                   | 默认值 | 用途说明       |
| --------- | ---------------------- | ------ | -------------- |
| checked   | bool                   | false  | 是否选中       |
| onChanged | ValueChanged<bool>?    | null   | 状态变化回调   |
| style     | ToggleButtonThemeData? | null   | 样式           |
| child     | Widget                 | -      | 子组件（必传） |

#### 3.5.2 完整示例

```dart
ToggleButton(
  checked: _isBold,
  onChanged: (value) => setState(() => _isBold = value),
  child: Icon(FluentIcons.bold),
)
```

---

### 3.6 SplitButton

SplitButton 是分割按钮，包含主按钮和下拉按钮两部分。

#### 3.6.1 核心属性

| 属性名    | 类型          | 默认值 | 用途说明           |
| --------- | ------------- | ------ | ------------------ |
| child     | Widget        | -      | 主按钮内容（必传） |
| flyout    | FlyoutContent | -      | 下拉内容（必传）   |
| onPressed | VoidCallback? | null   | 主按钮点击回调     |
| style     | ButtonStyle?  | null   | 按钮样式           |

#### 3.6.2 完整示例

```dart
SplitButton(
  flyout: FlyoutContent(
    child: Column(
      mainAxisSize: MainAxisSize.min,
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        MenuFlyoutItem(
          text: Text('复制'),
          onPressed: () {},
        ),
        MenuFlyoutItem(
          text: Text('粘贴'),
          onPressed: () {},
        ),
      ],
    ),
  ),
  onPressed: () => print('主按钮点击'),
  child: Text('保存'),
)
```

---

### 3.7 HyperlinkButton

HyperlinkButton 是超链接样式的按钮，主要用于文本内容。

```dart
HyperlinkButton(
  child: Text('了解更多'),
  onPressed: () {},
)
```

---

### 3.8 RepeatButton

RepeatButton 是重复按钮，按住时会持续触发 onPressed 回调。

#### 3.8.1 核心属性

| 属性名    | 类型          | 默认值                            | 用途说明         |
| --------- | ------------- | --------------------------------- | ---------------- |
| child     | Widget        | -                                 | 子组件（必传）   |
| onPressed | VoidCallback? | -                                 | 点击回调（必传） |
| interval  | Duration      | const Duration(milliseconds: 100) | 重复间隔         |
| style     | ButtonStyle?  | null                              | 按钮样式         |

```dart
RepeatButton(
  onPressed: () => setState(() => _counter++),
  interval: Duration(milliseconds: 50),
  child: Icon(FluentIcons.add),
)
```

> **Dart Tips语法小贴士**
>
> 在 Dart 中，Duration 类用于表示时间间隔。可以使用以下构造函数：
>
> ```dart
> Duration(days: 1, hours: 2, minutes: 30, seconds: 15, milliseconds: 500, microseconds: 100)
> ```

---

## 第4章 输入组件

### 4.1 TextBox

TextBox 是 Fluent UI 的文本输入框组件。

#### 4.1.1 基础用法

```dart
TextBox(
  placeholder: '请输入文本',
  onChanged: (value) => print(value),
)
```

#### 4.1.2 核心属性详解

**1. 文本相关**

| 属性名            | 类型                      | 默认值 | 用途说明     |
| ----------------- | ------------------------- | ------ | ------------ |
| controller        | TextEditingController?    | null   | 文本控制器   |
| placeholder       | String?                   | null   | 占位提示文本 |
| placeholderStyle  | TextStyle?                | null   | 占位文本样式 |
| onChanged         | ValueChanged<String>?     | null   | 文本变化回调 |
| onSubmitted       | ValueChanged<String>?     | null   | 提交回调     |
| onEditingComplete | VoidCallback?             | null   | 编辑完成回调 |
| inputFormatters   | List<TextInputFormatter>? | null   | 输入格式化器 |

**2. 外观相关**

| 属性名               | 类型                  | 默认值          | 用途说明         |
| -------------------- | --------------------- | --------------- | ---------------- |
| style                | TextStyle?            | null            | 文本样式         |
| textAlign            | TextAlign             | TextAlign.start | 文本对齐         |
| textAlignVertical    | TextAlignVertical?    | null            | 垂直对齐         |
| expands              | bool                  | false           | 是否扩展填满     |
| maxLines             | int?                  | 1               | 最大行数         |
| minLines             | int?                  | null            | 最小行数         |
| maxLength            | int?                  | null            | 最大字符数       |
| maxLengthEnforcement | MaxLengthEnforcement? | null            | 最大长度强制执行 |

**3. 键盘相关**

| 属性名             | 类型               | 默认值                  | 用途说明     |
| ------------------ | ------------------ | ----------------------- | ------------ |
| keyboardType       | TextInputType?     | null                    | 键盘类型     |
| textInputAction    | TextInputAction?   | null                    | 键盘操作按钮 |
| textCapitalization | TextCapitalization | TextCapitalization.none | 文本大写方式 |
| obscureText        | bool               | false                   | 是否隐藏文本 |
| autocorrect        | bool               | true                    | 是否自动纠正 |
| autofocus          | bool               | false                   | 是否自动聚焦 |
| enabled            | bool               | true                    | 是否启用     |
| readOnly           | bool               | false                   | 是否只读     |

**4. 前缀后缀**

| 属性名      | 类型                  | 默认值                       | 用途说明     |
| ----------- | --------------------- | ---------------------------- | ------------ |
| prefix      | Widget?               | null                         | 前缀组件     |
| prefixMode  | OverlayVisibilityMode | OverlayVisibilityMode.always | 前缀显示模式 |
| suffix      | Widget?               | null                         | 后缀组件     |
| suffixMode  | OverlayVisibilityMode | OverlayVisibilityMode.always | 后缀显示模式 |
| leadingIcon | Widget?               | null                         | 前置图标     |

#### 4.1.3 完整示例

```dart
class LoginForm extends StatelessWidget {
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 用户名输入框
        TextBox(
          controller: _usernameController,
          header: '用户名',
          placeholder: '请输入用户名',
          leadingIcon: Icon(FluentIcons.contact),
          textInputAction: TextInputAction.next,
        ),
        SizedBox(height: 16),

        // 密码输入框
        TextBox(
          controller: _passwordController,
          header: '密码',
          placeholder: '请输入密码',
          leadingIcon: Icon(FluentIcons.lock),
          obscureText: true,
          suffix: IconButton(
            icon: Icon(FluentIcons.red_eye),
            onPressed: () {
              // 切换密码可见性
            },
          ),
          textInputAction: TextInputAction.done,
        ),
        SizedBox(height: 24),

        // 多行文本框
        TextBox(
          maxLines: 5,
          minLines: 3,
          placeholder: '请输入描述',
        ),
      ],
    );
  }
}
```

---

### 4.2 AutoSuggestBox

AutoSuggestBox 是自动建议输入框，根据用户输入提供建议列表。

#### 4.2.1 核心属性

| 属性名             | 类型                                 | 默认值 | 用途说明           |
| ------------------ | ------------------------------------ | ------ | ------------------ |
| items              | List<AutoSuggestBoxItem<T>>          | -      | 建议项列表（必传） |
| onSelected         | ValueChanged<AutoSuggestBoxItem<T>>? | null   | 选中回调           |
| onChanged          | ValueChanged<String>?                | null   | 文本变化回调       |
| controller         | TextEditingController?               | null   | 控制器             |
| placeholder        | String?                              | null   | 占位文本           |
| placeholderStyle   | TextStyle?                           | null   | 占位文本样式       |
| style              | TextStyle?                           | null   | 文本样式           |
| textInputAction    | TextInputAction?                     | null   | 键盘操作           |
| keyboardType       | TextInputType?                       | null   | 键盘类型           |
| autofocus          | bool                                 | false  | 是否自动聚焦       |
| enabled            | bool                                 | true   | 是否启用           |
| focusNode          | FocusNode?                           | null   | 焦点节点           |
| leadingIcon        | Widget?                              | null   | 前置图标           |
| clearButtonEnabled | bool                                 | true   | 是否启用清除按钮   |

#### 4.2.2 完整示例

```dart
class AutoSuggestBoxDemo extends StatefulWidget {
  @override
  _AutoSuggestBoxDemoState createState() => _AutoSuggestBoxDemoState();
}

class _AutoSuggestBoxDemoState extends State<AutoSuggestBoxDemo> {
  final List<AutoSuggestBoxItem<String>> _items = [
    AutoSuggestBoxItem(value: 'Apple', label: 'Apple'),
    AutoSuggestBoxItem(value: 'Banana', label: 'Banana'),
    AutoSuggestBoxItem(value: 'Cherry', label: 'Cherry'),
    AutoSuggestBoxItem(value: 'Date', label: 'Date'),
    AutoSuggestBoxItem(value: 'Elderberry', label: 'Elderberry'),
  ];

  String _selected = '';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        AutoSuggestBox<String>(
          items: _items,
          onSelected: (item) {
            setState(() => _selected = item.value);
          },
          placeholder: '搜索水果',
          leadingIcon: Icon(FluentIcons.search),
        ),
        if (_selected.isNotEmpty)
          Text('选中: $_selected'),
      ],
    );
  }
}
```

---

### 4.3 ComboBox

ComboBox 是下拉选择框组件。

#### 4.3.1 核心属性

| 属性名              | 类型                  | 默认值 | 用途说明         |
| ------------------- | --------------------- | ------ | ---------------- |
| items               | List<ComboBoxItem<T>> | -      | 选项列表（必传） |
| value               | T?                    | null   | 当前值           |
| onChanged           | ValueChanged<T?>?     | null   | 值变化回调       |
| placeholder         | Widget?               | null   | 占位组件         |
| disabledPlaceholder | Widget?               | null   | 禁用时的占位组件 |
| isExpanded          | bool                  | true   | 是否扩展填满     |
| elevation           | int                   | 8      | 阴影高度         |
| focusColor          | Color?                | null   | 焦点颜色         |
| focusNode           | FocusNode?            | null   | 焦点节点         |
| autofocus           | bool                  | false  | 是否自动聚焦     |

#### 4.3.2 完整示例

```dart
class ComboBoxDemo extends StatefulWidget {
  @override
  _ComboBoxDemoState createState() => _ComboBoxDemoState();
}

class _ComboBoxDemoState extends State<ComboBoxDemo> {
  String? _selectedValue;

  @override
  Widget build(BuildContext context) {
    return ComboBox<String>(
      placeholder: Text('请选择'),
      value: _selectedValue,
      items: [
        ComboBoxItem(
          value: 'option1',
          child: Text('选项 1'),
        ),
        ComboBoxItem(
          value: 'option2',
          child: Text('选项 2'),
        ),
        ComboBoxItem(
          value: 'option3',
          child: Text('选项 3'),
        ),
      ],
      onChanged: (value) {
        setState(() => _selectedValue = value);
      },
    );
  }
}
```

---

### 4.4 Checkbox

Checkbox 是复选框组件。

#### 4.4.1 核心属性

| 属性名        | 类型                 | 默认值 | 用途说明     |
| ------------- | -------------------- | ------ | ------------ |
| checked       | bool?                | false  | 当前值       |
| onChanged     | ValueChanged<bool?>? | null   | 状态变化回调 |
| style         | CheckboxThemeData?   | null   | 样式         |
| content       | Widget?              | null   | 内容         |
| semanticLabel | String?              | null   | 语义标签     |

#### 4.4.2 完整示例

```dart
Checkbox(
  checked: _isChecked,
  onChanged: (value) => setState(() => _isChecked = value),
  content: Text('我同意条款'),
)
```

---

### 4.5 ToggleSwitch

ToggleSwitch 是切换开关组件。

#### 4.5.1 核心属性

| 属性名        | 类型                   | 默认值 | 用途说明     |
| ------------- | ---------------------- | ------ | ------------ |
| checked       | bool                   | false  | 当前状态     |
| onChanged     | ValueChanged<bool>?    | null   | 状态变化回调 |
| style         | ToggleSwitchThemeData? | null   | 样式         |
| content       | Widget?                | null   | 内容         |
| semanticLabel | String?                | null   | 语义标签     |

#### 4.5.2 完整示例

```dart
ToggleSwitch(
  checked: _isEnabled,
  onChanged: (value) => setState(() => _isEnabled = value),
  content: Text('启用通知'),
)
```

---

### 4.6 RadioButton

RadioButton 是单选按钮组件。

#### 4.6.1 核心属性

| 属性名        | 类型                  | 默认值 | 用途说明     |
| ------------- | --------------------- | ------ | ------------ |
| checked       | bool                  | false  | 是否选中     |
| onChanged     | ValueChanged<bool>?   | null   | 状态变化回调 |
| style         | RadioButtonThemeData? | null   | 样式         |
| content       | Widget?               | null   | 内容         |
| semanticLabel | String?               | null   | 语义标签     |

#### 4.6.2 完整示例

```dart
class RadioButtonDemo extends StatefulWidget {
  @override
  _RadioButtonDemoState createState() => _RadioButtonDemoState();
}

class _RadioButtonDemoState extends State<RadioButtonDemo> {
  String _selectedOption = 'option1';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        RadioButton(
          checked: _selectedOption == 'option1',
          onChanged: (value) => setState(() => _selectedOption = 'option1'),
          content: Text('选项 1'),
        ),
        RadioButton(
          checked: _selectedOption == 'option2',
          onChanged: (value) => setState(() => _selectedOption = 'option2'),
          content: Text('选项 2'),
        ),
        RadioButton(
          checked: _selectedOption == 'option3',
          onChanged: (value) => setState(() => _selectedOption = 'option3'),
          content: Text('选项 3'),
        ),
      ],
    );
  }
}
```

---

### 4.7 Slider

Slider 是滑块组件。

#### 4.7.1 核心属性

| 属性名        | 类型                  | 默认值 | 用途说明           |
| ------------- | --------------------- | ------ | ------------------ |
| value         | double                | -      | 当前值（必传）     |
| onChanged     | ValueChanged<double>? | -      | 值变化回调（必传） |
| onChangeStart | ValueChanged<double>? | null   | 开始拖动回调       |
| onChangeEnd   | ValueChanged<double>? | null   | 结束拖动回调       |
| min           | double                | 0.0    | 最小值             |
| max           | double                | 1.0    | 最大值             |
| divisions     | int?                  | null   | 分段数             |
| label         | String?               | null   | 标签               |
| style         | SliderThemeData?      | null   | 样式               |
| vertical      | bool                  | false  | 是否垂直           |

#### 4.7.2 完整示例

```dart
Slider(
  value: _volume,
  min: 0,
  max: 100,
  divisions: 100,
  label: '${_volume.toInt()}%',
  onChanged: (value) => setState(() => _volume = value),
)
```

---

## 第5章 选择器组件

### 5.1 DatePicker

DatePicker 是日期选择器组件。

#### 5.1.1 核心属性

| 属性名     | 类型                    | 默认值 | 用途说明     |
| ---------- | ----------------------- | ------ | ------------ |
| selected   | DateTime?               | null   | 选中的日期   |
| onChanged  | ValueChanged<DateTime>? | null   | 日期变化回调 |
| startYear  | int                     | 1970   | 起始年份     |
| endYear    | int                     | 2050   | 结束年份     |
| locale     | Locale?                 | null   | 语言区域     |
| fieldOrder | List<DatePickerField>?  | null   | 字段顺序     |
| fieldFlex  | List<int>?              | null   | 字段弹性     |

#### 5.1.2 完整示例

```dart
DatePicker(
  selected: _selectedDate,
  onChanged: (date) => setState(() => _selectedDate = date),
  startYear: 2000,
  endYear: 2030,
)
```

---

### 5.2 TimePicker

TimePicker 是时间选择器组件。

#### 5.2.1 核心属性

| 属性名     | 类型                     | 默认值       | 用途说明     |
| ---------- | ------------------------ | ------------ | ------------ |
| selected   | TimeOfDay?               | null         | 选中的时间   |
| onChanged  | ValueChanged<TimeOfDay>? | null         | 时间变化回调 |
| hourFormat | HourFormat               | HourFormat.h | 小时格式     |
| locale     | Locale?                  | null         | 语言区域     |

**HourFormat 枚举值：**

| 值  | 说明     |
| --- | -------- |
| h   | 12小时制 |
| HH  | 24小时制 |

#### 5.2.2 完整示例

```dart
TimePicker(
  selected: _selectedTime,
  onChanged: (time) => setState(() => _selectedTime = time),
  hourFormat: HourFormat.HH,
)
```

---

### 5.3 ColorPicker

ColorPicker 是颜色选择器组件。

#### 5.3.1 核心属性

| 属性名      | 类型                 | 默认值 | 用途说明             |
| ----------- | -------------------- | ------ | -------------------- |
| color       | Color                | -      | 当前颜色（必传）     |
| onChanged   | ValueChanged<Color>  | -      | 颜色变化回调（必传） |
| onApplied   | ValueChanged<Color>? | null   | 应用回调             |
| onCancelled | VoidCallback?        | null   | 取消回调             |

#### 5.3.2 完整示例

```dart
ColorPicker(
  color: _selectedColor,
  onChanged: (color) => setState(() => _selectedColor = color),
  onApplied: (color) {
    // 应用颜色
  },
  onCancelled: () {
    // 取消选择
  },
)
```

---

## 第6章 弹窗与对话框组件

### 6.1 ContentDialog

ContentDialog 是 Fluent UI 的内容对话框组件。

#### 6.1.1 基础用法

```dart
showDialog(
  context: context,
  builder: (context) => ContentDialog(
    title: Text('确认删除'),
    content: Text('确定要删除此项目吗？'),
    actions: [
      Button(
        child: Text('取消'),
        onPressed: () => Navigator.pop(context),
      ),
      FilledButton(
        child: Text('删除'),
        onPressed: () {
          // 删除操作
          Navigator.pop(context);
        },
      ),
    ],
  ),
);
```

#### 6.1.2 核心属性

| 属性名        | 类型                    | 默认值 | 用途说明       |
| ------------- | ----------------------- | ------ | -------------- |
| title         | Widget?                 | null   | 标题           |
| content       | Widget?                 | null   | 内容           |
| actions       | List<Widget>?           | null   | 操作按钮       |
| style         | ContentDialogThemeData? | null   | 样式           |
| constraints   | BoxConstraints?         | null   | 尺寸约束       |
| scrollContent | bool                    | true   | 内容是否可滚动 |

---

### 6.2 Flyout

Flyout 是弹出层组件，用于显示上下文菜单、提示信息等。

#### 6.2.1 基础用法

```dart
FlyoutTarget(
  controller: _flyoutController,
  child: Button(
    child: Text('显示 Flyout'),
    onPressed: () {
      _flyoutController.showFlyout(
        builder: (context) {
          return FlyoutContent(
            child: Text('这是 Flyout 内容'),
          );
        },
      );
    },
  ),
)
```

#### 6.2.2 FlyoutController

| 属性名     | 类型                                                                                                          | 用途说明    |
| ---------- | ------------------------------------------------------------------------------------------------------------- | ----------- |
| open       | bool                                                                                                          | 是否打开    |
| showFlyout | Future<T?> Function<T>({required FlyoutBuilder builder, Object? barrierDismissible, VoidCallback? onDismiss}) | 显示 Flyout |
| close      | void Function()                                                                                               | 关闭 Flyout |

#### 6.2.3 FlyoutContent

| 属性名      | 类型            | 默认值                   | 用途说明 |
| ----------- | --------------- | ------------------------ | -------- |
| child       | Widget?         | null                     | 子组件   |
| constraints | BoxConstraints? | null                     | 尺寸约束 |
| shape       | ShapeBorder?    | null                     | 形状     |
| color       | Color?          | null                     | 颜色     |
| padding     | EdgeInsets      | const EdgeInsets.all(12) | 内边距   |
| elevation   | double          | 8                        | 阴影高度 |

#### 6.2.4 完整示例

```dart
class FlyoutDemo extends StatefulWidget {
  @override
  _FlyoutDemoState createState() => _FlyoutDemoState();
}

class _FlyoutDemoState extends State<FlyoutDemo> {
  final _flyoutController = FlyoutController();

  @override
  void dispose() {
    _flyoutController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FlyoutTarget(
      controller: _flyoutController,
      child: Button(
        child: Text('显示菜单'),
        onPressed: () {
          _flyoutController.showFlyout(
            builder: (context) {
              return MenuFlyout(
                items: [
                  MenuFlyoutItem(
                    leading: Icon(FluentIcons.copy),
                    text: Text('复制'),
                    onPressed: () {
                      _flyoutController.close();
                      // 复制操作
                    },
                  ),
                  MenuFlyoutItem(
                    leading: Icon(FluentIcons.cut),
                    text: Text('剪切'),
                    onPressed: () {
                      _flyoutController.close();
                      // 剪切操作
                    },
                  ),
                  MenuFlyoutSeparator(),
                  MenuFlyoutItem(
                    leading: Icon(FluentIcons.delete),
                    text: Text('删除'),
                    onPressed: () {
                      _flyoutController.close();
                      // 删除操作
                    },
                  ),
                ],
              );
            },
          );
        },
      ),
    );
  }
}
```

---

### 6.3 MenuFlyout

MenuFlyout 是菜单弹出层组件。

#### 6.3.1 核心属性

| 属性名      | 类型                     | 默认值                  | 用途说明           |
| ----------- | ------------------------ | ----------------------- | ------------------ |
| items       | List<MenuFlyoutItemBase> | -                       | 菜单项列表（必传） |
| constraints | BoxConstraints?          | null                    | 尺寸约束           |
| shape       | ShapeBorder?             | null                    | 形状               |
| color       | Color?                   | null                    | 颜色               |
| padding     | EdgeInsets               | const EdgeInsets.all(4) | 内边距             |
| elevation   | double                   | 8                       | 阴影高度           |

#### 6.3.2 MenuFlyoutItem

| 属性名    | 类型         | 默认值 | 用途说明         |
| --------- | ------------ | ------ | ---------------- |
| text      | Widget       | -      | 文本（必传）     |
| onPressed | VoidCallback | -      | 点击回调（必传） |
| leading   | Widget?      | null   | 前置图标         |
| trailing  | Widget?      | null   | 后置图标         |
| selected  | bool         | false  | 是否选中         |

#### 6.3.3 MenuFlyoutSubItem

```dart
MenuFlyoutSubItem(
  text: Text('更多选项'),
  items: [
    MenuFlyoutItem(
      text: Text('子选项 1'),
      onPressed: () {},
    ),
    MenuFlyoutItem(
      text: Text('子选项 2'),
      onPressed: () {},
    ),
  ],
)
```

---

### 6.4 Tooltip

Tooltip 是工具提示组件。

#### 6.4.1 核心属性

| 属性名       | 类型              | 默认值 | 用途说明         |
| ------------ | ----------------- | ------ | ---------------- |
| message      | String            | -      | 提示文本（必传） |
| child        | Widget            | -      | 子组件（必传）   |
| style        | TooltipThemeData? | null   | 样式             |
| waitDuration | Duration?         | null   | 等待显示时长     |
| showDuration | Duration?         | null   | 显示时长         |

#### 6.4.2 使用示例

```dart
Tooltip(
  message: '这是一个提示',
  child: Icon(FluentIcons.info),
)
```

---

## 第7章 列表组件

### 7.1 ListTile

ListTile 是 Fluent UI 的列表项组件。

#### 7.1.1 核心属性

| 属性名            | 类型                | 默认值 | 用途说明     |
| ----------------- | ------------------- | ------ | ------------ |
| leading           | Widget?             | null   | 前置组件     |
| title             | Widget?             | null   | 标题         |
| subtitle          | Widget?             | null   | 副标题       |
| trailing          | Widget?             | null   | 后置组件     |
| onPressed         | VoidCallback?       | null   | 点击回调     |
| onSelectionChange | ValueChanged<bool>? | null   | 选择变化回调 |
| selected          | bool                | false  | 是否选中     |
| enabled           | bool                | true   | 是否启用     |

#### 7.1.2 完整示例

```dart
ListTile(
  leading: Icon(FluentIcons.document),
  title: Text('文档名称'),
  subtitle: Text('最后修改: 2024-01-01'),
  trailing: Icon(FluentIcons.more_vertical),
  onPressed: () {
    print('点击了列表项');
  },
)
```

---

### 7.2 ListView

Fluent UI 中的 ListView 与 Flutter 原生的 ListView 用法相同，但会应用 Fluent 主题样式。

---

### 7.3 TreeView

TreeView 是树形列表组件。

#### 7.3.1 核心属性

| 属性名             | 类型                              | 默认值                     | 用途说明           |
| ------------------ | --------------------------------- | -------------------------- | ------------------ |
| items              | List<TreeViewItem>                | -                          | 树节点列表（必传） |
| selectionMode      | TreeViewSelectionMode             | TreeViewSelectionMode.none | 选择模式           |
| onItemInvoked      | ValueChanged<TreeViewItem>?       | null                       | 节点调用回调       |
| onSelectionChanged | ValueChanged<List<TreeViewItem>>? | null                       | 选择变化回调       |
| onSecondaryTap     | ValueChanged<TreeViewItem>?       | null                       | 右键点击回调       |
| shrinkWrap         | bool                              | false                      | 是否收缩包裹       |
| scrollController   | ScrollController?                 | null                       | 滚动控制器         |
| primary            | bool?                             | null                       | 是否主滚动         |

**TreeViewSelectionMode 枚举值：**

| 值       | 说明     |
| -------- | -------- |
| none     | 不可选择 |
| single   | 单选     |
| multiple | 多选     |

#### 7.3.2 完整示例

```dart
TreeView(
  selectionMode: TreeViewSelectionMode.single,
  items: [
    TreeViewItem(
      content: Text('文件夹 1'),
      value: 'folder1',
      children: [
        TreeViewItem(
          content: Text('文件 1-1'),
          value: 'file1-1',
        ),
        TreeViewItem(
          content: Text('文件 1-2'),
          value: 'file1-2',
        ),
      ],
    ),
    TreeViewItem(
      content: Text('文件夹 2'),
      value: 'folder2',
      children: [
        TreeViewItem(
          content: Text('文件 2-1'),
          value: 'file2-1',
        ),
      ],
    ),
  ],
  onItemInvoked: (item) {
    print('点击了: ${item.value}');
  },
  onSelectionChanged: (selectedItems) {
    print('选中了: ${selectedItems.map((e) => e.value).join(', ')}');
  },
)
```

---

## 第8章 进度组件

### 8.1 ProgressBar

ProgressBar 是线性进度条组件。

#### 8.1.1 核心属性

| 属性名          | 类型    | 默认值 | 用途说明                        |
| --------------- | ------- | ------ | ------------------------------- |
| value           | double? | null   | 当前进度值（null 为不确定进度） |
| backgroundColor | Color?  | null   | 背景颜色                        |
| activeColor     | Color?  | null   | 激活颜色                        |
| semanticsLabel  | String? | null   | 语义标签                        |
| semanticsValue  | String? | null   | 语义值                          |

#### 8.1.2 完整示例

```dart
Column(
  children: [
    // 不确定进度
    ProgressBar(),
    SizedBox(height: 16),

    // 确定进度
    ProgressBar(value: 0.7),
    SizedBox(height: 16),

    // 自定义颜色
    ProgressBar(
      value: 0.5,
      activeColor: Colors.green,
    ),
  ],
)
```

---

### 8.2 ProgressRing

ProgressRing 是环形进度条组件。

#### 8.2.1 核心属性

| 属性名          | 类型    | 默认值 | 用途说明                        |
| --------------- | ------- | ------ | ------------------------------- |
| value           | double? | null   | 当前进度值（null 为不确定进度） |
| backgroundColor | Color?  | null   | 背景颜色                        |
| activeColor     | Color?  | null   | 激活颜色                        |
| strokeWidth     | double  | 4.0    | 线条宽度                        |
| semanticsLabel  | String? | null   | 语义标签                        |
| semanticsValue  | String? | null   | 语义值                          |

#### 8.2.2 完整示例

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: [
    // 不确定进度
    ProgressRing(),

    // 确定进度
    ProgressRing(value: 0.75),

    // 自定义样式
    ProgressRing(
      value: 0.5,
      strokeWidth: 6,
      activeColor: Colors.orange,
    ),
  ],
)
```

---

## 第9章 视觉效果组件

### 9.1 Acrylic

Acrylic 是 Fluent Design 特有的半透明材质效果，可以创建具有深度感的背景。

#### 9.1.1 核心属性

| 属性名          | 类型           | 默认值 | 用途说明   |
| --------------- | -------------- | ------ | ---------- |
| child           | Widget?        | null   | 子组件     |
| decoration      | BoxDecoration? | null   | 装饰       |
| luminosityAlpha | double         | 0.8    | 亮度透明度 |
| tint            | Color?         | null   | 着色       |
| tintAlpha       | double         | 0.8    | 着色透明度 |
| blurAmount      | double         | 30.0   | 模糊量     |

#### 9.1.2 完整示例

```dart
Acrylic(
  luminosityAlpha: 0.8,
  tint: Colors.blue.withOpacity(0.2),
  blurAmount: 30,
  child: Container(
    width: 300,
    height: 200,
    padding: EdgeInsets.all(16),
    child: Text('Acrylic 效果'),
  ),
)
```

---

### 9.2 Mica

Mica 是 Windows 11 引入的一种材质效果，比 Acrylic 更轻量。

```dart
Mica(
  child: Container(
    width: double.infinity,
    height: double.infinity,
    child: Text('Mica 效果'),
  ),
)
```

---

### 9.3 FocusBorder

FocusBorder 是焦点边框组件，当子组件获得焦点时显示边框。

#### 9.3.1 核心属性

| 属性名  | 类型            | 默认值 | 用途说明       |
| ------- | --------------- | ------ | -------------- |
| child   | Widget          | -      | 子组件（必传） |
| focused | bool            | false  | 是否聚焦       |
| style   | FocusThemeData? | null   | 样式           |

#### 9.3.2 完整示例

```dart
FocusBorder(
  focused: _isFocused,
  child: TextBox(
    onFocusChange: (focused) => setState(() => _isFocused = focused),
  ),
)
```

---

## 第10章 信息展示组件

### 10.1 InfoBar

InfoBar 是信息栏组件，用于显示警告、错误、成功等消息。

#### 10.1.1 核心属性

| 属性名   | 类型            | 默认值               | 用途说明     |
| -------- | --------------- | -------------------- | ------------ |
| title    | Widget?         | null                 | 标题         |
| content  | Widget?         | null                 | 内容         |
| action   | Widget?         | null                 | 操作按钮     |
| severity | InfoBarSeverity | InfoBarSeverity.info | 严重程度     |
| isLong   | bool            | false                | 是否为长消息 |
| onClose  | VoidCallback?   | null                 | 关闭回调     |

**InfoBarSeverity 枚举值：**

| 值      | 说明 |
| ------- | ---- |
| info    | 信息 |
| warning | 警告 |
| error   | 错误 |
| success | 成功 |

#### 10.1.2 完整示例

```dart
Column(
  children: [
    InfoBar(
      title: Text('信息'),
      content: Text('这是一条普通信息'),
      severity: InfoBarSeverity.info,
    ),
    InfoBar(
      title: Text('警告'),
      content: Text('这是一条警告信息'),
      severity: InfoBarSeverity.warning,
    ),
    InfoBar(
      title: Text('错误'),
      content: Text('这是一条错误信息'),
      severity: InfoBarSeverity.error,
    ),
    InfoBar(
      title: Text('成功'),
      content: Text('操作成功完成'),
      severity: InfoBarSeverity.success,
      action: Button(
        child: Text('撤销'),
        onPressed: () {},
      ),
    ),
  ],
)
```

---

### 10.2 InfoHeader

InfoHeader 是信息头部组件，用于在列表或表单区域前显示说明信息。

#### 10.2.1 核心属性

| 属性名  | 类型    | 默认值 | 用途说明     |
| ------- | ------- | ------ | ------------ |
| title   | Widget  | -      | 标题（必传） |
| body    | Widget? | null   | 正文         |
| isLarge | bool    | false  | 是否为大尺寸 |
| source  | Widget? | null   | 来源组件     |

#### 10.2.2 完整示例

```dart
InfoHeader(
  title: Text('账户设置'),
  body: Text('管理您的账户信息和偏好设置'),
  isLarge: true,
)
```

---

### 10.3 Badge

Badge 是徽章组件，用于显示计数或状态标记。

```dart
Badge(
  child: Icon(FluentIcons.mail),
  source: Text('5'),
)
```

---

### 10.4 Chip

Chip 是标签组件，用于显示分类、标签等信息。

#### 10.4.1 核心属性

| 属性名          | 类型          | 默认值 | 用途说明       |
| --------------- | ------------- | ------ | -------------- |
| child           | Widget        | -      | 子组件（必传） |
| onPressed       | VoidCallback? | null   | 点击回调       |
| onDeleted       | VoidCallback? | null   | 删除回调       |
| selected        | bool          | false  | 是否选中       |
| shape           | ShapeBorder?  | null   | 形状           |
| backgroundColor | Color?        | null   | 背景颜色       |

#### 10.4.2 完整示例

```dart
Wrap(
  spacing: 8,
  children: [
    Chip(
      child: Text('标签 1'),
      onDeleted: () {},
    ),
    Chip(
      child: Text('标签 2'),
      selected: true,
      onPressed: () {},
    ),
    Chip(
      child: Text('标签 3'),
    ),
  ],
)
```

---

## 第11章 移动端组件

### 11.1 PillButtonBar

PillButtonBar 是胶囊按钮栏组件，适用于移动设备的导航。

#### 11.1.1 核心属性

| 属性名    | 类型                    | 默认值 | 用途说明           |
| --------- | ----------------------- | ------ | ------------------ |
| items     | List<PillButtonBarItem> | -      | 按钮项列表（必传） |
| selected  | int                     | 0      | 当前选中索引       |
| onChanged | ValueChanged<int>?      | null   | 索引变化回调       |

#### 11.1.2 完整示例

```dart
PillButtonBar(
  selected: _selectedIndex,
  onChanged: (index) => setState(() => _selectedIndex = index),
  items: [
    PillButtonBarItem(text: Text('全部')),
    PillButtonBarItem(text: Text('未读')),
    PillButtonBarItem(text: Text('已读')),
  ],
)
```

---

### 11.2 Snackbar

Snackbar 是底部提示条组件。

#### 11.2.1 核心属性

| 属性名   | 类型    | 默认值 | 用途说明     |
| -------- | ------- | ------ | ------------ |
| content  | Widget  | -      | 内容（必传） |
| action   | Widget? | null   | 操作按钮     |
| extended | bool    | false  | 是否扩展     |

#### 11.2.2 使用示例

```dart
showSnackbar(
  context,
  Snackbar(
    content: Text('操作成功'),
    action: TextButton(
      child: Text('撤销'),
      onPressed: () {},
    ),
  ),
);
```

---

## 第12章 主题与样式

### 12.1 Colors

Fluent UI 提供了 Fluent Design 系统颜色。

#### 12.1.1 基础颜色

| 颜色           | 说明   |
| -------------- | ------ |
| Colors.white   | 白色   |
| Colors.black   | 黑色   |
| Colors.grey    | 灰色   |
| Colors.yellow  | 黄色   |
| Colors.orange  | 橙色   |
| Colors.red     | 红色   |
| Colors.magenta | 洋红色 |
| Colors.purple  | 紫色   |
| Colors.blue    | 蓝色   |
| Colors.teal    | 青色   |
| Colors.green   | 绿色   |

#### 12.1.2 灰度色阶

```dart
Colors.grey[10]  // 最浅
Colors.grey[20]
Colors.grey[30]
Colors.grey[40]
Colors.grey[50]
Colors.grey[60]
Colors.grey[70]
Colors.grey[80]
Colors.grey[90]  // 最深
Colors.grey[100] // 黑色
Colors.grey[110] // 更深的黑色
Colors.grey[120] // 最深的黑色
Colors.grey[130] // 最深
Colors.grey[140] // 最深
Colors.grey[150] // 最深
Colors.grey[160] // 最深
```

#### 12.1.3 强调色

```dart
// 将颜色转换为强调色
final accentColor = Colors.blue.toAccentColor();

// 使用强调色的不同色调
accentColor.lightest
accentColor.lighter
accentColor.light
accentColor.normal  // 默认
accentColor.dark
accentColor.darker
accentColor.darkest
```

---

### 12.2 FluentIcons

Fluent UI 提供了丰富的 Fluent 风格图标。

#### 12.2.1 常用图标

| 图标                      | 名称          | 用途     |
| ------------------------- | ------------- | -------- |
| FluentIcons.home          | home          | 首页     |
| FluentIcons.search        | search        | 搜索     |
| FluentIcons.settings      | settings      | 设置     |
| FluentIcons.person        | person        | 个人     |
| FluentIcons.add           | add           | 添加     |
| FluentIcons.delete        | delete        | 删除     |
| FluentIcons.edit          | edit          | 编辑     |
| FluentIcons.save          | save          | 保存     |
| FluentIcons.cancel        | cancel        | 取消     |
| FluentIcons.check_mark    | check_mark    | 确认     |
| FluentIcons.close         | close         | 关闭     |
| FluentIcons.more          | more          | 更多     |
| FluentIcons.back          | back          | 返回     |
| FluentIcons.forward       | forward       | 前进     |
| FluentIcons.refresh       | refresh       | 刷新     |
| FluentIcons.copy          | copy          | 复制     |
| FluentIcons.cut           | cut           | 剪切     |
| FluentIcons.paste         | paste         | 粘贴     |
| FluentIcons.undo          | undo          | 撤销     |
| FluentIcons.redo          | redo          | 重做     |
| FluentIcons.bold          | bold          | 粗体     |
| FluentIcons.italic        | italic        | 斜体     |
| FluentIcons.underline     | underline     | 下划线   |
| FluentIcons.lock          | lock          | 锁定     |
| FluentIcons.unlock        | unlock        | 解锁     |
| FluentIcons.mail          | mail          | 邮件     |
| FluentIcons.phone         | phone         | 电话     |
| FluentIcons.camera        | camera        | 相机     |
| FluentIcons.picture       | picture       | 图片     |
| FluentIcons.video         | video         | 视频     |
| FluentIcons.music         | music         | 音乐     |
| FluentIcons.play          | play          | 播放     |
| FluentIcons.pause         | pause         | 暂停     |
| FluentIcons.stop          | stop          | 停止     |
| FluentIcons.previous      | previous      | 上一首   |
| FluentIcons.next          | next          | 下一首   |
| FluentIcons.volume_0      | volume_0      | 静音     |
| FluentIcons.volume_1      | volume_1      | 低音量   |
| FluentIcons.volume_2      | volume_2      | 中音量   |
| FluentIcons.volume_3      | volume_3      | 高音量   |
| FluentIcons.wifi          | wifi          | Wi-Fi    |
| FluentIcons.bluetooth     | bluetooth     | 蓝牙     |
| FluentIcons.battery_0     | battery_0     | 电池空   |
| FluentIcons.battery_1     | battery_1     | 电池低   |
| FluentIcons.battery_2     | battery_2     | 电池中   |
| FluentIcons.battery_3     | battery_3     | 电池高   |
| FluentIcons.battery_4     | battery_4     | 电池满   |
| FluentIcons.calendar      | calendar      | 日历     |
| FluentIcons.clock         | clock         | 时钟     |
| FluentIcons.location      | location      | 位置     |
| FluentIcons.map_pin       | map_pin       | 地图标记 |
| FluentIcons.cloud         | cloud         | 云       |
| FluentIcons.sun           | sun           | 太阳     |
| FluentIcons.moon          | moon          | 月亮     |
| FluentIcons.star          | star          | 星星     |
| FluentIcons.heart         | heart         | 心形     |
| FluentIcons.flag          | flag          | 旗帜     |
| FluentIcons.tag           | tag           | 标签     |
| FluentIcons.folder        | folder        | 文件夹   |
| FluentIcons.document      | document      | 文档     |
| FluentIcons.file          | file          | 文件     |
| FluentIcons.download      | download      | 下载     |
| FluentIcons.upload        | upload        | 上传     |
| FluentIcons.share         | share         | 分享     |
| FluentIcons.link          | link          | 链接     |
| FluentIcons.external_link | external_link | 外部链接 |
| FluentIcons.filter        | filter        | 筛选     |
| FluentIcons.sort          | sort          | 排序     |
| FluentIcons.view          | view          | 视图     |
| FluentIcons.list          | list          | 列表     |
| FluentIcons.grid          | grid          | 网格     |
| FluentIcons.chart         | chart         | 图表     |
| FluentIcons.print         | print         | 打印     |
| FluentIcons.export        | export        | 导出     |
| FluentIcons.import\_      | import\_      | 导入     |

---

## 附录：fluent_ui 组件速查表

### 应用组件

| 组件名          | 用途         |
| --------------- | ------------ |
| FluentApp       | 应用顶层组件 |
| FluentTheme     | 主题组件     |
| FluentThemeData | 主题数据     |

### 导航组件

| 组件名           | 用途     |
| ---------------- | -------- |
| NavigationView   | 导航视图 |
| NavigationPane   | 导航面板 |
| NavigationAppBar | 应用栏   |
| PaneItem         | 导航项   |
| TabView          | 标签页   |
| BottomNavigation | 底部导航 |

### 按钮组件

| 组件名            | 用途       |
| ----------------- | ---------- |
| Button            | 基础按钮   |
| FilledButton      | 填充按钮   |
| TransparentButton | 透明按钮   |
| IconButton        | 图标按钮   |
| ToggleButton      | 切换按钮   |
| SplitButton       | 分割按钮   |
| HyperlinkButton   | 超链接按钮 |
| RepeatButton      | 重复按钮   |

### 输入组件

| 组件名         | 用途       |
| -------------- | ---------- |
| TextBox        | 文本输入框 |
| AutoSuggestBox | 自动建议框 |
| ComboBox       | 下拉选择框 |
| Checkbox       | 复选框     |
| ToggleSwitch   | 切换开关   |
| RadioButton    | 单选按钮   |
| Slider         | 滑块       |

### 选择器组件

| 组件名      | 用途       |
| ----------- | ---------- |
| DatePicker  | 日期选择器 |
| TimePicker  | 时间选择器 |
| ColorPicker | 颜色选择器 |

### 弹窗组件

| 组件名           | 用途         |
| ---------------- | ------------ |
| ContentDialog    | 内容对话框   |
| Flyout           | 弹出层       |
| FlyoutContent    | 弹出层内容   |
| FlyoutTarget     | 弹出层目标   |
| FlyoutController | 弹出层控制器 |
| MenuFlyout       | 菜单弹出层   |
| MenuFlyoutItem   | 菜单项       |
| Tooltip          | 工具提示     |

### 列表组件

| 组件名       | 用途     |
| ------------ | -------- |
| ListTile     | 列表项   |
| TreeView     | 树形视图 |
| TreeViewItem | 树形项   |

### 进度组件

| 组件名       | 用途       |
| ------------ | ---------- |
| ProgressBar  | 线性进度条 |
| ProgressRing | 环形进度条 |

### 视觉效果组件

| 组件名      | 用途             |
| ----------- | ---------------- |
| Acrylic     | Acrylic 材质效果 |
| Mica        | Mica 材质效果    |
| FocusBorder | 焦点边框         |

### 信息展示组件

| 组件名     | 用途     |
| ---------- | -------- |
| InfoBar    | 信息栏   |
| InfoHeader | 信息头部 |
| Badge      | 徽章     |
| Chip       | 标签     |

### 移动端组件

| 组件名        | 用途       |
| ------------- | ---------- |
| PillButtonBar | 胶囊按钮栏 |
| Snackbar      | 底部提示条 |

### 主题与样式

| 组件/类     | 用途     |
| ----------- | -------- |
| Colors      | 系统颜色 |
| AccentColor | 强调色   |
| FluentIcons | 系统图标 |
| Typography  | 排版     |

---

## Material 与 Fluent UI 组件对照表

| Material                  | Fluent UI        | 说明         |
| ------------------------- | ---------------- | ------------ |
| MaterialApp               | FluentApp        | 应用顶层组件 |
| Scaffold                  | ScaffoldPage     | 页面脚手架   |
| AppBar                    | NavigationAppBar | 应用栏       |
| Drawer                    | NavigationPane   | 侧边导航     |
| BottomNavigationBar       | BottomNavigation | 底部导航     |
| TextButton                | Button           | 文本按钮     |
| ElevatedButton            | FilledButton     | 填充按钮     |
| IconButton                | IconButton       | 图标按钮     |
| Checkbox                  | Checkbox         | 复选框       |
| Radio                     | RadioButton      | 单选按钮     |
| Switch                    | ToggleSwitch     | 切换开关     |
| TextField                 | TextBox          | 文本输入框   |
| DropdownButton            | ComboBox         | 下拉选择框   |
| AlertDialog               | ContentDialog    | 对话框       |
| Tooltip                   | Tooltip          | 工具提示     |
| LinearProgressIndicator   | ProgressBar      | 线性进度条   |
| CircularProgressIndicator | ProgressRing     | 环形进度条   |
| ListTile                  | ListTile         | 列表项       |
| Chip                      | Chip             | 标签         |
| SnackBar                  | Snackbar         | 底部提示条   |
| -                         | NavigationView   | 导航视图     |
| -                         | TabView          | 标签页       |
| -                         | Acrylic          | Acrylic 效果 |
| -                         | Flyout           | 弹出层       |
| -                         | InfoBar          | 信息栏       |
| -                         | AutoSuggestBox   | 自动建议框   |
| -                         | TreeView         | 树形视图     |

---

## 结语

本书详细介绍了 `fluent_ui` 4.13.0 版本的全部组件，从基础应用到复杂交互，从视觉效果到信息展示，涵盖了构建 Windows Fluent Design 风格应用所需的各种组件知识。

`fluent_ui` 是一个非官方但功能完善的 Flutter 包，它为开发者提供了在 Flutter 中实现 Microsoft Fluent Design System 的能力。通过本书的学习，你可以：

1. 理解 Fluent Design 的设计原则和视觉风格
2. 熟练使用各种 Fluent UI 组件
3. 创建具有 Windows 原生风格的应用程序
4. 根据需要自定义主题和样式

希望本书能够帮助读者深入理解 `fluent_ui` 包的使用方法，在实际开发中灵活运用这些组件，构建出美观、流畅、符合 Windows 设计规范的跨平台应用程序。

---

**版本信息**

- fluent_ui 版本：4.13.0
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
