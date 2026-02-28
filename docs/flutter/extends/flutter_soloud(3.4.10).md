# flutter_soloud 3.4.10 详解与实战

## 第1章 认识 flutter_soloud

### 1.1 什么是 flutter_soloud

`flutter_soloud` 是一个基于 SoLoud C++ 音频引擎的高性能 Flutter 音频插件，专为游戏和沉浸式应用设计。它通过 Dart FFI 直接调用底层 C/C++ 方法，提供了低延迟、高质量的音频播放体验。

与其他 Flutter 音频库相比，`flutter_soloud` 的核心优势在于：

| 特性       | flutter_soloud  | 其他库   |
| ---------- | --------------- | -------- |
| 延迟       | ⭐⭐⭐⭐⭐ 极低 | 一般     |
| 3D 音频    | ✅ 完整支持     | ❌       |
| 效果系统   | ✅ 丰富         | 有限     |
| 实时可视化 | ✅ FFT/波形     | 有限     |
| 流式播放   | ✅ 支持         | 部分支持 |
| 复杂度     | 中等            | 简单     |

Flutter 框架小知识

**SoLoud 音频引擎简介**

SoLoud 是一个易于使用、免费、便携的 C/C++ 音频引擎，设计哲学是：

> "让简单的事情变得简单，同时不让复杂的事情变得不可能。"

SoLoud 的主要特点：

- 支持多种音频格式（MP3、WAV、OGG、FLAC）
- 低延迟音频播放
- 3D 定位音频
- 丰富的音频效果
- 多平台支持

`flutter_soloud` 通过 Dart FFI 直接调用 SoLoud 引擎，避免了传统 MethodChannel 的性能开销。

### 1.2 flutter_soloud 3.4.10 的新特性

版本 3.4.10 是 `flutter_soloud` 的稳定版本，主要包含以下特性：

#### 1.2.1 核心特性

- **低延迟音频播放**：适合游戏音效和实时应用
- **3D 定位音频**：支持多普勒效应的空间音频
- **无缝循环播放**：支持音频的无缝循环
- **流式播放**：支持 PCM、MP3、Ogg（Opus、Vorbis、FLAC）
- **实时可视化**：获取音频波形和 FFT 数据
- **丰富效果系统**：混响、回声、限幅器、低音增强等
- **渐变器（Faders）**：属性渐变控制
- **振荡器（Oscillators）**：属性振荡控制
- **波形生成**：实时生成正弦波、方波、锯齿波、三角波
- **多声音支持**：同时播放多个声音实例

#### 1.2.2 平台支持

| 平台    | 支持状态 | 最低版本     |
| ------- | -------- | ------------ |
| Android | ✅       | API 21+      |
| iOS     | ✅       | iOS 13.0+    |
| macOS   | ✅       | macOS 10.15+ |
| Windows | ✅       | Windows 10+  |
| Linux   | ✅       | 任意版本     |
| Web     | ✅       | 现代浏览器   |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_soloud: ^3.4.10
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 Web 平台配置

Web 平台需要特殊配置，请参考官方文档设置 Web 支持。

Dart Tips 语法小贴士

**FFI（Foreign Function Interface）**

`flutter_soloud` 使用 Dart FFI 直接调用 C/C++ 代码，相比传统的 Platform Channel：

| 特性     | FFI        | Platform Channel |
| -------- | ---------- | ---------------- |
| 性能     | 极高       | 一般             |
| 延迟     | 极低       | 较高             |
| 复杂度   | 中等       | 简单             |
| 适用场景 | 高性能需求 | 一般功能调用     |

FFI 适合需要高性能的场景，如游戏音频、实时处理等。

## 第2章 SoLoud 类详解

`SoLoud` 是 `flutter_soloud` 包的核心类，采用单例模式管理音频引擎。

### 2.1 获取实例

```dart
final soloud = SoLoud.instance;
```

### 2.2 初始化方法

```dart
Future<void> init({
  PlaybackDevice? device,           // 指定播放设备
  bool automaticCleanup = false,    // 自动清理
  int sampleRate = 44100,           // 采样率
  int bufferSize = 2048,            // 缓冲区大小
  Channels channels = Channels.stereo,  // 声道配置
})
```

**使用示例**：

```dart
// 基本初始化
await soloud.init();

// 自定义初始化
await soloud.init(
  sampleRate: 48000,
  bufferSize: 4096,
  channels: Channels.stereo,
);
```

### 2.3 核心属性详解

| 属性            | 类型                    | 说明             |
| --------------- | ----------------------- | ---------------- |
| `isInitialized` | `bool`                  | 引擎是否已初始化 |
| `activeSounds`  | `Iterable<AudioSource>` | 当前加载的音频   |
| `filters`       | `FiltersGlobal`         | 全局滤镜         |

