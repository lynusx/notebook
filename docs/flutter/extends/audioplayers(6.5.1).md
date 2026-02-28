# audioplayers 6.5.1 详解与实战

## 第1章 认识 audioplayers

### 1.1 什么是 audioplayers

`audioplayers` 是 Flutter 社区最流行的音频播放器包之一，由 Blue Fire 团队维护。它提供了一个简单易用的 API 来播放本地和远程音频文件，支持多种音频格式和平台。

与其他 Flutter 音频库相比，`audioplayers` 的核心优势在于：

| 特性       | audioplayers        | 其他库                 |
| ---------- | ------------------- | ---------------------- |
| 易用性     | ⭐⭐⭐⭐⭐ 非常简单 | 中等                   |
| 多实例     | 支持多个播放器实例  | 部分支持               |
| 平台支持   | 全面（6个平台）     | 部分                   |
| 音频格式   | 多种格式            | 有限                   |
| 低延迟     | 一般                | 优秀（flutter_soloud） |
| 功能丰富度 | 基础功能完整        | 各有侧重               |

Flutter 框架小知识

**何时选择 audioplayers？**

`audioplayers` 最适合以下场景：

1. **简单音频播放**：按钮点击音、通知音
2. **背景音乐**：游戏背景音乐、应用背景音乐
3. **短音频片段**：音效、提示音
4. **多平台应用**：需要同时支持 Web、移动端和桌面端
5. **快速开发**：需要快速实现音频功能

对于需要复杂播放列表、后台播放、音频缓存的应用，建议使用 `just_audio`。

### 1.2 audioplayers 6.5.1 的新特性

版本 6.5.1 是 `audioplayers` 的重要版本，主要包含以下改进：

#### 1.2.1 核心特性

- **多实例支持**：可以同时播放多个音频
- **全局音量控制**：支持全局音量设置
- **播放模式**：支持循环播放、单曲循环
- **播放速度**：支持变速播放
- **音频缓存**：支持音频预加载
- **音频事件**：丰富的播放状态回调

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
  audioplayers: ^6.5.1
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加：

```xml
<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
</array>
```

Dart Tips 语法小贴士

**音频文件路径处理**

不同平台的音频路径格式：

```dart
// 网络音频
final url = 'https://example.com/audio.mp3';

// 本地文件
final filePath = '/path/to/audio.mp3';

// Flutter 资源
final assetPath = 'assets/audio/sound.mp3';
```

## 第2章 AudioPlayer 类详解

`AudioPlayer` 是 `audioplayers` 包的核心类，每个实例可以播放一个音频。

### 2.1 构造函数

```dart
AudioPlayer({
  PlayerMode mode = PlayerMode.mediaPlayer,  // 播放器模式
  String? playerId,                           // 播放器 ID
})
```

**PlayerMode 枚举**：

| 值                       | 说明                     |
| ------------------------ | ------------------------ |
| `PlayerMode.mediaPlayer` | 媒体播放器模式（默认）   |
| `PlayerMode.lowLatency`  | 低延迟模式（适合短音效） |

### 2.2 核心属性详解

#### 2.2.1 播放状态属性

| 属性           | 类型          | 说明              |
| -------------- | ------------- | ----------------- |
| `state`        | `PlayerState` | 当前播放状态      |
| `position`     | `Duration`    | 当前播放位置      |
| `duration`     | `Duration?`   | 音频总时长        |
| `volume`       | `double`      | 音量（0.0 - 1.0） |
| `playbackRate` | `double`      | 播放速度          |
| `releaseMode`  | `ReleaseMode` | 释放模式          |

**PlayerState 枚举**：

| 值          | 说明     |
| ----------- | -------- |
| `stopped`   | 停止状态 |
| `playing`   | 正在播放 |
| `paused`    | 已暂停   |
| `completed` | 播放完成 |

**ReleaseMode 枚举**：

| 值        | 说明           |
| --------- | -------------- |
| `release` | 停止后释放资源 |
| `loop`    | 循环播放       |
| `stop`    | 停止但不释放   |

### 2.3 核心方法详解

#### 2.3.1 播放控制方法

```dart
// 播放网络音频
Future<void> play(
  String url, {
  bool? isLocal,           // 是否为本地文件
  double volume = 1.0,     // 音量
  Duration? position,      // 起始位置
  bool respectSilence = false,  // 是否尊重静音模式
  bool stayAwake = false,  // 是否保持唤醒
  bool duckAudio = false,  // 是否降低其他音频音量
})

// 播放本地文件
Future<void> play(
  String filePath,
  isLocal: true,
)

// 播放字节数据（Android API 23+）
Future<void> playBytes(
  Uint8List bytes, {
  double volume = 1.0,
  Duration? position,
})

// 暂停
Future<void> pause()

// 恢复播放
Future<void> resume()

// 停止
Future<void> stop()

// 释放资源
Future<void> release()

// 跳转到指定位置
Future<void> seek(Duration position)
```

