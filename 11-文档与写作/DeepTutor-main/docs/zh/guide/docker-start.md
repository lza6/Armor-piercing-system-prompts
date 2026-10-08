# Docker 部署

Docker 部署无需 Python 或 Node.js 设置——一切都在容器中运行。

## 前提条件

- **Docker** — [安装 Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** — [安装 Docker Compose](https://docs.docker.com/compose/install/)

## 快速启动选项

选择您喜欢的部署方式：

<div class="method-tabs">

### 选项 A：从源码构建

从源代码本地构建并运行 DeepTutor。

```bash
# 克隆仓库（如果尚未完成）
git clone https://github.com/HKUDS/DeepTutor.git
cd DeepTutor

# 创建并配置 .env 文件
cp .env.example .env
# 编辑 .env 填入您的 API 密钥

# 构建并启动
docker compose up

# 首次运行在 Mac Mini M4 上约需 11 分钟
```

#### 常用命令

```bash
# 后台启动
docker compose up -d

# 停止服务
docker compose down

# 查看日志
docker compose logs -f

# 代码更改后重建
docker compose up --build

# 清除缓存并重建
docker compose build --no-cache
```

</div>

<div class="method-tabs">

### 选项 B：拉取预构建镜像（更快）

使用我们在 GitHub Container Registry 上的官方预构建镜像。

```bash
# 适用于所有平台 - Docker 自动检测您的架构
docker run -d --name deeptutor \
  -p 8001:8001 -p 3782:3782 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  -v $(pwd)/config:/app/config:ro \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning Windows PowerShell
使用 `${PWD}` 代替 `$(pwd)`：

```powershell
docker run -d --name deeptutor `
  -p 8001:8001 -p 3782:3782 `
  --env-file .env `
  -v ${PWD}/data:/app/data `
  -v ${PWD}/config:/app/config:ro `
  ghcr.io/hkuds/deeptutor:latest
```
:::

</div>

## 可用镜像标签

| 标签 | 架构 | 说明 |
|:----|:--------------|:------------|
| `:latest` | AMD64 + ARM64 | 最新稳定版（自动检测架构） |
| `:v0.5.x` | AMD64 + ARM64 | 特定版本（自动检测架构） |
| `:v0.5.x-amd64` | 仅 AMD64 | 显式 AMD64 镜像 |
| `:v0.5.x-arm64` | 仅 ARM64 | 显式 ARM64 镜像 |

::: tip 多架构支持
`:latest` 标签是**多架构镜像**——Docker 会自动为您的系统（Intel/AMD 或 Apple Silicon/ARM）拉取正确的版本。
:::

## 云端部署

部署到云服务器时，必须设置外部 API URL：

```bash
docker run -d --name deeptutor \
  -p 8001:8001 -p 3782:3782 \
  -e NEXT_PUBLIC_API_BASE_EXTERNAL=https://your-server.com:8001 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning 为什么需要这样做？
默认 API URL 是 `localhost:8001`，在浏览器中这指向用户的本地机器，而不是您的服务器。设置 `NEXT_PUBLIC_API_BASE_EXTERNAL` 确保前端连接到您服务器的公共地址。
:::

## 自定义端口

使用不同的端口：

```bash
docker run -d --name deeptutor \
  -p 9001:9001 -p 4000:4000 \
  -e BACKEND_PORT=9001 \
  -e FRONTEND_PORT=4000 \
  -e NEXT_PUBLIC_API_BASE_EXTERNAL=http://localhost:9001 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning 重要
`-p` 端口映射必须与 `BACKEND_PORT`/`FRONTEND_PORT` 值匹配。
:::

## 访问地址

| 服务 | URL | 说明 |
|:---:|:---|:---|
| **前端** | http://localhost:3782 | 主 Web 界面 |
| **API 文档** | http://localhost:8001/docs | 交互式 API 文档 |

## 容器管理

### 查看运行中的容器

```bash
docker ps
```

### 停止容器

```bash
docker stop deeptutor
```

### 删除容器

```bash
docker rm deeptutor
```

### 查看日志

```bash
# 所有日志
docker logs deeptutor

# 实时跟踪日志
docker logs -f deeptutor

# 最后 100 行
docker logs --tail 100 deeptutor
```

## 故障排除

### 云端部署时前端无法连接

**问题：** 前端显示 "Failed to fetch" 或 "NEXT_PUBLIC_API_BASE is not configured"

**解决方案：** 将 `NEXT_PUBLIC_API_BASE_EXTERNAL` 设置为您服务器的公共 URL：

```bash
-e NEXT_PUBLIC_API_BASE_EXTERNAL=https://your-server.com:8001
```

### 使用 HTTPS 反向代理时设置页面显示 "Error loading data"

**问题：** HTTPS 请求被重定向到 HTTP（307 重定向）

**解决方案：** 此问题已在 v0.5.0+ 中修复。请更新到最新版本。

**nginx 配置：**

```nginx
# 前端
location / {
    proxy_pass http://localhost:3782;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# 后端 API
location /api/ {
    proxy_pass http://localhost:8001;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# WebSocket 支持
location /api/v1/ {
    proxy_pass http://localhost:8001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### 重启后数据未保留

**解决方案：** 确保挂载了数据目录：

```bash
-v $(pwd)/data:/app/data
```

### HuggingFace 模型下载失败（受限网络）

**解决方案：** 在 `.env` 中添加 HuggingFace 配置：

```bash
# 使用镜像端点
HF_ENDPOINT=https://your-hf-mirror.example.com

# 或强制离线模式（需要预缓存的模型）
HF_HUB_OFFLINE=1
```

---

**下一步：** [参与贡献 →](/zh/contribution)

<style>
.method-tabs {
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-border);
  border-radius: 12px;
  padding: 20px;
  margin: 16px 0;
}

.method-tabs h3 {
  margin-top: 0;
  padding-top: 0;
  border-top: none;
}
</style>
