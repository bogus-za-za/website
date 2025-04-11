# NestJS 后端项目概述

## 项目简介

本项目是一个基于 NestJS 框架构建的现代化后端服务，采用 TypeScript 开发，提供完整的 API 服务和数据管理功能。项目将参考 youlai-boot 的功能架构，使用 NestJS 重新实现。

## 技术栈

- 核心框架：NestJS 10.x
- 数据库：MySQL 8.x
- ORM 框架：TypeORM
- 缓存：Redis
- 认证：JWT + Session
- API 文档：Swagger
- 日志：Winston
- 配置：ConfigModule
- 验证：class-validator
- 工具库：lodash、moment

## 主要功能

1. 用户认证
   - JWT 认证
   - 角色权限
   - 动态权限
   - 单点登录

2. 系统管理
   - 用户管理
   - 角色管理
   - 菜单管理
   - 部门管理
   - 岗位管理

3. 系统监控
   - 操作日志
   - 登录日志
   - 服务监控
   - 缓存监控

4. 系统工具
   - 数据字典
   - 代码生成
   - 系统接口
   - 缓存管理

## 项目特点

1. 模块化设计
   - 业务模块解耦
   - 功能模块独立
   - 可插拔式架构

2. 安全性
   - 请求加密
   - XSS 防护
   - CSRF 防护
   - 参数校验
   - 接口限流

3. 规范化
   - RESTful API
   - 统一响应
   - 统一异常
   - 参数验证
   - 代码规范

4. 性能优化
   - 数据库优化
   - 缓存策略
   - 日志分级
   - 异步处理

## 开发规范

1. 代码风格
   - TypeScript 规范
   - ESLint 检查
   - Prettier 格式化
   - 注释规范

2. 提交规范
   - Git Flow
   - Commit Message
   - 分支管理
   - 版本控制

3. 文档规范
   - API 文档
   - 数据库文档
   - 部署文档
   - 开发文档 