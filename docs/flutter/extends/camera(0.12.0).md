# camera 0.12.0 详解与实战

## 第1章 认识 camera

### 1.1 什么是 camera

`camera` 是 Flutter 官方团队维护的相机插件，提供对设备相机硬件的访问能力。它是 Flutter 生态中最基础、最稳定的相机解决方案，被广泛应用于各类需要拍照、录像功能的 Flutter 应用中。

与其他 Flutter 相机库相比，`camera` 的核心特点在于：

| 特性       | camera             | 其他库   |
| ---------- | ------------------ | -------- |
| 官方维护   | ✅ Flutter 团队    | 社区维护 |
| 稳定性     | ⭐⭐⭐⭐⭐ 极高    | 较高     |
| 功能完整度 | 拍照、录像、图像流 | 各异     |
| 平台支持   | Android/iOS/Web    | 各异     |
| 学习曲线   | 较陡峭             | 较平缓   |
| 自定义程度 | 极高               | 中等     |

Flutter 框架小知识

**底层相机实现**

`camera` 插件在不同平台上使用不同的原生相机 API：

| 平台    | 底层实现          | 说明                  |
| ------- | ----------------- | --------------------- |
| Android | CameraX / Camera2 | Google 推荐的相机框架 |
| iOS     | AVFoundation      | Apple 原生相机框架    |
| Web     | getUserMedia      | Web 标准媒体 API      |

这种设计确保了在各平台上都能获得最佳的相机性能和兼容性。

### 1.2 camera 0.12.0 的新特性

版本 0.12.0 是 `camera` 插件的重要版本，主要包含以下特性：

#### 1.2.1 核心特性

- **相机预览**：实时显示相机画面
- **拍照功能**：捕获高质量静态图片
- **视频录制**：录制视频并保存到文件
- **图像流**：实时获取相机帧数据（用于图像处理）
- **闪光灯控制**：支持开关和自动模式
- **变焦控制**：支持数字变焦
- **曝光控制**：调整曝光补偿
- **对焦控制**：自动对焦和手动对焦
- **前后摄像头切换**：支持多摄像头设备

#### 1.2.2 平台支持

| 平台    | 支持状态 | 最低版本要求           |
| ------- | -------- | ---------------------- |
| Android | ✅       | SDK 24+ (Android 7.0+) |
| iOS     | ✅       | iOS 13.0+              |
| Web     | ✅       | 现代浏览器（需 HTTPS） |

### 1.3 安装与配置

#### 1.3.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  camera: ^0.12.0
  path_provider: ^2.1.0 # 用于获取存储路径
  path: ^1.9.0 # 用于处理文件路径
```

然后运行：

```bash
flutter pub get
```

#### 1.3.2 平台配置

**Android 配置**：

在 `android/app/build.gradle` 中设置最低 SDK 版本：

```gradle
android {
    defaultConfig {
        minSdkVersion 24  // 必须 >= 24
    }
}
```

**iOS 配置**：

在 `ios/Runner/Info.plist` 中添加相机和麦克风权限说明：

```xml
<dict>
    <!-- 其他配置... -->

    <!-- 相机权限 -->
    <key>NSCameraUsageDescription</key>
    <string>此应用需要访问相机来拍摄照片和视频</string>

    <!-- 麦克风权限 -->
    <key>NSMicrophoneUsageDescription</key>
    <string>此应用需要访问麦克风来录制视频声音</string>
</dict>
```

Dart Tips 语法小贴士

**异步初始化模式**

相机插件需要在应用启动时进行异步初始化，推荐使用以下模式：

```dart
Future<void> main() async {
  // 确保 Flutter 绑定初始化
  WidgetsFlutterBinding.ensureInitialized();

  // 异步获取相机列表
  final cameras = await availableCameras();

  runApp(MyApp(cameras: cameras));
}
```

`WidgetsFlutterBinding.ensureInitialized()` 是调用任何插件方法前必须执行的步骤。

---

## 第2章 CameraDescription 类详解

### 2.1 类概述

`CameraDescription` 是一个不可变的数据类，用于描述设备上可用的相机硬件信息。它是相机插件中最基础的类之一。

```dart
@immutable
class CameraDescription {
  final String name;
  final CameraLensDirection lensDirection;
  final int sensorOrientation;
}
```

### 2.2 属性详解

#### 2.2.1 name

| 属性   | 说明                         |
| ------ | ---------------------------- |
| 类型   | `String`                     |
| 用途   | 相机的唯一标识符             |
| 示例值 | `"0"`（后置）、`"1"`（前置） |

`name` 是平台特定的相机标识符，用于在创建 `CameraController` 时指定要使用的相机。

```dart
// 获取所有可用相机
final cameras = await availableCameras();

// 使用第一个相机的 name
final firstCamera = cameras.first;
print(firstCamera.name); // 输出: "0"
```

#### 2.2.2 lensDirection

| 属性   | 说明                        |
| ------ | --------------------------- |
| 类型   | `CameraLensDirection`       |
| 用途   | 指示相机的朝向              |
| 枚举值 | `front`、`back`、`external` |

`CameraLensDirection` 枚举定义：

```dart
enum CameraLensDirection {
  front,    // 前置摄像头（自拍）
  back,     // 后置摄像头（主摄像头）
  external, // 外接摄像头（如 USB 摄像头）
}
```

使用示例：

```dart
// 查找后置摄像头
final backCamera = cameras.firstWhere(
  (camera) => camera.lensDirection == CameraLensDirection.back,
  orElse: () => cameras.first,
);

