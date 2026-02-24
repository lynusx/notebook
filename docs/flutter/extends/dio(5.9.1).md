# Flutter dio 5.9.1 完整详解与实战指南

## 前言

在 Flutter 应用开发中，网络请求是不可或缺的一部分。虽然 Dart 提供了原生的 `dart:io` 和 `package:http` 来处理 HTTP 请求，但对于复杂的应用场景，它们往往显得力不从心。

**dio** 是 Flutter 中文网（flutter.cn）开源的一个强大的 Dart HTTP 请求库，它提供了丰富的功能和灵活的配置选项，包括：

- 全局配置和拦截器
- 请求取消
- FormData 和文件上传/下载
- 超时设置
- 自定义适配器
- 转换器
- HTTP/2 支持

本书基于 dio 5.9.1 版本，详细介绍该包的所有核心类、API 用法，以及如何将其集成到 Flutter 应用中。

---

## 第1章 dio 基础

### 1.1 什么是 dio

dio 是一个强大的 Dart HTTP 客户端，相比原生的 `http` 包，它提供了更多高级功能：

| 特性 | dio | http 包 |
|------|-----|---------|
| 拦截器 | ✅ 支持 | ❌ 不支持 |
| 请求取消 | ✅ 支持 | ❌ 不支持 |
| 文件上传/下载 | ✅ 原生支持 | ⚠️ 需额外处理 |
| 超时设置 | ✅ 支持 | ⚠️ 有限支持 |
| FormData | ✅ 原生支持 | ❌ 不支持 |
| 全局配置 | ✅ 支持 | ❌ 不支持 |
| HTTP/2 | ✅ 支持插件 | ❌ 不支持 |

### 1.2 安装与配置

#### 1.2.1 添加依赖

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  flutter:
    sdk: flutter
  dio: ^5.9.1

# 可选插件
dev_dependencies:
  # Cookie 管理
  dio_cookie_manager: ^3.1.1
  # HTTP/2 支持
  dio_http2_adapter: ^2.5.0
  # 智能重试
  dio_smart_retry: ^6.0.0
```

#### 1.2.2 基础项目结构

```
lib/
├── main.dart
├── api/
│   ├── api_client.dart       # Dio 实例配置
│   ├── interceptors/         # 拦截器
│   │   ├── auth_interceptor.dart
│   │   ├── log_interceptor.dart
│   │   └── error_interceptor.dart
│   └── services/             # API 服务
│       ├── user_service.dart
│       └── product_service.dart
├── models/                   # 数据模型
│   ├── user.dart
│   └── product.dart
└── utils/
    └── exceptions.dart
```

---

### 1.3 第一个 dio 请求

#### 1.3.1 创建 Dio 实例

```dart
import 'package:dio/dio.dart';

// 创建默认实例
final dio = Dio();

// 或使用自定义配置
final dio = Dio(
  BaseOptions(
    baseUrl: 'https://api.example.com',
    connectTimeout: Duration(seconds: 5),
    receiveTimeout: Duration(seconds: 3),
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    },
  ),
);
```

#### 1.3.2 发起 GET 请求

```dart
import 'package:dio/dio.dart';

