+++
title = 'Go 語言系列（十八）：Kubernetes 部署'
date = 2026-02-25T18:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'Kubernetes', 'K8s', 'Docker', 'Helm', '部署', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習如何在 Kubernetes 上部署 Go 應用：Dockerfile 最佳化、K8s 資源定義、健康檢查、滾動更新和 Helm Chart 基礎'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十八篇，也是本系列的最終篇。我們將把前面建構的 Go 應用部署到 Kubernetes 上，學習容器編排的核心概念。

## Kubernetes 基礎概念

### 核心元件

```
┌─────────────────────────────────────────┐
│              Kubernetes Cluster          │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │           Namespace: default       │  │
│  │                                    │  │
│  │  ┌──────────┐   ┌──────────┐     │  │
│  │  │Deployment│   │ Service  │     │  │
│  │  │          │   │(ClusterIP│     │  │
│  │  │ ┌──────┐ │   │ or LB)  │     │  │
│  │  │ │ Pod  │ │←──│          │     │  │
│  │  │ │┌────┐│ │   └──────────┘     │  │
│  │  │ ││容器││ │                     │  │
│  │  │ │└────┘│ │                     │  │
│  │  │ └──────┘ │                     │  │
│  │  └──────────┘                     │  │
│  └────────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

| 元件 | 說明 |
|------|------|
| **Pod** | 最小部署單位，一個或多個容器的組合 |
| **Deployment** | 管理 Pod 的副本數、更新策略 |
| **Service** | 為 Pod 提供穩定的網路端點 |
| **Namespace** | 資源的邏輯隔離 |
| **ConfigMap** | 配置資料（非機密） |
| **Secret** | 機密資料（密碼、token） |

## 本地環境設定

### 使用 minikube

```bash
# 安裝
brew install minikube

# 啟動叢集
minikube start

# 確認狀態
kubectl cluster-info
kubectl get nodes
```

### 使用 kind（Kubernetes in Docker）

```bash
# 安裝
brew install kind

# 建立叢集
kind create cluster --name bookstore

# 確認
kubectl cluster-info --context kind-bookstore
```

`kind` 更輕量，適合 CI/CD 和快速測試。

## Go 應用的 Dockerfile 最佳化

### Multi-stage Build

```dockerfile
# ===== Build Stage =====
FROM golang:1.22-alpine AS builder

WORKDIR /app

# 先複製 go.mod 和 go.sum（利用 Docker 快取層）
COPY go.mod go.sum ./
RUN go mod download

# 複製原始碼並編譯
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/server ./cmd/server

# ===== Production Stage =====
FROM scratch

# 從 builder 階段複製執行檔
COPY --from=builder /app/server /server

# 如果需要 TLS 憑證（呼叫外部 HTTPS API）
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

EXPOSE 8080
ENTRYPOINT ["/server"]
```

### 映像大小對比

| Base Image | 大小 |
|-----------|------|
| `golang:1.22` | ~800MB |
| `golang:1.22-alpine` | ~250MB |
| `alpine:3.19` | ~5MB + 你的 binary |
| `scratch` | 0MB + 你的 binary |
| `gcr.io/distroless/static` | ~2MB + 你的 binary |

**建議：**
- `scratch`：最小，但沒有 shell，debug 困難
- `distroless/static`：比 scratch 多一些基本檔案，推薦用於生產
- `alpine`：需要 debug 工具時使用

### 建構與推送

```bash
# 建構映像
docker build -t bookstore-api:v1.0.0 .

# 如果用 minikube，先載入映像
minikube image load bookstore-api:v1.0.0

# 如果用 kind
kind load docker-image bookstore-api:v1.0.0 --name bookstore
```

## Kubernetes 資源定義

### Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstore-api
  labels:
    app: bookstore-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bookstore-api
  template:
    metadata:
      labels:
        app: bookstore-api
    spec:
      containers:
        - name: api
          image: bookstore-api:v1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: bookstore-secrets
                  key: database-url
            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: bookstore-config
                  key: log-level
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 500m
              memory: 128Mi
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
```

### Service

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: bookstore-api
spec:
  type: ClusterIP  # 叢集內部存取
  selector:
    app: bookstore-api
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
---
# 如果需要對外暴露
apiVersion: v1
kind: Service
metadata:
  name: bookstore-api-lb
spec:
  type: LoadBalancer
  selector:
    app: bookstore-api
  ports:
    - port: 80
      targetPort: 8080
```

### ConfigMap

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: bookstore-config
data:
  log-level: "info"
  server-port: "8080"
  cors-origin: "https://bookstore.example.com"
```

### Secret

```bash
# 用 kubectl 建立 Secret（不要把 Secret 明文存在 YAML 中）
kubectl create secret generic bookstore-secrets \
  --from-literal=database-url='postgres://app:secret@postgres:5432/bookstore' \
  --from-literal=jwt-secret='your-jwt-secret-key'
```

### 部署

```bash
# 部署所有資源
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# 查看狀態
kubectl get pods
kubectl get services
kubectl logs -f deployment/bookstore-api
```

## 健康檢查與滾動更新

### Go 中實作 Health Endpoint

