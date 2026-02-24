# Flutter animated_text_kit 4.3.0 完整详解与实战指南

## 前言

在 Flutter 应用开发中，文本动画是吸引用户注意力、提升用户体验的重要手段。一个精心设计的文本动画可以让应用更加生动有趣，给用户留下深刻印象。

`animated_text_kit` 是 Flutter 社区最受欢迎的文本动画包之一，它提供了丰富多样的文本动画效果，包括旋转、淡入淡出、打字机效果、缩放、彩虹色变换、液体填充、波浪效果等。这个包被 Flutter 官方评为 "Flutter Favorite"，是 Flutter 生态中的明星包。

本书基于 `animated_text_kit` 4.3.0 版本，详细介绍该包的所有动画类型、API 用法，以及如何将其集成到 Flutter 应用中。

---

## 第1章 animated_text_kit 基础

### 1.1 什么是 animated_text_kit

`animated_text_kit` 是一个 Flutter 文本动画包，它提供了一系列酷炫的文本动画效果：

| 动画类型 | 说明 | 适用场景 |
|----------|------|----------|
| **Rotate** | 旋转动画 | 标题切换、标语展示 |
| **Fade** | 淡入淡出 | 平滑过渡、提示信息 |
| **Typer** | 打字效果 | 代码展示、实时输入 |
| **Typewriter** | 打字机效果（带光标） | 复古风格、故事叙述 |
| **Scale** | 缩放动画 | 强调重点、动态标题 |
| **Colorize** | 彩虹色变换 | 炫酷效果、品牌展示 |
| **TextLiquidFill** | 液体填充 | 创意展示、加载动画 |
| **Wavy** | 波浪效果 | 活泼氛围、儿童应用 |
| **Flicker** | 闪烁效果 | 霓虹灯风格、夜间模式 |
| **Scramble** | 字符打乱 | 科技感、解密效果 |
| **Bounce** | 弹跳效果 | 活泼有趣、游戏应用 |

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  animated_text_kit: ^4.3.0
```

#### 1.2.2 导入包

```dart
import 'package:animated_text_kit/animated_text_kit.dart';
```

### 1.3 第一个动画文本

```dart
import 'package:flutter/material.dart';
import 'package:animated_text_kit/animated_text_kit.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Animated Text Kit Demo',
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
      appBar: AppBar(title: const Text('Animated Text Kit')),
      body: Center(
        child: AnimatedTextKit(
          animatedTexts: [
            TypewriterAnimatedText(
              'Hello World!',
              textStyle: const TextStyle(
                fontSize: 32.0,
                fontWeight: FontWeight.bold,
              ),
              speed: const Duration(milliseconds: 100),
            ),
            TypewriterAnimatedText(
              'Welcome to Flutter',
              textStyle: const TextStyle(
                fontSize: 32.0,
                fontWeight: FontWeight.bold,
              ),
              speed: const Duration(milliseconds: 100),
            ),
          ],
          totalRepeatCount: 4,
          pause: const Duration(milliseconds: 1000),
          displayFullTextOnTap: true,
          stopPauseOnTap: true,
        ),
      ),
    );
  }
}
```

> **Flutter框架小知识**
> 
> `AnimatedTextKit` 是一个 StatefulWidget，它负责管理动画的生命周期。你可以通过 `animatedTexts` 参数传入多个动画文本，它们会按顺序播放。使用 `DefaultTextStyle` 可以为所有动画文本设置统一的样式。

---

## 第2章 AnimatedTextKit 核心类详解

### 2.1 AnimatedTextKit

`AnimatedTextKit` 是所有文本动画的容器组件，负责管理和协调动画的播放。

#### 2.1.1 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| animatedTexts | List<AnimatedText> | 必填 | 动画文本列表 |
| pause | Duration | Duration(milliseconds: 1000) | 动画之间的暂停时间 |
| displayFullTextOnTap | bool | false | 点击是否立即显示完整文本 |
| stopPauseOnTap | bool | false | 点击是否停止暂停 |
| onTap | VoidCallback? | null | 点击回调 |
| onNext | void Function(int, bool)? | null | 下一个动画前的回调 |
| onNextBeforePause | void Function(int, bool)? | null | 暂停前的回调 |
| onFinished | VoidCallback? | null | 动画完成回调 |
| isRepeatingAnimation | bool | true | 是否重复播放 |
| repeatForever | bool | false | 是否永远重复 |
| totalRepeatCount | int | 3 | 重复次数 |
| controller | AnimatedTextController? | null | 动画控制器 |

#### 2.1.2 完整示例

```dart
AnimatedTextKit(
  animatedTexts: [
    FadeAnimatedText('Hello!'),
    ScaleAnimatedText('World!'),
  ],
  pause: const Duration(milliseconds: 500),
  displayFullTextOnTap: true,
  stopPauseOnTap: true,
  onTap: () {
    print('Text tapped!');
  },
  onNext: (index, isLast) {
    print('Next animation: $index, isLast: $isLast');
  },
  onFinished: () {
    print('Animation finished!');
  },
  isRepeatingAnimation: true,
  repeatForever: false,
  totalRepeatCount: 5,
)
```

### 2.2 AnimatedTextController

`AnimatedTextController` 用于显式控制动画的播放、暂停和重置。

#### 2.2.1 方法

| 方法名 | 返回值 | 说明 |
|--------|--------|------|
| play | void | 播放动画 |
| pause | void | 暂停动画 |
| reset | void | 重置动画 |

#### 2.2.2 使用示例

```dart
class ControlledAnimation extends StatefulWidget {
  const ControlledAnimation({super.key});

