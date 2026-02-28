# Flutter Lints 6.0.0 详解与实战

## 第1章 认识 Flutter Lints

### 1.1 什么是 Flutter Lints

`package:flutter_lints` 是 Flutter 团队官方推荐的静态代码分析规则集合，专门为 Flutter 应用程序、插件和包设计。它建立在 Dart 的 `package:lints` 基础之上，额外添加了 Flutter 特定的 lint 规则，旨在帮助开发者编写高质量、符合 Flutter 最佳实践的代码。

与纯 Dart 的 `package:lints` 不同，`flutter_lints` 专注于 Flutter 框架特有的编码模式和最佳实践，涵盖了 Widget 构建、状态管理、异步操作、性能优化等 Flutter 开发中的核心场景。

#### 1.1.1 Flutter Lints 与 Dart Lints 的关系

`flutter_lints` 采用继承扩展的方式组织规则集：

```
package:lints/core.yaml (33条核心规则)
    ↓ 包含
package:lints/recommended.yaml (88条推荐规则，包含所有核心规则)
    ↓ 继承
package:flutter_lints/flutter.yaml (10条Flutter特定规则 + 88条推荐规则)
```

Flutter 框架小知识

**lints、flutter_lints 与第三方规则集的选择**

在实际项目中，开发者可以根据项目类型选择不同的规则集：

1. **纯 Dart 库/应用**：使用 `package:lints/recommended.yaml`
2. **Flutter 应用/插件**：使用 `package:flutter_lints/flutter.yaml`
3. **更严格的代码规范**：考虑第三方包如 `very_good_analysis` 或 `lint`

选择 `flutter_lints` 的优势在于：

- 官方维护，与 Flutter 版本同步更新
- 规则经过 Flutter 团队精心筛选，平衡了严格性和实用性
- 被 pub.dev 评分系统认可，有助于提升包的质量分数

### 1.2 Flutter Lints 6.0.0 的新特性

Flutter Lints 6.0.0 于 2025 年发布，是该包的重要版本更新，主要包含以下变更：

#### 1.2.1 依赖升级

- **更新 `package:lints` 依赖至 6.0.0**：继承了 `lints` 6.0.0 的所有更新
  - 新增 `strict_top_level_inference`：要求指定顶级类型注解
  - 新增 `unnecessary_underscores`：检测不必要的下划线

#### 1.2.2 最低 SDK 版本要求

- **最低支持 Flutter 3.32 / Dart 3.8**：确保可以使用 Dart 语言的最新特性

#### 1.2.3 移除的规则

6.0.0 版本移除了以下三条规则（详见 GitHub Issue #205）：

| 移除的规则                                   | 移除原因                                                    |
| -------------------------------------------- | ----------------------------------------------------------- |
| `prefer_const_constructors`                  | 在 Flutter 开发中过度限制，某些场景下非 const 构造更灵活    |
| `prefer_const_declarations`                  | 与 `prefer_const_constructors` 类似，移除以减少不必要的限制 |
| `prefer_const_literals_to_create_immutables` | 同上，避免过度使用 const 导致的开发体验问题                 |

Dart Tips 语法小贴士

**关于 const 的权衡**

虽然 const 构造在性能上有优势（可以复用实例、减少重建），但 Flutter 团队在 6.0.0 中移除了强制 const 的规则，原因包括：

1. **开发体验**：在热重载（Hot Reload）时，const 对象不会被重新构建，可能导致 UI 不更新的困惑
2. **灵活性**：某些场景下需要根据运行时条件决定是否使用 const
3. **性能收益有限**：在现代设备上，非 const 构造的性能开销通常可以忽略

开发者仍可以手动启用这些规则，如果项目对性能有极高要求：

```yaml
linter:
  rules:
    prefer_const_constructors: true
    prefer_const_declarations: true
    prefer_const_literals_to_create_immutables: true
```

### 1.3 启用 Flutter Lints

#### 1.3.1 新项目自动启用

从 Flutter 2.3.0 开始，使用 `flutter create` 命令创建的新项目已经自动配置好 `flutter_lints`。项目会包含：

**pubspec.yaml**：

```yaml
dev_dependencies:
  flutter_lints: ^6.0.0
```

**analysis_options.yaml**：

```yaml
# This file configures the analyzer, which statically analyzes Dart code to
# check for errors, warnings, and lints.
#
# The issues identified by the analyzer are surfaced in the UI of Dart-enabled
# IDEs (https://dart.dev/tools#ides-and-editors). The analyzer can also be
# invoked from the command line by running `flutter analyze`.

# The following line activates a set of recommended lints for Flutter apps,
# packages, and plugins designed to encourage good coding practices.
include: package:flutter_lints/flutter.yaml

linter:
  # The lint rules applied to this project can be customized in the
  # section below to disable rules from the `package:flutter_lints/flutter.yaml`
  # included above or to enable additional rules. A list of all available lints
  # and their documentation is published at https://dart.dev/tools/linter-rules.
  #
  # Instead of disabling a lint rule for the entire project in the
  # section below, it can also be suppressed for a single line of code
  # or a specific dart file by using the `// ignore: name_of_lint` and
  # `// ignore_for_file: name_of_lint` syntax on the line or in the file
  # producing the lint.
  rules:
    # avoid_print: false  # Uncomment to disable the `avoid_print` rule
    # prefer_single_quotes: true  # Uncomment to enable the `prefer_single_quotes` rule

