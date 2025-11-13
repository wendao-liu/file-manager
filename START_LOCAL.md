# 本地启动指南

本指南将帮助你在本地环境中启动和验证文件管理系统。

## 方式一：使用 Docker Compose（推荐）

这是最简单的方式，所有服务都会在 Docker 容器中运行。

### 前置要求
- Docker 和 Docker Compose 已安装
- 确保 Docker 服务正在运行

### 步骤

1. **创建必要的目录**
```bash
mkdir -p ~/middleware/postgres/data
mkdir -p ~/middleware/minio/data
```

2. **创建 `.env` 文件**
```bash
# 如果还没有 .env 文件，从示例文件复制
cp .env.example .env
```

然后编辑 `.env` 文件，确保配置正确：
```env
DATABASE_URL=postgresql+asyncpg://root:123456@postgres:5432/doc_management
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=root
MINIO_SECRET_KEY=12345678
MINIO_BUCKET_NAME=documents
JWT_SECRET_KEY=your-secret-key-change-this-in-production
```

3. **启动所有服务**
```bash
# 在项目根目录执行
docker-compose -f docker/docker-compose.yml up -d
```

4. **查看服务状态**
```bash
docker-compose -f docker/docker-compose.yml ps
```

5. **查看日志（如果需要）**
```bash
# 查看所有服务日志
docker-compose -f docker/docker-compose.yml logs -f

# 查看特定服务日志
docker-compose -f docker/docker-compose.yml logs -f backend
docker-compose -f docker/docker-compose.yml logs -f frontend
```

6. **访问应用**
- 前端：http://localhost:31000
- 后端 API：http://localhost:8002
- API 文档（Swagger）：http://localhost:8002/docs
- MinIO 控制台：http://localhost:9001
  - 用户名：root
  - 密码：12345678

7. **停止服务**
```bash
docker-compose -f docker/docker-compose.yml down
```

---

## 方式二：本地开发模式

这种方式适合开发和调试，前端和后端在本地运行，但数据库和 MinIO 仍使用 Docker。

### 前置要求
- Python 3.9+ 和 PDM
- Node.js 20+ 和 npm
- Docker（用于运行 PostgreSQL 和 MinIO）

### 步骤

#### 1. 启动基础服务（PostgreSQL 和 MinIO）

```bash
# 只启动数据库和 MinIO
docker-compose -f docker/docker-compose.yml up -d postgres minio
```

#### 2. 配置后端

创建 `.env` 文件（如果还没有）：
```env
DATABASE_URL=postgresql+asyncpg://root:123456@localhost:5433/doc_management
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=root
MINIO_SECRET_KEY=12345678
MINIO_BUCKET_NAME=documents
JWT_SECRET_KEY=your-secret-key-change-this-in-production
```

安装后端依赖：
```bash
# 安装 PDM（如果还没有）
pip install pdm

# 安装项目依赖
pdm install
```

启动后端服务：
```bash
# 在项目根目录执行
pdm run python -m uvicorn src.main:app --reload --host 0.0.0.0 --port 8002
```

后端将在 http://localhost:8002 运行

#### 3. 配置前端

```bash
cd frontend

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

前端将在 http://localhost:5173 运行（Vite 默认端口）

#### 4. 验证

- 前端：http://localhost:5173
- 后端 API：http://localhost:8002
- API 文档：http://localhost:8002/docs
- MinIO 控制台：http://localhost:9001

---

## 验证步骤

### 1. 检查服务是否正常运行

**检查 Docker 容器：**
```bash
docker ps
```

应该看到 `postgres` 和 `minio` 容器在运行。

**检查后端 API：**
```bash
curl http://localhost:8002/docs
```

**检查前端：**
在浏览器中打开 http://localhost:3001（Docker 模式）或 http://localhost:5173（开发模式）

### 2. 测试用户注册和登录

1. 访问前端页面
2. 点击"注册"创建新用户
3. 使用注册的账号登录
4. 测试文件上传功能

### 3. 检查 API 文档

访问 http://localhost:8002/docs 查看 Swagger UI，可以：
- 查看所有可用的 API 端点
- 测试 API 调用
- 查看请求/响应格式

### 4. 检查 MinIO

访问 http://localhost:9001：
- 使用 root/12345678 登录
- 检查 `documents` bucket 是否存在
- 上传文件后，可以在 MinIO 控制台看到文件

---

## 常见问题

### 1. 端口被占用

如果端口被占用，可以：
- 修改 `docker/docker-compose.yml` 中的端口映射
- 或者停止占用端口的服务

### 2. 数据库连接失败

确保：
- PostgreSQL 容器正在运行
- `.env` 文件中的 `DATABASE_URL` 配置正确
- 如果是本地开发模式，使用 `localhost` 而不是 `postgres`

### 3. MinIO 连接失败

确保：
- MinIO 容器正在运行
- `.env` 文件中的 MinIO 配置正确
- 如果是本地开发模式，使用 `localhost:9000` 而不是 `minio:9000`

### 4. 前端无法连接后端

检查：
- 后端服务是否正在运行
- `frontend/vite.config.ts` 中的代理配置是否正确
- 浏览器控制台是否有错误信息

### 5. 权限问题（macOS/Linux）

如果遇到目录权限问题：
```bash
sudo chown -R $USER ~/middleware
```

---

## 清理

停止并删除所有容器和数据：
```bash
docker-compose -f docker/docker-compose.yml down -v
```

删除数据目录（注意：这会删除所有数据）：
```bash
rm -rf ~/middleware/postgres/data
rm -rf ~/middleware/minio/data
```

