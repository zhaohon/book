## 1. 总体心智模型：从 Vue 到 React Native + Expo

先用一句话帮你对齐思路（类比 Vue）：

- **Vue 的世界**
  - `main.ts` 里 `createApp(App).use(router).use(pinia).mount('#app')`
  - 路由在 `router` 里
  - 全局状态在 `pinia` / `vuex`
  - 模板用 `<template>` + `<script setup>`
- **这个 RN + Expo 项目**
  - `app/_layout.tsx` 里的 `RootLayout` 就像 **整套应用的外壳**  
    （相当于：`main.ts + App.vue + router + pinia + i18n + 全局组件注册` 一起）
  - 路由 = `expo-router` 的 **文件路由**：所有页面都在 `app/` 目录下
  - 全局状态 = `zustand`（`stores/` 目录） + `@tanstack/react-query`
  - 写 UI 用 **JSX**，不是 `<template>`

只要你把上面这对类比记住，后面的代码就有“归宿”了：

- “这是路由吗？” → 去看 `app/`
- “这是全局 provider 吗？” → 去看 `app/_layout.tsx`
- “这是全局状态吗？” → 去看 `stores/`

---

## 2. React / React Native vs Vue 速查表

### 2.1 组件写法对比（最核心）

**Vue（组合式 `<script setup>`）：**

```vue
<template>
  <div class="page">
    <h1>{{ title }}</h1>
    <button @click="inc">count: {{ count }}</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

const title = "Hello Vue";
const count = ref(0);

const inc = () => {
  count.value++;
};
</script>
```

**React Native（函数组件）：**

```tsx
import { useState } from "react";
import { View, Text, Pressable } from "react-native";

export default function HelloReact() {
  const [count, setCount] = useState(0);

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>Hello React</Text>
      <Pressable onPress={() => setCount(count + 1)}>
        <Text>count: {count}</Text>
      </Pressable>
    </View>
  );
}
```

**关键差异点：**

- 没有 `<template>`，**JSX = 模板 + 逻辑混在一起写**（注意大括号 `{}`）
- 状态用 `useState`，不需要 `ref.value`
- 事件写成 `onPress={() => ...}`，不是 `@click="xxx"`
- UI 组件来自 RN：`View` / `Text` / `Image` / `ScrollView` 等

### 2.2 基础标签对照表

| Vue (Web DOM) | React Native      | 说明                       |
| ------------- | ----------------- | -------------------------- |
| `div`         | `View`            | 容器视图                   |
| `span`/`p`    | `Text`            | 文本                       |
| `img`         | `Image`           | 图片                       |
| `button`      | `Pressable`/`Button` | 按钮/可点击区域        |
| `input`       | `TextInput`       | 输入框                     |
| `ul`/`ol`     | `FlatList`/`ScrollView` | 列表/可滚动区域     |

**重要：** RN 里没有 DOM（没有 `document`、`window`、`<div>`），所有 UI 必须用 RN 提供的组件或项目里的封装组件。

### 2.3 样式写法对比

- **Vue Web：** CSS/SCSS + `class="..."` 或 `:style="{ color: 'red' }"`
- **本项目 RN：** 两种方式混用
  1. `className="..."`（NativeWind + Tailwind）
  2. `style={{ ... }}`（标准 RN Style）

例子（你在项目里常见的写法）：

```tsx
<View className="flex-1 items-center justify-center bg-f1">
  <Text className="text-base font-bold text-primary">标题</Text>
</View>
```

类比 Vue：

- `className="flex-1 items-center"` 类似 `class="flex-1 items-center"`
- 具体含义由 `tailwind.config.js` 决定（和 Web Tailwind 很像）

---

## 3. Expo / expo-router 速查

### 3.1 Expo 是什么？

类比：

- **没有 Expo 的纯 RN 项目**：自己配原生工程（Xcode/AndroidStudio）、bundle、打包、Dev Server，非常麻烦
- **Expo**：帮你把打包、开发服务器、原生配置、OTA 更新等都封装好了，**你更像在写一个“跨端前端项目”**

在这个项目里：

- `expo` 负责：
  - 启动开发服务器：`npm run start` / `npm run android` / `npm run ios` / `npm run web`
  - 打包、导出 H5：`npx expo export --platform web`
  - 一些内置 API：`expo-font` / `expo-splash-screen` / `expo-status-bar` 等

### 3.2 expo-router 是什么？

**类比：**

- Vue Router：在 `router/index.ts` 里手写路由表
- Next.js：根据 `pages/` 目录自动生成路由
- **expo-router**：根据 `app/` 目录自动生成路由

规则（简化）：

- `app/index.tsx` → `/`
- `app/profile/index.tsx` → `/profile`
- `app/profile/info.tsx` → `/profile/info`
- `app/(tabs)/index.tsx` → `/`（分组名 `(tabs)` 不计入 URL）
- `app/(auth)/index.tsx` → `/` 或指定路由，视项目封装而定

**你可以把 app/ 当成这个项目的 `pages/` 目录。**

---

## 4. 项目中最常用的 React / RN / Expo API 速查

### 4.1 React Hook 速查（在项目里随处可见）