  @override
  State<ControlledAnimation> createState() => _ControlledAnimationState();
}

class _ControlledAnimationState extends State<ControlledAnimation> {
  final AnimatedTextController _controller = AnimatedTextController();

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        AnimatedTextKit(
          animatedTexts: [
            TypewriterAnimatedText('Controlled Animation'),
          ],
          controller: _controller,
          isRepeatingAnimation: false,
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => _controller.play(),
              child: const Text('Play'),
            ),
            ElevatedButton(
              onPressed: () => _controller.pause(),
              child: const Text('Pause'),
            ),
            ElevatedButton(
              onPressed: () => _controller.reset(),
              child: const Text('Reset'),
            ),
          ],
        ),
      ],
    );
  }
}
```

### 2.3 AnimatedText 基类

所有动画文本类都继承自 `AnimatedText`，它定义了动画文本的基本结构。

#### 2.3.1 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | null | 文本对齐方式 |
| textDirection | TextDirection? | null | 文本方向 |

---

## 第3章 旋转动画（RotateAnimatedText）

`RotateAnimatedText` 实现文本旋转动画，文本像翻页一样旋转切换。

### 3.1 核心特性

| 特性 | 说明 |
|------|------|
| 旋转切换 | 文本像翻页一样旋转进入和退出 |
| 可禁用旋转退出 | 可以只旋转进入，不旋转退出 |
| 自定义过渡高度 | 可以调整旋转动画的高度 |

### 3.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| duration | Duration | Duration(milliseconds: 2000) | 动画持续时间 |
| rotateOut | bool | true | 是否旋转退出 |
| transitionHeight | double? | null | 过渡高度 |

### 3.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    RotateAnimatedText('AWESOME'),
    RotateAnimatedText('OPTIMISTIC'),
    RotateAnimatedText('DIFFERENT'),
  ],
  onTap: () {
    print('Tap Event');
  },
)
```

### 3.4 完整示例

```dart
class RotateDemo extends StatelessWidget {
  const RotateDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Rotate Animation')),
      body: Center(
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'Be ',
              style: TextStyle(fontSize: 43.0),
            ),
            DefaultTextStyle(
              style: const TextStyle(
                fontSize: 40.0,
                fontFamily: 'Horizon',
                fontWeight: FontWeight.bold,
              ),
              child: AnimatedTextKit(
                animatedTexts: [
                  RotateAnimatedText(
                    'AWESOME',
                    textStyle: const TextStyle(color: Colors.blue),
                  ),
                  RotateAnimatedText(
                    'OPTIMISTIC',
                    textStyle: const TextStyle(color: Colors.green),
                  ),
                  RotateAnimatedText(
                    'DIFFERENT',
                    textStyle: const TextStyle(color: Colors.orange),
                  ),
                ],
                pause: const Duration(milliseconds: 1000),
                displayFullTextOnTap: true,
                stopPauseOnTap: true,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 3.5 禁用旋转退出

```dart
AnimatedTextKit(
  animatedTexts: [
    RotateAnimatedText(
      'Only Rotate In',
      rotateOut: false,  // 只旋转进入，不旋转退出
    ),
  ],
)
```

---

## 第4章 淡入淡出动画（FadeAnimatedText）

`FadeAnimatedText` 实现文本淡入淡出动画。

### 4.1 核心特性

| 特性 | 说明 |
|------|------|
| 淡入淡出 | 文本平滑地淡入和淡出 |
| 可自定义淡入淡出间隔 | 可以调整淡入和淡出的时间比例 |

### 4.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| duration | Duration | Duration(milliseconds: 2000) | 动画持续时间 |
| fadeInEnd | double | 0.5 | 淡入结束位置（0-1） |
| fadeOutBegin | double | 0.8 | 淡出开始位置（0-1） |

### 4.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    FadeAnimatedText('do IT!'),
    FadeAnimatedText('do it RIGHT!!'),
    FadeAnimatedText('do it RIGHT NOW!!!'),
  ],
  onTap: () {
    print('Tap Event');
  },
)
```

