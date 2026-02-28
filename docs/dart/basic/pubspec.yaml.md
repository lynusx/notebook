# pubspec.yaml 详解与实战

## 第1章 认识 pubspec.yaml

### 1.1 什么是 pubspec.yaml

`pubspec.yaml` 是每一个 Dart 和 Flutter 项目的核心配置文件，通常被称为"pubspec"。它位于项目的根目录，使用 YAML（YAML Ain't Markup Language）格式编写，为 Dart 和 Flutter 工具链提供了项目所需的元数据和配置信息。

当运行 `flutter pub get` 或 `dart pub get` 命令时，工具会读取 `pubspec.yaml`，解析其中的依赖声明，下载所需的包，并生成 `pubspec.lock` 文件锁定具体版本。第一次构建项目时，还会创建 `.dart_tool` 目录和 `pubspec.lock` 文件，确保团队成员和 CI/CD 环境使用完全相同的依赖版本。

#### 1.1.1 pubspec.yaml 的作用

`pubspec.yaml` 主要承担以下职责：

| 职责 | 说明 |
|------|------|
| **项目元数据** | 定义项目名称、版本、描述等基本信息 |
| **依赖管理** | 声明项目运行和开发所需的第三方包 |
| **资源管理** | 配置图片、字体、JSON 等静态资源 |
| **环境约束** | 指定 Dart/Flutter SDK 版本要求 |
| **发布配置** | 配置包发布到 pub.dev 的相关信息 |
| **脚本定义** | 定义可执行的命令行工具 |

Flutter 框架小知识

**YAML 格式注意事项**

`pubspec.yaml` 使用 YAML 格式，需要特别注意以下事项：

1. **缩进敏感**：YAML 使用缩进表示层级关系，必须使用空格（推荐 2 个空格），不能使用 Tab
2. **冒号后必须有空格**：`name: my_app` 正确，`name:my_app` 错误
3. **字符串引号**：大部分情况下不需要引号，但包含特殊字符时需要
4. **列表格式**：使用 `- `（减号加空格）表示列表项

```yaml
# 正确示例
name: my_app
dependencies:
  flutter:
    sdk: flutter
  http: ^1.0.0

# 错误示例 - 缺少空格
name:my_app
dependencies:
  flutter:
    sdk:flutter
```

### 1.2 基本结构概览

一个典型的 Flutter 项目的 `pubspec.yaml` 文件结构如下：

```yaml
# ========== 项目元数据 ==========
name: my_flutter_app
description: A new Flutter project.
publish_to: 'none'
version: 1.0.0+1

# ========== 环境配置 ==========
environment:
  sdk: ^3.6.0
  flutter: ^3.27.0

# ========== 运行时依赖 ==========
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  http: ^1.2.0

# ========== 开发依赖 ==========
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

# ========== Flutter 特定配置 ==========
flutter:
  uses-material-design: true
  
  assets:
    - assets/images/
    - assets/icons/logo.png
  
  fonts:
    - family: CustomFont
      fonts:
        - asset: fonts/CustomFont-Regular.ttf
        - asset: fonts/CustomFont-Bold.ttf
          weight: 700
```

### 1.3 新建项目生成的默认 pubspec.yaml

使用 `flutter create` 命令创建新项目时，会自动生成以下 `pubspec.yaml`：

```yaml
name: my_app
description: "A new Flutter project."
# 以下行阻止包被意外发布到 pub.dev
publish_to: 'none'

version: 1.0.0+1

environment:
  sdk: ^3.6.0

dependencies:
  flutter:
    sdk: flutter

  # 以下包增加了 Cupertino (iOS 风格) 图标
  cupertino_icons: ^1.0.8

dev_dependencies:
  flutter_test:
    sdk: flutter

  # 包含一组推荐的 Flutter 代码 lint 规则
  flutter_lints: ^6.0.0

flutter:
  # 以下行确保 Material Icons 字体包含在应用中
  uses-material-design: true

  # 添加资源到应用，可添加图片、字体等资源
  # assets:
  #   - images/a_dot_burr.jpeg
  #   - images/a_dot_ham.jpeg

  # 自定义字体配置
  # fonts:
  #   - family: Schyler
  #     fonts:
  #       - asset: fonts/Schyler-Regular.ttf
  #       - asset: fonts/Schyler-Italic.ttf
  #         style: italic
  #   - family: Trajan Pro
  #     fonts:
  #       - asset: fonts/TrajanPro.ttf
  #       - asset: fonts/TrajanPro_Bold.ttf
  #         weight: 700
```