### 2.4 音频加载方法

#### 2.4.1 从资源加载

```dart
Future<AudioSource> loadAsset(
  String key, {
  LoadMode mode = LoadMode.memory,  // 加载模式
  AssetBundle? assetBundle,         // 资源包
})
```

**LoadMode 枚举**：

| 值                | 说明       |
| ----------------- | ---------- |
| `LoadMode.memory` | 加载到内存 |
| `LoadMode.disk`   | 从磁盘加载 |

**使用示例**：

```dart
final sound = await soloud.loadAsset('assets/sounds/explosion.mp3');
```

#### 2.4.2 从文件加载

```dart
Future<AudioSource> loadFile(
  String path, {
  LoadMode mode = LoadMode.memory,
})
```

**使用示例**：

```dart
final sound = await soloud.loadFile('/path/to/sound.mp3');
```

#### 2.4.3 从内存加载

```dart
Future<AudioSource> loadMem(
  String path,
  Uint8List buffer, {
  LoadMode mode = LoadMode.memory,
})
```

**使用示例**：

```dart
final bytes = await File('sound.mp3').readAsBytes();
final sound = await soloud.loadMem('sound.mp3', bytes);
```

#### 2.4.4 从网络加载

```dart
Future<AudioSource> loadUrl(
  String url, {
  LoadMode mode = LoadMode.memory,
  Client? httpClient,
})
```

**使用示例**：

```dart
final sound = await soloud.loadUrl('https://example.com/sound.mp3');
```

#### 2.4.5 生成波形

```dart
Future<AudioSource> loadWaveform(
  WaveForm waveform,    // 波形类型
  bool superWave,       // 超级波形
  double scale,         // 缩放
  double detune,        // 失谐
)
```

**WaveForm 枚举**：

| 值                  | 说明       |
| ------------------- | ---------- |
| `WaveForm.square`   | 方波       |
| `WaveForm.saw`      | 锯齿波     |
| `WaveForm.sin`      | 正弦波     |
| `WaveForm.triangle` | 三角波     |
| `WaveForm.bounce`   | 弹跳波     |
| `WaveForm.jaws`     | 锯齿波变体 |
| `WaveForm.humps`    | 驼峰波     |
| `WaveForm.fsquare`  | 滤波方波   |
| `WaveForm.fsaw`     | 滤波锯齿波 |

**使用示例**：

```dart
final wave = await soloud.loadWaveform(
  WaveForm.sin,
  false,
  1.0,
  0.0,
);
```

### 2.5 播放控制方法

#### 2.5.1 播放音频

```dart
Future<SoundHandle> play(
  AudioSource sound, {
  double volume = 1.0,           // 音量
  double pan = 0.0,              // 声像（-1.0 左，1.0 右）
  bool paused = false,           // 是否暂停
  bool looping = false,          // 是否循环
  double loopingStartAt = 0.0,   // 循环起始点
})
```

**使用示例**：

```dart
// 播放一次
final handle = await soloud.play(sound);

// 循环播放
final handle = await soloud.play(
  sound,
  volume: 0.8,
  looping: true,
);
```

#### 2.5.2 停止播放

```dart
Future<void> stop(SoundHandle handle)
Future<void> stopAll()
```

**使用示例**：

```dart
// 停止单个声音
await soloud.stop(handle);

// 停止所有声音
await soloud.stopAll();
```

#### 2.5.3 暂停/恢复

```dart
Future<void> pause(SoundHandle handle)
Future<void> resume(SoundHandle handle)
```

#### 2.5.4 设置属性

```dart
Future<void> setVolume(SoundHandle handle, double volume)
Future<void> setPan(SoundHandle handle, double pan)
Future<void> setRelativePlaySpeed(SoundHandle handle, double speed)
```

### 2.6 3D 音频方法

#### 2.6.1 设置 3D 位置

```dart
Future<void> set3dSourcePosition(
  SoundHandle handle,
  double x,
  double y,
  double z,
)

Future<void> set3dSourceParameters(
  SoundHandle handle,
  double x,
  double y,
  double z,
  double vx,
  double vy,
  double vz,
)
```

**使用示例**：

```dart
// 设置声音在 3D 空间中的位置
await soloud.set3dSourcePosition(handle, 10.0, 0.0, 0.0);

// 设置位置和速度（用于多普勒效应）
await soloud.set3dSourceParameters(
  handle,
  10.0, 0.0, 0.0,  // 位置
  5.0, 0.0, 0.0,   // 速度
);
```

### 2.7 可视化方法

#### 2.7.1 启用可视化

```dart
Future<void> setVisualizationEnabled(bool enabled)
```

#### 2.7.2 获取 FFT 数据

