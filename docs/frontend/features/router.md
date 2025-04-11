# 前端路由配置说明

## 路由系统概述

本项目使用 Vue Router 进行路由管理，路由配置采用模块化设计，结合动态路由和权限控制实现了灵活的页面访问控制。

## 路由结构设计

### 1. 路由类型

路由分为以下几种类型：

- **基础路由**：不需要登录即可访问的路由，如登录页、404页面等
- **动态路由**：需要根据用户权限动态生成的路由，主要是系统功能模块
- **嵌套路由**：具有父子关系的路由，用于实现多级菜单

### 2. 路由元数据设计

路由元数据（Meta）用于定义路由的附加信息：

```typescript
interface RouteMeta {
  title?: string;            // 页面标题
  icon?: string;             // 图标
  hidden?: boolean;          // 是否在菜单中隐藏
  alwaysShow?: boolean;      // 是否总是显示根路由
  breadcrumb?: boolean;      // 是否在面包屑中显示
  activeMenu?: string;       // 激活的菜单路径
  noCache?: boolean;         // 是否禁用缓存
  permissions?: string[];    // 所需权限标识
  roles?: string[];          // 所需角色
}
```

## 路由配置示例

### 1. 基础路由配置

基础路由定义在`src/router/index.ts`中：

```typescript
// 基础路由 - 不需要登录即可访问
export const constantRoutes: RouteRecordRaw[] = [
  {
    path: '/login',
    component: () => import('@/views/login/index.vue'),
    meta: { hidden: true }
  },
  {
    path: '/404',
    component: () => import('@/views/error-page/404.vue'),
    meta: { hidden: true }
  },
  {
    path: '/401',
    component: () => import('@/views/error-page/401.vue'),
    meta: { hidden: true }
  },
  {
    path: '/',
    component: Layout,
    redirect: '/dashboard',
    children: [
      {
        path: 'dashboard',
        component: () => import('@/views/dashboard/index.vue'),
        name: 'Dashboard',
        meta: { title: '首页', icon: 'dashboard', affix: true }
      }
    ]
  }
]
```

### 2. 动态路由配置

动态路由模块定义在`src/router/modules/`目录下：

```typescript
// src/router/modules/system.ts
export default {
  path: '/system',
  component: Layout,
  redirect: '/system/user',
  name: 'System',
  meta: { 
    title: '系统管理', 
    icon: 'system',
    permissions: ['sys:manage']
  },
  children: [
    {
      path: 'user',
      component: () => import('@/views/system/user/index.vue'),
      name: 'User',
      meta: { 
        title: '用户管理', 
        icon: 'user',
        permissions: ['sys:user:list']
      }
    },
    {
      path: 'role',
      component: () => import('@/views/system/role/index.vue'),
      name: 'Role',
      meta: { 
        title: '角色管理', 
        icon: 'role',
        permissions: ['sys:role:list']
      }
    },
    {
      path: 'menu',
      component: () => import('@/views/system/menu/index.vue'),
      name: 'Menu',
      meta: { 
        title: '菜单管理', 
        icon: 'menu',
        permissions: ['sys:menu:list'] 
      }
    }
  ]
}
```

### 3. 路由控制与加载

路由根据用户权限动态加载：

```typescript
// src/store/modules/permission.ts
import { defineStore } from 'pinia'
import { RouteRecordRaw } from 'vue-router'
import { constantRoutes } from '@/router'
import { store } from '@/store'
import { getUserMenus } from '@/api/menu'

// 动态路由模块
import systemRoutes from '@/router/modules/system'
// ... 其他模块

// 路由映射表
const moduleNameMap = {
  'system': systemRoutes,
  // ... 其他映射
}

export const usePermissionStore = defineStore('permission', {
  state: () => ({
    routes: [] as RouteRecordRaw[],
    addRoutes: [] as RouteRecordRaw[]
  }),
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

## 路由守卫

路由守卫用于控制路由的访问权限，在`src/router/index.ts`中定义：

```typescript
// 全局前置守卫
router.beforeEach(async (to, from, next) => {
  // 设置页面标题
  if (to.meta.title) {
    document.title = `${to.meta.title} - ${settings.title}`
  }
  
  // 获取用户信息
  const userStore = useUserStore()
  const hasToken = userStore.token
  
  if (hasToken) {
    if (to.path === '/login') {
      // 已登录用户重定向到首页
      next({ path: '/' })
    } else {
      // 判断用户信息是否已获取
      const hasRoles = userStore.roles && userStore.roles.length > 0
      
      if (hasRoles) {
        next()
      } else {
        try {
          // 获取用户信息
          await userStore.getUserInfo()
          
          // 根据用户权限生成可访问路由
          const permissionStore = usePermissionStore()
          const accessRoutes = await permissionStore.generateRoutes()
          
          // 动态添加可访问路由
          accessRoutes.forEach(route => {
            router.addRoute(route)
          })
          
          // 确保路由已添加完成
          next({ ...to, replace: true })
        } catch (error) {
          // 重置token并跳转到登录页
          await userStore.resetToken()
          next(`/login?redirect=${to.path}`)
        }
      }
    }
  } else {
    // 未登录用户可以访问白名单页面
    if (whiteList.includes(to.path)) {
      next()
    } else {
      // 其他页面重定向到登录页
      next(`/login?redirect=${to.path}`)
    }
  }
})
```

## 路由导航

### 1. 编程式导航

```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'

