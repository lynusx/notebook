# Dart Lints 6.1.0 详解与实战

## 第1章 认识 Dart Lints

### 1.1 什么是 Lints

Lints（静态代码分析规则）是 Dart 语言中用于识别源代码中潜在问题的重要工具。Dart 语言自带超过一百条 linter 规则，能够检测从潜在的类型问题、编码风格到代码格式化的各个方面。`package:lints` 是 Dart 团队官方推荐的 lint 规则集合，旨在帮助开发者编写高质量、符合 Dart 语言习惯的代码。

在 Dart 开发中，Linter 通过静态分析的方式检查代码，无需实际运行程序即可发现潜在问题。这些规则由 Dart 分析器（Analyzer）执行，结果会在支持 Dart 的集成开发环境（IDE）中直观展示，开发者也可以通过命令行运行 `dart analyze` 手动触发分析。

#### 1.1.1 为什么需要 Lints

在实际开发过程中，即使是经验丰富的开发者也难免会出现一些常见的编码错误或风格不一致的问题。例如：

- **空语句问题**：在 `if-else` 语句中不小心多写了分号，导致逻辑错误
- **命名不规范**：类名、变量名不符合 Dart 命名规范，降低代码可读性
- **类型安全问题**：未正确处理可空类型，可能导致运行时空指针异常
- **代码冗余**：使用冗长的语法实现简单功能，如用 `new` 关键字创建对象

通过启用合适的 lint 规则，可以在编码阶段就捕获这些问题，避免它们进入生产环境。同时，统一的代码风格规范也有助于团队协作，使代码库保持一致性和可维护性。

#### 1.1.2 Lints 6.1.0 的新特性

Lints 6.1.0 是 Dart 团队在 2025 年初发布的版本，相较于之前的版本，主要包含以下更新：

**Core 规则集的变更：**
- **新增** `library_annotations`：要求将库注解附加到库指令上
- **新增** `no_wildcard_variable_uses`：禁止使用通配符参数或变量
- **移除** `package_prefixed_library_names`：不再要求库名前缀包含包名

**Recommended 规则集的变更：**
- **移除** `library_names`：不再强制要求库名使用特定格式

此外，该版本将 SDK 的最低版本要求提升至 Dart 3.6，并将代码库迁移到了 `dart-lang/core` monorepo 中进行统一管理。

### 1.2 Lints 规则集的分类

`package:lints` 包含两套核心规则集，分别适用于不同的场景：

#### 1.2.1 Core Lints（核心规则集）

Core lints 包含帮助识别可能导致运行或消费 Dart 代码时出现问题的关键问题的规则。**所有代码都应该通过这些规则的检查**。这些规则主要关注：

- 潜在的运行时错误
- 类型安全问题
- 基本的代码规范

#### 1.2.2 Recommended Lints（推荐规则集）

Recommended lints 在 Core lints 的基础上，增加了帮助识别可能导致问题的其他问题，以及强制使用单一、惯用的 Dart 风格和格式的规则。**所有代码都建议通过这些规则的检查**。Recommended lints 包含了所有的 Core lints。

#### 1.2.3 Flutter Lints

除了 `package:lints` 提供的两套规则集外，Dart 团队还提供了 `package:flutter_lints`，它在 Recommended lints 的基础上增加了 Flutter 特定的推荐规则。对于 Flutter 项目，建议直接使用 `package:flutter_lints`。

Dart 框架小知识

**lints、flutter_lints 与自定义规则集的关系**

在实际项目中，开发者可以选择：
1. 使用 `package:lints/core.yaml` - 仅启用最基础的规则
2. 使用 `package:lints/recommended.yaml` - 启用推荐规则（包含 core）
3. 使用 `package:flutter_lints/flutter.yaml` - 启用 Flutter 推荐规则（包含 recommended）
4. 自定义规则集 - 基于上述规则集进行扩展或修改

Dart 团队明确表示不会提供超出 `core` 和 `recommended` 的官方规则集（如 `strict` 规则集）。但生态系统中存在许多社区维护的规则集，可在 pub.dev 上搜索 `topic:lints` 发现更多选择。

### 1.3 启用 Lints

#### 1.3.1 新项目自动启用

使用 `dart create` 命令创建的新 Dart 项目，默认会启用 `package:lints` 的 `recommended` 规则集。项目根目录下会自动生成 `analysis_options.yaml` 文件：

```yaml
include: package:lints/recommended.yaml
```

#### 1.3.2 现有项目手动启用

对于已有的 Dart 项目，可以通过以下步骤启用 lints：

**步骤 1：添加依赖**

在终端中，定位到项目根目录，运行以下命令：

```bash
dart pub add dev:lints
```

这会在 `pubspec.yaml` 的 `dev_dependencies` 中添加 `lints` 包：

```yaml
dev_dependencies:
  lints: ^6.1.0
```

**步骤 2：创建配置文件**

在项目根目录（与 `pubspec.yaml` 同级）创建 `analysis_options.yaml` 文件，内容如下：

```yaml
# 使用推荐规则集
include: package:lints/recommended.yaml
```

或者，如果只需要核心规则：

```yaml
# 使用核心规则集
include: package:lints/core.yaml
```

#### 1.3.3 升级到最新版本

要将 lint 规则集升级到最新版本，只需运行：

```bash
dart pub add dev:lints
```

这将自动获取 `pubspec.yaml` 中指定的版本范围内的最新版本。

Dart Tips 语法小贴士

**pubspec.yaml 中的版本约束**

在 `pubspec.yaml` 中，`^6.1.0` 表示兼容 6.1.0 及以上但小于 7.0.0 的版本。这种写法称为"Caret 语法"，是 Dart 包管理中的常见做法。它允许自动获取次版本和补丁版本的更新，但不会自动升级到主版本（可能包含破坏性变更）。

例如：
- `^6.1.0` 允许 6.1.0、6.2.0、6.1.1 等，但不允许 7.0.0
- `>=6.1.0 <7.0.0` 与 `^6.1.0` 等价
- `6.1.0` 表示精确匹配 6.1.0 版本

### 1.4 自定义 Lints 配置

#### 1.4.1 禁用特定规则

有时某些规则可能不适用于特定项目，可以在 `analysis_options.yaml` 中禁用它们：

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    avoid_print: false  # 禁用 avoid_print 规则
    prefer_single_quotes: false  # 禁用 prefer_single_quotes 规则
```

#### 1.4.2 启用额外规则

如果需要启用推荐规则集中未包含的规则，可以这样配置：

```yaml
include: package:lints/recommended.yaml

linter:
  rules:
    always_declare_return_types: true  # 启用额外规则
    prefer_final_locals: true  # 启用额外规则
```

#### 1.4.3 忽略特定行的警告

在某些情况下，可能需要在特定代码行禁用某个规则。可以使用注释来实现：

```dart
// 忽略单行
// ignore: avoid_print
print('调试信息');

// 忽略整个文件
// ignore_for_file: avoid_print
void main() {
  print('这不会触发警告');
}
```

## 第2章 Core Lints 详解

Core lints 是所有 Dart 代码都应该通过的基础规则集。本章将详细介绍每一条 Core lint 规则，包括其用途、示例以及修复建议。

### 2.1 错误预防类规则

这类规则帮助开发者在编码阶段捕获可能导致运行时错误或逻辑错误的问题。

#### 2.1.1 avoid_empty_else

**规则说明**：避免在 `else` 子句中使用空语句。

**用途**：此规则检测 `if` 语句的 `else` 子句后紧跟分号的情况，这通常是由于误输入导致的逻辑错误。

**错误示例**：

```dart
// 错误：else 后面有一个多余的分号
if (x > y)
  print('x 更大');
else ;  // 这个分号导致 print('y 更大') 无条件执行
  print('y 更大');
```

在上述代码中，由于 `else` 后面多了一个分号，`print('y 更大')` 实际上会在任何情况下都执行，而不是仅在 `x <= y` 时执行。

**正确写法**：

```dart
// 正确：移除多余的分号
if (x > y)
  print('x 更大');
else
  print('y 更大');

// 或者使用花括号使结构更清晰
if (x > y) {
  print('x 更大');
} else {
  print('y 更大');
}
```

如果希望 `print('y 更大')` 无条件执行，应该移除 `else` 子句：

```dart
if (x > y) print('x 更大');
print('y 更大');  // 无条件执行
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.1.2 avoid_relative_lib_imports

