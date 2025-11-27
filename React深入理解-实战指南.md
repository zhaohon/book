# React 深入理解实战指南
## 从 Vue 到 React 的深度迁移与实战

---

## 📋 目录

1. [项目架构深度解析](#项目架构深度解析)
2. [路由系统详解](#路由系统详解)
3. [状态管理深入](#状态管理深入)
4. [数据获取与缓存](#数据获取与缓存)
5. [WebSocket 实时通信](#websocket-实时通信)
6. [自定义 Hooks 模式](#自定义-hooks-模式)
7. [组件设计模式](#组件设计模式)
8. [移动端开发（React Native）](#移动端开发react-native)
9. [样式系统](#样式系统)
10. [错误处理与边界](#错误处理与边界)
11. [性能优化实战](#性能优化实战)
12. [开发最佳实践](#开发最佳实践)

---

## 🏗️ 项目架构深度解析

### 1. 项目结构对比

#### ylchat-frontend（Web 端）
```
src/
├── components/        # 可复用组件
│   ├── Common/        # 通用组件
│   ├── Header/        # 头部组件
│   ├── Trade/         # 交易相关组件
│   └── ui/            # UI 基础组件
├── pages/             # 页面组件（路由级别）
├── hooks/             # 自定义 Hooks
│   ├── auth/          # 认证相关
│   ├── fund/          # 资金相关
│   ├── order/         # 订单相关
│   └── marketStompClient/  # WebSocket 相关
├── stores/            # Zustand 状态管理
├── service/           # API 服务层
│   ├── axiosInstance.ts
│   └── rest/          # REST API
├── utils/             # 工具函数
├── types/             # TypeScript 类型定义
└── config/            # 配置文件
```

#### ylchat-mobile（移动端）
```
app/                   # Expo Router 文件系统路由
├── (tabs)/            # Tab 导航组
│   ├── _layout.tsx    # Tab 布局
│   ├── index.tsx      # 首页
│   └── profile.tsx    # 个人中心
├── (auth)/            # 认证路由组
│   ├── _layout.tsx
│   └── index.tsx      # 登录页
└── _layout.tsx         # 根布局

components/             # 组件（与 Web 端类似）
hooks/                  # 自定义 Hooks
stores/                 # Zustand 状态管理
service/                # API 服务层
```

### 2. 架构设计模式

#### 分层架构
```
┌─────────────────────────────────┐
│   Presentation Layer (组件层)   │
│   - UI Components                │
│   - Pages                        │
└──────────────┬──────────────────┘
               │
┌──────────────▼──────────────────┐
│   Business Logic Layer          │
│   - Custom Hooks                │
│   - State Management (Zustand)  │
└──────────────┬──────────────────┘
               │
┌──────────────▼──────────────────┐
│   Data Layer                     │
│   - API Services                 │
│   - WebSocket Clients           │
│   - React Query                  │
└─────────────────────────────────┘
```

---

## 🛣️ 路由系统详解

### 1. Web 端：React Router v7

#### 基础配置
```tsx
// main.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,  // 布局组件
    children: [
      { path: '/', element: <Trade /> },
      { path: '/fund-detail', element: <FundDetail /> },
    ],
  },
])

<RouterProvider router={router} />
```

#### 嵌套路由与 Outlet
```tsx
// App.tsx
function App() {
  return (
    <div>
      <Header />
      <Outlet />  {/* 子路由渲染位置 */}
      <PersonalCenter />
    </div>
  )
}
```

**关键点：**
- `Outlet` 类似 Vue Router 的 `<router-view>`
- 路由配置在顶层，不在组件内
- 使用 `createBrowserRouter` 创建路由实例

#### 编程式导航
```tsx
import { useNavigate, useParams, useSearchParams } from 'react-router-dom'

function Component() {
  const navigate = useNavigate()
  const { id } = useParams()
  const [searchParams, setSearchParams] = useSearchParams()
  
  // 导航
  navigate('/home')
  navigate('/home', { replace: true })  // 替换历史记录
  navigate(-1)  // 返回
  
  // 获取查询参数
  const name = searchParams.get('name')
  setSearchParams({ name: 'new' })
}
```

### 2. 移动端：Expo Router（文件系统路由）

#### 文件系统路由规则
```
app/
├── _layout.tsx          # 根布局
├── index.tsx            # / 路由
├── (tabs)/               # 路由组（不显示在 URL）
│   ├── _layout.tsx      # Tab 布局
│   ├── index.tsx        # /tabs 路由
│   └── profile.tsx      # /tabs/profile 路由
└── profile/
    └── info.tsx         # /profile/info 路由
```

#### 路由组（Route Groups）
```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router/ui'

export default function TabLayout() {
  return (
    <Tabs>
      <TabSlot />  {/* 子路由渲染位置 */}
      <TabList>
        <TabTrigger name="index" href="/">
          <TabButton />
        </TabTrigger>
      </TabList>
    </Tabs>
  )
}
```

**关键点：**
- 文件夹名 `(tabs)` 用括号包裹，不会出现在 URL 中
- `_layout.tsx` 定义布局
- 使用 `expo-router` 的导航 API

#### 导航 API
```tsx
import { router, useRouter, useSegments } from 'expo-router'

// 编程式导航
router.push('/profile')
router.replace('/home')
router.back()

// 获取当前路由信息
const segments = useSegments()  // ['tabs', 'profile']
```

**与 Vue Router 对比：**
- Vue: 配置文件式路由
- Expo Router: 文件系统路由（类似 Next.js）

---

## 🗄️ 状态管理深入

### 1. Zustand 基础

#### Store 创建模式
```tsx
// stores/userStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface UserState {
  token?: string
  userInfo?: UserInfo
}

interface UserActions {
  setToken: (v: string) => void
  setUserInfo: (v: UserInfo) => void
  reset: () => void
}

const useUserStore = create<UserState & UserActions>()(
  persist(
    (set) => ({
      // 初始状态
      token: undefined,
      userInfo: undefined,
      
      // Actions
      setToken: (v) => set({ token: v }),
      setUserInfo: (v) => set({ userInfo: v }),
      reset: () => set({ token: undefined, userInfo: undefined }),
    }),
    {
      name: 'userInfo',  // localStorage key
      partialize: (state) => ({
        // 只持久化需要的字段
        token: state.token,
        userInfo: state.userInfo,
      }),
    }
  )
)
```

### 2. 在组件中使用

#### 方式一：直接解构（推荐）
```tsx
function Component() {
  const { token, setToken } = useUserStore()
  
  return <div>{token}</div>
}
```

#### 方式二：选择器（性能优化）
```tsx
// 只订阅 token，userInfo 变化不会触发重渲染
const token = useUserStore(state => state.token)
const setToken = useUserStore(state => state.setToken)
```

#### 方式三：浅比较（对象/数组）
```tsx
import { shallow } from 'zustand/shallow'

const { userInfo, setUserInfo } = useUserStore(
  state => ({ userInfo: state.userInfo, setUserInfo: state.setUserInfo }),
  shallow
)
```

### 3. 异步操作

```tsx
const useUserStore = create((set, get) => ({
  userInfo: null,
  loading: false,
  
  fetchUser: async (id: string) => {
    set({ loading: true })
    try {
      const user = await fetchUserAPI(id)
      set({ userInfo: user, loading: false })
    } catch (error) {
      set({ loading: false })
      throw error
    }
  },
}))
```

### 4. 持久化（Web vs Mobile）

#### Web 端（localStorage）
```tsx
import { createJSONStorage } from 'zustand/middleware'

persist(
  (set) => ({ ... }),
  {
    name: 'userInfo',
    storage: createJSONStorage(() => localStorage),
  }
)
```

#### Mobile 端（AsyncStorage）
```tsx
import AsyncStorage from '@react-native-async-storage/async-storage'
import { createJSONStorage } from 'zustand/middleware'

persist(
  (set) => ({ ... }),
  {
    name: 'userInfo',
    storage: createJSONStorage(() => AsyncStorage),
  }
)
```

### 5. 状态持久化 Hydration

```tsx
// Mobile 端需要等待 hydration 完成
const [userHydrated, setUserHydrated] = useState(
  useUserStore.persist?.hasHydrated() ?? false
)

useEffect(() => {
  const unsubscribe = useUserStore.persist.onFinishHydration(() => {
    setUserHydrated(true)
  })
  
  if (useUserStore.persist?.hasHydrated()) {
    setUserHydrated(true)
  }
  
  return unsubscribe
}, [])
```

---

## 📡 数据获取与缓存

### 1. React Query 核心概念

#### Query（查询数据）
```tsx
import { useQuery } from '@tanstack/react-query'

function Component() {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['user', userId],  // 缓存 key
    queryFn: () => fetchUser(userId),  // 查询函数
    enabled: !!userId,  // 是否启用查询
    staleTime: 5 * 60 * 1000,  // 数据新鲜时间（5分钟）
    gcTime: 10 * 60 * 1000,  // 缓存保留时间（10分钟）
    refetchOnWindowFocus: false,  // 窗口聚焦时不重新获取
  })
  
  if (isLoading) return <div>加载中...</div>
  if (error) return <div>错误: {error.message}</div>
  
  return <div>{data?.name}</div>
}
```

#### Mutation（修改数据）
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function Component() {
  const queryClient = useQueryClient()
  
  const mutation = useMutation({
    mutationFn: (data) => createUser(data),
    onSuccess: (data) => {
      // 成功后更新缓存
      queryClient.setQueryData(['user', data.id], data)
      // 或者使相关查询失效，触发重新获取
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
    onError: (error) => {
      // 错误处理
      console.error(error)
    },
  })
  
  const handleSubmit = () => {
    mutation.mutate({ name: 'John' })
  }
  
  return (
    <button onClick={handleSubmit} disabled={mutation.isPending}>
      {mutation.isPending ? '提交中...' : '提交'}
    </button>
  )
}
```

### 2. 项目中的实际应用

#### 自定义 Hook 封装
```tsx
// hooks/user/useGetUserInfo.ts
import { getUserInfo } from '@/service/rest/user'
import useUserStore from '@/stores/userStore'
import { useQuery } from '@tanstack/react-query'
import { useEffect } from 'react'

export function useGetUserInfo() {
  const setUserInfo = useUserStore((state) => state.setUserInfo)
  const token = useUserStore((state) => state.token)

  const { data, refetch } = useQuery({
    queryKey: ['fetch-user-info', token],
    queryFn: getUserInfo,
    enabled: !!token,  // 只有 token 存在时才查询
    refetchOnWindowFocus: false,
  })

  // 将查询结果同步到 Zustand store
  useEffect(() => {
    if (data?.data) {
      setUserInfo(data?.data)
    }
  }, [data?.data, setUserInfo])

  return { refetch }
}
```

**设计模式：**
- React Query 负责数据获取和缓存
- Zustand 负责全局状态管理
- 通过 `useEffect` 同步数据

### 3. 查询依赖与条件查询

```tsx
// 依赖其他查询
const { data: user } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})

const { data: posts } = useQuery({
  queryKey: ['posts', userId],
  queryFn: () => fetchPosts(userId),
  enabled: !!user,  // 只有 user 存在时才查询
})
```

### 4. 乐观更新（Optimistic Updates）

```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newUser) => {
    // 取消正在进行的查询
    await queryClient.cancelQueries({ queryKey: ['user', newUser.id] })
    
    // 保存当前值
    const previousUser = queryClient.getQueryData(['user', newUser.id])
    
    // 乐观更新
    queryClient.setQueryData(['user', newUser.id], newUser)
    
    return { previousUser }
  },
  onError: (err, newUser, context) => {
    // 回滚
    queryClient.setQueryData(['user', newUser.id], context.previousUser)
  },
  onSettled: () => {
    // 重新获取确保数据最新
    queryClient.invalidateQueries({ queryKey: ['user'] })
  },
})
```

---

## 🔌 WebSocket 实时通信

### 1. STOMP 协议基础

项目使用 `@stomp/stompjs` + `sockjs-client` 实现 WebSocket 通信。

#### 连接建立
```tsx
// hooks/marketStompClient/useConnectMarketStomp.ts
import { Client } from '@stomp/stompjs'
import SockJS from 'sockjs-client'

export function useConnectMarketStomp() {
  const clientRef = useRef<Client | undefined>(undefined)
  const token = useUserStore((s) => s.token)
  
  const connectWebSocket = () => {
    const client = new Client({
      // 使用 SockJS 作为传输层
      webSocketFactory: () =>
        new SockJS(`${import.meta.env.VITE_API_URL}/market-ws`),
      
      // 连接头（包含认证 token）
      connectHeaders: {
        Authorization: token,
        'heart-beat': '10000,10000',  // 心跳间隔
      },
      
      // 重连设置
      reconnectDelay: 5000,
      
      // 连接成功回调
      onConnect: function () {
        console.log('WebSocket 连接已建立')
        
        // 订阅主题
        client.subscribe('/user/topic/price/', handlePriceMessage)
        client.subscribe('/user/topic/orderBook/', handleOrderBookMessage)
      },
      
      // 错误回调
      onStompError: function (frame) {
        console.error('连接错误:', frame.headers['message'])
      },
      
      // 断开回调
      onDisconnect: function () {
        console.log('连接已断开')
      },
    })
    
    clientRef.current = client
    client.activate()  // 激活连接
  }
  
  useEffect(() => {
    connectWebSocket()
    return () => {
      clientRef.current?.deactivate()  // 组件卸载时断开
    }
  }, [])
}
```

### 2. 消息订阅与处理

```tsx
// 订阅消息
client.subscribe('/user/topic/price/', (message) => {
  const data = JSON.parse(message.body)
  // 处理消息
  handlePriceData(data)
})

// 发送消息
client.publish({
  destination: '/app/subscribe/price',
  body: JSON.stringify({ productCode: 'BTC/USDT' }),
})
```

### 3. 连接管理最佳实践

```tsx
// 1. 使用 useRef 保存客户端引用
const clientRef = useRef<Client | undefined>(undefined)

// 2. 在 useEffect 中管理生命周期
useEffect(() => {
  connectWebSocket()
  return () => {
    clientRef.current?.deactivate()
  }
}, [])

// 3. 处理重连逻辑
const client = new Client({
  reconnectDelay: 5000,
  // 自动重连
})

// 4. 心跳检测
connectHeaders: {
  'heart-beat': '10000,10000',  // 10秒心跳
}
```

### 4. 多 WebSocket 连接管理

项目中有两个 WebSocket 连接：
- `market-ws`: 市场数据（价格、K线、盘口）
- `order-ws`: 订单数据（订单状态、持仓）

```tsx
// 分别管理
useConnectMarketStomp()  // 市场数据
useConnectOrderStomp()    // 订单数据
```

---

## 🎣 自定义 Hooks 模式

### 1. 数据获取 Hook

```tsx
// hooks/user/useGetUserInfo.ts
export function useGetUserInfo() {
  const setUserInfo = useUserStore((state) => state.setUserInfo)
  const token = useUserStore((state) => state.token)

  const { data, refetch } = useQuery({
    queryKey: ['fetch-user-info', token],
    queryFn: getUserInfo,
    enabled: !!token,
  })

  useEffect(() => {
    if (data?.data) {
      setUserInfo(data?.data)
    }
  }, [data?.data, setUserInfo])

  return { refetch }
}
```

**使用：**
```tsx
function Component() {
  const { refetch } = useGetUserInfo()
  // 自动获取用户信息并更新 store
}
```

### 2. 业务逻辑 Hook

```tsx
// hooks/action/useValidateOrder.ts
export function useValidateOrder() {
  const { data: productConfig } = useGetProductConfig()
  const balance = useBalanceStore((state) => state.balance)
  
  const validateOrder = (orderData: OrderData) => {
    // 验证逻辑
    if (orderData.volume > balance) {
      throw new Error('余额不足')
    }
    // ...
  }
  
  return { validateOrder }
}
```

### 3. WebSocket Hook

```tsx
// hooks/marketStompClient/useConnectMarketStomp.ts
export function useConnectMarketStomp() {
  const clientRef = useRef<Client | undefined>(undefined)
  const [connected, setConnected] = useState(false)
  
  useEffect(() => {
    const client = new Client({ ... })
    clientRef.current = client
    client.activate()
    
    return () => {
      clientRef.current?.deactivate()
    }
  }, [])
  
  return { connected, client: clientRef.current }
}
```

### 4. 组合 Hooks

```tsx
// 在组件中组合使用多个 Hooks
function TradeComponent() {
  // 数据获取
  const { data: products } = useGetProducts()
  const { data: balance } = useGetBalance()
  
  // 业务逻辑
  const { validateOrder } = useValidateOrder()
  
  // WebSocket
  useConnectMarketStomp()
  
  // 状态管理
  const selectedProduct = useProductStore(state => state.selectedProduct)
}
```

---

## 🧩 组件设计模式

### 1. 容器组件与展示组件

```tsx
// 展示组件（纯组件，只负责 UI）
function ProductCard({ product, onSelect }: ProductCardProps) {
  return (
    <div onClick={() => onSelect(product)}>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  )
}

// 容器组件（负责数据获取和业务逻辑）
function ProductList() {
  const { data: products } = useGetProducts()
  const selectProduct = useProductStore(state => state.selectProduct)
  
  return (
    <div>
      {products?.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onSelect={selectProduct}
        />
      ))}
    </div>
  )
}
```

### 2. 复合组件模式

```tsx
// 类似 Ant Design 的 Form.Item
function Form({ children }: { children: React.ReactNode }) {
  return <form>{children}</form>
}

function FormItem({ label, children }: FormItemProps) {
  return (
    <div>
      <label>{label}</label>
      {children}
    </div>
  )
}

// 使用
<Form>
  <FormItem label="用户名">
    <Input />
  </FormItem>
</Form>
```

### 3. Render Props 模式

```tsx
function DataFetcher({ 
  queryKey, 
  queryFn, 
  children 
}: DataFetcherProps) {
  const { data, isLoading, error } = useQuery({
    queryKey,
    queryFn,
  })
  
  return children({ data, isLoading, error })
}

// 使用
<DataFetcher queryKey={['user']} queryFn={fetchUser}>
  {({ data, isLoading, error }) => {
    if (isLoading) return <Loading />
    if (error) return <Error />
    return <UserInfo user={data} />
  }}
</DataFetcher>
```

### 4. 高阶组件（HOC）

```tsx
function withAuth<P extends object>(Component: React.ComponentType<P>) {
  return function AuthenticatedComponent(props: P) {
    const token = useUserStore(state => state.token)
    
    if (!token) {
      return <Navigate to="/login" />
    }
    
    return <Component {...props} />
  }
}

// 使用
const ProtectedPage = withAuth(MyPage)
```

---

## 📱 移动端开发（React Native）

### 1. Expo Router 文件系统路由

#### 路由结构
```
app/
├── _layout.tsx          # 根布局
├── index.tsx             # / 首页
├── (tabs)/               # Tab 导航组
│   ├── _layout.tsx      # Tab 布局
│   ├── index.tsx        # Tab 首页
│   └── profile.tsx      # Tab 个人中心
└── profile/
    └── info.tsx         # /profile/info
```

#### 导航
```tsx
import { router } from 'expo-router'

// 导航
router.push('/profile/info')
router.replace('/home')
router.back()

// 获取路由参数
import { useLocalSearchParams } from 'expo-router'
const { id } = useLocalSearchParams()
```

### 2. 样式系统（NativeWind）

```tsx
// 使用 Tailwind CSS 类名
<View className="flex-1 bg-white p-4">
  <Text className="text-lg font-bold text-gray-800">
    Hello World
  </Text>
</View>
```

### 3. 平台特定代码

```tsx
// 使用 Platform 检测平台
import { Platform } from 'react-native'

if (Platform.OS === 'ios') {
  // iOS 特定代码
} else if (Platform.OS === 'android') {
  // Android 特定代码
}

// 平台特定文件
// Component.ios.tsx
// Component.android.tsx
```

### 4. 原生模块使用

```tsx
// 使用 Expo 模块
import * as ImagePicker from 'expo-image-picker'
import * as Haptics from 'expo-haptics'

// 选择图片
const result = await ImagePicker.launchImageLibraryAsync()

// 触觉反馈
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)
```

---

## 🎨 样式系统

### 1. Tailwind CSS

#### Web 端
```tsx
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow">
  <h2 className="text-xl font-bold text-gray-800">标题</h2>
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    按钮
  </button>
</div>
```

#### Mobile 端（NativeWind）
```tsx
<View className="flex-row items-center justify-between p-4 bg-white rounded-lg">
  <Text className="text-xl font-bold text-gray-800">标题</Text>
  <TouchableOpacity className="px-4 py-2 bg-blue-500 rounded">
    <Text className="text-white">按钮</Text>
  </TouchableOpacity>
</View>
```

### 2. Chakra UI（Web 端）

```tsx
import { Box, Button, Text } from '@chakra-ui/react'

<Box p={4} bg="white" borderRadius="lg">
  <Text fontSize="xl" fontWeight="bold">
    标题
  </Text>
  <Button colorScheme="blue">按钮</Button>
</Box>
```

### 3. CSS Modules / Less

```tsx
// styles.module.less
.container {
  padding: 16px;
  background: white;
}

// 使用
import styles from './styles.module.less'

<div className={styles.container}>内容</div>
```

---

## ⚠️ 错误处理与边界

### 1. Error Boundary

```tsx
// components/Common/FallbackRender.tsx
import { ErrorBoundary } from 'react-error-boundary'

function FallbackRender({ error, resetErrorBoundary }) {
  return (
    <div>
      <h2>出错了</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>重试</button>
    </div>
  )
}

// 使用
<ErrorBoundary FallbackComponent={FallbackRender}>
  <App />
</ErrorBoundary>
```

### 2. API 错误处理

```tsx
// service/errorHandler.ts
export const errorHandler = (error: any) => {
  const status = error?.response?.status
  
  switch (status) {
    case 401:
      // 未授权，清除 token 并跳转登录
      localStorage.removeItem('token')
      window.location.href = '/login'
      break
    case 403:
      // 无权限
      Toast.error('无权限访问')
      break
    default:
      Toast.error(error?.message || '请求失败')
  }
}
```

### 3. React Query 错误处理

```tsx
const { data, error } = useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
  retry: 3,  // 重试 3 次
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  onError: (error) => {
    // 全局错误处理
    errorHandler(error)
  },
})
```

---

## ⚡ 性能优化实战

### 1. React.memo

```tsx
// 防止不必要的重渲染
const ProductCard = React.memo(({ product }: ProductCardProps) => {
  return <div>{product.name}</div>
}, (prevProps, nextProps) => {
  // 自定义比较函数
  return prevProps.product.id === nextProps.product.id
})
```

### 2. useMemo 和 useCallback

```tsx
// 缓存计算结果
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b)
}, [a, b])

