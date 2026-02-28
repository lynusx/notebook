# video_player 2.11.0 详解与实战

## 第1章 认识 video_player

### 1.1 什么是 video_player

`video_player` 是 Flutter 官方团队维护的视频播放器插件，提供对视频播放的低级访问。它是 Flutter 生态中最基础、最稳定的视频播放解决方案，被广泛应用于各类需要视频播放功能的 Flutter 应用中。

与其他 Flutter 视频库相比，`video_player` 的核心特点在于：

| 特性       | video_player     | 其他库   |
| ---------- | ---------------- | -------- |
| 官方维护   | ✅ Flutter 团队  | 社区维护 |
| 稳定性     | ⭐⭐⭐⭐⭐ 极高  | 较高     |
| 功能丰富度 | 基础功能         | 更丰富   |
| UI 控件    | 无（需自行实现） | 内置     |
| 学习曲线   | 较陡峭           | 较平缓   |
| 自定义程度 | 极高             | 中等     |

Flutter 框架小知识

**底层播放器实现**

`video_player` 在不同平台上使用不同的原生播放器：

| 平台      | 底层实现    | 说明                        |
| --------- | ----------- | --------------------------- |
| Android   | ExoPlayer   | Google 开源播放器，功能强大 |
| iOS/macOS | AVPlayer    | Apple 原生播放器            |
| Web       | HTML5 Video | 浏览器原生支持              |

这种设计确保了在各平台上都能获得最佳的播放性能和兼容性。

### 1.2 video_player 2.11.0 的新特性

版本 2.11.0 是 `video_player` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **多种视频源**：支持网络 URL、本地文件、Flutter 资源
- **播放控制**：播放、暂停、跳转、循环
- **播放速度**：支持 0.5x - 2.0x 变速播放
- **音量控制**：独立音量设置
- **字幕支持**：SRT 字幕文件解析和显示
- **缓冲状态**：实时获取缓冲进度
- **平台视图**：支持 PlatformView 渲染模式

#### 1.2.2 平台支持

| 平台    | 支持状态 | 说明         |
| ------- | -------- | ------------ |
| Android | ✅       | API 21+      |
| iOS     | ✅       | iOS 11.0+    |
| macOS   | ✅       | macOS 10.15+ |
| Web     | ✅       | 现代浏览器   |

**注意**：Web 平台不支持 `VideoPlayerController.file()` 方法。

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  video_player: ^2.11.0
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加网络权限：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application ...>
    </application>

    <uses-permission android:name="android.permission.INTERNET"/>
</manifest>
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

**macOS 配置**：

在 `macos/Runner/DebugProfile.entitlements` 和 `macos/Runner/Release.entitlements` 中添加：

```xml
<key>com.apple.security.network.client</key>
<true/>
```

Dart Tips 语法小贴士

**视频格式支持**

不同平台支持的视频格式有所不同：

| 格式             | Android  | iOS | Web      |
| ---------------- | -------- | --- | -------- |
| MP4 (H.264)      | ✅       | ✅  | ✅       |
| MP4 (H.265/HEVC) | ✅       | ✅  | 部分支持 |
| WebM (VP8/VP9)   | ✅       | ❌  | ✅       |
| MOV              | 部分支持 | ✅  | 部分支持 |
| MKV              | 部分支持 | ❌  | ❌       |

建议使用 MP4 (H.264) 格式以获得最佳兼容性。

## 第2章 VideoPlayerController 类详解

`VideoPlayerController` 是 `video_player` 包的核心类，负责视频播放的控制和管理。

### 2.1 构造函数

#### 2.1.1 网络视频

```dart
VideoPlayerController.networkUrl(
  Uri url, {
  VideoFormat? formatHint,           // 视频格式提示
  Future<ClosedCaptionFile>? closedCaptionFile,  // 字幕文件
  VideoPlayerOptions? videoPlayerOptions,        // 播放器选项
  Map<String, String> httpHeaders = const {},    // HTTP 请求头
})
```

**使用示例**：

