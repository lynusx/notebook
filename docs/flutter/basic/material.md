# Flutter 3.41 Material 组件详解与实战

## 前言

Flutter 是 Google 推出的一款开源 UI 软件开发工具包，它允许开发者使用一套代码库构建跨平台应用程序。Material Design 是 Google 于 2014 年提出的设计语言，强调程序界面的质感、层次和物体之间的叠放逻辑。Flutter 框架内置了大量 Material 风格的组件，帮助开发者快速且高效地搭建优美的用户界面。

本书基于 Flutter 3.41 版本，详细介绍 Material 库中包含的全部组件，包括每个组件的属性、方法用途及意义，并提供丰富的示例代码，帮助读者深入理解和灵活运用这些组件。

---

## 第1章 基础布局组件

### 1.1 Container

Container 是 Flutter 中最常用的布局组件之一，它是一个多功能容器，可以同时实现尺寸约束、内外边距、背景装饰、变换等多种功能。

#### 1.1.1 基础用法

Container 的基础用法非常简单，只需将其作为父级组件包裹子组件即可：

```dart
Container(
  child: Text('Hello Flutter'),
)
```

#### 1.1.2 核心属性详解

**1. 尺寸约束属性**

| 属性名      | 类型            | 用途说明                 |
| ----------- | --------------- | ------------------------ |
| width       | double?         | 设置容器的宽度           |
| height      | double?         | 设置容器的高度           |
| constraints | BoxConstraints? | 设置更复杂的尺寸约束条件 |

当同时设置 width/height 和 constraints 时，constraints 会优先生效。例如：

```dart
Container(
  width: 200,
  height: 100,
  constraints: BoxConstraints(
    minWidth: 150,
    maxWidth: 250,
    minHeight: 80,
    maxHeight: 120,
  ),
  child: FlutterLogo(),
)
```

**2. 边距属性**

| 属性名  | 类型                | 用途说明                                   |
| ------- | ------------------- | ------------------------------------------ |
| padding | EdgeInsetsGeometry? | 设置内边距，即子组件与容器边框之间的距离   |
| margin  | EdgeInsetsGeometry? | 设置外边距，即容器与外部其他组件之间的距离 |

EdgeInsets 提供了多种构造函数：

```dart
// 统一设置四周边距
EdgeInsets.all(16.0)

// 分别设置四个方向的边距
EdgeInsets.fromLTRB(left, top, right, bottom)

// 只设置特定方向的边距
EdgeInsets.only(left: 10, top: 20)

// 设置对称边距
EdgeInsets.symmetric(horizontal: 20, vertical: 10)
```

**3. 装饰属性**

| 属性名               | 类型        | 用途说明                                 |
| -------------------- | ----------- | ---------------------------------------- |
| decoration           | Decoration? | 设置背景装饰，如颜色、边框、圆角、阴影等 |
| foregroundDecoration | Decoration? | 设置前景装饰，会覆盖在子组件之上         |

BoxDecoration 是最常用的装饰类型：

```dart
Container(
  decoration: BoxDecoration(
    color: Colors.blue,                    // 背景颜色
    borderRadius: BorderRadius.circular(8), // 圆角
    border: Border.all(                     // 边框
      color: Colors.black,
      width: 2,
    ),
    boxShadow: [                            // 阴影
      BoxShadow(
        color: Colors.grey.withOpacity(0.5),
        blurRadius: 10,
        offset: Offset(0, 5),
      ),
    ],
    gradient: LinearGradient(               // 渐变
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: [Colors.blue, Colors.green],
    ),
  ),
  child: Text('装饰效果'),
)
```

**4. 对齐与变换属性**

| 属性名             | 类型               | 用途说明                     |
| ------------------ | ------------------ | ---------------------------- |
| alignment          | AlignmentGeometry? | 设置子组件在容器内的对齐方式 |
| transform          | Matrix4?           | 对子组件应用矩阵变换         |
| transformAlignment | AlignmentGeometry? | 设置变换的锚点               |

```dart
Container(
  alignment: Alignment.center,
  transform: Matrix4.rotationZ(0.1), // 旋转10度
  transformAlignment: Alignment.center,
  child: Text('变换效果'),
)
```

#### 1.1.3 完整示例

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    home: Scaffold(
      body: Center(
        child: Container(
          width: 300,
          height: 200,
          padding: EdgeInsets.all(20),
          margin: EdgeInsets.all(10),
          alignment: Alignment.center,
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(16),
            boxShadow: [
              BoxShadow(
                color: Colors.grey.withOpacity(0.3),
                blurRadius: 20,
                offset: Offset(0, 10),
              ),
            ],
          ),
          child: Text(
            '精美卡片',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
        ),
      ),
    ),
  ));
}
```

> **Flutter框架小知识**
>
> Container 组件本质上是一个语法糖，它内部会根据设置的属性自动组合多个基础组件（如 ConstrainedBox、DecoratedBox、Padding、Transform 等）来实现效果。如果只需要单一功能，建议直接使用对应的基础组件以提高代码可读性。

---

### 1.2 Row 与 Column

Row 和 Column 是 Flutter 中最基础的弹性布局组件，分别用于水平方向和垂直方向排列子组件。

#### 1.2.1 Row 组件

Row 组件将其子组件在水平方向上依次排列。

**核心属性：**

| 属性名             | 类型               | 默认值   | 用途说明                       |
| ------------------ | ------------------ | -------- | ------------------------------ |
| mainAxisAlignment  | MainAxisAlignment  | start    | 主轴（水平方向）对齐方式       |
| crossAxisAlignment | CrossAxisAlignment | center   | 交叉轴（垂直方向）对齐方式     |
| mainAxisSize       | MainAxisSize       | max      | 主轴尺寸，max 表示占满可用空间 |
| textDirection      | TextDirection?     | null     | 子组件排列方向                 |
| verticalDirection  | VerticalDirection  | down     | 垂直方向排列顺序               |
| children           | List<Widget>       | const [] | 子组件列表                     |

**MainAxisAlignment 枚举值：**

| 值           | 说明                         |
| ------------ | ---------------------------- |
| start        | 从起始位置开始排列           |
| end          | 从结束位置开始排列           |
| center       | 居中对齐                     |
| spaceBetween | 两端对齐，子组件之间均匀分布 |
| spaceAround  | 子组件周围均匀分布空间       |
| spaceEvenly  | 所有空间均匀分布             |

**CrossAxisAlignment 枚举值：**

| 值       | 说明                                 |
| -------- | ------------------------------------ |
| start    | 顶部对齐                             |
| end      | 底部对齐                             |
| center   | 居中对齐（默认值）                   |
| stretch  | 拉伸填满交叉轴                       |
| baseline | 基线对齐（需配合 textBaseline 使用） |

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Icon(Icons.home, size: 40),
    Icon(Icons.search, size: 40),
    Icon(Icons.person, size: 40),
  ],
)
```

#### 1.2.2 Column 组件

