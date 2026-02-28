# media_kit 1.2.6 详解与实战

## 第1章 认识 media_kit

### 1.1 什么是 media_kit

`media_kit` 是一个全面的跨平台媒体播放库，由 Hitesh Kumar Saini 开发，为 Flutter 和 Dart 应用提供统一的视频和音频播放能力。它支持 6 个主要平台，通过 libmpv（桌面和移动端）和 hls.js（Web）提供强大的多媒体功能。

与其他 Flutter 视频库相比，`media_kit` 的核心优势在于：

| 特性         | media_kit           | video_player | chewie            |
| ------------ | ------------------- | ------------ | ----------------- |
| 跨平台       | ⭐⭐⭐⭐⭐ 6 平台   | 4 平台       | 6 平台            |
| 编解码器支持 | ⭐⭐⭐⭐⭐ 丰富     | 有限         | 依赖 video_player |
| 硬件加速     | ✅ 支持             | 有限         | 有限              |
| 播放格式     | ⭐⭐⭐⭐⭐ 几乎所有 | 常见格式     | 常见格式          |
| 包大小       | 较大                | 小           | 中等              |
| 学习曲线     | 中等                | 较陡峭       | 平缓              |

Flutter 框架小知识

**libmpv 简介**

`media_kit` 在桌面和移动端使用 libmpv 作为底层播放器：

- **libmpv** 是 mpv 播放器的库版本
- **mpv** 是基于 MPlayer/mplayer2 的自由开源媒体播放器
- 支持几乎所有视频和音频格式
- 提供硬件加速解码
- 支持高级功能如滤镜、着色器等

在 Web 平台上，`media_kit` 使用 hls.js 处理 HLS 流媒体播放。

### 1.2 media_kit 1.2.6 的新特性

版本 1.2.6 是 `media_kit` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **统一 API**：跨平台一致的 Dart API
- **硬件加速**：支持 GPU 硬件解码
- **丰富格式支持**：通过 libmpv 支持几乎所有媒体格式
- **视频渲染**：内置 Video 控件
- **音频播放**：完整的音频播放支持
- **播放列表**：支持播放列表管理
- **字幕支持**：内置字幕渲染
- **截图功能**：支持视频截图
- **音频可视化**：支持音频波形显示

#### 1.2.2 平台支持

| 平台    | 底层实现 | 最低版本     |
| ------- | -------- | ------------ |
| Android | libmpv   | API 21+      |
| iOS     | libmpv   | iOS 13.0+    |
| macOS   | libmpv   | macOS 10.15+ |
| Windows | libmpv   | Windows 10+  |
| Linux   | libmpv   | 任意版本     |
| Web     | hls.js   | 现代浏览器   |

#### 1.2.3 支持的格式

| 格式              | 支持状态 |
| ----------------- | -------- |
| MP4 (H.264/H.265) | ✅       |
| MKV               | ✅       |
| AVI               | ✅       |
| MOV               | ✅       |
| WebM              | ✅       |
| FLV               | ✅       |
| MPEG              | ✅       |
| HLS (m3u8)        | ✅       |
| DASH              | ✅       |
| RTMP              | ✅       |

### 1.3 安装与配置

#### 1.3.1 添加依赖

`media_kit` 采用多包架构，需要添加多个依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter

  # 核心包（必需）
  media_kit: ^1.2.6

  # 视频渲染（视频播放必需）
  media_kit_video: ^1.2.6

  # 原生库（必需）
  media_kit_libs_video: ^1.2.6