```dart
// 基本用法
final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/video.mp4'),
);

// 带请求头
final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/video.mp4'),
  httpHeaders: {'Authorization': 'Bearer token'},
);

// HLS 流媒体
final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/playlist.m3u8'),
  formatHint: VideoFormat.hls,
);
```

#### 2.1.2 本地文件

```dart
VideoPlayerController.file(
  File file, {
  Future<ClosedCaptionFile>? closedCaptionFile,
  VideoPlayerOptions? videoPlayerOptions,
})
```

**使用示例**：

```dart
final file = File('/path/to/video.mp4');
final controller = VideoPlayerController.file(file);
```

#### 2.1.3 Flutter 资源

```dart
VideoPlayerController.asset(
  String dataSource, {
  String? package,                    // 包名
  Future<ClosedCaptionFile>? closedCaptionFile,
  VideoPlayerOptions? videoPlayerOptions,
})
```

**使用示例**：

```dart
// 基本用法
final controller = VideoPlayerController.asset('assets/videos/demo.mp4');

// 从依赖包加载
final controller = VideoPlayerController.asset(
  'assets/videos/demo.mp4',
  package: 'my_video_package',
);
```

#### 2.1.4 内容 URI（Android）

```dart
VideoPlayerController.contentUri(
  Uri contentUri, {
  Future<ClosedCaptionFile>? closedCaptionFile,
  VideoPlayerOptions? videoPlayerOptions,
})
```

### 2.2 VideoPlayerOptions 配置

```dart
VideoPlayerOptions({
  bool mixWithOthers = false,    // 是否与其他音频混合
  bool allowBackgroundPlayback = false,  // 是否允许后台播放
  VideoViewType? videoViewType,  // 视频视图类型
})
```

**VideoViewType 枚举**：

| 值                           | 说明             |
| ---------------------------- | ---------------- |
| `VideoViewType.textureView`  | 纹理视图（默认） |
| `VideoViewType.platformView` | 平台视图         |

**使用示例**：

```dart
final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/video.mp4'),
  videoPlayerOptions: VideoPlayerOptions(
    mixWithOthers: true,
    allowBackgroundPlayback: false,
  ),
);
```

### 2.3 核心属性详解

#### 2.3.1 播放状态属性

| 属性    | 类型               | 说明             |
| ------- | ------------------ | ---------------- |
| `value` | `VideoPlayerValue` | 播放器当前状态值 |

**VideoPlayerValue 属性**：

| 属性               | 类型                  | 说明              |
| ------------------ | --------------------- | ----------------- |
| `duration`         | `Duration?`           | 视频总时长        |
| `position`         | `Duration`            | 当前播放位置      |
| `buffered`         | `List<DurationRange>` | 已缓冲的时间范围  |
| `isInitialized`    | `bool`                | 是否已初始化      |
| `isPlaying`        | `bool`                | 是否正在播放      |
| `isLooping`        | `bool`                | 是否循环播放      |
| `isBuffering`      | `bool`                | 是否正在缓冲      |
| `volume`           | `double`              | 音量（0.0 - 1.0） |
| `playbackSpeed`    | `double`              | 播放速度          |
| `errorDescription` | `String?`             | 错误描述          |
| `size`             | `Size?`               | 视频尺寸          |
| `caption`          | `Caption?`            | 当前字幕          |
| `captionOffset`    | `Duration`            | 字幕偏移          |

#### 2.3.2 其他属性

| 属性          | 类型     | 说明       |
| ------------- | -------- | ---------- |
| `aspectRatio` | `double` | 视频宽高比 |

### 2.4 核心方法详解

#### 2.4.1 初始化方法

```dart
// 初始化控制器
Future<void> initialize()
```

**使用示例**：

```dart
final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/video.mp4'),
);

// 初始化
await controller.initialize();
```

#### 2.4.2 播放控制方法

```dart
// 播放视频
Future<void> play()

// 暂停视频
Future<void> pause()

// 跳转到指定位置
Future<void> seekTo(Duration position)

// 设置循环播放
Future<void> setLooping(bool looping)

// 设置音量
Future<void> setVolume(double volume)  // 0.0 - 1.0

// 设置播放速度
Future<void> setPlaybackSpeed(double speed)  // 0.5 - 2.0
```

