# 前端API调用规范

## API架构设计

### 1. 目录结构

```
src/api/
├── index.ts                # API入口文件，统一导出
├── auth.api.ts             # 认证相关API
├── user.api.ts             # 用户相关API
├── system/                 # 系统管理模块API
│   ├── role.api.ts         # 角色管理API
│   ├── menu.api.ts         # 菜单管理API
│   └── dept.api.ts         # 部门管理API
├── monitor/                # 系统监控模块API
│   ├── log.api.ts          # 日志管理API
│   └── server.api.ts       # 服务监控API
└── types/                  # 类型定义
    ├── auth.d.ts           # 认证相关类型
    ├── user.d.ts           # 用户相关类型
    └── common.d.ts         # 通用类型
```

### 2. 请求封装

项目使用Axios进行HTTP请求封装，统一处理请求拦截、响应拦截、错误处理和身份验证。

```typescript
// src/utils/request.ts
import axios, { type InternalAxiosRequestConfig, type AxiosResponse } from "axios";
import { useUserStoreHook } from "@/store/modules/user.store";
import { ResultEnum } from "@/enums/api/result.enum";
import { getAccessToken } from "@/utils/auth";

// 创建axios实例
const service = axios.create({
  baseURL: import.meta.env.VITE_APP_BASE_API,
  timeout: 50000,
  headers: { "Content-Type": "application/json;charset=utf-8" }
});

// 请求拦截器
service.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = getAccessToken();
    if (token && config.headers.Authorization !== "no-auth") {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器
service.interceptors.response.use(
  (response: AxiosResponse) => {
    // 二进制数据直接返回
    if (response.config.responseType === "blob") {
      return response;
    }
    
    const { code, data, msg } = response.data;
    
    // 成功响应，直接返回数据
    if (code === ResultEnum.SUCCESS) {
      return data;
    }
    
    // 错误处理
    ElMessage.error(msg || "系统出错");
    return Promise.reject(new Error(msg || "Error"));
  },
  async (error) => {
    // Token相关错误处理
    if (error.response?.status === 401) {
      await handleUnauthorized();
    } else {
      const message = error.response?.data?.msg || error.message;
      ElMessage.error(message);
    }
    return Promise.reject(error);
  }
);

export default service;
```

## API模块设计

### 1. 模块化设计

每个API模块应该是一个独立的文件，按照业务功能划分。

```typescript
// src/api/system/user.api.ts
import request from "@/utils/request";
import type { UserInfo, UserQuery, UserForm } from "@/types/user";

const BASE_URL = "/api/v1/system/users";

const UserAPI = {
  /**
   * 获取用户列表
   * @param params 查询参数
   * @returns 用户列表和分页信息
   */
  getList(params: UserQuery) {
    return request<UserQuery, PageResult<UserInfo>>({
      url: BASE_URL,
      method: "get",
      params
    });
  },

  /**
   * 获取用户详情
   * @param id 用户ID
   * @returns 用户详情
   */
  getDetail(id: number) {
    return request<null, UserInfo>({
      url: `${BASE_URL}/${id}`,
      method: "get"
    });
  },

  /**
   * 创建用户
   * @param data 用户表单数据
   * @returns 创建结果
   */
  create(data: UserForm) {
    return request<UserForm, any>({
      url: BASE_URL,
      method: "post",
      data
    });
  },

  /**
   * 更新用户
   * @param id 用户ID
   * @param data 用户表单数据
   * @returns 更新结果
   */
  update(id: number, data: UserForm) {
    return request<UserForm, any>({
      url: `${BASE_URL}/${id}`,
      method: "put",
      data
    });
  },

  /**
   * 删除用户
   * @param id 用户ID
   * @returns 删除结果
   */
  delete(id: number) {
    return request<null, any>({
      url: `${BASE_URL}/${id}`,
      method: "delete"
    });
  }
};

export default UserAPI;
```

### 2. 类型定义

为每个API模块定义明确的请求参数和响应数据类型。

