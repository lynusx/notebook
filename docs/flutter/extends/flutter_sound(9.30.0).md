# flutter_sound 9.30.0 详解与实战

## 第1章 认识 flutter_sound

### 1.1 什么是 flutter_sound

`flutter_sound`（代号 τ - tau）是由 Canardoux 团队开发的全功能 Flutter 音频插件，支持音频录制和播放，提供了一套完整的音频处理 API。它是 Flutter 生态中最全面的音频解决方案之一，特别适合需要同时处理录制和播放的应用。

与其他 Flutter 音频库相比，`flutter_sound` 的核心优势在于：

| 特性      | flutter_sound | 其他库   |
| --------- | ------------- | -------- |
| 录制+播放 | ✅ 完整支持   | 部分支持 |
| 编解码器  | 20+ 种        | 有限     |
| 流式处理  | ✅ 支持       | 部分支持 |
| 后台录制  | ✅ 支持       | 有限     |
| 音频转换  | ✅ FFmpeg     | ❌       |
| 复杂度    | 较高          | 中等     |

Flutter 框架小知识

**flutter_sound 的架构**

`flutter_sound` 采用分层架构设计：

1. **FlutterSoundPlayer**：音频播放器
2. **FlutterSoundRecorder**：音频录制器
3. **FlutterSoundHelper**：音频辅助工具（FFmpeg 集成）

这种设计使得开发者可以根据需求选择使用播放器、录制器或两者都用。

### 1.2 flutter_sound 9.30.0 的新特性

版本 9.30.0 是 `flutter_sound` 的重要版本，主要包含以下特性：

#### 1.2.1 核心特性

- **音频播放**：支持多种格式的音频播放
- **音频录制**：支持多种格式的音频录制
- **后台播放/录制**：支持 iOS 和 Android 后台操作
- **流式处理**：支持实时音频流处理
- **FFmpeg 集成**：支持音频格式转换
- **播放速度控制**：支持变速播放
- **音量控制**：支持独立音量设置
- **音频可视化**：支持实时音频数据获取

#### 1.2.2 支持的编解码器

| 编解码器   | Android | iOS | Web |
| ---------- | ------- | --- | --- |
| AAC        | ✅      | ✅  | ✅  |
| MP3        | ✅      | ✅  | ✅  |
| OGG/Vorbis | ✅      | ✅  | ❌  |
| Opus       | ✅      | ✅  | ✅  |
| PCM        | ✅      | ✅  | ✅  |
| WAV        | ✅      | ✅  | ✅  |
| FLAC       | ✅      | ✅  | ❌  |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_sound: ^9.30.0
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/src/main/AndroidManifest.xml` 中添加权限：

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加：

```xml
<key>NSMicrophoneUsageDescription</key>
<string>需要麦克风权限来录制音频</string>

<key>UIBackgroundModes</key>
<array>
    <string>audio</string>
</array>
```

Dart Tips 语法小贴士

**权限处理**

使用 `flutter_sound` 前需要处理权限请求：

```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> requestPermissions() async {
  // 请求麦克风权限
  final status = await Permission.microphone.request();
  return status.isGranted;
}
```

## 第2章 FlutterSoundPlayer 类详解

`FlutterSoundPlayer` 是 `flutter_sound` 的音频播放器类。

### 2.1 构造函数

```dart
FlutterSoundPlayer({
  Level logLevel = Level.debug,  // 日志级别
})
```

### 2.2 核心属性详解

| 属性        | 类型   | 说明             |
| ----------- | ------ | ---------------- |
| `isOpen`    | `bool` | 播放器是否已打开 |
| `isPlaying` | `bool` | 是否正在播放     |
| `isPaused`  | `bool` | 是否已暂停       |
| `isStopped` | `bool` | 是否已停止       |

### 2.3 核心方法详解

#### 2.3.1 打开/关闭播放器

```dart
// 打开播放器
Future<void> openPlayer({
  bool withUI = false,  // 是否带 UI
})

// 关闭播放器
Future<void> closePlayer()
```

**使用示例**：

```dart
final player = FlutterSoundPlayer();

// 打开播放器
await player.openPlayer();

// 使用播放器...

// 关闭播放器
await player.closePlayer();
```

#### 2.3.2 播放控制方法

```dart
// 开始播放
Future<void> startPlayer({
  String? fromURI,                    // 从 URI 播放
  Uint8List? fromDataBuffer,          // 从数据缓冲区播放
  StreamController<Food>? toStream,   // 输出到流
  Codec codec = Codec.aacADTS,        // 编解码器
  int sampleRate = 16000,             // 采样率
  int numChannels = 1,                // 声道数
  TWhenFinished? whenFinished,        // 完成回调
})

// 停止播放
Future<void> stopPlayer()

