# API 接口文档

## 接口规范

### 1. 请求规范

- **基础URL**: `/api/v1`
- **请求方式**:
  - `GET`: 查询资源
  - `POST`: 创建资源
  - `PUT`: 更新资源
  - `DELETE`: 删除资源
- **Content-Type**: `application/json`
- **请求头**:
  - `Authorization`: Bearer {token} (JWT认证)

### 2. 响应规范

所有接口统一返回JSON格式的响应体，结构如下：

```json
{
  "code": 200,      // 状态码：200成功，其他表示错误
  "data": {},       // 业务数据
  "msg": "success"  // 提示信息
}
```

### 3. 分页响应规范

列表查询接口采用统一的分页响应格式：

```json
{
  "code": 200,
  "data": {
    "list": [],     // 数据列表
    "total": 100,   // 总记录数
    "page": 1,      // 当前页码
    "size": 10      // 每页记录数
  },
  "msg": "success"
}
```

### 4. 错误响应规范

错误响应采用统一的格式：

```json
{
  "code": 400,        // 错误状态码
  "msg": "错误描述",  // 错误描述
  "data": null        // 业务数据为空
}
```

## 接口列表

### 1. 用户模块

#### 1.1 用户登录

- **请求URL**: `/api/v1/auth/login`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "username": "admin",  // 用户名
  "password": "123456"  // 密码
}
```

- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",  // JWT令牌
    "tokenType": "Bearer",
    "expireIn": 86400  // 过期时间（秒）
  },
  "msg": "success"
}
```

#### 1.2 获取用户信息

- **请求URL**: `/api/v1/auth/user-info`
- **请求方式**: `GET`
- **请求参数**: 无
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "username": "admin",
    "nickname": "管理员",
    "avatar": "https://example.com/avatar.jpg",
    "roles": ["ADMIN"],
    "permissions": ["sys:user:add", "sys:user:edit", "sys:user:delete"]
  },
  "msg": "success"
}
```

#### 1.3 用户注销

- **请求URL**: `/api/v1/auth/logout`
- **请求方式**: `POST`
- **请求参数**: 无
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "注销成功"
}
```

#### 1.4 用户列表查询

- **请求URL**: `/api/v1/system/users`
- **请求方式**: `GET`
- **请求参数**:
  - `page`: 页码（默认1）
  - `size`: 每页记录数（默认10）
  - `username`: 用户名（可选）
  - `status`: 状态（可选）
  - `deptId`: 部门ID（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 1,
        "username": "admin",
        "nickname": "管理员",
        "mobile": "13800138000",
        "gender": 1,
        "avatar": "https://example.com/avatar.jpg",
        "status": 1,
        "deptId": 1,
        "deptName": "研发部",
        "createTime": "2023-01-01 00:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 10
  },
  "msg": "success"
}
```

#### 1.5 用户详情

- **请求URL**: `/api/v1/system/users/{id}`
- **请求方式**: `GET`
- **请求参数**: 
  - `id`: 用户ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "username": "admin",
    "nickname": "管理员",
    "mobile": "13800138000",
    "email": "admin@example.com",
    "gender": 1,
    "avatar": "https://example.com/avatar.jpg",
    "status": 1,
    "deptId": 1,
    "roles": [1, 2],  // 角色ID列表
    "createTime": "2023-01-01 00:00:00"
  },
  "msg": "success"
}
```

#### 1.6 新增用户

