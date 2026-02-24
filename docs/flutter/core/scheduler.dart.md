# Flutter Scheduler Dart 完整参考手册

## package:flutter/scheduler.dart

> **版本**: Flutter 3.41.0  
> **适用范围**: Flutter 应用帧调度、动画同步、性能监控  
> **官方文档**: [api.flutter.dev/flutter/scheduler/scheduler-library.html](https://api.flutter.dev/flutter/scheduler/scheduler-library.html)

---

## 第一章：库概述

`flutter/scheduler.dart` 是 Flutter 框架的核心调度库，负责管理应用的帧生命周期、协调动画同步、处理 VSync 信号以及提供性能监控能力。这个库是连接 Flutter 框架与底层引擎的桥梁，对于理解 Flutter 的渲染机制和实现高性能动画至关重要。

### 1.1 库的主要组成部分

| 类别 | 说明 | 主要类/方法 |
|------|------|-------------|
| **调度绑定** | 核心调度管理 | SchedulerBinding |
| **动画运行器** | 帧同步动画驱动 | Ticker、TickerProvider |
| **帧回调** | 帧生命周期回调 | FrameCallback、Transient/Persistent/PostFrame |
| **调度阶段** | 帧处理阶段管理 | SchedulerPhase |
| **性能监控** | 帧性能指标采集 | FrameTiming、addTimingsCallback |
| **生命周期** | 应用状态管理 | AppLifecycleState、LifecycleMessage |
| **任务优先级** | 任务调度优先级 | Priority、TaskPriority |

### 1.2 导入方式

```dart
import 'package:flutter/scheduler.dart';
```

### 1.3 核心架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    Flutter Engine                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   VSync     │───▶│  schedule   │───▶│  onBegin    │     │
│  │   Signal    │    │   Frame     │    │   Frame     │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                 SchedulerBinding                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │ handleBeginFrame│  │ handleDrawFrame │  │   Phase     │ │
│  │                 │  │                 │  │  Control    │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │TransientCallback│  │PersistentCallback│  │PostFrame    │ │
│  │   (Animation)   │  │  (Build/Layout)  │  │  Callback   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Ticker                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │    start    │───▶│ scheduleTick│───▶│    _tick    │     │
│  │             │    │             │    │  (callback) │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

> **补充知识**：`SchedulerBinding` 是 Flutter 框架中唯一一个直接与引擎通信的绑定类，它通过 `PlatformDispatcher` 接收 VSync 信号并协调整个帧的处理流程。

---

## 第二章：SchedulerBinding

`SchedulerBinding` 是 Flutter 调度系统的核心，它是一个 mixin，负责管理帧的生命周期、注册和处理各种类型的帧回调、以及协调应用的渲染流程。

### 2.1 类定义与继承关系

```dart
mixin SchedulerBinding on BindingBase implements ServicesBinding {
  // 核心实现...
}
```

### 2.2 核心属性详解

| 属性名 | 类型 | 说明 |
|--------|------|------|
| `instance` | SchedulerBinding | 单例实例，全局访问点 |
| `schedulerPhase` | SchedulerPhase | 当前调度阶段 |
| `currentFrameTimeStamp` | Duration? | 当前帧的时间戳 |
| `currentSystemFrameTimeStamp` | Duration | 系统帧时间戳（原始值） |
| `hasScheduledFrame` | bool | 是否已调度帧 |
| `framesEnabled` | bool | 帧调度是否启用 |
| `lifecycleState` | AppLifecycleState? | 应用生命周期状态 |

### 2.3 帧回调管理

#### 2.3.1 Transient Frame Callbacks（瞬时帧回调）

瞬时帧回调在每一帧开始时执行一次，主要用于动画更新。

```dart
/// 调度瞬时帧回调
/// 返回回调ID，可用于取消回调
int scheduleFrameCallback(
  FrameCallback callback, {
  bool rescheduling = false,
});

/// 取消指定ID的帧回调
void cancelFrameCallbackWithId(int id);
```

#### 2.3.2 Persistent Frame Callbacks（持久帧回调）

持久帧回调在每一帧的绘制阶段执行，用于布局和渲染。

```dart
/// 添加持久帧回调
void addPersistentFrameCallback(FrameCallback callback);
```

#### 2.3.3 Post-Frame Callbacks（帧后回调）

帧后回调在当前帧完全渲染完成后执行，只执行一次。

```dart
/// 添加帧后回调
void addPostFrameCallback(FrameCallback callback);
```

### 2.4 帧生命周期方法

| 方法名 | 说明 | 调用时机 |
|--------|------|----------|
| `scheduleFrame()` | 请求调度新帧 | 需要更新UI时 |
| `ensureVisualUpdate()` | 确保视觉更新 | 需要重绘时 |
| `handleBeginFrame()` | 处理帧开始 | VSync信号到达时 |
| `handleDrawFrame()` | 处理帧绘制 | BeginFrame完成后 |
| `scheduleWarmUpFrame()` | 调度预热帧 | 应用启动时 |
| `handleAppLifecycleStateChanged()` | 处理生命周期变化 | 应用状态变化时 |

### 2.5 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  // 在应用启动前注册帧回调
  SchedulerBinding.instance.scheduleFrameCallback((timeStamp) {
    debugPrint('第一帧回调: $timeStamp');
  });
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'SchedulerBinding Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const SchedulerDemoPage(),
    );
  }
}

class SchedulerDemoPage extends StatefulWidget {
  const SchedulerDemoPage({super.key});

  @override
  State<SchedulerDemoPage> createState() => _SchedulerDemoPageState();
}

class _SchedulerDemoPageState extends State<SchedulerDemoPage> {
  String _phase = 'idle';
  int _frameCount = 0;
  Duration? _lastFrameTime;
  
