# just_audio 0.10.5 详解与实战

## 第1章 认识 just_audio

### 1.1 什么是 just_audio

`just_audio` 是由 ryanheise.com 开发的功能丰富的 Flutter 音频播放器库，支持循环、剪辑和序列化任意来源（资源/文件/URL/流）的音频，并提供无缝播放列表功能。该包是 Flutter 社区最受欢迎的音频播放器之一，被广泛应用于音乐播放器、播客应用、有声读物等场景。

与其他 Flutter 音频库相比，`just_audio` 提供了以下核心优势：

| 特性     | just_audio     | 其他库     |
| -------- | -------------- | ---------- |
| 播放列表 | 内置支持       | 需手动实现 |
| 无缝播放 | 支持           | 有限支持   |
| 音频剪辑 | 内置支持       | 需手动实现 |
| 后台播放 | 支持           | 需额外配置 |
| 缓存机制 | 内置           | 需额外实现 |
| 状态管理 | 完善的流式 API | 较简单     |

Flutter 框架小知识

**音频播放器的选择指南**

Flutter 生态中有多个音频播放器库，选择时需要考虑：

1. **just_audio**：功能最全面，适合复杂音频应用（音乐播放器、播客）
2. **audioplayers**：简单易用，适合基础音频播放
3. **flutter_soloud**：低延迟，适合游戏音频
4. **flutter_sound**：支持录制和播放，功能全面但较复杂

对于需要播放列表、后台播放、音频缓存的应用，推荐使用 `just_audio`。

### 1.2 just_audio 0.10.5 的新特性

版本 0.10.5 是 `just_audio` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **多种音频源支持**：assets、files、URLs、streams
- **播放列表管理**：支持多首歌曲的无缝切换
- **音频剪辑**：支持设置开始和结束时间点
- **循环播放**：支持单曲循环和列表循环
- **后台播放**：支持 iOS 和 Android 后台播放
- **音频缓存**：支持网络音频的本地缓存
- **均衡器**：支持音频效果处理
- **速度控制**：支持变速播放（0.5x - 2.0x）

#### 1.2.2 平台支持

| 平台    | 支持状态 | 最低版本     |
| ------- | -------- | ------------ |
| Android | ✅       | API 16+      |
| iOS     | ✅       | iOS 11.0+    |
| macOS   | ✅       | macOS 10.15+ |
| Web     | ✅       | 现代浏览器   |
| Windows | ✅       | Windows 10+  |
| Linux   | ✅       | 任意版本     |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  just_audio: ^0.10.5
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加网络权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加后台播放配置：

```xml
<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
</array>
```

Dart Tips 语法小贴士

**音频会话管理**

`just_audio` 依赖 `audio_session` 包来管理音频会话。对于需要后台播放的应用，建议添加：

```yaml
dependencies:
  audio_session: ^0.1.0
```

并在应用启动时配置音频会话：

```dart
final session = await AudioSession.instance;
await session.configure(AudioSessionConfiguration.music());
```

## 第2章 AudioPlayer 类详解

`AudioPlayer` 是 `just_audio` 包的核心类，负责管理和控制音频播放。

### 2.1 构造函数

```dart
AudioPlayer({
  String? userAgent,                          // HTTP 请求的 User-Agent
  bool handleInterruptions = true,            // 是否自动处理音频中断
  bool androidApplyAudioAttributes = true,    // 是否应用 Android 音频属性
  bool handleAudioSessionActivation = true,   // 是否自动激活音频会话
  AudioLoadConfiguration? audioLoadConfiguration, // 音频加载配置
  AudioPipeline? audioPipeline,               // 音频效果管道
  bool androidOffloadSchedulingEnabled = false, // Android 是否启用卸载调度
  bool useProxyForRequestHeaders = true,      // 是否使用代理处理请求头
  bool useLazyPreparation = true,             // 是否延迟准备
  ShuffleOrder? shuffleOrder,                 // 随机播放顺序
  int maxSkipsOnError = 0,                    // 错误时自动跳过次数
})
```

### 2.2 核心属性详解

#### 2.2.1 播放状态属性

