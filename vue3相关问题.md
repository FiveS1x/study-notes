

# Pinia相关

## 安装

```bash
npm install pinia
```



## 配置Pinia

### 创建主 store 文件

```ts
// src/stores/index.ts
import { defineStore } from 'pinia'

// 定义根状态类型
interface RootState {
  appName: string
  version: string
}

export const useMainStore = defineStore('main', {
  state: (): RootState => ({
    appName: 'My Vue3 App',
    version: '1.0.0'
  }),
  getters: {
    appInfo: (state): string => `${state.appName} v${state.version}`
  },
  actions: {
    updateAppName(newName: string) {
      this.appName = newName
    }
  }
})
```



### 在 main.ts 中安装 Pinia

```ts
// src/main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

// 创建 Pinia 实例
const pinia = createPinia()
app.use(pinia)

app.mount('#app')
```



## 使用Pinia

### 在组件中使用 Pinia

```vue
<!-- src/components/AppHeader.vue -->
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/stores'

const mainStore = useMainStore()
const { appName, version } = storeToRefs(mainStore)

// 修改状态
const updateName = () => {
  mainStore.updateAppName('New App Name')
}
</script>

<template>
  <header>
    <h1>{{ appName }} - v{{ version }}</h1>
    <button @click="updateName">更改应用名</button>
  </header>
</template>
```



## Pinia 持久化存储

### 安装插件、引入插件、使用

```
npm install pinia-plugin-persistedstate
```

```ts
// src/main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

app.use(pinia)
```

```ts
export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[]
  }),
  persist: true // 启用持久化
})
```



## 类型安全的 store

```ts
// 创建 store 类型声明文件
// src/types/pinia.d.ts
import { User } from './user'

interface MainStore {
  appName: string
  user: User | null
  isAuthenticated: boolean
}

// 扩展 pinia 类型
declare module 'pinia' {
  export interface PiniaCustomProperties {
    // 可以在这里添加全局方法
    $resetAllStores(): void
  }
  
  export interface DefineStoreOptionsBase<S, Store> {
    // 允许为所有 store 定义持久化选项
    persist?: boolean | PersistOptions
  }
}
```





# Vue Router相关

## 安装

```bash
npm install vue-router@4
```



## 配置Vue Router

### 创建路由配置文件

```ts
// src/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

// 定义路由组件 (使用懒加载)
const Home = () => import('@/views/HomeView.vue')
const About = () => import('@/views/AboutView.vue')

// 定义路由数组
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: {
      title: '首页'
    }
  },
  {
    path: '/about',
    name: 'About',
    component: About,
    meta: {
      requiresAuth: true,
      title: '关于我们'
    }
  },
  // 404 页面
  {
    path: '/:pathMatch(.*)*',
    redirect: '/'
  }
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes
})

// 全局前置守卫
router.beforeEach((to, from, next) => {
  const store = useMainStore()
  
  // 设置页面标题
  const title = to.meta.title as string || store.appName
  document.title = `${title} | ${store.appName}`
  
  // 认证检查
  if (to.meta.requiresAuth && !store.isAuthenticated) {
    next({ name: 'Login' })
  } else {
    next()
  }
})

export default router
```



### 在 main.ts 中安装 Vue Router

```ts
// src/main.ts (更新)
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router' // 导入路由配置

const app = createApp(App)

// 使用 Pinia
const pinia = createPinia()
app.use(pinia)

// 使用 Vue Router
app.use(router)

app.mount('#app')
```



## 使用Vue Router

### 在组件中使用 Vue Router

```vue
<!-- src/App.vue -->
<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import AppHeader from '@/components/AppHeader.vue'
</script>

<template>
  <AppHeader />
  
  <nav>
    <RouterLink :to="{ name: 'Home' }">首页</RouterLink>
    <RouterLink :to="{ name: 'About' }">关于</RouterLink>
  </nav>

  <main>
    <RouterView />
  </main>
</template>
```



### 创建基础视图组件

**HomeView.vue**