Column 组件将其子组件在垂直方向上依次排列，其属性与 Row 完全相同，只是主轴和交叉轴互换。

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.stretch,
  children: [
    Text('第一行', style: TextStyle(fontSize: 20)),
    SizedBox(height: 20),
    Text('第二行', style: TextStyle(fontSize: 20)),
    SizedBox(height: 20),
    Text('第三行', style: TextStyle(fontSize: 20)),
  ],
)
```

#### 1.2.3 Expanded 与 Flexible

当子组件需要占据剩余空间时，可以使用 Expanded 或 Flexible 包裹子组件。

**Expanded 组件：**

```dart
Row(
  children: [
    Container(width: 50, height: 50, color: Colors.red),
    Expanded(
      flex: 2,  // 弹性系数，默认值为1
      child: Container(height: 50, color: Colors.green),
    ),
    Expanded(
      flex: 1,
      child: Container(height: 50, color: Colors.blue),
    ),
  ],
)
```

**Flexible 组件：**

Flexible 与 Expanded 的区别在于 fit 属性。Expanded 相当于 Flexible 的 fit 设置为 FlexFit.tight。

```dart
Row(
  children: [
    Flexible(
      fit: FlexFit.loose,  // 子组件可以保持自身尺寸
      child: Container(width: 100, height: 50, color: Colors.red),
    ),
    Flexible(
      fit: FlexFit.tight,  // 子组件必须填满分配的空间
      child: Container(height: 50, color: Colors.green),
    ),
  ],
)
```

> **Dart Tips语法小贴士**
>
> 在 Dart 的列表中，可以使用展开运算符 `...` 将一个列表的元素插入到另一个列表中：
>
> ```dart
> var list1 = [1, 2, 3];
> var list2 = [0, ...list1, 4];  // [0, 1, 2, 3, 4]
> ```
>
> 这在构建动态子组件列表时非常有用。

---

### 1.3 Stack 与 Positioned

Stack 组件允许子组件重叠显示，配合 Positioned 组件可以精确控制子组件的位置。

#### 1.3.1 Stack 组件

**核心属性：**

| 属性名        | 类型              | 默认值   | 用途说明                            |
| ------------- | ----------------- | -------- | ----------------------------------- |
| alignment     | AlignmentGeometry | topStart | 非 Positioned 子组件的对齐方式      |
| textDirection | TextDirection?    | null     | 文本方向，影响 alignment 的起始位置 |
| fit           | StackFit          | loose    | 非 Positioned 子组件如何适应 Stack  |
| clipBehavior  | Clip              | hardEdge | 超出 Stack 边界的裁剪行为           |
| children      | List<Widget>      | const [] | 子组件列表                          |

**StackFit 枚举值：**

| 值          | 说明                             |
| ----------- | -------------------------------- |
| loose       | 子组件可以保持自身尺寸（默认值） |
| expand      | 子组件必须扩展填满 Stack         |
| passthrough | 将 Stack 的约束直接传递给子组件  |

```dart
Stack(
  alignment: Alignment.center,
  children: [
    Container(width: 200, height: 200, color: Colors.red),
    Container(width: 150, height: 150, color: Colors.green),
    Container(width: 100, height: 100, color: Colors.blue),
  ],
)
```

#### 1.3.2 Positioned 组件

Positioned 组件必须作为 Stack 的子组件使用，用于精确控制子组件的位置。

**核心属性：**

| 属性名 | 类型    | 用途说明              |
| ------ | ------- | --------------------- |
| left   | double? | 距离 Stack 左边的距离 |
| top    | double? | 距离 Stack 顶部的距离 |
| right  | double? | 距离 Stack 右边的距离 |
| bottom | double? | 距离 Stack 底部的距离 |
| width  | double? | 子组件的宽度          |
| height | double? | 子组件的高度          |

> **注意**：同一方向上最多只能设置两个属性。例如，设置了 left 和 width 后，right 会自动计算得出。

```dart
Stack(
  children: [
    Container(width: 300, height: 300, color: Colors.grey),
    Positioned(
      left: 20,
      top: 20,
      child: Container(width: 80, height: 80, color: Colors.red),
    ),
    Positioned(
      right: 20,
      bottom: 20,
      child: Container(width: 80, height: 80, color: Colors.blue),
    ),
    Positioned(
      left: 0,
      right: 0,
      top: 100,
      child: Container(height: 50, color: Colors.green),
    ),
  ],
)
```

#### 1.3.3 PositionedDirectional 组件

PositionedDirectional 与 Positioned 类似，但使用 start 和 end 替代 left 和 right，可以自动适应从右到左（RTL）的语言环境。

```dart
Stack(
  children: [
    PositionedDirectional(
      start: 20,  // 起始方向（左或右，取决于语言）
      top: 20,
      child: Container(width: 80, height: 80, color: Colors.red),
    ),
  ],
)
```

---

### 1.4 Padding 与 Align

#### 1.4.1 Padding 组件

Padding 组件用于在子组件周围添加内边距。

**核心属性：**

| 属性名  | 类型               | 用途说明           |
| ------- | ------------------ | ------------------ |
| padding | EdgeInsetsGeometry | 内边距值，必传参数 |
| child   | Widget?            | 子组件             |

```dart
Padding(
  padding: EdgeInsets.all(16.0),
  child: Text('带内边距的文本'),
)
```

#### 1.4.2 Align 组件

Align 组件用于控制子组件在其父组件中的对齐方式。

**核心属性：**

| 属性名       | 类型              | 默认值 | 用途说明                                |
| ------------ | ----------------- | ------ | --------------------------------------- |
| alignment    | AlignmentGeometry | center | 子组件的对齐方式                        |
| widthFactor  | double?           | null   | 宽度因子，用于确定 Align 组件自身的宽度 |
| heightFactor | double?           | null   | 高度因子，用于确定 Align 组件自身的高度 |
| child        | Widget?           | null   | 子组件                                  |

**Alignment 常用值：**

| 值           | 说明     |
| ------------ | -------- |
| topLeft      | 左上角   |
| topCenter    | 顶部居中 |
| topRight     | 右上角   |
| centerLeft   | 左侧居中 |
| center       | 正中心   |
| centerRight  | 右侧居中 |
| bottomLeft   | 左下角   |
| bottomCenter | 底部居中 |
| bottomRight  | 右下角   |

```dart
Align(
  alignment: Alignment.bottomRight,
  child: Container(width: 100, height: 100, color: Colors.blue),
)
```

**Center 组件：**

Center 是 Align 的特例，相当于 alignment 设置为 Alignment.center。

```dart
Center(
  child: Text('居中显示'),
)
```

---

### 1.5 SizedBox

SizedBox 是一个用于设置固定尺寸的组件，也可用于添加固定尺寸的空白间隔。

#### 1.5.1 设置固定尺寸

```dart
SizedBox(
  width: 200,
  height: 100,
  child: ElevatedButton(
    onPressed: () {},
    child: Text('固定尺寸按钮'),
  ),
)
```

#### 1.5.2 作为空白间隔

```dart
Column(
  children: [
    Text('第一行'),
    SizedBox(height: 20),  // 添加20像素的垂直间隔
    Text('第二行'),
  ],
)
```

#### 1.5.3 扩展填满空间

当 SizedBox 的 width 或 height 设置为 double.infinity 时，子组件会填满可用空间。

```dart
SizedBox(
  width: double.infinity,
  height: 50,
  child: ElevatedButton(
    onPressed: () {},
    child: Text('填满宽度'),
  ),
)
```

---

### 1.6 SafeArea

SafeArea 组件用于确保子组件不会被设备的屏幕缺陷区域（如刘海、圆角、底部导航条等）遮挡。

#### 1.6.1 基础用法

```dart
SafeArea(
  child: ListView(
    children: List.generate(50, (index) => ListTile(title: Text('Item $index'))),
  ),
)
```

#### 1.6.2 核心属性

| 属性名                    | 类型       | 默认值          | 用途说明                   |
| ------------------------- | ---------- | --------------- | -------------------------- |
| left                      | bool       | true            | 是否避让左侧屏幕缺陷       |
| top                       | bool       | true            | 是否避让顶部屏幕缺陷       |
| right                     | bool       | true            | 是否避让右侧屏幕缺陷       |
| bottom                    | bool       | true            | 是否避让底部屏幕缺陷       |
| minimum                   | EdgeInsets | EdgeInsets.zero | 最小边距                   |
| maintainBottomViewPadding | bool       | false           | 是否保持底部视图的 padding |

```dart
SafeArea(
  top: false,  // 不避让顶部刘海，常用于需要全屏背景的场景
  minimum: EdgeInsets.all(8),
  child: Container(color: Colors.blue),
)
```

---

## 第2章 文本与图标组件

### 2.1 Text

Text 组件用于显示文本，是 Flutter 中最常用的组件之一。

#### 2.1.1 基础用法

```dart
Text('Hello, Flutter!')
```

#### 2.1.2 核心属性详解

**1. 文本内容相关**

| 属性名   | 类型      | 用途说明           |
| -------- | --------- | ------------------ |
| data     | String?   | 要显示的文本内容   |
| textSpan | TextSpan? | 使用富文本格式显示 |

**2. 样式相关**

| 属性名         | 类型           | 用途说明               |
| -------------- | -------------- | ---------------------- |
| style          | TextStyle?     | 文本样式               |
| textAlign      | TextAlign?     | 文本对齐方式           |
| textDirection  | TextDirection? | 文本方向               |
| softWrap       | bool?          | 是否自动换行           |
| overflow       | TextOverflow?  | 文本溢出时的处理方式   |
| maxLines       | int?           | 最大行数               |
| semanticsLabel | String?        | 语义标签，用于辅助功能 |

**TextStyle 常用属性：**

```dart
Text(
  '样式丰富的文本',
  style: TextStyle(
    color: Colors.blue,                    // 颜色
    fontSize: 24,                          // 字号
    fontWeight: FontWeight.bold,           // 粗细
    fontStyle: FontStyle.italic,           // 斜体
    letterSpacing: 2,                      // 字间距
    wordSpacing: 4,                        // 词间距
    height: 1.5,                           // 行高
    decoration: TextDecoration.underline,  // 装饰线
    decorationColor: Colors.red,           // 装饰线颜色
    decorationStyle: TextDecorationStyle.dashed,  // 装饰线样式
    shadows: [                             // 阴影
      Shadow(color: Colors.grey, blurRadius: 2, offset: Offset(1, 1)),
    ],
  ),
)
```

**TextOverflow 枚举值：**

| 值       | 说明       |
| -------- | ---------- |
| clip     | 直接裁剪   |
| fade     | 渐变消失   |
| ellipsis | 显示省略号 |
| visible  | 溢出可见   |

```dart
Text(
  '这是一段很长的文本，当超出容器宽度时会显示省略号',
  overflow: TextOverflow.ellipsis,
  maxLines: 1,
)
```

#### 2.1.3 富文本 Text.rich

使用 Text.rich 可以创建包含多种样式的文本：

```dart
Text.rich(
  TextSpan(
    text: 'Hello ',
    style: TextStyle(fontSize: 20),
    children: [
      TextSpan(
        text: 'Flutter',
        style: TextStyle(
          fontWeight: FontWeight.bold,
          color: Colors.blue,
        ),
      ),
      TextSpan(text: '!'),
    ],
  ),
)
```

#### 2.1.4 完整示例

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    home: Scaffold(
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              '标题文本',
              style: TextStyle(
                fontSize: 28,
                fontWeight: FontWeight.bold,
                color: Colors.black87,
              ),
            ),
            SizedBox(height: 16),
            Text(
              '这是一段普通的正文文本，用于展示 Text 组件的基本用法。',
              style: TextStyle(
                fontSize: 16,
                color: Colors.grey[700],
                height: 1.6,
              ),
            ),
            SizedBox(height: 16),
            Text.rich(
              TextSpan(
                children: [
                  TextSpan(
                    text: '富文本示例：',
                    style: TextStyle(fontSize: 16),
                  ),
                  TextSpan(
                    text: '蓝色加粗',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                      color: Colors.blue,
                    ),
                  ),
                  TextSpan(
                    text: ' 和 ',
                    style: TextStyle(fontSize: 16),
                  ),
                  TextSpan(
                    text: '红色斜体',
                    style: TextStyle(
                      fontSize: 16,
                      fontStyle: FontStyle.italic,
                      color: Colors.red,
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    ),
  ));
}
```

---

### 2.2 Icon

Icon 组件用于显示 Material Design 图标。

#### 2.2.1 基础用法

```dart
Icon(Icons.home)
```

#### 2.2.2 核心属性

| 属性名        | 类型           | 用途说明           |
| ------------- | -------------- | ------------------ |
| icon          | IconData       | 图标数据，必传参数 |
| size          | double?        | 图标大小           |
| color         | Color?         | 图标颜色           |
| semanticLabel | String?        | 语义标签           |
| textDirection | TextDirection? | 文本方向           |

```dart
Icon(
  Icons.favorite,
  size: 48,
  color: Colors.red,
)
```

#### 2.2.3 常用图标

Flutter 提供了丰富的 Material 图标：

| 图标                | 名称   | 用途     |
| ------------------- | ------ | -------- |
| Icons.home          | 首页   | 导航首页 |
| Icons.search        | 搜索   | 搜索功能 |
| Icons.settings      | 设置   | 设置页面 |
| Icons.person        | 个人   | 个人中心 |
| Icons.favorite      | 收藏   | 收藏功能 |
| Icons.share         | 分享   | 分享功能 |
| Icons.add           | 添加   | 添加操作 |
| Icons.delete        | 删除   | 删除操作 |
| Icons.edit          | 编辑   | 编辑操作 |
| Icons.arrow_back    | 返回   | 返回按钮 |
| Icons.menu          | 菜单   | 菜单按钮 |
| Icons.more_vert     | 更多   | 更多选项 |
| Icons.notifications | 通知   | 通知中心 |
| Icons.shopping_cart | 购物车 | 购物车   |

