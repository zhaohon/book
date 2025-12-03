## 1. 项目定位与技术栈

**项目名**：`ylchat-mobile`  
**类型**：基于 **React Native + Expo + expo-router** 的移动端应用，同时支持 **原生 App + H5(Web)**。

你可以把它简单理解为：

- 用 **React 写移动端 App**（React Native）
- 用 **Expo** 当脚手架和运行环境
- 用 **expo-router** 当路由系统（类似 Next.js 的文件路由）
- 再加上一套自己的业务组件、业务 Hook、接口封装

**主要技术栈一览：**

- **核心框架**
  - `react`：19.x
  - `react-native`：0.81.x
  - `expo`：^54.0.0
  - `expo-router`：~6.0.10
- **语言与工具**
  - `typescript`：~5.9
  - `jest` + `jest-expo`：测试
  - `eslint`：代码规范
- **UI / 样式**
  - `nativewind` + `tailwindcss`：用 Tailwind 风格写 RN 样式
  - 一些 Expo UI 能力：`expo-blur`、`expo-linear-gradient`、`expo-image` 等
- **状态与数据**
  - `zustand`：应用状态管理（替代 Redux）
  - `@tanstack/react-query`：数据请求、缓存与刷新
- **网络与工具**
  - `axios`：HTTP 请求
  - `dayjs`：时间处理
  - `immer`：不可变数据
  - `numeral`：数字/金额格式化
- **国际化**
  - `i18next` + `react-i18next`
  - `i18next-scanner` + `i18nexus-cli`：自动扫描文案 key 并同步到翻译平台
- **图表 / 行情**
  - `klinecharts`：K 线图
  - 自己封装了一套 `components/chart` 组件
- **WebSocket / STOMP**
  - `@stomp/stompjs`
  - `sockjs-client`
- **其他**
  - `react-native-reanimated` + `react-native-gesture-handler`
  - `react-native-webview`（H5 内嵌）
  - `tronweb`、`ethers` 等（区块链相关）

> 作为刚从 React 过渡来的同学：你可以先把 RN 视为“换了一套基础组件 + 样式写法”，其余业务结构（组件拆分、hooks、service）跟你熟悉的 React Web 项目差异不大。

---

## 2. Node 版本与开发环境准备

### 2.1 推荐 Node 版本

项目 README 中对于 H5 打包的说明建议：

- **Node.js 版本：v18.x.x 及以上**

保持和 CI/构建环境相近，可以减少很多莫名其妙的问题。

### 2.2 建议工具链

- **操作系统**：macOS / Windows / Linux 均可（你当前是 macOS）
- **必须**
  - Git
  - Node.js 18+
  - npm（项目带有 `package-lock.json`，默认用 npm）
- **建议**
  - Node 版本管理工具：`nvm` / `fnm` / `asdf`
  - 移动端模拟器：
    - Android：Android Studio + AVD
    - iOS：Xcode + iOS Simulator（macOS 专属）
  - VS Code / Cursor 等编辑器

### 2.3 Node 版本切换示例（nvm）

```bash
# 安装 Node 18
nvm install 18

# 切换到 18
nvm use 18

# 设置默认版本（可选）
nvm alias default 18
```

如果使用的是 `fnm` / `asdf`，根据它们各自的文档设置即可。

---

## 3. 安装依赖

进入项目根目录（注意路径）：

```bash
cd /Users/hello/Desktop/project/thailand/ex/all/ylchat-mobile
```

安装依赖：

```bash
npm install
```

如遇到网络问题可以先设置镜像：

```bash
npm config set registry https://registry.npmmirror.com
```

> 如果你个人更习惯 `pnpm` / `yarn`，可以在本地使用，但要注意锁文件的一致性（团队协作时尽量统一）。

---

## 4. 项目脚本说明（从 package.json 读懂项目）

`package.json` 中的关键脚本如下（节选）：

