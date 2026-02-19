# Docker 部署说明 / Docker Deployment Guide

本文档说明如何使用 Docker 构建和部署 ColorLake 应用。

This document explains how to build and deploy the ColorLake application using Docker.

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

### 方式 2: 推送到镜像仓库 / Method 2: Push to Registry

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

## 相关文档 / Related Documentation

- [Docker 官方文档](https://docs.docker.com/)
- [Nginx 官方文档](https://nginx.org/en/docs/)
- [Vite 部署指南](https://vitejs.dev/guide/static-deploy.html)
