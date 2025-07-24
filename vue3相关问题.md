

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





# Axios相关

## 封装axios

### 1. 安装依赖

```bash
npm install axios
```



### 2. 创建 Axios 封装文件

`src/utils/http.ts`（具体路径可自定义）

```ts
import axios from "axios";
import type {
  AxiosInstance,
  AxiosRequestConfig,
  AxiosResponse,
  InternalAxiosRequestConfig,
} from "axios";

// 定义响应数据结构 (根据后端接口调整)
interface ApiResponse<T = any> {
  code: number
  data: T
  message: string
}

class Http {
  private instance: AxiosInstance
  private readonly baseURL: string

  constructor() {
    // 从环境变量获取
    // 需要带上前缀http/https
    this.baseURL = import.meta.env.VITE_API_BASE_URL 
      
    this.instance = axios.create({
      baseURL: this.baseURL,
      timeout: 10000,
      headers: { 'Content-Type': 'application/json' }
    })

    this.setupInterceptors()
  }

  // 设置拦截器
  private setupInterceptors() {
    // 请求拦截器
    this.instance.interceptors.request.use(
      (config: InternalAxiosRequestConfig) => {
        // 可在此添加 token
        const token = localStorage.getItem('token')
        if (token) {
          config.headers.Authorization = `Bearer ${token}`
        }
        return config
      },
      (error: any) => {
        return Promise.reject(error)
      }
    )

    // 响应拦截器
    this.instance.interceptors.response.use(
      (response: AxiosResponse) => {
        // 处理响应数据
        const res = response.data
        if (res.code !== 200) {
          // 统一处理业务错误
          console.error(`API Error: ${res.message}`)
          return Promise.reject(new Error(res.message || 'Error'))
        }
        return res
      },
      (error: any) => {
        // 处理 HTTP 错误
        const message = error.response?.data?.message || error.message
        console.error(`HTTP Error: ${message}`)
        return Promise.reject(error)
      }
    )
  }

  // 封装请求方法
  public request<T = any>(config: AxiosRequestConfig): Promise<ApiResponse<T>> {
    return this.instance.request(config)
  }

  public get<T = any>(url: string, params?: any): Promise<ApiResponse<T>> {
    return this.request({ method: 'GET', url, params })
  }

  public post<T = any>(url: string, data?: any): Promise<ApiResponse<T>> {
    return this.request({ method: 'POST', url, data })
  }

  public put<T = any>(url: string, data?: any): Promise<ApiResponse<T>> {
    return this.request({ method: 'PUT', url, data })
  }

  public delete<T = any>(url: string, params?: any): Promise<ApiResponse<T>> {
    return this.request({ method: 'DELETE', url, params })
  }
}

const http = new Http()
export default http
```



### 3. 配置环境变量

`.env.development`

```ini
VITE_API_BASE_URL = http://localhost:3000/api
```



`.env.production`

```ini
VITE_API_BASE_URL = https://production-domain.com/api
```



### 4. 创建 API 模块统一管理接口

`src/api/user.ts`（具体路径可自定义）

```ts
import http from '@/utils/http'

// 定义用户数据类型
interface User {
  id: number
  name: string
  email: string
}

// 登录参数类型
interface LoginParams {
  username: string
  password: string
}

// 登录响应类型
interface LoginResponse {
  token: string
  user: User
}

export const userAPI = {
  // 用户登录
  login: (data: LoginParams) => {
    return http.post<LoginResponse>('/user/login', data)
  },

  // 获取用户信息
  getUserInfo: (id: number) => {
    return http.get<User>(`/user/${id}`)
  },

  // 更新用户信息
  updateUser: (id: number, data: Partial<User>) => {
    return http.put<User>(`/user/${id}`, data)
  }
}
```



### 6. 全局挂载 (可选)

在`src/main.ts`中

```ts
import { createApp } from 'vue'
import App from './App.vue'
import http from '@/utils/http'

const app = createApp(App)

// 挂载到全局属性
app.config.globalProperties.$http = http

app.mount('#app')
```



### 高级功能扩展

1. **取消请求**：

   ```ts
   import { CancelTokenSource } from 'axios'
   
   const source = CancelToken.source()
   
   http.get('/data', {
     cancelToken: source.token
   })
   
   // 取消请求
   source.cancel('Operation canceled by user')
   ```

2. **请求重试**：在响应拦截器中添加重试逻辑

3. **缓存机制**：实现 GET 请求的缓存功能

4. **文件上传**：添加专门的 upload 方法处理文件上传

5. **Mock 数据**：开发环境下集成 mock 服务



## Axios跨域问题（前端）

**使用开发服务器代理（Vue/React 通用），前端开发服务器充当中间层，将请求转发到目标服务器。**