**规则说明**：避免对 `lib/` 目录下的文件使用相对导入。

**用途**：在 Dart 包中，`lib/` 目录是包的公共 API。使用相对路径导入 `lib/` 下的文件可能导致问题，特别是当文件被移动到不同位置时。

**错误示例**：

```dart
// 假设文件位于 lib/src/utils.dart
// 错误：使用相对路径导入 lib/ 下的文件
import '../models/user.dart';  // 相对导入
```

**正确写法**：

```dart
// 正确：使用包导入
import 'package:my_package/models/user.dart';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

Dart 框架小知识

**包导入 vs 相对导入**

在 Dart 中有两种导入方式：
1. **包导入（Package imports）**：`package:package_name/path/to/file.dart`
   - 用于导入当前包或其他包的公共 API
   - 从包的 `lib/` 目录开始解析路径
   
2. **相对导入（Relative imports）**：`../path/to/file.dart` 或 `./file.dart`
   - 相对于当前文件的位置
   - 适用于导入同一目录或子目录下的文件

在 `lib/` 目录内的文件应该始终使用包导入来引用 `lib/` 下的其他文件。相对导入应该只用于导入不在 `lib/` 下的文件（如 `test/`、`example/` 目录中的文件）。

#### 2.1.3 avoid_shadowing_type_parameters

**规则说明**：避免遮蔽（shadowing）类型参数。

**用途**：当泛型类或泛型方法的类型参数与外部作用域的类型参数同名时，内部的类型参数会"遮蔽"外部的类型参数，导致混淆和潜在的错误。

**错误示例**：

```dart
// 错误：方法类型参数 T 遮蔽了类类型参数 T
class Box<T> {
  T value;
  
  // 这里的 T 遮蔽了类的 T
  void setValue<T>(T newValue) {
    value = newValue;  // 编译错误：类型不匹配
  }
}
```

**正确写法**：

```dart
// 正确：使用不同的类型参数名
class Box<T> {
  T value;
  
  void setValue<E>(E newValue) {
    // 如果确实需要存储不同类型的值，需要重新设计
  }
}

// 或者，如果方法应该使用类的类型参数
class Box<T> {
  T value;
  
