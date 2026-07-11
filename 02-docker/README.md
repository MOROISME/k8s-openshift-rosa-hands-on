# 02. Docker の基本

この Step の目的: **イメージとコンテナの違いを体験し、あとで Kubernetes が管理する「箱」を自分で作れるようにする。**

## 前提（無料）

次のいずれか:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)（個人学習）
- または Colima / Podman などローカル無料ランタイム

### 確認コマンド

```bash
docker version
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `docker version` | Client / Server の版を表示 | **daemon（エンジン）に繋がっているか**を確認する。Client だけ出て Server がエラーなら Desktop 未起動 |

成功: `Client:` と `Server:` の両方が出る。

---

## 用語（先にこれだけ）

| 用語 | 意味 |
|------|------|
| イメージ | アプリの設計図・テンプレ（例: `nginx:alpine`） |
| コンテナ | イメージから起動した実行中の実体 |
| タグ | イメージの版名（例: `:alpine` / `:1`） |
| レジストリ | イメージの置き場（Docker Hub など） |

---

## ハンズオン

### 1. 公式イメージを動かす

**この節の目的:** 他人が作ったイメージを pull して動かし、「コンテナ = 動いているプロセス」を体感する。

```bash
docker run --rm -p 8080:80 nginx:alpine
```

| 部分 | 意味 | 目的 |
|------|------|------|
| `docker run` | イメージからコンテナを起動 | 実行する |
| `--rm` | 終了時にコンテナ削除 | お試しのゴミを残さない |
| `-p 8080:80` | ホスト8080 → コンテナ80 | Mac から中の nginx に届ける |
| `nginx:alpine` | 使うイメージ | 軽量 Web サーバの定番サンプル |

別ターミナル:

```bash
curl -I http://localhost:8080
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `curl -I` | HTTP ヘッダだけ取得 | コンテナ内 nginx に届いているか確認（200 なら成功） |

起動側は `Ctrl+C` で停止（`--rm` なのでコンテナも消える）。

### 2. 自分のイメージを作る

**この節の目的:** Dockerfile からイメージをビルドし、「自分のアプリをコンテナ化する」最小体験をする。

```bash
cd 02-docker/exercises
docker build -t k8s-learn-web:1 .
docker run --rm -p 8081:80 k8s-learn-web:1
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `cd 02-docker/exercises` | 作業ディレクトリ移動 | Dockerfile がある場所へ行く |
| `docker build -t 名前:タグ .` | カレントの Dockerfile からイメージ作成 | `-t` で名前付け。`.` はビルド文脈（このフォルダ） |
| `docker run ... k8s-learn-web:1` | 作ったイメージを起動 | 公式イメージと同じ流れで自分製を動かす |

`exercises/Dockerfile` の意味:

| 行 | 意味 |
|----|------|
| `FROM nginx:alpine` | 土台イメージ |
| `COPY index.html ...` | 自分の HTML を nginx の公開ディレクトリへコピー |

```bash
curl http://localhost:8081
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `curl`（`-I` なし） | レスポンス本体も取得 | 自分の `index.html` が返るか確認 |

### 3. 一覧・タグ・レイヤ

**この節の目的:** イメージの管理（名前・版・中身の積み重ね）を見る。現場の「どの版をデプロイしたか」問題の原型。

```bash
docker images
docker tag k8s-learn-web:1 k8s-learn-web:latest
docker images | grep k8s-learn-web
docker history k8s-learn-web:1
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `docker images` | ローカルイメージ一覧 | 何が入っているか確認 |
| `docker tag A B` | 同じ実体に別名タグを付ける | レジストリへ推す前によく使う。実体のコピーではない |
| `... \| grep ...` | 一覧から絞る | 目的のイメージだけ見る |
| `docker history` | イメージのレイヤ履歴 | 「どう積み上がったか」を眺める |

学習では **明示タグ（`:1`）** を優先（`latest` 頼みは現場で事故りやすい）。

---

## 期待結果

| 操作 | 成功の目安 |
|------|------------|
| `curl -I http://localhost:8080` | `HTTP/1.1 200 OK` |
| `curl http://localhost:8081` | exercises の HTML |
| `docker images` | `k8s-learn-web` が見える |

## トラブル時

| 症状 | 対処 |
|------|------|
| `Cannot connect to the Docker daemon` | Desktop / Colima を起動し、`docker version` で Server を確認 |
| ポート in use | `-p 8082:80` など空ポートへ |
| pull 失敗 | ネット再試行。VPN 切断が有効なことあり |

## 完了条件（DoD）

- [ ] イメージとコンテナの違いを説明できる
- [ ] `build` / `run` / `-p` / タグの目的を説明できる
- [ ] 「なぜ自分でイメージを作るか」を一文で言える

次: [03-kubernetes](../03-kubernetes/)
