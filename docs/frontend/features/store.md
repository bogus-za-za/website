# 前端状态管理说明

## 状态管理概述

本项目使用 Pinia 作为状态管理库，相比 Vuex，Pinia 具有以下优势：

- 更简洁的 API：无需创建 mutations
- 完整的 TypeScript 支持
- 去除了 modules 的概念，每个 store 都是独立的
- 更好的开发体验和性能

## Pinia 基本结构

### 1. Store 定义

每个 Store 都是通过 `defineStore` 函数创建：

```typescript
// src/store/modules/user.ts
import { defineStore } from 'pinia'
import { login, logout, getUserInfo } from '@/api/auth'
import { setToken, getToken, removeToken } from '@/utils/auth'

export const useUserStore = defineStore('user', {
  // 状态
  state: () => ({
    token: getToken() || '',
    userInfo: {},
    roles: [],
    permissions: []
  }),
  
  // 计算属性
  getters: {
    // 是否有token
    hasToken(): boolean {
      return !!this.token
    },
    
    // 用户名
    username(): string {
      return this.userInfo?.username || ''
    },
    
    // 是否为管理员
    isAdmin(): boolean {
      return this.roles.includes('ADMIN')
    }
  },
  
  // 操作方法
  actions: {
    // 设置Token
    setToken(token: string) {
      this.token = token
      setToken(token)
    },
    
    // 设置用户信息
    setUserInfo(userInfo: any) {
      this.userInfo = userInfo
    },
    
    // 设置角色
    setRoles(roles: string[]) {
      this.roles = roles
    },
    
    // 设置权限
    setPermissions(permissions: string[]) {
      this.permissions = permissions
    },
    
    // 登录
    async login(username: string, password: string) {
      try {
        const { data } = await login(username, password)
        this.setToken(data.token)
        return data
      } catch (error) {
        throw error
      }
    },
    
    // 获取用户信息
    async getUserInfo() {
      try {
        const { data } = await getUserInfo()
        
        if (!data) {
          throw new Error('获取用户信息失败，请重新登录')
        }
        
        const { roles, permissions, ...userInfo } = data
        
        // 验证返回的roles是否是一个非空数组
        if (!roles || roles.length <= 0) {
          throw new Error('用户角色不能为空')
        }
        
        this.setUserInfo(userInfo)
        this.setRoles(roles)
        this.setPermissions(permissions)
        
        return data
      } catch (error) {
        throw error
      }
    },
    
    // 注销
    async logout() {
      try {
        await logout()
      } catch (error) {
        console.error('登出失败', error)
      } finally {
        this.resetToken()
      }
    },
    
    // 重置Token
    resetToken() {
      this.token = ''
      this.userInfo = {} as UserInfo
      this.roles = []
      this.permissions = []
      removeToken()
    }
  }
})
```

### 2. Store 目录结构

```
src/store/
├── index.ts                # 主入口，初始化Pinia
├── modules/               # 模块目录
│   ├── app.ts             # 应用配置相关状态
│   ├── permission.ts      # 权限相关状态
│   ├── user.ts            # 用户相关状态
│   ├── tagsView.ts        # 标签页相关状态
│   └── settings.ts        # 系统设置相关状态
└── types.ts               # 类型定义
```

### 3. 初始化Pinia

```typescript
// src/store/index.ts
import { createPinia } from 'pinia'
import type { App } from 'vue'

// 创建pinia实例
const pinia = createPinia()

// 安装pinia
export function setupStore(app: App<Element>) {
  app.use(pinia)
}

// 导出store实例，便于在非组件中使用
export { pinia }

// 重新导出模块，方便使用
export * from './modules/app'
export * from './modules/user'
export * from './modules/permission'
export * from './modules/tagsView'
export * from './modules/settings'
```

## 主要Store模块说明

### 1. User Store

用户状态管理，主要包括：

- 用户登录、注销
- 用户信息获取
- 用户权限和角色管理

