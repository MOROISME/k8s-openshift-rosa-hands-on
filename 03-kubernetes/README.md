# 03. Kubernetes の基本（Minikube ハンズオン）

## Kubernetes とは

Linux サーバー上で、コンテナ化したアプリを複数サーバーにわたって自動管理する仕組み。  
起動・停止・再起動・スケーリング・配置などを任せられる。

| 用語 | 例え |
|------|------|
| コンテナ化したアプリ | 完成済みの料理キット |
| サーバー | 店舗の厨房 |
| Kubernetes | 店舗全体を管理する店長・エリアマネージャー |

## この章で触る概念

| 概念 | ざっくり意味 |
|------|----------------|
| Pod | コンテナを動かす最小単位 |
| Deployment | Pod の望ましい台数・更新を管理 |
| Service | Pod への安定したアクセス口 |
| Namespace | クラスタ内の論理的な仕切り |
| ConfigMap / Secret | 設定値・機密情報 |
| Ingress | 外部 HTTP(S) 入り口（後で OpenShift では Route） |

## 前提セットアップ

```bash
# Minikube / kubectl が入っていること
minikube version
kubectl version --client

# クラスタ起動（初回は時間がかかる）
minikube start

# 動作確認
kubectl get nodes
```

公式チュートリアル: https://kubernetes.io/ja/docs/tutorials/

## 学習の進め方

順番に README を進めてください。

1. [01-pod](./01-pod/)
2. [02-deployment](./02-deployment/)
3. [03-service](./03-service/)
4. [04-namespace-config](./04-namespace-config/)
5. [05-ingress](./05-ingress/)（任意）

マニフェストのまとめ: [manifests/](./manifests/)

## 章のゴール

- [ ] Pod / Deployment / Service の関係を説明できる
- [ ] `kubectl apply` / `get` / `describe` / `logs` / `delete` が使える
- [ ] Minikube 上でサンプルアプリを公開できる

次: [04-openshift](../04-openshift/)
