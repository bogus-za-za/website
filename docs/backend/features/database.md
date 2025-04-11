# 数据库设计文档

## 数据库概述

本系统使用 MySQL 8.x 作为关系型数据库，使用 TypeORM 作为 ORM 框架。数据库设计遵循以下原则：

- 统一命名规范，表名采用下划线命名法
- 合理设计索引，提高查询效率
- 使用外键约束，保证数据一致性
- 字段设计采用"业务无关"原则

## 数据库表设计

### 1. 用户表 (sys_user)

用户表存储系统用户信息，包括账号、密码、个人资料等。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| username | varchar | 50 | 否 | - | 用户名 |
| nickname | varchar | 50 | 否 | - | 昵称 |
| password | varchar | 100 | 否 | - | 密码(加密) |
| mobile | varchar | 11 | 是 | NULL | 手机号 |
| email | varchar | 100 | 是 | NULL | 邮箱 |
| gender | tinyint | - | 是 | 1 | 性别(1-男 2-女) |
| avatar | varchar | 255 | 是 | NULL | 头像URL |
| status | tinyint | - | 是 | 1 | 状态(1-启用 0-禁用) |
| dept_id | bigint | - | 是 | NULL | 部门ID |
| create_by | varchar | 50 | 是 | NULL | 创建人 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| update_by | varchar | 50 | 是 | NULL | 更新人 |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| is_deleted | tinyint | - | 是 | 0 | 删除标记(0-未删除 1-已删除) |

**索引：**
- 主键：`id`
- 唯一索引：`username`

### 2. 角色表 (sys_role)

角色表存储系统角色信息，与用户和菜单关联构成权限体系。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| name | varchar | 50 | 否 | - | 角色名称 |
| code | varchar | 50 | 否 | - | 角色编码 |
| sort | int | - | 是 | 0 | 显示顺序 |
| status | tinyint | - | 是 | 1 | 状态(1-正常 0-禁用) |
| create_by | varchar | 50 | 是 | NULL | 创建人 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| update_by | varchar | 50 | 是 | NULL | 更新人 |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| is_deleted | tinyint | - | 是 | 0 | 删除标记(0-未删除 1-已删除) |

**索引：**
- 主键：`id`
- 唯一索引：`code`

### 3. 菜单表 (sys_menu)

菜单表存储系统菜单和权限信息，构建系统菜单树和权限控制。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| parent_id | bigint | - | 否 | 0 | 父菜单ID |
| tree_path | varchar | 255 | 是 | - | 父节点ID路径 |
| name | varchar | 64 | 否 | - | 菜单名称 |
| type | tinyint | - | 否 | - | 菜单类型(1-菜单 2-目录 3-外链 4-按钮) |
| route_name | varchar | 255 | 是 | NULL | 路由名称 |
| route_path | varchar | 128 | 是 | NULL | 路由路径 |
| component | varchar | 128 | 是 | NULL | 组件路径 |
| perm | varchar | 128 | 是 | NULL | 权限标识 |
| visible | tinyint | - | 是 | 1 | 显示状态(1-显示 0-隐藏) |
| sort | int | - | 是 | 0 | 排序 |
| icon | varchar | 64 | 是 | NULL | 菜单图标 |
| always_show | tinyint | - | 是 | 0 | 是否始终显示(1-是 0-否) |
| keep_alive | tinyint | - | 是 | 0 | 是否缓存(1-是 0-否) |
| redirect | varchar | 128 | 是 | NULL | 跳转路径 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| update_time | datetime | - | 是 | NULL | 更新时间 |

**索引：**
- 主键：`id`

### 4. 部门表 (sys_dept)

部门表存储组织架构信息，构建部门树。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| name | varchar | 100 | 否 | - | 部门名称 |
| code | varchar | 100 | 否 | - | 部门编号 |
| parent_id | bigint | - | 是 | 0 | 父部门ID |
| tree_path | varchar | 255 | 否 | - | 父节点ID路径 |
| sort | smallint | - | 是 | 0 | 显示顺序 |
| status | tinyint | - | 是 | 1 | 状态(1-正常 0-禁用) |
| create_by | bigint | - | 是 | NULL | 创建人ID |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| update_by | bigint | - | 是 | NULL | 更新人ID |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| is_deleted | tinyint | - | 是 | 0 | 删除标记(0-未删除 1-已删除) |

**索引：**
- 主键：`id`
- 唯一索引：`code`

### 5. 用户角色关联表 (sys_user_role)

用户角色关联表存储用户与角色的多对多关系。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| user_id | bigint | - | 否 | - | 用户ID |
| role_id | bigint | - | 否 | - | 角色ID |

**索引：**
- 主键：`user_id, role_id`
- 外键：`user_id` 关联 `sys_user.id`
- 外键：`role_id` 关联 `sys_role.id`

### 6. 角色菜单关联表 (sys_role_menu)

角色菜单关联表存储角色与菜单的多对多关系。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| role_id | bigint | - | 否 | - | 角色ID |
| menu_id | bigint | - | 否 | - | 菜单ID |

**索引：**
- 主键：`role_id, menu_id`
- 外键：`role_id` 关联 `sys_role.id`
- 外键：`menu_id` 关联 `sys_menu.id`

### 7. 字典表 (sys_dict)