  @override
  void initState() {
    super.initState();
    
    // 注册持久帧回调 - 每帧都会执行
    SchedulerBinding.instance.addPersistentFrameCallback((timeStamp) {
      setState(() {
        _frameCount++;
        _lastFrameTime = timeStamp;
        _phase = SchedulerBinding.instance.schedulerPhase.toString();
      });
    });
    
    // 注册帧后回调 - 只执行一次
    SchedulerBinding.instance.addPostFrameCallback((timeStamp) {
      debugPrint('首帧渲染完成: $timeStamp');
      
      // 显示启动对话框
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('欢迎'),
          content: const Text('应用已启动完成！'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('确定'),
            ),
          ],
        ),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('SchedulerBinding 演示'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 调度阶段显示
            _buildInfoCard(
              '当前调度阶段',
              _phase,
              Icons.timer,
            ),
            const SizedBox(height: 16),
            
            // 帧计数
            _buildInfoCard(
              '帧计数',
              '$_frameCount 帧',
              Icons.countertops,
            ),
            const SizedBox(height: 16),
            
            // 最后一帧时间
            _buildInfoCard(
              '最后一帧时间戳',
              _lastFrameTime?.toString() ?? '无',
              Icons.access_time,
            ),
            const SizedBox(height: 16),
            
            // 生命周期状态
            _buildInfoCard(
              '应用生命周期',
              SchedulerBinding.instance.lifecycleState.toString(),
              Icons.lifecycle,
            ),
            const SizedBox(height: 32),
            
            // 操作按钮
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () {
                    // 手动调度一帧
                    SchedulerBinding.instance.scheduleFrame();
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(content: Text('已手动调度一帧')),
                    );
                  },
                  child: const Text('调度帧'),
                ),
                ElevatedButton(
                  onPressed: () {
                    // 注册一次性帧回调
                    final callbackId = SchedulerBinding.instance.scheduleFrameCallback(
                      (timeStamp) {
                        debugPrint('瞬时回调执行: $timeStamp');
                      },
                    );
                    debugPrint('注册回调 ID: $callbackId');
                  },
                  child: const Text('注册瞬时回调'),
                ),
                ElevatedButton(
                  onPressed: () {
                    // 添加帧后回调
                    SchedulerBinding.instance.addPostFrameCallback((timeStamp) {
                      ScaffoldMessenger.of(context).showSnackBar(
                        SnackBar(content: Text('帧后回调执行: $timeStamp')),
                      );
                    });
                  },
                  child: const Text('添加帧后回调'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildInfoCard(String title, String value, IconData icon) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Icon(icon, size: 32, color: Theme.of(context).colorScheme.primary),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: TextStyle(
                      fontSize: 12,
                      color: Colors.grey[600],
                    ),
                  ),
                  const SizedBox(height: 4),
                  Text(
                    value,
                    style: const TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
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
```

> **补充知识**：`SchedulerBinding.instance` 是一个全局单例，在 Flutter 应用启动时自动初始化。它通过 `BindingBase` 的初始化机制与引擎建立连接。

---

## 第三章：SchedulerPhase 枚举

`SchedulerPhase` 枚举定义了调度器处理帧的各个阶段，用于跟踪当前帧的处理状态。

### 3.1 枚举定义

```dart
enum SchedulerPhase {
  /// 空闲状态，没有帧正在处理
  idle,
  
  /// 执行瞬时回调（如动画）
  transientCallbacks,
  
  /// 执行微任务（Future 完成回调等）
  midFrameMicrotasks,
  
  /// 执行持久回调（构建、布局、绘制）
  persistentCallbacks,
  
  /// 执行帧后回调
  postFrameCallbacks,
}
```

### 3.2 阶段流转图

```
┌─────────┐     scheduleFrame()      ┌─────────────────────┐
│  idle   │ ───────────────────────▶ │ transientCallbacks  │
└─────────┘                          └─────────────────────┘
                                              │
                                              │ 所有瞬时回调执行完毕
                                              ▼
                                     ┌─────────────────────┐
                                     │ midFrameMicrotasks  │
                                     └─────────────────────┘
                                              │
                                              │ 微任务执行完毕
                                              ▼
                                     ┌─────────────────────┐
                                     │ persistentCallbacks │
                                     └─────────────────────┘
                                              │
                                              │ 构建/布局/绘制完成
                                              ▼
                                     ┌─────────────────────┐
                                     │ postFrameCallbacks  │
                                     └─────────────────────┘
                                              │
                                              │ 帧后回调执行完毕
                                              ▼
                                       ┌─────────┐
                                       │  idle   │
                                       └─────────┘
```

### 3.3 使用场景

```dart
// 检查当前调度阶段
void checkSchedulerPhase() {
  final phase = SchedulerBinding.instance.schedulerPhase;
  
  switch (phase) {
    case SchedulerPhase.idle:
      debugPrint('调度器空闲');
      break;
    case SchedulerPhase.transientCallbacks:
      debugPrint('正在执行动画回调');
      break;
    case SchedulerPhase.midFrameMicrotasks:
      debugPrint('正在执行微任务');
      break;
    case SchedulerPhase.persistentCallbacks:
      debugPrint('正在构建/布局/绘制');
      break;
    case SchedulerPhase.postFrameCallbacks:
      debugPrint('正在执行帧后回调');
      break;
  }
}

// 在特定阶段执行操作
void doSomethingInRightPhase() {
  final binding = SchedulerBinding.instance;
  
  if (binding.schedulerPhase == SchedulerPhase.persistentCallbacks) {
    // 如果在构建阶段，延迟到帧后执行
    binding.addPostFrameCallback((_) {
      // 执行操作
    });
  } else {
    // 直接执行
    // ...
  }
}
```

---

## 第四章：Ticker

`Ticker` 是 Flutter 动画系统的核心组件，它通过调度器的帧回调机制，在每一帧触发回调，从而驱动动画的更新。

### 4.1 类定义

```dart
class Ticker {
  /// 创建 Ticker
  /// [onTick] - 每帧触发的回调函数
  /// [debugLabel] - 调试标签
  Ticker(
    TickerCallback onTick, {
    String? debugLabel,
  });
}
```

### 4.2 核心属性详解

| 属性名 | 类型 | 说明 |
|--------|------|------|
| `isActive` | bool | Ticker 是否正在运行（只读） |
| `isTicking` | bool | 是否已调度下一帧回调（只读） |
| `scheduled` | bool | 是否已安排帧回调（只读） |
| `muted` | bool | 是否静音（可读写） |
| `debugLabel` | String? | 调试标签 |
| `shouldScheduleTick` | bool | 是否应该调度下一帧（只读） |

### 4.3 核心方法

| 方法名 | 返回类型 | 说明 |
|--------|----------|------|
| `start()` | TickerFuture | 启动 Ticker |
| `stop()` | void | 停止 Ticker |
| `scheduleTick()` | void | 调度下一帧回调 |
| `absorbTicker()` | void | 吸收另一个 Ticker 的时间 |
| `dispose()` | void | 释放资源 |

### 4.4 TickerFuture

`TickerFuture` 是 `start()` 方法返回的 Future，用于监听 Ticker 的完成状态。

```dart
class TickerFuture implements Future<void> {
  /// Ticker 完成时调用
  TickerFuture whenCompleteOrCancel(VoidCallback callback);
  
  /// 取消 Ticker
  void cancel();
}
```

### 4.5 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Ticker Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const TickerDemoPage(),
    );
  }
}

class TickerDemoPage extends StatefulWidget {
  const TickerDemoPage({super.key});

  @override
  State<TickerDemoPage> createState() => _TickerDemoPageState();
}

class _TickerDemoPageState extends State<TickerDemoPage> {
  Ticker? _ticker;
  double _progress = 0.0;
  int _tickCount = 0;
  bool _isRunning = false;
  Duration _elapsed = Duration.zero;
  
  @override
  void initState() {
    super.initState();
    _createTicker();
  }
  
  void _createTicker() {
    _ticker = Ticker((elapsed) {
      setState(() {
        _elapsed = elapsed;
        _tickCount++;
        // 模拟动画进度（2秒一个周期）
        _progress = (elapsed.inMilliseconds % 2000) / 2000;
      });
    }, debugLabel: 'Demo Ticker');
  }

  @override
  void dispose() {
    _ticker?.dispose();
    super.dispose();
  }

  void _startTicker() {
    if (_ticker == null || !_ticker!.isActive) {
      _ticker?.start();
      setState(() {
        _isRunning = true;
      });
    }
  }

  void _stopTicker() {
    _ticker?.stop();
    setState(() {
      _isRunning = false;
    });
  }

  void _muteTicker() {
    if (_ticker != null) {
      _ticker!.muted = !_ticker!.muted;
      setState(() {});
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Ticker 演示'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          children: [
            // 动画展示
            Container(
              width: double.infinity,
              height: 200,
              decoration: BoxDecoration(
                color: Colors.grey[200],
                borderRadius: BorderRadius.circular(12),
              ),
              child: Center(
                child: AnimatedBuilder(
                  animation: AlwaysStoppedAnimation(_progress),
                  builder: (context, child) {
                    return Transform.rotate(
                      angle: _progress * 2 * 3.14159,
                      child: Container(
                        width: 100,
                        height: 100,
                        decoration: BoxDecoration(
                          gradient: LinearGradient(
                            colors: [
                              Colors.blue,
                              Colors.purple,
                            ],
                            transform: GradientRotation(_progress * 2 * 3.14159),
                          ),
                          borderRadius: BorderRadius.circular(12),
                        ),
                        child: const Center(
                          child: Icon(
                            Icons.animation,
                            color: Colors.white,
                            size: 48,
                          ),
                        ),
                      ),
                    );
                  },
                ),
              ),
            ),
            const SizedBox(height: 24),
            
            // 状态信息
            _buildInfoCard('运行状态', _isRunning ? '运行中' : '已停止'),
            _buildInfoCard('静音状态', _ticker?.muted == true ? '已静音' : '未静音'),
            _buildInfoCard('已运行时间', '${_elapsed.inMilliseconds}ms'),
            _buildInfoCard('帧回调次数', '$_tickCount'),
            _buildInfoCard('当前进度', '${(_progress * 100).toStringAsFixed(1)}%'),
            const SizedBox(height: 24),
            
            // 控制按钮
            Wrap(
              spacing: 8,
              runSpacing: 8,
              alignment: WrapAlignment.center,
              children: [
                ElevatedButton.icon(
                  onPressed: _isRunning ? null : _startTicker,
                  icon: const Icon(Icons.play_arrow),
                  label: const Text('启动'),
                ),
                ElevatedButton.icon(
                  onPressed: _isRunning ? _stopTicker : null,
                  icon: const Icon(Icons.stop),
                  label: const Text('停止'),
                ),
                ElevatedButton.icon(
                  onPressed: _muteTicker,
                  icon: Icon(_ticker?.muted == true ? Icons.volume_off : Icons.volume_up),
                  label: Text(_ticker?.muted == true ? '取消静音' : '静音'),
                ),
                ElevatedButton.icon(
                  onPressed: () {
                    setState(() {
                      _tickCount = 0;
                      _progress = 0;
                      _elapsed = Duration.zero;
                    });
                  },
                  icon: const Icon(Icons.refresh),
                  label: const Text('重置'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildInfoCard(String label, String value) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: TextStyle(
              fontSize: 14,
              color: Colors.grey[600],
            ),
          ),
          Text(
            value,
            style: const TextStyle(
              fontSize: 14,
              fontWeight: FontWeight.bold,
            ),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：`Ticker` 是 `AnimationController` 的底层实现基础。当你使用 `AnimationController` 时，它内部会创建一个 `Ticker` 来驱动动画。直接使用 `Ticker` 可以实现更精细的动画控制。

---

## 第五章：TickerProvider

`TickerProvider` 是一个抽象类，用于创建和管理 `Ticker` 实例。它是 Flutter 动画系统中实现 vsync 机制的关键。

### 5.1 类定义

```dart
/// Ticker 提供者抽象类
abstract class TickerProvider {
  /// 创建一个 Ticker
  /// [onTick] - 每帧触发的回调
  Ticker createTicker(TickerCallback onTick);
}
```

### 5.2 内置 TickerProvider 实现

| 类名 | 用途 | 使用场景 |
|------|------|----------|
| `TickerProviderStateMixin` | StatefulWidget 的 Mixin | 单动画控制器 |
| `SingleTickerProviderStateMixin` | 单 Ticker 的 Mixin | 单个动画 |
| `TickerMode` | Widget 级别的 Ticker 控制 | 控制子树的 Ticker 行为 |

### 5.3 SingleTickerProviderStateMixin

```dart
class MyAnimatedWidget extends StatefulWidget {
  const MyAnimatedWidget({super.key});

  @override
  State<MyAnimatedWidget> createState() => _MyAnimatedWidgetState();
}

class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    // 使用 this 作为 TickerProvider
    _controller = AnimationController(
      vsync: this,  // 关键参数
      duration: const Duration(seconds: 2),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### 5.4 TickerProviderStateMixin

```dart
class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
    with TickerProviderStateMixin {
  late AnimationController _controller1;
  late AnimationController _controller2;

  @override
  void initState() {
    super.initState();
    // 可以创建多个动画控制器
    _controller1 = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    _controller2 = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );
  }

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### 5.5 TickerMode

`TickerMode` 是一个 Widget，用于控制其子树中的 Ticker 行为。当 `enabled` 为 `false` 时，子树中的所有 Ticker 都会被静音。

```dart
class TickerModeDemo extends StatefulWidget {
  const TickerModeDemo({super.key});

  @override
  State<TickerModeDemo> createState() => _TickerModeDemoState();
}

class _TickerModeDemoState extends State<TickerModeDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  bool _tickerEnabled = true;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        SwitchListTile(
          title: const Text('启用 Ticker'),
          value: _tickerEnabled,
          onChanged: (value) {
            setState(() {
              _tickerEnabled = value;
            });
          },
        ),
        // 使用 TickerMode 控制子树
        TickerMode(
          enabled: _tickerEnabled,
          child: RotationTransition(
            turns: _controller,
            child: Container(
              width: 100,
              height: 100,
              color: Colors.blue,
              child: const Icon(Icons.refresh, color: Colors.white, size: 48),
            ),
          ),
        ),
      ],
    );
  }
}
```

### 5.6 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TickerProvider Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const TickerProviderDemoPage(),
    );
  }
}

class TickerProviderDemoPage extends StatelessWidget {
  const TickerProviderDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('TickerProvider 演示'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'Single'),
              Tab(text: 'Multiple'),
              Tab(text: 'TickerMode'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            SingleTickerDemo(),
            MultipleTickerDemo(),
            TickerModeDemo(),
          ],
        ),
      ),
    );
  }
}

// SingleTickerProviderStateMixin 演示
class SingleTickerDemo extends StatefulWidget {
  const SingleTickerDemo({super.key});

  @override
  State<SingleTickerDemo> createState() => _SingleTickerDemoState();
}

class _SingleTickerDemoState extends State<SingleTickerDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: RotationTransition(
        turns: _controller,
        child: Container(
          width: 150,
          height: 150,
          decoration: BoxDecoration(
            gradient: const LinearGradient(
              colors: [Colors.blue, Colors.purple],
            ),
            borderRadius: BorderRadius.circular(16),
          ),
          child: const Icon(Icons.sync, color: Colors.white, size: 64),
        ),
      ),
    );
  }
}

// TickerProviderStateMixin 演示
class MultipleTickerDemo extends StatefulWidget {
  const MultipleTickerDemo({super.key});

  @override
  State<MultipleTickerDemo> createState() => _MultipleTickerDemoState();
}

class _MultipleTickerDemoState extends State<MultipleTickerDemo>
    with TickerProviderStateMixin {
  late AnimationController _controller1;
  late AnimationController _controller2;
  late AnimationController _controller3;

  @override
  void initState() {
    super.initState();
    _controller1 = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat();
    _controller2 = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
    _controller3 = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat();
  }

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    _controller3.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Stack(
        alignment: Alignment.center,
        children: [
          RotationTransition(
            turns: _controller1,
            child: Container(
              width: 200,
              height: 200,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.red, width: 4),
                borderRadius: BorderRadius.circular(20),
              ),
            ),
          ),
          RotationTransition(
            turns: _controller2,
            child: Container(
              width: 140,
              height: 140,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.green, width: 4),
                borderRadius: BorderRadius.circular(16),
              ),
            ),
          ),
          RotationTransition(
            turns: _controller3,
            child: Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.blue, width: 4),
                borderRadius: BorderRadius.circular(12),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// TickerMode 演示
class TickerModeDemo extends StatefulWidget {
  const TickerModeDemo({super.key});

  @override
  State<TickerModeDemo> createState() => _TickerModeDemoState();
}

class _TickerModeDemoState extends State<TickerModeDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  bool _enabled = true;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        SwitchListTile(
          title: const Text('启用 Ticker'),
          subtitle: const Text('禁用后动画会暂停'),
          value: _enabled,
          onChanged: (value) {
            setState(() {
              _enabled = value;
            });
          },
        ),
        const SizedBox(height: 40),
        TickerMode(
          enabled: _enabled,
          child: RotationTransition(
            turns: _controller,
            child: Container(
              width: 150,
              height: 150,
              decoration: BoxDecoration(
                color: _enabled ? Colors.blue : Colors.grey,
                borderRadius: BorderRadius.circular(16),
              ),
              child: const Icon(Icons.animation, color: Colors.white, size: 64),
            ),
          ),
        ),
        const SizedBox(height: 40),
        Text(
          _enabled ? '动画运行中' : '动画已暂停',
          style: TextStyle(
            fontSize: 18,
            color: _enabled ? Colors.green : Colors.red,
          ),
        ),
      ],
    );
  }
}
```

