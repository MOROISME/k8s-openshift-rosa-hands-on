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

#### Dockerfile とは

**イメージをどう作るかを書いたレシピ**です。  
`docker build` は、このファイルの指示を上から実行してイメージを積み上げます。

この演習のファイル: [`exercises/Dockerfile`](./exercises/Dockerfile)

```dockerfile
# 公式 nginx（Alpine 版）を土台にする
FROM nginx:alpine

# 自分の HTML を nginx の公開ディレクトリへコピーする
COPY index.html /usr/share/nginx/html/index.html
```

#### 行ごとの意味と目的

| 命令 | 書き方 | 意味 | 目的 |
|------|--------|------|------|
| `FROM` | `FROM nginx:alpine` | ベースにするイメージを指定 | ゼロから OS を組み立てず、動く nginx 付き土台から始める |
| `COPY` | `COPY 元 先` | ビルド文脈（`.`）のファイルをイメージ内へコピー | 自分の `index.html` を Web の公開場所に置く |

補足:

| 項目 | 内容 |
|------|------|
| `nginx:alpine` | Alpine Linux 上の軽量 nginx。コンテナ内では通常 **80** 番で待ち受け |
| `index.html`（左側） | `exercises/` にある自分のファイル（ビルド文脈からの相対パス） |
| `/usr/share/nginx/html/index.html`（右側） | コンテナ内のパス。nginx 公式イメージが「ここに置いたファイルを返す」場所 |
| コメント（`#`） | 人間向け。ビルドには影響しない |

#### ビルドの流れ（イメージ）

```text
exercises/
  Dockerfile      ← レシピ
  index.html      ← コピーされる材料
        │
        │  docker build -t k8s-learn-web:1 .
        ▼
イメージ k8s-learn-web:1
  = nginx:alpine の中身
    + 差し替えた index.html
        │
        │  docker run -p 8081:80
        ▼
コンテナ（中の nginx が 80 で待ち受け）
  ← ホストの 8081 からアクセス
```

#### `index.html` との関係

[`exercises/index.html`](./exercises/index.html) は、返す Web ページ本体です。  
Dockerfile の `COPY` がこれをイメージに焼き込みます。HTML を変えたら **再 `docker build`** しないとコンテナには反映されません。

#### ビルドと起動

```bash
cd 02-docker/exercises
docker build -t k8s-learn-web:1 .
docker run --rm -p 8081:80 k8s-learn-web:1
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `cd 02-docker/exercises` | 作業ディレクトリ移動 | Dockerfile と index.html がある場所へ行く（`.` の中身になる） |
| `docker build -t 名前:タグ .` | Dockerfile に従ってイメージ作成 | `-t` で名前付け。`.` = ビルド文脈（このフォルダを材料にする） |
| `docker run ... k8s-learn-web:1` | 作ったイメージを起動 | 公式イメージと同じ流れで自分製を動かす |

```bash
curl http://localhost:8081
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `curl`（`-I` なし） | レスポンス本体も取得 | 「Welcome to nginx」ではなく、自分の `Hello from Docker` が返るか確認 |

公式の `nginx:alpine` だけ動かしたとき（8080）との違い:

| | 公式イメージそのまま | 自分で build したイメージ |
|--|----------------------|---------------------------|
| 中身 | nginx 既定の Welcome ページ | `exercises/index.html` |
| 確認 | `curl -I` で 200 なら十分 | `curl` で HTML 本文を見る |

#### よくある疑問

| 疑問 | 答え |
|------|------|
| なぜ `FROM` が先？ | 土台が無いと `COPY` 先のファイルシステムがない。Dockerfile は通常 `FROM` から始める |
| `RUN` は？ | パッケージインストール等で使う。この最小例では不要 |
| `CMD` / `ENTRYPOINT` は？ | 「起動時に何を実行するか」。nginx イメージが既に持っているので省略している |
| なぜ再 build が必要？ | `COPY` は **build 時** に焼き込む。起動中のコンテナのファイルを外から自動更新しない |
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
- [ ] Dockerfile の `FROM` と `COPY` の意味・目的を説明できる
- [ ] 「なぜ自分でイメージを作るか」を一文で言える

次: [03-kubernetes](../03-kubernetes/)
