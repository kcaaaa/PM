# 技术栈 · jetpack-compose

> **来源**：`stacks/jetpack-compose.csv`。Android Jetpack Compose + Material 3。

---

## 1. Composables 基础

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
  Text(
    text = "Hello, $name!",
    style = MaterialTheme.typography.headlineMedium,
    modifier = modifier.padding(16.dp)
  )
}
```

- **`@Composable`** 函数 = UI 构建块
- **PascalCase** 命名（`UserCard` 不是 `userCard`）
- **参数最后是 `Modifier`**（约定）

## 2. State Hoisting

```kotlin
@Composable
fun Counter() {
  var count by remember { mutableStateOf(0) }
  Counter(count = count, onIncrement = { count++ })
}

@Composable
fun Counter(count: Int, onIncrement: () -> Unit) {
  Button(onClick = onIncrement) { Text("$count") }
}
```

- **State 上提**——stateless composable 更易复用与测试
- **`remember`** 跨 recomposition 保持
- **`rememberSaveable`** 跨进程 / 配置变化保持（`onBackPressed`、旋转）

## 3. Modifier

```kotlin
Modifier
  .fillMaxWidth()
  .padding(16.dp)
  .background(MaterialTheme.colorScheme.primary)
  .clip(RoundedCornerShape(12.dp))
  .clickable { /* ... */ }
  .semantics { contentDescription = "Close" }
```

- **顺序重要**——`.padding().background()` ≠ `.background().padding()`
- `Modifier` 为默认参数，允许调用方传入额外修饰

## 4. Layout

| Composable | 对应 |
|---|---|
| `Column` | 垂直布局 |
| `Row` | 水平布局 |
| `Box` | 堆叠（Z 轴） |
| `LazyColumn` | 虚拟化垂直列表（替代 `RecyclerView`） |
| `LazyRow` | 虚拟化水平 |
| `LazyVerticalGrid` | 网格 |
| `Scaffold` | 含 TopBar / FAB / Drawer 的页面骨架 |
| `Surface` | 主题背景 |

## 5. Material 3 Theme

```kotlin
MaterialTheme(
  colorScheme = lightColorScheme(
    primary = Color(0xFF2563EB),       // 从《03-配色方案》取
    secondary = Color(0xFF3B82F6),
    tertiary = Color(0xFFF97316),
  ),
  typography = Typography(...)
) {
  // content
}
```

- Material 3 的 **Dynamic Color**（Android 12+）：`dynamicLightColorScheme(LocalContext.current)`
- **ColorScheme / Typography / Shapes** 三大主题对象

## 6. Navigation（Compose Navigation）

```kotlin
@Composable
fun AppNavigation() {
  val navController = rememberNavController()
  NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(onItemClick = { id -> navController.navigate("detail/$id") }) }
    composable("detail/{id}") { backStackEntry ->
      DetailScreen(id = backStackEntry.arguments?.getString("id")!!)
    }
  }
}
```

**Navigation Compose（Type-safe，推荐）** with Kotlin Serialization：

```kotlin
@Serializable data object Home
@Serializable data class Detail(val id: String)

composable<Home> { ... }
composable<Detail> { backStackEntry ->
  val detail = backStackEntry.toRoute<Detail>()
}
```

## 7. Animation

```kotlin
val scale by animateFloatAsState(if (expanded) 1.2f else 1f)
Box(Modifier.scale(scale))

AnimatedVisibility(visible = isVisible, enter = fadeIn(), exit = fadeOut()) {
  Content()
}

Crossfade(targetState = current) { state -> /* ... */ }
```

## 8. Side Effects

| API | 用途 |
|---|---|
| `LaunchedEffect(key)` | 协程在 composable 中启动 |
| `DisposableEffect(key)` | 注册 / 清理 |
| `SideEffect` | 每次 recomposition 执行 |
| `derivedStateOf` | 派生 state 防抖 |

```kotlin
LaunchedEffect(userId) {
  user = fetchUser(userId)  // suspend 函数
}
```

## 9. Performance

- **`remember { ... }`** 昂贵值
- **`key()`** 给列表项稳定 key
- **`LazyColumn`** 带 `key = { it.id }`
- **避免 recomposition** —— 看 Layout Inspector
- **`@Stable` / `@Immutable`** 数据类帮助跳过 recomposition
- **`derivedStateOf`** 限制派生

## 10. Accessibility

```kotlin
IconButton(onClick = {}, modifier = Modifier.semantics { contentDescription = "Close menu" }) {
  Icon(Icons.Default.Close, contentDescription = null)  // 避免重复
}
```

- 最小 `Modifier.size(48.dp)` 触控目标
- `LocalAccessibilityManager.current.isEnabled` 检测开关

## 11. Gradle 依赖

```kotlin
dependencies {
  implementation(platform("androidx.compose:compose-bom:2024.05.00"))
  implementation("androidx.compose.material3:material3")
  implementation("androidx.compose.ui:ui")
  implementation("androidx.navigation:navigation-compose:2.7.7")
  implementation("androidx.lifecycle:lifecycle-viewmodel-compose")
}
```

---

## 关键红线

- [ ] State hoisting —— stateless composable
- [ ] `LazyColumn` + `key = {}` 做列表
- [ ] Material 3 (`useMaterial3` via Gradle)
- [ ] Modifier 顺序看效果 —— padding 与 background 的位置不同
- [ ] 触控目标 ≥ 48.dp
- [ ] 图标按钮 `contentDescription` 必填
- [ ] 动画用 `animate*AsState` / `AnimatedVisibility`
