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

| 見ること | 意味 |
|----------|------|
| Developer Sandbox の説明ページ | 無料の学習用 OpenShift 環境の入口 |
| **Get started** / **Launch** 系のボタン | 申込・起動フローへ進む |

目的: 「無料 Sandbox を始める公式の入口」にたどり着く。

### 手順 2: Red Hat アカウントでログイン（無料）

1. **Get started in the Sandbox**（または同趣旨のボタン）を押す  
2. ログイン画面が出たら **Red Hat アカウント**でサインイン  
3. アカウントが無い場合は **Register** で無料作成（メール確認が必要なことがある）  
4. 初回は電話番号 SMS 確認を求められることがある → 案内どおり入力  

| ポイント | 意味 |
|----------|------|
| Red Hat アカウント | 開発者向け無料アカウント（この教材では有料契約は不要） |
| SMS 確認 | 不正利用防止。Sandbox 初回でよくある |

目的: 「誰として Sandbox を使うか」を確定する。

### 手順 3: Sandbox を起動し Web Console を開く

ログイン後の流れ（文言は前後することがある）:

1. **Launch your Developer Sandbox** / **Start using your sandbox** などを押す  
2. 認証の選択で **DevSandbox** が出たらそれを選ぶ  
3. 利用規約に同意を求められたら同意する  
4. しばらく待つ（準備中の画面が出ることがある）  
5. 準備ができたら **OpenShift Web Console** が開く（または **Console** へのリンクを押す）  

Console に入れたら成功の目安:

| Console で見るもの | 意味 |
|--------------------|------|
| 左上付近のロゴ / プロジェクト名 | OpenShift の管理画面に入れている |
| 右上のユーザー名 | ログイン中の自分 |
| **Developer** / **Administrator** の切替 | 画面の見え方が違う（最初は Developer で十分） |

目的: ブラウザだけでクラスタを触れる状態にする（ここまでで GUI ハンズオンは可能）。

つまずき:

| 症状 | 対処 |
|------|------|
| 期限切れ / 使えない | 同じページから再申請。その間は Minikube に戻る |
| ログイン画面がループする | 別タブ・シークレットウィンドウ、または一度ログアウトして再試行 |
| Console が真っ白 | しばらく待って再読み込み。ブラウザを変える |

### 手順 4:（推奨）`oc` でログインする — Copy login command

YAML ハンズオン（後の節）ではターミナルの `oc` が楽です。Console からログイン用コマンドをコピーします。

#### 4-1. `oc` が無い場合（インストール）

Console 右上の **`?`（ヘルプ）** → **Command Line Tools** を開く。  
macOS 向けの `oc` ダウンロードリンクがあるので、自分の環境に合わせて入れて PATH を通す。

確認:

```bash
oc version --client
# Client Version: ... が出れば OK
```

（Homebrew 等で入れても可。公式の Command Line Tools ページからでも可。）

#### 4-2. ログインコマンドをコピーする

やり方は Console の版で次のどちらか（両方探してよい）:

**A. よくある新しい UI**

1. Console **右上の自分のユーザー名** をクリック  
2. **Copy login command** を選ぶ  

**B. ヘルプ経由**

1. 右上の **`?`** → **Command Line Tools**  
2. **Copy login command** リンクをクリック  

どちらの場合も続きは同じです:

3. 新しいタブが開く → 認証で **DevSandbox** を選ぶ（出た場合）  
4. **Display Token** をクリック（トークンは最初マスクされていることが多い）  
5. **`oc login --token=... --server=https://...`** の行をすべてコピー  

#### 4-3. ターミナルに貼り付けてログイン

```bash
# 例（トークンと URL は自分の画面のものを使う。ここに書かれた例のままだと失敗する）
oc login --token=sha256~xxxxxxxx --server=https://api.xxxxx.openshiftapps.com:6443
```

成功するとだいたい次のような表示になります:

```text
Logged into "https://api....:6443" as "...." using the token provided.
...
Using project "...."
```

#### 4-4. ログイン確認

リポジトリのこの Step のディレクトリで:

```bash
cd 05-openshift

oc whoami
oc project
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc whoami` | 今のユーザー名を表示 | ログインできているか確認（`kubectl` に相当する OpenShift クライアントが `oc`） |
| `oc project` | 現在の Project | どの仕切りで作業するか確認（だいたい Namespace + 権限の寄せ集め） |

目的: 以降の `oc apply -f manifests/...` が「正しいクラスタ・正しい Project」に届くようにする。

注意:

| こと | 意味 |
|------|------|
| トークンは秘密情報 | チャットや Git に貼らない。期限切れしたら Copy login command をやり直す |
| `oc` 未インストールでも | Console だけでも一部はできるが、この教材の YAML 手順は `oc` 推奨 |
| Minikube と混同しない | `kubectl` はローカル Minikube、`oc`（このログイン後）はクラウド上の Sandbox |

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