```go
package main

import (
    "encoding/json"
    "net/http"
    "sync/atomic"
)

var ready atomic.Bool

func main() {
    mux := http.NewServeMux()

    // Liveness：應用是否還活著
    mux.HandleFunc("/health/live", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]string{"status": "alive"})
    })

    // Readiness：應用是否準備好接收流量
    mux.HandleFunc("/health/ready", func(w http.ResponseWriter, r *http.Request) {
        if !ready.Load() {
            w.WriteHeader(http.StatusServiceUnavailable)
            json.NewEncoder(w).Encode(map[string]string{"status": "not ready"})
            return
        }
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]string{"status": "ready"})
    })

    // 應用啟動完成後設定 ready
    go func() {
        // 初始化資料庫連線、快取等...
        initDatabase()
        ready.Store(true)
    }()

    mux.HandleFunc("/books", handleBooks)

    http.ListenAndServe(":8080", mux)
}
```

**Liveness vs Readiness：**
- **Liveness Probe**：失敗時，K8s 會重啟 Pod
- **Readiness Probe**：失敗時，K8s 會從 Service 移除該 Pod（不接收流量）

### 滾動更新

```yaml
# 在 Deployment spec 中設定更新策略
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # 更新時最多有 1 個 Pod 不可用
      maxSurge: 1          # 更新時最多多出 1 個 Pod
```

```bash
# 觸發更新（修改映像版本）
kubectl set image deployment/bookstore-api api=bookstore-api:v1.1.0

# 查看更新進度
kubectl rollout status deployment/bookstore-api

# 回滾到上一版
kubectl rollout undo deployment/bookstore-api

# 查看更新歷史
kubectl rollout history deployment/bookstore-api
```

## Helm 基礎

Helm 是 Kubernetes 的套件管理工具，用 Chart 打包 K8s 資源。

### 建立 Chart

```bash
helm create bookstore
```

產生的結構：

```
bookstore/
├── Chart.yaml          # Chart 的中繼資料
├── values.yaml         # 預設配置值
├── templates/
│   ├── deployment.yaml # Deployment 模板
│   ├── service.yaml    # Service 模板
│   ├── configmap.yaml
│   ├── _helpers.tpl    # 模板函式
│   └── NOTES.txt       # 安裝後的提示訊息
└── charts/             # 依賴的其他 Chart
```

### Chart.yaml

```yaml
apiVersion: v2
name: bookstore
description: Bookstore API service
type: application
version: 0.1.0        # Chart 版本
appVersion: "1.0.0"   # 應用版本
```

### values.yaml

```yaml
replicaCount: 3

image:
  repository: bookstore-api
  tag: "v1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 500m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 64Mi

config:
  logLevel: info
  serverPort: "8080"
```

### 模板中使用 values

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "bookstore.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "bookstore.name" . }}
  template:
    spec:
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 8080
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

### 使用 Helm

```bash
# 安裝
helm install bookstore ./bookstore

# 用自訂值安裝
helm install bookstore ./bookstore --set replicaCount=5

# 用自訂 values 檔案
helm install bookstore ./bookstore -f production-values.yaml

# 更新
helm upgrade bookstore ./bookstore --set image.tag=v1.1.0

# 回滾
helm rollback bookstore 1

# 移除
helm uninstall bookstore
```

## 系列回顧

恭喜你完成了 **Go 語言從零到 Web 應用**完整系列！18 篇文章走過了一條完整的學習路徑：

### 基礎篇（一至五）

| 篇 | 主題 | 核心收穫 |
|----|------|----------|
| 1 | 為什麼學 Go | 設計哲學、適用場景 |
| 2 | 環境設定 | 安裝、VS Code、Go Modules |
| 3 | 基礎語法 | 變數、型別、流程控制、函式 |
| 4 | 資料結構 | Slice、Map、Struct、Interface |
| 5 | 錯誤處理與併發 | error、Goroutine、Channel |

### 實戰篇（六至十）

| 篇 | 主題 | 核心收穫 |
|----|------|----------|
| 6 | HTTP 基礎 | net/http、Handler、JSON |
| 7 | 專案結構 | 目錄結構、中間件、路由 |
| 8 | 資料庫 | database/sql、SQLite、CRUD |
| 9 | 認證 | bcrypt、JWT、保護端點 |
| 10 | 測試與部署 | testing、httptest、Docker |

### 面試與陷阱篇（十一至十二）

| 篇 | 主題 | 核心收穫 |
|----|------|----------|
| 11 | 面試必問問題 | GMP、Interface、Slice 底層 |
| 12 | 常見陷阱 | nil interface、closure 捕獲、map 併發 |

### 進階篇（十三至十八）

| 篇 | 主題 | 核心收穫 |
|----|------|----------|
| 13 | Web 框架 | Gin vs Echo 對比 |
| 14 | ORM 工具 | GORM vs sqlx 對比 |
| 15 | PostgreSQL | pgx、Migration、JSONB |
| 16 | gRPC | protobuf、Streaming、攔截器 |
| 17 | 微服務 | 拆分策略、通訊模式、設計模式 |
| 18 | Kubernetes | Docker 最佳化、K8s 部署、Helm |

從一行 `fmt.Println("Hello, World!")` 到在 Kubernetes 上部署微服務，你已經掌握了 Go 開發的完整技能樹。

**接下來的建議：**
- 動手做一個真實的 side project
- 閱讀優秀的開源 Go 專案程式碼
- 參與 Go 社群，關注 Go Blog 和 Release Notes
- 持續練習，保持寫程式的手感

Happy Coding with Go!
