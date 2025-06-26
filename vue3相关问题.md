## Pinia相关

### 安装

```bash
npm install pinia
```



### 配置Pinia

#### 创建主 store 文件

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



#### 在 main.ts 中安装 Pinia

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



## Vue Router相关

### 安装

```bash
npm install vue-router@4
```



### 配置Vue Router

#### 创建路由配置文件

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



#### 在 main.ts 中安装 Vue Router

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



### 可能会遇到的问题

#### `@`路径配置

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



#### “@/views/HomeView.vue”报错（找不到模块）

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



