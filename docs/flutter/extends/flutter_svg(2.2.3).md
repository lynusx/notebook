# flutter_svg 2.2.3 详解与实战

## 第1章 认识 flutter_svg

### 1.1 什么是 flutter_svg

`flutter_svg` 是 Flutter 团队官方维护的 SVG 渲染和 Widget 库，允许开发者在 Flutter 应用中绘制和显示可缩放矢量图形（Scalable Vector Graphics，SVG）1.1 文件。作为 Flutter Favorite 计划的成员，该包被 Flutter 团队推荐用于生产环境。

与位图图片（PNG、JPEG）不同，SVG 是基于 XML 的矢量图形格式，具有以下优势：

| 特性     | SVG              | 位图图片   |
| -------- | ---------------- | ---------- |
| 缩放性   | 无限缩放不失真   | 放大后模糊 |
| 文件大小 | 通常更小         | 通常较大   |
| 可编辑性 | 可修改颜色、形状 | 不可编辑   |
| 动画支持 | 支持路径动画     | 不支持     |
| 清晰度   | 始终清晰         | 依赖分辨率 |

Flutter 框架小知识

**SVG 与 Flutter 的渲染原理**

`flutter_svg` 并不是将 SVG 转换为位图后显示，而是：

1. **解析阶段**：将 SVG 的 XML 结构解析为 Flutter 的 `Picture` 对象
2. **缓存阶段**：解析后的 `Picture` 被缓存到内存中，避免重复解析
3. **渲染阶段**：使用 Skia 图形引擎直接绘制矢量路径

这种架构使得 SVG 可以：

- 在不同 DPI 设备上保持清晰
- 支持运行时动态修改颜色
- 占用较少的内存（相比加载多个分辨率的位图）

### 1.2 flutter_svg 2.2.3 的新特性

版本 2.2.3 是 `flutter_svg` 的重要版本，基于 `vector_graphics` 后端进行了全面重构：

#### 1.2.1 vector_graphics 后端

从 2.0 版本开始，`flutter_svg` 迁移到了 `vector_graphics` 后端，带来以下改进：

- **更快的解析速度**：二进制格式比 XML 解析更快
- **更好的性能**：优化的渲染管线
- **预编译支持**：可以将 SVG 预编译为 `.vec` 二进制格式

#### 1.2.2 渲染策略（RenderingStrategy）

新增 `RenderingStrategy` 枚举，允许开发者选择渲染方式：

```dart
enum RenderingStrategy {
  picture,  // 默认，使用 Picture 绘制，支持灵活缩放
  raster,   // 栅格化渲染，性能更好但牺牲缩放灵活性
}
```

#### 1.2.3 ColorMapper

新增 `ColorMapper` 类，提供更灵活的颜色替换机制：

```dart
class MyColorMapper extends ColorMapper {
  @override
  Color substitute(String? id, String elementName, String attributeName, Color color) {
    // 根据元素 ID、元素名、属性名或原始颜色进行替换
    return color;
  }
}
```

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter_svg: ^2.2.3
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 配置资源文件

在 `pubspec.yaml` 中声明 SVG 资源：

```yaml
flutter:
  assets:
    - assets/icons/ # 整个目录
    - assets/images/logo.svg # 单个文件
```

Dart Tips 语法小贴士

**SVG 资源目录组织建议**

推荐按用途组织 SVG 资源：

```
assets/
├── icons/              # 图标类 SVG
│   ├── home.svg
│   ├── settings.svg
│   └── user.svg
├── illustrations/      # 插图类 SVG
│   ├── empty_state.svg
│   └── onboarding_1.svg
├── logos/              # Logo 类 SVG
│   ├── company_logo.svg
│   └── app_logo.svg
└── flags/              # 国旗类 SVG
    ├── us.svg
    └── cn.svg
```

## 第2章 SvgPicture 类详解

`SvgPicture` 是 `flutter_svg` 包的核心 Widget，用于显示 SVG 图片。它是一个 `StatelessWidget`，提供了多种构造函数来从不同来源加载 SVG。

### 2.1 构造函数概览

| 构造函数               | 用途         | 数据来源            |
| ---------------------- | ------------ | ------------------- |
| `SvgPicture()`         | 主构造函数   | `BytesLoader`       |
| `SvgPicture.asset()`   | 从资源加载   | `AssetBundle`       |
| `SvgPicture.network()` | 从网络加载   | HTTP URL            |
| `SvgPicture.file()`    | 从文件加载   | 本地文件系统        |
| `SvgPicture.string()`  | 从字符串加载 | 内存中的 SVG 字符串 |
| `SvgPicture.memory()`  | 从字节加载   | 内存中的字节数据    |

### 2.2 通用属性详解

所有 `SvgPicture` 构造函数共享以下属性：

#### 2.2.1 width 和 height（尺寸）

```dart
SvgPicture.asset(
  'assets/icon.svg',
  width: 48.0,   // 指定宽度（逻辑像素）
  height: 48.0,  // 指定高度（逻辑像素）
)
```

**属性说明**：

| 属性     | 类型      | 默认值 | 说明            |
| -------- | --------- | ------ | --------------- |
| `width`  | `double?` | `null` | 指定 SVG 的宽度 |
| `height` | `double?` | `null` | 指定 SVG 的高度 |

