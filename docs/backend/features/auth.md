# 认证授权设计文档

## 认证设计

### 1. 认证流程

本系统采用基于JWT (JSON Web Token) 的认证机制，流程如下：

```
┌───────────┐       ┌───────────┐      ┌───────────┐
│           │       │           │      │           │
│  客户端   │       │  服务端   │      │  数据库   │
│           │       │           │      │           │
└─────┬─────┘       └─────┬─────┘      └─────┬─────┘
      │                   │                  │
      │ 1.登录请求(用户名/密码) │                  │
      │─────────────────>│                  │
      │                   │                  │
      │                   │ 2.查询用户信息    │
      │                   │─────────────────>│
      │                   │                  │
      │                   │ 3.返回用户信息    │
      │                   │<─────────────────│
      │                   │                  │
      │                   │ 4.验证密码       │
      │                   │────┐             │
      │                   │    │             │
      │                   │<───┘             │
      │                   │                  │
      │                   │ 5.生成JWT Token  │
      │                   │────┐             │
      │                   │    │             │
      │                   │<───┘             │
      │                   │                  │
      │ 6.返回Token        │                  │
      │<─────────────────│                  │
      │                   │                  │
      │ 7.后续请求携带Token │                  │
      │─────────────────>│                  │
      │                   │                  │
      │                   │ 8.验证Token      │
      │                   │────┐             │
      │                   │    │             │
      │                   │<───┘             │
      │                   │                  │
      │ 9.返回响应         │                  │
      │<─────────────────│                  │
      │                   │                  │
```

### 2. JWT 结构设计

JWT 由三部分组成：Header（头部）、Payload（载荷）、Signature（签名）。

#### Header

```json
{
  "alg": "HS256",  // 算法
  "typ": "JWT"     // 类型
}
```

#### Payload

```json
{
  "sub": "1",              // 主题(用户ID)
  "username": "admin",     // 用户名
  "roles": ["ADMIN"],      // 角色
  "iat": 1622505600,       // 签发时间
  "exp": 1622592000        // 过期时间
}
```

#### Signature

签名由以下部分组成：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

### 3. Token 存储与刷新

- **存储位置**：客户端存储在 localStorage 或 sessionStorage 中
- **有效期**：默认 24 小时
- **刷新机制**：
  - 方案一：使用刷新令牌（Refresh Token）
  - 方案二：Token 过期时，用户重新登录获取新 Token

## 授权设计

### 1. RBAC 模型

系统采用基于角色的访问控制（Role-Based Access Control，RBAC）模型：

```
┌───────────┐       ┌───────────┐       ┌───────────┐
│           │       │           │       │           │
│   用户    │───────│   角色    │───────│   权限    │
│           │       │           │       │           │
└───────────┘       └───────────┘       └───────────┘
```

- **用户（User）**：系统的使用者
- **角色（Role）**：用户的分类，如管理员、普通用户等
- **权限（Permission）**：系统中的操作权限，如增删改查等

### 2. 权限层级

系统权限分为四个层级：

1. **菜单权限**：控制用户可访问的页面
2. **按钮权限**：控制用户可操作的按钮
3. **数据权限**：控制用户可访问的数据范围
4. **API权限**：控制用户可调用的API接口

### 3. 权限标识格式

权限标识采用以下格式：`模块:资源:操作`

例如：
- `sys:user:add`：用户管理-新增用户
- `sys:role:edit`：角色管理-编辑角色
- `sys:menu:delete`：菜单管理-删除菜单

### 4. 权限验证流程

```
┌───────────┐       ┌───────────┐       ┌───────────┐
│           │       │           │       │           │
│  客户端   │       │  服务端   │       │  数据库   │
│           │       │           │       │           │
└─────┬─────┘       └─────┬─────┘       └─────┬─────┘
      │                   │                   │
      │ 1.请求访问资源     │                   │
      │─────────────────>│                   │
      │                   │                   │
      │                   │ 2.解析Token获取用户ID │
      │                   │────┐              │
      │                   │    │              │
      │                   │<───┘              │
      │                   │                   │
      │                   │ 3.查询用户角色和权限 │
      │                   │──────────────────>│
      │                   │                   │
      │                   │ 4.返回角色和权限    │
      │                   │<──────────────────│
      │                   │                   │
      │                   │ 5.检查是否有权限    │
      │                   │────┐              │
      │                   │    │              │
      │                   │<───┘              │
      │                   │                   │
      │ 6.有权限：返回数据  │                   │
      │ 无权限：返回403错误 │                   │
      │<─────────────────│                   │
      │                   │                   │
```

## 前端权限控制

### 1. 路由权限控制

采用动态路由方案，根据用户角色和权限动态生成可访问的路由表：

