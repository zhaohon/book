# 📖 @ylchat-mobile 项目源码全流程深度解读 (Project Walkthrough)

> **目标**: 这份文档将带你从 APP 启动开始，像我们在结对编程一样，逐个页面、逐行代码地走一遍整个核心业务流程。
> **约定**: 遇到 **RN 知识点**，我会用 `💡` 标记。遇到 **业务逻辑**，我会用 `🛒` 标记。

---

## 1. 启动与入口 (Bootstrapping)

一切的开始并不是 `App.tsx`，而是 `app/_layout.tsx`。

### 📁 文件: `app/_layout.tsx` (Root Component)

这是整个 App 的**外壳**，无论你跳转到哪个页面，这个外壳始终存在。

```typescript
// app/_layout.tsx 核心逻辑简化

export default function RootLayout() {
  // 1. 状态恢复 (Hydration)
  // 💡 RN 没有 LocalStorage，通常用 AsyncStorage。Zustand 的 persist 中间件是异步的。
  // 我们必须等待 store 从手机硬盘里读完数据，才能显示 UI，否则用户 login 状态会跳变。
  const [userHydrated] = useState(useUserStore.persist?.hasHydrated());

  // 2. 加载字体
  // 💡 RN 没法像 Web 那样 <link> 引入字体，必须用 useFonts 加载本地 .ttf 文件。
  const [loaded] = useFonts({ ... });

  // 3. 只有当“状态恢复” + “字体加载” + “多语言” 全部 OK，才隐藏启动屏
  useEffect(() => {
    if (loaded && ready && i18nextReady) {
      SplashScreen.hideAsync(); // 💡 隐藏 Launch Screen
    }
  }, [...]);

  // 如果没准备好，就一直显示 Loading (其实用户看到的是启动图)
  if (!loaded) return <View>Loading...</View>;

  return (
    // 💡 全局 Provider 包裹
    <QueryClientProvider client={queryClient}> {/* API 缓存 */}
       <AppLayout /> {/* 真正的路由出口，Expo Router 会在这里渲染 app/index.tsx 等页面 */}
       <AppToast />  {/* 全局 Toast 组件，必须放在最顶层才能覆盖所有内容 */}
    </QueryClientProvider>
  );
}
```

---

## 2. 注册流程 (Registration Flow)

如果检测到没有 Token，`useAppRouter` 会把我们踢到注册或登录页。先看注册。

### 📁 文件: `app/(auth)/register.tsx`

#### 2.1 页面结构与 Formik

这个页面是一个长的滚动视图 (`AppScrollView`)，包含了一个表单。

- **💡 AppScrollView**:
  这个组件封装了 `KeyboardAvoidingView`。在手机上，**输入框如果在屏幕底部，键盘弹起会遮住它**。这个组件会自动把页面往上顶。Web 开发从来不用操心这个，但在 RN 这是必修课。

- **💡 Formik + Yup**:
  这里没有用 `const [email, setEmail] = useState('')`。因为表单太复杂了，有校验、有脏检查（Dirty Check）、有提交状态。

  ```typescript
  // 定义校验规则（Schema）
  const validationSchema = Yup.object({
     // 邮箱校验：如果是邮箱注册模式，则必填，且格式必须是 email
     email: Yup.string().when("type", { ... }),
     // 验证码校验：必须是 6 位数字
     verificationCode: Yup.string().length(6, "必须6位"),
  });

  // 渲染
  <Formik onSubmit={onSubmit} ...>
    {(formik) => (
       // 双向绑定：RN 没有 v-model
       // onChangeText 类似 Web 的 onInput
       <InputWithTitle
          value={formik.values.verificationCode}
          onChangeText={formik.handleChange("verificationCode")}
       />
    )}
  </Formik>
  ```

#### 2.2 业务逻辑：环境判断与邀请码

```typescript
// 读取环境变量
const remarkEnable = process.env.EXPO_PUBLIC_REGISTER_REMARK_ENABLE === "1";

// 🛒 业务：如果是 Web 版且是套壳环境，可能需要隐藏某些输入框
// 这里实际上是根据 params 自动填入邀请码
initialValues={{
  inviteCode: (params.code as string) || "",
}}
```

---

## 3. 登录流程 (Login Flow)

注册成功后，跳转回登录页。

### 📁 文件: `app/(auth)/index.tsx`

#### 3.1 跨端黑科技：Web 套壳通信

你会在代码里看到一段非常诡异的 `window.addEventListener("message")` 逻辑。

```typescript
// app/(auth)/index.tsx

// 🛒 场景：这个 App 既是 Native App，也是一个 H5 网页 (Web)。
// 当它作为 H5 被嵌入别人的 App (Wrapper) 时，需要从宿主 App 获取账号密码自动填充。
useEffect(() => {
  if (Platform.OS !== "web") return; // 💡 Platform.OS 控制平台差异代码

  const handler = (event) => {
    // 收到宿主发来的账号密码
    if (event.data.topic === MessageToWrapperTopic.ReplyAccount) {
      setAccount(event.data.value); // 自动填入 Store
    }
  };
  window.addEventListener("message", handler);
}, []);
```

**注意**: 这段代码在真机 App 上永远不会执行，是专门为了 H5 打包准备的。

#### 3.2 登录提交

当用户点击“Start”按钮：