// 暂停播放
Future<void> pausePlayer()

// 恢复播放
Future<void> resumePlayer()

// 跳转到指定位置
Future<void> seekToPlayer(Duration duration)
```

**使用示例**：

```dart
// 播放网络音频
await player.startPlayer(
  fromURI: 'https://example.com/audio.mp3',
  whenFinished: () {
    print('播放完成');
  },
);

// 播放本地文件
await player.startPlayer(
  fromURI: '/path/to/audio.mp3',
);

// 暂停
await player.pausePlayer();

// 恢复
await player.resumePlayer();

// 停止
await player.stopPlayer();
```

#### 2.3.3 播放设置方法

```dart
// 设置音量
Future<void> setVolume(double volume)  // 0.0 - 1.0

// 设置播放速度
Future<void> setSpeed(double speed)    // 0.5 - 2.0

// 设置订阅间隔
Future<void> setSubscriptionDuration(Duration duration)
```

**使用示例**：

```dart
// 设置音量
await player.setVolume(0.5);

// 设置 1.5 倍速
await player.setSpeed(1.5);
```

### 2.4 事件监听

```dart
// 监听播放进度
player.onProgress!.listen((event) {
  print('位置: ${event.position}, 时长: ${event.duration}');
});
```

## 第3章 FlutterSoundRecorder 类详解

`FlutterSoundRecorder` 是 `flutter_sound` 的音频录制器类。

### 3.1 构造函数

```dart
FlutterSoundRecorder({
  Level logLevel = Level.debug,
})
```

### 3.2 核心属性详解

| 属性          | 类型   | 说明             |
| ------------- | ------ | ---------------- |
| `isOpen`      | `bool` | 录制器是否已打开 |
| `isRecording` | `bool` | 是否正在录制     |
| `isPaused`    | `bool` | 是否已暂停       |
| `isStopped`   | `bool` | 是否已停止       |

### 3.3 核心方法详解

#### 3.3.1 打开/关闭录制器

```dart
// 打开录制器
Future<void> openRecorder({
  bool isBGService = false,  // 是否作为后台服务
})

// 关闭录制器
Future<void> closeRecorder()
```

**使用示例**：

```dart
final recorder = FlutterSoundRecorder();

// 打开录制器
await recorder.openRecorder();

// 使用录制器...

// 关闭录制器
await recorder.closeRecorder();
```

#### 3.3.2 录制控制方法

```dart
// 开始录制
Future<void> startRecorder({
  String? toFile,                     // 输出文件路径
  StreamController<Food>? toStream,   // 输出到流
  Codec codec = Codec.aacADTS,        // 编解码器
  int sampleRate = 16000,             // 采样率
  int numChannels = 1,                // 声道数
  int bitRate = 16000,                // 比特率
  AudioSource audioSource = AudioSource.defaultSource,  // 音频源
})

// 停止录制
Future<String?> stopRecorder()

// 暂停录制
Future<void> pauseRecorder()

// 恢复录制
Future<void> resumeRecorder()
```

**使用示例**：

```dart
// 开始录制到文件
await recorder.startRecorder(
  toFile: '/path/to/recording.aac',
  codec: Codec.aacADTS,
);

// 暂停录制
await recorder.pauseRecorder();

// 恢复录制
await recorder.resumeRecorder();

// 停止录制
final path = await recorder.stopRecorder();
print('录制文件: $path');
```

### 3.4 事件监听

```dart
// 监听录制进度
recorder.onProgress!.listen((event) {
  print('时长: ${event.duration}, 分贝: ${event.decibels}');
});
```

## 第4章 FlutterSoundHelper 类详解

`FlutterSoundHelper` 提供音频辅助功能，包括 FFmpeg 集成。

### 4.1 获取实例

```dart
final helper = FlutterSoundHelper();
```

### 4.2 核心方法详解

```dart
// 获取音频时长
Future<Duration> duration(String filePath)

// 获取 FFmpeg 返回码
int getLastFFmpegReturnCode()

// 获取 FFmpeg 命令输出
String getLastFFmpegCommandOutput()

// 获取媒体信息
Future<Map<String, dynamic>>FFmpegGetMediaInformation(String filePath)
```

**使用示例**：

```dart
// 获取音频时长
final duration = await helper.duration('/path/to/audio.mp3');
print('时长: $duration');

// 获取媒体信息
final info = await helper.FFmpegGetMediaInformation('/path/to/audio.mp3');
print('媒体信息: $info');
```

## 第5章 flutter_sound 实战应用

### 5.1 录音机应用

```dart
class RecorderScreen extends StatefulWidget {
  @override
  _RecorderScreenState createState() => _RecorderScreenState();
}

