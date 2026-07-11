# 02. Docker の基本

## 前提（無料）

次のいずれか:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)（個人学習）
- または Colima / Podman などローカル無料ランタイム

確認:

```bash
docker version
```

Client / Server 双方が表示されれば OK。

## ハンズオン

### 1. 公式イメージを動かす

```bash
docker run --rm -p 8080:80 nginx:alpine
```

別ターミナル:

```bash
curl -I http://localhost:8080
```

起動側は `Ctrl+C` で停止。

### 2. 自分のイメージ

```bash
cd 02-docker/exercises
docker build -t k8s-learn-web:1 .
docker run --rm -p 8081:80 k8s-learn-web:1
```

```bash
curl http://localhost:8081
```

### 3. タグ

```bash
docker images
docker tag k8s-learn-web:1 k8s-learn-web:latest
docker images | grep k8s-learn-web
docker history k8s-learn-web:1
```

学習では **明示タグ（`:1`）** を優先する（`latest` 頼みは現場で事故りやすい）。

## 期待結果

| 操作 | 成功の目安 |
|------|------------|
| `curl -I http://localhost:8080` | `HTTP/1.1 200 OK` |
| `curl http://localhost:8081` | HTML（exercises の内容）が返る |
| `docker images` | `k8s-learn-web` が見える |

## トラブル時

| 症状 | 対処 |
|------|------|
| `Cannot connect to the Docker daemon` | Desktop / Colima を起動 |
| ポート in use | `-p 8082:80` など空いているポートへ変更 |
| pull が遅い・失敗 | ネットワーク再試行。会社 VPN 切断も有効なことあり |

## 完了条件（DoD）

- [ ] イメージとコンテナの違いを説明できる
- [ ] `build` / `run` ができる
- [ ] `-p` とタグの意味が分かる

次: [03-kubernetes](../03-kubernetes/)
