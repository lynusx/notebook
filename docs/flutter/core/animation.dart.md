# Flutter animation.dart 完整详解与实战指南

## 前言

动画是 Flutter 框架的核心特性之一，它让应用界面变得生动、流畅，提升用户体验。Flutter 的动画系统建立在 `animation.dart` 库之上，这是一个功能强大且灵活的动画框架，提供了从简单到复杂的各种动画能力。

`package:flutter/animation.dart` 是 Flutter SDK 内置的核心动画库，它定义了动画系统的所有基础类和接口，包括：

- **Animation**：动画值的抽象表示
- **AnimationController**：动画控制器，管理动画的播放状态
- **Tween**：补间动画，定义动画的起始值和结束值
- **Curve**：动画曲线，控制动画的速度变化
- **Animatable**：可动画化的对象

本书将详细介绍 `animation.dart` 库中的所有类、接口和方法，帮助你深入理解 Flutter 动画系统的工作原理。

---

## 第1章 动画基础

### 1.1 Flutter 动画系统架构

Flutter 的动画系统采用分层设计：

```
┌─────────────────────────────────────────┐
│           高层 API（Widgets）            │
│  AnimatedWidget、AnimatedBuilder         │
├─────────────────────────────────────────┤
│           中层 API（动画库）              │
│  AnimationController、Tween、Curve       │
├─────────────────────────────────────────┤
│           底层 API（渲染）                │
│  SchedulerBinding、Ticker               │
└─────────────────────────────────────────┘
```

### 1.2 核心概念

| 概念                    | 说明                                     |
| ----------------------- | ---------------------------------------- |
| **Animation**           | 动画值的抽象，可以监听值的变化           |
| **AnimationController** | 控制动画的播放、暂停、停止、反向等       |
| **Tween**               | 定义动画的起始值和结束值之间的插值       |
| **Curve**               | 定义动画的速度曲线（线性、加速、减速等） |
| **Ticker**              | 定时器，每帧触发一次回调                 |

---

## 第2章 Animation 类

`Animation` 是 Flutter 动画系统的核心抽象类，它代表一个可以在一段时间内变化的值。

### 2.1 核心特性

| 特性       | 说明                                |
| ---------- | ----------------------------------- |
| 泛型支持   | `Animation<T>` 支持任意类型的动画值 |
| 可监听     | 可以添加监听器，在值变化时收到通知  |
| 可状态监听 | 可以监听动画状态的变化              |

### 2.2 核心属性

| 属性名      | 类型            | 说明                 |
| ----------- | --------------- | -------------------- |
| value       | T               | 当前动画值           |
| status      | AnimationStatus | 动画当前状态         |
| isCompleted | bool            | 动画是否已完成       |
| isDismissed | bool            | 动画是否已回到起始点 |

### 2.3 AnimationStatus 枚举

| 状态      | 说明                   |
| --------- | ---------------------- |
| dismissed | 动画在起始点           |
| forward   | 动画正在正向播放       |
| reverse   | 动画正在反向播放       |
| completed | 动画已完成（到达终点） |

### 2.4 核心方法

| 方法名               | 返回值       | 说明                |
| -------------------- | ------------ | ------------------- |
| addListener          | void         | 添加值变化监听器    |
| removeListener       | void         | 移除值变化监听器    |
| addStatusListener    | void         | 添加状态监听器      |
| removeStatusListener | void         | 移除状态监听器      |
| drive                | Animation<T> | 使用 Tween 驱动动画 |

### 2.5 使用示例