#### 2.2.4 IconButton

IconButton 是一个可点击的图标按钮：

```dart
IconButton(
  icon: Icon(Icons.favorite),
  color: Colors.red,
  iconSize: 32,
  onPressed: () {
    print('点击了收藏按钮');
  },
)
```

**IconButton 核心属性：**

| 属性名        | 类型                | 用途说明                  |
| ------------- | ------------------- | ------------------------- |
| icon          | Widget              | 图标组件                  |
| onPressed     | VoidCallback?       | 点击回调，null 时按钮禁用 |
| color         | Color?              | 图标颜色                  |
| disabledColor | Color?              | 禁用时的颜色              |
| iconSize      | double?             | 图标大小                  |
| padding       | EdgeInsetsGeometry? | 内边距                    |
| tooltip       | String?             | 提示文本                  |

---

### 2.3 Image

Image 组件用于显示图片，支持多种图片来源。

#### 2.3.1 图片来源

**1. 从网络加载**

```dart
Image.network(
  'https://example.com/image.jpg',
  width: 200,
  height: 200,
  fit: BoxFit.cover,
)
```

**2. 从本地资源加载**

```dart
// 在 pubspec.yaml 中配置资源
// assets:
//   - images/photo.jpg

Image.asset(
  'images/photo.jpg',
  width: 200,
  height: 200,
  fit: BoxFit.cover,
)
```

**3. 从文件加载**

```dart
import 'dart:io';

Image.file(
  File('/path/to/image.jpg'),
  width: 200,
  height: 200,
)
```

**4. 从内存加载**

```dart
import 'dart:typed_data';

Image.memory(
  Uint8List.fromList(imageBytes),
  width: 200,
  height: 200,
)
```

#### 2.3.2 核心属性

| 属性名          | 类型        | 用途说明                     |
| --------------- | ----------- | ---------------------------- |
| width           | double?     | 图片宽度                     |
| height          | double?     | 图片高度                     |
| fit             | BoxFit?     | 图片适应模式                 |
| alignment       | Alignment   | 图片对齐方式                 |
| color           | Color?      | 混合颜色                     |
| colorBlendMode  | BlendMode?  | 颜色混合模式                 |
| repeat          | ImageRepeat | 重复模式                     |
| gaplessPlayback | bool        | 是否在加载新图片时保持旧图片 |

**BoxFit 枚举值：**

| 值        | 说明                           |
| --------- | ------------------------------ |
| contain   | 保持比例，完整显示，可能有留白 |
| cover     | 保持比例，填满容器，可能裁剪   |
| fill      | 拉伸填满，不保持比例           |
| fitWidth  | 保持比例，宽度填满             |
| fitHeight | 保持比例，高度填满             |
| none      | 不缩放，可能裁剪               |
| scaleDown | 等比缩小，不放大               |

#### 2.3.3 加载状态处理

```dart
Image.network(
  'https://example.com/image.jpg',
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Center(
      child: CircularProgressIndicator(
        value: loadingProgress.expectedTotalBytes != null
            ? loadingProgress.cumulativeBytesLoaded /
                loadingProgress.expectedTotalBytes!
            : null,
      ),
    );
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error, color: Colors.red);
  },
)
```

---

## 第3章 按钮组件

### 3.1 ElevatedButton

ElevatedButton 是 Material Design 风格的凸起按钮，具有阴影效果，用于强调重要的操作。

#### 3.1.1 基础用法

```dart
ElevatedButton(
  onPressed: () {
    print('按钮被点击');
  },
  child: Text('点击我'),
)
```

#### 3.1.2 核心属性

| 属性名      | 类型          | 用途说明                  |
| ----------- | ------------- | ------------------------- |
| onPressed   | VoidCallback? | 点击回调，null 时按钮禁用 |
| onLongPress | VoidCallback? | 长按回调                  |
| style       | ButtonStyle?  | 按钮样式                  |
| child       | Widget        | 子组件                    |

#### 3.1.3 样式定制

**使用 styleFrom 方法：**

```dart
ElevatedButton(
  onPressed: () {},
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue,      // 背景颜色
    foregroundColor: Colors.white,     // 前景颜色（文字/图标）
    elevation: 4,                       // 阴影高度
    padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
    minimumSize: Size(120, 48),        // 最小尺寸
    maximumSize: Size(200, 60),        // 最大尺寸
  ),
  child: Text('自定义样式'),
)
```

**使用 ButtonStyle：**

```dart
ElevatedButton(
  onPressed: () {},
  style: ButtonStyle(
    backgroundColor: MaterialStateProperty.all(Colors.blue),
    foregroundColor: MaterialStateProperty.all(Colors.white),
    padding: MaterialStateProperty.all(
      EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    ),
    shape: MaterialStateProperty.all(
      RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(8),
      ),
    ),
  ),
  child: Text('高级定制'),
)
```

> **Flutter框架小知识**
>
> MaterialStateProperty 是一个用于根据按钮的不同状态（正常、悬停、按下、禁用等）返回不同值的类。使用 MaterialStateProperty.resolveWith 可以为不同状态设置不同的样式：
>
> ```dart
> backgroundColor: MaterialStateProperty.resolveWith((states) {
>   if (states.contains(MaterialState.pressed)) {
>     return Colors.blue[700];  // 按下时的颜色
>   }
>   if (states.contains(MaterialState.disabled)) {
>     return Colors.grey;  // 禁用时的颜色
>   }
>   return Colors.blue;  // 正常状态的颜色
> }),
> ```

#### 3.1.4 图标按钮

```dart
ElevatedButton.icon(
  onPressed: () {},
  icon: Icon(Icons.add),
  label: Text('添加'),
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.green,
  ),
)
```

---

### 3.2 TextButton

TextButton 是 Material Design 风格的扁平按钮，没有背景填充，常用于工具栏或对话框中。

#### 3.2.1 基础用法

```dart
TextButton(
  onPressed: () {
    print('扁平按钮被点击');
  },
  child: Text('点击我'),
)
```

#### 3.2.2 样式定制

```dart
TextButton(
  onPressed: () {},
  style: TextButton.styleFrom(
    foregroundColor: Colors.blue,      // 文字颜色
    padding: EdgeInsets.all(16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: Text('自定义样式'),
)
```

#### 3.2.3 图标按钮

```dart
TextButton.icon(
  onPressed: () {},
  icon: Icon(Icons.edit),
  label: Text('编辑'),
)
```

---

### 3.3 OutlinedButton

OutlinedButton 是带边框的轮廓按钮，常用于次要操作。

#### 3.3.1 基础用法

```dart
OutlinedButton(
  onPressed: () {
    print('轮廓按钮被点击');
  },
  child: Text('点击我'),
)
```

#### 3.3.2 样式定制

```dart
OutlinedButton(
  onPressed: () {},
  style: OutlinedButton.styleFrom(
    foregroundColor: Colors.blue,
    side: BorderSide(color: Colors.blue, width: 2),
    padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: Text('自定义边框'),
)
```

#### 3.3.3 图标按钮

```dart
OutlinedButton.icon(
  onPressed: () {},
  icon: Icon(Icons.settings),
  label: Text('设置'),
  style: OutlinedButton.styleFrom(
    side: BorderSide(color: Colors.grey),
  ),
)
```

---

### 3.4 FloatingActionButton

FloatingActionButton（FAB）是悬浮操作按钮，通常位于屏幕右下角，用于执行最重要的操作。

#### 3.4.1 基础用法

```dart
Scaffold(
  floatingActionButton: FloatingActionButton(
    onPressed: () {
      print('悬浮按钮被点击');
    },
    child: Icon(Icons.add),
  ),
  body: Container(),
)
```

#### 3.4.2 核心属性

| 属性名          | 类型          | 默认值 | 用途说明       |
| --------------- | ------------- | ------ | -------------- |
| onPressed       | VoidCallback? | null   | 点击回调       |
| child           | Widget?       | null   | 子组件         |
| backgroundColor | Color?        | null   | 背景颜色       |
| foregroundColor | Color?        | null   | 前景颜色       |
| elevation       | double?       | 6      | 阴影高度       |
| mini            | bool          | false  | 是否使用小尺寸 |
| shape           | ShapeBorder?  | null   | 形状           |
| tooltip         | String?       | null   | 提示文本       |

#### 3.4.3 扩展悬浮按钮

```dart
Scaffold(
  floatingActionButton: FloatingActionButton.extended(
    onPressed: () {},
    icon: Icon(Icons.add),
    label: Text('新建'),
    backgroundColor: Colors.blue,
  ),
  body: Container(),
)
```

#### 3.4.4 定位

```dart
Scaffold(
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.add),
  ),
  floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
  bottomNavigationBar: BottomAppBar(
    shape: CircularNotchedRectangle(),
    child: Container(height: 56),
  ),
  body: Container(),
)
```

---

### 3.5 IconButton

IconButton 是图标按钮，常用于 AppBar 的 actions 中。

#### 3.5.1 基础用法

```dart
IconButton(
  icon: Icon(Icons.search),
  onPressed: () {
    print('搜索按钮被点击');
  },
)
```

#### 3.5.2 核心属性

| 属性名         | 类型                | 用途说明     |
| -------------- | ------------------- | ------------ |
| icon           | Widget              | 图标组件     |
| onPressed      | VoidCallback?       | 点击回调     |
| color          | Color?              | 图标颜色     |
| disabledColor  | Color?              | 禁用时的颜色 |
| iconSize       | double?             | 图标大小     |
| padding        | EdgeInsetsGeometry? | 内边距       |
| tooltip        | String?             | 提示文本     |
| splashColor    | Color?              | 水波纹颜色   |
| highlightColor | Color?              | 高亮颜色     |

#### 3.5.3 样式定制

```dart
IconButton(
  icon: Icon(Icons.favorite),
  color: Colors.red,
  iconSize: 32,
  padding: EdgeInsets.all(12),
  splashColor: Colors.red.withOpacity(0.3),
  tooltip: '收藏',
  onPressed: () {},
)
```

---

## 第4章 输入组件

### 4.1 TextField

TextField 是 Material Design 风格的文本输入框，用于接收用户输入。

#### 4.1.1 基础用法

```dart
TextField()
```