void getUser() async {
  try {
    final response = await dio.get('/user/123');
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}
```

#### 1.3.3 发起 POST 请求

```dart
void createUser() async {
  try {
    final response = await dio.post(
      '/users',
      data: {
        'name': 'John Doe',
        'email': 'john@example.com',
      },
    );
    print(response.data);
  } catch (e) {
    print('Error: $e');
  }
}
```

> **Flutter框架小知识**
> 
> dio 的请求方法（get、post、put 等）返回的都是 `Future<Response>`，这意味着你可以使用 `async/await` 来处理异步请求，也可以使用 `.then()` 和 `.catchError()` 来处理响应和错误。

---

## 第2章 Dio 核心类详解

### 2.1 Dio 类

`Dio` 是 dio 包的核心类，用于发起 HTTP 请求。

#### 2.1.1 构造函数

```dart
Dio([BaseOptions? options])
```

#### 2.1.2 主要属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| options | BaseOptions | 默认请求配置 |
| interceptors | Interceptors | 拦截器列表 |
| httpClientAdapter | HttpClientAdapter | HTTP 客户端适配器 |
| transformer | Transformer | 请求/响应转换器 |

#### 2.1.3 主要方法

| 方法名 | 返回值 | 说明 |
|--------|--------|------|
| get | Future<Response> | GET 请求 |
| post | Future<Response> | POST 请求 |
| put | Future<Response> | PUT 请求 |
| delete | Future<Response> | DELETE 请求 |
| patch | Future<Response> | PATCH 请求 |
| head | Future<Response> | HEAD 请求 |
| download | Future<Response> | 下载文件 |
| request | Future<Response> | 通用请求方法 |
| fetch | Future<Response> | 使用 RequestOptions 发起请求 |
| close | void | 关闭 Dio 实例 |

#### 2.1.4 完整示例

```dart
final dio = Dio();

// GET 请求
final getResponse = await dio.get(
  'https://api.example.com/users',
  queryParameters: {'page': 1, 'limit': 10},
  options: Options(headers: {'Authorization': 'Bearer token'}),
);

// POST 请求
final postResponse = await dio.post(
  'https://api.example.com/users',
  data: {'name': 'John', 'email': 'john@example.com'},
);

// PUT 请求
final putResponse = await dio.put(
  'https://api.example.com/users/1',
  data: {'name': 'Jane'},
);

// DELETE 请求
final deleteResponse = await dio.delete('https://api.example.com/users/1');
```

---

### 2.2 BaseOptions

`BaseOptions` 用于配置 Dio 实例的默认选项。

#### 2.2.1 构造函数参数

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| method | String? | null | 默认请求方法 |
| baseUrl | String? | null | 基础 URL |
| headers | Map<String, dynamic>? | null | 默认请求头 |
| queryParameters | Map<String, dynamic>? | null | 默认查询参数 |
| connectTimeout | Duration? | null | 连接超时时间 |
| receiveTimeout | Duration? | null | 接收超时时间 |
| sendTimeout | Duration? | null | 发送超时时间 |
| contentType | String? | null | 内容类型 |
| responseType | ResponseType? | null | 响应类型 |
| validateStatus | ValidateStatus? | null | 状态码验证函数 |
| receiveDataWhenStatusError | bool? | null | 错误状态时是否接收数据 |
| followRedirects | bool? | null | 是否跟随重定向 |
| maxRedirects | int? | null | 最大重定向次数 |
| persistentConnection | bool? | null | 是否保持持久连接 |
| requestEncoder | RequestEncoder? | null | 请求编码器 |
| responseDecoder | ResponseDecoder? | null | 响应解码器 |
| listFormat | ListFormat? | null | 列表参数格式 |

#### 2.2.2 完整示例

```dart
final options = BaseOptions(
  baseUrl: 'https://api.example.com/v1',
  connectTimeout: Duration(seconds: 30),
  receiveTimeout: Duration(seconds: 30),
  sendTimeout: Duration(seconds: 30),
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'X-API-Key': 'your-api-key',
  },
  queryParameters: {
    'api_version': '1.0',
  },
  responseType: ResponseType.json,
  validateStatus: (status) {
    return status != null && status >= 200 && status < 300;
  },
);

final dio = Dio(options);
```

---

### 2.3 Options

`Options` 用于配置单次请求的选项，会覆盖 `BaseOptions` 中的配置。

#### 2.3.1 构造函数参数

与 `BaseOptions` 类似，但优先级更高。

#### 2.3.2 使用示例

```dart
final response = await dio.get(
  '/users',
  options: Options(
    headers: {'Authorization': 'Bearer $token'},
    responseType: ResponseType.plain,
    followRedirects: false,
  ),
);
```

---

### 2.4 RequestOptions

`RequestOptions` 包含完整的请求配置信息，在拦截器中使用。

#### 2.4.1 主要属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| method | String | 请求方法 |
| path | String | 请求路径 |
| data | dynamic | 请求数据 |
| queryParameters | Map<String, dynamic>? | 查询参数 |
| headers | Map<String, dynamic> | 请求头 |
| baseUrl | String | 基础 URL |
| connectTimeout | Duration? | 连接超时 |
| receiveTimeout | Duration? | 接收超时 |
| sendTimeout | Duration? | 发送超时 |
| cancelToken | CancelToken? | 取消令牌 |
| extra | Map<String, dynamic> | 额外数据 |

---

### 2.5 Response

`Response` 封装了 HTTP 响应的所有信息。

#### 2.5.1 主要属性

| 属性名 | 类型 | 说明 |
|--------|------|------|
| data | T | 响应数据 |
| statusCode | int? | HTTP 状态码 |
| statusMessage | String? | 状态消息 |
| headers | Headers | 响应头 |
| requestOptions | RequestOptions | 请求配置 |
| isRedirect | bool | 是否重定向 |
| redirects | List<RedirectRecord> | 重定向记录 |
| extra | Map<String, dynamic> | 额外数据 |

#### 2.5.2 使用示例

```dart
final response = await dio.get('/user/123');

