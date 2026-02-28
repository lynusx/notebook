# flutter_animate 4.5.2 详解与实战

## 第1章 认识 flutter_animate

### 1.1 什么是 flutter_animate

`flutter_animate` 是由 gskinner.com 开发的高性能 Flutter 动画库，它提供了一种简单、统一的方式来为 Flutter 应用添加几乎任何类型的动画效果。该包是 Flutter Gems 推荐的动画库，被广泛应用于生产环境。

与传统的 Flutter 动画实现方式相比，`flutter_animate` 提供了以下核心优势：

| 特性       | flutter_animate  | 传统方式                         |
| ---------- | ---------------- | -------------------------------- |
| 代码量     | 极少（一行代码） | 较多（需要 AnimationController） |
| 学习曲线   | 平缓             | 较陡峭                           |
| 预置效果   | 20+ 种内置效果   | 需要手动实现                     |
| 链式语法   | 支持             | 不支持                           |
| 列表动画   | 内置支持         | 需要额外实现                     |
| 自定义效果 | 简单             | 较复杂                           |
| 性能       | 优化             | 依赖实现                         |

Flutter 框架小知识

**Flutter 动画的演进**

Flutter 提供了多种动画实现方式：

1. **隐式动画（Implicit Animation）**：使用 AnimatedContainer、AnimatedOpacity 等，适合简单场景
2. **显式动画（Explicit Animation）**：使用 AnimationController 和 Tween，需要 StatefulWidget
3. **flutter_animate**：在隐式动画的简洁性和显式动画的灵活性之间取得平衡

`flutter_animate` 的设计哲学是：

- 让简单动画变得极其简单
- 让复杂动画变得可控
- 无需管理 AnimationController 的生命周期

### 1.2 flutter_animate 4.5.2 的新特性

版本 4.5.2 是 `flutter_animate` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **20+ 种预置动画效果**：fade、scale、slide、align、flip、blur、shake、shimmer、shadows 等
- **GLSL 着色器支持**：可以将 GLSL 片段着色器应用到 Widget
- **简化的动画构建器**：易于创建自定义效果
- **同步动画**：可以将动画同步到滚动、通知器或其他数据源
- **集成事件**：支持 onPlay、onComplete 等回调

#### 1.2.2 新增效果（4.x 版本）

| 效果               | 说明            |
| ------------------ | --------------- |
| `AlignEffect`      | 动画对齐方式    |
| `FollowPathEffect` | 沿路径移动      |
| `ShaderEffect`     | GLSL 着色器效果 |
| `CrossfadeEffect`  | 交叉淡入淡出    |
| `ColorEffect`      | 颜色动画        |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_animate: ^4.5.2
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 导入包

```dart
import 'package:flutter_animate/flutter_animate.dart';
```

Dart Tips 语法小贴士

**Duration 扩展方法**

`flutter_animate` 为 `num` 类型添加了便捷的 Duration 扩展方法：

```dart
// 毫秒
300.ms     // Duration(milliseconds: 300)
500.ms     // Duration(milliseconds: 500)

// 秒
2.seconds  // Duration(seconds: 2)
0.5.seconds // Duration(milliseconds: 500)

// 分钟
1.minutes  // Duration(minutes: 1)
```

这些扩展方法让动画时长的设置更加直观和简洁。

## 第2章 基础使用方式

### 2.1 两种使用风格

`flutter_animate` 支持两种等价的语法风格，可以根据个人喜好选择。

#### 2.1.1 声明式风格（Declarative）

使用 `Animate` Widget 包裹目标 Widget，并指定效果列表：

```dart
Animate(
  effects: [
    FadeEffect(),
    ScaleEffect(),
  ],
  child: Text("Hello World!"),
)
```

#### 2.1.2 链式风格（Chained）

使用 `.animate()` 扩展方法，然后链式调用效果：

```dart
Text("Hello World!").animate().fade().scale()
```

**注意**：两种风格功能完全等价，本文档主要使用链式风格，因为它更简洁。

### 2.2 基础动画示例

#### 2.2.1 淡入动画（Fade）

```dart
// 淡入效果
Text("Hello").animate().fadeIn()

// 淡出效果
Text("Goodbye").animate().fadeOut()

// 自定义淡入
Text("Hello").animate().fadeIn(
  duration: 500.ms,
  curve: Curves.easeIn,
)
```

#### 2.2.2 缩放动画（Scale）

```dart
// 缩放进入
Container(
  width: 100,
  height: 100,
  color: Colors.red,
).animate().scale()

// 自定义缩放
Container(
  width: 100,
  height: 100,
  color: Colors.blue,
).animate().scale(
  begin: 0.5,  // 从 50% 开始
  end: 1.0,    // 到 100%
  duration: 800.ms,
  curve: Curves.elasticOut,
)
```

