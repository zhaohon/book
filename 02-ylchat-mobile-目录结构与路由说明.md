## 1. 顶层目录总览（只看关键的）

`ylchat-mobile` 根目录的关键内容大致如下（省略无关文件）：

```text
ylchat-mobile/
  app/                  # expo-router 路由页面（最重要）
  assets/               # 图片、字体等静态资源
  components/           # 组件（UI + 业务）
  config/               # 配置相关
  constants/            # 各种常量
  hooks/                # 自定义 hooks（业务逻辑）
  libs/                 # 内嵌子库，比如手机号选择器等
  service/              # 接口请求封装
  stores/               # 全局状态（zustand 等）
  types/                # TypeScript 类型
  utils/                # 工具函数
  public/               # 静态资源（主要是多语言 JSON）
  i18n.js               # i18next 初始化
  metro.config.js       # RN 打包配置，含路径别名等
  tailwind.config.js    # Tailwind / NativeWind 样式配置
  app.json              # Expo app 配置
  package.json          # 依赖与脚本
  tsconfig.json         # TypeScript 配置
```

对刚上手的你，只需要记住一个“口诀”：

> **页面进 `app`，UI 找 `components`，逻辑找 `hooks + service + stores`，类型/常量去 `types` 和 `constants`。**

---

## 2. expo-router 基础概念

项目使用的是 `expo-router`，并且在 `package.json` 中配置了：

```json
"main": "expo-router/entry"
```

**核心思想：**

- `app/` 目录 = Web 里的 `pages/` 目录
- 文件 = 路由
  - `app/index.tsx` → `/`
  - `app/profile/index.tsx` → `/profile`
  - `app/profile/info.tsx` → `/profile/info`
- `_layout.tsx` = 布局组件，包裹同级子页面
- `()` 包裹的目录是“分组”，不会出现在 URL 中，只是用来归类文件
  - 例如：`app/(tabs)/index.tsx` 实际路径仍是 `/`（首页）

**大致规则示例：**

```text
app/
  _layout.tsx           # 全局布局
  index.tsx             # /  根路由

  profile/
    index.tsx           # /profile
    info.tsx            # /profile/info

  (auth)/
    _layout.tsx         # 登录模块布局
    index.tsx           # /(auth) -> 实际路径通常简化为 /login（看 Router 用法）

  (tabs)/
    _layout.tsx         # 底部 Tab 布局
    index.tsx           # Tab: 首页
    position.tsx        # Tab: 持仓
    finance.tsx         # Tab: 资金
    news.tsx            # Tab: 新闻
    profile.tsx         # Tab: 个人中心入口
```

> 学习建议：先把 expo-router 理解为“一个通过目录结构来生成路由的库”，然后在 `app/` 里随便点几页，观察“文件路径 -> 实际路径”的对应关系。

---

## 3. app/ 目录结构详解（路由与模块）

当前 `app/` 目录（简化版）：

```text
app/
  _layout.tsx
  +html.tsx
  +not-found.tsx

  (auth)/
    _layout.tsx
    index.tsx
    forget-password.tsx
    register.tsx

  (home)/
    ai-trade/
      ai-list.tsx
      ai-order.tsx
      ai-purchase.tsx
    credit-loan/
      borrow.tsx
      breach.tsx
      history.tsx
      index.tsx
      loan-agreement.tsx
      loan-requirements.tsx
      repayment.tsx
    product-detail.tsx
    quotes-list.tsx
    trade/
      future.tsx
      open-trade.tsx
      trade-result.tsx

  (tabs)/
    _layout.tsx
    finance.tsx
    index.tsx
    news.tsx
    position.tsx
    profile.tsx

  profile/
    change-password.tsx
    info.tsx
    real-name.tsx
    referral-code.tsx
    service.tsx
    finance/
      deposit.tsx
      record.tsx
      withdraw.tsx
    setting/
      index.tsx
      language.tsx
      privacy.tsx
      statement.tsx
    wallet/
      add-bankcard.tsx
      add-wallet.tsx
      index.tsx

  trader/
    _layout.tsx
    index.tsx
    mine.tsx
```

下面按模块逐一解释。

### 3.1 根级 `_layout.tsx` —— 全局外壳