### 4.4 完整示例

```dart
class FadeDemo extends StatelessWidget {
  const FadeDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Fade Animation')),
      body: Center(
        child: SizedBox(
          width: 250.0,
          child: DefaultTextStyle(
            style: const TextStyle(
              fontSize: 32.0,
              fontWeight: FontWeight.bold,
            ),
            child: AnimatedTextKit(
              animatedTexts: [
                FadeAnimatedText(
                  'First',
                  textStyle: const TextStyle(color: Colors.blue),
                  fadeInEnd: 0.3,  // 30% 时间用于淡入
                  fadeOutBegin: 0.7,  // 70% 时间开始淡出
                ),
                FadeAnimatedText(
                  'Second',
                  textStyle: const TextStyle(color: Colors.green),
                ),
                FadeAnimatedText(
                  'Third',
                  textStyle: const TextStyle(color: Colors.orange),
                ),
              ],
              pause: const Duration(milliseconds: 500),
              displayFullTextOnTap: true,
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 第5章 打字效果（TyperAnimatedText）

`TyperAnimatedText` 实现打字效果，文本逐个字符显示。

### 5.1 核心特性

| 特性 | 说明 |
|------|------|
| 打字效果 | 文本像打字一样逐个字符显示 |
| 可调节速度 | 可以控制打字速度 |

### 5.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| speed | Duration | Duration(milliseconds: 40) | 打字速度 |

### 5.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    TyperAnimatedText('This is a typing animation'),
    TyperAnimatedText('It looks like someone is typing'),
  ],
)
```

### 5.4 完整示例

```dart
class TyperDemo extends StatelessWidget {
  const TyperDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Typer Animation')),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(16.0),
          decoration: BoxDecoration(
            color: Colors.grey[200],
            borderRadius: BorderRadius.circular(8.0),
          ),
          child: AnimatedTextKit(
            animatedTexts: [
              TyperAnimatedText(
                'Initializing system...',
                textStyle: const TextStyle(
                  fontSize: 20.0,
                  fontFamily: 'Courier',
                  color: Colors.green,
                ),
                speed: const Duration(milliseconds: 50),
              ),
              TyperAnimatedText(
                'Loading modules...',
                textStyle: const TextStyle(
                  fontSize: 20.0,
                  fontFamily: 'Courier',
                  color: Colors.green,
                ),
                speed: const Duration(milliseconds: 50),
              ),
              TyperAnimatedText(
                'System ready!',
                textStyle: const TextStyle(
                  fontSize: 20.0,
                  fontFamily: 'Courier',
                  color: Colors.green,
                  fontWeight: FontWeight.bold,
                ),
                speed: const Duration(milliseconds: 50),
              ),
            ],
            pause: const Duration(milliseconds: 1000),
            displayFullTextOnTap: true,
            repeatForever: true,
          ),
        ),
      ),
    );
  }
}
```

---

## 第6章 打字机效果（TypewriterAnimatedText）

`TypewriterAnimatedText` 实现打字机效果，带有闪烁的光标。

### 6.1 核心特性

| 特性 | 说明 |
|------|------|
| 打字机效果 | 文本逐个字符显示，带有闪烁光标 |
| 可自定义光标 | 可以调整光标样式 |

### 6.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| speed | Duration | Duration(milliseconds: 30) | 打字速度 |
| cursor | String | '_' | 光标字符 |