```

**纯音频播放**：

```yaml
dependencies:
  media_kit: ^1.2.6
  media_kit_libs_audio: ^1.2.6 # 音频专用库（更小体积）
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/build.gradle` 中配置：

```gradle
android {
    defaultConfig {
        minSdkVersion 21
    }
}
```

**iOS/macOS 配置**：

无需额外配置。

**Windows/Linux 配置**：

无需额外配置。

**Web 配置**：

在 `web/index.html` 中添加：

```html
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
```

Dart Tips 语法小贴士

**包大小优化**

`media_kit` 提供两种原生库包：

| 包名                   | 用途      | 大小 |
| ---------------------- | --------- | ---- |
| `media_kit_libs_video` | 视频+音频 | 较大 |
| `media_kit_libs_audio` | 仅音频    | 较小 |

如果只需要音频播放，使用 `media_kit_libs_audio` 可以显著减小应用体积。

## 第2章 Player 类详解

`Player` 是 `media_kit` 包的核心类，负责媒体播放的控制和管理。

### 2.1 构造函数

```dart
Player({
  PlayerConfiguration? configuration,  // 播放器配置
})
```

### 2.2 PlayerConfiguration 配置

```dart
PlayerConfiguration({
  bool? ready,                              // 是否就绪
  bool? pauseUponEnteringBackgroundMode,    // 进入后台时暂停
  bool? normalizeVolume,                    // 标准化音量
  String? title,                            // 标题
  String? logLevel,                         // 日志级别
  List<String>? vo,                         // 视频输出选项
  List<String>? ao,                         // 音频输出选项
  String? libass,                           // libass 配置
  String? libassAndroidFont,                // Android 字体
  String? libassAndroidFontConfig,          // Android 字体配置
  String? libassOSDFonts,                   // OSD 字体
  String? preferredAudioLanguage,           // 首选音频语言
  String? preferredSubtitleLanguage,        // 首选字幕语言
})
```

**使用示例**：

```dart
final player = Player(
  configuration: PlayerConfiguration(
    ready: true,
    pauseUponEnteringBackgroundMode: true,
    normalizeVolume: true,
  ),
);
```

### 2.3 核心属性详解

#### 2.3.1 播放状态属性

| 属性     | 类型           | 说明           |
| -------- | -------------- | -------------- |
| `state`  | `PlayerState`  | 播放器当前状态 |
| `stream` | `PlayerStream` | 播放器状态流   |

**PlayerState 属性**：

| 属性           | 类型                | 说明                |
| -------------- | ------------------- | ------------------- |
| `playing`      | `bool`              | 是否正在播放        |
| `completed`    | `bool`              | 是否播放完成        |
| `position`     | `Duration`          | 当前播放位置        |
| `duration`     | `Duration?`         | 媒体总时长          |
| `buffer`       | `Duration`          | 缓冲位置            |
| `playlist`     | `Playlist`          | 播放列表            |
| `volume`       | `double`            | 音量（0.0 - 100.0） |
| `rate`         | `double`            | 播放速度            |
| `pitch`        | `double`            | 音高                |
| `audioDevice`  | `AudioDevice`       | 当前音频设备        |
| `audioDevices` | `List<AudioDevice>` | 可用音频设备列表    |
| `track`        | `Track`             | 当前轨道            |
| `tracks`       | `Tracks`            | 可用轨道            |

#### 2.3.2 PlayerStream 属性

| 属性           | 类型                        | 说明           |
| -------------- | --------------------------- | -------------- |
| `playing`      | `Stream<bool>`              | 播放状态流     |
| `completed`    | `Stream<bool>`              | 完成状态流     |
| `position`     | `Stream<Duration>`          | 位置流         |
| `duration`     | `Stream<Duration?>`         | 时长流         |
| `buffer`       | `Stream<Duration>`          | 缓冲流         |
| `playlist`     | `Stream<Playlist>`          | 播放列表流     |
| `volume`       | `Stream<double>`            | 音量流         |
| `rate`         | `Stream<double>`            | 速度流         |
| `pitch`        | `Stream<double>`            | 音高流         |
| `audioDevice`  | `Stream<AudioDevice>`       | 音频设备流     |
| `audioDevices` | `Stream<List<AudioDevice>>` | 音频设备列表流 |
| `track`        | `Stream<Track>`             | 轨道流         |
| `tracks`       | `Stream<Tracks>`            | 轨道列表流     |

### 2.4 核心方法详解

#### 2.4.1 打开媒体

```dart
// 打开单个媒体
Future<void> open(
  Media media, {
  bool play = true,  // 是否立即播放
})