> **补充知识**：`SingleTickerProviderStateMixin` 和 `TickerProviderStateMixin` 的区别在于前者只能创建一个 Ticker，后者可以创建多个。如果你只需要一个动画控制器，使用 `SingleTickerProviderStateMixin` 更高效。

---

## 第六章：FrameTiming 性能监控

`FrameTiming` 提供了帧的性能指标，用于监控应用的帧率和性能表现。

### 6.1 FrameTiming 类

```dart
class FrameTiming {
  /// 构建帧的持续时间（UI 线程）
  Duration get buildDuration;
  
  /// 光栅化帧的持续时间（Raster 线程）
  Duration get rasterDuration;
  
  /// 接收 VSync 信号到开始构建的延迟
  Duration get vsyncOverhead;
  
  /// 从 VSync 开始到光栅化完成的总时间
  Duration get totalSpan;
  
  /// 层缓存数量
  int get layerCacheCount;
  
  /// 层缓存字节数
  int get layerCacheBytes;
  
  /// 图片缓存数量
  int get pictureCacheCount;
  
  /// 图片缓存字节数
  int get pictureCacheBytes;
  
  /// 帧编号
  int get frameNumber;
}
```

### 6.2 FramePhase 枚举

```dart
enum FramePhase {
  vsyncStart,      // VSync 信号开始
  buildStart,      // 构建开始
  buildFinish,     // 构建完成
  rasterStart,     // 光栅化开始
  rasterFinish,    // 光栅化完成
}
```

