# Flutter Flame 1.35.1 完整详解与实战指南

## 前言

Flutter 是一个强大的跨平台 UI 框架，而 Flame 则是专为 Flutter 设计的轻量级游戏引擎。Flame 建立在 Flutter 的渲染管道之上，让开发者能够轻松创建高性能的 2D 游戏，同时支持移动端、Web 和桌面端。

Flame 提供了游戏开发所需的核心功能：
- 游戏循环（Game Loop）
- 组件系统（Flame Component System）
- 精灵和动画
- 输入处理（触摸、键盘、手柄）
- 碰撞检测
- 粒子系统
- 音频管理
- 物理引擎集成

本书基于 Flame 1.35.1 版本，详细介绍该引擎的所有核心概念、API 用法，以及如何构建完整的 Flutter 游戏。

---

## 第1章 Flame 基础

### 1.1 什么是 Flame

Flame 是一个模块化的 Flutter 游戏引擎，它的设计理念是简单、高效、可扩展。

| 特性 | 说明 |
|------|------|
| **游戏循环** | 自动管理 update/render 循环 |
| **组件系统** | 基于组件的架构，易于组织和复用 |
| **跨平台** | 支持 iOS、Android、Web、Windows、macOS、Linux |
| **高性能** | 基于 Flutter 的 Skia 渲染引擎 |
| **模块化** | 按需引入功能模块 |

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flame: ^1.35.1

# 可选扩展包
  flame_audio: ^2.10.2          # 音频
  flame_forge2d: ^0.18.2        # 物理引擎
  flame_tiled: ^1.20.3          # Tiled 地图
  flame_lottie: ^0.4.1          # Lottie 动画
  flame_riverpod: ^5.4.5        # Riverpod 集成
  flame_bloc: ^1.12.5           # Bloc 集成
```

#### 1.2.2 项目结构

```
lib/
├── main.dart                    # 应用入口
├── game/
│   ├── my_game.dart             # 游戏主类
│   └── world.dart               # 游戏世界
├── components/
│   ├── player.dart              # 玩家组件
│   ├── enemy.dart               # 敌人组件
│   ├── bullet.dart              # 子弹组件
│   └── background.dart          # 背景组件
├── managers/
│   ├── audio_manager.dart       # 音频管理
│   └── game_manager.dart        # 游戏状态管理
└── utils/
    └── constants.dart           # 常量定义

assets/
├── images/                      # 图片资源
│   ├── player.png
│   ├── enemy.png
│   └── background.png
├── audio/                       # 音频资源
│   ├── bgm/
│   └── sfx/
└── tiles/                       # Tiled 地图
    └── map.tmx
```

#### 1.2.3 配置资源

在 `pubspec.yaml` 中声明资源：

```yaml
flutter:
  assets:
    - assets/images/
    - assets/audio/bgm/
    - assets/audio/sfx/
    - assets/tiles/
```

---

### 1.3 第一个 Flame 游戏

#### 1.3.1 创建游戏主类

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    GameWidget(
      game: MyFirstGame(),
    ),
  );
}

class MyFirstGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    // 游戏初始化
    print('Game loaded!');
  }

  @override
  void update(double dt) {
    // 游戏逻辑更新
    super.update(dt);
  }

  @override
  void render(Canvas canvas) {
    // 游戏渲染
    super.render(canvas);
  }
}
```

> **Flutter框架小知识**
> 
> `GameWidget` 是 Flame 提供的 Widget，它将 Flame 游戏嵌入到 Flutter 的 Widget 树中。你可以将它放在 Flutter 应用的任何位置，与其他 Widget 混合使用。

#### 1.3.2 绘制简单图形

```dart
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

class SimpleGame extends FlameGame {
  @override
  void render(Canvas canvas) {
    // 绘制红色背景
    canvas.drawRect(
      size.toRect(),
      Paint()..color = Colors.black,
    );

    // 绘制圆形
    canvas.drawCircle(
      Offset(size.x / 2, size.y / 2),
      50,
      Paint()..color = Colors.red,
    );

    super.render(canvas);
  }
}
```

