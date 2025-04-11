# NestJS 后端项目结构说明

## 目录结构

```
backend/
├── src/ # 源代码目录
│   ├── config/ # 配置模块
│   ├── modules/ # 业务模块
│   │   ├── auth/ # 认证模块
│   │   ├── system/ # 系统管理模块
│   │   └── monitor/ # 系统监控模块
│   ├── shared/ # 共享模块
│   │   ├── decorators/ # 自定义装饰器
│   │   ├── filters/ # 全局过滤器
│   │   ├── guards/ # 全局守卫
│   │   ├── interceptors/ # 全局拦截器
│   │   ├── middlewares/ # 全局中间件
│   │   ├── pipes/ # 全局管道
│   │   └── utils/ # 工具函数
│   ├── common/ # 通用模块
│   │   ├── constants/ # 常量定义
│   │   ├── dto/ # 数据传输对象
│   │   ├── entities/ # 实体定义
│   │   ├── enums/ # 枚举定义
│   │   └── interfaces/ # 接口定义
│   ├── core/ # 核心模块
│   │   ├── database/ # 数据库配置
│   │   ├── cache/ # 缓存配置
│   │   ├── logger/ # 日志配置
│   │   └── scheduler/ # 定时任务
│   ├── app.module.ts # 应用模块
│   └── main.ts # 入口文件
├── test/ # 测试目录
├── dist/ # 编译输出目录
├── node_modules/ # 依赖目录
├── nest-cli.json # Nest CLI配置
├── package.json # 项目依赖
└── tsconfig.json # TypeScript配置
```

## 核心模块说明

### 1. auth 认证模块

认证模块负责用户登录、注销、权限验证等功能。

```
modules/auth/
├── controllers/ # 控制器
│   └── auth.controller.ts # 认证控制器
├── dto/ # 数据传输对象
│   ├── login.dto.ts # 登录DTO
│   └── register.dto.ts # 注册DTO
├── guards/ # 守卫
│   └── jwt-auth.guard.ts # JWT认证守卫
├── services/ # 服务
│   └── auth.service.ts # 认证服务
├── strategies/ # 策略
│   └── jwt.strategy.ts # JWT策略
└── auth.module.ts # 认证模块
```

### 2. system 系统管理模块

系统管理模块包含用户、角色、菜单、部门、岗位等管理功能。

```
modules/system/
├── controllers/ # 控制器
│   ├── user.controller.ts # 用户控制器
│   ├── role.controller.ts # 角色控制器
│   ├── menu.controller.ts # 菜单控制器
│   ├── dept.controller.ts # 部门控制器
│   └── post.controller.ts # 岗位控制器
├── dto/ # 数据传输对象
│   ├── user/ # 用户DTO
│   ├── role/ # 角色DTO
│   ├── menu/ # 菜单DTO
│   ├── dept/ # 部门DTO
│   └── post/ # 岗位DTO
├── entities/ # 实体
│   ├── user.entity.ts # 用户实体
│   ├── role.entity.ts # 角色实体
│   ├── menu.entity.ts # 菜单实体
│   ├── dept.entity.ts # 部门实体
│   └── post.entity.ts # 岗位实体
├── services/ # 服务
│   ├── user.service.ts # 用户服务
│   ├── role.service.ts # 角色服务
│   ├── menu.service.ts # 菜单服务
│   ├── dept.service.ts # 部门服务
│   └── post.service.ts # 岗位服务
└── system.module.ts # 系统模块
```

### 3. monitor 系统监控模块

系统监控模块包含操作日志、登录日志、服务监控、缓存监控等功能。

