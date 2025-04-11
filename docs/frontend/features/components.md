# 前端组件开发规范

## 组件设计原则

### 1. 单一职责

- 每个组件应该只负责单一的功能
- 当组件变得复杂时，应该拆分为更小的子组件
- 避免创建超过300行代码的大型组件

### 2. 可复用性

- 组件设计应考虑复用场景
- 通过props和事件提供灵活的配置选项
- 避免在可复用组件中使用硬编码值

### 3. 可测试性

- 组件逻辑应易于测试
- 关注点分离，将业务逻辑与UI渲染分开
- 使用组合式API (Composition API) 抽离复杂逻辑

## 组件目录结构

```
src/components/
├── base/                # 基础组件
│   ├── BaseButton/
│   ├── BaseInput/
│   └── ...
├── business/            # 业务组件
│   ├── UserInfo/
│   ├── RoleSelector/
│   └── ...
├── layout/              # 布局组件
│   ├── AppHeader/
│   ├── AppSidebar/
│   └── ...
└── shared/              # 共享组件
    ├── SearchForm/
    ├── DataTable/
    └── ...
```

## 命名规范

### 1. 组件文件命名

- 使用PascalCase (大驼峰) 命名组件文件：`UserProfile.vue`
- 基础组件使用`Base`前缀：`BaseButton.vue`
- 单例组件使用`App`前缀：`AppHeader.vue`
- 紧密耦合的组件使用父组件名作为前缀：`UserProfileAddress.vue`

### 2. 组件名称

- 组件名应该始终使用多个单词，避免与HTML元素冲突：使用`UserList`而不是`List`
- 组件名应使用PascalCase：`<UserProfile />`

### 3. Props命名

- 使用camelCase (小驼峰) 命名props：`itemList`
- 避免使用单个单词命名props
- 布尔类型的props应该使用`is`或`has`前缀：`isVisible`、`hasPermission`

## 组件实现规范

### 1. 组件结构

```vue
<template>
  <div class="component-container">
    <!-- 组件模板代码 -->
  </div>
</template>

<script setup lang="ts">
// 导入语句
import { ref, computed, onMounted } from 'vue'
import type { PropType } from 'vue'

// Props定义
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  items: {
    type: Array as PropType<Item[]>,
    default: () => []
  },
  isLoading: {
    type: Boolean,
    default: false
  }
})

// Emits定义
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
}>()

// 响应式状态
const isVisible = ref(false)

// 计算属性
const filteredItems = computed(() => {
  return props.items.filter(item => item.isActive)
})

// 方法
const handleClick = () => {
  isVisible.value = !isVisible.value
  emit('update', 'New value')
}

// 生命周期钩子
onMounted(() => {
  console.log('Component mounted')
})

// 暴露给父组件的方法或属性
defineExpose({
  reset: () => {
    isVisible.value = false
  }
})
</script>

<style scoped>
.component-container {
  /* 组件样式 */
}
</style>
```

### 2. Props类型定义

```typescript
// 简单类型定义
const props = defineProps({
  title: String,
  count: Number,
  isActive: Boolean,
  items: Array,
  callback: Function,
  settings: Object
})

// 复杂类型定义
interface User {
  id: number
  name: string
  email: string
}

const props = defineProps({
  user: {
    type: Object as PropType<User>,
    required: true
  },
  userList: {
    type: Array as PropType<User[]>,
    default: () => []
  }
})
```

### 3. 组件通信

- 父子组件通信：使用props和emit
- 兄弟组件通信：使用共同的父组件或事件总线
- 跨多级组件通信：使用Provide/Inject或Pinia

```typescript
// 父组件向子组件传递数据
<UserProfile :user="currentUser" />

// 子组件向父组件发送事件
<button @click="$emit('update', newValue)">更新</button>

// Provide/Inject示例
// 在父组件中
provide('theme', ref('dark'))

// 在子组件中
const theme = inject('theme')
```

## 业务组件封装规范

### 1. CURD组件

