# 技术栈 · swiftui

> **来源**：`stacks/swiftui.csv`。iOS 17+ / macOS 14+ / VisionOS 1+。

---

## 1. Views 基础

```swift
struct ContentView: View {
  @State private var count = 0

  var body: some View {
    VStack(spacing: 16) {
      Text("Count: \(count)")
        .font(.largeTitle)
        .foregroundStyle(.primary)
      Button("Increment") { count += 1 }
        .buttonStyle(.borderedProminent)
    }
    .padding()
  }
}
```

## 2. State 管理

| Property Wrapper | 用途 |
|---|---|
| `@State` | View 自己的局部状态 |
| `@Binding` | 从父 View 接收可双向的状态 |
| `@Observable`（Swift 5.9+） | 自定义 class 的响应式（替代 `ObservableObject`） |
| `@Environment(\.dismiss)` | 关闭当前 sheet / navigation |
| `@AppStorage("key")` | 绑定 `UserDefaults` |
| `@SceneStorage` | 场景级持久化 |
| `@FocusState` | 焦点管理 |

**新写法（推荐）**：

```swift
@Observable
class UserStore {
  var name = ""
}

struct HomeView: View {
  @State private var store = UserStore()
  // ...
}
```

## 3. Navigation

```swift
NavigationStack {
  List(items) { item in
    NavigationLink(item.name, value: item)
  }
  .navigationDestination(for: Item.self) { item in
    DetailView(item: item)
  }
  .navigationTitle("Items")
}
```

- iOS 16+ 用 `NavigationStack` 替代 `NavigationView`
- 支持 deep linking：绑定 `path: Binding<NavigationPath>`

## 4. Layout

```swift
VStack { } HStack { } ZStack { }   // 基础布局

// 自适应网格
LazyVGrid(columns: [GridItem(.adaptive(minimum: 120))]) {
  ForEach(items) { Item(...) }
}

// 视图层次
ViewThatFits { WideLayout() ; NarrowLayout() }
```

## 5. Animation

```swift
@State private var isExpanded = false

Button("Toggle") {
  withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
    isExpanded.toggle()
  }
}

// 显式动画
.animation(.easeInOut(duration: 0.3), value: isExpanded)
```

- `.spring()` 为默认首选
- 尊重 `accessibilityReduceMotion`：`@Environment(\.accessibilityReduceMotion) private var reduce`

## 6. Accessibility

```swift
Image(systemName: "star.fill")
  .accessibilityLabel("Favorite")
  .accessibilityHint("Tap to remove from favorites")

Button("Delete", role: .destructive) { }
```

## 7. Visual Polish

- **系统色**：`.primary / .secondary / .tint`，勿硬编码
- **材质**：`.background(.ultraThinMaterial)` / `.regularMaterial`
- **圆角**：`.clipShape(RoundedRectangle(cornerRadius: 12))`
- **阴影**：`.shadow(radius: 4)`
- **Gradient**：`LinearGradient(colors: [.blue, .purple], startPoint: .top, endPoint: .bottom)`

## 8. Data

- **`Observation`**（`@Observable`）替代 `ObservableObject`
- **Swift Data**（iOS 17+）替代 Core Data
- **`@Query`** 直接在 View 里拉取

## 9. Concurrency

```swift
Task {
  do {
    let data = try await fetchData()
  } catch {
    print(error)
  }
}

.task { await loadInitialData() }  // 视图生命周期绑定
```

## 10. VisionOS

- 参考《02-风格库 · Spatial UI (VisionOS)》+《07-UX 规范 · Spatial UI》
- `.hoverEffect()` 响应注视
- `.glassBackgroundEffect()` 玻璃材质
- `RealityView` 嵌入 3D 内容

---

## 关键红线

- [ ] Swift 5.9+ 用 `@Observable` 替代 `ObservableObject`
- [ ] 16+ 用 `NavigationStack`
- [ ] 动画尊重 `accessibilityReduceMotion`
- [ ] 用系统语义色 `.primary/.secondary` 不硬编码
- [ ] 图标用 SF Symbols（`Image(systemName: "...")`）