class _RecorderScreenState extends State<RecorderScreen> {
  final recorder = FlutterSoundRecorder();
  bool isRecording = false;
  bool isPaused = false;
  String? recordingPath;
  Duration duration = Duration.zero;

  @override
  void initState() {
    super.initState();
    initRecorder();
  }

  Future<void> initRecorder() async {
    // 请求权限
    final status = await Permission.microphone.request();
    if (status != PermissionStatus.granted) {
      throw RecordingPermissionException('麦克风权限被拒绝');
    }

    // 打开录制器
    await recorder.openRecorder();

    // 监听进度
    recorder.onProgress!.listen((event) {
      setState(() {
        duration = event.duration;
      });
    });
  }

  Future<void> startRecording() async {
    final directory = await getApplicationDocumentsDirectory();
    final path = '${directory.path}/recording_${DateTime.now().millisecondsSinceEpoch}.aac';

    await recorder.startRecorder(
      toFile: path,
      codec: Codec.aacADTS,
    );

    setState(() {
      isRecording = true;
      recordingPath = path;
    });
  }

  Future<void> stopRecording() async {
    await recorder.stopRecorder();
    setState(() {
      isRecording = false;
      isPaused = false;
    });
  }

  Future<void> pauseRecording() async {
    await recorder.pauseRecorder();
    setState(() {
      isPaused = true;
    });
  }

  Future<void> resumeRecording() async {
    await recorder.resumeRecorder();
    setState(() {
      isPaused = false;
    });
  }

  @override
  void dispose() {
    recorder.closeRecorder();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('录音机')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 时长显示
            Text(
              duration.toString().split('.').first,
              style: TextStyle(fontSize: 48),
            ),
            SizedBox(height: 32),

            // 控制按钮
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                if (!isRecording)
                  FloatingActionButton(
                    onPressed: startRecording,
                    child: Icon(Icons.mic),
                    backgroundColor: Colors.red,
                  )
                else ...[
                  // 暂停/恢复
                  FloatingActionButton(
                    onPressed: isPaused ? resumeRecording : pauseRecording,
                    child: Icon(isPaused ? Icons.play_arrow : Icons.pause),
                  ),
                  SizedBox(width: 16),
                  // 停止
                  FloatingActionButton(
                    onPressed: stopRecording,
                    child: Icon(Icons.stop),
                    backgroundColor: Colors.red,
                  ),
                ],
              ],
            ),

            // 播放录制的音频
            if (recordingPath != null && !isRecording)
              ElevatedButton(
                onPressed: () => playRecording(recordingPath!),
                child: Text('播放录音'),
              ),
          ],
        ),
      ),
    );
  }

  Future<void> playRecording(String path) async {
    final player = FlutterSoundPlayer();
    await player.openPlayer();
    await player.startPlayer(fromURI: path);
  }
}
```

### 5.2 音频播放器

```dart
class AudioPlayerWidget extends StatefulWidget {
  final String audioPath;

  const AudioPlayerWidget({super.key, required this.audioPath});

  @override
  _AudioPlayerWidgetState createState() => _AudioPlayerWidgetState();
}

class _AudioPlayerWidgetState extends State<AudioPlayerWidget> {
  final player = FlutterSoundPlayer();
  bool isPlaying = false;
  bool isPaused = false;
  Duration position = Duration.zero;
  Duration duration = Duration.zero;

  @override
  void initState() {
    super.initState();
    initPlayer();
  }

  Future<void> initPlayer() async {
    await player.openPlayer();

    // 设置订阅间隔
    await player.setSubscriptionDuration(Duration(milliseconds: 100));

    // 监听进度
    player.onProgress!.listen((event) {
      setState(() {
        position = event.position;
        duration = event.duration;
      });
    });
  }

  Future<void> play() async {
    await player.startPlayer(
      fromURI: widget.audioPath,
      whenFinished: () {
        setState(() {
          isPlaying = false;
          position = Duration.zero;
        });
      },
    );
    setState(() {
      isPlaying = true;
      isPaused = false;
    });
  }

  Future<void> pause() async {
    await player.pausePlayer();
    setState(() {
      isPaused = true;
    });
  }

  Future<void> resume() async {
    await player.resumePlayer();
    setState(() {
      isPaused = false;
    });
  }

  Future<void> stop() async {
    await player.stopPlayer();
    setState(() {
      isPlaying = false;
      isPaused = false;
      position = Duration.zero;
    });
  }

  Future<void> seek(double value) async {
    final newPosition = Duration(
      milliseconds: (value * duration.inMilliseconds).toInt(),
    );
    await player.seekToPlayer(newPosition);
  }