# Additional information about this file can be found at
# https://dart.dev/tools/analysis
```

#### 1.3.2 现有项目手动启用

对于旧项目，可以通过以下步骤启用：

**步骤 1：添加依赖**

```bash
flutter pub add dev:flutter_lints
```

或在 `pubspec.yaml` 中手动添加：

```yaml
dev_dependencies:
  flutter_lints: ^6.0.0
```

**步骤 2：创建配置文件**

在项目根目录创建 `analysis_options.yaml`：

```yaml
include: package:flutter_lints/flutter.yaml
```

**步骤 3：运行分析**

```bash
flutter pub get
flutter analyze
```

### 1.4 自定义 Flutter Lints 配置

#### 1.4.1 禁用特定规则

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    avoid_print: false # 允许使用 print（开发阶段常用）
    use_key_in_widget_constructors: false # 如果不需要 Widget key
```

#### 1.4.2 启用额外规则

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # 启用更严格的规则
    always_declare_return_types: true
    prefer_single_quotes: true
    prefer_final_locals: true
    require_trailing_commas: true

    # 性能相关
    avoid_unnecessary_containers: true

    # 异步相关
    unawaited_futures: true
    avoid_catching_errors: true
```

#### 1.4.3 启用严格模式

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  language:
    strict-casts: true # 禁止隐式类型转换
    strict-inference: true # 要求类型推断有明确结果
    strict-raw-types: true # 禁止原始类型（如 List 而不是 List<String>）

linter:
  rules:
    # 额外的严格规则
    avoid_dynamic_calls: true
    avoid_annotating_with_dynamic: false
```

Flutter 框架小知识

**严格模式（Strict Modes）详解**

从 Dart 2.17 开始引入的三种严格模式：

1. **strict-casts**：禁止隐式类型转换

   ```dart
   // 错误：隐式 dynamic 到 String 的转换
   dynamic value = 'hello';
   String text = value;  // 严格模式下报错

   // 正确：显式转换
   String text = value as String;
   ```

2. **strict-inference**：要求类型推断有明确结果

   ```dart
   // 错误：类型不明确
   var list = [];  // 无法推断元素类型

   // 正确：明确类型
   var list = <String>[];
   ```

3. **strict-raw-types**：禁止原始类型

   ```dart
   // 错误：使用原始类型
   List items = <String>['a', 'b'];

   // 正确：指定泛型参数
   List<String> items = <String>['a', 'b'];
   ```

## 第2章 Flutter 特定规则详解

`flutter_lints` 在 `package:lints/recommended.yaml` 的基础上，额外添加了 10 条 Flutter 特定的规则。本章将详细介绍每一条规则。

### 2.1 调试输出类规则

#### 2.1.1 avoid_print

**规则说明**：避免使用 `print` 函数输出调试信息。

**用途**：`print` 语句在开发阶段有用，但不应该出现在生产代码中。Flutter 提供了更灵活的日志 API（`dart:developer` 的 `log` 函数或 `package:logging`）。

**错误示例**：

```dart
// 错误：直接使用 print
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print('Building MyWidget');  // 不应该在生产代码中使用
    return Container();
  }
}

void fetchData() {
  print('Fetching data...');  // 错误
  // ...
  print('Data fetched successfully');  // 错误
}
```

**正确写法**：

```dart
import 'dart:developer' as developer;

class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    developer.log('Building MyWidget', name: 'my.app.widget');
    return Container();
  }
}

// 或者使用日志包
import 'package:logging/logging.dart';

final _logger = Logger('MyService');

void fetchData() {
  _logger.info('Fetching data...');
  // ...
  _logger.info('Data fetched successfully');
}
```

**实战建议**：

在开发阶段，可以临时禁用此规则：

```yaml
linter:
  rules:
    avoid_print: false # 开发阶段允许 print
```

或者使用条件编译，只在调试模式下使用 print：

```dart
void debugPrint(String message) {
  if (kDebugMode) {
    print(message);
  }
}
```

### 2.2 Widget 构建优化类规则

#### 2.2.1 avoid_unnecessary_containers

**规则说明**：避免使用不必要的 `Container`。

**用途**：`Container` 是一个多功能但相对昂贵的 Widget。如果只需要设置边距、对齐或其他单一功能，有更轻量的替代方案。

**错误示例**：

```dart
// 错误：不必要的 Container
return Container(  // 没有设置任何属性，只是包裹子元素
  child: Text('Hello'),
);

// 错误：可以用 SizedBox 替代
return Container(
  width: 100,
  height: 100,
  child: FlutterLogo(),
);

// 错误：可以用 Padding 替代
return Container(
  padding: EdgeInsets.all(16),
  child: Text('Content'),
);

// 错误：可以用 Center 替代
return Container(
  alignment: Alignment.center,
  child: Text('Centered'),
);
```

**正确写法**：