---

## 第2章 核心类详解

### 2.1 FlameGame

`FlameGame` 是 Flame 的核心游戏类，它管理游戏的生命周期、组件系统和渲染。

#### 2.1.1 核心方法

| 方法名 | 返回值 | 说明 |
|--------|--------|------|
| onLoad | Future<void> | 游戏加载时调用，用于初始化资源 |
| update | void | 每帧调用，用于更新游戏逻辑 |
| render | void | 每帧调用，用于渲染游戏画面 |
| onMount | void | 组件挂载时调用 |
| onRemove | void | 组件移除时调用 |

#### 2.1.2 核心属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| size | Vector2 | 游戏视口大小 |
| camera | CameraComponent | 游戏相机 |
| world | World | 游戏世界 |
| children | ComponentSet | 子组件集合 |
| overlays | ActiveOverlays | 覆盖层管理 |

#### 2.1.3 完整示例

```dart
class MyGame extends FlameGame {
  late Player player;
  late EnemyManager enemyManager;

  int score = 0;
  bool isGameOver = false;

  @override
  Future<void> onLoad() async {
    // 加载精灵
    final playerSprite = await loadSprite('player.png');
    
    // 创建玩家
    player = Player(sprite: playerSprite);
    add(player);

    // 创建敌人管理器
    enemyManager = EnemyManager();
    add(enemyManager);

    // 添加背景
    add(BackgroundComponent());
  }

  @override
  void update(double dt) {
    super.update(dt);

    if (isGameOver) return;

    // 更新分数
    score += (dt * 10).toInt();
  }

  void gameOver() {
    isGameOver = true;
    overlays.add('gameOver');
  }

  void restart() {
    isGameOver = false;
    score = 0;
    overlays.remove('gameOver');
    player.reset();
    enemyManager.reset();
  }
}
```

---

### 2.2 GameWidget

`GameWidget` 用于将 Flame 游戏嵌入到 Flutter 应用中。

#### 2.2.1 构造函数参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| game | Game | 游戏实例 |
| loadingBuilder | WidgetBuilder? | 加载时显示的 Widget |
| errorBuilder | GameErrorWidgetBuilder? | 错误时显示的 Widget |
| backgroundBuilder | GameBackgroundWidgetBuilder? | 背景构建器 |
| overlayBuilderMap | Map<String, Widget Function(BuildContext, Game)>? | 覆盖层构建器 |
| initialActiveOverlays | List<String>? | 初始激活的覆盖层 |

#### 2.2.2 完整示例

```dart
GameWidget(
  game: MyGame(),
  loadingBuilder: (context) => const Center(
    child: CircularProgressIndicator(),
  ),
  errorBuilder: (context, error) => Center(
    child: Text('Error: $error'),
  ),
  overlayBuilderMap: {
    'pause': (context, game) => PauseOverlay(game: game as MyGame),
    'gameOver': (context, game) => GameOverOverlay(game: game as MyGame),
  },
  initialActiveOverlays: const [],
)
```

---

### 2.3 Component（组件基类）

`Component` 是所有 Flame 组件的基类。

#### 2.3.1 核心方法

| 方法名 | 返回值 | 说明 |
|--------|--------|------|
| onLoad | Future<void>? | 组件加载时调用 |
| update | void | 每帧更新 |
| render | void | 每帧渲染 |
| onMount | void | 组件挂载时调用 |
| onRemove | void | 组件移除时调用 |
| add | FutureOr<void> | 添加子组件 |
| remove | FutureOr<void> | 移除子组件 |

#### 2.3.2 核心属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| parent | Component? | 父组件 |
| children | ComponentSet | 子组件集合 |
| priority | int | 渲染优先级（值越大越在上层） |
| isLoaded | bool | 是否已加载 |
| isMounted | bool | 是否已挂载 |

---

