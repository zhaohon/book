# React 快速上手学习指南
## 针对区块链项目维护（Vue开发者转React）

---

## 📚 一、核心基础知识（必须掌握）

### 1. React 基础概念（与Vue对比）

#### 组件定义方式
```tsx
// React 函数组件（类似Vue 3 Composition API）
export default function Home() {
  const { t } = useTranslation()  // 类似Vue的useTranslation()
  
  return (
    <div className={styles.homePage}>
      <Header />
    </div>
  )
}
```

**Vue对比：**
- Vue: `<template>` + `<script setup>`
- React: 直接在函数中返回JSX

#### Hooks（类似Vue的Composition API）
```tsx
// useState - 类似Vue的ref/reactive
const [count, setCount] = useState(0)

// useEffect - 类似Vue的watch/onMounted/onUnmounted
useEffect(() => {
  // 组件挂载时执行
  return () => {
    // 组件卸载时清理
  }
}, [dependencies])

// useMemo - 类似Vue的computed
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])
```

**关键差异：**
- React的依赖数组必须手动管理
- useEffect的清理函数在依赖变化或卸载时执行
- 没有Vue的自动依赖追踪

---

## 🔧 二、项目使用的核心技术栈

### 1. **路由管理：React Router v7**

**项目中的使用：**
```tsx
// router/index.tsx
import { createBrowserRouter, Navigate } from 'react-router-dom'
import { lazy } from 'react'

// 路由懒加载
const router = [
  {
    path: '/home',
    element: lazyLoad(lazy(() => import('@/views/home'))),
  },
  {
    path: '/',
    element: <Navigate to="/mining" />,  // 重定向
  },
]

export default createBrowserRouter(router)
```

**需要学习：**
- `createBrowserRouter` - 创建路由配置
- `RouterProvider` - 路由提供者
- `useNavigate` - 编程式导航（类似Vue的`router.push`）
- `useParams` - 获取路由参数（类似Vue的`route.params`）
- `Navigate` - 重定向组件
- `lazy` + `Suspense` - 路由懒加载

**Vue对比：**
- Vue Router: `router.push()`, `route.params`
- React Router: `navigate()`, `useParams()`

---

### 2. **状态管理：Zustand**

**项目中的使用：**
```tsx
// store/userStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

const useUserStore = create(
  persist(
    (set) => ({
      token: undefined,
      userInfo: undefined,
      setToken: (v) => set({ token: v }),
      reset: () => set({ token: undefined, userInfo: undefined }),
    }),
    {
      name: 'userInfo',
      storage: createJSONStorage(() => localStorage),
    }
  )
)

// 在组件中使用
const { token, setToken } = useUserStore()
```

**需要学习：**
- `create` - 创建store
- `persist` - 持久化中间件（类似Vue的pinia-plugin-persistedstate）
- `set` - 更新状态（类似Vue的`$patch`）
- 在组件中直接解构使用（类似Pinia）

**Vue对比：**
- Pinia: `defineStore`, `store.xxx`
- Zustand: `create`, 直接解构使用

---

### 3. **数据获取：React Query (@tanstack/react-query)**

**项目中的使用：**
```tsx
// App.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()

<QueryClientProvider client={queryClient}>
  <RouterProvider router={router} />
</QueryClientProvider>
```

**需要学习：**
- `useQuery` - 获取数据（类似Vue的`useFetch`）
- `useMutation` - 修改数据（POST/PUT/DELETE）
- `QueryClient` - 查询客户端配置
- 缓存、重新获取、错误处理

**Vue对比：**
- Vue: `useFetch`, `$fetch`
- React Query: `useQuery`, `useMutation`（功能更强大，有缓存）

---

### 4. **UI组件库**

#### blockchain项目：Ant Design Mobile
```tsx
import { Toast } from 'antd-mobile'

Toast.show({
  content: '提示信息',
})
```

#### blockchain-admin项目：Ant Design
```tsx
import { Button, Table, Form } from 'antd'
```

**需要学习：**
- 组件API文档
- Form表单处理（与Vue的`el-form`类似）
- Table表格（与Vue的`el-table`类似）
- 主题定制

