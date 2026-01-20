# main.dart - 数据模型

## 概述

本文档解释 Cloud Firestore 示例应用中的数据模型部分，包括 Movie 类的定义和 Firestore 数据转换器的配置。

## Movie 数据类

```dart 506:548:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
@immutable
class Movie {
  Movie({
    required this.genre,
    required this.likes,
    required this.poster,
    required this.rated,
    required this.runtime,
    required this.title,
    required this.year,
  });

  Movie.fromJson(Map<String, Object?> json)
      : this(
          genre: (json['genre']! as List).cast<String>(),
          likes: json['likes']! as int,
          poster: json['poster']! as String,
          rated: json['rated']! as String,
          runtime: json['runtime']! as String,
          title: json['title']! as String,
          year: json['year']! as int,
        );

  final String poster;
  final int likes;
  final String title;
  final int year;
  final String runtime;
  final String rated;
  final List<String> genre;

  Map<String, Object?> toJson() {
    return {
      'genre': genre,
      'likes': likes,
      'poster': poster,
      'rated': rated,
      'runtime': runtime,
      'title': title,
      'year': year,
    };
  }
}
```

Movie 类表示电影数据模型：

### 属性字段

- `poster`: 电影海报图片的 URL 字符串
- `likes`: 电影的点赞数量（整数）
- `title`: 电影标题
- `year`: 上映年份
- `runtime`: 电影时长（如 "120 min"）
- `rated`: 电影评级（如 "PG-13"）
- `genre`: 电影类型标签列表（如 ["Action", "Sci-Fi"]）

### 构造函数

- **默认构造函数**: 接受所有必需参数的构造函数
- **fromJson 工厂构造函数**: 从 Firestore 文档数据创建 Movie 实例
  - 将 `genre` 字段从动态 List 转换为 `List<String>`
  - 其他字段直接进行类型转换

### 数据转换方法

- `toJson()`: 将 Movie 实例转换为可存储在 Firestore 中的 Map 格式

## Firestore 数据转换器

```dart 42:50:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// A reference to the list of movies.
/// We are using `withConverter` to ensure that interactions with the collection
/// are type-safe.
final moviesRef = FirebaseFirestore.instance
    .collection('firestore-example-app')
    .withConverter<Movie>(
      fromFirestore: (snapshots, _) => Movie.fromJson(snapshots.data()!),
      toFirestore: (movie, _) => movie.toJson(),
    );
```

### 集合引用配置

- `moviesRef`: 类型安全的电影集合引用
- 集合名称: `'firestore-example-app'`
- 使用 `withConverter<Movie>()` 启用类型安全的数据转换

### 转换器函数

- **fromFirestore**: 将 Firestore 文档快照转换为 Movie 实例
  - 接收 `DocumentSnapshot` 和选项参数
  - 调用 `Movie.fromJson()` 进行转换
- **toFirestore**: 将 Movie 实例转换为可存储的数据格式
  - 接收 Movie 实例和选项参数
  - 调用 `movie.toJson()` 进行转换

## 类型安全的好处

使用 `withConverter` 提供以下优势：

1. **自动类型转换**: 读写操作自动处理数据转换
2. **编译时类型检查**: 确保数据结构的一致性
3. **减少运行时错误**: 类型不匹配的问题在编译时就被发现
4. **更好的开发体验**: IDE 可以提供更好的代码提示和错误检测

## 数据结构示例

Firestore 中的文档结构可能如下：

```json
{
  "title": "Inception",
  "year": 2010,
  "poster": "https://example.com/inception.jpg",
  "rated": "PG-13",
  "runtime": "148 min",
  "genre": ["Action", "Sci-Fi", "Thriller"],
  "likes": 1250
}
```

这个数据结构会被自动转换为 Movie 实例，反之亦然。

## 总结

数据模型部分定义了应用的核心数据结构：

- Movie 类封装了电影的所有属性
- 提供了 JSON 序列化/反序列化功能
- 通过 Firestore 转换器实现了类型安全的数据操作

这样的设计确保了数据一致性和类型安全，是构建健壮的 Firestore 应用的基础。