```typescript
import { defineStore } from 'pinia'
import { login, logout, getUserInfo } from '@/api/auth'
import { setToken, getToken, removeToken } from '@/utils/auth'
import { UserInfo } from '@/types'

export const useUserStore = defineStore('user', {
  state: () => ({
    token: getToken() || '',
    userInfo: {} as UserInfo,
    roles: [] as string[],
    permissions: [] as string[]
  }),
  
  getters: {
    hasToken: state => !!state.token,
    username: state => state.userInfo?.username || '',
    nickname: state => state.userInfo?.nickname || '',
    avatar: state => state.userInfo?.avatar || '',
    hasRoles: state => state.roles && state.roles.length > 0
  },
  
  actions: {
    // 登录
    async login(username: string, password: string) {
      try {
        const { data } = await login(username, password)
        this.token = data.token
        setToken(data.token)
        return data
      } catch (error) {
        throw error
      }
    },
    
    // 获取用户信息
    async getUserInfo() {
      try {
        const { data } = await getUserInfo()
        
        if (!data) {
          throw new Error('获取用户信息失败，请重新登录')
        }
        
        const { roles, permissions, ...userInfo } = data
        
        // 验证返回的roles是否是一个非空数组
        if (!roles || roles.length <= 0) {
          throw new Error('用户角色不能为空')
        }
        
        this.userInfo = userInfo
        this.roles = roles
        this.permissions = permissions
        
        return data
      } catch (error) {
        throw error
      }
    },
    
    // 注销
    async logout() {
      try {
        await logout()
      } catch (error) {
        console.error('登出失败', error)
      } finally {
        this.resetToken()
      }
    },
    
    // 重置Token
    resetToken() {
      this.token = ''
      this.userInfo = {} as UserInfo
      this.roles = []
      this.permissions = []
      removeToken()
    }
  }
})
```

### 2. Permission Store

权限状态管理，主要包括：

- 路由权限控制
- 动态路由生成

```typescript
import { defineStore } from 'pinia'
import { RouteRecordRaw } from 'vue-router'
import { constantRoutes } from '@/router'
import { getUserMenus } from '@/api/menu'

// 动态路由模块
import systemRoutes from '@/router/modules/system'
import monitorRoutes from '@/router/modules/monitor'
import toolRoutes from '@/router/modules/tool'

// 路由映射表
const moduleNameMap = {
  'system': systemRoutes,
  'monitor': monitorRoutes,
  'tool': toolRoutes
}

export const usePermissionStore = defineStore('permission', {
  state: () => ({
    routes: [] as RouteRecordRaw[],
    addRoutes: [] as RouteRecordRaw[]
  }),
  
  getters: {
    // 获取路由
    getRoutes: state => state.routes
  },
  
  actions: {
    // 生成路由
    async generateRoutes() {
      // 获取用户菜单
      const { data } = await getUserMenus()
      const menuList = data || []
      
      // 根据后端返回的菜单生成路由
      const accessedRoutes = this.filterAsyncRoutes(menuList)
      
      // 添加404路由放在最后
      accessedRoutes.push({ 
        path: '/:pathMatch(.*)*', 
        redirect: '/404', 
        meta: { hidden: true } 
      })
      
      this.addRoutes = accessedRoutes
      this.routes = constantRoutes.concat(accessedRoutes)
      
      return accessedRoutes
    },
    
    // 过滤路由
    filterAsyncRoutes(menuList) {
      const routes = []
      
      // 遍历菜单，生成路由
      menuList.forEach(menu => {
        // 根据菜单信息匹配路由模块
        const routeModule = moduleNameMap[menu.path]
        if (routeModule) {
          routes.push(routeModule)
        }
      })
      
      return routes
    }
  }
})
```

### 3. App Store

应用全局状态管理，主要包括：

- 侧边栏状态
- 设备类型
- 语言设置

```typescript
import { defineStore } from 'pinia'
import Cookies from 'js-cookie'

export const useAppStore = defineStore('app', {
  state: () => ({
    sidebar: {
      opened: Cookies.get('sidebarStatus') !== '0',
      withoutAnimation: false
    },
    device: 'desktop',
    language: Cookies.get('language') || 'zh-CN',
    size: Cookies.get('size') || 'default'
  }),
  
  actions: {
    // 切换侧边栏
    toggleSidebar() {
      this.sidebar.opened = !this.sidebar.opened
      this.sidebar.withoutAnimation = false
      if (this.sidebar.opened) {
        Cookies.set('sidebarStatus', '1')
      } else {
        Cookies.set('sidebarStatus', '0')
      }
    },
    
    // 关闭侧边栏
    closeSidebar(withoutAnimation: boolean) {
      Cookies.set('sidebarStatus', '0')
      this.sidebar.opened = false
      this.sidebar.withoutAnimation = withoutAnimation
    },
    
    // 切换设备
    toggleDevice(device: string) {
      this.device = device
    },
    
    // 设置语言
    setLanguage(language: string) {
      this.language = language
      Cookies.set('language', language)
    },
    
    // 设置大小
    setSize(size: string) {
      this.size = size
      Cookies.set('size', size)
    }
  }
})
```

### 4. TagsView Store

标签页导航状态管理：