```typescript
// src/types/user.d.ts
/** 用户信息 */
export interface UserInfo {
  /** 用户ID */
  id: number;
  /** 用户名 */
  username: string;
  /** 昵称 */
  nickname: string;
  /** 手机号 */
  mobile: string;
  /** 邮箱 */
  email: string;
  /** 性别(1-男 2-女) */
  gender: number;
  /** 头像URL */
  avatar: string;
  /** 状态(1-启用 0-禁用) */
  status: number;
  /** 部门ID */
  deptId: number;
  /** 部门名称 */
  deptName: string;
  /** 角色ID列表 */
  roleIds: number[];
  /** 创建时间 */
  createTime: string;
}

/** 用户查询参数 */
export interface UserQuery extends PageQuery {
  /** 关键字(用户名/昵称/手机号) */
  keywords?: string;
  /** 状态 */
  status?: number;
  /** 部门ID */
  deptId?: number;
  /** 开始时间 */
  startTime?: string;
  /** 结束时间 */
  endTime?: string;
}

/** 用户表单数据 */
export interface UserForm {
  /** 用户ID(编辑时需要) */
  id?: number;
  /** 用户名 */
  username: string;
  /** 昵称 */
  nickname: string;
  /** 密码(创建时必填) */
  password?: string;
  /** 手机号 */
  mobile?: string;
  /** 邮箱 */
  email?: string;
  /** 性别 */
  gender?: number;
  /** 头像URL */
  avatar?: string;
  /** 状态 */
  status?: number;
  /** 部门ID */
  deptId?: number;
  /** 角色ID列表 */
  roleIds?: number[];
}
```

### 3. 通用响应类型

定义通用的响应类型，用于处理分页、结果等通用数据结构。

```typescript
// src/types/common.d.ts
/** API响应结构 */
export interface ApiResponse<T> {
  /** 状态码 */
  code: number;
  /** 数据 */
  data: T;
  /** 消息 */
  msg: string;
}

/** 分页查询参数 */
export interface PageQuery {
  /** 页码 */
  page?: number;
  /** 每页记录数 */
  size?: number;
  /** 排序字段 */
  sortField?: string;
  /** 排序方式(asc/desc) */
  sortOrder?: string;
}

/** 分页结果 */
export interface PageResult<T> {
  /** 数据列表 */
  list: T[];
  /** 总记录数 */
  total: number;
  /** 当前页码 */
  page: number;
  /** 每页记录数 */
  size: number;
}
```

## API调用规范

### 1. 在组件中使用API

```typescript
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import UserAPI from '@/api/system/user.api'
import type { UserInfo, UserQuery } from '@/types/user'

// 查询参数
const queryParams = ref<UserQuery>({
  page: 1,
  size: 10
})

// 用户列表数据
const userList = ref<UserInfo[]>([])
const total = ref(0)
const loading = ref(false)

// 获取用户列表
const getUserList = async () => {
  loading.value = true
  try {
    const { list, total: totalCount } = await UserAPI.getList(queryParams.value)
    userList.value = list
    total.value = totalCount
  } catch (error) {
    console.error('获取用户列表失败', error)
  } finally {
    loading.value = false
  }
}

// 组件挂载时获取数据
onMounted(() => {
  getUserList()
})

// 处理分页变化
const handlePageChange = (page: number) => {
  queryParams.value.page = page
  getUserList()
}

// 处理每页条数变化
const handleSizeChange = (size: number) => {
  queryParams.value.size = size
  queryParams.value.page = 1
  getUserList()
}
</script>
```

### 2. 使用自定义Hook封装API调用

创建通用的数据加载Hook，简化API调用。

```typescript
// src/hooks/useDataList.ts
import { ref, reactive, onMounted } from 'vue'
import type { PageQuery, PageResult } from '@/types/common'

export function useDataList<P extends PageQuery, T>(
  api: (params: P) => Promise<PageResult<T>>,
  defaultParams: Partial<P> = {},
  immediate = true
) {
  // 查询参数
  const queryParams = reactive<P>({
    page: 1,
    size: 10,
    ...defaultParams
  } as P)

  // 数据列表
  const list = ref<T[]>([])
  const total = ref(0)
  const loading = ref(false)

  // 获取数据
  const loadData = async () => {
    loading.value = true
    try {
      const { list: resultList, total: resultTotal } = await api(queryParams)
      list.value = resultList
      total.value = resultTotal
    } catch (error) {
      console.error('加载数据失败', error)
    } finally {
      loading.value = false
    }
  }

  // 重置查询参数
  const resetQuery = () => {
    Object.assign(queryParams, {
      page: 1,
      size: 10,
      ...defaultParams
    })
    loadData()
  }

  // 处理分页变化
  const handlePageChange = (page: number) => {
    queryParams.page = page
    loadData()
  }

  // 处理每页条数变化
  const handleSizeChange = (size: number) => {
    queryParams.size = size
    queryParams.page = 1
    loadData()
  }

  // 立即加载数据
  if (immediate) {
    onMounted(loadData)
  }

  return {
    queryParams,
    list,
    total,
    loading,
    loadData,
    resetQuery,
    handlePageChange,
    handleSizeChange
  }
}

// 使用示例
const {
  queryParams,
  list: userList,
  total,
  loading,
  loadData: getUserList,
  resetQuery,
  handlePageChange,
  handleSizeChange
} = useDataList(UserAPI.getList, { status: 1 })
```