// 打开播放列表
Future<void> open(
  Playlist playlist, {
  bool play = true,
})
```

**Media 类**：

```dart
Media(String uri, {
  Map<String, String>? httpHeaders,  // HTTP 请求头
  dynamic extras,                   // 额外数据
})
```

**使用示例**：

```dart
// 打开网络视频
await player.open(
  Media('https://example.com/video.mp4'),
);

// 打开本地文件
await player.open(
  Media('file:///path/to/video.mp4'),
);

// 打开带请求头的媒体
await player.open(
  Media(
    'https://example.com/video.mp4',
    httpHeaders: {'Authorization': 'Bearer token'},
  ),
);

// 打开播放列表
await player.open(
  Playlist(
    [
      Media('https://example.com/video1.mp4'),
      Media('https://example.com/video2.mp4'),
      Media('https://example.com/video3.mp4'),
    ],
    index: 0,  // 起始索引
  ),
);
```

#### 2.4.2 播放控制方法

```dart
// 播放
Future<void> play()

// 暂停
Future<void> pause()

// 播放或暂停
Future<void> playOrPause()

// 停止
Future<void> stop()

// 跳转到指定位置
Future<void> seek(Duration duration)
```

**使用示例**：

```dart
// 播放
await player.play();

// 暂停
await player.pause();

// 切换播放/暂停
await player.playOrPause();

// 跳转到 30 秒
await player.seek(Duration(seconds: 30));
```

#### 2.4.3 播放列表方法

```dart
// 播放下一项
Future<void> next()

// 播放上一项
Future<void> previous()

// 跳转到指定索引
Future<void> jump(int index)

// 添加到播放列表
Future<void> add(Media media)

// 移除指定索引
Future<void> remove(int index)

// 移动项目
Future<void> move(int from, int to)
```

**使用示例**：

```dart
// 播放下一项
await player.next();

// 播放上一项
await player.previous();

// 跳转到第 3 项
await player.jump(2);

// 添加媒体
await player.add(Media('https://example.com/new_video.mp4'));
```

#### 2.4.4 播放设置方法

```dart
// 设置音量
Future<void> setVolume(double volume)  // 0.0 - 100.0

// 设置播放速度
Future<void> setRate(double rate)      // 0.5 - 2.0

// 设置音高
Future<void> setPitch(double pitch)

// 设置循环模式
Future<void> setPlaylistMode(PlaylistMode mode)
```

**PlaylistMode 枚举**：

| 值                    | 说明     |
| --------------------- | -------- |
| `PlaylistMode.none`   | 不循环   |
| `PlaylistMode.single` | 单曲循环 |
| `PlaylistMode.loop`   | 列表循环 |

**使用示例**：

```dart
// 设置音量 50%
await player.setVolume(50.0);

// 设置 1.5 倍速
await player.setRate(1.5);

// 设置单曲循环
await player.setPlaylistMode(PlaylistMode.single);
```

#### 2.4.5 音频设备方法

```dart
// 设置音频设备
Future<void> setAudioDevice(AudioDevice audioDevice)

// 获取可用音频设备
Future<List<AudioDevice>> observeAudioDevices()
```

**使用示例**：

```dart
// 获取可用音频设备
final devices = await player.observeAudioDevices();
for (final device in devices) {
  print('设备: ${device.name}, ID: ${device.id}');
}

// 设置音频设备
await player.setAudioDevice(devices.first);
```

#### 2.4.6 轨道控制方法

```dart
// 设置视频轨道
Future<void> setVideoTrack(VideoTrack track)

// 设置音频轨道
Future<void> setAudioTrack(AudioTrack track)

// 设置字幕轨道
Future<void> setSubtitleTrack(SubtitleTrack track)
```

**使用示例**：

```dart
// 获取可用轨道
final tracks = await player.tracks;

