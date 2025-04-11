# OA管理系统环境搭建文档

## 系统架构概述

本OA管理系统采用前后端分离架构：

- 前端：Vue 3 + TypeScript + Element Plus
- 后端：NestJS + TypeScript + TypeORM
- 数据库：MySQL 8.x
- 缓存：Redis

## 开发环境要求

### 1. 硬件要求

| 项目 | 最低配置 | 推荐配置 |
|------|---------|---------|
| CPU  | 双核    | 四核或更高 |
| 内存 | 8GB     | 16GB或更高 |
| 硬盘 | 20GB可用空间 | 50GB或更高 |

### 2. 软件要求

| 软件 | 版本 | 说明 |
|------|------|------|
| Node.js | 16.x 或更高 | 运行JavaScript环境 |
| pnpm | 7.x 或更高 | 包管理工具 |
| Git | 最新版 | 版本控制工具 |
| Docker | 最新版 | 容器化工具（可选，用于数据库和Redis） |
| Docker Compose | 最新版 | 多容器管理工具（可选） |
| VS Code | 最新版 | 推荐的IDE |

## 环境搭建步骤

### 1. 安装Node.js和PNPM

#### Windows安装方式

1. 下载Node.js安装包：https://nodejs.org/
2. 运行安装包，按照提示完成安装
3. 安装PNPM：
   ```bash
   npm install -g pnpm
   ```

#### Mac安装方式

1. 使用Homebrew安装Node.js：
   ```bash
   brew install node
   ```
2. 安装PNPM：
   ```bash
   npm install -g pnpm
   ```

#### Linux安装方式

1. 使用NVM安装Node.js：
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
   nvm install 16
   ```
2. 安装PNPM：
   ```bash
   npm install -g pnpm
   ```

### 2. 安装Git

#### Windows安装方式

1. 下载Git安装包：https://git-scm.com/download/win
2. 运行安装包，按照提示完成安装

#### Mac安装方式

1. 使用Homebrew安装Git：
   ```bash
   brew install git
   ```

#### Linux安装方式

1. 使用包管理器安装Git：
   ```bash
   # Debian/Ubuntu
   sudo apt-get install git
   
   # CentOS/RHEL
   sudo yum install git
   ```

### 3. 安装Docker和Docker Compose（可选）

如果你希望使用Docker容器运行MySQL和Redis，可以按照以下步骤安装Docker：

#### Windows安装方式

1. 下载Docker Desktop：https://www.docker.com/products/docker-desktop
2. 运行安装包，按照提示完成安装
3. Docker Desktop已经包含了Docker Compose

#### Mac安装方式

1. 下载Docker Desktop：https://www.docker.com/products/docker-desktop
2. 运行安装包，按照提示完成安装
3. Docker Desktop已经包含了Docker Compose

#### Linux安装方式

1. 安装Docker：
   ```bash
   # Debian/Ubuntu
   sudo apt-get update
   sudo apt-get install docker.io
   
   # CentOS/RHEL
   sudo yum install docker
   ```

2. 安装Docker Compose：
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.10.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### 4. 使用Docker Compose配置开发环境

创建`docker-compose.yml`文件，内容如下：

```yaml
version: '3'

services:
  mysql:
    image: mysql:8.0
    container_name: oa-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: oa_system
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      - ./docker/mysql/data:/var/lib/mysql
      - ./docker/mysql/conf:/etc/mysql/conf.d
      - ./docker/mysql/init:/docker-entrypoint-initdb.d
    command: --default-authentication-plugin=mysql_native_password
    networks:
      - oa-network

  redis:
    image: redis:6.2
    container_name: oa-redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - ./docker/redis/data:/data
    command: redis-server --appendonly yes
    networks:
      - oa-network

networks:
  oa-network:
    driver: bridge
```

在项目根目录下创建以下目录结构：

```
docker/
├── mysql/
│   ├── conf/
│   │   └── my.cnf
│   ├── data/
│   └── init/
│       └── init.sql
└── redis/
    └── data/
```

MySQL配置文件`my.cnf`内容：

```ini
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
default-time-zone='+8:00'

[client]
default-character-set=utf8mb4
```

启动Docker容器：

```bash
docker-compose up -d
```

### 5. 克隆项目仓库

```bash
# 克隆前端项目
git clone <前端仓库地址> frontend