### 6.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    TypewriterAnimatedText('Hello World!'),
    TypewriterAnimatedText('Welcome to Flutter'),
  ],
)
```

### 6.4 完整示例

```dart
class TypewriterDemo extends StatelessWidget {
  const TypewriterDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Typewriter Animation')),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              AnimatedTextKit(
                animatedTexts: [
                  TypewriterAnimatedText(
                    'Once upon a time...',
                    textStyle: const TextStyle(
                      fontSize: 24.0,
                      fontFamily: 'Georgia',
                      fontStyle: FontStyle.italic,
                    ),
                    speed: const Duration(milliseconds: 100),
                    cursor: '|',
                  ),
                  TypewriterAnimatedText(
                    'In a land far, far away...',
                    textStyle: const TextStyle(
                      fontSize: 24.0,
                      fontFamily: 'Georgia',
                      fontStyle: FontStyle.italic,
                    ),
                    speed: const Duration(milliseconds: 100),
                    cursor: '|',
                  ),
                ],
                pause: const Duration(milliseconds: 2000),
                displayFullTextOnTap: true,
                stopPauseOnTap: true,
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 第7章 缩放动画（ScaleAnimatedText）

`ScaleAnimatedText` 实现文本缩放动画，文本从小到大缩放显示。

### 7.1 核心特性

| 特性 | 说明 |
|------|------|
| 缩放效果 | 文本从小到大缩放显示 |
| 可调节持续时间 | 可以控制缩放速度 |

### 7.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| duration | Duration | Duration(milliseconds: 1500) | 动画持续时间 |
| scalingFactor | double | 1.5 | 缩放因子 |

### 7.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    ScaleAnimatedText('Scale Up!'),
    ScaleAnimatedText('Get Bigger!'),
  ],
)
```

### 7.4 完整示例

```dart
class ScaleDemo extends StatelessWidget {
  const ScaleDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Scale Animation')),
      body: Center(
        child: AnimatedTextKit(
          animatedTexts: [
            ScaleAnimatedText(
              'BIG',
              textStyle: const TextStyle(
                fontSize: 50.0,
                fontWeight: FontWeight.bold,
                color: Colors.red,
              ),
              scalingFactor: 2.0,
            ),
            ScaleAnimatedText(
              'BIGGER',
              textStyle: const TextStyle(
                fontSize: 50.0,
                fontWeight: FontWeight.bold,
                color: Colors.orange,
              ),
              scalingFactor: 2.5,
            ),
            ScaleAnimatedText(
              'BIGGEST',
              textStyle: const TextStyle(
                fontSize: 50.0,
                fontWeight: FontWeight.bold,
                color: Colors.purple,
              ),
              scalingFactor: 3.0,
            ),
          ],
          pause: const Duration(milliseconds: 500),
          repeatForever: true,
        ),
      ),
    );
  }
}
```

---

## 第8章 彩虹色变换（ColorizeAnimatedText）

`ColorizeAnimatedText` 实现文本彩虹色变换效果，文本颜色在多种颜色之间渐变切换。

### 8.1 核心特性

| 特性 | 说明 |
|------|------|
| 颜色渐变 | 文本颜色在多种颜色之间渐变 |
| 可自定义颜色列表 | 可以设置任意数量的颜色 |

### 8.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle | 必填 | 文本样式（必须包含字体大小） |
| colors | List<Color> | 必填 | 颜色列表（至少两个颜色） |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| speed | Duration | Duration(milliseconds: 200) | 颜色切换速度 |

### 8.3 基础用法

```dart
const colorizeColors = [
  Colors.purple,
  Colors.blue,
  Colors.yellow,
  Colors.red,
];

const colorizeTextStyle = TextStyle(
  fontSize: 50.0,
  fontFamily: 'Horizon',
);

AnimatedTextKit(
  animatedTexts: [
    ColorizeAnimatedText(
      'Larry Page',
      textStyle: colorizeTextStyle,
      colors: colorizeColors,
    ),
    ColorizeAnimatedText(
      'Bill Gates',
      textStyle: colorizeTextStyle,
      colors: colorizeColors,
    ),
  ],
)
```

### 8.4 完整示例

```dart
class ColorizeDemo extends StatelessWidget {
  const ColorizeDemo({super.key});

  static const List<Color> colorizeColors = [
    Colors.purple,
    Colors.blue,
    Colors.yellow,
    Colors.red,
    Colors.orange,
    Colors.green,
  ];