**该方法==仅在前端开发环境时可用==，生存环境生产环境需使用 Nginx 或后端配置 CORS**



### 1. 配置代理规则

`vue.config.js` 中：

```javascript
module.exports = {
  devServer: {
    proxy: {
      '/api': {  // 匹配所有以 /api 开头的请求
        target: 'http://your-backend.com',  // 目标服务器地址
        changeOrigin: true,                // 修改请求头中的 Host
        pathRewrite: {
          '^/api': ''                      // 重写路径（去掉 /api 前缀）
        }
      }
    }
  }
}
```

`vite.config.ts` 中：

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    proxy: {
      // 代理规则1：简单写法
      '/api': {
        target: 'http://your-backend.com',  // 后端真实地址
        changeOrigin: true,                // 启用跨域
        rewrite: (path) => path.replace(/^\/api/, '') // 重写路径（可选）
      },	
      
      // 代理规则2：更复杂的配置
      '/graphql': {
        target: 'http://api.example.com',
        changeOrigin: true,
        secure: false,  // 如果是https需要验证证书，开发环境可关闭
        ws: true        // 代理WebSockets
      },
      
      // 代理规则3：代理特定路径
      '/uploads': 'http://cdn.example.com' // 简写形式
    }
  }
})
```



### 2. 在 Axios 请求中使用代理路径

```javascript
// 所有以 /api 开头的请求都会被代理
axios.get('/api/users') // → 实际请求 http://your-backend.com/users

// 或者在封装时使用
const service = axios.create({
  baseURL: '/api', // 使用代理前缀（与vite配置中的路径一致）
  timeout: 5000
})
```



### 3. 多个后端服务的配置

```ts
export default defineConfig({
  server: {
    proxy: {
      // 主API服务
      '/api': {
        target: 'http://api.main.com',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      },
      
      // 支付服务
      '/pay': {
        target: 'http://api.payment.com',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/pay/, '/v1/payment')
      },
      
      // 文件服务
      '/files': 'http://cdn.example.com'
    }
  }
})
```



### 4. 高级配置选项

如果需要更细粒度的控制

```ts
proxy: {
  '/api': {
    target: 'http://your-backend.com',
    changeOrigin: true,
    configure: (proxy, options) => {
      // 访问代理前的钩子
      proxy.on('proxyReq', (proxyReq, req, res) => {
        console.log(`请求代理到: ${req.url}`)
      })
      
      // 错误处理
      proxy.on('error', (err, req, res) => {
        res.writeHead(500, {'Content-Type': 'text/plain'})
        res.end('代理错误: ' + err.message)
      })
    }
  }
}
```



### 5. 环境变量配置

结合环境变量使用更安全

```ini
# .env.development
VITE_API_BASE=/api
VITE_API_TARGET=http://dev-api.example.com
```

```ts
// vite.config.ts
export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())
  
  return {
    server: {
      proxy: {
        [env.VITE_API_BASE]: {
          target: env.VITE_API_TARGET,
          changeOrigin: true,
          rewrite: path => path.replace(new RegExp(`^${env.VITE_API_BASE}`), '')
        }
      }
    }
  }
})
```



### 注意事项

1. **仅开发环境有效**：代理只在 `vite dev` 命令下生效，生产环境需要配置 Nginx 或后端解决跨域

2. **路径匹配**：确保请求路径前缀与代理配置一致

3. **重启服务**：修改 vite.config.js 后需要重启开发服务器

4. **携带凭证**：如果需要发送 cookies

   ```ts
   // axios配置
   axios.defaults.withCredentials = true
   
   // 后端需要配置
   Access-Control-Allow-Credentials: true
   Access-Control-Allow-Origin: http://localhost:5173 (不能是 *)
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



# 响应式相关

## `ref` 和 `reactive`

### `ref`