### 6.3 性能监控方法

```dart
// 添加性能监控回调
SchedulerBinding.instance.addTimingsCallback((List<FrameTiming> timings) {
  for (final timing in timings) {
    final buildMs = timing.buildDuration.inMicroseconds / 1000;
    final rasterMs = timing.rasterDuration.inMicroseconds / 1000;
    final totalMs = timing.totalSpan.inMicroseconds / 1000;
    
    debugPrint('帧 ${timing.frameNumber}: '
        '构建=${buildMs.toStringAsFixed(2)}ms, '
        '光栅化=${rasterMs.toStringAsFixed(2)}ms, '
        '总计=${totalMs.toStringAsFixed(2)}ms');
  }
});
```

### 6.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  // 注册性能监控
  SchedulerBinding.instance.addTimingsCallback((timings) {
    for (final timing in timings) {
      final buildMs = timing.buildDuration.inMicroseconds / 1000;
      final rasterMs = timing.rasterDuration.inMicroseconds / 1000;
      
      // 检测掉帧（>16.67ms 对于 60fps）
      if (buildMs > 16.67 || rasterMs > 16.67) {
        debugPrint('⚠️ 掉帧检测: 帧 ${timing.frameNumber}, '
            '构建=${buildMs.toStringAsFixed(2)}ms, '
            '光栅化=${rasterMs.toStringAsFixed(2)}ms');
      }
    }
  });
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'FrameTiming Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const FrameTimingDemoPage(),
    );
  }
}

class FrameTimingDemoPage extends StatefulWidget {
  const FrameTimingDemoPage({super.key});

  @override
  State<FrameTimingDemoPage> createState() => _FrameTimingDemoPageState();
}

class _FrameTimingDemoPageState extends State<FrameTimingDemoPage> {
  final List<FrameData> _frameData = [];
  bool _isMonitoring = false;
  
  @override
  void initState() {
    super.initState();
    _startMonitoring();
  }
  
  void _startMonitoring() {
    SchedulerBinding.instance.addTimingsCallback(_onTimings);
    setState(() {
      _isMonitoring = true;
    });
  }
  
  void _stopMonitoring() {
    SchedulerBinding.instance.removeTimingsCallback(_onTimings);
    setState(() {
      _isMonitoring = false;
    });
  }
  
