## 1. 新增一个最简单的页面

目标：新增一个 `/demo` 页面，只显示一句话“Hello RN + Expo Router”，用于快速验证路由和样式。

### 1.1 创建页面文件

在 `app/` 目录下新建文件：`app/demo.tsx`

```tsx
import { View, Text } from "react-native";
import { PageSafeAreaView } from "@/components/app/SafeAreaView/PageSafeAreaView";

export default function DemoScreen() {
  return (
    <PageSafeAreaView>
      <View className="flex-1 items-center justify-center bg-white">
        <Text className="text-lg font-bold">Hello RN + Expo Router</Text>
      </View>
    </PageSafeAreaView>
  );
}
```

说明：

- `PageSafeAreaView`：项目中的统一安全区容器，会自动处理 iOS 刘海、底部 Home 条等问题
- `className`：使用 NativeWind/Tailwind 写样式
  - `flex-1`：填满整个屏幕
  - `items-center`：横向居中
  - `justify-center`：纵向居中
  - `bg-white`：白色背景

### 1.2 运行并访问

```bash
npm run web      # 或 npm run android / npm run ios
```

在浏览器或 App 中访问 `/demo`：

- Web 模式：浏览器地址栏输入 `http://localhost:xxxx/demo`
- App 模式：可以在代码中增加一个按钮跳转（见第 4 节）

如果你能看到那句 “Hello RN + Expo Router”，说明：

- expo-router 路由工作正常
- NativeWind 样式生效
- 你已经完成了第一个 RN 页面 🎉

---

## 2. 在底部 Tab 中新增一个 Tab

目标：在底部导航栏中新增一个「市场」Tab，点击后进入 `/market` 页面。

### 2.1 创建页面：`app/(tabs)/market.tsx`

```tsx
import { View, Text } from "react-native";
import { PageSafeAreaView } from "@/components/app/SafeAreaView/PageSafeAreaView";

export default function MarketScreen() {
  return (
    <PageSafeAreaView>
      <View className="flex-1 items-center justify-center bg-f1">
        <Text className="text-base font-bold">Market Page</Text>
      </View>
    </PageSafeAreaView>
  );
}
```

### 2.2 在 Tab 配置中加入新项

编辑：`app/(tabs)/_layout.tsx`

1. 找到 `tabList` 数组，在最后新增一项：

```ts
const tabList = [
  // ...已有 tab
  {
    name: "market",
    href: "/market",
    title: t("Market", "Market"),
    icon: require("@/assets/images/tab/market.png"),
    greenIcon: require("@/assets/images/tab/market-green.png"),
    redIcon: require("@/assets/images/tab/market-red.png"),
  },
];
```

2. 准备好对应图标，放到 `assets/images/tab/` 目录下：

```text
assets/images/tab/
  market.png
  market-green.png
  market-red.png
```

3. 重新运行 / 热更新后，就可以在底部看到新增加的「Market」Tab。

---

## 3. 在页面中发起接口请求并展示数据

标准做法：**页面只使用 Hook 获取数据，不直接调用 axios**。

下面以「获取用户信息并展示」为例，总结一条完整链路：`页面 -> hooks -> service -> 接口`。

### 3.1 编写接口函数（service 层）

假设在 `service/userService.ts` 中新增：

```ts
import request from "@/service/request";
import type { UserInfo } from "@/types/user";

export function getUserInfo() {
  return request.get<UserInfo>("/user/info");
}
```

说明：

- `request`：项目统一封装的 axios 实例（含 baseURL、拦截器等）
- `UserInfo`：用户信息类型定义，放在 `types/user.ts`

### 3.2 编写 Hook（hooks 层）

在 `hooks/user/useGetUserInfo.ts` 中：

```ts
import { useQuery } from "@tanstack/react-query";
import { getUserInfo } from "@/service/userService";

export function useGetUserInfo() {
  return useQuery({
    queryKey: ["userInfo"],
    queryFn: getUserInfo,
  });
}
```

说明：

- `queryKey`：缓存 key，决定 react-query 如何缓存/区分不同请求
- `queryFn`：真正发请求的函数（我们刚刚在 service 中写的）

### 3.3 在页面中使用（app/profile/info.tsx 例子）

```tsx
import { Text } from "react-native";
import { PageSafeAreaView } from "@/components/app/SafeAreaView/PageSafeAreaView";
import { Loading } from "@/components/ui/Loading/Loading";
import { UserInfo } from "@/components/profile/UserInfo";
import { useGetUserInfo } from "@/hooks/user/useGetUserInfo";

export default function ProfileInfoScreen() {
  const { data: user, isLoading, error } = useGetUserInfo();

  if (isLoading) return <Loading />;
  if (error) return <Text>加载用户信息失败</Text>;

  return (
    <PageSafeAreaView>
      <UserInfo user={user} />
    </PageSafeAreaView>
  );
}
```

