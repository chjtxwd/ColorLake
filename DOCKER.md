# Docker 部署说明 / Docker Deployment Guide

本文档说明如何使用 Docker 构建和部署 ColorLake 应用。

This document explains how to build and deploy the ColorLake application using Docker.

## GitHub Actions 自动构建 / Automated Build with GitHub Actions

本项目已配置 GitHub Actions 工作流，可自动构建多架构（ARM64 和 AMD64）Docker 镜像并推送到 GitHub Container Registry。

This project is configured with GitHub Actions workflow to automatically build multi-architecture (ARM64 and AMD64) Docker images and push them to GitHub Container Registry.

### 触发条件 / Trigger Conditions

工作流在以下情况下自动触发 / The workflow is automatically triggered when:

- 推送到 `main` 或 `master` 分支 / Push to `main` or `master` branch
- 创建版本标签（如 `v1.0.0`）/ Create a version tag (e.g., `v1.0.0`)
- 创建 Pull Request / Create a Pull Request (仅构建，不推送 / build only, no push)
- 手动触发工作流 / Manually trigger the workflow

### 使用 GitHub 镜像 / Using GitHub Container Registry Images

```bash
# 拉取最新镜像 / Pull the latest image
docker pull ghcr.io/chjtxwd/colorlake:latest

# 运行容器 / Run container
docker run -d -p 8080:80 --name colorlake ghcr.io/chjtxwd/colorlake:latest

# 拉取特定版本 / Pull a specific version
docker pull ghcr.io/chjtxwd/colorlake:v1.0.0
```

### 多架构支持 / Multi-Architecture Support

镜像支持以下架构 / The images support the following architectures:
- `linux/amd64` (x86_64)
- `linux/arm64` (ARM64/aarch64)

Docker 会自动选择与您的系统架构匹配的镜像。

Docker will automatically select the image that matches your system architecture.

## 构建 Docker 镜像 / Build Docker Image

```bash
# 构建镜像 / Build the image
docker build -t colorlake:latest .

# 或者指定自定义标签 / Or with a custom tag
docker build -t colorlake:v1.0.0 .
```

## 运行容器 / Run Container

```bash
# 运行容器，映射到本地 8080 端口 / Run container, map to local port 8080
docker run -d -p 8080:80 --name colorlake colorlake:latest

# 访问应用 / Access the application
# 打开浏览器访问 / Open browser at: http://localhost:8080
```

## Docker 镜像说明 / Docker Image Details

### 多阶段构建 / Multi-stage Build

Dockerfile 使用多阶段构建以优化镜像大小：

The Dockerfile uses multi-stage build to optimize image size:

1. **构建阶段 (Build Stage)**:
   - 基础镜像：`node:18-alpine`
   - 安装依赖并构建应用
   - 输出静态文件到 `dist/` 目录

2. **运行阶段 (Runtime Stage)**:
   - 基础镜像：`nginx:alpine`
   - 仅复制构建产物
   - 使用 Nginx 提供静态文件服务

### 镜像大小优化 / Image Size Optimization

- 使用 Alpine Linux 基础镜像（体积小）
- 多阶段构建，最终镜像不包含 Node.js 和构建依赖
- 通过 `.dockerignore` 排除不必要的文件

## 常用 Docker 命令 / Common Docker Commands

```bash
# 查看运行中的容器 / List running containers
docker ps

# 查看所有容器 / List all containers
docker ps -a

# 停止容器 / Stop container
docker stop colorlake

# 启动容器 / Start container
docker start colorlake

# 删除容器 / Remove container
docker rm colorlake

# 查看容器日志 / View container logs
docker logs colorlake

# 进入容器 / Enter container
docker exec -it colorlake sh

# 删除镜像 / Remove image
docker rmi colorlake:latest
```

## 使用 Docker Compose (可选) / Using Docker Compose (Optional)

如果需要，可以创建 `docker-compose.yml` 文件：

You can create a `docker-compose.yml` file if needed:

```yaml
version: '3.8'

services:
  colorlake:
    build: .
    ports:
      - "8080:80"
    restart: unless-stopped
```

然后使用以下命令：

Then use these commands:

```bash
# 构建并启动 / Build and start
docker-compose up -d

# 停止 / Stop
docker-compose down

# 查看日志 / View logs
docker-compose logs -f
```

## 部署到生产环境 / Deploy to Production

### 方式 1: 直接使用 Docker / Method 1: Direct Docker

```bash
# 构建生产镜像 / Build production image
docker build -t colorlake:production .

# 运行生产容器 / Run production container
docker run -d \
  -p 80:80 \
  --name colorlake-prod \
  --restart unless-stopped \
  colorlake:production
```