  void _onTimings(List<FrameTiming> timings) {
    for (final timing in timings) {
      setState(() {
        _frameData.add(FrameData(
          frameNumber: timing.frameNumber,
          buildMs: timing.buildDuration.inMicroseconds / 1000,
          rasterMs: timing.rasterDuration.inMicroseconds / 1000,
          totalMs: timing.totalSpan.inMicroseconds / 1000,
          timestamp: DateTime.now(),
        ));
        
        // 只保留最近 100 帧
        if (_frameData.length > 100) {
          _frameData.removeAt(0);
        }
      });
    }
  }

  @override
  void dispose() {
    _stopMonitoring();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final avgBuildMs = _frameData.isEmpty
        ? 0
        : _frameData.map((f) => f.buildMs).reduce((a, b) => a + b) / _frameData.length;
    final avgRasterMs = _frameData.isEmpty
        ? 0
        : _frameData.map((f) => f.rasterMs).reduce((a, b) => a + b) / _frameData.length;
    final avgFps = avgBuildMs + avgRasterMs > 0
        ? 1000 / (avgBuildMs + avgRasterMs)
        : 0;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('FrameTiming 性能监控'),
        actions: [
          IconButton(
            icon: Icon(_isMonitoring ? Icons.pause : Icons.play_arrow),
            onPressed: _isMonitoring ? _stopMonitoring : _startMonitoring,
          ),
          IconButton(
            icon: const Icon(Icons.delete),
            onPressed: () {
              setState(() {
                _frameData.clear();
              });
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // 统计卡片
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                _buildStatCard('平均构建', '${avgBuildMs.toStringAsFixed(2)}ms', Colors.blue),
                const SizedBox(width: 8),
                _buildStatCard('平均光栅化', '${avgRasterMs.toStringAsFixed(2)}ms', Colors.green),
                const SizedBox(width: 8),
                _buildStatCard('平均 FPS', avgFps.toStringAsFixed(1), Colors.purple),
              ],
            ),
          ),
          
          // FPS 图表
          Container(
            height: 150,
            margin: const EdgeInsets.symmetric(horizontal: 16),
            decoration: BoxDecoration(
              color: Colors.grey[100],
              borderRadius: BorderRadius.circular(12),
            ),
            child: CustomPaint(
              size: const Size(double.infinity, 150),
              painter: FpsChartPainter(_frameData),
            ),
          ),
          
          const Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  flex: 2,
                  child: Text('帧号', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
                Expanded(
                  child: Text('构建', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
                Expanded(
                  child: Text('光栅化', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
                Expanded(
                  child: Text('总计', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
              ],
            ),
          ),
          
          // 帧数据列表
          Expanded(
            child: ListView.builder(
              itemCount: _frameData.length,
              reverse: true,
              itemBuilder: (context, index) {
                final data = _frameData[_frameData.length - 1 - index];
                final isJank = data.totalMs > 16.67;
                
                return Container(
                  color: isJank ? Colors.red.withOpacity(0.1) : null,
                  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                  child: Row(
                    children: [
                      Expanded(
                        flex: 2,
                        child: Text('#${data.frameNumber}'),
                      ),
                      Expanded(
                        child: Text('${data.buildMs.toStringAsFixed(2)}ms'),
                      ),
                      Expanded(
                        child: Text('${data.rasterMs.toStringAsFixed(2)}ms'),
                      ),
                      Expanded(
                        child: Text(
                          '${data.totalMs.toStringAsFixed(2)}ms',
                          style: TextStyle(
                            color: isJank ? Colors.red : null,
                            fontWeight: isJank ? FontWeight.bold : null,
                          ),
                        ),
                      ),
                    ],
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildStatCard(String label, String value, Color color) {
    return Expanded(
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Column(
            children: [
              Text(
                label,
                style: TextStyle(
                  fontSize: 12,
                  color: Colors.grey[600],
                ),
              ),
              const SizedBox(height: 4),
              Text(
                value,
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                  color: color,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class FrameData {
  final int frameNumber;
  final double buildMs;
  final double rasterMs;
  final double totalMs;
  final DateTime timestamp;

  FrameData({
    required this.frameNumber,
    required this.buildMs,
    required this.rasterMs,
    required this.totalMs,
    required this.timestamp,
  });
}

class FpsChartPainter extends CustomPainter {
  final List<FrameData> data;

  FpsChartPainter(this.data);

  @override
  void paint(Canvas canvas, Size size) {
    if (data.isEmpty) return;

    final paint = Paint()
      ..strokeWidth = 2
      ..style = PaintingStyle.stroke;

    final maxMs = data.map((d) => d.totalMs).reduce((a, b) => a > b ? a : b);
    final scaleY = size.height / (maxMs * 1.2);
    final stepX = size.width / (data.length > 50 ? 50 : data.length);

    // 绘制 60fps 线 (16.67ms)
    final fps60Y = size.height - (16.67 * scaleY);
    canvas.drawLine(
      Offset(0, fps60Y),
      Offset(size.width, fps60Y),
      Paint()
        ..color = Colors.green.withOpacity(0.5)
        ..strokeWidth = 1
        ..style = PaintingStyle.stroke,
    );

    // 绘制数据曲线
    final path = Path();
    final startIndex = data.length > 50 ? data.length - 50 : 0;
    
    for (int i = startIndex; i < data.length; i++) {
      final x = (i - startIndex) * stepX;
      final y = size.height - (data[i].totalMs * scaleY);
      
      if (i == startIndex) {
        path.moveTo(x, y);
      } else {
        path.lineTo(x, y);
      }
    }

    paint.color = Colors.blue;
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

> **补充知识**：性能监控建议在 Profile 或 Release 模式下进行，Debug 模式的性能数据可能不准确，因为包含了调试开销。使用 `addTimingsCallback` 可以注册多个回调函数，方便模块化监控。

---

## 第七章：应用生命周期管理

`SchedulerBinding` 提供了应用生命周期状态的监听和管理功能，用于响应应用的前后台切换等事件。

### 7.1 AppLifecycleState 枚举

```dart
enum AppLifecycleState {
  /// 应用正在运行且可见，可以接收用户输入
  resumed,
  
  /// 应用可见但不可接收用户输入（如来电、系统对话框）
  inactive,
  
  /// 应用不可见，进入后台
  paused,
  
  /// 应用被挂起，即将被销毁（仅 iOS）
  detached,
  
  /// 应用被隐藏（仅 iOS，应用切换器）
  hidden,
}
```

### 7.2 生命周期状态流转

```
┌───────────┐
│  detached │
└─────┬─────┘
      │ 应用启动
      ▼
┌───────────┐     来电/系统对话框      ┌───────────┐
│  resumed  │ ◀──────────────────────▶ │  inactive │
│  (前台)   │                          │  (中断)   │
└─────┬─────┘                          └─────┬─────┘
      │                                      │
      │ 进入后台                              │ 恢复前台
      ▼                                      │
┌───────────┐                                │
│   paused  │ ◀──────────────────────────────┘
│  (后台)   │
└───────────┘
```

### 7.3 生命周期监听方法

| 方法/属性 | 说明 |
|-----------|------|
| `lifecycleState` | 获取当前生命周期状态 |
| `handleAppLifecycleStateChanged()` | 处理生命周期状态变化 |
| `addObserver()` | 添加生命周期观察者 |
| `removeObserver()` | 移除生命周期观察者 |

### 7.4 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Lifecycle Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const LifecycleDemoPage(),
    );
  }
}

class LifecycleDemoPage extends StatefulWidget {
  const LifecycleDemoPage({super.key});

  @override
  State<LifecycleDemoPage> createState() => _LifecycleDemoPageState();
}

class _LifecycleDemoPageState extends State<LifecycleDemoPage>
    with WidgetsBindingObserver {
  final List<LifecycleEvent> _events = [];
  AppLifecycleState? _currentState;
  int _resumeCount = 0;
  int _pauseCount = 0;
  
  @override
  void initState() {
    super.initState();
    // 注册生命周期观察者
    WidgetsBinding.instance.addObserver(this);
    _currentState = SchedulerBinding.instance.lifecycleState;
    _addEvent('初始化', _currentState);
  }

  @override
  void dispose() {
    // 移除生命周期观察者
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    setState(() {
      _currentState = state;
      _addEvent('状态变化', state);
      
      switch (state) {
        case AppLifecycleState.resumed:
          _resumeCount++;
          // 恢复前台操作
          debugPrint('应用回到前台');
          break;
        case AppLifecycleState.paused:
          _pauseCount++;
          // 进入后台操作
          debugPrint('应用进入后台');
          break;
        case AppLifecycleState.inactive:
          debugPrint('应用变为非活跃');
          break;
        case AppLifecycleState.detached:
          debugPrint('应用被分离');
          break;
        case AppLifecycleState.hidden:
          debugPrint('应用被隐藏');
          break;
      }
    });
  }
  
  void _addEvent(String action, AppLifecycleState? state) {
    _events.add(LifecycleEvent(
      action: action,
      state: state,
      timestamp: DateTime.now(),
    ));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('应用生命周期管理'),
      ),
      body: Column(
        children: [
          // 当前状态卡片
          Card(
            margin: const EdgeInsets.all(16),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(
                        _getStateIcon(_currentState),
                        size: 48,
                        color: _getStateColor(_currentState),
                      ),
                      const SizedBox(width: 16),
                      Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            '当前状态',
                            style: TextStyle(
                              fontSize: 14,
                              color: Colors.grey[600],
                            ),
                          ),
                          Text(
                            _currentState?.toString().split('.').last ?? '未知',
                            style: const TextStyle(
                              fontSize: 24,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                  const Divider(height: 32),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                    children: [
                      _buildCountItem('前台次数', _resumeCount, Colors.green),
                      _buildCountItem('后台次数', _pauseCount, Colors.orange),
                    ],
                  ),
                ],
              ),
            ),
          ),
          
          // 事件列表
          const Padding(
            padding: EdgeInsets.symmetric(horizontal: 16),
            child: Row(
              children: [
                Expanded(
                  flex: 2,
                  child: Text('时间', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
                Expanded(
                  flex: 2,
                  child: Text('动作', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
                Expanded(
                  child: Text('状态', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
              ],
            ),
          ),
          const Divider(),
          
          Expanded(
            child: ListView.builder(
              itemCount: _events.length,
              reverse: true,
              itemBuilder: (context, index) {
                final event = _events[_events.length - 1 - index];
                return ListTile(
                  dense: true,
                  title: Row(
                    children: [
                      Expanded(
                        flex: 2,
                        child: Text(
                          '${event.timestamp.hour.toString().padLeft(2, '0')}:''
                          '${event.timestamp.minute.toString().padLeft(2, '0')}:''
                          '${event.timestamp.second.toString().padLeft(2, '0')}.''
                          '${event.timestamp.millisecond.toString().padLeft(3, '0')}',
                          style: const TextStyle(fontSize: 12),
                        ),
                      ),
                      Expanded(
                        flex: 2,
                        child: Text(event.action),
                      ),
                      Expanded(
                        child: Text(
                          event.state?.toString().split('.').last ?? '-',
                          style: TextStyle(
                            color: _getStateColor(event.state),
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                    ],
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildCountItem(String label, int count, Color color) {
    return Column(
      children: [
        Text(
          label,
          style: TextStyle(
            fontSize: 12,
            color: Colors.grey[600],
          ),
        ),
        const SizedBox(height: 4),
        Text(
          '$count',
          style: TextStyle(
            fontSize: 24,
            fontWeight: FontWeight.bold,
            color: color,
          ),
        ),
      ],
    );
  }

  IconData _getStateIcon(AppLifecycleState? state) {
    switch (state) {
      case AppLifecycleState.resumed:
        return Icons.play_circle;
      case AppLifecycleState.paused:
        return Icons.pause_circle;
      case AppLifecycleState.inactive:
        return Icons.remove_circle;
      case AppLifecycleState.detached:
        return Icons.stop_circle;
      case AppLifecycleState.hidden:
        return Icons.visibility_off;
      default:
        return Icons.help;
    }
  }

  Color _getStateColor(AppLifecycleState? state) {
    switch (state) {
      case AppLifecycleState.resumed:
        return Colors.green;
      case AppLifecycleState.paused:
        return Colors.orange;
      case AppLifecycleState.inactive:
        return Colors.yellow;
      case AppLifecycleState.detached:
        return Colors.red;
      case AppLifecycleState.hidden:
        return Colors.grey;
      default:
        return Colors.black;
    }
  }
}

class LifecycleEvent {
  final String action;
  final AppLifecycleState? state;
  final DateTime timestamp;

  LifecycleEvent({
    required this.action,
    required this.state,
    required this.timestamp,
  });
}
```

> **补充知识**：在 `paused` 状态下，Flutter 会停止帧调度以节省电量。如果你需要在后台执行任务，应该使用 Flutter 的后台服务插件（如 `workmanager`）。

---

## 第八章：任务优先级与调度

`SchedulerBinding` 提供了任务优先级机制，用于控制任务的执行顺序。

### 8.1 Priority 类

```dart
class Priority {
  /// 空闲时执行（最低优先级）
  static const Priority idle = Priority._(0);
  
  /// 动画相关任务
  static const Priority animation = Priority._(100000);
  
  /// 触摸交互相关任务
  static const Priority touch = Priority._(200000);
  
  /// 构建和布局任务
  static const Priority build = Priority._(300000);
  
  /// 创建优先级
  const Priority._(this._value);
  
  /// 获取更高优先级（值更小）
  Priority get higher => Priority._(_value - 1);
  
  /// 获取更低优先级（值更大）
  Priority get lower => Priority._(_value + 1);
}
```

### 8.2 任务调度方法

```dart
/// 调度任务
void scheduleTask(
  TaskCallback<T> task,
  Priority priority, {
  String? debugLabel,
  Flow? flow,
});
```

### 8.3 完整示例代码

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';

class TaskPriorityDemo extends StatefulWidget {
  const TaskPriorityDemo({super.key});

  @override
  State<TaskPriorityDemo> createState() => _TaskPriorityDemoState();
}

class _TaskPriorityDemoState extends State<TaskPriorityDemo> {
  final List<String> _logs = [];
  bool _isRunning = false;

  void _addLog(String message) {
    setState(() {
      _logs.add('[${DateTime.now().millisecond}] $message');
      if (_logs.length > 50) {
        _logs.removeAt(0);
      }
    });
  }

  Future<void> _runTasks() async {
    setState(() {
      _isRunning = true;
      _logs.clear();
    });

    // 调度不同优先级的任务
    SchedulerBinding.instance.scheduleTask(
      () => _addLog('🟢 空闲任务执行'),
      Priority.idle,
      debugLabel: 'idle-task',
    );

    SchedulerBinding.instance.scheduleTask(
      () => _addLog('🔵 动画任务执行'),
      Priority.animation,
      debugLabel: 'animation-task',
    );

    SchedulerBinding.instance.scheduleTask(
      () => _addLog('🟡 触摸任务执行'),
      Priority.touch,
      debugLabel: 'touch-task',
    );

    SchedulerBinding.instance.scheduleTask(
      () => _addLog('🔴 构建任务执行'),
      Priority.build,
      debugLabel: 'build-task',
    );

    // 自定义优先级
    SchedulerBinding.instance.scheduleTask(
      () => _addLog('⚪ 自定义高优先级任务'),
      Priority.build.higher,
      debugLabel: 'custom-high-task',
    );

    await Future.delayed(const Duration(seconds: 1));
    
    setState(() {
      _isRunning = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('任务优先级调度'),
      ),
      body: Column(
        children: [
          // 优先级说明
          Card(
            margin: const EdgeInsets.all(16),
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    '优先级顺序（从高到低）',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  _buildPriorityItem('build (300000)', '构建和布局任务', Colors.red),
                  _buildPriorityItem('touch (200000)', '触摸交互任务', Colors.yellow),
                  _buildPriorityItem('animation (100000)', '动画任务', Colors.blue),
                  _buildPriorityItem('idle (0)', '空闲任务', Colors.green),
                ],
              ),
            ),
          ),
          
          // 执行按钮
          ElevatedButton(
            onPressed: _isRunning ? null : _runTasks,
            child: Text(_isRunning ? '执行中...' : '运行任务'),
          ),
          
          const Divider(),
          
          // 日志输出
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.all(16),
              itemCount: _logs.length,
              itemBuilder: (context, index) {
                return Text(
                  _logs[index],
                  style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildPriorityItem(String name, String desc, Color color) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 4),
      child: Row(
        children: [
          Container(
            width: 12,
            height: 12,
            decoration: BoxDecoration(
              color: color,
              shape: BoxShape.circle,
            ),
          ),
          const SizedBox(width: 8),
          Expanded(
            child: Text('$name - $desc'),
          ),
        ],
      ),
    );
  }
}
```

> **补充知识**：任务优先级只在当前帧内有效，高优先级任务会优先于低优先级任务执行。但所有任务都会在下一帧开始前完成。

---

## 第九章：高级用法与最佳实践

### 9.1 帧回调的最佳实践

```dart
class FrameCallbackBestPractices {
  /// 1. 使用 addPostFrameCallback 获取 Widget 尺寸
  void measureWidget(BuildContext context) {
    SchedulerBinding.instance.addPostFrameCallback((_) {
      final renderBox = context.findRenderObject() as RenderBox?;
      final size = renderBox?.size;
      debugPrint('Widget 尺寸: $size');
    });
  }

  /// 2. 使用 scheduleFrameCallback 实现自定义动画
  void customAnimation() {
    var startTime = Duration.zero;
    
    void animate(Duration elapsed) {
      if (startTime == Duration.zero) {
        startTime = elapsed;
      }
      
      final progress = (elapsed - startTime).inMilliseconds / 1000;
      
      if (progress < 1.0) {
        // 继续动画
        SchedulerBinding.instance.scheduleFrameCallback(animate);
      }
    }
    
    SchedulerBinding.instance.scheduleFrameCallback(animate);
  }

  /// 3. 使用 persistentCallbacks 实现自定义渲染
  void customRender() {
    SchedulerBinding.instance.addPersistentFrameCallback((_) {
      // 每帧执行自定义渲染逻辑
      // 注意：不要在这里执行耗时操作
    });
  }
}
```

### 9.2 性能优化技巧

```dart
class PerformanceOptimization {
  /// 1. 避免在帧回调中执行耗时操作
  void avoidExpensiveOperations() {
    SchedulerBinding.instance.addPersistentFrameCallback((_) {
      // ❌ 错误：耗时操作会阻塞帧
      // heavyComputation();
      
      // ✅ 正确：使用 isolate 或延迟执行
      Future.microtask(() => heavyComputation());
    });
  }

  /// 2. 合理使用 TickerMode
  Widget optimizeWithTickerMode(bool isVisible, Widget child) {
    return TickerMode(
      enabled: isVisible,
      child: child,
    );
  }

  /// 3. 及时释放 Ticker
  Ticker? _ticker;
  
  void createTicker() {
    _ticker?.dispose(); // 先释放旧的
    _ticker = Ticker((_) {
      // 动画逻辑
    });
  }

  /// 4. 批量处理帧回调
  void batchFrameCallbacks() {
    final callbacks = <VoidCallback>[];
    
    void registerCallback(VoidCallback callback) {
      callbacks.add(callback);
    }
    
    SchedulerBinding.instance.addPostFrameCallback((_) {
      // 批量执行所有回调
      for (final callback in callbacks) {
        callback();
      }
      callbacks.clear();
    });
  }

  void heavyComputation() {
    // 模拟耗时操作
  }
}
```

### 9.3 常见错误与解决方案

```dart
class CommonMistakes {
  /// 错误 1：在 build 方法中调用 setState
  /// 解决方案：使用 addPostFrameCallback
  Widget wrongWay(BuildContext context) {
    // ❌ 错误
    // setState(() {});
    
    // ✅ 正确
    SchedulerBinding.instance.addPostFrameCallback((_) {
      // setState(() {});
    });
    
    return Container();
  }

  /// 错误 2：忘记释放 Ticker
  /// 解决方案：在 dispose 中释放
  Ticker? _ticker;
  
  void initTicker() {
    _ticker = Ticker((_) {})..start();
  }
  
  void dispose() {
    _ticker?.dispose(); // ✅ 正确
    // super.dispose();
  }

  /// 错误 3：在帧回调中修改状态
  /// 解决方案：检查当前阶段
  void safeStateUpdate() {
    final binding = SchedulerBinding.instance;
    
    if (binding.schedulerPhase == SchedulerPhase.persistentCallbacks) {
      // 如果在构建阶段，延迟到帧后
      binding.addPostFrameCallback((_) {
        // 安全地更新状态
      });
    } else {
      // 直接更新状态
    }
  }
}
```

---

## 附录 A：API 速查表

### A.1 SchedulerBinding 方法速查

| 方法 | 用途 | 返回值 |
|------|------|--------|
| `scheduleFrame()` | 请求调度新帧 | void |
| `scheduleFrameCallback()` | 注册瞬时帧回调 | int (callback ID) |
| `cancelFrameCallbackWithId()` | 取消帧回调 | void |
| `addPersistentFrameCallback()` | 添加持久帧回调 | void |
| `addPostFrameCallback()` | 添加帧后回调 | void |
| `addTimingsCallback()` | 添加性能监控回调 | void |
| `removeTimingsCallback()` | 移除性能监控回调 | void |
| `ensureVisualUpdate()` | 确保视觉更新 | void |
| `scheduleTask()` | 调度任务 | void |
| `handleBeginFrame()` | 处理帧开始 | void |
| `handleDrawFrame()` | 处理帧绘制 | void |

### A.2 Ticker 方法速查

| 方法 | 用途 | 返回值 |
|------|------|--------|
| `start()` | 启动 Ticker | TickerFuture |
| `stop()` | 停止 Ticker | void |
| `scheduleTick()` | 调度下一帧 | void |
| `absorbTicker()` | 吸收其他 Ticker | void |
| `dispose()` | 释放资源 | void |

### A.3 回调类型定义

```dart
/// 帧回调签名
typedef FrameCallback = void Function(Duration timeStamp);

/// Ticker 回调签名
typedef TickerCallback = void Function(Duration elapsed);

/// 任务回调签名
typedef TaskCallback<T> = Future<T> Function();

/// 时间戳回调签名
typedef TimingsCallback = void Function(List<FrameTiming> timings);
```

### A.4 SchedulerPhase 枚举值

| 值 | 说明 | 典型操作 |
|----|------|----------|
| `idle` | 空闲状态 | 等待下一帧 |
| `transientCallbacks` | 瞬时回调阶段 | 动画更新 |
| `midFrameMicrotasks` | 微任务阶段 | Future 完成回调 |
| `persistentCallbacks` | 持久回调阶段 | 构建、布局、绘制 |
| `postFrameCallbacks` | 帧后回调阶段 | 一次性任务 |

### A.5 AppLifecycleState 枚举值

| 值 | 说明 | 典型场景 |
|----|------|----------|
| `resumed` | 前台运行 | 正常交互 |
| `inactive` | 非活跃 | 来电、系统对话框 |
| `paused` | 后台暂停 | 应用进入后台 |
| `detached` | 已分离 | 应用即将销毁 |
| `hidden` | 隐藏 | 应用切换器 |

---

## 附录 B：性能指标参考

### B.1 帧率标准

| 目标帧率 | 每帧时间 | 适用场景 |
|----------|----------|----------|
| 120 FPS | 8.33 ms | 高刷新率设备 |
| 90 FPS | 11.11 ms | 中高端设备 |
| 60 FPS | 16.67 ms | 标准帧率 |
| 30 FPS | 33.33 ms | 低功耗模式 |

### B.2 性能指标阈值

| 指标 | 优秀 | 良好 | 需优化 |
|------|------|------|--------|
| buildDuration | < 8ms | < 16ms | > 16ms |
| rasterDuration | < 8ms | < 16ms | > 16ms |
| totalSpan | < 16ms | < 33ms | > 33ms |
| vsyncOverhead | < 2ms | < 4ms | > 4ms |

### B.3 性能监控代码模板

```dart
class PerformanceMonitor {
  static void startMonitoring() {
    SchedulerBinding.instance.addTimingsCallback((timings) {
      for (final timing in timings) {
        final buildMs = timing.buildDuration.inMicroseconds / 1000;
        final rasterMs = timing.rasterDuration.inMicroseconds / 1000;
        final totalMs = timing.totalSpan.inMicroseconds / 1000;
        
        // 检测掉帧
        if (totalMs > 16.67) {
          debugPrint('⚠️ 掉帧: 帧 ${timing.frameNumber}, '
              '总计 ${totalMs.toStringAsFixed(2)}ms');
        }
        
        // 检测构建耗时
        if (buildMs > 8) {
          debugPrint('⚠️ 构建耗时: ${buildMs.toStringAsFixed(2)}ms');
        }
        
        // 检测光栅化耗时
        if (rasterMs > 8) {
          debugPrint('⚠️ 光栅化耗时: ${rasterMs.toStringAsFixed(2)}ms');
        }
      }
    });
  }
}
```

---

## 附录 C：常见问题解答

### Q1: 什么时候使用 addPostFrameCallback？

A: 当你需要在当前帧渲染完成后执行操作时，例如：
- 获取 Widget 的最终尺寸
- 显示对话框或 SnackBar
- 执行需要完整布局信息的操作

### Q2: Ticker 和 Timer 有什么区别？

A: 
- `Ticker` 与屏幕刷新同步，适合动画
- `Timer` 基于时间间隔，不保证与帧同步
- `Ticker` 可以暂停/恢复，`Timer` 只能取消

### Q3: 如何检测应用是否掉帧？

A: 使用 `addTimingsCallback` 监控帧时间：

```dart
SchedulerBinding.instance.addTimingsCallback((timings) {
  for (final timing in timings) {
    if (timing.totalSpan.inMilliseconds > 16) {
      debugPrint('掉帧: ${timing.totalSpan.inMilliseconds}ms');
    }
  }
});
```

### Q4: 为什么动画在后台还在运行？

A: 使用 `TickerMode` 控制动画：

```dart
TickerMode(
  enabled: isVisible, // 根据可见性控制
  child: AnimatedWidget(...),
)
```

### Q5: 如何优化大量动画的性能？

A:
1. 使用 `TickerMode` 暂停不可见动画
2. 使用 `RepaintBoundary` 隔离重绘
3. 避免在动画中执行耗时操作
4. 考虑使用 `AnimatedBuilder` 优化重建范围

---

**文档版本**: 1.0  
**最后更新**: 2024年  
**Flutter 版本**: 3.41.0

---

> **补充知识**：`flutter/scheduler.dart` 是 Flutter 框架的底层核心库，深入理解它的工作原理对于开发高性能 Flutter 应用至关重要。建议结合实际项目中的性能优化需求，灵活运用各种帧回调和性能监控工具。

