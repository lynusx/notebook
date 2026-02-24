# Flutter 项目架构设计指南

> **版本**: Flutter 3.41.0 
> **适用范围**: 中大型 Flutter 项目架构设计 
> **最后更新**: 2024年

---

## 第一章：架构模式概述

### 1.1 为什么需要架构设计

在 Flutter 开发中，良好的架构设计能够带来以下好处：

| 优势 | 说明 |
|------|------|
| **可维护性** | 代码结构清晰，易于理解和修改 |
| **可测试性** | 业务逻辑与 UI 分离，便于单元测试 |
| **可扩展性** | 新增功能时不会影响现有代码 |
| **团队协作** | 多人开发时减少代码冲突 |
| **代码复用** | 业务逻辑可以在不同平台复用 |

### 1.2 常见架构模式对比

| 架构模式 | 复杂度 | 适用场景 | 学习曲线 |
|----------|--------|----------|----------|
| **MVC** | 低 | 小型项目、原型开发 | 低 |
| **MVP** | 中 | 中小型项目 | 中 |
| **MVVM** | 中 | 中大型项目 | 中 |
| **Clean Architecture** | 高 | 大型项目、企业级应用 | 高 |
| **Feature-First** | 中 | 模块化项目 | 中 |

### 1.3 架构选择决策树

```
项目规模评估
    │
    ├── 小型项目 ( < 10 个页面 )
    │       └── 推荐: 简单状态管理 (StatefulWidget + Provider)
    │
    ├── 中型项目 ( 10-30 个页面 )
    │       ├── 需要跨平台复用业务逻辑
    │       │       └── 推荐: MVVM + Repository Pattern
    │       └── 团队熟悉原生开发模式
    │               └── 推荐: MVP
    │
    └── 大型项目 ( > 30 个页面 )
            ├── 多团队协作
            │       └── 推荐: Clean Architecture + Feature-First
            └── 需要高度可测试性
                    └── 推荐: Clean Architecture + TDD
```

> **补充知识**：Flutter 官方推荐使用 **分层架构（Layered Architecture）**，将应用分为 UI 层、Domain 层和 Data 层。这种架构与 Clean Architecture 理念一致，是 Flutter 项目的最佳实践。

---

## 第二章：MVVM 架构详解

### 2.1 MVVM 架构概述

MVVM（Model-View-ViewModel）是一种将业务逻辑与 UI 分离的架构模式，非常适合 Flutter 的响应式编程模型。

```
┌─────────────────────────────────────────────────────────────────┐
│                      MVVM 架构图                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │      View       │◀───────▶│    ViewModel    │               │
│  │   (Widget)      │  数据绑定   │  (ChangeNotifier│               │
│  │                 │         │   /StateNotifier)│               │
│  │ • 显示数据       │         │                 │               │
│  │ • 响应用户输入    │         │ • 业务逻辑       │               │
│  │ • 观察状态变化   │         │ • 状态管理       │               │
│  └─────────────────┘         │ • 数据转换       │               │
│           │                  └────────┬────────┘               │
│           │                           │                        │
│           │                    调用方法/获取数据                 │
│           │                           │                        │
│           │                           ▼                        │
│           │                  ┌─────────────────┐               │
│           │                  │  Repository     │               │
│           │                  │  (数据仓库)      │               │
│           │                  │                 │               │
│           │                  │ • 数据抽象       │               │
│           │                  │ • 数据源协调     │               │
│           │                  └────────┬────────┘               │
│           │                           │                        │
│           └───────────────────────────┘                        │
│                                       │                        │
│                                       ▼                        │
│                              ┌─────────────────┐               │
│                              │  Data Sources   │               │
│                              │  (API/Database) │               │
│                              └─────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 MVVM 核心组件

#### Model（模型层）

```dart
// lib/features/user/data/models/user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String name,
    required String email,
    String? avatarUrl,
    @Default(false) bool isActive,
    DateTime? createdAt,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
}
```

#### ViewModel（视图模型层）

```dart
// lib/features/user/presentation/viewmodels/user_viewmodel.dart
import 'package:flutter/foundation.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../data/models/user_model.dart';
import '../../data/repositories/user_repository.dart';

// ViewModel 状态
@immutable
class UserState {
  final UserModel? user;
  final bool isLoading;
  final String? errorMessage;

  const UserState({
    this.user,
    this.isLoading = false,
    this.errorMessage,
  });

  UserState copyWith({
    UserModel? user,
    bool? isLoading,
    String? errorMessage,
  }) {
    return UserState(
      user: user ?? this.user,
      isLoading: isLoading ?? this.isLoading,
      errorMessage: errorMessage,
    );
  }
}