  static const TextStyle colorizeTextStyle = TextStyle(
    fontSize: 50.0,
    fontFamily: 'Horizon',
    fontWeight: FontWeight.bold,
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Colorize Animation')),
      body: Center(
        child: AnimatedTextKit(
          animatedTexts: [
            ColorizeAnimatedText(
              'FLUTTER',
              textStyle: colorizeTextStyle,
              colors: colorizeColors,
              speed: const Duration(milliseconds: 200),
            ),
            ColorizeAnimatedText(
              'ANIMATIONS',
              textStyle: colorizeTextStyle,
              colors: colorizeColors,
              speed: const Duration(milliseconds: 200),
            ),
            ColorizeAnimatedText(
              'AWESOME',
              textStyle: colorizeTextStyle,
              colors: colorizeColors,
              speed: const Duration(milliseconds: 200),
            ),
          ],
          pause: const Duration(milliseconds: 500),
          repeatForever: true,
        ),
      ),
    );
  }
}
```

---

## 第9章 液体填充（TextLiquidFill）

`TextLiquidFill` 实现液体填充效果，文本像被液体填充一样显示。

### 9.1 核心特性

| 特性 | 说明 |
|------|------|
| 液体填充 | 文本像被液体填充一样显示 |
| 波浪效果 | 填充时有波浪动画 |
| 可自定义颜色 | 可以设置波浪颜色和背景颜色 |

### 9.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| waveColor | Color | Colors.blueAccent | 波浪颜色 |
| boxBackgroundColor | Color | Colors.redAccent | 盒子背景颜色 |
| textStyle | TextStyle? | null | 文本样式 |
| boxHeight | double | 250.0 | 盒子高度 |
| boxWidth | double | 400.0 | 盒子宽度 |
| loadUntil | double | 1.0 | 填充到多少（0-1） |
| waveDuration | Duration | Duration(seconds: 2) | 波浪动画持续时间 |
| loadDuration | Duration | Duration(seconds: 6) | 填充持续时间 |

### 9.3 基础用法

```dart
TextLiquidFill(
  text: 'LIQUIDY',
  waveColor: Colors.blueAccent,
  boxBackgroundColor: Colors.redAccent,
  textStyle: TextStyle(
    fontSize: 80.0,
    fontWeight: FontWeight.bold,
  ),
  boxHeight: 300.0,
)
```

### 9.4 完整示例

```dart
class TextLiquidFillDemo extends StatelessWidget {
  const TextLiquidFillDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Text Liquid Fill')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextLiquidFill(
              text: 'LOADING',
              waveColor: Colors.blue,
              boxBackgroundColor: Colors.white,
              textStyle: const TextStyle(
                fontSize: 80.0,
                fontWeight: FontWeight.bold,
                color: Colors.black,
              ),
              boxHeight: 150.0,
              boxWidth: 350.0,
              loadUntil: 1.0,
              waveDuration: const Duration(seconds: 1),
              loadDuration: const Duration(seconds: 4),
            ),
            const SizedBox(height: 50),
            TextLiquidFill(
              text: '50%',
              waveColor: Colors.green,
              boxBackgroundColor: Colors.grey[200]!,
              textStyle: const TextStyle(
                fontSize: 60.0,
                fontWeight: FontWeight.bold,
              ),
              boxHeight: 100.0,
              boxWidth: 200.0,
              loadUntil: 0.5,
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 第10章 波浪效果（WavyAnimatedText）

`WavyAnimatedText` 实现波浪效果，文本像波浪一样起伏。

### 10.1 核心特性

| 特性 | 说明 |
|------|------|
| 波浪起伏 | 文本像波浪一样起伏波动 |
| 活泼有趣 | 适合儿童应用或活泼的场景 |

### 10.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| speed | Duration | Duration(milliseconds: 50) | 波浪速度 |

### 10.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    WavyAnimatedText('Hello World'),
    WavyAnimatedText('Look at the waves'),
  ],
)
```

### 10.4 完整示例

```dart
class WavyDemo extends StatelessWidget {
  const WavyDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Wavy Animation')),
      body: Center(
        child: Container(
          padding: const EdgeInsets.all(24.0),
          decoration: BoxDecoration(
            gradient: LinearGradient(
              colors: [Colors.blue[100]!, Colors.blue[300]!],
            ),
            borderRadius: BorderRadius.circular(16.0),
          ),
          child: AnimatedTextKit(
            animatedTexts: [
              WavyAnimatedText(
                'Ocean Waves',
                textStyle: const TextStyle(
                  fontSize: 40.0,
                  fontWeight: FontWeight.bold,
                  color: Colors.blue,
                ),
                speed: const Duration(milliseconds: 70),
              ),
              WavyAnimatedText(
                'Flowing Water',
                textStyle: const TextStyle(
                  fontSize: 40.0,
                  fontWeight: FontWeight.bold,
                  color: Colors.cyan,
                ),
                speed: const Duration(milliseconds: 70),
              ),
            ],
            pause: const Duration(milliseconds: 1000),
            repeatForever: true,
          ),
        ),
      ),
    );
  }
}
```

---

## 第11章 闪烁效果（FlickerAnimatedText）

`FlickerAnimatedText` 实现闪烁效果，文本像霓虹灯一样闪烁。

### 11.1 核心特性

| 特性 | 说明 |
|------|------|
| 霓虹灯闪烁 | 文本像霓虹灯一样闪烁 |
| 适合夜间模式 | 适合夜间场景或霓虹灯风格 |

### 11.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| entryEnd | double | 0.5 | 进入动画结束位置 |

### 11.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    FlickerAnimatedText('Flicker Frenzy'),
    FlickerAnimatedText('Night Vibes On'),
  ],
)
```

### 11.4 完整示例

```dart
class FlickerDemo extends StatelessWidget {
  const FlickerDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        title: const Text('Flicker Animation'),
        backgroundColor: Colors.black,
      ),
      body: Center(
        child: AnimatedTextKit(
          repeatForever: true,
          animatedTexts: [
            FlickerAnimatedText(
              'NEON',
              textStyle: const TextStyle(
                fontSize: 50,
                color: Colors.pink,
                shadows: [
                  Shadow(
                    blurRadius: 20.0,
                    color: Colors.pink,
                    offset: Offset(0, 0),
                  ),
                ],
              ),
            ),
            FlickerAnimatedText(
              'LIGHTS',
              textStyle: const TextStyle(
                fontSize: 50,
                color: Colors.cyan,
                shadows: [
                  Shadow(
                    blurRadius: 20.0,
                    color: Colors.cyan,
                    offset: Offset(0, 0),
                  ),
                ],
              ),
            ),
            FlickerAnimatedText(
              'NIGHT',
              textStyle: const TextStyle(
                fontSize: 50,
                color: Colors.yellow,
                shadows: [
                  Shadow(
                    blurRadius: 20.0,
                    color: Colors.yellow,
                    offset: Offset(0, 0),
                  ),
                ],
              ),
            ),
          ],
          onTap: () {
            print('Tap Event');
          },
        ),
      ),
    );
  }
}
```