#### 2.2.3 滑动动画（Slide）

```dart
// 从下方滑入
Text("Slide Up").animate().slideY(
  begin: 1.0,  // 从下方一个屏幕高度
  end: 0.0,    // 到原位
)

// 从左侧滑入
Text("Slide In").animate().slideX(
  begin: -1.0, // 从左侧一个屏幕宽度
  end: 0.0,
)

// 对角线滑入
Text("Diagonal").animate().slide(
  begin: Offset(-0.5, 0.5),
  end: Offset.zero,
)
```

## 第3章 Animate Widget 详解

`Animate` 是 `flutter_animate` 包的核心 Widget，负责管理和执行动画效果。

### 3.1 构造函数参数

```dart
Animate({
  Key? key,
  List<Effect>? effects,           // 动画效果列表
  AnimationController? controller, // 外部动画控制器
  bool autoPlay = true,            // 是否自动播放
  void Function(AnimationController)? onPlay,   // 播放回调
  void Function(AnimationController)? onComplete, // 完成回调
  void Function(AnimationController)? onInit,    // 初始化回调
  Widget? child,                   // 子 Widget
})
```

### 3.2 参数详解

#### 3.2.1 effects（效果列表）

```dart
Animate(
  effects: [
    FadeEffect(duration: 300.ms),
    ScaleEffect(delay: 300.ms, duration: 500.ms),
    SlideEffect(delay: 800.ms),
  ],
  child: MyWidget(),
)
```

**效果继承规则**：

- 第一个效果使用 `Animate.defaultDuration` 和 `Animate.defaultCurve`
- 后续效果继承前一个效果的 `delay`、`duration` 和 `curve`（如果未指定）

#### 3.2.2 autoPlay（自动播放）

```dart
// 自动播放（默认）
Text("Auto").animate().fade()

// 不自动播放
Text("Manual").animate(
  autoPlay: false,
).fade()
```

#### 3.2.3 onPlay（播放回调）

```dart
// 循环播放
Text("Loop").animate(
  onPlay: (controller) => controller.repeat(),
).fade()

// 往返循环
Text("Reverse Loop").animate(
  onPlay: (controller) => controller.repeat(reverse: true),
).fade()

// 循环指定次数
Text("Loop 3 times").animate(
  onPlay: (controller) => controller.loop(count: 3),
).fade()
```

#### 3.2.4 onComplete（完成回调）

```dart
Text("Done").animate(
  onComplete: (controller) {
    print("Animation completed!");
  },
).fade(duration: 1000.ms)
```

#### 3.2.5 onInit（初始化回调）

```dart
Text("Initialized").animate(
  onInit: (controller) {
    // 在动画初始化时执行
    print("Animation initialized");
  },
).fade()
```

### 3.3 默认值配置

```dart
// 设置全局默认值
Animate.defaultDuration = 300.ms;
Animate.defaultCurve = Curves.easeOut;

// 使用默认值
Text("Default").animate().fade() // 300ms, easeOut
```

## 第4章 预置动画效果详解

### 4.1 淡入淡出效果（FadeEffect）

#### 4.1.1 方法签名

```dart
Animate fadeIn({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 1.0,
})

Animate fadeOut({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 1.0,
  double end = 0.0,
})
```

#### 4.1.2 参数说明

| 参数       | 类型        | 默认值      | 说明         |
| ---------- | ----------- | ----------- | ------------ |
| `delay`    | `Duration?` | `null`      | 延迟开始时间 |
| `duration` | `Duration?` | `null`      | 动画持续时间 |
| `curve`    | `Curve?`    | `null`      | 动画曲线     |
| `begin`    | `double`    | `0.0`/`1.0` | 起始不透明度 |
| `end`      | `double`    | `1.0`/`0.0` | 结束不透明度 |

#### 4.1.3 使用示例

```dart
// 淡入
Text("Fade In").animate().fadeIn()

// 淡出
Text("Fade Out").animate().fadeOut()

// 部分淡入
Text("Partial").animate().fadeIn(begin: 0.3, end: 0.8)

// 带延迟的淡入
Text("Delayed").animate().fadeIn(delay: 500.ms)
```

### 4.2 缩放效果（ScaleEffect）

#### 4.2.1 方法签名

```dart
Animate scale({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 1.0,
  Alignment alignment = Alignment.center,
  bool transformHitTests = true,
})
```

#### 4.2.2 参数说明