print('Status: ${response.statusCode}');
print('Data: ${response.data}');
print('Headers: ${response.headers}');
print('Content-Type: ${response.headers.value('content-type')}');
```

---

## 第3章 HTTP 请求方法详解

### 3.1 GET 请求

```dart
// 基础 GET 请求
final response = await dio.get('/users');

// 带查询参数
final response = await dio.get(
  '/users',
  queryParameters: {
    'page': 1,
    'limit': 10,
    'search': 'john',
  },
);

// 带自定义选项
final response = await dio.get(
  '/users',
  queryParameters: {'page': 1},
  options: Options(
    headers: {'Authorization': 'Bearer $token'},
    responseType: ResponseType.json,
  ),
);
```

### 3.2 POST 请求

```dart
// JSON 数据
final response = await dio.post(
  '/users',
  data: {
    'name': 'John Doe',
    'email': 'john@example.com',
    'age': 30,
  },
);

// FormData（表单数据）
final formData = FormData.fromMap({
  'name': 'John Doe',
  'email': 'john@example.com',
});
final response = await dio.post('/users', data: formData);

// 纯文本
final response = await dio.post(
  '/users',
  data: 'Plain text body',
  options: Options(contentType: 'text/plain'),
);

// 二进制数据
final bytes = Uint8List.fromList([1, 2, 3, 4, 5]);
final response = await dio.post(
  '/upload',
  data: bytes,
  options: Options(contentType: 'application/octet-stream'),
);
```

### 3.3 PUT 请求

```dart
final response = await dio.put(
  '/users/123',
  data: {
    'name': 'Jane Doe',
    'email': 'jane@example.com',
  },
);
```

### 3.4 DELETE 请求

```dart
// 简单删除
final response = await dio.delete('/users/123');

// 带请求体
final response = await dio.delete(
  '/users/batch',
  data: {
    'ids': [1, 2, 3],
  },
);
```

### 3.5 PATCH 请求

```dart
final response = await dio.patch(
  '/users/123',
  data: {
    'name': 'Updated Name',
  },
);
```

### 3.6 通用 request 方法

```dart
final response = await dio.request(
  '/users',
  options: Options(method: 'GET'),
  queryParameters: {'page': 1},
);
```

---

## 第4章 拦截器

拦截器是 dio 最强大的功能之一，可以在请求发送前或响应接收后执行自定义逻辑。

### 4.1 Interceptor 类

```dart
class MyInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // 请求发送前的处理
    print('Request: ${options.method} ${options.path}');
    handler.next(options); // 继续请求
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // 响应接收后的处理
    print('Response: ${response.statusCode}');
    handler.next(response); // 继续响应
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // 错误处理
    print('Error: ${err.message}');
    handler.next(err); // 继续错误传播
  }
}
```

### 4.2 添加拦截器

```dart
final dio = Dio();

// 添加单个拦截器
dio.interceptors.add(MyInterceptor());

// 添加多个拦截器
dio.interceptors.addAll([
  LogInterceptor(),
  AuthInterceptor(),
  ErrorInterceptor(),
]);

// 使用 InterceptorsWrapper 快速创建拦截器
dio.interceptors.add(
  InterceptorsWrapper(
    onRequest: (options, handler) {
      print('Request: ${options.uri}');
      handler.next(options);
    },
    onResponse: (response, handler) {
      print('Response: ${response.statusCode}');
      handler.next(response);
    },
    onError: (error, handler) {
      print('Error: ${error.message}');
      handler.next(error);
    },
  ),
);
```

### 4.3 认证拦截器

```dart
class AuthInterceptor extends Interceptor {
  String? _token;

  void setToken(String token) {
    _token = token;
  }