> **套路记忆：**  
> 所有“页面需要的数据”，尽量通过 `useXXX` Hook 拿，页面只负责显示，不自己发请求。

---

## 4. 页面之间的跳转（导航）

跳转方式分两类：

1. 直接使用 `expo-router` 的 `useRouter`
2. 使用项目封装好的 `useAppRouter`（推荐）

### 4.1 使用 `useRouter`（原生方式）

```tsx
import { useRouter } from "expo-router";
import { Pressable, Text } from "react-native";

export default function Example() {
  const router = useRouter();

  return (
    <Pressable onPress={() => router.push("/profile/info")}>
      <Text>去个人信息</Text>
    </Pressable>
  );
}
```

常用 API：

- `router.push(path)`：入栈跳转
- `router.replace(path)`：替换当前页面
- `router.back()`：返回上一页

### 4.2 使用项目自带的 `useAppRouter`（推荐）

在 `hooks/router/useAppRouter.ts` 中通常有类似封装（示意）：

```ts
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
      router.replace("/");
    },
    // ...其他统一封装
  };
}
```

页面中使用：

```tsx
import { useAppRouter } from "@/hooks/router/useAppRouter";
import { Pressable, Text } from "react-native";

export default function Example() {
  const { toProfileInfo } = useAppRouter();

  return (
    <Pressable onPress={toProfileInfo}>
      <Text>去个人信息</Text>
    </Pressable>
  );
}
```

优点：

- 所有路径统一定义，路径改动只需改一处
- 便于做权限控制（如：跳个人信息前先判断是否登录）

---

## 5. 使用多语言（i18n）改文案

项目中已经集成了 `i18next + react-i18next`，并配置了自动扫描脚本 `npm run sync`。

### 5.1 基本用法：`useTranslation`

在页面或组件中：

```tsx
import { useTranslation } from "react-i18next";
import { Text } from "react-native";

export function ExampleTitle() {
  const { t } = useTranslation();

  return <Text>{t("Profile.Title", "Profile")}</Text>;
}
```

说明：

- 第一个参数 `"Profile.Title"`：语言 key
- 第二个参数 `"Profile"`：当语言文件里没有这个 key 时的默认展示文案

### 5.2 新增一个多语言 key 的标准流程

1. 在代码里加入：

   ```tsx
   t("Loan.ApplyTitle", "Apply for Loan")
   ```

2. 执行多语言扫描脚本：

   ```bash
   npm run sync
   ```

   它会：
   - 用 `i18next-scanner` 扫描所有 `t("xxx")` 调用
   - 在 `public/static/langs` 下生成/更新 JSON
   - 使用 `i18nexus` CLI 将结果同步到翻译平台（需要配置密钥）

3. 在翻译平台或 JSON 文件中为该 key 配置多语言翻译

4. App 下次启动或刷新后即可看到生效的多语言文案。

---

## 6. 使用 Tailwind / NativeWind 写样式

项目中大量组件使用 `className` 属性写样式，这得益于 `nativewind` 与 `tailwindcss` 的集成。

### 6.1 基本例子

```tsx
import { View, Text } from "react-native";

export function StyledBox() {
  return (
    <View className="bg-f1 px-4 py-3 rounded-lg">
      <Text className="text-base font-bold text-primary">
        标题文字
      </Text>
    </View>
  );
}
```

解析：

- `bg-f1`：在 `tailwind.config.js` 中定义的背景色
- `px-4` / `py-3`：水平/垂直 padding
- `rounded-lg`：圆角
- `text-primary`：主色文字

### 6.2 常用调试方式

1. 找不到某个类名：打开 `tailwind.config.js` 查一下是否定义
2. 样式不生效：
   - 确保组件是 RN 的 `View` / `Text` / `Image` 而不是 DOM 元素
   - 确保 `className` 没有写错（注意大小写）

### 6.3 与 StyleSheet 混合使用

当样式特别复杂时，可以混用：

```tsx
import { StyleSheet, View } from "react-native";

export function MixedStyle() {
  return (
    <View className="bg-f1" style={styles.box}>
      {/* ... */}
    </View>
  );
}

const styles = StyleSheet.create({
  box: {
    shadowColor: "#000",
    shadowOpacity: 0.1,
    shadowRadius: 10,
  },
});
```

> 一般推荐：**简单布局走 Tailwind，特殊效果走 StyleSheet 或内联 style**。

---

