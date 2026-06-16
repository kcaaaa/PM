# 技术栈 · react-native

> **来源**：`stacks/react-native.csv`。React Native + Expo（默认推荐）。

---

## 1. 新项目起手

```bash
npx create-expo-app my-app -t
cd my-app
npm run ios    # 或 android / web
```

**绝对优先 Expo**——管理 SDK、EAS 打包、OTA 更新开箱即用。

## 2. Core Components

| RN 组件 | 对应 Web |
|---|---|
| `<View>` | `<div>` |
| `<Text>` | `<p>/<span>` |
| `<Image>` | `<img>`（但需 Expo `<Image>`） |
| `<ScrollView>` | 长列表用 `<FlatList>` 而非这个 |
| `<FlatList>` / `<SectionList>` | 虚拟化列表 |
| `<Pressable>` | 通用可点击（优先于 `TouchableOpacity`） |
| `<TextInput>` | `<input>` |
| `<Modal>` | 模态 |
| `<SafeAreaView>` | 安全区（刘海 / Home Indicator） |

## 3. Navigation

**React Navigation**（事实标准）：

```bash
npx expo install @react-navigation/native @react-navigation/native-stack
```

```tsx
const Stack = createNativeStackNavigator()

<NavigationContainer>
  <Stack.Navigator>
    <Stack.Screen name="Home" component={Home} />
    <Stack.Screen name="Detail" component={Detail} />
  </Stack.Navigator>
</NavigationContainer>
```

**Expo Router**（文件路由，推荐）：

```
app/
├── _layout.tsx
├── index.tsx
└── user/[id].tsx
```

## 4. Styling

```tsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 16,
  },
})
```

- **Flexbox 为默认**（方向默认 `column`）
- 不支持 CSS cascade，每个组件 inline
- `px` 单位省略；密度由 `PixelRatio` 处理
- **NativeWind**（Tailwind for RN）：`npm i nativewind` 可用 `className`

## 5. 平台差异

```tsx
import { Platform } from 'react-native'

Platform.OS  // 'ios' | 'android' | 'web'
Platform.select({ ios: 16, android: 20, default: 14 })
```

平台专属文件：`Component.ios.tsx` / `Component.android.tsx`

## 6. Performance

- **`FlatList` 虚拟化** 大列表（`keyExtractor`、`getItemLayout`、`removeClippedSubviews`）
- **Image 缓存**：`expo-image`（`<Image>` from `expo-image`）
- **避免 Inline Functions** 给 `<FlatList renderItem>` —— 用 `useCallback`
- **`react-native-reanimated` v3** 做复杂动画（UI 线程）
- **`react-native-gesture-handler`** 替代 RN 内置手势

## 7. Animations

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated'

const scale = useSharedValue(1)
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }))

<Animated.View style={style} />
<Pressable onPress={() => scale.value = withSpring(1.2)}>
```

## 8. 触觉 / 反馈

```tsx
import * as Haptics from 'expo-haptics'
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)
```

## 9. Accessibility

```tsx
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Close modal"
  accessibilityHint="Closes the current modal"
  accessible
>
```

- iOS VoiceOver / Android TalkBack 测试必做
- 最小触控目标 44×44 pt（iOS）/ 48×48 dp（Android）

## 10. 状态管理

- 局部：`useState` / `useReducer`
- 全局：Zustand（最推荐）、Redux Toolkit、或 Context
- 数据层：TanStack Query（replaces Redux data）

---

## 关键红线

- [ ] 新项目用 Expo + Expo Router
- [ ] 列表 > 20 项必须 `<FlatList>`
- [ ] 图片用 `expo-image`（带缓存）
- [ ] 动画用 `react-native-reanimated`（UI 线程）
- [ ] 触控目标 ≥ 44×44
- [ ] `<SafeAreaView>` 包裹根视图
- [ ] 区分 `ios` / `android` 通过 `Platform.select`