**使用示例**：

```dart
final player = AudioPlayer();

// 播放网络音频
await player.play('https://example.com/audio.mp3');

// 播放本地文件
await player.play('/path/to/audio.mp3', isLocal: true);

// 暂停
await player.pause();

// 恢复
await player.resume();

// 停止
await player.stop();

// 释放资源
await player.release();
```

#### 2.3.2 播放设置方法

```dart
// 设置音量
Future<void> setVolume(double volume)  // 0.0 - 1.0

// 设置播放速度
Future<void> setPlaybackRate(double playbackRate)  // 0.5 - 2.0

// 设置释放模式
Future<void> setReleaseMode(ReleaseMode releaseMode)

// 设置音频源 URL
Future<void> setSourceUrl(String url, {bool? isLocal})

// 设置音频源设备文件
Future<void> setSourceDeviceFile(String path)

// 设置音频源资产
Future<void> setSourceAsset(String path)

// 设置音频源字节
Future<void> setSourceBytes(Uint8List bytes)
```

**使用示例**：

```dart
// 设置音量 50%
await player.setVolume(0.5);

// 设置 1.5 倍速
await player.setPlaybackRate(1.5);

// 设置循环播放
await player.setReleaseMode(ReleaseMode.loop);
```

#### 2.3.3 状态获取方法

```dart
// 获取当前位置
Future<Duration> getCurrentPosition()

// 获取总时长
Future<Duration?> getDuration()
```

### 2.4 事件监听

#### 2.4.1 播放状态监听

```dart
// 监听播放状态
player.onPlayerStateChanged.listen((PlayerState state) {
  print('播放状态: $state');
});
```

#### 2.4.2 位置监听

```dart
// 监听播放位置
player.onPositionChanged.listen((Duration position) {
  print('当前位置: $position');
});

// 监听时长
player.onDurationChanged.listen((Duration duration) {
  print('总时长: $duration');
});
```

#### 2.4.3 完成监听

```dart
// 监听播放完成
player.onPlayerComplete.listen((event) {
  print('播放完成');
});
```

## 第3章 AudioCache 类详解

`AudioCache` 用于预加载和缓存音频文件，适合播放短音效。

### 3.1 构造函数

```dart
AudioCache({
  String prefix = 'assets/',     // 资源前缀
  AudioPlayer? fixedPlayer,      // 固定播放器实例
  bool respectSilence = false,   // 是否尊重静音模式
  bool stayAwake = false,        // 是否保持唤醒
})
```

### 3.2 核心方法详解

```dart
// 加载音频文件到缓存
Future<ByteData> load(String fileName)

// 加载并播放音频
Future<AudioPlayer> play(
  String fileName, {
  double volume = 1.0,
  bool? isNotification,
  PlayerMode mode = PlayerPlayerMode.mediaPlayer,
  bool respectSilence = false,
  bool stayAwake = false,
})

// 加载并循环播放
Future<AudioPlayer> loop(
  String fileName, {
  double volume = 1.0,
  bool? isNotification,
  PlayerMode mode = PlayerMode.mediaPlayer,
  bool respectSilence = false,
  bool stayAwake = false,
})

// 清空缓存
void clearCache()

// 清空指定文件的缓存
void clear(String fileName)
```

**使用示例**：

```dart
final cache = AudioCache();

// 播放音效
await cache.play('sounds/click.mp3');

// 循环播放背景音乐
await cache.loop('sounds/background.mp3');

// 使用固定播放器
final fixedPlayer = AudioPlayer();
final cache = AudioCache(fixedPlayer: fixedPlayer);
await cache.play('sounds/effect.mp3');
```

## 第4章 全局音频控制

### 4.1 AudioPlayer 全局设置

```dart
// 设置全局音量
AudioPlayer.global.setGlobalVolume(0.5);

// 获取全局音量
final volume = AudioPlayer.global.globalVolume;
```

### 4.2 日志控制

```dart
// 启用日志
AudioPlayer.logEnabled = true;

// 禁用日志
AudioPlayer.logEnabled = false;
```

## 第5章 audioplayers 实战应用

### 5.1 简单音效播放器

```dart
class SoundEffectPlayer {
  static final AudioCache _cache = AudioCache(
    prefix: 'assets/sounds/',
  );

  static Future<void> playClick() async {
    await _cache.play('click.mp3');
  }

  static Future<void> playSuccess() async {
    await _cache.play('success.mp3');
  }

  static Future<void> playError() async {
    await _cache.play('error.mp3');
  }
}

// 使用
ElevatedButton(
  onPressed: () async {
    await SoundEffectPlayer.playClick();
    // 执行点击操作
  },
  child: Text('Click Me'),
)
```

### 5.2 背景音乐播放器