  @override
  void dispose() {
    player.closePlayer();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 进度条
        Slider(
          value: duration.inMilliseconds > 0
              ? position.inMilliseconds / duration.inMilliseconds
              : 0,
          onChanged: seek,
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
              icon: Icon(Icons.stop),
              onPressed: isPlaying ? stop : null,
            ),
            IconButton(
              icon: Icon(isPlaying && !isPaused ? Icons.pause : Icons.play_arrow),
              iconSize: 48,
              onPressed: () {
                if (!isPlaying) {
                  play();
                } else if (isPaused) {
                  resume();
                } else {
                  pause();
                }
              },
            ),
          ],
        ),
      ],
    );
  }
}
```

### 5.3 语音消息组件

```dart
class VoiceMessage extends StatefulWidget {
  final String audioPath;
  final Duration duration;

  const VoiceMessage({
    super.key,
    required this.audioPath,
    required this.duration,
  });

  @override
  _VoiceMessageState createState() => _VoiceMessageState();
}

class _VoiceMessageState extends State<VoiceMessage> {
  final player = FlutterSoundPlayer();
  bool isPlaying = false;
  double progress = 0.0;

  @override
  void initState() {
    super.initState();
    player.openPlayer();
  }

  Future<void> togglePlay() async {
    if (isPlaying) {
      await player.stopPlayer();
      setState(() {
        isPlaying = false;
        progress = 0.0;
      });
    } else {
      await player.startPlayer(
        fromURI: widget.audioPath,
        whenFinished: () {
          setState(() {
            isPlaying = false;
            progress = 0.0;
          });
        },
      );

      // 监听进度
      player.onProgress!.listen((event) {
        setState(() {
          progress = event.position.inMilliseconds /
              event.duration.inMilliseconds;
        });
      });

      setState(() {
        isPlaying = true;
      });
    }
  }

  @override
  void dispose() {
    player.closePlayer();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: togglePlay,
      child: Container(
        padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        decoration: BoxDecoration(
          color: Colors.grey[200],
          borderRadius: BorderRadius.circular(20),
        ),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(isPlaying ? Icons.stop : Icons.play_arrow),
            SizedBox(width: 8),
            // 波形可视化
            Container(
              width: 100,
              height: 30,
              child: CustomPaint(
                painter: WaveformPainter(progress),
              ),
            ),
            SizedBox(width: 8),
            Text(
              '${widget.duration.inSeconds}"',
              style: TextStyle(fontSize: 12),
            ),
          ],
        ),
      ),
    );
  }
}

class WaveformPainter extends CustomPainter {
  final double progress;

  WaveformPainter(this.progress);

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2;

    final barCount = 20;
    final barWidth = size.width / barCount;

    for (int i = 0; i < barCount; i++) {
      final barHeight = (i % 3 + 1) * size.height / 4;
      final isPlayed = i / barCount <= progress;

      canvas.drawRect(
        Rect.fromLTWH(
          i * barWidth + 1,
          (size.height - barHeight) / 2,
          barWidth - 2,
          barHeight,
        ),
        Paint()
          ..color = isPlayed ? Colors.blue : Colors.grey
          ..strokeWidth = 2,
      );
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

## 附录 A：FlutterSoundPlayer 方法速查表

| 方法             | 返回值         | 说明       |
| ---------------- | -------------- | ---------- |
| `openPlayer()`   | `Future<void>` | 打开播放器 |
| `closePlayer()`  | `Future<void>` | 关闭播放器 |
| `startPlayer()`  | `Future<void>` | 开始播放   |
| `stopPlayer()`   | `Future<void>` | 停止播放   |
| `pausePlayer()`  | `Future<void>` | 暂停播放   |
| `resumePlayer()` | `Future<void>` | 恢复播放   |
| `seekToPlayer()` | `Future<void>` | 跳转位置   |
| `setVolume()`    | `Future<void>` | 设置音量   |
| `setSpeed()`     | `Future<void>` | 设置速度   |

## 附录 B：FlutterSoundRecorder 方法速查表

| 方法               | 返回值            | 说明       |
| ------------------ | ----------------- | ---------- |
| `openRecorder()`   | `Future<void>`    | 打开录制器 |
| `closeRecorder()`  | `Future<void>`    | 关闭录制器 |
| `startRecorder()`  | `Future<void>`    | 开始录制   |
| `stopRecorder()`   | `Future<String?>` | 停止录制   |
| `pauseRecorder()`  | `Future<void>`    | 暂停录制   |
| `resumeRecorder()` | `Future<void>`    | 恢复录制   |

## 附录 C：参考资源

- [flutter_sound 官方文档](https://flutter-sound.canardoux.xyz/)
- [flutter_sound API 参考](https://pub.dev/documentation/flutter_sound/latest/)
- [FFmpeg 文档](https://ffmpeg.org/documentation.html)
