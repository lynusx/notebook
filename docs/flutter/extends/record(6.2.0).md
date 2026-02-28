# record 6.2.0 详解与实战

## 第1章 认识 record

### 1.1 什么是 record

`record` 是一个专注于音频录制的 Flutter 插件，由 Baptiste Dupuch 开发。它提供了简洁的 API 来从麦克风录制音频到文件或流，支持多种编解码器、比特率和采样率选项。

与其他 Flutter 音频库相比，`record` 的核心优势在于：

| 特性       | record          | 其他库   |
| ---------- | --------------- | -------- |
| 专注录制   | ✅ 专业         | 部分支持 |
| 流式录制   | ✅ 支持         | 部分支持 |
| 编解码器   | 多种            | 有限     |
| API 简洁度 | ⭐⭐⭐⭐⭐ 极简 | 中等     |
| 平台支持   | 全面            | 部分     |
| 后台录制   | ✅ 支持         | 有限     |

Flutter 框架小知识

**音频录制库的选择**

Flutter 生态中有多个音频录制库：

1. **record**：专注录制，API 简洁，推荐用于纯录制场景
2. **flutter_sound**：录制+播放，功能全面
3. **audio_waveforms**：录制+波形可视化

对于只需要录制功能的应用，`record` 是最佳选择。

### 1.2 record 6.2.0 的新特性

版本 6.2.0 是 `record` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **文件录制**：录制到指定文件路径
- **流式录制**：实时获取音频数据流
- **多种编解码器**：AAC、Opus、PCM、WAV 等
- **可配置参数**：比特率、采样率、声道数
- **权限管理**：内置权限检查和请求
- **振幅监测**：实时获取录音振幅
- **暂停/恢复**：支持录制暂停和恢复
- **后台录制**：支持后台持续录制

#### 1.2.2 平台支持

| 平台    | 支持状态 | 最低版本     |
| ------- | -------- | ------------ |
| Android | ✅       | API 21+      |
| iOS     | ✅       | iOS 11.0+    |
| macOS   | ✅       | macOS 10.15+ |
| Windows | ✅       | Windows 10+  |
| Linux   | ✅       | 任意版本     |
| Web     | ✅       | 现代浏览器   |

#### 1.2.3 编解码器支持

| 编解码器 | Android | iOS | macOS | Windows | Linux | Web |
| -------- | ------- | --- | ----- | ------- | ----- | --- |
| AAC      | ✅      | ✅  | ✅    | ❌      | ❌    | ❌  |
| Opus     | ✅      | ✅  | ❌    | ❌      | ❌    | ✅  |
| PCM      | ✅      | ✅  | ✅    | ✅      | ✅    | ✅  |
| WAV      | ✅      | ✅  | ✅    | ✅      | ✅    | ✅  |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  record: ^6.2.0
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
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加：

```xml
<key>NSMicrophoneUsageDescription</key>
<string>需要麦克风权限来录制音频</string>
```

**macOS 配置**：

在 `macos/Runner/Info.plist` 中添加：

```xml
<key>NSMicrophoneUsageDescription</key>
<string>需要麦克风权限来录制音频</string>
```

Dart Tips 语法小贴士

**路径处理**

使用 `path_provider` 获取合适的存储路径：

```yaml
dependencies:
  path_provider: ^2.1.0
```

```dart
import 'package:path_provider/path_provider.dart';

// 获取应用文档目录
final directory = await getApplicationDocumentsDirectory();
final path = '${directory.path}/recording.m4a';
```

## 第2章 AudioRecorder 类详解

`AudioRecorder` 是 `record` 包的核心类，负责音频录制功能。

### 2.1 构造函数

```dart
AudioRecorder()
```

### 2.2 核心方法详解

#### 2.2.1 权限检查

```dart
// 检查是否有权限
Future<bool> hasPermission({bool request = true})
```

**参数说明**：

| 参数      | 类型   | 默认值 | 说明                       |
| --------- | ------ | ------ | -------------------------- |
| `request` | `bool` | `true` | 如果没有权限，是否请求权限 |

**使用示例**：

```dart
final record = AudioRecorder();

// 检查并请求权限
if (await record.hasPermission()) {
  // 有权限，可以录制
}

// 仅检查权限，不请求
final hasPermission = await record.hasPermission(request: false);
if (!hasPermission) {
  // 显示权限请求 UI
}
```