- 作用：提供**全局布局**和**全局 Provider**（例如主题、React Query、SafeArea、全局 Toast 等）。
- 所有页面都会被它包住，相当于 Web 项目中的 `App.tsx` 或 `RootLayout`。
- 一般结构会是：
  - 包一层 `Stack`，配置全局的 `screenOptions`
  - 把 `PageContainer` / `AppLayout` 之类的通用布局组件放到合适的位置

使用技巧：

- 如果你想要“每个页面顶部都有统一导航栏/背景色”，就来这里改。
- 如果你要为全局引入一个 Provider（比如新增一个全局 Store），也应该在这里改。

### 3.2 `(tabs)/_layout.tsx` —— 底部 Tab 栏（非常重要）

`app/(tabs)/_layout.tsx` 的关键代码逻辑如下（略去注释）：

```tsx
import { Tabs, TabList, TabTrigger, TabSlot } from "expo-router/ui";
import { useTranslation } from "react-i18next";
import { useSafeAreaInsets } from "react-native-safe-area-context";
import { TAB_HEIGHT, TAB_PADDING_BOTTOM } from "@/constants/Common";
import { Href } from "expo-router";
import { TabButton } from "@/components/app/Tabs/TabButton";
import { isSafariPwaFullscreen } from "@/utils/utils";

export default function TabLayout() {
  const { t } = useTranslation();
  const insets = useSafeAreaInsets();
  const needAddExtraBottom = isSafariPwaFullscreen();

  const tabList = [
    {
      name: "index",
      href: "/",
      title: t("Dashboard", "Dashboard"),
      icon: require("@/assets/images/tab/home.png"),
      greenIcon: require("@/assets/images/tab/home-green.png"),
      redIcon: require("@/assets/images/tab/home-red.png"),
    },
    // ... 省略 position / finance / news
    {
      name: "profile",
      href: "/profile",
      title: t("Account", "Account"),
      icon: require("@/assets/images/tab/profile.png"),
      greenIcon: require("@/assets/images/tab/profile-green.png"),
      redIcon: require("@/assets/images/tab/profile-red.png"),
    },
  ];

  return (
    <Tabs>
      <TabSlot
        style={{ paddingBottom: needAddExtraBottom ? TAB_PADDING_BOTTOM : 0 }}
      />
      <TabList
        className="bg-f1 fixed bottom-0 left-0 w-full"
        style={{
          height:
            TAB_HEIGHT +
            (needAddExtraBottom ? TAB_PADDING_BOTTOM : insets.bottom),
        }}
      >
        {tabList.map((item) => (
          <TabTrigger
            asChild
            key={item.name}
            name={item.name}
            href={item.href as Href}
            className="flex-1"
          >
            <TabButton key={item.name} {...item} />
          </TabTrigger>
        ))}
      </TabList>
    </Tabs>
  );
}
```

**要点总结：**

- `Tabs` / `TabList` / `TabTrigger` / `TabSlot` 全部来自 `expo-router/ui`
  - `TabSlot`：上方展示页面内容的区域
  - `TabList`：底部 Tab 栏容器
  - `TabTrigger`：单个 Tab 项
- `tabList` 定义了 5 个 Tab：
  - name: `"index"` / `"position"` / `"finance"` / `"news"` / `"profile"`
  - href: 分别对应 `/` / `/position` / `/finance` / `/news` / `/profile`
  - title + 图标：按钮文案和图标
- 真正绘制 Tab 按钮的是 `components/app/Tabs/TabButton.tsx`

**新增一个 Tab 的步骤：**

1. 在 `app/(tabs)/` 下新增一个页面文件，比如 `market.tsx`
2. 在 `tabList` 中新增一项：

   ```ts
   {
     name: "market",
     href: "/market",
     title: t("Market", "Market"),
     icon: require("@/assets/images/tab/market.png"),
     greenIcon: require("@/assets/images/tab/market-green.png"),
     redIcon: require("@/assets/images/tab/market-red.png"),
   }
   ```

3. 准备好对应图标放入 `assets/images/tab` 下

完成这三步，你就会多出一个完整的新 Tab。

### 3.3 `(tabs)/` 下的各个页面

- `app/(tabs)/index.tsx` → Dashboard / Home 页（对应 Tab「Dashboard」）
- `app/(tabs)/position.tsx` → 持仓页（Tab「Portfolio」）
- `app/(tabs)/finance.tsx` → 收益/表现页（Tab「Performance」）
- `app/(tabs)/news.tsx` → 新闻列表（Tab「News」）
- `app/(tabs)/profile.tsx` → 个人中心入口页（Tab「Account」）