```vue
<!-- src/views/HomeView.vue -->
<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

const goToAbout = () => {
  router.push({ name: 'About' })
}
</script>

<template>
  <div class="home">
    <h2>欢迎来到首页</h2>
    <button @click="goToAbout">前往关于页面</button>
  </div>
</template>
```



**AboutView.vue**

```vue
<!-- src/views/AboutView.vue -->
<script setup lang="ts">
import { useMainStore } from '@/stores'

const mainStore = useMainStore()
const { appInfo } = storeToRefs(mainStore)
</script>

<template>
  <div class="about">
    <h2>关于我们</h2>
    <p>{{ appInfo }}</p>
    <p>当前用户: {{ mainStore.user?.name || '未登录' }}</p>
  </div>
</template>
```



## 路由懒加载优化

```ts
// 使用 webpackChunkName 注释
const UserProfile = () => import(/* webpackChunkName: "user" */ '@/views/UserProfile.vue')
```





# 组件相关

## 使用组件

在 `script` 中引入后，直接在 `template` 中使用即可。

```vue
<template>
  <div>
    <h1>homeView</h1>
    <!-- 直接使用 -->
    <!-- 格式1 -->
    <testOne />
    <!-- 格式2 -->
    <test-one></test-one>
  </div>
</template>

<script setup lang="ts">
// 引入组件
import testOne from "@/components/testOne.vue";
</script>
```



## Vue 2 (选项式 API) 与 Vue 3 (组合式 API) 对比

| 特性       | Vue 2 选项式 API          | Vue 3 组合式 API          |
| :--------- | :------------------------ | :------------------------ |
| 组件定义   | `data/methods/props` 选项 | `<script setup>` + 函数式 |
| 响应式数据 | `data()` 返回对象         | `ref()`/`reactive()`      |
| 生命周期   | `mounted()` 等选项        | `onMounted()` 等函数      |
| 代码组织   | 按选项类型分组            | 按功能逻辑组织            |



## 组件通信与类型标注

### Props 类型声明

1. **运行时声明**：通过对象定义类型校验，但代码较繁琐

   ```ts
   defineProps({
     title: { type: String, required: true },
     count: { type: Number, default: 0 }
   })
   ```

   

2. **基于类型的声明**（推荐）：使用泛型或接口，简洁直观

   ```ts
   interface Props { title: string; count?: number }
   const props = defineProps<Props>()
   ```

   

3. **默认值处理**：通过 `withDefaults` 或响应性语法糖（需配置 Vite）

   ```ts
   const { title, count = 10 } = defineProps<Props>() // 需开启 reactivityTransform
   ```



### Emits 类型声明

使用带调用签名的类型字面量定义事件及参数

```ts
const emit = defineEmits<{
  (e: 'update:count', value: number): void
  (e: 'submit'): void
}>()
```



## 响应式数据与类型标注

1. `ref()` 类型标注

   简单类型可自动推导，复杂类型需显式声明

   ```ts
   const count = ref<number>(0) // 显式泛型
   const user = ref<User | null>(null) // 联合类型
   ```

   

2. `reactive()` 类型标注

   优先使用接口定义对象结构

   ```ts
   interface User { name: string; age: number }
   const user: User = reactive({ name: 'Alice', age: 25 })
   ```

   

3. `computed()` 类型标注

   自动推导返回值类型，或通过泛型显式指定

   ```ts
   const double = computed<number>(() => count.value * 2)
   ```



## 模板与事件处理类型标注

1. **DOM 事件处理**

   为事件参数标注具体类型（如 `MouseEvent`）

   ```ts
   const handleClick = (e: MouseEvent) => {
     console.log((e.target as HTMLButtonElement).value)
   }
   ```

   

2. **模板引用（Refs）**

   - **DOM 元素**：指定原生元素类型

     ```ts
     const inputRef = ref<HTMLInputElement | null>(null)
     ```

     

   - **子组件**：通过 `InstanceType` 获取组件实例类型

     ```ts
     const childRef = ref<InstanceType<typeof ChildComponent>>()
     ```



## 依赖注入与上下文

**Provide / Inject**

使用 `InjectionKey` 同步类型

