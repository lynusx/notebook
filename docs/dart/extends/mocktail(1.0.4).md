# Mocktail 库详解

## 目录

1. [概述](#概述)
2. [核心概念](#核心概念)
3. [安装与配置](#安装与配置)
4. [Mock 类详解](#mock-类详解)
5. [Fake 类详解](#fake-类详解)
6. [when 存根详解](#when-存根详解)
7. [verify 验证详解](#verify-验证详解)
8. [参数匹配器](#参数匹配器)
9. [参数捕获](#参数捕获)
10. [重置与清理](#重置与清理)
11. [实战应用示例](#实战应用示例)
12. [附录](#附录)

---

## 概述

Mocktail 是一个用于 Dart 和 Flutter 的现代化 Mock 库，它简化了单元测试中的依赖模拟，原生支持 Dart 的 null safety，并且**无需代码生成**。

### 与 Mockito 的对比

| 特性        | Mocktail                    | Mockito                 |
| ----------- | --------------------------- | ----------------------- |
| Null Safety | ✅ 原生支持                 | ⚠️ 需要代码生成         |
| 代码生成    | ❌ 不需要                   | ✅ 需要 build_runner    |
| 语法风格    | 闭包风格 `when(() => ...)`  | 直接调用 `when(...)`    |
| 参数匹配    | `any()` / `any(named: 'x')` | `any` / `anyNamed('x')` |
| 学习曲线    | 平缓                        | 较陡峭                  |
| 维护成本    | 低                          | 中等                    |

### 核心优势

1. **零配置** - 无需 build_runner 或代码生成
2. **类型安全** - 完整的 null safety 支持
3. **简洁语法** - 使用闭包捕获调用
4. **灵活匹配** - 强大的参数匹配器系统
5. **易于调试** - 清晰的错误信息

---

## 核心概念

### Mock 测试基本原理

```
┌─────────────────────────────────────────────────────────────┐
│                      被测试的类 (SUT)                         │
│                          (UserService)                      │
│                              │                              │
│                              ▼                              │
│                    ┌──────────────────┐                     │
│                    │     依赖接口      │                     │
│                    │ (UserRepository) │                     │
│                    └────────┬─────────┘                     │
│                             │                                │
│              ┌──────────────┼──────────────┐                │
│              │              │              │                │
│              ▼              ▼              ▼                │
│        ┌─────────┐   ┌──────────┐   ┌──────────┐          │
│        │ 真实实现 │   │  Mock实现 │   │  Fake实现 │          │
│        │(数据库)  │   │(存根/验证)│   │(简单实现) │          │
│        └─────────┘   └──────────┘   └──────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### Mocktail 工作流程

```dart
// 1. 创建 Mock
final mockRepository = MockUserRepository();

// 2. 设置存根（Stubbing）
when(() => mockRepository.getUser(any()))
    .thenReturn(User(id: '1', name: 'Test'));

// 3. 执行被测试代码
final userService = UserService(mockRepository);
final user = userService.getUser('1');

// 4. 验证交互
verify(() => mockRepository.getUser('1')).called(1);
```

---

## 安装与配置

### 添加依赖

```yaml
# pubspec.yaml
dev_dependencies:
  mocktail: ^1.0.4
  test: ^1.24.0 # 或 flutter_test
```

### 基础导入

```dart
import 'package:mocktail/mocktail.dart';
import 'package:test/test.dart';  // 或 flutter_test/flutter_test.dart
```

### 项目结构建议

```
lib/
├── repositories/
│   └── user_repository.dart
├── services/
│   └── user_service.dart
└── models/
    └── user.dart
test/
├── mocks/                    # Mock 类定义
│   └── mock_repositories.dart
├── repositories/
│   └── user_repository_test.dart
└── services/
    └── user_service_test.dart
```

---

## Mock 类详解

`Mock` 是 Mocktail 的核心类，用于创建模拟对象。

### 基本用法

```dart
// 定义要模拟的接口
abstract class UserRepository {
  User? getUser(String id);
  Future<List<User>> getAllUsers();
  Future<void> saveUser(User user);
  void deleteUser(String id);
}

// 创建 Mock 类
class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });
}
```

### 重要特性

| 特性       | 说明                    |
| ---------- | ----------------------- |
| 默认返回值 | 未存根的方法返回 `null` |
| 类型安全   | 编译时类型检查          |
| 可验证     | 支持 `verify` 验证调用  |
| 可存根     | 支持 `when` 设置返回值  |

### 存根基础方法

```dart
void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  test('基本的存根和调用', () {
    // 设置存根
    when(() => mockRepository.getUser('1'))
        .thenReturn(User(id: '1', name: 'Alice'));

    // 调用方法
    final user = mockRepository.getUser('1');

    // 验证结果
    expect(user?.name, equals('Alice'));
  });
}
```

### 处理非空返回类型

```dart
class UserService {
  final UserRepository repository;
  UserService(this.repository);

  Future<User> fetchUser(String id) async {
    final user = await repository.getUser(id);
    if (user == null) {
      throw Exception('User not found');
    }
    return user;
  }
}

void main() {
  late MockUserRepository mockRepository;
  late UserService userService;

  setUp(() {
    mockRepository = MockUserRepository();
    userService = UserService(mockRepository);
  });

  test('处理非空返回类型', () async {
    // 必须存根异步方法，否则会返回 null 导致类型错误
    when(() => mockRepository.getUser(any()))
        .thenAnswer((_) async => User(id: '1', name: 'Bob'));

    final user = await userService.fetchUser('1');
    expect(user.name, equals('Bob'));
  });
}
```

---

## Fake 类详解

`Fake` 是一个轻量级的替代方案，用于创建具有特定行为的简单实现。

### Mock vs Fake

| 特性     | Mock               | Fake         |
| -------- | ------------------ | ------------ |
| 用途     | 验证交互、设置存根 | 提供简单实现 |
| 验证     | ✅ 支持            | ❌ 不支持    |
| 存根     | ✅ 支持            | ❌ 不支持    |
| 实现方式 | 运行时动态         | 手动覆盖     |
| 性能     | 较低               | 较高         |

### Fake 基本用法

```dart
// 使用 Fake 创建简单实现
class FakeUserRepository extends Fake implements UserRepository {
  final Map<String, User> _users = {};

  @override
  User? getUser(String id) => _users[id];

  @override
  Future<void> saveUser(User user) async {
    _users[user.id] = user;
  }

  @override
  Future<List<User>> getAllUsers() async => _users.values.toList();

  @override
  void deleteUser(String id) => _users.remove(id);
}

void main() {
  test('使用 Fake 进行集成测试', () async {
    final repository = FakeUserRepository();
    final service = UserService(repository);

    // 不需要存根，Fake 有真实实现
    await repository.saveUser(User(id: '1', name: 'Alice'));

    final user = await service.fetchUser('1');
    expect(user.name, equals('Alice'));
  });
}
```

### 为参数匹配注册 Fake

```dart
// 当使用 any() 匹配自定义类型时，需要注册 fallback
class Food {
  final String name;
  Food(this.name);
}

class Cat {
  bool likes(Food food) => food.name == 'fish';
}

class MockCat extends Mock implements Cat {}

// 创建 Fake 类
class FakeFood extends Fake implements Food {}

void main() {
  setUpAll(() {
    // 注册 fallback 值，只需要调用一次
    registerFallbackValue(FakeFood());
  });

  test('使用 any() 匹配自定义类型', () {
    final cat = MockCat();

    when(() => cat.likes(any())).thenReturn(true);

    expect(cat.likes(Food('fish')), isTrue);
    expect(cat.likes(Food('meat')), isTrue);
  });
}
```

---

## when 存根详解

`when` 函数用于设置 Mock 对象的方法返回值。

### 基本存根方法

```dart
void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  test('thenReturn - 返回固定值', () {
    when(() => mockRepository.getUser('1'))
        .thenReturn(User(id: '1', name: 'Alice'));

    final user1 = mockRepository.getUser('1');
    final user2 = mockRepository.getUser('1');

    expect(user1?.name, equals('Alice'));
    expect(user2?.name, equals('Alice')); // 多次调用返回相同值
  });

  test('thenAnswer - 动态返回值', () async {
    var counter = 0;

    when(() => mockRepository.getAllUsers())
        .thenAnswer((_) async {
      counter++;
      return [User(id: '$counter', name: 'User $counter')];
    });

    final users1 = await mockRepository.getAllUsers();
    final users2 = await mockRepository.getAllUsers();

    expect(users1.first.name, equals('User 1'));
    expect(users2.first.name, equals('User 2'));
  });

  test('thenThrow - 抛出异常', () {
    when(() => mockRepository.getUser('999'))
        .thenThrow(Exception('User not found'));

    expect(
      () => mockRepository.getUser('999'),
      throwsA(isA<Exception>()),
    );
  });
}
```

### 连续返回值

```dart
test('thenReturnInOrder - 依次返回不同值', () {
  when(() => mockRepository.getUser(any()))
      .thenReturnInOrder([
        User(id: '1', name: 'First'),
        User(id: '1', name: 'Second'),
        User(id: '1', name: 'Third'),
      ]);

  expect(mockRepository.getUser('1')?.name, equals('First'));
  expect(mockRepository.getUser('1')?.name, equals('Second'));
  expect(mockRepository.getUser('1')?.name, equals('Third'));
  expect(mockRepository.getUser('1')?.name, equals('Third')); // 后续返回最后一个值
});
```

### 存根 Getter

```dart
abstract class Config {
  String get apiUrl;
  int get timeout;
}

class MockConfig extends Mock implements Config {}

void main() {
  test('存根 Getter', () {
    final config = MockConfig();

    when(() => config.apiUrl).thenReturn('https://api.example.com');
    when(() => config.timeout).thenReturn(5000);

    expect(config.apiUrl, equals('https://api.example.com'));
    expect(config.timeout, equals(5000));
  });
}
```

### 存根 Setter

```dart
abstract class Settings {
  String get theme;
  set theme(String value);
}

class MockSettings extends Mock implements Settings {}

void main() {
  test('存根和验证 Setter', () {
    final settings = MockSettings();

    // Setter 通常不需要存根，直接验证即可
    settings.theme = 'dark';

    verify(() => settings.theme = 'dark').called(1);
  });
}
```

### 异步存根

```dart
test('异步方法存根', () async {
  // thenAnswer 用于异步方法
  when(() => mockRepository.getAllUsers())
      .thenAnswer((_) async => [
            User(id: '1', name: 'Alice'),
            User(id: '2', name: 'Bob'),
          ]);

  final users = await mockRepository.getAllUsers();
  expect(users.length, equals(2));
});

test('延迟响应', () async {
  when(() => mockRepository.getUser(any()))
      .thenAnswer((_) async {
    await Future.delayed(Duration(seconds: 1));
    return User(id: '1', name: 'Alice');
  });

  final stopwatch = Stopwatch()..start();
  await mockRepository.getUser('1');
  stopwatch.stop();

  expect(stopwatch.elapsed.inSeconds, greaterThanOrEqualTo(1));
});
```

---

## verify 验证详解

`verify` 用于验证 Mock 对象的方法是否被正确调用。

### 基本验证

```dart
void main() {
  late MockUserRepository mockRepository;
  late UserService userService;

  setUp(() {
    mockRepository = MockUserRepository();
    userService = UserService(mockRepository);
  });

  test('验证方法被调用', () {
    when(() => mockRepository.getUser(any()))
        .thenReturn(User(id: '1', name: 'Alice'));

    userService.getUser('1');

    // 验证 getUser 被调用
    verify(() => mockRepository.getUser('1')).called(1);
  });

  test('验证调用次数', () {
    when(() => mockRepository.getUser(any()))
        .thenReturn(User(id: '1', name: 'Alice'));

    userService.getUser('1');
    userService.getUser('1');
    userService.getUser('1');

    verify(() => mockRepository.getUser('1')).called(3);
  });

  test('验证从未被调用', () {
    // 不调用任何方法

    verifyNever(() => mockRepository.getUser(any()));
  });
}
```

### 调用次数验证

```dart
test('灵活的调用次数验证', () {
  when(() => mockRepository.getUser(any()))
      .thenReturn(User(id: '1', name: 'Alice'));

  // 调用 3 次
  userService.getUser('1');
  userService.getUser('1');
  userService.getUser('1');

  // 至少调用 2 次
  verify(() => mockRepository.getUser('1')).called(greaterThanOrEqualTo(2));

  // 最多调用 5 次
  verify(() => mockRepository.getUser('1')).called(lessThanOrEqualTo(5));

  // 调用次数在 2 到 5 之间
  verify(() => mockRepository.getUser('1')).called(inInclusiveRange(2, 5));
});
```

### 零交互验证

```dart
test('验证没有任何交互', () {
  // 创建 Mock 但不使用
  final mock = MockUserRepository();

  verifyZeroInteractions(mock);
});

test('验证没有更多交互', () {
  when(() => mockRepository.getUser('1'))
      .thenReturn(User(id: '1', name: 'Alice'));

  userService.getUser('1');

  verify(() => mockRepository.getUser('1')).called(1);
  verifyNoMoreInteractions(mockRepository);
});
```

### 有序验证

```dart
test('验证调用顺序', () {
  final mockCat = MockCat();

  mockCat.eat();
  mockCat.sleep();
  mockCat.wakeUp();

  verifyInOrder([
    () => mockCat.eat(),
    () => mockCat.sleep(),
    () => mockCat.wakeUp(),
  ]);
});
```

### 等待异步调用

```dart
test('等待异步调用完成', () async {
  when(() => mockRepository.saveUser(any()))
      .thenAnswer((_) async {});

  await userService.createUser('Alice');

  // 使用 untilCalled 等待异步方法
  await untilCalled(() => mockRepository.saveUser(any()));

  verify(() => mockRepository.saveUser(any())).called(1);
});
```

---

## 参数匹配器

参数匹配器用于在存根和验证时匹配方法参数。

### 内置匹配器

| 匹配器                 | 用途               | 示例                                     |
| ---------------------- | ------------------ | ---------------------------------------- |
| `any()`                | 匹配任何值         | `when(() => obj.method(any()))`          |
| `any(that: matcher)`   | 匹配符合条件的值   | `any(that: equals('test'))`              |
| `any(named: 'x')`      | 匹配命名参数       | `any(named: 'id')`                       |
| `captureAny()`         | 捕获参数值         | `verify(() => obj.method(captureAny()))` |
| `captureThat(matcher)` | 捕获符合条件的参数 | `captureThat(startsWith('a'))`           |

### any() 使用详解

```dart
void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  test('any() 匹配任何值', () {
    when(() => mockRepository.getUser(any()))
        .thenReturn(User(id: '1', name: 'Test'));

    // 任何 id 都会返回相同的用户
    expect(mockRepository.getUser('1')?.name, equals('Test'));
    expect(mockRepository.getUser('2')?.name, equals('Test'));
    expect(mockRepository.getUser('abc')?.name, equals('Test'));
  });

  test('any(that: ...) 使用匹配器', () {
    when(() => mockRepository.getUser(any(that: startsWith('admin'))))
        .thenReturn(User(id: 'admin', name: 'Admin User'));

    when(() => mockRepository.getUser(any(that: isNot(startsWith('admin')))))
        .thenReturn(User(id: 'user', name: 'Regular User'));

    expect(mockRepository.getUser('admin_1')?.name, equals('Admin User'));
    expect(mockRepository.getUser('user_1')?.name, equals('Regular User'));
  });
}
```

### 命名参数匹配

```dart
abstract class SearchService {
  List<String> search({
    required String query,
    int? limit,
    bool caseSensitive = false,
  });
}

class MockSearchService extends Mock implements SearchService {}

void main() {
  test('匹配命名参数', () {
    final service = MockSearchService();

    // 匹配特定命名参数
    when(() => service.search(
      query: any(named: 'query'),
      limit: any(named: 'limit'),
    )).thenReturn(['result1', 'result2']);

    // 匹配带条件的命名参数
    when(() => service.search(
      query: any(named: 'query', that: contains('test')),
      caseSensitive: any(named: 'caseSensitive'),
    )).thenReturn(['test_result']);

    expect(
      service.search(query: 'hello', limit: 10),
      equals(['result1', 'result2']),
    );

    expect(
      service.search(query: 'this is a test', caseSensitive: true),
      equals(['test_result']),
    );
  });
}
```

### 自定义匹配器

```dart
import 'package:matcher/matcher.dart';

// 自定义 Matcher
class IsValidEmail extends Matcher {
  @override
  bool matches(dynamic item, Map matchState) {
    if (item is! String) return false;
    return RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(item);
  }

  @override
  Description describe(Description description) {
    return description.add('a valid email address');
  }
}

Matcher isValidEmail() => IsValidEmail();

void main() {
  late MockUserRepository mockRepository;

  test('使用自定义匹配器', () {
    when(() => mockRepository.findByEmail(any(that: isValidEmail())))
        .thenReturn(User(id: '1', name: 'Test User'));

    expect(
      mockRepository.findByEmail('test@example.com')?.name,
      equals('Test User'),
    );

    // 不匹配无效邮箱
    expect(mockRepository.findByEmail('invalid'), isNull);
  });
}
```

---

## 参数捕获

参数捕获用于获取方法调用时传入的实际参数值。

### 基本捕获

```dart
void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  test('捕获单个参数', () {
    when(() => mockRepository.saveUser(any()))
        .thenAnswer((_) async {});

    final user = User(id: '1', name: 'Alice');
    mockRepository.saveUser(user);

    final captured = verify(() => mockRepository.saveUser(captureAny())).captured;

    expect(captured.length, equals(1));
    expect(captured.first, equals(user));
  });

  test('捕获多个调用', () {
    when(() => mockRepository.saveUser(any()))
        .thenAnswer((_) async {});

    mockRepository.saveUser(User(id: '1', name: 'Alice'));
    mockRepository.saveUser(User(id: '2', name: 'Bob'));
    mockRepository.saveUser(User(id: '3', name: 'Charlie'));

    final captured = verify(() => mockRepository.saveUser(captureAny())).captured;

    expect(captured.length, equals(3));
    expect(captured[0].name, equals('Alice'));
    expect(captured[1].name, equals('Bob'));
    expect(captured[2].name, equals('Charlie'));
  });
}
```

### 条件捕获

```dart
test('捕获符合条件的参数', () {
  when(() => mockRepository.saveUser(any()))
      .thenAnswer((_) async {});

  mockRepository.saveUser(User(id: '1', name: 'Alice'));
  mockRepository.saveUser(User(id: '2', name: 'Bob'));
  mockRepository.saveUser(User(id: '3', name: 'Charlie'));

  // 只捕获名字以 'A' 开头的用户
  final captured = verify(
    () => mockRepository.saveUser(
      captureAny(that: predicate<User>((u) => u.name.startsWith('A'))),
    ),
  ).captured;

  expect(captured.length, equals(1));
  expect(captured.first.name, equals('Alice'));
});
```

### 捕获多个参数

```dart
abstract class EmailService {
  void sendEmail(String to, String subject, String body);
}

class MockEmailService extends Mock implements EmailService {}

void main() {
  test('捕获多个参数', () {
    final emailService = MockEmailService();

    emailService.sendEmail(
      'user@example.com',
      'Welcome',
      'Welcome to our app!',
    );

    final capturedTo = verify(
      () => emailService.sendEmail(
        captureAny(),
        any(),
        any(),
      ),
    ).captured;

    expect(capturedTo.first, equals('user@example.com'));
  });
}
```

---

## 重置与清理

### 重置方法

```dart
void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  tearDown(() {
    // 清理所有存根和交互记录
    reset(mockRepository);
  });

  test('重置存根', () {
    when(() => mockRepository.getUser('1'))
        .thenReturn(User(id: '1', name: 'Alice'));

    expect(mockRepository.getUser('1')?.name, equals('Alice'));

    // 重置后存根消失
    reset(mockRepository);
    expect(mockRepository.getUser('1'), isNull);
  });

  test('清除交互记录', () {
    when(() => mockRepository.getUser(any()))
        .thenReturn(User(id: '1', name: 'Test'));

    mockRepository.getUser('1');

    // 清除交互记录但保留存根
    clearInteractions(mockRepository);

    // 验证记录已被清除
    verifyNever(() => mockRepository.getUser(any()));

    // 存根仍然有效
    expect(mockRepository.getUser('1')?.name, equals('Test'));
  });
}
```

### 全局重置

```dart
void main() {
  setUpAll(() {
    // 注册 fallback 值
    registerFallbackValue(FakeUser());
  });

  tearDown(() {
    // 重置单个 Mock
    reset(mockRepository);
  });

  tearDownAll(() {
    // 重置整个 Mocktail 状态
    resetMocktailState();
  });
}
```

### 调试辅助

```dart
test('打印调用记录', () {
  mockRepository.getUser('1');
  mockRepository.getUser('2');
  mockRepository.saveUser(User(id: '3', name: 'Charlie'));

  // 打印所有调用记录
  logInvocations([mockRepository]);
  // 输出:
  // MockUserRepository.getUser("1")
  // MockUserRepository.getUser("2")
  // MockUserRepository.saveUser(Instance of 'User')
});
```

---

## 实战应用示例

### 示例 1：完整的 BLoC 测试

```dart
// user_bloc.dart
abstract class UserEvent {}
class LoadUser extends UserEvent {
  final String userId;
  LoadUser(this.userId);
}

abstract class UserState {}
class UserInitial extends UserState {}
class UserLoading extends UserState {}
class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}
class UserError extends UserState {
  final String message;
  UserError(this.message);
}

class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository repository;

  UserBloc({required this.repository}) : super(UserInitial()) {
    on<LoadUser>(_onLoadUser);
  }

  Future<void> _onLoadUser(LoadUser event, Emitter<UserState> emit) async {
    emit(UserLoading());
    try {
      final user = await repository.getUser(event.userId);
      if (user != null) {
        emit(UserLoaded(user));
      } else {
        emit(UserError('User not found'));
      }
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }
}

// user_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:mocktail/mocktail.dart';

class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepository;
  late UserBloc userBloc;

  setUp(() {
    mockRepository = MockUserRepository();
    userBloc = UserBloc(repository: mockRepository);
  });

  tearDown(() {
    userBloc.close();
  });

  group('UserBloc', () {
    blocTest<UserBloc, UserState>(
      'emits [UserLoading, UserLoaded] when LoadUser is added and user exists',
      build: () {
        when(() => mockRepository.getUser('1'))
            .thenAnswer((_) async => User(id: '1', name: 'Alice'));
        return userBloc;
      },
      act: (bloc) => bloc.add(LoadUser('1')),
      expect: () => [
        isA<UserLoading>(),
        isA<UserLoaded>().having((s) => (s as UserLoaded).user.name, 'name', 'Alice'),
      ],
      verify: (_) {
        verify(() => mockRepository.getUser('1')).called(1);
      },
    );

    blocTest<UserBloc, UserState>(
      'emits [UserLoading, UserError] when user is not found',
      build: () {
        when(() => mockRepository.getUser('999'))
            .thenAnswer((_) async => null);
        return userBloc;
      },
      act: (bloc) => bloc.add(LoadUser('999')),
      expect: () => [
        isA<UserLoading>(),
        isA<UserError>().having((s) => (s as UserError).message, 'message', 'User not found'),
      ],
    );

    blocTest<UserBloc, UserState>(
      'emits [UserLoading, UserError] when repository throws',
      build: () {
        when(() => mockRepository.getUser(any()))
            .thenThrow(Exception('Network error'));
        return userBloc;
      },
      act: (bloc) => bloc.add(LoadUser('1')),
      expect: () => [
        isA<UserLoading>(),
        isA<UserError>(),
      ],
    );
  });
}
```

### 示例 2：Repository 层测试

```dart
// user_repository_impl.dart
class UserRepositoryImpl implements UserRepository {
  final ApiClient apiClient;
  final CacheManager cache;

  UserRepositoryImpl({
    required this.apiClient,
    required this.cache,
  });

  @override
  Future<User?> getUser(String id) async {
    // 先尝试从缓存获取
    final cached = await cache.get('user_$id');
    if (cached != null) {
      return User.fromJson(cached);
    }

    // 从 API 获取
    try {
      final response = await apiClient.get('/users/$id');
      final user = User.fromJson(response.data);
      await cache.set('user_$id', response.data);
      return user;
    } on NotFoundException {
      return null;
    }
  }

  @override
  Future<List<User>> getAllUsers() async {
    final response = await apiClient.get('/users');
    return (response.data as List)
        .map((json) => User.fromJson(json))
        .toList();
  }
}

// user_repository_impl_test.dart
class MockApiClient extends Mock implements ApiClient {}
class MockCacheManager extends Mock implements CacheManager {}

void main() {
  late MockApiClient mockApiClient;
  late MockCacheManager mockCache;
  late UserRepositoryImpl repository;

  setUp(() {
    mockApiClient = MockApiClient();
    mockCache = MockCacheManager();
    repository = UserRepositoryImpl(
      apiClient: mockApiClient,
      cache: mockCache,
    );
  });

  group('getUser', () {
    test('returns cached user when available', () async {
      final cachedData = {'id': '1', 'name': 'Cached User'};

      when(() => mockCache.get('user_1'))
          .thenAnswer((_) async => cachedData);

      final user = await repository.getUser('1');

      expect(user?.name, equals('Cached User'));
      verify(() => mockCache.get('user_1')).called(1);
      verifyNever(() => mockApiClient.get(any()));
    });

    test('fetches from API when cache miss', () async {
      final apiResponse = {'id': '1', 'name': 'API User'};

      when(() => mockCache.get('user_1')).thenAnswer((_) async => null);
      when(() => mockApiClient.get('/users/1'))
          .thenAnswer((_) async => Response(data: apiResponse));
      when(() => mockCache.set('user_1', any()))
          .thenAnswer((_) async {});

      final user = await repository.getUser('1');

      expect(user?.name, equals('API User'));
      verifyInOrder([
        () => mockCache.get('user_1'),
        () => mockApiClient.get('/users/1'),
        () => mockCache.set('user_1', apiResponse),
      ]);
    });

    test('returns null when user not found', () async {
      when(() => mockCache.get('user_999')).thenAnswer((_) async => null);
      when(() => mockApiClient.get('/users/999'))
          .thenThrow(NotFoundException());

      final user = await repository.getUser('999');

      expect(user, isNull);
    });
  });

  group('getAllUsers', () {
    test('returns list of users', () async {
      final apiResponse = [
        {'id': '1', 'name': 'User 1'},
        {'id': '2', 'name': 'User 2'},
      ];

      when(() => mockApiClient.get('/users'))
          .thenAnswer((_) async => Response(data: apiResponse));

      final users = await repository.getAllUsers();

      expect(users.length, equals(2));
      expect(users[0].name, equals('User 1'));
      expect(users[1].name, equals('User 2'));
    });
  });
}
```

### 示例 3：HTTP 客户端测试

```dart
// api_client.dart
class ApiClient {
  final http.Client httpClient;
  final String baseUrl;

  ApiClient({
    required this.httpClient,
    required this.baseUrl,
  });

  Future<Response> get(String path) async {
    final response = await httpClient.get(Uri.parse('$baseUrl$path'));
    return _handleResponse(response);
  }

  Future<Response> post(String path, {Map<String, dynamic>? body}) async {
    final response = await httpClient.post(
      Uri.parse('$baseUrl$path'),
      body: jsonEncode(body),
      headers: {'Content-Type': 'application/json'},
    );
    return _handleResponse(response);
  }

  Response _handleResponse(http.Response response) {
    if (response.statusCode == 200) {
      return Response(data: jsonDecode(response.body));
    } else if (response.statusCode == 404) {
      throw NotFoundException();
    } else {
      throw ApiException('Request failed: ${response.statusCode}');
    }
  }
}

// api_client_test.dart
import 'package:http/http.dart' as http;

class MockHttpClient extends Mock implements http.Client {}

void main() {
  late MockHttpClient mockHttpClient;
  late ApiClient apiClient;

  setUp(() {
    mockHttpClient = MockHttpClient();
    apiClient = ApiClient(
      httpClient: mockHttpClient,
      baseUrl: 'https://api.example.com',
    );
  });

  group('GET requests', () {
    test('successful GET request', () async {
      when(() => mockHttpClient.get(Uri.parse('https://api.example.com/users')))
          .thenAnswer((_) async => http.Response(
                jsonEncode([{'id': '1', 'name': 'Alice'}]),
                200,
              ));

      final response = await apiClient.get('/users');

      expect(response.data, isA<List>());
      expect(response.data[0]['name'], equals('Alice'));
    });

    test('404 response throws NotFoundException', () async {
      when(() => mockHttpClient.get(any()))
          .thenAnswer((_) async => http.Response('Not Found', 404));

      expect(
        () => apiClient.get('/users/999'),
        throwsA(isA<NotFoundException>()),
      );
    });
  });

  group('POST requests', () {
    test('successful POST request', () async {
      final requestBody = {'name': 'New User'};

      when(() => mockHttpClient.post(
            Uri.parse('https://api.example.com/users'),
            body: jsonEncode(requestBody),
            headers: {'Content-Type': 'application/json'},
          )).thenAnswer((_) async => http.Response(
                jsonEncode({'id': '1', 'name': 'New User'}),
                200,
              ));

      final response = await apiClient.post('/users', body: requestBody);

      expect(response.data['id'], equals('1'));
    });

    test('captures request body', () async {
      when(() => mockHttpClient.post(
            any(),
            body: captureAny(named: 'body'),
            headers: any(named: 'headers'),
          )).thenAnswer((_) async => http.Response('{}', 200));

      await apiClient.post('/users', body: {'name': 'Test'});

      final captured = verify(() => mockHttpClient.post(
            any(),
            body: captureAny(named: 'body'),
            headers: any(named: 'headers'),
          )).captured;

      expect(captured.first, equals(jsonEncode({'name': 'Test'})));
    });
  });
}
```

### 示例 4：复杂参数匹配

```dart
// search_service.dart
class SearchCriteria {
  final String query;
  final List<String>? categories;
  final DateTime? fromDate;
  final DateTime? toDate;
  final double? minPrice;
  final double? maxPrice;
  final String? sortBy;
  final bool ascending;

  SearchCriteria({
    required this.query,
    this.categories,
    this.fromDate,
    this.toDate,
    this.minPrice,
    this.maxPrice,
    this.sortBy,
    this.ascending = true,
  });
}

abstract class ProductRepository {
  List<Product> search(SearchCriteria criteria);
}

class MockProductRepository extends Mock implements ProductRepository {}

void main() {
  late MockProductRepository mockRepository;

  setUp(() {
    mockRepository = MockProductRepository();
    registerFallbackValue(SearchCriteria(query: ''));
  });

  test('匹配复杂查询条件', () {
    when(() => mockRepository.search(any(that:
      predicate<SearchCriteria>((c) =>
        c.query.contains('phone') &&
        c.minPrice != null &&
        c.minPrice! > 1000
      ),
    ))).thenReturn([Product(name: 'iPhone', price: 9999)]);

    final results = mockRepository.search(SearchCriteria(
      query: 'smartphone',
      minPrice: 2000,
    ));

    expect(results.first.name, equals('iPhone'));
  });

  test('捕获并验证查询条件', () {
    when(() => mockRepository.search(any()))
        .thenReturn([Product(name: 'Test')]);

    mockRepository.search(SearchCriteria(
      query: 'laptop',
      categories: ['electronics'],
      minPrice: 5000,
      maxPrice: 10000,
    ));

    final captured = verify(
      () => mockRepository.search(captureAny()),
    ).captured;

    final criteria = captured.first as SearchCriteria;
    expect(criteria.query, equals('laptop'));
    expect(criteria.categories, contains('electronics'));
    expect(criteria.minPrice, equals(5000));
  });
}
```

### 示例 5：Stream 测试

```dart
// data_source.dart
abstract class DataSource {
  Stream<List<Item>> watchItems();
  Stream<Item?> watchItem(String id);
}

class MockDataSource extends Mock implements DataSource {}

void main() {
  late MockDataSource mockDataSource;

  setUp(() {
    mockDataSource = MockDataSource();
  });

  test('模拟 Stream 数据', () {
    final items = [
      Item(id: '1', name: 'Item 1'),
      Item(id: '2', name: 'Item 2'),
    ];

    when(() => mockDataSource.watchItems())
        .thenAnswer((_) => Stream.value(items));

    expect(
      mockDataSource.watchItems(),
      emits(items),
    );
  });

  test('模拟 Stream 序列', () {
    when(() => mockDataSource.watchItems())
        .thenAnswer((_) => Stream.fromIterable([
          [Item(id: '1', name: 'First')],
          [Item(id: '1', name: 'First'), Item(id: '2', name: 'Second')],
        ]));

    expect(
      mockDataSource.watchItems(),
      emitsInOrder([
        hasLength(1),
        hasLength(2),
      ]),
    );
  });

  test('模拟错误 Stream', () {
    when(() => mockDataSource.watchItems())
        .thenAnswer((_) => Stream.error(Exception('Database error')));

    expect(
      mockDataSource.watchItems(),
      emitsError(isA<Exception>()),
    );
  });
}
```

### 示例 6：最佳实践总结

```dart
// test_helpers.dart

// 集中管理所有 Mock 类
class MockUserRepository extends Mock implements UserRepository {}
class MockApiClient extends Mock implements ApiClient {}
class MockCacheManager extends Mock implements CacheManager {}
class MockAnalyticsService extends Mock implements AnalyticsService {}

// 集中注册 fallback 值
void registerAllFallbackValues() {
  registerFallbackValue(User(id: '', name: ''));
  registerFallbackValue(SearchCriteria(query: ''));
  registerFallbackValue(<String, dynamic>{});
}

// 共享的 setUp 逻辑
void setUpMocks() {
  registerAllFallbackValues();
}

// 自定义 matchers
Matcher isUserWithName(String name) =>
  isA<User>().having((u) => u.name, 'name', name);

Matcher isUserWithId(String id) =>
  isA<User>().having((u) => u.id, 'id', id);

// 具体测试文件中使用
void main() {
  setUpAll(setUpMocks);

  late MockUserRepository mockRepository;
  late UserService userService;

  setUp(() {
    mockRepository = MockUserRepository();
    userService = UserService(mockRepository);
  });

  tearDown(() {
    reset(mockRepository);
  });

  group('UserService', () {
    test('getUser returns user with correct name', () async {
      // Arrange
      when(() => mockRepository.getUser('1'))
          .thenAnswer((_) async => User(id: '1', name: 'Alice'));

      // Act
      final user = await userService.getUser('1');

      // Assert
      expect(user, isUserWithName('Alice'));
      expect(user, isUserWithId('1'));
      verify(() => mockRepository.getUser('1')).called(1);
    });
  });
}
```

---

## 附录

### A. 版本兼容性

| 版本  | Dart SDK | 说明                              |
| ----- | -------- | --------------------------------- |
| 1.0.4 | >=2.12.0 | 当前稳定版，完整 null safety 支持 |
| 0.3.0 | >=2.12.0 | 早期 null safety 版本             |
| 0.2.0 | >=2.12.0 | 初始 null safety 版本             |

### B. 完整 API 参考

#### 核心类

| 类名                 | 用途         |
| -------------------- | ------------ |
| `Mock`               | 创建模拟对象 |
| `Fake`               | 创建简单实现 |
| `When<T>`            | 存根构建器   |
| `VerificationResult` | 验证结果     |
| `Expectation<T>`     | 期望捕获     |

#### 核心函数

| 函数                             | 用途             |
| -------------------------------- | ---------------- |
| `when(() => ...)`                | 创建存根         |
| `verify(() => ...)`              | 验证调用         |
| `verifyNever(() => ...)`         | 验证从未调用     |
| `verifyInOrder([...])`           | 验证调用顺序     |
| `verifyZeroInteractions(mock)`   | 验证零交互       |
| `verifyNoMoreInteractions(mock)` | 验证无更多交互   |
| `any()`                          | 匹配任何值       |
| `any(that: ...)`                 | 条件匹配         |
| `any(named: 'x')`                | 命名参数匹配     |
| `captureAny()`                   | 捕获参数         |
| `captureThat(...)`               | 条件捕获         |
| `reset(mock)`                    | 重置 Mock        |
| `clearInteractions(mock)`        | 清除交互记录     |
| `resetMocktailState()`           | 重置全局状态     |
| `registerFallbackValue(value)`   | 注册 fallback 值 |
| `untilCalled(() => ...)`         | 等待调用         |
| `logInvocations([mocks])`        | 打印调用日志     |

#### 存根方法

| 方法                       | 用途       |
| -------------------------- | ---------- |
| `thenReturn(value)`        | 返回固定值 |
| `thenAnswer((_) => ...)`   | 动态返回值 |
| `thenThrow(exception)`     | 抛出异常   |
| `thenReturnInOrder([...])` | 依次返回   |

### C. 常见问题

**Q: 为什么需要使用闭包 `when(() => ...)`？**

A: 闭包用于捕获 `TypeError`，当存根非空返回类型的方法时，如果直接调用会抛出类型错误。闭包包装可以安全地捕获这些错误。

**Q: 什么时候需要 `registerFallbackValue`？**

A: 当在 `any()` 或 `captureAny()` 中使用自定义类型参数时需要注册。每个类型只需注册一次。

**Q: Mock 和 Fake 如何选择？**

A:

- 需要验证交互或动态存根 → 使用 Mock
- 只需要简单实现，不需要验证 → 使用 Fake

**Q: 如何处理 Flutter 的 Diagnosticable 类（如 ThemeData）？**

A: 使用 mixin 解决 toString 签名不匹配问题：

```dart
mixin DiagnosticableToStringMixin on Object {
  @override
  String toString({DiagnosticLevel minLevel = DiagnosticLevel.info}) {
    return super.toString();
  }
}

class FakeThemeData extends Fake
    with DiagnosticableToStringMixin
    implements ThemeData {}
```

**Q: 为什么不能存根扩展方法？**

A: 扩展方法是静态分派的，不通过实例调用。应该存根扩展方法内部调用的实例方法。

### D. 最佳实践总结

1. **每个测试独立** - 使用 `setUp` 和 `tearDown` 保持测试独立
2. **验证行为而非实现** - 关注"做了什么"而非"怎么做"
3. **使用 Fake 减少复杂度** - 不需要验证时使用 Fake
4. **集中管理 Mocks** - 将 Mock 类定义在单独文件中
5. **注册 fallback 值** - 在 `setUpAll` 中统一注册
6. **重置状态** - 在 `tearDown` 中重置 Mock
7. **有意义的测试名** - 清晰描述测试场景
8. **Arrange-Act-Assert** - 遵循 AAA 模式组织测试

### E. 参考资源

- [Pub 包页面](https://pub.dev/packages/mocktail)
- [API 文档](https://pub.dev/documentation/mocktail/latest/)
- [GitHub 仓库](https://github.com/felangel/mocktail)
- [Mockito 对比](https://pub.dev/packages/mocktail#why-mocktail)