```dart
class BackgroundMusicPlayer {
  static AudioPlayer? _player;

  static Future<void> play(String path) async {
    _player ??= AudioPlayer();
    await _player!.play(path, isLocal: true);
    await _player!.setReleaseMode(ReleaseMode.loop);
  }

  static Future<void> stop() async {
    await _player?.stop();
  }

  static Future<void> pause() async {
    await _player?.pause();
  }

  static Future<void> resume() async {
    await _player?.resume();
  }

  static Future<void> setVolume(double volume) async {
    await _player?.setVolume(volume);
  }
}
```

### 5.3 完整音频播放器

```dart
class AudioPlayerWidget extends StatefulWidget {
  final String audioUrl;

  const AudioPlayerWidget({super.key, required this.audioUrl});

  @override
  _AudioPlayerWidgetState createState() => _AudioPlayerWidgetState();
}

class _AudioPlayerWidgetState extends State<AudioPlayerWidget> {
  late AudioPlayer _player;
  bool isPlaying = false;
  Duration duration = Duration.zero;
  Duration position = Duration.zero;

  @override
  void initState() {
    super.initState();
    _player = AudioPlayer();
    _initPlayer();
  }

  void _initPlayer() {
    // 监听状态
    _player.onPlayerStateChanged.listen((state) {
      setState(() {
        isPlaying = state == PlayerState.playing;
      });
    });

    // 监听时长
    _player.onDurationChanged.listen((d) {
      setState(() {
        duration = d;
      });
    });

    // 监听位置
    _player.onPositionChanged.listen((p) {
      setState(() {
        position = p;
      });
    });
  }

  @override
  void dispose() {
    _player.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 进度条
        Slider(
          value: position.inSeconds.toDouble(),
          max: duration.inSeconds.toDouble(),
          onChanged: (value) async {
            await _player.seek(Duration(seconds: value.toInt()));
          },
        ),

        // 时间显示
        Text('${position.toString().split('.').first} / ${duration.toString().split('.').first}'),

        // 控制按钮
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            IconButton(
              icon: Icon(Icons.replay_10),
              onPressed: () async {
                await _player.seek(position - Duration(seconds: 10));
              },
            ),
            IconButton(
              icon: Icon(isPlaying ? Icons.pause : Icons.play_arrow),
              iconSize: 48,
              onPressed: () async {
                if (isPlaying) {
                  await _player.pause();
                } else {
                  await _player.play(widget.audioUrl);
                }
              },
            ),
            IconButton(
              icon: Icon(Icons.forward_10),
              onPressed: () async {
                await _player.seek(position + Duration(seconds: 10));
              },
            ),
          ],
        ),
      ],
    );
  }
}
```

### 5.4 多音频同时播放

```dart
class MultiAudioPlayer {
  final List<AudioPlayer> _players = [];

  Future<void> playSound(String path) async {
    final player = AudioPlayer();
    _players.add(player);

    await player.play(path);

    // 播放完成后释放
    player.onPlayerComplete.listen((_) {
      player.dispose();
      _players.remove(player);
    });
  }

  void dispose() {
    for (final player in _players) {
      player.dispose();
    }
    _players.clear();
  }
}
```

## 附录 A：AudioPlayer 属性/方法速查表

| 属性/方法           | 类型/返回值    | 说明       |
| ------------------- | -------------- | ---------- |
| `state`             | `PlayerState`  | 播放状态   |
| `position`          | `Duration`     | 当前位置   |
| `duration`          | `Duration?`    | 总时长     |
| `volume`            | `double`       | 音量       |
| `playbackRate`      | `double`       | 播放速度   |
| `play()`            | `Future<void>` | 播放       |
| `pause()`           | `Future<void>` | 暂停       |
| `resume()`          | `Future<void>` | 恢复       |
| `stop()`            | `Future<void>` | 停止       |
| `seek()`            | `Future<void>` | 跳转       |
| `setVolume()`       | `Future<void>` | 设置音量   |
| `setPlaybackRate()` | `Future<void>` | 设置速度   |
| `release()`         | `Future<void>` | 释放资源   |
| `dispose()`         | `void`         | 销毁播放器 |

## 附录 B：常见问题与解决方案

### Q1：音频无法播放

**检查项**：

1. 网络权限是否配置
2. 音频格式是否支持
3. URL 是否可访问

### Q2：Web 平台音频问题

**解决方案**：

```dart
// Web 平台需要特殊处理
if (kIsWeb) {
  await player.play('https://example.com/audio.mp3');
}
```

### Q3：内存泄漏

**解决方案**：

```dart
@override
void dispose() {
  player.dispose();
  super.dispose();
}
```

## 附录 C：参考资源

- [audioplayers 官方文档](https://pub.dev/packages/audioplayers)
- [audioplayers API 参考](https://pub.dev/documentation/audioplayers/latest/)
- [Blue Fire 团队](https://github.com/bluefireteam)
