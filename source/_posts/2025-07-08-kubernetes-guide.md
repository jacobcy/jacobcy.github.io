---
title: Kubernetes入门：微服务部署实践
date: 2025-07-08 14:00:00
updated: 2025-07-08 14:00:00
description: 'Kubernetes入门完全指南，从基础概念到生产部署，包含Pod、Service、Deployment实战，以及Helm包管理和CI/CD集成'
keywords: 'Kubernetes,K8s,微服务,容器编排,Helm,DevOps,云原生,部署实践'
tags: 
  - Kubernetes
  - K8s
  - 微服务
  - DevOps
  - 云原生
categories:
  - 技术笔记
  - DevOps
author: Yi Chen
toc: true
comments: true
---

> ☸️ "Kubernetes是云时代的操作系统。"

## 🤔 为什么需要Kubernetes

在用Docker部署了几个应用后，我遇到了新问题：

- **服务发现**：容器IP经常变，怎么找到它们？
- **负载均衡**：怎么把流量分发到多个容器？
- **故障恢复**：容器挂了怎么自动重启？
- **滚动更新**：怎么不停机更新应用？
- **资源管理**：怎么限制容器使用的CPU/内存？

**Docker解决了"打包"问题，Kubernetes解决了"编排"问题**。

## 🎯 Kubernetes核心概念

### 架构概览

```
┌─────────────────────────────────────────┐
│           Master节点（控制平面）          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ API Server│ │Scheduler│ │Controller│   │
│  └─────────┘ └─────────┘ └─────────┘   │
│              ┌─────────┐                │
│              │   etcd   │                │
│              └─────────┘                │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│           Worker节点（工作负载）          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  Pod    │ │  Pod    │ │  Pod    │   │
│  │ [App]   │ │ [App]   │ │ [App]   │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│              ┌─────────┐                │
│              │ kubelet │                │
│              └─────────┘                │
└─────────────────────────────────────────┘
```

### 核心资源对象

| 资源 | 作用 | 类比 |
|------|------|------|
| **Pod** | 最小部署单元，包含1+容器 | 豌豆荚 |
| **Deployment** | 管理Pod的部署和更新 | 运维经理 |
| **Service** | 服务发现和负载均衡 | 前台接待 |
| **ConfigMap** | 存储配置数据 | 配置文件 |
| **Secret** | 存储敏感数据 | 保险柜 |
| **Ingress** | HTTP路由 | 网关 |
| **PV/PVC** | 持久化存储 | 硬盘 |

## 🚀 快速入门

### 安装Kubernetes

**本地开发：Minikube**

```bash
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 启动集群
minikube start --driver=docker

# 验证
kubectl get nodes
```

**生产环境：云厂商托管**

- AWS EKS
- Google GKE
- Azure AKS
- 阿里云 ACK

### kubectl 常用命令

```bash
# 查看资源
kubectl get nodes              # 查看节点
kubectl get pods               # 查看Pod
kubectl get pods -o wide       # 详细信息
kubectl get services           # 查看服务
kubectl get all                # 查看所有资源

# 操作资源
kubectl apply -f filename.yaml # 创建/更新资源
kubectl delete -f filename.yaml # 删除资源
kubectl delete pod pod-name    # 删除Pod

# 查看日志
kubectl logs pod-name          # 查看日志
kubectl logs -f pod-name       # 实时日志
kubectl logs --previous pod-name # 上次崩溃日志

# 进入容器
kubectl exec -it pod-name -- /bin/bash

# 端口转发（本地调试）
kubectl port-forward pod-name 8080:80
```

## 💡 实战：部署微服务应用

### 场景

部署一个电商微服务系统：
- **frontend**: React前端
- **api**: Node.js后端API
- **worker**: 后台任务处理
- **redis**: 缓存
- **postgres**: 数据库