常见模式：

- 页面最外层用 `PageSafeAreaView` / `TabPageWrapper` 之类组件包裹
- 内部通过各种 `components/home`、`components/position` 等组件拼装 UI
- 通过 `hooks/` 里的 Hook 请求数据

### 3.4 `(auth)/` 模块 —— 登录注册相关

路径：`app/(auth)/`

- `_layout.tsx`：登录模块的专用布局，通常负责：
  - 设置状态栏颜色
  - 登录页背景、Logo 等
- `index.tsx`：登录页
- `forget-password.tsx`：忘记密码
- `register.tsx`：注册

常用组件：

- `components/auth/AccountInput.tsx`
- `components/auth/PhoneNumberInput.tsx`
- `components/auth/AuthHeader.tsx`
- `components/auth/SelectLoginType.tsx`

常用 hooks（在 `hooks/auth/` 下）：

- `useLogin`
- `useRegister`
- `useResetPwd`
- `useSendSms`
- `useRememberPassword`

你只要记住：**所有认证相关的页面都在 `(auth)` 里，逻辑都在 `hooks/auth/` 里**。

### 3.5 `(home)/` 模块 —— 行情/产品/借贷等二级页面

路径：`app/(home)/`

包含：

- `ai-trade/`：AI 交易相关
  - `ai-list.tsx`：AI 产品列表
  - `ai-order.tsx`：AI 订单列表
  - `ai-purchase.tsx`：下单/购买页
- `credit-loan/`：信用借贷
  - `borrow.tsx`：借款页
  - `breach.tsx`：违约相关
  - `history.tsx`：借贷历史
  - `loan-agreement.tsx`：协议文本
  - `loan-requirements.tsx`：借贷条件
  - `repayment.tsx`：还款
- `product-detail.tsx`：某个交易品种的详情
- `quotes-list.tsx`：行情列表
- `trade/`
  - `future.tsx`：某种期货模式
  - `open-trade.tsx`：开仓页
  - `trade-result.tsx`：成交结果页

这部分可以理解为「从首页/列表点击进去」的二级/三级业务页面。

### 3.6 `profile/` 模块 —— 个人中心

路径：`app/profile/`

主要内容：

- 基础信息：
  - `info.tsx`：个人信息主页
  - `change-password.tsx`：修改密码
  - `real-name.tsx`：实名认证
  - `referral-code.tsx`：邀请码
  - `service.tsx`：在线客服/服务
- 设置相关（子目录 `setting/`）：
  - `index.tsx`：设置主页
  - `language.tsx`：切换语言
  - `privacy.tsx`：隐私协议
  - `statement.tsx`：相关声明
- 资金/财务（子目录 `finance/`）：
  - `deposit.tsx`：充值
  - `withdraw.tsx`：提现
  - `record.tsx`：资金记录
- 钱包/银行卡（子目录 `wallet/`）：
  - `index.tsx`：钱包列表
  - `add-bankcard.tsx`：新增银行卡
  - `add-wallet.tsx`：新增钱包地址

常用组件位置：

- `components/profile/Avatar.tsx`：头像
- `components/profile/ProfileMenu.tsx`：个人中心菜单
- `components/profile/wallet/*`：钱包相关 UI
- `components/profile/finance/*`：资金相关 UI

常用 hooks：

- `hooks/user/*`：用户信息/实名认证/修改资料
- `hooks/wallet/*`：钱包/币种列表
- `hooks/bankcard/*`：银行卡增删改查
- `hooks/fund/*`：余额/资金记录

---

## 4. components/ 目录结构与使用建议

`components/` 大致可以分为三层，越往下越偏业务：

```text
components/
  ui/          # 通用 UI（Button/Input/Card/...）
  app/         # 应用级组件（布局、Header、Toast 等）
  ...业务域/
    home/
    finance/
    position/
    profile/
    loan/
    news/
    trader/
```

### 4.1 `components/ui/` —— 可复用的基础 UI

示例：

- `Button/AppButton.tsx`
- `Input/InputWithTitle.tsx`
- `Input/NumberInputWithTitle.tsx`
- `Card/GrayCard.tsx`
- `Tabs/Tabs.tsx`
- `Empty/Empty.tsx`
- `Loading/Loading.tsx`

使用建议：

