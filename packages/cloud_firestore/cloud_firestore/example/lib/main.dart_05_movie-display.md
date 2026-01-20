# main.dart - 电影显示组件

## 概述

本文档解释 Cloud Firestore 示例应用中的电影显示组件，包括 _MovieItem 组件及其子组件的实现。

## _MovieItem - 电影项组件

```dart 316:415:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// A single movie row.
class _MovieItem extends StatelessWidget {
  _MovieItem(this.movie, this.reference);

  final Movie movie;
  final DocumentReference<Movie> reference;

  /// Returns the movie poster.
  Widget get poster {
    return SizedBox(
      width: 100,
      child: Image.network(movie.poster),
    );
  }

  /// Returns movie details.
  Widget get details {
    return Padding(
      padding: const EdgeInsets.only(left: 8, right: 8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          title,
          metadata,
          genres,
          Likes(
            reference: reference,
            currentLikes: movie.likes,
          ),
        ],
      ),
    );
  }

  /// Return the movie title.
  Widget get title {
    return Text(
      '${movie.title} (${movie.year})',
      style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
    );
  }

  /// Returns metadata about the movie.
  Widget get metadata {
    return Padding(
      padding: const EdgeInsets.only(top: 8),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Padding(
            padding: const EdgeInsets.only(right: 8),
            child: Text('Rated: ${movie.rated}'),
          ),
          Text('Runtime: ${movie.runtime}'),
        ],
      ),
    );
  }

  /// Returns a list of genre movie tags.
  List<Widget> get genreItems {
    return [
      for (final genre in movie.genre)
        Padding(
          padding: const EdgeInsets.only(right: 2),
          child: Chip(
            backgroundColor: Colors.lightBlue,
            label: Text(
              genre,
              style: const TextStyle(color: Colors.white),
            ),
          ),
        ),
    ];
  }

  /// Returns all genres.
  Widget get genres {
    return Padding(
      padding: const EdgeInsets.only(top: 8),
      child: Wrap(
        children: genreItems,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 4, top: 4),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          poster,
          Flexible(child: details),
        ],
      ),
    );
  }
}
```

## 组件结构分析

### 构造函数和属性

```dart
_MovieItem(this.movie, this.reference);

final Movie movie;
final DocumentReference<Movie> reference;
```

- 接收 Movie 数据对象和对应的 Firestore 文档引用
- 使用类型安全的 DocumentReference<Movie\>

### 组件布局

- **水平布局**: Row 组件水平排列海报和详情
- **顶部对齐**: `CrossAxisAlignment.start` 确保内容顶部对齐
- **响应式宽度**: Flexible 组件使详情区域自适应宽度

## 海报显示组件

```dart 323:329:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Returns the movie poster.
Widget get poster {
  return SizedBox(
    width: 100,
    child: Image.network(movie.poster),
  );
}
```

### 设计特性

- **固定宽度**: 100 像素宽度确保布局一致性
- **网络图片**: 使用 `Image.network()` 加载远程海报
- **Getter 方法**: 保持组件代码的整洁性

## 电影详情组件

```dart 331:348:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Returns movie details.
Widget get details {
  return Padding(
    padding: const EdgeInsets.only(left: 8, right: 8),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        title,
        metadata,
        genres,
        Likes(
          reference: reference,
          currentLikes: movie.likes,
        ),
      ],
    ),
  );
}
```

### 布局结构

- **垂直排列**: Column 组件垂直排列各个信息块
- **左侧对齐**: 所有内容左对齐显示
- **水平内边距**: 左右各 8 像素的内边距

### 组件层次

1. **标题**: 电影名称和年份
2. **元数据**: 评级和时长
3. **类型标签**: 电影类型 Chip 组件
4. **点赞组件**: 交互式的点赞功能

## 标题组件

```dart 350:356:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Return the movie title.
Widget get title {
  return Text(
    '${movie.title} (${movie.year})',
    style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
  );
}
```

### 样式设计

- **字体大小**: 18 像素
- **粗体样式**: `FontWeight.bold` 突出显示
- **信息整合**: 标题和年份合并显示

## 元数据组件

```dart 358:373:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Returns metadata about the movie.
Widget get metadata {
  return Padding(
    padding: const EdgeInsets.only(top: 8),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Padding(
          padding: const EdgeInsets.only(right: 8),
          child: Text('Rated: ${movie.rated}'),
        ),
        Text('Runtime: ${movie.runtime}'),
      ],
    ),
  );
}
```

### 信息展示

- **评级信息**: "Rated: PG-13" 格式
- **时长信息**: "Runtime: 148 min" 格式
- **垂直间距**: 顶部 8 像素间距

## 类型标签组件

```dart 375:400:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Returns a list of genre movie tags.
List<Widget> get genreItems {
  return [
    for (final genre in movie.genre)
      Padding(
        padding: const EdgeInsets.only(right: 2),
        child: Chip(
          backgroundColor: Colors.lightBlue,
          label: Text(
            genre,
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ),
  ];
}

/// Returns all genres.
Widget get genres {
  return Padding(
    padding: const EdgeInsets.only(top: 8),
    child: Wrap(
      children: genreItems,
    ),
  );
}
```

### Chip 组件特性

- **集合遍历**: 为每个类型创建 Chip 组件
- **颜色设计**: 浅蓝色背景，白色文字
- **水平间距**: 每个 Chip 右侧 2 像素间距

### Wrap 布局

- **自动换行**: 当类型标签过多时自动换行
- **灵活布局**: 适应不同数量的类型标签
- **顶部间距**: 与上一个组件保持视觉间距

## 组件架构优势

### 代码组织

- **Getter 方法**: 将复杂组件分解为可管理的部分
- **关注点分离**: 每个 getter 负责特定的 UI 部分
- **可读性**: 代码结构清晰，易于理解和维护

### 布局设计

- **响应式设计**: 使用 Flexible 和 Wrap 适应不同屏幕尺寸
- **视觉层次**: 通过内边距和字体样式建立清晰的信息层次
- **一致性**: 统一的颜色和间距设计

### 类型安全

- **类型化引用**: 使用 `DocumentReference<Movie>` 确保类型安全
- **数据绑定**: 直接使用 Movie 对象的属性，无需类型转换

## 总结

_MovieItem 组件展示了优秀的 Flutter UI 设计模式：

- 模块化的组件结构，使用 getter 方法组织代码
- 响应式的布局设计，适应不同内容长度
- 清晰的视觉层次，通过字体大小、颜色和间距实现
- 类型安全的数据绑定和引用

这个组件设计既保持了代码的可维护性，又提供了良好的用户体验。
