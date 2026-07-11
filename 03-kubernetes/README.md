# 03. Kubernetes の基本（Minikube）

## 前提（無料）

```bash
minikube version
kubectl version --client

minikube start
kubectl get nodes
```

## 期待結果（セットアップ）

```text
kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1....
```

`STATUS` が `Ready` なら次へ。

## この章で触る概念

| 概念 | 意味 |
|------|------|
| Pod | コンテナ実行の最小単位 |
| Deployment | 台数・更新の管理 |
| Service | 安定したアクセス口 |
| Namespace | 論理仕切り |
| ConfigMap / Secret | 設定・機密 |
| Ingress | 外部 HTTP 入り口（OpenShift では Route） |

## 学習順

1. [01-pod](./01-pod/)
2. [02-deployment](./02-deployment/)
3. [03-service](./03-service/)
4. [04-namespace-config](./04-namespace-config/)
5. [05-ingress](./05-ingress/)（任意）

マニフェスト: [manifests/](./manifests/)

公式: https://kubernetes.io/ja/docs/tutorials/

## トラブル時（セットアップ）

| 症状 | 対処 |
|------|------|
| minikube start 失敗 | Docker が起動しているか確認。`minikube delete` 後に再 `start` |
| kubectl が別クラスタを見ている | `kubectl config use-context minikube` |
| メモリ不足 | 他アプリを閉じる。`minikube start --memory=3072` など調整 |

## 完了条件（DoD）

- [ ] Pod / Deployment / Service の関係を説明できる
- [ ] `apply` / `get` / `describe` / `logs` / `delete` が使える
- [ ] Minikube 上でサンプルを公開できた（Service まで必須、Ingress は任意）

次: [04-kubernetes-ops](../04-kubernetes-ops/)