#### 2.2.2 开始录制

```dart
// 录制到文件
Future<void> start(
  RecordConfig config, {
  required String path,
})

// 录制到流
Future<Stream<Uint8List>> startStream(
  RecordConfig config,
)
```

**RecordConfig 配置**：

```dart
RecordConfig({
  AudioEncoder encoder = AudioEncoder.aacLc,  // 编码器
  int bitRate = 128000,                         // 比特率
  int sampleRate = 44100,                       // 采样率
  int numChannels = 2,                          // 声道数
  bool autoGain = false,                        // 自动增益
  bool echoCancel = false,                      // 回声消除
  bool noiseSuppress = false,                   // 噪声抑制
})
```

**AudioEncoder 枚举**：

| 值                       | 说明           |
| ------------------------ | -------------- |
| `AudioEncoder.aacLc`     | AAC 低复杂度   |
| `AudioEncoder.aacEld`    | AAC 增强低延迟 |
| `AudioEncoder.aacHe`     | AAC 高效率     |
| `AudioEncoder.opus`      | Opus 编码      |
| `AudioEncoder.flac`      | FLAC 无损      |
| `AudioEncoder.pcm16bits` | PCM 16位       |
| `AudioEncoder.wav`       | WAV 格式       |
| `AudioEncoder.vorbisOgg` | Ogg Vorbis     |
| `AudioEncoder.amrNb`     | AMR-NB         |
| `AudioEncoder.amrWb`     | AMR-WB         |

**使用示例**：

```dart
// 录制到文件
await record.start(
  const RecordConfig(
    encoder: AudioEncoder.aacLc,
    bitRate: 128000,
    sampleRate: 44100,
    numChannels: 2,
  ),
  path: '/path/to/recording.m4a',
);

// 录制到流
final stream = await record.startStream(
  const RecordConfig(encoder: AudioEncoder.pcm16bits),
);

stream.listen((data) {
  // 处理音频数据
  print('收到 ${data.length} 字节数据');
});
```

#### 2.2.3 停止录制

```dart
// 停止录制（返回文件路径）
Future<String?> stop()

// 取消录制（删除文件）
Future<void> cancel()
```

**使用示例**：

```dart
// 停止录制
final path = await record.stop();
print('录制完成: $path');

// 取消录制
await record.cancel();
```

#### 2.2.4 暂停/恢复

```dart
// 暂停录制
Future<void> pause()

// 恢复录制
Future<void> resume()

// 检查是否正在录制
Future<bool> isRecording()

// 检查是否已暂停
Future<bool> isPaused()
```

**使用示例**：

```dart
// 暂停
await record.pause();

// 检查状态
if (await record.isPaused()) {
  print('已暂停');
}

// 恢复
await record.resume();

// 检查状态
if (await record.isRecording()) {
  print('正在录制');
}
```

#### 2.2.5 获取振幅

```dart
// 获取当前振幅
Future<Amplitude> getAmplitude()
```

**Amplitude 类**：

| 属性      | 类型     | 说明           |
| --------- | -------- | -------------- |
| `current` | `double` | 当前振幅（dB） |
| `max`     | `double` | 最大振幅（dB） |

**使用示例**：

```dart
// 获取振幅
final amplitude = await record.getAmplitude();
print('当前振幅: ${amplitude.current} dB');
print('最大振幅: ${amplitude.max} dB');
```

#### 2.2.6 列出输入设备

```dart
// 列出可用的音频输入设备
Future<List<InputDevice>> listInputDevices()
```

**使用示例**：

```dart
// 列出输入设备
final devices = await record.listInputDevices();
for (final device in devices) {
  print('设备: ${device.label}, ID: ${device.id}');
}
```

#### 2.2.7 资源释放

```dart
// 释放资源
Future<void> dispose()
```

**使用示例**：

```dart
@override
void dispose() {
  record.dispose();
  super.dispose();
}
```

### 2.3 振幅流

```dart
// 获取振幅流
Stream<Amplitude> get onAmplitude
```

**使用示例**：

```dart
// 监听振幅变化
record.onAmplitude.listen((amplitude) {
  print('振幅: ${amplitude.current} dB');
});
```