// ViewModel 使用 StateNotifier
class UserViewModel extends StateNotifier<UserState> {
  final UserRepository _repository;

  UserViewModel(this._repository) : super(const UserState());

  Future<void> loadUser(String userId) async {
    state = state.copyWith(isLoading: true, errorMessage: null);

    try {
      final user = await _repository.getUser(userId);
      state = state.copyWith(user: user, isLoading: false);
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        errorMessage: e.toString(),
      );
    }
  }

  Future<void> updateUser(UserModel user) async {
    state = state.copyWith(isLoading: true);

    try {
      final updatedUser = await _repository.updateUser(user);
      state = state.copyWith(user: updatedUser, isLoading: false);
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        errorMessage: e.toString(),
      );
    }
  }
}

// Provider 定义
final userRepositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository(ref.watch(dioProvider));
});

final userViewModelProvider = StateNotifierProvider.autoDispose
    .family<UserViewModel, UserState, String>((ref, userId) {
  final repository = ref.watch(userRepositoryProvider);
  return UserViewModel(repository);
});
```

#### View（视图层）

```dart
// lib/features/user/presentation/pages/user_profile_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../viewmodels/user_viewmodel.dart';

class UserProfilePage extends ConsumerWidget {
  final String userId;

  const UserProfilePage({super.key, required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(userViewModelProvider(userId));

    return Scaffold(
      appBar: AppBar(title: const Text('用户资料')),
      body: _buildBody(context, ref, state),
    );
  }

  Widget _buildBody(BuildContext context, WidgetRef ref, UserState state) {
    if (state.isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (state.errorMessage != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('错误: ${state.errorMessage}'),
            ElevatedButton(
              onPressed: () => ref
                  .read(userViewModelProvider(state.user!.id).notifier)
                  .loadUser(userId),
              child: const Text('重试'),
            ),
          ],
        ),
      );
    }

    final user = state.user;
    if (user == null) {
      return const Center(child: Text('用户不存在'));
    }

    return Padding(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          CircleAvatar(
            radius: 50,
            backgroundImage: user.avatarUrl != null
                ? NetworkImage(user.avatarUrl!)
                : null,
            child: user.avatarUrl == null
                ? Text(user.name[0].toUpperCase())
                : null,
          ),
          const SizedBox(height: 16),
          Text('姓名: ${user.name}', style: Theme.of(context).textTheme.titleLarge),
          Text('邮箱: ${user.email}'),
          Text('状态: ${user.isActive ? "活跃" : "非活跃"}'),
        ],
      ),
    );
  }
}
```

### 2.3 MVVM 文件夹结构

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/
│   │   ├── api_constants.dart
│   │   └── app_constants.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── dio_client.dart
│   │   └── network_info.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   └── utils/
│       └── extensions.dart
├── features/
│   └── user/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── user_local_datasource.dart
│       │   │   └── user_remote_datasource.dart
│       │   ├── models/
│       │   │   └── user_model.dart
│       │   └── repositories/
│       │       └── user_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── user_entity.dart
│       │   ├── repositories/
│       │   │   └── user_repository.dart
│       │   └── usecases/
│       │       ├── get_user.dart
│       │       └── update_user.dart
│       └── presentation/
│           ├── pages/
│           │   └── user_profile_page.dart
│           ├── viewmodels/
│           │   └── user_viewmodel.dart
│           └── widgets/
│               └── user_avatar.dart
└── shared/
    ├── widgets/
    │   └── common_button.dart
    └── providers/
        └── dio_provider.dart
```

### 2.4 MVVM 优缺点

| 优点 | 缺点 |
|------|------|
| 业务逻辑与 UI 完全分离 | 需要额外的样板代码 |
| 易于单元测试 | 初学者学习曲线较陡 |
| 状态管理清晰 | 小型项目可能过度设计 |
| 支持响应式编程 | 需要理解状态管理库 |
| 代码复用性高 | 调试时需要在多层跳转 |

---

## 第三章：MVP 架构详解

### 3.1 MVP 架构概述

MVP（Model-View-Presenter）是一种经典的架构模式，Presenter 作为 View 和 Model 之间的中介，处理所有业务逻辑。

