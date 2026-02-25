+++
title = 'Go 語言系列（十六）：gRPC 入門'
date = 2026-02-24T16:00:00+08:00
draft = false
tags = ['Golang', 'Go', 'gRPC', 'Protocol Buffers', 'protobuf', 'RPC', 'Golang系列']
categories = ['技術筆記']
author = 'Jack'
description = '學習 Go 的 gRPC 開發：Protocol Buffers 定義、程式碼生成、Unary RPC、Server Streaming、攔截器，從零建構高效能 RPC 服務'
toc = true
weight = 1
+++

> 這是 **Go 語言從零到 Web 應用**系列的第十六篇。前面我們用 REST API 建構服務，現在來學 Go 的另一個強項——gRPC，了解為什麼它在微服務架構中如此受歡迎。

## gRPC vs REST

| 面向 | REST | gRPC |
|------|------|------|
| **協議** | HTTP/1.1（通常） | HTTP/2 |
| **資料格式** | JSON（文字） | Protocol Buffers（二進位） |
| **型別安全** | 弱（需要手動解析） | 強（自動產生型別） |
| **串流** | 不原生支持 | 原生支持雙向串流 |
| **效能** | 較慢（JSON 解析） | 快 3-10 倍 |
| **瀏覽器支持** | 原生支持 | 需要 gRPC-Web |
| **適用場景** | 公開 API、前端 | 微服務間通訊 |

**什麼時候用 gRPC：**
- 微服務之間的內部通訊
- 需要高效能和低延遲
- 需要串流（streaming）
- 強型別的服務契約

## Protocol Buffers

### 安裝

```bash
# 安裝 protoc 編譯器
brew install protobuf  # macOS

# 安裝 Go 的 protoc plugin
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

確保 `$GOPATH/bin` 在你的 `PATH` 中。

### 定義 .proto 檔案

```protobuf
// proto/book.proto
syntax = "proto3";

package bookstore;

option go_package = "github.com/yourname/bookstore/pb";

// 訊息定義
message Book {
  int64 id = 1;
  string title = 2;
  string author = 3;
  int32 year = 4;
}

message CreateBookRequest {
  string title = 1;
  string author = 2;
  int32 year = 3;
}

message GetBookRequest {
  int64 id = 1;
}

message ListBooksRequest {}

message ListBooksResponse {
  repeated Book books = 1;
}

// 服務定義
service BookService {
  // Unary RPC
  rpc GetBook(GetBookRequest) returns (Book);
  rpc CreateBook(CreateBookRequest) returns (Book);
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse);

  // Server Streaming RPC
  rpc WatchBooks(ListBooksRequest) returns (stream Book);
}
```

### 產生 Go 程式碼

```bash
protoc --go_out=. --go_opt=paths=source_relative \
       --go-grpc_out=. --go-grpc_opt=paths=source_relative \
       proto/book.proto
```

這會產生兩個檔案：
- `proto/book.pb.go`：訊息的 Go 結構體
- `proto/book_grpc.pb.go`：gRPC 服務的介面和 client

## Unary RPC

最基本的 RPC 模式：client 發一個 request，server 回一個 response。

### Server 端

```go
package main