## 第3章 record 实战应用

### 3.1 录音机应用

```dart
class RecorderScreen extends StatefulWidget {
  @override
  _RecorderScreenState createState() => _RecorderScreenState();
}

class _RecorderScreenState extends State<RecorderScreen> {
  final record = AudioRecorder();
  bool isRecording = false;
  bool isPaused = false;
  String? recordingPath;
  double amplitude = -100.0;
  Timer? amplitudeTimer;

  @override
  void initState() {
    super.initState();
    checkPermission();
  }

  Future<void> checkPermission() async {
    final hasPermission = await record.hasPermission();
    if (!hasPermission) {
      // 显示权限被拒绝提示
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('需要麦克风权限')),
      );
    }
  }

  Future<void> startRecording() async {
    final directory = await getApplicationDocumentsDirectory();
    final path = '${directory.path}/recording_${DateTime.now().millisecondsSinceEpoch}.m4a';

    await record.start(
      const RecordConfig(
        encoder: AudioEncoder.aacLc,
        bitRate: 128000,
        sampleRate: 44100,
        numChannels: 2,
      ),
      path: path,
    );

    // 开始监听振幅
    amplitudeTimer = Timer.periodic(Duration(milliseconds: 100), (_) async {
      final amp = await record.getAmplitude();
      setState(() {
        amplitude = amp.current;
      });
    });

    setState(() {
      isRecording = true;
      recordingPath = path;
    });
  }

  Future<void> stopRecording() async {
    amplitudeTimer?.cancel();
    final path = await record.stop();
    setState(() {
      isRecording = false;
      isPaused = false;
      amplitude = -100.0;
    });
  }

  Future<void> pauseRecording() async {
    await record.pause();
    amplitudeTimer?.cancel();
    setState(() {
      isPaused = true;
    });
  }

  Future<void> resumeRecording() async {
    await record.resume();
    amplitudeTimer = Timer.periodic(Duration(milliseconds: 100), (_) async {
      final amp = await record.getAmplitude();
      setState(() {
        amplitude = amp.current;
      });
    });
    setState(() {
      isPaused = false;
    });
  }

  Future<void> cancelRecording() async {
    amplitudeTimer?.cancel();
    await record.cancel();
    setState(() {
      isRecording = false;
      isPaused = false;
      amplitude = -100.0;
    });
  }

  @override
  void dispose() {
    amplitudeTimer?.cancel();
    record.dispose();
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
            // 振幅可视化
            Container(
              width: 200,
              height: 100,
              child: CustomPaint(
                painter: AmplitudePainter(amplitude),
              ),
            ),
            SizedBox(height: 32),

            // 振幅数值
            Text(
              '${amplitude.toStringAsFixed(1)} dB',
              style: TextStyle(fontSize: 24),
            ),
            SizedBox(height: 32),

            // 控制按钮
            if (!isRecording)
              FloatingActionButton.large(
                onPressed: startRecording,
                child: Icon(Icons.mic),
                backgroundColor: Colors.red,
              )
            else
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
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
                  SizedBox(width: 16),
                  // 取消
                  FloatingActionButton(
                    onPressed: cancelRecording,
                    child: Icon(Icons.delete),
                    backgroundColor: Colors.grey,
                  ),
                ],
              ),

            // 播放录制的音频
            if (recordingPath != null && !isRecording) ...[
              SizedBox(height: 32),
              ElevatedButton.icon(
                onPressed: () => playRecording(recordingPath!),
                icon: Icon(Icons.play_arrow),
                label: Text('播放录音'),
              ),
            ],
          ],
        ),
      ),
    );
  }

  Future<void> playRecording(String path) async {
    // 使用 audioplayers 或其他播放器播放
    final player = AudioPlayer();
    await player.play(DeviceFileSource(path));
  }
}

// 振幅可视化
class AmplitudePainter extends CustomPainter {
  final double amplitude;

  AmplitudePainter(this.amplitude);

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 4
      ..strokeCap = StrokeCap.round;

    // 将 dB 转换为 0-1 范围
    final normalized = ((amplitude + 60) / 60).clamp(0.0, 1.0);
    final barHeight = size.height * normalized;

    canvas.drawLine(
      Offset(size.width / 2, size.height),
      Offset(size.width / 2, size.height - barHeight),
      paint,
    );
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

### 3.2 流式录制

```dart
class StreamRecorder extends StatefulWidget {
  @override
  _StreamRecorderState createState() => _StreamRecorderState();
}