| 参数                | 类型        | 默认值             | 说明             |
| ------------------- | ----------- | ------------------ | ---------------- |
| `begin`             | `double`    | `0.0`              | 起始缩放比例     |
| `end`               | `double`    | `1.0`              | 结束缩放比例     |
| `alignment`         | `Alignment` | `Alignment.center` | 缩放中心点       |
| `transformHitTests` | `bool`      | `true`             | 是否变换点击测试 |

#### 4.2.3 使用示例

```dart
// 从 0 缩放到 1
Container(
  width: 100,
  height: 100,
  color: Colors.red,
).animate().scale()

// 弹性缩放
Container(
  width: 100,
  height: 100,
  color: Colors.blue,
).animate().scale(
  curve: Curves.elasticOut,
  duration: 800.ms,
)

// 从左上角缩放
Container(
  width: 100,
  height: 100,
  color: Colors.green,
).animate().scale(
  alignment: Alignment.topLeft,
)

// 缩小效果
Container(
  width: 100,
  height: 100,
  color: Colors.orange,
).animate().scale(begin: 1.5, end: 1.0)
```

### 4.3 滑动效果（SlideEffect）

#### 4.3.1 方法签名

```dart
Animate slide({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  Offset begin = const Offset(1.0, 0.0),
  Offset end = Offset.zero,
})

Animate slideX({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 1.0,
  double end = 0.0,
})

Animate slideY({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 1.0,
  double end = 0.0,
})
```

#### 4.3.2 参数说明

| 参数    | 类型              | 默认值                   | 说明                           |
| ------- | ----------------- | ------------------------ | ------------------------------ |
| `begin` | `Offset`/`double` | `Offset(1.0, 0.0)`/`1.0` | 起始位置（相对于 Widget 大小） |
| `end`   | `Offset`/`double` | `Offset.zero`/`0.0`      | 结束位置                       |

**注意**：`1.0` 表示 Widget 的一个完整宽度或高度。

#### 4.3.3 使用示例

```dart
// 从右侧滑入
Text("From Right").animate().slideX(begin: 1.0, end: 0.0)

// 从左侧滑入
Text("From Left").animate().slideX(begin: -1.0, end: 0.0)

// 从下方滑入
Text("From Bottom").animate().slideY(begin: 1.0, end: 0.0)

// 从上方滑入
Text("From Top").animate().slideY(begin: -1.0, end: 0.0)

// 对角线滑入
Text("Diagonal").animate().slide(
  begin: Offset(-0.5, 0.5),
  end: Offset.zero,
)
```

### 4.4 对齐效果（AlignEffect）

#### 4.4.1 方法签名

```dart
Animate align({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  Alignment begin = Alignment.topCenter,
  Alignment end = Alignment.center,
})
```

#### 4.4.2 使用示例

```dart
// 从顶部居中移动到中心
Container(
  width: 100,
  height: 100,
  color: Colors.purple,
).animate().align()

// 自定义对齐
Container(
  width: 100,
  height: 100,
  color: Colors.teal,
).animate().align(
  begin: Alignment.bottomRight,
  end: Alignment.topLeft,
)
```

### 4.5 旋转效果（RotateEffect）

#### 4.5.1 方法签名

```dart
Animate rotate({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 1.0,
  Alignment alignment = Alignment.center,
  bool transformHitTests = true,
})
```

#### 4.5.2 参数说明

| 参数    | 类型     | 默认值 | 说明             |
| ------- | -------- | ------ | ---------------- |
| `begin` | `double` | `0.0`  | 起始角度（圈数） |
| `end`   | `double` | `1.0`  | 结束角度（圈数） |

**注意**：`1.0` 表示一圈（360度）。

#### 4.5.3 使用示例

```dart
// 旋转一圈
Icon(Icons.refresh).animate().rotate()

// 旋转 180 度
Icon(Icons.flip).animate().rotate(end: 0.5)

// 连续旋转
Icon(Icons.sync).animate(
  onPlay: (controller) => controller.repeat(),
).rotate(duration: 2.seconds)
```

### 4.6 翻转效果（FlipEffect）

#### 4.6.1 方法签名

```dart
Animate flip({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 1.0,
  FlipDirection direction = FlipDirection.h,
})

Animate flipH({...}) // 水平翻转
Animate flipV({...}) // 垂直翻转
```

#### 4.6.2 使用示例

```dart
// 水平翻转
Card(
  child: Text("Flip"),
).animate().flipH()

// 垂直翻转
Card(
  child: Text("Flip"),
).animate().flipV()

// 3D 翻转效果
Container(
  width: 100,
  height: 100,
  color: Colors.amber,
).animate().flip(
  duration: 800.ms,
  curve: Curves.easeInOut,
)
```

### 4.7 模糊效果（BlurEffect）

#### 4.7.1 方法签名