// 设置字幕轨道
if (tracks.subtitle.isNotEmpty) {
  await player.setSubtitleTrack(tracks.subtitle.first);
}
```

#### 2.4.7 截图方法

```dart
// 截取当前帧
Future<Uint8List?> screenshot({
  String? format,    // 格式：'image/jpeg', 'image/png'
})
```

**使用示例**：

```dart
// 截取当前帧
final screenshot = await player.screenshot(format: 'image/png');
if (screenshot != null) {
  // 保存或显示截图
}
```

#### 2.4.8 资源释放方法

```dart
// 释放播放器资源
Future<void> dispose()
```

**重要**：在不再使用播放器时，必须调用 `dispose()` 方法释放资源。

## 第3章 VideoController 类详解

`VideoController` 用于管理视频渲染。

### 3.1 构造函数

```dart
VideoController(
  Player player, {
  VideoControllerConfiguration? configuration,
})
```

### 3.2 VideoControllerConfiguration 配置

```dart
VideoControllerConfiguration({
  bool? enableHardwareAcceleration,  // 启用硬件加速
  String? vo,                        // 视频输出
  String? hwdec,                     // 硬件解码器
})
```

**使用示例**：

```dart
final videoController = VideoController(
  player,
  configuration: VideoControllerConfiguration(
    enableHardwareAcceleration: true,
    hwdec: 'auto',
  ),
);
```

### 3.3 核心属性

| 属性     | 类型           | 说明         |
| -------- | -------------- | ------------ |
| `player` | `Player`       | 关联的播放器 |
| `rect`   | `Stream<Rect>` | 视频矩形流   |
| `id`     | `int?`         | 视频输出 ID  |

## 第4章 Video Widget 详解

`Video` 是 `media_kit` 提供的视频显示 Widget。

### 4.1 构造函数

```dart
Video({
  required VideoController controller,  // 视频控制器（必填）
  double? width,                        // 宽度
  double? height,                       // 高度
  BoxFit fit = BoxFit.contain,          // 适应模式
  Color? fill,                          // 填充颜色
  Alignment alignment = Alignment.center, // 对齐方式
  double? aspectRatio,                  // 宽高比
  FilterQuality filterQuality = FilterQuality.low, // 滤镜质量
  Widget? placeholder,                  // 占位图
  Widget? errorBuilder,                 // 错误构建器
  Widget? loadingBuilder,               // 加载构建器
  SubtitleViewConfiguration? subtitleViewConfiguration, // 字幕配置
  List<VideoViewFilter>? filters,       // 视频滤镜
  List<VideoViewAttribute>? attributes, // 视频属性
})
```

### 4.2 使用示例

```dart
Video(
  controller: videoController,
  width: 640,
  height: 360,
  fit: BoxFit.contain,
)
```

## 第5章 media_kit 实战应用

### 5.1 基础视频播放器

```dart
class BasicMediaKitPlayer extends StatefulWidget {
  @override
  _BasicMediaKitPlayerState createState() => _BasicMediaKitPlayerState();
}

class _BasicMediaKitPlayerState extends State<BasicMediaKitPlayer> {
  late final Player player;
  late final VideoController videoController;

  @override
  void initState() {
    super.initState();
    player = Player();
    videoController = VideoController(player);
    _initializePlayer();
  }

  Future<void> _initializePlayer() async {
    await player.open(
      Media('https://example.com/video.mp4'),
    );
  }

  @override
  void dispose() {
    player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('media_kit 播放器')),
      body: Center(
        child: SizedBox(
          width: 640,
          height: 360,
          child: Video(controller: videoController),
        ),
      ),
    );
  }
}
```

### 5.2 完整视频播放器（带控制条）

```dart
class FullMediaKitPlayer extends StatefulWidget {
  final String videoUrl;

  const FullMediaKitPlayer({super.key, required this.videoUrl});

  @override
  _FullMediaKitPlayerState createState() => _FullMediaKitPlayerState();
}

class _FullMediaKitPlayerState extends State<FullMediaKitPlayer> {
  late final Player player;
  late final VideoController videoController;
  bool isPlaying = false;
  Duration position = Duration.zero;
  Duration duration = Duration.zero;
  double volume = 100.0;

  @override
  void initState() {
    super.initState();
    player = Player();
    videoController = VideoController(player);
    _initializePlayer();
  }