```
┌─────────────────────────────────────────────────────────────────┐
│                      MVP 架构图                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │      View       │◀───────▶│    Presenter    │               │
│  │   (Widget)      │  接口回调  │                 │               │
│  │                 │         │ • 业务逻辑       │               │
│  │ • 显示数据       │         │ • 协调 View/Model│               │
│  │ • 转发用户事件   │         │ • 状态管理       │               │
│  │ • 接口定义       │         │                 │               │
│  └─────────────────┘         └────────┬────────┘               │
│           │                           │                        │
│           │                    调用方法/获取数据                 │
│           │                           │                        │
│           │                           ▼                        │
│           │                  ┌─────────────────┐               │
│           │                  │      Model      │               │
│           │                  │  (Repository)   │               │
│           │                  │                 │               │
│           │                  │ • 数据实体       │               │
│           │                  │ • 数据访问       │               │
│           │                  └─────────────────┘               │
│           │                                                    │
│           └────────────────────────────────────────────────────┘
│                                                                 │
│  关键特点:                                                       │
│  • View 和 Presenter 通过接口通信                                │
│  • View 不包含任何业务逻辑                                       │
│  • Presenter 不依赖 Flutter 框架                                 │
│  • 易于单元测试 Presenter                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 MVP 核心组件

#### Contract（契约接口）

```dart
// lib/features/login/presentation/contracts/login_contract.dart

// View 接口 - 定义 UI 操作
abstract class LoginView {
  void showLoading();
  void hideLoading();
  void showError(String message);
  void navigateToHome();
  void showValidationError(String field, String message);
}

// Presenter 接口 - 定义业务逻辑操作
abstract class LoginPresenter {
  void attachView(LoginView view);
  void detachView();
  void login(String email, String password);
  void validateEmail(String email);
  void validatePassword(String password);
}
```

#### Presenter 实现

```dart
// lib/features/login/presentation/presenters/login_presenter_impl.dart
import '../contracts/login_contract.dart';
import '../../data/repositories/auth_repository.dart';

class LoginPresenterImpl implements LoginPresenter {
  LoginView? _view;
  final AuthRepository _authRepository;

  LoginPresenterImpl(this._authRepository);

  @override
  void attachView(LoginView view) {
    _view = view;
  }

  @override
  void detachView() {
    _view = null;
  }

  @override
  void login(String email, String password) async {
    if (!_validateInput(email, password)) return;

    _view?.showLoading();

    try {
      final result = await _authRepository.login(email, password);
      _view?.hideLoading();

      if (result.isSuccess) {
        _view?.navigateToHome();
      } else {
        _view?.showError(result.errorMessage ?? '登录失败');
      }
    } catch (e) {
      _view?.hideLoading();
      _view?.showError('网络错误，请稍后重试');
    }
  }

  @override
  void validateEmail(String email) {
    if (email.isEmpty) {
      _view?.showValidationError('email', '邮箱不能为空');
    } else if (!_isValidEmail(email)) {
      _view?.showValidationError('email', '请输入有效的邮箱地址');
    }
  }

  @override
  void validatePassword(String password) {
    if (password.isEmpty) {
      _view?.showValidationError('password', '密码不能为空');
    } else if (password.length < 6) {
      _view?.showValidationError('password', '密码至少6位');
    }
  }

  bool _validateInput(String email, String password) {
    validateEmail(email);
    validatePassword(password);
    return email.isNotEmpty &&
        password.isNotEmpty &&
        _isValidEmail(email) &&
        password.length >= 6;
  }

  bool _isValidEmail(String email) {
    return RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(email);
  }
}
```

#### View 实现

```dart
// lib/features/login/presentation/pages/login_page.dart
import 'package:flutter/material.dart';
import '../contracts/login_contract.dart';
import '../presenters/login_presenter_impl.dart';
import '../../data/repositories/auth_repository_impl.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> implements LoginView {
  late final LoginPresenter _presenter;
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  String? _emailError;
  String? _passwordError;

  @override
  void initState() {
    super.initState();
    _presenter = LoginPresenterImpl(AuthRepositoryImpl());
    _presenter.attachView(this);
  }

  @override
  void dispose() {
    _presenter.detachView();
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('登录')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _emailController,
              decoration: InputDecoration(
                labelText: '邮箱',
                errorText: _emailError,
              ),
              keyboardType: TextInputType.emailAddress,
              onChanged: _presenter.validateEmail,
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: '密码',
                errorText: _passwordError,
              ),
              obscureText: true,
              onChanged: _presenter.validatePassword,
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _isLoading
                    ? null
                    : () => _presenter.login(
                          _emailController.text,
                          _passwordController.text,
                        ),
                child: _isLoading
                    ? const CircularProgressIndicator()
                    : const Text('登录'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  // View 接口实现
  @override
  void showLoading() {
    setState(() => _isLoading = true);
  }

  @override
  void hideLoading() {
    setState(() => _isLoading = false);
  }

  @override
  void showError(String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message)),
    );
  }

  @override
  void navigateToHome() {
    Navigator.of(context).pushReplacementNamed('/home');
  }

  @override
  void showValidationError(String field, String message) {
    setState(() {
      if (field == 'email') {
        _emailError = message;
      } else if (field == 'password') {
        _passwordError = message;
      }
    });
  }
}
```

### 3.3 MVP 文件夹结构

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/
│   ├── errors/
│   ├── network/
│   └── utils/
├── features/
│   └── login/
│       ├── data/
│       │   ├── datasources/
│       │   ├── models/
│       │   └── repositories/
│       │       └── auth_repository_impl.dart
│       ├── domain/
│       │   └── repositories/
│       │       └── auth_repository.dart
│       └── presentation/
│           ├── contracts/
│           │   └── login_contract.dart      # View/Presenter 接口
│           ├── pages/
│           │   └── login_page.dart          # View 实现
│           └── presenters/
│               └── login_presenter_impl.dart # Presenter 实现
└── shared/
    └── widgets/
```