  void setValue(T newValue) {
    value = newValue;
  }
}
```

#### 2.1.4 avoid_types_as_parameter_names

**规则说明**：避免使用类型名作为参数名。

**用途**：使用类型名（如 `int`、`String`、`List` 等）作为函数参数名会造成混淆，降低代码可读性。

**错误示例**：

```dart
// 错误：使用类型名作为参数名
void process(int String) {  // 容易混淆
  print(String);
}
```

**正确写法**：

```dart
// 正确：使用有意义的参数名
void process(int count) {
  print(count);
}
```

#### 2.1.5 no_duplicate_case_values

**规则说明**：不要在 `switch` 语句中使用重复的 `case` 值。

**用途**：重复的 `case` 值会导致编译错误，因为 Dart 无法确定应该执行哪个分支。

**错误示例**：

```dart
// 错误：case 1 重复了
switch (value) {
  case 1:
    print('一');
    break;
  case 2:
    print('二');
    break;
  case 1:  // 重复！
    print('又是一');
    break;
}
```

**正确写法**：

```dart
// 正确：每个 case 值只出现一次
switch (value) {
  case 1:
    print('一');
    break;
  case 2:
    print('二');
    break;
  case 3:
    print('三');
    break;
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.1.6 hash_and_equals

**规则说明**：如果重写了 `==` 运算符，必须同时重写 `hashCode` 方法。

**用途**：在 Dart 中，如果两个对象相等（`==` 返回 `true`），它们的 `hashCode` 必须相同。这是使用对象作为 `Set` 元素或 `Map` 键的基本要求。

**错误示例**：

```dart
// 错误：只重写了 ==，没有重写 hashCode
class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is Person &&
        other.name == name &&
        other.age == age;
  }
  // 缺少 hashCode 重写！
}
```

**正确写法**：

```dart
// 正确：同时重写 == 和 hashCode
class Person {
  final String name;
  final int age;
  
  Person(this.name, this.age);
  
  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is Person &&
        other.name == name &&
        other.age == age;
  }
  
  @override
  int get hashCode => Object.hash(name, age);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

Dart Tips 语法小贴士

**Object.hash() 的使用**

从 Dart 2.14 开始，可以使用 `Object.hash()` 方法方便地计算多个字段的哈希码：

```dart
@override
int get hashCode => Object.hash(field1, field2, field3);
```

对于更复杂的类，可以使用 `Object.hashAll()` 处理列表：

```dart
@override
int get hashCode => Object.hashAll([field1, field2, ...listField]);
```

### 2.2 命名规范类规则

这类规则确保代码遵循 Dart 的命名约定，提高代码的一致性和可读性。

#### 2.2.1 camel_case_types

**规则说明**：类型名（类、枚举、typedef、泛型参数）应该使用 UpperCamelCase（大驼峰命名法）。

**用途**：统一的类型命名规范使代码更易读，也符合 Dart 社区的惯例。

**错误示例**：

```dart
// 错误：类型名不符合 UpperCamelCase
class userInfo { }
enum color { red, green, blue }
typedef stringList = List<String>;
```

**正确写法**：

```dart
// 正确：使用 UpperCamelCase
class UserInfo { }
enum Color { red, green, blue }
typedef StringList = List<String>;
```

#### 2.2.2 camel_case_extensions

**规则说明**：扩展名应该使用 UpperCamelCase（大驼峰命名法）。

**用途**：扩展（Extension）是一种类型，因此应该遵循类型命名规范。

**错误示例**：

```dart
// 错误：扩展名不符合 UpperCamelCase
extension stringUtils on String { }
```

**正确写法**：

```dart
// 正确：使用 UpperCamelCase
extension StringUtils on String { }
```

#### 2.2.3 non_constant_identifier_names

**规则说明**：非常量标识符（变量、函数、方法名等）应该使用 lowerCamelCase（小驼峰命名法）。

**用途**：统一的标识符命名规范使代码更易读。

**错误示例**：

```dart
// 错误：变量名和方法名不符合 lowerCamelCase
var User_Name = '张三';
void Print_Message() { }
```

**正确写法**：

```dart
// 正确：使用 lowerCamelCase
var userName = '张三';
void printMessage() { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.2.4 file_names

**规则说明**：源文件名应该使用 `lowercase_with_underscores`（小写加下划线）格式。

**用途**：统一的文件命名规范使项目结构更清晰，也避免在某些文件系统上出现大小写敏感问题。

**错误示例**：

```dart
// 错误文件名：
// UserInfo.dart
// user-info.dart
// userInfo.dart
```

**正确写法**：

```dart
// 正确文件名：
// user_info.dart
// my_class.dart
// main.dart
```

### 2.3 代码风格类规则

这类规则帮助保持代码风格的一致性和可读性。

#### 2.3.1 curly_braces_in_flow_control_structures

**规则说明**：在所有流程控制结构中使用花括号。

**用途**：即使只有一条语句，使用花括号也能使代码更清晰，避免在添加新语句时引入错误。

**错误示例**：

```dart
// 错误：缺少花括号
if (condition)
  print('条件为真');

for (var i = 0; i < 10; i++)
  print(i);
```

**正确写法**：

```dart
// 正确：使用花括号
if (condition) {
  print('条件为真');
}

for (var i = 0; i < 10; i++) {
  print(i);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.3.2 empty_catches

**规则说明**：避免空的 `catch` 块。

**用途**：空的 `catch` 块会静默吞掉异常，使调试变得困难。至少应该记录异常或重新抛出。

**错误示例**：

```dart
// 错误：空的 catch 块
try {
  riskyOperation();
} catch (e) {
  // 什么都没有做！
}
```

**正确写法**：

```dart
// 正确：至少记录异常
try {
  riskyOperation();
} catch (e) {
  print('操作失败: $e');
}

// 或者重新抛出
try {
  riskyOperation();
} catch (e) {
  // 处理或记录
  rethrow;
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.3.3 dangling_library_doc_comments

**规则说明**：将库文档注释附加到库指令上。

**用途**：在 Dart 中，库的文档注释应该放在 `library` 指令之前，而不是其他声明之前，以确保它被正确识别为库文档。

**错误示例**：

```dart
// 错误：文档注释没有附加到 library 指令
/// 这是一个工具库
import 'dart:io';
```

**正确写法**：

```dart
// 正确：文档注释附加到 library 指令
/// 这是一个工具库
library;

import 'dart:io';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.4 类型安全类规则

这类规则帮助确保代码的类型安全，避免运行时类型错误。

#### 2.4.1 avoid_shadowing_type_parameters

**规则说明**：避免遮蔽类型参数。

**用途**：当内部泛型声明的类型参数与外部泛型声明的类型参数同名时，内部的会遮蔽外部的，导致混淆。

**错误示例**：

```dart
// 错误：T 遮蔽了外部 T
class Container<T> {
  void add<T>(T value) { }  // 这里的 T 遮蔽了类的 T
}
```

**正确写法**：

```dart
// 正确：使用不同的类型参数名
class Container<T> {
  void add<E>(E value) { }
}
```

#### 2.4.2 null_check_on_nullable_type_parameter

**规则说明**：不要对可能为空的类型参数进行空检查。

**用途**：在泛型代码中，对类型参数进行空检查通常是不必要的，因为类型参数本身可以是可空或不可空的。

**错误示例**：

```dart
// 错误：对类型参数进行空检查
T? process<T>(T? value) {
  if (value == null) return null;  // 不必要的检查
  return value;
}
```

**正确写法**：

```dart
// 正确：使用空感知操作符
T? process<T>(T? value) {
  return value?.toString() as T?;
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.4.3 strict_top_level_inference

**规则说明**：指定顶级类型注解。

**用途**：顶级声明（变量、函数、getter、setter）应该显式指定类型，以提高代码可读性和类型安全。

**错误示例**：

```dart
// 错误：缺少类型注解
var globalValue = 42;  // 类型推断为 int，但不明确

process(data) {  // 返回类型和参数类型都不明确
  return data.toString();
}
```

**正确写法**：

```dart
// 正确：显式指定类型
int globalValue = 42;

String process(Object data) {
  return data.toString();
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.5 集合操作类规则

这类规则帮助正确使用集合相关的操作。

#### 2.5.1 collection_methods_unrelated_type

**规则说明**：检测对集合方法传递不相关类型参数的情况。

**用途**：向集合方法（如 `contains`、`remove` 等）传递与集合元素类型不相关的参数，通常会导致意外的结果。

**错误示例**：

```dart
// 错误：向 List<String> 传递 int
var names = <String>['Alice', 'Bob'];
names.contains(42);  // 总是返回 false
```

**正确写法**：

```dart
// 正确：传递正确类型的参数
var names = <String>['Alice', 'Bob'];
names.contains('Alice');  // 正确
```

#### 2.5.2 prefer_is_empty

**规则说明**：对 `Iterable` 和 `Map` 使用 `isEmpty`。

**用途**：使用 `isEmpty` 比使用 `.length == 0` 更简洁、更高效（某些集合计算长度需要遍历所有元素）。

**错误示例**：

```dart
// 错误：使用 .length == 0
if (list.length == 0) { }
if (map.length == 0) { }
```

**正确写法**：

```dart
// 正确：使用 isEmpty
if (list.isEmpty) { }
if (map.isEmpty) { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.5.3 prefer_is_not_empty

**规则说明**：对 `Iterable` 和 `Map` 使用 `isNotEmpty`。

**用途**：与 `prefer_is_empty` 类似，使用 `isNotEmpty` 更简洁、更高效。

**错误示例**：

```dart
// 错误：使用 .length != 0
if (list.length != 0) { }
if (list.length > 0) { }
```

**正确写法**：

```dart
// 正确：使用 isNotEmpty
if (list.isNotEmpty) { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.5.4 prefer_iterable_whereType

**规则说明**：在 `Iterable` 上优先使用 `whereType`。

**用途**：`whereType<T>()` 比 `where((e) => e is T)` 更简洁、类型更安全。

**错误示例**：

```dart
// 错误：使用 where + is
var numbers = objects.where((e) => e is int).cast<int>();
```

**正确写法**：

```dart
// 正确：使用 whereType
var numbers = objects.whereType<int>();
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.6 异步编程类规则

这类规则帮助正确使用 Dart 的异步特性。

#### 2.6.1 await_only_futures

**规则说明**：只对 `Future` 使用 `await`。

**用途**：对非 `Future` 值使用 `await` 是多余的，可能导致混淆。

**错误示例**：

```dart
// 错误：对非 Future 使用 await
var value = await 42;  // 多余
```

**正确写法**：

```dart
// 正确：直接赋值
var value = 42;

// 或者如果确实是 Future
Future<int> getValue() async => 42;
var value = await getValue();
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.7 元数据注解类规则

这类规则帮助正确使用注解。

#### 2.7.1 library_annotations

**规则说明**：将库注解附加到库指令上。

**用途**：在 Dart 3.0+ 中，库级别的注解（如 `@Deprecated`）应该放在 `library` 指令之前。

**错误示例**：

```dart
// 错误：注解没有附加到 library 指令
@Deprecated('使用新库')
import 'dart:io';
```

**正确写法**：

```dart
// 正确：注解附加到 library 指令
@Deprecated('使用新库')
library;

import 'dart:io';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.7.2 provide_deprecation_message

**规则说明**：通过 `@Deprecated("message")` 提供弃用消息。

**用途**：弃用声明应该包含说明性的消息，告诉用户为什么被弃用以及应该使用什么替代方案。

**错误示例**：

```dart
// 错误：缺少弃用消息
@Deprecated('')
void oldMethod() { }
```

**正确写法**：

```dart
// 正确：提供有意义的弃用消息
@Deprecated('使用 newMethod() 替代')
void oldMethod() { }

@Deprecated('将在 v2.0 中移除，请使用 newMethod()')
void oldMethod() { }
```

### 2.8 代码简化类规则

这类规则帮助简化代码，去除冗余。

#### 2.8.1 prefer_generic_function_type_aliases

**规则说明**：优先使用泛型函数类型别名。

**用途**：使用 `typedef` 的泛型形式比旧的 `Function` 语法更清晰。

**错误示例**：

```dart
// 错误：使用旧的 Function 语法
typedef int Comparator(String a, String b);
```

**正确写法**：

```dart
// 正确：使用泛型函数类型别名
typedef Comparator<T> = int Function(T a, T b);
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.8.2 prefer_typing_uninitialized_variables

**规则说明**：优先为未初始化的变量和字段指定类型。

**用途**：未初始化的变量如果没有类型注解，其类型会被推断为 `dynamic`，这可能导致运行时错误。

**错误示例**：

```dart
// 错误：未初始化的变量没有类型
var value;  // 推断为 dynamic
value = 42;
```

**正确写法**：

```dart
// 正确：显式指定类型
int value;
value = 42;

// 或者立即初始化
var value = 42;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.8.3 implicit_call_tearoffs

**规则说明**：在使用对象作为函数时显式地 tear-off `call` 方法。

**用途**：当对象实现了 `call` 方法时，显式地 tear-off 可以使代码意图更清晰。

**错误示例**：

```dart
// 错误：隐式 tear-off
class Callable {
  void call() => print('called');
}

void main() {
  var callable = Callable();
  var fn = callable;  // 隐式 tear-off
}
```

**正确写法**：

```dart
// 正确：显式 tear-off
class Callable {
  void call() => print('called');
}

void main() {
  var callable = Callable();
  var fn = callable.call;  // 显式 tear-off
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.9 导入和库组织类规则

#### 2.9.1 depend_on_referenced_packages

**规则说明**：依赖被引用的包。

**用途**：如果在代码中导入了某个包，应该在 `pubspec.yaml` 中显式声明对该包的依赖。

**错误示例**：

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  # 缺少 http 依赖
```

```dart
// 错误：使用了未声明依赖的包
import 'package:http/http.dart' as http;
```

**正确写法**：

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.0.0  # 声明依赖
```

```dart
import 'package:http/http.dart' as http;
```

#### 2.9.2 use_string_in_part_of_directives

**规则说明**：在 `part of` 指令中使用字符串。

**用途**：使用字符串形式的 `part of` 指令可以避免 Dart 分析器需要解析被引用的库文件。

**错误示例**：

```dart
// 错误：使用库名
part of my_library;
```

**正确写法**：

```dart
// 正确：使用字符串
part of 'my_library.dart';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 2.10 其他 Core 规则

#### 2.10.1 no_wildcard_variable_uses

**规则说明**：不要使用通配符参数或变量。

**用途**：在 Dart 3.0+ 中，`_` 可以用作通配符，表示"忽略此值"。使用通配符变量通常表明代码可以简化。

**错误示例**：

```dart
// 错误：使用通配符变量
var _ = someValue;
print(_);  // 通配符不应该被使用
```

**正确写法**：

```dart
// 正确：如果不需要，不使用变量
someFunction();

// 或者使用有意义的变量名
var value = someValue;
print(value);
```

#### 2.10.2 type_literal_in_constant_pattern

**规则说明**：不要在常量模式中使用类型字面量。

**用途**：在模式匹配中，类型字面量可能导致意外的行为。

**错误示例**：

```dart
// 错误：在常量模式中使用类型字面量
switch (value) {
  case int:  // 不正确
    print('是 int 类型');
}
```

**正确写法**：

```dart
// 正确：使用类型测试模式
switch (value) {
  case int _:
    print('是 int 类型');
}

// 或者使用 is 表达式
if (value is int) {
  print('是 int 类型');
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.10.3 unintended_html_in_doc_comment

**规则说明**：文档注释中的尖括号会被 Markdown 视为 HTML。

**用途**：在文档注释中使用 `<` 和 `>` 可能会被 Markdown 解析为 HTML 标签，导致渲染问题。

**错误示例**：

```dart
// 错误：尖括号可能被解析为 HTML
/// 返回 List<String> 类型的值
List<String> getValues() => [];
```

**正确写法**：

```dart
// 正确：使用反引号包裹代码
/// 返回 `List<String>` 类型的值
List<String> getValues() => [];
```

#### 2.10.4 unnecessary_overrides

**规则说明**：不要重写一个方法只是调用父类的同名方法。

**用途**：这种重写是多余的，没有任何实际作用。

**错误示例**：

```dart
// 错误：不必要的重写
class Parent {
  void doSomething() => print('Parent');
}

class Child extends Parent {
  @override
  void doSomething() => super.doSomething();  // 多余！
}
```

**正确写法**：

```dart
// 正确：不需要重写
class Parent {
  void doSomething() => print('Parent');
}

class Child extends Parent {
  // 直接继承父类方法
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 2.10.5 unrelated_type_equality_checks

**规则说明**：检测不相关类型之间的相等性比较。

**用途**：比较不相关类型的相等性通常总是返回 `false`，这通常是逻辑错误。

**错误示例**：

```dart
// 错误：比较不相关的类型
var result = 'hello' == 42;  // 总是 false
```

**正确写法**：

```dart
// 正确：比较相同或可比较的类型
var result = 'hello' == 'world';

// 或者先进行类型转换
var result = int.parse('42') == 42;
```

#### 2.10.6 valid_regexps

**规则说明**：使用有效的正则表达式语法。

**用途**：无效的正则表达式会在运行时抛出异常。

**错误示例**：

```dart
// 错误：无效的正则表达式
var pattern = RegExp('[');  // 缺少闭合括号
```

**正确写法**：

```dart
// 正确：有效的正则表达式
var pattern = RegExp(r'\[');  // 转义左括号
```

#### 2.10.7 void_checks

**规则说明**：不要给 `void` 赋值。

**用途**：`void` 表示"无值"，不应该被赋值或作为参数传递。

**错误示例**：

```dart
// 错误：给 void 赋值
void doSomething() {}

var result = doSomething();  // result 是 void
int value = result;  // 错误！
```

**正确写法**：

```dart
// 正确：void 值不应该被使用
void doSomething() {}

doSomething();  // 直接使用，不赋值
```

#### 2.10.8 secure_pubspec_urls

**规则说明**：在 `pubspec.yaml` 中使用安全的 URL。

**用途**：依赖项应该使用 HTTPS 而不是 HTTP，以确保安全性。

**错误示例**：

```yaml
# 错误：使用 HTTP
dependencies:
  my_package:
    hosted:
      name: my_package
      url: http://pub.example.com
```

**正确写法**：

```yaml
# 正确：使用 HTTPS
dependencies:
  my_package:
    hosted:
      name: my_package
      url: https://pub.example.com
```

## 第3章 Recommended Lints 详解

Recommended lints 在 Core lints 的基础上增加了更多规则，帮助识别可能导致问题的其他问题，并强制使用单一、惯用的 Dart 风格和格式。本章将详细介绍每一条 Recommended lint 规则。

### 3.1 继承和方法重写类规则

这类规则帮助正确处理类的继承和方法重写。

#### 3.1.1 annotate_overrides

**规则说明**：为重写的成员添加 `@override` 注解。

**用途**：`@override` 注解明确表示一个方法是重写父类的方法，提高代码可读性，并在父类方法签名改变时提供编译时警告。

**错误示例**：

```dart
// 错误：缺少 @override 注解
class Animal {
  void makeSound() => print('Some sound');
}

class Dog extends Animal {
  void makeSound() => print('Woof!');  // 应该添加 @override
}
```

**正确写法**：

```dart
// 正确：添加 @override 注解
class Animal {
  void makeSound() => print('Some sound');
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof!');
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.1.2 avoid_renaming_method_parameters

**规则说明**：不要重命名被重写方法的参数。

**用途**：保持参数名一致可以提高代码可读性，并使文档更一致。

**错误示例**：

```dart
// 错误：重命名了参数
class Animal {
  void eat(String food) => print('Eating $food');
}

class Dog extends Animal {
  @override
  void eat(String meal) => print('Dog eating $meal');  // 参数名不一致
}
```

**正确写法**：

```dart
// 正确：保持参数名一致
class Animal {
  void eat(String food) => print('Eating $food');
}

class Dog extends Animal {
  @override
  void eat(String food) => print('Dog eating $food');
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.1.3 overridden_fields

**规则说明**：不要重写字段。

**用途**：重写字段（而不是 getter/setter）可能导致意外的行为，特别是在多态场景下。

**错误示例**：

```dart
// 错误：重写字段
class Parent {
  String name = 'Parent';
}

class Child extends Parent {
  @override
  String name = 'Child';  // 不应该这样做
}
```

**正确写法**：

```dart
// 正确：使用 getter
class Parent {
  String get name => 'Parent';
}

class Child extends Parent {
  @override
  String get name => 'Child';
}

// 或者使用构造函数初始化
class Parent {
  final String name;
  Parent(this.name);
}

class Child extends Parent {
  Child() : super('Child');
}
```

### 3.2 代码简化类规则

这类规则帮助简化代码，去除冗余。

#### 3.2.1 avoid_function_literals_in_foreach_calls

**规则说明**：避免在 `forEach` 调用中使用函数字面量。

**用途**：使用 `for-in` 循环通常比 `forEach` 更简洁、更易读。

**错误示例**：

```dart
// 错误：使用 forEach + 函数字面量
items.forEach((item) {
  print(item);
});
```

**正确写法**：

```dart
// 正确：使用 for-in 循环
for (var item in items) {
  print(item);
}

// 或者使用方法 tear-off
items.forEach(print);
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.2 avoid_init_to_null

**规则说明**：不要显式地将变量初始化为 `null`。

**用途**：在 Dart 中，可空类型的变量默认就是 `null`，显式初始化是多余的。

**错误示例**：

```dart
// 错误：显式初始化为 null
String? name = null;
int? count = null;
```

**正确写法**：

```dart
// 正确：省略 = null
String? name;
int? count;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.3 avoid_return_types_on_setters

**规则说明**：避免在 setter 上指定返回类型。

**用途**：setter 的返回类型总是 `void`，显式指定是多余的。

**错误示例**：

```dart
// 错误：显式指定返回类型
class Person {
  String _name = '';
  
  void set name(String value) => _name = value;  // 多余
}
```

**正确写法**：

```dart
// 正确：省略返回类型
class Person {
  String _name = '';
  
  set name(String value) => _name = value;
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.4 avoid_returning_null_for_void

**规则说明**：避免为 `void` 返回类型返回 `null`。

**用途**：`void` 表示"无返回值"，返回 `null` 是多余的。

**错误示例**：

```dart
// 错误：为 void 返回 null
void doSomething() {
  if (condition) return null;  // 多余
}
```

**正确写法**：

```dart
// 正确：直接 return
void doSomething() {
  if (condition) return;
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.5 avoid_single_cascade_in_expression_statements

**规则说明**：避免在表达式语句中使用单个级联操作。

**用途**：单个级联操作可以使用普通的方法调用替代，更简单。

**错误示例**：

```dart
// 错误：单个级联操作
list..add(item);
```

**正确写法**：

```dart
// 正确：直接调用方法
list.add(item);

// 多个级联操作是允许的
list
  ..add(item1)
  ..add(item2)
  ..remove(item3);
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.6 empty_constructor_bodies

**规则说明**：空构造函数体使用 `;` 而不是 `{}`。

**用途**：空的构造函数体使用分号更简洁。

**错误示例**：

```dart
// 错误：空的构造函数体使用 {}
class Person {
  final String name;
  Person(this.name) {}
}
```

**正确写法**：

```dart
// 正确：使用 ;
class Person {
  final String name;
  Person(this.name);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.2.7 empty_statements

**规则说明**：避免空语句。

**用途**：空语句（单独的分号）通常是误输入，没有任何作用。

**错误示例**：

```dart
// 错误：空语句
if (condition);
  doSomething();
```

**正确写法**：

```dart
// 正确：移除空语句
if (condition) {
  doSomething();
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.3 命名规范类规则

#### 3.3.1 constant_identifier_names

**规则说明**：常量名使用 `lowerCamelCase`。

**用途**：虽然 Dart 允许常量使用 `SCREAMING_SNAKE_CASE`，但 `lowerCamelCase` 更一致。

**错误示例**：

```dart
// 错误：使用 SCREAMING_SNAKE_CASE
const MAX_COUNT = 100;
const PI_VALUE = 3.14;
```

**正确写法**：

```dart
// 正确：使用 lowerCamelCase
const maxCount = 100;
const piValue = 3.14;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.3.2 library_prefixes

**规则说明**：库前缀使用 `lowercase_with_underscores`。

**用途**：统一的库前缀命名规范。

**错误示例**：

```dart
// 错误：库前缀不符合规范
import 'package:my_package/utils.dart' as MyUtils;
```

**正确写法**：

```dart
// 正确：使用 lowercase_with_underscores
import 'package:my_package/utils.dart' as my_utils;
```

#### 3.3.3 no_leading_underscores_for_library_prefixes

**规则说明**：库前缀不要使用前导下划线。

**用途**：前导下划线通常表示私有，但库前缀不需要这种语义。

**错误示例**：

```dart
// 错误：库前缀使用前导下划线
import 'package:my_package/utils.dart' as _utils;
```

**正确写法**：

```dart
// 正确：移除前导下划线
import 'package:my_package/utils.dart' as utils;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.3.4 no_leading_underscores_for_local_identifiers

**规则说明**：局部标识符不要使用前导下划线。

**用途**：局部变量和参数本来就是私有的，前导下划线是多余的。

**错误示例**：

```dart
// 错误：局部变量使用前导下划线
void doSomething() {
  var _value = 42;  // 多余
  print(_value);
}
```

**正确写法**：

```dart
// 正确：移除前导下划线
void doSomething() {
  var value = 42;
  print(value);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.3.5 package_names

**规则说明**：包名使用 `lowercase_with_underscores`。

**用途**：统一的包命名规范。

**错误示例**：

```yaml
# 错误：包名不符合规范
name: MyPackage
name: my-package
```

**正确写法**：

```yaml
# 正确：使用 lowercase_with_underscores
name: my_package
```

### 3.4 集合和字符串操作类规则

#### 3.4.1 prefer_adjacent_string_concatenation

**规则说明**：使用相邻字符串来连接字符串字面量。

**用途**：相邻字符串（不使用 `+`）是连接字符串字面量的推荐方式。

**错误示例**：

```dart
// 错误：使用 + 连接字符串字面量
var message = 'Hello' + ' ' + 'World';
```

**正确写法**：

```dart
// 正确：使用相邻字符串
var message = 'Hello' ' ' 'World';

// 或者使用插值
var message = 'Hello World';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.2 prefer_collection_literals

**规则说明**：尽可能使用集合字面量。

**用途**：集合字面量（`[]`、`{}`）比构造函数更简洁。

**错误示例**：

```dart
// 错误：使用构造函数
var list = List<String>();
var map = Map<String, int>();
var set = Set<String>();
```

**正确写法**：

```dart
// 正确：使用集合字面量
var list = <String>[];
var map = <String, int>{};
var set = <String>{};
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.3 prefer_conditional_assignment

**规则说明**：优先使用 `??=` 而不是 `null` 检查。

**用途**：`??=` 是空感知赋值操作符，更简洁。

**错误示例**：

```dart
// 错误：使用 null 检查
if (value == null) {
  value = defaultValue;
}
```

**正确写法**：

```dart
// 正确：使用 ??=
value ??= defaultValue;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.4 prefer_contains

**规则说明**：对 `List` 和 `String` 实例使用 `contains`。

**用途**：`contains` 比 `indexOf` 更清晰。

**错误示例**：

```dart
// 错误：使用 indexOf
if (list.indexOf(item) >= 0) { }
if (string.indexOf(substring) != -1) { }
```

**正确写法**：

```dart
// 正确：使用 contains
if (list.contains(item)) { }
if (string.contains(substring)) { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.5 prefer_for_elements_to_map_fromIterable

**规则说明**：优先使用 `for` 元素来从 `Iterable` 构建 `Map`。

**用途**：集合字面量中的 `for` 元素比 `Map.fromIterable` 更简洁。

**错误示例**：

```dart
// 错误：使用 Map.fromIterable
var map = Map.fromIterable(
  items,
  key: (item) => item.id,
  value: (item) => item.name,
);
```

**正确写法**：

```dart
// 正确：使用 for 元素
var map = {for (var item in items) item.id: item.name};
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.6 prefer_inlined_adds

**规则说明**：尽可能内联列表项声明。

**用途**：在列表字面量中直接添加元素，而不是后续调用 `add`。

**错误示例**：

```dart
// 错误：后续调用 add
var list = <int>[]
  ..add(1)
  ..add(2);
```

**正确写法**：

```dart
// 正确：在字面量中声明
var list = [1, 2];
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.7 prefer_interpolation_to_compose_strings

**规则说明**：使用插值来组合字符串和值。

**用途**：字符串插值比 `+` 连接更清晰。

**错误示例**：

```dart
// 错误：使用 + 连接
var greeting = 'Hello, ' + name + '!';
```

**正确写法**：

```dart
// 正确：使用插值
var greeting = 'Hello, $name!';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.8 prefer_is_not_operator

**规则说明**：优先使用 `is!` 操作符。

**用途**：`is!` 比 `!(... is ...)` 更简洁。

**错误示例**：

```dart
// 错误：使用 !(... is ...)
if (!(value is String)) { }
```

**正确写法**：

```dart
// 正确：使用 is!
if (value is! String) { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.9 prefer_null_aware_operators

**规则说明**：优先使用空感知操作符。

**用途**：空感知操作符（`?.`、`??`、`??=`）使代码更简洁。

**错误示例**：

```dart
// 错误：使用 null 检查
if (value != null) {
  value.doSomething();
}
```

**正确写法**：

```dart
// 正确：使用空感知操作符
value?.doSomething();
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.4.10 prefer_spread_collections

**规则说明**：尽可能使用展开集合。

**用途**：展开操作符（`...` 和 `...?`）使代码更简洁。

**错误示例**：

```dart
// 错误：使用 addAll
var list = <int>[]
  ..addAll(items1)
  ..addAll(items2);
```

**正确写法**：

```dart
// 正确：使用展开操作符
var list = [...items1, ...items2];
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.5 函数和变量声明类规则

#### 3.5.1 prefer_final_fields

**规则说明**：私有字段可以是 `final` 的。

**用途**：如果字段只在构造函数中初始化且不再改变，应该声明为 `final`。

**错误示例**：

```dart
// 错误：可以是 final 的字段
class Person {
  String _name;  // 可以是 final
  
  Person(this._name);
}
```

**正确写法**：

```dart
// 正确：声明为 final
class Person {
  final String _name;
  
  Person(this._name);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.5.2 prefer_function_declarations_over_variables

**规则说明**：使用函数声明来将函数绑定到名称。

**用途**：函数声明比将函数字面量赋值给变量更清晰。

**错误示例**：

```dart
// 错误：使用变量存储函数
var greet = (String name) => print('Hello, $name');
```

**正确写法**：

```dart
// 正确：使用函数声明
void greet(String name) => print('Hello, $name');
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.5.3 prefer_if_null_operators

**规则说明**：优先使用 `??` 操作符。

**用途**：`??` 比 `?:` 更简洁。

**错误示例**：

```dart
// 错误：使用 ?:
var name = value != null ? value : 'default';
```

**正确写法**：

```dart
// 正确：使用 ??
var name = value ?? 'default';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.5.4 prefer_initializing_formals

**规则说明**：尽可能使用初始化形式参数。

**用途**：初始化形式参数（`this.field`）更简洁。

**错误示例**：

```dart
// 错误：在构造函数体中初始化
class Person {
  String name;
  
  Person(String name) {
    this.name = name;
  }
}
```

**正确写法**：

```dart
// 正确：使用初始化形式参数
class Person {
  String name;
  
  Person(this.name);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.5.5 use_super_parameters

**规则说明**：尽可能使用超类初始化参数。

**用途**：Dart 2.17+ 引入的超类初始化参数使代码更简洁。

**错误示例**：

```dart
// 错误：手动传递参数给父类
class Animal {
  final String name;
  Animal(this.name);
}

class Dog extends Animal {
  final String breed;
  
  Dog(String name, this.breed) : super(name);
}
```

**正确写法**：

```dart
// 正确：使用超类初始化参数
class Animal {
  final String name;
  Animal(this.name);
}

class Dog extends Animal {
  final String breed;
  
  Dog(super.name, this.breed);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.6 文档注释类规则

#### 3.6.1 slash_for_doc_comments

**规则说明**：文档注释优先使用 `///`。

**用途：`///` 比 `/** */` 更简洁，是 Dart 的推荐风格。

**错误示例**：

```dart
// 错误：使用 /** */
/**
 * 这是一个类
 */
class MyClass { }
```

**正确写法**：

```dart
// 正确：使用 ///
/// 这是一个类
class MyClass { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.7 类型注解类规则

#### 3.7.1 type_init_formals

**规则说明**：不要为初始化形式参数添加类型注解。

**用途**：初始化形式参数的类型已经从字段声明中推断出来，不需要重复。

**错误示例**：

```dart
// 错误：为初始化形式参数添加类型
class Person {
  String name;
  
  Person(String this.name);  // 多余
}
```

**正确写法**：

```dart
// 正确：省略类型
class Person {
  String name;
  
  Person(this.name);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.7.2 use_function_type_syntax_for_parameters

**规则说明**：对参数使用泛型函数类型语法。

**用途**：新的函数类型语法更清晰。

**错误示例**：

```dart
// 错误：使用旧的函数类型语法
void process(int compare(String a, String b)) { }
```

**正确写法**：

```dart
// 正确：使用新的函数类型语法
void process(int Function(String, String) compare) { }
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.8 空值处理类规则

#### 3.8.1 null_closures

**规则说明**：不要在期望闭包的地方传递 `null`。

**用途**：传递 `null` 作为闭包可能导致运行时错误。

**错误示例**：

```dart
// 错误：传递 null 作为闭包
list.expand(null);  // 错误！
```

**正确写法**：

```dart
// 正确：传递有效的闭包
list.expand((e) => [e, e]);
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.9 控制流类规则

#### 3.9.1 control_flow_in_finally

**规则说明**：避免在 `finally` 块中使用控制流语句。

**用途**：`finally` 块中的控制流语句（`return`、`break`、`continue`、`throw`）会覆盖 `try` 或 `catch` 块中的控制流，导致意外的行为。

**错误示例**：

```dart
// 错误：在 finally 中使用 return
int calculate() {
  try {
    return 42;
  } finally {
    return 0;  // 这会覆盖 try 中的返回值！
  }
}
```

**正确写法**：

```dart
// 正确：避免在 finally 中使用控制流
int calculate() {
  var result = 0;
  try {
    result = 42;
  } finally {
    cleanup();
  }
  return result;
}
```

#### 3.9.2 exhaustive_cases

**规则说明**：为枚举类常量定义所有 case 子句。

**用途**：确保 `switch` 语句处理枚举的所有可能值。

**错误示例**：

```dart
// 错误：缺少 case
enum Color { red, green, blue }

String getColorName(Color color) {
  switch (color) {
    case Color.red:
      return '红色';
    case Color.green:
      return '绿色';
    // 缺少 blue 的处理！
  }
}
```

**正确写法**：

```dart
// 正确：处理所有 case
String getColorName(Color color) {
  switch (color) {
    case Color.red:
      return '红色';
    case Color.green:
      return '绿色';
    case Color.blue:
      return '蓝色';
  }
}

// 或者使用 default
String getColorName(Color color) {
  switch (color) {
    case Color.red:
      return '红色';
    case Color.green:
      return '绿色';
    default:
      return '其他颜色';
  }
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.10 导入和库组织类规则

#### 3.10.1 implementation_imports

**规则说明**：不要从另一个包导入实现文件。

**用途**：实现文件（通常位于 `lib/src/`）是包的私有 API，不应该被外部导入。

**错误示例**：

```dart
// 错误：导入实现文件
import 'package:some_package/src/internal.dart';
```

**正确写法**：

```dart
// 正确：导入公共 API
import 'package:some_package/some_package.dart';
```

#### 3.10.2 library_private_types_in_public_api

**规则说明**：避免在公共 API 中使用私有类型。

**用途**：公共 API 中不应该暴露私有类型（以下划线开头的类型）。

**错误示例**：

```dart
// 错误：公共 API 返回私有类型
class _InternalClass { }

_InternalClass getValue() => _InternalClass();  // 错误！
```

**正确写法**：

```dart
// 正确：公共 API 使用公共类型
class PublicClass { }

PublicClass getValue() => PublicClass();
```

#### 3.10.3 invalid_runtime_check_with_js_interop_types

**规则说明**：避免对 JS 互操作类型进行运行时类型检查。

**用途**：JS 互操作类型的运行时类型检查结果在不同平台上可能不一致。

**错误示例**：

```dart
// 错误：对 JS 互操作类型进行类型检查
import 'dart:js_interop';

void check(JSObject obj) {
  if (obj is String) {  // 结果可能不一致
    print('是字符串');
  }
}
```

**正确写法**：

```dart
// 正确：使用其他方式处理
import 'dart:js_interop';

void process(JSObject obj) {
  // 使用 JS 互操作 API 处理
}
```

### 3.11 递归和循环引用类规则

#### 3.11.1 recursive_getters

**规则说明**：属性 getter 不应该递归返回自身。

**用途**：递归 getter 会导致无限循环。

**错误示例**：

```dart
// 错误：递归 getter
class Person {
  String get name => name;  // 无限递归！
}
```

**正确写法**：

```dart
// 正确：使用字段存储值
class Person {
  final String name;
  Person(this.name);
}
```

### 3.12 不必要的代码类规则

这类规则帮助识别和移除不必要的代码。

#### 3.12.1 unnecessary_brace_in_string_interps

**规则说明**：在不需要时避免在插值中使用花括号。

**用途**：简单变量插值不需要花括号。

**错误示例**：

```dart
// 错误：不必要的花括号
var greeting = 'Hello, ${name}!';
```

**正确写法**：

```dart
// 正确：省略花括号
var greeting = 'Hello, $name!';

// 需要时才使用花括号
var message = 'Hello, ${person.name}!';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.2 unnecessary_const

**规则说明**：避免不必要的 `const` 关键字。

**用途**：在常量上下文中，`const` 是多余的。

**错误示例**：

```dart
// 错误：多余的 const
const list = const [1, 2, 3];
```

**正确写法**：

```dart
// 正确：省略 const
const list = [1, 2, 3];
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.3 unnecessary_constructor_name

**规则说明**：避免不必要的 `.new` 构造函数名。

**用途**：`.new` 是多余的，直接使用类名即可。

**错误示例**：

```dart
// 错误：使用 .new
var person = Person.new('张三');
```

**正确写法**：

```dart
// 正确：直接使用类名
var person = Person('张三');
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.4 unnecessary_getters_setters

**规则说明**：避免只是为了"安全"而将字段包装在 getter 和 setter 中。

**用途**：简单的 getter/setter 包装是多余的，直接使用公共字段即可。

**错误示例**：

```dart
// 错误：不必要的 getter/setter
class Person {
  String _name;
  
  String get name => _name;
  set name(String value) => _name = value;
}
```

**正确写法**：

```dart
// 正确：直接使用公共字段
class Person {
  String name;
  
  Person(this.name);
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.5 unnecessary_late

**规则说明**：在不需要时不要指定 `late` 修饰符。

**用途：`late` 只在特定情况下需要，不必要的 `late` 可能导致运行时错误。

**错误示例**：

```dart
// 错误：不必要的 late
class Person {
  late final String name = '张三';  // 立即初始化，不需要 late
}
```

**正确写法**：

```dart
// 正确：移除 late
class Person {
  final String name = '张三';
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.6 unnecessary_library_name

**规则说明**：不要在 `library` 声明中包含库名。

**用途**：库名是遗留特性，现代 Dart 代码不需要。

**错误示例**：

```dart
// 错误：包含库名
library my_library;
```

**正确写法**：

```dart
// 正确：省略库名
library;

// 或者完全省略 library 声明
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.7 unnecessary_new

**规则说明**：避免不必要的 `new` 关键字。

**用途**：Dart 2 之后，`new` 是可选的。

**错误示例**：

```dart
// 错误：使用 new
var person = new Person('张三');
```

**正确写法**：

```dart
// 正确：省略 new
var person = Person('张三');
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.8 unnecessary_null_aware_assignments

**规则说明**：避免在空感知赋值中使用 `null`。

**用途：`value ??= null` 是多余的。

**错误示例**：

```dart
// 错误：多余的 null 感知赋值
value ??= null;
```

**正确写法**：

```dart
// 这行代码没有任何作用，应该移除
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.9 unnecessary_null_in_if_null_operators

**规则说明**：避免在 `??` 操作符中使用 `null`。

**用途：`value ?? null` 等同于 `value`。

**错误示例**：

```dart
// 错误：多余的 null
var result = value ?? null;
```

**正确写法**：

```dart
// 正确：直接使用值
var result = value;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.10 unnecessary_nullable_for_final_variable_declarations

**规则说明**：对用非空值初始化的 final 变量使用非空类型。

**用途**：如果 final 变量用非空值初始化，它应该是非空类型。

**错误示例**：

```dart
// 错误：不必要的可空类型
final String? name = '张三';  // 用非空值初始化，应该是非空类型
```

**正确写法**：

```dart
// 正确：使用非空类型
final String name = '张三';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.11 unnecessary_string_escapes

**规则说明**：移除字符串中不必要的反斜杠。

**用途**：不必要的转义会降低可读性。

**错误示例**：

```dart
// 错误：不必要的转义
var message = 'It\'s a nice day';
```

**正确写法**：

```dart
// 正确：使用双引号避免转义
var message = "It's a nice day";

// 或者使用原始字符串
var message = r'It\'s a nice day';
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.12 unnecessary_string_interpolations

**规则说明**：避免不必要的字符串插值。

**用途：`'${value}'` 等同于 `value.toString()`，通常是不必要的。

**错误示例**：

```dart
// 错误：不必要的插值
var message = '${value}';
```

**正确写法**：

```dart
// 正确：直接使用值
var message = value;

// 或者如果需要 toString
var message = value.toString();
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.13 unnecessary_this

**规则说明**：除非避免遮蔽，否则不要使用 `this` 访问成员。

**用途：`this` 在大多数情况下是多余的。

**错误示例**：

```dart
// 错误：多余的 this
class Person {
  String name;
  
  void greet() {
    print('Hello, ${this.name}');  // 多余
  }
}
```

**正确写法**：

```dart
// 正确：省略 this
class Person {
  String name;
  
  void greet() {
    print('Hello, $name');
  }
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.14 unnecessary_to_list_in_spreads

**规则说明**：避免在展开操作中使用不必要的 `toList()`。

**用途**：展开操作符可以直接展开 `Iterable`，不需要先转换为 `List`。

**错误示例**：

```dart
// 错误：不必要的 toList()
var list = [...items.where((e) => e > 0).toList()];
```

**正确写法**：

```dart
// 正确：直接展开
var list = [...items.where((e) => e > 0)];
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.12.15 unnecessary_underscores

**规则说明**：可以移除不必要的下划线。

**用途：命名中包含不必要的下划线会降低可读性。

**错误示例**：

```dart
// 错误：不必要的下划线
var __value = 42;
```

**正确写法**：

```dart
// 正确：移除多余下划线
var value = 42;
```

**自动修复**：✅ 支持 `dart fix` 自动修复

### 3.13 现代 Dart 特性类规则

#### 3.13.1 use_null_aware_elements

**规则说明**：测试 null 的 if 元素可以替换为空感知元素。

**用途**：Dart 3.0+ 引入的空感知元素使代码更简洁。

**错误示例**：

```dart
// 错误：使用 if 测试 null
var list = [if (value != null) value];
```

**正确写法**：

```dart
// 正确：使用空感知元素
var list = [?value];
```

**自动修复**：✅ 支持 `dart fix` 自动修复

#### 3.13.2 use_rethrow_when_possible

**规则说明**：尽可能使用 `rethrow` 重新抛出捕获的异常。

**用途：`rethrow` 保留原始堆栈跟踪，比 `throw e` 更好。

**错误示例**：

```dart
// 错误：使用 throw e
try {
  riskyOperation();
} catch (e) {
  print('出错了: $e');
  throw e;  // 会丢失原始堆栈跟踪
}
```

**正确写法**：

```dart
// 正确：使用 rethrow
try {
  riskyOperation();
} catch (e) {
  print('出错了: $e');
  rethrow;  // 保留原始堆栈跟踪
}
```

**自动修复**：✅ 支持 `dart fix` 自动修复

## 第4章 Lints 实战应用

### 4.1 配置最佳实践

#### 4.1.1 选择合适的规则集

对于不同的项目类型，建议采用以下配置策略：

**纯 Dart 库/应用：**

```yaml
# analysis_options.yaml
include: package:lints/recommended.yaml

linter:
  rules:
    # 根据需要启用额外规则
    prefer_single_quotes: true
    avoid_print: true
```

**Flutter 应用：**

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # 根据需要调整
    avoid_print: false  # 开发阶段允许 print
```

**高度严格的配置（适合大型团队）：**

```yaml
# analysis_options.yaml
include: package:lints/recommended.yaml

analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true

linter:
  rules:
    # 启用更多严格规则
    always_declare_return_types: true
    prefer_single_quotes: true
    avoid_print: true
    prefer_final_locals: true
    prefer_final_in_for_each: true
```

Dart 框架小知识

**严格模式（Strict Modes）**

从 Dart 2.17 开始，可以在 `analysis_options.yaml` 中启用三种严格模式：

1. **strict-casts**：禁止隐式类型转换
2. **strict-inference**：要求类型推断有明确结果
3. **strict-raw-types**：禁止原始类型（如 `List` 而不是 `List<String>`）

启用这些模式可以捕获更多潜在的类型问题，但也会增加一些样板代码。建议在大型项目或库中启用。

### 4.2 常见问题与解决方案

#### 4.2.1 如何处理遗留代码

对于有大量遗留代码的项目，不建议一次性启用所有规则。可以采用渐进式的方法：

1. **从 Core 规则开始**：先启用 `package:lints/core.yaml`
2. **逐步增加规则**：一次添加几条规则，修复后再继续
3. **使用忽略注释**：对于暂时无法修复的代码，使用 `// ignore:` 注释
4. **分模块启用**：对不同模块采用不同的严格程度

#### 4.2.2 与代码生成工具的协作

使用 `freezed`、`json_serializable` 等代码生成工具时，生成的代码可能不符合 lint 规则。可以在 `analysis_options.yaml` 中排除这些文件：

```yaml
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/*.mocks.dart"
```

#### 4.2.3 团队代码审查中的 Lints

在团队开发中，建议：

1. **统一配置**：将 `analysis_options.yaml` 纳入版本控制
2. **CI/CD 集成**：在持续集成中运行 `dart analyze`，将警告视为错误
3. **预提交钩子**：使用 git 预提交钩子自动检查代码

```yaml
# .github/workflows/dart.yml
name: Dart CI

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: dart-lang/setup-dart@v1
      - run: dart pub get
      - run: dart analyze --fatal-infos
```

### 4.3 使用 dart fix 自动修复

Dart 提供了 `dart fix` 命令来自动修复许多 lint 警告。使用方法：

```bash
# 预览可以修复的问题（不实际修改）
dart fix --dry-run

# 应用自动修复
dart fix --apply
```

**支持的修复类型：**

- 添加 `@override` 注解
- 移除不必要的 `new`、`const`
- 简化字符串插值
- 转换集合操作
- 修复命名规范问题
- 等等

**注意事项：**

- 不是所有问题都能自动修复
- 自动修复前建议先提交代码或使用版本控制
- 修复后应该运行测试确保功能正常

## 附录 A：Core Lints 规则速查表

| 规则名 | 说明 | 自动修复 |
|--------|------|----------|
| avoid_empty_else | 避免空的 else 子句 | ✅ |
| avoid_relative_lib_imports | 避免相对导入 lib/ 下的文件 | ✅ |
| avoid_shadowing_type_parameters | 避免遮蔽类型参数 | ❌ |
| avoid_types_as_parameter_names | 避免使用类型名作为参数名 | ❌ |
| await_only_futures | 只对 Future 使用 await | ✅ |
| camel_case_extensions | 扩展名使用 UpperCamelCase | ❌ |
| camel_case_types | 类型名使用 UpperCamelCase | ❌ |
| collection_methods_unrelated_type | 检测集合方法的不相关类型参数 | ❌ |
| curly_braces_in_flow_control_structures | 流程控制结构使用花括号 | ✅ |
| dangling_library_doc_comments | 库文档注释附加到库指令 | ✅ |
| depend_on_referenced_packages | 依赖被引用的包 | ❌ |
| empty_catches | 避免空的 catch 块 | ✅ |
| file_names | 文件名使用 lowercase_with_underscores | ❌ |
| hash_and_equals | 重写 == 时同时重写 hashCode | ✅ |
| implicit_call_tearoffs | 显式 tear-off call 方法 | ✅ |
| library_annotations | 库注解附加到库指令 | ✅ |
| no_duplicate_case_values | 避免重复的 case 值 | ✅ |
| no_wildcard_variable_uses | 不使用通配符变量 | ❌ |
| non_constant_identifier_names | 非常量标识符使用 lowerCamelCase | ✅ |
| null_check_on_nullable_type_parameter | 不对可空类型参数进行空检查 | ✅ |
| prefer_generic_function_type_aliases | 优先使用泛型函数类型别名 | ✅ |
| prefer_is_empty | 使用 isEmpty | ✅ |
| prefer_is_not_empty | 使用 isNotEmpty | ✅ |
| prefer_iterable_whereType | 优先使用 whereType | ✅ |
| prefer_typing_uninitialized_variables | 为未初始化变量指定类型 | ✅ |
| provide_deprecation_message | 提供弃用消息 | ❌ |
| secure_pubspec_urls | 使用安全的 pubspec URL | ❌ |
| strict_top_level_inference | 指定顶级类型注解 | ✅ |
| type_literal_in_constant_pattern | 不在常量模式中使用类型字面量 | ✅ |
| unintended_html_in_doc_comment | 避免文档注释中的 HTML | ❌ |
| unnecessary_overrides | 避免不必要的重写 | ✅ |
| unrelated_type_equality_checks | 检测不相关类型的相等性比较 | ❌ |
| use_string_in_part_of_directives | 在 part of 中使用字符串 | ✅ |
| valid_regexps | 使用有效的正则表达式 | ❌ |
| void_checks | 不给 void 赋值 | ❌ |

## 附录 B：Recommended Lints 规则速查表

| 规则名 | 说明 | 自动修复 |
|--------|------|----------|
| annotate_overrides | 为重写成员添加 @override | ✅ |
| avoid_function_literals_in_foreach_calls | 避免在 forEach 中使用函数字面量 | ✅ |
| avoid_init_to_null | 不显式初始化为 null | ✅ |
| avoid_renaming_method_parameters | 不重命名被重写方法的参数 | ✅ |
| avoid_return_types_on_setters | setter 不指定返回类型 | ✅ |
| avoid_returning_null_for_void | 不为 void 返回 null | ✅ |
| avoid_single_cascade_in_expression_statements | 避免单个级联操作 | ✅ |
| constant_identifier_names | 常量名使用 lowerCamelCase | ✅ |
| control_flow_in_finally | 避免 finally 中的控制流 | ❌ |
| empty_constructor_bodies | 空构造函数体使用 ; | ✅ |
| empty_statements | 避免空语句 | ✅ |
| exhaustive_cases | 处理枚举的所有 case | ✅ |
| implementation_imports | 不导入实现文件 | ❌ |
| invalid_runtime_check_with_js_interop_types | 避免对 JS 互操作类型进行类型检查 | ❌ |
| library_prefixes | 库前缀使用 lowercase_with_underscores | ❌ |
| library_private_types_in_public_api | 公共 API 不使用私有类型 | ❌ |
| no_leading_underscores_for_library_prefixes | 库前缀不使用前导下划线 | ✅ |
| no_leading_underscores_for_local_identifiers | 局部标识符不使用前导下划线 | ✅ |
| null_closures | 不传递 null 作为闭包 | ✅ |
| overridden_fields | 不重写字段 | ❌ |
| package_names | 包名使用 lowercase_with_underscores | ❌ |
| prefer_adjacent_string_concatenation | 使用相邻字符串连接 | ✅ |
| prefer_collection_literals | 使用集合字面量 | ✅ |
| prefer_conditional_assignment | 使用 ??= | ✅ |
| prefer_contains | 使用 contains | ✅ |
| prefer_final_fields | 私有字段可以是 final | ✅ |
| prefer_for_elements_to_map_fromIterable | 使用 for 元素构建 Map | ✅ |
| prefer_function_declarations_over_variables | 使用函数声明 | ✅ |
| prefer_if_null_operators | 使用 ?? | ✅ |
| prefer_initializing_formals | 使用初始化形式参数 | ✅ |
| prefer_inlined_adds | 内联列表项声明 | ✅ |
| prefer_interpolation_to_compose_strings | 使用字符串插值 | ✅ |
| prefer_is_not_operator | 使用 is! | ✅ |
| prefer_null_aware_operators | 使用空感知操作符 | ✅ |
| prefer_spread_collections | 使用展开集合 | ✅ |
| recursive_getters | 避免递归 getter | ❌ |
| slash_for_doc_comments | 文档注释使用 /// | ✅ |
| type_init_formals | 初始化形式参数不加类型 | ✅ |
| unnecessary_brace_in_string_interps | 避免不必要的插值花括号 | ✅ |
| unnecessary_const | 避免不必要的 const | ✅ |
| unnecessary_constructor_name | 避免 .new | ✅ |
| unnecessary_getters_setters | 避免不必要的 getter/setter | ✅ |
| unnecessary_late | 避免不必要的 late | ✅ |
| unnecessary_library_name | 避免库名 | ✅ |
| unnecessary_new | 避免 new | ✅ |
| unnecessary_null_aware_assignments | 避免 ??= null | ✅ |
| unnecessary_null_in_if_null_operators | 避免 ?? null | ✅ |
| unnecessary_nullable_for_final_variable_declarations | final 变量使用非空类型 | ✅ |
| unnecessary_string_escapes | 移除不必要的转义 | ✅ |
| unnecessary_string_interpolations | 避免不必要的插值 | ✅ |
| unnecessary_this | 避免不必要的 this | ✅ |
| unnecessary_to_list_in_spreads | 避免展开中的 toList | ✅ |
| unnecessary_underscores | 移除不必要的下划线 | ✅ |
| use_function_type_syntax_for_parameters | 使用函数类型语法 | ✅ |
| use_null_aware_elements | 使用空感知元素 | ✅ |
| use_rethrow_when_possible | 使用 rethrow | ✅ |
| use_super_parameters | 使用超类初始化参数 | ✅ |

## 附录 C：参考资源

- [Dart Lints 官方文档](https://pub.dev/packages/lints)
- [Dart Linter 规则完整列表](https://dart.dev/tools/linter-rules)
- [Dart 静态分析配置指南](https://dart.dev/tools/analysis)
- [Effective Dart](https://dart.dev/effective-dart)
- [Dart 官方 GitHub 仓库](https://github.com/dart-lang/core/tree/main/pkgs/lints)