## 第2章 项目元数据字段详解

### 2.1 name（项目名称）

**必填字段**，用于指定包或应用的名称。

**约束条件**：
- 必须全部小写
- 只能包含字母、数字和下划线
- 不能以数字开头
- 不能使用 Dart 保留关键字

**正确示例**：

```yaml
# 正确
name: my_app
name: flutter_utils
name: http_client

# 错误
name: MyApp          # 包含大写字母
name: my-app         # 包含连字符
name: 123app         # 以数字开头
name: dart           # 保留关键字
```

Dart Tips 语法小贴士

**包命名最佳实践**

1. **简洁明了**：使用简短、描述性的名称
2. **避免冲突**：在 pub.dev 上搜索确保名称唯一
3. **使用下划线分隔**：`my_package` 而不是 `mypackage`
4. **避免 flutter_ 前缀**：除非是 Flutter 团队官方包
5. **考虑作用域**：对于组织内部包，可使用前缀如 `company_name_package`

### 2.2 version（版本号）

**发布到 pub.dev 时必填**，遵循语义化版本控制（Semantic Versioning）。

**版本格式**：`主版本号.次版本号.修订号+构建号`

```yaml
version: 1.2.3+45
#      │ │ │ │ │
#      │ │ │ │ └── 构建号（可选）
#      │ │ │ └──── 修订号（bug 修复）
#      │ │ └────── 次版本号（向后兼容的功能添加）
#      │ └──────── 主版本号（破坏性变更）
#      └────────── 主版本号
```

**版本号变更规则**：

| 变更类型 | 版本号变化 | 示例 |
|----------|-----------|------|
| 破坏性变更 | 主版本号 +1 | `1.2.3` → `2.0.0` |
| 新功能（向后兼容） | 次版本号 +1 | `1.2.3` → `1.3.0` |
| Bug 修复 | 修订号 +1 | `1.2.3` → `1.2.4` |
| 构建更新 | 构建号 +1 | `1.2.3+1` → `1.2.3+2` |

**Flutter 应用版本号特殊说明**：

Flutter 应用版本号中的构建号会被用于：
- **Android**：`versionCode`
- **iOS/macOS**：`CFBundleVersion`

```yaml
# Flutter 应用版本示例
version: 1.2.3+45
# 在 Android 中对应：
# - versionName: "1.2.3"
# - versionCode: 45
#
# 在 iOS 中对应：
# - CFBundleShortVersionString: "1.2.3"
# - CFBundleVersion: "45"
```

### 2.3 description（项目描述）

**发布到 pub.dev 时必填**，用于描述包的功能和用途。

**约束条件**：
- 长度限制：60-180 个字符（推荐）
- 应该简洁明了，说明包的主要功能
- 不要使用 "A Dart package" 这种无意义的描述

**示例**：

```yaml
# 好的描述
description: A powerful HTTP client for Dart, which supports interceptors, FormData, request cancellation, file downloading, etc.
description: Flutter widgets for building customizable calendar interfaces with support for multiple view modes.

# 差的描述
description: A Dart package
description: My package
description: This is a package that does things and stuff with various features and capabilities for developers.
```

### 2.4 publish_to（发布目标）

**可选字段**，指定包的发布目标。

**常见值**：

```yaml
# 不发布（默认值，用于应用项目）
publish_to: 'none'

# 发布到 pub.dev（默认值，用于包项目）
# 省略此字段即可

# 发布到私有 pub 服务器
publish_to: https://pub.example.com

# 发布到本地文件系统（测试用）
publish_to: /path/to/local/repo
```

### 2.5 homepage（主页）

**可选字段**，指向项目主页的 URL。

```yaml
homepage: https://example.com/my_package
```

**用途**：
- 在 pub.dev 上显示为项目主页链接
- 通常指向 GitHub Pages 或项目官网

### 2.6 repository（代码仓库）

**可选字段**，指向源代码仓库的 URL。

```yaml
repository: https://github.com/username/my_package
```

**与 homepage 的区别**：
- `homepage`：面向用户的产品页面
- `repository`：面向开发者的源代码页面

**推荐做法**：如果只使用一个，优先使用 `repository`。

### 2.7 issue_tracker（问题追踪器）