  void clearToken() {
    _token = null;
  }

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (_token != null) {
      options.headers['Authorization'] = 'Bearer $_token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token 过期，尝试刷新
      try {
        final newToken = await refreshToken();
        setToken(newToken);
        
        // 重试原请求
        final opts = err.requestOptions;
        opts.headers['Authorization'] = 'Bearer $newToken';
        final response = await dio.fetch(opts);
        handler.resolve(response);
        return;
      } catch (e) {
        // 刷新失败，跳转到登录页
        clearToken();
      }
    }
    handler.next(err);
  }

  Future<String> refreshToken() async {
    // 实现 Token 刷新逻辑
    return 'new_token';
  }
}
```

### 4.4 日志拦截器

```dart
// 使用内置的 LogInterceptor
dio.interceptors.add(LogInterceptor(
  request: true,
  requestHeader: true,
  requestBody: true,
  responseHeader: true,
  responseBody: true,
  error: true,
  logPrint: (object) => print(object),
));

// 自定义日志拦截器
class CustomLogInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('┌────── Request ──────');
    print('│ ${options.method} ${options.uri}');
    print('│ Headers: ${options.headers}');
    print('│ Data: ${options.data}');
    print('└─────────────────────');
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('┌────── Response ──────');
    print('│ Status: ${response.statusCode}');
    print('│ Data: ${response.data}');
    print('└──────────────────────');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print('┌────── Error ──────');
    print('│ ${err.message}');
    print('│ ${err.response?.statusCode}');
    print('└───────────────────');
    handler.next(err);
  }
}
```

### 4.5 错误处理拦截器

```dart
class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    switch (err.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        throw TimeoutException('连接超时，请检查网络');
      case DioExceptionType.badResponse:
        final statusCode = err.response?.statusCode;
        if (statusCode == 400) {
          throw BadRequestException('请求参数错误');
        } else if (statusCode == 401) {
          throw UnauthorizedException('未授权，请重新登录');
        } else if (statusCode == 403) {
          throw ForbiddenException('拒绝访问');
        } else if (statusCode == 404) {
          throw NotFoundException('资源不存在');
        } else if (statusCode == 500) {
          throw ServerException('服务器错误');
        }
        break;
      case DioExceptionType.cancel:
        throw RequestCancelledException('请求已取消');
      case DioExceptionType.connectionError:
        throw NetworkException('网络连接错误');
      default:
        throw UnknownException('未知错误: ${err.message}');
    }
    handler.next(err);
  }
}

// 自定义异常类
class TimeoutException implements Exception {
  final String message;
  TimeoutException(this.message);
  @override
  String toString() => message;
}

class BadRequestException implements Exception {
  final String message;
  BadRequestException(this.message);
  @override
  String toString() => message;
}

// 其他异常类...
```

---

## 第5章 文件上传与下载

### 5.1 文件上传

#### 5.1.1 使用 FormData 上传单个文件

```dart
import 'dart:io';
import 'package:dio/dio.dart';

Future<void> uploadFile(File file) async {
  final formData = FormData.fromMap({
    'name': 'John Doe',
    'description': 'A test file',
    'file': await MultipartFile.fromFile(
      file.path,
      filename: 'upload.jpg',
      contentType: MediaType('image', 'jpeg'),
    ),
  });

  final response = await dio.post(
    '/upload',
    data: formData,
    onSendProgress: (sent, total) {
      final progress = (sent / total * 100).toStringAsFixed(0);
      print('Upload progress: $progress%');
    },
  );

  print('Upload complete: ${response.data}');
}
```

#### 5.1.2 上传多个文件

```dart
Future<void> uploadMultipleFiles(List<File> files) async {
  final formData = FormData();
  
  formData.fields.add(MapEntry('userId', '123'));
  
  for (var i = 0; i < files.length; i++) {
    formData.files.add(
      MapEntry(
        'files[]',
        await MultipartFile.fromFile(
          files[i].path,
          filename: 'file_$i.jpg',
        ),
      ),
    );
  }

  final response = await dio.post('/upload/multiple', data: formData);
  print(response.data);
}
```

#### 5.1.3 从字节数组上传

```dart
Future<void> uploadBytes(List<int> bytes, String filename) async {
  final formData = FormData.fromMap({
    'file': MultipartFile.fromBytes(
      bytes,
      filename: filename,
      contentType: MediaType('application', 'pdf'),
    ),
  });

  final response = await dio.post('/upload', data: formData);
  print(response.data);
}
```

### 5.2 文件下载

#### 5.2.1 基础下载

```dart
Future<void> downloadFile(String url, String savePath) async {
  final response = await dio.download(
    url,
    savePath,
    onReceiveProgress: (received, total) {
      if (total != -1) {
        final progress = (received / total * 100).toStringAsFixed(0);
        print('Download progress: $progress%');
      }
    },
  );

  print('Download complete: ${response.statusCode}');
}
```

#### 5.2.2 带取消令牌的下载

```dart
final cancelToken = CancelToken();