```dart
// 正确：直接使用子元素
return Text('Hello');

// 正确：使用 SizedBox 设置尺寸
return SizedBox(
  width: 100,
  height: 100,
  child: FlutterLogo(),
);

// 正确：使用 Padding 设置边距
return Padding(
  padding: EdgeInsets.all(16),
  child: Text('Content'),
);

// 正确：使用 Center 居中
return Center(
  child: Text('Centered'),
);

// 正确：需要多个属性时才使用 Container
return Container(
  margin: EdgeInsets.all(8),
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(8),
  ),
  child: Text('Styled content'),
);
```

**性能对比**：

| Widget      | 用途                             | 相对开销 |
| ----------- | -------------------------------- | -------- |
| `Container` | 多功能容器（边距、填充、装饰等） | 较高     |
| `SizedBox`  | 设置固定尺寸                     | 低       |
| `Padding`   | 设置内边距                       | 低       |
| `Center`    | 居中子元素                       | 低       |
| `Align`     | 对齐子元素                       | 低       |

#### 2.2.2 sized_box_for_whitespace

**规则说明**：使用 `SizedBox` 而不是 `Container` 来创建空白间隔。

**用途**：当需要在布局中创建空白间隔时，`SizedBox` 比 `Container` 更轻量、更明确。

**错误示例**：

```dart
// 错误：使用 Container 创建空白
Column(
  children: [
    Text('First'),
    Container(height: 16),  // 不应该用 Container
    Text('Second'),
  ],
)

// 错误：使用 Padding 创建垂直空白
Column(
  children: [
    Text('First'),
    Padding(
      padding: EdgeInsets.only(top: 16),
      child: Container(),  // 多余的 Container
    ),
    Text('Second'),
  ],
)
```

**正确写法**：

```dart
// 正确：使用 SizedBox
Column(
  children: [
    Text('First'),
    SizedBox(height: 16),  // 清晰、轻量
    Text('Second'),
  ],
)

// 正确：水平空白
Row(
  children: [
    Icon(Icons.star),
    SizedBox(width: 8),
    Text('Rating'),
  ],
)

// 正确：使用 const 进一步优化
Column(
  children: [
    Text('First'),
    const SizedBox(height: 16),  // const 构造
    Text('Second'),
  ],
)
```

Dart Tips 语法小贴士

**const SizedBox 的性能优势**

`SizedBox` 有一个 `const` 构造函数，可以在编译时创建实例。当使用 `const SizedBox` 时：

1. 实例在编译时创建，运行时不再分配内存
2. 相同的 const 实例会被复用
3. 在 Widget 重建时，const 实例不会触发子树的重建

```dart
// 推荐：使用 const
const SizedBox(height: 16)

// 不推荐：非 const 构造
SizedBox(height: 16)
```

#### 2.2.3 sort_child_properties_last

**规则说明**：将 `child` 属性放在最后。

**用途**：Flutter 的代码风格约定将 `child` 或 `children` 属性放在最后，使代码更易读，特别是当 Widget 有很多属性时。

**错误示例**：

```dart
// 错误：child 不在最后
return Container(
  child: Text('Hello'),
  color: Colors.blue,
  padding: EdgeInsets.all(16),
);

// 错误：children 不在最后
return Column(
  children: [
    Text('First'),
    Text('Second'),
  ],
  crossAxisAlignment: CrossAxisAlignment.start,
  mainAxisSize: MainAxisSize.min,
);
```

**正确写法**：

```dart
// 正确：child 在最后
return Container(
  color: Colors.blue,
  padding: EdgeInsets.all(16),
  child: Text('Hello'),
);

// 正确：children 在最后
return Column(
  crossAxisAlignment: CrossAxisAlignment.start,
  mainAxisSize: MainAxisSize.min,
  children: [
    Text('First'),
    Text('Second'),
  ],
);
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.3 Widget Key 类规则

#### 2.3.1 use_key_in_widget_constructors

**规则说明**：在 Widget 构造函数中使用 `Key` 参数。

**用途**：Key 帮助 Flutter 在 Widget 树重建时识别和区分 Widget 实例，对于动画、列表项和状态保持非常重要。

**错误示例**：

```dart
// 错误：没有 Key 参数
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// 错误：StatefulWidget 也没有 Key
class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}
```

**正确写法**：

```dart
// 正确：StatelessWidget 使用 Key
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// 正确：StatefulWidget 使用 Key
class Counter extends StatefulWidget {
  const Counter({super.key});

  @override
  State<Counter> createState() => _CounterState();
}

