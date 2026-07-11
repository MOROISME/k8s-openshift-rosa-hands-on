# 03. Kubernetes の基本（Minikube）

この章の目的: **ローカルに小さな Kubernetes を立て、Pod / Deployment / Service でアプリを動かす。**

費用: **すべて無料**（ローカルのみ。クラウドの有料 K8s は使わない）。

> **進め方:** 下のハンズオンは **自分のターミナルで順番に実行**してください。  
> インストール・クラスタ起動も学習の一部です（代理実行せず、手順どおりに実施する）。

---

## クラスタとは（先にこれ）

**クラスタ** = Kubernetes がアプリを動かすための **コンピュータの集まり（ひとまとまりの実行環境）** です。

```text
クラスタ
├── コントロールプレーン（頭脳）
│     API を受け取り、「どこで何を動かすか」を決める
└── ノード（手足・実際のマシン）
      コンテナ（Pod）が実際に動く場所
```

| 用語 | たとえ | 意味 |
|------|--------|------|
| クラスタ | 店舗（厨房が複数あっても「その店」） | K8s で管理する単位の全体 |
| ノード | 厨房（実機 / VM） | コンテナが載るマシン 1 台 |
| コントロールプレーン | 店長・司令塔 | クラスタ全体の制御 |
| `kubectl` | 店員への指示口 | あなたがクラスタに命令するクライアント |
| Minikube | 自宅学習用の小さな店舗 | PC 上に **1 ノードの学習用クラスタ** を作るツール |

### なぜ「クラスタ」と言うのか

本番では複数マシンにアプリを分散します。その集合をまとめてクラスタと呼びます。  
Minikube は学習用なので **マシンは実質 1 台**ですが、操作の型（`kubectl`・Pod・Deployment）は本番クラスタと同じです。

| 環境 | クラスタの実態 |
|------|----------------|
| Minikube（この章） | 自分の PC（Docker）上の小さな学習用クラスタ |
| 会社の本番 / ROSA | クラウド上の複数ノードからなる本格クラスタ |

`minikube start` = **この学習用クラスタを起動する**、という意味です。  
`kubectl get nodes` = **クラスタに属するマシン（ノード）の一覧を見る**、という意味です。

---

## ハンズオン A: 環境構築（インストール〜クラスタ起動）

### A-0. 全体の流れ

```text
1. Docker Desktop を入れる・起動する
2. Homebrew で minikube / kubectl を入れる
3. minikube start でクラスタを起動する
4. kubectl get nodes で Ready を確認する
5. ハンズオン B（01-pod 以降）へ進む
```

### A-1. Docker Desktop（前提）

Minikube（docker ドライバ）は、裏で Docker を使います。