```dart
class AnimationDemo extends StatefulWidget {
  const AnimationDemo({super.key});

  @override
  State<AnimationDemo> createState() => _AnimationDemoState();
}

class _AnimationDemoState extends State<AnimationDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();

    // 创建动画控制器
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    // 创建动画
    _animation = Tween<double>(begin: 0, end: 300).animate(_controller);

    // 添加监听器
    _animation.addListener(() {
      setState(() {});
    });

    // 添加状态监听器
    _animation.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        print('Animation completed!');
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animation Demo')),
      body: Center(
        child: Container(
          width: _animation.value,
          height: _animation.value,
          color: Colors.blue,
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          if (_controller.isAnimating) {
            _controller.stop();
          } else {
            _controller.forward();
          }
        },
        child: const Icon(Icons.play_arrow),
      ),
    );
  }
}
```

> **Flutter框架小知识**
>
> `Animation` 本身不存储任何状态，它只是值的抽象表示。实际的动画状态由 `AnimationController` 管理。这种设计使得动画可以被多个 Widget 共享，而不会产生状态冲突。

---

## 第3章 AnimationController 类

`AnimationController` 是动画的控制中心，它管理动画的播放、暂停、停止和反向等操作。

### 3.1 构造函数参数

| 参数名            | 类型              | 默认值                   | 说明                 |
| ----------------- | ----------------- | ------------------------ | -------------------- |
| duration          | Duration?         | null                     | 动画持续时间         |
| reverseDuration   | Duration?         | null                     | 反向动画持续时间     |
| value             | double?           | null                     | 初始值（0.0 到 1.0） |
| lowerBound        | double            | 0.0                      | 下限值               |
| upperBound        | double            | 1.0                      | 上限值               |
| animationBehavior | AnimationBehavior | AnimationBehavior.normal | 动画行为             |
| vsync             | TickerProvider    | 必填                     | Ticker 提供者        |

### 3.2 AnimationBehavior 枚举

| 值       | 说明                               |
| -------- | ---------------------------------- |
| normal   | 正常行为，动画可能因系统负载而延迟 |
| preserve | 保持行为，动画会尽可能按时完成     |

### 3.3 核心方法

| 方法名      | 返回值       | 说明             |
| ----------- | ------------ | ---------------- |
| forward     | TickerFuture | 正向播放动画     |
| reverse     | TickerFuture | 反向播放动画     |
| animateTo   | TickerFuture | 动画到指定值     |
| animateBack | TickerFuture | 反向动画到指定值 |
| repeat      | TickerFuture | 重复播放动画     |
| fling       | TickerFuture | 弹性动画         |
| stop        | void         | 停止动画         |
| reset       | void         | 重置动画到起始值 |
| dispose     | void         | 释放资源         |

### 3.4 核心属性

| 属性名              | 类型      | 说明             |
| ------------------- | --------- | ---------------- |
| value               | double    | 当前值           |
| velocity            | double    | 当前速度         |
| lastElapsedDuration | Duration? | 上次动画持续时间 |
| isAnimating         | bool      | 是否正在动画     |
| isCompleted         | bool      | 是否已完成       |
| isDismissed         | bool      | 是否已回到起始点 |

### 3.5 完整示例

