# google_fonts 8.0.2 详解与实战

## 第1章 认识 google_fonts

### 1.1 什么是 google_fonts

`google_fonts` 是 Flutter 团队官方维护的 Google Fonts 集成包，让开发者能够轻松地在 Flutter 应用中使用 fonts.google.com 上超过 1500 种免费开源字体。该包是 Flutter Favorite 计划的成员，被 Flutter 团队推荐用于生产环境。

与传统的字体集成方式相比，`google_fonts` 包提供了以下核心优势：

| 特性     | google_fonts 包      | 传统方式             |
| -------- | -------------------- | -------------------- |
| 字体获取 | 运行时 HTTP 下载     | 手动下载并打包       |
| 应用体积 | 更小（字体按需下载） | 较大（所有字体打包） |
| 字体缓存 | 自动文件系统缓存     | 无缓存机制           |
| 离线支持 | 支持（缓存后）       | 完全支持             |
| 字体更新 | 自动获取最新版本     | 手动更新             |
| 开发体验 | 即开即用             | 需要配置资源         |

Flutter 框架小知识

**Google Fonts 的工作原理**

`google_fonts` 包的工作流程如下：

1. **首次加载**：应用首次使用某字体时，通过 HTTP 从 Google Fonts CDN 下载字体文件
2. **本地缓存**：下载的字体文件自动缓存到设备文件系统（`getApplicationSupportDirectory`）
3. **后续使用**：从本地缓存读取，无需再次网络请求
4. **离线支持**：缓存后的字体在离线状态下仍可正常使用

这种设计使得：

- 开发阶段可以快速尝试不同字体
- 生产环境应用体积更小
- 用户只下载实际使用的字体

### 1.2 google_fonts 8.0.2 的新特性

版本 8.0.2 是 `google_fonts` 的重要版本更新，主要包含以下改进：

#### 1.2.1 字体库更新

- **同步 Google Fonts 最新字体库**：包含 1500+ 种字体，涵盖多种语言和风格
- **新增字体支持**：包含最新的 Google Fonts 发布

#### 1.2.2 性能优化

- **改进的字体加载机制**：更快的字体解析和应用
- **优化的缓存策略**：更高效的本地缓存管理

#### 1.2.3 兼容性提升

- **支持 Flutter 3.27+**：与最新 Flutter 版本兼容
- **Dart 3.6+ 支持**：利用最新的 Dart 语言特性

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  google_fonts: ^8.0.2
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台特定配置

**macOS 配置**：

对于 macOS 应用，需要在 `macos/Runner/DebugProfile.entitlements` 和 `macos/Runner/Release.entitlements` 中添加网络访问权限：

```xml
<key>com.apple.security.network.client</key>
<true/>
```

Dart Tips 语法小贴士

**网络权限配置说明**

不同平台的网络权限要求：

| 平台    | 配置要求                  |
| ------- | ------------------------- |
| Android | 默认允许网络访问          |
| iOS     | 默认允许网络访问          |
| macOS   | 需要显式配置 entitlements |
| Windows | 默认允许网络访问          |
| Linux   | 默认允许网络访问          |
| Web     | 需要 CORS 支持            |

## 第2章 GoogleFonts 类详解

`GoogleFonts` 是 `google_fonts` 包的核心类，提供了访问 Google Fonts 字体库的所有方法。它是一个静态类，包含超过 1500 个字体家族的 getter 方法。

### 2.1 核心方法概览

| 方法                              | 用途                     | 返回值         |
| --------------------------------- | ------------------------ | -------------- |
| `GoogleFonts.fontName()`          | 获取指定字体的 TextStyle | `TextStyle`    |
| `GoogleFonts.getFont()`           | 动态获取字体             | `TextStyle`    |
| `GoogleFonts.asTextTheme()`       | 创建完整 TextTheme       | `TextTheme`    |
| `GoogleFonts.fontNameTextTheme()` | 创建特定字体的 TextTheme | `TextTheme`    |
| `GoogleFonts.pendingFonts()`      | 预加载字体               | `Future<void>` |

### 2.2 GoogleFonts.fontName() 方法详解

这是最常用的字体获取方法，每个 Google Fonts 字体家族都有对应的 getter 方法。

#### 2.2.1 基本用法

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

// 使用 Lato 字体
Text(
  'Hello, Flutter!',
  style: GoogleFonts.lato(),
)