```dart
Animate blur({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 4.0,
  BlurStyle style = BlurStyle.normal,
})
```

#### 4.7.2 使用示例

```dart
// 模糊进入
Image.network('https://example.com/image.jpg')
  .animate()
  .blur(begin: 10.0, end: 0.0)

// 模糊退出
Text("Blurry").animate().blur(end: 8.0)

// 毛玻璃效果
Container(
  width: 200,
  height: 200,
  color: Colors.white.withOpacity(0.3),
).animate().blur(
  end: 10.0,
  style: BlurStyle.overlay,
)
```

### 4.8 抖动效果（ShakeEffect）

#### 4.8.1 方法签名

```dart
Animate shake({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double offset = 8.0,
  double rotation = 0.1,
  double hz = 5.0,
})
```

#### 4.8.2 参数说明

| 参数       | 类型     | 默认值 | 说明                 |
| ---------- | -------- | ------ | -------------------- |
| `offset`   | `double` | `8.0`  | 抖动偏移量（像素）   |
| `rotation` | `double` | `0.1`  | 旋转幅度（圈数）     |
| `hz`       | `double` | `5.0`  | 抖动频率（每秒次数） |

#### 4.8.3 使用示例

```dart
// 错误提示抖动
TextField(
  decoration: InputDecoration(
    errorText: "Invalid input",
  ),
).animate().shake()

// 自定义抖动
ElevatedButton(
  onPressed: () {},
  child: Text("Shake"),
).animate().shake(
  offset: 12.0,
  hz: 8.0,
  duration: 500.ms,
)

// 持续抖动
Icon(Icons.notification_important)
  .animate(onPlay: (c) => c.repeat(reverse: true))
  .shake(hz: 3.0)
```

### 4.9 闪光效果（ShimmerEffect）

#### 4.9.1 方法签名

```dart
Animate shimmer({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  Color color = Colors.white,
  double size = 0.5,
  double angle = -0.3,
})
```

#### 4.9.2 使用示例

```dart
// 加载占位闪光
Container(
  width: 200,
  height: 20,
  color: Colors.grey[300],
).animate(onPlay: (c) => c.repeat()).shimmer()

// 自定义闪光
Card(
  child: ListTile(
    title: Text("Shimmer"),
  ),
).animate(onPlay: (c) => c.repeat()).shimmer(
  color: Colors.purple.withOpacity(0.5),
  duration: 2.seconds,
)
```

### 4.10 阴影效果（BoxShadowEffect）

#### 4.10.1 方法签名

```dart
Animate boxShadow({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  BoxShadow? begin,
  BoxShadow? end,
  BorderRadius? borderRadius,
})
```

#### 4.10.2 使用示例

```dart
// 阴影出现
Container(
  width: 100,
  height: 100,
  color: Colors.white,
).animate().boxShadow(
  end: BoxShadow(
    color: Colors.black26,
    blurRadius: 20,
    offset: Offset(0, 10),
  ),
)

// 阴影变化
Card(
  child: Text("Shadow"),
).animate().boxShadow(
  begin: BoxShadow(
    color: Colors.black12,
    blurRadius: 5,
    offset: Offset(0, 2),
  ),
  end: BoxShadow(
    color: Colors.black38,
    blurRadius: 15,
    offset: Offset(0, 8),
  ),
)
```

### 4.11 颜色效果（ColorEffect）

#### 4.11.1 方法签名

```dart
Animate tint({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  Color? begin,
  Color? end = const Color(0x800099FF),
  BlendMode blendMode = BlendMode.srcIn,
})

Animate saturate({...})  // 饱和度
Animate desaturate({...}) // 去饱和
```

#### 4.11.2 使用示例

```dart
// 色调变化
Image.asset('photo.jpg').animate().tint(
  color: Colors.purple,
  end: 0.5,
)

// 饱和度变化
Image.asset('photo.jpg').animate().desaturate()

// 颜色动画
Container(
  width: 100,
  height: 100,
  color: Colors.red,
).animate().tint(
  begin: Colors.red,
  end: Colors.blue,
  duration: 2.seconds,
)
```

### 4.12 可见性效果（VisibilityEffect）

#### 4.12.1 方法签名

```dart
Animate hide({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 1.0,
  double end = 0.0,
  bool maintain = false,
})

Animate show({...})
```

#### 4.12.2 使用示例

```dart
// 隐藏
Text("Hide me").animate().hide()

// 显示
Text("Show me").animate().show()

// 保持布局
Text("Maintain").animate().hide(maintain: true)
```

## 第5章 高级效果详解

### 5.1 ThenEffect（序列效果）

`ThenEffect` 是一个特殊的"效果"，用于建立新的时间基准，使后续效果顺序执行。