class _StreamRecorderState extends State<StreamRecorder> {
  final record = AudioRecorder();
  Stream<Uint8List>? audioStream;
  List<int> audioData = [];
  bool isRecording = false;

  Future<void> startStreaming() async {
    if (!await record.hasPermission()) {
      return;
    }

    audioStream = await record.startStream(
      const RecordConfig(encoder: AudioEncoder.pcm16bits),
    );

    audioStream!.listen(
      (data) {
        setState(() {
          audioData.addAll(data);
        });
        print('收到 ${data.length} 字节');
      },
      onError: (error) {
        print('流错误: $error');
      },
      onDone: () {
        print('流结束');
      },
    );

    setState(() {
      isRecording = true;
    });
  }

  Future<void> stopStreaming() async {
    await record.stop();
    setState(() {
      isRecording = false;
    });
  }

  @override
  void dispose() {
    record.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('流式录制')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('已收集 ${audioData.length} 字节'),
            SizedBox(height: 32),
            ElevatedButton(
              onPressed: isRecording ? stopStreaming : startStreaming,
              child: Text(isRecording ? '停止' : '开始流式录制'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 3.3 语音消息录制

```dart
class VoiceRecorderButton extends StatefulWidget {
  final Function(String path, Duration duration) onRecordComplete;

  const VoiceRecorderButton({super.key, required this.onRecordComplete});

  @override
  _VoiceRecorderButtonState createState() => _VoiceRecorderButtonState();
}

class _VoiceRecorderButtonState extends State<VoiceRecorderButton> {
  final record = AudioRecorder();
  bool isRecording = false;
  DateTime? startTime;
  Timer? timer;
  Duration duration = Duration.zero;
  double amplitude = -100.0;

  Future<void> startRecording() async {
    if (!await record.hasPermission()) {
      return;
    }

    final directory = await getTemporaryDirectory();
    final path = '${directory.path}/voice_${DateTime.now().millisecondsSinceEpoch}.m4a';

    await record.start(
      const RecordConfig(
        encoder: AudioEncoder.aacLc,
        bitRate: 64000,  // 语音用较低比特率
        sampleRate: 22050,
        numChannels: 1,  // 单声道
      ),
      path: path,
    );

    startTime = DateTime.now();
    timer = Timer.periodic(Duration(milliseconds: 100), (_) async {
      final amp = await record.getAmplitude();
      setState(() {
        duration = DateTime.now().difference(startTime!);
        amplitude = amp.current;
      });
    });

    setState(() {
      isRecording = true;
    });
  }

  Future<void> stopRecording() async {
    timer?.cancel();
    final path = await record.stop();
    setState(() {
      isRecording = false;
    });

    if (path != null && duration.inSeconds > 1) {
      widget.onRecordComplete(path, duration);
    }
  }

  @override
  void dispose() {
    timer?.cancel();
    record.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onLongPressStart: (_) => startRecording(),
      onLongPressEnd: (_) => stopRecording(),
      child: AnimatedContainer(
        duration: Duration(milliseconds: 200),
        padding: EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: isRecording ? Colors.red : Colors.blue,
          shape: BoxShape.circle,
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(
              isRecording ? Icons.mic : Icons.mic_none,
              color: Colors.white,
              size: 32,
            ),
            if (isRecording) ...[
              SizedBox(height: 8),
              Text(
                '${duration.inSeconds}.${(duration.inMilliseconds % 1000) ~/ 100}s',
                style: TextStyle(color: Colors.white),
              ),
            ],
          ],
        ),
      ),
    );
  }
}

// 使用
VoiceRecorderButton(
  onRecordComplete: (path, duration) {
    print('录音完成: $path, 时长: $duration');
    // 发送语音消息
  },
)
```

### 3.4 音频波形录制

```dart
class WaveformRecorder extends StatefulWidget {
  @override
  _WaveformRecorderState createState() => _WaveformRecorderState();
}

class _WaveformRecorderState extends State<WaveformRecorder> {
  final record = AudioRecorder();
  final List<double> amplitudes = [];
  bool isRecording = false;
  Timer? timer;

  Future<void> startRecording() async {
    if (!await record.hasPermission()) {
      return;
    }

    final directory = await getApplicationDocumentsDirectory();
    final path = '${directory.path}/waveform_recording.m4a';

    await record.start(
      const RecordConfig(encoder: AudioEncoder.aacLc),
      path: path,
    );

    timer = Timer.periodic(Duration(milliseconds: 50), (_) async {
      final amp = await record.getAmplitude();
      setState(() {
        amplitudes.add(amp.current);
        if (amplitudes.length > 100) {
          amplitudes.removeAt(0);
        }
      });
    });

    setState(() {
      isRecording = true;
    });
  }

  Future<void> stopRecording() async {
    timer?.cancel();
    await record.stop();
    setState(() {
      isRecording = false;
    });
  }

  @override
  void dispose() {
    timer?.cancel();
    record.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('波形录制')),
      body: Column(
        children: [
          // 波形显示
          Expanded(
            child: CustomPaint(
              size: Size(double.infinity, double.infinity),
              painter: WaveformPainter(amplitudes),
            ),
          ),

          // 控制按钮
          Padding(
            padding: EdgeInsets.all(16),
            child: ElevatedButton(
              onPressed: isRecording ? stopRecording : startRecording,
              child: Text(isRecording ? '停止' : '开始录制'),
            ),
          ),
        ],
      ),
    );
  }
}

class WaveformPainter extends CustomPainter {
  final List<double> amplitudes;

  WaveformPainter(this.amplitudes);

  @override
  void paint(Canvas canvas, Size size) {
    if (amplitudes.isEmpty) return;

    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2
      ..style = PaintingStyle.stroke;

    final path = Path();
    final barWidth = size.width / amplitudes.length;

    for (int i = 0; i < amplitudes.length; i++) {
      final normalized = ((amplitudes[i] + 60) / 60).clamp(0.0, 1.0);
      final barHeight = size.height * normalized;
      final x = i * barWidth;
      final y = (size.height - barHeight) / 2;

      if (i == 0) {
        path.moveTo(x, y + barHeight / 2);
      } else {
        path.lineTo(x, y + barHeight / 2);
      }
    }

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

## 附录 A：AudioRecorder 方法速查表

| 方法                 | 返回值                      | 说明           |
| -------------------- | --------------------------- | -------------- |
| `hasPermission()`    | `Future<bool>`              | 检查权限       |
| `start()`            | `Future<void>`              | 开始录制到文件 |
| `startStream()`      | `Future<Stream<Uint8List>>` | 开始流式录制   |
| `stop()`             | `Future<String?>`           | 停止录制       |
| `cancel()`           | `Future<void>`              | 取消录制       |
| `pause()`            | `Future<void>`              | 暂停录制       |
| `resume()`           | `Future<void>`              | 恢复录制       |
| `isRecording()`      | `Future<bool>`              | 是否正在录制   |
| `isPaused()`         | `Future<bool>`              | 是否已暂停     |
| `getAmplitude()`     | `Future<Amplitude>`         | 获取振幅       |
| `listInputDevices()` | `Future<List<InputDevice>>` | 列出输入设备   |
| `dispose()`          | `Future<void>`              | 释放资源       |

## 附录 B：RecordConfig 参数速查表

| 参数            | 类型           | 默认值   | 说明          |
| --------------- | -------------- | -------- | ------------- |
| `encoder`       | `AudioEncoder` | `aacLc`  | 音频编码器    |
| `bitRate`       | `int`          | `128000` | 比特率（bps） |
| `sampleRate`    | `int`          | `44100`  | 采样率（Hz）  |
| `numChannels`   | `int`          | `2`      | 声道数        |
| `autoGain`      | `bool`         | `false`  | 自动增益控制  |
| `echoCancel`    | `bool`         | `false`  | 回声消除      |
| `noiseSuppress` | `bool`         | `false`  | 噪声抑制      |

## 附录 C：参考资源

- [record 官方文档](https://pub.dev/packages/record)
- [record API 参考](https://pub.dev/documentation/record/latest/)
- [GitHub 仓库](https://github.com/llfbandit/record)