// 正确：带有其他参数的 Widget
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.userName,
    required this.avatarUrl,
  });

  final String userName;
  final String avatarUrl;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: NetworkImage(avatarUrl),
        ),
        title: Text(userName),
      ),
    );
  }
}
```

Flutter 框架小知识

**什么时候必须使用 Key？**

虽然不是所有 Widget 都必须有 Key，但以下场景强烈建议使用：

1. **列表项**：`ListView`、`GridView` 中的 item 需要稳定的 Key

   ```dart
   ListView.builder(
     itemBuilder: (context, index) {
       return ListTile(
         key: ValueKey(items[index].id),  // 使用唯一标识
         title: Text(items[index].name),
       );
     },
   )
   ```

2. **动画 Widget**：`AnimatedSwitcher`、`Hero` 等需要 Key 来识别动画目标

3. **状态保持**：当 Widget 在树中移动位置时保持状态

4. **测试**：在 Widget 测试中，Key 可以帮助定位特定 Widget

### 2.4 状态管理类规则

#### 2.4.1 no_logic_in_create_state

**规则说明**：不要在 `createState` 方法中包含逻辑。

**用途**：`createState` 应该只返回一个 `State` 实例，不应该执行任何副作用或逻辑操作。

**错误示例**：

```dart
// 错误：在 createState 中包含逻辑
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() {
    print('Creating state');  // 不应该有副作用
    return _MyWidgetState();
  }
}

// 错误：条件判断在 createState 中
class MyWidget extends StatefulWidget {
  final bool useAlternativeState;

  MyWidget({required this.useAlternativeState});

  @override
  State createState() {
    if (useAlternativeState) {
      return _AlternativeState();
    }
    return _MyWidgetState();
  }
}
```

**正确写法**：

```dart
// 正确：简单的 createState
class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