项目中的CURD（增删改查）组件是最常用的业务组件，遵循以下规范：

```typescript
// 使用示例
<CurdPage
  :columns="columns"
  :service="userService"
  :search-form-items="searchFormItems"
  :form-items="formItems"
>
  <template #toolbar>
    <el-button type="primary" @click="handleAdd">新增用户</el-button>
  </template>
</CurdPage>
```

### 2. 表单组件

表单组件应该支持以下功能：

- 表单验证
- 表单项动态显示/隐藏
- 表单项依赖关系
- 表单重置

```typescript
// 使用示例
<el-form
  ref="formRef"
  :model="form"
  :rules="rules"
  label-width="100px"
>
  <el-form-item label="用户名" prop="username">
    <el-input v-model="form.username" />
  </el-form-item>
  
  <el-form-item label="角色" prop="role">
    <el-select v-model="form.role">
      <el-option 
        v-for="item in roleOptions" 
        :key="item.value" 
        :label="item.label" 
        :value="item.value" 
      />
    </el-select>
  </el-form-item>
  
  <el-form-item>
    <el-button type="primary" @click="submitForm(formRef)">提交</el-button>
    <el-button @click="resetForm(formRef)">重置</el-button>
  </el-form-item>
</el-form>
```

### 3. 字典组件

字典组件用于统一管理系统中的枚举值：

```typescript
// 使用示例
<dict-select dict-code="gender" v-model="form.gender" />
<dict-radio dict-code="status" v-model="form.status" />
<dict-tag dict-code="priority" :value="row.priority" />
```

## 自定义Hook开发规范

### 1. Hook命名

- 使用`use`前缀命名Hook：`useUserData`、`usePermission`
- 名称应该能清晰表达Hook的功能

### 2. Hook结构

```typescript
import { ref, onMounted, watch } from 'vue'
import type { Ref } from 'vue'

export function useDataFetching<T>(
  fetchFunc: () => Promise<T[]>,
  immediate = true
) {
  const data = ref<T[]>([]) as Ref<T[]>
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const fetchData = async () => {
    loading.value = true
    error.value = null
    try {
      data.value = await fetchFunc()
    } catch (err) {
      error.value = err as Error
      console.error(err)
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    onMounted(fetchData)
  }

  return {
    data,
    loading,
    error,
    fetchData
  }
}

// 使用示例
const { data: users, loading, fetchData } = useDataFetching(() => UserAPI.getList())
```

## 性能优化

### 1. 按需导入组件

```typescript
// 按需导入Element Plus组件
import { ElButton, ElInput } from 'element-plus'
```

### 2. 使用v-memo优化列表渲染

```vue
<template>
  <div>
    <div v-for="item in list" :key="item.id" v-memo="[item.id, item.updated]">
      {{ item.name }}
    </div>
  </div>
</template>
```

### 3. 使用shallowRef/shallowReactive优化大型数据

```typescript
// 大型数据结构使用shallowRef
const tableData = shallowRef([...largeDataSet])
```

### 4. 组件动态导入

```typescript
// 动态导入组件
const UserProfile = defineAsyncComponent(() => import('@/components/UserProfile.vue'))
```

## 辅助功能与国际化

### 1. 辅助功能

- 为表单控件添加适当的`aria-*`属性
- 确保应用的颜色对比度符合WCAG标准
- 确保组件可通过键盘操作

### 2. 国际化

```typescript
// i18n使用示例
<span>{{ $t('user.profile.title') }}</span>

// 复数形式
<span>{{ $tc('user.items', count) }}</span>

// 日期格式化
<span>{{ $d(date, 'long') }}</span>
```

## 最佳实践

1. **组合优于继承**：使用组合式API而不是继承
2. **使用TypeScript**：为组件提供类型定义
3. **性能考虑**：避免不必要的渲染，合理使用计算属性
4. **状态管理**：简单状态使用组件内部状态，复杂状态使用Pinia
5. **代码风格**：遵循ESLint规则和项目代码风格 