### 3.4 MVP 优缺点

| 优点 | 缺点 |
|------|------|
| Presenter 可独立测试 | 需要编写大量接口 |
| View 和 Presenter 完全解耦 | 代码量较大 |
| 业务逻辑集中管理 | 不适合小型项目 |
| 易于替换 View 实现 | 需要手动管理生命周期 |
| 适合团队协作开发 | 状态管理不如 MVVM 直观 |

---

## 第四章：Clean Architecture 详解

### 4.1 Clean Architecture 概述

Clean Architecture（整洁架构）是一种分层架构模式，通过依赖规则确保代码的内聚性和可测试性。

```
┌─────────────────────────────────────────────────────────────────┐
│                   Clean Architecture 分层                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Presentation Layer                    │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   Pages     │  │  ViewModels │  │   Widgets   │     │   │
│  │  │  (UI)       │  │ (State)     │  │ (Components)│     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  │                                                         │   │
│  │  依赖方向: ▼                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│  ┌───────────────────────────▼─────────────────────────────┐   │
│  │                      Domain Layer                        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │  Entities   │  │  Use Cases  │  │Repositories │     │   │
│  │  │ (Business)  │  │ (Business)  │  │ (Interface) │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  │                                                         │   │
│  │  依赖方向: ▼                                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│  ┌───────────────────────────▼─────────────────────────────┐   │
│  │                       Data Layer                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │Data Sources │  │   Models    │  │Repositories │     │   │
│  │  │ (API/DB)    │  │  (DTO)      │  │(Implementation)    │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  │                                                         │   │
│  │  依赖方向: ▼ (外部框架)                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│  ┌───────────────────────────▼─────────────────────────────┐   │
│  │                    External Layer                        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   Dio/Http  │  │   SQLite    │  │  Firebase   │     │   │
│  │  │   Retrofit  │  │   Hive      │  │   etc.      │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  核心原则:                                                       │
│  • 内层不依赖外层                                                │
│  • 依赖方向始终向内                                              │
│  • 使用接口解耦                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Clean Architecture 核心规则

| 规则 | 说明 |
|------|------|
| **依赖规则** | 内层代码不能依赖外层代码 |
| **抽象原则** | 使用接口/抽象类定义边界 |
| **单一职责** | 每个类只负责一个功能 |
| **开闭原则** | 对扩展开放，对修改关闭 |

### 4.3 Clean Architecture 完整实现

#### Domain Layer（最内层）

```dart
// lib/features/todo/domain/entities/todo_entity.dart
class TodoEntity {
  final String id;
  final String title;
  final String description;
  final bool isCompleted;
  final DateTime createdAt;

  const TodoEntity({
    required this.id,
    required this.title,
    required this.description,
    this.isCompleted = false,
    required this.createdAt,
  });

  TodoEntity copyWith({
    String? id,
    String? title,
    String? description,
    bool? isCompleted,
    DateTime? createdAt,
  }) {
    return TodoEntity(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
    );
  }
}

// lib/features/todo/domain/repositories/todo_repository.dart
abstract class TodoRepository {
  Future<List<TodoEntity>> getTodos();
  Future<TodoEntity> getTodoById(String id);
  Future<TodoEntity> createTodo(String title, String description);
  Future<TodoEntity> updateTodo(TodoEntity todo);
  Future<void> deleteTodo(String id);
  Future<TodoEntity> toggleTodoCompletion(String id);
}

