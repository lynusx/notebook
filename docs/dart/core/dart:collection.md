# Dart 核心库 dart:collection 详解

> 本文基于 Dart SDK 3.x 版本，深入解析 dart:collection 库中提供的扩展集合类型与工具类。这些类为标准集合（List、Map、Set）提供了强大的补充功能，包括队列、链表、有序树、不可修改视图等高级数据结构。

---

## 目录

1. [Queue（队列）](#1-queue队列)
2. [LinkedList（链表）](#2-linkedlist链表)
3. [SplayTreeMap 与 SplayTreeSet](#3-splaytreemap-与-splaytreeset)
4. [集合视图（Views）](#4-集合视图views)
5. [集合相等性比较](#5-集合相等性比较)
6. [集合基类与 Mixin](#6-集合基类与-mixin)
7. [CanonicalizedMap](#7-canonicalizedmap)
8. [实战应用示例](#8-实战应用示例)
9. [附录：速查表](#9-附录速查表)

---

## 1. Queue（队列）

队列是一种遵循**先进先出（FIFO, First-In-First-Out）**原则的数据结构。Dart 的 `dart:collection` 库提供了 `Queue` 抽象类及两种具体实现：`ListQueue` 和 `DoubleLinkedQueue`。

### 1.1 Queue 接口

`Queue<E>` 是一个抽象类，定义了队列的基本操作：

```dart
abstract class Queue<E> implements EfficientLengthIterable<E> {
  // 工厂构造函数
  factory Queue() = ListQueue<E>;
  factory Queue.of(Iterable<E> elements) = ListQueue<E>.of;
  factory Queue.from(Iterable elements) = ListQueue<E>.from;

  // 核心操作
  void add(E value);           // 添加元素到队尾
  void addAll(Iterable<E> iterable);  // 添加多个元素
  void addFirst(E value);      // 添加元素到队头（仅 DoubleLinkedQueue 高效）
  void addLast(E value);       // 添加元素到队尾

  E removeFirst();             // 移除并返回队头元素
  E removeLast();              // 移除并返回队尾元素（仅 DoubleLinkedQueue 高效）

  E get first;                 // 查看队头元素
  E get last;                  // 查看队尾元素

  void clear();                // 清空队列
  bool remove(Object? value);  // 移除指定元素
  void removeWhere(bool test(E element));  // 条件移除
  void retainWhere(bool test(E element));  // 条件保留
}
```

### 1.2 ListQueue

`ListQueue` 是基于**循环数组**实现的队列，在队尾添加和队头移除操作上具有 O(1) 的均摊时间复杂度。

#### 创建 ListQueue

```dart
import 'dart:collection';

void main() {
  // 方式1：空队列
  var queue1 = ListQueue<int>();

  // 方式2：指定初始容量
  var queue2 = ListQueue<int>(100);

  // 方式3：从可迭代对象创建
  var queue3 = ListQueue.of([1, 2, 3, 4, 5]);

  // 方式4：使用 Queue 工厂构造函数（默认创建 ListQueue）
  var queue4 = Queue<int>.from([10, 20, 30]);

  print(queue3);  // (1, 2, 3, 4, 5)
}
```

#### ListQueue 的核心特性

```dart
void main() {
  var queue = ListQueue<String>();

  // 添加元素
  queue.add('A');           // 队尾添加
  queue.addLast('B');       // 等同于 add
  queue.addFirst('Start');  // 队头添加（需要移动元素，O(n)）

  print(queue);  // (Start, A, B)

  // 移除元素
  var first = queue.removeFirst();  // 移除队头，O(1)
  print('移除: $first, 剩余: $queue');  // 移除: Start, 剩余: (A, B)

  // 查看但不移除
  print('队头: ${queue.first}');  // A
  print('队尾: ${queue.last}');   // B

  // 其他操作
  print('长度: ${queue.length}');
  print('是否为空: ${queue.isEmpty}');
  print('包含 "A": ${queue.contains('A')}');
}
```

#### ListQueue 的内部机制

`ListQueue` 使用循环数组实现，维护两个指针：`head`（队头索引）和 `tail`（队尾索引）。这种设计使得：

- `addLast()` / `removeFirst()`：O(1) 均摊时间复杂度
- `addFirst()` / `removeLast()`：O(n) 时间复杂度（需要移动元素）
- 当容量不足时，自动扩容（通常翻倍）

```dart
void demonstrateInternalMechanism() {
  // 可视化循环数组的工作方式
  var queue = ListQueue<int>(4);  // 初始容量为4

  queue.add(1);  // [1, _, _, _]  head=0, tail=1
  queue.add(2);  // [1, 2, _, _]  head=0, tail=2
  queue.add(3);  // [1, 2, 3, _]  head=0, tail=3

  queue.removeFirst();  // [_, 2, 3, _]  head=1, tail=3
  queue.removeFirst();  // [_, _, 3, _]  head=2, tail=3

  queue.add(4);  // [_, _, 3, 4]  head=2, tail=0 (循环!)
  queue.add(5);  // [5, _, 3, 4]  head=2, tail=1 (循环!)

  print(queue);  // (3, 4, 5)
}
```

### 1.3 DoubleLinkedQueue

`DoubleLinkedQueue` 是基于**双向链表**实现的队列，在队头和队尾的添加、移除操作都是 O(1)。

#### 创建 DoubleLinkedQueue

```dart
void main() {
  // 方式1：空队列
  var queue1 = DoubleLinkedQueue<String>();

  // 方式2：从可迭代对象创建
  var queue2 = DoubleLinkedQueue.of(['apple', 'banana', 'cherry']);

  print(queue2);  // (apple, banana, cherry)
}
```

#### DoubleLinkedQueue 的核心特性

```dart
void main() {
  var queue = DoubleLinkedQueue<int>();

  // 两端操作都是 O(1)
  queue.addFirst(1);  // 队头添加
  queue.addFirst(2);  // 队头添加
  queue.addLast(3);   // 队尾添加
  queue.addLast(4);   // 队尾添加

  print(queue);  // (2, 1, 3, 4)

  // 两端移除都是 O(1)
  print('移除队头: ${queue.removeFirst()}');  // 2
  print('移除队尾: ${queue.removeLast()}');    // 4
  print('剩余: $queue');  // (1, 3)
}
```

#### ListQueue vs DoubleLinkedQueue 对比

| 特性          | ListQueue              | DoubleLinkedQueue |
| ------------- | ---------------------- | ----------------- |
| 底层结构      | 循环数组               | 双向链表          |
| addFirst()    | O(n)                   | O(1)              |
| addLast()     | O(1)                   | O(1)              |
| removeFirst() | O(1)                   | O(1)              |
| removeLast()  | O(n)                   | O(1)              |
| 内存占用      | 较少（数组紧凑）       | 较多（节点开销）  |
| 随机访问      | 不支持                 | 不支持            |
| 迭代性能      | 缓存友好               | 缓存不友好        |
| 适用场景      | 主要队尾添加、队头移除 | 频繁两端操作      |

### 1.4 实战应用

#### 应用1：广度优先搜索（BFS）

队列是 BFS 算法的核心数据结构：

```dart
import 'dart:collection';

class Graph {
  final Map<int, List<int>> adjacency = {};

  void addEdge(int from, int to) {
    adjacency.putIfAbsent(from, () => []);
    adjacency.putIfAbsent(to, () => []);
    adjacency[from]!.add(to);
    adjacency[to]!.add(from);  // 无向图
  }

  // BFS 遍历
  List<int> bfs(int start) {
    var result = <int>[];
    var visited = <int>{};
    var queue = Queue<int>();

    queue.add(start);
    visited.add(start);

    while (queue.isNotEmpty) {
      var node = queue.removeFirst();
      result.add(node);

      for (var neighbor in adjacency[node] ?? []) {
        if (!visited.contains(neighbor)) {
          visited.add(neighbor);
          queue.add(neighbor);
        }
      }
    }

    return result;
  }

  // BFS 求最短路径
  List<int> shortestPath(int start, int target) {
    var visited = <int>{};
    var queue = Queue<List<int>>();  // 存储路径

    queue.add([start]);
    visited.add(start);

    while (queue.isNotEmpty) {
      var path = queue.removeFirst();
      var node = path.last;

      if (node == target) return path;

      for (var neighbor in adjacency[node] ?? []) {
        if (!visited.contains(neighbor)) {
          visited.add(neighbor);
          queue.add([...path, neighbor]);
        }
      }
    }

    return [];  // 无路径
  }
}

void main() {
  var graph = Graph();
  graph.addEdge(0, 1);
  graph.addEdge(0, 2);
  graph.addEdge(1, 3);
  graph.addEdge(2, 3);
  graph.addEdge(3, 4);

  print('BFS: ${graph.bfs(0)}');  // [0, 1, 2, 3, 4]
  print('最短路径 0->4: ${graph.shortestPath(0, 4)}');  // [0, 1, 3, 4] 或 [0, 2, 3, 4]
}
```

#### 应用2：滑动窗口最大值

使用双端队列（DoubleLinkedQueue）高效解决滑动窗口问题：

```dart
import 'dart:collection';

List<int> maxSlidingWindow(List<int> nums, int k) {
  if (nums.isEmpty || k == 0) return [];

  var result = <int>[];
  var deque = DoubleLinkedQueue<int>();  // 存储索引，保持递减顺序

  for (var i = 0; i < nums.length; i++) {
    // 移除窗口外的元素
    if (deque.isNotEmpty && deque.first <= i - k) {
      deque.removeFirst();
    }

    // 移除所有小于当前元素的索引
    while (deque.isNotEmpty && nums[deque.last] <= nums[i]) {
      deque.removeLast();
    }

    deque.addLast(i);

    // 记录窗口最大值
    if (i >= k - 1) {
      result.add(nums[deque.first]);
    }
  }

  return result;
}

void main() {
  var nums = [1, 3, -1, -3, 5, 3, 6, 7];
  var k = 3;
  print(maxSlidingWindow(nums, k));  // [3, 3, 5, 5, 6, 7]
}
```

#### 应用3：任务调度器

```dart
import 'dart:collection';

class Task {
  final String name;
  final int priority;
  final Duration duration;

  Task(this.name, this.priority, this.duration);

  @override
  String toString() => 'Task($name, P$priority)';
}

class TaskScheduler {
  // 使用优先队列（按优先级排序）
  final _queue = PriorityQueue<Task>((a, b) => b.priority.compareTo(a.priority));

  void schedule(Task task) {
    _queue.add(task);
    print('调度任务: $task');
  }

  Task? getNextTask() {
    if (_queue.isEmpty) return null;
    return _queue.removeFirst();
  }

  bool get hasTasks => _queue.isNotEmpty;
  int get taskCount => _queue.length;
}

// 简单的优先队列实现（基于 SplayTreeSet）
class PriorityQueue<E> {
  final Comparator<E> _compare;
  final _set = SplayTreeSet<E>((a, b) {
    // 需要处理相等的情况，否则会被视为重复
    var result = PriorityQueue._currentCompare!(a, b);
    return result == 0 ? a.hashCode.compareTo(b.hashCode) : result;
  });

  static Comparator? _currentCompare;

  PriorityQueue(this._compare);

  void add(E element) {
    _currentCompare = _compare;
    _set.add(element);
    _currentCompare = null;
  }

  E removeFirst() {
    var first = _set.first;
    _set.remove(first);
    return first;
  }

  bool get isEmpty => _set.isEmpty;
  bool get isNotEmpty => _set.isNotEmpty;
  int get length => _set.length;
}

void main() {
  var scheduler = TaskScheduler();

  scheduler.schedule(Task('数据备份', 1, Duration(minutes: 30)));
  scheduler.schedule(Task('邮件发送', 3, Duration(seconds: 10)));
  scheduler.schedule(Task('报表生成', 2, Duration(minutes: 5)));
  scheduler.schedule(Task('紧急修复', 5, Duration(minutes: 15)));

  print('\n按优先级执行任务:');
  while (scheduler.hasTasks) {
    var task = scheduler.getNextTask();
    print('执行: $task');
  }
}
```

---

## 2. LinkedList（链表）

`LinkedList` 是 Dart 提供的**双向链表**实现，与 `ListQueue` 不同，它直接暴露节点（`LinkedListEntry`），允许 O(1) 时间内在任意位置插入和删除。

### 2.1 基本用法

```dart
import 'dart:collection';

// 定义链表节点
class MyEntry extends LinkedListEntry<MyEntry> {
  final String value;
  MyEntry(this.value);

  @override
  String toString() => value;
}

void main() {
  // 创建链表
  var list = LinkedList<MyEntry>();

  // 添加节点
  list.add(MyEntry('A'));
  list.add(MyEntry('B'));
  list.add(MyEntry('C'));

  print(list);  // (A, B, C)

  // 遍历
  for (var entry in list) {
    print('值: ${entry.value}');
  }
}
```

### 2.2 链表节点操作

```dart
void linkedListOperations() {
  var list = LinkedList<MyEntry>();
  var entryA = MyEntry('A');
  var entryB = MyEntry('B');
  var entryC = MyEntry('C');
  var entryD = MyEntry('D');

  // 添加节点
  list.add(entryA);
  list.add(entryB);
  list.add(entryC);
  print('初始: $list');  // (A, B, C)

  // 在指定节点后插入
  entryA.insertAfter(entryD);
  print('A后插入D: $list');  // (A, D, B, C)

  // 在指定节点前插入
  var entryX = MyEntry('X');
  entryB.insertBefore(entryX);
  print('B前插入X: $list');  // (A, D, X, B, C)

  // 移除节点（O(1)）
  entryD.unlink();
  print('移除D: $list');  // (A, X, B, C)

  // 检查节点位置
  print('A有下一个: ${entryA.hasNext}');  // true
  print('C有下一个: ${entryC.hasNext}');  // false
  print('B有上一个: ${entryB.hasPrevious}');  // true
  print('A有上一个: ${entryA.hasPrevious}');  // false

  // 访问相邻节点
  print('A的下一个: ${entryA.next}');  // X
  print('B的上一个: ${entryB.previous}');  // X

  // 链表属性
  print('第一个: ${list.first}');  // A
  print('最后一个: ${list.last}');  // C
  print('长度: ${list.length}');  // 4
  print('是否为空: ${list.isEmpty}');  // false
}
```

### 2.3 实战应用：LRU 缓存

使用 `LinkedList` 和 `Map` 实现高效的 LRU（最近最少使用）缓存：

```dart
import 'dart:collection';

// 缓存条目节点
class _CacheEntry<K, V> extends LinkedListEntry<_CacheEntry<K, V>> {
  final K key;
  V value;

  _CacheEntry(this.key, this.value);
}

class LRUCache<K, V> {
  final int capacity;
  final Map<K, _CacheEntry<K, V>> _map = {};
  final LinkedList<_CacheEntry<K, V>> _list = LinkedList();

  LRUCache(this.capacity) {
    if (capacity <= 0) throw ArgumentError('容量必须大于0');
  }

  // 获取值，并将其移到最近使用位置
  V? get(K key) {
    var entry = _map[key];
    if (entry == null) return null;

    // 移到链表尾部（最近使用）
    entry.unlink();
    _list.add(entry);

    return entry.value;
  }

  // 设置值
  void put(K key, V value) {
    // 如果已存在，更新值并移到最近使用位置
    if (_map.containsKey(key)) {
      var entry = _map[key]!;
      entry.value = value;
      entry.unlink();
      _list.add(entry);
      return;
    }

    // 如果达到容量，移除最久未使用的（链表头部）
    if (_map.length >= capacity) {
      var oldest = _list.first;
      oldest.unlink();
      _map.remove(oldest.key);
    }

    // 添加新条目
    var entry = _CacheEntry(key, value);
    _map[key] = entry;
    _list.add(entry);
  }

  // 删除条目
  bool remove(K key) {
    var entry = _map.remove(key);
    if (entry != null) {
      entry.unlink();
      return true;
    }
    return false;
  }

  // 清空缓存
  void clear() {
    _map.clear();
    _list.clear();
  }

  // 获取当前缓存的所有键（按使用顺序）
  List<K> get keys => _list.map((e) => e.key).toList();

  // 获取当前缓存的所有值（按使用顺序）
  List<V> get values => _list.map((e) => e.value).toList();

  int get length => _map.length;
  bool get isEmpty => _map.isEmpty;
  bool get isNotEmpty => _map.isNotEmpty;

  @override
  String toString() {
    var entries = _list.map((e) => '${e.key}=${e.value}').join(', ');
    return 'LRUCache{$entries}';
  }
}

void main() {
  var cache = LRUCache<String, int>(3);

  cache.put('a', 1);
  cache.put('b', 2);
  cache.put('c', 3);
  print('添加 a,b,c: $cache');  // LRUCache{a=1, b=2, c=3}

  cache.get('a');  // 访问 a，使其变为最近使用
  print('访问 a 后: $cache');  // LRUCache{b=2, c=3, a=1}

  cache.put('d', 4);  // 添加 d，应该淘汰 b
  print('添加 d: $cache');  // LRUCache{c=3, a=1, d=4}

  print('获取 c: ${cache.get('c')}');  // 3
  print('获取 b: ${cache.get('b')}');  // null（已被淘汰）
}
```

---

## 3. SplayTreeMap 与 SplayTreeSet

`SplayTreeMap` 和 `SplayTreeSet` 是基于**伸展树（Splay Tree）**实现的有序集合。伸展树是一种自调整二叉搜索树，最近访问的元素会被移动到根部，使得频繁访问的元素具有更好的访问性能。

### 3.1 SplayTreeMap

`SplayTreeMap` 是按键的自然顺序或自定义比较器排序的 Map。

#### 创建 SplayTreeMap

```dart
import 'dart:collection';

void main() {
  // 方式1：空 Map（键必须可比较）
  var map1 = SplayTreeMap<String, int>();

  // 方式2：使用自定义比较器
  var map2 = SplayTreeMap<String, int>((a, b) => b.compareTo(a));  // 降序

  // 方式3：从现有 Map 创建
  var map3 = SplayTreeMap.from({'c': 3, 'a': 1, 'b': 2});
  print(map3);  // {a: 1, b: 2, c: 3}（自动排序）

  // 方式4：使用自定义比较器和相等判断
  var map4 = SplayTreeMap<String, int>(
    (a, b) => a.length.compareTo(b.length),
    isValidKey: (key) => key is String,
  );
}
```

#### SplayTreeMap 的核心特性

```dart
void splayTreeMapFeatures() {
  var scores = SplayTreeMap<String, int>();

  // 添加元素
  scores['Bob'] = 85;
  scores['Alice'] = 92;
  scores['Charlie'] = 78;
  scores['Diana'] = 95;

  // 自动按键排序
  print('排序后的 Map: $scores');
  // {Alice: 92, Bob: 85, Charlie: 78, Diana: 95}

  // 获取第一个和最后一个键值对
  print('第一个: ${scores.entries.first}');  // Alice: 92
  print('最后一个: ${scores.entries.last}');  // Diana: 95

  // 范围查询
  var range = scores.entries.where(
    (e) => e.key.compareTo('B') >= 0 && e.key.compareTo('D') < 0
  );
  print('B到D之间: ${range.toList()}');  // [Bob: 85, Charlie: 78]
}
```

#### 有序遍历与范围操作

```dart
void orderedOperations() {
  var tree = SplayTreeMap<int, String>();
  tree[50] = 'Fifty';
  tree[30] = 'Thirty';
  tree[70] = 'Seventy';
  tree[20] = 'Twenty';
  tree[40] = 'Forty';
  tree[60] = 'Sixty';
  tree[80] = 'Eighty';

  // 正序遍历
  print('正序:');
  tree.forEach((k, v) => print('  $k: $v'));

  // 倒序遍历
  print('\n倒序:');
  tree.entries.toList().reversed.forEach((e) => print('  ${e.key}: ${e.value}'));

  // 获取子 Map
  var subMap = SplayTreeMap<int, String>.from(
    Map.fromEntries(tree.entries.where((e) => e.key >= 30 && e.key <= 60))
  );
  print('\n30-60 范围: $subMap');  // {30: Thirty, 40: Forty, 50: Fifty, 60: Sixty}
}
```

### 3.2 SplayTreeSet

`SplayTreeSet` 是有序的 Set 实现，元素按自然顺序或自定义比较器排序。

```dart
void splayTreeSetDemo() {
  // 创建有序 Set
  var set = SplayTreeSet<int>();
  set.add(50);
  set.add(30);
  set.add(70);
  set.add(20);
  set.add(40);

  print('有序 Set: $set');  // {20, 30, 40, 50, 70}

  // 获取第一个和最后一个元素
  print('最小值: ${set.first}');  // 20
  print('最大值: ${set.last}');   // 70

  // 查找比指定值大的最小元素
  print('大于35的最小值: ${set.firstWhere((e) => e > 35)}');  // 40

  // 降序 Set
  var descSet = SplayTreeSet<int>((a, b) => b.compareTo(a));
  descSet.addAll([50, 30, 70, 20, 40]);
  print('降序 Set: $descSet');  // {70, 50, 40, 30, 20}
}
```

### 3.3 性能特点

伸展树的性能特点：

| 操作        | 平均时间复杂度 | 最坏时间复杂度 | 说明                     |
| ----------- | -------------- | -------------- | ------------------------ |
| 插入        | O(log n)       | O(n)           | 自调整使频繁访问元素更快 |
| 删除        | O(log n)       | O(n)           | -                        |
| 查找        | O(log n)       | O(n)           | 访问过的元素移到根部     |
| 遍历        | O(n)           | O(n)           | 中序遍历有序             |
| 最小/最大值 | O(log n)       | O(n)           | 伸展后变为 O(1)          |

**适用场景**：

- 需要有序遍历的数据
- 某些元素被频繁访问（工作集场景）
- 需要快速获取最小/最大值
- 实现优先队列

**不适用场景**：

- 需要严格 O(log n) 最坏情况保证（考虑使用 `SplayTreeMap` 的替代方案）
- 键的比较成本很高

### 3.4 实战应用

#### 应用1：时间窗口统计

```dart
import 'dart:collection';

class TimeWindowCounter {
  final Duration window;
  final SplayTreeMap<DateTime, int> _events = SplayTreeMap();

  TimeWindowCounter(this.window);

  void recordEvent([int count = 1]) {
    var now = DateTime.now();
    _events[now] = (_events[now] ?? 0) + count;
    _cleanupOldEvents(now);
  }

  void _cleanupOldEvents(DateTime now) {
    var cutoff = now.subtract(window);
    // 移除窗口外的事件
    var keysToRemove = _events.keys.where((k) => k.isBefore(cutoff)).toList();
    for (var key in keysToRemove) {
      _events.remove(key);
    }
  }

  int get countInWindow {
    return _events.values.fold(0, (sum, v) => sum + v);
  }

  Map<DateTime, int> get events => Map.unmodifiable(_events);
}

void main() {
  var counter = TimeWindowCounter(Duration(seconds: 5));

  counter.recordEvent(3);
  print('当前计数: ${counter.countInWindow}');  // 3

  Future.delayed(Duration(seconds: 2), () {
    counter.recordEvent(5);
    print('2秒后计数: ${counter.countInWindow}');  // 8
  });

  Future.delayed(Duration(seconds: 6), () {
    counter.recordEvent(2);
    print('6秒后计数: ${counter.countInWindow}');  // 7（第一次的3已过期）
  });
}
```

#### 应用2：排行榜实现

```dart
import 'dart:collection';

class Leaderboard {
  // 按分数降序排列的玩家
  final SplayTreeSet<_PlayerScore> _scores = SplayTreeSet(
    (a, b) => b.score != a.score
        ? b.score.compareTo(a.score)  // 分数降序
        : a.name.compareTo(b.name)     // 分数相同按名字升序
  );
  final Map<String, _PlayerScore> _players = {};

  void updateScore(String playerName, int newScore) {
    var existing = _players[playerName];
    if (existing != null) {
      _scores.remove(existing);
      existing.score = newScore;
    } else {
      existing = _PlayerScore(playerName, newScore);
      _players[playerName] = existing;
    }
    _scores.add(existing);
  }

  List<MapEntry<String, int>> getTop(int n) {
    return _scores.take(n).map((p) => MapEntry(p.name, p.score)).toList();
  }

  int? getRank(String playerName) {
    var player = _players[playerName];
    if (player == null) return null;

    var rank = 1;
    for (var p in _scores) {
      if (p.name == playerName) return rank;
      rank++;
    }
    return null;
  }

  void printLeaderboard() {
    print('=== 排行榜 ===');
    var rank = 1;
    for (var player in _scores) {
      print('$rank. ${player.name}: ${player.score}');
      rank++;
    }
  }
}

class _PlayerScore {
  final String name;
  int score;

  _PlayerScore(this.name, this.score);

  @override
  String toString() => '$name($score)';
}

void main() {
  var leaderboard = Leaderboard();

  leaderboard.updateScore('Alice', 100);
  leaderboard.updateScore('Bob', 150);
  leaderboard.updateScore('Charlie', 120);
  leaderboard.updateScore('Diana', 150);  // 并列
  leaderboard.updateScore('Bob', 180);    // 更新分数

  leaderboard.printLeaderboard();
  // === 排行榜 ===
  // 1. Bob: 180
  // 2. Bob: 150
  // 3. Diana: 150
  // 4. Charlie: 120
  // 5. Alice: 100

  print('\n前三名: ${leaderboard.getTop(3)}');
  print('Bob 的排名: ${leaderboard.getRank('Bob')}');
}
```

---

## 4. 集合视图（Views）

集合视图是包装现有集合的只读或受限访问接口，提供了一种安全的方式来共享集合数据而不暴露修改能力。

### 4.1 UnmodifiableListView

`UnmodifiableListView` 创建一个不可修改的 List 视图：

```dart
import 'dart:collection';

void unmodifiableListViewDemo() {
  var original = [1, 2, 3, 4, 5];
  var unmodifiable = UnmodifiableListView(original);

  // 可以读取
  print(unmodifiable[0]);  // 1
  print(unmodifiable.length);  // 5

  // 可以遍历
  for (var item in unmodifiable) {
    print(item);
  }

  // 尝试修改会抛出异常
  // unmodifiable[0] = 10;  // UnsupportedError
  // unmodifiable.add(6);   // UnsupportedError

  // 但原始列表的修改会反映到视图中
  original[0] = 100;
  print(unmodifiable[0]);  // 100
}
```

### 4.2 UnmodifiableMapView

```dart
void unmodifiableMapViewDemo() {
  var original = {'a': 1, 'b': 2, 'c': 3};
  var unmodifiable = UnmodifiableMapView(original);

  // 可以读取
  print(unmodifiable['a']);  // 1
  print(unmodifiable.keys);  // (a, b, c)

  // 尝试修改会抛出异常
  // unmodifiable['d'] = 4;  // UnsupportedError
  // unmodifiable.clear();   // UnsupportedError

  // 原始 Map 的修改会反映到视图中
  original['a'] = 100;
  print(unmodifiable['a']);  // 100
}
```

### 4.3 UnmodifiableSetView

```dart
void unmodifiableSetViewDemo() {
  var original = <int>{1, 2, 3};
  var unmodifiable = UnmodifiableSetView(original);

  // 可以读取
  print(unmodifiable.contains(2));  // true
  print(unmodifiable.length);  // 3

  // 尝试修改会抛出异常
  // unmodifiable.add(4);     // UnsupportedError
  // unmodifiable.remove(1);  // UnsupportedError

  // 原始 Set 的修改会反映到视图中
  original.add(4);
  print(unmodifiable.contains(4));  // true
}
```

### 4.4 MapView

`MapView` 是一个可修改的 Map 包装器，常用于创建自定义 Map 类：

```dart
class CaseInsensitiveMap<V> extends MapView<String, V> {
  final Map<String, V> _source;

  CaseInsensitiveMap() : this._({});
  CaseInsensitiveMap._(this._source) : super(_source);

  @override
  V? operator [](Object? key) {
    if (key is String) {
      return _source[key.toLowerCase()];
    }
    return null;
  }

  @override
  void operator []=(String key, V value) {
    _source[key.toLowerCase()] = value;
  }

  @override
  V? remove(Object? key) {
    if (key is String) {
      return _source.remove(key.toLowerCase());
    }
    return null;
  }

  @override
  bool containsKey(Object? key) {
    if (key is String) {
      return _source.containsKey(key.toLowerCase());
    }
    return false;
  }
}

void main() {
  var map = CaseInsensitiveMap<int>();

  map['Hello'] = 1;
  map['WORLD'] = 2;

  print(map['hello']);   // 1
  print(map['Hello']);   // 1
  print(map['HELLO']);   // 1
  print(map['world']);   // 2

  print(map.containsKey('WORLD'));  // true
  print(map);  // {hello: 1, world: 2}
}
```

### 4.5 CombinedIterableView

`CombinedIterableView` 将多个可迭代对象组合成一个统一的视图：

```dart
void combinedIterableViewDemo() {
  var list1 = [1, 2, 3];
  var list2 = [4, 5];
  var set = <int>{6, 7, 8};

  var combined = CombinedIterableView([list1, list2, set]);

  print('组合后的元素: ${combined.toList()}');  // [1, 2, 3, 4, 5, 6, 7, 8]
  print('长度: ${combined.length}');  // 8
  print('包含 5: ${combined.contains(5)}');  // true

  // 可以遍历
  for (var item in combined) {
    print(item);
  }

  // 底层集合的修改会反映到视图中
  list1.add(100);
  print('修改后: ${combined.toList()}');  // [1, 2, 3, 100, 4, 5, 6, 7, 8]
}
```

### 4.6 CombinedListView

`CombinedListView` 类似于 `CombinedIterableView`，但提供 List 接口（支持索引访问）：

```dart
void combinedListViewDemo() {
  var list1 = [1, 2, 3];
  var list2 = [4, 5, 6];
  var list3 = [7, 8, 9];

  var combined = CombinedListView([list1, list2, list3]);

  // 支持索引访问
  print('combined[0]: ${combined[0]}');  // 1
  print('combined[4]: ${combined[4]}');  // 5
  print('combined[7]: ${combined[7]}');  // 8

  // 支持长度查询
  print('长度: ${combined.length}');  // 9

  // 支持索引遍历
  for (var i = 0; i < combined.length; i++) {
    print('combined[$i] = ${combined[i]}');
  }

  // 底层列表的修改会反映到视图中
  list2[0] = 400;
  print('修改后 combined[3]: ${combined[3]}');  // 400
}
```

---

## 5. 集合相等性比较

`dart:collection` 提供了用于比较集合相等性的类，支持深度比较和自定义相等规则。

### 5.1 MapEquality

```dart
import 'dart:collection';

void mapEqualityDemo() {
  var map1 = {'a': 1, 'b': 2, 'c': 3};
  var map2 = {'b': 2, 'a': 1, 'c': 3};  // 顺序不同
  var map3 = {'a': 1, 'b': 2};

  var equality = MapEquality<String, int>();

  print('map1 == map2: ${equality.equals(map1, map2)}');  // true（顺序无关）
  print('map1 == map3: ${equality.equals(map1, map3)}');  // false

  // 计算哈希码
  print('map1 hash: ${equality.hash(map1)}');
  print('map2 hash: ${equality.hash(map2)}');  // 与 map1 相同
}
```

### 5.2 SetEquality

```dart
void setEqualityDemo() {
  var set1 = <int>{1, 2, 3};
  var set2 = <int>{3, 2, 1};  // 顺序不同
  var set3 = <int>{1, 2, 4};

  var equality = SetEquality<int>();

  print('set1 == set2: ${equality.equals(set1, set2)}');  // true
  print('set1 == set3: ${equality.equals(set1, set3)}');  // false
}
```

### 5.3 IterableEquality

```dart
void iterableEqualityDemo() {
  var list = [1, 2, 3];
  var set = <int>{1, 2, 3};

  var equality = IterableEquality<int>();

  print('list == set: ${equality.equals(list, set)}');  // true（内容相同即可）

  // 顺序敏感
  var list1 = [1, 2, 3];
  var list2 = [3, 2, 1];
  print('list1 == list2: ${equality.equals(list1, list2)}');  // false（顺序不同）
}
```

### 5.4 深度相等性比较

```dart
void deepEqualityDemo() {
  var map1 = {
    'users': [
      {'name': 'Alice', 'age': 30},
      {'name': 'Bob', 'age': 25},
    ]
  };

  var map2 = {
    'users': [
      {'name': 'Alice', 'age': 30},
      {'name': 'Bob', 'age': 25},
    ]
  };

  // 使用深度相等性
  var deepEquality = DeepCollectionEquality();
  print('深度相等: ${deepEquality.equals(map1, map2)}');  // true

  // 普通相等性
  print('普通相等: ${map1 == map2}');  // false（引用不同）
}
```

### 5.5 自定义相等性规则

```dart
void customEqualityDemo() {
  // 忽略大小写的字符串 Set 相等性
  var set1 = <String>{'Apple', 'Banana'};
  var set2 = <String>{('apple', 'banana'};

  var caseInsensitiveEquality = SetEquality(
    EqualityBy((String s) => s.toLowerCase()),
  );

  print('忽略大小写相等: ${caseInsensitiveEquality.equals(set1, set2)}');  // true
}

// 辅助类：基于投影的相等性
class EqualityBy<T, K> implements Equality<T> {
  final K Function(T) _keyExtractor;

  EqualityBy(this._keyExtractor);

  @override
  bool equals(T e1, T e2) => _keyExtractor(e1) == _keyExtractor(e2);

  @override
  int hash(T e) => _keyExtractor(e).hashCode;

  @override
  bool isValidKey(Object? o) => o is T;
}
```

---

## 6. 集合基类与 Mixin

`dart:collection` 提供了一系列基类和 Mixin，用于简化自定义集合类型的实现。

### 6.1 ListBase

`ListBase` 是一个抽象基类，只需实现 `length`、`operator[]` 和 `operator[]=` 即可获得完整的 List 功能：

```dart
import 'dart:collection';

class FixedSizeList<E> extends ListBase<E> {
  final List<E> _data;

  FixedSizeList(int size, E fillValue) : _data = List.filled(size, fillValue);

  @override
  int get length => _data.length;

  @override
  set length(int newLength) {
    throw UnsupportedError('固定长度列表不能调整大小');
  }

  @override
  E operator [](int index) => _data[index];

  @override
  void operator []=(int index, E value) {
    _data[index] = value;
  }
}

void main() {
  var list = FixedSizeList<int>(5, 0);
  print('初始: $list');  // [0, 0, 0, 0, 0]

  list[0] = 10;
  list[2] = 30;
  print('修改后: $list');  // [10, 0, 30, 0, 0]

  // 继承的方法
  list.add(100);  // 可以添加（因为 ListBase 实现了 add）
  print('添加后: $list');  // [10, 0, 30, 0, 0, 100]

  // 但 length setter 会抛出异常
  // list.length = 3;  // UnsupportedError
}
```

### 6.2 SetBase

```dart
class CaseInsensitiveSet extends SetBase<String> {
  final Set<String> _source = {};

  @override
  bool add(String value) => _source.add(value.toLowerCase());

  @override
  bool contains(Object? element) {
    if (element is String) {
      return _source.contains(element.toLowerCase());
    }
    return false;
  }

  @override
  String? lookup(Object? element) {
    if (contains(element)) {
      return _source.lookup((element as String).toLowerCase());
    }
    return null;
  }

  @override
  bool remove(Object? value) {
    if (value is String) {
      return _source.remove(value.toLowerCase());
    }
    return false;
  }

  @override
  Iterator<String> get iterator => _source.iterator;

  @override
  int get length => _source.length;

  @override
  Set<String> toSet() => CaseInsensitiveSet()..addAll(_source);
}

void main() {
  var set = CaseInsensitiveSet();
  set.add('Hello');
  set.add('WORLD');
  set.add('hello');  // 不会添加（已存在）

  print(set);  // {hello, world}
  print(set.contains('HELLO'));  // true
  print(set.contains('World'));  // true
}
```

### 6.3 MapBase

```dart
class DefaultValueMap<K, V> extends MapBase<K, V> {
  final Map<K, V> _source = {};
  final V Function() _defaultValue;

  DefaultValueMap(this._defaultValue);

  @override
  V? operator [](Object? key) {
    if (!_source.containsKey(key)) {
      _source[key as K] = _defaultValue();
    }
    return _source[key];
  }

  @override
  void operator []=(K key, V value) {
    _source[key] = value;
  }

  @override
  V? remove(Object? key) => _source.remove(key);

  @override
  void clear() => _source.clear();

  @override
  Iterable<K> get keys => _source.keys;
}

void main() {
  var map = DefaultValueMap<String, List<int>>(() => []);

  map['a'].add(1);
  map['a'].add(2);
  map['b'].add(3);

  print(map);  // {a: [1, 2], b: [3]}
}
```

### 6.4 ListMixin、SetMixin、MapMixin

Mixin 提供了与基类类似的功能，但可以通过 `with` 关键字与现有类组合：

```dart
class ReadOnlyList<E> with ListMixin<E> {
  final List<E> _source;

  ReadOnlyList(this._source);

  @override
  int get length => _source.length;

  @override
  set length(int newLength) {
    throw UnsupportedError('只读列表');
  }

  @override
  E operator [](int index) => _source[index];

  @override
  void operator []=(int index, E value) {
    throw UnsupportedError('只读列表');
  }
}

void main() {
  var original = [1, 2, 3, 4, 5];
  var readOnly = ReadOnlyList(original);

  print(readOnly[0]);  // 1
  print(readOnly.sublist(1, 3));  // [2, 3]

  // readOnly[0] = 10;  // UnsupportedError
}
```

---

## 7. CanonicalizedMap

`CanonicalizedMap` 是一个特殊的 Map，它使用**规范化函数**来确定键的相等性，而不是直接使用键的 `==` 运算符。

### 7.1 基本用法

```dart
import 'dart:collection';

void canonicalizedMapDemo() {
  // 创建大小写不敏感的 Map
  var map = CanonicalizedMap<String, String, int>(
    (key) => key.toLowerCase(),  // 规范化函数
    equals: (a, b) => a.toLowerCase() == b.toLowerCase(),
    isValidKey: (key) => key is String,
  );

  map['Hello'] = 1;
  map['WORLD'] = 2;
  map['HELLO'] = 3;  // 覆盖第一个条目（因为规范化后相同）

  print(map);  // {HELLO: 3, WORLD: 2}
  print(map['hello']);  // 3
  print(map['Hello']);  // 3
  print(map['HELLO']);  // 3
}
```

### 7.2 实战应用：规范化缓存

```dart
class NormalizedCache<K, V> {
  final CanonicalizedMap<K, K, V> _map;

  NormalizedCache(K Function(K) canonicalize)
      : _map = CanonicalizedMap(
          canonicalize,
          equals: (a, b) => canonicalize(a) == canonicalize(b),
          isValidKey: (_) => true,
        );

  V? get(K key) => _map[key];
  void set(K key, V value) => _map[key] = value;
  bool containsKey(K key) => _map.containsKey(key);
  V? remove(K key) => _map.remove(key);

  Iterable<MapEntry<K, V>> get entries => _map.entries;
  int get length => _map.length;
}

void main() {
  // URL 缓存（规范化 URL）
  var urlCache = NormalizedCache<String, String>((url) {
    return url.toLowerCase().replaceAll(RegExp(r'/+$'), '');  // 小写并去除末尾斜杠
  });

  urlCache.set('https://EXAMPLE.com/', '首页内容');

  print(urlCache.get('https://example.com'));      // 首页内容
  print(urlCache.get('https://EXAMPLE.COM/'));     // 首页内容
  print(urlCache.get('https://example.com///'));   // 首页内容

  print('缓存条目数: ${urlCache.length}');  // 1
}
```

---

## 8. 实战应用示例

### 8.1 事件总线（使用 Stream 和 Queue）

```dart
import 'dart:async';
import 'dart:collection';

class EventBus {
  final _controllers = <Type, StreamController<dynamic>>{};
  final _pendingEvents = Queue<_PendingEvent>();
  bool _isProcessing = false;

  Stream<T> on<T>() {
    return _controllers
        .putIfAbsent(T, () => StreamController<T>.broadcast())
        .stream as Stream<T>;
  }

  void emit<T>(T event) {
    _pendingEvents.add(_PendingEvent(T, event));
    _processEvents();
  }

  void _processEvents() async {
    if (_isProcessing) return;
    _isProcessing = true;

    while (_pendingEvents.isNotEmpty) {
      var pending = _pendingEvents.removeFirst();
      var controller = _controllers[pending.type];
      if (controller != null && !controller.isClosed) {
        controller.add(pending.event);
      }
    }

    _isProcessing = false;
  }

  void dispose() {
    for (var controller in _controllers.values) {
      controller.close();
    }
    _controllers.clear();
    _pendingEvents.clear();
  }
}

class _PendingEvent {
  final Type type;
  final dynamic event;
  _PendingEvent(this.type, this.event);
}

// 使用示例
class UserLoginEvent {
  final String username;
  UserLoginEvent(this.username);
}

class DataUpdatedEvent {
  final String entityType;
  DataUpdatedEvent(this.entityType);
}

void main() {
  var eventBus = EventBus();

  // 订阅事件
  eventBus.on<UserLoginEvent>().listen((event) {
    print('用户登录: ${event.username}');
  });

  eventBus.on<DataUpdatedEvent>().listen((event) {
    print('数据更新: ${event.entityType}');
  });

  // 发送事件
  eventBus.emit(UserLoginEvent('Alice'));
  eventBus.emit(DataUpdatedEvent('users'));
  eventBus.emit(UserLoginEvent('Bob'));

  eventBus.dispose();
}
```

### 8.2 带过期时间的缓存

```dart
import 'dart:collection';

class ExpiringCache<K, V> {
  final Duration defaultTtl;
  final _entries = <K, _CacheEntry<V>>{};
  final _accessOrder = LinkedList<_AccessNode<K>>();
  final _accessMap = <K, _AccessNode<K>>{};

  ExpiringCache({this.defaultTtl = const Duration(minutes: 5)});

  V? get(K key) {
    var entry = _entries[key];
    if (entry == null) return null;

    if (entry.isExpired) {
      _remove(key);
      return null;
    }

    _updateAccessOrder(key);
    return entry.value;
  }

  void set(K key, V value, {Duration? ttl}) {
    var entry = _CacheEntry(value, DateTime.now().add(ttl ?? defaultTtl));
    _entries[key] = entry;
    _updateAccessOrder(key);
    _cleanupExpired();
  }

  void _updateAccessOrder(K key) {
    var node = _accessMap[key];
    if (node != null) {
      node.unlink();
    }
    node = _AccessNode(key);
    _accessMap[key] = node;
    _accessOrder.add(node);
  }

  void _remove(K key) {
    _entries.remove(key);
    var node = _accessMap.remove(key);
    node?.unlink();
  }

  void _cleanupExpired() {
    var now = DateTime.now();
    var keysToRemove = <K>[];

    for (var entry in _entries.entries) {
      if (entry.value.expiresAt.isBefore(now)) {
        keysToRemove.add(entry.key);
      }
    }

    for (var key in keysToRemove) {
      _remove(key);
    }
  }

  void clear() {
    _entries.clear();
    _accessOrder.clear();
    _accessMap.clear();
  }

  int get length => _entries.length;
  bool get isEmpty => _entries.isEmpty;
}

class _CacheEntry<V> {
  final V value;
  final DateTime expiresAt;

  _CacheEntry(this.value, this.expiresAt);

  bool get isExpired => DateTime.now().isAfter(expiresAt);
}

class _AccessNode<K> extends LinkedListEntry<_AccessNode<K>> {
  final K key;
  _AccessNode(this.key);
}

void main() {
  var cache = ExpiringCache<String, String>(defaultTtl: Duration(seconds: 2));

  cache.set('key1', 'value1');
  print('设置 key1');

  Future.delayed(Duration(seconds: 1), () {
    print('1秒后获取 key1: ${cache.get('key1')}');  // value1
  });

  Future.delayed(Duration(seconds: 3), () {
    print('3秒后获取 key1: ${cache.get('key1')}');  // null（已过期）
  });
}
```

### 8.3 优先级任务队列

```dart
import 'dart:collection';

class PriorityTaskQueue<T> {
  final _queues = <int, DoubleLinkedQueue<T>>{};
  final _priorities = SplayTreeSet<int>((a, b) => b.compareTo(a));  // 降序

  void enqueue(T task, int priority) {
    _queues.putIfAbsent(priority, () => DoubleLinkedQueue());
    _queues[priority]!.addLast(task);
    _priorities.add(priority);
  }

  T? dequeue() {
    for (var priority in _priorities) {
      var queue = _queues[priority]!;
      if (queue.isNotEmpty) {
        var task = queue.removeFirst();
        if (queue.isEmpty) {
          _priorities.remove(priority);
        }
        return task;
      }
    }
    return null;
  }

  T? peek() {
    for (var priority in _priorities) {
      var queue = _queues[priority]!;
      if (queue.isNotEmpty) {
        return queue.first;
      }
    }
    return null;
  }

  bool get isEmpty => _priorities.isEmpty;
  bool get isNotEmpty => _priorities.isNotEmpty;

  int get length {
    return _queues.values.fold(0, (sum, q) => sum + q.length);
  }
}

void main() {
  var taskQueue = PriorityTaskQueue<String>();

  taskQueue.enqueue('普通任务1', 1);
  taskQueue.enqueue('紧急任务1', 5);
  taskQueue.enqueue('普通任务2', 1);
  taskQueue.enqueue('高优先级任务', 3);
  taskQueue.enqueue('紧急任务2', 5);

  print('按优先级执行任务:');
  while (taskQueue.isNotEmpty) {
    print('  ${taskQueue.dequeue()}');
  }
  // 紧急任务1
  // 紧急任务2
  // 高优先级任务
  // 普通任务1
  // 普通任务2
}
```

---

## 9. 附录：速查表

### 9.1 类对照表

| 类名                   | 类型       | 主要用途          | 时间复杂度                |
| ---------------------- | ---------- | ----------------- | ------------------------- |
| `ListQueue`            | Queue 实现 | FIFO 队列         | addLast/removeFirst: O(1) |
| `DoubleLinkedQueue`    | Queue 实现 | 双端队列          | 两端操作: O(1)            |
| `LinkedList`           | 链表       | O(1) 插入删除     | 插入/删除: O(1)           |
| `SplayTreeMap`         | 有序 Map   | 按键排序          | 插入/查找: O(log n)       |
| `SplayTreeSet`         | 有序 Set   | 有序集合          | 插入/查找: O(log n)       |
| `UnmodifiableListView` | 视图       | 只读 List         | 取决于底层                |
| `UnmodifiableMapView`  | 视图       | 只读 Map          | 取决于底层                |
| `UnmodifiableSetView`  | 视图       | 只读 Set          | 取决于底层                |
| `MapView`              | 视图       | 自定义 Map        | 取决于底层                |
| `CombinedIterableView` | 视图       | 组合多个 Iterable | 取决于底层                |
| `CombinedListView`     | 视图       | 组合多个 List     | 取决于底层                |
| `CanonicalizedMap`     | 特殊 Map   | 规范化键          | 取决于底层                |

### 9.2 基类与 Mixin 对照表

| 名称        | 类型   | 需要实现                  | 提供功能      |
| ----------- | ------ | ------------------------- | ------------- |
| `ListBase`  | 抽象类 | length, [], []=           | 完整 List API |
| `SetBase`   | 抽象类 | add, contains, iterator等 | 完整 Set API  |
| `MapBase`   | 抽象类 | [], []=, keys, remove等   | 完整 Map API  |
| `ListMixin` | Mixin  | length, [], []=           | 完整 List API |
| `SetMixin`  | Mixin  | add, contains, iterator等 | 完整 Set API  |
| `MapMixin`  | Mixin  | [], []=, keys, remove等   | 完整 Map API  |

### 9.3 相等性类对照表

| 类名                     | 用途          | 特点                |
| ------------------------ | ------------- | ------------------- |
| `MapEquality`            | 比较 Map      | 顺序无关            |
| `SetEquality`            | 比较 Set      | 顺序无关            |
| `IterableEquality`       | 比较 Iterable | 顺序敏感            |
| `DeepCollectionEquality` | 深度比较      | 递归比较嵌套集合    |
| `DefaultEquality`        | 默认相等性    | 使用 == 和 hashCode |
| `IdentityEquality`       | 身份相等性    | 使用 identical      |

---

## 总结

`dart:collection` 库为 Dart 开发者提供了丰富的集合类型和工具类，弥补了标准集合的不足：

1. **Queue 实现**：`ListQueue` 和 `DoubleLinkedQueue` 提供了高效的 FIFO 和双端操作能力，适用于任务调度、BFS 等场景。

2. **LinkedList**：通过暴露节点实现 O(1) 的任意位置插入删除，是 LRU 缓存等场景的理想选择。

3. **SplayTreeMap/Set**：基于伸展树的有序集合，支持高效的范围查询和有序遍历，适用于排行榜、时间窗口等场景。

4. **集合视图**：提供了一种安全的数据共享机制，可以创建只读视图或自定义集合行为。

5. **相等性比较**：支持深度比较和自定义相等规则，简化了集合比较的逻辑。

6. **基类与 Mixin**：大幅简化了自定义集合类型的实现，只需实现少量方法即可获得完整功能。

7. **CanonicalizedMap**：通过规范化函数实现灵活的键相等性判断，适用于 URL 缓存等需要规范化键的场景。

掌握这些工具类，可以让你在处理复杂集合操作时更加得心应手，编写出更高效、更优雅的 Dart 代码。