## 第3章 位置组件（PositionComponent）

`PositionComponent` 是具有位置、大小、角度等属性的组件基类。

### 3.1 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| position | Vector2? | null | 位置 |
| size | Vector2? | null | 大小 |
| scale | Vector2? | null | 缩放 |
| angle | double? | null | 旋转角度（弧度） |
| nativeAngle | double | 0 | 原生角度 |
| anchor | Anchor? | null | 锚点 |
| priority | int? | null | 渲染优先级 |

### 3.2 核心属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| position | Vector2 | 组件位置 |
| size | Vector2 | 组件大小 |
| scale | Vector2 | 缩放比例 |
| angle | double | 旋转角度 |
| anchor | Anchor | 锚点位置 |
| absolutePosition | Vector2 | 绝对位置（考虑父组件） |
| absoluteAngle | double | 绝对角度 |

### 3.3 完整示例

```dart
class MovingBox extends PositionComponent {
  double speed = 100;

  MovingBox() : super(
    position: Vector2(100, 100),
    size: Vector2(50, 50),
    anchor: Anchor.center,
  );

  @override
  void render(Canvas canvas) {
    canvas.drawRect(
      size.toRect(),
      Paint()..color = Colors.blue,
    );
  }

  @override
  void update(double dt) {
    // 向右移动
    position.x += speed * dt;

    // 边界检测
    if (position.x > 400) {
      position.x = 0;
    }
  }
}
```

---

## 第4章 精灵组件（SpriteComponent）

`SpriteComponent` 用于显示静态图片。

### 4.1 基础用法

```dart
class Player extends SpriteComponent {
  Player() : super(
    size: Vector2(64, 64),
    position: Vector2(100, 100),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    sprite = await Sprite.load('player.png');
  }
}
```

### 4.2 完整示例

```dart
class Player extends SpriteComponent with HasGameRef<MyGame> {
  double speed = 200;
  double health = 100;

  Player() : super(
    size: Vector2(64, 64),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('player.png');
    position = gameRef.size / 2;
  }

  @override
  void update(double dt) {
    // 键盘控制移动
    if (gameRef.keysPressed.contains(LogicalKeyboardKey.arrowLeft)) {
      position.x -= speed * dt;
    }
    if (gameRef.keysPressed.contains(LogicalKeyboardKey.arrowRight)) {
      position.x += speed * dt;
    }
    if (gameRef.keysPressed.contains(LogicalKeyboardKey.arrowUp)) {
      position.y -= speed * dt;
    }
    if (gameRef.keysPressed.contains(LogicalKeyboardKey.arrowDown)) {
      position.y += speed * dt;
    }

    // 边界限制
    position.clamp(
      Vector2(size.x / 2, size.y / 2),
      gameRef.size - Vector2(size.x / 2, size.y / 2),
    );
  }

  void takeDamage(double damage) {
    health -= damage;
    if (health <= 0) {
      gameRef.gameOver();
    }
  }

  void reset() {
    health = 100;
    position = gameRef.size / 2;
  }
}
```

---

## 第5章 精灵动画组件（SpriteAnimationComponent）

`SpriteAnimationComponent` 用于播放精灵动画。

### 5.1 基础用法

```dart
class AnimatedPlayer extends SpriteAnimationComponent {
  @override
  Future<void> onLoad() async {
    // 从图片序列创建动画
    final spriteSheet = await SpriteSheet.load(
      'player_spritesheet.png',
      srcSize: Vector2(32, 32),
    );

    animation = spriteSheet.createAnimation(
      row: 0,           // 行号
      stepTime: 0.1,    // 每帧持续时间
      from: 0,          // 起始帧
      to: 4,            // 结束帧
    );
  }
}
```

### 5.2 完整示例