### 3. 错误处理

统一处理API错误，避免在每个组件中重复编写错误处理逻辑。

```typescript
// src/utils/error-handler.ts
import { useUserStoreHook } from "@/store/modules/user.store";

// HTTP状态码错误处理
export function handleHttpStatusError(status: number, message: string) {
  switch (status) {
    case 400:
      ElMessage.error(`请求错误: ${message}`);
      break;
    case 401:
      handleUnauthorized();
      break;
    case 403:
      ElMessage.error("权限不足，无法访问");
      break;
    case 404:
      ElMessage.error("请求的资源不存在");
      break;
    case 500:
      ElMessage.error("服务器内部错误");
      break;
    default:
      ElMessage.error(`未知错误: ${message}`);
  }
}

// 处理未授权错误(401)
export async function handleUnauthorized() {
  const userStore = useUserStoreHook();
  
  ElMessage.error("登录已过期，请重新登录");
  await userStore.clearSessionAndCache();
  
  // 跳转到登录页，保留当前页面路径作为重定向目标
  const currentPath = window.location.pathname;
  window.location.href = `/login?redirect=${encodeURIComponent(currentPath)}`;
}
```

## 身份验证与Token管理

### 1. Token存储与获取

```typescript
// src/utils/auth.ts
import Cookies from 'js-cookie';

const ACCESS_TOKEN_KEY = 'access_token';
const REFRESH_TOKEN_KEY = 'refresh_token';

// 获取访问令牌
export function getAccessToken(): string {
  return Cookies.get(ACCESS_TOKEN_KEY) || '';
}

// 设置访问令牌，过期时间默认为1天
export function setAccessToken(token: string, expires = 1): void {
  Cookies.set(ACCESS_TOKEN_KEY, token, { expires });
}

// 获取刷新令牌
export function getRefreshToken(): string {
  return Cookies.get(REFRESH_TOKEN_KEY) || '';
}

// 设置刷新令牌，过期时间默认为7天
export function setRefreshToken(token: string, expires = 7): void {
  Cookies.set(REFRESH_TOKEN_KEY, token, { expires });
}

// 清除所有令牌
export function clearToken(): void {
  Cookies.remove(ACCESS_TOKEN_KEY);
  Cookies.remove(REFRESH_TOKEN_KEY);
}
```

### 2. Token刷新机制

当访问令牌过期时，自动使用刷新令牌获取新的访问令牌。

```typescript
// 在请求拦截器中实现自动刷新Token
let isRefreshing = false;
const waitingQueue: Array<() => void> = [];

async function handleTokenRefresh(config: InternalAxiosRequestConfig) {
  return new Promise((resolve) => {
    // 封装重试请求
    const retryRequest = () => {
      config.headers.Authorization = `Bearer ${getAccessToken()}`;
      resolve(service(config));
    };
    
    // 将请求添加到等待队列
    waitingQueue.push(retryRequest);
    
    // 避免多个请求同时刷新Token
    if (!isRefreshing) {
      isRefreshing = true;
      
      // 刷新Token
      useUserStoreHook()
        .refreshToken()
        .then(() => {
          // 重试所有等待的请求
          waitingQueue.forEach((callback) => callback());
          waitingQueue.length = 0;
        })
        .catch(() => {
          // 刷新失败，清除会话并跳转到登录页
          handleUnauthorized();
        })
        .finally(() => {
          isRefreshing = false;
        });
    }
  });
}
```

## API Mock配置

### 1. Mock服务配置

使用Mock.js和vite-plugin-mock进行本地API模拟。

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { viteMockServe } from 'vite-plugin-mock';

export default defineConfig({
  plugins: [
    vue(),
    viteMockServe({
      mockPath: 'mock',
      localEnabled: process.env.NODE_ENV === 'development',
      prodEnabled: false,
      injectCode: `
        import { setupProdMockServer } from './mock/index';
        setupProdMockServer();
      `,
      logger: false
    })
  ]
});
```

### 2. Mock数据示例

```typescript
// mock/user.ts
import { MockMethod } from 'vite-plugin-mock';
import { ResultEnum } from '../src/enums/api/result.enum';
import { Random } from 'mockjs';