// lib/features/todo/domain/usecases/get_todos.dart
class GetTodosUseCase {
  final TodoRepository repository;

  GetTodosUseCase(this.repository);

  Future<List<TodoEntity>> call() async {
    return await repository.getTodos();
  }
}

// lib/features/todo/domain/usecases/create_todo.dart
class CreateTodoUseCase {
  final TodoRepository repository;

  CreateTodoUseCase(this.repository);

  Future<TodoEntity> call(String title, String description) async {
    if (title.isEmpty) {
      throw ArgumentError('Title cannot be empty');
    }
    return await repository.createTodo(title, description);
  }
}
```

#### Data Layer（外层）

```dart
// lib/features/todo/data/models/todo_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../domain/entities/todo_entity.dart';

part 'todo_model.freezed.dart';
part 'todo_model.g.dart';

@freezed
class TodoModel with _$TodoModel {
  const factory TodoModel({
    required String id,
    required String title,
    required String description,
    @Default(false) bool isCompleted,
    required DateTime createdAt,
  }) = _TodoModel;

  factory TodoModel.fromJson(Map<String, dynamic> json) =>
      _$TodoModelFromJson(json);

  factory TodoModel.fromEntity(TodoEntity entity) => TodoModel(
        id: entity.id,
        title: entity.title,
        description: entity.description,
        isCompleted: entity.isCompleted,
        createdAt: entity.createdAt,
      );
}

extension TodoModelX on TodoModel {
  TodoEntity toEntity() => TodoEntity(
        id: id,
        title: title,
        description: description,
        isCompleted: isCompleted,
        createdAt: createdAt,
      );
}

// lib/features/todo/data/datasources/todo_remote_datasource.dart
abstract class TodoRemoteDataSource {
  Future<List<TodoModel>> getTodos();
  Future<TodoModel> getTodoById(String id);
  Future<TodoModel> createTodo(String title, String description);
  Future<TodoModel> updateTodo(TodoModel todo);
  Future<void> deleteTodo(String id);
}

// lib/features/todo/data/datasources/todo_remote_datasource_impl.dart
class TodoRemoteDataSourceImpl implements TodoRemoteDataSource {
  final Dio dio;

  TodoRemoteDataSourceImpl(this.dio);

  @override
  Future<List<TodoModel>> getTodos() async {
    final response = await dio.get('/todos');
    return (response.data as List)
        .map((json) => TodoModel.fromJson(json))
        .toList();
  }

  @override
  Future<TodoModel> getTodoById(String id) async {
    final response = await dio.get('/todos/$id');
    return TodoModel.fromJson(response.data);
  }

  @override
  Future<TodoModel> createTodo(String title, String description) async {
    final response = await dio.post('/todos', data: {
      'title': title,
      'description': description,
    });
    return TodoModel.fromJson(response.data);
  }

  @override
  Future<TodoModel> updateTodo(TodoModel todo) async {
    final response = await dio.put('/todos/${todo.id}', data: todo.toJson());
    return TodoModel.fromJson(response.data);
  }

  @override
  Future<void> deleteTodo(String id) async {
    await dio.delete('/todos/$id');
  }
}

// lib/features/todo/data/repositories/todo_repository_impl.dart
class TodoRepositoryImpl implements TodoRepository {
  final TodoRemoteDataSource remoteDataSource;
  final NetworkInfo networkInfo;

  TodoRepositoryImpl({
    required this.remoteDataSource,
    required this.networkInfo,
  });

  @override
  Future<List<TodoEntity>> getTodos() async {
    if (await networkInfo.isConnected) {
      final models = await remoteDataSource.getTodos();
      return models.map((m) => m.toEntity()).toList();
    } else {
      throw NetworkException('No internet connection');
    }
  }

  @override
  Future<TodoEntity> getTodoById(String id) async {
    final model = await remoteDataSource.getTodoById(id);
    return model.toEntity();
  }

  @override
  Future<TodoEntity> createTodo(String title, String description) async {
    final model = await remoteDataSource.createTodo(title, description);
    return model.toEntity();
  }

  @override
  Future<TodoEntity> updateTodo(TodoEntity todo) async {
    final model = await remoteDataSource.updateTodo(
      TodoModel.fromEntity(todo),
    );
    return model.toEntity();
  }

  @override
  Future<void> deleteTodo(String id) async {
    await remoteDataSource.deleteTodo(id);
  }