**可选字段**，指向问题追踪系统的 URL。

```yaml
issue_tracker: https://github.com/username/my_package/issues
```

**用途**：
- 在 pub.dev 上显示"View/report issues"链接
- 方便用户报告问题和查看已知问题

### 2.8 documentation（文档）

**可选字段**，指向项目文档的 URL。

```yaml
documentation: https://example.com/my_package/docs
```

**用途**：
- 在 pub.dev 上显示文档链接
- 如果省略，pub.dev 会自动生成 API 文档

### 2.9 funding（赞助）

**可选字段**，指定项目的赞助方式。

```yaml
funding:
  - https://github.com/sponsors/username
  - https://www.buymeacoffee.com/username
  - https://patreon.com/username
```

**用途**：
- 在 pub.dev 上显示赞助链接
- 支持开源项目的可持续发展

### 2.10 topics（主题标签）

**可选字段**，为包添加主题标签，便于在 pub.dev 上发现。

```yaml
topics:
  - http
  - network
  - rest-api
  - dio
```

**约束条件**：
- 最多 7 个主题
- 每个主题 2-32 个字符
- 只能包含小写字母、数字和连字符
- 不能以数字开头

### 2.11 platforms（支持平台）

**可选字段**，显式声明包支持的平台。

```yaml
platforms:
  android:
  ios:
  linux:
  macos:
  web:
  windows:
```

**用途**：
- 在 pub.dev 上显示支持的平台徽章
- 帮助用户快速了解包的兼容性

**注意**：对于 Flutter 插件，平台支持通常由插件本身决定，不需要显式声明。

### 2.12 screenshots（截图）

**可选字段**，为包添加截图，在 pub.dev 上展示。

```yaml
screenshots:
  - description: 'The main screen of the app'
    path: screenshots/main_screen.png
  - description: 'Settings page'
    path: screenshots/settings.png
```

**约束条件**：
- 最多 10 张截图
- 支持 PNG、JPEG 和 GIF 格式
- 推荐尺寸：1200x1800 像素（竖屏）或 1800x1200 像素（横屏）

### 2.13 false_secrets（假密钥）

**可选字段**，用于标记看起来像密钥但实际上不是的文件或模式。

```yaml
false_secrets:
  - /lib/src/constants.dart
  - /test/**/fixtures/*
  - example/**/*password*
```

**用途**：
- 防止 pub.dev 的密钥扫描误报
- 避免合法的测试数据被标记为密钥泄露

### 2.14 ignored_advisories（忽略的安全建议）

**可选字段**，指定要忽略的安全建议。

```yaml
ignored_advisories:
  - GHSA-1234-5678-90ab
  - dart-pub-advisory-2024-001
```

**用途**：
- 在已知误报或已评估风险的情况下忽略特定安全警告
- 谨慎使用，确保真正理解安全风险

## 第3章 环境配置字段详解

### 3.1 environment（环境约束）

**必填字段**（Dart 2 起），指定 Dart SDK 版本约束。

#### 3.1.1 SDK 版本约束

```yaml
environment:
  sdk: ^3.6.0
```

**版本约束语法**：

| 语法 | 含义 | 示例 |
|------|------|------|
| `^1.2.3` | 兼容 1.2.3，允许 1.x.x 但不允许 2.0.0 | `^1.2.3` = `>=1.2.3 <2.0.0` |
| `>=1.2.3` | 大于等于 1.2.3 | `>=1.2.3` |
| `>=1.2.3 <2.0.0` | 1.2.3 到 2.0.0 之间 | `>=1.2.3 <2.0.0` |
| `>=1.2.3 <2.0.0 || >=3.0.0` | 多个范围 | 复杂约束 |

Dart Tips 语法小贴士

**Caret (^) 语法详解**

`^` 是 Dart 中最常用的版本约束，称为"Caret 语法"或"兼容版本"。

```yaml
# ^1.2.3 的含义
dependencies:
  http: ^1.2.3  # 等同于 >=1.2.3 <2.0.0
```

**工作原理**：
- 允许更新次版本和修订版本（向后兼容的更新）
- 不允许更新主版本（可能包含破坏性变更）

**特殊情况**：
- `^0.1.2` = `>=0.1.2 <0.2.0`（0.x.x 版本每次次版本更新都可能破坏）
- `^1.0.0` = `>=1.0.0 <2.0.0`

#### 3.1.2 Flutter 版本约束

