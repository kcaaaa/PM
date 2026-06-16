# 技术栈 · flutter

> **来源**：`stacks/flutter.csv`。Flutter 3+ / Dart 3+。

---

## 1. Widgets 基础

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: const Center(child: Text('Hello, Flutter!')),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

- **Everything is a widget**
- `StatelessWidget` 无状态；`StatefulWidget` 有状态
- `const` 构造器优先——可被 Flutter 缓存

## 2. State 管理

| 方案 | 使用场景 |
|---|---|
| `setState` | 单个 Widget 局部 |
| **Riverpod**（推荐） | 全局 / 跨 Widget；编译期安全 |
| **Provider** | 简单注入（Riverpod 前身） |
| **Bloc / Cubit** | 复杂业务流 |
| **GetX** | 快速但争议大，不推荐新项目 |

```dart
// Riverpod
final counterProvider = StateProvider<int>((ref) => 0);

class Counter extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return TextButton(
      onPressed: () => ref.read(counterProvider.notifier).state++,
      child: Text('$count'),
    );
  }
}
```

## 3. Layout

| Widget | 用途 |
|---|---|
| `Row` / `Column` | 线性布局 |
| `Stack` | 堆叠 |
| `Expanded` / `Flexible` | 伸缩 |
| `Container` | 容器（padding / color / decoration） |
| `Padding` | 仅 padding |
| `SizedBox` | 固定尺寸 / spacing |
| `ListView` / `ListView.builder` | 列表（`.builder` 虚拟化） |
| `GridView.builder` | 网格 |
| `CustomScrollView` + `slivers` | 高级滚动 |
| `SafeArea` | 安全区 |

## 4. Material / Cupertino

```dart
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,  // Material 3
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
    textTheme: GoogleFonts.interTextTheme(),
  ),
  darkTheme: ThemeData.dark(),
  themeMode: ThemeMode.system,
  home: const HomeScreen(),
)
```

- Material 3 默认（`useMaterial3: true`）
- 从《03-配色方案》的 Primary hex 作为 `seedColor`

## 5. Navigation

**go_router**（官方推荐）：

```dart
final router = GoRouter(routes: [
  GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
  GoRoute(path: '/user/:id', builder: (_, state) => UserScreen(id: state.pathParameters['id']!)),
]);

MaterialApp.router(routerConfig: router)
```

## 6. 网络与数据

```dart
// Dio / http
import 'package:dio/dio.dart';
final dio = Dio();
final response = await dio.get('/api/users');

// JSON → Dart 类：用 json_serializable + build_runner
```

## 7. Animation

- `AnimatedContainer` / `AnimatedOpacity` / `AnimatedPositioned`（隐式）
- `AnimationController` + `Tween`（显式）
- `flutter_animate` 包：链式语法 `.fade().slide()`
- `Hero` 路由共享元素
- 尊重 `MediaQuery.of(context).disableAnimations`

## 8. Performance

- **`const` 构造器**优先 —— 避免 rebuild
- **`ListView.builder`**（虚拟化）> `ListView(children:)`
- **`RepaintBoundary`** 隔离频繁 repaint 的子树
- **避免 build 内做昂贵计算** —— 提到 Provider / State
- **`Image.network` 缓存**：用 `cached_network_image`

## 9. Responsive

```dart
LayoutBuilder(builder: (ctx, constraints) {
  if (constraints.maxWidth > 600) return WideLayout();
  return NarrowLayout();
})

MediaQuery.sizeOf(context).width  // 非完整 rebuild 订阅
```

## 10. Accessibility

```dart
Semantics(
  label: 'Close menu',
  button: true,
  child: IconButton(icon: const Icon(Icons.close), onPressed: () {}),
)
```

## 11. 包管理

`pubspec.yaml`：

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  go_router: ^14.0.0
  dio: ^5.0.0
  google_fonts: ^6.0.0
  cached_network_image: ^3.3.0
  flutter_animate: ^4.5.0
```

---

## 关键红线

- [ ] `const` 构造器能加就加
- [ ] 长列表必须 `ListView.builder`（不要 `ListView(children: items.map(...))`）
- [ ] 新项目 `useMaterial3: true`
- [ ] 导航用 `go_router`
- [ ] 全局状态用 Riverpod
- [ ] Text 用 `GoogleFonts.interTextTheme()` 等规范字体
- [ ] 动画尊重 `disableAnimations`