| 属性              | 类型              | 说明         |
| ----------------- | ----------------- | ------------ |
| `playing`         | `bool`            | 是否正在播放 |
| `processingState` | `ProcessingState` | 当前处理状态 |
| `playbackEvent`   | `PlaybackEvent`   | 播放事件流   |

**ProcessingState 枚举**：

| 值          | 说明     |
| ----------- | -------- |
| `idle`      | 空闲状态 |
| `loading`   | 正在加载 |
| `buffering` | 正在缓冲 |
| `ready`     | 准备就绪 |
| `completed` | 播放完成 |

#### 2.2.2 音频信息属性

| 属性               | 类型                        | 说明         |
| ------------------ | --------------------------- | ------------ |
| `duration`         | `Duration?`                 | 音频总时长   |
| `position`         | `Duration`                  | 当前播放位置 |
| `bufferedPosition` | `Duration`                  | 已缓冲位置   |
| `currentIndex`     | `int?`                      | 当前播放索引 |
| `sequence`         | `List<IndexedAudioSource>?` | 播放序列     |
| `hasNext`          | `bool`                      | 是否有下一首 |
| `hasPrevious`      | `bool`                      | 是否有上一首 |

#### 2.2.3 播放控制属性

| 属性                 | 类型       | 说明                  |
| -------------------- | ---------- | --------------------- |
| `volume`             | `double`   | 音量（0.0 - 1.0）     |
| `speed`              | `double`   | 播放速度（0.5 - 2.0） |
| `pitch`              | `double`   | 音高（Android 支持）  |
| `loopMode`           | `LoopMode` | 循环模式              |
| `shuffleModeEnabled` | `bool`     | 是否启用随机播放      |

**LoopMode 枚举**：

| 值    | 说明     |
| ----- | -------- |
| `off` | 不循环   |
| `one` | 单曲循环 |
| `all` | 列表循环 |

### 2.3 核心方法详解

#### 2.3.1 播放控制方法

```dart
// 设置并播放音频源
Future<Duration?> setAudioSource(
  AudioSource source, {
  bool preload = true,
  int? initialIndex,
  Duration? initialPosition,
})

// 播放
Future<void> play()

// 暂停
Future<void> pause()

// 停止
Future<void> stop()

// 跳转到指定位置
Future<void> seek(Duration position, {int? index})
```

**使用示例**：

```dart
final player = AudioPlayer();

// 设置音频源并播放
await player.setAudioSource(AudioSource.uri(Uri.parse('https://example.com/audio.mp3')));
await player.play();

// 暂停
await player.pause();

// 跳转到 30 秒处
await player.seek(Duration(seconds: 30));

// 停止
await player.stop();
```

#### 2.3.2 播放列表方法

```dart
// 加载播放列表
Future<Duration?> setAudioSource(
  ConcatenatingAudioSource playlist, {
  bool preload = true,
  int? initialIndex,
  Duration? initialPosition,
})

// 添加音频到播放列表
Future<void> add(AudioSource source)

// 插入音频到指定位置
Future<void> insert(int index, AudioSource source)

// 移除指定位置的音频
Future<void> removeAt(int index)

// 移动到指定位置
Future<void> move(int currentIndex, int newIndex)

// 清空播放列表
Future<void> clear()

// 播放下一首
Future<void> seekToNext()

// 播放上一首
Future<void> seekToPrevious()
```

**使用示例**：

```dart
// 创建播放列表
final playlist = ConcatenatingAudioSource(
  children: [
    AudioSource.uri(Uri.parse('https://example.com/song1.mp3')),
    AudioSource.uri(Uri.parse('https://example.com/song2.mp3')),
    AudioSource.uri(Uri.parse('https://example.com/song3.mp3')),
  ],
);

// 加载播放列表
await player.setAudioSource(playlist);

// 添加新歌曲
await player.add(AudioSource.uri(Uri.parse('https://example.com/song4.mp3')));

// 播放下一首
await player.seekToNext();
```

#### 2.3.3 播放设置方法

```dart
// 设置音量
Future<void> setVolume(double volume)  // 0.0 - 1.0

// 设置播放速度
Future<void> setSpeed(double speed)    // 0.5 - 2.0

// 设置音高（Android）
Future<void> setPitch(double pitch)    // 0.5 - 2.0

// 设置循环模式
Future<void> setLoopMode(LoopMode mode)

// 设置随机播放
Future<void> setShuffleModeEnabled(bool enabled)
```