#### 5.1.1 方法签名

```dart
Animate then({
  Duration? delay,
  Duration? duration,
  Curve? curve,
})
```

#### 5.1.2 工作原理

```dart
// 不使用 then - 效果并行执行
Text("Parallel").animate()
  .fade(duration: 600.ms)      // 0-600ms
  .scale(delay: 200.ms)         // 200-500ms (并行)

// 使用 then - 效果顺序执行
Text("Sequential").animate()
  .fadeIn(duration: 600.ms)    // 0-600ms
  .then(delay: 200.ms)          // 等待到 800ms
  .scale()                       // 800-1100ms
  .then()                        // 等待到 1100ms
  .slide()                       // 1100-1400ms
```

#### 5.1.3 使用示例

```dart
// 复杂的入场动画
Container(
  width: 100,
  height: 100,
  color: Colors.red,
).animate()
  .fadeIn(duration: 300.ms)     // 淡入
  .then()                        // 等待淡入完成
  .scale(duration: 400.ms)       // 缩放
  .then(delay: 100.ms)           // 等待
  .rotate(duration: 500.ms)      // 旋转
  .then()                        // 等待
  .shake(duration: 300.ms)       // 抖动

// 收集品闪光动画
collectible.animate(
  onPlay: (controller) => controller.repeat(),
).shimmer(delay: 4000.ms, duration: 1800.ms)
  .shake(hz: 4, curve: Curves.easeInOutCubic)
  .scale(begin: 1.0, end: 1.1, duration: 600.ms)
  .then(delay: 600.ms)
  .scale(begin: 1.0, end: 1 / 1.1)
```

### 5.2 SwapEffect（交换效果）

`SwapEffect` 允许在动画过程中将目标 Widget 替换为另一个 Widget。

#### 5.2.1 方法签名

```dart
Animate swap({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  required WidgetBuilder builder,
})
```

#### 5.2.2 使用示例

```dart
// 加载完成后显示内容
CircularProgressIndicator()
  .animate()
  .fadeOut(duration: 300.ms)
  .swap(builder: (context) => Text("Loaded!"))

// 图标变化
Icon(Icons.favorite_border)
  .animate()
  .scale(begin: 1.0, end: 0.0)
  .swap(builder: (context) => Icon(Icons.favorite))
  .scale(begin: 0.0, end: 1.0)
```

### 5.3 CrossfadeEffect（交叉淡入淡出）

`CrossfadeEffect` 在两个 Widget 之间执行交叉淡入淡出。

#### 5.3.1 方法签名

```dart
Animate crossfade({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  required WidgetBuilder builder,
})
```

#### 5.3.2 使用示例

```dart
// 图片切换
Image.asset('image1.jpg')
  .animate()
  .crossfade(
    duration: 500.ms,
    builder: (context) => Image.asset('image2.jpg'),
  )
```

### 5.4 FollowPathEffect（路径跟随）

`FollowPathEffect` 使 Widget 沿指定路径移动。

#### 5.4.1 方法签名

```dart
Animate followPath({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  required Path path,
  bool rotate = false,
  Offset offset = Offset.zero,
})
```

#### 5.4.2 使用示例

```dart
// 沿曲线路径移动
Container(
  width: 20,
  height: 20,
  decoration: BoxDecoration(
    color: Colors.red,
    shape: BoxShape.circle,
  ),
).animate().followPath(
  path: Path()
    ..moveTo(0, 0)
    ..quadraticBezierTo(100, 200, 200, 0),
  duration: 2.seconds,
  rotate: true,
)
```

### 5.5 CustomEffect（自定义效果）

`CustomEffect` 允许创建完全自定义的动画效果。

#### 5.5.1 方法签名

```dart
Animate custom({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  required CustomEffectBuilder builder,
})
```

#### 5.5.2 使用示例

```dart
// 脉冲效果
Text("Pulse")
  .animate(onPlay: (c) => c.repeat(reverse: true))
  .custom(
    duration: 1.seconds,
    builder: (context, value, child) {
      return Container(
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(20),
          boxShadow: [
            BoxShadow(
              color: Colors.purple.withOpacity(0.5 * value),
              blurRadius: 20 * value,
              spreadRadius: 5 * value,
            ),
          ],
        ),
        child: child,
      );
    },
  )

// 波浪文字
Text("Wave")
  .animate()
  .custom(
    duration: 1.seconds,
    builder: (context, value, child) {
      return Transform.translate(
        offset: Offset(0, math.sin(value * math.pi * 4) * 10),
        child: child,
      );
    },
  )
```

### 5.6 ToggleEffect（切换效果）

`ToggleEffect` 在动画过程中切换布尔值。