**使用场景**：

```dart
// 场景 1：固定尺寸图标
SvgPicture.asset(
  'assets/icons/home.svg',
  width: 24,
  height: 24,
)

// 场景 2：只指定宽度，高度自适应
SvgPicture.asset(
  'assets/banner.svg',
  width: double.infinity,  // 占满父容器宽度
)

// 场景 3：不指定尺寸，使用 SVG 原始尺寸
SvgPicture.asset('assets/logo.svg')
```

#### 2.2.2 fit（适配模式）

```dart
SvgPicture.asset(
  'assets/image.svg',
  fit: BoxFit.contain,  // 适配模式
)
```

**属性说明**：

| 属性  | 类型     | 默认值           | 说明             |
| ----- | -------- | ---------------- | ---------------- |
| `fit` | `BoxFit` | `BoxFit.contain` | 如何适配可用空间 |

**BoxFit 枚举值**：

| 值                 | 说明     | 效果                           |
| ------------------ | -------- | ------------------------------ |
| `BoxFit.contain`   | 包含     | 保持比例，完整显示，可能有留白 |
| `BoxFit.cover`     | 覆盖     | 保持比例，填满容器，可能裁剪   |
| `BoxFit.fill`      | 填充     | 不保持比例，拉伸填满           |
| `BoxFit.fitWidth`  | 适配宽度 | 保持比例，宽度填满             |
| `BoxFit.fitHeight` | 适配高度 | 保持比例，高度填满             |
| `BoxFit.none`      | 无       | 不缩放，可能溢出               |
| `BoxFit.scaleDown` | 缩小     | 类似 contain，但不会放大       |

**使用示例**：

```dart
// 图标：保持原始比例
SvgPicture.asset('assets/icon.svg', fit: BoxFit.contain)

// 背景图：填满容器
SvgPicture.asset('assets/bg.svg', fit: BoxFit.cover)

// 横幅：宽度填满
SvgPicture.asset('assets/banner.svg', fit: BoxFit.fitWidth)
```

#### 2.2.3 alignment（对齐方式）

```dart
SvgPicture.asset(
  'assets/image.svg',
  alignment: Alignment.center,  // 居中对齐
)
```

**属性说明**：

| 属性        | 类型                | 默认值             | 说明               |
| ----------- | ------------------- | ------------------ | ------------------ |
| `alignment` | `AlignmentGeometry` | `Alignment.center` | 在容器内的对齐方式 |

**常用对齐值**：

```dart
Alignment.topLeft      // 左上
Alignment.topCenter    // 顶部居中
Alignment.topRight     // 右上
Alignment.centerLeft   // 左中
Alignment.center       // 居中（默认）
Alignment.centerRight  // 右中
Alignment.bottomLeft   // 左下
Alignment.bottomCenter // 底部居中
Alignment.bottomRight  // 右下
```

#### 2.2.4 colorFilter（颜色滤镜）

```dart
SvgPicture.asset(
  'assets/icon.svg',
  colorFilter: ColorFilter.mode(Colors.red, BlendMode.srcIn),
)
```

**属性说明**：

| 属性          | 类型           | 默认值 | 说明                  |
| ------------- | -------------- | ------ | --------------------- |
| `colorFilter` | `ColorFilter?` | `null` | 应用到 SVG 的颜色滤镜 |

**常用 BlendMode**：

| 模式                 | 效果               | 适用场景     |
| -------------------- | ------------------ | ------------ |
| `BlendMode.srcIn`    | 用新颜色替换原颜色 | 单色图标着色 |
| `BlendMode.srcOver`  | 在新颜色上叠加原图 | 颜色混合     |
| `BlendMode.multiply` | 正片叠底           | 暗化效果     |
| `BlendMode.screen`   | 滤色               | 亮化效果     |

**使用示例**：

```dart
// 将 SVG 变为红色（推荐用于单色图标）
SvgPicture.asset(
  'assets/icons/home.svg',
  colorFilter: ColorFilter.mode(Colors.red, BlendMode.srcIn),
)

// 根据主题动态着色
SvgPicture.asset(
  'assets/icons/settings.svg',
  colorFilter: ColorFilter.mode(
    Theme.of(context).primaryColor,
    BlendMode.srcIn,
  ),
)
```

Flutter 框架小知识

**BlendMode.srcIn 的工作原理**

`BlendMode.srcIn` 是 SVG 着色的推荐模式，其工作原理是：

1. 使用目标颜色（新颜色）的 Alpha 通道
2. 保留源图像（SVG）的形状信息
3. 最终效果：SVG 的形状 + 新颜色

这要求 SVG 本身有一定的透明度或填充区域，纯色矩形 SVG 可能看不到效果。

#### 2.2.5 colorMapper（颜色映射器）