  @override
  Future<TodoEntity> toggleTodoCompletion(String id) async {
    final todo = await getTodoById(id);
    final updated = todo.copyWith(isCompleted: !todo.isCompleted);
    return await updateTodo(updated);
  }
}
```

#### Presentation Layer（最外层）

```dart
// lib/features/todo/presentation/providers/todo_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/entities/todo_entity.dart';
import '../../domain/usecases/get_todos.dart';
import '../../domain/usecases/create_todo.dart';
import '../../data/repositories/todo_repository_impl.dart';
import '../../data/datasources/todo_remote_datasource_impl.dart';
import '../../../../core/network/network_info.dart';
import '../../../../core/network/dio_client.dart';

// 依赖注入
final todoRemoteDataSourceProvider = Provider<TodoRemoteDataSourceImpl>((ref) {
  return TodoRemoteDataSourceImpl(ref.watch(dioClientProvider));
});

final todoRepositoryProvider = Provider<TodoRepositoryImpl>((ref) {
  return TodoRepositoryImpl(
    remoteDataSource: ref.watch(todoRemoteDataSourceProvider),
    networkInfo: ref.watch(networkInfoProvider),
  );
});

final getTodosUseCaseProvider = Provider<GetTodosUseCase>((ref) {
  return GetTodosUseCase(ref.watch(todoRepositoryProvider));
});

final createTodoUseCaseProvider = Provider<CreateTodoUseCase>((ref) {
  return CreateTodoUseCase(ref.watch(todoRepositoryProvider));
});

// 状态管理
class TodoState {
  final List<TodoEntity> todos;
  final bool isLoading;
  final String? errorMessage;

  const TodoState({
    this.todos = const [],
    this.isLoading = false,
    this.errorMessage,
  });

  TodoState copyWith({
    List<TodoEntity>? todos,
    bool? isLoading,
    String? errorMessage,
  }) {
    return TodoState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
      errorMessage: errorMessage,
    );
  }
}

class TodoNotifier extends StateNotifier<TodoState> {
  final GetTodosUseCase _getTodosUseCase;
  final CreateTodoUseCase _createTodoUseCase;

  TodoNotifier({
    required GetTodosUseCase getTodosUseCase,
    required CreateTodoUseCase createTodoUseCase,
  })  : _getTodosUseCase = getTodosUseCase,
        _createTodoUseCase = createTodoUseCase,
        super(const TodoState());

  Future<void> loadTodos() async {
    state = state.copyWith(isLoading: true, errorMessage: null);

    try {
      final todos = await _getTodosUseCase();
      state = state.copyWith(todos: todos, isLoading: false);
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        errorMessage: e.toString(),
      );
    }
  }

  Future<void> addTodo(String title, String description) async {
    try {
      await _createTodoUseCase(title, description);
      await loadTodos(); // 刷新列表
    } catch (e) {
      state = state.copyWith(errorMessage: e.toString());
    }
  }
}

final todoNotifierProvider = StateNotifierProvider<TodoNotifier, TodoState>(
  (ref) => TodoNotifier(
    getTodosUseCase: ref.watch(getTodosUseCaseProvider),
    createTodoUseCase: ref.watch(createTodoUseCaseProvider),
  ),
);
```

### 4.4 Clean Architecture 文件夹结构

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/
│   │   ├── api_constants.dart
│   │   └── app_constants.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── dio_client.dart
│   │   └── network_info.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   └── utils/
│       └── extensions.dart
├── features/
│   └── todo/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── todo_entity.dart
│       │   ├── repositories/
│       │   │   └── todo_repository.dart
│       │   └── usecases/
│       │       ├── get_todos.dart
│       │       ├── create_todo.dart
│       │       ├── update_todo.dart
│       │       └── delete_todo.dart
│       ├── data/
│       │   ├── models/
│       │   │   └── todo_model.dart
│       │   ├── datasources/
│       │   │   ├── todo_remote_datasource.dart
│       │   │   └── todo_local_datasource.dart
│       │   └── repositories/
│       │       └── todo_repository_impl.dart
│       └── presentation/
│           ├── pages/
│           │   └── todo_list_page.dart
│           ├── widgets/
│           │   └── todo_item.dart
│           └── providers/
│               └── todo_provider.dart
└── shared/
    ├── widgets/
    └── providers/
```

### 4.5 Clean Architecture 优缺点

| 优点 | 缺点 |
|------|------|
| 高度可测试 | 代码量较大 |
| 业务逻辑完全独立 | 学习曲线陡峭 |
| 易于替换数据源 | 小型项目过度设计 |
| 团队协作友好 | 需要严格遵循依赖规则 |
| 长期维护成本低 | 开发速度初期较慢 |