// 缓存函数引用
const handleClick = useCallback(() => {
  doSomething(a, b)
}, [a, b])
```

### 3. 代码分割（Lazy Loading）

```tsx
// 路由懒加载
const Trade = lazy(() => import('./pages/Trade/Trade'))

<Suspense fallback={<Loading />}>
  <Trade />
</Suspense>
```

### 4. 虚拟列表（大列表优化）

```tsx
// 使用 react-window 或 react-virtualized
import { FixedSizeList } from 'react-window'

<FixedSizeList
  height={600}
  itemCount={items.length}
  itemSize={50}
  width="100%"
>
  {({ index, style }) => (
    <div style={style}>{items[index]}</div>
  )}
</FixedSizeList>
```

### 5. Zustand 选择器优化

```tsx
// ❌ 错误：每次都会创建新对象
const { user, setUser } = useUserStore()

// ✅ 正确：使用选择器
const user = useUserStore(state => state.user)
const setUser = useUserStore(state => state.setUser)

// ✅ 或者使用 shallow
const { user, setUser } = useUserStore(
  state => ({ user: state.user, setUser: state.setUser }),
  shallow
)
```

---

## 💡 开发最佳实践

### 1. 文件命名规范

```
components/
  UserCard/
    index.tsx          # 主组件
    UserCard.tsx       # 或者直接命名
    UserCard.module.less
    types.ts           # 类型定义