// 查找前置摄像头
final frontCamera = cameras.firstWhere(
  (camera) => camera.lensDirection == CameraLensDirection.front,
  orElse: () => cameras.first,
);
```

#### 2.2.3 sensorOrientation

| 属性   | 说明                         |
| ------ | ---------------------------- |
| 类型   | `int`                        |
| 用途   | 相机传感器的自然方向（度数） |
| 典型值 | `0`、`90`、`180`、`270`      |

`sensorOrientation` 表示相机传感器相对于设备自然方向的角度。这个值在需要处理照片方向时非常重要。

```dart
// 检查传感器方向
final camera = cameras.first;
print(camera.sensorOrientation); // 输出: 90（大多数后置摄像头）
```

### 2.3 availableCameras 函数

`availableCameras()` 是一个顶层异步函数，用于获取设备上所有可用的相机列表。

```dart
Future<List<CameraDescription>> availableCameras()
```

返回值：

- 成功：返回 `List<CameraDescription>`，包含所有可用相机
- 失败：抛出 `CameraException`

使用示例：

```dart
Future<void> initializeCameras() async {
  try {
    final cameras = await availableCameras();

    if (cameras.isEmpty) {
      print('设备没有可用相机');
      return;
    }

    // 打印所有相机信息
    for (var camera in cameras) {
      print('相机: ${camera.name}');
      print('  方向: ${camera.lensDirection}');
      print('  传感器方向: ${camera.sensorOrientation}°');
    }
  } on CameraException catch (e) {
    print('获取相机列表失败: ${e.description}');
  }
}
```

Flutter 框架小知识

**相机方向处理**

不同设备的相机传感器方向可能不同：

| 设备类型            | 典型 sensorOrientation |
| ------------------- | ---------------------- |
| 大多数 Android 后置 | 90°                    |
| 大多数 Android 前置 | 270°                   |
| iPhone 后置         | 90°                    |
| iPhone 前置         | 90°                    |

`camera` 插件会自动处理方向转换，但在某些高级场景（如自定义图像处理）中，你可能需要手动考虑这个值。

---

## 第3章 CameraController 类详解

### 3.1 类概述

`CameraController` 是 camera 插件中最核心的类，负责与设备相机硬件建立连接并提供控制接口。

```dart
class CameraController extends ValueNotifier<CameraValue> {
  // 构造函数
  CameraController(
    CameraDescription description,
    ResolutionPreset resolutionPreset, {
    bool enableAudio = true,
    ImageFormatGroup? imageFormatGroup,
  });
}
```

### 3.2 构造函数参数

#### 3.2.1 description（必需）

| 参数 | 说明                            |
| ---- | ------------------------------- |
| 类型 | `CameraDescription`             |
| 用途 | 指定要控制的相机                |
| 来源 | `availableCameras()` 返回的列表 |

```dart
final cameras = await availableCameras();
final controller = CameraController(
  cameras.first,  // 使用第一个相机
  ResolutionPreset.medium,
);
```

#### 3.2.2 resolutionPreset（必需）

| 参数   | 说明                       |
| ------ | -------------------------- |
| 类型   | `ResolutionPreset`         |
| 用途   | 设置相机预览和捕获的分辨率 |
| 默认值 | 无（必须显式指定）         |

`ResolutionPreset` 枚举定义：

```dart
enum ResolutionPreset {
  low,      // 低分辨率（约 320x240）
  medium,   // 中等分辨率（约 720x480）
  high,     // 高分辨率（约 1280x720）
  veryHigh, // 很高分辨率（约 1920x1080）
  ultraHigh,// 超高分辨率（约 3840x2160）
  max,      // 设备支持的最大分辨率
}
```

使用建议：

| 场景       | 推荐 preset       | 说明                   |
| ---------- | ----------------- | ---------------------- |
| 简单预览   | `low`             | 节省电量和内存         |
| 普通拍照   | `medium` / `high` | 平衡质量和性能         |
| 高质量拍照 | `veryHigh`        | 需要较好的设备         |
| 专业摄影   | `max`             | 最高质量，消耗资源最多 |

```dart
// 高分辨率拍照
final controller = CameraController(
  cameras.first,
  ResolutionPreset.veryHigh,
);
```

#### 3.2.3 enableAudio（可选）

| 参数   | 说明             |
| ------ | ---------------- |
| 类型   | `bool`           |
| 默认值 | `true`           |
| 用途   | 是否启用音频录制 |

```dart
// 仅拍照，不需要音频
final controller = CameraController(
  cameras.first,
  ResolutionPreset.high,
  enableAudio: false,  // 禁用音频
);
```

#### 3.2.4 imageFormatGroup（可选）

| 参数   | 说明                 |
| ------ | -------------------- |
| 类型   | `ImageFormatGroup?`  |
| 默认值 | `null`（平台默认）   |
| 用途   | 设置图像流的数据格式 |

`ImageFormatGroup` 枚举定义：

```dart
enum ImageFormatGroup {
  bgra8888,  // iOS 默认格式
  yuv420,    // Android 默认格式
  jpeg,      // JPEG 压缩格式
  unknown,   // 未知格式
}
```

### 3.3 常用属性

#### 3.3.1 value

| 属性 | 说明               |
| ---- | ------------------ |
| 类型 | `CameraValue`      |
| 用途 | 获取相机的当前状态 |

`CameraValue` 包含以下重要属性：

```dart
class CameraValue {
  final bool isInitialized;           // 是否已初始化
  final bool isRecordingVideo;        // 是否正在录制视频
  final bool isTakingPicture;         // 是否正在拍照
  final bool isStreamingImages;       // 是否正在传输图像流
  final FlashMode flashMode;          // 当前闪光灯模式
  final ExposureMode exposureMode;    // 当前曝光模式
  final FocusMode focusMode;          // 当前对焦模式
  final double exposureOffset;        // 曝光补偿值
  final double zoomLevel;             // 当前变焦级别
  final double maxZoomLevel;          // 最大变焦级别
  final double minZoomLevel;          // 最小变焦级别
  final double aspectRatio;           // 预览画面宽高比
}
```

使用示例：

```dart
// 检查相机是否已初始化
if (controller.value.isInitialized) {
  print('相机已就绪');
}

// 检查是否正在录制
if (controller.value.isRecordingVideo) {
  print('正在录制视频...');
}

// 获取当前变焦级别
print('当前变焦: ${controller.value.zoomLevel}x');
print('支持变焦范围: ${controller.value.minZoomLevel}x - ${controller.value.maxZoomLevel}x');
```

#### 3.3.2 description

| 属性 | 说明                     |
| ---- | ------------------------ |
| 类型 | `CameraDescription`      |
| 用途 | 获取控制器关联的相机描述 |

```dart
// 获取当前使用的相机信息
print('当前相机: ${controller.description.name}');
print('相机方向: ${controller.description.lensDirection}');
```

### 3.4 核心方法

#### 3.4.1 initialize()

初始化相机控制器，建立与相机硬件的连接。

```dart
Future<void> initialize()
```

返回值：

- 成功：完成初始化
- 失败：抛出 `CameraException`

使用示例：

```dart
class CameraScreenState extends State<CameraScreen> {
  late CameraController _controller;
  late Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    _controller = CameraController(
      widget.camera,
      ResolutionPreset.medium,
    );
    // 初始化并保存 Future
    _initializeControllerFuture = _controller.initialize();
  }
}
```

#### 3.4.2 dispose()

释放相机资源，必须在 Widget 销毁时调用。

```dart
Future<void> dispose()
```

使用示例：

```dart
@override
void dispose() {
  // 必须先释放控制器
  _controller.dispose();
  super.dispose();
}
```

#### 3.4.3 takePicture()

拍摄照片并返回文件。

```dart
Future<XFile> takePicture()
```

返回值：

- 成功：返回 `XFile`，包含照片路径
- 失败：抛出 `CameraException`

使用示例：

```dart
try {
  // 确保相机已初始化
  await _initializeControllerFuture;

  // 拍照
  final XFile image = await _controller.takePicture();

  print('照片保存路径: ${image.path}');
  print('照片名称: ${image.name}');

  // 读取图片数据
  final Uint8List bytes = await image.readAsBytes();

} catch (e) {
  print('拍照失败: $e');
}
```

#### 3.4.4 startVideoRecording()

开始录制视频。

```dart
Future<void> startVideoRecording()
```

使用示例：

```dart
try {
  if (!_controller.value.isRecordingVideo) {
    await _controller.startVideoRecording();
    print('开始录制视频');
  }
} on CameraException catch (e) {
  print('开始录制失败: ${e.description}');
}
```

#### 3.4.5 stopVideoRecording()

停止视频录制并返回视频文件。

```dart
Future<XFile> stopVideoRecording()
```

返回值：

- 成功：返回 `XFile`，包含视频路径
- 失败：抛出 `CameraException`

使用示例：

```dart
try {
  if (_controller.value.isRecordingVideo) {
    final XFile video = await _controller.stopVideoRecording();
    print('视频保存路径: ${video.path}');
    print('视频大小: ${await video.length()} bytes');
  }
} on CameraException catch (e) {
  print('停止录制失败: ${e.description}');
}
```

#### 3.4.6 pauseVideoRecording()

暂停视频录制。

```dart
Future<void> pauseVideoRecording()
```

使用示例：

```dart
if (_controller.value.isRecordingVideo) {
  await _controller.pauseVideoRecording();
  print('视频录制已暂停');
}
```

#### 3.4.7 resumeVideoRecording()

恢复视频录制。

```dart
Future<void> resumeVideoRecording()
```

使用示例：

```dart
await _controller.resumeVideoRecording();
print('视频录制已恢复');
```

#### 3.4.8 startImageStream()

开始接收相机图像流，用于实时图像处理。

```dart
Future<void> startImageStream(
  Function(CameraImage image) onAvailable,
)
```

参数：

- `onAvailable`：每帧图像可用时的回调函数

使用示例：

```dart
await _controller.startImageStream((CameraImage image) {
  // 处理每一帧图像
  print('收到图像帧: ${image.width}x${image.height}');
  print('格式: ${image.format.group}');

  // 可以在这里进行图像识别、滤镜处理等
});
```

#### 3.4.9 stopImageStream()

停止图像流。

```dart
Future<void> stopImageStream()
```

使用示例：

```dart
if (_controller.value.isStreamingImages) {
  await _controller.stopImageStream();
  print('图像流已停止');
}
```

### 3.5 闪光灯控制方法

#### 3.5.1 setFlashMode()

设置闪光灯模式。

```dart
Future<void> setFlashMode(FlashMode mode)
```

`FlashMode` 枚举定义：

```dart
enum FlashMode {
  off,    // 关闭
  auto,   // 自动（根据光线条件）
  always, // 始终开启
  torch,  // 手电筒模式（常亮）
}
```

使用示例：

```dart
// 关闭闪光灯
await _controller.setFlashMode(FlashMode.off);

