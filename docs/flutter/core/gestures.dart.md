# Flutter Gestures Dart 完整参考手册

## package:flutter/gestures.dart

> **版本**: Flutter 3.41.0  
> **适用范围**: Flutter 手势识别、触摸事件处理  
> **官方文档**: [api.flutter.dev/flutter/gestures/gestures-library.html](https://api.flutter.dev/flutter/gestures/gestures-library.html)

---

## 第一章：库概述

`flutter/gestures.dart` 是 Flutter 框架的手势识别库，提供了从原始指针事件到高级手势识别的完整解决方案。这个库是构建交互式 Flutter 应用的核心，它通过手势竞技场（GestureArena）机制协调多个手势识别器之间的竞争，确保用户交互的准确性和流畅性。

### 1.1 库的主要组成部分

| 类别           | 说明              | 主要类                                                              |
| -------------- | ----------------- | ------------------------------------------------------------------- |
| **手势识别器** | 识别各种手势类型  | TapGestureRecognizer、DragGestureRecognizer、ScaleGestureRecognizer |
| **手势竞技场** | 协调手势竞争      | GestureArenaManager、GestureArenaEntry                              |
| **手势详情**   | 手势事件数据      | TapDownDetails、DragUpdateDetails、ScaleUpdateDetails               |
| **手势检测器** | Widget 层手势检测 | GestureDetector、RawGestureDetector                                 |
| **指针事件**   | 原始触摸事件      | PointerDownEvent、PointerMoveEvent、PointerUpEvent                  |
| **速度追踪**   | 手势速度计算      | VelocityTracker、VelocityEstimate                                   |
| **多点触控**   | 多指手势支持      | MultiDragGestureRecognizer、MultiTapGestureRecognizer               |

### 1.2 导入方式

```dart
import 'package:flutter/gestures.dart';
```

### 1.3 手势处理流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    手势处理流程                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │  触摸屏幕     │───▶│ PointerEvent  │───▶│ 命中测试      │      │
│  │  (手指/触控笔) │    │  (原始事件)   │    │ (hitTest)    │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│                                                   │             │
│                                                   ▼             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   手势回调    │◀───│  手势竞技场   │◀───│ GestureRecognizer │
│  │ (onTap/onDrag)│    │ (GestureArena)│    │  (手势识别器)      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│         ▲                    │                                  │
│         │                    │ 竞争/协调                         │
│         │                    ▼                                  │
│  ┌──────────────┐    ┌──────────────┐                          │
│  │   Widget更新  │◀───│ 多个识别器竞争 │                          │
│  │  (setState)  │    │ (Winner/Loser)│                          │
│  └──────────────┘    └──────────────┘                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> **补充知识**：Flutter 的手势系统采用竞技场模式，当多个手势识别器同时竞争同一触摸事件时，GestureArena 会决定哪个识别器获胜。获胜的识别器继续接收事件，失败的识别器则被拒绝。

---

## 第二章：GestureDetector

`GestureDetector` 是 Flutter 中最常用的手势检测 Widget，它封装了各种手势识别器，提供了简洁的回调接口。

### 2.1 完整构造函数

```dart
const GestureDetector({
  super.key,
  this.child,
  this.onTapDown,
  this.onTapUp,
  this.onTap,
  this.onTapCancel,
  this.onSecondaryTap,
  this.onSecondaryTapDown,
  this.onSecondaryTapUp,
  this.onSecondaryTapCancel,
  this.onTertiaryTapDown,
  this.onTertiaryTapUp,
  this.onTertiaryTapCancel,
  this.onDoubleTapDown,
  this.onDoubleTap,
  this.onDoubleTapCancel,
  this.onLongPressDown,
  this.onLongPressCancel,
  this.onLongPress,
  this.onLongPressStart,
  this.onLongPressMoveUpdate,
  this.onLongPressUp,
  this.onLongPressEnd,
  this.onSecondaryLongPressDown,
  this.onSecondaryLongPressCancel,
  this.onSecondaryLongPress,
  this.onSecondaryLongPressStart,
  this.onSecondaryLongPressMoveUpdate,
  this.onSecondaryLongPressUp,
  this.onSecondaryLongPressEnd,
  this.onVerticalDragDown,
  this.onVerticalDragStart,
  this.onVerticalDragUpdate,
  this.onVerticalDragEnd,
  this.onVerticalDragCancel,
  this.onHorizontalDragDown,
  this.onHorizontalDragStart,
  this.onHorizontalDragUpdate,
  this.onHorizontalDragEnd,
  this.onHorizontalDragCancel,
  this.onForcePressStart,
  this.onForcePressPeak,
  this.onForcePressUpdate,
  this.onForcePressEnd,
  this.onPanDown,
  this.onPanStart,
  this.onPanUpdate,
  this.onPanEnd,
  this.onPanCancel,
  this.onScaleStart,
  this.onScaleUpdate,
  this.onScaleEnd,
  this.behavior,
  this.excludeFromSemantics = false,
  this.dragStartBehavior = DragStartBehavior.start,
  this.trackpadScrollCausesScale = false,
});
```

### 2.2 核心属性详解

#### 点击手势回调

| 属性名        | 类型                      | 说明           |
| ------------- | ------------------------- | -------------- |
| `onTapDown`   | GestureTapDownCallback?   | 手指按下时触发 |
| `onTapUp`     | GestureTapUpCallback?     | 手指抬起时触发 |
| `onTap`       | GestureTapCallback?       | 点击完成时触发 |
| `onTapCancel` | GestureTapCancelCallback? | 点击取消时触发 |
| `onDoubleTap` | GestureTapCallback?       | 双击时触发     |
| `onLongPress` | GestureLongPressCallback? | 长按时触发     |

#### 拖动手势回调

| 属性名                   | 类型                       | 说明                   |
| ------------------------ | -------------------------- | ---------------------- |
| `onVerticalDragStart`    | GestureDragStartCallback?  | 垂直拖动开始时触发     |
| `onVerticalDragUpdate`   | GestureDragUpdateCallback? | 垂直拖动更新时触发     |
| `onVerticalDragEnd`      | GestureDragEndCallback?    | 垂直拖动结束时触发     |
| `onHorizontalDragStart`  | GestureDragStartCallback?  | 水平拖动开始时触发     |
| `onHorizontalDragUpdate` | GestureDragUpdateCallback? | 水平拖动更新时触发     |
| `onHorizontalDragEnd`    | GestureDragEndCallback?    | 水平拖动结束时触发     |
| `onPanStart`             | GestureDragStartCallback?  | 任意方向拖动开始时触发 |
| `onPanUpdate`            | GestureDragUpdateCallback? | 任意方向拖动更新时触发 |
| `onPanEnd`               | GestureDragEndCallback?    | 任意方向拖动结束时触发 |

#### 缩放手势回调

| 属性名          | 类型                        | 说明           |
| --------------- | --------------------------- | -------------- |
| `onScaleStart`  | GestureScaleStartCallback?  | 缩放开始时触发 |
| `onScaleUpdate` | GestureScaleUpdateCallback? | 缩放更新时触发 |
| `onScaleEnd`    | GestureScaleEndCallback?    | 缩放结束时触发 |

#### 行为配置

| 属性名                      | 类型              | 默认值                  | 说明                   |
| --------------------------- | ----------------- | ----------------------- | ---------------------- |
| `behavior`                  | HitTestBehavior?  | null                    | 命中测试行为           |
| `excludeFromSemantics`      | bool              | false                   | 是否排除语义           |
| `dragStartBehavior`         | DragStartBehavior | DragStartBehavior.start | 拖动开始行为           |
| `trackpadScrollCausesScale` | bool              | false                   | 触控板滚动是否触发缩放 |

### 2.3 HitTestBehavior 枚举

| 值                             | 说明                                              |
| ------------------------------ | ------------------------------------------------- |
| `HitTestBehavior.deferToChild` | 将命中测试委托给子 Widget                         |
| `HitTestBehavior.opaque`       | 将自身视为不透明，阻止事件传递给下方的 Widget     |
| `HitTestBehavior.translucent`  | 半透明，接收事件但允许事件继续传递给下方的 Widget |

### 2.4 DragStartBehavior 枚举

| 值                        | 说明                     |
| ------------------------- | ------------------------ |
| `DragStartBehavior.start` | 从拖动开始位置计算 delta |
| `DragStartBehavior.down`  | 从手指按下位置计算 delta |

### 2.5 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'GestureDetector Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const GestureDetectorDemoPage(),
    );
  }
}

