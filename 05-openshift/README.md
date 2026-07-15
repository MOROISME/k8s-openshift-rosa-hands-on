# 05. OpenShift の基本（Developer Sandbox・無料）

**この Step の目的:** Kubernetes との差分（Project / Route / SCC / Operator）を、無料 Sandbox で体験する。  
有料の OpenShift / ROSA クラスタは使いません。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## 前提（Developer Sandbox を使えるようにする）

ここは **ブラウザ操作が中心**です。ボタン文言はたまに変わりますが、流れは同じです。  
有料クラスタは作りません（[docs/free-tier.md](../docs/free-tier.md)）。

### 手順 1: Sandbox の案内ページを開く

ブラウザで次を開く:

https://developers.redhat.com/developer-sandbox

続けてログインすると、**Developer Sandbox**（Red Hat Developer Hub）のホームに入ることがある。  
見出しはだいたい次のどれか:

- `Developer Sandbox`
- `Try Red Hat products`

| 見ること | 意味 |
|----------|------|
| 製品カードが並ぶ画面（OpenShift / OpenShift AI など） | 無料トライアル用の入口（いまの公式 UI） |
| 右上に自分の名前 | ログイン済み |

目的: 「無料 Sandbox を始める公式の入口」にたどり着く。

### 手順 2: Red Hat アカウントでログイン（無料）

まだログイン前なら:

1. 案内ページの **Get started** / **Log in** 系を押す  
2. **Red Hat アカウント**でサインイン  
3. アカウントが無い場合は **Register** で無料作成（メール確認が必要なことがある）  
4. 初回は電話番号 SMS 確認を求められることがある → 案内どおり入力  

| ポイント | 意味 |
|----------|------|
| Red Hat アカウント | 開発者向け無料アカウント（この教材では有料契約は不要） |
| SMS 確認 | 不正利用防止。Sandbox 初回でよくある |

目的: 「誰として Sandbox を使うか」を確定する。

すでに右上に自分の名前が出ている画面なら、手順 2 は完了済み → 手順 3 へ。

### 手順 3: Sandbox を起動し Web Console を開く

**いまの UI（製品カードが並ぶ画面）での正しい操作:**

1. **OpenShift** のカードを探す（「Comprehensive cloud-native application platform」などと書いてある）  
2. そのカードの **Try it** を押す（この教材で使うのは OpenShift。AI / Ansible など他カードではない）  
3. 認証の選択で **DevSandbox** が出たらそれを選ぶ  
4. 利用規約に同意を求められたら同意する  
5. 準備中の画面が出たら待つ  
6. 準備ができたら **OpenShift Web Console** が開く（または **Console** / **Open console** へのリンクを押す）  

古い案内に「Launch your Developer Sandbox」とあっても、今は **OpenShift カードの Try it** がそれに相当します。

この教材で押す場所:

| 押してよい | 押さない（この Step では不要） |
|------------|--------------------------------|
| **OpenShift** の **Try it** | OpenShift AI / Dev Spaces / Ansible / Virtualization / OpenClaw など |

Console に入れたら成功の目安:

| Console で見るもの | 意味 |
|--------------------|------|
| URL に `console-openshift-console.apps....` | OpenShift の Web Console |
| Project（例: `moroisme-dev`）が **Active** | 作業する仕切りに入れている |
| 右上のユーザー名 | ログイン中の自分 |
| **Developer** / **Administrator** の切替 | 最初は Developer で十分 |

右上の **端末アイコン** を押すと、画面下に **OpenShift コマンドラインターミナル**（Web Terminal）が開くことがある。  
そこで `oc version --client` が動けば、Console 内でも `oc` は使える。

| やり方 | どこで打つか | この教材での扱い |
|--------|--------------|------------------|
| Web Terminal（右上の端末アイコン） | Console の下ペイン | 使ってよい |
| 自分の Mac のターミナル | 手元のシェル | 推奨。手順 4 の Copy login command が必要 |

**「Sandbox を起動し Web Console を開く」は、Project が見える Console まで来ていれば完了。**  
下の Web Terminal が開いているかは必須ではない。

目的: ブラウザだけでクラスタを触れる状態にする。

つまずき:

| 症状 | 対処 |
|------|------|
| 製品カードは見えるが Console が無い | **OpenShift** の **Try it** を押す |
| 期限切れ / 使えない | 再申請。その間は Minikube |
| ログイン画面がループする | 別タブ・シークレットウィンドウ、または再ログイン |
| Console が真っ白 | 再読み込み。ブラウザを変える |

