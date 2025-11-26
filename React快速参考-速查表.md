# React 快速参考速查表
## Vue开发者快速对照表

---

## 🔄 Vue → React 语法对照

### 组件定义
```vue
<!-- Vue -->
<template>
  <div>{{ message }}</div>
</template>
<script setup>
const message = ref('Hello')
</script>
```

```tsx
// React
export default function Component() {
  const [message, setMessage] = useState('Hello')
  return <div>{message}</div>
}
```

---

### 状态管理

| Vue | React |
|-----|-------|
| `const count = ref(0)` | `const [count, setCount] = useState(0)` |
| `count.value++` | `setCount(count + 1)` |
| `const state = reactive({})` | `const [state, setState] = useState({})` |
| `watch(count, (newVal) => {})` | `useEffect(() => {}, [count])` |
| `computed(() => a + b)` | `useMemo(() => a + b, [a, b])` |

---

### 生命周期

| Vue | React |
|-----|-------|
| `onMounted(() => {})` | `useEffect(() => {}, [])` |
| `onUnmounted(() => {})` | `useEffect(() => { return () => {} }, [])` |
| `onUpdated(() => {})` | `useEffect(() => {})` |
| `watch(() => count, () => {})` | `useEffect(() => {}, [count])` |

---

### 条件渲染

```vue
<!-- Vue -->
<div v-if="isShow">显示</div>
<div v-show="isShow">显示</div>
```

```tsx
// React
{isShow && <div>显示</div>}
{isShow ? <div>显示</div> : null}
```

---

### 列表渲染

```vue
<!-- Vue -->
<div v-for="item in list" :key="item.id">
  {{ item.name }}
</div>
```

```tsx
// React
{list.map(item => (
  <div key={item.id}>{item.name}</div>
))}
```

---

### 事件处理

```vue
<!-- Vue -->
<button @click="handleClick">点击</button>
<button @click="handleClick($event)">点击</button>
```

```tsx
// React
<button onClick={handleClick}>点击</button>
<button onClick={(e) => handleClick(e)}>点击</button>
```

---

### 双向绑定

```vue
<!-- Vue -->
<input v-model="value" />
```

```tsx
// React
const [value, setValue] = useState('')
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

---

### 样式绑定

```vue
<!-- Vue -->
<div :class="{ active: isActive }" :style="{ color: 'red' }">
```

```tsx
// React
<div className={isActive ? 'active' : ''} style={{ color: 'red' }}>
```

---

## 🛣️ 路由使用

### 路由导航

| Vue Router | React Router |
|------------|--------------|
| `router.push('/home')` | `navigate('/home')` |
| `router.replace('/home')` | `navigate('/home', { replace: true })` |
| `router.back()` | `navigate(-1)` |
| `route.params.id` | `const { id } = useParams()` |
| `route.query.name` | `useSearchParams()` |

### 代码示例
```tsx
import { useNavigate, useParams, useSearchParams } from 'react-router-dom'

function Component() {
  const navigate = useNavigate()
  const { id } = useParams()
  const [searchParams] = useSearchParams()
  
  const handleClick = () => {
    navigate('/home')
  }
  
  return <div>ID: {id}, Query: {searchParams.get('name')}</div>
}
```

---

## 🗄️ 状态管理（Zustand）

### 创建Store
```tsx
import { create } from 'zustand'

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}))
```

### 使用Store
```tsx
function Component() {
  const { count, increment } = useStore()
  // 或者
  const count = useStore(state => state.count)
  const increment = useStore(state => state.increment)
  
  return <button onClick={increment}>{count}</button>
}
```

### 持久化
```tsx
import { persist } from 'zustand/middleware'

const useStore = create(
  persist(
    (set) => ({
      token: '',
      setToken: (token) => set({ token }),
    }),
    {
      name: 'storage-key',
      storage: createJSONStorage(() => localStorage),
    }
  )
)
```

---

## 📡 数据获取（React Query）

### useQuery - 获取数据
```tsx
import { useQuery } from '@tanstack/react-query'

function Component() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })
  
  if (isLoading) return <div>加载中...</div>
  if (error) return <div>错误: {error.message}</div>
  
  return <div>{data?.name}</div>
}
```

### useMutation - 修改数据
```tsx
import { useMutation } from '@tanstack/react-query'