```ts
import type { InjectionKey } from 'vue'
const key = Symbol() as InjectionKey<string>
provide(key, 'hello') // 注入
const value = inject(key) // 获取 (类型: string | undefined)
```



## 插槽与作用域

**作用域插槽**

通过插槽 props 传递数据并标注类型

```vue
<!-- 父组件 -->
<Child>
  <template #default="{ item }: { item: Item }">
    {{ item.name }}
  </template>
</Child>
```



## 组件设计、最佳实践、使用

1. 组合式函数（Composable）

   封装逻辑时显式标注输入/输出类型

   ```ts
   export function useCounter(initial: number) {
     const count = ref(initial)
     const increment = () => count.value++
     return { count, increment }
   }
   ```

   

2. 性能与优化

   - 使用 `v-if/v-show` 条件渲染、`keep-alive` 缓存组件
   - 避免在模板中写复杂逻辑，用 `computed` 替代

3. 可维护性

   - 为复杂组件编写单元测试（Vitest + Vue Test Utils）
   - 使用 `scoped` 样式或 CSS Modules 隔离样式
   
4. 复杂组件使用组合式 API 按功能组织代码

5. Props 声明时指定类型和验证规则

6. 组件事件名使用 kebab-case（如 `update-count`）

7. 优先使用作用域插槽实现组件复用

8. 使用 `defineExpose()` 暴露子组件方法

9. 状态变更通过事件通知父组件（保持单向数据流）



## 案例

### 子组件 ChildComponent.vue

```vue
<template>
  <div class="child">
    <!-- 1. 接收父组件传递的 Props -->
    <h3>{{ title }}</h3>
    <p>计数: {{ count }}</p>
    
    <!-- 2. 触发自定义事件 -->
    <button @click="increment">子组件+1</button>
    
    <!-- 3. 插槽内容展示 -->
    <div class="slot-area">
      <slot>默认内容（父组件未提供时显示）</slot>
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script setup>
// 组合式 API 写法（<script setup>）
import { ref, onMounted } from 'vue'

// 4. 定义 Props
const props = defineProps({
  title: {
    type: String,
    default: '默认标题'
  },
  count: {
    type: Number,
    required: true
  }
})

// 5. 定义自定义事件
const emit = defineEmits(['update:count'])

// 6. 组件内部状态
const localData = ref('子组件私有数据')

// 7. 方法
const increment = () => {
  emit('update:count', props.count + 1) // 触发更新事件
}

// 8. 生命周期钩子
onMounted(() => {
  console.log('子组件已挂载')
})
</script>

<style scoped>
/* 9. 作用域样式 */
.child {
  border: 1px solid #eee;
  padding: 20px;
}
</style>
```



### 父组件 ParentComponent.vue

```vue
<template>
  <div>
    <h2>父组件</h2>
    
    <!-- 10. 使用子组件 -->
    <ChildComponent 
      :title="childTitle"
      :count="counter" 
      @update:count="handleCountUpdate"
    >
      <!-- 11. 插槽内容 -->
      <template #default>
        <p>这是默认插槽内容</p>
      </template>
      
      <template #footer>
        <button>来自父组件的按钮</button>
      </template>
    </ChildComponent>
    
    <p>父组件计数: {{ counter }}</p>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
// 12. 导入子组件（自动注册）
import ChildComponent from './ChildComponent.vue'

// 13. 响应式数据
const counter = ref(0)
const childTitle = ref('动态标题')

// 14. 处理子组件事件
const handleCountUpdate = (newValue) => {
  counter.value = newValue
}

// 15. 复杂状态管理
const state = reactive({
  message: '父组件状态'
})
</script>
```



### 案例核心知识点详解

1. **Props 传递**
   - 子组件通过 `defineProps` 声明接收数据
   - 父组件通过 `:propName="value"` 传递响应式数据
2. **自定义事件**
   - 子组件通过 `defineEmits` 声明事件
   - 通过 `emit('event', payload)` 触发事件
   - 父组件通过 `@event="handler"` 监听
