## ADDED Requirements

### Requirement: Kubernetes 文章結構
文章 SHALL 使用 Hugo front matter，title 為「Go 語言系列（十八）：Kubernetes 部署」，date 為 2026-02-25T18:00:00+08:00，tags 包含 'Golang系列'，weight = 1。

#### Scenario: Front matter 格式正確
- **WHEN** 文章被 Hugo 讀取
- **THEN** front matter 包含正確的 title、date、tags（含 'Golang系列'）、categories='技術筆記'、author='Jack'、toc=true、weight=1

### Requirement: K8s 基礎概念
文章 SHALL 介紹 Kubernetes 的核心概念：Pod、Deployment、Service、Namespace，用圖示或文字解釋元件關係。

#### Scenario: 概念說明
- **WHEN** 讀者閱讀基礎概念章節
- **THEN** 文章清楚解釋 Pod/Deployment/Service 的角色和關係

### Requirement: 本地環境設定
文章 SHALL 提供使用 minikube 或 kind 建立本地 K8s 叢集的步驟。

#### Scenario: 環境設定
- **WHEN** 讀者閱讀環境章節
- **THEN** 文章提供可執行的 minikube/kind 安裝和啟動指令

### Requirement: Go 應用的 Dockerfile 最佳化
文章 SHALL 介紹多階段建構（multi-stage build）和最小化映像的技巧。

#### Scenario: Multi-stage Dockerfile
- **WHEN** 讀者閱讀 Docker 章節
- **THEN** 文章提供 Go 應用的 multi-stage Dockerfile 和 scratch/distroless base image 範例

### Requirement: K8s 資源定義
文章 SHALL 提供完整的 Deployment、Service、ConfigMap、Secret YAML 範例，展示如何部署 Go 應用到 K8s。

#### Scenario: 部署 YAML
- **WHEN** 讀者閱讀部署章節
- **THEN** 文章提供可直接 kubectl apply 的完整 YAML 範例

### Requirement: 健康檢查與滾動更新
文章 SHALL 介紹 liveness/readiness probe 的設定和滾動更新策略。

#### Scenario: Health Check 設定
- **WHEN** 讀者閱讀健康檢查章節
- **THEN** 文章展示 Go 應用中實作 health endpoint 和 K8s probe 設定

### Requirement: Helm 基礎
文章 SHALL 介紹 Helm chart 的基本結構和使用方式。

#### Scenario: Helm chart 範例
- **WHEN** 讀者閱讀 Helm 章節
- **THEN** 文章展示基本的 Chart.yaml、values.yaml 和 template 結構

### Requirement: 系列導覽
文章開頭 SHALL 有引言說明此為系列最終篇，末尾 SHALL 有完整系列回顧。

#### Scenario: 系列完結
- **WHEN** 讀者開啟文章
- **THEN** 開頭有引言段落，末尾有 18 篇文章的完整回顧和學習路徑總結
