# chewie 1.13.0 详解与实战

## 第1章 认识 chewie

### 1.1 什么是 chewie

`chewie` 是一个基于 `video_player` 的 Flutter 视频播放器 UI 库，由 Flutter Community 维护。它为 `video_player` 提供了开箱即用的 Material Design 和 Cupertino 风格的播放控制界面，让开发者无需自行实现复杂的播放器 UI。

与直接使用 `video_player` 相比，`chewie` 的核心优势在于：

| 特性       | chewie            | 直接使用 video_player |
| ---------- | ----------------- | --------------------- |
| UI 控件    | ✅ 内置完整控制条 | ❌ 需自行实现         |
| 开发效率   | ⭐⭐⭐⭐⭐ 极高   | 较低                  |
| 自定义程度 | 高                | 极高                  |
| 代码量     | 极少              | 较多                  |
| 维护成本   | 低                | 高                    |
| 功能丰富度 | 丰富              | 基础                  |

Flutter 框架小知识

**Chewie 的设计理念**

Chewie 的设计哲学是：

> "The video player for Flutter with a heart of gold."

Chewie 在 `video_player` 之上提供了一层友好的 UI 封装：

- **MaterialControls**：Android 风格的控制界面
- **MaterialDesktopControls**：桌面端风格的控制界面
- **CupertinoControls**：iOS 风格的控制界面

Chewie 不处理底层播放逻辑，所有播放相关的问题都应向 `video_player` 团队反馈。

### 1.2 chewie 1.13.0 的新特性

版本 1.13.0 是 `chewie` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **开箱即用的 UI**：完整的播放控制条
- **多平台适配**：Material、Cupertino、Desktop 风格
- **全屏播放**：支持点击全屏按钮进入全屏模式
- **播放速度控制**：支持多种播放速度切换
- **字幕支持**：内置字幕显示功能
- **自定义选项**：丰富的自定义配置选项
- **进度条拖拽**：支持拖动进度条跳转
- **音量控制**：支持音量调节
- **亮度控制**：支持亮度调节（iOS）

#### 1.2.2 平台支持

| 平台    | 支持状态 |
| ------- | -------- |
| Android | ✅       |
| iOS     | ✅       |
| macOS   | ✅       |
| Windows | ✅       |
| Linux   | ✅       |
| Web     | ✅       |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  video_player: ^2.11.0
  chewie: ^1.13.0
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

Chewie 依赖 `video_player`，因此需要配置与 `video_player` 相同的平台权限。

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加：

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

Dart Tips 语法小贴士

**依赖版本兼容性**

Chewie 与 video_player 的版本兼容性很重要。如果遇到兼容性问题，可以尝试：

```yaml
dependencies:
  video_player: ^2.11.0
  chewie: ^1.13.0
```

确保两个包的版本都是最新的稳定版本。

## 第2章 ChewieController 类详解

`ChewieController` 是 `chewie` 包的核心类，负责配置和控制视频播放器的 UI 行为。

### 2.1 构造函数

```dart
ChewieController({
  required VideoPlayerController videoPlayerController,  // 视频控制器（必填）
  bool autoInitialize = false,                            // 自动初始化
  bool autoPlay = false,                                  // 自动播放
  Duration? startAt,                                      // 起始播放位置
  bool looping = false,                                   // 循环播放
  bool fullScreenByDefault = false,                       // 默认全屏
  double? aspectRatio,                                    // 宽高比
  Iterable<Duration>? cuePoints,                          // 提示点
  bool showControlsOnInitialize = true,                   // 初始化时显示控制条
  bool showOptions = true,                                // 显示选项菜单
  bool showControls = true,                               // 显示控制条
  List<double>? playbackSpeeds,                           // 播放速度选项
  List<SystemUiOverlay>? systemOverlaysAfterFullScreen,   // 全屏后系统 UI
  List<DeviceOrientation>? deviceOrientationsAfterFullScreen, // 全屏后方向
  ChewieProgressColors? materialProgressColors,           // Material 进度条颜色
  ChewieProgressColors? cupertinoProgressColors,          // Cupertino 进度条颜色
  Widget? placeholder,                                    // 占位图
  Widget? overlay,                                        // 覆盖层
  bool showSubtitles = false,                             // 显示字幕
  Subtitles? subtitle,                                    // 字幕数据
  Widget Function(BuildContext, dynamic)? errorBuilder,   // 错误构建器
  Widget Function(BuildContext, String, String)? routePageBuilder, // 路由构建器
  Duration? hideControlsTimer,                            // 隐藏控制条计时器
  bool draggableProgressBar = true,                       // 可拖拽进度条
  bool useRootNavigator = true,                           // 使用根导航器
  bool allowedScreenSleep = true,                         // 允许屏幕休眠
  bool zoomAndPan = false,                                // 缩放和平移
  double maxScale = 2.5,                                  // 最大缩放比例
  bool controlsSafeAreaMinimum = false,                   // 控制条安全区域最小值
  // 回调函数
  void Function()? onPlay,                                // 播放回调
  void Function()? onPause,                               // 暂停回调
  void Function()? onShowControls,                        // 显示控制条回调
  void Function()? onHideControls,                        // 隐藏控制条回调
})
```