**使用示例**：

```dart
// 播放
await controller.play();

// 暂停
await controller.pause();

// 跳转到 30 秒处
await controller.seekTo(Duration(seconds: 30));

// 设置循环
await controller.setLooping(true);

// 设置音量 50%
await controller.setVolume(0.5);

// 设置 1.5 倍速
await controller.setPlaybackSpeed(1.5);
```

#### 2.4.3 字幕方法

```dart
// 设置字幕偏移
Future<void> setCaptionOffset(Duration offset)
```

#### 2.4.4 资源释放方法

```dart
// 释放控制器资源
Future<void> dispose()
```

**重要**：在不再使用控制器时，必须调用 `dispose()` 方法释放资源。

### 2.5 事件监听

#### 2.5.1 添加监听器

```dart
// 添加状态变化监听器
void addListener(VoidCallback listener)

// 移除监听器
void removeListener(VoidCallback listener)
```

**使用示例**：

```dart
controller.addListener(() {
  if (controller.value.isInitialized) {
    print('视频已初始化');
    print('时长: ${controller.value.duration}');
    print('当前位置: ${controller.value.position}');
    print('是否播放中: ${controller.value.isPlaying}');
  }
});
```

## 第3章 VideoPlayer Widget 详解

`VideoPlayer` 是用于显示视频的 Widget。

### 3.1 构造函数

```dart
VideoPlayer(
  VideoPlayerController controller,  // 视频控制器
)
```

### 3.2 使用示例

```dart
VideoPlayer(controller)
```

**注意**：`VideoPlayer` 本身没有固定尺寸，通常需要配合 `AspectRatio` 或 `Container` 使用。

## 第4章 字幕支持

### 4.1 字幕文件格式

`video_player` 支持 SRT 格式的字幕文件。

### 4.2 SubRipCaptionFile 类

```dart
SubRipCaptionFile(String fileContents)
```

**使用示例**：

```dart
// 加载字幕文件
final subtitleFile = await rootBundle.loadString('assets/subtitles.srt');

final controller = VideoPlayerController.networkUrl(
  Uri.parse('https://example.com/video.mp4'),
  closedCaptionFile: SubRipCaptionFile(subtitleFile),
);
```

### 4.3 ClosedCaption Widget

```dart
ClosedCaption({
  TextStyle? textStyle,    // 字幕样式
  String? text,            // 字幕文本
})
```

**使用示例**：

```dart
Stack(
  children: [
    VideoPlayer(controller),
    ClosedCaption(
      text: controller.value.caption?.text,
      textStyle: TextStyle(
        color: Colors.white,
        fontSize: 16,
        backgroundColor: Colors.black54,
      ),
    ),
  ],
)
```

## 第5章 video_player 实战应用

### 5.1 基础视频播放器

```dart
class BasicVideoPlayer extends StatefulWidget {
  @override
  _BasicVideoPlayerState createState() => _BasicVideoPlayerState();
}

class _BasicVideoPlayerState extends State<BasicVideoPlayer> {
  late VideoPlayerController _controller;
  late Future<void> _initializeVideoPlayerFuture;

  @override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.networkUrl(
      Uri.parse(
        'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
      ),
    );
    _initializeVideoPlayerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('基础视频播放器')),
      body: FutureBuilder(
        future: _initializeVideoPlayerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            return AspectRatio(
              aspectRatio: _controller.value.aspectRatio,
              child: VideoPlayer(_controller),
            );
          } else {
            return Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            if (_controller.value.isPlaying) {
              _controller.pause();
            } else {
              _controller.play();
            }
          });
        },
        child: Icon(
          _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
        ),
      ),
    );
  }
}
```

### 5.2 完整视频播放器（带控制条）