```json
{
  "main": "expo-router/entry",
  "scripts": {
    "start": "expo start",
    "reset-project": "node ./scripts/reset-project.js",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "test": "jest --watchAll",
    "lint": "expo lint",
    "eas-build-post-install": "rm -rf $TMPDIR/metro-cache",
    "typecheck": "tsc --noEmit",
    "prepare": "husky",
    "postinstall": "patch-package",
    "sync": "rm -f ./public/static/langs/en/resource.json && i18next-scanner --config i18next-scanner.config.cjs && i18nexus import ./public/static/langs/en/resource.json --overwrite -ns default -k Cx3SzSokxVBlEXI_9S1H8A -t ee611fd6-262b-44e5-8da9-478c3ed00904"
  }
}
```

**核心脚本解释：**

- **开发调试相关**
  - `npm run start`：启动 Expo 开发服务器（通用入口）
  - `npm run android`：启动开发服务器并在 Android 模拟器或连接的真机上打开 App
  - `npm run ios`：启动开发服务器并在 iOS 模拟器或真机上打开 App（仅 macOS）
  - `npm run web`：以 Web(H5) 模式运行（浏览器打开）
- **质量与检查**
  - `npm run lint`：执行 ESLint 检查
  - `npm run typecheck`：只执行 TypeScript 类型检查（不产出构建结果）
  - `npm test`：Jest 单元测试，`--watchAll` 会持续监听文件变化
- **构建相关**
  - `eas-build-post-install`：供 EAS 构建时使用，清理 Metro 缓存
- **项目维护**
  - `npm run reset-project`：执行 `scripts/reset-project.js`，一般用于一键清理缓存/重置项目（可以打开脚本文件看具体做了什么）
  - `npm run sync`：扫描代码中的 i18n key，并与 i18nexus 平台进行同步
  - `prepare`：安装依赖后自动执行 `husky` 初始化 git hook
  - `postinstall`：每次安装依赖后自动执行 `patch-package`（应用 `patches` 目录中的补丁）

---

## 5. 本地运行（原生 App）

### 5.1 启动开发服务器

在项目根目录执行：

```bash
npm run start
```

执行后你会看到：

- 终端中出现一个二维码
- 提示本地开发服务器地址（如 `http://localhost:19000`）

Expo 会同时监听多个平台（Android/iOS/Web），你可以根据提示按键或使用 CLI 指令选择平台。

### 5.2 在 Android 上运行

#### 方式一：Android 模拟器

1. 安装 **Android Studio**
2. 创建一个 Android 虚拟设备（AVD）
3. 保持模拟器处于运行状态
4. 在项目根目录执行：

   ```bash
   npm run android
   ```

Expo 会自动连接到模拟器并安装/启动 App。

#### 方式二：真机 + Expo Go

1. 手机安装 `Expo Go` 应用
2. 确保手机和电脑在同一个局域网（相同 Wi-Fi）
3. `npm run start` 后，终端中会显示二维码
4. 使用 **Expo Go** 扫描该二维码，即可在手机上打开当前项目

### 5.3 在 iOS 上运行（仅 macOS）

#### 方式一：iOS 模拟器

1. 安装最新的 **Xcode**
2. 打开 iOS 模拟器（Xcode -> Open Developer Tool -> Simulator）
3. 在项目根目录执行：

   ```bash
   npm run ios
   ```

4. Expo 会自动在模拟器中安装并运行 App。

#### 方式二：真机 + Expo Go

步骤与 Android 类似：

1. 在 iPhone 上安装 `Expo Go`
2. 保证电脑和手机在同一网络环境
3. `npm run start`，使用 `Expo Go` 扫描终端中的二维码即可。

---

## 6. 运行 Web（H5 模式）

### 6.1 开发模式（Web）

```bash
npm run web
```

此命令会：

- 启动一个 Web Dev Server
- 在浏览器中打开一个类似 `http://localhost:8081` 或其他端口的地址
- 使用 `react-native-web` 将 RN 组件映射到 DOM 元素，实现 H5 预览

对于刚接触 RN 的同学，非常推荐先在 Web 模式下熟悉 UI 和路由，再切换到 App 端。

### 6.2 导出构建 H5 静态资源

