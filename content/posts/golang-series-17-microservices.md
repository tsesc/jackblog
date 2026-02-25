+++
title = 'Go 語言系列（十七）：微服務架構'
date = 2026-02-24T17:00:00+08:00
draft = false
tags = ['Golang', 'Go', '微服務', 'Microservices', 'API Gateway', 'Circuit Breaker', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習微服務架構的核心概念：服務拆分策略、HTTP/gRPC 通訊、API Gateway、Service Discovery、Circuit Breaker 和分散式追蹤'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十七篇。前面我們學了 REST API 和 gRPC 這兩種通訊方式，現在來學如何用微服務架構組織大型系統。

## 單體 vs 微服務

### 單體架構（Monolith）

所有功能在一個程式中：

```
┌─────────────────────────┐
│        單體應用          │
│  ┌─────┐ ┌─────┐ ┌───┐ │
│  │用戶 │ │訂單 │ │商品│ │
│  └──┬──┘ └──┬──┘ └─┬─┘ │
│     └───────┼──────┘   │
│          ┌──┴──┐       │
│          │ DB  │       │
│          └─────┘       │
└─────────────────────────┘
```

### 微服務架構

每個功能是獨立的服務：

```
┌──────┐  ┌──────┐  ┌──────┐
│用戶  │  │訂單  │  │商品  │
│服務  │  │服務  │  │服務  │
└──┬───┘  └──┬───┘  └──┬───┘
   │         │         │
┌──┴──┐  ┌──┴──┐  ┌──┴──┐
│Users│  │Orders│ │Products│
│ DB  │  │ DB  │  │  DB  │
└─────┘  └─────┘  └──────┘
```

### 對比

| 面向 | 單體 | 微服務 |
|------|------|--------|
| 開發速度（初期） | 快 | 慢（基礎設施多） |
| 部署 | 整包部署 | 獨立部署 |
| 擴展 | 整體擴展 | 按需擴展個別服務 |
| 技術選型 | 統一 | 每個服務可用不同技術 |
| 團隊組織 | 單一團隊 | 各服務獨立團隊 |
| 複雜度 | 程式碼內部 | 分散式系統 |
| 故障影響 | 全域 | 局部（做好隔離時） |

### 什麼時候該拆微服務？

**不要一開始就用微服務。** 先用單體，當出現以下訊號時再考慮拆分：

- 團隊超過 10 人，互相踩腳
- 部署一個功能要整包重新部署
- 某個模組的流量遠超其他模組
- 不同模組需要不同的擴展策略

## 服務拆分策略

### DDD Bounded Context

用 Domain-Driven Design 的 Bounded Context 來劃定服務邊界：

以電商系統為例：

```
┌─────────────────────────────────────────┐
│              電商系統                     │
│                                          │
│  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ 用戶上下文│  │ 訂單上下文│  │商品上下文│ │
│  │          │  │          │  │        │ │
│  │ - 註冊   │  │ - 建立訂單│  │- 商品  │ │
│  │ - 登入   │  │ - 付款   │  │- 庫存  │ │
│  │ - 個人資料│  │ - 出貨   │  │- 分類  │ │
│  └──────────┘  └──────────┘  └────────┘ │
└─────────────────────────────────────────┘
```

**拆分原則：**
- 每個 Bounded Context 變成一個微服務
- 服務之間透過 API 通訊，不共享資料庫
- 每個服務有自己的資料存儲

### Go 的專案結構

```
bookstore/
├── user-service/
│   ├── cmd/server/main.go
│   ├── internal/
│   │   ├── handler/
│   │   ├── service/
│   │   └── repository/
│   ├── proto/
│   └── go.mod
├── order-service/
│   ├── cmd/server/main.go
│   ├── internal/
│   └── go.mod
└── product-service/
    ├── cmd/server/main.go
    ├── internal/
    └── go.mod
```

## 服務間通訊

### 同步通訊：HTTP

```go
// order-service 呼叫 user-service
func (s *OrderService) CreateOrder(ctx context.Context, userID int64, items []Item) (*Order, error) {
    // 驗證使用者是否存在
    req, err := http.NewRequestWithContext(ctx, "GET",
        fmt.Sprintf("http://user-service:8080/users/%d", userID), nil)
    if err != nil {
        return nil, err
    }

    resp, err := s.httpClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("calling user-service: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode == http.StatusNotFound {
        return nil, ErrUserNotFound
    }

    // 建立訂單...
    return order, nil
}
```

### 同步通訊：gRPC

```go
// order-service 用 gRPC 呼叫 user-service
func (s *OrderService) CreateOrder(ctx context.Context, userID int64, items []Item) (*Order, error) {
    // gRPC client
    user, err := s.userClient.GetUser(ctx, &userpb.GetUserRequest{Id: userID})
    if err != nil {
        st, _ := status.FromError(err)
        if st.Code() == codes.NotFound {
            return nil, ErrUserNotFound
        }
        return nil, fmt.Errorf("calling user-service: %w", err)
    }

    // 建立訂單...
    return order, nil
}
```

### 非同步通訊：Message Queue

適用於不需要即時回應的場景：

```go
// order-service 發布事件
type OrderCreatedEvent struct {
    OrderID   int64     `json:"order_id"`
    UserID    int64     `json:"user_id"`
    Total     float64   `json:"total"`
    CreatedAt time.Time `json:"created_at"`
}

// 發布到 NATS
func (s *OrderService) publishOrderCreated(order *Order) error {
    event := OrderCreatedEvent{
        OrderID:   order.ID,
        UserID:    order.UserID,
        Total:     order.Total,
        CreatedAt: time.Now(),
    }
    data, _ := json.Marshal(event)
    return s.natsConn.Publish("orders.created", data)
}
```

```go
// notification-service 訂閱事件
func (s *NotificationService) subscribeOrderEvents() {
    s.natsConn.Subscribe("orders.created", func(msg *nats.Msg) {
        var event OrderCreatedEvent
        json.Unmarshal(msg.Data, &event)

        // 發送通知給使用者
        s.sendEmail(event.UserID, "Your order has been placed!")
    })
}
```

### 如何選擇通訊方式？

| 場景 | 建議 |
|------|------|
| 需要即時回應 | 同步（HTTP/gRPC） |
| 高效能、強型別 | gRPC |
| 對外 API | HTTP/REST |
| 事件通知（不需要回應） | 非同步（Message Queue） |
| 長時間處理的任務 | 非同步 |

## 微服務模式

### API Gateway

所有外部請求先經過 Gateway，再路由到對應的服務：

```
Client → API Gateway → User Service
                     → Order Service
                     → Product Service
```

```go
// 簡化版 API Gateway
func main() {
    mux := http.NewServeMux()

    // 反向代理到不同服務
    userProxy := httputil.NewSingleHostReverseProxy(
        &url.URL{Scheme: "http", Host: "user-service:8080"},
    )
    orderProxy := httputil.NewSingleHostReverseProxy(
        &url.URL{Scheme: "http", Host: "order-service:8080"},
    )

    mux.Handle("/api/users/", userProxy)
    mux.Handle("/api/orders/", orderProxy)

    http.ListenAndServe(":3000", mux)
}
```

生產環境通常用 Kong、Traefik 或 Envoy 等成熟的 API Gateway。

### Service Discovery

服務需要知道其他服務的地址。在 Kubernetes 中，DNS 自動處理。在非 K8s 環境中，可以用 Consul：

```go
// 註冊服務到 Consul
func registerService(consulAddr string) error {
    config := consulapi.DefaultConfig()
    config.Address = consulAddr
    client, err := consulapi.NewClient(config)
    if err != nil {
        return err
    }

    registration := &consulapi.AgentServiceRegistration{
        ID:      "user-service-1",
        Name:    "user-service",
        Port:    8080,
        Address: "10.0.0.5",
        Check: &consulapi.AgentServiceCheck{
            HTTP:     "http://10.0.0.5:8080/health",
            Interval: "10s",
        },
    }

    return client.Agent().ServiceRegister(registration)
}
```

### Circuit Breaker（斷路器）

當下游服務故障時，避免連鎖失敗：

```go
import "github.com/sony/gobreaker"

// 建立斷路器
cb := gobreaker.NewCircuitBreaker(gobreaker.Settings{
    Name:        "user-service",
    MaxRequests: 3,                // half-open 狀態最多嘗試次數
    Interval:    10 * time.Second, // closed 狀態下的統計區間
    Timeout:     30 * time.Second, // open 後多久嘗試 half-open
    ReadyToTrip: func(counts gobreaker.Counts) bool {
        // 連續失敗 5 次就開啟斷路器
        return counts.ConsecutiveFailures >= 5
    },
})

// 使用斷路器
result, err := cb.Execute(func() (interface{}, error) {
    return s.userClient.GetUser(ctx, &userpb.GetUserRequest{Id: userID})
})
if err != nil {
    if err == gobreaker.ErrOpenState {
        // 斷路器開啟中，使用降級策略
        return cachedUser, nil
    }
    return nil, err
}
```

**斷路器三種狀態：**
1. **Closed（關閉）**：正常運作，如果錯誤率超過閾值就打開
2. **Open（打開）**：直接返回錯誤，不呼叫下游服務
3. **Half-Open（半開）**：嘗試少量請求，成功就關閉，失敗就繼續打開

### Distributed Tracing（分散式追蹤）

追蹤一個請求在多個服務之間的流轉：

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
)

func (s *OrderService) CreateOrder(ctx context.Context, req *CreateOrderRequest) (*Order, error) {
    // 開始一個 span
    ctx, span := otel.Tracer("order-service").Start(ctx, "CreateOrder")
    defer span.End()

    // 呼叫 user-service（trace context 會自動傳遞）
    user, err := s.userClient.GetUser(ctx, &userpb.GetUserRequest{Id: req.UserID})
    if err != nil {
        span.RecordError(err)
        return nil, err
    }

    // 呼叫 product-service
    products, err := s.productClient.GetProducts(ctx, req.ProductIDs)
    if err != nil {
        span.RecordError(err)
        return nil, err
    }

    // 建立訂單...
    span.SetAttributes(attribute.Int64("order.id", order.ID))
    return order, nil
}
```

搭配 Jaeger 或 Grafana Tempo 等工具，可以在 UI 上看到完整的請求鏈路。

## 下一步

微服務設計好了，最後一步是把它們部署到生產環境。下一篇——也是本系列的最終篇——我們來學 Kubernetes，了解如何用容器編排平台管理和部署 Go 微服務。