```
modules/monitor/
├── controllers/ # 控制器
│   ├── operation-log.controller.ts # 操作日志控制器
│   ├── login-log.controller.ts # 登录日志控制器
│   ├── server.controller.ts # 服务监控控制器
│   └── cache.controller.ts # 缓存监控控制器
├── dto/ # 数据传输对象
│   ├── operation-log/ # 操作日志DTO
│   └── login-log/ # 登录日志DTO
├── entities/ # 实体
│   ├── operation-log.entity.ts # 操作日志实体
│   └── login-log.entity.ts # 登录日志实体
├── services/ # 服务
│   ├── operation-log.service.ts # 操作日志服务
│   ├── login-log.service.ts # 登录日志服务
│   ├── server.service.ts # 服务监控服务
│   └── cache.service.ts # 缓存监控服务
└── monitor.module.ts # 监控模块
```

### 4. shared 共享模块

共享模块包含全局使用的装饰器、过滤器、守卫、拦截器、中间件、管道和工具函数。

```
shared/
├── decorators/ # 自定义装饰器
│   ├── api-paginated-response.decorator.ts # 分页响应装饰器
│   ├── current-user.decorator.ts # 当前用户装饰器
│   └── permissions.decorator.ts # 权限装饰器
├── filters/ # 全局过滤器
│   └── http-exception.filter.ts # HTTP异常过滤器
├── guards/ # 全局守卫
│   └── permissions.guard.ts # 权限守卫
├── interceptors/ # 全局拦截器
│   ├── logging.interceptor.ts # 日志拦截器
│   └── transform.interceptor.ts # 响应转换拦截器
├── middlewares/ # 全局中间件
│   └── jwt.middleware.ts # JWT中间件
├── pipes/ # 全局管道
│   └── validation.pipe.ts # 验证管道
└── utils/ # 工具函数
    ├── crypto.util.ts # 加密工具
    └── date.util.ts # 日期工具
```

### 5. common 通用模块

通用模块包含常量、DTO、实体、枚举和接口定义。

```
common/
├── constants/ # 常量定义
│   ├── error-code.constant.ts # 错误代码常量
│   └── system.constant.ts # 系统常量
├── dto/ # 数据传输对象
│   ├── pagination.dto.ts # 分页DTO
│   └── response.dto.ts # 响应DTO
├── entities/ # 实体定义
│   └── base.entity.ts # 基础实体
├── enums/ # 枚举定义
│   ├── login-type.enum.ts # 登录类型枚举
│   └── status.enum.ts # 状态枚举
└── interfaces/ # 接口定义
    ├── pagination.interface.ts # 分页接口
    └── response.interface.ts # 响应接口
```

### 6. core 核心模块

核心模块包含数据库、缓存、日志和定时任务等配置。

```
core/
├── database/ # 数据库配置
│   ├── database.module.ts # 数据库模块
│   └── database.providers.ts # 数据库提供者
├── cache/ # 缓存配置
│   ├── cache.module.ts # 缓存模块
│   └── cache.service.ts # 缓存服务
├── logger/ # 日志配置
│   ├── logger.module.ts # 日志模块
│   └── logger.service.ts # 日志服务
└── scheduler/ # 定时任务
    ├── scheduler.module.ts # 定时任务模块
    └── tasks.service.ts # 任务服务
```

## 代码规范

### 1. 命名规范

- 文件名：使用kebab-case，如`user.service.ts`
- 类名：使用PascalCase，如`UserService`
- 方法名：使用camelCase，如`getUserById`
- 属性名：使用camelCase，如`firstName`
- 常量名：使用UPPER_SNAKE_CASE，如`MAX_CONNECTION`

### 2. 目录结构规范

- 按功能模块划分目录
- 按文件类型组织子目录
- 保持目录结构统一

### 3. 编码规范

- 使用async/await代替Promise链
- 使用TypeScript类型系统
- 优先使用接口定义数据结构
- 使用依赖注入管理依赖
- 异常统一处理

### 4. 数据库操作规范

- 使用TypeORM进行数据库操作
- 定义清晰的实体关系
- 使用事务保证数据一致性
- 合理使用索引优化查询
- 避免N+1查询问题

### 5. API设计规范

- 遵循RESTful设计原则
- 使用HTTP动词表示操作类型
- 合理使用HTTP状态码
- 统一API响应格式
- 使用DTO验证请求数据