### 方式 2: 使用 GitHub Container Registry / Method 2: Use GitHub Container Registry

```bash
# 拉取镜像 / Pull the image
docker pull ghcr.io/chjtxwd/colorlake:latest

# 运行生产容器 / Run production container
docker run -d \
  -p 80:80 \
  --name colorlake-prod \
  --restart unless-stopped \
  ghcr.io/chjtxwd/colorlake:latest
```

### 方式 3: 推送到自定义镜像仓库 / Method 3: Push to Custom Registry

```bash
# 标记镜像 / Tag the image
docker tag colorlake:latest your-registry.com/colorlake:latest

# 推送到仓库 / Push to registry
docker push your-registry.com/colorlake:latest

# 在生产服务器上拉取并运行 / Pull and run on production server
docker pull your-registry.com/colorlake:latest
docker run -d -p 80:80 --restart unless-stopped your-registry.com/colorlake:latest
```

## 故障排查 / Troubleshooting

### 构建失败 / Build Fails

```bash
# 查看详细构建日志 / View detailed build logs
docker build --progress=plain -t colorlake:latest .

# 清理构建缓存 / Clean build cache
docker builder prune
```

### 容器无法启动 / Container Won't Start

```bash
# 查看容器日志 / Check container logs
docker logs colorlake

# 检查容器状态 / Check container status
docker inspect colorlake
```

### 端口冲突 / Port Conflict

如果 8080 端口被占用，可以使用其他端口：

If port 8080 is in use, you can use another port:

```bash
docker run -d -p 3000:80 --name colorlake colorlake:latest
```

## 性能优化建议 / Performance Optimization Tips

1. 使用镜像缓存加速构建
2. 在生产环境中启用 Nginx gzip 压缩
3. 配置适当的 Nginx 缓存策略
4. 考虑使用 CDN 加速静态资源访问

## GitHub Actions 工作流详情 / GitHub Actions Workflow Details

### 工作流文件 / Workflow File

工作流配置文件位于 `.github/workflows/docker-build-push.yml`

The workflow configuration is located at `.github/workflows/docker-build-push.yml`

### 功能特性 / Features

- ✅ 支持多架构构建（AMD64 和 ARM64）/ Multi-architecture build support (AMD64 and ARM64)
- ✅ 使用 QEMU 进行跨平台构建 / Cross-platform build using QEMU
- ✅ 推送到 GitHub Container Registry (ghcr.io)
- ✅ 自动生成镜像标签和元数据 / Automatic image tags and metadata generation
- ✅ GitHub Actions 缓存优化构建速度 / Build speed optimization with GitHub Actions cache
- ✅ Pull Request 仅构建，不推送 / Pull Request builds without pushing

### 镜像标签策略 / Image Tagging Strategy

工作流会根据触发事件自动生成标签 / The workflow automatically generates tags based on trigger events:

- 推送到默认分支（main/master）/ Push to default branch: `latest`
- 推送到其他分支 / Push to other branches: `<branch-name>`
- 版本标签 / Version tags: `v1.2.3`, `1.2`, `1`
- Pull Request: `pr-<number>`

### 手动触发工作流 / Manually Trigger Workflow

1. 访问 GitHub 仓库的 Actions 标签页 / Go to the Actions tab in your GitHub repository
2. 选择 "Build and Push Docker Image" 工作流 / Select the "Build and Push Docker Image" workflow
3. 点击 "Run workflow" 按钮 / Click the "Run workflow" button
4. 选择分支并启动 / Select the branch and start

### 访问镜像 / Accessing Images

构建完成后，镜像会被推送到 GitHub Container Registry：

After the build completes, images are pushed to GitHub Container Registry:

```
ghcr.io/chjtxwd/colorlake:latest
ghcr.io/chjtxwd/colorlake:main
ghcr.io/chjtxwd/colorlake:v1.0.0  (如果使用版本标签 / if using version tags)
```

### 设置镜像可见性 / Setting Image Visibility

默认情况下，推送到 ghcr.io 的镜像可能是私有的。要将其设为公开：

By default, images pushed to ghcr.io may be private. To make them public:

1. 访问包管理页面 / Go to: https://github.com/chjtxwd?tab=packages
2. 找到 colorlake 包 / Find the colorlake package
3. 在包设置中将可见性更改为 Public / Change visibility to Public in package settings

## 相关文档 / Related Documentation

- [Docker 官方文档](https://docs.docker.com/)
- [Nginx 官方文档](https://nginx.org/en/docs/)
- [Vite 部署指南](https://vitejs.dev/guide/static-deploy.html)
- [GitHub Container Registry 文档](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