### 2.2 核心参数详解

#### 2.2.1 必需参数

| 参数                    | 类型                    | 说明                     |
| ----------------------- | ----------------------- | ------------------------ |
| `videoPlayerController` | `VideoPlayerController` | 视频播放器控制器（必填） |

#### 2.2.2 播放行为参数

| 参数                  | 类型            | 默认值                                     | 说明           |
| --------------------- | --------------- | ------------------------------------------ | -------------- |
| `autoInitialize`      | `bool`          | `false`                                    | 是否自动初始化 |
| `autoPlay`            | `bool`          | `false`                                    | 是否自动播放   |
| `startAt`             | `Duration?`     | `null`                                     | 起始播放位置   |
| `looping`             | `bool`          | `false`                                    | 是否循环播放   |
| `fullScreenByDefault` | `bool`          | `false`                                    | 是否默认全屏   |
| `playbackSpeeds`      | `List<double>?` | `[0.25, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2]` | 播放速度选项   |

#### 2.2.3 UI 显示参数

| 参数                       | 类型      | 默认值 | 说明                   |
| -------------------------- | --------- | ------ | ---------------------- |
| `showControls`             | `bool`    | `true` | 是否显示控制条         |
| `showControlsOnInitialize` | `bool`    | `true` | 初始化时是否显示控制条 |
| `showOptions`              | `bool`    | `true` | 是否显示选项菜单       |
| `placeholder`              | `Widget?` | `null` | 视频加载前的占位图     |
| `overlay`                  | `Widget?` | `null` | 视频上方的覆盖层       |

#### 2.2.4 样式参数

| 参数                      | 类型                    | 默认值 | 说明                 |
| ------------------------- | ----------------------- | ------ | -------------------- |
| `aspectRatio`             | `double?`               | `null` | 视频宽高比           |
| `materialProgressColors`  | `ChewieProgressColors?` | `null` | Material 进度条颜色  |
| `cupertinoProgressColors` | `ChewieProgressColors?` | `null` | Cupertino 进度条颜色 |

#### 2.2.5 字幕参数

| 参数            | 类型         | 默认值  | 说明         |
| --------------- | ------------ | ------- | ------------ |
| `showSubtitles` | `bool`       | `false` | 是否显示字幕 |
| `subtitle`      | `Subtitles?` | `null`  | 字幕数据     |

#### 2.2.6 全屏参数

| 参数                                | 类型                       | 默认值                           | 说明                    |
| ----------------------------------- | -------------------------- | -------------------------------- | ----------------------- |
| `systemOverlaysAfterFullScreen`     | `List<SystemUiOverlay>?`   | `SystemUiOverlay.values`         | 退出全屏后显示的系统 UI |
| `deviceOrientationsAfterFullScreen` | `List<DeviceOrientation>?` | `[DeviceOrientation.portraitUp]` | 退出全屏后的屏幕方向    |
| `useRootNavigator`                  | `bool`                     | `true`                           | 是否使用根导航器        |

#### 2.2.7 交互参数

| 参数                   | 类型     | 默认值  | 说明               |
| ---------------------- | -------- | ------- | ------------------ |
| `draggableProgressBar` | `bool`   | `true`  | 进度条是否可拖拽   |
| `zoomAndPan`           | `bool`   | `false` | 是否支持缩放和平移 |
| `maxScale`             | `double` | `2.5`   | 最大缩放比例       |

#### 2.2.8 回调函数

| 参数             | 类型            | 说明             |
| ---------------- | --------------- | ---------------- |
| `onPlay`         | `VoidCallback?` | 播放时回调       |
| `onPause`        | `VoidCallback?` | 暂停时回调       |
| `onShowControls` | `VoidCallback?` | 显示控制条时回调 |
| `onHideControls` | `VoidCallback?` | 隐藏控制条时回调 |