- 开发任何业务页面时**优先来这里找**有没有合适的基础组件
- 如果只是样式/文案有小差异，可以通过 props 或 `className` 定制，而不是复制粘贴

### 4.2 `components/app/` —— 应用级组件

示例：

- 布局容器：
  - `AppLayout.tsx`
  - `PageContainer.tsx`
  - `SafeAreaView/PageSafeAreaView.tsx`
  - `SafeAreaView/TabPageWrapper.tsx`
- 头部与导航：
  - `AppHeader.tsx`
  - `PageRightTop.tsx`
- 通用功能：
  - `AppToast.tsx`：全局消息提示
  - `FallbackRender.tsx`：错误边界
  - `ScrollToTop.tsx`、`ScrollView/AppScrollView.tsx` 等
- Tab 相关：
  - `Tabs/TabButton.tsx`

这些组件可以理解为「不直接带业务含义，但服务于全局体验」的一层。

### 4.3 业务域组件示例

- `components/home/*`：首页/行情相关（Banner、产品卡、K 线展示）
- `components/position/*`：持仓列表、平仓弹窗、止盈止损设置
- `components/finance/*`：资金记录、收益卡片、筛选日期等
- `components/profile/*`：个人信息卡、下载提示、钱包列表、银行卡列表
- `components/loan/*`：借贷流程的步骤条/提交按钮

用法模式：

1. 页面 -> 调用业务域组件
2. 业务域组件内部 -> 调用标准 UI（`components/ui`）
3. 业务逻辑 -> 调用 `hooks/*`

---

## 5. hooks/ 目录结构与职责

`hooks/` 采用**按业务域划分**的方式：

```text
hooks/
  auth/                # 登录注册/验证码
  user/                # 用户信息、实名认证、切换实盘/模拟等
  order/               # 下单、撤单、订单列表
  orderStompClient/    # 订单相关 WebSocket
  marketStompClient/   # 行情相关 WebSocket
  finance/             # 盈亏计算、资金汇总
  fund/                # 余额、资金记录
  product/             # 产品配置、到期时间配置、自选等
  loan/                # 借贷配置、借贷订单
  recharge/            # 充值相关
  wallet/              # 钱包列表、钱包管理
  bankcard/            # 银行卡列表、管理
  news/                # 新闻列表
  action/ / expiry-order/ 等 # 具体业务动作用的 Hook
  router/              # 导航相关封装，例如 useAppRouter
  common/              # 通用逻辑：上传文件、获取配色、自动隐藏 Tab 等
  useThemeColor.ts     # 主题颜色 Hook
  useColorScheme(.web).ts
```

典型模式（以用户信息为例）：

```ts
// hooks/user/useGetUserInfo.ts
import { useQuery } from "@tanstack/react-query";
import { getUserInfo } from "@/service/userService";

export function useGetUserInfo() {
  return useQuery({
    queryKey: ["userInfo"],
    queryFn: getUserInfo,
  });
}
```

在页面中使用：

```tsx
import { useGetUserInfo } from "@/hooks/user/useGetUserInfo";

const { data: user, isLoading, error } = useGetUserInfo();
```

> 记忆：**页面不直接调用 axios，而是调用 hooks；hooks 再去调用 `service` 并配合 `react-query` 管理数据。**

---

## 6. service/、stores/、constants/、types/、utils/

### 6.1 service/ —— 接口层

职责：

- 定义各种业务接口请求函数
- 一般基于 `axios` 封装
- 与接口文档/后端约定最强相关

典型形态类似：

```ts
// service/userService.ts （示意）
import request from "@/service/request";

export function getUserInfo() {
  return request.get("/user/info");
}
```

### 6.2 stores/ —— 全局状态层（Zustand）

职责：

- 存储全局共享状态，如：
  - 用户信息
  - 当前选中的交易品种
  - 主题颜色/模式
- 对外暴露 `useXXXStore` Hook

示例（伪代码）：