// 使用 Roboto 字体
Text(
  'Hello, Flutter!',
  style: GoogleFonts.roboto(),
)

// 使用 Open Sans 字体
Text(
  'Hello, Flutter!',
  style: GoogleFonts.openSans(),
)
```

#### 2.2.2 常用字体方法列表

| 字体方法                        | 字体名称         | 风格特点                             |
| ------------------------------- | ---------------- | ------------------------------------ |
| `GoogleFonts.roboto()`          | Roboto           | 现代无衬线，Material Design 默认字体 |
| `GoogleFonts.lato()`            | Lato             | 温暖友好的无衬线字体                 |
| `GoogleFonts.openSans()`        | Open Sans        | 中性、易读的无衬线字体               |
| `GoogleFonts.montserrat()`      | Montserrat       | 几何风格无衬线字体                   |
| `GoogleFonts.poppins()`         | Poppins          | 现代几何无衬线字体                   |
| `GoogleFonts.inter()`           | Inter            | 专为屏幕 UI 设计                     |
| `GoogleFonts.nunito()`          | Nunito           | 圆润友好的无衬线字体                 |
| `GoogleFonts.playfairDisplay()` | Playfair Display | 优雅的衬线字体                       |
| `GoogleFonts.merriweather()`    | Merriweather     | 专为屏幕阅读设计的衬线字体           |
| `GoogleFonts.sourceCodePro()`   | Source Code Pro  | 等宽字体，适合代码显示               |

#### 2.2.3 完整参数列表

```dart
TextStyle GoogleFonts.lato({
  TextStyle? textStyle,           // 基础 TextStyle
  Color? color,                   // 字体颜色
  Color? backgroundColor,         // 背景颜色
  double? fontSize,               // 字体大小
  FontWeight? fontWeight,         // 字重
  FontStyle? fontStyle,           // 字体样式（正常/斜体）
  double? letterSpacing,          // 字间距
  double? wordSpacing,            // 词间距
  TextBaseline? textBaseline,     // 文本基线
  double? height,                 // 行高
  Locale? locale,                 // 本地化
  List<Shadow>? shadows,          // 阴影
  List<FontFeature>? fontFeatures, // 字体特性
  TextDecoration? decoration,     // 文本装饰
  Color? decorationColor,         // 装饰颜色
  TextDecorationStyle? decorationStyle, // 装饰样式
  double? decorationThickness,    // 装饰厚度
})
```

**参数详解**：

| 参数              | 类型            | 默认值 | 说明                             |
| ----------------- | --------------- | ------ | -------------------------------- |
| `textStyle`       | `TextStyle?`    | `null` | 基础 TextStyle，其他属性会覆盖它 |
| `color`           | `Color?`        | `null` | 字体颜色                         |
| `backgroundColor` | `Color?`        | `null` | 文本背景颜色                     |
| `fontSize`        | `double?`       | `null` | 字体大小（逻辑像素）             |
| `fontWeight`      | `FontWeight?`   | `null` | 字重（w100-w900）                |
| `fontStyle`       | `FontStyle?`    | `null` | 字体样式                         |
| `letterSpacing`   | `double?`       | `null` | 字间距（逻辑像素）               |
| `wordSpacing`     | `double?`       | `null` | 词间距（逻辑像素）               |
| `height`          | `double?`       | `null` | 行高（字体大小的倍数）           |
| `shadows`         | `List<Shadow>?` | `null` | 文本阴影列表                     |

#### 2.2.4 使用示例

```dart
// 示例 1：基础用法
Text(
  'Hello, World!',
  style: GoogleFonts.lato(),
)

// 示例 2：自定义大小和颜色
Text(
  'Hello, World!',
  style: GoogleFonts.lato(
    fontSize: 24,
    color: Colors.blue,
  ),
)

// 示例 3：自定义字重和样式
Text(
  'Hello, World!',
  style: GoogleFonts.lato(
    fontSize: 18,
    fontWeight: FontWeight.w700,
    fontStyle: FontStyle.italic,
  ),
)