```dart
class BrandColorMapper extends ColorMapper {
  final Color primary;
  final Color secondary;

  const BrandColorMapper({
    required this.primary,
    required this.secondary,
  });

  @override
  Color substitute(
    String? id,
    String elementName,
    String attributeName,
    Color color,
  ) {
    // 将特定颜色替换为品牌色
    if (color == const Color(0xFFFF0000)) {
      return primary;  // 红色替换为主色
    }
    if (color == const Color(0xFF00FF00)) {
      return secondary;  // 绿色替换为次色
    }
    return color;  // 其他颜色保持不变
  }
}

// 使用
SvgPicture.asset(
  'assets/illustration.svg',
  colorMapper: BrandColorMapper(
    primary: Theme.of(context).primaryColor,
    secondary: Theme.of(context).colorScheme.secondary,
  ),
)
```

**ColorMapper 参数说明**：

| 参数            | 类型      | 说明                            |
| --------------- | --------- | ------------------------------- |
| `id`            | `String?` | SVG 元素的 ID                   |
| `elementName`   | `String`  | SVG 元素名称（如 rect、circle） |
| `attributeName` | `String`  | 属性名称（如 fill、stroke）     |
| `color`         | `Color`   | 原始颜色值                      |

**使用场景**：

1. **主题适配**：根据应用主题动态替换 SVG 中的颜色
2. **品牌定制**：将通用插图适配为品牌色彩
3. **多色图标**：精确控制多色 SVG 的每个颜色

#### 2.2.6 semanticsLabel（语义标签）

```dart
SvgPicture.asset(
  'assets/icons/home.svg',
  semanticsLabel: '首页图标',
)
```

**属性说明**：

| 属性             | 类型      | 默认值 | 说明         |
| ---------------- | --------- | ------ | ------------ |
| `semanticsLabel` | `String?` | `null` | 辅助功能描述 |

**用途**：

- 屏幕阅读器朗读
- 无障碍功能支持
- SEO 优化

#### 2.2.7 placeholderBuilder（占位构建器）

```dart
SvgPicture.network(
  'https://example.com/image.svg',
  placeholderBuilder: (BuildContext context) =>
    const CircularProgressIndicator(),
)
```

**属性说明**：

| 属性                 | 类型             | 默认值 | 说明                    |
| -------------------- | ---------------- | ------ | ----------------------- |
| `placeholderBuilder` | `WidgetBuilder?` | `null` | 加载时显示的占位 Widget |

**默认行为**：

- 未指定尺寸时：显示 `LimitedBox`（空盒子）
- 指定尺寸时：显示 `SizedBox`（指定尺寸的空盒子）

**使用示例**：

```dart
// 显示加载指示器
SvgPicture.network(
  imageUrl,
  placeholderBuilder: (context) => const Center(
    child: CircularProgressIndicator(),
  ),
)

// 显示骨架屏
SvgPicture.network(
  imageUrl,
  placeholderBuilder: (context) => Container(
    color: Colors.grey[300],
    child: const Icon(Icons.image),
  ),
)
```

#### 2.2.8 errorBuilder（错误构建器）

```dart
SvgPicture.network(
  'https://example.com/image.svg',
  errorBuilder: (context, error, stackTrace) =>
    const Icon(Icons.error),
)
```

**属性说明**：

| 属性           | 类型                     | 默认值 | 说明                    |
| -------------- | ------------------------ | ------ | ----------------------- |
| `errorBuilder` | `SvgErrorWidgetBuilder?` | `null` | 加载失败时显示的 Widget |

**注意**：错误信息会在 Debug 模式下输出到控制台，但默认不会显示视觉错误提示。

#### 2.2.9 renderingStrategy（渲染策略）

```dart
SvgPicture.asset(
  'assets/complex.svg',
  renderingStrategy: RenderingStrategy.raster,  // 栅格化渲染
)
```

**属性说明**：

| 属性                | 类型                | 默认值                      | 说明     |
| ------------------- | ------------------- | --------------------------- | -------- |
| `renderingStrategy` | `RenderingStrategy` | `RenderingStrategy.picture` | 渲染策略 |

**策略对比**：

| 策略      | 优点             | 缺点              | 适用场景       |
| --------- | ---------------- | ----------------- | -------------- |
| `picture` | 缩放灵活，质量高 | 复杂 SVG 性能稍低 | 大多数场景     |
| `raster`  | 渲染性能更好     | 缩放可能失真      | 复杂 SVG、动画 |

#### 2.2.10 clipBehavior（裁剪行为）

```dart
SvgPicture.asset(
  'assets/image.svg',
  clipBehavior: Clip.hardEdge,  // 硬边缘裁剪
)
```

**属性说明**：

| 属性           | 类型   | 默认值          | 说明                 |
| -------------- | ------ | --------------- | -------------------- |
| `clipBehavior` | `Clip` | `Clip.hardEdge` | 超出边界时的裁剪方式 |

**可选值**：

| 值                            | 说明               |
| ----------------------------- | ------------------ |
| `Clip.none`                   | 不裁剪             |
| `Clip.hardEdge`               | 硬边缘裁剪（默认） |
| `Clip.antiAlias`              | 抗锯齿裁剪         |
| `Clip.antiAliasWithSaveLayer` | 抗锯齿并保存图层   |

#### 2.2.11 allowDrawingOutsideViewBox

```dart
SvgPicture.asset(
  'assets/image.svg',
  allowDrawingOutsideViewBox: true,  // 允许绘制在 viewBox 外
)
```

