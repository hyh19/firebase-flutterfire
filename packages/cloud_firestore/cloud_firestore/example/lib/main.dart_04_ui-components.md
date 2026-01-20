# main.dart - 主 UI 组件

## 概述

本文档解释 Cloud Firestore 示例应用的主 UI 组件，包括应用入口组件和电影列表组件。

## FirestoreExampleApp - 应用入口

```dart 77:91:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// The entry point of the application.
///
/// Returns a [MaterialApp].
class FirestoreExampleApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firestore Example App',
      theme: ThemeData.dark(),
      home: const Scaffold(
        body: Center(child: FilmList()),
      ),
    );
  }
}
```

FirestoreExampleApp 是应用的根组件：

### 配置特性

- **MaterialApp**: 使用 Material Design 设计规范
- **深色主题**: `ThemeData.dark()` 提供深色主题界面
- **标题**: "Firestore Example App"
- **主页面**: FilmList 组件作为应用的主界面

### 设计选择

- 无状态组件 (`StatelessWidget`)：因为不需要内部状态管理
- 居中布局：使用 `Center` 小部件确保内容居中显示
- 简单架构：专注于展示 Firestore 功能，而不是复杂的导航

## FilmList - 电影列表组件

```dart 93:99:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Holds all example app films
class FilmList extends StatefulWidget {
  const FilmList({Key? key}) : super(key: key);

  @override
  _FilmListState createState() => _FilmListState();
}
```

FilmList 是有状态组件，管理电影列表的状态和交互：

- **状态管理**: 使用 `_FilmListState` 管理查询状态和 UI 交互
- **Key 参数**: 支持组件重建时的状态保持

## _FilmListState - 电影列表状态

```dart 101:103:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
class _FilmListState extends State<FilmList> {
  MovieQuery query = MovieQuery.year;
```

### 状态变量

- `query`: 当前选中的查询类型，默认为 `MovieQuery.year`（按年份排序）

## AppBar 设计

```dart 106:126:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
appBar: AppBar(
  title: Column(
    mainAxisSize: MainAxisSize.min,
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: [
      const Text('Firestore Example: Movies'),

      // This is a example use for 'snapshots in sync'.
      // The view reflects the time of the last Firestore sync; which happens any time a field is updated.
      StreamBuilder(
        stream: FirebaseFirestore.instance.snapshotsInSync(),
        builder: (context, _) {
          return Text(
            'Latest Snapshot: ${DateTime.now()}',
            style: Theme.of(context).textTheme.bodySmall,
          );
        },
      ),
    ],
  ),
```

### AppBar 特性

- **双行标题**: 显示应用名称和同步状态
- **实时同步指示器**: 使用 `snapshotsInSync()` 显示最后同步时间
- **紧凑布局**: `mainAxisSize: MainAxisSize.min` 节省垂直空间

### Snapshots In Sync 功能

- `FirebaseFirestore.instance.snapshotsInSync()`: 监听 Firestore 同步事件
- 每次数据更新时触发，包括本地和远程变更
- 显示当前时间戳作为同步状态指示器

## 查询选择菜单

```dart 127:159:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
actions: <Widget>[
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

### 排序菜单配置

- **图标**: 使用排序图标 (Icons.sort)
- **项目**: 6 个不同的查询选项
- **状态更新**: 选择时更新 `query` 状态，触发 UI 重建

## 操作菜单 - 聚合和批量操作

```dart 160:268:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
PopupMenuButton<String>(
  onSelected: (value) async {
    switch (value) {
      case 'reset_likes':
        return _resetLikes();
      case 'aggregate':
        // Count the number of movies
        final _count = await FirebaseFirestore.instance
            .collection('firestore-example-app')
            .count()
            .get();

        print('Count: ${_count.count}');

        // Average the number of likes
        final _average = await FirebaseFirestore.instance
            .collection('firestore-example-app')
            .aggregate(average('likes'))
            .get();

        print('Average: ${_average.getAverage('likes')}');

        // Sum the number of likes
        final _sum = await FirebaseFirestore.instance
            .collection('firestore-example-app')
            .aggregate(sum('likes'))
            .get();

        print('Sum: ${_sum.getSum('likes')}');

        // In one query
        final _all = await FirebaseFirestore.instance
            .collection('firestore-example-app')
            .aggregate(
              average('likes'),
              sum('likes'),
              count(),
            )
            .get();

        print('Average: ${_all.getAverage('likes')} '
            'Sum: ${_all.getSum('likes')} '
            'Count: ${_all.count}');

        return;
      case 'load_bundle':
        Uint8List buffer = await loadBundleSetup(2);
        LoadBundleTask task =
            FirebaseFirestore.instance.loadBundle(buffer);

        final list = await task.stream.toList();

        print(
          list.map((e) => e.totalDocuments),
        );
        print(
          list.map((e) => e.bytesLoaded),
        );
        print(
          list.map((e) => e.documentsLoaded),
        );
        print(
          list.map((e) => e.totalBytes),
        );
        print(
          list,
        );

        LoadBundleTaskSnapshot lastSnapshot = list.removeLast();
        print(lastSnapshot.taskState);

        print(
          list.map((e) => e.taskState),
        );
        return;
      case 'vectorValue':
        const vectorValue = VectorValue([1.0, 2.0, 3.0]);
        final vectorValueDoc = await FirebaseFirestore.instance
            .collection('firestore-example-app')
            .add({'vectorValue': vectorValue});

        final snapshot = await vectorValueDoc.get();
        print(snapshot.data());
        return;
      default:
        return;
    }
  },
```

### 高级功能菜单

提供四个高级 Firestore 功能演示：

1. **重置点赞数**: 批量更新操作
2. **聚合查询**: 计数、平均值、求和等统计功能
3. **Bundle 加载**: 离线数据包加载功能
4. **向量值测试**: VectorValue 数据类型演示

### 聚合查询详解

- `count()`: 统计文档数量
- `average('field')`: 计算字段平均值
- `sum('field')`: 计算字段总和
- 支持组合查询，同时获取多个聚合结果

### Bundle 加载功能

- 加载预打包的 Firestore 数据
- 监控加载进度（文档数、字节数、状态）
- 支持离线数据预加载

## 电影列表显示

```dart 271:296:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
body: StreamBuilder<QuerySnapshot<Movie>>(
  stream: moviesRef.queryBy(query).snapshots(),
  builder: (context, snapshot) {
    if (snapshot.hasError) {
      return Center(
        child: Text(snapshot.error.toString()),
      );
    }

    if (!snapshot.hasData) {
      return const Center(child: CircularProgressIndicator());
    }

    final data = snapshot.requireData;

    return ListView.builder(
      itemCount: data.size,
      itemBuilder: (context, index) {
        return _MovieItem(
          data.docs[index].data(),
          data.docs[index].reference,
        );
      },
    );
  },
),
```

### StreamBuilder 配置

- **数据流**: `moviesRef.queryBy(query).snapshots()` 提供实时数据
- **错误处理**: 显示错误信息
- **加载状态**: 显示进度指示器
- **数据渲染**: 使用 ListView 显示电影列表

### 列表项构建

- 每个电影项包含 Movie 数据和 DocumentReference
- 使用 `_MovieItem` 组件进行渲染
- 支持类型安全的数据访问

## 总结

主 UI 组件架构清晰，功能丰富：

- 提供直观的查询和排序界面
- 展示高级 Firestore 功能（聚合、批量操作、Bundle 加载）
- 实现实时数据同步和错误处理
- 使用类型安全的数据绑定

这个设计展示了如何构建功能完整的 Firestore 驱动的 Flutter 应用界面。