```dart
class ControllerDemo extends StatefulWidget {
  const ControllerDemo({super.key});

  @override
  State<ControllerDemo> createState() => _ControllerDemoState();
}

class _ControllerDemoState extends State<ControllerDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Controller Demo')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Container(
                width: 50 + _controller.value * 200,
                height: 50 + _controller.value * 200,
                color: Color.lerp(Colors.red, Colors.blue, _controller.value),
              );
            },
          ),
          const SizedBox(height: 50),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              ElevatedButton(
                onPressed: () => _controller.forward(),
                child: const Text('Forward'),
              ),
              ElevatedButton(
                onPressed: () => _controller.reverse(),
                child: const Text('Reverse'),
              ),
              ElevatedButton(
                onPressed: () => _controller.repeat(),
                child: const Text('Repeat'),
              ),
              ElevatedButton(
                onPressed: () => _controller.stop(),
                child: const Text('Stop'),
              ),
              ElevatedButton(
                onPressed: () => _controller.reset(),
                child: const Text('Reset'),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## 第4章 Tween 类

`Tween`（补间动画）定义了动画的起始值和结束值，并负责在这两个值之间进行插值。

### 4.1 核心特性

| 特性       | 说明                          |
| ---------- | ----------------------------- |
| 泛型支持   | `Tween<T>` 支持任意可插值类型 |
| 可链式调用 | 支持链式组合                  |
| 可自定义   | 可以创建自定义 Tween          |

### 4.2 构造函数参数

| 参数名 | 类型 | 默认值 | 说明   |
| ------ | ---- | ------ | ------ |
| begin  | T?   | null   | 起始值 |
| end    | T?   | null   | 结束值 |

### 4.3 核心方法

| 方法名    | 返回值        | 说明                        |
| --------- | ------------- | --------------------------- |
| lerp      | T             | 线性插值                    |
| transform | T             | 使用 Animation 的值进行插值 |
| chain     | Animatable<T> | 链式组合                    |
| animate   | Animation<T>  | 创建动画                    |
| evaluate  | T             | 评估当前值                  |

### 4.4 内置 Tween 类型

| Tween 类型            | 说明         | 示例                                                                          |
| --------------------- | ------------ | ----------------------------------------------------------------------------- |
| Tween<double>         | 双精度浮点数 | `Tween<double>(begin: 0, end: 100)`                                           |
| Tween<int>            | 整数         | `Tween<int>(begin: 0, end: 100)`                                              |
| ColorTween            | 颜色         | `ColorTween(begin: Colors.red, end: Colors.blue)`                             |
| SizeTween             | 尺寸         | `SizeTween(begin: Size.zero, end: Size(100, 100))`                            |
| RectTween             | 矩形         | `RectTween(begin: Rect.zero, end: Rect.largest)`                              |
| AlignmentTween        | 对齐         | `AlignmentTween(begin: Alignment.topLeft, end: Alignment.bottomRight)`        |
| EdgeInsetsTween       | 边距         | `EdgeInsetsTween(begin: EdgeInsets.zero, end: EdgeInsets.all(20))`            |
| BorderTween           | 边框         | `BorderTween(begin: Border.all(), end: Border.none)`                          |
| BorderRadiusTween     | 圆角         | `BorderRadiusTween(begin: BorderRadius.zero, end: BorderRadius.circular(10))` |
| BoxConstraintsTween   | 约束         | `BoxConstraintsTween(...)`                                                    |
| DecorationTween       | 装饰         | `DecorationTween(begin: BoxDecoration(), end: BoxDecoration())`               |
| TextStyleTween        | 文本样式     | `TextStyleTween(begin: TextStyle(), end: TextStyle())`                        |
| ThemeDataTween        | 主题         | `ThemeDataTween(begin: ThemeData.light(), end: ThemeData.dark())`             |
| MaterialPointArcTween | 弧形路径     | `MaterialPointArcTween(begin: Offset.zero, end: Offset(100, 100))`            |
| MaterialRectArcTween  | 矩形弧形路径 | `MaterialRectArcTween(...)`                                                   |
| ConstantTween         | 常量         | `ConstantTween(10)`                                                           |
| StepTween             | 步进         | `StepTween(begin: 0, end: 10)`                                                |

### 4.5 使用示例

```dart
class TweenDemo extends StatefulWidget {
  const TweenDemo({super.key});

  @override
  State<TweenDemo> createState() => _TweenDemoState();
}