```dart
class Player extends SpriteAnimationComponent with HasGameRef<MyGame> {
  late final SpriteAnimation _runAnimation;
  late final SpriteAnimation _idleAnimation;
  late final SpriteAnimation _jumpAnimation;

  bool isRunning = false;
  bool isJumping = false;

  @override
  Future<void> onLoad() async {
    final spriteSheet = await gameRef.images.load('player_spritesheet.png');

    // 创建动画
    _idleAnimation = _createAnimation(spriteSheet, 0, 4);
    _runAnimation = _createAnimation(spriteSheet, 1, 6);
    _jumpAnimation = _createAnimation(spriteSheet, 2, 2);

    animation = _idleAnimation;
    size = Vector2(64, 64);
    position = gameRef.size / 2;
    anchor = Anchor.center;
  }

  SpriteAnimation _createAnimation(
    Image image,
    int row,
    int frameCount,
  ) {
    return SpriteAnimation.fromFrameData(
      image,
      SpriteAnimationData.sequenced(
        amount: frameCount,
        amountPerRow: frameCount,
        textureSize: Vector2(32, 32),
        texturePosition: Vector2(0, row * 32.0),
        stepTime: 0.1,
      ),
    );
  }

  void run() {
    if (!isRunning) {
      isRunning = true;
      animation = _runAnimation;
    }
  }

  void idle() {
    if (isRunning) {
      isRunning = false;
      animation = _idleAnimation;
    }
  }

  void jump() {
    if (!isJumping) {
      isJumping = true;
      animation = _jumpAnimation;
    }
  }
}
```

---

## 第6章 输入处理

### 6.1 触摸输入（TapDetector）

```dart
class MyGame extends FlameGame with TapDetector {
  @override
  void onTapDown(TapDownInfo info) {
    print('Tap at: ${info.eventPosition.game}');
  }

  @override
  void onTapUp(TapUpInfo info) {
    print('Tap released at: ${info.eventPosition.game}');
  }
}
```

### 6.2 拖拽输入（PanDetector）

```dart
class MyGame extends FlameGame with PanDetector {
  @override
  void onPanStart(DragStartInfo info) {
    print('Pan started at: ${info.eventPosition.game}');
  }

  @override
  void onPanUpdate(DragUpdateInfo info) {
    print('Pan delta: ${info.delta.game}');
  }

  @override
  void onPanEnd(DragEndInfo info) {
    print('Pan ended');
  }
}
```

### 6.3 键盘输入（KeyboardEvents）

```dart
class MyGame extends FlameGame with KeyboardEvents {
  @override
  KeyEventResult onKeyEvent(
    KeyEvent event,
    Set<LogicalKeyboardKey> keysPressed,
  ) {
    if (event is KeyDownEvent) {
      if (event.logicalKey == LogicalKeyboardKey.space) {
        print('Space pressed!');
        return KeyEventResult.handled;
      }
    }
    return KeyEventResult.ignored;
  }
}
```

### 6.4 组件级输入（Tappable）

```dart
class Button extends PositionComponent with Tappable {
  final VoidCallback onPressed;

  Button({required this.onPressed});

  @override
  bool onTapDown(TapDownInfo info) {
    onPressed();
    return true;
  }

  @override
  void render(Canvas canvas) {
    canvas.drawRect(
      size.toRect(),
      Paint()..color = Colors.blue,
    );
  }
}
```

---

## 第7章 碰撞检测

### 7.1 基础碰撞检测

```dart
class Player extends SpriteComponent with CollisionCallbacks {
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    
    // 添加碰撞盒
    add(RectangleHitbox());
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    if (other is Enemy) {
      takeDamage(10);
    }
  }

  @override
  void onCollisionEnd(PositionComponent other) {
    print('Collision ended with $other');
  }
}
```

### 7.2 碰撞检测游戏

```dart
class MyGame extends FlameGame with HasCollisionDetection {
  @override
  Future<void> onLoad() async {
    add(Player());
    add(Enemy());
    add(Enemy());
  }
}
```

---

## 第8章 粒子系统

### 8.1 基础粒子

