---
name: mobile-advanced
description: 移动开发高级技能——React Native 和 Flutter 的高级技巧与最佳实践
when_to_use: 当需要开发跨平台移动应用、优化移动性能或实现复杂交互时阅读
source: 项目实战经验总结，适配 React Native 和 Flutter 开发
---

# 移动开发高级技能 Mobile Development Advanced

> 当前项目适配：移动开发必须记录到 `Base/04-方案设计/TD-*/方案说明.md`，并遵循平台特定的设计规范。

---

## 1. React Native 高级技巧

### 1.1 性能优化

```typescript
// 优化 FlatList
<FlatList
  data={items}
  keyExtractor={(item) => item.id}
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index
  })}
  removeClippedSubviews={true}
  initialNumToRender={10}
  windowSize={5}
  renderItem={({ item }) => <Item item={item} />}
/>

// 使用 useCallback 避免重新创建函数
const renderItem = useCallback(({ item }) => {
  return <Item item={item} onPress={handlePress} />;
}, [handlePress]);
```

### 1.2 动画优化

```typescript
// 使用 Reanimated v3
import Animated, { 
  useSharedValue, 
  useAnimatedStyle, 
  withSpring 
} from 'react-native-reanimated';

const scale = useSharedValue(1);
const style = useAnimatedStyle(() => ({
  transform: [{ scale: scale.value }]
}));

const handlePress = () => {
  scale.value = withSpring(1.2, {
    stiffness: 300,
    damping: 20
  });
};
```

### 1.3 状态管理

```typescript
// 使用 Zustand（推荐）
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 }))
}));

// 使用 TanStack Query
import { useQuery } from '@tanstack/react-query';

const { data, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
});
```

---

## 2. Flutter 高级技巧

### 2.1 性能优化

```dart
// 使用 const 构造器
const Text('Hello');
const Container(color: Colors.blue);

// 使用 ListView.builder 虚拟化列表
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
);

// 使用 RepaintBoundary 隔离重绘
RepaintBoundary(
  child: AnimatedWidget(),
);
```

### 2.2 状态管理

```dart
// 使用 Riverpod（推荐）
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

// 使用 Bloc 处理复杂业务逻辑
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    on<IncrementEvent>((event, emit) => emit(CounterState(count: state.count + 1)));
  }
}
```

### 2.3 动画效果

```dart
// 使用 flutter_animate
import 'package:flutter_animate/flutter_animate.dart';

Text('Hello').animate()
  .fade(duration: 500.ms)
  .slide(duration: 500.ms)
  .scale(duration: 500.ms);

// 使用 Hero 共享元素动画
Hero(
  tag: 'image-tag',
  child: Image.asset('assets/image.png'),
);
```

---

## 3. 跨平台设计规范

### 3.1 iOS 设计规范

- [ ] 使用系统字体（SF Pro）
- [ ] 最小触控目标 44×44 pt
- [ ] 遵循 Human Interface Guidelines
- [ ] 使用 SafeAreaView 适配刘海屏

### 3.2 Android 设计规范

- [ ] 使用 Material 3 设计语言
- [ ] 最小触控目标 48×48 dp
- [ ] 遵循 Material Design Guidelines
- [ ] 适配各种屏幕尺寸和密度

---

## 4. 移动端测试

### 4.1 单元测试

```typescript
// React Native
import { render, screen } from '@testing-library/react-native';

test('renders welcome message', () => {
  render(<WelcomeScreen />);
  expect(screen.getByText('Welcome')).toBeTruthy();
});
```

```dart
// Flutter
void main() {
  test('counter increments correctly', () {
    expect(Counter().increment(), equals(1));
  });
}
```

### 4.2 端到端测试

```powershell
# React Native - Detox
npx detox test --configuration ios.sim.release

# Flutter - Integration Tests
flutter test integration_test/app_test.dart
```

---

## 关键红线

- [ ] 移动开发必须记录到 TD-*/方案说明.md
- [ ] 必须遵循平台特定设计规范
- [ ] 长列表必须使用虚拟化
- [ ] 必须处理平台差异
- [ ] 必须实现测试覆盖