- 接受一个内部值，==返回一个响应式的、可更改的 ref 对象==，此对象只有一个指向其内部值的属性 `.value`。
- ref 对象是可更改的，也就是说你可以为 `.value` 赋予新的值。它也是响应式的，即所有对 `.value` 的操作都将被追踪，并且写操作会触发与之相关的副作用。
- ==如果将一个对象赋值给 ref，那么这个对象将通过 [reactive()](https://cn.vuejs.org/api/reactivity-core.html#reactive) 转为具有深层次响应式的对象。==这也意味着如果对象中包含了嵌套的 ref，它们将被深层地解包。
- 若要避免这种深层次的转换，请使用 [`shallowRef()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowref) 来替代。

```ts
const count = ref(0)
console.log(count.value) // 0

count.value = 1
console.log(count.value) // 1
```



### `reactive`

- 返回一个对象的响应式代理。

- 响应式转换是“深层”的：它会影响到所有嵌套的属性。一个响应式对象也将深层地解包任何 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref) 属性，同时保持响应性。

- 值得注意的是，**当访问到某个响应式数组或 `Map` 这样的原生集合类型中的 ref 元素时，不会执行 ref 的解包**。

  ```ts
  const books = reactive([ref('Vue 3 Guide')])
  // 这里需要 .value
  console.log(books[0].value)
  
  const map = reactive(new Map([['count', ref(0)]]))
  // 这里需要 .value
  console.log(map.get('count').value)
  ```

  

- 若要避免深层响应式转换，只想保留对这个对象顶层次访问的响应性，请使用 [shallowReactive()](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreactive) 作替代。

- 返回的对象以及其中嵌套的对象都会通过 [ES Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 包裹，因此**不等于**源对象，建议只使用响应式代理，避免使用原始对象。



### `ref` 与 `reactive` 对比

| 特性               | `reactive`                       | `ref`                                |
| :----------------- | :------------------------------- | :----------------------------------- |
| **适用数据类型**   | 只接受 **对象/数组**             | 接受 **任何类型** (基本类型+对象)    |
| **返回值**         | 原始对象的 Proxy 代理对象        | 包含 `.value` 属性的 ref 对象        |
| **访问方式**       | 直接访问属性 (`obj.key`)         | 需要通过 `.value` 访问 (`val.value`) |
| **模板中使用**     | 需要保持对象引用 (`state.count`) | 自动解包 (模板中直接 `{{ count }}`)  |
| **响应性丢失风险** | 解构/展开时易丢失响应性          | 通过 `.value` 始终保持响应引用       |
| **替换整个对象**   | 直接替换会 **丢失响应性**        | 替换 `.value` **保持响应性**         |



### 将变量恢复到初始状态

看下面的文章

https://blog.csdn.net/weixin_42960907/article/details/138950507



## `computed` 和 `watch`

### `computed` 与 `watch` 对比

| 特性          | `computed`                  | `watch`              |
| :------------ | :-------------------------- | :------------------- |
| **目的**      | 声明式派生值                | 响应式副作用         |
| **返回值**    | 响应式引用 (Ref)            | 无 (用于执行操作)    |
| **缓存机制**  | ✅ 有缓存 (依赖不变时复用值) | ❌ 无缓存             |
| **执行时机**  | 惰性计算 (访问时才计算)     | 立即执行或依赖变化时 |
| **依赖追踪**  | 自动追踪所有依赖            | 显式指定监听源       |
| **同步/异步** | 必须同步返回                | 可处理异步操作       |
| **使用场景**  | 派生数据/模板计算           | 执行副作用/响应变化  |





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



## Vue3小问题合辑

https://blog.csdn.net/zhuangvi/article/details/142448741



## 环境变量配置相关（.env）

### 不同的环境文件

在项目根目录（与 `package.json` 同级）创建以下文件：

| 文件名             | 用途                            |
| :----------------- | :------------------------------ |
| `.env`             | 所有环境共享的默认变量          |
| `.env.development` | 开发环境专用变量 (本地运行)     |
| `.env.production`  | 生产环境专用变量 (构建生产版本) |
| `.env.staging`     | 预发布环境变量 (可选)           |



### 环境文件中添加变量

**示例内容 (.env.development):**

```ini
# 本地开发环境 API 地址
VITE_API_BASE_URL = "http://localhost:3000/api"
```

**示例内容 (.env.production):**

```ini
# 生产环境 API 地址
VITE_API_BASE_URL = "https://api.yourdomain.com/v1"
```



### 使用环境变量

代码中可以直接使用：

```js
const baseURL = import.meta.env.VITE_API_BASE_URL;
```



### 添加 TypeScript 类型支持 (可选但推荐)

创建 `src/env.d.ts` 文件：

```typescript
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string;
  // 添加其他环境变量...
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```



### 配置不同环境的启动命令

`package.json` 中：

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "build:staging": "vite build --mode staging",
    "preview": "vite preview"
  }
}
```



### 使用不同环境变量

```bash
# 使用 .env.development 中的变量
npm run dev

# 使用 .env.production 中的变量
npm run build

# 使用 .env.staging 中的变量
npm run build:staging
```



### 安全注意事项

1. **不要提交敏感信息**，只提交 `.env.example` 作为模板，在 `.gitignore` 中添加

   ```text
   .env
   .env.local
   *.env
   ```

2. 示例文件 `.env.example`

   ```ini
   # API 基础地址
   VITE_API_BASE_URL = "https://api.example.com/v1"
   ```



### 高级配置 (vite.config.js)

```js
export default ({ mode }) => {
  // 根据环境加载不同配置
  const env = loadEnv(mode, process.cwd(), "");
  
  return {
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV),
    },
    // 其他配置...
  };
};
```



### 在 HTML 中使用环境变量

```html
<title>%VITE_APP_NAME%</title>
```