#### 4.1.2 核心属性详解

**1. 控制器与回调**

| 属性名            | 类型                   | 用途说明                      |
| ----------------- | ---------------------- | ----------------------------- |
| controller        | TextEditingController? | 文本控制器，用于获取/设置文本 |
| onChanged         | ValueChanged<String>?  | 文本变化时的回调              |
| onSubmitted       | ValueChanged<String>?  | 提交时的回调                  |
| onEditingComplete | VoidCallback?          | 编辑完成时的回调              |

**2. 装饰样式**

| 属性名     | 类型             | 用途说明       |
| ---------- | ---------------- | -------------- |
| decoration | InputDecoration? | 输入框装饰样式 |
| style      | TextStyle?       | 文本样式       |
| textAlign  | TextAlign?       | 文本对齐方式   |

**3. 键盘与输入**

| 属性名          | 类型             | 用途说明                 |
| --------------- | ---------------- | ------------------------ |
| keyboardType    | TextInputType?   | 键盘类型                 |
| textInputAction | TextInputAction? | 键盘上的操作按钮         |
| obscureText     | bool             | 是否隐藏文本（密码模式） |
| maxLines        | int?             | 最大行数                 |
| maxLength       | int?             | 最大字符数               |
| enabled         | bool?            | 是否启用                 |
| readOnly        | bool             | 是否只读                 |

**InputDecoration 常用属性：**

```dart
TextField(
  decoration: InputDecoration(
    labelText: '用户名',                    // 标签文本
    hintText: '请输入用户名',               // 提示文本
    helperText: '用户名长度为6-20个字符',    // 辅助文本
    prefixIcon: Icon(Icons.person),         // 前缀图标
    suffixIcon: Icon(Icons.clear),          // 后缀图标
    border: OutlineInputBorder(             // 边框
      borderRadius: BorderRadius.circular(8),
    ),
    enabledBorder: OutlineInputBorder(      // 启用时的边框
      borderSide: BorderSide(color: Colors.grey),
    ),
    focusedBorder: OutlineInputBorder(      // 聚焦时的边框
      borderSide: BorderSide(color: Colors.blue),
    ),
    errorBorder: OutlineInputBorder(        // 错误时的边框
      borderSide: BorderSide(color: Colors.red),
    ),
    filled: true,                           // 是否填充背景
    fillColor: Colors.grey[200],            // 填充颜色
    contentPadding: EdgeInsets.all(16),     // 内容内边距
  ),
)
```

**TextInputType 枚举值：**

| 值           | 说明     |
| ------------ | -------- |
| text         | 普通文本 |
| number       | 数字     |
| phone        | 电话号码 |
| emailAddress | 邮箱地址 |
| url          | URL      |
| multiline    | 多行文本 |
| datetime     | 日期时间 |

#### 4.1.3 完整示例

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    home: LoginPage(),
  ));
}

class LoginPage extends StatefulWidget {
  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;

