# 前端技术栈详细说明

## 核心技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| Vue | 3.x | 渐进式JavaScript框架 |
| TypeScript | 5.x | JavaScript的超集，提供类型系统 |
| Vite | 4.x | 前端构建工具 |
| Element Plus | 2.x | 基于Vue 3的UI组件库 |
| Pinia | 2.x | Vue的状态管理库 |
| Vue Router | 4.x | Vue的官方路由 |
| Axios | 1.x | 基于Promise的HTTP客户端 |

## 项目依赖详解

### 1. 开发环境

```json
{
  "devDependencies": {
    "@iconify/vue": "^4.1.1",
    "@types/lodash-es": "^4.17.7",
    "@types/nprogress": "^0.2.0",
    "@vitejs/plugin-vue": "^4.2.3",
    "eslint": "^8.43.0",
    "prettier": "^2.8.8",
    "sass": "^1.63.6",
    "typescript": "~5.0.4",
    "unplugin-auto-import": "^0.16.4",
    "unplugin-vue-components": "^0.25.1",
    "vite": "^4.3.9",
    "vite-plugin-svg-icons": "^2.0.1"
  }
}
```

### 2. 生产依赖

```json
{
  "dependencies": {
    "axios": "^1.4.0",
    "element-plus": "^2.3.7",
    "lodash-es": "^4.17.21",
    "nprogress": "^0.2.0",
    "pinia": "^2.1.4",
    "vue": "^3.3.4",
    "vue-i18n": "^9.2.2",
    "vue-router": "^4.2.2"
  }
}
```

## 技术栈详解

### 1. Vue 3

Vue 3是一个渐进式JavaScript框架，采用了Composition API，相比Vue 2有以下优势：

- **性能提升**：更小的体积、更快的渲染速度
- **Composition API**：更好的逻辑复用和代码组织
- **更好的TypeScript支持**：内置类型定义
- **片段(Fragments)**：组件可以有多个根节点
- **Teleport组件**：将内容渲染到DOM的其他部分

#### 项目中的应用

项目中广泛采用了Composition API和`<script setup>`语法，实现了更简洁、更高效的组件编写方式。

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 计算属性
const doubleCount = computed(() => count.value * 2)

// 生命周期钩子
onMounted(() => {
  console.log('Component mounted')
})

// 方法
const increment = () => {
  count.value++
}
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double Count: {{ doubleCount }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>
```

### 2. TypeScript

TypeScript为JavaScript添加了静态类型系统，提供了更好的开发体验和代码可靠性。

#### 项目中的应用

项目中广泛使用TypeScript：

- 定义接口和类型
- 类型推断和检查
- 泛型编程
- 类型声明文件

```typescript
// 定义接口
interface User {
  id: number
  username: string
  email: string
  roles: Role[]
}

// 定义类型
type Role = 'admin' | 'editor' | 'visitor'

// 使用泛型
function fetchData<T>(url: string): Promise<T> {
  return fetch(url).then(res => res.json())
}

// 使用类型
const getUser = () => fetchData<User>('/api/user')
```

### 3. Element Plus

Element Plus是基于Vue 3的UI组件库，提供了丰富的组件和主题定制能力。

#### 项目中的应用

项目使用了Element Plus的按需引入方式，减小了打包体积：

```typescript
// plugins/element-plus/index.ts
import { ElButton, ElTable, ElForm } from 'element-plus'

export default {
  install(app) {
    // 全局注册组件
    app.use(ElButton)
    app.use(ElTable)
    app.use(ElForm)
  }
}
```

自定义主题配置：

```scss
// styles/element/index.scss
@forward 'element-plus/theme-chalk/src/common/var.scss' with (
  $colors: (
    'primary': (
      'base': #409eff,
    ),
    'success': (
      'base': #67c23a,
    ),
    'warning': (
      'base': #e6a23c,
    ),
    'danger': (
      'base': #f56c6c,
    ),
  )
);
```

### 4. Pinia

Pinia是Vue官方推荐的状态管理库，相比Vuex更加轻量和TypeScript友好。

#### 项目中的应用

项目使用模块化的Pinia store：

```typescript
// store/modules/user.ts
import { defineStore } from 'pinia'
import { login, logout, getUserInfo } from '@/api/auth'
import { UserInfo } from '@/types'

export const useUserStore = defineStore('user', {
  state: () => ({
    token: localStorage.getItem('token') || '',
    userInfo: {} as UserInfo,
  }),
  
  getters: {
    hasToken: state => !!state.token,
    roles: state => state.userInfo.roles || [],
  },
  
  actions: {
    async login(username: string, password: string) {
      const { data } = await login(username, password)
      this.token = data.token
      localStorage.setItem('token', data.token)
    },
    
    async getUserInfo() {
      const { data } = await getUserInfo()
      this.userInfo = data
      return data
    },
    
    async logout() {
      await logout()
      this.token = ''
      this.userInfo = {} as UserInfo
      localStorage.removeItem('token')
    }
  }
})
```

### 5. Vite

Vite是一个现代化的前端构建工具，它的特点是快速的冷启动和热更新。

#### 项目中的应用

项目使用了Vite的多种插件和配置优化：

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia'],
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
    createSvgIconsPlugin({
      iconDirs: [path.resolve(process.cwd(), 'src/assets/icons')],
      symbolId: 'icon-[dir]-[name]',
    }),
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
  build: {
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
})
```

## 开发规范

### 1. 组件开发规范

- 使用SFC (Single File Component) 文件格式
- 采用`<script setup>`语法
- 组件名使用PascalCase命名
- Props使用小驼峰命名，并定义类型和默认值
- 事件名使用kebab-case命名

### 2. 类型定义规范

- 接口名使用大驼峰命名，并以I开头
- 类型别名使用大驼峰命名，并以T开头
- 枚举使用大驼峰命名，并以E开头
- 函数类型使用大驼峰命名，并以F开头

### 3. 样式规范

- 使用SCSS预处理器
- 采用BEM命名规范
- 组件样式使用scoped属性隔离
- 全局样式统一管理在styles目录下

### 4. 代码质量规范

- 使用ESLint进行代码检查
- 使用Prettier进行代码格式化
- 提交前运行lint检查
- 单元测试覆盖核心功能 