```dart
Future<List<double>> getFFT()
```

#### 2.7.3 获取波形数据

```dart
Future<List<double>> getWave()
```

**使用示例**：

```dart
// 启用可视化
await soloud.setVisualizationEnabled(true);

// 获取 FFT 数据
final fft = await soloud.getFFT();

// 获取波形数据
final wave = await soloud.getWave();
```

### 2.8 资源释放方法

```dart
Future<void> disposeSource(AudioSource sound)
Future<void> deinit()
```

**使用示例**：

```dart
// 释放单个音频源
await soloud.disposeSource(sound);

// 释放整个引擎
await soloud.deinit();
```

## 第3章 AudioSource 类详解

`AudioSource` 表示加载的音频源，可以多次播放。

### 3.1 核心属性

| 属性          | 类型       | 说明         |
| ------------- | ---------- | ------------ |
| `soundHash`   | `int`      | 音频哈希标识 |
| `soundLength` | `Duration` | 音频长度     |

### 3.2 获取音频长度

```dart
final length = sound.soundLength;
print('音频长度: $length');
```

## 第4章 SoundHandle 类详解

`SoundHandle` 表示正在播放的声音实例，用于控制单个播放。

### 4.1 核心属性

| 属性       | 类型   | 说明         |
| ---------- | ------ | ------------ |
| `isValid`  | `bool` | 句柄是否有效 |
| `isPaused` | `bool` | 是否暂停     |

### 4.2 使用示例

```dart
// 播放并获取句柄
final handle = await soloud.play(sound);

// 检查句柄有效性
if (handle.isValid) {
  // 控制播放
  await soloud.setVolume(handle, 0.5);
}
```

## 第5章 滤镜系统详解

`flutter_soloud` 提供了丰富的音频滤镜效果。

### 5.1 滤镜类型

| 滤镜                        | 说明             |
| --------------------------- | ---------------- |
| `FilterType.biquadResonant` | 双二次谐振滤波器 |
| `FilterType.echo`           | 回声             |
| `FilterType.limiter`        | 限幅器           |
| `FilterType.reverb`         | 混响             |
| `FilterType.waveShaper`     | 波形整形器       |
| `FilterType.freeverbFilter` | 自由混响         |
| `FilterType.rolofi`         | 低保真           |
| `FilterType.eq`             | 均衡器           |

### 5.2 全局滤镜

```dart
// 激活滤镜
await soloud.filters.activate(FilterType.reverb);

// 设置滤镜参数
await soloud.filters.setParams(
  FilterType.reverb,
  0,  // 参数索引
  0.5,  // 参数值
);

// 停用滤镜
await soloud.filters.deactivate(FilterType.reverb);
```

## 第6章 flutter_soloud 实战应用

### 6.1 游戏音效管理器

```dart
class GameAudioManager {
  static final GameAudioManager _instance = GameAudioManager._internal();
  factory GameAudioManager() => _instance;
  GameAudioManager._internal();

  final soloud = SoLoud.instance;
  final Map<String, AudioSource> _sounds = {};

  Future<void> init() async {
    await soloud.init();
  }

  Future<void> loadSound(String name, String path) async {
    _sounds[name] = await soloud.loadAsset(path);
  }

  Future<SoundHandle> play(String name, {double volume = 1.0}) async {
    final sound = _sounds[name];
    if (sound != null) {
      return await soloud.play(sound, volume: volume);
    }
    throw Exception('Sound $name not found');
  }

  Future<void> playLoop(String name, {double volume = 1.0}) async {
    final sound = _sounds[name];
    if (sound != null) {
      await soloud.play(sound, volume: volume, looping: true);
    }
  }

  Future<void> stopAll() async {
    await soloud.stopAll();
  }

  Future<void> dispose() async {
    for (final sound in _sounds.values) {
      await soloud.disposeSource(sound);
    }
    _sounds.clear();
    await soloud.deinit();
  }
}

// 使用
final audio = GameAudioManager();

// 初始化
await audio.init();

// 加载音效
await audio.loadSound('explosion', 'assets/sounds/explosion.mp3');
await audio.loadSound('shoot', 'assets/sounds/shoot.mp3');

// 播放音效
await audio.play('explosion', volume: 0.8);
await audio.play('shoot');
```

### 6.2 3D 音频示例

```dart
class Audio3DExample {
  final soloud = SoLoud.instance;
  AudioSource? engineSound;
  SoundHandle? engineHandle;

  Future<void> init() async {
    await soloud.init();
    engineSound = await soloud.loadAsset('assets/sounds/engine.mp3');
  }

  Future<void> startEngine() async {
    if (engineSound != null) {
      engineHandle = await soloud.play(
        engineSound!,
        volume: 0.5,
        looping: true,
      );
    }
  }

  Future<void> updateCarPosition(double x, double y, double z) async {
    if (engineHandle != null && engineHandle!.isValid) {
      await soloud.set3dSourcePosition(
        engineHandle!,
        x, y, z,
      );
    }
  }

  Future<void> stopEngine() async {
    if (engineHandle != null) {
      await soloud.stop(engineHandle!);
    }
  }
}
```