  @override
  void dispose() {
    _usernameController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('登录')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _usernameController,
              decoration: InputDecoration(
                labelText: '用户名',
                hintText: '请输入用户名',
                prefixIcon: Icon(Icons.person),
                border: OutlineInputBorder(),
              ),
              textInputAction: TextInputAction.next,
            ),
            SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              decoration: InputDecoration(
                labelText: '密码',
                hintText: '请输入密码',
                prefixIcon: Icon(Icons.lock),
                suffixIcon: IconButton(
                  icon: Icon(
                    _obscurePassword ? Icons.visibility : Icons.visibility_off,
                  ),
                  onPressed: () {
                    setState(() {
                      _obscurePassword = !_obscurePassword;
                    });
                  },
                ),
                border: OutlineInputBorder(),
              ),
              obscureText: _obscurePassword,
              textInputAction: TextInputAction.done,
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: () {
                print('用户名: ${_usernameController.text}');
                print('密码: ${_passwordController.text}');
              },
              child: Text('登录'),
              style: ElevatedButton.styleFrom(
                minimumSize: Size(double.infinity, 48),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 4.2 Checkbox

Checkbox 是复选框组件，用于表示布尔值状态。

#### 4.2.1 基础用法

```dart
Checkbox(
  value: _isChecked,
  onChanged: (bool? value) {
    setState(() {
      _isChecked = value ?? false;
    });
  },
)
```

#### 4.2.2 核心属性

| 属性名      | 类型                 | 用途说明                    |
| ----------- | -------------------- | --------------------------- |
| value       | bool?                | 当前值，null 表示不确定状态 |
| onChanged   | ValueChanged<bool?>? | 状态变化回调                |
| activeColor | Color?               | 选中时的颜色                |
| checkColor  | Color?               | 对勾的颜色                  |
| tristate    | bool                 | 是否支持三种状态            |

#### 4.2.3 三种状态

```dart
Checkbox(
  value: _value,  // 可以是 true, false, 或 null
  tristate: true,  // 启用三种状态
  onChanged: (bool? value) {
    setState(() {
      _value = value;
    });
  },
)
```

#### 4.2.4 CheckboxListTile

CheckboxListTile 将复选框与列表项结合：

```dart
CheckboxListTile(
  title: Text('选项一'),
  subtitle: Text('这是选项一的描述'),
  secondary: Icon(Icons.settings),
  value: _isChecked,
  onChanged: (bool? value) {
    setState(() {
      _isChecked = value ?? false;
    });
  },
)
```

---

### 4.3 Radio

Radio 是单选按钮组件，用于在一组选项中选择一个。

#### 4.3.1 基础用法

```dart
Radio<String>(
  value: 'option1',
  groupValue: _selectedValue,
  onChanged: (String? value) {
    setState(() {
      _selectedValue = value;
    });
  },
)
```

#### 4.3.2 核心属性

| 属性名                | 类型                   | 用途说明     |
| --------------------- | ---------------------- | ------------ |
| value                 | T                      | 该选项的值   |
| groupValue            | T?                     | 当前选中的值 |
| onChanged             | ValueChanged<T?>?      | 选择变化回调 |
| activeColor           | Color?                 | 选中时的颜色 |
| materialTapTargetSize | MaterialTapTargetSize? | 点击目标大小 |

#### 4.3.3 完整示例

```dart
class RadioDemo extends StatefulWidget {
  @override
  _RadioDemoState createState() => _RadioDemoState();
}

class _RadioDemoState extends State<RadioDemo> {
  String _selectedGender = 'male';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        RadioListTile<String>(
          title: Text('男'),
          value: 'male',
          groupValue: _selectedGender,
          onChanged: (String? value) {
            setState(() {
              _selectedGender = value!;
            });
          },
        ),
        RadioListTile<String>(
          title: Text('女'),
          value: 'female',
          groupValue: _selectedGender,
          onChanged: (String? value) {
            setState(() {
              _selectedGender = value!;
            });
          },
        ),
      ],
    );
  }
}
```

---

### 4.4 Switch

Switch 是开关组件，用于表示开启/关闭状态。

#### 4.4.1 基础用法

```dart
Switch(
  value: _isOn,
  onChanged: (bool value) {
    setState(() {
      _isOn = value;
    });
  },
)
```

#### 4.4.2 核心属性

| 属性名             | 类型                | 用途说明         |
| ------------------ | ------------------- | ---------------- |
| value              | bool                | 当前状态         |
| onChanged          | ValueChanged<bool>? | 状态变化回调     |
| activeColor        | Color?              | 开启时的颜色     |
| activeTrackColor   | Color?              | 开启时轨道的颜色 |
| inactiveThumbColor | Color?              | 关闭时滑块的颜色 |
| inactiveTrackColor | Color?              | 关闭时轨道的颜色 |

#### 4.4.3 自适应风格

```dart
Switch.adaptive(
  value: _isOn,
  onChanged: (bool value) {
    setState(() {
      _isOn = value;
    });
  },
)
```

> 使用 Switch.adaptive 可以根据当前平台自动选择 Material 或 Cupertino 风格。

#### 4.4.4 SwitchListTile

```dart
SwitchListTile(
  title: Text('通知'),
  subtitle: Text('接收推送通知'),
  secondary: Icon(Icons.notifications),
  value: _isOn,
  onChanged: (bool value) {
    setState(() {
      _isOn = value;
    });
  },
)
```

---

### 4.5 Slider

Slider 是滑块组件，用于在一个范围内选择一个值。

#### 4.5.1 基础用法

```dart
Slider(
  value: _currentValue,
  min: 0,
  max: 100,
  onChanged: (double value) {
    setState(() {
      _currentValue = value;
    });
  },
)
```

#### 4.5.2 核心属性

| 属性名        | 类型                  | 默认值 | 用途说明           |
| ------------- | --------------------- | ------ | ------------------ |
| value         | double                | -      | 当前值（必传）     |
| min           | double                | 0.0    | 最小值             |
| max           | double                | 1.0    | 最大值             |
| divisions     | int?                  | null   | 分段数，用于离散值 |
| label         | String?               | null   | 滑块标签           |
| activeColor   | Color?                | null   | 激活部分的颜色     |
| inactiveColor | Color?                | null   | 未激活部分的颜色   |
| onChanged     | ValueChanged<double>? | null   | 值变化回调         |
| onChangeStart | ValueChanged<double>? | null   | 开始拖动时的回调   |
| onChangeEnd   | ValueChanged<double>? | null   | 结束拖动时的回调   |

#### 4.5.3 离散值

```dart
Slider(
  value: _currentValue,
  min: 0,
  max: 100,
  divisions: 10,  // 分成10段，每段10
  label: _currentValue.round().toString(),
  onChanged: (double value) {
    setState(() {
      _currentValue = value;
    });
  },
)
```

#### 5.5.4 范围滑块

```dart
RangeSlider(
  values: _currentRange,
  min: 0,
  max: 100,
  divisions: 10,
  labels: RangeLabels(
    _currentRange.start.round().toString(),
    _currentRange.end.round().toString(),
  ),
  onChanged: (RangeValues values) {
    setState(() {
      _currentRange = values;
    });
  },
)
```

---

## 第5章 列表与网格组件

### 5.1 ListView

ListView 是最常用的滚动列表组件，用于显示一系列可滚动的子组件。

#### 5.1.1 基础用法

```dart
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
  ],
)
```

#### 5.1.2 核心属性

| 属性名          | 类型                | 默认值   | 用途说明               |
| --------------- | ------------------- | -------- | ---------------------- |
| scrollDirection | Axis                | vertical | 滚动方向               |
| reverse         | bool                | false    | 是否反向排列           |
| controller      | ScrollController?   | null     | 滚动控制器             |
| primary         | bool?               | null     | 是否使用主滚动视图     |
| physics         | ScrollPhysics?      | null     | 滚动物理效果           |
| shrinkWrap      | bool                | false    | 是否根据子组件调整高度 |
| padding         | EdgeInsetsGeometry? | null     | 内边距                 |
| itemExtent      | double?             | null     | 子组件固定高度         |
| children        | List<Widget>        | const [] | 子组件列表             |

#### 5.1.3 动态加载（ListView.builder）

当列表项较多时，应使用 ListView.builder 实现懒加载：

```dart
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text('Item $index'),
    );
  },
)
```

**核心属性：**

| 属性名           | 类型                  | 用途说明                             |
| ---------------- | --------------------- | ------------------------------------ |
| itemCount        | int?                  | 列表项总数                           |
| itemBuilder      | IndexedWidgetBuilder  | 列表项构建函数                       |
| separatorBuilder | IndexedWidgetBuilder? | 分隔符构建函数（ListView.separated） |

#### 5.1.4 带分隔符的列表

```dart
ListView.separated(
  itemCount: 20,
  itemBuilder: (context, index) {
    return ListTile(title: Text('Item $index'));
  },
  separatorBuilder: (context, index) {
    return Divider();
  },
)
```

#### 5.1.5 滚动控制器

```dart
class _MyPageState extends State<MyPage> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(() {
      print('当前位置: ${_scrollController.offset}');
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _scrollToTop() {
    _scrollController.animateTo(
      0,
      duration: Duration(milliseconds: 500),
      curve: Curves.easeInOut,
    );
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(title: Text('Item $index'));
      },
    );
  }
}
```

---

### 5.2 ListTile

ListTile 是 Material Design 风格的列表项组件，通常用于 ListView 中。

#### 5.2.1 基础用法

```dart
ListTile(
  title: Text('标题'),
)
```

#### 5.2.2 核心属性

| 属性名         | 类型                      | 用途说明         |
| -------------- | ------------------------- | ---------------- |
| leading        | Widget?                   | 左侧组件         |
| title          | Widget?                   | 标题             |
| subtitle       | Widget?                   | 副标题           |
| trailing       | Widget?                   | 右侧组件         |
| isThreeLine    | bool                      | 是否显示三行     |
| dense          | bool?                     | 是否使用紧凑模式 |
| visualDensity  | VisualDensity?            | 视觉密度         |
| shape          | ShapeBorder?              | 形状             |
| selected       | bool                      | 是否选中         |
| enabled        | bool                      | 是否启用         |
| onTap          | GestureTapCallback?       | 点击回调         |
| onLongPress    | GestureLongPressCallback? | 长按回调         |
| contentPadding | EdgeInsetsGeometry?       | 内容内边距       |

#### 5.2.3 完整示例

```dart
ListTile(
  leading: CircleAvatar(
    backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
  ),
  title: Text('张三'),
  subtitle: Text('你好，最近怎么样？'),
  trailing: Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: [
      Text('10:30', style: TextStyle(fontSize: 12)),
      SizedBox(height: 4),
      Container(
        padding: EdgeInsets.all(4),
        decoration: BoxDecoration(
          color: Colors.red,
          shape: BoxShape.circle,
        ),
        child: Text(
          '3',
          style: TextStyle(color: Colors.white, fontSize: 10),
        ),
      ),
    ],
  ),
  onTap: () {
    print('点击了列表项');
  },
)
```

---

### 5.3 GridView

GridView 是网格布局组件，用于显示二维网格列表。

#### 5.3.1 基础用法

```dart
GridView.count(
  crossAxisCount: 3,  // 每行3列
  children: List.generate(9, (index) {
    return Container(
      color: Colors.blue[(index + 1) * 100],
      child: Center(child: Text('Item $index')),
    );
  }),
)
```

#### 5.3.2 构造函数

| 构造函数           | 用途说明                          |
| ------------------ | --------------------------------- |
| GridView()         | 默认构造函数，需传入 gridDelegate |
| GridView.count()   | 固定列数                          |
| GridView.extent()  | 固定最大宽度                      |
| GridView.builder() | 动态构建，适用于大量数据          |

#### 5.3.3 核心属性

| 属性名          | 类型                  | 用途说明                |
| --------------- | --------------------- | ----------------------- |
| gridDelegate    | SliverGridDelegate    | 网格布局代理            |
| scrollDirection | Axis                  | 滚动方向                |
| reverse         | bool                  | 是否反向                |
| controller      | ScrollController?     | 滚动控制器              |
| physics         | ScrollPhysics?        | 滚动物理效果            |
| shrinkWrap      | bool                  | 是否根据内容调整高度    |
| padding         | EdgeInsetsGeometry?   | 内边距                  |
| itemCount       | int?                  | 项目数量（builder）     |
| itemBuilder     | IndexedWidgetBuilder? | 项目构建函数（builder） |

#### 5.3.4 GridView.builder

```dart
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,        // 2列
    crossAxisSpacing: 10,     // 列间距
    mainAxisSpacing: 10,      // 行间距
    childAspectRatio: 0.75,   // 宽高比
  ),
  itemCount: 100,
  itemBuilder: (context, index) {
    return Card(
      child: Column(
        children: [
          Expanded(
            child: Container(
              color: Colors.blue[(index % 9 + 1) * 100],
            ),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Text('Item $index'),
          ),
        ],
      ),
    );
  },
)
```

#### 5.3.5 SliverGridDelegate

| 类型                                      | 用途说明     |
| ----------------------------------------- | ------------ |
| SliverGridDelegateWithFixedCrossAxisCount | 固定列数     |
| SliverGridDelegateWithMaxCrossAxisExtent  | 固定最大宽度 |

```dart
// 固定列数
SliverGridDelegateWithFixedCrossAxisCount(
  crossAxisCount: 3,
  crossAxisSpacing: 10,
  mainAxisSpacing: 10,
  childAspectRatio: 1.0,
)

// 固定最大宽度
SliverGridDelegateWithMaxCrossAxisExtent(
  maxCrossAxisExtent: 150,  // 每个项目最大宽度150
  crossAxisSpacing: 10,
  mainAxisSpacing: 10,
  childAspectRatio: 1.0,
)
```

---

### 5.4 Card

Card 是 Material Design 风格的卡片组件，具有圆角和阴影效果。

#### 5.4.1 基础用法

```dart
Card(
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Text('卡片内容'),
  ),
)
```

#### 5.4.2 核心属性

| 属性名             | 类型                | 默认值                    | 用途说明       |
| ------------------ | ------------------- | ------------------------- | -------------- |
| color              | Color?              | null                      | 卡片颜色       |
| elevation          | double?             | 1                         | 阴影高度       |
| shape              | ShapeBorder?        | null                      | 形状           |
| borderOnForeground | bool                | true                      | 边框是否在前景 |
| margin             | EdgeInsetsGeometry? | const EdgeInsets.all(4.0) | 外边距         |
| clipBehavior       | Clip                | none                      | 裁剪行为       |
| child              | Widget?             | null                      | 子组件         |

#### 5.4.3 样式定制

```dart
Card(
  elevation: 4,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(16),
  ),
  child: Column(
    mainAxisSize: MainAxisSize.min,
    children: [
      ListTile(
        leading: Icon(Icons.album),
        title: Text('专辑标题'),
        subtitle: Text('艺术家名称'),
      ),
      ButtonBar(
        children: [
          TextButton(onPressed: () {}, child: Text('购买')),
          TextButton(onPressed: () {}, child: Text('试听')),
        ],
      ),
    ],
  ),
)
```

---

### 5.5 ExpansionTile

ExpansionTile 是可展开的列表项，点击后显示更多内容。

#### 5.5.1 基础用法

```dart
ExpansionTile(
  title: Text('可展开项'),
  children: [
    ListTile(title: Text('子项 1')),
    ListTile(title: Text('子项 2')),
    ListTile(title: Text('子项 3')),
  ],
)
```

#### 5.5.2 核心属性

| 属性名                     | 类型                | 用途说明             |
| -------------------------- | ------------------- | -------------------- |
| leading                    | Widget?             | 左侧组件             |
| title                      | Widget              | 标题                 |
| subtitle                   | Widget?             | 副标题               |
| trailing                   | Widget?             | 右侧组件             |
| initiallyExpanded          | bool                | 初始是否展开         |
| maintainState              | bool                | 折叠时是否保持状态   |
| expandedCrossAxisAlignment | CrossAxisAlignment? | 展开内容的交叉轴对齐 |
| expandedAlignment          | Alignment?          | 展开内容的对齐       |
| children                   | List<Widget>        | 展开后的子组件       |
| onExpansionChanged         | ValueChanged<bool>? | 展开状态变化回调     |

#### 5.5.3 完整示例

```dart
ExpansionTile(
  leading: Icon(Icons.settings),
  title: Text('设置'),
  subtitle: Text('应用设置选项'),
  initiallyExpanded: true,
  childrenPadding: EdgeInsets.only(left: 16),
  children: [
    ListTile(
      title: Text('通知'),
      trailing: Switch(value: true, onChanged: (v) {}),
    ),
    ListTile(
      title: Text('隐私'),
      trailing: Icon(Icons.arrow_forward_ios, size: 16),
      onTap: () {},
    ),
    ListTile(
      title: Text('关于'),
      trailing: Icon(Icons.arrow_forward_ios, size: 16),
      onTap: () {},
    ),
  ],
)
```

---

## 第6章 弹窗与对话框组件

### 6.1 AlertDialog

AlertDialog 是 Material Design 风格的警告对话框，用于向用户显示重要信息或请求确认。

#### 6.1.1 基础用法

```dart
showDialog(
  context: context,
  builder: (context) {
    return AlertDialog(
      title: Text('提示'),
      content: Text('确定要删除吗？'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text('取消'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text('确定'),
        ),
      ],
    );
  },
);
```

#### 6.1.2 核心属性

| 属性名           | 类型                | 用途说明       |
| ---------------- | ------------------- | -------------- |
| title            | Widget?             | 标题           |
| content          | Widget?             | 内容           |
| actions          | List<Widget>?       | 操作按钮       |
| titlePadding     | EdgeInsetsGeometry? | 标题内边距     |
| contentPadding   | EdgeInsetsGeometry? | 内容内边距     |
| actionsPadding   | EdgeInsetsGeometry? | 操作按钮内边距 |
| titleTextStyle   | TextStyle?          | 标题文本样式   |
| contentTextStyle | TextStyle?          | 内容文本样式   |
| backgroundColor  | Color?              | 背景颜色       |
| elevation        | double?             | 阴影高度       |
| shape            | ShapeBorder?        | 形状           |
| scrollable       | bool                | 内容是否可滚动 |

#### 6.1.3 完整示例

```dart
Future<bool?> showDeleteConfirmDialog(BuildContext context) {
  return showDialog<bool>(
    context: context,
    builder: (context) {
      return AlertDialog(
        title: Text('确认删除'),
        content: Text('删除后将无法恢复，是否继续？'),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(16),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: Text('取消'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pop(context, true),
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.red,
            ),
            child: Text('删除'),
          ),
        ],
      );
    },
  );
}

// 使用
void _handleDelete() async {
  bool? confirm = await showDeleteConfirmDialog(context);
  if (confirm == true) {
    print('执行删除操作');
  }
}
```

---

### 6.2 SimpleDialog

SimpleDialog 是简单的选项对话框，常用于提供多个选项供用户选择。

#### 6.2.1 基础用法

```dart
showDialog(
  context: context,
  builder: (context) {
    return SimpleDialog(
      title: Text('选择语言'),
      children: [
        SimpleDialogOption(
          onPressed: () => Navigator.pop(context, '中文'),
          child: Text('中文'),
        ),
        SimpleDialogOption(
          onPressed: () => Navigator.pop(context, 'English'),
          child: Text('English'),
        ),
        SimpleDialogOption(
          onPressed: () => Navigator.pop(context, '日本語'),
          child: Text('日本語'),
        ),
      ],
    );
  },
);
```

#### 6.2.2 核心属性

| 属性名          | 类型                | 用途说明   |
| --------------- | ------------------- | ---------- |
| title           | Widget?             | 标题       |
| children        | List<Widget>?       | 选项列表   |
| titlePadding    | EdgeInsetsGeometry? | 标题内边距 |
| contentPadding  | EdgeInsetsGeometry? | 内容内边距 |
| backgroundColor | Color?              | 背景颜色   |
| elevation       | double?             | 阴影高度   |
| shape           | ShapeBorder?        | 形状       |

---

### 6.3 BottomSheet

BottomSheet 是从屏幕底部弹出的面板，常用于显示更多选项或操作。

#### 6.3.1 showModalBottomSheet

```dart
showModalBottomSheet(
  context: context,
  builder: (context) {
    return Container(
      height: 200,
      child: Column(
        children: [
          ListTile(
            leading: Icon(Icons.photo),
            title: Text('从相册选择'),
            onTap: () => Navigator.pop(context),
          ),
          ListTile(
            leading: Icon(Icons.camera),
            title: Text('拍照'),
            onTap: () => Navigator.pop(context),
          ),
          ListTile(
            leading: Icon(Icons.cancel),
            title: Text('取消'),
            onTap: () => Navigator.pop(context),
          ),
        ],
      ),
    );
  },
);
```

#### 6.3.2 核心属性

| 属性名             | 类型            | 用途说明           |
| ------------------ | --------------- | ------------------ |
| backgroundColor    | Color?          | 背景颜色           |
| elevation          | double?         | 阴影高度           |
| shape              | ShapeBorder?    | 形状               |
| clipBehavior       | Clip?           | 裁剪行为           |
| constraints        | BoxConstraints? | 尺寸约束           |
| isScrollControlled | bool            | 是否由内容控制高度 |
| enableDrag         | bool            | 是否允许拖动关闭   |
| isDismissible      | bool            | 是否可点击外部关闭 |

#### 6.3.3 全屏底部弹窗

```dart
showModalBottomSheet(
  context: context,
  isScrollControlled: true,
  backgroundColor: Colors.transparent,
  builder: (context) {
    return DraggableScrollableSheet(
      initialChildSize: 0.9,
      minChildSize: 0.5,
      maxChildSize: 0.95,
      builder: (context, scrollController) {
        return Container(
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
          ),
          child: ListView.builder(
            controller: scrollController,
            itemCount: 50,
            itemBuilder: (context, index) {
              return ListTile(title: Text('Item $index'));
            },
          ),
        );
      },
    );
  },
);
```

---

### 6.4 SnackBar

SnackBar 是底部弹出的轻量级提示消息，用于显示操作反馈。

#### 6.4.1 基础用法

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Text('操作成功'),
    duration: Duration(seconds: 2),
  ),
);
```

#### 6.4.2 核心属性

| 属性名          | 类型                | 默认值 | 用途说明               |
| --------------- | ------------------- | ------ | ---------------------- |
| content         | Widget              | -      | 内容（必传）           |
| duration        | Duration            | 4秒    | 显示时长               |
| action          | SnackBarAction?     | null   | 操作按钮               |
| backgroundColor | Color?              | null   | 背景颜色               |
| elevation       | double?             | 6      | 阴影高度               |
| shape           | ShapeBorder?        | null   | 形状                   |
| behavior        | SnackBarBehavior?   | fixed  | 显示模式               |
| margin          | EdgeInsetsGeometry? | null   | 外边距（floating模式） |
| padding         | EdgeInsetsGeometry? | null   | 内边距                 |

**SnackBarBehavior 枚举值：**

| 值       | 说明                 |
| -------- | -------------------- |
| fixed    | 固定在底部，占满宽度 |
| floating | 悬浮显示，可设置边距 |

#### 6.4.3 完整示例

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: Row(
      children: [
        Icon(Icons.check_circle, color: Colors.white),
        SizedBox(width: 8),
        Text('文件已保存'),
      ],
    ),
    backgroundColor: Colors.green,
    behavior: SnackBarBehavior.floating,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
    margin: EdgeInsets.all(16),
    action: SnackBarAction(
      label: '撤销',
      textColor: Colors.white,
      onPressed: () {
        print('撤销操作');
      },
    ),
    duration: Duration(seconds: 3),
  ),
);
```