### 手順 4:（推奨）手元の Mac で `oc` ログイン — Copy login command

このリポジトリの `manifests/` を apply するなら、**Mac のターミナル**の方が扱いやすい。  
そのためのログインが Copy login command。

> Web Terminal だけで GUI / 簡単な `oc` 確認するなら、手順 4 はスキップしてよい。  
> Web Terminal にログインしていても、**Mac 側は別セッション**なので別途 `oc login` が必要。

#### 4-1. `oc` が無い場合（Mac 用）

Console 右上の **`?`** → **Command Line Tools** から macOS 向け `oc` をダウンロードする。  
CPU が `arm64` なら **Apple Silicon / arm64**、`x86_64` なら **Intel / amd64** を選ぶ（確認: `uname -m`）。

多くの場合 `.tar.gz` / `.zip` で来るので、ダブルクリックまたはターミナルで展開し、中の **`oc` ファイル**を使う。

注意:

- `oc` は **GUI アプリではなくコマンド**。Finder でダブルクリックして開くものではない
- ダウンロード直後に開こうとすると、次の警告が出ることがある（正常）:

```text
“oc” は開いていません
Apple は、“oc” に ... マルウェアが含まれていないことを検証できませんでした。
```

**「ゴミ箱に入れる」は押さない。** 公式の CLI でも、未公証バイナリだと macOS が止めることがある。

許可する（どれか 1 つ）:

**A. システム設定（わかりやすい）**

1. ダイアログは **完了** で閉じる  
2. **システム設定** → **プライバシーとセキュリティ**  
3. 下の方に「"oc"は使用するためにブロックされました」などが出ていたら **このまま許可**  

**B. 右クリックから開く**

1. Finder で `oc` を **Control + クリック**（または右クリック）→ **開く**  
2. 再度警告が出たら **開く**  

**C. ターミナルで隔離属性を外す（よくやる）**

```bash
# 展開後の実パスに合わせる（例: Downloads 直下）
xattr -d com.apple.quarantine ~/Downloads/oc
chmod +x ~/Downloads/oc
```

##### PATH を通す（必須・詳細）

**PATH とは:** ターミナルが「コマンド名だけ」で探すフォルダの一覧。  
`oc` を PATH 上のどこかに置き、そのフォルダを一覧に入れないと、`oc: command not found` になる。

この教材の定番は **`~/bin/oc`** に置き、`~/bin` を PATH に追加するやり方。

**手順 P1: `oc` の実体を探す**

Finder の「ダウンロード」か、展開したフォルダの中に `oc` がある。ターミナルなら:

```bash
ls ~/Downloads/oc
# 無いときは展開先を探す例:
ls ~/Downloads/*/oc 2>/dev/null
find ~/Downloads -name oc -type f 2>/dev/null
```

見つかったパスを以降では `（ocの場所）` と呼ぶ。例: `/Users/あなた/Downloads/oc`

**手順 P2: 実行権限を付ける**

```bash
chmod +x "（ocの場所）"
# 例:
chmod +x ~/Downloads/oc
```

**手順 P3: `~/bin` に移す（名前は必ず `oc`）**

```bash
mkdir -p ~/bin
mv "（ocの場所）" ~/bin/oc
# 例:
# mv ~/Downloads/oc ~/bin/oc

chmod +x ~/bin/oc
ls -l ~/bin/oc
# -rwxr-xr-x ... oc のように x が付いていれば OK
```

すでに `~/bin/oc` がある場合は上書き確認される。差し替えてよい。

**手順 P4: `~/.zshrc` に PATH を追加する（永続化）**

macOS のデフォルトシェルは zsh。新しいターミナルでも効くように設定ファイルへ書く。

```bash
# すでに書いてあるか確認（何も出なければ未設定）
grep 'HOME/bin' ~/.zshrc || true

# 未設定なら 1 行追加
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
```

| やっていること | 意味 |
|----------------|------|
| `export PATH=...` | このシェルで使う検索パスを更新 |
| `$HOME/bin` を先頭に | `~/bin` 内の `oc` を優先して見つける |
| `>> ~/.zshrc` | 今後開くターミナルにも残す |

**手順 P5: 今のターミナルに即反映する**

```bash
source ~/.zshrc
```

または、ターミナルを一度閉じて開き直す。

**手順 P6: 通ったか確認する**