**使用示例**：

```dart
// 设置音量 50%
await player.setVolume(0.5);

// 设置 1.5 倍速播放
await player.setSpeed(1.5);

// 设置单曲循环
await player.setLoopMode(LoopMode.one);

// 启用随机播放
await player.setShuffleModeEnabled(true);
```

#### 2.3.4 资源释放方法

```dart
// 释放播放器资源
Future<void> dispose()
```

**重要**：在不再使用播放器时，必须调用 `dispose()` 方法释放资源。

### 2.4 状态监听

#### 2.4.1 播放状态流

```dart
// 监听播放状态
player.playerStateStream.listen((state) {
  if (state.playing) {
    print('正在播放');
  } else {
    print('已暂停');
  }

  switch (state.processingState) {
    case ProcessingState.idle:
      print('空闲');
      break;
    case ProcessingState.loading:
      print('加载中');
      break;
    case ProcessingState.buffering:
      print('缓冲中');
      break;
    case ProcessingState.ready:
      print('准备就绪');
      break;
    case ProcessingState.completed:
      print('播放完成');
      break;
  }
});
```

#### 2.4.2 位置监听

```dart
// 监听播放位置（每秒更新）
player.positionStream.listen((position) {
  print('当前位置: $position');
});

// 监听缓冲位置
player.bufferedPositionStream.listen((bufferedPosition) {
  print('已缓冲: $bufferedPosition');
});

// 监听总时长
player.durationStream.listen((duration) {
  print('总时长: $duration');
});
```

#### 2.4.3 当前歌曲监听

```dart
// 监听当前播放索引
player.currentIndexStream.listen((index) {
  print('当前播放: $index');
});

// 监听播放序列
player.sequenceStream.listen((sequence) {
  print('播放列表: $sequence');
});
```

## 第3章 AudioSource 详解

`AudioSource` 是 `just_audio` 中表示音频源的抽象类，有多种具体实现。

### 3.1 音频源类型

| 类型                         | 用途           | 示例                       |
| ---------------------------- | -------------- | -------------------------- |
| `AudioSource.uri()`          | 网络或本地文件 | URL、本地文件路径          |
| `AudioSource.asset()`        | Flutter 资源   | assets/audio/song.mp3      |
| `AudioSource.file()`         | 本地文件       | File('/path/to/audio.mp3') |
| `ClippingAudioSource()`      | 音频剪辑       | 截取音频的一部分           |
| `LoopingAudioSource()`       | 循环播放       | 单曲循环                   |
| `ConcatenatingAudioSource()` | 播放列表       | 多首歌曲                   |
| `ProgressiveAudioSource()`   | 渐进式加载     | 大文件流式播放             |
| `DashAudioSource()`          | DASH 流媒体    | 自适应流媒体               |
| `HlsAudioSource()`           | HLS 流媒体     | Apple 流媒体               |

### 3.2 网络音频源

```dart
// 基本网络音频
final source = AudioSource.uri(
  Uri.parse('https://example.com/audio.mp3'),
);

// 带请求头的网络音频
final source = AudioSource.uri(
  Uri.parse('https://example.com/audio.mp3'),
  headers: {'Authorization': 'Bearer token'},
);

// 带标签的网络音频
final source = AudioSource.uri(
  Uri.parse('https://example.com/audio.mp3'),
  tag: MediaItem(
    id: '1',
    title: '歌曲名称',
    artist: '艺术家',
    artUri: Uri.parse('https://example.com/cover.jpg'),
  ),
);
```

### 3.3 本地音频源

```dart
// Flutter 资源
final source = AudioSource.asset('assets/audio/song.mp3');

// 本地文件
final file = File('/path/to/audio.mp3');
final source = AudioSource.file(file);

// URI 方式
final source = AudioSource.uri(Uri.file('/path/to/audio.mp3'));
```

### 3.4 音频剪辑