1.  触发 `onSubmit`。
2.  调用 `useLogin` hook (在 `hooks/auth/useLogin.ts`)。
3.  **成功后 (`onSuccess`)**:
    - `setToken(...)`: 把 Token 存入 Zustand Store（并自动持久化到 Async Storage）。
    - `remember(params)`: 如果用户勾选了“记住密码”，把账号密码存到另一个 Store。
    - **跳转**: 此时 `useUserStore` 里的 Token 变了，App 可能会自动重定向，或者需要手动 `router.push('/')`。

---

## 4. 核心主页 (Dashboard)

登录进来就是 Home Tab。

### 📁 文件: `app/(tabs)/index.tsx`

#### 4.1 页面布局与 TabPageScrollView

```typescript
<TabPageScrollView>
  {/* 💡 为什么又是这一层？ */}
  {/* Tab 页比较特殊，它的顶部可能有 Header，底部有 TabBar。*/}
  {/* 这个组件会自动计算 paddingTop 和 paddingBottom，防止内容被遮挡。*/}

  <View className="py-[40px] px-[20px] gap-[24px]">
    <UserInfo /> {/* 顶部: 你好, VIP1 */}
    <Balance /> {/* 资产卡片: $10,000.00 */}
    <Quotes /> {/* 🛒 核心: 实时行情列表 */}
  </View>
</TabPageScrollView>
```

#### 4.2 核心业务：行情订阅 (`useFocusEffect`)

这是本页最复杂的逻辑。我们不希望用户不在这个页面时还一直建立着高频的 Socket 连接。

```typescript
useFocusEffect(
  useCallback(() => {
    // 1. 只有 Socket 连接成功才干活
    if (!marketClient?.connected) return;

    // 🛒 2. 发送订阅指令：我要看 "热门搜索" (HOTSEARCH) 的币种价格
    sendOrderBookSubscriptionByType(ProductType.HOTSEARCH);

    // 3. 离开页面（切换 Tab）时，取消订阅
    return () => {
      unsubscribeOrderBookType();
    };
  }, [marketClient?.connected])
);
```

#### 4.3 核心组件：`Quotes.tsx`

打开 `components/home/Quotes/Quotes.tsx`。

- 它从 `useMarketStompStore` 获取 `orderBookList` (行情数据)。
- **渲染逻辑**:
  - 前 3 个币种 -> `<AppScrollView horizontal>` -> **横向滚动卡片**。
  - 剩下的币种 -> 垂直列表 -> **普通列表**。
- **💡 性能点**: 这里使用了 store 的订阅机制，只有当价格变动时，`ProductCard` 才会重绘。

---

## 5. 其他 Tab 全解读

### 5.1 业绩页 (Finance)

### 📁 文件: `app/(tabs)/finance.tsx`

这个页面是看你的历史订单盈亏的。

- **逻辑**:
  1.  `useGetProfitSummary`: 调用 HTTP 接口获取昨天的盈利、今天的盈利。
  2.  `useFocusEffect` -> `refetch()`: 每次进页面都刷新一下数据。
- **条件渲染**:
  ```typescript
  // 🛒 根据你是在看“合约”还是“交割”，渲染不同的列表组件
  {
    isContract ? <Contract /> : <Future />;
  }
  ```
  - 这个 `isContract` 状态来自于 `useAppSettingStore`，在 Header 上有个 Switch 开关切换的。

### 5.2 资讯页 (News)

### 📁 文件: `app/(tabs)/news.tsx`

- **列表组件**: 这里用了 `<TabPageFlatList>`。
- **💡 FlatList vs ScrollView**:
  - News 可能有 1000 条。如果不以前只用 `ScrollView` 渲染，手机内存会炸。
  - `FlatList` (虚拟列表) 只渲染屏幕里看到的 10 条，滑出去的会被回收。
  - `onEndReached`: 触底加载下一页（分页加载）。

### 5.3 个人中心 (Profile)

### 📁 文件: `app/(tabs)/profile.tsx`

这里主要要注意的是 **暗黑模式** 的处理。

```typescript
const giftSrc = useThemeValue(
  require("@/assets/images/profile/gift-green.png"), // ☀️ 亮色模式图片
  require("@/assets/images/profile/gift-red.png") // 🌙 暗色/其他模式图片
);
```

- **💡 useThemeValue**: 这是一个自定义 Hook，自动检测当前主题，返回对应的资源。在 RN 里，图片不像 CSS background-image 可以写在 class 里，很多时候是作为 Props 传进去的，所以需要用 JS 判断。

---

## 6. 持仓页 (Position)

### 📁 文件: `app/(tabs)/position.tsx`

非常简洁，主要充当路由容器。

```typescript
const Position = () => {
  // 🛒 又是同一个 Switch 状态，控制显示合约持仓还是交割持仓
  const positionCurrent = useAppSettingStore((s) => s.positionCurrent);

  return (
    <TabPageWrapper>
      <PositionHeader /> {/* 那个切换开关在这里面 */}
      {positionCurrent === PositionType.Contract ? (
        <ContractList />
      ) : (
        <FutureList />
      )}
    </TabPageWrapper>
  );
};
```

- 你会在 `ContractList` 里看到它是如何通过 Socket (`orderStompStore`) 实时更新你的持仓盈亏的（逻辑和首页行情类似）。

---

## 总结：如何接手？

1.  **改 UI**: 就像改 HTML 一样改 `index.tsx` 里的 JSX，记得 `className` 是 Tailwind。
2.  **改逻辑**:
    - 接口请求 -> 去 `hooks/` 找对应的 useQuery/useMutation。
    - Socket 逻辑 -> 去 `stores/*StompStore` 和 `hooks/*StompClient`。
3.  **加页面**: 在 `app/` 下新建 `.tsx` 文件，Expo Router 会自动生成路由。
