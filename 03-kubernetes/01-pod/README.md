# 01. Pod

Pod は Kubernetes でコンテナを動かす**最小単位**です。  
**この節の目的:** 「クラスタの上でコンテナが 1 個 Running になる」感覚と、観察コマンドを身につける。

## 前提

先に [../README.md](../README.md) の **ハンズオン A（環境構築）** を自分のターミナルで完了し、`kubectl get nodes` が Ready であること。

```bash
kubectl get nodes
# minikube   Ready   ...
```

## ハンズオン

**作業ディレクトリ:** 先に `03-kubernetes/01-pod` へ移動する（`02-docker/exercises` など別フォルダでは相対パスが解決しない）。

```bash
cd 03-kubernetes/01-pod
# リポジトリ直下からの例:
# cd /Users/あなた/study/k8s-openshift-rosa-hands-on/03-kubernetes/01-pod

kubectl apply -f ../manifests/01-pod.yaml
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod
```

リポジトリのルートから打つ場合:

```bash
kubectl apply -f 03-kubernetes/manifests/01-pod.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | YAML の内容をクラスタに反映 | Pod を作成する |
| `kubectl get pods` | Pod 一覧 | STATUS / READY を一次確認 |
| `kubectl describe pod NAME` | 詳細と Events | 「なぜその状態か」を読む（障害の入口） |
| `kubectl logs NAME` | コンテナ標準出力 | アプリが出したログを見る |

### 任意: 対話型でコンテナの中に入る（`kubectl exec -it`）

**目的:** Pod の中を Linux シェルのように覗き、障害切り分けや「中身の確認」をする。

#### 「中」とはどこか（入れ子のイメージ）

Minikube（`--driver=docker`）では、だいたい次の入れ子になっています。

```text
あなたの Mac
  └── Docker Desktop
        └── Minikube 用のコンテナ（＝ノードの実体）
              └── Pod（例: hello-pod）
                    └── アプリコンテナ（例: nginx）  ← exec で入る「中」
```

| 言うとき | 指している場所 |
|----------|----------------|
| Docker の中 | Docker Desktop が動かしている世界（Minikube のノードもここにいる） |
| Pod の中 | K8s 上の単位。中に 1 つ以上のコンテナがある |
| **`exec` の「中」** | **その Pod 内のアプリコンテナ（今回は nginx）の Linux 環境** |

覚え方:

- Pod ≈ 「箱のグループの名前」
- コンテナ ≈ 「実際にシェルがある中身」
- `kubectl exec` ≈ 「その中身の OS に入る」

つまり「Docker の中の Pod の中」というより、  
**「Minikube クラスタ上の Pod の中にある、nginx コンテナの中」** です。Docker はその外側の土台です。

#### 入り方

```bash
kubectl exec -it hello-pod -- /bin/sh
```

| 部分 | 意味 | 目的 |
|------|------|------|
| `kubectl exec` | Pod 内でコマンドを実行する | 外から中を操作する |
| `-i` | 標準入力を繋ぐ | キー入力を中に渡す |
| `-t` | 疑似ターミナルを割り当てる | 対話シェルとして使える |
| `-it` | 上の 2 つセット | **対話型セッション**にする定番 |
| `hello-pod` | 対象 Pod | 入る先 |
| `--` | 区切り | 以降は「Pod の中で実行するコマンド」 |
| `/bin/sh` | シェルを起動 | bash が無いイメージ（alpine 等）でも使えることが多い |

プロンプトが変わったら **コンテナの中**にいます。抜けるときは:

```bash
exit
# または Ctrl+D
```

#### 対話中にできること（例）

`nginx:alpine` の中では、Step 01 の Linux コマンドがだいたい使えます。

| やりたいこと | 例 | 目的 |
|--------------|-----|------|
| 今どこにいるか | `pwd` | 作業場所の確認 |
| ファイル一覧 | `ls -la` / `ls -la /usr/share/nginx/html` | 公開ファイルや設定の有無 |
| ファイルの中身 | `cat /usr/share/nginx/html/index.html` | nginx が返している HTML を見る |
| プロセス確認 | `ps aux` | nginx など何が動いているか |
| 自分は誰か | `whoami` / `id` | コンテナ内のユーザー・UID |
| 疎通 | `wget -qO- http://127.0.0.1/` や `curl`（入っていれば） | **コンテナの内側から**アプリ応答を確認 |
| 設定を覗く | `ls /etc/nginx` / `cat /etc/nginx/nginx.conf` | 設定ファイルの場所を知る |
| 環境変数 | `env` | 渡されている設定値の確認（ConfigMap 演習でも使う） |

対話に入らず、**1 コマンドだけ中で実行**することもできます（現場でよく使う）:

```bash
kubectl exec hello-pod -- ls -la /usr/share/nginx/html
kubectl exec hello-pod -- cat /usr/share/nginx/html/index.html
kubectl exec hello-pod -- ps aux
```

| 形 | 用途 |
|----|------|
| `kubectl exec -it POD -- /bin/sh` | 中を manifest に歩き回って調べる（対話） |
| `kubectl exec POD -- コマンド` | 確認したいことだけ実行してすぐ終わる |

#### できないこと・注意

| 注意 | 理由 |
|------|------|
| 本番で常用しない | 中を直接いじると「宣言（YAML）と実体」がずれる |
| 永続しない変更が多い | コンテナ再作成で中の手動変更は消えることが多い |
| シェルが無いイメージもある | その場合は `exec` できない。別のデバッグ用イメージを使う |
| 権限で拒否されることがある | RBAC / SCC（OpenShift）で制限されている |

覚える一言: **対話型 `exec` は「中を見て確かめる」ための道具。恒久対応はマニフェスト側で行う。**

片付け:

```bash
kubectl delete -f ../manifests/01-pod.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl delete -f ...` | YAML で定義したリソースを削除 | 学習用リソースを残さない |

## 期待結果

```text
NAME        READY   STATUS    RESTARTS   AGE
hello-pod   1/1     Running   0          ...
```

- `describe` の Events に失敗が並んでいない
- `logs` が取れる（nginx はアクセスが無いと静かでもよい）

## ポイント

- 本番で Pod を直接作り続けることは少ない（Deployment 経由が基本）
- まずは「1 個動いている」を掴む

## トラブル時

| 症状 | 対処 |
|------|------|
| `ImagePullBackOff` | ネット確認。`describe` の Events を読む |
| `Pending` | `minikube status`。ノード Ready か |

## 完了条件（DoD）

- [ ] `Running` / `1/1` を確認した
- [ ] `get` / `describe` / `logs` それぞれの目的を説明できる
- [ ] （任意）`kubectl exec -it` で中に入り、`ls` / `ps` などが使えること・本番常用しない理由を説明できる