1. 未導入なら [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストール
2. Docker Desktop を起動し、メニューバーのアイコンが安定するまで待つ
3. ターミナルで確認:

```bash
docker version
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `docker version` | Client / Server 表示 | **Server が出ること**＝エンジン起動済み。Client だけのエラーなら Desktop 未起動 |

- [ ] `Client:` と `Server:` の両方表示された

### A-2. Homebrew の確認

```bash
brew --version
```

| 結果 | 次にすること |
|------|----------------|
| 版が出る | A-3 へ |
| `command not found` | 先に [Homebrew](https://brew.sh/) を入れる |

Homebrew が無い場合の公式インストール例:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

終わったら画面の指示どおり `PATH` を通し、**新しいターミナル**で `brew --version` を再確認。

- [ ] `brew --version` が通る

### A-3. minikube / kubectl のインストール

**ここを自分で実行する（ハンズオン本体）。**

```bash
brew install minikube
brew install kubectl
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `brew install minikube` | ローカル K8s ツールを入れる | 学習用クラスタを作・消できる |
| `brew install kubectl` | Kubernetes 操作クライアントを入れる | Pod 等を apply / get する |

`brew install minikube` の依存で `kubernetes-cli`（kubectl）が入ることがあります。その場合は次の確認で `kubectl` が通れば、`brew install kubectl` は省略してよい。

確認:

```bash
which minikube
which kubectl
minikube version
kubectl version --client
```

| コマンド | 成功の目安 | 目的 |
|----------|------------|------|
| `which minikube` | `/opt/homebrew/bin/minikube` など | PATH に載っている |
| `which kubectl` | パスが出る | 同上 |
| `minikube version` | `minikube version: v1....` | 実行できる |
| `kubectl version --client` | `Client Version: v1....` | クラスタ無しでもクライアント確認可 |

`command not found` のとき: ターミナルを開き直す。`echo $PATH` に `/opt/homebrew/bin` があるか確認。

- [ ] `minikube version` が通る
- [ ] `kubectl version --client` が通る

### A-4. クラスタ起動

```bash
minikube start --driver=docker
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube start` | ローカルに K8s を 1 クラスタ作って起動 | ハンズオン用環境を用意する |
| `--driver=docker` | Docker Desktop 上で動かす | この教材の推奨 |

初回はベースイメージのダウンロードで **数分**かかることがあります。  
`Done! kubectl is now configured to use "minikube"` の趣旨が出れば起動成功。

続けて確認:

```bash
minikube status
kubectl get nodes
kubectl config current-context
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube status` | host / kubelet 等 | Minikube が Running か |
| `kubectl get nodes` | ノード一覧 | K8s として Ready か |
| `kubectl config current-context` | 操作中クラスタ名 | `minikube` か確認 |

期待結果:

```text
kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1....
```

`current-context` が `minikube` でない場合:

```bash
kubectl config use-context minikube
```

- [ ] `minikube status` が Running 系
- [ ] `kubectl get nodes` が **Ready**
- [ ] context が `minikube`

**ここまでできたら** → [01-pod](./01-pod/) へ（ハンズオン B）。

### A-5. 日常の起動・停止（参考）

| やりたいこと | コマンド | 目的 |
|--------------|----------|------|
| 起動 | `minikube start` | 学習再開 |
| 状態 | `minikube status` | 確認 |
| 停止（残す） | `minikube stop` | リソースを空けつつ定義は残す |
| 削除（作り直し） | `minikube delete` | 壊れたとき・最初から |

---

## kubectl でこれから何度も使う型

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f FILE` | マニフェストを適用 | リソース作成・更新 |
| `kubectl get TYPE` | 一覧 | 状態の一次確認 |
| `kubectl describe TYPE NAME` | 詳細 + Events | 障害の理由を読む |
| `kubectl logs ...` | コンテナログ | アプリの出力を見る |
| `kubectl delete -f FILE` | マニフェスト削除 | 片付け |

**マニフェストのパス**は、今いるディレクトリ基準です。

```bash
# 例: リポジトリルートから
kubectl apply -f 03-kubernetes/manifests/01-pod.yaml

# 例: 03-kubernetes/01-pod にいるとき
kubectl apply -f ../manifests/01-pod.yaml
```

---

## この章で触る概念

| 概念 | 意味 | なぜ学ぶか |
|------|------|------------|
| Pod | コンテナ実行の最小単位 | 実際に動く単位 |
| Deployment | 台数・更新の管理 | 本番で普段触るもの |
| Service | 安定したアクセス口 | Pod が入れ替わっても届ける |
| Namespace | 論理仕切り | 環境・チーム分離 |
| ConfigMap / Secret | 設定・機密 | イメージに焼き込まない設定 |
| Ingress | 外部 HTTP 入り口 | OpenShift では Route に相当 |

## 学習順（ハンズオン B）

**ハンズオン A（環境構築）が終わってから:**

1. [01-pod](./01-pod/) … 1 個動かす
2. [02-deployment](./02-deployment/) … 台数管理
3. [03-service](./03-service/) … 外から届ける
4. [04-namespace-config](./04-namespace-config/) … 仕切りと設定
5. [05-ingress](./05-ingress/)（任意）… HTTP 入り口

マニフェスト: [manifests/](./manifests/)  
公式: https://kubernetes.io/ja/docs/tutorials/  
Minikube 公式: https://minikube.sigs.k8s.io/docs/start/

---

## トラブル時（インストール〜起動）

| 症状 | 原因の目安 | 対処 |
|------|------------|------|
| `the path ".../manifests/..." does not exist` | 作業ディレクトリが違う | `cd` してから、またはルートから `03-kubernetes/manifests/...` を指定 |
| `minikube: command not found` | 未インストール / PATH | `brew install minikube` → ターミナル再起動 |
| `Cannot connect to the Docker daemon` | Docker 未起動 | Docker Desktop を起動 → `docker version` で Server 確認 |
| `localhost:8080 connection refused` | クラスタ未起動 or 文脈違い | `minikube start` → `kubectl config use-context minikube` |
| start が途中で失敗 | 初回 DL・資源不足 | 再実行。ダメなら `minikube delete` → `minikube start --driver=docker` |
| メモリ不足 | 他アプリが重い | 終了させる。`minikube start --memory=3072` |
| Apple Silicon で怪しい | 古いバイナリ等 | `brew reinstall minikube` |

---

## セットアップの完了条件（DoD）＝ ハンズオン A

- [ ] `docker version` で Server が見える
- [ ] **自分で** `brew install minikube`（と必要なら kubectl）を実行した
- [ ] `minikube version` / `kubectl version --client` が通る
- [ ] **自分で** `minikube start --driver=docker` した
- [ ] `kubectl get nodes` が Ready、context が `minikube`

これが揃ったら [01-pod](./01-pod/) へ。

## 章全体の完了条件（DoD）

- [ ] Pod / Deployment / Service の関係と、各コマンドの目的を説明できる
- [ ] `apply` / `get` / `describe` / `logs` / `delete` が使える
- [ ] Service まで公開できた（Ingress は任意）

次: [04-kubernetes-ops](../04-kubernetes-ops/)（章のハンズオン完了後）