import (
    "context"
    "log"
    "net"
    "sync"

    pb "github.com/yourname/bookstore/proto"
    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

type bookServer struct {
    pb.UnimplementedBookServiceServer
    mu     sync.RWMutex
    books  map[int64]*pb.Book
    nextID int64
}

func newBookServer() *bookServer {
    return &bookServer{
        books:  make(map[int64]*pb.Book),
        nextID: 1,
    }
}

func (s *bookServer) CreateBook(ctx context.Context, req *pb.CreateBookRequest) (*pb.Book, error) {
    s.mu.Lock()
    defer s.mu.Unlock()

    book := &pb.Book{
        Id:     s.nextID,
        Title:  req.Title,
        Author: req.Author,
        Year:   req.Year,
    }
    s.books[s.nextID] = book
    s.nextID++

    return book, nil
}

func (s *bookServer) GetBook(ctx context.Context, req *pb.GetBookRequest) (*pb.Book, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()

    book, ok := s.books[req.Id]
    if !ok {
        return nil, status.Errorf(codes.NotFound, "book %d not found", req.Id)
    }
    return book, nil
}

func (s *bookServer) ListBooks(ctx context.Context, req *pb.ListBooksRequest) (*pb.ListBooksResponse, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()

    books := make([]*pb.Book, 0, len(s.books))
    for _, b := range s.books {
        books = append(books, b)
    }
    return &pb.ListBooksResponse{Books: books}, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    grpcServer := grpc.NewServer()
    pb.RegisterBookServiceServer(grpcServer, newBookServer())

    log.Println("gRPC server listening on :50051")
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### Client 端

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    pb "github.com/yourname/bookstore/proto"
    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
)

func main() {
    // 建立連線
    conn, err := grpc.NewClient("localhost:50051",
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    if err != nil {
        log.Fatalf("failed to connect: %v", err)
    }
    defer conn.Close()

    client := pb.NewBookServiceClient(conn)
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    // 建立書籍
    book, err := client.CreateBook(ctx, &pb.CreateBookRequest{
        Title:  "The Go Programming Language",
        Author: "Donovan & Kernighan",
        Year:   2015,
    })
    if err != nil {
        log.Fatalf("CreateBook failed: %v", err)
    }
    fmt.Printf("Created: %+v\n", book)

    // 取得書籍
    got, err := client.GetBook(ctx, &pb.GetBookRequest{Id: book.Id})
    if err != nil {
        log.Fatalf("GetBook failed: %v", err)
    }
    fmt.Printf("Got: %+v\n", got)

    // 列出所有書籍
    resp, err := client.ListBooks(ctx, &pb.ListBooksRequest{})
    if err != nil {
        log.Fatalf("ListBooks failed: %v", err)
    }
    for _, b := range resp.Books {
        fmt.Printf("  - %s by %s (%d)\n", b.Title, b.Author, b.Year)
    }
}
```

## Streaming RPC

### Server Streaming

Server 一次回傳多個 response（例如即時推送新增的書籍）：

```go
// Server 端
func (s *bookServer) WatchBooks(req *pb.ListBooksRequest, stream pb.BookService_WatchBooksServer) error {
    // 先送出現有的書籍
    s.mu.RLock()
    for _, book := range s.books {
        if err := stream.Send(book); err != nil {
            s.mu.RUnlock()
            return err
        }
    }
    s.mu.RUnlock()

    // 持續監聽新書（簡化範例用 ticker 模擬）
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()

    for i := 0; i < 5; i++ {
        select {
        case <-ticker.C:
            book := &pb.Book{
                Id:    int64(100 + i),
                Title: fmt.Sprintf("New Book %d", i),
            }
            if err := stream.Send(book); err != nil {
                return err
            }
        case <-stream.Context().Done():
            return stream.Context().Err()
        }
    }
    return nil
}
```

```go
// Client 端
stream, err := client.WatchBooks(ctx, &pb.ListBooksRequest{})
if err != nil {
    log.Fatal(err)
}

for {
    book, err := stream.Recv()
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Received: %s\n", book.Title)
}
```

## gRPC 攔截器（Interceptor）

攔截器是 gRPC 版本的中間件，可以在 RPC 呼叫前後加入邏輯。

### Unary Interceptor

```go
// Logging Interceptor
func loggingInterceptor(
    ctx context.Context,
    req interface{},
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (interface{}, error) {
    start := time.Now()

    // 呼叫實際的 handler
    resp, err := handler(ctx, req)

    // 記錄日誌
    log.Printf("method=%s duration=%s error=%v",
        info.FullMethod,
        time.Since(start),
        err,
    )

    return resp, err
}

// 認證 Interceptor
func authInterceptor(
    ctx context.Context,
    req interface{},
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (interface{}, error) {
    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return nil, status.Errorf(codes.Unauthenticated, "missing metadata")
    }

    tokens := md.Get("authorization")
    if len(tokens) == 0 {
        return nil, status.Errorf(codes.Unauthenticated, "missing token")
    }

    // 驗證 token...
    return handler(ctx, req)
}

// 註冊攔截器
grpcServer := grpc.NewServer(
    grpc.ChainUnaryInterceptor(
        loggingInterceptor,
        authInterceptor,
    ),
)
```

### Stream Interceptor

```go
func streamLoggingInterceptor(
    srv interface{},
    ss grpc.ServerStream,
    info *grpc.StreamServerInfo,
    handler grpc.StreamHandler,
) error {
    start := time.Now()
    err := handler(srv, ss)
    log.Printf("stream method=%s duration=%s error=%v",
        info.FullMethod,
        time.Since(start),
        err,
    )
    return err
}

grpcServer := grpc.NewServer(
    grpc.ChainUnaryInterceptor(loggingInterceptor),
    grpc.ChainStreamInterceptor(streamLoggingInterceptor),
)
```

## 錯誤處理

gRPC 有自己的錯誤碼系統：

```go
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

// Server 端回傳錯誤
return nil, status.Errorf(codes.NotFound, "book %d not found", id)
return nil, status.Errorf(codes.InvalidArgument, "title is required")
return nil, status.Errorf(codes.Internal, "database error: %v", err)

// Client 端處理錯誤
book, err := client.GetBook(ctx, req)
if err != nil {
    st, ok := status.FromError(err)
    if ok {
        switch st.Code() {
        case codes.NotFound:
            fmt.Println("Book not found")
        case codes.InvalidArgument:
            fmt.Println("Invalid request:", st.Message())
        default:
            fmt.Println("Error:", st.Message())
        }
    }
}
```

常用的 gRPC 狀態碼：

| 碼 | 說明 | 對應 HTTP |
|----|------|-----------|
| OK | 成功 | 200 |
| NotFound | 資源不存在 | 404 |
| InvalidArgument | 參數錯誤 | 400 |
| Unauthenticated | 未認證 | 401 |
| PermissionDenied | 無權限 | 403 |
| Internal | 內部錯誤 | 500 |

## 下一步

學會了 gRPC，我們有了微服務之間通訊的利器。下一篇來學微服務架構的核心概念——如何拆分服務、服務間如何溝通、以及必須知道的設計模式。