字典表存储系统中的各种字典类型。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| dict_code | varchar | 50 | 是 | NULL | 字典编码 |
| name | varchar | 50 | 是 | NULL | 字典名称 |
| status | tinyint | - | 是 | 0 | 状态(1-正常 0-禁用) |
| remark | varchar | 255 | 是 | NULL | 备注 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| create_by | bigint | - | 是 | NULL | 创建人ID |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| update_by | bigint | - | 是 | NULL | 更新人ID |
| is_deleted | tinyint | - | 是 | 0 | 删除标记(0-未删除 1-已删除) |

**索引：**
- 主键：`id`
- 索引：`dict_code`

### 8. 字典项表 (sys_dict_item)

字典项表存储字典的具体项。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| dict_code | varchar | 50 | 是 | NULL | 关联字典编码 |
| value | varchar | 50 | 是 | NULL | 字典项值 |
| label | varchar | 100 | 是 | NULL | 字典项标签 |
| tag_type | varchar | 50 | 是 | NULL | 标签类型 |
| status | tinyint | - | 是 | 0 | 状态(1-正常 0-禁用) |
| sort | int | - | 是 | 0 | 排序 |
| remark | varchar | 255 | 是 | NULL | 备注 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| create_by | bigint | - | 是 | NULL | 创建人ID |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| update_by | bigint | - | 是 | NULL | 更新人ID |

**索引：**
- 主键：`id`

### 9. 系统日志 (sys_log)

系统日志表存储用户操作日志和系统日志。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| module | varchar | 50 | 是 | NULL | 操作模块 |
| operation | varchar | 100 | 是 | NULL | 操作内容 |
| type | tinyint | - | 是 | NULL | 日志类型 |
| method | varchar | 100 | 是 | NULL | 方法名 |
| request_url | varchar | 255 | 是 | NULL | 请求URL |
| request_method | varchar | 10 | 是 | NULL | 请求方式 |
| request_params | text | - | 是 | NULL | 请求参数 |
| request_time | bigint | - | 是 | NULL | 请求时长(ms) |
| ip | varchar | 50 | 是 | NULL | 操作IP |
| user_id | bigint | - | 是 | NULL | 操作人ID |
| status | int | - | 是 | NULL | 操作状态 |
| error_msg | text | - | 是 | NULL | 错误信息 |
| create_time | datetime | - | 是 | NULL | 创建时间 |

**索引：**
- 主键：`id`
- 索引：`create_time`

### 10. 系统配置表 (sys_config)

系统配置表存储系统参数配置。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| name | varchar | 100 | 否 | - | 参数名称 |
| key | varchar | 100 | 否 | - | 参数键名 |
| value | varchar | 500 | 否 | - | 参数键值 |
| type | tinyint | - | 是 | 1 | 系统内置(1-是 0-否) |
| remark | varchar | 500 | 是 | NULL | 备注 |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| create_by | bigint | - | 是 | NULL | 创建人ID |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| update_by | bigint | - | 是 | NULL | 更新人ID |

**索引：**
- 主键：`id`
- 唯一索引：`key`

### 11. 通知公告表 (sys_notice)

通知公告表存储系统公告信息。

| 字段名 | 数据类型 | 长度 | 允许空 | 默认值 | 说明 |
|-------|--------|------|-------|-------|------|
| id | bigint | - | 否 | - | 主键ID |
| title | varchar | 100 | 否 | - | 公告标题 |
| content | text | - | 是 | NULL | 公告内容 |
| type | varchar | 50 | 是 | NULL | 公告类型 |
| level | varchar | 10 | 是 | NULL | 公告级别 |
| publish_time | datetime | - | 是 | NULL | 发布时间 |
| create_by | bigint | - | 是 | NULL | 创建人ID |
| create_time | datetime | - | 是 | NULL | 创建时间 |
| update_by | bigint | - | 是 | NULL | 更新人ID |
| update_time | datetime | - | 是 | NULL | 更新时间 |
| status | tinyint | - | 是 | 0 | 状态(1-已发布 0-未发布) |
| is_deleted | tinyint | - | 是 | 0 | 删除标记(0-未删除 1-已删除) |

**索引：**
- 主键：`id`

## 数据库关系设计

### 1. 用户-角色关系

用户与角色是多对多关系，通过`sys_user_role`关联表建立关系：
- 一个用户可以拥有多个角色
- 一个角色可以分配给多个用户

### 2. 角色-菜单关系

角色与菜单是多对多关系，通过`sys_role_menu`关联表建立关系：
- 一个角色可以拥有多个菜单权限
- 一个菜单可以被多个角色使用

### 3. 用户-部门关系

用户与部门是多对一关系：
- 一个用户只能属于一个部门
- 一个部门可以有多个用户

### 4. 部门层级关系

部门表是自关联的树状结构：
- 通过`parent_id`字段建立父子关系
- 通过`tree_path`字段记录完整的层级路径

### 5. 菜单层级关系

菜单表也是自关联的树状结构：
- 通过`parent_id`字段建立父子关系
- 通过`tree_path`字段记录完整的层级路径

## 数据库设计注意事项

1. **软删除**
   - 重要数据表使用`is_deleted`字段实现软删除
   - 查询时需要加入`is_deleted = 0`条件

2. **审计字段**
   - 所有表都包含创建人、创建时间、更新人、更新时间字段
   - 通过AOP实现自动填充

3. **索引设计**
   - 主键采用自增长整数
   - 根据查询需求设计合适的索引
   - 避免过多索引影响写入性能

4. **字段命名**
   - 采用下划线命名法
   - 字段名具有描述性
   - 外键字段名使用关联表名_id格式

5. **数据安全**
   - 密码字段需要加密存储
   - 敏感信息需要加密处理 