// 示例 4：完整自定义
Text(
  'Hello, World!',
  style: GoogleFonts.lato(
    textStyle: Theme.of(context).textTheme.headlineLarge,
    fontSize: 48,
    fontWeight: FontWeight.w900,
    letterSpacing: 2.0,
    color: Colors.deepPurple,
    shadows: [
      Shadow(
        color: Colors.black26,
        offset: Offset(2, 2),
        blurRadius: 4,
      ),
    ],
  ),
)
```

### 2.3 GoogleFonts.getFont() 方法详解

动态获取字体的方法，适用于需要在运行时决定字体的场景。

#### 2.3.1 方法签名

```dart
static TextStyle getFont(
  String fontName, {
  TextStyle? textStyle,
  Color? color,
  Color? backgroundColor,
  double? fontSize,
  FontWeight? fontWeight,
  FontStyle? fontStyle,
  double? letterSpacing,
  double? wordSpacing,
  TextBaseline? textBaseline,
  double? height,
  Locale? locale,
  List<Shadow>? shadows,
  List<FontFeature>? fontFeatures,
  TextDecoration? decoration,
  Color? decorationColor,
  TextDecorationStyle? decorationStyle,
  double? decorationThickness,
})
```

#### 2.3.2 使用场景

```dart
// 场景 1：动态字体选择
class DynamicFontText extends StatelessWidget {
  final String fontName;
  final String text;

  const DynamicFontText({
    super.key,
    required this.fontName,
    required this.text,
  });

  @override
  Widget build(BuildContext context) {
    return Text(
      text,
      style: GoogleFonts.getFont(fontName),
    );
  }
}

// 场景 2：用户自定义字体
class UserFontSelector extends StatefulWidget {
  @override
  _UserFontSelectorState createState() => _UserFontSelectorState();
}

class _UserFontSelectorState extends State<UserFontSelector> {
  String selectedFont = 'Lato';

  final List<String> fonts = [
    'Lato',
    'Roboto',
    'Open Sans',
    'Montserrat',
    'Poppins',
  ];

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        DropdownButton<String>(
          value: selectedFont,
          items: fonts.map((font) {
            return DropdownMenuItem(
              value: font,
              child: Text(font),
            );
          }).toList(),
          onChanged: (value) {
            setState(() {
              selectedFont = value!;
            });
          },
        ),
        Text(
          'Preview Text',
          style: GoogleFonts.getFont(selectedFont, fontSize: 24),
        ),
      ],
    );
  }
}
```

### 2.4 GoogleFonts.fontNameTextTheme() 方法详解

创建完整的 `TextTheme`，将指定字体应用到所有 Material Design 文本样式。

#### 2.4.1 方法签名

```dart
static TextTheme latoTextTheme([TextTheme? textTheme])

// 其他字体类似：
// robotoTextTheme([TextTheme? textTheme])
// openSansTextTheme([TextTheme? textTheme])
// ...
```

#### 2.4.2 TextTheme 包含的样式

| 样式             | 用途         | 默认大小 |
| ---------------- | ------------ | -------- |
| `displayLarge`   | 超大标题     | 57.0     |
| `displayMedium`  | 大标题       | 45.0     |
| `displaySmall`   | 中等标题     | 36.0     |
| `headlineLarge`  | 大标题       | 32.0     |
| `headlineMedium` | 中等标题     | 28.0     |
| `headlineSmall`  | 小标题       | 24.0     |
| `titleLarge`     | 大标题文本   | 22.0     |
| `titleMedium`    | 中等标题文本 | 16.0     |
| `titleSmall`     | 小标题文本   | 14.0     |
| `bodyLarge`      | 大正文       | 16.0     |
| `bodyMedium`     | 中等正文     | 14.0     |
| `bodySmall`      | 小正文       | 12.0     |
| `labelLarge`     | 大标签       | 14.0     |
| `labelMedium`    | 中等标签     | 12.0     |
| `labelSmall`     | 小标签       | 11.0     |

#### 2.4.3 使用示例

```dart
// 示例 1：应用全局字体主题
MaterialApp(
  title: 'Google Fonts Demo',
  theme: ThemeData(
    textTheme: GoogleFonts.latoTextTheme(),
  ),
  home: HomePage(),
)

// 示例 2：基于现有主题修改
MaterialApp(
  title: 'Google Fonts Demo',
  theme: ThemeData.light().copyWith(
    textTheme: GoogleFonts.robotoTextTheme(
      ThemeData.light().textTheme,
    ),
  ),
  home: HomePage(),
)