class _TweenDemoState extends State<TweenDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Color?> _colorAnimation;
  late Animation<double> _sizeAnimation;
  late Animation<Alignment> _alignmentAnimation;
  late Animation<BorderRadius> _borderRadiusAnimation;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    );

    // 颜色动画
    _colorAnimation = ColorTween(
      begin: Colors.red,
      end: Colors.blue,
    ).animate(_controller);

    // 尺寸动画
    _sizeAnimation = Tween<double>(
      begin: 100,
      end: 200,
    ).animate(_controller);

    // 对齐动画
    _alignmentAnimation = AlignmentTween(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
    ).animate(_controller);

    // 圆角动画
    _borderRadiusAnimation = BorderRadiusTween(
      begin: BorderRadius.zero,
      end: BorderRadius.circular(50),
    ).animate(_controller);

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
      animation: _controller,
      builder: (context, child) {
        return Scaffold(
          appBar: AppBar(title: const Text('Tween Demo')),
          body: Align(
            alignment: _alignmentAnimation.value,
            child: Container(
              width: _sizeAnimation.value,
              height: _sizeAnimation.value,
              decoration: BoxDecoration(
                color: _colorAnimation.value,
                borderRadius: _borderRadiusAnimation.value,
              ),
            ),
          ),
        );
      },
    );
  }
}
```

### 4.6 链式组合

```dart
// 使用 chain 方法组合多个 Tween
final animation = Tween<double>(begin: 0, end: 100)
    .chain(CurveTween(curve: Curves.easeIn))
    .chain(Tween<double>(begin: 0, end: 1))
    .animate(_controller);
```

---

## 第5章 Curve 类

`Curve` 定义了动画的速度变化曲线，让动画更加自然流畅。

### 5.1 核心特性

| 特性     | 说明                                    |
| -------- | --------------------------------------- |
| 可组合   | 可以使用 `CurvedAnimation` 包装其他动画 |
| 可自定义 | 可以创建自定义 Curve                    |

### 5.2 核心方法

| 方法名            | 返回值 | 说明                             |
| ----------------- | ------ | -------------------------------- |
| transform         | double | 转换输入值（0.0 到 1.0）到输出值 |
| transformInternal | double | 内部转换方法（子类重写）         |
| flipped           | Curve  | 获取反向曲线                     |

### 5.3 内置 Curves

#### 线性曲线

| 曲线          | 说明           |
| ------------- | -------------- |
| Curves.linear | 线性，匀速运动 |

#### 加速曲线

| 曲线               | 说明                 |
| ------------------ | -------------------- |
| Curves.easeIn      | 缓入，开始慢，结束快 |
| Curves.easeInQuad  | 二次方缓入           |
| Curves.easeInCubic | 三次方缓入           |
| Curves.easeInQuart | 四次方缓入           |
| Curves.easeInQuint | 五次方缓入           |
| Curves.easeInExpo  | 指数缓入             |
| Curves.easeInCirc  | 圆形缓入             |
| Curves.easeInBack  | 回弹缓入             |
| Curves.easeInSine  | 正弦缓入             |

#### 减速曲线

| 曲线                | 说明                 |
| ------------------- | -------------------- |
| Curves.easeOut      | 缓出，开始快，结束慢 |
| Curves.easeOutQuad  | 二次方缓出           |
| Curves.easeOutCubic | 三次方缓出           |
| Curves.easeOutQuart | 四次方缓出           |
| Curves.easeOutQuint | 五次方缓出           |
| Curves.easeOutExpo  | 指数缓出             |
| Curves.easeOutCirc  | 圆形缓出             |
| Curves.easeOutBack  | 回弹缓出             |
| Curves.easeOutSine  | 正弦缓出             |

#### 先加速后减速曲线

| 曲线                  | 说明           |
| --------------------- | -------------- |
| Curves.easeInOut      | 缓入缓出       |
| Curves.easeInOutQuad  | 二次方缓入缓出 |
| Curves.easeInOutCubic | 三次方缓入缓出 |
| Curves.easeInOutQuart | 四次方缓入缓出 |
| Curves.easeInOutQuint | 五次方缓入缓出 |
| Curves.easeInOutExpo  | 指数缓入缓出   |
| Curves.easeInOutCirc  | 圆形缓入缓出   |
| Curves.easeInOutBack  | 回弹缓入缓出   |
| Curves.easeInOutSine  | 正弦缓入缓出   |

#### 弹性曲线

| 曲线                | 说明         |
| ------------------- | ------------ |
| Curves.elasticIn    | 弹性缓入     |
| Curves.elasticOut   | 弹性缓出     |
| Curves.elasticInOut | 弹性缓入缓出 |

#### 弹跳曲线

| 曲线               | 说明         |
| ------------------ | ------------ |
| Curves.bounceIn    | 弹跳缓入     |
| Curves.bounceOut   | 弹跳缓出     |
| Curves.bounceInOut | 弹跳缓入缓出 |

#### 其他曲线

| 曲线                          | 说明                     |
| ----------------------------- | ------------------------ |
| Curves.fastOutSlowIn          | Material Design 标准曲线 |
| Curves.slowMiddle             | 中间慢                   |
| Curves.decelerate             | 减速                     |
| Curves.fastLinearToSlowEaseIn | 快速线性到慢速缓入       |

### 5.4 使用示例

```dart
class CurveDemo extends StatefulWidget {
  const CurveDemo({super.key});