---

## 第7章 进度与指示器组件

### 7.1 CircularProgressIndicator

CircularProgressIndicator 是 Material Design 风格的圆形进度指示器。

#### 7.1.1 基础用法

```dart
// 不确定进度（循环旋转）
CircularProgressIndicator()

// 确定进度
CircularProgressIndicator(
  value: 0.7,  // 0.0 ~ 1.0
)
```

#### 7.1.2 核心属性

| 属性名          | 类型    | 用途说明                        |
| --------------- | ------- | ------------------------------- |
| value           | double? | 当前进度值（null 为不确定进度） |
| backgroundColor | Color?  | 背景颜色                        |
| color           | Color?  | 进度条颜色                      |
| strokeWidth     | double  | 线条宽度（默认 4.0）            |
| semanticsLabel  | String? | 语义标签                        |
| semanticsValue  | String? | 语义值                          |

#### 7.1.3 样式定制

```dart
CircularProgressIndicator(
  value: 0.75,
  strokeWidth: 8,
  backgroundColor: Colors.grey[300],
  color: Colors.blue,
)
```

---

### 7.2 LinearProgressIndicator

LinearProgressIndicator 是 Material Design 风格的线性进度指示器。

#### 7.2.1 基础用法

```dart
// 不确定进度
LinearProgressIndicator()

// 确定进度
LinearProgressIndicator(
  value: 0.5,
)
```

#### 7.2.2 核心属性

| 属性名          | 类型    | 用途说明   |
| --------------- | ------- | ---------- |
| value           | double? | 当前进度值 |
| backgroundColor | Color?  | 背景颜色   |
| color           | Color?  | 进度条颜色 |
| minHeight       | double? | 最小高度   |
| semanticsLabel  | String? | 语义标签   |
| semanticsValue  | String? | 语义值     |

#### 7.2.3 完整示例

```dart
Column(
  children: [
    LinearProgressIndicator(
      value: _progress,
      backgroundColor: Colors.grey[300],
      color: Colors.green,
      minHeight: 10,
    ),
    SizedBox(height: 8),
    Text('${(_progress * 100).toInt()}%'),
  ],
)
```

---

### 7.3 RefreshIndicator

RefreshIndicator 用于实现下拉刷新功能。

#### 7.3.1 基础用法

```dart
RefreshIndicator(
  onRefresh: () async {
    // 刷新逻辑
    await Future.delayed(Duration(seconds: 2));
  },
  child: ListView(
    children: [
      ListTile(title: Text('Item 1')),
      ListTile(title: Text('Item 2')),
    ],
  ),
)
```

#### 7.3.2 核心属性

| 属性名          | 类型                        | 默认值 | 用途说明         |
| --------------- | --------------------------- | ------ | ---------------- |
| onRefresh       | RefreshCallback             | -      | 刷新回调（必传） |
| color           | Color?                      | null   | 进度指示器颜色   |
| backgroundColor | Color?                      | null   | 背景颜色         |
| displacement    | double                      | 40.0   | 触发距离         |
| edgeOffset      | double                      | 0.0    | 边缘偏移         |
| strokeWidth     | double                      | 2.0    | 线条宽度         |
| triggerMode     | RefreshIndicatorTriggerMode | onEdge | 触发模式         |

```dart
RefreshIndicator(
  onRefresh: _refreshData,
  color: Colors.white,
  backgroundColor: Colors.blue,
  displacement: 60,
  strokeWidth: 3,
  child: ListView.builder(
    itemCount: _items.length,
    itemBuilder: (context, index) {
      return ListTile(title: Text(_items[index]));
    },
  ),
)
```

---

## 第8章 导航组件

### 8.1 AppBar

AppBar 是 Material Design 风格的顶部应用栏。

#### 8.1.1 基础用法

```dart
Scaffold(
  appBar: AppBar(
    title: Text('首页'),
  ),
  body: Container(),
)
```

#### 8.1.2 核心属性