**属性说明**：

| 属性                         | 类型   | 默认值  | 说明                      |
| ---------------------------- | ------ | ------- | ------------------------- |
| `allowDrawingOutsideViewBox` | `bool` | `false` | 是否允许在 viewBox 外绘制 |

**用途**：某些 SVG 可能包含超出 viewBox 的内容，启用此选项可以完整显示。

#### 2.2.12 matchTextDirection

```dart
SvgPicture.asset(
  'assets/arrow.svg',
  matchTextDirection: true,  // 根据文字方向翻转
)
```

**属性说明**：

| 属性                 | 类型   | 默认值  | 说明                  |
| -------------------- | ------ | ------- | --------------------- |
| `matchTextDirection` | `bool` | `false` | 在 RTL 布局中水平翻转 |

**用途**：支持从右到左（RTL）语言布局时，自动翻转方向性图标。

### 2.3 各构造函数详解

#### 2.3.1 SvgPicture.asset() - 从资源加载

```dart
SvgPicture.asset(
  String assetName, {
  Key? key,
  bool matchTextDirection = false,
  AssetBundle? bundle,
  String? package,
  double? width,
  double? height,
  BoxFit fit = BoxFit.contain,
  AlignmentGeometry alignment = Alignment.center,
  bool allowDrawingOutsideViewBox = false,
  WidgetBuilder? placeholderBuilder,
  String? semanticsLabel,
  bool excludeFromSemantics = false,
  Clip clipBehavior = Clip.hardEdge,
  SvgErrorWidgetBuilder? errorBuilder,
  SvgTheme? theme,
  ColorMapper? colorMapper,
  ColorFilter? colorFilter,
  RenderingStrategy renderingStrategy = RenderingStrategy.picture,
})
```

**特有参数**：

| 参数        | 类型           | 说明                       |
| ----------- | -------------- | -------------------------- |
| `assetName` | `String`       | 资源路径（必填）           |
| `bundle`    | `AssetBundle?` | 自定义资源包               |
| `package`   | `String?`      | 包名（用于依赖包中的资源） |

**使用示例**：

```dart
// 基本用法
SvgPicture.asset('assets/icons/home.svg')

// 带尺寸和颜色
SvgPicture.asset(
  'assets/icons/home.svg',
  width: 24,
  height: 24,
  colorFilter: ColorFilter.mode(Colors.blue, BlendMode.srcIn),
)

// 从依赖包加载资源
SvgPicture.asset(
  'assets/icons/home.svg',
  package: 'my_icons_package',
)
```

#### 2.3.2 SvgPicture.network() - 从网络加载

```dart
SvgPicture.network(
  String url, {
  Key? key,
  Map<String, String>? headers,
  double? width,
  double? height,
  BoxFit fit = BoxFit.contain,
  AlignmentGeometry alignment = Alignment.center,
  bool matchTextDirection = false,
  bool allowDrawingOutsideViewBox = false,
  WidgetBuilder? placeholderBuilder,
  ColorFilter? colorFilter,
  String? semanticsLabel,
  bool excludeFromSemantics = false,
  Clip clipBehavior = Clip.hardEdge,
  SvgErrorWidgetBuilder? errorBuilder,
  SvgTheme? theme,
  ColorMapper? colorMapper,
  Client? httpClient,
  RenderingStrategy renderingStrategy = RenderingStrategy.picture,
})
```

**特有参数**：

| 参数         | 类型                   | 说明               |
| ------------ | ---------------------- | ------------------ |
| `url`        | `String`               | SVG 的 URL（必填） |
| `headers`    | `Map<String, String>?` | HTTP 请求头        |
| `httpClient` | `Client?`              | 自定义 HTTP 客户端 |

**使用示例**：

```dart
// 基本用法
SvgPicture.network('https://example.com/logo.svg')

// 带加载占位符和错误处理
SvgPicture.network(
  'https://example.com/logo.svg',
  placeholderBuilder: (context) => const CircularProgressIndicator(),
  errorBuilder: (context, error, stackTrace) => const Icon(Icons.error),
)

// 带认证头
SvgPicture.network(
  'https://api.example.com/svg/123',
  headers: {'Authorization': 'Bearer token'},
)
```

#### 2.3.3 SvgPicture.file() - 从文件加载

```dart
SvgPicture.file(
  File file, {
  Key? key,
  double? width,
  double? height,
  BoxFit fit = BoxFit.contain,
  AlignmentGeometry alignment = Alignment.center,
  bool matchTextDirection = false,
  bool allowDrawingOutsideViewBox = false,
  WidgetBuilder? placeholderBuilder,
  ColorFilter? colorFilter,
  String? semanticsLabel,
  bool excludeFromSemantics = false,
  Clip clipBehavior = Clip.hardEdge,
  SvgErrorWidgetBuilder? errorBuilder,
  SvgTheme? theme,
  ColorMapper? colorMapper,
  RenderingStrategy renderingStrategy = RenderingStrategy.picture,
})
```

**特有参数**：

| 参数   | 类型   | 说明             |
| ------ | ------ | ---------------- |
| `file` | `File` | 本地文件（必填） |

**使用示例**：