  @override
  State<CurveDemo> createState() => _CurveDemoState();
}

class _CurveDemoState extends State<CurveDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  Curve _selectedCurve = Curves.linear;

  final List<Map<String, dynamic>> _curves = [
    {'name': 'Linear', 'curve': Curves.linear},
    {'name': 'Ease In', 'curve': Curves.easeIn},
    {'name': 'Ease Out', 'curve': Curves.easeOut},
    {'name': 'Ease In Out', 'curve': Curves.easeInOut},
    {'name': 'Bounce Out', 'curve': Curves.bounceOut},
    {'name': 'Elastic Out', 'curve': Curves.elasticOut},
    {'name': 'Back Out', 'curve': Curves.easeOutBack},
  ];

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final animation = Tween<double>(begin: 0, end: 300).animate(
      CurvedAnimation(parent: _controller, curve: _selectedCurve),
    );

    return Scaffold(
      appBar: AppBar(title: const Text('Curve Demo')),
      body: Column(
        children: [
          Expanded(
            child: AnimatedBuilder(
              animation: animation,
              builder: (context, child) {
                return Stack(
                  children: [
                    Positioned(
                      left: animation.value,
                      top: 100,
                      child: Container(
                        width: 50,
                        height: 50,
                        decoration: BoxDecoration(
                          color: Colors.blue,
                          borderRadius: BorderRadius.circular(25),
                        ),
                      ),
                    ),
                  ],
                );
              },
            ),
          ),
          Wrap(
            spacing: 8,
            runSpacing: 8,
            children: _curves.map((curveData) {
              return ElevatedButton(
                onPressed: () {
                  setState(() {
                    _selectedCurve = curveData['curve'];
                  });
                  _controller.reset();
                  _controller.forward();
                },
                child: Text(curveData['name']),
              );
            }).toList(),
          ),
          const SizedBox(height: 20),
        ],
      ),
    );
  }
}
```

### 5.5 自定义 Curve

```dart
class BounceCurve extends Curve {
  @override
  double transformInternal(double t) {
    if (t < 0.5) {
      return 4 * t * t * t;
    } else {
      final f = (t - 1);
      return 1 + 4 * f * f * f;
    }
  }
}

class ElasticCurve extends Curve {
  final double amplitude;
  final double period;

  const ElasticCurve({this.amplitude = 1.0, this.period = 0.4});

  @override
  double transformInternal(double t) {
    final s = period / 4.0;
    return amplitude *
        pow(2.0, -10.0 * t) *
        sin((t - s) * (2.0 * pi) / period) +
        1.0;
  }
}
```

---

## 第6章 CurvedAnimation 类

`CurvedAnimation` 将 Curve 应用到 Animation 上，产生曲线动画效果。

### 6.1 构造函数参数

| 参数名       | 类型              | 默认值 | 说明     |
| ------------ | ----------------- | ------ | -------- |
| parent       | Animation<double> | 必填   | 父动画   |
| curve        | Curve             | 必填   | 正向曲线 |
| reverseCurve | Curve?            | null   | 反向曲线 |

### 6.2 使用示例

```dart
final curvedAnimation = CurvedAnimation(
  parent: _controller,
  curve: Curves.easeInOut,
  reverseCurve: Curves.easeOut,
);