### 2.3 核心方法详解

#### 2.3.1 进入/退出全屏

```dart
// 进入全屏
void enterFullScreen()

// 退出全屏
void exitFullScreen()

// 切换全屏状态
void toggleFullScreen()
```

**使用示例**：

```dart
// 进入全屏
chewieController.enterFullScreen();

// 退出全屏
chewieController.exitFullScreen();

// 切换全屏
chewieController.toggleFullScreen();
```

#### 2.3.2 播放控制

```dart
// 播放
void play()

// 暂停
void pause()

// 跳转到指定位置
void seekTo(Duration position)

// 设置播放速度
void setPlaybackSpeed(double speed)
```

**使用示例**：

```dart
// 播放
chewieController.play();

// 暂停
chewieController.pause();

// 跳转到 30 秒
chewieController.seekTo(Duration(seconds: 30));

// 设置 1.5 倍速
chewieController.setPlaybackSpeed(1.5);
```

#### 2.3.3 资源释放

```dart
// 释放控制器
void dispose()
```

**重要**：在不再使用 ChewieController 时，必须调用 `dispose()` 方法释放资源。

## 第3章 Chewie Widget 详解

`Chewie` 是用于显示带 UI 控制的视频播放器的 Widget。

### 3.1 构造函数

```dart
Chewie({
  required ChewieController controller,  // Chewie 控制器（必填）
})
```

### 3.2 使用示例

```dart
Chewie(
  controller: chewieController,
)
```

## 第4章 ChewieProgressColors 类详解

`ChewieProgressColors` 用于自定义进度条的颜色。

### 4.1 构造函数

```dart
ChewieProgressColors({
  Color playedColor = const Color.fromRGBO(255, 0, 0, 0.7),      // 已播放部分颜色
  Color bufferedColor = const Color.fromRGBO(50, 50, 200, 0.2),  // 已缓冲部分颜色
  Color handleColor = const Color.fromRGBO(200, 200, 200, 1.0),  // 拖拽手柄颜色
  Color backgroundColor = const Color.fromRGBO(200, 200, 200, 0.5), // 背景颜色
})
```

### 4.2 使用示例

```dart
final chewieController = ChewieController(
  videoPlayerController: videoController,
  materialProgressColors: ChewieProgressColors(
    playedColor: Colors.red,
    handleColor: Colors.redAccent,
    bufferedColor: Colors.white,
    backgroundColor: Colors.grey,
  ),
);
```

## 第5章 Subtitles 类详解

`Subtitles` 用于配置视频字幕。

### 5.1 构造函数

```dart
Subtitles(List<Subtitle> subtitles)
```

### 5.2 Subtitle 类

```dart
Subtitle({
  required int index,           // 字幕索引
  required Duration start,      // 开始时间
  required Duration end,        // 结束时间
  required String text,         // 字幕文本
})
```

### 5.3 使用示例

```dart
final chewieController = ChewieController(
  videoPlayerController: videoController,
  showSubtitles: true,
  subtitle: Subtitles([
    Subtitle(
      index: 0,
      start: Duration(seconds: 0),
      end: Duration(seconds: 5),
      text: 'Hello, world!',
    ),
    Subtitle(
      index: 1,
      start: Duration(seconds: 5),
      end: Duration(seconds: 10),
      text: 'How are you?',
    ),
  ]),
);
```

## 第6章 chewie 实战应用

### 6.1 基础 Chewie 播放器

```dart
class BasicChewiePlayer extends StatefulWidget {
  @override
  _BasicChewiePlayerState createState() => _BasicChewiePlayerState();
}

class _BasicChewiePlayerState extends State<BasicChewiePlayer> {
  late VideoPlayerController _videoPlayerController;
  late ChewieController _chewieController;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse(
        'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
      ),
    );

    await _videoPlayerController.initialize();

    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController,
      autoPlay: true,
      looping: true,
    );

    setState(() {});
  }

  @override
  void dispose() {
    _videoPlayerController.dispose();
    _chewieController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Chewie 播放器')),
      body: _chewieController.videoPlayerController.value.isInitialized
          ? Chewie(controller: _chewieController)
          : Center(child: CircularProgressIndicator()),
    );
  }
}
```

### 6.2 自定义样式的 Chewie 播放器