```dart
class Explosion extends ParticleSystemComponent {
  Explosion({required Vector2 position}) {
    this.position = position;
  }

  @override
  Future<void> onLoad() async {
    particleSystem = ParticleSystemComponent(
      particle: AcceleratedParticle(
        acceleration: Vector2(0, 100),
        child: CircleParticle(
          radius: 5,
          paint: Paint()..color = Colors.orange,
        ),
      ),
    );
  }
}
```

### 8.2 完整粒子效果

```dart
void createExplosion(Vector2 position) {
  final random = Random();
  
  final particleSystem = ParticleSystemComponent(
    particle: Particle.generate(
      count: 30,
      lifespan: 1,
      generator: (i) => AcceleratedParticle(
        acceleration: Vector2(
          random.nextDouble() * 200 - 100,
          random.nextDouble() * 200 - 100,
        ),
        speed: Vector2(
          random.nextDouble() * 200 - 100,
          random.nextDouble() * 200 - 100,
        ),
        position: position,
        child: CircleParticle(
          radius: random.nextDouble() * 5 + 2,
          paint: Paint()
            ..color = Colors.orange.withOpacity(random.nextDouble()),
        ),
      ),
    ),
  );

  add(particleSystem);
}
```

---

## 第9章 相机系统

### 9.1 基础相机

```dart
class MyGame extends FlameGame {
  late Player player;

  @override
  Future<void> onLoad() async {
    player = Player();
    add(player);

    // 设置相机跟随玩家
    camera.follow(player);
  }
}
```

### 9.2 完整相机配置

```dart
class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    final world = World();
    add(world);

    final player = Player();
    world.add(player);

    camera = CameraComponent.withFixedResolution(
      world: world,
      width: 800,
      height: 600,
    );
    add(camera);

    // 相机跟随玩家
    camera.follow(player);

    // 设置相机边界
    camera.setBounds(
      Rectangle.fromLTRB(
        0,
        0,
        worldWidth,
        worldHeight,
      ),
    );
  }
}
```

---

## 第10章 音频管理

### 10.1 基础音频

```dart
import 'package:flame_audio/flame_audio.dart';

class MyGame extends FlameGame {
  @override
  Future<void> onLoad() async {
    // 预加载音频
    await FlameAudio.audioCache.load('bgm.mp3');
    await FlameAudio.audioCache.load('shoot.wav');

    // 播放背景音乐
    FlameAudio.bgm.play('bgm.mp3');
  }

  void shoot() {
    // 播放音效
    FlameAudio.play('shoot.wav');
  }
}
```

### 10.2 音频管理器

```dart
class AudioManager {
  static final AudioManager _instance = AudioManager._internal();
  factory AudioManager() => _instance;
  AudioManager._internal();

  bool _isBgmPlaying = false;
  double _bgmVolume = 1.0;
  double _sfxVolume = 1.0;

  Future<void> init() async {
    await FlameAudio.audioCache.loadAll([
      'bgm/menu.mp3',
      'bgm/game.mp3',
      'sfx/shoot.wav',
      'sfx/explosion.wav',
      'sfx/powerup.wav',
    ]);
  }

  void playBgm(String fileName) {
    FlameAudio.bgm.play(fileName, volume: _bgmVolume);
    _isBgmPlaying = true;
  }

  void stopBgm() {
    FlameAudio.bgm.stop();
    _isBgmPlaying = false;
  }

  void playSfx(String fileName) {
    FlameAudio.play(fileName, volume: _sfxVolume);
  }

  void setBgmVolume(double volume) {
    _bgmVolume = volume;
    FlameAudio.bgm.audioPlayer.setVolume(volume);
  }

  void setSfxVolume(double volume) {
    _sfxVolume = volume;
  }
}
```

---

## 第11章 实战案例

### 11.1 太空射击游戏