// 正确：如果需要条件逻辑，使用工厂构造函数
class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  factory MyWidget.withAlternativeState() {
    return _AlternativeWidget();
  }

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _AlternativeWidget extends MyWidget {
  @override
  State createState() => _AlternativeState();
}
```

#### 2.4.2 prefer_const_constructors_in_immutables

**规则说明**：在不可变类中优先使用 const 构造函数。

**用途**：对于标记为 `@immutable` 的类（如 `StatelessWidget`、`State` 等），使用 const 构造函数可以提高性能。

**错误示例**：

```dart
// 错误：没有 const 构造函数
@immutable
class MyWidget extends StatelessWidget {
  MyWidget({super.key});  // 缺少 const

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

**正确写法**：

```dart
// 正确：使用 const 构造函数
@immutable
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// 正确：带有参数的 const 构造函数
@immutable
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.userName,
  });

  final String userName;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Text(userName),
    );
  }
}
```

### 2.5 异步操作类规则

#### 2.5.1 use_build_context_synchronously

**规则说明**：在异步操作后正确使用 `BuildContext`。

**用途**：在 `async` 操作（如 `await Future.delayed`、网络请求）之后，`BuildContext` 可能已经无效（对应的 Widget 已被销毁），直接使用会导致运行时错误。

**错误示例**：

```dart
// 错误：异步操作后直接使用 context
class LoginButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        await Future.delayed(Duration(seconds: 1));
        // 此时 context 可能已无效！
        Navigator.of(context).pushNamed('/home');
      },
      child: Text('Login'),
    );
  }
}

// 错误：网络请求后使用 context
class DataLoader extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        final data = await fetchData();
        // 危险：context 可能已无效
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Loaded: $data')),
        );
      },
      child: Text('Load Data'),
    );
  }
}
```

**正确写法**：

```dart
// 正确：检查 mounted（StatefulWidget）
class LoginButton extends StatefulWidget {
  @override
  _LoginButtonState createState() => _LoginButtonState();
}

class _LoginButtonState extends State<LoginButton> {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        await Future.delayed(Duration(seconds: 1));
        if (mounted) {  // 检查 Widget 是否仍在树中
          Navigator.of(context).pushNamed('/home');
        }
      },
      child: Text('Login'),
    );
  }
}

// 正确：使用 BuildContext 的同步方法
class DataLoader extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        final scaffoldMessenger = ScaffoldMessenger.of(context);
        final data = await fetchData();
        // 使用之前获取的 scaffoldMessenger，而不是 context
        scaffoldMessenger.showSnackBar(
          SnackBar(content: Text('Loaded: $data')),
        );
      },
      child: Text('Load Data'),
    );
  }
}

// 正确：在 StatelessWidget 中使用 if (context.mounted)（Flutter 3.7+）
class ModernLoader extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        final data = await fetchData();
        if (context.mounted) {  // Flutter 3.7+ 支持
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('Loaded: $data')),
          );
        }
      },
      child: Text('Load Data'),
    );
  }
}
```

**常见场景处理**：

```dart
// 场景 1：导航
Future<void> navigateToDetails(BuildContext context) async {
  await Future.delayed(Duration(milliseconds: 300));
  if (context.mounted) {
    await Navigator.of(context).pushNamed('/details');
  }
}

// 场景 2：显示对话框
Future<void> showResultDialog(BuildContext context) async {
  final result = await fetchData();
  if (context.mounted) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        content: Text('Result: $result'),
      ),
    );
  }
}

// 场景 3：显示 SnackBar
void showMessage(BuildContext context, String message) async {
  final messenger = ScaffoldMessenger.of(context);
  await Future.delayed(Duration(seconds: 1));
  messenger.showSnackBar(
    SnackBar(content: Text(message)),
  );
}
```

### 2.6 平台兼容类规则

#### 2.6.1 avoid_web_libraries_in_flutter

**规则说明**：避免在 Flutter 代码中直接使用 Web 库。

**用途**：`dart:html`、`dart:js` 等 Web 库只在 Web 平台可用，在移动或桌面平台会导致编译错误。Flutter 应用如果需要 Web 支持，应该使用条件导入或跨平台方案。

**错误示例**：

```dart
// 错误：直接导入 Web 库
import 'dart:html';  // 只在 Web 平台可用
import 'dart:js' as js;  // 只在 Web 平台可用

void launchUrl(String url) {
  // 这会编译失败（在非 Web 平台）
  window.open(url, '_blank');
}
```

**正确写法**：

```dart
// 正确：使用 Flutter 官方插件
import 'package:url_launcher/url_launcher.dart';

Future<void> launchUrl(String url) async {
  final uri = Uri.parse(url);
  if (await canLaunchUrl(uri)) {
    await launchUrl(uri);
  }
}

// 正确：使用条件导入
import 'platform_stub.dart'
    if (dart.library.html) 'platform_web.dart'
    if (dart.library.io) 'platform_io.dart';

// platform_web.dart
import 'dart:html';
void openUrl(String url) => window.open(url, '_blank');

// platform_io.dart
import 'package:url_launcher/url_launcher.dart';
Future<void> openUrl(String url) async {
  await launchUrl(Uri.parse(url));
}

// platform_stub.dart
void openUrl(String url) => throw UnsupportedError('Unsupported platform');
```

### 2.7 颜色使用类规则

#### 2.7.1 use_full_hex_values_for_flutter_colors

**规则说明**：使用完整的十六进制值表示 Flutter 颜色。

**用途**：Flutter 的 `Color` 类需要 8 位十六进制值（包含 Alpha 通道），使用简写形式可能导致意外的透明效果。

**错误示例**：

```dart
// 错误：使用 6 位十六进制
Color primary = Color(0xFF2196F3);  // 缺少 Alpha 通道
Color accent = Color(0x2196F3);     // 错误的格式

// 错误：使用 CSS 风格的十六进制
Color background = Color(0x2196F3);  // 会被解释为带透明度的颜色
```

**正确写法**：

```dart
// 正确：使用 8 位十六进制（ARGB）
Color primary = Color(0xFF2196F3);  // 不透明蓝色
Color semiTransparent = Color(0x802196F3);  // 50% 透明蓝色

// 正确：使用 Colors 常量
Color primary = Colors.blue;
Color accent = Colors.blueAccent;

// 正确：使用 ColorScheme
Color primary = Theme.of(context).colorScheme.primary;
```

**颜色格式说明**：

| 格式         | 含义         | 示例                              |
| ------------ | ------------ | --------------------------------- |
| `0xFF2196F3` | 不透明蓝色   | ARGB: A=FF(255), R=21, G=96, B=F3 |
| `0x802196F3` | 50% 透明蓝色 | ARGB: A=80(128), R=21, G=96, B=F3 |
| `0x002196F3` | 完全透明     | ARGB: A=00, R=21, G=96, B=F3      |

## 第3章 继承的 Dart Lints 规则概览

`flutter_lints` 继承了 `package:lints/recommended.yaml` 的所有规则。以下是这些规则的分类概览。

### 3.1 Core Lints（核心规则）

这些是所有 Dart 代码都应该通过的基础规则（33 条）：

| 规则名                                  | 说明                                  | 自动修复 |
| --------------------------------------- | ------------------------------------- | -------- |
| avoid_empty_else                        | 避免空的 else 子句                    | ✅       |
| avoid_relative_lib_imports              | 避免相对导入 lib/ 下的文件            | ✅       |
| avoid_shadowing_type_parameters         | 避免遮蔽类型参数                      | ❌       |
| avoid_types_as_parameter_names          | 避免使用类型名作为参数名              | ❌       |
| await_only_futures                      | 只对 Future 使用 await                | ✅       |
| camel_case_extensions                   | 扩展名使用 UpperCamelCase             | ❌       |
| camel_case_types                        | 类型名使用 UpperCamelCase             | ❌       |
| collection_methods_unrelated_type       | 检测集合方法的不相关类型参数          | ❌       |
| curly_braces_in_flow_control_structures | 流程控制结构使用花括号                | ✅       |
| dangling_library_doc_comments           | 库文档注释附加到库指令                | ✅       |
| depend_on_referenced_packages           | 依赖被引用的包                        | ❌       |
| empty_catches                           | 避免空的 catch 块                     | ✅       |
| file_names                              | 文件名使用 lowercase_with_underscores | ❌       |
| hash_and_equals                         | 重写 == 时同时重写 hashCode           | ✅       |
| implicit_call_tearoffs                  | 显式 tear-off call 方法               | ✅       |
| library_annotations                     | 库注解附加到库指令                    | ✅       |
| no_duplicate_case_values                | 避免重复的 case 值                    | ✅       |
| no_wildcard_variable_uses               | 不使用通配符变量                      | ❌       |
| non_constant_identifier_names           | 非常量标识符使用 lowerCamelCase       | ✅       |
| null_check_on_nullable_type_parameter   | 不对可空类型参数进行空检查            | ✅       |
| prefer_generic_function_type_aliases    | 优先使用泛型函数类型别名              | ✅       |
| prefer_is_empty                         | 使用 isEmpty                          | ✅       |
| prefer_is_not_empty                     | 使用 isNotEmpty                       | ✅       |
| prefer_iterable_whereType               | 优先使用 whereType                    | ✅       |
| prefer_typing_uninitialized_variables   | 为未初始化变量指定类型                | ✅       |
| provide_deprecation_message             | 提供弃用消息                          | ❌       |
| secure_pubspec_urls                     | 使用安全的 pubspec URL                | ❌       |
| strict_top_level_inference              | 指定顶级类型注解                      | ✅       |
| type_literal_in_constant_pattern        | 不在常量模式中使用类型字面量          | ✅       |
| unintended_html_in_doc_comment          | 避免文档注释中的 HTML                 | ❌       |
| unnecessary_overrides                   | 避免不必要的重写                      | ✅       |
| unrelated_type_equality_checks          | 检测不相关类型的相等性比较            | ❌       |
| use_string_in_part_of_directives        | 在 part of 中使用字符串               | ✅       |
| valid_regexps                           | 使用有效的正则表达式                  | ❌       |
| void_checks                             | 不给 void 赋值                        | ❌       |

### 3.2 Recommended Lints（推荐规则）

这些是在 Core 基础上增加的推荐规则（55 条）：

| 规则名                                               | 说明                                  | 自动修复 |
| ---------------------------------------------------- | ------------------------------------- | -------- |
| annotate_overrides                                   | 为重写成员添加 @override              | ✅       |
| avoid_function_literals_in_foreach_calls             | 避免在 forEach 中使用函数字面量       | ✅       |
| avoid_init_to_null                                   | 不显式初始化为 null                   | ✅       |
| avoid_renaming_method_parameters                     | 不重命名被重写方法的参数              | ✅       |
| avoid_return_types_on_setters                        | setter 不指定返回类型                 | ✅       |
| avoid_returning_null_for_void                        | 不为 void 返回 null                   | ✅       |
| avoid_single_cascade_in_expression_statements        | 避免单个级联操作                      | ✅       |
| constant_identifier_names                            | 常量名使用 lowerCamelCase             | ✅       |
| control_flow_in_finally                              | 避免 finally 中的控制流               | ❌       |
| empty_constructor_bodies                             | 空构造函数体使用 ;                    | ✅       |
| empty_statements                                     | 避免空语句                            | ✅       |
| exhaustive_cases                                     | 处理枚举的所有 case                   | ✅       |
| implementation_imports                               | 不导入实现文件                        | ❌       |
| invalid_runtime_check_with_js_interop_types          | 避免对 JS 互操作类型进行类型检查      | ❌       |
| library_prefixes                                     | 库前缀使用 lowercase_with_underscores | ❌       |
| library_private_types_in_public_api                  | 公共 API 不使用私有类型               | ❌       |
| no_leading_underscores_for_library_prefixes          | 库前缀不使用前导下划线                | ✅       |
| no_leading_underscores_for_local_identifiers         | 局部标识符不使用前导下划线            | ✅       |
| null_closures                                        | 不传递 null 作为闭包                  | ✅       |
| overridden_fields                                    | 不重写字段                            | ❌       |
| package_names                                        | 包名使用 lowercase_with_underscores   | ❌       |
| prefer_adjacent_string_concatenation                 | 使用相邻字符串连接                    | ✅       |
| prefer_collection_literals                           | 使用集合字面量                        | ✅       |
| prefer_conditional_assignment                        | 使用 ??=                              | ✅       |
| prefer_contains                                      | 使用 contains                         | ✅       |
| prefer_final_fields                                  | 私有字段可以是 final                  | ✅       |
| prefer_for_elements_to_map_fromIterable              | 使用 for 元素构建 Map                 | ✅       |
| prefer_function_declarations_over_variables          | 使用函数声明                          | ✅       |
| prefer_if_null_operators                             | 使用 ??                               | ✅       |
| prefer_initializing_formals                          | 使用初始化形式参数                    | ✅       |
| prefer_inlined_adds                                  | 内联列表项声明                        | ✅       |
| prefer_interpolation_to_compose_strings              | 使用字符串插值                        | ✅       |
| prefer_is_not_operator                               | 使用 is!                              | ✅       |
| prefer_null_aware_operators                          | 使用空感知操作符                      | ✅       |
| prefer_spread_collections                            | 使用展开集合                          | ✅       |
| recursive_getters                                    | 避免递归 getter                       | ❌       |
| slash_for_doc_comments                               | 文档注释使用 ///                      | ✅       |
| type_init_formals                                    | 初始化形式参数不加类型                | ✅       |
| unnecessary_brace_in_string_interps                  | 避免不必要的插值花括号                | ✅       |
| unnecessary_const                                    | 避免不必要的 const                    | ✅       |
| unnecessary_constructor_name                         | 避免 .new                             | ✅       |
| unnecessary_getters_setters                          | 避免不必要的 getter/setter            | ✅       |
| unnecessary_late                                     | 避免不必要的 late                     | ✅       |
| unnecessary_library_name                             | 避免库名                              | ✅       |
| unnecessary_new                                      | 避免 new                              | ✅       |
| unnecessary_null_aware_assignments                   | 避免 ??= null                         | ✅       |
| unnecessary_null_in_if_null_operators                | 避免 ?? null                          | ✅       |
| unnecessary_nullable_for_final_variable_declarations | final 变量使用非空类型                | ✅       |
| unnecessary_string_escapes                           | 移除不必要的转义                      | ✅       |
| unnecessary_string_interpolations                    | 避免不必要的插值                      | ✅       |
| unnecessary_this                                     | 避免不必要的 this                     | ✅       |
| unnecessary_to_list_in_spreads                       | 避免展开中的 toList                   | ✅       |
| unnecessary_underscores                              | 移除不必要的下划线                    | ✅       |
| use_function_type_syntax_for_parameters              | 使用函数类型语法                      | ✅       |
| use_null_aware_elements                              | 使用空感知元素                        | ✅       |
| use_rethrow_when_possible                            | 使用 rethrow                          | ✅       |
| use_super_parameters                                 | 使用超类初始化参数                    | ✅       |

## 第4章 Flutter Lints 实战应用

### 4.1 配置最佳实践

#### 4.1.1 标准 Flutter 应用配置

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true
  errors:
    missing_required_param: error
    missing_return: error
    invalid_assignment: warning

linter:
  rules:
    # 代码风格
    prefer_single_quotes: true
    require_trailing_commas: true