- **请求URL**: `/api/v1/system/users`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "username": "zhangsan",
  "nickname": "张三",
  "password": "123456",
  "mobile": "13800138001",
  "email": "zhangsan@example.com",
  "gender": 1,
  "status": 1,
  "deptId": 1,
  "roles": [1, 2]  // 角色ID列表
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 2
  },
  "msg": "success"
}
```

#### 1.7 修改用户

- **请求URL**: `/api/v1/system/users/{id}`
- **请求方式**: `PUT`
- **请求参数**:

```json
{
  "nickname": "张三",
  "mobile": "13800138001",
  "email": "zhangsan@example.com",
  "gender": 1,
  "status": 1,
  "deptId": 1,
  "roles": [1, 2]  // 角色ID列表
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 1.8 删除用户

- **请求URL**: `/api/v1/system/users/{id}`
- **请求方式**: `DELETE`
- **请求参数**: 
  - `id`: 用户ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 1.9 重置密码

- **请求URL**: `/api/v1/system/users/{id}/password`
- **请求方式**: `PATCH`
- **请求参数**:

```json
{
  "password": "123456"  // 新密码
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

### 2. 角色模块

#### 2.1 角色列表查询

- **请求URL**: `/api/v1/system/roles`
- **请求方式**: `GET`
- **请求参数**:
  - `page`: 页码（默认1）
  - `size`: 每页记录数（默认10）
  - `name`: 角色名称（可选）
  - `code`: 角色编码（可选）
  - `status`: 状态（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 1,
        "name": "管理员",
        "code": "ADMIN",
        "sort": 1,
        "status": 1,
        "createTime": "2023-01-01 00:00:00"
      }
    ],
    "total": 10,
    "page": 1,
    "size": 10
  },
  "msg": "success"
}
```

#### 2.2 角色详情

- **请求URL**: `/api/v1/system/roles/{id}`
- **请求方式**: `GET`
- **请求参数**: 
  - `id`: 角色ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "name": "管理员",
    "code": "ADMIN",
    "sort": 1,
    "status": 1,
    "createTime": "2023-01-01 00:00:00",
    "menuIds": [1, 2, 3]  // 菜单ID列表
  },
  "msg": "success"
}
```

#### 2.3 新增角色

- **请求URL**: `/api/v1/system/roles`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "name": "运维",
  "code": "OPS",
  "sort": 2,
  "status": 1,
  "menuIds": [1, 2, 3]  // 菜单ID列表
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 2
  },
  "msg": "success"
}
```

#### 2.4 修改角色

- **请求URL**: `/api/v1/system/roles/{id}`
- **请求方式**: `PUT`
- **请求参数**:

```json
{
  "name": "运维",
  "sort": 2,
  "status": 1,
  "menuIds": [1, 2, 3]  // 菜单ID列表
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 2.5 删除角色

- **请求URL**: `/api/v1/system/roles/{id}`
- **请求方式**: `DELETE`
- **请求参数**: 
  - `id`: 角色ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

### 3. 菜单模块

#### 3.1 菜单树形列表

- **请求URL**: `/api/v1/system/menus/tree`
- **请求方式**: `GET`
- **请求参数**:
  - `status`: 状态（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "parentId": 0,
      "name": "系统管理",
      "type": 2,
      "routePath": "/system",
      "component": "Layout",
      "perm": null,
      "visible": 1,
      "sort": 1,
      "icon": "system",
      "children": [
        {
          "id": 2,
          "parentId": 1,
          "name": "用户管理",
          "type": 1,
          "routePath": "user",
          "component": "system/user/index",
          "perm": null,
          "visible": 1,
          "sort": 1,
          "icon": "user",
          "children": []
        }
      ]
    }
  ],
  "msg": "success"
}
```

#### 3.2 菜单详情

- **请求URL**: `/api/v1/system/menus/{id}`
- **请求方式**: `GET`
- **请求参数**: 
  - `id`: 菜单ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "parentId": 0,
    "name": "系统管理",
    "type": 2,
    "routeName": "",
    "routePath": "/system",
    "component": "Layout",
    "perm": null,
    "visible": 1,
    "sort": 1,
    "icon": "system",
    "redirect": "/system/user"
  },
  "msg": "success"
}
```

#### 3.3 新增菜单

- **请求URL**: `/api/v1/system/menus`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "parentId": 1,
  "name": "菜单管理",
  "type": 1,
  "routeName": "Menu",
  "routePath": "menu",
  "component": "system/menu/index",
  "perm": null,
  "visible": 1,
  "sort": 3,
  "icon": "menu",
  "redirect": null
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 4
  },
  "msg": "success"
}
```