```dart
// 从本地文件加载
final file = File('/path/to/image.svg');
SvgPicture.file(file)

// 从应用文档目录加载
final directory = await getApplicationDocumentsDirectory();
final file = File('${directory.path}/downloaded.svg');
SvgPicture.file(file)
```

#### 2.3.4 SvgPicture.string() - 从字符串加载

```dart
SvgPicture.string(
  String string, {
  Key? key,
  double? width,
  double? height,
  BoxFit fit = BoxFit.contain,
  AlignmentGeometry alignment = Alignment.center,
  bool matchTextDirection = false,
  bool allowDrawingOutsideViewBox = false,
  WidgetBuilder? placeholderBuilder,
  ColorFilter? colorFilter,
  String? semanticsLabel,
  bool excludeFromSemantics = false,
  Clip clipBehavior = Clip.hardEdge,
  SvgErrorWidgetBuilder? errorBuilder,
  SvgTheme? theme,
  ColorMapper? colorMapper,
  RenderingStrategy renderingStrategy = RenderingStrategy.picture,
})
```

**特有参数**：

| 参数     | 类型     | 说明                   |
| -------- | -------- | ---------------------- |
| `string` | `String` | SVG XML 字符串（必填） |

**使用示例**：

```dart
// 内联 SVG
const svgString = '''
<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="blue"/>
</svg>
''';

SvgPicture.string(svgString)

// 动态生成 SVG
String generateCircleSvg(Color color) => '''
<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="${color.toHex()}"/>
</svg>
''';

SvgPicture.string(generateCircleSvg(Colors.red))
```

#### 2.3.5 SvgPicture.memory() - 从内存加载

```dart
SvgPicture.memory(
  Uint8List bytes, {
  Key? key,
  double? width,
  double? height,
  BoxFit fit = BoxFit.contain,
  AlignmentGeometry alignment = Alignment.center,
  bool matchTextDirection = false,
  bool allowDrawingOutsideViewBox = false,
  WidgetBuilder? placeholderBuilder,
  ColorFilter? colorFilter,
  String? semanticsLabel,
  bool excludeFromSemantics = false,
  Clip clipBehavior = Clip.hardEdge,
  SvgErrorWidgetBuilder? errorBuilder,
  SvgTheme? theme,
  ColorMapper? colorMapper,
  RenderingStrategy renderingStrategy = RenderingStrategy.picture,
})
```

**特有参数**：

| 参数    | 类型        | 说明                 |
| ------- | ----------- | -------------------- |
| `bytes` | `Uint8List` | SVG 字节数据（必填） |

**使用示例**：

```dart
// 从 base64 解码
final base64String = 'PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==';
final bytes = base64Decode(base64String);
SvgPicture.memory(bytes)

// 从网络下载的字节
final response = await http.get(Uri.parse('https://example.com/image.svg'));
SvgPicture.memory(response.bodyBytes)
```

## 第3章 高级类与功能详解

### 3.1 BytesLoader 抽象类

`BytesLoader` 是 `flutter_svg` 中用于加载 SVG 数据的抽象类。所有 SVG 数据源都实现了这个接口。

#### 3.1.1 SvgAssetLoader - 资源加载器

```dart
const SvgAssetLoader(
  String assetName, {
  AssetBundle? bundle,
  String? package,
})
```

**使用示例**：

```dart
SvgPicture(
  const SvgAssetLoader('assets/icons/home.svg'),
  width: 24,
  height: 24,
)
```

#### 3.1.2 SvgNetworkLoader - 网络加载器

```dart
SvgNetworkLoader(
  String url, {
  Map<String, String>? headers,
  Client? httpClient,
})
```

**使用示例**：

```dart
SvgPicture(
  SvgNetworkLoader(
    'https://example.com/logo.svg',
    headers: {'Authorization': 'Bearer token'},
  ),
)
```

#### 3.1.3 SvgFileLoader - 文件加载器

```dart
SvgFileLoader(File file)
```

**使用示例**：

```dart
SvgPicture(
  SvgFileLoader(File('/path/to/image.svg')),
)
```

#### 3.1.4 SvgStringLoader - 字符串加载器

```dart
const SvgStringLoader(String string)
```

**使用示例**：

```dart
SvgPicture(
  const SvgStringLoader('<svg>...</svg>'),
)
```

#### 3.1.5 SvgBytesLoader - 字节加载器

```dart
const SvgBytesLoader(Uint8List bytes)
```

**使用示例**：

```dart
SvgPicture(
  SvgBytesLoader(bytes),
)
```

### 3.2 ColorMapper 类详解

`ColorMapper` 是一个抽象类，用于在 SVG 解析过程中动态替换颜色。

#### 3.2.1 基本用法

```dart
abstract class ColorMapper {
  const ColorMapper();

  Color substitute(
    String? id,           // 元素 ID
    String elementName,   // 元素名称（如 rect、circle）
    String attributeName, // 属性名称（如 fill、stroke）
    Color color,          // 原始颜色
  );
}
```

#### 3.2.2 实战示例：主题适配