```bash
which oc
# 期待: /Users/あなた/bin/oc  （~/bin/oc）

oc version --client
# Client Version: ... が出れば OK
```

| 結果 | 意味 | 対処 |
|------|------|------|
| `which oc` が `~/bin/oc` | PATH 成功 | 次のログイン手順へ |
| `oc: command not found` | PATH 未反映 or 置き場所違い | P3〜P5 を見直す。`ls ~/bin/oc` があるか確認 |
| `permission denied` | 実行権限なし | `chmod +x ~/bin/oc` |
| 警告ダイアログだけ出る | Gatekeeper | 上の許可手順 A〜C |

つまずき（PATH）:

| 症状 | 原因 | 対処 |
|------|------|------|
| Downloads では動くが `oc` 単体はダメ | PATH に Downloads を入れていない | `~/bin` 方式にする（推奨） |
| 新しいターミナルだけダメ | `source` したけど `.zshrc` に書いてない | P4 をやり直す |
| `echo` を何回もやって PATH が重複 | 何度も追記した | `~/.zshrc` を開き、同じ行は 1 つ残す |

（Homebrew で入れる場合の例: `brew install openshift-cli`。入ったら `which oc` で場所を確認。その場合も「PATH に載っているか」の確認は同じ。）

#### 4-2. ログインコマンドをコピーする

場所: **Web Console**（製品カードのホームではない。Project が見える画面）

**A.** 右上のユーザー名 → **Copy login command**（または「ログインコマンドをコピー」）  
**B.** 右上の **`?`** → **Command Line Tools** → **Copy login command**

続き:

1. 新タブで **DevSandbox**（出た場合）  
2. **Display Token**  
3. `oc login --token=... --server=https://...` の行をすべてコピー  

#### 4-3. Mac のターミナルに貼り付けてログイン

```bash
# トークンと URL は自分の画面のものを使う
oc login --token=sha256~xxxxxxxx --server=https://api.xxxxx.openshiftapps.com:6443
```

#### 4-4. ログイン確認

```bash
cd 05-openshift

oc whoami
oc project
# 例: Using project "moroisme-dev" など
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc whoami` | 今のユーザー | ログイン確認 |
| `oc project` | 現在の Project | 正しい仕切りか確認 |

注意:

| こと | 意味 |
|------|------|
| トークンは秘密情報 | チャットや Git に貼らない |
| Web Terminal ≠ Mac | 片方にログインしても、もう片方は別途必要 |
| Minikube と混同しない | `kubectl`（ローカル）と Sandbox の `oc` は別物 |

## A. 概念チェック

- [ ] Project ≈ Namespace + 権限
- [ ] Route ≈ Ingress
- [ ] DeploymentConfig はレガシー寄り
- [ ] SCC が Pod 権限を制限
- [ ] Operator が運用機能を担う

## B. Web Console ハンズオン

**目的:** GUI でも同じオブジェクト（Pod / Route / ログ）を追えるようにする。

1. Project を確認  
2. カタログまたは YAML でデプロイ  
3. Route URL を開く  
4. Pod ログを見る  

## C. YAML ハンズオン

**目的:** 制限付き SCC でも動く非特権アプリを、Deployment → Service → Route で公開する。

### マニフェスト解説

#### 1) Deployment（`manifests/app-deployment.yaml`）

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: web
      # OpenShift 制限付き SCC 向け（特権ポートを使わない）
      image: nginxinc/nginx-unprivileged:1.25-alpine
      ports:
        - containerPort: 8080
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
        runAsNonRoot: true
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `nginx-unprivileged` | root 以外で 8080 を聞く nginx | OpenShift の制限付き SCC でも動きやすい |
| `containerPort: 8080` | 特権ポート（80）を避ける | 非特権ユーザーでも listen できる |
| `runAsNonRoot` | root で動かさない | SCC / セキュリティ要件に合わせる |
| `capabilities.drop: ALL` | Linux capability を捨てる | 余計な権限を持たせない |
| Probe / resources | 健康診断と枠 | Minikube 演習と同じ型 |

#### 2) Service（`manifests/app-service.yaml`）