```dart
import 'dart:math';
import 'package:flame/components.dart';
import 'package:flame/game.dart';
import 'package:flame/input.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(
    GameWidget(
      game: SpaceShooterGame(),
      overlayBuilderMap: {
        'gameOver': (context, game) => GameOverOverlay(
          game: game as SpaceShooterGame,
        ),
      },
    ),
  );
}

class SpaceShooterGame extends FlameGame
    with KeyboardEvents, HasCollisionDetection {
  late Player player;
  late SpawnComponent enemySpawner;
  int score = 0;
  bool isGameOver = false;

  @override
  Future<void> onLoad() async {
    // 添加背景
    add(StarBackground());

    // 创建玩家
    player = Player();
    add(player);

    // 敌人生成器
    enemySpawner = SpawnComponent(
      factory: (index) => Enemy(),
      period: 2,
      area: Rectangle.fromLTWH(0, -50, size.x, 50),
    );
    add(enemySpawner);

    // HUD
    add(ScoreDisplay());
  }

  @override
  void update(double dt) {
    super.update(dt);
    if (!isGameOver) {
      score += (dt * 10).toInt();
    }
  }

  void gameOver() {
    isGameOver = true;
    enemySpawner.timer.stop();
    overlays.add('gameOver');
  }

  void restart() {
    isGameOver = false;
    score = 0;
    overlays.remove('gameOver');
    
    player.reset();
    
    children.whereType<Enemy>().forEach((e) => e.removeFromParent());
    children.whereType<Bullet>().forEach((b) => b.removeFromParent());
    
    enemySpawner.timer.start();
  }
}

class Player extends SpriteComponent
    with CollisionCallbacks, HasGameRef<SpaceShooterGame> {
  double speed = 300;
  double shootTimer = 0;

  Player() : super(
    size: Vector2(48, 48),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('player_ship.png');
    position = Vector2(gameRef.size.x / 2, gameRef.size.y - 100);
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    // 键盘控制
    final keys = gameRef.keysPressed;
    if (keys.contains(LogicalKeyboardKey.arrowLeft)) {
      position.x -= speed * dt;
    }
    if (keys.contains(LogicalKeyboardKey.arrowRight)) {
      position.x += speed * dt;
    }
    if (keys.contains(LogicalKeyboardKey.space)) {
      shootTimer -= dt;
      if (shootTimer <= 0) {
        shoot();
        shootTimer = 0.3;
      }
    }

    // 边界限制
    position.x = position.x.clamp(size.x / 2, gameRef.size.x - size.x / 2);
  }

  void shoot() {
    gameRef.add(Bullet(position: position + Vector2(0, -30)));
  }

  void reset() {
    position = Vector2(gameRef.size.x / 2, gameRef.size.y - 100);
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    if (other is Enemy) {
      gameRef.gameOver();
    }
  }
}

class Enemy extends SpriteComponent
    with CollisionCallbacks, HasGameRef<SpaceShooterGame> {
  double speed = 100 + Random().nextDouble() * 100;

  Enemy() : super(
    size: Vector2(40, 40),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('enemy_ship.png');
    position.x = Random().nextDouble() * gameRef.size.x;
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    position.y += speed * dt;
    
    if (position.y > gameRef.size.y + 50) {
      removeFromParent();
    }
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    if (other is Bullet) {
      removeFromParent();
      other.removeFromParent();
      gameRef.score += 100;
    }
  }
}

class Bullet extends SpriteComponent with HasGameRef<SpaceShooterGame> {
  double speed = 500;

  Bullet({required Vector2 position}) : super(
    position: position,
    size: Vector2(8, 16),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    sprite = await gameRef.loadSprite('bullet.png');
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    position.y -= speed * dt;
    
    if (position.y < -50) {
      removeFromParent();
    }
  }
}

class StarBackground extends Component with HasGameRef<SpaceShooterGame> {
  final List<Star> stars = [];

  @override
  Future<void> onLoad() async {
    for (var i = 0; i < 100; i++) {
      stars.add(Star(gameRef.size));
    }
  }

  @override
  void render(Canvas canvas) {
    for (final star in stars) {
      star.render(canvas);
      star.update(0.016);
    }
  }
}

class Star {
  Vector2 position;
  double speed;
  double size;
  Vector2 gameSize;

  Star(this.gameSize)
      : position = Vector2(
          Random().nextDouble() * gameSize.x,
          Random().nextDouble() * gameSize.y,
        ),
        speed = 20 + Random().nextDouble() * 100,
        size = 1 + Random().nextDouble() * 2;

  void render(Canvas canvas) {
    canvas.drawCircle(
      Offset(position.x, position.y),
      size,
      Paint()..color = Colors.white.withOpacity(0.8),
    );
  }

  void update(double dt) {
    position.y += speed * dt;
    if (position.y > gameSize.y) {
      position.y = 0;
      position.x = Random().nextDouble() * gameSize.x;
    }
  }
}

class ScoreDisplay extends Component with HasGameRef<SpaceShooterGame> {
  late TextComponent textComponent;

  @override
  Future<void> onLoad() async {
    textComponent = TextComponent(
      text: 'Score: 0',
      position: Vector2(10, 10),
      textRenderer: TextPaint(
        style: const TextStyle(
          color: Colors.white,
          fontSize: 20,
        ),
      ),
    );
    add(textComponent);
  }

  @override
  void update(double dt) {
    textComponent.text = 'Score: ${gameRef.score}';
  }
}

class GameOverOverlay extends StatelessWidget {
  final SpaceShooterGame game;

  const GameOverOverlay({super.key, required this.game});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        padding: const EdgeInsets.all(20),
        decoration: BoxDecoration(
          color: Colors.black.withOpacity(0.8),
          borderRadius: BorderRadius.circular(10),
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'GAME OVER',
              style: TextStyle(
                color: Colors.white,
                fontSize: 40,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 20),
            Text(
              'Score: ${game.score}',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 24,
              ),
            ),
            const SizedBox(height: 20),
            ElevatedButton(
              onPressed: game.restart,
              child: const Text('RESTART'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 11.2 平台跳跃游戏

```dart
class PlatformerGame extends FlameGame
    with KeyboardEvents, HasCollisionDetection {
  late Player player;

  @override
  Future<void> onLoad() async {
    // 添加平台
    add(Platform(position: Vector2(100, 400), size: Vector2(200, 20)));
    add(Platform(position: Vector2(400, 300), size: Vector2(200, 20)));
    add(Platform(position: Vector2(700, 200), size: Vector2(200, 20)));

    // 添加地面
    add(Ground());

    // 添加玩家
    player = Player();
    add(player);
  }
}