## 技术实现示例

### 1. 控制器示例

```typescript
// modules/system/controllers/user.controller.ts
import { Controller, Get, Post, Body, Param, Query, Delete, Put, UseGuards } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';
import { UserService } from '../services/user.service';
import { CreateUserDto, UpdateUserDto, UserQueryDto } from '../dto/user';
import { JwtAuthGuard } from '../../auth/guards/jwt-auth.guard';
import { PermissionsGuard } from '../../../shared/guards/permissions.guard';
import { Permissions } from '../../../shared/decorators/permissions.decorator';
import { ApiPaginatedResponse } from '../../../shared/decorators/api-paginated-response.decorator';
import { UserEntity } from '../entities/user.entity';

@ApiTags('用户管理')
@Controller('system/users')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @ApiOperation({ summary: '创建用户' })
  @ApiResponse({ status: 201, description: '创建成功', type: UserEntity })
  @Permissions('system:user:add')
  async create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @Get()
  @ApiOperation({ summary: '查询用户列表' })
  @ApiPaginatedResponse(UserEntity)
  @Permissions('system:user:list')
  async findAll(@Query() userQueryDto: UserQueryDto) {
    return this.userService.findAll(userQueryDto);
  }

  @Get(':id')
  @ApiOperation({ summary: '查询用户详情' })
  @ApiResponse({ status: 200, description: '查询成功', type: UserEntity })
  @Permissions('system:user:query')
  async findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);
  }

  @Put(':id')
  @ApiOperation({ summary: '更新用户' })
  @ApiResponse({ status: 200, description: '更新成功', type: UserEntity })
  @Permissions('system:user:edit')
  async update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.userService.update(+id, updateUserDto);
  }

  @Delete(':id')
  @ApiOperation({ summary: '删除用户' })
  @ApiResponse({ status: 200, description: '删除成功' })
  @Permissions('system:user:delete')
  async remove(@Param('id') id: string) {
    return this.userService.remove(+id);
  }
}
```

### 2. 服务示例

```typescript
// modules/system/services/user.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { UserEntity } from '../entities/user.entity';
import { CreateUserDto, UpdateUserDto, UserQueryDto } from '../dto/user';
import { PaginationResult } from '../../../common/interfaces/pagination.interface';
import { hashPassword } from '../../../shared/utils/crypto.util';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserEntity)
    private readonly userRepository: Repository<UserEntity>,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<UserEntity> {
    const { password, ...userData } = createUserDto;
    
    // 密码加密
    const hashedPassword = await hashPassword(password);
    
    const user = this.userRepository.create({
      ...userData,
      password: hashedPassword,
    });
    
    return this.userRepository.save(user);
  }

  async findAll(userQueryDto: UserQueryDto): Promise<PaginationResult<UserEntity>> {
    const { page = 1, size = 10, username, status, deptId } = userQueryDto;
    
    const queryBuilder = this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.roles', 'role')
      .leftJoinAndSelect('user.dept', 'dept')
      .leftJoinAndSelect('user.post', 'post');
    
    if (username) {
      queryBuilder.andWhere('user.username LIKE :username', { username: `%${username}%` });
    }
    
    if (status !== undefined) {
      queryBuilder.andWhere('user.status = :status', { status });
    }
    
    if (deptId) {
      queryBuilder.andWhere('user.deptId = :deptId', { deptId });
    }
    
    const total = await queryBuilder.getCount();
    
    const users = await queryBuilder
      .skip((page - 1) * size)
      .take(size)
      .orderBy('user.createTime', 'DESC')
      .getMany();
    
    return {
      list: users,
      total,
      page,
      size,
    };
  }

  async findOne(id: number): Promise<UserEntity> {
    const user = await this.userRepository.findOne({
      where: { id },
      relations: ['roles', 'dept', 'post'],
    });
    
    if (!user) {
      throw new NotFoundException(`用户ID为${id}的用户不存在`);
    }
    
    return user;
  }

  async update(id: number, updateUserDto: UpdateUserDto): Promise<UserEntity> {
    const user = await this.findOne(id);
    
    const { password, ...userData } = updateUserDto;
    
    // 如果有密码，进行加密
    if (password) {
      userData['password'] = await hashPassword(password);
    }
    
    Object.assign(user, userData);
    
    return this.userRepository.save(user);
  }

  async remove(id: number): Promise<void> {
    const user = await this.findOne(id);
    
    await this.userRepository.remove(user);
  }
}
```

