# React 代码示例与实战模式
## 项目真实代码示例解析

---

## 📋 目录

1. [完整页面组件示例](#完整页面组件示例)
2. [自定义 Hooks 示例](#自定义-hooks-示例)
3. [状态管理示例](#状态管理示例)
4. [API 服务层示例](#api-服务层示例)
5. [WebSocket 连接示例](#websocket-连接示例)
6. [表单处理示例](#表单处理示例)
7. [条件渲染模式](#条件渲染模式)
8. [性能优化示例](#性能优化示例)

---

## 📄 完整页面组件示例

### 示例：Trade 页面

```tsx
// pages/Trade/Trade.tsx
import LeftMenu from "@/components/Trade/LeftMenu/LeftMenu"
import Optional from "@/components/Trade/Optional/Optional"
import PairList from "@/components/Trade/PairList/PairList"
import useAppSettingStore from "@/stores/appSettingStore"
import Chart from "@/components/Trade/Chart/Chart"
import useTradeListStore from "@/stores/tradeListStore"
import { TradeListMode } from "@/types/app"
import ActionWithFuture from "@/components/Trade/Action/ActionWithFuture"
import AliveTradeAndFutureTable from "@/components/Trade/TradeTable/AliveTradeAndFutureTable"
import HistoryTradeAndFutureTable from "@/components/Trade/TradeTable/HistoryTradeAndFutureTable"
import { Flex } from "@chakra-ui/react"

const Trade = () => {
  // 1. 从 Store 获取状态（使用选择器优化性能）
  const showProductList = useAppSettingStore((state) => state.showProductList)
  const showOptional = useAppSettingStore((state) => state.showOptional)
  const showPlaceOrder = useAppSettingStore((state) => state.showPlaceOrder)
  const mode = useTradeListStore((s) => s.mode)

  // 2. 渲染 UI
  return (
    <Flex
      minH="calc(100dvh - var(--header-height))"
      h={{ base: "100%", xl: "calc(100dvh - var(--header-height))" }}
    >
      <LeftMenu />
      <div className="flex flex-col w-full min-w-[var(--min-content-width)]">
        <Flex
          className="h-full"
          flexWrap={{ base: "wrap", xl: "nowrap" }}
          style={{
            height:
              mode !== TradeListMode.None
                ? "calc(100% - var(--table-height))"
                : "100%",
          }}
        >
          {/* 3. 条件渲染 */}
          {showProductList && <PairList />}
          {showOptional && <Optional />}
          {showPlaceOrder && <ActionWithFuture />}
          <Chart />
        </Flex>
        <div className="flex-shrink-0 bg-f1 relative">
          {mode === TradeListMode.Alive && <AliveTradeAndFutureTable />}
          {mode === TradeListMode.History && <HistoryTradeAndFutureTable />}
        </div>
      </div>
    </Flex>
  )
}

export default Trade
```

**关键点：**
- 使用 Zustand 选择器获取状态
- 条件渲染使用 `&&` 和三元运算符
- 混合使用 Chakra UI 和 Tailwind CSS
- 响应式样式使用对象语法

---

## 🎣 自定义 Hooks 示例

### 示例 1：数据获取 Hook

```tsx
// hooks/user/useGetUserInfo.ts
import { getUserInfo } from "@/service/rest/user"
import useUserStore from "@/stores/userStore"
import { useQuery } from "@tanstack/react-query"
import { useEffect } from "react"

export function useGetUserInfo() {
  // 1. 从 Store 获取依赖
  const setUserInfo = useUserStore((state) => state.setUserInfo)
  const token = useUserStore((state) => state.token)

  // 2. 使用 React Query 获取数据
  const { data, refetch } = useQuery({
    queryKey: ["fetch-user-info", token],  // 缓存 key，包含 token
    queryFn: getUserInfo,  // 查询函数
    enabled: !!token,  // 只有 token 存在时才查询
    refetchOnWindowFocus: false,  // 窗口聚焦时不重新获取
  })

  // 3. 将查询结果同步到 Zustand Store
  useEffect(() => {
    if (data?.data) {
      setUserInfo(data?.data)
    }
  }, [data?.data, setUserInfo])

  // 4. 返回需要的方法和数据
  return { refetch }
}
```

**使用：**
```tsx
function Component() {
  // 自动获取用户信息并更新 store
  useGetUserInfo()
  
  // 从 store 读取用户信息
  const userInfo = useUserStore(state => state.userInfo)
  
  return <div>{userInfo?.name}</div>
}
```

### 示例 2：带参数的 Hook

```tsx
// hooks/fund/useFetchBalance.ts
import { getBalance } from "@/service/rest/fund"
import { useBalanceStore } from "@/stores/balanceStore"
import useUserStore from "@/stores/userStore"
import { useQuery, useQueryClient } from "@tanstack/react-query"
import { useEffect } from "react"

export function useFetchBalance({
  enabled = true,
}: { enabled?: boolean } = {}) {
  const token = useUserStore((state) => state.token)
  const userInfo = useUserStore((state) => state.userInfo)
  const setBalance = useBalanceStore((state) => state.setBalance)
  const queryClient = useQueryClient()

  // 1. 基础查询
  const { data, refetch: originalRefetch } = useQuery({
    queryKey: ["fetch-balance", token, userInfo?.mock],
    queryFn: () => getBalance(!!userInfo?.mock),
    enabled: enabled && !!token,
    refetchInterval: 5 * 1000,  // 每 5 秒自动刷新
  })

  // 2. 同步到 Store
  useEffect(() => {
    setBalance(data?.data)
  }, [data, setBalance])

  // 3. 自定义 refetch，支持参数覆盖
  const refetch = (mockOverride?: boolean) => {
    if (mockOverride === undefined) {
      return originalRefetch()  // 默认行为
    }

    // 用新参数做一次手动请求
    return queryClient.fetchQuery({
      queryKey: ["fetch-balance", token, mockOverride],
      queryFn: () => getBalance(mockOverride),
    })
  }

  return { balance: data?.data, refetch }
}
```

**使用：**
```tsx
function Component() {
  const { balance, refetch } = useFetchBalance()
  
  // 手动刷新
  const handleRefresh = () => {
    refetch(true)  // 刷新模拟账户余额
  }
  
  return (
    <div>
      <div>余额: {balance}</div>
      <button onClick={handleRefresh}>刷新</button>
    </div>
  )
}
```

### 示例 3：Mutation Hook

```tsx
// hooks/user/useHandleRealNameAuth.ts
import { handleRealNameAuth } from "@/service/rest/user"
import { useMutation } from "@tanstack/react-query"
import { toast } from "@/components/ui/toaster"

export function useHandleRealNameAuth() {
  const { mutate, isPending } = useMutation({
    mutationFn: handleRealNameAuth,
    onSuccess: () => {
      toast.success("实名认证成功")
    },
    onError: (error) => {
      toast.error(error?.message || "认证失败")
    },
  })

  return { mutate, isPending }
}
```

**使用：**
```tsx
function RealNameForm() {
  const { mutate, isPending } = useHandleRealNameAuth()
  
  const handleSubmit = (data: FormData) => {
    mutate(data)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={isPending}>
        {isPending ? "提交中..." : "提交"}
      </button>
    </form>
  )
}
```

---

## 🗄️ 状态管理示例

### 示例 1：基础 Store

```tsx
// stores/userStore.ts
import { create } from "zustand"
import { persist } from "zustand/middleware"
import { storageKeys } from "@/constants/storage"

interface UserState {
  token?: string
  userInfo?: UserInfo
}

interface UserActions {
  setToken: (v: string) => void
  setUserInfo: (v: UserInfo) => void
  reset: () => void
}

const initialState: UserState = {
  token: undefined,
  userInfo: undefined,
}

const useUserStore = create<UserState & UserActions>()(
  persist(
    (set) => ({
      ...initialState,
      setToken: (v) => set({ token: v }),
      setUserInfo: (v) => set({ userInfo: v }),
      reset: () => {
        set(initialState)
      },
    }),
    {
      name: storageKeys.userInfo,
      partialize: (state) => ({
        token: state.token,
        userInfo: state.userInfo,
      }),
    }
  )
)

export default useUserStore
```

### 示例 2：复杂 Store（带异步操作）

```tsx
// stores/marketStompStore.ts
import { create } from "zustand"
import { Client } from "@stomp/stompjs"

interface MarketStompState {
  stompClient: Client | undefined
  orderBookList: Record<string, OrderBook>
  subPriceProduct: Product | null
  setStompClient: (client: Client | undefined) => void
  setOrderBookList: (list: Record<string, OrderBook>) => void
  setSubPriceProduct: (product: Product | null) => void
}

export const useMarketStompStore = create<MarketStompState>((set) => ({
  stompClient: undefined,
  orderBookList: {},
  subPriceProduct: null,
  setStompClient: (client) => set({ stompClient: client }),
  setOrderBookList: (list) => set({ orderBookList: list }),
  setSubPriceProduct: (product) => set({ subPriceProduct: product }),
}))
```

### 示例 3：在组件中使用 Store

```tsx
// 方式 1：直接解构（简单场景）
function Component() {
  const { token, setToken } = useUserStore()
  return <div>{token}</div>
}

// 方式 2：选择器（性能优化）
function Component() {
  // 只订阅 token，userInfo 变化不会触发重渲染
  const token = useUserStore(state => state.token)
  const setToken = useUserStore(state => state.setToken)
  return <div>{token}</div>
}

// 方式 3：多个选择器
function Component() {
  const token = useUserStore(state => state.token)
  const userInfo = useUserStore(state => state.userInfo)
  const setUserInfo = useUserStore(state => state.setUserInfo)
  return <div>{userInfo?.name}</div>
}

// 方式 4：浅比较（对象/数组）
import { shallow } from 'zustand/shallow'

function Component() {
  const { userInfo, setUserInfo } = useUserStore(
    state => ({ 
      userInfo: state.userInfo, 
      setUserInfo: state.setUserInfo 
    }),
    shallow
  )
  return <div>{userInfo?.name}</div>
}
```

---

## 🌐 API 服务层示例

### 示例：REST API 封装

```tsx
// service/rest/user.ts
import { Result } from "@/types/service/service"
import { restClient } from "../index"
import { UserInfo } from "@/types/service/user-info"

// 1. GET 请求
export const getUserInfo = async () => {
  const result = await restClient.get<Result<UserInfo>>("/api/user/info")
  return result.data
}

// 2. POST 请求（JSON）
export const updatePwd = async (payload: { 
  oldPwd: string
  pwd: string 
}) => {
  const result = await restClient.post("/api/user/updatePwd", payload)
  return result.data
}

// 3. POST 请求（FormData - 文件上传）
export const updateInfo = async (payload: {
  userName: string
  avatarImg?: File
  oldAvatarImg?: string
}) => {
  const formData = new FormData()
  if (payload.avatarImg) {
    formData.append("avatarImg", payload.avatarImg)
  }

  const result = await restClient.post(
    `/api/user/updateInfo?userName=${payload.userName}&oldAvatarImg=${
      payload.oldAvatarImg ?? ""
    }`,
    formData
  )
  return result.data
}

// 4. POST 请求（查询参数）
export const updateInfoV2 = async (payload: {
  userName: string
  avatarImgUrl?: string
}) => {
  const result = await restClient.post(
    `/api/user/updateInfoV2?userName=${payload.userName}&avatarImgUrl=${
      payload.avatarImgUrl ?? ""
    }`
  )
  return result.data
}

// 5. GET 请求（查询参数）
export const getQueryParam = async (key: QueryParam) => {
  const result = await restClient.get(`/api/user/queryParam`, {
    params: { key },
  })
  return result.data
}
```

### Axios 实例配置

```tsx
// service/axiosInstance.ts
import axios from "axios"
import { errorHandler } from "./errorHandler"
import { storageKeys } from "@/constants/storage"

export const createAxiosInstance = (baseURL: string) => {
  const instance = axios.create({
    baseURL,
  })

  // 请求拦截器：添加 token
  instance.interceptors.request.use(
    (config) => {
      const token = getToken()
      if (token) {
        config.headers.authorization = token
      }
      return config
    },
    (err) => Promise.reject(err)
  )

  // 响应拦截器：统一处理错误
  instance.interceptors.response.use(
    (res) => {
      const code = res.data?.code
      if (code === -1) {
        return Promise.reject(res.data)
      }
      return res
    },
    (err) => {
      const status = err?.response?.status
      if (status === 401) {
        return errorHandler(err)  // 401 统一处理
      }
      return Promise.reject(err?.response?.data)
    }
  )

  return instance
}

function getToken() {
  try {
    const userInfo = localStorage?.getItem(storageKeys.userInfo)
    const token = userInfo ? JSON.parse(userInfo)?.state?.token : ""
    return token
  } catch (err) {
    console.log("err", err)
    return ""
  }
}
```

---

## 🔌 WebSocket 连接示例

### 完整 WebSocket Hook

```tsx
// hooks/marketStompClient/useConnectMarketStomp.ts
import { useMarketStompStore } from "@/stores/marketStompStore"
import { useEffect, useRef, useState } from "react"
import { Client } from "@stomp/stompjs"
import SockJS from "sockjs-client/dist/sockjs"
import { useOrderBookSubscribeRes } from "./useOrderBookSubscribeRes"
import { useSubscribeOrderBookType } from "./useSubscribeOrderBookType"
import useAppSettingStore from "@/stores/appSettingStore"
import { useSubSingleProduct } from "./useSubSingleProduct"
import useUserStore from "@/stores/userStore"
import { useHandleKlineSubscribeRes } from "./useHandleKlineSubscribeRes"
import { useHandlePriceSubscribeRes } from "./useHandlePriceSubscribeRes"

export function useConnectMarketStomp() {
  // 1. 使用 useRef 保存客户端引用
  const clientRef = useRef<Client | undefined>(undefined)
  const [didSubscribeType, setDidSubscribeType] = useState(false)
  const [didSubscribeSingle, setDidSubscribeSingle] = useState(false)
  
  // 2. 从 Store 获取状态
  const productType = useAppSettingStore((state) => state.productType)
  const orderBookList = useMarketStompStore((state) => state.orderBookList)
  const subPriceProduct = useMarketStompStore((state) => state.subPriceProduct)
  const token = useUserStore((s) => s.token)

  const setAppLoading = useAppSettingStore((state) => state.setAppLoading)
  const setStompClient = useMarketStompStore((state) => state.setStompClient)
  const handlePriceSubscribeResponse = useHandlePriceSubscribeRes()
  const handleOrderBookData = useOrderBookSubscribeRes()
  const subscribeOrderBookType = useSubscribeOrderBookType()
  const toSubSingleProduct = useSubSingleProduct()
  const handleKlineData = useHandleKlineSubscribeRes()

  // 3. 连接函数
  const connectWebSocket = () => {
    try {
      if (!token) {
        return
      }

      const headers = {
        Authorization: token,
      }
      
      // 启用 STOMP 内置心跳机制
      const connectHeaders = {
        ...headers,
        "heart-beat": "10000,10000",  // 10秒心跳
      }
      
      console.log("正在连接 WebSocket...")
      
      // 创建 STOMP 客户端
      const client = new Client({
        // 使用 SockJS 作为 WebSocket 传输
        webSocketFactory: () =>
          new SockJS(`${import.meta.env.VITE_API_URL}/market-ws`),
        connectHeaders,
        reconnectDelay: 5000,  // 重连延迟

        // 连接成功回调
        onConnect: function () {
          console.log("Market WebSocket 连接已建立")
          setStompClient(client)
          setDidSubscribeType(false)
          setDidSubscribeSingle(false)

          // 订阅各种主题
          client?.subscribe("/user/topic/price/", handlePriceSubscribeResponse)
          client?.subscribe("/user/topic/orderBook/", handleOrderBookData)
          client?.subscribe("/user/topic/kline/", handleKlineData)

          // 连接成功后隐藏全局 loading
          setAppLoading(false)
        },

        // 连接错误回调
        onStompError: function (frame) {
          console.log(`Market WebSocket 连接错误: ${frame.headers["message"]}`)
        },

        // 连接断开回调
        onDisconnect: function () {
          console.log("Market WebSocket 连接已断开")
        },

        onWebSocketClose: function (event) {
          console.warn("Market WebSocket 被关闭:", event)
        },
      })

      // 激活连接
      clientRef.current = client
      client.activate()
    } catch (e) {
      console.error("WebSocket 连接错误:", e)
      // 尝试重连
      setTimeout(connectWebSocket, 5000)
    }
  }

  // 4. 连接成功后订阅盘口类型
  useEffect(() => {
    if (!didSubscribeType && clientRef.current?.connected) {
      subscribeOrderBookType({
        type: productType,
      })
      setDidSubscribeType(true)
    }
  }, [didSubscribeType, clientRef.current?.connected])

  // 5. 连接成功后订阅单个产品
  useEffect(() => {
    const productCodes = Object.keys(orderBookList)
    if (
      !didSubscribeSingle &&
      clientRef.current?.connected &&
      productCodes.length
    ) {
      const code = subPriceProduct
        ? subPriceProduct.productCode
        : productCodes?.[0]
      if (code) {
        toSubSingleProduct(code)
        setDidSubscribeSingle(true)
      }
    }
  }, [
    didSubscribeSingle,
    orderBookList,
    subPriceProduct,
    clientRef.current?.connected,
  ])

  // 6. 生命周期管理
  useEffect(() => {
    connectWebSocket()
    return () => {
      clientRef.current?.deactivate()  // 组件卸载时断开连接
    }
  }, [])
}
```

**使用：**
```tsx
// App.tsx
function App() {
  // 在根组件中连接 WebSocket
  useConnectMarketStomp()
  
  return <Outlet />
}
```

---

## 📝 表单处理示例

### 使用 Formik + Yup

```tsx
import { useFormik } from "formik"
import * as Yup from "yup"
import { useMutation } from "@tanstack/react-query"

const validationSchema = Yup.object({
  userName: Yup.string()
    .required("用户名不能为空")
    .min(3, "用户名至少3个字符"),
  email: Yup.string()
    .email("邮箱格式不正确")
    .required("邮箱不能为空"),
  password: Yup.string()
    .required("密码不能为空")
    .min(6, "密码至少6个字符"),
})

function RegisterForm() {
  const { mutate, isPending } = useMutation({
    mutationFn: register,
    onSuccess: () => {
      toast.success("注册成功")
    },
  })

  const formik = useFormik({
    initialValues: {
      userName: "",
      email: "",
      password: "",
    },
    validationSchema,
    onSubmit: (values) => {
      mutate(values)
    },
  })

  return (
    <form onSubmit={formik.handleSubmit}>
      <div>
        <input
          name="userName"
          value={formik.values.userName}
          onChange={formik.handleChange}
          onBlur={formik.handleBlur}
        />
        {formik.touched.userName && formik.errors.userName && (
          <div>{formik.errors.userName}</div>
        )}
      </div>
      
      <div>
        <input
          name="email"
          type="email"
          value={formik.values.email}
          onChange={formik.handleChange}
          onBlur={formik.handleBlur}
        />
        {formik.touched.email && formik.errors.email && (
          <div>{formik.errors.email}</div>
        )}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={formik.values.password}
          onChange={formik.handleChange}
          onBlur={formik.handleBlur}
        />
        {formik.touched.password && formik.errors.password && (
          <div>{formik.errors.password}</div>
        )}
      </div>
      
      <button type="submit" disabled={isPending}>
        {isPending ? "提交中..." : "注册"}
      </button>
    </form>
  )
}
```

---

## 🔀 条件渲染模式

### 模式 1：简单条件

```tsx
// 显示/隐藏
{isShow && <Component />}

// 二选一
{isLoading ? <Loading /> : <Content />}

// 多条件
{mode === 'edit' && <EditForm />}
{mode === 'view' && <ViewForm />}
```

### 模式 2：复杂条件

```tsx
function Component() {
  const mode = useStore(state => state.mode)
  
  const renderContent = () => {
    switch (mode) {
      case TradeListMode.Alive:
        return <AliveTradeAndFutureTable />
      case TradeListMode.History:
        return <HistoryTradeAndFutureTable />
      default:
        return null
    }
  }
  
  return (
    <div>
      {renderContent()}
    </div>
  )
}
```

### 模式 3：列表条件渲染

```tsx
function ProductList() {
  const { data: products, isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  })
  
  if (isLoading) return <Loading />
  if (!products?.length) return <Empty />
  
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

---

## ⚡ 性能优化示例

### 示例 1：React.memo

```tsx
// 防止不必要的重渲染
const ProductCard = React.memo(({ product }: ProductCardProps) => {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}</p>
    </div>
  )
}, (prevProps, nextProps) => {
  // 自定义比较：只有 id 变化才重渲染
  return prevProps.product.id === nextProps.product.id
})
```

### 示例 2：useMemo

```tsx
function Component({ items, filter }) {
  // 缓存计算结果
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter)
  }, [items, filter])
  
  return (
    <div>
      {filteredItems.map(item => (
        <Item key={item.id} item={item} />
      ))}
    </div>
  )
}
```

### 示例 3：useCallback

```tsx
function Parent() {
  const [count, setCount] = useState(0)
  
  // 缓存函数引用，避免子组件不必要的重渲染
  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child onClick={handleClick} />
    </div>
  )
}

const Child = React.memo(({ onClick }: { onClick: () => void }) => {
  return <button onClick={onClick}>Child</button>
})
```

### 示例 4：Zustand 选择器优化

```tsx
// ❌ 错误：每次都会创建新对象
function Component() {
  const { user, setUser } = useUserStore()
  return <div>{user.name}</div>
}

// ✅ 正确：使用选择器
function Component() {
  const user = useUserStore(state => state.user)
  const setUser = useUserStore(state => state.setUser)
  return <div>{user.name}</div>
}

// ✅ 或者使用 shallow
import { shallow } from 'zustand/shallow'

function Component() {
  const { user, setUser } = useUserStore(
    state => ({ user: state.user, setUser: state.setUser }),
    shallow
  )
  return <div>{user.name}</div>
}
```

---

## 📚 总结

### 代码组织原则

1. **单一职责**：每个 Hook、组件、Store 只负责一件事
2. **关注点分离**：UI、业务逻辑、数据获取分离
3. **可复用性**：提取公共逻辑到 Hooks
4. **类型安全**：充分利用 TypeScript

### 常见模式

- **数据获取**：React Query + 自定义 Hook
- **状态管理**：Zustand + 选择器优化
- **表单处理**：Formik + Yup
- **实时通信**：WebSocket + STOMP
- **错误处理**：Error Boundary + 统一错误处理

### 性能优化要点

- 使用 `React.memo` 防止不必要的重渲染
- 使用 `useMemo` 和 `useCallback` 缓存计算结果和函数
- 使用 Zustand 选择器避免不必要的订阅
- 路由懒加载减少初始包大小

---

**这些示例都是项目中的真实代码模式，可以直接参考使用！** 🎯