class Player extends SpriteAnimationComponent
    with CollisionCallbacks, HasGameRef<PlatformerGame> {
  Vector2 velocity = Vector2.zero();
  double gravity = 800;
  double jumpForce = -400;
  double moveSpeed = 200;
  bool isGrounded = false;

  Player() : super(
    size: Vector2(32, 48),
    anchor: Anchor.center,
  );

  @override
  Future<void> onLoad() async {
    // 加载动画
    final spriteSheet = await gameRef.images.load('player_sheet.png');
    
    animation = SpriteAnimation.fromFrameData(
      spriteSheet,
      SpriteAnimationData.sequenced(
        amount: 4,
        amountPerRow: 4,
        textureSize: Vector2(32, 48),
        stepTime: 0.1,
      ),
    );

    position = Vector2(100, 300);
    add(RectangleHitbox());
  }

  @override
  void update(double dt) {
    // 重力
    velocity.y += gravity * dt;

    // 水平移动
    final keys = gameRef.keysPressed;
    if (keys.contains(LogicalKeyboardKey.arrowLeft)) {
      velocity.x = -moveSpeed;
      flipHorizontally();
    } else if (keys.contains(LogicalKeyboardKey.arrowRight)) {
      velocity.x = moveSpeed;
      flipHorizontally(left: false);
    } else {
      velocity.x = 0;
    }

    // 跳跃
    if (keys.contains(LogicalKeyboardKey.space) && isGrounded) {
      velocity.y = jumpForce;
      isGrounded = false;
    }

    // 应用速度
    position += velocity * dt;

    // 边界限制
    if (position.y > gameRef.size.y + 100) {
      // 掉出屏幕，重置位置
      position = Vector2(100, 300);
      velocity = Vector2.zero();
    }
  }

  @override
  void onCollisionStart(
    Set<Vector2> intersectionPoints,
    PositionComponent other,
  ) {
    if (other is Platform || other is Ground) {
      // 简单的地面检测
      if (velocity.y > 0) {
        isGrounded = true;
        velocity.y = 0;
      }
    }
  }
}