#### 3.4 修改菜单

- **请求URL**: `/api/v1/system/menus/{id}`
- **请求方式**: `PUT`
- **请求参数**:

```json
{
  "parentId": 1,
  "name": "菜单管理",
  "type": 1,
  "routeName": "Menu",
  "routePath": "menu",
  "component": "system/menu/index",
  "perm": null,
  "visible": 1,
  "sort": 3,
  "icon": "menu",
  "redirect": null
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 3.5 删除菜单

- **请求URL**: `/api/v1/system/menus/{id}`
- **请求方式**: `DELETE`
- **请求参数**: 
  - `id`: 菜单ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

### 4. 部门模块

#### 4.1 部门树形列表

- **请求URL**: `/api/v1/system/depts/tree`
- **请求方式**: `GET`
- **请求参数**:
  - `status`: 状态（可选）
  - `name`: 部门名称（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "parentId": 0,
      "name": "有来技术",
      "code": "YOULAI",
      "sort": 1,
      "status": 1,
      "children": [
        {
          "id": 2,
          "parentId": 1,
          "name": "研发部门",
          "code": "RD001",
          "sort": 1,
          "status": 1,
          "children": []
        }
      ]
    }
  ],
  "msg": "success"
}
```

#### 4.2 部门详情

- **请求URL**: `/api/v1/system/depts/{id}`
- **请求方式**: `GET`
- **请求参数**: 
  - `id`: 部门ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 2,
    "parentId": 1,
    "name": "研发部门",
    "code": "RD001",
    "sort": 1,
    "status": 1
  },
  "msg": "success"
}
```

#### 4.3 新增部门

- **请求URL**: `/api/v1/system/depts`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "parentId": 1,
  "name": "测试部门",
  "code": "TEST001",
  "sort": 2,
  "status": 1
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 3
  },
  "msg": "success"
}
```

#### 4.4 修改部门

- **请求URL**: `/api/v1/system/depts/{id}`
- **请求方式**: `PUT`
- **请求参数**:

```json
{
  "parentId": 1,
  "name": "测试部门",
  "sort": 2,
  "status": 1
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 4.5 删除部门

- **请求URL**: `/api/v1/system/depts/{id}`
- **请求方式**: `DELETE`
- **请求参数**: 
  - `id`: 部门ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

### 5. 字典模块

#### 5.1 字典列表查询

- **请求URL**: `/api/v1/system/dicts`
- **请求方式**: `GET`
- **请求参数**:
  - `page`: 页码（默认1）
  - `size`: 每页记录数（默认10）
  - `name`: 字典名称（可选）
  - `dictCode`: 字典编码（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 1,
        "dictCode": "gender",
        "name": "性别",
        "status": 1,
        "remark": null,
        "createTime": "2023-01-01 00:00:00"
      }
    ],
    "total": 10,
    "page": 1,
    "size": 10
  },
  "msg": "success"
}
```

#### 5.2 字典详情

- **请求URL**: `/api/v1/system/dicts/{id}`
- **请求方式**: `GET`
- **请求参数**: 
  - `id`: 字典ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "dictCode": "gender",
    "name": "性别",
    "status": 1,
    "remark": null
  },
  "msg": "success"
}
```

#### 5.3 新增字典

- **请求URL**: `/api/v1/system/dicts`
- **请求方式**: `POST`
- **请求参数**:

```json
{
  "dictCode": "status",
  "name": "状态",
  "status": 1,
  "remark": "系统状态"
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "id": 2
  },
  "msg": "success"
}
```

#### 5.4 修改字典

- **请求URL**: `/api/v1/system/dicts/{id}`
- **请求方式**: `PUT`
- **请求参数**:

```json
{
  "name": "状态",
  "status": 1,
  "remark": "系统状态"
}
```

- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 5.5 删除字典

- **请求URL**: `/api/v1/system/dicts/{id}`
- **请求方式**: `DELETE`
- **请求参数**: 
  - `id`: 字典ID
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": null,
  "msg": "success"
}
```