---

## 第12章 字符打乱（ScrambleAnimatedText）

`ScrambleAnimatedText` 实现字符打乱效果，文本字符随机变化后最终显示正确文本。

### 12.1 核心特性

| 特性 | 说明 |
|------|------|
| 字符打乱 | 文本字符随机变化 |
| 科技感 | 适合科技风格、解密效果 |

### 12.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| speed | Duration | Duration(milliseconds: 40) | 打乱速度 |
| chars | String | '!<>-_\\/[]{}—=+*^?#________' | 用于打乱的字符集 |

### 12.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    ScrambleAnimatedText('Mobile Dev.'),
    ScrambleAnimatedText('Explorer'),
  ],
)
```

### 12.4 完整示例

```dart
class ScrambleDemo extends StatelessWidget {
  const ScrambleDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        title: const Text('Scramble Animation'),
        backgroundColor: Colors.black,
      ),
      body: Center(
        child: AnimatedTextKit(
          animatedTexts: [
            ScrambleAnimatedText(
              'HACKING...',
              textStyle: const TextStyle(
                fontSize: 40.0,
                fontWeight: FontWeight.bold,
                color: Colors.green,
                fontFamily: 'Courier',
              ),
              speed: const Duration(milliseconds: 50),
            ),
            ScrambleAnimatedText(
              'DECRYPTING...',
              textStyle: const TextStyle(
                fontSize: 40.0,
                fontWeight: FontWeight.bold,
                color: Colors.green,
                fontFamily: 'Courier',
              ),
              speed: const Duration(milliseconds: 50),
            ),
            ScrambleAnimatedText(
              'ACCESS GRANTED',
              textStyle: const TextStyle(
                fontSize: 40.0,
                fontWeight: FontWeight.bold,
                color: Colors.green,
                fontFamily: 'Courier',
              ),
              speed: const Duration(milliseconds: 50),
            ),
          ],
          pause: const Duration(milliseconds: 1000),
          repeatForever: true,
        ),
      ),
    );
  }
}
```

---

## 第13章 弹跳效果（BounceAnimatedText）

`BounceAnimatedText` 实现弹跳效果，文本像弹簧一样弹入。

### 13.1 核心特性

| 特性 | 说明 |
|------|------|
| 弹跳效果 | 文本从下方弹入 |
| 弹性曲线 | 使用弹性曲线，效果活泼 |

### 13.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| text | String | 必填 | 动画文本内容 |
| textStyle | TextStyle? | null | 文本样式 |
| textAlign | TextAlign? | TextAlign.start | 文本对齐方式 |
| duration | Duration | Duration(milliseconds: 800) | 动画持续时间 |
| scalingFactor | double | 1.0 | 缩放因子 |

### 13.3 基础用法

```dart
AnimatedTextKit(
  animatedTexts: [
    BounceAnimatedText('Bounce!'),
    BounceAnimatedText('Spring!'),
    BounceAnimatedText('Jump!'),
  ],
)
```

### 13.4 完整示例

```dart
class BounceDemo extends StatelessWidget {
  const BounceDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Bounce Animation')),
      body: Center(
        child: AnimatedTextKit(
          animatedTexts: [
            BounceAnimatedText(
              'FUN',
              textStyle: const TextStyle(
                fontSize: 60.0,
                fontWeight: FontWeight.bold,
                color: Colors.orange,
              ),
            ),
            BounceAnimatedText(
              'PLAYFUL',
              textStyle: const TextStyle(
                fontSize: 60.0,
                fontWeight: FontWeight.bold,
                color: Colors.pink,
              ),
            ),
            BounceAnimatedText(
              'ENERGETIC',
              textStyle: const TextStyle(
                fontSize: 60.0,
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
          ],
          pause: const Duration(milliseconds: 500),
          repeatForever: true,
        ),
      ),
    );
  }
}
```

---

## 第14章 组合动画

### 14.1 混合使用多种动画

```dart
AnimatedTextKit(
  animatedTexts: [
    FadeAnimatedText(
      'Fade First',
      textStyle: TextStyle(fontSize: 32.0, fontWeight: FontWeight.bold),
    ),
    ScaleAnimatedText(
      'Then Scale',
      textStyle: TextStyle(fontSize: 70.0, fontFamily: 'Canterbury'),
    ),
    RotateAnimatedText(
      'Then Rotate',
      textStyle: TextStyle(fontSize: 32.0, fontWeight: FontWeight.bold),
    ),
  ],
)
```

### 14.2 完整示例：欢迎页面

```dart
class WelcomePage extends StatelessWidget {
  const WelcomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: [Colors.purple, Colors.blue],
          ),
        ),
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Logo 动画
              ScaleAnimatedTextKit(
                animatedTexts: [
                  ScaleAnimatedText(
                    'APP',
                    textStyle: const TextStyle(
                      fontSize: 80.0,
                      fontWeight: FontWeight.bold,
                      color: Colors.white,
                    ),
                    scalingFactor: 1.5,
                  ),
                ],
                totalRepeatCount: 1,
              ),
              const SizedBox(height: 30),
              // 标语动画
              AnimatedTextKit(
                animatedTexts: [
                  FadeAnimatedText(
                    'Welcome to',
                    textStyle: const TextStyle(
                      fontSize: 24.0,
                      color: Colors.white70,
                    ),
                  ),
                  FadeAnimatedText(
                    'the future of',
                    textStyle: const TextStyle(
                      fontSize: 24.0,
                      color: Colors.white70,
                    ),
                  ),
                  FadeAnimatedText(
                    'mobile apps',
                    textStyle: const TextStyle(
                      fontSize: 24.0,
                      color: Colors.white70,
                    ),
                  ),
                ],
                pause: const Duration(milliseconds: 500),
                repeatForever: true,
              ),
              const SizedBox(height: 50),
              // 开始按钮
              ElevatedButton(
                onPressed: () {
                  Navigator.pushReplacement(
                    context,
                    MaterialPageRoute(builder: (context) => const HomePage()),
                  );
                },
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.white,
                  foregroundColor: Colors.purple,
                  padding: const EdgeInsets.symmetric(
                    horizontal: 50,
                    vertical: 15,
                  ),
                ),
                child: const Text(
                  'Get Started',
                  style: TextStyle(fontSize: 18),
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

---

## 第15章 自定义动画

### 15.1 创建自定义动画类

你可以通过继承 `AnimatedText` 类来创建自定义动画。

```dart
class CustomAnimatedText extends AnimatedText {
  final double rotationAngle;

  CustomAnimatedText(
    String text, {
    TextStyle? textStyle,
    TextAlign textAlign = TextAlign.start,
    TextDirection? textDirection,
    Duration duration = const Duration(milliseconds: 2000),
    this.rotationAngle = 360.0,
  }) : super(
          text: text,
          textStyle: textStyle,
          textAlign: textAlign,
          textDirection: textDirection,
          duration: duration,
        );

  late Animation<double> _rotationAnimation;

  @override
  void initAnimation(AnimationController controller) {
    _rotationAnimation = Tween<double>(
      begin: 0.0,
      end: rotationAngle,
    ).animate(
      CurvedAnimation(
        parent: controller,
        curve: Curves.easeInOut,
      ),
    );
  }

  @override
  Widget animatedBuilder(BuildContext context, Widget child) {
    return Transform.rotate(
      angle: _rotationAnimation.value * 3.14159 / 180,
      child: Opacity(
        opacity: _rotationAnimation.value / rotationAngle,
        child: child,
      ),
    );
  }

  @override
  Widget completeText(BuildContext context) {
    return DefaultTextStyle(
      style: textStyle ?? const TextStyle(),
      child: Text(
        text,
        textAlign: textAlign,
        textDirection: textDirection,
      ),
    );
  }
}
```

### 15.2 使用自定义动画

```dart
AnimatedTextKit(
  animatedTexts: [
    CustomAnimatedText(
      'Custom Animation',
      textStyle: const TextStyle(
        fontSize: 32.0,
        fontWeight: FontWeight.bold,
      ),
      rotationAngle: 720.0,
    ),
  ],
)
```

---

## 第16章 实战案例

### 16.1 启动页动画

```dart
class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    Future.delayed(const Duration(seconds: 5), () {
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => const HomePage()),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple,
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ScaleAnimatedTextKit(
              animatedTexts: [
                ScaleAnimatedText(
                  'MY APP',
                  textStyle: const TextStyle(
                    fontSize: 60.0,
                    fontWeight: FontWeight.bold,
                    color: Colors.white,
                  ),
                  scalingFactor: 1.5,
                ),
              ],
              totalRepeatCount: 1,
            ),
            const SizedBox(height: 20),
            TypewriterAnimatedTextKit(
              animatedTexts: [
                TypewriterAnimatedText(
                  'Loading...',
                  textStyle: const TextStyle(
                    fontSize: 20.0,
                    color: Colors.white70,
                  ),
                  speed: const Duration(milliseconds: 100),
                ),
              ],
              repeatForever: true,
            ),
          ],
        ),
      ),
    );
  }
}
```

### 16.2 聊天应用：正在输入提示

```dart
class TypingIndicator extends StatelessWidget {
  const TypingIndicator({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: Colors.grey[200],
        borderRadius: BorderRadius.circular(20),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          TyperAnimatedTextKit(
            animatedTexts: [
              TyperAnimatedText(
                '...',
                textStyle: const TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
                speed: const Duration(milliseconds: 300),
              ),
            ],
            repeatForever: true,
          ),
        ],
      ),
    );
  }
}
```

### 16.3 加载状态提示

```dart
class LoadingWidget extends StatelessWidget {
  final String message;

  const LoadingWidget({super.key, required this.message});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const CircularProgressIndicator(),
          const SizedBox(height: 20),
          AnimatedTextKit(
            animatedTexts: [
              FadeAnimatedText(
                message,
                textStyle: const TextStyle(
                  fontSize: 18,
                  color: Colors.grey,
                ),
              ),
            ],
            repeatForever: true,
          ),
        ],
      ),
    );
  }
}
```

---

## 第17章 最佳实践

### 17.1 动画选择指南

| 场景 | 推荐动画 |
|------|----------|
| 启动页 Logo | `ScaleAnimatedText` |
| 欢迎语 | `FadeAnimatedText`、`TypewriterAnimatedText` |
| 标语切换 | `RotateAnimatedText` |
| 加载提示 | `TyperAnimatedText`、`FadeAnimatedText` |
| 炫酷效果 | `ColorizeAnimatedText`、`TextLiquidFill` |
| 活泼氛围 | `WavyAnimatedText`、`BounceAnimatedText` |
| 科技感 | `ScrambleAnimatedText` |
| 夜间模式 | `FlickerAnimatedText` |

### 17.2 性能优化

```dart
// 使用 const 构造函数
const AnimatedTextKit(
  animatedTexts: [
    FadeAnimatedText('Optimized'),
  ],
)

// 避免过多的重复次数
AnimatedTextKit(
  animatedTexts: [...],
  totalRepeatCount: 3,  // 不要设置过大
)

// 合理设置暂停时间
AnimatedTextKit(
  animatedTexts: [...],
  pause: const Duration(milliseconds: 500),  // 不要太短
)
```

### 17.3 无障碍支持

```dart
AnimatedTextKit(
  animatedTexts: [
    FadeAnimatedText('Accessible Text'),
  ],
  child: Semantics(
    label: 'Animated text showing: Accessible Text',
    child: Container(),
  ),
)
```

### 17.4 DO 和 DON'T

#### ✅ DO

- 根据场景选择合适的动画类型
- 使用 `DefaultTextStyle` 统一样式
- 合理设置动画速度和暂停时间
- 使用 `displayFullTextOnTap` 提升用户体验
- 测试动画在不同设备上的性能

#### ❌ DON'T

- 不要过度使用动画
- 不要让动画速度过快或过慢
- 不要在重要信息上使用过于花哨的动画
- 不要忽略无障碍支持
- 不要让动画干扰用户操作

---

## 附录：animated_text_kit API 速查表

### 动画类型

| 动画类 | 说明 |
|--------|------|
| `RotateAnimatedText` | 旋转动画 |
| `FadeAnimatedText` | 淡入淡出 |
| `TyperAnimatedText` | 打字效果 |
| `TypewriterAnimatedText` | 打字机效果 |
| `ScaleAnimatedText` | 缩放动画 |
| `ColorizeAnimatedText` | 彩虹色变换 |
| `TextLiquidFill` | 液体填充 |
| `WavyAnimatedText` | 波浪效果 |
| `FlickerAnimatedText` | 闪烁效果 |
| `ScrambleAnimatedText` | 字符打乱 |
| `BounceAnimatedText` | 弹跳效果 |

### AnimatedTextKit 参数

| 参数 | 说明 |
|------|------|
| `animatedTexts` | 动画文本列表 |
| `pause` | 动画间隔 |
| `displayFullTextOnTap` | 点击显示完整文本 |
| `stopPauseOnTap` | 点击停止暂停 |
| `onTap` | 点击回调 |
| `onNext` | 下一个动画回调 |
| `onFinished` | 完成回调 |
| `isRepeatingAnimation` | 是否重复 |
| `repeatForever` | 永远重复 |
| `totalRepeatCount` | 重复次数 |
| `controller` | 动画控制器 |

---

## 结语

`animated_text_kit` 4.3.0 是 Flutter 生态中最强大的文本动画包之一。通过本书的学习，你应该已经掌握了：

1. 所有 11 种文本动画的使用方法
2. 如何组合使用多种动画
3. 如何创建自定义动画
4. 实战案例的开发经验
5. 最佳实践和性能优化技巧

希望本书能够帮助你在 Flutter 开发中更好地使用 `animated_text_kit`，为应用添加酷炫的文本动画效果。

---

**版本信息**

- animated_text_kit 版本：4.3.0
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