function Component() {
  const mutation = useMutation({
    mutationFn: (data) => createUser(data),
    onSuccess: () => {
      // 成功回调
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

---

## 🎨 UI组件库

### Ant Design（Admin项目）
```tsx
import { Button, Table, Form, Input } from 'antd'

// 表单
<Form onFinish={handleSubmit}>
  <Form.Item name="username" rules={[{ required: true }]}>
    <Input />
  </Form.Item>
  <Button type="primary" htmlType="submit">提交</Button>
</Form>

// 表格
<Table
  columns={columns}
  dataSource={data}
  loading={loading}
/>
```

### Ant Design Mobile（Mobile项目）
```tsx
import { Toast, Dialog, Popup } from 'antd-mobile'

// 提示
Toast.show({ content: '提示信息' })

// 对话框
Dialog.confirm({
  content: '确认删除？',
  onConfirm: () => {},
})
```

---

## 🎯 常用Hooks

### useState
```tsx
const [count, setCount] = useState(0)
const [user, setUser] = useState({ name: '', age: 0 })

// 更新对象
setUser(prev => ({ ...prev, name: 'John' }))
```

### useEffect
```tsx
// 组件挂载时执行
useEffect(() => {
  fetchData()
}, [])

// 依赖变化时执行
useEffect(() => {
  fetchData(id)
}, [id])

// 清理函数
useEffect(() => {
  const timer = setInterval(() => {}, 1000)
  return () => clearInterval(timer)
}, [])
```

### useMemo
```tsx
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b)
}, [a, b])
```

### useCallback
```tsx
const handleClick = useCallback(() => {
  doSomething(a, b)
}, [a, b])
```

### useRef
```tsx
const inputRef = useRef<HTMLInputElement>(null)

<input ref={inputRef} />
inputRef.current?.focus()
```

---

## 🌐 国际化（i18next）

```tsx
import { useTranslation } from 'react-i18next'

function Component() {
  const { t } = useTranslation()
  
  return <div>{t('home.title')}</div>
}
```

---

## 📦 项目常用工具

### 环境变量
```tsx
const apiUrl = import.meta.env.VITE_BASE_URL
```

### 路径别名
```tsx
import Component from '@/components/Component'
import { api } from '@/api/user'
```

### 类型定义
```tsx
interface User {
  id: string
  name: string
}

const user: User = { id: '1', name: 'John' }
```

---

## 🔧 常用工具函数

### 类名合并（classnames）
```tsx
import classNames from 'classnames'

<div className={classNames('base', { active: isActive })}>
```

### 日期处理（dayjs）
```tsx
import dayjs from 'dayjs'

dayjs().format('YYYY-MM-DD')
dayjs(date).fromNow()
```

### 数字格式化（numeral）
```tsx
import numeral from 'numeral'

numeral(1000).format('0,0')  // 1,000
```

---

## ⚠️ 常见错误和解决方案

### 1. 依赖数组缺失
```tsx
// ❌ 错误
useEffect(() => {
  fetchData(id)
}, [])

// ✅ 正确
useEffect(() => {
  fetchData(id)
}, [id])
```

### 2. 状态更新后立即读取
```tsx
// ❌ 错误
setCount(count + 1)
console.log(count)  // 旧值

// ✅ 正确
setCount(prev => prev + 1)
```

### 3. 列表渲染缺少key
```tsx
// ❌ 错误
{list.map(item => <div>{item.name}</div>)}

// ✅ 正确
{list.map(item => <div key={item.id}>{item.name}</div>)}
```

### 4. 条件渲染返回undefined
```tsx
// ❌ 错误
{condition && <Component />}  // condition为0时显示0

// ✅ 正确
{condition ? <Component /> : null}
{!!condition && <Component />}
```

---

## 📚 项目文件快速定位

| 需求 | 文件位置 |
|------|---------|
| 添加新页面 | `src/views/` + `src/router/index.tsx` |
| 添加API接口 | `src/api/` |
| 添加状态管理 | `src/store/` |
| 添加公共组件 | `src/components/` |
| 添加工具函数 | `src/utils/` |
| 添加类型定义 | `src/types/` |
| 修改路由配置 | `src/router/index.tsx` |
| 修改请求配置 | `src/utils/request/index.ts` |

---

## 🚀 快速调试技巧

### 1. React DevTools
- 安装浏览器扩展：React Developer Tools
- 查看组件树、状态、Props

### 2. 控制台调试
```tsx
console.log('当前状态:', state)
console.log('Props:', props)
```

### 3. 断点调试
- 在代码中添加 `debugger`
- 使用浏览器开发者工具

---

**保存此文档，随时查阅！** 📌