```dart
class StyledChewiePlayer extends StatefulWidget {
  @override
  _StyledChewiePlayerState createState() => _StyledChewiePlayerState();
}

class _StyledChewiePlayerState extends State<StyledChewiePlayer> {
  late VideoPlayerController _videoPlayerController;
  late ChewieController _chewieController;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse('https://example.com/video.mp4'),
    );

    await _videoPlayerController.initialize();

    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController,
      autoPlay: false,
      looping: false,
      aspectRatio: 16 / 9,
      placeholder: Container(
        color: Colors.black,
        child: Center(
          child: CircularProgressIndicator(),
        ),
      ),
      materialProgressColors: ChewieProgressColors(
        playedColor: Colors.red,
        handleColor: Colors.redAccent,
        bufferedColor: Colors.white54,
        backgroundColor: Colors.grey,
      ),
      cupertinoProgressColors: ChewieProgressColors(
        playedColor: Colors.blue,
        handleColor: Colors.blueAccent,
        bufferedColor: Colors.white54,
        backgroundColor: Colors.grey,
      ),
      playbackSpeeds: [0.5, 1.0, 1.5, 2.0],
    );

    setState(() {});
  }

  @override
  void dispose() {
    _videoPlayerController.dispose();
    _chewieController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('自定义样式播放器')),
      body: _chewieController.videoPlayerController.value.isInitialized
          ? Chewie(controller: _chewieController)
          : Center(child: CircularProgressIndicator()),
    );
  }
}
```

### 6.3 带字幕的 Chewie 播放器

```dart
class SubtitleChewiePlayer extends StatefulWidget {
  @override
  _SubtitleChewiePlayerState createState() => _SubtitleChewiePlayerState();
}

class _SubtitleChewiePlayerState extends State<SubtitleChewiePlayer> {
  late VideoPlayerController _videoPlayerController;
  late ChewieController _chewieController;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse('https://example.com/video.mp4'),
    );

    await _videoPlayerController.initialize();

    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController,
      autoPlay: true,
      showSubtitles: true,
      subtitle: Subtitles([
        Subtitle(
          index: 0,
          start: Duration(seconds: 0),
          end: Duration(seconds: 4),
          text: 'Welcome to Flutter!',
        ),
        Subtitle(
          index: 1,
          start: Duration(seconds: 4),
          end: Duration(seconds: 8),
          text: 'This is a subtitle example.',
        ),
        Subtitle(
          index: 2,
          start: Duration(seconds: 8),
          end: Duration(seconds: 12),
          text: 'Chewie makes video playback easy.',
        ),
      ]),
    );

    setState(() {});
  }

  @override
  void dispose() {
    _videoPlayerController.dispose();
    _chewieController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('带字幕播放器')),
      body: _chewieController.videoPlayerController.value.isInitialized
          ? Chewie(controller: _chewieController)
          : Center(child: CircularProgressIndicator()),
    );
  }
}
```

### 6.4 全屏视频播放器

```dart
class FullscreenChewiePlayer extends StatefulWidget {
  final String videoUrl;

  const FullscreenChewiePlayer({super.key, required this.videoUrl});

  @override
  _FullscreenChewiePlayerState createState() => _FullscreenChewiePlayerState();
}

class _FullscreenChewiePlayerState extends State<FullscreenChewiePlayer> {
  late VideoPlayerController _videoPlayerController;
  late ChewieController _chewieController;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse(widget.videoUrl),
    );

    await _videoPlayerController.initialize();

    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController,
      autoPlay: true,
      fullScreenByDefault: true,  // 默认全屏
      allowMuting: true,
      allowPlaybackSpeedChanging: true,
      showControlsOnInitialize: false,
      systemOverlaysAfterFullScreen: SystemUiOverlay.values,
      deviceOrientationsAfterFullScreen: [
        DeviceOrientation.portraitUp,
        DeviceOrientation.portraitDown,
      ],
    );

    setState(() {});
  }

  @override
  void dispose() {
    _videoPlayerController.dispose();
    _chewieController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: _chewieController.videoPlayerController.value.isInitialized
          ? Chewie(controller: _chewieController)
          : Center(
              child: CircularProgressIndicator(
                color: Colors.white,
              ),
            ),
    );
  }
}
```

### 6.5 列表中的 Chewie 播放器

