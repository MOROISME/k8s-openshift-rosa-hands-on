# 05. OpenShift の基本（Developer Sandbox・無料）

**この Step の目的:** Kubernetes との差分（Project / Route / SCC / Operator）を、無料 Sandbox で体験する。  
有料の OpenShift / ROSA クラスタは使いません。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## 前提（Developer Sandbox）

有料クラスタは作らない。[docs/free-tier.md](../docs/free-tier.md)

### 1. Console を開く

1. [https://developers.redhat.com/developer-sandbox](https://developers.redhat.com/developer-sandbox) を開く
2. Red Hat アカウントでログイン（無ければ無料登録。SMS 確認が出ることがある）
3. **Try Red Hat products** のカード一覧で、**OpenShift** の **Try it** を押す
  （AI / Ansible など他カードは使わない）
4. **DevSandbox** / 同意が出たら進み、**Web Console** を開く

成功の目安: URL が `console-openshift-console.apps....`、Project（例: `xxxxx-dev`）が **Active**。


| 補足               |                               |
| ---------------- | ----------------------------- |
| 右上の端末アイコン        | Console 内蔵の Web Terminal（任意）  |
| この教材の YAML apply | **Mac のターミナル + `oc`** を推奨（次へ） |


### 2. Mac に `oc` を入れる

Console 右上 `**?` → Command Line Tools** から macOS 用を取得。  
`uname -m` が `arm64` → Apple Silicon、`x86_64` → Intel。

展開して出てきた `oc` を使う（ダブルクリックで開くアプリではない）。

```bash
# Gatekeeper で止まると出る警告は「ゴミ箱に入れる」せず完了で閉じる
xattr -d com.apple.quarantine ~/Downloads/oc   # パスは実体に合わせる
chmod +x ~/Downloads/oc

mkdir -p ~/bin
mv ~/Downloads/oc ~/bin/oc

# 未設定なら 1 回だけ（重複追記しない）
grep -q 'HOME/bin' ~/.zshrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

which oc          # → .../bin/oc
oc version --client
```


| 用語                       | 短く言うと                            |
| ------------------------ | -------------------------------- |
| PATH                     | `oc` と打ったとき探すフォルダ一覧。`~/bin` を載せる |
| `xattr -d ...quarantine` | ダウンロード隔離マークを外す（Gatekeeper 対策）    |


別案: `brew install openshift-cli`

### 3. Mac で `oc login`

1. Console（Project が見える画面）右上のユーザー名 → **Copy login command**
  （または `?` → Command Line Tools → Copy login command）
2. **Display Token** → `oc login --token=... --server=...` をコピー
3. **Mac のターミナル**に貼り付けて実行

```bash
cd 05-openshift
oc whoami
oc project
```

トークンは秘密情報。Web Terminal と Mac は別セッション。`kubectl`（Minikube）と混同しない。

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


| フィールド                    | 意味                      | 目的                          |
| ------------------------ | ----------------------- | --------------------------- |
| `nginx-unprivileged`     | root 以外で 8080 を聞く nginx | OpenShift の制限付き SCC でも動きやすい |
| `containerPort: 8080`    | 特権ポート（80）を避ける           | 非特権ユーザーでも listen できる        |
| `runAsNonRoot`           | root で動かさない             | SCC / セキュリティ要件に合わせる         |
| `capabilities.drop: ALL` | Linux capability を捨てる   | 余計な権限を持たせない                 |
| Probe / resources        | 健康診断と枠                  | Minikube 演習と同じ型             |


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


| フィールド                 | 意味         | 目的                                     |
| --------------------- | ---------- | -------------------------------------- |
| `selector.app`        | Pod のラベル条件 | Deployment が付ける `app: sandbox-web` と一致 |
| `port` / `targetPort` | 窓口 → コンテナ  | どちらも 8080（非特権）                         |
| `ports[].name: http`  | ポート名       | Route から `targetPort: http` で参照する      |


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


| フィールド                   | 意味               | 目的                            |
| ----------------------- | ---------------- | ----------------------------- |
| `kind: Route`           | OpenShift の外部入り口 | Ingress に近い役割                 |
| `to.name`               | 届け先 Service      | `sandbox-web` に転送             |
| `port.targetPort: http` | Service のポート名    | 上の Service の `name: http` と対応 |
| `tls.termination: edge` | 入口で TLS 終端       | HTTPS で受けて中は HTTP             |
| `Redirect`              | HTTP → HTTPS     | 平文アクセスをリダイレクト                 |


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


| コマンド                        | 意味               | 目的                         |
| --------------------------- | ---------------- | -------------------------- |
| `oc project`                | 現在 Project 表示/切替 | 間違った Project にデプロイしない      |
| `oc apply -f ...deployment` | アプリ本体            | Pod を Deployment で管理       |
| `oc apply -f ...service`    | クラスタ内入口          | Pod への安定アクセス               |
| `oc apply -f ...route`      | 外部 URL           | OpenShift 流の公開（Ingress 相当） |
| `oc get pods/svc/route`     | 各リソース確認          | Running と HOST を見る         |


```bash
curl -I https://<route-host>
```


| コマンド                  | 目的                |
| --------------------- | ----------------- |
| `curl -I https://...` | Route 経由で外から届くか確認 |

> **ここではまだ削除しない。** このあと D で `logs` / `describe` に使う。  
> 片付けは Step 末尾（F のあと）でまとめて行う。

### 期待結果

- Pod `1/1 Running`
- Route に HOST が出る
- curl / ブラウザで 200 相当（HTTP 直叩きは 302 → HTTPS も正しい）

## D. `oc` 基本（観察セット）

```bash
oc get project
oc get pods
oc get svc
oc get route
oc logs deploy/sandbox-web
oc describe route/sandbox-web
```


| コマンド                    | 目的              |
| ----------------------- | --------------- |
| `oc get project`        | 触れる Project 一覧  |
| `oc logs deploy/...`    | アプリログ（K8s と同じ型） |
| `oc describe route/...` | Route の詳細・イベント  |


## E. SCC / RBAC（観察）

**目的:** 権限不足の切り分け入口を知る（Sandbox では変更できないことが多い）。

```bash
oc auth can-i create deployment
oc auth can-i get scc --all-namespaces
oc get scc 2>/dev/null || echo "SCC list not permitted (expected on Sandbox)"
```


| コマンド                      | 目的                                   |
| ------------------------- | ------------------------------------ |
| `oc auth can-i ...`       | 自分にその API 操作が許されるか                   |
| `oc get scc`              | SCC 一覧（権限があれば）。拒否されても「制限がある」と分かれば OK |
| `2>/dev/null || echo ...` | エラーを握りつぶして学習用メッセージ                   |


- RBAC = API を叩けるか  
- SCC = Pod がホストに対してどこまでできるか

## F. Operator（観察）

```bash
# まず素で実行（権限エラーが出ることが多い）
oc get csv -A

# エラーを隠す書き方（何も出なくても異常ではない）
oc get csv -A 2>/dev/null || true
```

| 結果 | 意味 |
|------|------|
| 一覧が出る | Operator（CSV）が見えている |
| `Forbidden` など | Sandbox ではよくある。この節は概念理解で OK |
| **何も表示されない** | 権限エラーが `2>/dev/null` で消えただけ。失敗扱いにしない |

| コマンド部品 | 意味 |
|--------------|------|
| `csv` | ClusterServiceVersion＝Operator の導入記録 |
| `-A` | 全 Namespace |
| `2>/dev/null` | エラー出力を捨てる |
| `\|\| true` | 失敗してもシェルを落とさない |

**Operator = 運用手順のソフトウェア化。**  
Sandbox では取れない／空に見えるのが普通です。

## 片付け（D〜F が終わってから）

C でデプロイしたアプリが残っている前提で D をやったあと、最後に消す。

```bash
cd 05-openshift

oc delete -f manifests/app-route.yaml
oc delete -f manifests/app-service.yaml
oc delete -f manifests/app-deployment.yaml
```

| コマンド | 目的 |
|----------|------|
| `oc delete -f ...` | 学習用リソースを残さない（Route → Service → Deployment の順でも可） |

## トラブル時


| 症状          | 対処                            |
| ----------- | ----------------------------- |
| 期限切れ        | 再申請。その間は Minikube             |
| 権限系で Pod 失敗 | このリポジトリの非特権 YAML を使う          |
| Route 無し    | apply 漏れ・Service 名不一致         |
| login 失敗    | Console から login command 再コピー |


## 完了条件（DoD）

- [ ] アプリを 1 つ動かして Route で到達した
- [ ] Deployment / Service / Route の YAML の役割を説明できる
- [ ] 主要 `oc` コマンドの目的を説明できる
- [ ] 有料クラスタを作っていない

次: [06-aws](../06-aws/)