```dart
// 剪辑音频的 10-30 秒部分
final source = ClippingAudioSource(
  child: AudioSource.uri(Uri.parse('https://example.com/audio.mp3')),
  start: Duration(seconds: 10),
  end: Duration(seconds: 30),
);
```

### 3.5 循环播放

```dart
// 循环播放 3 次
final source = LoopingAudioSource(
  child: AudioSource.uri(Uri.parse('https://example.com/audio.mp3')),
  count: 3,
);

// 无限循环
final source = LoopingAudioSource(
  child: AudioSource.uri(Uri.parse('https://example.com/audio.mp3')),
);
```

### 3.6 播放列表

```dart
// 创建播放列表
final playlist = ConcatenatingAudioSource(
  useLazyPreparation: true,  // 延迟准备
  shuffleOrder: DefaultShuffleOrder(),  // 随机顺序
  children: [
    AudioSource.uri(
      Uri.parse('https://example.com/song1.mp3'),
      tag: MediaItem(id: '1', title: '歌曲1'),
    ),
    AudioSource.uri(
      Uri.parse('https://example.com/song2.mp3'),
      tag: MediaItem(id: '2', title: '歌曲2'),
    ),
    AudioSource.uri(
      Uri.parse('https://example.com/song3.mp3'),
      tag: MediaItem(id: '3', title: '歌曲3'),
    ),
  ],
);
```

## 第4章 音频缓存

### 4.1 缓存配置

```dart
// 创建带缓存的音频源
final source = LockCachingAudioSource(
  Uri.parse('https://example.com/audio.mp3'),
  cacheFile: File('${(await getTemporaryDirectory()).path}/cached_audio.mp3'),
);

// 加载带缓存的音频
await player.setAudioSource(source);
```

### 4.2 缓存管理

```dart
// 清除缓存
await source.clearCache();

// 检查缓存是否存在
final exists = await source.cacheFile.exists();
```

## 第5章 音频效果

### 5.1 均衡器

```dart
// 创建带均衡器的播放器
final player = AudioPlayer(
  audioPipeline: AudioPipeline(
    androidAudioEffects: [
      AndroidEqualizer(),
    ],
  ),
);

// 设置均衡器参数
final equalizer = player.androidEqualizer!;
await equalizer.setEnabled(true);
```

### 5.2 音量增强

```dart
// 创建带音量增强的播放器
final player = AudioPlayer(
  audioPipeline: AudioPipeline(
    androidAudioEffects: [
      AndroidLoudnessEnhancer(),
    ],
  ),
);
```

## 第6章 just_audio 实战应用

### 6.1 简单音乐播放器

```dart
class SimpleMusicPlayer extends StatefulWidget {
  @override
  _SimpleMusicPlayerState createState() => _SimpleMusicPlayerState();
}

class _SimpleMusicPlayerState extends State<SimpleMusicPlayer> {
  final player = AudioPlayer();
  bool isPlaying = false;
  Duration duration = Duration.zero;
  Duration position = Duration.zero;

  @override
  void initState() {
    super.initState();
    initPlayer();
  }

  Future<void> initPlayer() async {
    // 设置音频源
    await player.setAudioSource(
      AudioSource.uri(Uri.parse('https://example.com/song.mp3')),
    );

    // 监听播放状态
    player.playerStateStream.listen((state) {
      setState(() {
        isPlaying = state.playing;
      });
    });

    // 监听时长
    player.durationStream.listen((d) {
      setState(() {
        duration = d ?? Duration.zero;
      });
    });

    // 监听位置
    player.positionStream.listen((p) {
      setState(() {
        position = p;
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
      appBar: AppBar(title: Text('音乐播放器')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          // 进度条
          Slider(
            value: position.inSeconds.toDouble(),
            max: duration.inSeconds.toDouble(),
            onChanged: (value) async {
              await player.seek(Duration(seconds: value.toInt()));
            },
          ),

          // 时间显示
          Text(
            '${position.toString().split('.').first} / ${duration.toString().split('.').first}',
          ),

          // 控制按钮
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              IconButton(
                icon: Icon(Icons.skip_previous),
                onPressed: () {},
              ),
              IconButton(
                icon: Icon(isPlaying ? Icons.pause : Icons.play_arrow),
                onPressed: () async {
                  if (isPlaying) {
                    await player.pause();
                  } else {
                    await player.play();
                  }
                },
              ),
              IconButton(
                icon: Icon(Icons.skip_next),
                onPressed: () {},
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

### 6.2 完整音乐播放器（带播放列表）

```dart
class MusicPlayer extends StatefulWidget {
  @override
  _MusicPlayerState createState() => _MusicPlayerState();
}