### 3. 实体示例

```typescript
// modules/system/entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, ManyToMany, JoinTable, ManyToOne, JoinColumn } from 'typeorm';
import { ApiProperty } from '@nestjs/swagger';
import { Exclude } from 'class-transformer';
import { RoleEntity } from './role.entity';
import { DeptEntity } from './dept.entity';
import { PostEntity } from './post.entity';

@Entity('sys_user')
export class UserEntity {
  @ApiProperty({ description: '用户ID' })
  @PrimaryGeneratedColumn()
  id: number;

  @ApiProperty({ description: '用户名' })
  @Column({ length: 50, unique: true })
  username: string;

  @ApiProperty({ description: '昵称' })
  @Column({ length: 50 })
  nickname: string;

  @Exclude()
  @Column({ length: 100 })
  password: string;

  @ApiProperty({ description: '手机号' })
  @Column({ length: 11, nullable: true })
  mobile: string;

  @ApiProperty({ description: '邮箱' })
  @Column({ length: 100, nullable: true })
  email: string;

  @ApiProperty({ description: '性别(1:男;2:女;)' })
  @Column({ type: 'tinyint', default: 1 })
  gender: number;

  @ApiProperty({ description: '头像' })
  @Column({ length: 255, nullable: true })
  avatar: string;

  @ApiProperty({ description: '状态(1:启用;0:禁用)' })
  @Column({ type: 'tinyint', default: 1 })
  status: number;

  @ApiProperty({ description: '部门ID' })
  @Column({ name: 'dept_id', nullable: true })
  deptId: number;

  @ApiProperty({ description: '创建者' })
  @Column({ name: 'create_by', length: 50, nullable: true })
  createBy: string;

  @ApiProperty({ description: '创建时间' })
  @CreateDateColumn({ name: 'create_time' })
  createTime: Date;

  @ApiProperty({ description: '更新者' })
  @Column({ name: 'update_by', length: 50, nullable: true })
  updateBy: string;

  @ApiProperty({ description: '更新时间' })
  @UpdateDateColumn({ name: 'update_time' })
  updateTime: Date;

  @ApiProperty({ description: '是否删除(0:未删除;1:已删除)' })
  @Column({ name: 'is_deleted', type: 'tinyint', default: 0 })
  isDeleted: number;

  @ApiProperty({ description: '角色列表', type: [RoleEntity] })
  @ManyToMany(() => RoleEntity, role => role.users)
  @JoinTable({
    name: 'sys_user_role',
    joinColumn: { name: 'user_id', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'role_id', referencedColumnName: 'id' },
  })
  roles: RoleEntity[];

  @ApiProperty({ description: '部门', type: DeptEntity })
  @ManyToOne(() => DeptEntity)
  @JoinColumn({ name: 'dept_id' })
  dept: DeptEntity;

  @ApiProperty({ description: '岗位', type: PostEntity })
  @ManyToOne(() => PostEntity)
  @JoinColumn({ name: 'post_id' })
  post: PostEntity;
}
```

## 模块依赖关系

模块之间的依赖关系如下：

1. AppModule 依赖 CoreModule, AuthModule, SystemModule, MonitorModule
2. AuthModule 依赖 SharedModule, SystemModule(UserService)
3. SystemModule 依赖 SharedModule
4. MonitorModule 依赖 SharedModule
5. SharedModule 依赖 CoreModule

这种依赖关系确保了模块之间的低耦合性和高内聚性，使得系统更加易于维护和扩展。 