```ts
import { create } from "zustand";

interface UserState {
  user: User | null;
  setUser: (user: User | null) => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

### 6.3 constants/、types/、utils/

- `constants/*`：
  - 各种枚举：订单状态、方向（多/空）、颜色常量、时间区间等
  - 一些项目级别配置（如 `TAB_HEIGHT`、Padding 等）
- `types/*`：
  - 单独定义接口响应类型、业务实体类型，尽量减少在业务代码里写“any”
- `utils/*`：
  - 全是纯函数工具：
    - 金额格式化
    - 时间格式化
    - K 线数据处理
    - 是否在 App 内打开（PWA、浏览器等判断）

当你不确定某个“通用功能”在哪实现时，优先搜：

1. `utils/` 是否有工具函数
2. `hooks/common/` 是否有对应 Hook

---

## 7. Router 使用习惯（导航与路径管理）

### 7.1 直接使用 expo-router 的方式

最基础的导航方式是使用 `expo-router` 提供的 `useRouter`：

```tsx
import { useRouter } from "expo-router";

const router = useRouter();

router.push("/profile/info");
router.replace("/(auth)");
router.back();
```

这种写法简单直接，但字符串容易散落在各处，不利于统一维护。

### 7.2 项目自带的 `useAppRouter`

在 `hooks/router/useAppRouter.ts` 中，通常会封装一层：

```ts
// hooks/router/useAppRouter.ts （示意代码）
import { useRouter } from "expo-router";

export function useAppRouter() {
  const router = useRouter();

  return {
    toLogin() {
      router.replace("/(auth)");
    },
    toProfileInfo() {
      router.push("/profile/info");
    },
    toHome() {
      router.replace("/(tabs)/index");
    },
    // ...更多高层封装
  };
}
```

页面里这样用：

```tsx
import { useAppRouter } from "@/hooks/router/useAppRouter";

const { toProfileInfo } = useAppRouter();

<Button onPress={toProfileInfo} />;
```

优势：

- 所有路由路径集中在一个地方管理
- 如果改了实际路径，只需要在 `useAppRouter` 里改一处

**建议：写业务逻辑时优先使用 `useAppRouter`，除非是非常临时的跳转。**

---

## 8. 作为新手你可以怎样“逛”这个项目

给你一条推荐路线，你可以照着点文件：

1. **从 Tab 布局开始**
   - 看：`app/(tabs)/_layout.tsx`
   - 理解：每个 Tab 对应的 `name` / `href` 分别指向哪里
2. **看首页**
   - 打开：`app/(tabs)/index.tsx`
   - 看它引入了哪些组件（通常在 `components/home` 和 `components/app` 下面）
3. **看一个简单的详情页**
   - 比如：`app/profile/info.tsx`
   - 观察它是怎么：
     - 使用 `PageSafeAreaView` 包裹布局
     - 使用 `hooks/user/useGetUserInfo` 获取数据
     - 使用 `components/profile/UserInfo` 渲染内容
4. **跳到 hooks 层**
   - 打开：`hooks/user/useGetUserInfo.ts`
   - 再看：`service/userService.ts`
   - 明白：页面并不直接管“请求怎么发”，它只关心“怎么展示数据”
5. **随便改一点东西**
   - 比如把 `info.tsx` 页面某个标题的文案改一改
   - 刷新 App / Web 看效果

你多刷几遍这个路线，大脑里自然会形成这张“地图”：

- **路由和页面** → `app/`
- **UI 组件** → `components/`
- **数据 & 逻辑** → `hooks/` + `service/` + `stores/`
- **类型 & 配置** → `types/` + `constants/` + `utils/`

---

## 9. 小结：RN 项目 vs 你熟悉的 React Web 项目

用你已经熟悉的 React Web 思维来类比：

- Web 的 `src/pages/*` → 这里的 `app/*`
- Web 的 `src/components/*` → 这里的 `components/*`
- Web 中的 `useXXX` Hook → 这里大量的 `hooks/*`
- Web 中的 `services/api.ts` → 这里的 `service/*`
- Web 中的 `store`（Redux/Mobx/Pinia）→ 这里的 `stores/*`（Zustand）

不同点主要有三处：

1. **基础组件不一样**
   - DOM：`div` / `span` / `img`
   - RN：`View` / `Text` / `Image` / `ScrollView` 等
2. **样式写法不一样**
   - Web：`className="..."` + CSS/SCSS/Tailwind
   - RN：`style={{}}` + 这里集成的 NativeWind/Tailwind，仍然可以用 `className="px-4"` 这种写法
3. **路由不一样**
   - Web：`react-router` / `next/router`
   - RN：`expo-router`（文件系统路由），更接近 Next.js

如果你在 Web 项目里已经习惯了“组件 + hooks + service”的结构，那么这个 RN 项目只是在“运行环境和 UI 部分”变了一下，本质上非常相似。