// 自动闪光灯
await _controller.setFlashMode(FlashMode.auto);

// 强制开启闪光灯
await _controller.setFlashMode(FlashMode.always);

// 手电筒模式（持续照明）
await _controller.setFlashMode(FlashMode.torch);
```

### 3.6 变焦控制方法

#### 3.6.1 setZoomLevel()

设置变焦级别。

```dart
Future<void> setZoomLevel(double zoom)
```

参数：

- `zoom`：变焦倍数，必须在 `minZoomLevel` 和 `maxZoomLevel` 之间

使用示例：

```dart
// 获取变焦范围
final minZoom = _controller.value.minZoomLevel;
final maxZoom = _controller.value.maxZoomLevel;

// 设置 2 倍变焦
await _controller.setZoomLevel(2.0);

// 设置最大变焦
await _controller.setZoomLevel(maxZoom);
```

### 3.7 曝光控制方法

#### 3.7.1 setExposureMode()

设置曝光模式。

```dart
Future<void> setExposureMode(ExposureMode mode)
```

`ExposureMode` 枚举定义：

```dart
enum ExposureMode {
  auto,   // 自动曝光
  locked, // 锁定当前曝光
}
```

#### 3.7.2 setExposureOffset()

设置曝光补偿。

```dart
Future<void> setExposureOffset(double offset)
```

参数：

- `offset`：曝光补偿值，通常为 -1.0 到 1.0 之间

使用示例：

```dart
// 增加曝光（画面变亮）
await _controller.setExposureOffset(0.5);

// 减少曝光（画面变暗）
await _controller.setExposureOffset(-0.5);
```

### 3.8 对焦控制方法

#### 3.8.1 setFocusMode()

设置对焦模式。

```dart
Future<void> setFocusMode(FocusMode mode)
```

`FocusMode` 枚举定义：

```dart
enum FocusMode {
  auto,   // 自动对焦
  locked, // 锁定当前对焦
}
```

#### 3.8.2 setFocusPoint()

设置对焦点位置。

```dart
Future<void> setFocusPoint(Offset? point)
```

参数：

- `point`：对焦点的归一化坐标（0.0-1.0），null 表示重置

使用示例：

```dart
// 在画面中心对焦
await _controller.setFocusPoint(const Offset(0.5, 0.5));

// 在画面左上角对焦
await _controller.setFocusPoint(const Offset(0.25, 0.25));

// 重置对焦点
await _controller.setFocusPoint(null);
```

Dart Tips 语法小贴士

**资源管理最佳实践**

相机是系统级资源，必须妥善管理：

```dart
class CameraScreen extends StatefulWidget {
  @override
  CameraScreenState createState() => CameraScreenState();
}

class CameraScreenState extends State<CameraScreen>
    with WidgetsBindingObserver {  // 监听应用生命周期

  CameraController? _controller;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    _initializeCamera();
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    _controller?.dispose();  // 使用 ?. 安全调用
    super.dispose();
  }

  // 监听应用生命周期变化
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.inactive) {
      // 应用进入后台，释放相机
      _controller?.dispose();
    } else if (state == AppLifecycleState.resumed) {
      // 应用回到前台，重新初始化
      _initializeCamera();
    }
  }
}
```

---

## 第4章 CameraPreview Widget 详解

### 4.1 Widget 概述

`CameraPreview` 是一个用于显示相机实时预览画面的 Widget。它自动处理画面比例和方向，是构建相机界面的核心组件。

```dart
class CameraPreview extends StatelessWidget {
  const CameraPreview(
    this.controller, {
    super.key,
    this.child,
  });

  final CameraController controller;
  final Widget? child;
}
```

### 4.2 构造函数参数

#### 4.2.1 controller（必需）

| 参数 | 说明               |
| ---- | ------------------ |
| 类型 | `CameraController` |
| 用途 | 提供相机预览数据   |
| 要求 | 必须已初始化完成   |

#### 4.2.2 child（可选）

| 参数     | 说明                      |
| -------- | ------------------------- |
| 类型     | `Widget?`                 |
| 用途     | 在预览画面上叠加的 Widget |
| 常见用途 | 对焦框、网格线、UI 控件   |

### 4.3 基本使用

```dart
FutureBuilder<void>(
  future: _initializeControllerFuture,
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.done) {
      // 初始化完成，显示预览
      return CameraPreview(_controller);
    } else {
      // 初始化中，显示加载指示器
      return const Center(child: CircularProgressIndicator());
    }
  },
)
```

### 4.4 高级用法：叠加 UI 元素

```dart
CameraPreview(
  _controller,
  child: Stack(
    children: [
      // 网格线（摄影三分法则）
      CustomPaint(
        size: Size.infinite,
        painter: GridPainter(),
      ),

      // 对焦框
      Center(
        child: Container(
          width: 80,
          height: 80,
          decoration: BoxDecoration(
            border: Border.all(color: Colors.yellow, width: 2),
          ),
        ),
      ),

      // 底部控制栏
      Positioned(
        bottom: 20,
        left: 0,
        right: 0,
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            IconButton(
              icon: const Icon(Icons.flash_off),
              onPressed: _toggleFlash,
            ),
            FloatingActionButton(
              onPressed: _takePicture,
              child: const Icon(Icons.camera),
            ),
            IconButton(
              icon: const Icon(Icons.flip_camera_ios),
              onPressed: _switchCamera,
            ),
          ],
        ),
      ),
    ],
  ),
)
```

### 4.5 处理画面比例

`CameraPreview` 会自动适应相机的宽高比，但有时你可能需要自定义显示方式：

```dart
// 方式1：使用 AspectRatio 保持比例
AspectRatio(
  aspectRatio: _controller.value.aspectRatio,
  child: CameraPreview(_controller),
)