Future<void> downloadWithCancel(String url, String savePath) async {
  try {
    final response = await dio.download(
      url,
      savePath,
      cancelToken: cancelToken,
      onReceiveProgress: (received, total) {
        print('Progress: $received / $total');
      },
    );
    print('Download complete');
  } on DioException catch (e) {
    if (CancelToken.isCancel(e)) {
      print('Download cancelled');
    }
  }
}

// 取消下载
void cancelDownload() {
  cancelToken.cancel('User cancelled');
}
```

#### 5.2.3 分段下载（断点续传）

```dart
Future<void> downloadWithResume(String url, String savePath) async {
  final file = File(savePath);
  int downloadedBytes = 0;
  
  if (await file.exists()) {
    downloadedBytes = await file.length();
  }

  final response = await dio.download(
    url,
    savePath,
    options: Options(
      headers: {
        'Range': 'bytes=$downloadedBytes-',
      },
    ),
    onReceiveProgress: (received, total) {
      final actualTotal = total + downloadedBytes;
      final progress = ((received + downloadedBytes) / actualTotal * 100)
          .toStringAsFixed(0);
      print('Progress: $progress%');
    },
  );

  print('Download complete: ${response.statusCode}');
}
```

#### 5.2.4 下载到内存

```dart
Future<Uint8List> downloadToMemory(String url) async {
  final response = await dio.get(
    url,
    options: Options(responseType: ResponseType.bytes),
  );
  
  return Uint8List.fromList(response.data);
}
```

---

## 第6章 请求取消

### 6.1 CancelToken

`CancelToken` 用于取消正在进行的请求。

```dart
// 创建 CancelToken
final cancelToken = CancelToken();

// 发起请求时传入
try {
  final response = await dio.get(
    '/long-request',
    cancelToken: cancelToken,
  );
  print(response.data);
} on DioException catch (e) {
  if (CancelToken.isCancel(e)) {
    print('Request cancelled: ${e.message}');
  }
}

// 取消请求
cancelToken.cancel('Cancelled by user');
```

### 6.2 多个请求共享 CancelToken

```dart
final cancelToken = CancelToken();

// 同时发起多个请求
final futures = [
  dio.get('/api/1', cancelToken: cancelToken),
  dio.get('/api/2', cancelToken: cancelToken),
  dio.get('/api/3', cancelToken: cancelToken),
];

// 取消所有请求
cancelToken.cancel('Cancel all');
```

---

## 第7章 转换器

转换器用于在请求发送前和响应接收后对数据进行转换。

### 7.1 Transformer 类

```dart
class CustomTransformer extends Transformer {
  @override
  Future<String> transformRequest(RequestOptions options) async {
    // 转换请求数据
    if (options.data is Map) {
      return jsonEncode(options.data);
    }
    return options.data.toString();
  }

  @override
  Future transformResponse(
    RequestOptions options,
    ResponseBody responseBody,
  ) async {
    // 转换响应数据
    final response = await responseBody.stream.bytesToString();
    return jsonDecode(response);
  }
}
```

### 7.2 设置转换器

```dart
final dio = Dio();
dio.transformer = CustomTransformer();

// 或使用内置的 BackgroundTransformer（在后台线程解析 JSON）
dio.transformer = BackgroundTransformer();
```

### 7.3 Flutter 中的 JSON 解析优化

```dart
// 在 Flutter 中使用 compute 在后台解析 JSON
_parseAndDecode(String response) {
  return jsonDecode(response);
}

parseJson(String text) {
  return compute(_parseAndDecode, text);
}