#### 5.6.1 使用示例

```dart
// 控制可见性
Text("Toggle").animate().toggle(
  builder: (context, value, child) {
    return Opacity(
      opacity: value ? 1.0 : 0.0,
      child: child,
    );
  },
)
```

## 第6章 回调与监听效果

### 6.1 CallbackEffect（回调效果）

`CallbackEffect` 在动画的特定时间点调用回调函数。

#### 6.1.1 方法签名

```dart
Animate callback({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  required void Function(bool isReverse) callback,
})
```

#### 6.1.2 使用示例

```dart
// 动画中途回调
Text("Hello").animate()
  .fadeIn(duration: 600.ms)
  .callback(
    duration: 300.ms,
    callback: (isReverse) => print('Halfway!'),
  )

// 动画完成回调
Text("Done").animate()
  .scale(duration: 400.ms)
  .callback(callback: (_) => print('Scale complete'))
```

### 6.2 ListenEffect（监听效果）

`ListenEffect` 注册一个回调来接收动画值。

#### 6.2.1 方法签名

```dart
Animate listen({
  Duration? delay,
  Duration? duration,
  Curve? curve,
  double begin = 0.0,
  double end = 1.0,
  required void Function(double value) callback,
})
```

#### 6.2.2 使用示例

```dart
// 监听动画值
double currentOpacity = 0.0;

Text("Listen")
  .animate()
  .fadeIn(curve: Curves.easeOutExpo)
  .listen(callback: (value) {
    currentOpacity = value;
    print('Current opacity: $value');
  })
```

## 第7章 列表动画（AnimateList）

### 7.1 基本用法

`AnimateList` 用于为列表中的每个 Widget 应用动画效果，并可以设置动画间隔。

#### 7.1.1 声明式风格

```dart
Column(
  children: AnimateList(
    interval: 100.ms,  // 每个子项动画间隔
    effects: [FadeEffect(), ScaleEffect()],
    children: [
      Text("Item 1"),
      Text("Item 2"),
      Text("Item 3"),
    ],
  ),
)
```

#### 7.1.2 链式风格

```dart
Column(
  children: [
    Text("Item 1"),
    Text("Item 2"),
    Text("Item 3"),
  ].animate(interval: 100.ms).fade().scale(),
)
```

### 7.2 参数详解

| 参数         | 类型            | 默认值 | 说明         |
| ------------ | --------------- | ------ | ------------ |
| `interval`   | `Duration?`     | `null` | 子项动画间隔 |
| `effects`    | `List<Effect>?` | `null` | 效果列表     |
| `autoPlay`   | `bool`          | `true` | 是否自动播放 |
| `onPlay`     | `Function?`     | `null` | 播放回调     |
| `onComplete` | `Function?`     | `null` | 完成回调     |
| `onInit`     | `Function?`     | `null` | 初始化回调   |

### 7.3 使用示例

```dart
// 列表项依次进入
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(items[index]),
    );
  },
).animate(interval: 50.ms).fade().slideX()

// 网格动画
GridView.count(
  crossAxisCount: 2,
  children: images.map((img) {
    return Image.asset(img);
  }).toList(),
).animate(interval: 100.ms).scale().fade()

// 带延迟的列表动画
Column(
  children: [
    Header(),
    ...items.map((item) => ItemWidget(item)),
    Footer(),
  ].animate(interval: 80.ms).fadeIn().slideY(),
)
```

## 第8章 适配器（Adapter）

适配器提供了一种机制，可以将动画与任意数据源同步，如滚动、滑块输入等。

### 8.1 ScrollAdapter（滚动适配器）

将动画与滚动位置同步。

```dart
// 创建滚动控制器
final scrollController = ScrollController();

// 创建适配器
final scrollAdapter = ScrollAdapter(
  scrollController,
  begin: 0,      // 起始滚动位置
  end: 200,      // 结束滚动位置
);

// 使用适配器
Text("Parallax")
  .animate(adapter: scrollAdapter)
  .fade()
  .slideY(begin: 0.5, end: -0.5)
```

### 8.2 ValueNotifierAdapter（值通知适配器）

将动画与 ValueNotifier 同步。

```dart
// 创建 ValueNotifier
final sliderValue = ValueNotifier<double>(0.0);

// 创建适配器
final adapter = ValueNotifierAdapter(
  sliderValue,
  min: 0.0,
  max: 1.0,
);

// 使用适配器
Container()
  .animate(adapter: adapter)
  .rotate(begin: 0, end: 2)

// 在 Slider 中更新
Slider(
  value: sliderValue.value,
  onChanged: (value) => sliderValue.value = value,
)
```

### 8.3 AnimationControllerAdapter

将动画与外部 AnimationController 同步。