// 示例 3：混合多种字体
MaterialApp(
  title: 'Google Fonts Demo',
  theme: ThemeData(
    textTheme: GoogleFonts.latoTextTheme().copyWith(
      // 标题使用另一种字体
      headlineLarge: GoogleFonts.montserrat(
        fontSize: 32,
        fontWeight: FontWeight.bold,
      ),
      // 代码使用等宽字体
      bodyMedium: GoogleFonts.sourceCodePro(
        fontSize: 14,
      ),
    ),
  ),
  home: HomePage(),
)
```

### 2.5 GoogleFonts.asTextTheme() 方法详解

从现有 `TextTheme` 创建新的 `TextTheme`，应用指定字体。

#### 2.5.1 方法签名

```dart
static TextTheme asTextTheme(
  TextTheme textTheme, {
  String fontName = 'Roboto',
})
```

#### 2.5.2 使用示例

```dart
// 将现有主题字体替换为指定字体
final customTheme = ThemeData.light().copyWith(
  textTheme: GoogleFonts.asTextTheme(
    ThemeData.light().textTheme,
    fontName: 'Poppins',
  ),
);
```

### 2.6 GoogleFonts.pendingFonts() 方法详解

预加载字体，避免首次显示时的字体切换闪烁（FOUT - Flash of Unstyled Text）。

#### 2.6.1 方法签名

```dart
static Future<void> pendingFonts(List<TextStyle> fonts)
```

#### 2.6.2 使用场景

```dart
// 场景 1：启动时预加载字体
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late Future<void> _fontsPending;

  @override
  void initState() {
    super.initState();
    _fontsPending = GoogleFonts.pendingFonts([
      GoogleFonts.lato(fontWeight: FontWeight.w400),
      GoogleFonts.lato(fontWeight: FontWeight.w700),
      GoogleFonts.montserrat(fontWeight: FontWeight.w600),
    ]);
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _fontsPending,
      builder: (context, snapshot) {
        return MaterialApp(
          // 字体加载完成后显示应用
          home: snapshot.connectionState == ConnectionState.done
              ? HomePage()
              : LoadingScreen(),
        );
      },
    );
  }
}

// 场景 2：使用 FutureBuilder 避免字体闪烁
class FontPreloader extends StatelessWidget {
  final Widget child;

  const FontPreloader({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: GoogleFonts.pendingFonts([
        GoogleFonts.inter(),
        GoogleFonts.inter(fontWeight: FontWeight.w600),
      ]),
      builder: (context, snapshot) {
        if (snapshot.connectionState != ConnectionState.done) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        return child;
      },
    );
  }
}
```

Flutter 框架小知识

**字体加载闪烁问题（FOUT）**

FOUT（Flash of Unstyled Text）是指字体加载完成前，文本使用系统默认字体显示，加载完成后突然切换为自定义字体的现象。

**解决方案**：

1. **预加载字体**：使用 `GoogleFonts.pendingFonts()` 在应用启动时加载字体
2. **显示加载指示器**：字体加载完成前显示加载界面
3. **使用本地字体**：将字体打包到应用中，避免网络加载

## 第3章 字体字重详解

### 3.1 FontWeight 枚举

Google Fonts 支持多种字重，通过 `FontWeight` 枚举指定。

| 字重值            | 名称           | 说明         |
| ----------------- | -------------- | ------------ |
| `FontWeight.w100` | Thin           | 极细         |
| `FontWeight.w200` | Extra Light    | 超细         |
| `FontWeight.w300` | Light          | 细体         |
| `FontWeight.w400` | Regular/Normal | 常规（默认） |
| `FontWeight.w500` | Medium         | 中等         |
| `FontWeight.w600` | Semi Bold      | 半粗         |
| `FontWeight.w700` | Bold           | 粗体         |
| `FontWeight.w800` | Extra Bold     | 超粗         |
| `FontWeight.w900` | Black          | 极粗         |

### 3.2 使用示例

```dart
// 不同字重对比
Column(
  children: [
    Text(
      'Thin (w100)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w100),
    ),
    Text(
      'Light (w300)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w300),
    ),
    Text(
      'Regular (w400)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w400),
    ),
    Text(
      'Medium (w500)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w500),
    ),
    Text(
      'Bold (w700)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w700),
    ),
    Text(
      'Black (w900)',
      style: GoogleFonts.roboto(fontWeight: FontWeight.w900),
    ),
  ],
)
```

### 3.3 注意事项

**并非所有字体都支持所有字重**。如果指定的字重不可用，系统会自动选择最接近的可用字重。

```dart
// 检查字体是否支持特定字重
// 建议在设计阶段确认字体支持的字重范围
```

## 第4章 本地字体打包

### 4.1 为什么要本地打包

虽然 `google_fonts` 包支持运行时下载字体，但在以下场景建议本地打包：

| 场景         | 原因                 |
| ------------ | -------------------- |
| 离线优先应用 | 确保字体始终可用     |
| 严格合规要求 | 避免外部网络请求     |
| 性能敏感场景 | 消除网络加载延迟     |
| 字体定制     | 使用修改后的字体文件 |

### 4.2 本地字体配置步骤

#### 步骤 1：下载字体文件

从 https://fonts.google.com 下载需要的字体文件。

**字体文件名映射**：

| FontWeight | 文件名后缀 |
| ---------- | ---------- |
| w100       | Thin       |
| w200       | ExtraLight |
| w300       | Light      |
| w400       | Regular    |
| w500       | Medium     |
| w600       | SemiBold   |
| w700       | Bold       |
| w800       | ExtraBold  |
| w900       | Black      |

**斜体字体**：在文件名后添加 `Italic`，如 `Lato-Italic.ttf`。

#### 步骤 2：放置字体文件

将字体文件放入项目目录，例如 `assets/google_fonts/`：

```
assets/
└── google_fonts/
    ├── Lato-Regular.ttf
    ├── Lato-Bold.ttf
    ├── Lato-Italic.ttf
    └── Lato-BoldItalic.ttf
