# 03. Kubernetes の基本（Minikube）

この章の目的: **ローカルに小さな Kubernetes を立て、Pod / Deployment / Service でアプリを動かす。**

費用: **すべて無料**（ローカルのみ。クラウドの有料 K8s は使わない）。

---

## 0. 全体の流れ（先にこれ）

```text
1. Docker Desktop を入れる・起動する
2. Homebrew で minikube / kubectl を入れる
3. minikube start でクラスタを起動する
4. kubectl get nodes で Ready を確認する
5. 01-pod 以降のハンズオンへ進む
```

---

## 1. 前提ソフト: Docker Desktop

Minikube（docker ドライバ）は、裏で Docker を使います。

1. [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストール（未導入なら）
2. Docker Desktop を起動し、メニューバーのアイコンが安定するまで待つ
3. 確認:

```bash
docker version
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `docker version` | Client / Server 表示 | **Server が出ること**＝エンジン起動済み。Client だけのエラーなら Desktop 未起動 |

---

## 2. Minikube / kubectl のインストール（macOS）

### 2-1. Homebrew があるか確認

```bash
brew --version
```

| 結果 | 次にすること |
|------|----------------|
| 版が出る | 次の「インストール」へ |
| `command not found` | 先に [Homebrew](https://brew.sh/) を入れる（公式の install スクリプト） |

Homebrew 新規インストール例（公式サイトのコマンドを使う）:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール後、画面の指示どおり `PATH` を通す（Apple Silicon では `/opt/homebrew/bin` を PATH に追加、など）。

### 2-2. minikube と kubectl を入れる

```bash
brew install minikube
brew install kubectl
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `brew install minikube` | ローカル K8s ツールを入れる | 学習用クラスタを作・消できる |
| `brew install kubectl` | Kubernetes 操作クライアントを入れる | Pod 等を apply / get する |

`brew install minikube` のときに依存で `kubernetes-cli`（kubectl）が入ることもあります。その場合は `kubectl version --client` が通れば `brew install kubectl` は省略可。

### 2-3. インストールできたか確認

```bash
which minikube
which kubectl
minikube version
kubectl version --client
```

| コマンド | 成功の目安 | 目的 |
|----------|------------|------|
| `which minikube` | `/opt/homebrew/bin/minikube` などパスが出る | コマンドが PATH にいる |
| `which kubectl` | パスが出る | 同上 |
| `minikube version` | `minikube version: v1....` | 実行できる |
| `kubectl version --client` | `Client Version: v1....` | クラスタ無しでもクライアント確認可 |

`command not found` のとき:

- 新しいターミナルを開き直す
- `echo $PATH` に `/opt/homebrew/bin`（または Homebrew が表示したパス）があるか確認

---

## 3. クラスタ起動

```bash
minikube start --driver=docker
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube start` | ローカルに K8s を 1 クラスタ作って起動 | ハンズオン用の「小さな本番に似た環境」 |
| `--driver=docker` | Docker Desktop 上で動かす | この教材の推奨（明示すると迷いが減る） |

初回はベースイメージのダウンロードで数分かかることがあります。  
成功すると `Done! kubectl is now configured to use "minikube"` の趣旨のメッセージが出ます。

状態確認:

```bash
minikube status
kubectl get nodes
kubectl config current-context
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube status` | host / kubelet 等の状態 | Minikube 自体が Running か |
| `kubectl get nodes` | ノード一覧 | K8s として Ready か |
| `kubectl config current-context` | 今操作しているクラスタ名 | `minikube` になっているか確認 |

### 期待結果

```text
kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1....
```

`STATUS` が **Ready** ならセットアップ完了 → [01-pod](./01-pod/) へ。

`current-context` が `minikube` でない場合:

```bash
kubectl config use-context minikube
```

| コマンド | 目的 |
|----------|------|
| `kubectl config use-context minikube` | kubectl の操作先を Minikube に切り替える |

---

## 4. よく使う日常コマンド（起動・停止）

| やりたいこと | コマンド | 目的 |
|--------------|----------|------|
| 起動 | `minikube start` | 学習再開 |
| 状態 | `minikube status` | 動いているか確認 |
| 停止（残す） | `minikube stop` | PC リソースを空けつつクラスタ定義は残す |
| 削除（作り直し） | `minikube delete` | 壊れたとき・綺麗に最初から |

---

## 5. kubectl でこれから何度も使う型

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

## 学習順

セットアップ（上記 1〜3）が終わってから:

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

## セットアップの完了条件（DoD）

- [ ] `docker version` で Server が見える
- [ ] `minikube version` / `kubectl version --client` が通る
- [ ] `minikube start` 後、`kubectl get nodes` が Ready
- [ ] `kubectl config current-context` が `minikube`

これが揃ったら [01-pod](./01-pod/) へ。

## 章全体の完了条件（DoD）

- [ ] Pod / Deployment / Service の関係と、各コマンドの目的を説明できる
- [ ] `apply` / `get` / `describe` / `logs` / `delete` が使える
- [ ] Service まで公開できた（Ingress は任意）

次: [04-kubernetes-ops](../04-kubernetes-ops/)（章のハンズオン完了後）