final sizeAnimation = Tween<double>(begin: 100, end: 200)
    .animate(curvedAnimation);
```

---

## 第7章 Animatable 类

`Animatable` 是一个抽象类，代表可以产生动画值的对象。Tween 继承自 Animatable。

### 7.1 核心方法

| 方法名   | 返回值        | 说明             |
| -------- | ------------- | ---------------- |
| evaluate | T             | 评估给定动画的值 |
| animate  | Animation<T>  | 创建动画         |
| chain    | Animatable<T> | 链式组合         |

### 7.2 使用示例

```dart
// 创建可动画对象
final animatable = TweenSequence<double>([
  TweenSequenceItem(
    tween: Tween<double>(begin: 0, end: 100)
        .chain(CurveTween(curve: Curves.easeIn)),
    weight: 1.0,
  ),
  TweenSequenceItem(
    tween: Tween<double>(begin: 100, end: 200)
        .chain(CurveTween(curve: Curves.easeOut)),
    weight: 1.0,
  ),
]);

final animation = animatable.animate(_controller);
```

---

## 第8章 TweenSequence 类

`TweenSequence` 允许将多个 Tween 串联起来，形成连续的动画序列。

### 8.1 构造函数参数

| 参数名 | 类型                       | 说明       |
| ------ | -------------------------- | ---------- |
| items  | List<TweenSequenceItem<T>> | 序列项列表 |

### 8.2 TweenSequenceItem

| 参数名 | 类型          | 说明                               |
| ------ | ------------- | ---------------------------------- |
| tween  | Animatable<T> | 补间动画                           |
| weight | double        | 权重（决定该段动画占总时长的比例） |

### 8.3 使用示例

```dart
class SequenceDemo extends StatefulWidget {
  const SequenceDemo({super.key});

  @override
  State<SequenceDemo> createState() => _SequenceDemoState();
}

class _SequenceDemoState extends State<SequenceDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Color?> _colorSequence;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      duration: const Duration(seconds: 4),
      vsync: this,
    );

    _colorSequence = TweenSequence<Color?>([
      TweenSequenceItem(
        tween: ColorTween(begin: Colors.red, end: Colors.orange)
            .chain(CurveTween(curve: Curves.easeIn)),
        weight: 1.0,
      ),
      TweenSequenceItem(
        tween: ColorTween(begin: Colors.orange, end: Colors.yellow)
            .chain(CurveTween(curve: Curves.linear)),
        weight: 1.0,
      ),
      TweenSequenceItem(
        tween: ColorTween(begin: Colors.yellow, end: Colors.green)
            .chain(CurveTween(curve: Curves.easeOut)),
        weight: 1.0,
      ),
      TweenSequenceItem(
        tween: ColorTween(begin: Colors.green, end: Colors.blue)
            .chain(CurveTween(curve: Curves.bounceOut)),
        weight: 1.0,
      ),
    ]).animate(_controller);

    _controller.repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _colorSequence,
      builder: (context, child) {
        return Scaffold(
          backgroundColor: _colorSequence.value,
          appBar: AppBar(
            title: const Text('Sequence Demo'),
            backgroundColor: Colors.transparent,
          ),
          body: const Center(
            child: Text(
              'Color Sequence Animation',
              style: TextStyle(
                fontSize: 24,
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        );
      },
    );
  }
}
```

---

## 第9章 ReverseAnimation 类

`ReverseAnimation` 将动画反向播放。

### 9.1 使用示例

```dart
final forwardAnimation = Tween<double>(begin: 0, end: 100)
    .animate(_controller);