#### 5.6 字典项列表查询

- **请求URL**: `/api/v1/system/dict-items`
- **请求方式**: `GET`
- **请求参数**:
  - `dictCode`: 字典编码
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": [
    {
      "id": 1,
      "dictCode": "gender",
      "value": "1",
      "label": "男",
      "tagType": "primary",
      "status": 1,
      "sort": 1
    },
    {
      "id": 2,
      "dictCode": "gender",
      "value": "2",
      "label": "女",
      "tagType": "danger",
      "status": 1,
      "sort": 2
    }
  ],
  "msg": "success"
}
```

### 6. 系统监控

#### 6.1 服务监控

- **请求URL**: `/api/v1/monitor/server`
- **请求方式**: `GET`
- **请求参数**: 无
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "cpu": {
      "usageRate": 12.34,
      "cores": 8
    },
    "memory": {
      "total": 16384,
      "used": 8192,
      "usageRate": 50.0
    },
    "jvm": {
      "total": 1024,
      "used": 512,
      "usageRate": 50.0
    },
    "disk": {
      "total": 512000,
      "used": 256000,
      "usageRate": 50.0
    }
  },
  "msg": "success"
}
```

#### 6.2 操作日志查询

- **请求URL**: `/api/v1/monitor/operation-logs`
- **请求方式**: `GET`
- **请求参数**:
  - `page`: 页码（默认1）
  - `size`: 每页记录数（默认10）
  - `module`: 操作模块（可选）
  - `userId`: 操作人ID（可选）
  - `startTime`: 开始时间（可选）
  - `endTime`: 结束时间（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 1,
        "module": "用户管理",
        "operation": "新增用户",
        "type": 1,
        "method": "com.example.controller.UserController.add",
        "requestUrl": "/api/v1/system/users",
        "requestMethod": "POST",
        "requestParams": "{\"username\":\"test\"}",
        "requestTime": 100,
        "ip": "192.168.1.1",
        "userId": 1,
        "status": 1,
        "errorMsg": null,
        "createTime": "2023-01-01 00:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 10
  },
  "msg": "success"
}
```

#### 6.3 登录日志查询

- **请求URL**: `/api/v1/monitor/login-logs`
- **请求方式**: `GET`
- **请求参数**:
  - `page`: 页码（默认1）
  - `size`: 每页记录数（默认10）
  - `username`: 用户名（可选）
  - `status`: 状态（可选）
  - `startTime`: 开始时间（可选）
  - `endTime`: 结束时间（可选）
- **请求头**: 需要携带token
- **成功响应**:

```json
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 1,
        "username": "admin",
        "ip": "192.168.1.1",
        "os": "Windows 10",
        "browser": "Chrome",
        "status": 1,
        "msg": "登录成功",
        "createTime": "2023-01-01 00:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 10
  },
  "msg": "success"
}
```

## 状态码说明

| 状态码 | 说明 |
|-------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |
| 1001 | 用户名或密码错误 |
| 1002 | 账号已被禁用 |
| 1003 | 验证码错误 |
| 1004 | 令牌已过期 |
| 1005 | 令牌无效 |

## 接口鉴权

本系统采用JWT (JSON Web Token) 进行接口鉴权：

1. 用户登录成功后获取token
2. 后续请求在请求头中携带token：`Authorization: Bearer {token}`
3. 服务端验证token的有效性，决定是否允许访问
4. token过期后需要重新登录获取token

## 接口权限控制

本系统采用基于角色的权限控制 (RBAC)：

1. 用户关联角色，角色关联权限
2. 接口通过权限标识进行权限控制
3. 用户请求接口时，检查用户是否拥有对应的权限
4. 权限不足时返回403状态码 