```yaml
environment:
  sdk: ^3.6.0
  flutter: ^3.27.0
```

**用途**：
- 指定 Flutter SDK 的最低版本要求
- 确保包在特定 Flutter 版本下正常工作

### 3.2 完整的 environment 配置示例

```yaml
environment:
  # Dart SDK 版本约束
  sdk: '>=3.6.0 <4.0.0'
  
  # Flutter SDK 版本约束（Flutter 项目使用）
  flutter: '>=3.27.0'
```

## 第4章 依赖管理字段详解

### 4.1 dependencies（运行时依赖）

**可选字段**，声明项目运行所需的依赖包。

#### 4.1.1 基本依赖声明

```yaml
dependencies:
  # 从 pub.dev 获取包
  http: ^1.2.0
  
  # 指定版本范围
  dio: '>=5.0.0 <6.0.0'
  
  # 精确版本
  uuid: 4.0.0
  
  # 任意版本（不推荐）
  some_package: any
```

#### 4.1.2 SDK 依赖

```yaml
dependencies:
  # Flutter SDK（Flutter 项目必须）
  flutter:
    sdk: flutter
  
  # Flutter 本地化支持
  flutter_localizations:
    sdk: flutter
  
  # Dart SDK（通常不需要显式声明）
  # dart:  # 内置库不需要声明
```

#### 4.1.3 Git 依赖

```yaml
dependencies:
  # 从 Git 仓库获取
  my_package:
    git:
      url: https://github.com/username/my_package.git
  
  # 指定分支
  my_package:
    git:
      url: https://github.com/username/my_package.git
      ref: develop
  
  # 指定标签
  my_package:
    git:
      url: https://github.com/username/my_package.git
      ref: v1.0.0
  
  # 指定提交
  my_package:
    git:
      url: https://github.com/username/my_package.git
      ref: a1b2c3d
  
  # 指定子目录
  my_package:
    git:
      url: https://github.com/username/monorepo.git
      path: packages/my_package
```

#### 4.1.4 本地路径依赖

```yaml
dependencies:
  # 本地相对路径
  my_local_package:
    path: ../my_local_package
  
  # 本地绝对路径
  my_local_package:
    path: /Users/username/projects/my_local_package
```

**使用场景**：
- 开发中的本地包
- 尚未发布的内部包
- 修改第三方包源码

#### 4.1.5 托管依赖（私有 pub 服务器）

```yaml
dependencies:
  my_private_package:
    hosted:
      name: my_private_package
      url: https://pub.example.com
      version: ^1.0.0
```

#### 4.1.6 完整的 dependencies 示例

```yaml
dependencies:
  # Flutter SDK
  flutter:
    sdk: flutter
  
  # Flutter 本地化
  flutter_localizations:
    sdk: flutter
  
  # UI 组件
  cupertino_icons: ^1.0.8
  flutter_svg: ^2.0.9
  
  # 网络请求
  dio: ^5.4.0
  retrofit: ^4.0.3
  
  # 状态管理
  flutter_bloc: ^8.1.3
  provider: ^6.1.1
  
  # 本地存储
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  
  # 路由
  go_router: ^13.0.1
  
  # 工具
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1
  
  # 内部包（本地路径）
  core_package:
    path: ../packages/core
  
  # 内部包（Git）
  auth_package:
    git:
      url: https://github.com/company/auth.git
      ref: v2.0.0
```

### 4.2 dev_dependencies（开发依赖）

**可选字段**，声明仅在开发、测试阶段需要的依赖包，不会被打包到生产应用中。

#### 4.2.1 常见的开发依赖

```yaml
dev_dependencies:
  # 测试框架
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  mockito: ^5.4.4
  
  # 代码生成
  build_runner: ^2.4.8
  freezed: ^2.4.6
  json_serializable: ^6.7.1
  retrofit_generator: ^8.0.6
  
  # 代码质量
  flutter_lints: ^6.0.0
  very_good_analysis: ^5.1.0
  
  # 文档生成
  dartdoc: ^8.0.8
```

#### 4.2.2 dependencies 与 dev_dependencies 的区别

| 特性 | dependencies | dev_dependencies |
|------|--------------|------------------|
| 打包到应用 | 是 | 否 |
| 运行时可用 | 是 | 否（仅开发时） |
| 典型用途 | 功能库、UI 组件 | 测试、代码生成、lint |
| 安装命令 | `flutter pub add package` | `flutter pub add dev:package` |