```yaml
kind: Service
spec:
  selector:
    app: sandbox-web
  ports:
    - name: http
      port: 8080
      targetPort: 8080
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `selector.app` | Pod のラベル条件 | Deployment が付ける `app: sandbox-web` と一致 |
| `port` / `targetPort` | 窓口 → コンテナ | どちらも 8080（非特権） |
| `ports[].name: http` | ポート名 | Route から `targetPort: http` で参照する |

#### 3) Route（`manifests/app-route.yaml`）

```yaml
apiVersion: route.openshift.io/v1
kind: Route
spec:
  to:
    kind: Service
    name: sandbox-web
  port:
    targetPort: http
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `kind: Route` | OpenShift の外部入り口 | Ingress に近い役割 |
| `to.name` | 届け先 Service | `sandbox-web` に転送 |
| `port.targetPort: http` | Service のポート名 | 上の Service の `name: http` と対応 |
| `tls.termination: edge` | 入口で TLS 終端 | HTTPS で受けて中は HTTP |
| `Redirect` | HTTP → HTTPS | 平文アクセスをリダイレクト |

流れ:

```text
ブラウザ / curl
  → Route（HTTPS）
    → Service (sandbox-web:8080)
      → Pod（nginx-unprivileged）
```

### 適用手順

```bash
oc project

oc apply -f manifests/app-deployment.yaml
oc apply -f manifests/app-service.yaml
oc apply -f manifests/app-route.yaml

oc get pods
oc get svc
oc get route
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc project` | 現在 Project 表示/切替 | 間違った Project にデプロイしない |
| `oc apply -f ...deployment` | アプリ本体 | Pod を Deployment で管理 |
| `oc apply -f ...service` | クラスタ内入口 | Pod への安定アクセス |
| `oc apply -f ...route` | 外部 URL | OpenShift 流の公開（Ingress 相当） |
| `oc get pods/svc/route` | 各リソース確認 | Running と HOST を見る |

```bash
curl -I https://<route-host>
```

| コマンド | 目的 |
|----------|------|
| `curl -I https://...` | Route 経由で外から届くか確認 |

片付け:

```bash
oc delete -f manifests/app-route.yaml
oc delete -f manifests/app-service.yaml
oc delete -f manifests/app-deployment.yaml
```

| コマンド | 目的 |
|----------|------|
| `oc delete -f ...` | 依存の逆順でも可。学習用を残さない |

### 期待結果

- Pod `1/1 Running`
- Route に HOST が出る
- curl / ブラウザで 200 相当

## D. `oc` 基本（観察セット）

```bash
oc get project
oc get pods
oc get svc
oc get route
oc logs deploy/sandbox-web
oc describe route/sandbox-web
```

| コマンド | 目的 |
|----------|------|
| `oc get project` | 触れる Project 一覧 |
| `oc logs deploy/...` | アプリログ（K8s と同じ型） |
| `oc describe route/...` | Route の詳細・イベント |

## E. SCC / RBAC（観察）

**目的:** 権限不足の切り分け入口を知る（Sandbox では変更できないことが多い）。

```bash
oc auth can-i create deployment
oc auth can-i get scc --all-namespaces
oc get scc 2>/dev/null || echo "SCC list not permitted (expected on Sandbox)"
```

| コマンド | 目的 |
|----------|------|
| `oc auth can-i ...` | 自分にその API 操作が許されるか |
| `oc get scc` | SCC 一覧（権限があれば）。拒否されても「制限がある」と分かれば OK |
| `2>/dev/null \|\| echo ...` | エラーを握りつぶして学習用メッセージ | 権限不足を失敗扱いにしない |

- RBAC = API を叩けるか  
- SCC = Pod がホストに対してどこまでできるか  

## F. Operator（観察）

```bash
oc get csv -A 2>/dev/null || true
```

| コマンド | 目的 |
|----------|------|
| `oc get csv -A` | ClusterServiceVersion＝Operator の導入状態を見る（環境差あり） |
| `\|\| true` | 失敗してもシェルを落とさない | Sandbox 制限への備え |

**Operator = 運用手順のソフトウェア化。**

## トラブル時

| 症状 | 対処 |
|------|------|
| 期限切れ | 再申請。その間は Minikube |
| 権限系で Pod 失敗 | このリポジトリの非特権 YAML を使う |
| Route 無し | apply 漏れ・Service 名不一致 |
| login 失敗 | Console から login command 再コピー |

## 完了条件（DoD）

- [ ] アプリを 1 つ動かして Route で到達した
- [ ] Deployment / Service / Route の YAML の役割を説明できる
- [ ] 主要 `oc` コマンドの目的を説明できる
- [ ] 有料クラスタを作っていない

次: [06-aws](../06-aws/)