class Platform extends PositionComponent {
  Platform({required Vector2 position, required Vector2 size})
      : super(position: position, size: size);

  @override
  void render(Canvas canvas) {
    canvas.drawRect(
      size.toRect(),
      Paint()..color = Colors.brown,
    );
  }
}

class Ground extends PositionComponent {
  Ground() : super(position: Vector2(0, 550), size: Vector2(1000, 50));

  @override
  void render(Canvas canvas) {
    canvas.drawRect(
      size.toRect(),
      Paint()..color = Colors.green,
    );
  }
}
```

---

## 第12章 最佳实践

### 12.1 性能优化

```dart
// 使用对象池避免频繁创建
class BulletPool {
  final List<Bullet> _available = [];
  final List<Bullet> _inUse = [];

  Bullet getBullet() {
    if (_available.isNotEmpty) {
      final bullet = _available.removeLast();
      _inUse.add(bullet);
      return bullet;
    }
    return Bullet();
  }

  void releaseBullet(Bullet bullet) {
    _inUse.remove(bullet);
    _available.add(bullet);
  }
}

// 使用优先渲染
class Background extends Component with HasPriority {
  Background() : super(priority: -1);  // 低优先级，先渲染
}

class UI extends Component with HasPriority {
  UI() : super(priority: 100);  // 高优先级，后渲染
}
```

### 12.2 DO 和 DON'T

#### ✅ DO

- 使用 `onLoad` 异步加载资源
- 使用组件系统组织代码
- 使用对象池管理频繁创建的对象
- 及时移除不再使用的组件
- 使用 `dt` 进行帧率无关的更新

#### ❌ DON'T

- 在 `update` 中创建新对象
- 忽略 `dt` 导致帧率相关的问题
- 在 `render` 中执行复杂计算
- 忘记处理组件的生命周期
- 过度使用碰撞检测

---

## 附录：Flame API 速查表

### 核心类

| 类名 | 说明 |
|------|------|
| FlameGame | 游戏主类 |
| GameWidget | 游戏 Widget |
| Component | 组件基类 |
| PositionComponent | 位置组件 |
| SpriteComponent | 精灵组件 |
| SpriteAnimationComponent | 动画组件 |

### Mixin

| Mixin | 功能 |
|-------|------|
| TapDetector | 触摸检测 |
| PanDetector | 拖拽检测 |
| KeyboardEvents | 键盘事件 |
| CollisionCallbacks | 碰撞回调 |
| HasGameRef | 游戏引用 |

### 工具类

| 类名 | 说明 |
|------|------|
| Vector2 | 二维向量 |
| SpriteSheet | 精灵图集 |
| Timer | 计时器 |
| SpawnComponent | 生成器 |

---

## 结语

Flame 1.35.1 是一个功能强大且易于使用的 Flutter 游戏引擎。通过本书的学习，你应该已经掌握了：

1. Flame 的核心概念和游戏循环
2. 组件系统的使用
3. 精灵和动画的创建
4. 输入处理（触摸、键盘）
5. 碰撞检测
6. 粒子系统
7. 相机系统
8. 音频管理
9. 实战游戏开发

希望本书能够帮助你在 Flutter 游戏开发的道路上走得更远，创造出令人惊叹的游戏作品！

---

**版本信息**

- Flame 版本：1.35.1
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