export default [
  {
    url: '/api/v1/system/users',
    method: 'get',
    response: ({ query }) => {
      const { page = 1, size = 10 } = query;
      
      // 生成模拟数据
      const list = Array.from({ length: size }).map((_, index) => ({
        id: Number(page - 1) * 10 + index + 1,
        username: Random.name().toLowerCase().replace(/\s/g, ''),
        nickname: Random.cname(),
        mobile: Random.string('number', 11),
        email: Random.email(),
        gender: Random.integer(1, 2),
        avatar: Random.image('100x100', '#50B347', '#FFF', 'Avatar'),
        status: Random.integer(0, 1),
        deptId: Random.integer(1, 5),
        deptName: Random.pick(['研发部', '测试部', '运维部', '市场部', '销售部']),
        createTime: Random.datetime()
      }));
      
      return {
        code: ResultEnum.SUCCESS,
        data: {
          list,
          total: 100,
          page: Number(page),
          size: Number(size)
        },
        msg: 'success'
      };
    }
  }
] as MockMethod[];
```

## 文件上传/下载

### 1. 文件上传

```typescript
// src/api/file.api.ts
import request from "@/utils/request";

const FileAPI = {
  /**
   * 上传文件
   * @param file 文件对象
   * @param type 文件类型(avatar/attachment)
   * @returns 上传结果
   */
  upload(file: File, type = 'attachment') {
    const formData = new FormData();
    formData.append('file', file);
    
    return request<FormData, { url: string; name: string; size: number }>({
      url: '/api/v1/files/upload',
      method: 'post',
      data: formData,
      params: { type },
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });
  },
  
  /**
   * 批量上传文件
   * @param files 文件对象数组
   * @param type 文件类型
   * @returns 上传结果
   */
  batchUpload(files: File[], type = 'attachment') {
    const formData = new FormData();
    files.forEach(file => {
      formData.append('files', file);
    });
    
    return request<FormData, Array<{ url: string; name: string; size: number }>>({
      url: '/api/v1/files/batch-upload',
      method: 'post',
      data: formData,
      params: { type },
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });
  }
};

export default FileAPI;
```

### 2. 文件下载

```typescript
// src/api/file.api.ts
/**
 * 下载文件
 * @param url 文件URL
 * @param fileName 文件名称
 */
export function downloadFile(url: string, fileName?: string) {
  request({
    url,
    method: 'get',
    responseType: 'blob'
  }).then((response: any) => {
    // 从响应头获取文件名
    const contentDisposition = response.headers['content-disposition'];
    let downloadFileName = fileName;
    
    if (!downloadFileName && contentDisposition) {
      const filenameRegex = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/;
      const matches = filenameRegex.exec(contentDisposition);
      if (matches && matches[1]) {
        downloadFileName = matches[1].replace(/['"]/g, '');
      }
    }
    
    // 创建下载链接
    const blob = new Blob([response.data]);
    const link = document.createElement('a');
    link.href = window.URL.createObjectURL(blob);
    link.download = downloadFileName || 'download';
    link.click();
    
    // 释放URL对象
    setTimeout(() => {
      window.URL.revokeObjectURL(link.href);
    }, 100);
  }).catch(error => {
    ElMessage.error('下载文件失败');
    console.error('Download file error', error);
  });
}
```

## 最佳实践

### 1. API命名规范

- API文件命名：使用`kebab-case.api.ts`格式，例如：`user.api.ts`
- API方法命名：使用动词+名词的形式，例如：`getUser`、`createUser`
- API常量命名：使用大写字母和下划线，例如：`BASE_URL`

### 2. 注释规范

- 每个API方法都应该有JSDoc注释，包括功能描述、参数说明和返回值说明
- 使用TypeScript类型注解确保类型安全

### 3. 错误处理

- 在API层处理通用错误，在业务层处理业务相关错误
- 使用try-catch捕获异步操作的错误
- 展示友好的错误提示，隐藏技术细节

### 4. 性能优化

- 使用请求缓存减少重复请求
- 合理设置请求超时时间
- 大数据量分页加载
- 防抖和节流处理频繁的API调用

### 5. 安全性

- 敏感数据传输时使用HTTPS
- 防止XSS攻击，对输入输出进行过滤
- 防止CSRF攻击，使用Token验证
- 避免在URL中直接传递敏感信息 