### 1. Namespace（命名空间）

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
```

创建：
```bash
kubectl apply -f namespace.yaml
kubectl config set-context --current --namespace=ecommerce
```

### 2. ConfigMap & Secret

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  NODE_ENV: "production"
  API_PORT: "3000"
  REDIS_HOST: "redis"
  DB_HOST: "postgres"
  DB_NAME: "ecommerce"
```

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # echo -n 'your-password' | base64
  DB_PASSWORD: eW91ci1wYXNzd29yZA==
  REDIS_PASSWORD: cmVkaXMtcGFzcw==
  JWT_SECRET: c29tZS1zZWNyZXQta2V5
```

### 3. Deployment

```yaml
# api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  labels:
    app: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: your-registry/api:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: app-config
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 4. Service

```yaml
# api-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: ClusterIP
```

### 5. Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.yourdomain.com
    - www.yourdomain.com
    secretName: ecommerce-tls
  rules:
  - host: api.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 80
  - host: www.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

### 6. 持久化存储

```yaml
# postgres-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

```yaml
# postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        env:
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_NAME
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

## 🎨 Helm：包管理器

### 为什么用Helm

手动管理几十个YAML文件很痛苦。Helm提供：
- 模板化配置
- 版本管理
- 依赖管理

### 安装Helm

```bash
brew install helm
```

### 创建Chart

```bash
helm create mychart
cd mychart
```

目录结构：
```
mychart/
├── Chart.yaml          # Chart元数据
├── values.yaml         # 默认配置值
├── charts/             # 依赖的Charts
└── templates/          # K8s资源模板
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── _helpers.tpl    # 辅助模板
```

### values.yaml示例

```yaml
# values.yaml
replicaCount: 3

image:
  repository: your-registry/api
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  hosts:
    - host: api.yourdomain.com
      paths:
        - /
  tls:
    - secretName: api-tls
      hosts:
        - api.yourdomain.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

### 部署Helm Chart

```bash
# 安装
helm install myapp ./mychart

# 升级
helm upgrade myapp ./mychart

# 查看
helm list

# 回滚
helm rollback myapp 1
```

## 📊 监控与日志

### Prometheus + Grafana

```yaml
# prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
```

### ELK Stack日志

```yaml
# fluentd-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 🚀 CI/CD集成

### GitLab CI示例

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_REGISTRY: registry.gitlab.com
  KUBECONFIG: /etc/deploy/config

build:
  stage: build
  script:
    - docker build -t $DOCKER_REGISTRY/$CI_PROJECT_PATH:$CI_COMMIT_SHA .
    - docker push $DOCKER_REGISTRY/$CI_PROJECT_PATH:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production
    - helm upgrade --install myapp ./helm-chart
      --set image.tag=$CI_COMMIT_SHA
  only:
    - main
```

### ArgoCD：GitOps部署

```yaml
# application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/myapp.git
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## ⚠️ 生产环境最佳实践

### 1. 资源限制

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 2. 健康检查

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 3. 安全策略

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

### 4. HPA自动扩缩容

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## 📚 学习资源

### 官方文档
- [Kubernetes官方文档](https://kubernetes.io/docs/)
- [Helm官方文档](https://helm.sh/docs/)

### 书籍
- 《Kubernetes in Action》
- 《Cloud Native DevOps with Kubernetes》

### 在线课程
- KodeKloud Kubernetes课程
- Udemy Kubernetes课程

### 实践平台
- Katacoda（已被关闭，找替代）
- Play with Kubernetes
- Killer Coda

---

## 💭 写在最后

学习Kubernetes是一个循序渐进的过程。

建议的学习路径：
1. **Week 1-2**：理解核心概念，本地Minikube练习
2. **Week 3-4**：部署简单应用，熟悉kubectl
3. **Month 2**：学习Helm，理解模板
4. **Month 3+**：生产环境实践，监控日志

> "Kubernetes的学习曲线很陡，但一旦掌握，你就拥有了云原生世界的能力。"

如果你也在学习K8s，欢迎交流！☸️

---

*写于 2025年7月8日 | K8s学习第3个月*