```

#### 步骤 3：配置 pubspec.yaml

```yaml
flutter:
  assets:
    - assets/google_fonts/
```

#### 步骤 4：使用本地字体

`google_fonts` 包会自动检测并使用本地字体文件，无需修改代码：

```dart
// 会自动使用 assets/google_fonts/ 下的字体文件
Text(
  'Hello, World!',
  style: GoogleFonts.lato(),
)
```

### 4.3 本地字体优先级

`google_fonts` 包的字体加载优先级：

1. **本地字体文件**：`assets/google_fonts/` 目录下的字体
2. **缓存字体**：设备文件系统中的缓存字体
3. **网络字体**：从 Google Fonts CDN 下载

## 第5章 google_fonts 实战应用

### 5.1 完整的主题配置示例

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Google Fonts Demo',
      theme: _buildLightTheme(),
      darkTheme: _buildDarkTheme(),
      themeMode: ThemeMode.system,
      home: HomePage(),
    );
  }

  ThemeData _buildLightTheme() {
    final baseTheme = ThemeData.light();

    return baseTheme.copyWith(
      // 主文本主题
      textTheme: GoogleFonts.interTextTheme(baseTheme.textTheme),

      // 标题使用不同字体
      primaryTextTheme: GoogleFonts.montserratTextTheme(
        baseTheme.primaryTextTheme,
      ),

      // 应用栏主题
      appBarTheme: AppBarTheme(
        titleTextStyle: GoogleFonts.montserrat(
          fontSize: 20,
          fontWeight: FontWeight.w600,
          color: Colors.black,
        ),
      ),

      // 按钮主题
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          textStyle: GoogleFonts.inter(
            fontWeight: FontWeight.w600,
          ),
        ),
      ),

      // 输入框主题
      inputDecorationTheme: InputDecorationTheme(
        labelStyle: GoogleFonts.inter(),
        hintStyle: GoogleFonts.inter(
          color: Colors.grey,
        ),
      ),
    );
  }

  ThemeData _buildDarkTheme() {
    final baseTheme = ThemeData.dark();

    return baseTheme.copyWith(
      textTheme: GoogleFonts.interTextTheme(baseTheme.textTheme),
      primaryTextTheme: GoogleFonts.montserratTextTheme(
        baseTheme.primaryTextTheme,
      ),
    );
  }
}
```

### 5.2 字体切换功能实现