# 克隆后端项目
git clone <后端仓库地址> backend
```

### 6. 前端项目初始化

```bash
# 进入前端项目目录
cd frontend

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

### 7. 后端项目初始化

```bash
# 进入后端项目目录
cd backend

# 安装依赖
pnpm install

# 启动开发服务器
pnpm start:dev
```

## 项目配置说明

### 1. 前端项目配置

前端项目的主要配置文件：

- `.env.development`: 开发环境配置
- `.env.production`: 生产环境配置
- `vite.config.ts`: Vite构建工具配置

开发环境配置示例（`.env.development`）：

```
# API基础URL
VITE_API_BASE_URL=/api/v1

# 代理配置
VITE_API_TARGET_URL=http://localhost:3000
```

### 2. 后端项目配置

后端项目的主要配置文件：

- `.env`: 环境变量配置
- `nest-cli.json`: NestJS CLI配置

环境变量配置示例（`.env`）：

```
# 应用配置
APP_PORT=3000
APP_ENV=development

# 数据库配置
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=123456
DB_DATABASE=oa_system

# Redis配置
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT配置
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=86400
```

## 代码规范和提交规范

### 1. 代码规范

前端和后端项目均使用ESLint和Prettier进行代码规范检查和格式化：

```bash
# 运行代码规范检查
pnpm lint

# 运行代码格式化
pnpm format
```

### 2. Git提交规范

项目使用Commitlint规范化Git提交信息，请按照以下格式提交代码：

```
<type>(<scope>): <subject>

<body>

<footer>
```

其中`type`可以是以下类型：

- `feat`: 新功能
- `fix`: 修复Bug
- `docs`: 文档修改
- `style`: 代码格式修改
- `refactor`: 代码重构
- `perf`: 性能优化
- `test`: 测试相关
- `build`: 构建相关
- `ci`: CI配置相关
- `chore`: 其他修改

示例：

```
feat(user): 新增用户详情页面

- 添加用户详情页组件
- 完善用户详情API调用

Closes #123
```

## 常见问题解决

### 1. Node.js版本问题

如果遇到Node.js版本不兼容的问题，推荐使用NVM（Node Version Manager）管理多个Node.js版本：

```bash
# 安装指定的Node.js版本
nvm install 16

# 使用指定的Node.js版本
nvm use 16
```

### 2. 权限问题

在Linux或Mac系统中，如果遇到权限问题，可以尝试以下命令：

```bash
# 修改目录权限
sudo chown -R $(whoami) frontend backend
```

### 3. 网络问题

如果在安装依赖时遇到网络问题，可以尝试配置国内镜像：

```bash
# 配置PNPM镜像
pnpm config set registry https://registry.npmmirror.com
```

## 部署指南

### 1. 前端部署

```bash
# 构建生产环境代码
cd frontend
pnpm build

# 将dist目录部署到Web服务器
```

### 2. 后端部署

```bash
# 构建生产环境代码
cd backend
pnpm build

# 启动生产环境服务
pnpm start:prod
```

### 3. 使用Docker部署（推荐）

前端Dockerfile示例：

```dockerfile
FROM node:16-alpine as build
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install
COPY . .
RUN pnpm build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

后端Dockerfile示例：

```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install
COPY . .
RUN pnpm build
EXPOSE 3000
CMD ["pnpm", "start:prod"]
```

部署Docker Compose配置示例：

```yaml
version: '3'

services:
  frontend:
    build: ./frontend
    restart: always
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - oa-network

  backend:
    build: ./backend
    restart: always
    ports:
      - "3000:3000"
    depends_on:
      - mysql
      - redis
    environment:
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USERNAME=root
      - DB_PASSWORD=123456
      - DB_DATABASE=oa_system
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    networks:
      - oa-network

  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: oa_system
      TZ: Asia/Shanghai
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - oa-network

  redis:
    image: redis:6.2
    restart: always
    volumes:
      - redis-data:/data
    networks:
      - oa-network

volumes:
  mysql-data:
  redis-data:

networks:
  oa-network:
    driver: bridge
```

启动容器：

```bash
docker-compose -f docker-compose.prod.yml up -d
``` 