final reverseAnimation = ReverseAnimation(forwardAnimation);
```

---

## 第10章 ProxyAnimation 类

`ProxyAnimation` 是一个代理动画，它可以将动画的控制权转移给另一个动画。

### 10.1 使用示例

```dart
final proxyAnimation = ProxyAnimation();

// 稍后设置父动画
proxyAnimation.parent = someAnimation;
```

---

## 第11章 实战案例

### 11.1 心跳动画

```dart
class HeartbeatAnimation extends StatefulWidget {
  const HeartbeatAnimation({super.key});

  @override
  State<HeartbeatAnimation> createState() => _HeartbeatAnimationState();
}

class _HeartbeatAnimationState extends State<HeartbeatAnimation>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      duration: const Duration(milliseconds: 800),
      vsync: this,
    );

    _scaleAnimation = TweenSequence<double>([
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.0, end: 1.3)
            .chain(CurveTween(curve: Curves.easeOut)),
        weight: 30,
      ),
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.3, end: 1.0)
            .chain(CurveTween(curve: Curves.easeIn)),
        weight: 30,
      ),
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.0, end: 1.2)
            .chain(CurveTween(curve: Curves.easeOut)),
        weight: 20,
      ),
      TweenSequenceItem(
        tween: Tween<double>(begin: 1.2, end: 1.0)
            .chain(CurveTween(curve: Curves.easeIn)),
        weight: 20,
      ),
    ]).animate(_controller);

    _controller.repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: AnimatedBuilder(
          animation: _scaleAnimation,
          builder: (context, child) {
            return Transform.scale(
              scale: _scaleAnimation.value,
              child: const Icon(
                Icons.favorite,
                size: 100,
                color: Colors.red,
              ),
            );
          },
        ),
      ),
    );
  }
}
```

### 11.2 加载动画

```dart
class LoadingAnimation extends StatefulWidget {
  const LoadingAnimation({super.key});

  @override
  State<LoadingAnimation> createState() => _LoadingAnimationState();
}

class _LoadingAnimationState extends State<LoadingAnimation>
    with TickerProviderStateMixin {
  late List<AnimationController> _controllers;
  late List<Animation<double>> _animations;

  @override
  void initState() {
    super.initState();

    _controllers = List.generate(3, (index) {
      return AnimationController(
        duration: const Duration(milliseconds: 600),
        vsync: this,
      );
    });

    _animations = _controllers.map((controller) {
      return Tween<double>(begin: 0.5, end: 1.0).animate(
        CurvedAnimation(parent: controller, curve: Curves.easeInOut),
      );
    }).toList();

    // 依次启动动画
    for (var i = 0; i < _controllers.length; i++) {
      Future.delayed(Duration(milliseconds: i * 150), () {
        _controllers[i].repeat(reverse: true);
      });
    }
  }

  @override
  void dispose() {
    for (final controller in _controllers) {
      controller.dispose();
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: List.generate(3, (index) {
            return AnimatedBuilder(
              animation: _animations[index],
              builder: (context, child) {
                return Container(
                  width: 20,
                  height: 20,
                  margin: const EdgeInsets.symmetric(horizontal: 4),
                  decoration: BoxDecoration(
                    color: Colors.blue,
                    borderRadius: BorderRadius.circular(10),
                  ),
                  transform: Matrix4.identity()
                    ..scale(_animations[index].value),
                );
              },
            );
          }),
        ),
      ),
    );
  }
}
```

### 11.3 呼吸灯效果

```dart
class BreathingLight extends StatefulWidget {
  const BreathingLight({super.key});

  @override
  State<BreathingLight> createState() => _BreathingLightState();
}

