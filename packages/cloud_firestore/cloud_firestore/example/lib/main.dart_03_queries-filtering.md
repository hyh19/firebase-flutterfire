# main.dart - 查询与过滤

## 概述

本文档解释 Cloud Firestore 示例应用中的查询和过滤功能，包括 MovieQuery 枚举和查询扩展方法。

## MovieQuery 枚举

```dart 52:60:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// The different ways that we can filter/sort movies.
enum MovieQuery {
  year,
  likesAsc,
  likesDesc,
  rated,
  sciFi,
  fantasy,
}
```

MovieQuery 枚举定义了应用支持的不同查询类型：

- `year`: 按年份降序排序
- `likesAsc`: 按点赞数升序排序
- `likesDesc`: 按点赞数降序排序
- `rated`: 按评级降序排序
- `sciFi`: 过滤科幻电影
- `fantasy`: 过滤奇幻电影

## 查询扩展方法

```dart 62:75:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
extension on Query<Movie> {
  /// Create a firebase query from a [MovieQuery]
  Query<Movie> queryBy(MovieQuery query) {
    return switch (query) {
      MovieQuery.fantasy => where('genre', arrayContainsAny: ['fantasy']),
      MovieQuery.sciFi => where('genre', arrayContainsAny: ['sci-fi']),
      MovieQuery.likesAsc ||
      MovieQuery.likesDesc =>
        orderBy('likes', descending: query == MovieQuery.likesDesc),
      MovieQuery.year => orderBy('year', descending: true),
      MovieQuery.rated => orderBy('rated', descending: true)
    };
  }
}
```

### 扩展方法设计

- 在 `Query<Movie>` 上添加 `queryBy` 扩展方法
- 使用 switch 表达式根据 MovieQuery 类型返回相应的 Firestore 查询

### 查询类型详解

#### 1. 分类过滤查询

```dart
MovieQuery.fantasy => where('genre', arrayContainsAny: ['fantasy']),
MovieQuery.sciFi => where('genre', arrayContainsAny: ['sci-fi']),
```

- 使用 `arrayContainsAny` 运算符
- 检查电影的 `genre` 数组是否包含指定的类型
- 支持部分匹配，电影可以有多个类型标签

#### 2. 排序查询

```dart
MovieQuery.likesAsc || MovieQuery.likesDesc =>
  orderBy('likes', descending: query == MovieQuery.likesDesc),
```

- 点赞排序：升序和降序
- 使用 `orderBy` 方法指定排序字段和方向
- `descending` 参数控制排序方向

```dart
MovieQuery.year => orderBy('year', descending: true),
MovieQuery.rated => orderBy('rated', descending: true)
```

- 年份和评级排序：降序排列
- 最新的电影和最高的评级排在前面

## 查询使用示例

在 FilmList 组件中使用查询：

```dart 102:103:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
class _FilmListState extends State<FilmList> {
  MovieQuery query = MovieQuery.year;
```

```dart 271:272:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
body: StreamBuilder<QuerySnapshot<Movie>>(
  stream: moviesRef.queryBy(query).snapshots(),
```

### 查询执行流程

1. 用户选择查询类型（通过 PopupMenuButton）
2. `query` 状态变量更新
3. `moviesRef.queryBy(query)` 生成对应的 Firestore 查询
4. `.snapshots()` 创建实时数据流
5. StreamBuilder 监听数据变化并重新构建 UI

## Firestore 查询特性

### 数组包含查询

- `arrayContainsAny`: 检查数组字段是否包含指定的任何值
- 适用于标签、多选分类等场景
- 比 `arrayContains` 更灵活，支持多个值的 OR 查询

### 排序查询

- `orderBy`: 指定排序字段和方向
- 支持字符串、数字等各种数据类型的排序
- 可以与过滤条件组合使用

### 类型安全查询

- 通过 `withConverter<Movie>` 确保查询返回正确类型
- 编译时类型检查，减少运行时错误
- IDE 提供更好的代码提示

## UI 中的查询选择

应用提供直观的查询选择界面：

```dart 128:159:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
PopupMenuButton<MovieQuery>(
  onSelected: (value) => setState(() => query = value),
  icon: const Icon(Icons.sort),
  itemBuilder: (BuildContext context) {
    return [
      const PopupMenuItem(
        value: MovieQuery.year,
        child: Text('Sort by Year'),
      ),
      const PopupMenuItem(
        value: MovieQuery.rated,
        child: Text('Sort by Rated'),
      ),
      const PopupMenuItem(
        value: MovieQuery.likesAsc,
        child: Text('Sort by Likes ascending'),
      ),
      const PopupMenuItem(
        value: MovieQuery.likesDesc,
        child: Text('Sort by Likes descending'),
      ),
      const PopupMenuItem(
        value: MovieQuery.fantasy,
        child: Text('Filter genre fantasy'),
      ),
      const PopupMenuItem(
        value: MovieQuery.sciFi,
        child: Text('Filter genre sci-fi'),
      ),
    ];
  },
),
```

## 总结

查询与过滤系统提供了灵活的数据检索功能：

- 支持多种排序方式（年份、评级、点赞数）
- 支持分类过滤（科幻、奇幻电影）
- 使用类型安全的查询扩展方法
- 提供直观的 UI 进行查询选择

这个设计展示了如何在 Flutter 应用中有效地使用 Firestore 的查询功能，实现复杂的数据过滤和排序需求。