---

### 5. **样式方案**

#### Tailwind CSS（原子化CSS）
```tsx
<div className="h-[100vh] flex flex-col items-center justify-center gap-9">
  <div className="font-bold text-[#999BA9]">内容</div>
</div>
```

#### Less（CSS预处理器）
```tsx
import styles from './index.module.less'

<div className={styles.homePage}>
```

**需要学习：**
- Tailwind常用类名（flex, grid, spacing等）
- Less模块化样式
- CSS Modules（`styles.className`）

---

### 6. **国际化：i18next**

**项目中的使用：**
```tsx
import { useTranslation } from 'react-i18next'

function Component() {
  const { t } = useTranslation()
  return <div>{t('home.yourFutureExchange')}</div>
}
```

**Vue对比：**
- Vue: `$t('key')` 或 `t('key')`
- React: `t('key')`（使用方式相同）

---

### 7. **HTTP请求：Axios**

**项目中的封装：**
```tsx
// utils/request/index.ts
import axios from 'axios'

const instance = axios.create({
  baseURL: import.meta.env.VITE_BASE_URL,
  timeout: 10000,
})

// 请求拦截器
instance.interceptors.request.use(config => {
  const token = getToken()
  config.headers.Authorization = token
  return config
})

// 响应拦截器
instance.interceptors.response.use(response => {
  return response.data
})
```

**与Vue相同：** Axios的使用方式完全一致

---

### 8. **WebSocket：STOMP协议**

**项目中的使用：**
```tsx
import { Client } from '@stomp/stompjs'
import SockJS from 'sockjs-client'

const client = new Client({
  webSocketFactory: () => new SockJS(url),
  onConnect: () => {
    client.subscribe('/topic/xxx', message => {
      // 处理消息
    })
  },
})
```

**需要学习：**
- STOMP协议基础
- 连接、订阅、取消订阅
- 错误处理和重连

---

### 9. **区块链相关：Ethers.js**

**项目中的使用：**
```tsx
import { ethers } from 'ethers'

// 连接钱包、签名交易等
```

**需要学习：**
- 钱包连接（MetaMask等）
- 合约交互
- 交易签名

---

## 🏗️ 三、项目构建架构

### 1. **构建工具：Vite**

**配置文件：vite.config.ts**
```tsx
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3005,
  },
})
```

**与Vue相同：** Vite配置方式基本一致

---

### 2. **TypeScript**

**需要掌握：**
- 类型定义（interface, type）
- 泛型
- 类型推断
- 与Vue的TypeScript使用方式相同

---

### 3. **代码规范工具**

- **ESLint** - 代码检查
- **Prettier** - 代码格式化
- **Husky** - Git hooks
- **Commitlint** - 提交信息规范

---

## 📁 四、项目目录结构

```
blockchain/
├── src/
│   ├── api/              # API接口定义
│   ├── components/       # 公共组件
│   ├── hooks/            # 自定义Hooks
│   ├── layout/           # 布局组件
│   ├── router/           # 路由配置
│   ├── store/            # 状态管理（Zustand）
│   ├── styles/           # 全局样式
│   ├── utils/            # 工具函数
│   ├── views/            # 页面组件
│   ├── locales/          # 国际化文件
│   ├── types/            # TypeScript类型定义
│   ├── App.tsx           # 根组件
│   └── main.tsx          # 入口文件
```

---

## 🎯 五、四天学习计划

### 第1天：React基础 + 路由
- [ ] React基础语法（组件、JSX、Props）
- [ ] Hooks（useState, useEffect, useMemo）
- [ ] React Router v7基础使用
- [ ] 完成一个小demo（路由跳转、状态管理）

### 第2天：状态管理 + 数据获取
- [ ] Zustand基础使用
- [ ] React Query基础使用
- [ ] 理解项目中的store结构
- [ ] 理解项目中的API调用方式

### 第3天：UI组件 + 样式
- [ ] Ant Design / Ant Design Mobile文档
- [ ] Tailwind CSS常用类名
- [ ] Less/CSS Modules使用
- [ ] 查看项目中的组件实现