---
Widget build(BuildContext context, WidgetRef ref) {
    final usersAsync = ref.watch(usersNotifierProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('用户列表'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              ref.read(usersNotifierProvider.notifier).refresh();
            },
          ),
        ],
      ),
      body: usersAsync.when(
        data: (users) => ListView.builder(
          itemCount: users.length,
          itemBuilder: (context, index) {
            final user = users[index];
            return ListTile(
              title: Text(user.name),
              subtitle: Text(user.email),
            );
          },
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Text('错误: $error'),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          ref.read(usersNotifierProvider.notifier).addUser(
                '新用户',
                'new@example.com',
              );
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### 6.3 Dio HTTP 客户端集成

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';
import '../constants/api_constants.dart';

class DioClient {
  late final Dio dio;

  DioClient() {
    dio = Dio(
      BaseOptions(
        baseUrl: ApiConstants.baseUrl,
        connectTimeout: const Duration(seconds: 30),
        receiveTimeout: const Duration(seconds: 30),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    _setupInterceptors();
  }

  void _setupInterceptors() {
    // 请求拦截器
    dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) async {
          // 添加认证令牌
          final token = await _getToken();
          if (token != null) {
            options.headers['Authorization'] = 'Bearer $token';
          }

          if (kDebugMode) {
            debugPrint('REQUEST[${options.method}] => PATH: ${options.path}');
          }

          return handler.next(options);
        },
        onResponse: (response, handler) {
          if (kDebugMode) {
            debugPrint(
              'RESPONSE[${response.statusCode}] => PATH: ${response.requestOptions.path}',
            );
          }
          return handler.next(response);
        },
        onError: (DioException e, handler) {
          if (kDebugMode) {
            debugPrint(
              'ERROR[${e.response?.statusCode}] => PATH: ${e.requestOptions.path}',
            );
          }

          // 处理 401 未授权
          if (e.response?.statusCode == 401) {
            // 触发重新登录或刷新令牌
          }

          return handler.next(e);
        },
      ),
    );

    // 日志拦截器（仅调试模式）
    if (kDebugMode) {
      dio.interceptors.add(
        LogInterceptor(
          requestBody: true,
          responseBody: true,
        ),
      );
    }
  }

  Future<String?> _getToken() async {
    // 从本地存储获取令牌
    return null;
  }
}
```

### 6.4 本地存储集成（Hive）

```dart
// lib/core/storage/local_storage.dart
import 'package:hive_flutter/hive_flutter.dart';

class LocalStorage {
  static const String _tokenBox = 'tokenBox';
  static const String _userBox = 'userBox';
  static const String _settingsBox = 'settingsBox';

  static Future<void> init() async {
    await Hive.initFlutter();
    await Hive.openBox<String>(_tokenBox);
    await Hive.openBox<Map>(_userBox);
    await Hive.openBox<dynamic>(_settingsBox);
  }

  // Token 操作
  static Future<void> saveToken(String token) async {
    final box = Hive.box<String>(_tokenBox);
    await box.put('accessToken', token);
  }

  static String? getToken() {
    final box = Hive.box<String>(_tokenBox);
    return box.get('accessToken');
  }

  static Future<void> clearToken() async {
    final box = Hive.box<String>(_tokenBox);
    await box.delete('accessToken');
  }

  // 设置操作
  static Future<void> saveSetting(String key, dynamic value) async {
    final box = Hive.box<dynamic>(_settingsBox);
    await box.put(key, value);
  }

  static T? getSetting<T>(String key) {
    final box = Hive.box<dynamic>(_settingsBox);
    return box.get(key) as T?;
  }
}
```

---

## 第七章：完整项目案例

### 7.1 电商应用案例

```
ecommerce_app/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── constants/
│   │   ├── errors/
│   │   ├── network/
│   │   ├── theme/
│   │   └── utils/
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── products/
│   │   │   ├── data/
│   │   │   │   ├── datasources/
│   │   │   │   │   └── product_remote_datasource.dart
│   │   │   │   ├── models/
│   │   │   │   │   └── product_model.dart
│   │   │   │   └── repositories/
│   │   │   │       └── product_repository_impl.dart
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── product_entity.dart
│   │   │   │   ├── repositories/
│   │   │   │   │   └── product_repository.dart
│   │   │   │   └── usecases/
│   │   │   │       ├── get_products.dart
│   │   │   │       ├── get_product_by_id.dart
│   │   │   │       └── search_products.dart
│   │   │   └── presentation/
│   │   │       ├── pages/
│   │   │       │   ├── product_list_page.dart
│   │   │       │   └── product_detail_page.dart
│   │   │       ├── widgets/
│   │   │       │   ├── product_card.dart
│   │   │       │   └── product_grid.dart
│   │   │       └── providers/
│   │   │           └── product_provider.dart
│   │   │
│   │   ├── cart/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── orders/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   └── profile/
│   │       ├── data/
│   │       ├── domain/
│   │       └── presentation/
│   │
│   ├── router/
│   │   └── app_router.dart
│   │
│   └── shared/
│       ├── widgets/
│       └── providers/
│
├── test/
│   ├── unit/
│   ├── widget/
│   └── integration/
│
└── pubspec.yaml
```

### 7.2 社交媒体应用案例

```
social_app/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   └── ...
│   │
│   ├── features/
│   │   ├── auth/
│   │   │   └── ...
│   │   │
│   │   ├── feed/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── post/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── chat/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   ├── notifications/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   │
│   │   └── profile/
│   │       ├── data/
│   │       ├── domain/
│   │       └── presentation/
│   │
│   └── ...
│
└── pubspec.yaml
```

---

## 附录 A：架构对比总结

| 特性 | MVC | MVP | MVVM | Clean Architecture | Feature-First |
|------|-----|-----|------|-------------------|---------------|
| 复杂度 | 低 | 中 | 中 | 高 | 中 |
| 可测试性 | 低 | 高 | 高 | 极高 | 高 |
| 可维护性 | 低 | 中 | 高 | 极高 | 高 |
| 学习曲线 | 低 | 中 | 中 | 高 | 中 |
| 适用项目 | 小型 | 中小型 | 中大型 | 大型 | 中大型 |
| 代码复用 | 低 | 中 | 高 | 极高 | 高 |
| 团队协作 | 差 | 中 | 好 | 极好 | 好 |
| 推荐状态管理 | setState | Provider | Riverpod/Bloc | Riverpod/Bloc | Riverpod/Bloc |

## 附录 B：推荐依赖配置

```yaml
# pubspec.yaml
name: my_app
description: A modern Flutter application

version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # 状态管理
  flutter_riverpod: ^2.4.9
  riverpod_annotation: ^2.3.3

  # 路由
  go_router: ^13.0.1

  # 网络
  dio: ^5.4.0
  retrofit: ^4.0.3

  # 本地存储
  hive: ^2.2.3
  hive_flutter: ^1.1.0

  # 代码生成
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1

  # 工具
  equatable: ^2.0.5
  dartz: ^0.10.1
  get_it: ^7.6.4
  injectable: ^2.3.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.1

  # 代码生成
  build_runner: ^2.4.7
  freezed: ^2.4.5
  json_serializable: ^6.7.1
  retrofit_generator: ^8.0.6
  riverpod_generator: ^2.3.9
  injectable_generator: ^2.4.1
  hive_generator: ^2.0.1

flutter:
  uses-material-design: true
```

## 附录 C：常见问题解答

### Q1: 新项目应该选择哪种架构？

A: 根据项目规模选择：
- **小型项目**（< 10 页面）：简单状态管理（StatefulWidget + Provider）
- **中型项目**（10-30 页面）：MVVM + Feature-First
- **大型项目**（> 30 页面）：Clean Architecture + Feature-First

### Q2: Riverpod 和 Bloc 如何选择？

A: 
- **Riverpod**：更轻量、类型安全、编译时安全
- **Bloc**：事件驱动、适合复杂状态机、社区成熟
- 推荐：新项目使用 Riverpod，已有 Bloc 项目可继续使用

### Q3: 如何处理跨功能的数据共享？

A: 
1. 在 `core/providers` 中定义全局 Provider
2. 使用 Repository 模式抽象数据访问
3. 通过依赖注入共享实例

### Q4: 如何组织测试代码？

A: 
```
test/
├── unit/              # 单元测试（UseCase、Repository）
├── widget/            # Widget 测试
├── integration/       # 集成测试
└── mocks/             # Mock 数据
```

### Q5: 如何处理错误和异常？

A: 
1. 在 `core/errors` 中定义统一的异常类型
2. Repository 层捕获异常并转换为 Domain 异常
3. ViewModel/Provider 层处理异常并更新 UI 状态

---

**文档版本**: 1.0  
**最后更新**: 2024年  
**Flutter 版本**: 3.41.0

---

> **补充知识**：架构设计没有银弹，选择适合团队和项目的架构才是最重要的。建议从简单开始，随着项目复杂度增加逐步演进架构。记住："过早优化是万恶之源"。

