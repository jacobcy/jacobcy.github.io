---
title: Docker容器化实战：从开发到部署
date: 2025-04-08 13:00:00
updated: 2025-04-08 13:00:00
description: 'Docker容器化完整实战指南，从基础概念到生产部署，包含Dockerfile编写、多阶段构建、Compose编排和CI/CD集成'
keywords: 'Docker,容器化,DevOps,Dockerfile,CI/CD,微服务,Docker Compose,生产部署'
tags: 
  - Docker
  - DevOps
  - 容器化
  - CI/CD
categories:
  - 技术笔记
  - DevOps
author: Yi Chen
toc: true
comments: true
---

> 🐳 "Build once, run anywhere."

## 🤔 为什么需要Docker

还记得2014年那个著名的梗吗？

> "在我的机器上能跑！"

环境不一致导致的问题：
- 开发环境OK，测试环境报错
- Python版本不同导致依赖冲突
- 系统库缺失导致编译失败
- 新同事环境配置花了一整天

**Docker解决了这一切**。

## 🎯 Docker核心概念

### 镜像 vs 容器

| 概念 | 类比 | 说明 |
|------|------|------|
| **镜像（Image）** | 类（Class） | 只读模板，包含应用和依赖 |
| **容器（Container）** | 对象（Object） | 镜像的运行实例 |
| **仓库（Registry）** | GitHub | 存储和分发镜像的地方 |
| **Dockerfile** | 源代码 | 定义镜像的构建步骤 |

### 关键优势

```
┌─────────────────────────────────────────┐
│  传统部署                                  │
│  App A → 需要 Python 3.9                  │
│  App B → 需要 Python 3.11 ← 冲突！         │
│  App C → 需要 Node 18                     │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  Docker部署                               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐     │
│  │Container│ │Container│ │Container│     │
│  │  App A  │ │  App B  │ │  App C  │     │
│  │Python3.9│ │Python3.11│ │ Node 18 │     │
│  └─────────┘ └─────────┘ └─────────┘     │
│         互不干扰，独立运行                   │
└─────────────────────────────────────────┘
```

## 🚀 快速入门

### 安装Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# macOS
brew install --cask docker

# 验证安装
docker --version
```

### 第一个容器

```bash
# 运行Hello World
docker run hello-world

# 运行Nginx
docker run -d -p 8080:80 nginx

# 访问 http://localhost:8080
```

### 常用命令速查

```bash
# 镜像管理
docker pull ubuntu:22.04          # 拉取镜像
docker images                      # 查看本地镜像
docker rmi image_id               # 删除镜像

# 容器管理
docker ps                         # 运行中的容器
docker ps -a                      # 所有容器
docker run -d --name myapp nginx  # 后台运行
docker stop myapp                 # 停止容器
docker rm myapp                   # 删除容器
docker exec -it myapp bash        # 进入容器

# 其他
docker logs myapp                 # 查看日志
docker stats                      # 资源使用
docker system prune               # 清理无用资源
```

## 💡 实战：容器化Python Web应用

### 场景

一个Flask应用，需要：
- Python 3.11
- Redis缓存
- PostgreSQL数据库

### 项目结构

```
myapp/
├── app/
│   ├── __init__.py
│   ├── routes.py
│   └── models.py
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

### 第一步：编写Dockerfile

```dockerfile
# 使用官方Python基础镜像
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装Python依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 5000

# 健康检查
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# 启动命令
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

### 第二步：多阶段构建优化

生产环境使用多阶段构建，减小镜像体积：

```dockerfile
# 构建阶段
FROM python:3.11-slim as builder

WORKDIR /app

# 安装构建依赖
RUN apt-get update && apt-get install -y gcc

# 安装Python依赖到虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 生产阶段
FROM python:3.11-slim

WORKDIR /app

# 从构建阶段复制虚拟环境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 只复制必要的代码
COPY app/ ./app/

# 使用非root用户运行
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

**优化效果**：
- 原始镜像：1.2GB
- 多阶段构建后：180MB
- 构建时间：减少40%

### 第三步：Docker Compose编排

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379/0
      - FLASK_ENV=production
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    restart: unless-stopped
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - web
    restart: unless-stopped
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### 第四步：运行

```bash
# 构建并启动所有服务
docker-compose up -d --build

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f web

# 扩展web服务到3个实例
docker-compose up -d --scale web=3

# 停止并清理
docker-compose down -v
```

## 🛠️ 生产环境最佳实践

### 1. 镜像安全

```dockerfile
# 使用官方镜像，指定版本
FROM python:3.11.6-slim-bookworm

# 不要以root运行
RUN useradd -m -u 1000 appuser
USER appuser

# 扫描镜像漏洞
docker scan myapp:latest
```

### 2. 资源限制

```yaml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### 3. 日志管理

```yaml
services:
  web:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 4. 健康检查

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:5000/health || exit 1
```

## 🚀 CI/CD集成

### GitHub Actions工作流

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d
            docker system prune -f
```

## 📊 性能优化技巧

### 1. 镜像分层优化

```dockerfile
# 好的做法：把不常变的层放在前面
FROM python:3.11-slim
WORKDIR /app

# 依赖变化频率低，放前面
COPY requirements.txt .
RUN pip install -r requirements.txt

# 代码变化频率高，放后面
COPY . .
```

### 2. 使用.dockerignore

```
.git
.gitignore
README.md
Dockerfile
.dockerignore
node_modules
__pycache__
*.pyc
.env
.vscode
.idea
```

### 3. 多架构构建

```bash
docker buildx create --use
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push .
```

## ⚠️ 常见陷阱

### 陷阱1：镜像体积过大

**问题**：安装不必要的依赖

**解决**：使用多阶段构建，只保留运行必需的依赖

### 陷阱2：数据丢失

**问题**：容器重启后数据消失

**解决**：使用Volumes持久化数据

### 陷阱3：时区问题

**问题**：容器内时区不对

**解决**：
```dockerfile
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

### 陷阱4：权限问题

**问题**：容器内文件权限不对

**解决**：确保宿主机和容器用户UID一致

## 🎯 下一步学习

### 进阶主题

- [ ] Kubernetes编排
- [ ] Docker Swarm集群
- [ ] 服务网格（Istio）
- [ ] 容器安全扫描
- [ ] 监控和可观测性（Prometheus + Grafana）

### 推荐资源

- [Docker官方文档](https://docs.docker.com/)
- [Dockerfile最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Kubernetes官方教程](https://kubernetes.io/docs/tutorials/)

---

## 💭 写在最后

Docker彻底改变了我的开发和部署方式。

以前部署一个新应用，需要：
1. 准备服务器环境
2. 安装各种依赖
3. 配置系统参数
4. 祈祷不出错

现在只需要：
```bash
docker-compose up -d
```

**容器化不是银弹，但它解决了很多实际问题**。

如果你还没开始使用Docker，我强烈建议现在就开始。它会让你的工作流顺畅很多。

有任何问题，欢迎在评论区交流！🐳

---

*写于 2025年4月8日 | 容器化实践总结*