### 6.3 音频可视化示例

```dart
class AudioVisualizer extends StatefulWidget {
  @override
  _AudioVisualizerState createState() => _AudioVisualizerState();
}

class _AudioVisualizerState extends State<AudioVisualizer> {
  final soloud = SoLoud.instance;
  List<double> fftData = [];
  Timer? timer;

  @override
  void initState() {
    super.initState();
    initAudio();
  }

  Future<void> initAudio() async {
    await soloud.init();
    await soloud.setVisualizationEnabled(true);

    final sound = await soloud.loadAsset('assets/music.mp3');
    await soloud.play(sound);

    // 定时获取 FFT 数据
    timer = Timer.periodic(Duration(milliseconds: 16), (_) async {
      final fft = await soloud.getFFT();
      setState(() {
        fftData = fft;
      });
    });
  }

  @override
  void dispose() {
    timer?.cancel();
    soloud.deinit();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(double.infinity, 200),
      painter: FFTVisualizer(fftData),
    );
  }
}

class FFTVisualizer extends CustomPainter {
  final List<double> fftData;

  FFTVisualizer(this.fftData);

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2;

    final barWidth = size.width / fftData.length;

    for (int i = 0; i < fftData.length; i++) {
      final barHeight = fftData[i] * size.height;
      canvas.drawRect(
        Rect.fromLTWH(
          i * barWidth,
          size.height - barHeight,
          barWidth - 1,
          barHeight,
        ),
        paint,
      );
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

### 6.4 波形生成器

```dart
class WaveformGenerator {
  final soloud = SoLoud.instance;

  Future<void> init() async {
    await soloud.init();
  }

  Future<SoundHandle> playTone(
    WaveForm waveform,
    double frequency,
    double duration,
  ) async {
    final wave = await soloud.loadWaveform(
      waveform,
      false,
      frequency,
      0.0,
    );

    return await soloud.play(wave);
  }

  Future<void> playBeep() async {
    await playTone(WaveForm.sin, 440.0, 0.2);  // A4 音符
  }

  Future<void> playAlarm() async {
    await playTone(WaveForm.square, 880.0, 0.5);  // A5 音符
  }
}
```

## 附录 A：SoLoud 方法速查表

| 方法                    | 返回值                 | 说明         |
| ----------------------- | ---------------------- | ------------ |
| `init()`                | `Future<void>`         | 初始化引擎   |
| `deinit()`              | `Future<void>`         | 释放引擎     |
| `loadAsset()`           | `Future<AudioSource>`  | 从资源加载   |
| `loadFile()`            | `Future<AudioSource>`  | 从文件加载   |
| `loadMem()`             | `Future<AudioSource>`  | 从内存加载   |
| `loadUrl()`             | `Future<AudioSource>`  | 从网络加载   |
| `loadWaveform()`        | `Future<AudioSource>`  | 生成波形     |
| `play()`                | `Future<SoundHandle>`  | 播放音频     |
| `stop()`                | `Future<void>`         | 停止播放     |
| `stopAll()`             | `Future<void>`         | 停止所有     |
| `pause()`               | `Future<void>`         | 暂停         |
| `resume()`              | `Future<void>`         | 恢复         |
| `setVolume()`           | `Future<void>`         | 设置音量     |
| `setPan()`              | `Future<void>`         | 设置声像     |
| `set3dSourcePosition()` | `Future<void>`         | 设置 3D 位置 |
| `getFFT()`              | `Future<List<double>>` | 获取 FFT     |
| `getWave()`             | `Future<List<double>>` | 获取波形     |

## 附录 B：常见问题与解决方案

### Q1：初始化失败

**解决方案**：

```dart
// 确保只初始化一次
if (!soloud.isInitialized) {
  await soloud.init();
}
```

### Q2：音频无法加载

**检查项**：

1. 资源路径是否正确
2. 音频格式是否支持
3. 文件是否存在

### Q3：Web 平台问题

Web 平台需要特殊配置，请参考官方文档。

## 附录 C：参考资源

- [flutter_soloud 官方文档](https://pub.dev/packages/flutter_soloud)
- [flutter_soloud API 参考](https://pub.dev/documentation/flutter_soloud/latest/)
- [SoLoud 官网](https://solhsa.com/soloud/)
- [音频可视化示例](https://github.com/alnitak/audio_flux)