void main() {
  final dio = Dio();
  (dio.transformer as BackgroundTransformer).jsonDecodeCallback = parseJson;
}
```

---

## 第8章 错误处理

### 8.1 DioException

`DioException` 是 dio 中所有异常的基类。

#### 8.1.1 DioExceptionType 枚举

| 类型 | 说明 |
|------|------|
| connectionTimeout | 连接超时 |
| sendTimeout | 发送超时 |
| receiveTimeout | 接收超时 |
| badCertificate | 证书验证失败 |
| badResponse | 错误的响应（状态码不符合预期） |
| cancel | 请求被取消 |
| connectionError | 连接错误 |
| unknown | 未知错误 |

#### 8.1.2 错误处理示例

```dart
try {
  final response = await dio.get('/users');
} on DioException catch (e) {
  switch (e.type) {
    case DioExceptionType.connectionTimeout:
      print('连接超时');
      break;
    case DioExceptionType.sendTimeout:
      print('发送超时');
      break;
    case DioExceptionType.receiveTimeout:
      print('接收超时');
      break;
    case DioExceptionType.badResponse:
      print('服务器错误: ${e.response?.statusCode}');
      break;
    case DioExceptionType.cancel:
      print('请求被取消');
      break;
    case DioExceptionType.connectionError:
      print('连接错误');
      break;
    default:
      print('未知错误: ${e.message}');
  }
} catch (e) {
  print('其他错误: $e');
}
```

### 8.2 全局错误处理

```dart
class ApiClient {
  late final Dio _dio;

  ApiClient() {
    _dio = Dio(BaseOptions(baseUrl: 'https://api.example.com'));
    
    _dio.interceptors.add(
      InterceptorsWrapper(
        onError: (error, handler) {
          final customError = _handleError(error);
          handler.reject(customError);
        },
      ),
    );
  }

  DioException _handleError(DioException error) {
    String message;
    
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        message = '网络连接超时，请检查网络后重试';
        break;
      case DioExceptionType.badResponse:
        message = _getErrorMessage(error.response?.statusCode);
        break;
      case DioExceptionType.cancel:
        message = '请求已取消';
        break;
      case DioExceptionType.connectionError:
        message = '网络连接失败，请检查网络设置';
        break;
      default:
        message = '发生未知错误，请稍后重试';
    }
    
    return DioException(
      requestOptions: error.requestOptions,
      error: message,
      type: error.type,
    );
  }

  String _getErrorMessage(int? statusCode) {
    switch (statusCode) {
      case 400:
        return '请求参数错误';
      case 401:
        return '登录已过期，请重新登录';
      case 403:
        return '没有权限访问该资源';
      case 404:
        return '请求的资源不存在';
      case 500:
        return '服务器内部错误';
      default:
        return '服务器错误: $statusCode';
    }
  }
}
```

---

## 第9章 实战案例

### 9.1 完整的 API 客户端封装

```dart
// api/api_client.dart
import 'package:dio/dio.dart';

class ApiClient {
  static final ApiClient _instance = ApiClient._internal();
  factory ApiClient() => _instance;
  
  late final Dio _dio;
  String? _token;

  ApiClient._internal() {
    _dio = Dio(
      BaseOptions(
        baseUrl: 'https://api.example.com/v1',
        connectTimeout: Duration(seconds: 30),
        receiveTimeout: Duration(seconds: 30),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    _setupInterceptors();
  }

  void _setupInterceptors() {
    // 日志拦截器
    _dio.interceptors.add(LogInterceptor(
      request: true,
      requestHeader: true,
      requestBody: true,
      responseHeader: true,
      responseBody: true,
      error: true,
    ));

    // 认证拦截器
    _dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) {
          if (_token != null) {
            options.headers['Authorization'] = 'Bearer $_token';
          }
          handler.next(options);
        },
        onError: (error, handler) async {
          if (error.response?.statusCode == 401) {
            // Token 过期处理
            final refreshed = await _refreshToken();
            if (refreshed) {
              // 重试原请求
              final opts = error.requestOptions;
              opts.headers['Authorization'] = 'Bearer $_token';
              final response = await _dio.fetch(opts);
              handler.resolve(response);
              return;
            }
          }
          handler.next(error);
        },
      ),
    );
  }

  Future<bool> _refreshToken() async {
    try {
      // 实现 Token 刷新逻辑
      // final response = await _dio.post('/auth/refresh');
      // _token = response.data['token'];
      return true;
    } catch (e) {
      _token = null;
      return false;
    }
  }

  void setToken(String token) {
    _token = token;
  }

  void clearToken() {
    _token = null;
  }

  Dio get dio => _dio;
}
```

### 9.2 用户服务封装

```dart
// api/services/user_service.dart
import 'package:dio/dio.dart';
import '../api_client.dart';
import '../../models/user.dart';