// 方式2：全屏裁剪
SizedBox.expand(
  child: FittedBox(
    fit: BoxFit.cover,
    child: SizedBox(
      width: _controller.value.previewSize?.width ?? 0,
      height: _controller.value.previewSize?.height ?? 0,
      child: CameraPreview(_controller),
    ),
  ),
)

// 方式3：居中显示，保持比例
Center(
  child: AspectRatio(
    aspectRatio: _controller.value.aspectRatio,
    child: CameraPreview(_controller),
  ),
)
```

Flutter 框架小知识

**PlatformView 渲染原理**

`CameraPreview` 在不同平台上使用不同的渲染机制：

| 平台    | 渲染方式                   | 说明                |
| ------- | -------------------------- | ------------------- |
| Android | TextureView / SurfaceView  | 硬件加速渲染        |
| iOS     | AVCaptureVideoPreviewLayer | Core Animation 渲染 |
| Web     | HTMLVideoElement           | 浏览器原生渲染      |

这种设计确保了在各平台上都能获得流畅的预览性能。

---

## 第5章 拍照功能详解

### 5.1 拍照流程

完整的拍照流程包括以下步骤：

```
1. 获取可用相机列表
   ↓
2. 创建并初始化 CameraController
   ↓
3. 使用 CameraPreview 显示预览
   ↓
4. 调用 takePicture() 拍摄照片
   ↓
5. 处理拍摄的照片文件
```

### 5.2 完整拍照示例

```dart
import 'dart:io';
import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:path/path.dart' show join;
import 'package:path_provider/path_provider.dart';

class TakePictureScreen extends StatefulWidget {
  final CameraDescription camera;

  const TakePictureScreen({
    super.key,
    required this.camera,
  });

  @override
  TakePictureScreenState createState() => TakePictureScreenState();
}

class TakePictureScreenState extends State<TakePictureScreen> {
  late CameraController _controller;
  late Future<void> _initializeControllerFuture;

  @override
  void initState() {
    super.initState();
    _controller = CameraController(
      widget.camera,
      ResolutionPreset.veryHigh,
    );
    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _takePicture() async {
    try {
      await _initializeControllerFuture;

      // 获取保存路径
      final directory = await getTemporaryDirectory();
      final path = join(
        directory.path,
        '${DateTime.now().millisecondsSinceEpoch}.jpg',
      );

      // 拍照
      final XFile image = await _controller.takePicture();

      // 将照片复制到指定路径
      await File(image.path).copy(path);

      if (!mounted) return;

      // 显示照片
      await Navigator.push(
        context,
        MaterialPageRoute(
          builder: (context) => DisplayPictureScreen(imagePath: path),
        ),
      );
    } catch (e) {
      print('拍照失败: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('拍照')),
      body: FutureBuilder<void>(
        future: _initializeControllerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            return CameraPreview(_controller);
          } else {
            return const Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _takePicture,
        child: const Icon(Icons.camera_alt),
      ),
    );
  }
}

// 显示照片页面
class DisplayPictureScreen extends StatelessWidget {
  final String imagePath;

  const DisplayPictureScreen({
    super.key,
    required this.imagePath,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('照片预览')),
      body: Image.file(File(imagePath)),
    );
  }
}
```

### 5.3 拍照参数设置

#### 5.3.1 闪光灯设置

```dart
// 拍照前设置闪光灯
await _controller.setFlashMode(FlashMode.auto);

// 拍照
final image = await _controller.takePicture();

// 拍照后恢复闪光灯设置
await _controller.setFlashMode(FlashMode.off);
```

#### 5.3.2 变焦设置

```dart
// 设置 2 倍变焦后拍照
await _controller.setZoomLevel(2.0);
final image = await _controller.takePicture();
```

#### 5.3.3 曝光设置

```dart
// 增加曝光补偿
await _controller.setExposureOffset(0.3);
final image = await _controller.takePicture();
```

### 5.4 处理 XFile

`takePicture()` 返回的 `XFile` 对象提供了多种文件操作方法：

```dart
final XFile image = await _controller.takePicture();

// 获取文件路径
final String path = image.path;

// 获取文件名
final String name = image.name;

// 读取为字节
final Uint8List bytes = await image.readAsBytes();

// 读取为字符串（通常不用于图片）
final String content = await image.readAsString();

// 获取文件大小
final int length = await image.length();

// 打开文件
final File file = File(image.path);
```

Dart Tips 语法小贴士

**XFile 与 File 的区别**

| 特性     | XFile          | File         |
| -------- | -------------- | ------------ |
| 来源     | cross_file 包  | dart:io      |
| 用途     | 跨平台文件抽象 | 原生文件操作 |
| Web 支持 | ✅             | ❌           |
| 方法     | 有限但通用     | 完整文件 API |

在需要更多文件操作时，可以将 XFile 转换为 File：

```dart
final XFile xFile = await _controller.takePicture();
final File file = File(xFile.path);

// 现在可以使用 File 的所有方法
final stat = await file.stat();
print('文件大小: ${stat.size} bytes');
print('修改时间: ${stat.modified}');
```

---

## 第6章 视频录制功能详解

### 6.1 视频录制流程

```
1. 创建 CameraController（enableAudio: true）
   ↓
2. 初始化控制器
   ↓
3. 调用 startVideoRecording() 开始录制
   ↓
4. （可选）pauseVideoRecording() 暂停
   ↓
5. （可选）resumeVideoRecording() 恢复
   ↓
6. 调用 stopVideoRecording() 停止录制
   ↓
7. 处理返回的视频文件
```

### 6.2 完整视频录制示例

```dart
class VideoRecorderScreen extends StatefulWidget {
  final CameraDescription camera;

  const VideoRecorderScreen({
    super.key,
    required this.camera,
  });

  @override
  VideoRecorderScreenState createState() => VideoRecorderScreenState();
}

class VideoRecorderScreenState extends State<VideoRecorderScreen> {
  late CameraController _controller;
  late Future<void> _initializeControllerFuture;
  bool _isRecording = false;
  bool _isPaused = false;
  Timer? _timer;
  int _recordingSeconds = 0;

  @override
  void initState() {
    super.initState();
    _controller = CameraController(
      widget.camera,
      ResolutionPreset.high,
      enableAudio: true,  // 启用音频
    );
    _initializeControllerFuture = _controller.initialize();
  }

  @override
  void dispose() {
    _timer?.cancel();
    _controller.dispose();
    super.dispose();
  }