```typescript
import { defineStore } from 'pinia'
import { RouteLocationNormalized } from 'vue-router'

export const useTagsViewStore = defineStore('tagsView', {
  state: () => ({
    visitedViews: [] as RouteLocationNormalized[],
    cachedViews: [] as string[]
  }),
  
  actions: {
    // 添加已访问视图
    addVisitedView(view: RouteLocationNormalized) {
      if (this.visitedViews.some(v => v.path === view.path)) return
      
      this.visitedViews.push(
        Object.assign({}, view, {
          title: view.meta?.title || 'no-name'
        })
      )
    },
    
    // 添加缓存视图
    addCachedView(view: RouteLocationNormalized) {
      if (this.cachedViews.includes(view.name as string)) return
      
      if (!view.meta?.noCache) {
        this.cachedViews.push(view.name as string)
      }
    },
    
    // 添加视图
    addView(view: RouteLocationNormalized) {
      this.addVisitedView(view)
      this.addCachedView(view)
    },
    
    // 删除已访问视图
    delVisitedView(view: RouteLocationNormalized) {
      for (const [i, v] of this.visitedViews.entries()) {
        if (v.path === view.path) {
          this.visitedViews.splice(i, 1)
          break
        }
      }
    },
    
    // 删除缓存视图
    delCachedView(view: RouteLocationNormalized) {
      const index = this.cachedViews.indexOf(view.name as string)
      index > -1 && this.cachedViews.splice(index, 1)
    },
    
    // 删除视图
    delView(view: RouteLocationNormalized) {
      this.delVisitedView(view)
      this.delCachedView(view)
    },
    
    // 删除其他视图
    delOthersViews(view: RouteLocationNormalized) {
      this.visitedViews = this.visitedViews.filter(v => {
        return v.meta?.affix || v.path === view.path
      })
      
      const index = this.cachedViews.indexOf(view.name as string)
      if (index > -1) {
        this.cachedViews = this.cachedViews.slice(index, index + 1)
      } else {
        this.cachedViews = []
      }
    },
    
    // 删除所有视图
    delAllViews() {
      // 保留固定的标签
      this.visitedViews = this.visitedViews.filter(tag => tag.meta?.affix)
      this.cachedViews = []
    }
  }
})
```

## 状态管理最佳实践

### 1. Store组织原则

- 按功能模块拆分Store
- 保持Store的单一职责
- 避免状态冗余，减少不必要的状态

### 2. 在组件中使用Store

```vue
<script setup lang="ts">
import { computed } from 'vue'
import { useUserStore } from '@/store'

// 获取用户store
const userStore = useUserStore()

// 使用计算属性获取状态
const username = computed(() => userStore.username)
const avatar = computed(() => userStore.avatar)

// 使用actions修改状态
const handleLogout = async () => {
  await userStore.logout()
  router.push('/login')
}
</script>

<template>
  <div>
    <div>用户名: {{ username }}</div>
    <el-avatar :src="avatar" />
    <el-button @click="handleLogout">退出登录</el-button>
  </div>
</template>
```

### 3. 在非组件中使用Store

```typescript
// 在API拦截器中使用store
import axios from 'axios'
import { pinia } from '@/store'
import { useUserStore } from '@/store'

// 创建axios实例
const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000
})

// 请求拦截器
service.interceptors.request.use(
  config => {
    // 在这里使用store
    const userStore = useUserStore(pinia)
    
    if (userStore.token) {
      config.headers['Authorization'] = `Bearer ${userStore.token}`
    }
    
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(
  response => {
    return response.data
  },
  error => {
    // 处理401错误
    if (error.response && error.response.status === 401) {
      const userStore = useUserStore(pinia)
      userStore.resetToken()
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default service
```

### 4. 持久化状态

对需要持久化的状态，可以使用本地存储：

```typescript
import { defineStore } from 'pinia'

export const useSettingsStore = defineStore('settings', {
  state: () => ({
    theme: localStorage.getItem('theme') || 'light',
    layout: localStorage.getItem('layout') || 'sidebar'
  }),
  
  actions: {
    // 设置主题
    setTheme(theme: string) {
      this.theme = theme
      localStorage.setItem('theme', theme)
    },
    
    // 设置布局
    setLayout(layout: string) {
      this.layout = layout
      localStorage.setItem('layout', layout)
    }
  }
})
```

也可以使用Pinia插件实现自动持久化：