## 7. 使用 WebSocket 行情 / 订单订阅（了解即可）

项目中行情和订单推送使用 STOMP + SockJS，封装在 `hooks/marketStompClient` 和 `hooks/orderStompClient` 下。

### 7.1 行情相关 hooks

- `useConnectMarketStomp`：建立行情 WebSocket 连接
- `useSubscribeKLinePrice`：订阅 K 线价格
- `useSubKLineInfo`：订阅 K 线数据
- `useHandleKlineSubscribeRes`：处理 K 线推送
- `useHandlePriceSubscribeRes`：处理价格推送

典型使用模式：

```tsx
// 示例：在某个行情页面中
import { useConnectMarketStomp } from "@/hooks/marketStompClient/useConnectMarketStomp";
import { useSubscribeKLinePrice } from "@/hooks/marketStompClient/useSubscribeKLinePrice";

export default function MarketPage() {
  useConnectMarketStomp();      // 建立连接
  useSubscribeKLinePrice();     // 订阅价格推送

  // ...渲染行情组件
}
```

### 7.2 订单相关 hooks

- `useConnectOrderStomp`：建立订单相关 WebSocket 连接
- `useSubscribeUserOrders`：订阅当前用户订单
- `useOrderSubscribeRes`：处理订单推送
- `useExpiryOrderSubscribeRes`：处理到期订单推送

**对你而言，刚上手时重点是知道这些逻辑在哪里，而不是立刻改动它们。等熟悉普通 HTTP 流程后，再来研究 WebSocket 逻辑。**

---

## 8. 常见修改场景：改一个表单字段

例子：在修改密码页面 `app/profile/change-password.tsx` 中调整校验逻辑。

### 8.1 找到页面

1. 路由路径：`/profile/change-password`
2. 对应文件：`app/profile/change-password.tsx`

打开文件后你会看到：

- 使用了 `Formik` 或 `Yup` 进行表单校验
- 使用了 `InputWithTitle` / `NumberInputWithTitle` 等组件
- 使用了 `hooks/user/useUpdatePwd` 或类似 Hook 提交请求

### 8.2 修改校验规则（示意）

假设原来规则是“密码至少 6 位”，你想改为“至少 8 位 + 必须包含数字”：

```ts
// 原本
Yup.string().min(6, "密码至少 6 位");

// 修改后
Yup.string()
  .min(8, "密码至少 8 位")
  .matches(/\d/, "密码中至少包含一个数字");
```

### 8.3 验证效果

1. 运行项目：`npm run web` 或 `npm run android`
2. 打开修改密码页面
3. 尝试输入不符合规则的密码，看提示是否正确展示

---

## 9. 开发/调试小技巧总结

### 9.1 不知道某个东西在哪？

1. 看页面路径，定位 `app/` 下对应 tsx
2. 看页面引用的组件路径：
   - `/components/...` → 说明是 UI 组件
   - `/hooks/...` → 说明是业务逻辑 Hook
   - `/service/...` → 说明是接口函数
3. 用全局搜索（或者 Cursor 的 code search）搜函数名/变量名

### 9.2 界面样式出问题

- 确认是否使用了 RN 组件（`View` / `Text`），而不是 Web DOM
- 看 `className` 拼写是否正确
- 看 `tailwind.config.js` 是否有这个类/颜色

### 9.3 接口出问题

- 打开对应 Hook（通常在 `hooks/xxx` 下）看请求调用
- 再进入 `service/xxxService.ts` 看接口地址和参数
- 暂时加 `console.log` 打印请求参数/返回结果

### 9.4 App 启动异常 / 白屏

- 尝试清理缓存：

  ```bash
  rm -rf $TMPDIR/metro-cache
  npm run start
  ```

- 检查最近修改：
  - TypeScript 是否报错
  - 某个组件是否返回了 `null` 或抛错（可配合 `AppToast` / `FallbackRender`）

---

## 10. 学习路线（从新手到能独立开发页面）

1. **跑起来**：参考《01-项目总览与运行指南》，能在 Web + 模拟器里跑通项目
2. **看懂路由**：参考《02-目录结构与路由说明》，搞懂 `app/` 结构与 Tab 布局
3. **做一个 demo 页面**：按照本章第 1 节，加一个 `/demo`
4. **加一个 Tab**：照第 2 节，加一个简单 Tab
5. **改一个真实页面的小逻辑**：例如个人信息页的一个字段展示
6. **自己新建一个真实业务页面**：
   - 定义接口 + 类型
   - 编写 service + hooks
   - 使用组件拼 UI
   - 加入路由或 Tab

走完这条路径，你已经可以在这个 RN 项目里承担普通需求的开发与联调了。