- `useState(initial)`：组件内部可变状态
- `useEffect(fn, deps)`：副作用（类似 Vue 的 `watchEffect` + 生命周期）
- `useMemo` / `useCallback`：性能优化

示例：

```tsx
const [count, setCount] = useState(0);

useEffect(() => {
  // 类似 onMounted + watch(count)
  console.log("count changed", count);
}, [count]);
```

### 4.2 React Native 常用组件速查

- `View`：最常用容器
- `Text`：文本
- `Image`：图片
- `ScrollView` / `FlatList`：可滚动内容/列表
- `StatusBar`：状态栏控制（Expo 提供了 `expo-status-bar` 版本）
- `SafeAreaView`：安全区域组件（本项目封装在 `components/app/SafeAreaView` 下）

### 4.3 Expo 常用模块速查（在 `_layout.tsx` 里）

- `expo-font` → `useFonts`：加载自定义字体
- `expo-splash-screen`：控制启动屏显示/隐藏
- `expo-status-bar`：控制状态栏样式
- `expo-router/head` → `<Head>`：Web/H5 下修改 `<title>` 等头部信息

在 `app/_layout.tsx` 中你可以看到：

```tsx
import { useFonts } from "expo-font";
import * as SplashScreen from "expo-splash-screen";
import { StatusBar } from "expo-status-bar";
import Head from "expo-router/head";
```

---

## 5. 项目结构 + 常见问题“去哪找”

### 5.1 路由 / 页面

- 所有路由页面：`app/` 目录
- 底部 Tab 布局：`app/(tabs)/_layout.tsx`
- 认证（登录/注册）：`app/(auth)/`
- 个人中心：`app/profile/`
- 交易/行情：`app/(home)/`, `app/(tabs)/position.tsx`, `app/(tabs)/finance.tsx`

**问题：“这个 URL 对应哪个文件？”**

1. 先看路径，比如 `/profile/info`
2. 在 `app/profile/info.tsx` 找

### 5.2 UI 组件

- 通用 UI（按钮、输入框、卡片、Loading）：`components/ui/`
- 应用级组件（头部、布局、Toast、Tab 按钮）：`components/app/`
- 各业务块组件：
  - 首页/行情：`components/home/`
  - 持仓：`components/position/`
  - 个人中心：`components/profile/`
  - 借贷：`components/loan/`

**问题：“这个页面上的某个小组件是在哪写的？”**

- 在页面文件里看它 `import` 的路径，沿着路径打开看源码即可

### 5.3 业务逻辑 / 数据请求

- 所有 Hook：`hooks/`
  - 用户：`hooks/user/*`
  - 登录：`hooks/auth/*`
  - 订单：`hooks/order/*`
  - 行情 WebSocket：`hooks/marketStompClient/*`
  - 订单 WebSocket：`hooks/orderStompClient/*`
  - 通用逻辑：`hooks/common/*`
- 接口请求封装：`service/*`
- 全局状态：`stores/*`（`useUserStore` / `useAppSettingStore` / `useAuthStore` 等）

**问题：“这个数据是从哪里请求的？”**

1. 在页面里搜 `useXXX`（比如 `useGetUserInfo`）
2. 去 `hooks/xxx/useGetUserInfo.ts` 里找 `queryFn`，看它调用了哪个 `service`

---

## 6. 常用“怎么做”速查（语法级）

### 6.1 条件渲染

**Vue：**

```vue
<div v-if="ok">OK</div>
<div v-else>NO</div>
```

**React：**

```tsx
{ok ? <Text>OK</Text> : <Text>NO</Text>}
```

### 6.2 列表渲染

**Vue：**

```vue
<li v-for="item in list" :key="item.id">{{ item.name }}</li>
```

**React：**

```tsx
{list.map((item) => (
  <Text key={item.id}>{item.name}</Text>
))}
```

### 6.3 事件绑定

**Vue：**

```vue
<button @click="onClick">点我</button>
```

**React：**

```tsx
<Pressable onPress={onClick}>
  <Text>点我</Text>
</Pressable>
```

---

## 7. 学习建议：看 `_layout.tsx` 不要一次吃完

`app/_layout.tsx` 这类文件属于“**全局架构级**”代码，一口气看确实会晕。推荐分三步：

1. **先看你关心的那一小段（116–137），知道“有哪些全局东西包在外面”**  
   -> 这个在《02-RootLayout 和全局架构逐行讲解与实战》里有详细拆解。
2. **再回头看前半部分的状态/副作用：**
   - 字体加载（`useFonts`）
   - 启动屏控制（`SplashScreen`）
   - 状态持久化（`useUserStore.persist...`）
   - 多语言初始化（`setupI18n`）
3. **最后才看整体的数据流：**
   - “什么时候认为 app ready？” → `loaded && ready && i18nextReady`
   - “ready 之前显示什么？” → 一个 `Loading` 页面
   - “ready 之后显示什么？” → 你给的那段 JSX 结构（各种 Provider + AppLayout）

你可以先把这份速查表当成「语法/概念翻译器」，然后配合下一份文档（逐行讲解 `_layout.tsx`），就能把那段看不懂的代码拆开啃了。


