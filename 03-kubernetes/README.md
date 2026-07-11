# 03. Kubernetes の基本（Minikube）

この章の目的: **ローカルに小さな Kubernetes を立て、Pod / Deployment / Service でアプリを動かす。**

## 前提コマンド（無料）

Docker Desktop が起動していること（Minikube のドライバに使う）。

```bash
minikube version
kubectl version --client

minikube start
kubectl get nodes
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube version` | Minikube の版を表示 | インストール済みか確認 |
| `kubectl version --client` | kubectl（操作クライアント）の版 | クラスタ操作ツールがあるか確認。`--client` はサーバに繋がなくてよい |
| `minikube start` | ローカル K8s クラスタ起動 | 学習用の「小さな本番に似た環境」を作る |
| `kubectl get nodes` | ノード一覧 | クラスタが Ready か確認する最初の健全性チェック |

## 期待結果（セットアップ）

```text
kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1....
```

`STATUS` が `Ready` なら次へ。

## kubectl でこれから何度も使う型

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f FILE` | マニフェストを適用（宣言どおりにする） | リソース作成・更新 |
| `kubectl get TYPE` | 一覧 | 状態の一次確認 |
| `kubectl describe TYPE NAME` | 詳細 + Events | 障害の理由を読む |
| `kubectl logs ...` | コンテナログ | アプリの出力を見る |
| `kubectl delete -f FILE` | マニフェストのリソース削除 | 片付け |

## この章で触る概念

| 概念 | 意味 | なぜ学ぶか |
|------|------|------------|
| Pod | コンテナ実行の最小単位 | 実際に動く単位 |
| Deployment | 台数・更新の管理 | 本番で普段触るもの |
| Service | 安定したアクセス口 | Pod が入れ替わっても届ける |
| Namespace | 論理仕切り | 環境・チーム分離 |
| ConfigMap / Secret | 設定・機密 | イメージに焼き込まない設定 |
| Ingress | 外部 HTTP 入り口 | OpenShift では Route に相当 |

## 学習順

1. [01-pod](./01-pod/) … 1 個動かす
2. [02-deployment](./02-deployment/) … 台数管理
3. [03-service](./03-service/) … 外から届ける
4. [04-namespace-config](./04-namespace-config/) … 仕切りと設定
5. [05-ingress](./05-ingress/)（任意）… HTTP 入り口

マニフェスト: [manifests/](./manifests/)  
公式: https://kubernetes.io/ja/docs/tutorials/

## トラブル時（セットアップ）

| 症状 | 対処 |
|------|------|
| minikube start 失敗 | Docker 起動確認 → `minikube delete` → `minikube start` |
| 別クラスタを見ている | `kubectl config use-context minikube`（操作先クラスタを切替） |
| メモリ不足 | 他アプリ終了。`minikube start --memory=3072` |

## 完了条件（DoD）

- [ ] Pod / Deployment / Service の関係と、各コマンドの目的を説明できる
- [ ] `apply` / `get` / `describe` / `logs` / `delete` が使える
- [ ] Service まで公開できた（Ingress は任意）

次: [04-kubernetes-ops](../04-kubernetes-ops/)