```typescript
// 路由守卫实现
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore();
  
  // 判断是否有token
  if (userStore.hasToken) {
    if (to.path === '/login') {
      // 已登录状态访问登录页，重定向到首页
      next({ path: '/' });
    } else {
      // 判断是否已获取用户信息
      if (userStore.hasUserInfo) {
        next();
      } else {
        try {
          // 获取用户信息
          await userStore.getUserInfo();
          
          // 根据权限生成可访问路由
          const permissionStore = usePermissionStore();
          const accessRoutes = await permissionStore.generateRoutes();
          
          // 动态添加路由
          accessRoutes.forEach(route => {
            router.addRoute(route);
          });
          
          // 继续路由导航
          next({ ...to, replace: true });
        } catch (error) {
          // 获取用户信息失败，重置token并跳转到登录页
          await userStore.resetToken();
          next(`/login?redirect=${to.path}`);
        }
      }
    }
  } else {
    // 未登录状态访问白名单页面放行
    if (whiteList.includes(to.path)) {
      next();
    } else {
      // 其他页面重定向到登录页
      next(`/login?redirect=${to.path}`);
    }
  }
});
```

### 2. 按钮权限控制

使用自定义指令控制按钮的显示隐藏：

```typescript
// 权限指令定义
const hasPermission = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const { value } = binding;
    const userStore = useUserStore();
    const permissions = userStore.permissions;
    
    if (value && value instanceof Array && value.length > 0) {
      const hasPermission = permissions.some(permission => {
        return value.includes(permission);
      });
      
      if (!hasPermission) {
        el.parentNode?.removeChild(el);
      }
    } else {
      throw new Error('need permissions! Like v-has-permission="[\'sys:user:add\',\'sys:user:edit\']"');
    }
  }
};

// 在组件中使用
<el-button v-has-permission="['sys:user:add']" type="primary">新增用户</el-button>
```

## 后端权限控制

### 1. 接口权限控制

使用守卫（Guard）和装饰器（Decorator）实现接口级别的权限控制：

```typescript
// 权限装饰器
@SetMetadata('permissions', permissions)
export const Permissions = (...permissions: string[]) => SetMetadata('permissions', permissions);

// 权限守卫
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly authService: AuthService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // 获取路由所需权限
    const requiredPermissions = this.reflector.get<string[]>(
      'permissions',
      context.getHandler(),
    );
    
    // 如果没有设置权限，直接放行
    if (!requiredPermissions || requiredPermissions.length === 0) {
      return true;
    }
    
    // 获取请求对象
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // 获取用户权限
    const userPermissions = await this.authService.getUserPermissions(user.id);
    
    // 判断用户是否拥有所需权限
    return requiredPermissions.some(permission => userPermissions.includes(permission));
  }
}

// 在控制器中使用
@Post()
@UseGuards(JwtAuthGuard, PermissionsGuard)
@Permissions('sys:user:add')
async create(@Body() createUserDto: CreateUserDto) {
  return this.userService.create(createUserDto);
}
```

### 2. 数据权限控制

基于用户的角色和部门，控制用户可访问的数据范围：

```typescript
@Injectable()
export class DataScopeInterceptor implements NestInterceptor {
  constructor(private readonly authService: AuthService) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // 获取用户的数据权限范围
    const dataScope = await this.authService.getUserDataScope(user.id);
    
    // 将数据权限范围添加到请求中
    request.dataScope = dataScope;
    
    return next.handle();
  }
}

// 在服务中应用数据权限
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserEntity)
    private readonly userRepository: Repository<UserEntity>,
  ) {}

  async findAll(userQueryDto: UserQueryDto, dataScope: DataScope): Promise<PaginationResult<UserEntity>> {
    const queryBuilder = this.userRepository.createQueryBuilder('user');
    
    // 应用数据权限
    if (dataScope.type === 'ALL') {
      // 全部数据权限，不做限制
    } else if (dataScope.type === 'DEPT') {
      // 本部门数据权限
      queryBuilder.andWhere('user.deptId = :deptId', { deptId: dataScope.deptId });
    } else if (dataScope.type === 'DEPT_AND_CHILD') {
      // 本部门及以下数据权限
      queryBuilder.andWhere('user.deptId IN (:...deptIds)', { deptIds: dataScope.deptIds });
    } else if (dataScope.type === 'CUSTOM') {
      // 自定义数据权限
      queryBuilder.andWhere('user.deptId IN (:...deptIds)', { deptIds: dataScope.customDeptIds });
    } else if (dataScope.type === 'SELF') {
      // 仅本人数据权限
      queryBuilder.andWhere('user.createBy = :userId', { userId: dataScope.userId });
    }
    
    // 继续查询逻辑...
  }
}
```