3. **插槽机制**
   - `<slot>` 定义内容分发位置
   - 具名插槽 `<slot name="footer">` + `<template #footer>`
   - 默认插槽内容作为后备内容
4. **组合式 API**
   - `<script setup>` 语法糖
   - `ref`/`reactive` 创建响应式数据
   - 生命周期钩子直接导入使用（如 `onMounted`）
5. **组件通信模式**
   - Props 向下传递
   - 事件向上传递
   - 使用 `v-model:count` 实现双向绑定（需子组件触发 `update:count` 事件）
6. **样式作用域**
   - `<style scoped>` 自动添加属性选择器实现样式隔离
7. **自动组件注册**
   - `<script setup>` 中导入的组件可直接在模板使用
8. **响应式原理**
   - 基本类型用 `ref`
   - 复杂对象用 `reactive`
   - 模板中自动解包 `.value`





# 生命周期相关

| **生命周期钩子**  | **触发时机**                          | **选项式 API**    | **组合式 API (setup)**     |
| :---------------- | :------------------------------------ | :---------------- | :------------------------- |
| `beforeCreate`    | 实例初始化之后，数据观测之前          | `beforeCreate`    | ❌ 无需使用 (由 setup 替代) |
| `created`         | 实例创建完成，数据观测完成            | `created`         | ❌ 无需使用 (由 setup 替代) |
| `beforeMount`     | 挂载开始之前                          | `beforeMount`     | `onBeforeMount`            |
| `mounted`         | 挂载完成 (DOM 已插入)                 | `mounted`         | `onMounted`                |
| `beforeUpdate`    | 数据更新时，DOM 更新前                | `beforeUpdate`    | `onBeforeUpdate`           |
| `updated`         | 数据更新后，DOM 更新完成              | `updated`         | `onUpdated`                |
| `beforeUnmount`   | 实例销毁前 (Vue 2 的 `beforeDestroy`) | `beforeUnmount`   | `onBeforeUnmount`          |
| `unmounted`       | 实例销毁后 (Vue 2 的 `destroyed`)     | `unmounted`       | `onUnmounted`              |
| `errorCaptured`   | 捕获后代组件错误时                    | `errorCaptured`   | `onErrorCaptured`          |
| `renderTracked`   | 渲染函数依赖被追踪时 (开发模式)       | `renderTracked`   | `onRenderTracked`          |
| `renderTriggered` | 依赖变更触发渲染时 (开发模式)         | `renderTriggered` | `onRenderTriggered`        |
| `activated`       | 被 keep-alive 缓存的组件激活时        | `activated`       | `onActivated`              |
| `deactivated`     | 被 keep-alive 缓存的组件失活时        | `deactivated`     | `onDeactivated`            |



## 具体用法示例

### 选项式 API

```vue
<script>
export default {
  data() {
    return { count: 0 }
  },
  beforeCreate() {
    console.log('组件初始化前')
  },
  created() {
    console.log('组件创建完成，可访问数据')
  },
  beforeMount() {
    console.log('挂载前，DOM 尚未生成')
  },
  mounted() {
    console.log('挂载完成，可访问 DOM')
  },
  beforeUpdate() {
    console.log('数据更新，DOM 更新前')
  },
  updated() {
    console.log('DOM 更新完成')
  },
  beforeUnmount() {
    console.log('组件销毁前')
  },
  unmounted() {
    console.log('组件销毁后')
  }
}
</script>
```



### 组合式 API

```vue
<script setup lang="ts">
import { 
  onBeforeMount, 
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onActivated,
  onDeactivated,
  onErrorCaptured,
  ref
} from 'vue'

const count = ref(0)

// 挂载阶段
onBeforeMount(() => {
  console.log('挂载前')
})

onMounted(() => {
  console.log('挂载完成')
})

// 更新阶段
onBeforeUpdate(() => {
  console.log('更新前')
})

onUpdated(() => {
  console.log('更新完成')
})

// 卸载阶段
onBeforeUnmount(() => {
  console.log('卸载前')
})

onUnmounted(() => {
  console.log('卸载后')
})

// keep-alive 相关
onActivated(() => {
  console.log('被激活')
})

onDeactivated(() => {
  console.log('失活')
})

// 错误捕获
onErrorCaptured((err, instance, info) => {
  console.error('捕获到错误:', err)
  return false // 是否阻止错误继续向上传播
})
</script>
```