```dart
class ChewieListPlayer extends StatefulWidget {
  @override
  _ChewieListPlayerState createState() => _ChewieListPlayerState();
}

class _ChewieListPlayerState extends State<ChewieListPlayer> {
  final List<String> videoUrls = [
    'https://example.com/video1.mp4',
    'https://example.com/video2.mp4',
    'https://example.com/video3.mp4',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('视频列表')),
      body: ListView.builder(
        itemCount: videoUrls.length,
        itemBuilder: (context, index) {
          return VideoListItem(videoUrl: videoUrls[index]);
        },
      ),
    );
  }
}

class VideoListItem extends StatefulWidget {
  final String videoUrl;

  const VideoListItem({super.key, required this.videoUrl});

  @override
  _VideoListItemState createState() => _VideoListItemState();
}

class _VideoListItemState extends State<VideoListItem> {
  VideoPlayerController? _videoPlayerController;
  ChewieController? _chewieController;
  bool _isInitialized = false;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    _videoPlayerController = VideoPlayerController.networkUrl(
      Uri.parse(widget.videoUrl),
    );

    await _videoPlayerController!.initialize();

    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController!,
      autoPlay: false,
      showControls: true,
      customControls: MaterialControls(
        showPlayButton: true,
      ),
    );

    setState(() {
      _isInitialized = true;
    });
  }

  @override
  void dispose() {
    _videoPlayerController?.dispose();
    _chewieController?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(8),
      child: Column(
        children: [
          AspectRatio(
            aspectRatio: 16 / 9,
            child: _isInitialized
                ? Chewie(controller: _chewieController!)
                : Center(child: CircularProgressIndicator()),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Text('视频 ${widget.videoUrl.hashCode}'),
          ),
        ],
      ),
    );
  }
}
```

## 附录 A：ChewieController 参数速查表

| 参数                    | 类型                    | 默认值                                     | 说明         |
| ----------------------- | ----------------------- | ------------------------------------------ | ------------ |
| `videoPlayerController` | `VideoPlayerController` | 必填                                       | 视频控制器   |
| `autoInitialize`        | `bool`                  | `false`                                    | 自动初始化   |
| `autoPlay`              | `bool`                  | `false`                                    | 自动播放     |
| `startAt`               | `Duration?`             | `null`                                     | 起始位置     |
| `looping`               | `bool`                  | `false`                                    | 循环播放     |
| `fullScreenByDefault`   | `bool`                  | `false`                                    | 默认全屏     |
| `aspectRatio`           | `double?`               | `null`                                     | 宽高比       |
| `showControls`          | `bool`                  | `true`                                     | 显示控制条   |
| `showOptions`           | `bool`                  | `true`                                     | 显示选项菜单 |
| `playbackSpeeds`        | `List<double>?`         | `[0.25, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2]` | 播放速度     |
| `placeholder`           | `Widget?`               | `null`                                     | 占位图       |
| `overlay`               | `Widget?`               | `null`                                     | 覆盖层       |
| `showSubtitles`         | `bool`                  | `false`                                    | 显示字幕     |
| `subtitle`              | `Subtitles?`            | `null`                                     | 字幕数据     |
| `draggableProgressBar`  | `bool`                  | `true`                                     | 可拖拽进度条 |
| `zoomAndPan`            | `bool`                  | `false`                                    | 缩放和平移   |

## 附录 B：ChewieController 方法速查表

| 方法                 | 说明         |
| -------------------- | ------------ |
| `play()`             | 播放         |
| `pause()`            | 暂停         |
| `seekTo()`           | 跳转         |
| `setPlaybackSpeed()` | 设置播放速度 |
| `enterFullScreen()`  | 进入全屏     |
| `exitFullScreen()`   | 退出全屏     |
| `toggleFullScreen()` | 切换全屏     |
| `dispose()`          | 释放资源     |

## 附录 C：常见问题与解决方案

### Q1：Android 缓冲状态显示不正确

**问题**：缓冲状态不更新，导致控制条被隐藏。

**解决方案**：

```dart
_chewieController = ChewieController(
  videoPlayerController: _videoPlayerController,
  // 延迟显示加载指示器
  progressIndicatorDelay: Platform.isAndroid
      ? const Duration(days: 1)  // 禁用加载指示器
      : null,
);
```

### Q2：控制条位置偏移

**问题**：在某些布局中控制条位置不正确。

**解决方案**：

```dart
Chewie(
  controller: _chewieController,
)
// 避免使用 SafeArea 包裹 Chewie
```

### Q3：iOS 模拟器无法播放

**问题**：iOS 模拟器不支持视频播放。

**解决方案**：
在真机上测试视频播放功能。

## 附录 D：参考资源

- [chewie 官方文档](https://pub.dev/packages/chewie)
- [Flutter Community](https://github.com/fluttercommunity)
- [video_player 文档](https://pub.dev/packages/video_player)