```dart
class ThemeColorMapper extends ColorMapper {
  final Color primary;
  final Color secondary;
  final Color surface;

  const ThemeColorMapper({
    required this.primary,
    required this.secondary,
    required this.surface,
  });

  @override
  Color substitute(String? id, String elementName, String attributeName, Color color) {
    // 将特定颜色映射到主题色
    const originalPrimary = Color(0xFF2196F3);
    const originalSecondary = Color(0xFFFF9800);
    const originalSurface = Color(0xFFFFFFFF);

    if (color == originalPrimary) return primary;
    if (color == originalSecondary) return secondary;
    if (color == originalSurface) return surface;

    return color;
  }
}

// 使用
class ThemedSvg extends StatelessWidget {
  final String assetName;

  const ThemedSvg({super.key, required this.assetName});

  @override
  Widget build(BuildContext context) {
    final colorScheme = Theme.of(context).colorScheme;

    return SvgPicture.asset(
      assetName,
      colorMapper: ThemeColorMapper(
        primary: colorScheme.primary,
        secondary: colorScheme.secondary,
        surface: colorScheme.surface,
      ),
    );
  }
}
```

#### 3.2.3 实战示例：动态着色

```dart
class DynamicColorMapper extends ColorMapper {
  final Color targetColor;
  final double saturation;

  const DynamicColorMapper({
    required this.targetColor,
    this.saturation = 1.0,
  });

  @override
  Color substitute(String? id, String elementName, String attributeName, Color color) {
    // 只替换填充色，保留描边色
    if (attributeName == 'fill') {
      return targetColor.withOpacity(color.opacity);
    }
    return color;
  }
}
```

### 3.3 底层渲染 API

对于需要更底层控制的场景，`flutter_svg` 提供了直接操作 `Picture` 的 API。

#### 3.3.1 vg.loadPicture()

```dart
import 'package:flutter_svg/flutter_svg.dart' as vg;
import 'dart:ui' as ui;

// 加载 SVG 为 Picture
final PictureInfo pictureInfo = await vg.loadPicture(
  const SvgStringLoader('<svg>...</svg>'),
  null,
);

// 绘制到 Canvas
canvas.drawPicture(pictureInfo.picture);

// 转换为 Image
final ui.Image image = await pictureInfo.picture.toImage(width, height);

// 释放资源
pictureInfo.picture.dispose();
```

#### 3.3.2 PictureInfo 类

```dart
class PictureInfo {
  final ui.Picture picture;  // Flutter Picture 对象
  final Size size;           // SVG 尺寸
  final Color? color;        // 颜色信息
}
```

## 第4章 SVG 预编译与优化

### 4.1 为什么需要预编译

`flutter_svg` 2.x 引入了 SVG 预编译功能，可以将 SVG 文件编译为二进制 `.vec` 格式：

| 优势           | 说明                                 |
| -------------- | ------------------------------------ |
| 更快的解析速度 | 二进制格式比 XML 解析快 3-5 倍       |
| 更小的体积     | 优化的二进制格式通常比 SVG 小 20-40% |
| 减少裁剪和遮罩 | 编译器可以优化掉不必要的图形操作     |
| 减少过度绘制   | 优化渲染路径，提高性能               |

### 4.2 安装编译器

```bash
dart pub add dev:vector_graphics_compiler
```

### 4.3 编译 SVG 文件

#### 4.3.1 命令行编译

```bash
# 基本用法
dart run vector_graphics_compiler -i assets/foo.svg -o assets/foo.svg.vec

# 编译整个目录
for file in assets/icons/*.svg; do
  dart run vector_graphics_compiler -i "$file" -o "${file}.vec"
done
```

#### 4.3.2 编译选项

```bash
# 禁用优化（用于调试）
dart run vector_graphics_compiler \
  -i assets/foo.svg \
  -o assets/foo.svg.vec \
  --no-optimize-masks \
  --no-optimize-clips \
  --no-optimize-overdraw \
  --no-tessellate
```

### 4.4 使用编译后的文件

```dart
import 'package:flutter_svg/flutter_svg.dart';
import 'package:vector_graphics/vector_graphics.dart';

// 加载编译后的 SVG
final Widget svg = SvgPicture(
  const AssetBytesLoader('assets/foo.svg.vec'),
);
```

### 4.5 在 pubspec.yaml 中配置资源

```yaml
flutter:
  assets:
    # 原始 SVG（开发时使用）
    - assets/icons/
    # 编译后的 SVG（生产环境使用）
    - assets/icons_compiled/
```

### 4.6 构建时自动编译

可以在 `build.yaml` 中配置自动编译：

```yaml
targets:
  $default:
    builders:
      vector_graphics_compiler|vector_graphics:
        enabled: true
        options:
          optimize_masks: true
          optimize_clips: true
          optimize_overdraw: true
          tessellate: true
```

## 第5章 flutter_svg 实战应用

### 5.1 图标系统实现

#### 5.1.1 定义图标类

```dart
class AppIcons {
  AppIcons._();

  static const String _basePath = 'assets/icons';

  static const String home = '$_basePath/home.svg';
  static const String settings = '$_basePath/settings.svg';
  static const String user = '$_basePath/user.svg';
  static const String search = '$_basePath/search.svg';
  static const String notification = '$_basePath/notification.svg';
}
```

#### 5.1.2 创建图标组件