## 安全防护措施

### 1. 密码加密

用户密码使用BCrypt算法进行加密存储：

```typescript
// 密码加密
export const hashPassword = async (password: string): Promise<string> => {
  const salt = await bcrypt.genSalt(10);
  return bcrypt.hash(password, salt);
};

// 密码验证
export const comparePassword = async (
  password: string,
  hashedPassword: string,
): Promise<boolean> => {
  return bcrypt.compare(password, hashedPassword);
};
```

### 2. CSRF 防护

采用 CSRF Token 机制防止跨站请求伪造：

```typescript
import * as csurf from 'csurf';

// 配置CSRF保护
const csrfProtection = csurf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
  },
});

// 应用CSRF中间件
app.use(csrfProtection);

// 提供CSRF Token
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});
```

### 3. XSS 防护

使用 helmet 中间件集增强安全头：

```typescript
import helmet from 'helmet';

// 应用安全头
app.use(helmet());
```

### 4. 接口限流

使用 throttler 中间件实现接口限流：

```typescript
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60,              // 时间窗口（秒）
      limit: 100,           // 在时间窗口内的最大请求数
    }),
  ],
})
export class AppModule {}

// 在控制器中应用
@UseGuards(ThrottlerGuard)
@Controller('api/v1/auth')
export class AuthController {
  // 对特定接口设置不同的限流规则
  @Throttle(5, 60) // 60秒内最多5次请求
  @Post('login')
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

## 敏感操作保护

### 1. 敏感操作日志

对敏感操作进行日志记录和审计：

```typescript
// 操作日志装饰器
export const OperationLog = (module: string, operation: string) => {
  return applyDecorators(
    SetMetadata('operation_log', { module, operation }),
  );
};

// 操作日志拦截器
@Injectable()
export class OperationLogInterceptor implements NestInterceptor {
  constructor(private readonly logService: LogService) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // 获取操作日志元数据
    const handler = context.getHandler();
    const operationLog = Reflect.getMetadata('operation_log', handler);
    
    if (!operationLog) {
      return next.handle();
    }
    
    const { module, operation } = operationLog;
    const method = handler.name;
    const requestUrl = request.url;
    const requestMethod = request.method;
    const requestParams = { ...request.body, ...request.query, ...request.params };
    const ip = request.ip;
    const userId = user?.id;
    
    // 记录请求时间
    const startTime = Date.now();
    
    return next.handle().pipe(
      tap(
        async (response) => {
          // 成功响应
          const requestTime = Date.now() - startTime;
          await this.logService.createOperationLog({
            module,
            operation,
            type: 1, // 成功
            method,
            requestUrl,
            requestMethod,
            requestParams: JSON.stringify(requestParams),
            requestTime,
            ip,
            userId,
            status: 1,
            errorMsg: null,
          });
        },
        async (error) => {
          // 错误响应
          const requestTime = Date.now() - startTime;
          await this.logService.createOperationLog({
            module,
            operation,
            type: 2, // 失败
            method,
            requestUrl,
            requestMethod,
            requestParams: JSON.stringify(requestParams),
            requestTime,
            ip,
            userId,
            status: 0,
            errorMsg: error.message,
          });
        },
      ),
    );
  }
}

// 在控制器中使用
@Post()
@UseGuards(JwtAuthGuard, PermissionsGuard)
@Permissions('sys:user:add')
@OperationLog('用户管理', '新增用户')
async create(@Body() createUserDto: CreateUserDto) {
  return this.userService.create(createUserDto);
}
```

### 2. 敏感操作二次验证

对敏感操作进行二次验证保护：

```typescript
// 二次验证装饰器
export const SecondVerify = () => SetMetadata('second_verify', true);

// 二次验证守卫
@Injectable()
export class SecondVerifyGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly authService: AuthService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // 获取是否需要二次验证
    const requireSecondVerify = this.reflector.get<boolean>(
      'second_verify',
      context.getHandler(),
    );
    
    // 如果不需要二次验证，直接放行
    if (!requireSecondVerify) {
      return true;
    }
    
    // 获取请求对象
    const request = context.switchToHttp().getRequest();
    const verifyToken = request.headers['x-verify-token'];
    
    // 验证二次验证令牌
    if (!verifyToken) {
      throw new UnauthorizedException('需要二次验证');
    }
    
    return this.authService.verifySecondVerifyToken(verifyToken);
  }
}

// 在控制器中使用
@Delete(':id')
@UseGuards(JwtAuthGuard, PermissionsGuard, SecondVerifyGuard)
@Permissions('sys:user:delete')
@SecondVerify()
@OperationLog('用户管理', '删除用户')
async remove(@Param('id') id: string) {
  return this.userService.remove(+id);
}
``` 