  void _initializePlayer() {
    player.open(Media(widget.videoUrl));

    // 监听播放状态
    player.stream.playing.listen((playing) {
      setState(() {
        isPlaying = playing;
      });
    });

    // 监听位置
    player.stream.position.listen((pos) {
      setState(() {
        position = pos;
      });
    });

    // 监听时长
    player.stream.duration.listen((dur) {
      setState(() {
        duration = dur ?? Duration.zero;
      });
    });

    // 监听音量
    player.stream.volume.listen((vol) {
      setState(() {
        volume = vol;
      });
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
    player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: Column(
        children: [
          // 视频区域
          Expanded(
            child: Center(
              child: Video(
                controller: videoController,
                fit: BoxFit.contain,
              ),
            ),
          ),

          // 控制条
          Container(
            color: Colors.black87,
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                // 进度条
                Slider(
                  value: position.inMilliseconds.toDouble(),
                  max: duration.inMilliseconds.toDouble(),
                  onChanged: (value) {
                    player.seek(Duration(milliseconds: value.toInt()));
                  },
                ),

                // 时间和控制按钮
                Row(
                  children: [
                    // 播放/暂停
                    IconButton(
                      icon: Icon(
                        isPlaying ? Icons.pause : Icons.play_arrow,
                        color: Colors.white,
                      ),
                      onPressed: () => player.playOrPause(),
                    ),

                    // 时间显示
                    Text(
                      '${_formatDuration(position)} / ${_formatDuration(duration)}',
                      style: TextStyle(color: Colors.white),
                    ),

                    Spacer(),

                    // 音量
                    Icon(
                      volume > 0 ? Icons.volume_up : Icons.volume_off,
                      color: Colors.white,
                      size: 20,
                    ),
                    SizedBox(width: 8),
                    SizedBox(
                      width: 100,
                      child: Slider(
                        value: volume,
                        max: 100,
                        onChanged: (value) => player.setVolume(value),
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 5.3 播放列表播放器

```dart
class PlaylistMediaKitPlayer extends StatefulWidget {
  @override
  _PlaylistMediaKitPlayerState createState() => _PlaylistMediaKitPlayerState();
}

class _PlaylistMediaKitPlayerState extends State<PlaylistMediaKitPlayer> {
  late final Player player;
  late final VideoController videoController;
  int currentIndex = 0;

  final List<String> videos = [
    'https://example.com/video1.mp4',
    'https://example.com/video2.mp4',
    'https://example.com/video3.mp4',
  ];

  @override
  void initState() {
    super.initState();
    player = Player();
    videoController = VideoController(player);
    _initializePlaylist();
  }

  Future<void> _initializePlaylist() async {
    await player.open(
      Playlist(
        videos.map((url) => Media(url)).toList(),
        index: 0,
      ),
    );

    // 监听当前索引
    player.stream.playlist.listen((playlist) {
      setState(() {
        currentIndex = playlist.index;
      });
    });
  }

  @override
  void dispose() {
    player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('播放列表播放器')),
      body: Column(
        children: [
          // 视频区域
          AspectRatio(
            aspectRatio: 16 / 9,
            child: Video(controller: videoController),
          ),

          // 播放列表
          Expanded(
            child: ListView.builder(
              itemCount: videos.length,
              itemBuilder: (context, index) {
                return ListTile(
                  title: Text('视频 ${index + 1}'),
                  leading: currentIndex == index
                      ? Icon(Icons.play_arrow, color: Colors.blue)
                      : null,
                  selected: currentIndex == index,
                  onTap: () => player.jump(index),
                );
              },
            ),
          ),

          // 播放控制
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                icon: Icon(Icons.skip_previous),
                onPressed: () => player.previous(),
              ),
              IconButton(
                icon: Icon(Icons.play_arrow),
                onPressed: () => player.play(),
              ),
              IconButton(
                icon: Icon(Icons.pause),
                onPressed: () => player.pause(),
              ),
              IconButton(
                icon: Icon(Icons.skip_next),
                onPressed: () => player.next(),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

### 5.4 带截图功能的播放器

```dart
class ScreenshotPlayer extends StatefulWidget {
  @override
  _ScreenshotPlayerState createState() => _ScreenshotPlayerState();
}

class _ScreenshotPlayerState extends State<ScreenshotPlayer> {
  late final Player player;
  late final VideoController videoController;
  Uint8List? screenshot;

  @override
  void initState() {
    super.initState();
    player = Player();
    videoController = VideoController(player);
    player.open(Media('https://example.com/video.mp4'));
  }

  Future<void> _takeScreenshot() async {
    final image = await player.screenshot(format: 'image/png');
    setState(() {
      screenshot = image;
    });
  }

  @override
  void dispose() {
    player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('截图播放器')),
      body: Column(
        children: [
          // 视频区域
          AspectRatio(
            aspectRatio: 16 / 9,
            child: Video(controller: videoController),
          ),

          // 截图按钮
          ElevatedButton.icon(
            onPressed: _takeScreenshot,
            icon: Icon(Icons.camera_alt),
            label: Text('截图'),
          ),

          // 显示截图
          if (screenshot != null)
            Image.memory(
              screenshot!,
              height: 200,
            ),
        ],
      ),
    );
  }
}
```

## 附录 A：Player 方法速查表

| 方法                 | 返回值               | 说明           |
| -------------------- | -------------------- | -------------- |
| `open()`             | `Future<void>`       | 打开媒体       |
| `play()`             | `Future<void>`       | 播放           |
| `pause()`            | `Future<void>`       | 暂停           |
| `playOrPause()`      | `Future<void>`       | 播放或暂停     |
| `stop()`             | `Future<void>`       | 停止           |
| `seek()`             | `Future<void>`       | 跳转           |
| `next()`             | `Future<void>`       | 下一项         |
| `previous()`         | `Future<void>`       | 上一项         |
| `jump()`             | `Future<void>`       | 跳转到指定索引 |
| `add()`              | `Future<void>`       | 添加到播放列表 |
| `setVolume()`        | `Future<void>`       | 设置音量       |
| `setRate()`          | `Future<void>`       | 设置播放速度   |
| `setPitch()`         | `Future<void>`       | 设置音高       |
| `setPlaylistMode()`  | `Future<void>`       | 设置循环模式   |
| `setAudioDevice()`   | `Future<void>`       | 设置音频设备   |
| `setVideoTrack()`    | `Future<void>`       | 设置视频轨道   |
| `setAudioTrack()`    | `Future<void>`       | 设置音频轨道   |
| `setSubtitleTrack()` | `Future<void>`       | 设置字幕轨道   |
| `screenshot()`       | `Future<Uint8List?>` | 截图           |
| `dispose()`          | `Future<void>`       | 释放资源       |

## 附录 B：PlayerState 属性速查表

| 属性        | 类型        | 说明       |
| ----------- | ----------- | ---------- |
| `playing`   | `bool`      | 是否播放中 |
| `completed` | `bool`      | 是否完成   |
| `position`  | `Duration`  | 当前位置   |
| `duration`  | `Duration?` | 总时长     |
| `buffer`    | `Duration`  | 缓冲位置   |
| `volume`    | `double`    | 音量       |
| `rate`      | `double`    | 播放速度   |
| `pitch`     | `double`    | 音高       |

## 附录 C：常见问题与解决方案

### Q1：编译失败

**解决方案**：

1. 确保 `minSdkVersion` >= 21（Android）
2. 运行 `flutter clean` 后重新构建
3. 检查依赖版本兼容性

### Q2：视频无法播放

**检查项**：

1. 视频格式是否支持
2. URL 是否可访问
3. 网络权限是否配置

### Q3：Web 平台问题

**解决方案**：
确保在 `index.html` 中添加了 hls.js：

```html
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
```

## 附录 D：参考资源

- [media_kit 官方文档](https://pub.dev/packages/media_kit)
- [media_kit GitHub](https://github.com/media-kit/media-kit)
- [mpv 文档](https://mpv.io/manual/)
- [hls.js 文档](https://github.com/video-dev/hls.js/)