### 4.3 dependency_overrides（依赖覆盖）

**可选字段**，用于临时覆盖依赖版本，解决版本冲突或测试未发布的包。

#### 4.3.1 基本用法

```yaml
dependency_overrides:
  # 覆盖为特定版本
  http: ^1.2.0
  
  # 覆盖为本地路径
  my_package:
    path: ../my_package
  
  # 覆盖为 Git 版本
  my_package:
    git:
      url: https://github.com/username/my_package.git
      ref: fix-bug-branch
```

#### 4.3.2 使用场景

```yaml
# 场景 1：解决版本冲突
# 包 A 依赖 http ^0.13.0，包 B 依赖 http ^1.0.0
dependency_overrides:
  http: ^1.0.0  # 强制使用 1.0.0

# 场景 2：测试本地修改
# 修改了第三方包的源码，测试是否解决问题
dependency_overrides:
  third_party_package:
    path: ../third_party_package

# 场景 3：使用 fork 版本
dependency_overrides:
  original_package:
    git:
      url: https://github.com/myfork/original_package.git
      ref: my-fix
```

Flutter 框架小知识

**dependency_overrides 的注意事项**

1. **临时使用**：`dependency_overrides` 应该只在开发和测试阶段使用，发布包时不应该包含
2. **团队协作**：如果必须使用，确保团队成员都知道覆盖的原因
3. **文档记录**：在 README 或代码注释中说明为什么需要覆盖
4. **及时移除**：问题解决后及时移除覆盖配置

## 第5章 Flutter 特定配置字段详解

### 5.1 flutter 根节点

Flutter 项目特有的配置都放在 `flutter:` 节点下。

### 5.2 uses-material-design

**可选字段**，启用 Material Design 图标字体。

```yaml
flutter:
  uses-material-design: true
```

**用途**：
- 启用 `Icons` 类中的 Material 图标
- 使用 `Icon(Icons.home)` 等图标时需要

**注意**：如果不使用 Material 图标，可以设置为 `false` 减小应用体积。

### 5.3 generate

**可选字段**，启用代码生成功能。

```yaml
flutter:
  generate: true
```

**用途**：
- 启用本地化字符串的自动生成
- 配合 `l10n.yaml` 使用

### 5.4 config

**可选字段**，配置 Flutter 工具的行为。

```yaml
flutter:
  config:
    enable-swift-package-manager: true
```

**可用配置**：

| 配置项 | 说明 |
|--------|------|
| `enable-swift-package-manager` | 启用 Swift Package Manager（iOS/macOS） |

### 5.5 assets（资源文件）

**可选字段**，声明应用需要的静态资源文件。

#### 5.5.1 基本用法

```yaml
flutter:
  assets:
    # 单个文件
    - assets/logo.png
    
    # 整个目录（递归包含）
    - assets/images/
    
    # 多个目录
    - assets/icons/
    - assets/fonts/
    
    # JSON 文件
    - assets/data/config.json
    
    # 字体文件（也可以在 fonts 中声明）
    - assets/fonts/CustomFont.ttf
```

#### 5.5.2 变体资源（Variant Assets）

Flutter 支持根据设备像素比自动选择合适的资源变体：

```
assets/
  logo.png          # 默认（1x）
  2.0x/logo.png     # 2x 分辨率
  3.0x/logo.png     # 3x 分辨率
```

```yaml
flutter:
  assets:
    - assets/logo.png  # 自动根据设备选择 2.0x 或 3.0x 变体
```

#### 5.5.3 在代码中使用资源

```dart
// 加载图片
Image.asset('assets/logo.png')

// 加载字符串
final String config = await rootBundle.loadString('assets/data/config.json');

// 加载字节
final ByteData data = await rootBundle.load('assets/data/file.bin');
```

### 5.6 fonts（字体）

**可选字段**，声明应用使用的自定义字体。

#### 5.6.1 基本用法

```yaml
flutter:
  fonts:
    # 字体族 1
    - family: Schyler
      fonts:
        - asset: fonts/Schyler-Regular.ttf
        - asset: fonts/Schyler-Italic.ttf
          style: italic
        - asset: fonts/Schyler-Bold.ttf
          weight: 700
    
    # 字体族 2
    - family: Roboto
      fonts:
        - asset: fonts/Roboto-Regular.ttf
        - asset: fonts/Roboto-Medium.ttf
          weight: 500
        - asset: fonts/Roboto-Bold.ttf
          weight: 700
        - asset: fonts/Roboto-Italic.ttf
          style: italic
```