```dart
class FontSwitcher extends StatefulWidget {
  @override
  _FontSwitcherState createState() => _FontSwitcherState();
}

class _FontSwitcherState extends State<FontSwitcher> {
  String _currentFont = 'Inter';

  final Map<String, String Function()> _fonts = {
    'Inter': () => 'Inter',
    'Roboto': () => 'Roboto',
    'Lato': () => 'Lato',
    'Montserrat': () => 'Montserrat',
    'Open Sans': () => 'Open Sans',
    'Poppins': () => 'Poppins',
  };

  TextStyle _getTextStyle() {
    return GoogleFonts.getFont(
      _currentFont,
      fontSize: 24,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Font Switcher'),
      ),
      body: Column(
        children: [
          // 字体选择器
          DropdownButton<String>(
            value: _currentFont,
            items: _fonts.keys.map((font) {
              return DropdownMenuItem(
                value: font,
                child: Text(font),
              );
            }).toList(),
            onChanged: (value) {
              setState(() {
                _currentFont = value!;
              });
            },
          ),

          // 预览文本
          Expanded(
            child: Center(
              child: Text(
                'The quick brown fox\njumps over the lazy dog',
                style: _getTextStyle(),
                textAlign: TextAlign.center,
              ),
            ),
          ),

          // 不同大小的预览
          Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Text(
                  'Headline',
                  style: _getTextStyle().copyWith(fontSize: 32),
                ),
                Text(
                  'Body text',
                  style: _getTextStyle().copyWith(fontSize: 16),
                ),
                Text(
                  'Caption',
                  style: _getTextStyle().copyWith(fontSize: 12),
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

### 5.3 字体预览组件

```dart
class FontPreview extends StatelessWidget {
  final String fontName;

  const FontPreview({super.key, required this.fontName});