```dart
class FullVideoPlayer extends StatefulWidget {
  final String videoUrl;

  const FullVideoPlayer({super.key, required this.videoUrl});

  @override
  _FullVideoPlayerState createState() => _FullVideoPlayerState();
}

class _FullVideoPlayerState extends State<FullVideoPlayer> {
  late VideoPlayerController _controller;
  bool _isPlaying = false;
  bool _showControls = true;
  Timer? _hideControlsTimer;

  @override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.networkUrl(Uri.parse(widget.videoUrl))
      ..initialize().then((_) {
        setState(() {});
      })
      ..addListener(_onControllerUpdate);
  }

  void _onControllerUpdate() {
    if (mounted) {
      setState(() {
        _isPlaying = _controller.value.isPlaying;
      });
    }
  }

  void _showControlsTemporarily() {
    setState(() {
      _showControls = true;
    });
    _hideControlsTimer?.cancel();
    _hideControlsTimer = Timer(Duration(seconds: 3), () {
      if (mounted && _controller.value.isPlaying) {
        setState(() {
          _showControls = false;
        });
      }
    });
  }

  String _formatDuration(Duration duration) {
    String twoDigits(int n) => n.toString().padLeft(2, '0');
    final hours = twoDigits(duration.inHours);
    final minutes = twoDigits(duration.inMinutes.remainder(60));
    final seconds = twoDigits(duration.inSeconds.remainder(60));
    return duration.inHours > 0 ? '$hours:$minutes:$seconds' : '$minutes:$seconds';
  }

  @override
  void dispose() {
    _hideControlsTimer?.cancel();
    _controller.removeListener(_onControllerUpdate);
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: _controller.value.isInitialized
          ? GestureDetector(
              onTap: _showControlsTemporarily,
              child: Stack(
                alignment: Alignment.center,
                children: [
                  // 视频
                  AspectRatio(
                    aspectRatio: _controller.value.aspectRatio,
                    child: VideoPlayer(_controller),
                  ),

                  // 控制层
                  if (_showControls)
                    Container(
                      color: Colors.black26,
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.end,
                        children: [
                          // 进度条
                          VideoProgressIndicator(
                            _controller,
                            allowScrubbing: true,
                            colors: VideoProgressColors(
                              playedColor: Colors.red,
                              bufferedColor: Colors.grey,
                              backgroundColor: Colors.white24,
                            ),
                          ),

                          // 控制按钮
                          Padding(
                            padding: EdgeInsets.all(16),
                            child: Row(
                              children: [
                                // 播放/暂停
                                IconButton(
                                  icon: Icon(
                                    _isPlaying ? Icons.pause : Icons.play_arrow,
                                    color: Colors.white,
                                    size: 32,
                                  ),
                                  onPressed: () {
                                    setState(() {
                                      if (_isPlaying) {
                                        _controller.pause();
                                      } else {
                                        _controller.play();
                                      }
                                    });
                                  },
                                ),

                                // 时间显示
                                Text(
                                  '${_formatDuration(_controller.value.position)} / ${_formatDuration(_controller.value.duration ?? Duration.zero)}',
                                  style: TextStyle(color: Colors.white),
                                ),

                                Spacer(),

                                // 全屏按钮
                                IconButton(
                                  icon: Icon(Icons.fullscreen, color: Colors.white),
                                  onPressed: () {
                                    // 进入全屏
                                  },
                                ),
                              ],
                            ),
                          ),
                        ],
                      ),
                    ),
                ],
              ),
            )
          : Center(child: CircularProgressIndicator()),
    );
  }
}
```

### 5.3 带字幕的视频播放器

```dart
class SubtitleVideoPlayer extends StatefulWidget {
  @override
  _SubtitleVideoPlayerState createState() => _SubtitleVideoPlayerState();
}

class _SubtitleVideoPlayerState extends State<SubtitleVideoPlayer> {
  late VideoPlayerController _controller;

  @override
  void initState() {
    super.initState();
    _loadVideoWithSubtitles();
  }

  Future<void> _loadVideoWithSubtitles() async {
    final subtitleContent = await rootBundle.loadString('assets/subtitles.srt');

    _controller = VideoPlayerController.networkUrl(
      Uri.parse('https://example.com/video.mp4'),
      closedCaptionFile: SubRipCaptionFile(subtitleContent),
    );

    await _controller.initialize();
    setState(() {});
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('带字幕的视频播放器')),
      body: _controller.value.isInitialized
          ? Stack(
              alignment: Alignment.bottomCenter,
              children: [
                VideoPlayer(_controller),
                ClosedCaption(
                  text: _controller.value.caption?.text ?? '',
                  textStyle: TextStyle(
                    fontSize: 16,
                    color: Colors.white,
                    backgroundColor: Colors.black54,
                  ),
                ),
              ],
            )
          : Center(child: CircularProgressIndicator()),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            if (_controller.value.isPlaying) {
              _controller.pause();
            } else {
              _controller.play();
            }
          });
        },
        child: Icon(
          _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
        ),
      ),
    );
  }
}
```