| 属性名           | 类型                 | 用途说明              |
| ---------------- | -------------------- | --------------------- |
| leading          | Widget?              | 左侧组件              |
| title            | Widget?              | 标题                  |
| actions          | List<Widget>?        | 右侧操作按钮          |
| bottom           | PreferredSizeWidget? | 底部组件（如 TabBar） |
| elevation        | double?              | 阴影高度              |
| shadowColor      | Color?               | 阴影颜色              |
| backgroundColor  | Color?               | 背景颜色              |
| foregroundColor  | Color?               | 前景颜色              |
| iconTheme        | IconThemeData?       | 图标主题              |
| actionsIconTheme | IconThemeData?       | 操作按钮图标主题      |
| centerTitle      | bool?                | 标题是否居中          |
| titleSpacing     | double?              | 标题间距              |
| toolbarHeight    | double?              | 工具栏高度            |
| shape            | ShapeBorder?         | 形状                  |
| flexibleSpace    | Widget?              | 灵活空间              |

#### 8.1.3 完整示例

```dart
AppBar(
  leading: IconButton(
    icon: Icon(Icons.menu),
    onPressed: () {},
  ),
  title: Text('首页'),
  centerTitle: true,
  actions: [
    IconButton(
      icon: Icon(Icons.search),
      onPressed: () {},
    ),
    IconButton(
      icon: Icon(Icons.more_vert),
      onPressed: () {},
    ),
  ],
  backgroundColor: Colors.blue,
  foregroundColor: Colors.white,
  elevation: 4,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(
      bottom: Radius.circular(16),
    ),
  ),
)
```

---

### 8.2 BottomNavigationBar

BottomNavigationBar 是底部导航栏组件。

#### 8.2.1 基础用法

```dart
Scaffold(
  bottomNavigationBar: BottomNavigationBar(
    currentIndex: _selectedIndex,
    onTap: (index) {
      setState(() {
        _selectedIndex = index;
      });
    },
    items: [
      BottomNavigationBarItem(
        icon: Icon(Icons.home),
        label: '首页',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.search),
        label: '发现',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.person),
        label: '我的',
      ),
    ],
  ),
  body: _pages[_selectedIndex],
)
```

#### 8.2.2 核心属性

| 属性名               | 类型                                | 默认值 | 用途说明           |
| -------------------- | ----------------------------------- | ------ | ------------------ |
| items                | List<BottomNavigationBarItem>       | -      | 导航项列表（必传） |
| onTap                | ValueChanged<int>?                  | null   | 点击回调           |
| currentIndex         | int                                 | 0      | 当前选中索引       |
| elevation            | double?                             | null   | 阴影高度           |
| type                 | BottomNavigationBarType?            | null   | 类型               |
| fixedColor           | Color?                              | null   | 固定颜色（已弃用） |
| backgroundColor      | Color?                              | null   | 背景颜色           |
| iconSize             | double                              | 24     | 图标大小           |
| selectedItemColor    | Color?                              | null   | 选中项颜色         |
| unselectedItemColor  | Color?                              | null   | 未选中项颜色       |
| selectedIconTheme    | IconThemeData?                      | null   | 选中图标主题       |
| unselectedIconTheme  | IconThemeData?                      | null   | 未选中图标主题     |
| selectedFontSize     | double                              | 14     | 选中字体大小       |
| unselectedFontSize   | double                              | 12     | 未选中字体大小     |
| selectedLabelStyle   | TextStyle?                          | null   | 选中标签样式       |
| unselectedLabelStyle | TextStyle?                          | null   | 未选中标签样式     |
| showSelectedLabels   | bool?                               | null   | 是否显示选中标签   |
| showUnselectedLabels | bool?                               | null   | 是否显示未选中标签 |
| enableFeedback       | bool?                               | null   | 是否启用反馈       |
| landscapeLayout      | BottomNavigationBarLandscapeLayout? | null   | 横屏布局           |

**BottomNavigationBarType 枚举值：**

| 值       | 说明                       |
| -------- | -------------------------- |
| fixed    | 固定宽度，所有项都显示标签 |
| shifting | 选中项放大，其他项隐藏标签 |

---

### 8.3 Drawer

Drawer 是侧边抽屉导航组件。

#### 8.3.1 基础用法

```dart
Scaffold(
  appBar: AppBar(title: Text('首页')),
  drawer: Drawer(
    child: ListView(
      padding: EdgeInsets.zero,
      children: [
        DrawerHeader(
          decoration: BoxDecoration(
            color: Colors.blue,
          ),
          child: Text(
            '菜单',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
            ),
          ),
        ),
        ListTile(
          leading: Icon(Icons.home),
          title: Text('首页'),
          onTap: () {
            Navigator.pop(context);
          },
        ),
        ListTile(
          leading: Icon(Icons.settings),
          title: Text('设置'),
          onTap: () {
            Navigator.pop(context);
          },
        ),
      ],
    ),
  ),
  body: Container(),
)
```

#### 8.3.2 核心属性

| 属性名          | 类型         | 默认值 | 用途说明       |
| --------------- | ------------ | ------ | -------------- |
| child           | Widget       | -      | 子组件（必传） |
| backgroundColor | Color?       | null   | 背景颜色       |
| elevation       | double       | 16     | 阴影高度       |
| shape           | ShapeBorder? | null   | 形状           |
| semanticLabel   | String?      | null   | 语义标签       |

---

### 8.4 TabBar 与 TabBarView

TabBar 和 TabBarView 配合使用实现标签页切换功能。

#### 8.4.1 基础用法

```dart
class TabDemo extends StatefulWidget {
  @override
  _TabDemoState createState() => _TabDemoState();
}

class _TabDemoState extends State<TabDemo>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('标签页'),
        bottom: TabBar(
          controller: _tabController,
          tabs: [
            Tab(icon: Icon(Icons.home), text: '首页'),
            Tab(icon: Icon(Icons.business), text: '商务'),
            Tab(icon: Icon(Icons.school), text: '学习'),
          ],
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: [
          Center(child: Text('首页内容')),
          Center(child: Text('商务内容')),
          Center(child: Text('学习内容')),
        ],
      ),
    );
  }
}
```

#### 8.4.2 TabBar 核心属性

| 属性名               | 类型                 | 用途说明         |
| -------------------- | -------------------- | ---------------- |
| tabs                 | List<Widget>         | 标签列表（必传） |
| controller           | TabController?       | 标签控制器       |
| isScrollable         | bool                 | 是否可滚动       |
| indicatorColor       | Color?               | 指示器颜色       |
| indicatorWeight      | double               | 指示器高度       |
| indicatorPadding     | EdgeInsetsGeometry   | 指示器内边距     |
| indicator            | Decoration?          | 指示器装饰       |
| indicatorSize        | TabBarIndicatorSize? | 指示器大小       |
| labelColor           | Color?               | 标签颜色         |
| labelStyle           | TextStyle?           | 标签样式         |
| unselectedLabelColor | Color?               | 未选中标签颜色   |
| unselectedLabelStyle | TextStyle?           | 未选中标签样式   |

---

## 第9章 其他常用组件

### 9.1 Chip

Chip 是 Material Design 风格的标签组件。

#### 9.1.1 基础用法

```dart
Chip(
  label: Text('标签'),
)
```

#### 9.1.2 核心属性

| 属性名                     | 类型                   | 用途说明         |
| -------------------------- | ---------------------- | ---------------- |
| avatar                     | Widget?                | 左侧头像         |
| label                      | Widget                 | 标签文本（必传） |
| labelStyle                 | TextStyle?             | 标签样式         |
| labelPadding               | EdgeInsetsGeometry?    | 标签内边距       |
| deleteIcon                 | Widget?                | 删除图标         |
| onDeleted                  | VoidCallback?          | 删除回调         |
| deleteIconColor            | Color?                 | 删除图标颜色     |
| deleteButtonTooltipMessage | String?                | 删除按钮提示     |
| side                       | BorderSide?            | 边框             |
| shape                      | OutlinedBorder?        | 形状             |
| backgroundColor            | Color?                 | 背景颜色         |
| padding                    | EdgeInsetsGeometry?    | 内边距           |
| elevation                  | double?                | 阴影高度         |
| shadowColor                | Color?                 | 阴影颜色         |
| surfaceTintColor           | Color?                 | 表面色调         |
| materialTapTargetSize      | MaterialTapTargetSize? | 点击目标大小     |

#### 9.1.3 相关组件

| 组件名     | 用途说明         |
| ---------- | ---------------- |
| ActionChip | 可点击的操作标签 |
| ChoiceChip | 单选标签         |
| FilterChip | 过滤标签         |
| InputChip  | 输入标签         |

```dart
// ActionChip
ActionChip(
  avatar: Icon(Icons.favorite),
  label: Text('收藏'),
  onPressed: () {},
)

// ChoiceChip
ChoiceChip(
  label: Text('选项一'),
  selected: _isSelected,
  onSelected: (bool selected) {
    setState(() {
      _isSelected = selected;
    });
  },
)

// FilterChip
FilterChip(
  label: Text('筛选'),
  selected: _isSelected,
  onSelected: (bool selected) {
    setState(() {
      _isSelected = selected;
    });
  },
)
```

---

### 9.2 Tooltip

Tooltip 是提示信息组件，长按或悬停时显示提示文本。

#### 9.2.1 基础用法

```dart
Tooltip(
  message: '这是一个提示',
  child: Icon(Icons.info),
)
```

#### 9.2.2 核心属性

| 属性名               | 类型                | 默认值 | 用途说明           |
| -------------------- | ------------------- | ------ | ------------------ |
| message              | String              | -      | 提示文本（必传）   |
| child                | Widget              | -      | 子组件（必传）     |
| height               | double?             | null   | 高度               |
| padding              | EdgeInsetsGeometry? | null   | 内边距             |
| margin               | EdgeInsetsGeometry? | null   | 外边距             |
| verticalOffset       | double?             | null   | 垂直偏移           |
| preferBelow          | bool?               | null   | 是否优先显示在下方 |
| excludeFromSemantics | bool                | false  | 是否排除语义       |
| decoration           | Decoration?         | null   | 装饰               |
| textStyle            | TextStyle?          | null   | 文本样式           |
| textAlign            | TextAlign?          | null   | 文本对齐           |
| waitDuration         | Duration?           | null   | 等待显示时长       |
| showDuration         | Duration?           | null   | 显示时长           |
| triggerMode          | TooltipTriggerMode? | null   | 触发模式           |
| enableFeedback       | bool?               | null   | 是否启用反馈       |

---

### 9.3 Divider

Divider 是分隔线组件。

#### 9.3.1 基础用法

```dart
Divider()
```

#### 9.3.2 核心属性