```dart
// 创建控制器
final controller = AnimationController(vsync: this);

// 创建适配器
final adapter = AnimationControllerAdapter(controller);

// 使用适配器
Text("Controlled")
  .animate(adapter: adapter)
  .fade()
  .scale()
```

## 第9章 flutter_animate 实战应用

### 9.1 页面入场动画

```dart
class AnimatedPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 标题动画
          Text("Welcome")
            .animate()
            .fadeIn(duration: 600.ms)
            .slideY(begin: -0.5, end: 0),

          // 副标题动画
          Text("Get started today")
            .animate(delay: 200.ms)
            .fadeIn()
            .slideY(begin: 0.3, end: 0),

          // 按钮动画
          ElevatedButton(
            onPressed: () {},
            child: Text("Start"),
          ).animate(delay: 400.ms).scale().fadeIn(),
        ],
      ),
    );
  }
}
```

### 9.2 列表加载动画

```dart
class AnimatedListPage extends StatelessWidget {
  final List<String> items = List.generate(20, (i) => "Item $i");

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(items[index]),
          leading: CircleAvatar(child: Text('$index')),
        )
        .animate(
          delay: (index * 50).ms,  // 渐进式延迟
        )
        .fadeIn()
        .slideX(begin: -0.2, end: 0);
      },
    );
  }
}
```

### 9.3 按钮交互动画

```dart
class AnimatedButton extends StatefulWidget {
  @override
  _AnimatedButtonState createState() => _AnimatedButtonState();
}

class _AnimatedButtonState extends State<AnimatedButton> {
  bool isPressed = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => setState(() => isPressed = true),
      onTapUp: (_) => setState(() => isPressed = false),
      onTapCancel: () => setState(() => isPressed = false),
      child: Container(
        padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
        decoration: BoxDecoration(
          color: Colors.blue,
          borderRadius: BorderRadius.circular(8),
        ),
        child: Text("Press Me", style: TextStyle(color: Colors.white)),
      )
      .animate(target: isPressed ? 1 : 0)
      .scale(begin: 1.0, end: 0.95)
      .boxShadow(
        end: BoxShadow(
          color: Colors.blue.withOpacity(0.4),
          blurRadius: 20,
          offset: Offset(0, 10),
        ),
      ),
    );
  }
}
```

### 9.4 错误提示动画

```dart
class ErrorMessage extends StatelessWidget {
  final String message;

  ErrorMessage(this.message);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      color: Colors.red[100],
      child: Row(
        children: [
          Icon(Icons.error, color: Colors.red),
          SizedBox(width: 8),
          Text(message, style: TextStyle(color: Colors.red[800])),
        ],
      ),
    )
    .animate()
    .shake(hz: 4)
    .fadeIn(duration: 300.ms);
  }
}
```

### 9.5 骨架屏动画

```dart
class SkeletonLoading extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 头像占位
        Container(
          width: 80,
          height: 80,
          decoration: BoxDecoration(
            color: Colors.grey[300],
            shape: BoxShape.circle,
          ),
        ).animate(onPlay: (c) => c.repeat()).shimmer(),

        SizedBox(height: 16),

        // 标题占位
        Container(
          width: 200,
          height: 20,
          color: Colors.grey[300],
        ).animate(onPlay: (c) => c.repeat()).shimmer(),

        SizedBox(height: 8),

        // 内容占位
        Container(
          width: double.infinity,
          height: 100,
          color: Colors.grey[300],
        ).animate(onPlay: (c) => c.repeat()).shimmer(),
      ],
    );
  }
}
```

### 9.6 全局动画效果复用

```dart
// 定义全局效果
class AppAnimations {
  static List<Effect> get fadeIn => [
    FadeEffect(duration: 300.ms, curve: Curves.easeOut),
  ];

  static List<Effect> get slideUp => [
    SlideEffect(
      begin: Offset(0, 0.3),
      end: Offset.zero,
      duration: 400.ms,
      curve: Curves.easeOutCubic,
    ),
  ];

  static List<Effect> get scaleIn => [
    ScaleEffect(
      begin: 0.8,
      end: 1.0,
      duration: 300.ms,
      curve: Curves.easeOutBack,
    ),
  ];

  static List<Effect> get entrance => [
    ...fadeIn,
    ...slideUp,
    ...scaleIn,
  ];
}

// 使用
Text("Hello").animate(effects: AppAnimations.entrance)
```

## 第10章 性能优化与最佳实践

### 10.1 性能优化建议

#### 10.1.1 避免过度动画

```dart
// 不推荐：过多的动画效果
Text("Too much").animate()
  .fade()
  .scale()
  .rotate()
  .blur()
  .shake()
  .slide()

// 推荐：简洁有效的动画
Text("Just right").animate()
  .fadeIn()
  .slideY()
```