#### 5.6.2 字体属性说明

| 属性 | 说明 | 可选值 |
|------|------|--------|
| `family` | 字体族名称 | 任意字符串 |
| `asset` | 字体文件路径 | 相对于项目根目录的路径 |
| `weight` | 字重 | 100, 200, 300, 400, 500, 600, 700, 800, 900 |
| `style` | 字体样式 | `normal`, `italic` |

#### 5.6.3 在代码中使用字体

```dart
// 使用自定义字体
Text(
  'Hello Flutter',
  style: TextStyle(
    fontFamily: 'Schyler',
    fontWeight: FontWeight.bold,
    fontStyle: FontStyle.italic,
  ),
)

// 在主题中配置
MaterialApp(
  theme: ThemeData(
    fontFamily: 'Roboto',
    textTheme: TextTheme(
      headline1: TextStyle(fontFamily: 'Schyler', fontSize: 24),
    ),
  ),
)
```

### 5.7 licenses（许可证）

**可选字段**，声明额外的许可证文件。

```yaml
flutter:
  licenses:
    - assets/licenses/my_license.txt
    - assets/licenses/third_party_license.txt
```

**用途**：
- 将额外的许可证文件打包到应用中
- 在应用的许可证页面显示

## 第6章 依赖版本管理详解

### 6.1 版本约束详解

#### 6.1.1 版本约束类型

| 约束类型 | 语法 | 说明 |
|----------|------|------|
| 精确版本 | `1.2.3` | 只使用 1.2.3 版本 |
| Caret | `^1.2.3` | >=1.2.3 <2.0.0 |
| Tilde | `~1.2.3` | >=1.2.3 <1.3.0 |
| 大于等于 | `>=1.2.3` | 1.2.3 及以上 |
| 小于 | `<2.0.0` | 2.0.0 以下 |
| 范围 | `>=1.2.3 <2.0.0` | 指定范围 |
| 任意 | `any` | 任意版本（不推荐） |

#### 6.1.2 Caret (^) vs Tilde (~)

```yaml
dependencies:
  # ^1.2.3 = >=1.2.3 <2.0.0
  # 允许次版本和修订版本更新
  http: ^1.2.3
  
  # ~1.2.3 = >=1.2.3 <1.3.0
  # 只允许修订版本更新
  dio: ~1.2.3
```

**选择建议**：
- 使用 `^`（推荐）：获取向后兼容的功能更新和 bug 修复
- 使用 `~`：只在需要严格控制次版本更新时使用

### 6.2 pubspec.lock 文件

`pubspec.lock` 是自动生成的文件，用于锁定依赖的确切版本。

**工作原理**：
1. 第一次运行 `flutter pub get` 时，根据 `pubspec.yaml` 的版本约束解析依赖
2. 将解析后的确切版本写入 `pubspec.lock`
3. 后续运行 `flutter pub get` 时，优先使用 `pubspec.lock` 中的版本

**更新锁定版本**：

```bash
# 更新单个包到最新兼容版本
flutter pub upgrade package_name

# 更新所有包到最新兼容版本
flutter pub upgrade

# 强制更新（忽略 pubspec.lock）
flutter pub upgrade --major-versions
```

**版本控制建议**：
- **应用项目**：提交 `pubspec.lock` 到版本控制
- **库/包项目**：不提交 `pubspec.lock`，让用户自行解析

### 6.3 版本冲突解决

#### 6.3.1 常见冲突场景

```yaml
# 场景：包 A 依赖 http ^0.13.0，包 B 依赖 http ^1.0.0
# 这两个版本范围没有交集，会导致冲突
```

#### 6.3.2 解决方法

```yaml
# 方法 1：使用 dependency_overrides 强制版本
dependency_overrides:
  http: ^1.0.0

# 方法 2：升级依赖包到兼容版本
dependencies:
  package_a: ^2.0.0  # 新版本可能已升级 http 依赖
  package_b: ^1.0.0

# 方法 3：使用 dependency_overrides 指向本地修复版本
dependency_overrides:
  package_a:
    path: ../package_a_fixed
```

#### 6.3.3 诊断版本冲突

```bash
# 查看依赖树
flutter pub deps

# 查看过时的包
flutter pub outdated

# 查看可升级的包
flutter pub upgrade --dry-run
```