class _MusicPlayerState extends State<MusicPlayer> {
  final player = AudioPlayer();
  final playlist = ConcatenatingAudioSource(
    children: [
      AudioSource.uri(
        Uri.parse('https://example.com/song1.mp3'),
        tag: MediaItem(id: '1', title: '歌曲1', artist: '艺术家1'),
      ),
      AudioSource.uri(
        Uri.parse('https://example.com/song2.mp3'),
        tag: MediaItem(id: '2', title: '歌曲2', artist: '艺术家2'),
      ),
      AudioSource.uri(
        Uri.parse('https://example.com/song3.mp3'),
        tag: MediaItem(id: '3', title: '歌曲3', artist: '艺术家3'),
      ),
    ],
  );

  @override
  void initState() {
    super.initState();
    initPlayer();
  }

  Future<void> initPlayer() async {
    await player.setAudioSource(playlist);
  }

  @override
  void dispose() {
    player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('音乐播放器')),
      body: Column(
        children: [
          // 播放列表
          Expanded(
            child: StreamBuilder<SequenceState?>(
              stream: player.sequenceStateStream,
              builder: (context, snapshot) {
                final state = snapshot.data;
                final sequence = state?.sequence ?? [];

                return ListView.builder(
                  itemCount: sequence.length,
                  itemBuilder: (context, index) {
                    final mediaItem = sequence[index].tag as MediaItem;
                    final isCurrent = state?.currentIndex == index;

                    return ListTile(
                      title: Text(mediaItem.title),
                      subtitle: Text(mediaItem.artist ?? ''),
                      leading: isCurrent
                          ? Icon(Icons.volume_up, color: Colors.blue)
                          : null,
                      onTap: () async {
                        await player.seek(Duration.zero, index: index);
                        await player.play();
                      },
                    );
                  },
                );
              },
            ),
          ),

          // 播放控制
          _buildPlayerControls(),
        ],
      ),
    );
  }

  Widget _buildPlayerControls() {
    return Container(
      padding: EdgeInsets.all(16),
      child: Column(
        children: [
          // 进度条
          StreamBuilder<Duration>(
            stream: player.positionStream,
            builder: (context, snapshot) {
              final position = snapshot.data ?? Duration.zero;
              return StreamBuilder<Duration?>(
                stream: player.durationStream,
                builder: (context, snapshot) {
                  final duration = snapshot.data ?? Duration.zero;
                  return Slider(
                    value: position.inMilliseconds.toDouble(),
                    max: duration.inMilliseconds.toDouble(),
                    onChanged: (value) {
                      player.seek(Duration(milliseconds: value.toInt()));
                    },
                  );
                },
              );
            },
          ),

          // 控制按钮
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // 随机播放
              StreamBuilder<bool>(
                stream: player.shuffleModeEnabledStream,
                builder: (context, snapshot) {
                  final enabled = snapshot.data ?? false;
                  return IconButton(
                    icon: Icon(Icons.shuffle, color: enabled ? Colors.blue : null),
                    onPressed: () => player.setShuffleModeEnabled(!enabled),
                  );
                },
              ),

              // 上一首
              IconButton(
                icon: Icon(Icons.skip_previous),
                onPressed: player.seekToPrevious,
              ),

              // 播放/暂停
              StreamBuilder<bool>(
                stream: player.playingStream,
                builder: (context, snapshot) {
                  final playing = snapshot.data ?? false;
                  return IconButton(
                    icon: Icon(playing ? Icons.pause : Icons.play_arrow),
                    iconSize: 48,
                    onPressed: playing ? player.pause : player.play,
                  );
                },
              ),

              // 下一首
              IconButton(
                icon: Icon(Icons.skip_next),
                onPressed: player.seekToNext,
              ),

              // 循环模式
              StreamBuilder<LoopMode>(
                stream: player.loopModeStream,
                builder: (context, snapshot) {
                  final loopMode = snapshot.data ?? LoopMode.off;
                  return IconButton(
                    icon: Icon(
                      loopMode == LoopMode.one
                          ? Icons.repeat_one
                          : Icons.repeat,
                      color: loopMode != LoopMode.off ? Colors.blue : null,
                    ),
                    onPressed: () {
                      final modes = [LoopMode.off, LoopMode.one, LoopMode.all];
                      final index = modes.indexOf(loopMode);
                      player.setLoopMode(modes[(index + 1) % modes.length]);
                    },
                  );
                },
              ),
            ],
          ),
        ],
      ),
    );
  }
}
```

### 6.3 后台播放配置

```dart
// 使用 audio_service 实现后台播放
class AudioPlayerHandler extends BaseAudioHandler with SeekHandler {
  final _player = AudioPlayer();