class UserService {
  final Dio _dio = ApiClient().dio;

  Future<User> getUser(int id) async {
    final response = await _dio.get('/users/$id');
    return User.fromJson(response.data);
  }

  Future<List<User>> getUsers({int page = 1, int limit = 10}) async {
    final response = await _dio.get('/users', queryParameters: {
      'page': page,
      'limit': limit,
    });
    return (response.data as List)
        .map((json) => User.fromJson(json))
        .toList();
  }

  Future<User> createUser(Map<String, dynamic> data) async {
    final response = await _dio.post('/users', data: data);
    return User.fromJson(response.data);
  }

  Future<User> updateUser(int id, Map<String, dynamic> data) async {
    final response = await _dio.put('/users/$id', data: data);
    return User.fromJson(response.data);
  }

  Future<void> deleteUser(int id) async {
    await _dio.delete('/users/$id');
  }

  Future<String> uploadAvatar(int userId, String filePath) async {
    final formData = FormData.fromMap({
      'avatar': await MultipartFile.fromFile(filePath),
    });
    
    final response = await _dio.post(
      '/users/$userId/avatar',
      data: formData,
    );
    
    return response.data['avatarUrl'];
  }
}
```

### 9.3 带重试机制的请求

```dart
// utils/retry_helper.dart
import 'package:dio/dio.dart';

class RetryHelper {
  static Future<T> retry<T>(
    Future<T> Function() operation, {
    int maxAttempts = 3,
    Duration delay = const Duration(seconds: 1),
    bool Function(Exception)? shouldRetry,
  }) async {
    int attempts = 0;
    
    while (true) {
      try {
        attempts++;
        return await operation();
      } on Exception catch (e) {
        if (attempts >= maxAttempts) {
          rethrow;
        }
        
        if (shouldRetry != null && !shouldRetry(e)) {
          rethrow;
        }
        
        await Future.delayed(delay * attempts);
      }
    }
  }
}

// 使用
Future<void> fetchWithRetry() async {
  final response = await RetryHelper.retry(
    () => dio.get('/unstable-endpoint'),
    maxAttempts: 3,
    delay: Duration(seconds: 2),
    shouldRetry: (e) {
      if (e is DioException) {
        return e.type == DioExceptionType.connectionTimeout ||
               e.response?.statusCode == 503;
      }
      return false;
    },
  );
  
  print(response.data);
}
```

### 9.4 并发请求

```dart
Future<void> fetchMultiple() async {
  // 同时发起多个请求
  final responses = await Future.wait([
    dio.get('/users'),
    dio.get('/posts'),
    dio.get('/comments'),
  ]);
  
  final users = responses[0].data;
  final posts = responses[1].data;
  final comments = responses[2].data;
  
  print('Users: ${users.length}');
  print('Posts: ${posts.length}');
  print('Comments: ${comments.length}');
}

// 带错误处理的并发请求
Future<void> fetchMultipleWithErrorHandling() async {
  final results = await Future.wait([
    dio.get('/users').then((r) => r.data).catchError((e) => null),
    dio.get('/posts').then((r) => r.data).catchError((e) => null),
    dio.get('/comments').then((r) => r.data).catchError((e) => null),
  ]);
  
  // 处理结果
  for (var i = 0; i < results.length; i++) {
    if (results[i] != null) {
      print('Request $i succeeded');
    } else {
      print('Request $i failed');
    }
  }
}
```

---

## 第10章 高级功能

### 10.1 Cookie 管理

```dart
import 'package:dio_cookie_manager/dio_cookie_manager.dart';
import 'package:cookie_jar/cookie_jar.dart';

final dio = Dio();

// 使用内存 CookieJar
final cookieJar = CookieJar();
dio.interceptors.add(CookieManager(cookieJar));

// 使用持久化 CookieJar
final persistCookieJar = PersistCookieJar(
  storage: FileStorage('./cookies'),
);
dio.interceptors.add(CookieManager(persistCookieJar));