  Future<void> _startRecording() async {
    try {
      await _initializeControllerFuture;

      if (!_controller.value.isRecordingVideo) {
        await _controller.startVideoRecording();

        setState(() {
          _isRecording = true;
          _isPaused = false;
          _recordingSeconds = 0;
        });

        // 开始计时
        _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
          setState(() => _recordingSeconds++);
        });
      }
    } catch (e) {
      print('开始录制失败: $e');
    }
  }

  Future<void> _pauseRecording() async {
    if (_controller.value.isRecordingVideo && !_isPaused) {
      await _controller.pauseVideoRecording();
      _timer?.cancel();
      setState(() => _isPaused = true);
    }
  }

  Future<void> _resumeRecording() async {
    if (_controller.value.isRecordingVideo && _isPaused) {
      await _controller.resumeVideoRecording();
      _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
        setState(() => _recordingSeconds++);
      });
      setState(() => _isPaused = false);
    }
  }

  Future<void> _stopRecording() async {
    if (_controller.value.isRecordingVideo) {
      _timer?.cancel();

      final XFile video = await _controller.stopVideoRecording();

      setState(() {
        _isRecording = false;
        _isPaused = false;
        _recordingSeconds = 0;
      });

      if (!mounted) return;

      // 显示视频
      await Navigator.push(
        context,
        MaterialPageRoute(
          builder: (context) => VideoPlayerScreen(videoPath: video.path),
        ),
      );
    }
  }

  String get _formattedTime {
    final minutes = (_recordingSeconds ~/ 60).toString().padLeft(2, '0');
    final seconds = (_recordingSeconds % 60).toString().padLeft(2, '0');
    return '$minutes:$seconds';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('视频录制')),
      body: FutureBuilder<void>(
        future: _initializeControllerFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            return Stack(
              children: [
                CameraPreview(_controller),

                // 录制时间显示
                if (_isRecording)
                  Positioned(
                    top: 60,
                    left: 0,
                    right: 0,
                    child: Center(
                      child: Container(
                        padding: const EdgeInsets.symmetric(
                          horizontal: 16,
                          vertical: 8,
                        ),
                        decoration: BoxDecoration(
                          color: Colors.black54,
                          borderRadius: BorderRadius.circular(20),
                        ),
                        child: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            Container(
                              width: 12,
                              height: 12,
                              decoration: BoxDecoration(
                                color: _isPaused ? Colors.yellow : Colors.red,
                                shape: BoxShape.circle,
                              ),
                            ),
                            const SizedBox(width: 8),
                            Text(
                              _formattedTime,
                              style: const TextStyle(
                                color: Colors.white,
                                fontSize: 18,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                          ],
                        ),
                      ),
                    ),
                  ),
              ],
            );
          } else {
            return const Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          if (_isRecording && !_isPaused)
            FloatingActionButton(
              heroTag: 'pause',
              onPressed: _pauseRecording,
              backgroundColor: Colors.orange,
              child: const Icon(Icons.pause),
            ),

          if (_isRecording && _isPaused)
            FloatingActionButton(
              heroTag: 'resume',
              onPressed: _resumeRecording,
              backgroundColor: Colors.green,
              child: const Icon(Icons.play_arrow),
            ),

          const SizedBox(width: 20),

          FloatingActionButton(
            heroTag: 'record',
            onPressed: _isRecording ? _stopRecording : _startRecording,
            backgroundColor: _isRecording ? Colors.red : Colors.white,
            child: Icon(
              _isRecording ? Icons.stop : Icons.videocam,
              color: _isRecording ? Colors.white : Colors.red,
            ),
          ),
        ],
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
    );
  }
}
```

### 6.3 视频录制状态管理

```dart
class RecordingState {
  final bool isInitialized;
  final bool isRecording;
  final bool isPaused;
  final int duration;

  RecordingState({
    this.isInitialized = false,
    this.isRecording = false,
    this.isPaused = false,
    this.duration = 0,
  });

  RecordingState copyWith({
    bool? isInitialized,
    bool? isRecording,
    bool? isPaused,
    int? duration,
  }) {
    return RecordingState(
      isInitialized: isInitialized ?? this.isInitialized,
      isRecording: isRecording ?? this.isRecording,
      isPaused: isPaused ?? this.isPaused,
      duration: duration ?? this.duration,
    );
  }
}
```

### 6.4 视频文件处理

```dart
Future<void> processVideo(String videoPath) async {
  final file = File(videoPath);

  // 获取文件信息
  final stat = await file.stat();
  print('视频大小: ${_formatFileSize(stat.size)}');
  print('创建时间: ${stat.changed}');

  // 保存到相册（需要 photo_manager 或 gallery_saver 插件）
  // await GallerySaver.saveVideo(videoPath);

  // 上传服务器
  // await uploadVideo(file);

  // 压缩视频（需要 video_compress 插件）
  // final compressed = await VideoCompress.compressVideo(
  //   videoPath,
  //   quality: VideoQuality.MediumQuality,
  // );
}

String _formatFileSize(int bytes) {
  if (bytes < 1024) return '$bytes B';
  if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(2)} KB';
  if (bytes < 1024 * 1024 * 1024) {
    return '${(bytes / (1024 * 1024)).toStringAsFixed(2)} MB';
  }
  return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(2)} GB';
}
```

---

## 第7章 图像流处理详解

### 7.1 图像流概述

图像流（Image Stream）功能允许你实时获取相机捕获的每一帧图像数据。这在以下场景中非常有用：

- 实时图像识别（如二维码扫描）
- 实时滤镜处理
- 运动检测
- 人脸识别
- AR 增强现实

### 7.2 CameraImage 结构

```dart
class CameraImage {
  final CameraImageFormat format;     // 图像格式信息
  final int height;                    // 图像高度
  final int width;                     // 图像宽度
  final List<Plane> planes;            // 图像数据平面
}

class CameraImageFormat {
  final ImageFormatGroup group;        // 格式组
  final int raw;                       // 原始格式标识
}

class Plane {
  final Uint8List bytes;               // 图像数据
  final int bytesPerRow;               // 每行字节数
  final int? bytesPerPixel;            // 每像素字节数（可能为 null）
}
```

### 7.3 启动图像流

```dart
Future<void> startImageStream() async {
  await _controller.startImageStream((CameraImage image) {
    // 处理每一帧图像
    _processImage(image);
  });
}

void _processImage(CameraImage image) {
  print('图像尺寸: ${image.width}x${image.height}');
  print('图像格式: ${image.format.group}');
  print('平面数量: ${image.planes.length}');

  // YUV420 格式通常有 3 个平面
  // Plane 0: Y 亮度数据
  // Plane 1: U 色度数据
  // Plane 2: V 色度数据
  for (int i = 0; i < image.planes.length; i++) {
    final plane = image.planes[i];
    print('Plane $i:');
    print('  数据长度: ${plane.bytes.length}');
    print('  每行字节: ${plane.bytesPerRow}');
  }
}
```

### 7.4 二维码扫描示例

```dart
import 'package:google_mlkit_barcode_scanning/google_mlkit_barcode_scanning.dart';

class QRCodeScanner extends StatefulWidget {
  @override
  QRCodeScannerState createState() => QRCodeScannerState();
}

class QRCodeScannerState extends State<QRCodeScanner> {
  late CameraController _controller;
  final BarcodeScanner _barcodeScanner = BarcodeScanner();
  bool _isProcessing = false;
  String? _detectedCode;

  @override
  void initState() {
    super.initState();
    _initializeCamera();
  }

  Future<void> _initializeCamera() async {
    final cameras = await availableCameras();
    _controller = CameraController(
      cameras.first,
      ResolutionPreset.medium,
      enableAudio: false,
    );

    await _controller.initialize();

    // 启动图像流
    await _controller.startImageStream(_processImageStream);

    if (mounted) setState(() {});
  }

  Future<void> _processImageStream(CameraImage image) async {
    if (_isProcessing) return;

    _isProcessing = true;

    try {
      // 创建 InputImage
      final inputImage = _convertToInputImage(image);

      // 扫描条码
      final barcodes = await _barcodeScanner.processImage(inputImage);

      if (barcodes.isNotEmpty) {
        final code = barcodes.first.rawValue;
        if (code != null && code != _detectedCode) {
          setState(() => _detectedCode = code);
          print('检测到二维码: $code');
        }
      }
    } catch (e) {
      print('处理图像失败: $e');
    } finally {
      _isProcessing = false;
    }
  }