  AudioPlayerHandler() {
    _player.playbackEventStream.map(_transformEvent).pipe(playbackState);
  }

  PlaybackState _transformEvent(PlaybackEvent event) {
    return PlaybackState(
      controls: [
        MediaControl.rewind,
        if (_player.playing) MediaControl.pause else MediaControl.play,
        MediaControl.stop,
        MediaControl.fastForward,
      ],
      systemActions: const {
        MediaAction.seek,
        MediaAction.seekForward,
        MediaAction.seekBackward,
      },
      androidCompactActionIndices: const [0, 1, 3],
      processingState: const {
        ProcessingState.idle: AudioProcessingState.idle,
        ProcessingState.loading: AudioProcessingState.loading,
        ProcessingState.buffering: AudioProcessingState.buffering,
        ProcessingState.ready: AudioProcessingState.ready,
        ProcessingState.completed: AudioProcessingState.completed,
      }[_player.processingState]!,
      playing: _player.playing,
      updatePosition: _player.position,
      bufferedPosition: _player.bufferedPosition,
      speed: _player.speed,
      queueIndex: event.currentIndex,
    );
  }

  @override
  Future<void> play() => _player.play();

  @override
  Future<void> pause() => _player.pause();

  @override
  Future<void> seek(Duration position) => _player.seek(position);
}
```

## 附录 A：AudioPlayer 属性/方法速查表

| 属性/方法         | 类型/返回值       | 说明         |
| ----------------- | ----------------- | ------------ |
| `playing`         | `bool`            | 是否正在播放 |
| `processingState` | `ProcessingState` | 处理状态     |
| `duration`        | `Duration?`       | 总时长       |
| `position`        | `Duration`        | 当前位置     |
| `volume`          | `double`          | 音量         |
| `speed`           | `double`          | 播放速度     |
| `loopMode`        | `LoopMode`        | 循环模式     |
| `play()`          | `Future<void>`    | 播放         |
| `pause()`         | `Future<void>`    | 暂停         |
| `stop()`          | `Future<void>`    | 停止         |
| `seek()`          | `Future<void>`    | 跳转         |
| `setVolume()`     | `Future<void>`    | 设置音量     |
| `setSpeed()`      | `Future<void>`    | 设置速度     |
| `setLoopMode()`   | `Future<void>`    | 设置循环模式 |
| `dispose()`       | `Future<void>`    | 释放资源     |

## 附录 B：常见问题与解决方案

### Q1：网络音频无法播放

**解决方案**：

1. 检查网络权限配置
2. 检查 URL 是否可访问
3. 检查是否支持音频格式

### Q2：后台播放不工作

**解决方案**：

1. 配置 iOS/Android 后台播放权限
2. 使用 `audio_service` 包
3. 配置音频会话

### Q3：内存泄漏

**解决方案**：

```dart
@override
void dispose() {
  player.dispose();  // 必须调用
  super.dispose();
}
```

## 附录 C：参考资源

- [just_audio 官方文档](https://pub.dev/packages/just_audio)
- [just_audio API 参考](https://pub.dev/documentation/just_audio/latest/)
- [audio_service 包](https://pub.dev/packages/audio_service)
- [Flutter 音频播放指南](https://docs.flutter.dev/cookbook/plugins/play-video)