| 属性名    | 类型    | 默认值 | 用途说明 |
| --------- | ------- | ------ | -------- |
| height    | double? | null   | 高度     |
| thickness | double? | null   | 线条厚度 |
| indent    | double  | 0      | 起始缩进 |
| endIndent | double  | 0      | 结束缩进 |
| color     | Color?  | null   | 颜色     |

#### 9.3.3 垂直分隔线

```dart
VerticalDivider(
  width: 20,
  thickness: 1,
  color: Colors.grey,
)
```

---

### 9.4 CircleAvatar

CircleAvatar 是圆形头像组件。

#### 9.4.1 基础用法

```dart
CircleAvatar(
  child: Text('A'),
)
```

#### 9.4.2 核心属性

| 属性名                 | 类型                | 用途说明         |
| ---------------------- | ------------------- | ---------------- |
| child                  | Widget?             | 子组件           |
| backgroundColor        | Color?              | 背景颜色         |
| backgroundImage        | ImageProvider?      | 背景图片         |
| foregroundImage        | ImageProvider?      | 前景图片         |
| onBackgroundImageError | ImageErrorListener? | 背景图片错误回调 |
| onForegroundImageError | ImageErrorListener? | 前景图片错误回调 |
| radius                 | double?             | 半径             |
| minRadius              | double?             | 最小半径         |
| maxRadius              | double?             | 最大半径         |

#### 9.4.3 完整示例

```dart
// 文字头像
CircleAvatar(
  backgroundColor: Colors.blue,
  foregroundColor: Colors.white,
  radius: 30,
  child: Text(
    'AB',
    style: TextStyle(fontSize: 20),
  ),
)

// 图片头像
CircleAvatar(
  radius: 40,
  backgroundImage: NetworkImage('https://example.com/avatar.jpg'),
)
```

---

### 9.5 Badge

Badge 是徽章组件，用于显示计数或状态标记。

#### 9.5.1 基础用法

```dart
Badge(
  child: Icon(Icons.notifications),
)
```

#### 9.5.2 核心属性

| 属性名            | 类型                   | 默认值                        | 用途说明           |
| ----------------- | ---------------------- | ----------------------------- | ------------------ |
| child             | Widget?                | null                          | 子组件             |
| label             | Widget?                | null                          | 标签内容           |
| backgroundColor   | Color?                 | null                          | 背景颜色           |
| textColor         | Color?                 | null                          | 文本颜色           |
| smallSize         | double                 | 6                             | 小尺寸（无标签时） |
| largeSize         | double                 | 16                            | 大尺寸（有标签时） |
| textStyle         | TextStyle?             | null                          | 文本样式           |
| padding           | EdgeInsetsGeometry?    | null                          | 内边距             |
| alignment         | AlignmentGeometry      | Alignment.topEnd              | 对齐方式           |
| offset            | Offset?                | null                          | 偏移               |
| isLabelVisible    | bool                   | true                          | 标签是否可见       |
| overflowAlignment | BadgeOverflowAlignment | BadgeOverflowAlignment.scroll | 溢出对齐           |

#### 9.5.3 完整示例

```dart
Badge(
  label: Text('99+'),
  backgroundColor: Colors.red,
  textColor: Colors.white,
  child: IconButton(
    icon: Icon(Icons.notifications),
    onPressed: () {},
  ),
)
```

---

## 第10章 动画组件

### 10.1 AnimatedContainer

AnimatedContainer 是 Container 的动画版本，当属性变化时会自动产生过渡动画。

#### 10.1.1 基础用法

```dart
class AnimatedContainerDemo extends StatefulWidget {
  @override
  _AnimatedContainerDemoState createState() => _AnimatedContainerDemoState();
}

class _AnimatedContainerDemoState extends State<AnimatedContainerDemo> {
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        setState(() {
          _isExpanded = !_isExpanded;
        });
      },
      child: AnimatedContainer(
        duration: Duration(milliseconds: 300),
        width: _isExpanded ? 200 : 100,
        height: _isExpanded ? 200 : 100,
        color: _isExpanded ? Colors.blue : Colors.red,
        curve: Curves.easeInOut,
        child: Center(child: Text('点击我')),
      ),
    );
  }
}
```

#### 10.1.2 核心属性

| 属性名   | 类型          | 用途说明         |
| -------- | ------------- | ---------------- | ------------ |
| duration | Duration      | 动画时长（必传） |
| curve    | Curve         | Curves.linear    | 动画曲线     |
| onEnd    | VoidCallback? | null             | 动画结束回调 |

> 其他属性与 Container 相同，包括 width、height、color、decoration、padding、margin、alignment 等。

---

### 10.2 AnimatedOpacity

AnimatedOpacity 是 Opacity 的动画版本，用于实现淡入淡出效果。

#### 10.2.1 基础用法

```dart
AnimatedOpacity(
  opacity: _isVisible ? 1.0 : 0.0,
  duration: Duration(milliseconds: 500),
  child: Text('淡入淡出的文本'),
)
```

#### 10.2.2 核心属性

| 属性名                 | 类型          | 默认值        | 用途说明         |
| ---------------------- | ------------- | ------------- | ---------------- |
| opacity                | double        | -             | 不透明度（必传） |
| duration               | Duration      | -             | 动画时长（必传） |
| curve                  | Curve         | Curves.linear | 动画曲线         |
| onEnd                  | VoidCallback? | null          | 动画结束回调     |
| child                  | Widget?       | null          | 子组件           |
| alwaysIncludeSemantics | bool          | false         | 是否始终包含语义 |

---

### 10.3 AnimatedSwitcher

AnimatedSwitcher 用于在子组件切换时添加过渡动画。

#### 10.3.1 基础用法

```dart
AnimatedSwitcher(
  duration: Duration(milliseconds: 300),
  child: _isLoading
      ? CircularProgressIndicator(key: ValueKey('loading'))
      : Icon(Icons.check, key: ValueKey('success')),
)
```

#### 10.3.2 核心属性

| 属性名            | 类型                              | 默认值        | 用途说明         |
| ----------------- | --------------------------------- | ------------- | ---------------- |
| child             | Widget?                           | null          | 子组件           |
| duration          | Duration                          | -             | 动画时长（必传） |
| reverseDuration   | Duration?                         | null          | 反向动画时长     |
| switchInCurve     | Curve                             | Curves.linear | 进入动画曲线     |
| switchOutCurve    | Curve                             | Curves.linear | 退出动画曲线     |
| transitionBuilder | AnimatedSwitcherTransitionBuilder | 默认淡入淡出  | 过渡动画构建器   |
| layoutBuilder     | AnimatedSwitcherLayoutBuilder     | 默认布局      | 布局构建器       |

#### 10.3.3 自定义过渡动画

```dart
AnimatedSwitcher(
  duration: Duration(milliseconds: 300),
  transitionBuilder: (child, animation) {
    return ScaleTransition(
      scale: animation,
      child: FadeTransition(
        opacity: animation,
        child: child,
      ),
    );
  },
  child: Text(
    '$_count',
    key: ValueKey<int>(_count),
    style: TextStyle(fontSize: 48),
  ),
)
```

> **Dart Tips语法小贴士**
>
> ValueKey 用于帮助 Flutter 区分不同的子组件。当子组件类型相同但内容不同时，需要使用不同的 key，否则 AnimatedSwitcher 无法正确识别组件变化。

---

## 附录：Material 组件速查表

### 布局组件

| 组件名     | 用途       |
| ---------- | ---------- |
| Container  | 多功能容器 |
| Row        | 水平布局   |
| Column     | 垂直布局   |
| Stack      | 层叠布局   |
| Positioned | 绝对定位   |
| Padding    | 内边距     |
| Align      | 对齐       |
| Center     | 居中       |
| SizedBox   | 固定尺寸   |
| Expanded   | 扩展填充   |
| Flexible   | 弹性布局   |
| SafeArea   | 安全区域   |

### 按钮组件

| 组件名               | 用途     |
| -------------------- | -------- |
| ElevatedButton       | 凸起按钮 |
| TextButton           | 扁平按钮 |
| OutlinedButton       | 轮廓按钮 |
| FloatingActionButton | 悬浮按钮 |
| IconButton           | 图标按钮 |

### 输入组件

| 组件名    | 用途     |
| --------- | -------- |
| TextField | 文本输入 |
| Checkbox  | 复选框   |
| Radio     | 单选按钮 |
| Switch    | 开关     |
| Slider    | 滑块     |

### 列表组件

| 组件名        | 用途         |
| ------------- | ------------ |
| ListView      | 列表         |
| ListTile      | 列表项       |
| GridView      | 网格         |
| Card          | 卡片         |
| ExpansionTile | 可展开列表项 |

### 弹窗组件

| 组件名       | 用途       |
| ------------ | ---------- |
| AlertDialog  | 警告对话框 |
| SimpleDialog | 简单对话框 |
| BottomSheet  | 底部面板   |
| SnackBar     | 轻量提示   |

### 导航组件

| 组件名              | 用途       |
| ------------------- | ---------- |
| AppBar              | 应用栏     |
| BottomNavigationBar | 底部导航   |
| Drawer              | 侧边抽屉   |
| TabBar              | 标签栏     |
| TabBarView          | 标签页内容 |

### 进度组件

| 组件名                    | 用途       |
| ------------------------- | ---------- |
| CircularProgressIndicator | 圆形进度条 |
| LinearProgressIndicator   | 线性进度条 |
| RefreshIndicator          | 下拉刷新   |

### 其他组件

| 组件名       | 用途     |
| ------------ | -------- |
| Chip         | 标签     |
| Tooltip      | 提示     |
| Divider      | 分隔线   |
| CircleAvatar | 圆形头像 |
| Badge        | 徽章     |

---

## 结语

本书详细介绍了 Flutter 3.41 版本中 Material 库的全部组件，从基础布局到复杂交互，从静态展示到动态动画，涵盖了开发 Flutter 应用所需的各种组件知识。希望本书能够帮助读者深入理解 Flutter Material 组件的使用方法，在实际开发中灵活运用这些组件，构建出美观、流畅、易用的跨平台应用程序。

Flutter 框架在不断发展，新的组件和功能会持续添加。建议读者关注 Flutter 官方文档和社区动态，及时了解最新的组件和最佳实践。

---

**版本信息**

- Flutter 版本：3.41.0
- Dart SDK 版本：3.x
- 文档更新时间：2026年2月