class GestureDetectorDemoPage extends StatefulWidget {
  const GestureDetectorDemoPage({super.key});

  @override
  State<GestureDetectorDemoPage> createState() => _GestureDetectorDemoPageState();
}

class _GestureDetectorDemoPageState extends State<GestureDetectorDemoPage> {
  String _lastGesture = '等待手势...';
  Offset _tapPosition = Offset.zero;
  double _scale = 1.0;
  double _rotation = 0.0;
  Offset _offset = Offset.zero;

  void _updateGesture(String gesture, [Offset? position]) {
    setState(() {
      _lastGesture = gesture;
      if (position != null) {
        _tapPosition = position;
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('GestureDetector 演示'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 状态显示
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('最后手势: $_lastGesture'),
                    Text('位置: (${_tapPosition.dx.toStringAsFixed(1)}, ${_tapPosition.dy.toStringAsFixed(1)})'),
                    Text('缩放: ${_scale.toStringAsFixed(2)}'),
                    Text('旋转: ${_rotation.toStringAsFixed(2)}°'),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 1. 点击手势
            _buildSectionTitle('1. 点击手势'),
            GestureDetector(
              onTapDown: (details) => _updateGesture('TapDown', details.localPosition),
              onTapUp: (details) => _updateGesture('TapUp', details.localPosition),
              onTap: () => _updateGesture('Tap'),
              onTapCancel: () => _updateGesture('TapCancel'),
              onDoubleTap: () => _updateGesture('DoubleTap'),
              onLongPress: () => _updateGesture('LongPress'),
              child: Container(
                width: double.infinity,
                height: 100,
                color: Colors.blue,
                child: const Center(
                  child: Text(
                    '点击/双击/长按我',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 2. 垂直拖动
            _buildSectionTitle('2. 垂直拖动'),
            GestureDetector(
              onVerticalDragStart: (details) => _updateGesture('VerticalDragStart'),
              onVerticalDragUpdate: (details) {
                _updateGesture('VerticalDragUpdate: ${details.delta.dy.toStringAsFixed(1)}');
              },
              onVerticalDragEnd: (details) => _updateGesture('VerticalDragEnd'),
              child: Container(
                width: double.infinity,
                height: 100,
                color: Colors.green,
                child: const Center(
                  child: Text(
                    '垂直拖动我',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 3. 水平拖动
            _buildSectionTitle('3. 水平拖动'),
            GestureDetector(
              onHorizontalDragStart: (details) => _updateGesture('HorizontalDragStart'),
              onHorizontalDragUpdate: (details) {
                _updateGesture('HorizontalDragUpdate: ${details.delta.dx.toStringAsFixed(1)}');
              },
              onHorizontalDragEnd: (details) => _updateGesture('HorizontalDragEnd'),
              child: Container(
                width: double.infinity,
                height: 100,
                color: Colors.orange,
                child: const Center(
                  child: Text(
                    '水平拖动我',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 4. 任意方向拖动 (Pan)
            _buildSectionTitle('4. 任意方向拖动 (Pan)'),
            GestureDetector(
              onPanStart: (details) => _updateGesture('PanStart'),
              onPanUpdate: (details) {
                setState(() {
                  _offset += details.delta;
                  _updateGesture('PanUpdate: (${details.delta.dx.toStringAsFixed(1)}, ${details.delta.dy.toStringAsFixed(1)})');
                });
              },
              onPanEnd: (details) => _updateGesture('PanEnd'),
              child: Transform.translate(
                offset: _offset,
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.purple,
                  child: const Center(
                    child: Text(
                      '拖动我',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 5. 缩放手势
            _buildSectionTitle('5. 缩放手势'),
            GestureDetector(
              onScaleStart: (details) => _updateGesture('ScaleStart'),
              onScaleUpdate: (details) {
                setState(() {
                  _scale = details.scale;
                  _rotation = details.rotation * 180 / 3.14159;
                  _updateGesture('ScaleUpdate: ${details.scale.toStringAsFixed(2)}');
                });
              },
              onScaleEnd: (details) => _updateGesture('ScaleEnd'),
              child: Transform.scale(
                scale: _scale,
                child: Transform.rotate(
                  angle: details: details.rotation,
                  child: Container(
                    width: 150,
                    height: 150,
                    color: Colors.teal,
                    child: const Center(
                      child: Text(
                        '双指缩放/旋转',
                        style: TextStyle(color: Colors.white),
                        textAlign: TextAlign.center,
                      ),
                    ),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // 6. 嵌套手势
            _buildSectionTitle('6. 嵌套手势 (父子关系)'),
            GestureDetector(
              onTap: () => _updateGesture('父容器点击'),
              child: Container(
                width: double.infinity,
                height: 150,
                color: Colors.indigo,
                child: Center(
                  child: GestureDetector(
                    onTap: () => _updateGesture('子容器点击'),
                    child: Container(
                      width: 100,
                      height: 100,
                      color: Colors.pink,
                      child: const Center(
                        child: Text(
                          '点击我',
                          style: TextStyle(color: Colors.white),
                        ),
                      ),
                    ),
                  ),
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
        style: const TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}
```

> **补充知识**：`GestureDetector` 内部使用 `RawGestureDetector` 和各种手势识别器（GestureRecognizer）来实现手势检测。当需要更精细的控制时，可以直接使用 `RawGestureDetector` 和自定义识别器。

---

## 第三章：RawGestureDetector

`RawGestureDetector` 是 `GestureDetector` 的底层实现，提供了更灵活的手势识别器配置方式。当需要自定义手势识别器或配置识别器参数时，应该使用 `RawGestureDetector`。

### 3.1 完整构造函数

```dart
const RawGestureDetector({
  super.key,
  this.child,
  this.gestures = const <Type, GestureRecognizerFactory>{},
  this.behavior,
  this.excludeFromSemantics = false,
  this.semantics,
});
```

### 3.2 核心属性详解

| 属性名                 | 类型                                | 默认值 | 说明               |
| ---------------------- | ----------------------------------- | ------ | ------------------ |
| `child`                | Widget?                             | null   | 子 Widget          |
| `gestures`             | Map<Type, GestureRecognizerFactory> | {}     | 手势识别器配置映射 |
| `behavior`             | HitTestBehavior?                    | null   | 命中测试行为       |
| `excludeFromSemantics` | bool                                | false  | 是否排除语义       |
| `semantics`            | SemanticsGestureDelegate?           | null   | 语义手势委托       |

### 3.3 手势识别器工厂

```dart
// 创建手势识别器工厂的常用方式
GestureRecognizerFactoryWithHandlers<TapGestureRecognizer>(
  () => TapGestureRecognizer(), // 创建识别器
  (TapGestureRecognizer instance) {
    // 配置识别器
    instance.onTap = () => debugPrint('Tap');
    instance.onTapDown = (details) => debugPrint('TapDown: $details');
  },
)
```

### 3.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class RawGestureDetectorDemo extends StatefulWidget {
  const RawGestureDetectorDemo({super.key});

  @override
  State<RawGestureDetectorDemo> createState() => _RawGestureDetectorDemoState();
}

class _RawGestureDetectorDemoState extends State<RawGestureDetectorDemo> {
  String _gestureLog = '';
  int _tapCount = 0;
  double _scale = 1.0;

  void _log(String message) {
    setState(() {
      _gestureLog = '$message\n$_gestureLog';
      if (_gestureLog.length > 500) {
        _gestureLog = _gestureLog.substring(0, 500);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('RawGestureDetector 演示'),
      ),
      body: Column(
        children: [
          // 使用 RawGestureDetector 配置自定义手势识别器
          Expanded(
            flex: 2,
            child: RawGestureDetector(
              gestures: {
                // 配置点击识别器
                TapGestureRecognizer: GestureRecognizerFactoryWithHandlers<
                    TapGestureRecognizer>(
                  () => TapGestureRecognizer(),
                  (TapGestureRecognizer instance) {
                    instance
                      ..onTapDown = (details) {
                        _log('TapDown: ${details.localPosition}');
                      }
                      ..onTapUp = (details) {
                        _log('TapUp: ${details.localPosition}');
                      }
                      ..onTap = () {
                        _tapCount++;
                        _log('Tap #$_tapCount');
                      }
                      ..onTapCancel = () {
                        _log('TapCancel');
                      };
                  },
                ),
                // 配置双击识别器
                DoubleTapGestureRecognizer: GestureRecognizerFactoryWithHandlers<
                    DoubleTapGestureRecognizer>(
                  () => DoubleTapGestureRecognizer(),
                  (DoubleTapGestureRecognizer instance) {
                    instance
                      ..onDoubleTapDown = (details) {
                        _log('DoubleTapDown');
                      }
                      ..onDoubleTap = () {
                        _log('DoubleTap!');
                      }
                      ..onDoubleTapCancel = () {
                        _log('DoubleTapCancel');
                      };
                  },
                ),
                // 配置长按识别器
                LongPressGestureRecognizer: GestureRecognizerFactoryWithHandlers<
                    LongPressGestureRecognizer>(
                  () => LongPressGestureRecognizer(),
                  (LongPressGestureRecognizer instance) {
                    instance
                      ..onLongPressDown = (details) {
                        _log('LongPressDown');
                      }
                      ..onLongPress = () {
                        _log('LongPress!');
                      }
                      ..onLongPressStart = (details) {
                        _log('LongPressStart');
                      }
                      ..onLongPressMoveUpdate = (details) {
                        _log('LongPressMoveUpdate: ${details.offsetFromOrigin}');
                      }
                      ..onLongPressEnd = (details) {
                        _log('LongPressEnd');
                      };
                  },
                ),
                // 配置缩放手势识别器
                ScaleGestureRecognizer: GestureRecognizerFactoryWithHandlers<
                    ScaleGestureRecognizer>(
                  () => ScaleGestureRecognizer(),
                  (ScaleGestureRecognizer instance) {
                    instance
                      ..onStart = (details) {
                        _log('ScaleStart');
                      }
                      ..onUpdate = (details) {
                        setState(() {
                          _scale = details.scale;
                        });
                        _log('ScaleUpdate: ${details.scale.toStringAsFixed(2)}');
                      }
                      ..onEnd = (details) {
                        _log('ScaleEnd');
                      };
                  },
                ),
              },
              behavior: HitTestBehavior.opaque,
              child: Container(
                color: Colors.blue.withOpacity(0.3),
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Text(
                        '在此区域进行手势操作',
                        style: TextStyle(fontSize: 18),
                      ),
                      const SizedBox(height: 16),
                      Transform.scale(
                        scale: _scale,
                        child: Container(
                          width: 100,
                          height: 100,
                          color: Colors.blue,
                          child: const Center(
                            child: Text(
                              '缩放',
                              style: TextStyle(color: Colors.white),
                            ),
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
          // 手势日志显示
          Expanded(
            flex: 1,
            child: Container(
              width: double.infinity,
              color: Colors.grey[200],
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        '手势日志:',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      TextButton(
                        onPressed: () => setState(() => _gestureLog = ''),
                        child: const Text('清除'),
                      ),
                    ],
                  ),
                  Expanded(
                    child: SingleChildScrollView(
                      child: Text(
                        _gestureLog.isEmpty ? '等待手势...' : _gestureLog,
                        style: const TextStyle(
                          fontFamily: 'monospace',
                          fontSize: 12,
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：`RawGestureDetector` 允许同时配置多个相同类型的手势识别器，这是 `GestureDetector` 无法做到的。例如，可以同时配置两个 `TapGestureRecognizer`，一个用于单击，一个用于右键点击。

---

## 第四章：手势识别器（GestureRecognizer）

手势识别器是 Flutter 手势系统的核心，负责识别特定的手势模式并将原始指针事件转换为高级手势事件。

### 4.1 GestureRecognizer 基类

```dart
abstract class GestureRecognizer extends GestureArenaMember
    with DiagnosticableTreeMixin, GestureRecognizerStateMixin {
  /// 创建手势识别器
  GestureRecognizer({
    this.debugOwner,
    this.supportedDevices,
    this.allowedButtonsFilter = _defaultButtonAcceptBehavior,
  });

  /// 调试所有者
  final Object? debugOwner;

  /// 支持的设备类型
  final Set<PointerDeviceKind>? supportedDevices;

  /// 允许的按钮过滤器
  final AllowedButtonsFilter allowedButtonsFilter;

  /// 添加指针到识别器
  void addPointer(PointerDownEvent event);

  /// 处理指针事件
  void handleEvent(PointerEvent event);

  /// 接受手势
  void acceptGesture(int pointer);

  /// 拒绝手势
  void rejectGesture(int pointer);

  /// 释放资源
  void dispose();
}
```

### 4.2 点击手势识别器（TapGestureRecognizer）

```dart
class TapGestureRecognizer extends PrimaryPointerGestureRecognizer {
  /// 创建点击手势识别器
  TapGestureRecognizer({
    super.debugOwner,
    super.supportedDevices,
    super.allowedButtonsFilter,
    this.onTapDown,
    this.onTapUp,
    this.onTap,
    this.onTapCancel,
  });

  /// 点击按下回调
  GestureTapDownCallback? onTapDown;

  /// 点击抬起回调
  GestureTapUpCallback? onTapUp;

  /// 点击完成回调
  GestureTapCallback? onTap;

  /// 点击取消回调
  GestureTapCancelCallback? onTapCancel;
}
```

### 4.3 拖动手势识别器（DragGestureRecognizer）

```dart
abstract class DragGestureRecognizer extends OneSequenceGestureRecognizer {
  /// 拖动开始行为
  DragStartBehavior dragStartBehavior;

  /// 拖动开始回调
  GestureDragDownCallback? onDown;

  /// 拖动开始回调
  GestureDragStartCallback? onStart;

  /// 拖动更新回调
  GestureDragUpdateCallback? onUpdate;

  /// 拖动结束回调
  GestureDragEndCallback? onEnd;

  /// 拖动取消回调
  GestureDragCancelCallback? onCancel;
}
```

### 4.4 缩放手势识别器（ScaleGestureRecognizer）

```dart
class ScaleGestureRecognizer extends OneSequenceGestureRecognizer {
  /// 创建缩放手势识别器
  ScaleGestureRecognizer({
    super.debugOwner,
    super.supportedDevices,
    super.allowedButtonsFilter,
    this.dragStartBehavior = DragStartBehavior.down,
    this.trackpadScrollCausesScale = false,
    this.trackpadScrollToScaleFactor = kDefaultTrackpadScrollToScaleFactor,
  });

  /// 拖动开始行为
  DragStartBehavior dragStartBehavior;

  /// 触控板滚动是否触发缩放
  bool trackpadScrollCausesScale;

  /// 触控板滚动到缩放的转换因子
  Offset trackpadScrollToScaleFactor;

  /// 缩放开始回调
  GestureScaleStartCallback? onStart;

  /// 缩放更新回调
  GestureScaleUpdateCallback? onUpdate;

  /// 缩放结束回调
  GestureScaleEndCallback? onEnd;
}
```

### 4.5 手势识别器类型速查表

| 识别器类                          | 识别手势     | 主要回调                                      |
| --------------------------------- | ------------ | --------------------------------------------- |
| `TapGestureRecognizer`            | 单击         | onTap, onTapDown, onTapUp                     |
| `DoubleTapGestureRecognizer`      | 双击         | onDoubleTap, onDoubleTapDown                  |
| `LongPressGestureRecognizer`      | 长按         | onLongPress, onLongPressStart                 |
| `VerticalDragGestureRecognizer`   | 垂直拖动     | onVerticalDragStart, onVerticalDragUpdate     |
| `HorizontalDragGestureRecognizer` | 水平拖动     | onHorizontalDragStart, onHorizontalDragUpdate |
| `PanGestureRecognizer`            | 任意方向拖动 | onPanStart, onPanUpdate, onPanEnd             |
| `ScaleGestureRecognizer`          | 双指缩放     | onScaleStart, onScaleUpdate, onScaleEnd       |
| `SerialTapGestureRecognizer`      | 连续点击     | onSerialTapDown, onSerialTapUp                |
| `TapAndPanGestureRecognizer`      | 点击+拖动    | onTapDown, onPanStart, onPanUpdate            |

### 4.6 完整示例代码：自定义手势识别器

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

/// 自定义三击手势识别器
class TripleTapGestureRecognizer extends GestureRecognizer {
  TripleTapGestureRecognizer({
    super.debugOwner,
    super.supportedDevices,
    super.allowedButtonsFilter,
    this.onTripleTap,
  });

  /// 三击回调
  GestureTapCallback? onTripleTap;

  int _tapCount = 0;
  DateTime? _firstTapTime;
  Timer? _resetTimer;

  static const _maxInterval = Duration(milliseconds: 300);
  static const _maxDuration = Duration(milliseconds: 500);

  @override
  void addPointer(PointerDownEvent event) {
    if (onTripleTap == null) {
      resolve(GestureDisposition.rejected);
      return;
    }

    final now = DateTime.now();

    // 检查是否在有效时间间隔内
    if (_firstTapTime != null &&
        now.difference(_firstTapTime!) > _maxDuration) {
      _reset();
    }

    if (_tapCount == 0) {
      _firstTapTime = now;
    }

    _tapCount++;

    // 取消之前的重置定时器
    _resetTimer?.cancel();

    if (_tapCount == 3) {
      // 三击成功
      resolve(GestureDisposition.accepted);
      onTripleTap?.call();
      _reset();
    } else {
      // 继续等待更多点击
      resolve(GestureDisposition.accepted);
      _resetTimer = Timer(_maxInterval, _reset);
    }

    startTrackingPointer(event.pointer, event.transform);
  }

  @override
  void handleEvent(PointerEvent event) {
    if (event is PointerUpEvent) {
      stopTrackingPointer(event.pointer);
    }
  }

  @override
  void acceptGesture(int pointer) {
    // 已接受
  }

  @override
  void rejectGesture(int pointer) {
    _reset();
  }

  void _reset() {
    _tapCount = 0;
    _firstTapTime = null;
    _resetTimer?.cancel();
    _resetTimer = null;
  }

  @override
  void dispose() {
    _resetTimer?.cancel();
    super.dispose();
  }

  @override
  String get debugDescription => 'triple tap';
}

/// 使用自定义手势识别器
class CustomGestureRecognizerDemo extends StatefulWidget {
  const CustomGestureRecognizerDemo({super.key});

  @override
  State<CustomGestureRecognizerDemo> createState() => _CustomGestureRecognizerDemoState();
}

class _CustomGestureRecognizerDemoState extends State<CustomGestureRecognizerDemo> {
  int _tripleTapCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('自定义手势识别器'),
      ),
      body: Center(
        child: RawGestureDetector(
          gestures: {
            TripleTapGestureRecognizer: GestureRecognizerFactoryWithHandlers<
                TripleTapGestureRecognizer>(
              () => TripleTapGestureRecognizer(),
              (TripleTapGestureRecognizer instance) {
                instance.onTripleTap = () {
                  setState(() {
                    _tripleTapCount++;
                  });
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text('三击成功! 总计: $_tripleTapCount'),
                      duration: const Duration(milliseconds: 500),
                    ),
                  );
                };
              },
            ),
          },
          child: Container(
            width: 300,
            height: 300,
            color: Colors.purple,
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Text(
                    '快速点击三次',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                    ),
                  ),
                  const SizedBox(height: 16),
                  Text(
                    '三击次数: $_tripleTapCount',
                    style: const TextStyle(
                      color: Colors.white70,
                      fontSize: 18,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

> **补充知识**：创建自定义手势识别器时，需要继承 `GestureRecognizer` 或其子类（如 `OneSequenceGestureRecognizer`），并实现 `addPointer`、`handleEvent`、`acceptGesture` 和 `rejectGesture` 方法。自定义识别器需要正确处理手势竞技场的竞争逻辑。

---

## 第五章：手势竞技场（GestureArena）

手势竞技场是 Flutter 手势系统的核心机制，用于协调多个手势识别器之间的竞争，确保同一触摸事件序列只被一个识别器处理。

### 5.1 竞技场工作原理

```
┌─────────────────────────────────────────────────────────────────┐
│                     手势竞技场流程                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 指针按下 (PointerDown)                                       │
│           │                                                     │
│           ▼                                                     │
│  2. 创建竞技场条目 (GestureArenaEntry)                            │
│           │                                                     │
│           ▼                                                     │
│  3. 多个识别器加入竞争                                            │
│     ┌─────────┐  ┌─────────┐  ┌─────────┐                      │
│     │ 识别器A │  │ 识别器B │  │ 识别器C │                      │
│     │ (Tap)   │  │ (Pan)   │  │ (Long)  │                      │
│     └────┬────┘  └────┬────┘  └────┬────┘                      │
│          │            │            │                            │
│          └────────────┼────────────┘                            │
│                       ▼                                         │
│  4. 竞技场管理器协调竞争                                          │
│     ┌─────────────────────────────┐                            │
│     │     GestureArenaManager     │                            │
│     │  ┌─────────────────────┐    │                            │
│     │  │ 等待所有识别器表态   │    │                            │
│     │  │ (accept/reject)     │    │                            │
│     │  └─────────────────────┘    │                            │
│     └─────────────────────────────┘                            │
│                       │                                         │
│           ┌───────────┼───────────┐                            │
│           ▼           ▼           ▼                            │
│  5. 结果: 接受      拒绝       竞争                              │
│     ┌───────┐    ┌───────┐   ┌───────────┐                    │
│     │A获胜  │    │B失败  │   │继续竞争   │                    │
│     └───────┘    └───────┘   └───────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 GestureArenaManager

```dart
class GestureArenaManager {
  /// 为指针打开竞技场
  GestureArenaEntry add(
    int pointer,
    GestureArenaMember member, {
    GestureDisposition disposition = GestureDisposition.pending,
  });

  /// 关闭指针的竞技场
  void close(int pointer);

  /// 释放指针的竞技场
  void sweep(int pointer);

  /// 释放所有竞技场
  void releaseAll();
}
```

### 5.3 GestureDisposition 枚举

```dart
enum GestureDisposition {
  /// 接受手势
  accepted,

  /// 拒绝手势
  rejected,

  /// 等待决定
  pending,
}
```

### 5.4 竞技场竞争示例

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class GestureArenaDemo extends StatefulWidget {
  const GestureArenaDemo({super.key});

  @override
  State<GestureArenaDemo> createState() => _GestureArenaDemoState();
}

class _GestureArenaDemoState extends State<GestureArenaDemo> {
  String _log = '';

  void _addLog(String message) {
    setState(() {
      _log = '$message\n$_log';
      if (_log.length > 800) {
        _log = _log.substring(0, 800);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('手势竞技场演示'),
      ),
      body: Column(
        children: [
          // 演示区域：点击和长按竞争
          Expanded(
            flex: 2,
            child: GestureDetector(
              onTap: () => _addLog('✅ Tap 获胜'),
              onLongPress: () => _addLog('✅ LongPress 获胜'),
              child: Container(
                width: double.infinity,
                color: Colors.blue.withOpacity(0.3),
                child: const Center(
                  child: Text(
                    '点击或长按\n(Tap vs LongPress 竞争)',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 18),
                  ),
                ),
              ),
            ),
          ),

          // 演示区域：水平和垂直拖动竞争
          Expanded(
            flex: 2,
            child: GestureDetector(
              onHorizontalDragUpdate: (details) {
                _addLog('✅ HorizontalDrag 获胜: ${details.delta.dx.toStringAsFixed(1)}');
              },
              onVerticalDragUpdate: (details) {
                _addLog('✅ VerticalDrag 获胜: ${details.delta.dy.toStringAsFixed(1)}');
              },
              child: Container(
                width: double.infinity,
                color: Colors.green.withOpacity(0.3),
                child: const Center(
                  child: Text(
                    '水平或垂直拖动\n(Horizontal vs Vertical 竞争)',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 18),
                  ),
                ),
              ),
            ),
          ),

          // 演示区域：父子手势竞争
          Expanded(
            flex: 2,
            child: GestureDetector(
              onTap: () => _addLog('👆 父容器 Tap'),
              child: Container(
                width: double.infinity,
                color: Colors.orange.withOpacity(0.3),
                child: Center(
                  child: GestureDetector(
                    onTap: () => _addLog('👶 子容器 Tap (获胜)'),
                    child: Container(
                      width: 150,
                      height: 150,
                      color: Colors.orange,
                      child: const Center(
                        child: Text(
                          '点击我\n(子容器优先)',
                          textAlign: TextAlign.center,
                          style: TextStyle(color: Colors.white),
                        ),
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ),

          // 日志显示
          Expanded(
            flex: 1,
            child: Container(
              width: double.infinity,
              color: Colors.grey[200],
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        '竞技场日志:',
                        style: TextStyle(fontWeight: FontWeight.bold),
                      ),
                      TextButton(
                        onPressed: () => setState(() => _log = ''),
                        child: const Text('清除'),
                      ),
                    ],
                  ),
                  Expanded(
                    child: SingleChildScrollView(
                      child: Text(
                        _log.isEmpty ? '等待手势...' : _log,
                        style: const TextStyle(
                          fontFamily: 'monospace',
                          fontSize: 11,
                        ),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：手势竞技场的竞争规则是：
>
> 1. 当指针按下时，所有感兴趣的识别器加入竞技场
> 2. 识别器可以主动接受（accept）或拒绝（reject）手势
> 3. 如果只有一个识别器接受，它获胜
> 4. 如果多个识别器接受，竞技场会选择最具体的那个（如 LongPress 比 Tap 更具体）
> 5. 子 Widget 的识别器通常优先于父 Widget

---

## 第六章：手势详情（Gesture Details）

手势详情类封装了手势事件的各种信息，如位置、速度、缩放比例等。

### 6.1 点击详情

```dart
class TapDownDetails {
  /// 全局坐标
  final Offset globalPosition;

  /// 本地坐标
  final Offset localPosition;

  /// 指针设备类型
  final PointerDeviceKind kind;

  /// 按钮
  final int buttons;
}

class TapUpDetails {
  /// 全局坐标
  final Offset globalPosition;

  /// 本地坐标
  final Offset localPosition;

  /// 指针设备类型
  final PointerDeviceKind kind;
}
```

### 6.2 拖动详情

```dart
class DragStartDetails {
  /// 全局坐标
  final Offset globalPosition;

  /// 本地坐标
  final Offset localPosition;

  /// 源时间戳
  final Duration? sourceTimeStamp;
}

class DragUpdateDetails {
  /// 全局坐标
  final Offset globalPosition;

  /// 本地坐标
  final Offset localPosition;

  /// 偏移量（从拖动开始）
  final Offset delta;

  /// 主方向偏移量
  final double primaryDelta;

  /// 源时间戳
  final Duration? sourceTimeStamp;
}

class DragEndDetails {
  /// 速度
  final Velocity velocity;

  /// 主方向速度
  final double? primaryVelocity;
}
```

### 6.3 缩放详情

```dart
class ScaleStartDetails {
  /// 焦点全局坐标
  final Offset focalPoint;

  /// 焦点本地坐标
  final Offset localFocalPoint;

  /// 指针数量
  final int pointerCount;
}

class ScaleUpdateDetails {
  /// 焦点全局坐标
  final Offset focalPoint;

  /// 焦点本地坐标
  final Offset localFocalPoint;

  /// 缩放比例
  final double scale;

  /// 水平缩放
  final double horizontalScale;

  /// 垂直缩放
  final double verticalScale;

  /// 旋转角度（弧度）
  final double rotation;

  /// 指针数量
  final int pointerCount;
}

class ScaleEndDetails {
  /// 速度
  final Velocity velocity;

  /// 指针数量
  final int pointerCount;
}
```

### 6.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class GestureDetailsDemo extends StatefulWidget {
  const GestureDetailsDemo({super.key});

  @override
  State<GestureDetailsDemo> createState() => _GestureDetailsDemoState();
}

class _GestureDetailsDemoState extends State<GestureDetailsDemo> {
  String _details = '';
  Offset _position = Offset.zero;
  double _scale = 1.0;
  double _rotation = 0.0;

  void _updateDetails(String details) {
    setState(() {
      _details = details;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('手势详情演示'),
      ),
      body: Column(
        children: [
          // 详情显示
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            color: Colors.grey[200],
            child: Text(
              _details.isEmpty ? '进行手势操作查看详情' : _details,
              style: const TextStyle(
                fontFamily: 'monospace',
                fontSize: 12,
              ),
            ),
          ),

          // 手势区域
          Expanded(
            child: GestureDetector(
              onTapDown: (details) {
                _updateDetails(
                  'TapDown:\n'
                  '  globalPosition: ${details.globalPosition}\n'
                  '  localPosition: ${details.localPosition}\n'
                  '  kind: ${details.kind}',
                );
              },
              onTapUp: (details) {
                _updateDetails(
                  'TapUp:\n'
                  '  globalPosition: ${details.globalPosition}\n'
                  '  localPosition: ${details.localPosition}',
                );
              },
              onPanStart: (details) {
                _updateDetails(
                  'PanStart:\n'
                  '  globalPosition: ${details.globalPosition}\n'
                  '  localPosition: ${details.localPosition}',
                );
              },
              onPanUpdate: (details) {
                setState(() {
                  _position += details.delta;
                });
                _updateDetails(
                  'PanUpdate:\n'
                  '  delta: ${details.delta}\n'
                  '  globalPosition: ${details.globalPosition}\n'
                  '  localPosition: ${details.localPosition}',
                );
              },
              onPanEnd: (details) {
                _updateDetails(
                  'PanEnd:\n'
                  '  velocity: ${details.velocity}\n'
                  '  pixelsPerSecond: ${details.velocity.pixelsPerSecond}',
                );
              },
              onScaleStart: (details) {
                _updateDetails(
                  'ScaleStart:\n'
                  '  focalPoint: ${details.focalPoint}\n'
                  '  localFocalPoint: ${details.localFocalPoint}\n'
                  '  pointerCount: ${details.pointerCount}',
                );
              },
              onScaleUpdate: (details) {
                setState(() {
                  _scale = details.scale;
                  _rotation = details.rotation;
                });
                _updateDetails(
                  'ScaleUpdate:\n'
                  '  scale: ${details.scale.toStringAsFixed(2)}\n'
                  '  horizontalScale: ${details.horizontalScale.toStringAsFixed(2)}\n'
                  '  verticalScale: ${details.verticalScale.toStringAsFixed(2)}\n'
                  '  rotation: ${(details.rotation * 180 / 3.14159).toStringAsFixed(1)}°\n'
                  '  pointerCount: ${details.pointerCount}',
                );
              },
              onScaleEnd: (details) {
                _updateDetails(
                  'ScaleEnd:\n'
                  '  velocity: ${details.velocity}\n'
                  '  pointerCount: ${details.pointerCount}',
                );
              },
              child: Container(
                width: double.infinity,
                color: Colors.blue.withOpacity(0.1),
                child: Center(
                  child: Transform.translate(
                    offset: _position,
                    child: Transform.scale(
                      scale: _scale,
                      child: Transform.rotate(
                        angle: _rotation,
                        child: Container(
                          width: 150,
                          height: 150,
                          color: Colors.blue,
                          child: const Center(
                            child: Text(
                              '拖动/缩放/旋转',
                              style: TextStyle(color: Colors.white),
                              textAlign: TextAlign.center,
                            ),
                          ),
                        ),
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 第七章：速度追踪（Velocity Tracking）

速度追踪用于计算手势的速度，常用于实现惯性滚动、甩动等效果。

### 7.1 Velocity 类

```dart
class Velocity {
  /// 零速度常量
  static const Velocity zero = Velocity(pixelsPerSecond: Offset.zero);

  /// 创建速度
  const Velocity({
    required this.pixelsPerSecond,
  });

  /// 每秒像素偏移
  final Offset pixelsPerSecond;

  /// 获取速度大小
  double get magnitude => pixelsPerSecond.distance;

  /// 获取速度方向（弧度）
  double get direction => pixelsPerSecond.direction;
}
```

### 7.2 VelocityTracker 类

```dart
class VelocityTracker {
  /// 创建速度追踪器
  VelocityTracker.withKind(this.kind);

  /// 指针设备类型
  final PointerDeviceKind kind;

  /// 添加位置数据点
  void addPosition(Duration time, Offset position);

  /// 获取速度估计
  VelocityEstimate getVelocityEstimate();

  /// 重置追踪器
  void reset();
}
```

### 7.3 VelocityEstimate 类

```dart
class VelocityEstimate {
  /// 像素每秒
  final Offset pixelsPerSecond;

  /// 偏移量
  final Offset offset;

  /// 持续时间
  final Duration duration;

  /// 置信度
  final double confidence;
}
```

### 7.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class VelocityTrackingDemo extends StatefulWidget {
  const VelocityTrackingDemo({super.key});

  @override
  State<VelocityTrackingDemo> createState() => _VelocityTrackingDemoState();
}

class _VelocityTrackingDemoState extends State<VelocityTrackingDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Offset> _animation;
  Offset _position = Offset.zero;
  Velocity _velocity = Velocity.zero;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 500),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _onPanEnd(DragEndDetails details) {
    _velocity = details.velocity;

    // 计算惯性动画
    final velocityPixels = _velocity.pixelsPerSecond;
    final distance = velocityPixels.distance * 0.5; // 减速距离
    final direction = velocityPixels.direction;

    final endPosition = Offset(
      _position.dx + distance * cos(direction),
      _position.dy + distance * sin(direction),
    );

    _animation = Tween<Offset>(
      begin: _position,
      end: endPosition,
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.decelerate,
    ));

    _animation.addListener(() {
      setState(() {
        _position = _animation.value;
      });
    });

    _controller.forward(from: 0);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('速度追踪演示'),
      ),
      body: Column(
        children: [
          // 速度显示
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            color: Colors.grey[200],
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('速度: ${_velocity.pixelsPerSecond}'),
                Text('大小: ${_velocity.magnitude.toStringAsFixed(2)} px/s'),
                Text('方向: ${(_velocity.direction * 180 / 3.14159).toStringAsFixed(1)}°'),
              ],
            ),
          ),

          // 手势区域
          Expanded(
            child: GestureDetector(
              onPanUpdate: (details) {
                if (!_controller.isAnimating) {
                  setState(() {
                    _position += details.delta;
                  });
                }
              },
              onPanEnd: _onPanEnd,
              child: Container(
                width: double.infinity,
                color: Colors.blue.withOpacity(0.1),
                child: Stack(
                  children: [
                    Positioned(
                      left: _position.dx + MediaQuery.of(context).size.width / 2 - 50,
                      top: _position.dy + MediaQuery.of(context).size.height / 3 - 50,
                      child: Container(
                        width: 100,
                        height: 100,
                        decoration: BoxDecoration(
                          color: Colors.blue,
                          borderRadius: BorderRadius.circular(50),
                        ),
                        child: const Center(
                          child: Text(
                            '甩动我',
                            style: TextStyle(color: Colors.white),
                          ),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 第八章：指针事件（Pointer Events）

指针事件是 Flutter 手势系统的最底层，直接对应于硬件触摸事件。

### 8.1 指针事件类型

| 事件类                | 说明     | 触发时机           |
| --------------------- | -------- | ------------------ |
| `PointerDownEvent`    | 指针按下 | 手指接触屏幕       |
| `PointerMoveEvent`    | 指针移动 | 手指在屏幕上移动   |
| `PointerUpEvent`      | 指针抬起 | 手指离开屏幕       |
| `PointerCancelEvent`  | 指针取消 | 手势被系统取消     |
| `PointerAddedEvent`   | 指针添加 | 新指针设备添加     |
| `PointerRemovedEvent` | 指针移除 | 指针设备移除       |
| `PointerHoverEvent`   | 指针悬停 | 鼠标悬停（无接触） |
| `PointerSignalEvent`  | 指针信号 | 滚轮等信号事件     |

### 8.2 PointerEvent 基类

```dart
abstract class PointerEvent {
  /// 指针 ID
  final int pointer;

  /// 时间戳
  final Duration timeStamp;

  /// 全局坐标
  final Offset position;

  /// 本地坐标
  final Offset localPosition;

  /// 设备类型
  final PointerDeviceKind kind;

  /// 按钮
  final int buttons;

  /// 压力（0.0 - 1.0）
  final double pressure;

  /// 压力最小值
  final double pressureMin;

  /// 压力最大值
  final double pressureMax;

  /// 接触面积（水平）
  final double size;

  /// 接触面积（垂直）
  final double? orientation;

  /// 倾斜（水平）
  final double? tilt;
}
```

### 8.3 Listener Widget

`Listener` 是监听原始指针事件的 Widget。

```dart
const Listener({
  super.key,
  this.onPointerDown,
  this.onPointerMove,
  this.onPointerUp,
  this.onPointerHover,
  this.onPointerCancel,
  this.onPointerSignal,
  this.behavior = HitTestBehavior.deferToChild,
  super.child,
});
```

### 8.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class PointerEventsDemo extends StatefulWidget {
  const PointerEventsDemo({super.key});

  @override
  State<PointerEventsDemo> createState() => _PointerEventsDemoState();
}

class _PointerEventsDemoState extends State<PointerEventsDemo> {
  final List<String> _events = [];
  int _activePointers = 0;

  void _addEvent(String event) {
    setState(() {
      _events.insert(0, '[${DateTime.now().millisecond}] $event');
      if (_events.length > 50) {
        _events.removeLast();
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('指针事件演示'),
      ),
      body: Column(
        children: [
          // 状态显示
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(16),
            color: Colors.grey[200],
            child: Text('活动指针数: $_activePointers'),
          ),

          // 指针事件监听区域
          Expanded(
            flex: 2,
            child: Listener(
              onPointerDown: (event) {
                _activePointers++;
                _addEvent(
                  'PointerDown: id=${event.pointer}, '
                  'pos=${event.position}, '
                  'kind=${event.kind}, '
                  'pressure=${event.pressure.toStringAsFixed(2)}',
                );
              },
              onPointerMove: (event) {
                _addEvent(
                  'PointerMove: id=${event.pointer}, '
                  'pos=${event.position}, '
                  'delta=${event.delta}',
                );
              },
              onPointerUp: (event) {
                _activePointers--;
                _addEvent('PointerUp: id=${event.pointer}');
              },
              onPointerCancel: (event) {
                _activePointers--;
                _addEvent('PointerCancel: id=${event.pointer}');
              },
              onPointerHover: (event) {
                _addEvent(
                  'PointerHover: pos=${event.position}, '
                  'kind=${event.kind}',
                );
              },
              onPointerSignal: (event) {
                if (event is PointerScrollEvent) {
                  _addEvent(
                    'PointerScroll: delta=${event.scrollDelta}',
                  );
                }
              },
              behavior: HitTestBehavior.opaque,
              child: Container(
                width: double.infinity,
                color: Colors.blue.withOpacity(0.3),
                child: const Center(
                  child: Text(
                    '在此区域进行触摸/鼠标操作\n'
                    '支持多点触控、滚轮',
                    textAlign: TextAlign.center,
                    style: TextStyle(fontSize: 16),
                  ),
                ),
              ),
            ),
          ),

          // 事件日志
          Expanded(
            flex: 1,
            child: Container(
              width: double.infinity,
              color: Colors.black87,
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text(
                        '事件日志:',
                        style: TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      TextButton(
                        onPressed: () => setState(() => _events.clear()),
                        child: const Text('清除', style: TextStyle(color: Colors.white)),
                      ),
                    ],
                  ),
                  Expanded(
                    child: ListView.builder(
                      itemCount: _events.length,
                      itemBuilder: (context, index) {
                        return Text(
                          _events[index],
                          style: const TextStyle(
                            color: Colors.green,
                            fontFamily: 'monospace',
                            fontSize: 10,
                          ),
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：指针事件是 Flutter 手势系统的最底层，通常不需要直接使用。但在需要处理特殊手势或获取原始触摸数据时，可以使用 `Listener` Widget 监听指针事件。

---

## 第九章：多点触控（Multi-Touch）

Flutter 支持多点触控手势，可以同时追踪多个手指的触摸。

### 9.1 多点触控识别器

| 识别器类                               | 说明               |
| -------------------------------------- | ------------------ |
| `MultiDragGestureRecognizer`           | 多指拖动基类       |
| `ImmediateMultiDragGestureRecognizer`  | 立即响应的多指拖动 |
| `HorizontalMultiDragGestureRecognizer` | 水平多指拖动       |
| `VerticalMultiDragGestureRecognizer`   | 垂直多指拖动       |
| `DelayedMultiDragGestureRecognizer`    | 延迟响应的多指拖动 |
| `MultiTapGestureRecognizer`            | 多指点击           |

### 9.2 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

class MultiTouchDemo extends StatefulWidget {
  const MultiTouchDemo({super.key});

  @override
  State<MultiTouchDemo> createState() => _MultiTouchDemoState();
}

class _MultiTouchDemoState extends State<MultiTouchDemo> {
  final Map<int, Offset> _pointers = {};
  final Map<int, Color> _pointerColors = {};

  Color _getColorForPointer(int pointer) {
    if (!_pointerColors.containsKey(pointer)) {
      _pointerColors[pointer] = Colors.primaries[
          pointer % Colors.primaries.length];
    }
    return _pointerColors[pointer]!;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('多点触控演示'),
      ),
      body: Listener(
        onPointerDown: (event) {
          setState(() {
            _pointers[event.pointer] = event.localPosition;
          });
        },
        onPointerMove: (event) {
          setState(() {
            _pointers[event.pointer] = event.localPosition;
          });
        },
        onPointerUp: (event) {
          setState(() {
            _pointers.remove(event.pointer);
          });
        },
        onPointerCancel: (event) {
          setState(() {
            _pointers.remove(event.pointer);
          });
        },
        behavior: HitTestBehavior.opaque,
        child: CustomPaint(
          size: Size.infinite,
          painter: MultiTouchPainter(
            pointers: _pointers,
            getColor: _getColorForPointer,
          ),
        ),
      ),
    );
  }
}

class MultiTouchPainter extends CustomPainter {
  final Map<int, Offset> pointers;
  final Color Function(int) getColor;

  MultiTouchPainter({
    required this.pointers,
    required this.getColor,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // 绘制背景
    canvas.drawRect(
      Offset.zero & size,
      Paint()..color = Colors.black12,
    );

    // 绘制每个触摸点
    pointers.forEach((pointer, position) {
      final color = getColor(pointer);

      // 绘制外圈
      canvas.drawCircle(
        position,
        50,
        Paint()
          ..color = color.withOpacity(0.3)
          ..style = PaintingStyle.fill,
      );

      // 绘制内圈
      canvas.drawCircle(
        position,
        30,
        Paint()
          ..color = color
          ..style = PaintingStyle.fill,
      );

      // 绘制指针ID
      final textPainter = TextPainter(
        text: TextSpan(
          text: '$pointer',
          style: const TextStyle(
            color: Colors.white,
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        textDirection: TextDirection.ltr,
      );
      textPainter.layout();
      textPainter.paint(
        canvas,
        position - Offset(textPainter.width / 2, textPainter.height / 2),
      );
    });

    // 绘制连接线（如果有多点）
    if (pointers.length >= 2) {
      final positions = pointers.values.toList();
      final path = Path();
      path.moveTo(positions.first.dx, positions.first.dy);
      for (int i = 1; i < positions.length; i++) {
        path.lineTo(positions[i].dx, positions[i].dy);
      }
      if (pointers.length > 2) {
        path.close();
      }

      canvas.drawPath(
        path,
        Paint()
          ..color = Colors.purple.withOpacity(0.3)
          ..strokeWidth = 2
          ..style = PaintingStyle.stroke,
      );
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

---

## 第十章：常用常量

### 10.1 手势常量

| 常量                | 类型     | 默认值 | 说明         |
| ------------------- | -------- | ------ | ------------ |
| `kPressTimeout`     | Duration | 100ms  | 按下超时时间 |
| `kLongPressTimeout` | Duration | 500ms  | 长按超时时间 |
| `kDoubleTapTimeout` | Duration | 300ms  | 双击超时时间 |
| `kDoubleTapSlop`    | double   | 100.0  | 双击容差距离 |
| `kTouchSlop`        | double   | 18.0   | 触摸滑动阈值 |
| `kPanSlop`          | double   | 36.0   | 平移滑动阈值 |
| `kScaleSlop`        | double   | 18.0   | 缩放滑动阈值 |
| `kMaxFlingVelocity` | double   | 8000.0 | 最大甩动速度 |
| `kMinFlingVelocity` | double   | 50.0   | 最小甩动速度 |

### 10.2 触控板常量

| 常量                                  | 类型   | 默认值           | 说明                     |
| ------------------------------------- | ------ | ---------------- | ------------------------ |
| `kDefaultTrackpadScrollToScaleFactor` | Offset | Offset(200, 200) | 触控板滚动到缩放转换因子 |

---

## 附录 A：API 速查表

### A.1 GestureDetector 回调类型

| 回调类型                        | 参数                  | 说明     |
| ------------------------------- | --------------------- | -------- |
| `GestureTapCallback`            | void                  | 点击完成 |
| `GestureTapDownCallback`        | TapDownDetails        | 点击按下 |
| `GestureTapUpCallback`          | TapUpDetails          | 点击抬起 |
| `GestureTapCancelCallback`      | void                  | 点击取消 |
| `GestureDragStartCallback`      | DragStartDetails      | 拖动开始 |
| `GestureDragUpdateCallback`     | DragUpdateDetails     | 拖动更新 |
| `GestureDragEndCallback`        | DragEndDetails        | 拖动结束 |
| `GestureScaleStartCallback`     | ScaleStartDetails     | 缩放开始 |
| `GestureScaleUpdateCallback`    | ScaleUpdateDetails    | 缩放更新 |
| `GestureScaleEndCallback`       | ScaleEndDetails       | 缩放结束 |
| `GestureLongPressCallback`      | void                  | 长按     |
| `GestureLongPressStartCallback` | LongPressStartDetails | 长按开始 |
| `GestureLongPressEndCallback`   | LongPressEndDetails   | 长按结束 |

### A.2 手势识别器速查

| 识别器                            | 适用场景     | 竞争优先级 |
| --------------------------------- | ------------ | ---------- |
| `TapGestureRecognizer`            | 单击         | 低         |
| `DoubleTapGestureRecognizer`      | 双击         | 中         |
| `LongPressGestureRecognizer`      | 长按         | 高         |
| `VerticalDragGestureRecognizer`   | 垂直拖动     | 中         |
| `HorizontalDragGestureRecognizer` | 水平拖动     | 中         |
| `PanGestureRecognizer`            | 任意方向拖动 | 中         |
| `ScaleGestureRecognizer`          | 双指缩放     | 高         |

### A.3 HitTestBehavior 选择指南

| 行为           | 使用场景                        |
| -------------- | ------------------------------- |
| `deferToChild` | 默认行为，将事件传递给子 Widget |
| `opaque`       | 需要阻止事件传递给下方 Widget   |
| `translucent`  | 需要接收事件同时允许事件穿透    |

---

## 附录 B：常见问题解答

### Q1: GestureDetector 的 onPan 和 onVerticalDrag/onHorizontalDrag 能同时使用吗？

A: 不能。`onPan` 和 `onVerticalDrag`/`onHorizontalDrag` 是互斥的，因为它们会竞争同一手势事件。如果需要检测任意方向拖动，使用 `onPan`；如果需要限制特定方向，使用对应的拖动回调。

### Q2: 如何让父容器和子容器都能响应点击事件？

A: 使用 `HitTestBehavior.translucent` 或自定义手势识别器：

```dart
GestureDetector(
  behavior: HitTestBehavior.translucent,
  onTap: () => print('父容器'),
  child: GestureDetector(
    onTap: () => print('子容器'),
    child: Container(),
  ),
)
```

### Q3: 如何检测三击手势？

A: 创建自定义手势识别器（见第四章示例）或使用 `SerialTapGestureRecognizer`：

```dart
RawGestureDetector(
  gestures: {
    SerialTapGestureRecognizer: GestureRecognizerFactoryWithHandlers<
        SerialTapGestureRecognizer>(
      () => SerialTapGestureRecognizer(),
      (instance) {
        instance.onSerialTapUp = (details) {
          if (details.count == 3) {
            print('三击!');
          }
        };
      },
    ),
  },
  child: Container(),
)
```

### Q4: 如何区分手指和触控笔输入？

A: 通过 `PointerEvent.kind` 或 `TapDownDetails.kind`：

```dart
GestureDetector(
  onTapDown: (details) {
    if (details.kind == PointerDeviceKind.stylus) {
      print('触控笔点击');
    } else if (details.kind == PointerDeviceKind.touch) {
      print('手指点击');
    }
  },
  child: Container(),
)
```

### Q5: 如何实现惯性滚动效果？

A: 使用 `VelocityTracker` 和动画：

```dart
void _onPanEnd(DragEndDetails details) {
  final velocity = details.velocity.pixelsPerSecond;
  final animation = Tween<Offset>(
    begin: _position,
    end: _position + velocity * 0.5,
  ).animate(CurvedAnimation(
    parent: _controller,
    curve: Curves.decelerate,
  ));
  // ...
}
```

---

**文档版本**: 1.0  
**最后更新**: 2024年  
**Flutter 版本**: 3.41.0

---

> **补充知识**：`flutter/gestures.dart` 是 Flutter 交互系统的核心库。理解手势竞技场的工作原理对于处理复杂的手势交互至关重要。建议在实际项目中多尝试不同的手势组合，以深入理解它们之间的竞争关系。