```dart
class AppIcon extends StatelessWidget {
  final String assetName;
  final double? size;
  final Color? color;
  final BoxFit fit;

  const AppIcon({
    super.key,
    required this.assetName,
    this.size = 24,
    this.color,
    this.fit = BoxFit.contain,
  });

  @override
  Widget build(BuildContext context) {
    final iconColor = color ?? IconTheme.of(context).color;

    return SvgPicture.asset(
      assetName,
      width: size,
      height: size,
      fit: fit,
      colorFilter: iconColor != null
          ? ColorFilter.mode(iconColor, BlendMode.srcIn)
          : null,
    );
  }
}
```

#### 5.1.3 使用图标

```dart
// 基本用法
AppIcon(assetName: AppIcons.home)

// 自定义大小和颜色
AppIcon(
  assetName: AppIcons.settings,
  size: 32,
  color: Colors.blue,
)

// 在按钮中使用
IconButton(
  icon: AppIcon(assetName: AppIcons.search),
  onPressed: () {},
)
```

### 5.2 主题适配图标

```dart
class ThemedIcon extends StatelessWidget {
  final String assetName;
  final double? size;

  const ThemedIcon({
    super.key,
    required this.assetName,
    this.size = 24,
  });

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return SvgPicture.asset(
      assetName,
      width: size,
      height: size,
      colorFilter: ColorFilter.mode(
        theme.colorScheme.onSurface,
        BlendMode.srcIn,
      ),
    );
  }
}
```

### 5.3 网络 SVG 图片组件

```dart
class NetworkSvgImage extends StatelessWidget {
  final String url;
  final double? width;
  final double? height;
  final BoxFit fit;
  final Widget? placeholder;
  final Widget? errorWidget;

  const NetworkSvgImage({
    super.key,
    required this.url,
    this.width,
    this.height,
    this.fit = BoxFit.contain,
    this.placeholder,
    this.errorWidget,
  });

  @override
  Widget build(BuildContext context) {
    return SvgPicture.network(
      url,
      width: width,
      height: height,
      fit: fit,
      placeholderBuilder: placeholder != null
          ? (context) => placeholder!
          : null,
      errorBuilder: errorWidget != null
          ? (context, error, stackTrace) => errorWidget!
          : null,
    );
  }
}
```

### 5.4 带缓存的网络 SVG

```dart
class CachedNetworkSvg extends StatefulWidget {
  final String url;
  final double? width;
  final double? height;

  const CachedNetworkSvg({
    super.key,
    required this.url,
    this.width,
    this.height,
  });

  @override
  State<CachedNetworkSvg> createState() => _CachedNetworkSvgState();
}

class _CachedNetworkSvgState extends State<CachedNetworkSvg> {
  Uint8List? _cachedBytes;
  bool _isLoading = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadSvg();
  }

  Future<void> _loadSvg() async {
    try {
      final response = await http.get(Uri.parse(widget.url));
      if (response.statusCode == 200) {
        setState(() {
          _cachedBytes = response.bodyBytes;
          _isLoading = false;
        });
      } else {
        setState(() {
          _error = 'Failed to load';
          _isLoading = false;
        });
      }
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return const Icon(Icons.error);
    }

    return SvgPicture.memory(
      _cachedBytes!,
      width: widget.width,
      height: widget.height,
    );
  }
}
```

### 5.5 SVG 动画实现

虽然 `flutter_svg` 不直接支持 SMIL 动画，但可以通过 Flutter 动画实现：

```dart
class AnimatedSvg extends StatefulWidget {
  final String assetName;
  final Duration duration;

  const AnimatedSvg({
    super.key,
    required this.assetName,
    this.duration = const Duration(seconds: 2),
  });

  @override
  State<AnimatedSvg> createState() => _AnimatedSvgState();
}

class _AnimatedSvgState extends State<AnimatedSvg>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.duration,
      vsync: this,
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
    _controller.repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return Transform.rotate(
          angle: _animation.value * 2 * math.pi,
          child: SvgPicture.asset(widget.assetName),
        );
      },
    );
  }
}
```

## 第6章 最佳实践与常见问题

### 6.1 SVG 设计规范

#### 6.1.1 Adobe Illustrator 导出设置

1. **样式（Styling）**：选择 "Presentation Attributes" 而非 "Inline CSS"
   - CSS 支持不完全，可能导致渲染问题

2. **图片（Images）**：选择 "Embed" 而非 "Linked"
   - 确保 SVG 是独立的单一文件

3. **对象 ID（Objects IDs）**：选择 "Layer Names"
   - 便于在 ColorMapper 中通过 ID 识别元素

#### 6.1.2 SVG 优化建议

1. **简化路径**：减少不必要的节点
2. **合并形状**：减少元素数量
3. **移除元数据**：删除编辑器相关的元数据
4. **使用 viewBox**：确保正确缩放

### 6.2 性能优化

#### 6.2.1 使用 const 构造

```dart
// 推荐
const SvgIcon({super.key, required this.assetName});

// 使用
const SvgIcon(assetName: AppIcons.home)
```

#### 6.2.2 预编译 SVG

```bash
# 编译所有 SVG
for file in assets/icons/*.svg; do
  dart run vector_graphics_compiler -i "$file" -o "${file}.vec"
done
```