#### 10.1.2 使用 const 构造

```dart
// 推荐
const Text("Static").animate().fade()

// 缓存效果列表
class Styles {
  static const entranceEffects = [
    FadeEffect(duration: 300.ms),
    ScaleEffect(begin: 0.9, end: 1.0),
  ];
}
```

#### 10.1.3 合理使用延迟

```dart
// 避免同时启动过多动画
Column(
  children: [
    Item1().animate(delay: 0.ms).fade(),
    Item2().animate(delay: 50.ms).fade(),
    Item3().animate(delay: 100.ms).fade(),
  ],
)
```

### 10.2 最佳实践

#### 10.2.1 动画时长建议

| 动画类型 | 建议时长  | 说明               |
| -------- | --------- | ------------------ |
| 微交互   | 100-200ms | 按钮点击、状态变化 |
| 入场动画 | 300-500ms | 页面元素进入       |
| 过渡动画 | 200-400ms | 页面切换、模态框   |
| 强调动画 | 400-800ms | 重要提示、引导     |

#### 10.2.2 动画曲线建议

| 场景 | 推荐曲线            | 说明               |
| ---- | ------------------- | ------------------ |
| 入场 | `Curves.easeOut`    | 快速开始，缓慢结束 |
| 出场 | `Curves.easeIn`     | 缓慢开始，快速结束 |
| 弹性 | `Curves.elasticOut` | 带有弹性效果       |
| 回弹 | `Curves.bounceOut`  | 带有回弹效果       |

## 附录 A：效果速查表

| 效果     | 方法           | 主要参数                       |
| -------- | -------------- | ------------------------------ |
| 淡入     | `fadeIn()`     | `begin`, `end`                 |
| 淡出     | `fadeOut()`    | `begin`, `end`                 |
| 缩放     | `scale()`      | `begin`, `end`, `alignment`    |
| 滑动     | `slide()`      | `begin`, `end`                 |
| 水平滑动 | `slideX()`     | `begin`, `end`                 |
| 垂直滑动 | `slideY()`     | `begin`, `end`                 |
| 旋转     | `rotate()`     | `begin`, `end`                 |
| 翻转     | `flip()`       | `begin`, `end`, `direction`    |
| 模糊     | `blur()`       | `begin`, `end`, `style`        |
| 抖动     | `shake()`      | `offset`, `rotation`, `hz`     |
| 闪光     | `shimmer()`    | `color`, `size`, `angle`       |
| 阴影     | `boxShadow()`  | `begin`, `end`, `borderRadius` |
| 色调     | `tint()`       | `color`, `blendMode`           |
| 饱和     | `saturate()`   | `begin`, `end`                 |
| 去饱和   | `desaturate()` | `begin`, `end`                 |
| 对齐     | `align()`      | `begin`, `end`                 |
| 隐藏     | `hide()`       | `maintain`                     |
| 显示     | `show()`       | `maintain`                     |
| 序列     | `then()`       | `delay`                        |
| 交换     | `swap()`       | `builder`                      |
| 交叉淡入 | `crossfade()`  | `builder`                      |
| 路径跟随 | `followPath()` | `path`, `rotate`               |
| 自定义   | `custom()`     | `builder`                      |
| 回调     | `callback()`   | `callback`                     |
| 监听     | `listen()`     | `callback`                     |

## 附录 B：常见问题与解决方案

### Q1：动画不生效

**可能原因**：

- Widget 没有设置尺寸
- 动画被其他效果覆盖

**解决方案**：

```dart
// 确保 Widget 有尺寸
Container(
  width: 100,
  height: 100,
  child: Text("Animate me"),
).animate().fade()
```

### Q2：列表动画卡顿

**解决方案**：

```dart
// 使用合适的间隔
ListView.builder(
  itemBuilder: (context, index) {
    return Item()
      .animate(delay: (index * 30).ms)  // 减少间隔
      .fade();
  },
)
```

### Q3：动画重复播放

**解决方案**：

```dart
// 使用 target 控制
Text("Toggle")
  .animate(target: isVisible ? 1 : 0)
  .fade()
```

## 附录 C：参考资源

- [flutter_animate 官方文档](https://pub.dev/packages/flutter_animate)
- [flutter_animate API 文档](https://pub.dev/documentation/flutter_animate/latest/)
- [Flutter 动画官方指南](https://docs.flutter.dev/ui/animations)
- [Material Design 动画](https://m3.material.io/styles/motion)
- [gskinner 博客](https://blog.gskinner.com/archives/2022/09/introducing-flutter-animate.html)