const router = useRouter()

const goToUserDetail = (userId: number) => {
  router.push({
    path: `/system/user/detail/${userId}`,
    query: { from: 'list' }
  })
}

const goBack = () => {
  router.back()
}
</script>
```

### 2. 声明式导航

```vue
<template>
  <el-menu router>
    <el-menu-item index="/dashboard">
      <span>首页</span>
    </el-menu-item>
    <el-sub-menu index="/system">
      <template #title>系统管理</template>
      <el-menu-item index="/system/user">
        <span>用户管理</span>
      </el-menu-item>
      <el-menu-item index="/system/role">
        <span>角色管理</span>
      </el-menu-item>
    </el-sub-menu>
  </el-menu>
  
  <!-- 或者使用router-link -->
  <router-link to="/dashboard">首页</router-link>
  <router-link :to="{ path: '/system/user', query: { status: 1 } }">用户管理</router-link>
</template>
```

## 路由参数传递

### 1. 动态路由参数

```typescript
// 路由定义
{
  path: 'detail/:id',
  component: () => import('@/views/system/user/detail.vue'),
  name: 'UserDetail',
  meta: { 
    title: '用户详情', 
    activeMenu: '/system/user',
    hidden: true 
  }
}

// 在组件中获取参数
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
const userId = route.params.id
</script>
```

### 2. 查询参数

```typescript
// 跳转时传递参数
router.push({
  path: '/system/user',
  query: { status: 1, keyword: 'admin' }
})

// 在组件中获取参数
<script setup lang="ts">
import { useRoute } from 'vue-router'

const route = useRoute()
const status = route.query.status
const keyword = route.query.keyword
</script>
```

## 路由缓存控制

通过`keep-alive`和路由元数据实现页面缓存：

```vue
<!-- src/layout/components/AppMain.vue -->
<template>
  <section class="app-main">
    <router-view v-slot="{ Component }">
      <transition name="fade-transform" mode="out-in">
        <keep-alive :include="cachedViews">
          <component :is="Component" :key="key" />
        </keep-alive>
      </transition>
    </router-view>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useTagsViewStore } from '@/store/modules/tagsView'

const route = useRoute()
const tagsViewStore = useTagsViewStore()

// 需要缓存的视图组件名称列表
const cachedViews = computed(() => tagsViewStore.cachedViews)

// 组件的key，用于强制重新渲染
const key = computed(() => route.path)
</script>
```

## 最佳实践

### 1. 路由命名规范

- 路由名称使用PascalCase命名法，如`UserDetail`
- 路由路径使用kebab-case命名法，如`user-detail`
- 路由组件使用按需加载，减小首屏加载时间

### 2. 路由组织建议

- 根据业务模块组织路由文件
- 使用嵌套路由实现多级菜单
- 合理使用路由元数据控制菜单显示和权限

### 3. 常见问题与解决方案

1. **路由无法刷新**
   
   解决方案：使用`router.replace`或动态key强制组件重新渲染

   ```typescript
   // 方案1：使用replace
   router.replace({
     path: route.path,
     query: route.query
   })
   
   // 方案2：使用动态key
   const key = ref(0)
   const refreshPage = () => {
     key.value++
   }
   ```

2. **动态路由加载顺序问题**
   
   解决方案：确保在获取用户信息后才加载动态路由，并使用`router.addRoute`依次添加

3. **路由切换后数据不更新**

   解决方案：在组件内使用`watch`监听路由参数变化

   ```typescript
   import { watch } from 'vue'
   import { useRoute } from 'vue-router'
   
   const route = useRoute()
   
   // 监听路由参数变化
   watch(
     () => route.params.id,
     (newId) => {
       // 重新加载数据
       loadData(newId)
     },
     { immediate: true }
   )
   ``` 