```

### 2. 类型定义

```tsx
// types/user.ts
export interface User {
  id: string
  name: string
  email: string
}

// 组件中使用
interface UserCardProps {
  user: User
  onSelect?: (user: User) => void
}
```

### 3. 自定义 Hooks 命名

```tsx
// 数据获取：use + 动词 + 名词
useGetUser()
useFetchProducts()

// 业务逻辑：use + 动词
useValidateOrder()
useHandleSubmit()

// 状态：use + 名词
useUserStore()
useTheme()
```

### 4. 组件组织

```tsx
// 1. Imports（分组）
import React from 'react'
import { useQuery } from '@tanstack/react-query'
import { useUserStore } from '@/stores/userStore'
import { Button } from '@/components/ui/Button'
import styles from './styles.module.less'

// 2. Types
interface Props { ... }

// 3. Component
export function Component({ ... }: Props) {
  // 4. Hooks
  const user = useUserStore(state => state.user)
  const { data } = useQuery(...)
  
  // 5. Handlers
  const handleClick = () => { ... }
  
  // 6. Render
  return <div>...</div>
}
```

### 5. 状态管理原则

- **本地状态**：使用 `useState`
- **组件间共享**：使用 `useContext` 或 Props
- **全局状态**：使用 Zustand
- **服务器状态**：使用 React Query

### 6. 测试

```tsx
// 使用 React Testing Library
import { render, screen } from '@testing-library/react'
import { Component } from './Component'

test('renders component', () => {
  render(<Component />)
  expect(screen.getByText('Hello')).toBeInTheDocument()
})
```

---

## 📚 总结

### 关键差异总结

| 方面 | Vue 3 | React |
|------|-------|-------|
| **组件** | `<template>` + `<script setup>` | 函数返回 JSX |
| **状态** | `ref()`, `reactive()` | `useState()`, Zustand |
| **计算** | `computed()` | `useMemo()` |
| **监听** | `watch()` | `useEffect()` |
| **路由** | Vue Router | React Router / Expo Router |
| **数据获取** | `useFetch()` | React Query |
| **样式** | Scoped CSS | Tailwind / CSS Modules |

### 学习路径

1. **基础**：React Hooks、JSX 语法
2. **路由**：React Router / Expo Router
3. **状态**：Zustand
4. **数据**：React Query
5. **样式**：Tailwind CSS
6. **移动端**：React Native + Expo

### 实战建议

1. 先理解项目结构，找到入口文件
2. 阅读一个完整的功能模块（从路由到组件到 API）
3. 尝试修改一个小功能
4. 理解 WebSocket 连接逻辑
5. 熟悉自定义 Hooks 的使用模式

**祝你开发顺利！** 🚀

