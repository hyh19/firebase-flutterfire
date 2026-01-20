# main.dart - 交互功能

## 概述

本文档解释 Cloud Firestore 示例应用中的交互功能，包括点赞组件、事务处理和批量操作。

## Likes - 点赞组件

```dart 417:435:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
/// Displays and manages the movie 'like' count.
class Likes extends StatefulWidget {
  /// Constructs a new [Likes] instance with a given [DocumentReference] and
  /// current like count.
  Likes({
    Key? key,
    required this.reference,
    required this.currentLikes,
  }) : super(key: key);

  /// The reference relating to the counter.
  final DocumentReference<Movie> reference;

  /// The number of current likes (before manipulation).
  final int currentLikes;

  @override
  _LikesState createState() => _LikesState();
}
```

Likes 组件管理电影的点赞功能：

### 构造函数参数

- `reference`: 指向电影文档的类型安全引用
- `currentLikes`: 当前点赞数（用于初始化显示）
- `key`: Widget 键，用于状态保持

### 组件特性

- **有状态组件**: 需要管理点赞状态和用户交互
- **类型安全**: 使用 `DocumentReference<Movie>` 确保数据类型正确

## _LikesState - 点赞状态管理

```dart 437:441:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
class _LikesState extends State<Likes> {
  /// A local cache of the current likes, used to immediately render the updated
  /// likes count after an update, even while the request isn't completed yet.
  late int _likes = widget.currentLikes;
```

### 状态变量

- `_likes`: 本地缓存的点赞数
- 使用 `late` 关键字延迟初始化，从 `widget.currentLikes` 获取初始值

## 点赞处理逻辑

```dart 442:478:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
Future<void> _onLike() async {
  final currentLikes = _likes;

  // Increment the 'like' count straight away to show feedback to the user.
  setState(() {
    _likes = currentLikes + 1;
  });

  try {
    // Update the likes using a transaction.
    // We use a transaction because multiple users could update the likes count
    // simultaneously. As such, our likes count may be different from the likes
    // count on the server.
    int newLikes = await FirebaseFirestore.instance
        .runTransaction<int>((transaction) async {
      DocumentSnapshot<Movie> movie =
          await transaction.get<Movie>(widget.reference);

      if (!movie.exists) {
        throw Exception('Document does not exist!');
      }

      int updatedLikes = movie.data()!.likes + 1;
      transaction.update(widget.reference, {'likes': updatedLikes});
      return updatedLikes;
    });

    // Update with the real count once the transaction has completed.
    setState(() => _likes = newLikes);
  } catch (e, s) {
    print(s);
    print('Failed to update likes for document! $e');

    // If the transaction fails, revert back to the old count
    setState(() => _likes = currentLikes);
  }
}
```

### 乐观更新策略

1. **立即反馈**: 先更新本地状态，给用户即时视觉反馈
2. **事务处理**: 使用 Firestore 事务确保数据一致性
3. **错误回滚**: 失败时恢复到原始状态

### 事务执行流程

1. **读取当前数据**: `transaction.get<Movie>(widget.reference)`
2. **存在性检查**: 确保文档仍然存在
3. **计算新值**: 在服务器端计算新的点赞数
4. **原子更新**: 使用事务更新文档
5. **返回结果**: 返回实际的更新后点赞数

### 错误处理

- **异常捕获**: 捕获事务执行中的任何错误
- **日志记录**: 打印详细的错误信息
- **状态回滚**: 恢复到事务前的状态

## 组件更新处理

```dart 480:489:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
@override
void didUpdateWidget(Likes oldWidget) {
  super.didUpdateWidget(oldWidget);
  // The likes on the server changed, so we need to update our local cache to
  // keep things in sync. Otherwise if another user updates the likes,
  // we won't see the update.
  if (widget.currentLikes != oldWidget.currentLikes) {
    _likes = widget.currentLikes;
  }
}
```

### 外部更新同步

- **监听属性变化**: 当父组件传递新的 `currentLikes` 时触发
- **状态同步**: 更新本地缓存以反映服务器端的最新状态
- **多用户支持**: 确保显示其他用户点赞操作的结果

## UI 渲染

```dart 491:504:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
@override
Widget build(BuildContext context) {
  return Row(
    children: [
      IconButton(
        iconSize: 20,
        onPressed: _onLike,
        icon: const Icon(Icons.favorite),
      ),
      Text('$_likes likes'),
    ],
  );
}
```

### 界面元素

- **点赞按钮**: 心形图标按钮，点击触发点赞
- **点赞计数**: 显示当前点赞数的文本
- **水平布局**: 按钮和文本水平排列

## 批量操作 - 重置点赞数

```dart 300:313:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
Future<void> _resetLikes() async {
  final movies = await moviesRef.get(
    const GetOptions(
      serverTimestampBehavior: ServerTimestampBehavior.previous,
    ),
  );

  WriteBatch batch = FirebaseFirestore.instance.batch();

  for (final movie in movies.docs) {
    batch.update(movie.reference, {'likes': 0});
  }
  await batch.commit();
}
```

### 批量更新流程

1. **获取所有电影**: 从集合中获取所有文档
2. **创建批处理**: `FirebaseFirestore.instance.batch()`
3. **添加操作**: 为每个文档添加更新操作
4. **提交事务**: `batch.commit()` 原子性地执行所有更新

### GetOptions 配置

- `serverTimestampBehavior: ServerTimestampBehavior.previous`: 确保获取服务器上的最新数据，避免本地缓存影响

## 事务 vs 批量操作

### Firestore 事务 (Transactions)

- **原子性**: 要么全部成功，要么全部失败
- **隔离性**: 并发事务不会相互干扰
- **一致性**: 确保数据处于有效状态
- **适用场景**: 需要基于当前值进行计算的更新

### 批量写入 (WriteBatch)

- **原子性**: 批处理中的所有操作要么全部成功，要么全部失败
- **性能优化**: 减少网络往返次数
- **简单操作**: 适用于不需要读取当前值的更新
- **适用场景**: 多个独立的操作，可以预先知道要写入的值

## 实时数据同步

整个应用使用 StreamBuilder 实现实时数据同步：

```dart 271:296:packages/cloud_firestore/cloud_firestore/example/lib/main.dart
body: StreamBuilder<QuerySnapshot<Movie>>(
  stream: moviesRef.queryBy(query).snapshots(),
  builder: (context, snapshot) {
    // 实时更新UI当数据改变时
  },
),
```

### 同步机制

- **快照监听**: `snapshots()` 创建数据流
- **实时更新**: 任何数据变化都会触发 UI 重建
- **类型安全**: 使用 `QuerySnapshot<Movie>` 确保类型正确

## 总结

交互功能展示了复杂的 Firestore 操作模式：

- 乐观更新结合事务处理确保良好的用户体验
- 批量操作支持高效的数据管理
- 实时数据同步保持界面与服务器状态一致
- 错误处理和状态回滚确保应用稳定性

这个设计演示了如何在 Flutter 应用中实现复杂的数据库交互，同时保持良好的用户体验和数据一致性。