### 注意事项

1. **组合式 API 中：**

   - 所有钩子函数都是 **同步注册** 的（在 `setup` 中直接调用）

   - 钩子函数名以 `on` 开头（如 `onMounted`）

   - 没有 `beforeCreate` 和 `created` 钩子，因为 `setup` 本身在这两个阶段之间运行

     ```ts
     setup() {
       // 相当于 created 阶段
       console.log('这里相当于 created')
     }
     ```

     

2. **执行顺序：**

   - 父组件 `beforeCreate` → 父组件 `created` → 父组件 `beforeMount` →
   - 子组件 `beforeCreate` → 子组件 `created` → 子组件 `beforeMount` → 子组件 `mounted` →
   - 父组件 `mounted`

3. **异步请求放在哪里？**

   - 需要访问 DOM → 用 `onMounted`
   - 不需要访问 DOM → 可直接在 `setup` 中调用（或 `onBeforeMount`）

4. **避免在 `updated` 中修改状态**
   可能导致无限更新循环！



### **建议**

- **组合式 API 优先：** 更灵活的逻辑组织方式

- **清理副作用：** 在 `onUnmounted` 中清除定时器、事件监听等

  ```ts
  onMounted(() => {
    const timer = setInterval(() => { /* ... */ }, 1000)
    onUnmounted(() => clearInterval(timer))
  })
  ```

  

- **逻辑复用：** 将生命周期逻辑封装到自定义 Hook 中

  ```ts
  // useLifecycleLogger.ts
  export function useLifecycleLogger(name: string) {
    onMounted(() => console.log(`${name} mounted`))
    onUnmounted(() => console.log(`${name} unmounted`))
  }
  
  // 在组件中使用
  useLifecycleLogger('MyComponent')
  ```





# 可能会遇到的问题

## `@`路径配置

- vite的vue-ts模板会出现没有配置 `@` 路径的问题，在 `vite.config.ts`中使用如下配置。
- 配置时，可能会出现`path` 找不到的情况，是因为在浏览器环境中没有，执行 `npm install @types/node --save-dev` 安装一下就可以了。

```ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path'; // 引入 path 模块

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src') // 配置 @ 指向 src 目录
    }
  }
});
```



## “@/views/HomeView.vue”报错（找不到模块）

出现这个问题的原因大概就是，ts只支持导出导入模块，但是vue不是模块，我们需要申明一下vue是个模块，ts可以导入。在 `vite-env.d.ts` / `env.d.ts` 中使用如下配置，如没有该文件则新建一个。

```ts
declare module '*.vue' {
   import type { DefineComponent } from 'vue'
   const component: DefineComponent<{}, {}, any>
   export default component
}

// 或者

declare module '*.vue' {
   import type { DefineComponent } from 'vue'
   const component: ComponentOptions | ComponentOptions['setup']
   export default component
}
```



## Pinia store 未初始化

确保在 `app.use(router)` 之前使用 `app.use(pinia)`



## 路由跳转时类型错误

```ts
// 使用类型安全的跳转
import { useRouter } from 'vue-router'
const router = useRouter()

// 命名路由跳转
router.push({ name: 'About' })

// 带参数跳转
router.push({ 
  name: 'UserProfile', 
  params: { id: 123 } 
})
```



## 组件内访问 $route 和 $router

```vue
<script setup lang="ts">
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

console.log(route.params.id)
router.push('/home')
</script>
```



## 在 store 中使用 router

避免在 store 中直接导入 router,更好的方式是在组件中调用 action

```ts
// store 中
actions: {
  async login(credentials) {
    // 登录逻辑...
    this.isAuthenticated = true
  }
}

// 组件中
const handleLogin = async () => {
  await authStore.login(credentials)
  router.push('/dashboard')
}
```

