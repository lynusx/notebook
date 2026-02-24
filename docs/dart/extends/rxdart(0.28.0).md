# RxDart 库详解

> 本文基于 rxdart: ^0.28.0 版本，深入解析这个为 Dart Streams 提供响应式扩展的强大库。RxDart 通过丰富的操作符和特殊的 Subject 类型，让你能够以声明式的方式处理异步数据流，构建优雅、可组合的异步程序。

---

## 目录

1. [简介与安装](#1-简介与安装)
2. [核心概念：Dart Streams vs RxDart](#2-核心概念dart-streams-vs-rxdart)
3. [Subjects：增强的 StreamController](#3-subjects增强的-streamcontroller)
4. [创建型操作符](#4-创建型操作符)
5. [转换型操作符](#5-转换型操作符)
6. [过滤型操作符](#6-过滤型操作符)
7. [组合型操作符](#7-组合型操作符)
8. [高阶映射操作符](#8-高阶映射操作符)
9. [缓冲与窗口操作符](#9-缓冲与窗口操作符)
10. [错误处理操作符](#10-错误处理操作符)
11. [工具操作符](#11-工具操作符)
12. [实战应用示例](#12-实战应用示例)
13. [附录：速查表](#13-附录速查表)

---

## 1. 简介与安装

### 1.1 什么是 RxDart？

RxDart 是一个基于 Dart Streams 构建的响应式编程库，它提供了丰富的操作符（operators）来转换、过滤、组合和操作异步数据流。RxDart 的设计灵感来自于 ReactiveX（Rx）规范，这是跨多种编程语言的响应式编程标准。

```dart
import 'package:rxdart/rxdart.dart';

void main() {
  // 使用 RxDart 操作符链式处理数据流
  Stream.fromIterable([1, 2, 3, 4, 5])
    .where((n) => n.isOdd)      // 过滤奇数
    .map((n) => n * n)          // 平方
    .listen(print);             // 输出: 1, 9, 25
}
```

### 1.2 安装

在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  rxdart: ^0.28.0
```

然后运行安装命令：

```bash
# Dart 项目
dart pub get

# Flutter 项目
flutter pub get
```

### 1.3 RxDart 与 Dart Streams 的关系

RxDart **不是**要替代 Dart 的 Streams，而是对其进行增强：

| 特性 | Dart Streams | RxDart |
|------|-------------|--------|
| 基础功能 | ✅ 完整支持 | ✅ 完全兼容 |
| 操作符 | ⚠️ 基础操作符 | ✅ 50+ 高级操作符 |
| Subjects | ❌ 无 | ✅ BehaviorSubject, ReplaySubject |
| 错误处理 | ⚠️ 基础 | ✅ 丰富的错误恢复操作符 |
| 组合能力 | ⚠️ 有限 | ✅ 强大的流组合操作符 |

---

## 2. 核心概念：Dart Streams vs RxDart

### 2.1 冷流与热流

```dart
// 冷流（Cold Stream）：每个订阅者获得独立的数据流
final coldStream = Stream.fromIterable([1, 2, 3]);
coldStream.listen(print);  // 订阅者1: 1, 2, 3
coldStream.listen(print);  // 订阅者2: 1, 2, 3（独立的数据流）

// 热流（Hot Stream）：多个订阅者共享同一数据流
final hotStream = BehaviorSubject<int>();
hotStream.add(1);
hotStream.stream.listen(print);  // 订阅者1
hotStream.stream.listen(print);  // 订阅者2（共享数据）
```

### 2.2 RxDart 与标准 Rx 的区别

| 情况 | Rx Observables | Dart Streams |
|------|----------------|--------------|
| 错误处理 | Observable 终止并抛出错误 | 错误被发射，Stream 继续 |
| 冷流订阅 | 多个订阅者可以监听同一个冷 Observable，每个订阅获得独立数据流 | 单订阅者 |
| 热流 | 支持 | 支持（Broadcast Streams） |
| 默认同步 | 是 | 否 |
| 支持暂停/恢复 | 否 | 是 |
| 能否发射 null | RxJava 除外：可以 | 可以 |

---

## 3. Subjects：增强的 StreamController

Subjects 既是 Observable（可被订阅）又是 Observer（可以接收事件）。RxDart 提供了两种特殊的 Subject 类型。

### 3.1 PublishSubject

`PublishSubject` 是一个广播 StreamController，只向订阅者发送订阅**之后**产生的事件。

```dart
import 'package:rxdart/rxdart.dart';

void main() {
  final subject = PublishSubject<String>();
  
  // 在订阅前发送的事件不会被接收
  subject.add('Event 1');
  subject.add('Event 2');
  
  // 订阅
  subject.stream.listen(
    (data) => print('订阅者1: $data'),
    onError: (err) => print('错误: $err'),
    onDone: () => print('完成'),
  );
  
  // 订阅后发送的事件会被接收
  subject.add('Event 3');  // 订阅者1: Event 3
  subject.add('Event 4');  // 订阅者1: Event 4
  
  // 添加第二个订阅者
  subject.stream.listen(
    (data) => print('订阅者2: $data'),
  );
  
  subject.add('Event 5');  // 两个订阅者都会收到
  
  // 关闭 Subject
  subject.close();
}
```

**适用场景**：实时事件流（用户输入、网络事件、通知）。

### 3.2 BehaviorSubject

`BehaviorSubject` 是一个广播 StreamController，它会缓存**最新的值或错误**。当新订阅者订阅时，会立即收到这个最新值。

```dart
void behaviorSubjectDemo() {
  final subject = BehaviorSubject<String>.seeded('初始值');
  
  // 发送一些事件
  subject.add('Hello');
  subject.add('World');
  
  // 新订阅者会立即收到最新值 'World'
  subject.stream.listen((data) {
    print('订阅者1: $data');  // 立即输出: World
  });
  
  subject.add('RxDart');  // 订阅者1: RxDart
  
  // 获取当前值（同步）
  print('当前值: ${subject.value}');  // RxDart
  
  // 获取是否有值
  print('hasValue: ${subject.hasValue}');  // true
  
  // 获取值流
  subject.valueStream.listen((value) {
    print('值流: $value');
  });
  
  subject.close();
}
```

**BehaviorSubject 的核心特性**：

```dart
void behaviorSubjectFeatures() {
  final subject = BehaviorSubject<int>();
  
  // 1. seed 构造函数 - 提供初始值
  final seeded = BehaviorSubject<int>.seeded(0);
  print(seeded.value);  // 0
  
  // 2. 同步获取当前值
  subject.add(42);
  print(subject.value);  // 42
  
  // 3. 获取最近的错误
  subject.addError(Exception('Test Error'));
  if (subject.hasError) {
    print('错误: ${subject.error}');
    print('堆栈: ${subject.stackTrace}');
  }
  
  // 4. 值流 - 只发射值，不发射错误
  subject.valueStream.listen(print);
  
  subject.close();
}
```

**适用场景**：状态管理（当前用户、应用配置、表单值）。

### 3.3 ReplaySubject

`ReplaySubject` 是一个广播 StreamController，它会缓存**所有发送过的事件**（或指定数量的事件），并在新订阅者订阅时重放这些事件。

```dart
void replaySubjectDemo() {
  // 无界缓存 - 缓存所有事件
  final unbounded = ReplaySubject<String>();
  unbounded.add('A');
  unbounded.add('B');
  unbounded.add('C');
  
  unbounded.stream.listen((data) {
    print('无界: $data');  // A, B, C
  });
  
  // 有界缓存 - 只缓存最近 N 个事件
  final bounded = ReplaySubject<String>(maxSize: 2);
  bounded.add('X');
  bounded.add('Y');
  bounded.add('Z');  // X 被挤出缓存
  
  bounded.stream.listen((data) {
    print('有界: $data');  // Y, Z
  });
  
  unbounded.close();
  bounded.close();
}
```

**适用场景**：需要回放历史数据的场景（聊天记录、配置变更日志）。

### 3.4 Subject 对比表

| Subject 类型 | 缓存策略 | 新订阅者行为 | 适用场景 |
|-------------|----------|-------------|----------|
| `PublishSubject` | 不缓存 | 只接收订阅后的事件 | 实时事件流 |
| `BehaviorSubject` | 缓存最新值 | 立即接收最新值 | 状态管理 |
| `ReplaySubject` | 缓存所有/部分 | 接收所有缓存的事件 | 历史回放 |

---

## 4. 创建型操作符

### 4.1 Rx.combineLatest

组合多个 Stream 的最新值，当任意一个 Stream 发射值时，使用所有 Stream 的最新值调用组合函数。

```dart
void combineLatestDemo() {
  final stream1 = Stream.periodic(
    Duration(seconds: 1), 
    (i) => 'A$i',
  ).take(3);
  
  final stream2 = Stream.periodic(
    Duration(milliseconds: 1500), 
    (i) => 'B$i',
  ).take(3);
  
  Rx.combineLatest2(
    stream1,
    stream2,
    (a, b) => '($a, $b)',
  ).listen(print);
  
  // 输出:
  // (A0, B0)
  // (A1, B0)
  // (A1, B1)
  // (A2, B1)
  // (A2, B2)
}
```

### 4.2 Rx.merge

将多个 Stream 合并成一个 Stream，按时间顺序交错发射所有事件。

```dart
void mergeDemo() {
  final stream1 = Stream.periodic(
    Duration(seconds: 1), 
    (i) => 'A$i',
  ).take(3);
  
  final stream2 = Stream.periodic(
    Duration(milliseconds: 1500), 
    (i) => 'B$i',
  ).take(3);
  
  Rx.merge([stream1, stream2]).listen(print);
  
  // 输出（按时间顺序）:
  // A0
  // B0
  // A1
  // B1
  // A2
  // B2
}
```

### 4.3 Rx.zip

将多个 Stream 按索引配对，只有当所有 Stream 都有对应位置的值时才发射。

```dart
void zipDemo() {
  final names = Stream.fromIterable(['Alice', 'Bob', 'Charlie']);
  final ages = Stream.fromIterable([25, 30, 35]);
  final cities = Stream.fromIterable(['NYC', 'LA', 'Chicago']);
  
  Rx.zip3(
    names,
    ages,
    cities,
    (name, age, city) => '$name, $age岁, 来自$city',
  ).listen(print);
  
  // 输出:
  // Alice, 25岁, 来自NYC
  // Bob, 30岁, 来自LA
  // Charlie, 35岁, 来自Chicago
}
```

### 4.4 RetryStream

当源 Stream 发生错误时，自动重试指定次数。

```dart
void retryStreamDemo() {
  var attempt = 0;
  
  RetryStream(
    () {
      attempt++;
      print('尝试 $attempt');
      if (attempt < 3) {
        return Stream<int>.error(Exception('失败'));
      }
      return Stream.value(42);
    },
    3,  // 最大重试次数
  ).listen(
    (data) => print('成功: $data'),
    onError: (e) => print('最终失败: $e'),
  );
  
  // 输出:
  // 尝试 1
  // 尝试 2
  // 尝试 3
  // 成功: 42
}
```

---

## 5. 转换型操作符

### 5.1 map

将每个事件转换为另一种类型。

```dart
void mapDemo() {
  Stream.fromIterable([1, 2, 3, 4, 5])
    .map((n) => n * n)
    .listen(print);  // 1, 4, 9, 16, 25
}
```

### 5.2 mapTo

将每个事件映射为固定值。

```dart
void mapToDemo() {
  Stream.fromIterable([1, 2, 3])
    .mapTo('固定值')
    .listen(print);  // 固定值, 固定值, 固定值
}
```

### 5.3 scan

累积计算，类似于 `reduce`，但会发射中间结果。

```dart
void scanDemo() {
  // 累加求和
  Stream.fromIterable([1, 2, 3, 4, 5])
    .scan((acc, value, index) => acc + value, 0)
    .listen(print);  // 1, 3, 6, 10, 15
  
  // 构建列表
  Stream.fromIterable(['a', 'b', 'c'])
    .scan((list, value, index) => [...list, value], <String>[])
    .listen(print);  // [a], [a, b], [a, b, c]
}
```

### 5.4 asyncMap

异步映射，每个事件通过异步函数转换。

```dart
void asyncMapDemo() async {
  Stream.fromIterable([1, 2, 3])
    .asyncMap((n) async {
      await Future.delayed(Duration(milliseconds: 100));
      return n * 2;
    })
    .listen(print);  // 2, 4, 6（每个间隔100ms）
}
```

### 5.5 expand

将每个事件展开为多个事件。

```dart
void expandDemo() {
  Stream.fromIterable([1, 2, 3])
    .expand((n) => [n, n * 10])
    .listen(print);  // 1, 10, 2, 20, 3, 30
}
```

### 5.6 groupBy

按指定键对事件进行分组。

```dart
void groupByDemo() {
  final users = [
    {'name': 'Alice', 'department': 'Engineering'},
    {'name': 'Bob', 'department': 'Sales'},
    {'name': 'Charlie', 'department': 'Engineering'},
    {'name': 'Diana', 'department': 'Sales'},
  ];
  
  Stream.fromIterable(users)
    .groupBy((user) => user['department']!)
    .asyncExpand((group) async* {
      print('部门: ${group.key}');
      yield* group.map((u) => u['name']!);
    })
    .listen(print);
  
  // 输出:
  // 部门: Engineering
  // Alice
  // Charlie
  // 部门: Sales
  // Bob
  // Diana
}
```

---

## 6. 过滤型操作符

### 6.1 where

只发射满足条件的事件。

```dart
void whereDemo() {
  Stream.fromIterable([1, 2, 3, 4, 5, 6])
    .where((n) => n.isEven)
    .listen(print);  // 2, 4, 6
}
```

### 6.2 distinct

过滤掉重复的事件（只保留与前一个不同的事件）。

```dart
void distinctDemo() {
  Stream.fromIterable([1, 1, 2, 2, 2, 3, 1])
    .distinct()
    .listen(print);  // 1, 2, 3, 1
}
```

### 6.3 distinctUnique

过滤掉整个流中已经出现过的事件。

```dart
void distinctUniqueDemo() {
  Stream.fromIterable([1, 1, 2, 2, 2, 3, 1])
    .distinctUnique()
    .listen(print);  // 1, 2, 3（最后的1被过滤）
}
```

### 6.4 skip 和 skipWhile

跳过前 N 个事件或满足条件的事件。

```dart
void skipDemo() {
  // 跳过前3个
  Stream.fromIterable([1, 2, 3, 4, 5])
    .skip(3)
    .listen(print);  // 4, 5
  
  // 跳过小于10的
  Stream.fromIterable([2, 5, 8, 12, 15, 3])
    .skipWhile((n) => n < 10)
    .listen(print);  // 12, 15, 3
}
```

### 6.5 take 和 takeWhile

只取前 N 个事件或满足条件的事件。

```dart
void takeDemo() {
  // 取前3个
  Stream.fromIterable([1, 2, 3, 4, 5])
    .take(3)
    .listen(print);  // 1, 2, 3
  
  // 取小于10的
  Stream.fromIterable([2, 5, 8, 12, 15])
    .takeWhile((n) => n < 10)
    .listen(print);  // 2, 5, 8
}
```

### 6.6 debounceTime

防抖：等待指定时间无新事件后，发射最后一个事件。

```dart
void debounceTimeDemo() async {
  final subject = PublishSubject<String>();
  
  subject
    .debounceTime(Duration(milliseconds: 500))
    .listen((value) => print('搜索: $value'));
  
  // 模拟快速输入
  subject.add('H');
  await Future.delayed(Duration(milliseconds: 100));
  subject.add('He');
  await Future.delayed(Duration(milliseconds: 100));
  subject.add('Hel');
  await Future.delayed(Duration(milliseconds: 100));
  subject.add('Hell');
  await Future.delayed(Duration(milliseconds: 100));
  subject.add('Hello');
  
  // 等待500ms无输入后输出:
  // 搜索: Hello
}
```

### 6.7 throttleTime

节流：按固定时间间隔发射事件。

```dart
void throttleTimeDemo() async {
  final subject = PublishSubject<int>();
  
  subject
    .throttleTime(Duration(milliseconds: 300))
    .listen((value) => print('发射: $value'));
  
  // 快速发射事件
  for (var i = 0; i < 10; i++) {
    subject.add(i);
    await Future.delayed(Duration(milliseconds: 100));
  }
  
  // 输出（大约每300ms发射一个）:
  // 发射: 0
  // 发射: 3
  // 发射: 6
  // 发射: 9
}
```

### 6.8 sampleTime

定期采样，发射最近的事件。

```dart
void sampleTimeDemo() async {
  final stream = Stream.periodic(
    Duration(milliseconds: 100),
    (i) => i,
  );
  
  stream
    .sampleTime(Duration(milliseconds: 350))
    .take(5)
    .listen(print);  // 大约每350ms采样一次
}
```

---

## 7. 组合型操作符

### 7.1 mergeWith

将当前 Stream 与另一个 Stream 合并。

```dart
void mergeWithDemo() {
  final stream1 = Stream.periodic(Duration(seconds: 1), (i) => 'A$i').take(3);
  final stream2 = Stream.periodic(Duration(seconds: 2), (i) => 'B$i').take(2);
  
  stream1.mergeWith([stream2]).listen(print);
  
  // 输出:
  // A0
  // B0
  // A1
  // A2
  // B1
}
```

### 7.2 concatWith

将多个 Stream 按顺序连接。

```dart
void concatWithDemo() {
  final stream1 = Stream.fromIterable([1, 2, 3]);
  final stream2 = Stream.fromIterable([4, 5, 6]);
  final stream3 = Stream.fromIterable([7, 8, 9]);
  
  stream1.concatWith([stream2, stream3]).listen(print);
  
  // 输出: 1, 2, 3, 4, 5, 6, 7, 8, 9
}
```

### 7.3 zipWith

将当前 Stream 与另一个 Stream 按索引配对。

```dart
void zipWithDemo() {
  final names = Stream.fromIterable(['Alice', 'Bob', 'Charlie']);
  final scores = Stream.fromIterable([85, 90, 78]);
  
  names
    .zipWith(scores, (name, score) => '$name: $score分')
    .listen(print);
  
  // 输出:
  // Alice: 85分
  // Bob: 90分
  // Charlie: 78分
}
```

### 7.4 withLatestFrom

当源 Stream 发射事件时，结合另一个 Stream 的最新值。

```dart
void withLatestFromDemo() async {
  final buttonClicks = PublishSubject<void>();
  final textInput = BehaviorSubject<String>.seeded('');
  
  buttonClicks
    .withLatestFrom(textInput, (_, text) => text)
    .listen((text) => print('提交: $text'));
  
  textInput.add('Hello');
  buttonClicks.add(null);  // 提交: Hello
  
  textInput.add('World');
  textInput.add('RxDart');
  buttonClicks.add(null);  // 提交: RxDart
}
```

### 7.5 startWith

在 Stream 开始处添加初始值。

```dart
void startWithDemo() {
  Stream.fromIterable([2, 3, 4])
    .startWith(1)
    .listen(print);  // 1, 2, 3, 4
}
```

### 7.6 endWith

在 Stream 结束时添加值。

```dart
void endWithDemo() {
  Stream.fromIterable([1, 2, 3])
    .endWith(4)
    .listen(print);  // 1, 2, 3, 4
}
```

---

## 8. 高阶映射操作符

### 8.1 flatMap（mergeMap）

将每个事件映射为一个 Stream，并合并所有内部 Stream 的输出。

```dart
void flatMapDemo() {
  Stream.fromIterable([1, 2, 3])
    .flatMap((n) => Stream.fromIterable([n, n * 10]))
    .listen(print);  // 1, 10, 2, 20, 3, 30（可能交错）
}
```

### 8.2 switchMap

将每个事件映射为一个 Stream，但只保留最新的内部 Stream，取消之前的。

```dart
void switchMapDemo() async {
  final searchQueries = PublishSubject<String>();
  
  searchQueries
    .debounceTime(Duration(milliseconds: 300))
    .switchMap((query) async* {
      // 模拟 API 请求
      await Future.delayed(Duration(milliseconds: 500));
      yield '结果: $query';
    })
    .listen(print);
  
  // 快速输入
  searchQueries.add('A');
  await Future.delayed(Duration(milliseconds: 100));
  searchQueries.add('AB');
  await Future.delayed(Duration(milliseconds: 100));
  searchQueries.add('ABC');
  
  // 只有最后一个请求的结果会被发射（之前的被取消）
  // 输出: 结果: ABC
}
```

### 8.3 concatMap

将每个事件映射为一个 Stream，按顺序执行，等待前一个完成。

```dart
void concatMapDemo() async {
  Stream.fromIterable([1, 2, 3])
    .concatMap((n) async* {
      await Future.delayed(Duration(milliseconds: 100));
      yield '处理: $n';
    })
    .listen(print);
  
  // 输出（按顺序，每个间隔100ms）:
  // 处理: 1
  // 处理: 2
  // 处理: 3
}
```

### 8.4 exhaustMap

将每个事件映射为一个 Stream，但在当前内部 Stream 完成前忽略新的事件。

```dart
void exhaustMapDemo() async {
  final buttonClicks = PublishSubject<void>();
  
  buttonClicks
    .exhaustMap((_) async* {
      print('开始请求');
      await Future.delayed(Duration(seconds: 2));
      yield '请求完成';
    })
    .listen(print);
  
  // 点击按钮
  buttonClicks.add(null);
  await Future.delayed(Duration(milliseconds: 500));
  buttonClicks.add(null);  // 被忽略（前一个未完成）
  await Future.delayed(Duration(milliseconds: 500));
  buttonClicks.add(null);  // 被忽略
  
  // 2秒后:
  // 开始请求
  // 请求完成
}
```

### 8.5 高阶映射操作符对比

| 操作符 | 行为 | 适用场景 |
|--------|------|----------|
| `flatMap` | 并行处理，合并输出 | 并行请求 |
| `switchMap` | 取消前一个，保留最新 | 搜索、自动完成 |
| `concatMap` | 顺序处理，等待完成 | 顺序依赖操作 |
| `exhaustMap` | 忽略新事件直到完成 | 防重复提交 |

---

## 9. 缓冲与窗口操作符

### 9.1 bufferCount

按指定数量缓冲事件。

```dart
void bufferCountDemo() {
  Stream.fromIterable([1, 2, 3, 4, 5, 6, 7])
    .bufferCount(3)
    .listen(print);  // [1, 2, 3], [4, 5, 6], [7]
}
```

### 9.2 bufferTime

按时间间隔缓冲事件。

```dart
void bufferTimeDemo() async {
  Stream.periodic(Duration(milliseconds: 100), (i) => i)
    .take(10)
    .bufferTime(Duration(milliseconds: 250))
    .listen(print);
  
  // 输出（大约每250ms缓冲一次）:
  // [0, 1]
  // [2, 3, 4]
  // [5, 6, 7]
  // [8, 9]
}
```

### 9.3 bufferTest

按条件缓冲事件。

```dart
void bufferTestDemo() {
  Stream.fromIterable([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .bufferTest((value) => value % 3 == 0)  // 每遇到3的倍数就缓冲
    .listen(print);  // [1, 2, 3], [4, 5, 6], [7, 8, 9], [10]
}
```

### 9.4 windowCount

将事件分窗口发射，每个窗口是一个 Stream。

```dart
void windowCountDemo() async {
  await for (final window in Stream.fromIterable([1, 2, 3, 4, 5, 6]).windowCount(3)) {
    final values = await window.toList();
    print(values);  // [1, 2, 3], [4, 5, 6]
  }
}
```

### 9.5 pairwise

将相邻的两个事件配对。

```dart
void pairwiseDemo() {
  Stream.fromIterable([1, 2, 3, 4, 5])
    .pairwise()
    .listen(print);  // [1, 2], [2, 3], [3, 4], [4, 5]
}
```

---

## 10. 错误处理操作符

### 10.1 onErrorReturn

遇到错误时返回指定值并结束。

```dart
void onErrorReturnDemo() {
  Stream<int>.error(Exception('出错了'))
    .onErrorReturn(0)
    .listen(print);  // 0
}
```

### 10.2 onErrorReturnWith

遇到错误时通过函数计算返回值。

```dart
void onErrorReturnWithDemo() {
  Stream<int>.error(Exception('出错了'))
    .onErrorReturnWith((error, stackTrace) {
      print('捕获错误: $error');
      return -1;
    })
    .listen(print);  // -1
}
```

### 10.3 onErrorResumeNext

遇到错误时切换到备用 Stream。

```dart
void onErrorResumeNextDemo() {
  Stream.fromIterable([1, 2])
    .concatWith([Stream<int>.error(Exception('错误'))])
    .onErrorResumeNext(Stream.fromIterable([3, 4, 5]))
    .listen(print);  // 1, 2, 3, 4, 5
}
```

### 10.4 onErrorResume

遇到错误时通过函数创建备用 Stream。

```dart
void onErrorResumeDemo() {
  Stream.fromIterable([1, 2])
    .concatWith([Stream<int>.error(Exception('错误'))])
    .onErrorResume((error, stackTrace) {
      print('错误: $error');
      return Stream.value(-1);
    })
    .listen(print);  // 1, 2, -1
}
```

### 10.5 retry

遇到错误时重试。

```dart
void retryDemo() {
  var attempt = 0;
  
  Stream.fromIterable([1])
    .concatWith([
      Stream.error(Exception('失败'), 2),
    ])
    .doOnData((_) => attempt++)
    .retry(3)  // 最多重试3次
    .listen(
      print,
      onError: (e) => print('最终失败: $e'),
    );
}
```

---

## 11. 工具操作符

### 11.1 interval

在事件之间添加时间间隔。

```dart
void intervalDemo() async {
  Stream.fromIterable([1, 2, 3, 4, 5])
    .interval(Duration(milliseconds: 500))
    .listen((n) => print('${DateTime.now().second}: $n'));
  
  // 每个事件间隔500ms
}
```

### 11.2 delay

延迟整个 Stream 的开始。

```dart
void delayDemo() async {
  Stream.value('Hello')
    .delay(Duration(seconds: 2))
    .listen((v) => print('${DateTime.now()}: $v'));
  
  // 2秒后输出
}
```

### 11.3 timestamp

为每个事件添加时间戳。

```dart
void timestampDemo() {
  Stream.fromIterable([1, 2, 3])
    .timestamp()
    .listen((ts) {
      print('值: ${ts.value}, 时间: ${ts.timestamp}');
    });
}
```

### 11.4 timeInterval

计算事件之间的时间间隔。

```dart
void timeIntervalDemo() async {
  final subject = PublishSubject<int>();
  
  subject
    .timeInterval()
    .listen((ti) {
      print('值: ${ti.value}, 间隔: ${ti.interval}');
    });
  
  subject.add(1);
  await Future.delayed(Duration(milliseconds: 100));
  subject.add(2);
  await Future.delayed(Duration(milliseconds: 200));
  subject.add(3);
}
```

### 11.5 materialize / dematerialize

将事件包装为通知对象，或反向解包。

```dart
void materializeDemo() {
  Stream.fromIterable([1, 2, 3])
    .materialize()
    .listen((notification) {
      if (notification.isOnData) {
        print('数据: ${notification.data}');
      } else if (notification.isOnDone) {
        print('完成');
      }
    });
}
```

### 11.6 doOn 系列

在 Stream 生命周期的各个阶段执行副作用。

```dart
void doOnDemo() {
  Stream.fromIterable([1, 2, 3])
    .doOnListen(() => print('开始监听'))
    .doOnData((data) => print('收到数据: $data'))
    .doOnError((e, st) => print('错误: $e'))
    .doOnDone(() => print('完成'))
    .doOnCancel(() => print('取消订阅'))
    .listen(print);
  
  // 输出:
  // 开始监听
  // 收到数据: 1
  // 1
  // 收到数据: 2
  // 2
  // 收到数据: 3
  // 3
  // 完成
}
```

### 11.7 defaultIfEmpty

当 Stream 为空时发射默认值。

```dart
void defaultIfEmptyDemo() {
  Stream<int>.empty()
    .defaultIfEmpty(0)
    .listen(print);  // 0
}
```

### 11.8 switchIfEmpty

当 Stream 为空时切换到备用 Stream。

```dart
void switchIfEmptyDemo() {
  Stream<int>.empty()
    .switchIfEmpty(Stream.fromIterable([1, 2, 3]))
    .listen(print);  // 1, 2, 3
}
```

---

## 12. 实战应用示例

### 12.1 搜索自动完成

```dart
import 'package:rxdart/rxdart.dart';

class SearchBloc {
  final _searchQuery = PublishSubject<String>();
  final _searchResults = BehaviorSubject<List<String>>.seeded([]);
  
  Stream<List<String>> get searchResults => _searchResults.stream;
  
  void search(String query) {
    _searchQuery.add(query);
  }
  
  SearchBloc() {
    _searchQuery
      .debounceTime(Duration(milliseconds: 300))  // 防抖
      .where((query) => query.length >= 2)         // 至少2个字符
      .distinct()                                  // 去重
      .switchMap((query) => _fetchResults(query))  // 取消旧请求
      .listen(_searchResults.add);
  }
  
  Stream<List<String>> _fetchResults(String query) async* {
    // 模拟 API 请求
    await Future.delayed(Duration(milliseconds: 500));
    yield ['$query 结果1', '$query 结果2', '$query 结果3'];
  }
  
  void dispose() {
    _searchQuery.close();
    _searchResults.close();
  }
}

void main() async {
  final bloc = SearchBloc();
  
  bloc.searchResults.listen(print);
  
  bloc.search('Da');
  await Future.delayed(Duration(milliseconds: 100));
  bloc.search('Dar');
  await Future.delayed(Duration(milliseconds: 100));
  bloc.search('Dart');  // 只有这个会触发请求
  
  await Future.delayed(Duration(seconds: 2));
  bloc.dispose();
}
```

### 12.2 表单验证

```dart
import 'package:rxdart/rxdart.dart';

class FormBloc {
  final _email = BehaviorSubject<String>.seeded('');
  final _password = BehaviorSubject<String>.seeded('');
  
  // 输入
  void setEmail(String value) => _email.add(value);
  void setPassword(String value) => _password.add(value);
  
  // 验证流
  Stream<bool> get isEmailValid => _email
    .map((email) => email.contains('@') && email.contains('.'))
    .distinct();
  
  Stream<bool> get isPasswordValid => _password
    .map((password) => password.length >= 6)
    .distinct();
  
  // 组合验证
  Stream<bool> get isFormValid => Rx.combineLatest2(
    isEmailValid,
    isPasswordValid,
    (emailValid, passwordValid) => emailValid && passwordValid,
  );
  
  // 错误信息
  Stream<String?> get emailError => _email
    .map((email) {
      if (email.isEmpty) return null;
      if (!email.contains('@')) return '请输入有效的邮箱';
      return null;
    })
    .distinct();
  
  void dispose() {
    _email.close();
    _password.close();
  }
}
```

### 12.3 下拉刷新与上拉加载

```dart
import 'package:rxdart/rxdart.dart';

class PaginationBloc<T> {
  final Future<List<T>> Function(int page) fetchPage;
  
  final _refresh = PublishSubject<void>();
  final _loadMore = PublishSubject<void>();
  final _items = BehaviorSubject<List<T>>.seeded([]);
  final _isLoading = BehaviorSubject<bool>.seeded(false);
  final _hasMore = BehaviorSubject<bool>.seeded(true);
  
  int _currentPage = 0;
  
  PaginationBloc({required this.fetchPage}) {
    // 刷新
    _refresh
      .exhaustMap((_) => _fetchData(0, clear: true))
      .listen(_items.add);
    
    // 加载更多
    _loadMore
      .where((_) => !_isLoading.value && _hasMore.value)
      .exhaustMap((_) => _fetchData(_currentPage + 1))
      .listen((newItems) {
        _items.add([..._items.value, ...newItems]);
      });
  }
  
  Stream<List<T>> _fetchData(int page, {bool clear = false}) async* {
    _isLoading.add(true);
    try {
      final items = await fetchPage(page);
      _currentPage = page;
      _hasMore.add(items.isNotEmpty);
      yield clear ? items : items;
    } finally {
      _isLoading.add(false);
    }
  }
  
  void refresh() => _refresh.add(null);
  void loadMore() => _loadMore.add(null);
  
  void dispose() {
    _refresh.close();
    _loadMore.close();
    _items.close();
    _isLoading.close();
    _hasMore.close();
  }
}
```

### 12.4 网络请求重试

```dart
import 'package:rxdart/rxdart.dart';

class ApiClient {
  Stream<T> fetchWithRetry<T>(
    Future<T> Function() request, {
    int maxRetries = 3,
    Duration retryDelay = const Duration(seconds: 1),
  }) {
    return Stream.fromFuture(request())
      .retryWhen((errors, stackTraces) => errors
        .zipWith(
          Stream.fromIterable(List.generate(maxRetries, (i) => i)),
          (error, retryCount) => (error, retryCount),
        )
        .asyncExpand((record) async* {
          final (error, retryCount) = record;
          if (retryCount >= maxRetries - 1) {
            throw error;
          }
          print('重试第 ${retryCount + 1} 次...');
          await Future.delayed(retryDelay * (retryCount + 1));
          yield null;
        }));
  }
}
```

### 12.5 状态管理（BLoC 模式）

```dart
import 'package:rxdart/rxdart.dart';

// 事件
abstract class CounterEvent {}
class Increment extends CounterEvent {}
class Decrement extends CounterEvent {}

// 状态
class CounterState {
  final int count;
  final bool isLoading;
  
  CounterState({this.count = 0, this.isLoading = false});
  
  CounterState copyWith({int? count, bool? isLoading}) {
    return CounterState(
      count: count ?? this.count,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

// BLoC
class CounterBloc {
  final _events = PublishSubject<CounterEvent>();
  final _state = BehaviorSubject<CounterState>.seeded(CounterState());
  
  Stream<CounterState> get state => _state.stream;
  
  void add(CounterEvent event) => _events.add(event);
  
  CounterBloc() {
    _events
      .scan<CounterState>(
        (currentState, event, index) {
          if (event is Increment) {
            return currentState.copyWith(count: currentState.count + 1);
          } else if (event is Decrement) {
            return currentState.copyWith(count: currentState.count - 1);
          }
          return currentState;
        },
        CounterState(),
      )
      .listen(_state.add);
  }
  
  void dispose() {
    _events.close();
    _state.close();
  }
}
```

---

## 13. 附录：速查表

### 13.1 操作符分类表

| 分类 | 操作符 | 用途 |
|------|--------|------|
| **创建** | `Rx.combineLatest`, `Rx.merge`, `Rx.zip`, `RetryStream` | 创建新 Stream |
| **转换** | `map`, `mapTo`, `scan`, `asyncMap`, `expand`, `groupBy` | 转换事件 |
| **过滤** | `where`, `distinct`, `skip`, `take`, `debounceTime`, `throttleTime` | 过滤事件 |
| **组合** | `mergeWith`, `concatWith`, `zipWith`, `withLatestFrom`, `startWith` | 组合多个 Stream |
| **高阶映射** | `flatMap`, `switchMap`, `concatMap`, `exhaustMap` | 映射为 Stream |
| **缓冲** | `bufferCount`, `bufferTime`, `bufferTest`, `windowCount`, `pairwise` | 缓冲事件 |
| **错误处理** | `onErrorReturn`, `onErrorResume`, `retry` | 处理错误 |
| **工具** | `interval`, `delay`, `timestamp`, `doOnXxx`, `defaultIfEmpty` | 辅助功能 |

### 13.2 Subject 选择指南

| 需求 | 推荐 Subject | 说明 |
|------|-------------|------|
| 只关心订阅后的事件 | `PublishSubject` | 实时事件流 |
| 需要当前状态 | `BehaviorSubject` | 状态管理 |
| 需要历史记录 | `ReplaySubject` | 聊天记录等 |

### 13.3 高阶映射操作符选择指南

| 场景 | 推荐操作符 | 说明 |
|------|-----------|------|
| 并行处理多个请求 | `flatMap` | 不取消之前的 |
| 搜索自动完成 | `switchMap` | 取消旧请求 |
| 顺序依赖操作 | `concatMap` | 等待前一个完成 |
| 防重复提交 | `exhaustMap` | 忽略新请求直到完成 |

### 13.4 防抖与节流选择指南

| 需求 | 推荐操作符 | 说明 |
|------|-----------|------|
| 用户停止输入后触发 | `debounceTime` | 等待静默期 |
| 限制触发频率 | `throttleTime` | 固定间隔触发 |
| 定期采样 | `sampleTime` | 按时间间隔采样 |

---

## 总结

RxDart 为 Dart 开发者提供了强大的响应式编程能力：

1. **丰富的操作符**：50+ 操作符覆盖创建、转换、过滤、组合、缓冲、错误处理等场景

2. **特殊的 Subjects**：`BehaviorSubject` 和 `ReplaySubject` 提供了比标准 StreamController 更强大的功能

3. **无缝集成**：与 Dart Streams 完全兼容，可以渐进式采用

4. **声明式编程**：通过操作符链式组合，代码更易读、易维护

5. **实战验证**：广泛应用于 Flutter 状态管理、表单验证、网络请求等场景

掌握 RxDart，你将能够以更优雅、更简洁的方式处理复杂的异步数据流，构建响应式的 Dart/Flutter 应用程序。