  InputImage _convertToInputImage(CameraImage image) {
    // 转换 CameraImage 为 InputImage
    // 具体实现取决于使用的 ML Kit 版本
    // ...
    throw UnimplementedError();
  }

  @override
  void dispose() {
    _controller.stopImageStream();
    _controller.dispose();
    _barcodeScanner.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (!_controller.value.isInitialized) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    return Scaffold(
      body: Stack(
        children: [
          CameraPreview(_controller),

          // 扫描框
          Center(
            child: Container(
              width: 250,
              height: 250,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.green, width: 2),
                borderRadius: BorderRadius.circular(12),
              ),
            ),
          ),

          // 结果显示
          if (_detectedCode != null)
            Positioned(
              bottom: 100,
              left: 20,
              right: 20,
              child: Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.black87,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Text(
                  '扫描结果: $_detectedCode',
                  style: const TextStyle(color: Colors.white),
                  textAlign: TextAlign.center,
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```

### 7.5 性能优化建议

```dart
class OptimizedImageStream extends StatefulWidget {
  @override
  OptimizedImageStreamState createState() => OptimizedImageStreamState();
}

class OptimizedImageStreamState extends State<OptimizedImageStream> {
  late CameraController _controller;
  DateTime? _lastProcessTime;
  final _processInterval = const Duration(milliseconds: 100); // 限制处理频率

  Future<void> _processImageStream(CameraImage image) async {
    final now = DateTime.now();

    // 限制处理频率，避免 CPU 过载
    if (_lastProcessTime != null &&
        now.difference(_lastProcessTime!) < _processInterval) {
      return;
    }
    _lastProcessTime = now;

    // 在 isolate 中处理，避免阻塞 UI
    await compute(_processInIsolate, image);
  }

  static void _processInIsolate(CameraImage image) {
    // 在后台 isolate 中处理图像
    // 可以进行复杂的图像处理而不会阻塞 UI
  }

  @override
  void dispose() {
    _controller.stopImageStream();
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

Flutter 框架小知识

**图像流性能注意事项**

| 因素     | 建议                        |
| -------- | --------------------------- |
| 分辨率   | 使用 `low` 或 `medium` 预设 |
| 处理频率 | 限制帧处理频率（如 10 FPS） |
| 处理位置 | 使用 isolate 避免阻塞 UI    |
| 内存管理 | 及时释放不再使用的图像数据  |
| 电池消耗 | 不需要时立即停止图像流      |

---

## 第8章 实战应用

### 8.1 完整相机应用示例

下面是一个功能完整的相机应用，包含拍照、录像、切换摄像头、闪光灯控制等功能：

```dart
import 'dart:async';
import 'dart:io';
import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' show join;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final cameras = await availableCameras();
  runApp(MyApp(cameras: cameras));
}

class MyApp extends StatelessWidget {
  final List<CameraDescription> cameras;

  const MyApp({super.key, required this.cameras});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Camera',
      theme: ThemeData.dark(),
      home: CameraHome(cameras: cameras),
    );
  }
}

class CameraHome extends StatefulWidget {
  final List<CameraDescription> cameras;

  const CameraHome({super.key, required this.cameras});

  @override
  CameraHomeState createState() => CameraHomeState();
}

class CameraHomeState extends State<CameraHome>
    with WidgetsBindingObserver, TickerProviderStateMixin {
  CameraController? _controller;
  bool _isReady = false;
  int _selectedCameraIndex = 0;
  FlashMode _flashMode = FlashMode.auto;
  double _zoomLevel = 1.0;
  double _maxZoom = 1.0;
  double _minZoom = 1.0;

  // 视频录制相关
  bool _isRecording = false;
  bool _isPaused = false;
  Timer? _recordingTimer;
  int _recordingSeconds = 0;

  // 模式切换
  bool _isVideoMode = false;

  late AnimationController _modeSwitchController;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    _modeSwitchController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 300),
    );
    _initializeCamera();
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    _modeSwitchController.dispose();
    _recordingTimer?.cancel();
    _controller?.dispose();
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.inactive) {
      _controller?.dispose();
    } else if (state == AppLifecycleState.resumed) {
      _initializeCamera();
    }
  }

  Future<void> _initializeCamera() async {
    if (widget.cameras.isEmpty) {
      print('没有可用相机');
      return;
    }

    await _initializeCameraController(widget.cameras[_selectedCameraIndex]);
  }

  Future<void> _initializeCameraController(
    CameraDescription cameraDescription,
  ) async {
    final controller = CameraController(
      cameraDescription,
      ResolutionPreset.veryHigh,
      enableAudio: true,
    );

    _controller = controller;

    try {
      await controller.initialize();

      // 获取变焦范围
      final maxZoom = await controller.getMaxZoomLevel();
      final minZoom = await controller.getMinZoomLevel();

      setState(() {
        _isReady = true;
        _maxZoom = maxZoom;
        _minZoom = minZoom;
      });
    } on CameraException catch (e) {
      print('相机初始化失败: ${e.description}');
    }
  }

  Future<void> _switchCamera() async {
    if (widget.cameras.length < 2) return;

    _selectedCameraIndex = (_selectedCameraIndex + 1) % widget.cameras.length;

    await _initializeCameraController(widget.cameras[_selectedCameraIndex]);
  }

  Future<void> _toggleFlash() async {
    switch (_flashMode) {
      case FlashMode.off:
        _flashMode = FlashMode.auto;
        break;
      case FlashMode.auto:
        _flashMode = FlashMode.always;
        break;
      case FlashMode.always:
        _flashMode = FlashMode.torch;
        break;
      case FlashMode.torch:
        _flashMode = FlashMode.off;
        break;
    }

    await _controller?.setFlashMode(_flashMode);
    setState(() {});
  }

  IconData get _flashIcon {
    switch (_flashMode) {
      case FlashMode.off:
        return Icons.flash_off;
      case FlashMode.auto:
        return Icons.flash_auto;
      case FlashMode.always:
        return Icons.flash_on;
      case FlashMode.torch:
        return Icons.highlight;
    }
  }

  Future<void> _takePicture() async {
    if (!_isReady || _controller == null) return;

    try {
      final XFile image = await _controller!.takePicture();

      if (!mounted) return;

      // 显示预览
      await Navigator.push(
        context,
        MaterialPageRoute(
          builder: (context) => MediaPreview(
            path: image.path,
            isVideo: false,
          ),
        ),
      );
    } catch (e) {
      print('拍照失败: $e');
    }
  }

  Future<void> _toggleRecording() async {
    if (_isRecording) {
      // 停止录制
      _recordingTimer?.cancel();

      try {
        final XFile video = await _controller!.stopVideoRecording();

        setState(() {
          _isRecording = false;
          _isPaused = false;
          _recordingSeconds = 0;
        });

        if (!mounted) return;

        await Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) => MediaPreview(
              path: video.path,
              isVideo: true,
            ),
          ),
        );
      } catch (e) {
        print('停止录制失败: $e');
      }
    } else {
      // 开始录制
      try {
        await _controller!.startVideoRecording();

        setState(() {
          _isRecording = true;
          _isPaused = false;
          _recordingSeconds = 0;
        });

        _recordingTimer = Timer.periodic(
          const Duration(seconds: 1),
          (timer) => setState(() => _recordingSeconds++),
        );
      } catch (e) {
        print('开始录制失败: $e');
      }
    }
  }

  Future<void> _pauseResumeRecording() async {
    if (!_isRecording) return;

    if (_isPaused) {
      await _controller!.resumeVideoRecording();
      _recordingTimer = Timer.periodic(
        const Duration(seconds: 1),
        (timer) => setState(() => _recordingSeconds++),
      );
    } else {
      await _controller!.pauseVideoRecording();
      _recordingTimer?.cancel();
    }

    setState(() => _isPaused = !_isPaused);
  }

  String get _formattedTime {
    final minutes = (_recordingSeconds ~/ 60).toString().padLeft(2, '0');
    final seconds = (_recordingSeconds % 60).toString().padLeft(2, '0');
    return '$minutes:$seconds';
  }

  void _setZoom(double zoom) {
    zoom = zoom.clamp(_minZoom, _maxZoom);
    _controller?.setZoomLevel(zoom);
    setState(() => _zoomLevel = zoom);
  }

  @override
  Widget build(BuildContext context) {
    if (!_isReady || _controller == null) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    return Scaffold(
      backgroundColor: Colors.black,
      body: Stack(
        children: [
          // 相机预览
          Positioned.fill(
            child: AspectRatio(
              aspectRatio: _controller!.value.aspectRatio,
              child: CameraPreview(_controller!),
            ),
          ),

          // 顶部控制栏
          SafeArea(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  // 闪光灯按钮
                  IconButton(
                    icon: Icon(_flashIcon),
                    color: Colors.white,
                    onPressed: _toggleFlash,
                  ),

                  // 录制时间显示
                  if (_isRecording)
                    Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 12,
                        vertical: 6,
                      ),
                      decoration: BoxDecoration(
                        color: Colors.black54,
                        borderRadius: BorderRadius.circular(16),
                      ),
                      child: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          Container(
                            width: 10,
                            height: 10,
                            decoration: BoxDecoration(
                              color: _isPaused ? Colors.yellow : Colors.red,
                              shape: BoxShape.circle,
                            ),
                          ),
                          const SizedBox(width: 6),
                          Text(
                            _formattedTime,
                            style: const TextStyle(
                              color: Colors.white,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),

                  // 切换摄像头按钮
                  IconButton(
                    icon: const Icon(Icons.flip_camera_ios),
                    color: Colors.white,
                    onPressed: _switchCamera,
                  ),
                ],
              ),
            ),
          ),

          // 变焦滑块
          Positioned(
            top: 100,
            right: 16,
            child: RotatedBox(
              quarterTurns: 3,
              child: SizedBox(
                width: 200,
                child: Slider(
                  value: _zoomLevel,
                  min: _minZoom,
                  max: _maxZoom,
                  onChanged: _setZoom,
                  activeColor: Colors.white,
                  inactiveColor: Colors.white54,
                ),
              ),
            ),
          ),

          // 底部控制区
          Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            child: Container(
              padding: const EdgeInsets.all(24),
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.bottomCenter,
                  end: Alignment.topCenter,
                  colors: [
                    Colors.black87,
                    Colors.transparent,
                  ],
                ),
              ),
              child: SafeArea(
                child: Column(
                  children: [
                    // 模式切换
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        TextButton(
                          onPressed: () => setState(() => _isVideoMode = false),
                          child: Text(
                            '照片',
                            style: TextStyle(
                              color: !_isVideoMode ? Colors.yellow : Colors.white,
                              fontWeight: !_isVideoMode
                                  ? FontWeight.bold
                                  : FontWeight.normal,
                            ),
                          ),
                        ),
                        const SizedBox(width: 24),
                        TextButton(
                          onPressed: () => setState(() => _isVideoMode = true),
                          child: Text(
                            '视频',
                            style: TextStyle(
                              color: _isVideoMode ? Colors.yellow : Colors.white,
                              fontWeight: _isVideoMode
                                  ? FontWeight.bold
                                  : FontWeight.normal,
                            ),
                          ),
                        ),
                      ],
                    ),

                    const SizedBox(height: 20),

                    // 快门按钮
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: [
                        // 占位
                        const SizedBox(width: 60),

                        // 快门按钮
                        GestureDetector(
                          onTap: _isVideoMode ? _toggleRecording : _takePicture,
                          child: Container(
                            width: 80,
                            height: 80,
                            decoration: BoxDecoration(
                              shape: BoxShape.circle,
                              border: Border.all(
                                color: Colors.white,
                                width: 4,
                              ),
                              color: _isRecording ? Colors.red : Colors.transparent,
                            ),
                            child: _isRecording
                                ? const Icon(
                                    Icons.stop,
                                    color: Colors.white,
                                    size: 40,
                                  )
                                : _isVideoMode
                                    ? const Icon(
                                        Icons.videocam,
                                        color: Colors.red,
                                        size: 40,
                                      )
                                    : const Icon(
                                        Icons.camera_alt,
                                        color: Colors.white,
                                        size: 40,
                                      ),
                          ),
                        ),

                        // 暂停按钮（仅录制视频时显示）
                        if (_isRecording)
                          IconButton(
                            icon: Icon(
                              _isPaused ? Icons.play_arrow : Icons.pause,
                            ),
                            color: Colors.white,
                            iconSize: 40,
                            onPressed: _pauseResumeRecording,
                          )
                        else
                          const SizedBox(width: 60),
                      ],
                    ),
                  ],
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// 媒体预览页面
class MediaPreview extends StatelessWidget {
  final String path;
  final bool isVideo;

  const MediaPreview({
    super.key,
    required this.path,
    required this.isVideo,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        backgroundColor: Colors.black,
        actions: [
          IconButton(
            icon: const Icon(Icons.share),
            onPressed: () {
              // 分享功能
            },
          ),
          IconButton(
            icon: const Icon(Icons.delete),
            onPressed: () {
              File(path).delete();
              Navigator.pop(context);
            },
          ),
        ],
      ),
      body: Center(
        child: isVideo
            ? const Icon(
                Icons.videocam,
                size: 100,
                color: Colors.white,
              )
            : Image.file(File(path)),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 保存到相册
          Navigator.pop(context);
        },
        child: const Icon(Icons.save),
      ),
    );
  }
}
```

### 8.2 相机权限处理

```dart
import 'package:permission_handler/permission_handler.dart';

class PermissionHandler {
  // 检查并请求相机权限
  static Future<bool> requestCameraPermission() async {
    final status = await Permission.camera.request();
    return status.isGranted;
  }

  // 检查并请求麦克风权限
  static Future<bool> requestMicrophonePermission() async {
    final status = await Permission.microphone.request();
    return status.isGranted;
  }

  // 检查并请求存储权限（Android）
  static Future<bool> requestStoragePermission() async {
    if (Platform.isAndroid) {
      final status = await Permission.storage.request();
      return status.isGranted;
    }
    return true;
  }

  // 请求所有需要的权限
  static Future<bool> requestAllPermissions() async {
    final camera = await requestCameraPermission();
    final microphone = await requestMicrophonePermission();
    final storage = await requestStoragePermission();

    return camera && microphone && storage;
  }
}

// 使用示例
class PermissionWrapper extends StatelessWidget {
  final Widget child;

  const PermissionWrapper({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<bool>(
      future: PermissionHandler.requestAllPermissions(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }

        if (snapshot.data == true) {
          return child;
        }

        return Scaffold(
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.error, size: 64, color: Colors.red),
                const SizedBox(height: 16),
                const Text('需要相机和麦克风权限'),
                const SizedBox(height: 16),
                ElevatedButton(
                  onPressed: () => openAppSettings(),
                  child: const Text('前往设置'),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

---

## 附录

### 附录A：CameraController 方法速查表

| 方法                     | 返回值           | 用途           |
| ------------------------ | ---------------- | -------------- |
| `initialize()`           | `Future<void>`   | 初始化控制器   |
| `dispose()`              | `Future<void>`   | 释放资源       |
| `takePicture()`          | `Future<XFile>`  | 拍照           |
| `startVideoRecording()`  | `Future<void>`   | 开始录像       |
| `stopVideoRecording()`   | `Future<XFile>`  | 停止录像       |
| `pauseVideoRecording()`  | `Future<void>`   | 暂停录像       |
| `resumeVideoRecording()` | `Future<void>`   | 恢复录像       |
| `startImageStream()`     | `Future<void>`   | 开始图像流     |
| `stopImageStream()`      | `Future<void>`   | 停止图像流     |
| `setFlashMode()`         | `Future<void>`   | 设置闪光灯模式 |
| `setZoomLevel()`         | `Future<void>`   | 设置变焦级别   |
| `getMaxZoomLevel()`      | `Future<double>` | 获取最大变焦   |
| `getMinZoomLevel()`      | `Future<double>` | 获取最小变焦   |
| `setExposureMode()`      | `Future<void>`   | 设置曝光模式   |
| `setExposureOffset()`    | `Future<void>`   | 设置曝光补偿   |
| `setFocusMode()`         | `Future<void>`   | 设置对焦模式   |
| `setFocusPoint()`        | `Future<void>`   | 设置对焦点     |

### 附录B：CameraValue 属性速查表

| 属性                | 类型           | 说明               |
| ------------------- | -------------- | ------------------ |
| `isInitialized`     | `bool`         | 是否已初始化       |
| `isRecordingVideo`  | `bool`         | 是否正在录像       |
| `isTakingPicture`   | `bool`         | 是否正在拍照       |
| `isStreamingImages` | `bool`         | 是否正在传输图像流 |
| `flashMode`         | `FlashMode`    | 当前闪光灯模式     |
| `exposureMode`      | `ExposureMode` | 当前曝光模式       |
| `focusMode`         | `FocusMode`    | 当前对焦模式       |
| `exposureOffset`    | `double`       | 曝光补偿值         |
| `zoomLevel`         | `double`       | 当前变焦级别       |
| `maxZoomLevel`      | `double`       | 最大变焦级别       |
| `minZoomLevel`      | `double`       | 最小变焦级别       |
| `aspectRatio`       | `double`       | 预览画面宽高比     |
| `previewSize`       | `Size?`        | 预览画面尺寸       |

### 附录C：枚举类型速查表

#### FlashMode

| 值       | 说明       |
| -------- | ---------- |
| `off`    | 关闭       |
| `auto`   | 自动       |
| `always` | 始终开启   |
| `torch`  | 手电筒模式 |

#### ExposureMode

| 值       | 说明     |
| -------- | -------- |
| `auto`   | 自动曝光 |
| `locked` | 锁定曝光 |

#### FocusMode

| 值       | 说明     |
| -------- | -------- |
| `auto`   | 自动对焦 |
| `locked` | 锁定对焦 |

#### ResolutionPreset

| 值          | 分辨率（约） |
| ----------- | ------------ |
| `low`       | 320x240      |
| `medium`    | 720x480      |
| `high`      | 1280x720     |
| `veryHigh`  | 1920x1080    |
| `ultraHigh` | 3840x2160    |
| `max`       | 设备最大     |

#### CameraLensDirection

| 值         | 说明       |
| ---------- | ---------- |
| `front`    | 前置摄像头 |
| `back`     | 后置摄像头 |
| `external` | 外接摄像头 |

### 附录D：常见问题与解决方案

#### Q1: 相机初始化失败

**问题**：调用 `initialize()` 时抛出异常。

**解决方案**：

```dart
// 1. 确保已添加权限
// 2. 检查相机是否被其他应用占用
// 3. 确保在主 isolate 中初始化

try {
  await controller.initialize();
} on CameraException catch (e) {
  if (e.code == 'CameraAccessDenied') {
    // 引导用户开启权限
  } else if (e.code == 'CameraAccessRestricted') {
    // 相机被限制使用（如家长控制）
  }
}
```

#### Q2: 拍照后图片方向不正确

**问题**：拍摄的照片方向与预览不一致。

**解决方案**：

```dart
// 使用 exif 包读取图片方向信息
import 'package:exif/exif.dart';

Future<void> fixImageOrientation(String path) async {
  final file = File(path);
  final bytes = await file.readAsBytes();
  final data = await readExifFromBytes(bytes);

  final orientation = data['Image Orientation'];
  print('图片方向: $orientation');

  // 根据方向信息旋转图片
}
```

#### Q3: 图像流导致应用卡顿

**问题**：开启图像流后 UI 卡顿。

**解决方案**：

```dart
// 1. 降低处理频率
DateTime? lastProcess;
await controller.startImageStream((image) {
  final now = DateTime.now();
  if (lastProcess != null &&
      now.difference(lastProcess!).inMilliseconds < 100) {
    return; // 跳过此帧
  }
  lastProcess = now;
  processImage(image);
});

// 2. 在 isolate 中处理
import 'package:flutter/foundation.dart';

await compute(processInIsolate, imageData);
```

#### Q4: 视频录制没有声音

**问题**：录制的视频没有音频。

**解决方案**：

```dart
// 确保启用音频
final controller = CameraController(
  camera,
  ResolutionPreset.high,
  enableAudio: true,  // 必须设置为 true
);

// 检查麦克风权限
final status = await Permission.microphone.request();
if (!status.isGranted) {
  print('需要麦克风权限');
}
```

### 附录E：参考资源

| 资源             | 链接                                                                   |
| ---------------- | ---------------------------------------------------------------------- |
| Pub 包页面       | https://pub.dev/packages/camera                                        |
| 官方文档         | https://docs.flutter.dev/cookbook/plugins/picture-using-camera         |
| GitHub 仓库      | https://github.com/flutter/packages/tree/main/packages/camera/camera   |
| API 参考         | https://pub.dev/documentation/camera/latest/camera/camera-library.html |
| Flutter 中文文档 | https://docs.flutter.cn/cookbook/plugins/picture-using-camera/         |

### 附录F：相关插件推荐

| 插件                  | 用途                       |
| --------------------- | -------------------------- |
| `image_picker`        | 从相册选择图片/视频        |
| `photo_manager`       | 访问相册资源               |
| `gallery_saver`       | 保存到系统相册             |
| `google_mlkit_vision` | 图像识别（二维码、人脸等） |
| `video_player`        | 播放录制的视频             |
| `video_compress`      | 视频压缩                   |
| `image_cropper`       | 图片裁剪                   |
| `permission_handler`  | 权限管理                   |

---

**文档版本**: 1.0  
**适用版本**: camera ^0.12.0  
**最后更新**: 2025年
