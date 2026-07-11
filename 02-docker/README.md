# 02. Docker の基本

Docker は「アプリをコンテナ化する技術」です。Kubernetes は、そのコンテナを多数のマシンで管理します。

## 前提

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) がインストール済み
- ターミナルで `docker version` が通る

## 学ぶこと

- イメージとコンテナの違い
- ビルド / 実行 / 停止 / 削除
- ポート公開

## ハンズオン

### 1. 公式イメージを動かす

```bash
docker run --rm -p 8080:80 nginx:alpine
```

別ターミナル:

```bash
curl -I http://localhost:8080
```

止めるときは、起動したターミナルで `Ctrl+C`。

### 2. 自分のイメージを作る

このリポジトリの `exercises/` を使います。

```bash
cd 02-docker/exercises
docker build -t k8s-learn-web:1 .
docker run --rm -p 8081:80 k8s-learn-web:1
```

```bash
curl http://localhost:8081
```

### 3. 状態確認

```bash
docker images
docker ps -a
```

## チェックリスト

- [ ] イメージとコンテナの違いを説明できる
- [ ] `docker build` / `docker run` ができる
- [ ] ポート `-p` の意味が分かる

次: [03-kubernetes](../03-kubernetes/)