项目根目录下的 `README.md` 已经写了简单的 H5 构建步骤：

```bash
# 安装依赖
npm install

# 打包 H5
npx expo export --platform web
```

`expo export --platform web` 会：

- 基于当前 expo 配置
- 生成一份可部署的静态 H5 站点

你可以将导出的静态文件部署到任意静态服务器（如 S3、Nginx、Vercel 等）。

> 如果你经常需要导出 H5，可以在 `package.json` 里新增一个脚本，比如 `"build:web": "expo export --platform web"`，便于记忆和使用。

---

## 7. 初次上手推荐步骤（给 React 开发者的路线）

### 步骤 1：环境准备

1. 安装 Node 18+，并配置环境变量
2. 安装 npm 相关工具（可选：切换镜像）
3. （建议）安装 Android Studio 和 Xcode，方便调试原生

### 步骤 2：安装依赖

```bash
cd /Users/hello/Desktop/project/thailand/ex/all/ylchat-mobile
npm install
```

### 步骤 3：先跑 Web

```bash
npm run web
```

在浏览器中访问项目，熟悉：

- 底部 Tab 栏（来自 `app/(tabs)/_layout.tsx`）
- 首页 / 个人中心等 Tab 页

### 步骤 4：了解目录结构（配合《02-目录结构与路由说明》）

重点认识：

- `app/`：所有路由页面入口
- `components/`：通用组件
- `hooks/`：业务逻辑
- `service/`：接口请求
- `stores/`：全局状态

### 步骤 5：跑起来原生应用

```bash
npm run android
```

或：

```bash
npm run ios
```

在模拟器/真机上看看效果。

### 步骤 6：做一个“小改动验证理解”

例如：

- 打开 `app/(tabs)/index.tsx`（Dashboard 页）
- 修改某个标题文字
- 观察 Web / App 中是否实时更新

通过这一轮，你会对：

- RN 基本组件（`View` / `Text` / `Image`）
- Tailwind 风格样式（`className="px-4 py-2"`）
- expo-router 文件路由

有一个非常直观的感受。

---

## 8. 常见问题（FAQ）

### 8.1 启动时报错：React / React Native 版本不兼容

项目中对 React 版本有 `overrides` 和 `resolutions`：

```json
"overrides": {
  "react": "19.1.0"
},
"resolutions": {
  "react-native-safe-area-context": "5.6.0"
}
```

如果安装时遇到 peerDependencies 警告或冲突，优先：

1. 删除 `node_modules` 和 `package-lock.json`
2. 重新 `npm install`

一般不建议随意升级/降级 React / RN 版本，除非你完全理解整条依赖链。

### 8.2 Metro 缓存问题（代码不更新 / 打包失败）

症状：

- 改了代码但 App 不刷新
- 启动时报一些诡异的错误

解决方式：

```bash
# 手动清理缓存
rm -rf $TMPDIR/metro-cache

# 重新启动
npm run start
```

或者使用：

```bash
expo start -c
```

### 8.3 iOS/Android 连接不上开发服务器

常见原因：

- 电脑和手机不在同一局域网
- 公司网络防火墙拦截了某些端口

解决建议：

- 尽量让两者使用同一个 Wi-Fi
- 尝试切换到 LAN / Tunnel 模式（Expo DevTools 界面中可选）

---

## 9. 记忆小结

如果你已经能熟练跑起一个 React Web 项目，那么在这个 RN 项目中，你只需要重点记住三点：

- **跑项目**：`npm install` + `npm run start` / `npm run android` / `npm run ios` / `npm run web`
- **看结构**：界面都在 `app/`，UI 组件在 `components/`，逻辑在 `hooks/ + service/ + stores/`
- **有问题先清缓存**：`rm -rf $TMPDIR/metro-cache` 或 `expo start -c`

等你熟悉了本篇内容，可以再配合阅读：

- 《02-ylchat-mobile-目录结构与路由说明》
- 《03-ylchat-mobile-常见开发任务与示例》

你就基本能在这个项目里“自由活动”了。