```typescript
// src/store/plugins/persist.ts
import { PiniaPluginContext } from 'pinia'

// 持久化配置
type PersistOptions = {
  enabled: boolean
  strategies?: PersistStrategy[]
}

type PersistStrategy = {
  key?: string
  storage?: Storage
  paths?: string[]
}

// 默认配置
const defaultStrategies: PersistStrategy[] = [
  {
    key: 'pinia',
    storage: localStorage,
    paths: ['user.token', 'app.language', 'settings.theme']
  }
]

// 持久化插件
export const persist = ({ options, store }: PiniaPluginContext) => {
  // 获取当前store的持久化配置
  const persistOptions: PersistOptions = options.persist || {}
  
  if (!persistOptions.enabled) {
    return
  }
  
  // 使用默认配置
  const strategies = persistOptions.strategies || defaultStrategies
  
  // 恢复状态
  strategies.forEach(strategy => {
    const storage = strategy.storage || localStorage
    const key = strategy.key || store.$id
    const storageData = storage.getItem(key)
    
    if (storageData) {
      const data = JSON.parse(storageData)
      
      // 只恢复paths中的状态
      if (strategy.paths) {
        strategy.paths.forEach(path => {
          const pathParts = path.split('.')
          let value = data
          
          // 深度获取值
          for (const part of pathParts) {
            if (value) {
              value = value[part]
            }
          }
          
          // 设置值
          if (value !== undefined) {
            // 深度设置值
            let storeValue = store
            for (let i = 0; i < pathParts.length - 1; i++) {
              const part = pathParts[i]
              if (!storeValue[part]) {
                storeValue[part] = {}
              }
              storeValue = storeValue[part]
            }
            storeValue[pathParts[pathParts.length - 1]] = value
          }
        })
      } else {
        // 恢复所有状态
        store.$patch(data)
      }
    }
  })
  
  // 监听状态变化
  store.$subscribe(
    (mutation, state) => {
      strategies.forEach(strategy => {
        const storage = strategy.storage || localStorage
        const key = strategy.key || store.$id
        const paths = strategy.paths
        
        // 只保存paths中的状态
        if (paths) {
          const partialState = paths.reduce((result, path) => {
            const pathParts = path.split('.')
            let value = state
            
            // 深度获取值
            for (const part of pathParts) {
              if (value) {
                value = value[part]
              }
            }
            
            // 设置值
            if (value !== undefined) {
              // 深度设置值
              let resultValue = result
              for (let i = 0; i < pathParts.length - 1; i++) {
                const part = pathParts[i]
                if (!resultValue[part]) {
                  resultValue[part] = {}
                }
                resultValue = resultValue[part]
              }
              resultValue[pathParts[pathParts.length - 1]] = value
            }
            
            return result
          }, {})
          
          storage.setItem(key, JSON.stringify(partialState))
        } else {
          // 保存所有状态
          storage.setItem(key, JSON.stringify(state))
        }
      })
    },
    { detached: true }
  )
}
```

在Pinia初始化时使用插件：

```typescript
// src/store/index.ts
import { createPinia } from 'pinia'
import { persist } from './plugins/persist'
import type { App } from 'vue'

// 创建pinia实例
const pinia = createPinia()
pinia.use(persist)

export function setupStore(app: App<Element>) {
  app.use(pinia)
}

export { pinia }
```

## 常见问题与解决方案

### 1. 状态变更但视图不更新

**问题描述**：修改了Store中的状态，但视图没有更新

**解决方案**：
- 确保状态是响应式的，避免直接修改深层对象
- 使用`$patch`方法批量更新状态
- 使用`computed`计算属性获取状态

```typescript
// 错误用法
const userStore = useUserStore()
userStore.userInfo.nickname = 'New Name' // 可能不会触发更新

// 正确用法1：使用$patch
userStore.$patch({
  userInfo: {
    ...userStore.userInfo,
    nickname: 'New Name'
  }
})

// 正确用法2：使用actions
userStore.updateUserInfo({
  nickname: 'New Name'
})
```

### 2. 复杂状态管理

**问题描述**：处理复杂的状态管理场景，如多个store之间的交互

**解决方案**：
- 使用组合式API结合多个store
- 在一个store中引用另一个store
- 创建专门的actions处理跨store操作

```typescript
// 在一个store中引用另一个store
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const usePermissionStore = defineStore('permission', {
  state: () => ({ /* ... */ }),
  actions: {
    async generateRoutes() {
      const userStore = useUserStore()
      
      // 使用userStore中的状态
      const roles = userStore.roles
      // ...
    }
  }
})
```

### 3. 类型安全

**问题描述**：在TypeScript环境下保证状态的类型安全

**解决方案**：
- 为状态定义明确的接口
- 使用泛型增强类型推导
- 避免使用`any`类型

```typescript
// 定义状态接口
interface UserState {
  token: string
  userInfo: UserInfo
  roles: string[]
  permissions: string[]
}

// 定义用户信息接口
interface UserInfo {
  id: number
  username: string
  nickname: string
  avatar: string
  email: string
}

// 使用接口定义store
export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    token: getToken() || '',
    userInfo: {} as UserInfo,
    roles: [],
    permissions: []
  }),
  // ...
})
``` 