class _BreathingLightState extends State<BreathingLight>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _opacityAnimation;
  late Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    _opacityAnimation = Tween<double>(begin: 0.3, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );

    _scaleAnimation = Tween<double>(begin: 0.8, end: 1.2).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
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
    return Scaffold(
      backgroundColor: Colors.black,
      body: Center(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Opacity(
              opacity: _opacityAnimation.value,
              child: Transform.scale(
                scale: _scaleAnimation.value,
                child: Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: Colors.cyan,
                    shape: BoxShape.circle,
                    boxShadow: [
                      BoxShadow(
                        color: Colors.cyan.withOpacity(_opacityAnimation.value),
                        blurRadius: 50,
                        spreadRadius: 20,
                      ),
                    ],
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

---

## 第12章 最佳实践

### 12.1 性能优化

```dart
// 使用 AnimatedBuilder 而不是 setState
AnimatedBuilder(
  animation: _animation,
  builder: (context, child) {
    return Transform.translate(
      offset: Offset(_animation.value, 0),
      child: child,  // 复用子组件
    );
  },
  child: const ExpensiveWidget(),  // 不会重建
)

// 及时释放资源
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

### 12.2 常用模式

```dart
// 1. 正向播放一次
_controller.forward();

// 2. 反向播放
_controller.reverse();

// 3. 循环播放
_controller.repeat();

// 4. 往返播放
_controller.repeat(reverse: true);

// 5. 弹性动画
_controller.fling(velocity: 1.0);

// 6. 动画到指定值
_controller.animateTo(0.5);
```

### 12.3 DO 和 DON'T

#### ✅ DO

- 使用 `TickerProviderStateMixin` 或 `SingleTickerProviderStateMixin`
- 在 `dispose` 中释放 `AnimationController`
- 使用 `AnimatedBuilder` 优化性能
- 使用 `Tween` 定义动画范围
- 使用 `Curve` 让动画更自然

#### ❌ DON'T

- 在 `setState` 中直接修改动画值
- 忘记释放 `AnimationController`
- 在动画中执行耗时操作
- 创建过多的 `AnimationController`

---

## 附录：animation.dart API 速查表

### 核心类

| 类名                | 说明           |
| ------------------- | -------------- |
| Animation<T>        | 动画抽象类     |
| AnimationController | 动画控制器     |
| AnimationStatus     | 动画状态枚举   |
| Tween<T>            | 补间动画       |
| Curve               | 动画曲线       |
| CurvedAnimation     | 曲线动画包装器 |
| Animatable<T>       | 可动画对象     |
| TweenSequence<T>    | 补间序列       |
| ReverseAnimation    | 反向动画       |
| ProxyAnimation      | 代理动画       |

### 常用 Tween

| Tween             | 说明         |
| ----------------- | ------------ |
| Tween<double>     | 双精度浮点数 |
| Tween<int>        | 整数         |
| ColorTween        | 颜色         |
| SizeTween         | 尺寸         |
| RectTween         | 矩形         |
| AlignmentTween    | 对齐         |
| EdgeInsetsTween   | 边距         |
| BorderRadiusTween | 圆角         |
| TextStyleTween    | 文本样式     |

### 常用 Curves

| Curve                | 说明              |
| -------------------- | ----------------- |
| Curves.linear        | 线性              |
| Curves.easeIn        | 缓入              |
| Curves.easeOut       | 缓出              |
| Curves.easeInOut     | 缓入缓出          |
| Curves.bounceIn      | 弹跳缓入          |
| Curves.bounceOut     | 弹跳缓出          |
| Curves.elasticIn     | 弹性缓入          |
| Curves.elasticOut    | 弹性缓出          |
| Curves.fastOutSlowIn | Material 标准曲线 |

---

## 结语

Flutter 的 `animation.dart` 库是一个功能强大且灵活的动画框架。通过本书的学习，你应该已经掌握了：

1. Animation 和 AnimationController 的核心概念
2. Tween 的使用和各种类型
3. Curve 的选择和自定义
4. TweenSequence 的链式动画
5. 实战动画案例的开发
6. 性能优化和最佳实践

希望本书能够帮助你在 Flutter 开发中创建出流畅、自然的动画效果！

---

**版本信息**

- Flutter SDK 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