// 获取 Cookie
final cookies = await cookieJar.loadForRequest(Uri.parse('https://example.com'));
print(cookies);
```

### 10.2 HTTPS 证书验证

```dart
// 忽略证书验证（仅用于开发测试）
(dio.httpClientAdapter as IOHttpClientAdapter).onHttpClientCreate =
    (client) {
  client.badCertificateCallback =
      (X509Certificate cert, String host, int port) => true;
  return client;
};

// 使用自定义证书
(dio.httpClientAdapter as IOHttpClientAdapter).onHttpClientCreate =
    (client) {
  final cert = File('./certificate.pem').readAsBytesSync();
  final securityContext = SecurityContext();
  securityContext.setTrustedCertificatesBytes(cert);
  
  return HttpClient(context: securityContext);
};
```

### 10.3 代理设置

```dart
(dio.httpClientAdapter as IOHttpClientAdapter).onHttpClientCreate =
    (client) {
  client.findProxy = (uri) {
    // 使用系统代理
    // return 'PROXY localhost:8888';
    
    // 直连
    // return 'DIRECT';
    
    // 根据 URL 选择代理
    if (uri.host == 'api.example.com') {
      return 'PROXY proxy.example.com:8080';
    }
    return 'DIRECT';
  };
  
  return client;
};
```

### 10.4 HTTP/2 支持

```dart
import 'package:dio_http2_adapter/dio_http2_adapter.dart';

final dio = Dio();
dio.httpClientAdapter = Http2Adapter(
  ConnectionManager(
    idleTimeout: Duration(seconds: 10),
    /// 忽略证书验证
    onClientCreate: (uri, config) {
      config.onBadCertificate = (certificate, host, port) => true;
    },
  ),
);
```

---

## 第11章 最佳实践

### 11.1 项目结构建议

```
lib/
├── main.dart
├── api/
│   ├── api_client.dart
│   ├── interceptors/
│   │   ├── auth_interceptor.dart
│   │   ├── error_interceptor.dart
│   │   └── log_interceptor.dart
│   └── services/
│       ├── user_service.dart
│       ├── post_service.dart
│       └── auth_service.dart
├── models/
│   ├── user.dart
│   ├── post.dart
│   └── api_response.dart
├── repositories/
│   ├── user_repository.dart
│   └── post_repository.dart
└── utils/
    ├── exceptions.dart
    └── constants.dart
```

### 11.2 DO 和 DON'T

#### ✅ DO

- 使用单例模式管理 Dio 实例
- 使用拦截器统一处理认证和错误
- 使用 CancelToken 支持请求取消
- 使用 FormData 上传文件
- 使用 BackgroundTransformer 解析 JSON
- 封装服务层，不直接在 UI 层调用 Dio

#### ❌ DON'T

- 在每个页面创建新的 Dio 实例
- 在拦截器中执行耗时操作
- 忽略错误处理
- 在生产环境忽略 HTTPS 证书验证
- 在 UI 层直接处理 DioException

---

## 附录：dio API 速查表

### HTTP 方法

| 方法 | 说明 |
|------|------|
| dio.get() | GET 请求 |
| dio.post() | POST 请求 |
| dio.put() | PUT 请求 |
| dio.delete() | DELETE 请求 |
| dio.patch() | PATCH 请求 |
| dio.head() | HEAD 请求 |
| dio.download() | 下载文件 |
| dio.request() | 通用请求 |

### ResponseType

| 类型 | 说明 |
|------|------|
| json | JSON 格式（默认） |
| stream | 流格式 |
| plain | 纯文本 |
| bytes | 字节数组 |

### DioExceptionType

| 类型 | 说明 |
|------|------|
| connectionTimeout | 连接超时 |
| sendTimeout | 发送超时 |
| receiveTimeout | 接收超时 |
| badCertificate | 证书错误 |
| badResponse | 错误响应 |
| cancel | 请求取消 |
| connectionError | 连接错误 |
| unknown | 未知错误 |

---

## 结语

dio 5.9.1 是 Flutter 生态中最强大的 HTTP 客户端之一。通过本书的学习，你应该已经掌握了：

1. dio 的核心概念和基础用法
2. 各种 HTTP 请求方法的使用
3. 拦截器的配置和自定义
4. 文件上传和下载
5. 请求取消和错误处理
6. 转换器和高级功能

希望本书能够帮助你在 Flutter 开发中更好地使用 dio，构建出网络功能强大的应用程序。

---

**版本信息**

- dio 版本：5.9.1
- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