#### 6.2.3 合理选择渲染策略

```dart
// 复杂 SVG 使用 raster 策略
SvgPicture.asset(
  'assets/complex_illustration.svg',
  renderingStrategy: RenderingStrategy.raster,
)

// 简单图标使用 picture 策略（默认）
SvgPicture.asset('assets/icon.svg')
```

### 6.3 常见问题与解决方案

#### Q1：SVG 显示为纯色方块

**原因**：SVG 没有透明区域，或使用了不支持的 BlendMode

**解决**：

```dart
// 确保 SVG 有透明背景
// 使用正确的 BlendMode
SvgPicture.asset(
  'assets/icon.svg',
  colorFilter: ColorFilter.mode(Colors.red, BlendMode.srcIn),
)
```

#### Q2：网络 SVG 加载慢

**解决**：

```dart
// 添加占位符
SvgPicture.network(
  url,
  placeholderBuilder: (context) => const CircularProgressIndicator(),
)

// 或使用预编译格式
```

#### Q3：SVG 颜色不随主题变化

**解决**：

```dart
SvgPicture.asset(
  assetName,
  colorFilter: ColorFilter.mode(
    Theme.of(context).colorScheme.primary,
    BlendMode.srcIn,
  ),
)
```

#### Q4：SVG 在某些设备上模糊

**解决**：

```dart
// 使用 picture 渲染策略保持清晰度
SvgPicture.asset(
  assetName,
  renderingStrategy: RenderingStrategy.picture,
)
```

### 6.4 调试技巧

#### 6.4.1 检查 SVG 兼容性

```bash
# 测试 SVG 是否能正常编译
dart run vector_graphics_compiler \
  -i input.svg \
  -o output.vec \
  --no-optimize-masks \
  --no-optimize-clips \
  --no-optimize-overdraw \
  --no-tessellate
```

#### 6.4.2 查看解析错误

在 Debug 模式下，解析错误会输出到控制台。确保查看日志以定位问题。

## 附录 A：SvgPicture 属性速查表

| 属性                         | 类型                     | 默认值                      | 说明                  |
| ---------------------------- | ------------------------ | --------------------------- | --------------------- |
| `width`                      | `double?`                | `null`                      | 宽度                  |
| `height`                     | `double?`                | `null`                      | 高度                  |
| `fit`                        | `BoxFit`                 | `BoxFit.contain`            | 适配模式              |
| `alignment`                  | `AlignmentGeometry`      | `Alignment.center`          | 对齐方式              |
| `colorFilter`                | `ColorFilter?`           | `null`                      | 颜色滤镜              |
| `colorMapper`                | `ColorMapper?`           | `null`                      | 颜色映射器            |
| `semanticsLabel`             | `String?`                | `null`                      | 语义标签              |
| `placeholderBuilder`         | `WidgetBuilder?`         | `null`                      | 占位构建器            |
| `errorBuilder`               | `SvgErrorWidgetBuilder?` | `null`                      | 错误构建器            |
| `clipBehavior`               | `Clip`                   | `Clip.hardEdge`             | 裁剪行为              |
| `renderingStrategy`          | `RenderingStrategy`      | `RenderingStrategy.picture` | 渲染策略              |
| `allowDrawingOutsideViewBox` | `bool`                   | `false`                     | 允许绘制在 viewBox 外 |
| `matchTextDirection`         | `bool`                   | `false`                     | 匹配文字方向          |
| `excludeFromSemantics`       | `bool`                   | `false`                     | 排除语义              |

## 附录 B：构造函数参数对比表

| 参数          | asset | network | file | string | memory |
| ------------- | ----- | ------- | ---- | ------ | ------ |
| `assetName`   | ✓     | -       | -    | -      | -      |
| `url`         | -     | ✓       | -    | -      | -      |
| `file`        | -     | -       | ✓    | -      | -      |
| `string`      | -     | -       | -    | ✓      | -      |
| `bytes`       | -     | -       | -    | -      | ✓      |
| `bundle`      | ○     | -       | -    | -      | -      |
| `package`     | ○     | -       | -    | -      | -      |
| `headers`     | -     | ○       | -    | -      | -      |
| `httpClient`  | -     | ○       | -    | -      | -      |
| `width`       | ○     | ○       | ○    | ○      | ○      |
| `height`      | ○     | ○       | ○    | ○      | ○      |
| `fit`         | ○     | ○       | ○    | ○      | ○      |
| `colorFilter` | ○     | ○       | ○    | ○      | ○      |
| `colorMapper` | ○     | ○       | ○    | ○      | ○      |

✓ = 必填参数，○ = 可选参数，- = 不支持

## 附录 C：参考资源

- [flutter_svg 官方文档](https://pub.dev/packages/flutter_svg)
- [flutter_svg API 参考](https://pub.dev/documentation/flutter_svg/latest/)
- [vector_graphics 编译器](https://pub.dev/packages/vector_graphics_compiler)
- [SVG 1.1 规范](https://www.w3.org/TR/SVG11/)
- [Flutter 官方 SVG 指南](https://docs.flutter.dev/ui/assets/assets-and-images)