## 第7章 pubspec.yaml 实战应用

### 7.1 不同类型项目的配置模板

#### 7.1.1 Flutter 应用项目

```yaml
name: my_flutter_app
description: A production-ready Flutter application.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.6.0
  flutter: ^3.27.0

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  
  # UI
  cupertino_icons: ^1.0.8
  flutter_svg: ^2.0.9
  shimmer: ^3.0.0
  
  # State Management
  flutter_bloc: ^8.1.3
  provider: ^6.1.1
  
  # Navigation
  go_router: ^13.0.1
  
  # Network
  dio: ^5.4.0
  retrofit: ^4.0.3
  
  # Storage
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  
  # Utils
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1
  get_it: ^7.6.4
  injectable: ^2.3.2
  logger: ^2.0.2
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  
  # Code Generation
  build_runner: ^2.4.8
  freezed: ^2.4.6
  json_serializable: ^6.7.1
  retrofit_generator: ^8.0.6
  injectable_generator: ^2.4.1
  hive_generator: ^2.0.1
  
  # Testing
  mockito: ^5.4.4
  bloc_test: ^9.1.5
  
  # Lint
  flutter_lints: ^6.0.0
  very_good_analysis: ^5.1.0

flutter:
  uses-material-design: true
  generate: true
  
  assets:
    - assets/images/
    - assets/icons/
    - assets/fonts/
    - assets/translations/
  
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto-Medium.ttf
          weight: 500
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
```

#### 7.1.2 Flutter 插件项目

```yaml
name: my_flutter_plugin
description: A new Flutter plugin project.
version: 0.0.1
homepage: https://example.com
repository: https://github.com/username/my_flutter_plugin

environment:
  sdk: ^3.6.0
  flutter: '>=3.27.0'

flutter:
  plugin:
    platforms:
      android:
        package: com.example.my_flutter_plugin
        pluginClass: MyFlutterPlugin
      ios:
        pluginClass: MyFlutterPlugin
      linux:
        pluginClass: MyFlutterPlugin
      macos:
        pluginClass: MyFlutterPlugin
      windows:
        pluginClass: MyFlutterPluginCApi
      web:
        pluginClass: MyFlutterPluginWeb
        fileName: my_flutter_plugin_web.dart

dependencies:
  flutter:
    sdk: flutter
  plugin_platform_interface: ^2.1.8

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0

topics:
  - plugin
  - platform
```

#### 7.1.3 Dart 包项目

```yaml
name: my_dart_package
description: A starting point for Dart libraries or applications.
version: 1.0.0
homepage: https://example.com
repository: https://github.com/username/my_dart_package
issue_tracker: https://github.com/username/my_dart_package/issues
documentation: https://example.com/docs

environment:
  sdk: ^3.6.0

dependencies:
  http: ^1.2.0
  meta: ^1.11.0

dev_dependencies:
  lints: ^6.0.0
  test: ^1.25.0
  mockito: ^5.4.4
  build_runner: ^2.4.8

topics:
  - http
  - network
  - utils
```

### 7.2 常用命令

#### 7.2.1 依赖管理命令

```bash
# 获取依赖
flutter pub get

# 升级依赖
flutter pub upgrade

# 升级所有依赖到最新主版本
flutter pub upgrade --major-versions

# 查看依赖树
flutter pub deps

# 查看过时的包
flutter pub outdated

# 添加依赖
flutter pub add package_name

# 添加开发依赖
flutter pub add dev:package_name

# 移除依赖
flutter pub remove package_name
```

#### 7.2.2 包发布命令

```bash
# 验证包
flutter pub publish --dry-run

# 发布包
flutter pub publish

# 登录 pub.dev
flutter pub login

# 登出
flutter pub logout
```

### 7.3 最佳实践

#### 7.3.1 依赖管理最佳实践

1. **使用精确的版本约束**
   ```yaml
   # 推荐
   http: ^1.2.0
   
   # 不推荐
   http: any
   http: '>=1.0.0'
   ```

2. **定期更新依赖**
   ```bash
   # 每月运行一次
   flutter pub outdated
   flutter pub upgrade
   ```

3. **审查新依赖**
   - 检查包的维护状态
   - 查看 GitHub 上的 star 数和 issue 数量
   - 阅读文档和示例代码