### 第4天：项目实战
- [ ] 阅读项目代码，理解业务逻辑
- [ ] 尝试修改一个小功能
- [ ] 理解WebSocket连接逻辑
- [ ] 理解国际化配置

---

## 🔑 六、关键差异总结（Vue vs React）

| 特性 | Vue 3 | React |
|------|-------|-------|
| **组件定义** | `<template>` + `<script setup>` | 函数返回JSX |
| **状态管理** | `ref()`, `reactive()` | `useState()`, Zustand |
| **生命周期** | `onMounted()`, `onUnmounted()` | `useEffect()` |
| **计算属性** | `computed()` | `useMemo()` |
| **路由导航** | `router.push()` | `navigate()` |
| **获取参数** | `route.params` | `useParams()` |
| **样式绑定** | `:class`, `:style` | `className`, `style={{}}` |
| **条件渲染** | `v-if`, `v-show` | `{condition && <div>}` |
| **列表渲染** | `v-for` | `{list.map(item => <div>)}` |
| **事件处理** | `@click` | `onClick` |
| **双向绑定** | `v-model` | `value` + `onChange` |

---

## 📖 七、推荐学习资源

### 官方文档（必读）
1. **React官方文档**：https://react.dev
2. **React Router v7**：https://reactrouter.com
3. **Zustand**：https://zustand-demo.pmnd.rs
4. **React Query**：https://tanstack.com/query/latest
5. **Ant Design**：https://ant.design
6. **Ant Design Mobile**：https://mobile.ant.design
7. **Tailwind CSS**：https://tailwindcss.com

### 快速上手视频
- React基础教程（B站搜索"React入门"）
- React Router教程
- Zustand状态管理教程

---

## ⚠️ 八、常见陷阱和注意事项

### 1. **依赖数组问题**
```tsx
// ❌ 错误：缺少依赖
useEffect(() => {
  fetchData(id)
}, [])  // id变化时不会重新执行

// ✅ 正确
useEffect(() => {
  fetchData(id)
}, [id])
```

### 2. **状态更新是异步的**
```tsx
// ❌ 错误：立即读取新值
setCount(count + 1)
console.log(count)  // 还是旧值

// ✅ 正确：使用函数式更新
setCount(prev => prev + 1)
```

### 3. **条件渲染**
```tsx
// ✅ React的条件渲染
{isShow && <Component />}
{isShow ? <ComponentA /> : <ComponentB />}
```

### 4. **列表渲染必须加key**
```tsx
// ✅ 必须加key
{list.map(item => (
  <div key={item.id}>{item.name}</div>
))}
```

---

## 🚀 九、快速上手建议

1. **先看项目入口**：`main.tsx` → `App.tsx` → `router/index.tsx`
2. **理解路由结构**：查看`router/index.tsx`了解所有页面
3. **理解状态管理**：查看`store/`目录下的store定义
4. **理解API调用**：查看`api/`目录和`utils/request/`
5. **看一个完整页面**：从路由 → 组件 → API → Store，完整走一遍

---

## 💡 十、实战练习建议

### 练习1：添加一个新页面
1. 在`views/`下创建新组件
2. 在`router/index.tsx`中添加路由
3. 测试路由跳转

### 练习2：添加一个API调用
1. 在`api/`下定义接口
2. 在组件中使用`useQuery`或`useMutation`
3. 处理加载和错误状态

### 练习3：添加一个状态管理
1. 在`store/`下创建新的store
2. 在组件中使用store
3. 测试状态更新

---

## 📝 总结

作为Vue开发者，你已经掌握了：
- ✅ 组件化思想
- ✅ 状态管理概念
- ✅ 路由管理
- ✅ 生命周期概念
- ✅ TypeScript使用

**主要需要学习：**
- React的语法和Hooks
- React Router的使用方式
- Zustand状态管理
- React Query数据获取
- JSX语法

**四天时间足够上手维护项目！** 重点是理解项目结构，遇到问题查文档即可。

---

**祝你学习顺利！有问题随时查阅文档或询问团队。** 🎉