### 5.4 列表视频播放器

```dart
class VideoListPlayer extends StatefulWidget {
  @override
  _VideoListPlayerState createState() => _VideoListPlayerState();
}

class _VideoListPlayerState extends State<VideoListPlayer> {
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
  late VideoPlayerController _controller;
  bool _isInitialized = false;

  @override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.networkUrl(Uri.parse(widget.videoUrl))
      ..initialize().then((_) {
        setState(() {
          _isInitialized = true;
        });
      });
  }

  @override
  void dispose() {
    _controller.dispose();
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
                ? Stack(
                    alignment: Alignment.center,
                    children: [
                      VideoPlayer(_controller),
                      IconButton(
                        icon: Icon(
                          _controller.value.isPlaying
                              ? Icons.pause
                              : Icons.play_arrow,
                          size: 48,
                          color: Colors.white,
                        ),
                        onPressed: () {
                          setState(() {
                            if (_controller.value.isPlaying) {
                              _controller.pause();
                            } else {
                              _controller.play();
                            }
                          });
                        },
                      ),
                    ],
                  )
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

## 附录 A：VideoPlayerController 方法速查表

| 方法                 | 返回值         | 说明           |
| -------------------- | -------------- | -------------- |
| `initialize()`       | `Future<void>` | 初始化控制器   |
| `play()`             | `Future<void>` | 播放视频       |
| `pause()`            | `Future<void>` | 暂停视频       |
| `seekTo()`           | `Future<void>` | 跳转到指定位置 |
| `setLooping()`       | `Future<void>` | 设置循环播放   |
| `setVolume()`        | `Future<void>` | 设置音量       |
| `setPlaybackSpeed()` | `Future<void>` | 设置播放速度   |
| `setCaptionOffset()` | `Future<void>` | 设置字幕偏移   |
| `dispose()`          | `Future<void>` | 释放资源       |

## 附录 B：VideoPlayerValue 属性速查表

| 属性            | 类型                  | 说明         |
| --------------- | --------------------- | ------------ |
| `duration`      | `Duration?`           | 总时长       |
| `position`      | `Duration`            | 当前位置     |
| `buffered`      | `List<DurationRange>` | 缓冲范围     |
| `isInitialized` | `bool`                | 是否已初始化 |
| `isPlaying`     | `bool`                | 是否播放中   |
| `isLooping`     | `bool`                | 是否循环     |
| `isBuffering`   | `bool`                | 是否缓冲中   |
| `volume`        | `double`              | 音量         |
| `playbackSpeed` | `double`              | 播放速度     |
| `size`          | `Size?`               | 视频尺寸     |
| `caption`       | `Caption?`            | 当前字幕     |

## 附录 C：常见问题与解决方案

### Q1：视频无法播放

**检查项**：

1. 网络权限是否配置
2. 视频格式是否支持
3. URL 是否可访问

### Q2：iOS 模拟器无法播放

**解决方案**：
iOS 模拟器对视频播放有限制，建议在真机上测试。

### Q3：Web 平台文件播放失败

**解决方案**：
Web 平台不支持 `VideoPlayerController.file()`，请使用网络 URL。

## 附录 D：参考资源

- [video_player 官方文档](https://pub.dev/packages/video_player)
- [Flutter 视频播放指南](https://docs.flutter.dev/cookbook/plugins/play-video)
- [ExoPlayer 文档](https://exoplayer.dev/)
- [AVPlayer 文档](https://developer.apple.com/documentation/avfoundation/avplayer)