    # 性能优化
    avoid_unnecessary_containers: true

    # 异步处理
    unawaited_futures: true
    avoid_catching_errors: true

    # 测试相关（只在测试目录启用）
    # avoid_print: false  # 测试中允许 print
```

#### 4.1.2 Flutter 插件/包配置

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true
  exclude:
    - '**/*.g.dart' # 生成的代码
    - '**/*.freezed.dart' # freezed 生成
    - '**/*.mocks.dart' # mockito 生成

linter:
  rules:
    # 公共 API 文档
    public_member_api_docs: true

    # 包名规范
    package_names: true

    # 类型注解
    always_declare_return_types: true

    # 避免 print
    avoid_print: true
```

#### 4.1.3 团队项目配置

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true
  errors:
    # 将某些警告提升为错误
    invalid_assignment: error
    missing_return: error
    dead_code: info
  exclude:
    - 'lib/generated/**'
    - 'test/mocks/**'

linter:
  rules:
    # 代码风格（团队约定）
    prefer_single_quotes: true
    require_trailing_commas: true
    prefer_final_locals: true
    prefer_final_in_for_each: true

    # 代码质量
    avoid_print: true
    avoid_unnecessary_containers: true
    use_key_in_widget_constructors: true

    # 异步处理
    unawaited_futures: true
    use_build_context_synchronously: true