4. **最小化依赖**
   - 只添加必要的依赖
   - 优先使用 Dart/Flutter 内置功能
   - 考虑依赖的体积影响

#### 7.3.2 资源配置最佳实践

1. **组织资源目录**
   ```
   assets/
   ├── images/          # 图片资源
   │   ├── onboarding/
   │   ├── products/
   │   └── users/
   ├── icons/           # 图标资源
   ├── fonts/           # 字体文件
   ├── animations/      # 动画文件
   └── data/            # JSON 数据文件
   ```

2. **使用适当的图片格式**
   - 图标：SVG（矢量）
   - 照片：JPEG
   - 透明图片：PNG
   - 动画：Lottie JSON

3. **提供多分辨率图片**
   ```
   assets/images/
   ├── logo.png         # 1x
   ├── 2.0x/logo.png    # 2x
   └── 3.0x/logo.png    # 3x
   ```

#### 7.3.3 版本控制最佳实践

1. **提交的文件**
   ```
   # 应该提交
   pubspec.yaml
   pubspec.lock（应用项目）
   
   # 不应该提交
   .dart_tool/
   .packages
   build/
   ```

2. **.gitignore 配置**
   ```
   # Dart/Flutter
   .dart_tool/
   .packages
   build/
   pubspec.lock  # 库项目不提交
   
   # IDE
   .idea/
   .vscode/
   *.iml
   
   # 其他
   .flutter-plugins
   .flutter-plugins-dependencies
   ```

## 附录 A：pubspec.yaml 完整字段速查表

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 包名称 |
| `version` | 发布时 | 版本号 |
| `description` | 发布时 | 包描述 |
| `publish_to` | 否 | 发布目标 |
| `homepage` | 否 | 主页 URL |
| `repository` | 否 | 代码仓库 URL |
| `issue_tracker` | 否 | 问题追踪器 URL |
| `documentation` | 否 | 文档 URL |
| `funding` | 否 | 赞助链接列表 |
| `topics` | 否 | 主题标签列表 |
| `platforms` | 否 | 支持平台 |
| `screenshots` | 否 | 截图列表 |
| `false_secrets` | 否 | 假密钥模式 |
| `ignored_advisories` | 否 | 忽略的安全建议 |
| `environment` | 是 | SDK 约束 |
| `dependencies` | 否 | 运行时依赖 |
| `dev_dependencies` | 否 | 开发依赖 |
| `dependency_overrides` | 否 | 依赖覆盖 |
| `executables` | 否 | 可执行脚本 |
| `flutter` | Flutter | Flutter 配置 |
| `flutter:uses-material-design` | 否 | 启用 Material 图标 |
| `flutter:generate` | 否 | 启用代码生成 |
| `flutter:config` | 否 | Flutter 工具配置 |
| `flutter:assets` | 否 | 资源文件列表 |
| `flutter:fonts` | 否 | 字体配置 |
| `flutter:licenses` | 否 | 许可证文件 |

## 附录 B：常见问题与解决方案

### Q1：如何添加 GitHub 上的包？

```yaml
dependencies:
  my_package:
    git:
      url: https://github.com/username/repo.git
      ref: main  # 分支、标签或提交
```

### Q2：如何解决版本冲突？

```yaml
# 使用 dependency_overrides 强制版本
dependency_overrides:
  http: ^1.0.0
```

### Q3：如何添加本地包？

```yaml
dependencies:
  my_local_package:
    path: ../my_local_package
```

### Q4：如何查看依赖树？

```bash
flutter pub deps
```

### Q5：如何更新所有依赖到最新版本？

```bash
flutter pub upgrade --major-versions
```

### Q6：pubspec.yaml 修改后需要做什么？

```bash
flutter pub get
```

### Q7：如何发布包到 pub.dev？

```bash
flutter pub publish --dry-run  # 先验证
flutter pub publish            # 正式发布
```

### Q8：如何配置私有 pub 服务器？

```yaml
dependencies:
  my_package:
    hosted:
      name: my_package
      url: https://pub.example.com
      version: ^1.0.0
```

## 附录 C：参考资源

- [Dart pubspec 文档](https://dart.dev/tools/pub/pubspec)
- [Flutter pubspec 选项](https://docs.flutter.dev/tools/pubspec)
- [pub.dev](https://pub.dev)
- [语义化版本控制](https://semver.org/lang/zh-CN/)
- [YAML 规范](https://yaml.org/)