  @override
  Widget build(BuildContext context) {
    final textStyle = GoogleFonts.getFont(fontName);

    return Card(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 字体名称
            Text(
              fontName,
              style: Theme.of(context).textTheme.titleMedium,
            ),
            SizedBox(height: 8),

            // 预览文本
            Text(
              'Aa Bb Cc Dd Ee Ff',
              style: textStyle.copyWith(fontSize: 24),
            ),

            // 不同字重
            Wrap(
              spacing: 8,
              children: [
                _WeightPreview(fontName: fontName, weight: FontWeight.w300, label: 'Light'),
                _WeightPreview(fontName: fontName, weight: FontWeight.w400, label: 'Regular'),
                _WeightPreview(fontName: fontName, weight: FontWeight.w700, label: 'Bold'),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

class _WeightPreview extends StatelessWidget {
  final String fontName;
  final FontWeight weight;
  final String label;

  const _WeightPreview({
    required this.fontName,
    required this.weight,
    required this.label,
  });

  @override
  Widget build(BuildContext context) {
    return Chip(
      label: Text(
        label,
        style: GoogleFonts.getFont(
          fontName,
          fontWeight: weight,
          fontSize: 12,
        ),
      ),
    );
  }
}
```

### 5.4 性能优化建议

#### 5.4.1 限制字体数量

```dart
// 不推荐：使用太多不同字体
Text('Title', style: GoogleFonts.roboto()),
Text('Subtitle', style: GoogleFonts.lato()),
Text('Body', style: GoogleFonts.openSans()),
Text('Caption', style: GoogleFonts.montserrat()),

// 推荐：使用 1-2 种字体
Text('Title', style: GoogleFonts.inter(fontWeight: FontWeight.bold)),
Text('Subtitle', style: GoogleFonts.inter(fontWeight: FontWeight.w600)),
Text('Body', style: GoogleFonts.inter()),
Text('Caption', style: GoogleFonts.inter(fontSize: 12)),
```

#### 5.4.2 预加载关键字体

```dart
class App extends StatefulWidget {
  @override
  _AppState createState() => _AppState();
}

class _AppState extends State<App> {
  late Future<void> _criticalFonts;

  @override
  void initState() {
    super.initState();
    // 预加载关键字体
    _criticalFonts = GoogleFonts.pendingFonts([
      GoogleFonts.inter(fontWeight: FontWeight.w400),
      GoogleFonts.inter(fontWeight: FontWeight.w600),
      GoogleFonts.inter(fontWeight: FontWeight.w700),
    ]);

    // 后台加载其他字体
    _preloadOtherFonts();
  }

  Future<void> _preloadOtherFonts() async {
    // 不等待这些字体
    GoogleFonts.pendingFonts([
      GoogleFonts.inter(fontWeight: FontWeight.w300),
      GoogleFonts.inter(fontWeight: FontWeight.w500),
    ]);
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: _criticalFonts,
      builder: (context, snapshot) {
        if (snapshot.connectionState != ConnectionState.done) {
          return MaterialApp(
            home: Scaffold(
              body: Center(child: CircularProgressIndicator()),
            ),
          );
        }
        return MainApp();
      },
    );
  }
}
```

#### 5.4.3 使用 const 构造

```dart
// 推荐：使用 const
const Text(
  'Hello',
  style: TextStyle(fontFamily: 'Roboto'),
)

// 对于 GoogleFonts，缓存 TextStyle 实例
class Styles {
  static final headline = GoogleFonts.inter(
    fontSize: 24,
    fontWeight: FontWeight.bold,
  );
  static final body = GoogleFonts.inter(
    fontSize: 16,
  );
}
```

## 第6章 常见问题与解决方案

### 6.1 字体不显示

**问题原因**：

1. 网络连接问题
2. 字体名称拼写错误
3. 平台权限问题（macOS）

**解决方案**：

```dart
// 检查字体名称拼写
// 错误：GoogleFonts.lato() 拼写为 GoogleFonts.Lato()

// 添加错误处理
Text(
  'Hello',
  style: GoogleFonts.lato().copyWith(
    // 设置后备字体
    fontFamilyFallback: ['Arial', 'sans-serif'],
  ),
)
```

### 6.2 字体加载慢

**解决方案**：

```dart
// 1. 预加载字体
@override
void initState() {
  super.initState();
  GoogleFonts.pendingFonts([
    GoogleFonts.lato(),
  ]);
}

// 2. 使用本地字体
// 将字体文件放入 assets/google_fonts/

// 3. 显示加载指示器
FutureBuilder(
  future: GoogleFonts.pendingFonts([...]),
  builder: (context, snapshot) {
    if (snapshot.connectionState != ConnectionState.done) {
      return LoadingWidget();
    }
    return MainContent();
  },
)
```

### 6.3 字体闪烁（FOUT）

**解决方案**：

```dart
// 使用 MaterialApp.builder 添加加载遮罩
MaterialApp(
  builder: (context, child) {
    return FutureBuilder(
      future: _fontsPending,
      builder: (context, snapshot) {
        final loading = snapshot.connectionState != ConnectionState.done;
        return Stack(
          children: [
            child!,
            if (loading)
              Container(
                color: Colors.white,
                child: Center(child: CircularProgressIndicator()),
              ),
          ],
        );
      },
    );
  },
)
```

## 附录 A：GoogleFonts 方法速查表

| 方法                              | 返回值         | 用途                     |
| --------------------------------- | -------------- | ------------------------ |
| `GoogleFonts.fontName()`          | `TextStyle`    | 获取指定字体的 TextStyle |
| `GoogleFonts.getFont(name)`       | `TextStyle`    | 动态获取字体             |
| `GoogleFonts.fontNameTextTheme()` | `TextTheme`    | 创建字体主题             |
| `GoogleFonts.asTextTheme()`       | `TextTheme`    | 从现有主题创建           |
| `GoogleFonts.pendingFonts()`      | `Future<void>` | 预加载字体               |

## 附录 B：常用 Google Fonts 推荐

### 无衬线字体（Sans Serif）

| 字体       | 风格       | 适用场景   |
| ---------- | ---------- | ---------- |
| Inter      | 现代、清晰 | UI、正文   |
| Roboto     | 中性、通用 | 通用       |
| Open Sans  | 友好、易读 | 正文       |
| Lato       | 温暖、专业 | 品牌       |
| Montserrat | 几何、现代 | 标题       |
| Poppins    | 圆润、现代 | 标题、品牌 |

### 衬线字体（Serif）

| 字体             | 风格       | 适用场景   |
| ---------------- | ---------- | ---------- |
| Playfair Display | 优雅、经典 | 标题、品牌 |
| Merriweather     | 易读、专业 | 长文阅读   |
| Lora             | 现代、优雅 | 正文       |

### 等宽字体（Monospace）

| 字体            | 风格       | 适用场景 |
| --------------- | ---------- | -------- |
| Source Code Pro | 清晰、专业 | 代码显示 |
| Fira Code       | 现代、连字 | 代码显示 |
| JetBrains Mono  | 清晰、易读 | 代码显示 |

## 附录 C：参考资源

- [google_fonts 官方文档](https://pub.dev/packages/google_fonts)
- [Google Fonts 官网](https://fonts.google.com)
- [Flutter 字体排版指南](https://docs.flutter.dev/ui/design/text/typography)
- [Material Design 排版](https://m3.material.io/styles/typography)