    # 类型安全
    always_specify_types: false # 根据需要启用
    avoid_dynamic_calls: false # 根据需要启用
```

### 4.2 CI/CD 集成

#### 4.2.1 GitHub Actions 配置

```yaml
# .github/workflows/flutter_analyze.yml
name: Flutter Analyze

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.32.0'
          channel: 'stable'

      - name: Install dependencies
        run: flutter pub get

      - name: Analyze code
        run: flutter analyze --fatal-infos --fatal-warnings

      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .

      - name: Run tests
        run: flutter test --coverage
```

#### 4.2.2 GitLab CI 配置

```yaml
# .gitlab-ci.yml
stages:
  - analyze
  - test

flutter_analyze:
  stage: analyze
  image: cirrusci/flutter:3.32.0
  script:
    - flutter pub get
    - flutter analyze --fatal-infos
    - dart format --output=none --set-exit-if-changed .
  only:
    - merge_requests
    - main

flutter_test:
  stage: test
  image: cirrusci/flutter:3.32.0
  script:
    - flutter pub get
    - flutter test --coverage
  coverage: '/lines\.\.\.\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/lcov.info
```

### 4.3 常见问题与解决方案

#### 4.3.1 处理遗留代码

对于有大量遗留代码的项目，建议采用渐进式迁移：

**步骤 1：创建基础配置**

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    # 暂时将所有警告降级为信息
    unused_import: info
    unused_local_variable: info
    dead_code: info
```

**步骤 2：使用 dart fix 自动修复**

```bash
# 预览可修复的问题
dart fix --dry-run

# 应用自动修复
dart fix --apply
```

**步骤 3：逐步启用严格规则**

```yaml
# 每周启用几条新规则
linter:
  rules:
    # 已修复的规则启用为错误
    unused_import: true

    # 待修复的规则保持为警告
    prefer_single_quotes: warning
```

#### 4.3.2 忽略特定警告

**单行忽略**：

```dart
// 忽略单行
// ignore: avoid_print
print('调试信息');

// 忽略多条规则
// ignore: avoid_print, avoid_unnecessary_containers
print('调试');
return Container(child: Text('Hello'));
```

**文件级别忽略**：

```dart
// 文件顶部
// ignore_for_file: avoid_print, avoid_unnecessary_containers

// 或者针对特定类型
// ignore_for_file: type=lint
```

**目录级别忽略**：

```yaml
# analysis_options.yaml
analyzer:
  exclude:
    - 'lib/generated/**'
    - 'test/mocks/**'
    - 'lib/**/*.g.dart'
```

#### 4.3.3 与代码生成工具协作

```yaml
# analysis_options.yaml
analyzer:
  exclude:
    # 生成的代码
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/*.mocks.dart"
    - "lib/generated/**"

  # 为生成的代码使用单独的配置
  plugins:
    - custom_lint

# 为生成的代码目录创建单独的配置
# analysis_options_generated.yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    # 对生成的代码放宽要求
    public_member_api_docs: ignore
    avoid_print: ignore
```

### 4.4 性能优化建议

#### 4.4.1 启用 const 构造

虽然 `flutter_lints` 6.0.0 移除了强制 const 的规则，但在性能敏感场景下仍建议手动启用：

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # 性能关键场景启用
    prefer_const_constructors: true
    prefer_const_declarations: true
    prefer_const_literals_to_create_immutables: true
```

**const 的性能优势**：

```dart
// 推荐：const 构造
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        const Text('Title'),  // const 实例复用
        const SizedBox(height: 16),
        const Icon(Icons.star),
      ],
    );
  }
}
```

#### 4.4.2 避免不必要的重建

```dart
// 推荐：使用 const 子 Widget
class OptimizedWidget extends StatelessWidget {
  const OptimizedWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return const ListTile(  // const 避免重建
          leading: Icon(Icons.person),
          title: Text('User'),
        );
      },
    );
  }
}
```

## 附录 A：Flutter Lints 6.0.0 完整规则列表

### Flutter 特定规则（10 条）

| 规则名                                  | 说明                        | 类别        |
| --------------------------------------- | --------------------------- | ----------- |
| avoid_print                             | 避免使用 print              | 调试输出    |
| avoid_unnecessary_containers            | 避免不必要的 Container      | Widget 优化 |
| avoid_web_libraries_in_flutter          | 避免使用 Web 库             | 平台兼容    |
| no_logic_in_create_state                | create_state 中不包含逻辑   | 状态管理    |
| prefer_const_constructors_in_immutables | 不可变类优先使用 const 构造 | 性能优化    |
| sized_box_for_whitespace                | 使用 SizedBox 创建空白      | Widget 优化 |
| sort_child_properties_last              | child 属性放在最后          | 代码风格    |
| use_build_context_synchronously         | 异步后正确使用 BuildContext | 异步安全    |
| use_full_hex_values_for_flutter_colors  | 使用完整十六进制颜色值      | 颜色使用    |
| use_key_in_widget_constructors          | Widget 构造函数使用 Key     | Widget 标识 |

### 继承自 lints/recommended.yaml 的规则（88 条）

详见第 3 章的规则概览表格。

## 附录 B：版本变更历史

### 6.0.0（2025 年）

- 更新 `package:lints` 依赖至 6.0.0
- 最低支持 SDK 版本提升至 Flutter 3.32 / Dart 3.8
- **移除**以下规则：
  - `prefer_const_constructors`
  - `prefer_const_declarations`
  - `prefer_const_literals_to_create_immutables`

### 5.0.0（2024 年）

- 更新 `package:lints` 依赖至 5.0.0
- 最低支持 SDK 版本提升至 Flutter 3.24 / Dart 3.5
- **移除** `avoid_null_checks_in_equality_operators`

### 4.0.0（2024 年）

- 更新 `package:lints` 依赖至 4.0.0
- 最低支持 SDK 版本提升至 Flutter 3.13 / Dart 3.1
- **新增**规则：
  - `library_annotations`
  - `no_wildcard_variable_uses`
- **移除** `package_prefixed_library_names`

### 3.0.0（2023 年）

- 最低支持 SDK 版本提升至 Flutter 3.10 / Dart 3.0
- **新增**规则：
  - `collection_methods_unrelated_type`
  - `dangling_library_doc_comments`
  - `implicit_call_tearoffs`
  - `secure_pubspec_urls`
  - `type_literal_in_constant_pattern`
  - `unnecessary_to_list_in_spreads`
  - `use_string_in_part_of_directives`
  - `use_super_parameters`
- **移除**规则：
  - `iterable_contains_unrelated_type`
  - `list_remove_unrelated_type`
  - `prefer_equal_for_default_values`
  - `prefer_void_to_null`

### 2.0.0（2022 年）

- **新增**规则：
  - `sort_child_properties_last`
  - `use_build_context_synchronously`

### 1.0.0（2021 年）

- 初始版本
- 包含 10 条 Flutter 特定规则
- 继承 `package:lints` 2.0.0 的所有规则

## 附录 C：参考资源

- [Flutter Lints 官方文档](https://pub.dev/packages/flutter_lints)
- [Dart Linter 规则完整列表](https://dart.dev/tools/linter-rules)
- [Flutter 静态分析配置指南](https://dart.dev/tools/analysis)
- [Effective Dart](https://dart.dev/effective-dart)
- [Flutter 官方 GitHub 仓库](https://github.com/flutter/packages/tree/main/packages/flutter_lints)
- [Flutter 性能优化最佳实践](https://docs.flutter.dev/perf/best-practices)
