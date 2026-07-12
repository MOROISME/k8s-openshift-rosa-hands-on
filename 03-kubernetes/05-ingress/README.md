# 05. Ingress（任意）

Ingress はクラスタ外からの HTTP(S) 入り口です。OpenShift では **Route** が多いです。  
**この節の目的:** 「Service の前に HTTP ルーティング層がある」ことを知る（環境差あり・任意）。

飛ばして Step 4 に進んでも L4 は達成可能です。

## マニフェスト解説（`../manifests/05-ingress.yaml`）

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: hello.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-svc
                port:
                  number: 80
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `kind: Ingress` | HTTP 入り口ルール | ホスト/パス → Service の対応を宣言 |
| `annotations` | Controller 向けヒント | Minikube の nginx Ingress 用の書き換え例 |
| `rules[].host` | 受け付けるホスト名 | `hello.local` で来たリクエストをこのルールに載せる |
| `path` / `pathType` | URL パスの条件 | `/` 配下を対象にする |
| `backend.service` | 届け先 Service | 奥の窓口は `hello-svc:80`（Deployment ではない） |

### `hello.local` とは何か（ここが本題）

`hello.local` は **インターネット上の本物のサイト名ではありません**。  
この教材が Ingress の練習用に決めた **仮のホスト名（ニックネーム）** です。

ブラウザや curl でサイトを開くとき、URL はだいたいこうなります:

```text
http://hello.local/
        └─────┬─────┘
           ホスト名
```

HTTP では、このホスト名がリクエストの **`Host` ヘッダー** としてサーバーに伝わります。  
Ingress の `host: hello.local` は、次の意味です:

> 「`Host: hello.local` で来た HTTP だけ、このルールで `hello-svc` に送れ」

たとえ話:

| 現実のホテル | Ingress |
|--------------|---------|
| フロントの受付 | Ingress Controller |
| 「○○ホテル宛て」の荷物ラベル | `Host` ヘッダー（例: `hello.local`） |
| 部屋番号の対応表 | Ingress の `rules` |
| 実際の部屋 | Service → Pod |

同じ入り口（Ingress）に複数アプリを載せたいとき、パス（`/app-a`）だけでなく **ホスト名で振り分ける**のがよくあるやり方です。  
本番では `shop.example.com` のような本物の DNS 名を書きます。学習では公開 DNS を買わなくてよいので、`.local` っぽい仮名 `hello.local` を使っています。

重要な区別:

| もの | 役割 |
|------|------|
| `hello.local` | 「どのルールに乗せるか」の名前（YAML に書いた条件） |
| Minikube / Ingress の IP | 「パケットをどこに届けるか」の住所 |
| `/etc/hosts` や DNS | 名前 → IP の対応表（PC 側） |

つまり:

1. YAML に `host: hello.local` と書いて **振り分け条件** を宣言する  
2. 自分の Mac が `hello.local` を **どの IP に向けるか** は別問題（名前解決）  
3. 名前解決ができて初めて、ブラウザで `http://hello.local/` が Ingress に届く  

名前解決ができていないと、「Ingress リソースはあるのにブラウザで開けない」が起きます。  
さらに **macOS + Minikube（Docker ドライバ）** では、名前解決だけでは足りません（次節）。

学習で Host を付ける例:

```bash
# tunnel 後はだいたい 127.0.0.1（後述）。ADDRESS 列の 192.168.x.x を直接叩いても Mac からは届かないことが多い
curl -H "Host: hello.local" http://127.0.0.1/
```

`Host: hello.local` を付けないと、Ingress は「このルール用の荷物ではない」と判断し、届かないことがあります。

流れ:

```text
あなたが開く URL:  http://hello.local/
        │
        │  Host ヘッダー = hello.local
        ▼
Ingress（rules.host: hello.local に一致）
        ▼
Service (hello-svc)
        ▼
Pod（app=hello）
```

前提として、先に Deployment と Service（`02` / `03`）があること。Ingress だけではアプリは動きません。

## ハンズオン

**この節の必須ゴールは「Ingress オブジェクトがあること」です。**  
`http://hello.local/` をブラウザで開くのは **任意**（macOS Docker ドライバでは一手間必要）。

```bash
minikube addons enable ingress
# 表示に "please run minikube tunnel" と出ることがある → 後述。無視せず覚える

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml
kubectl apply -f ../manifests/05-ingress.yaml

kubectl get ingress
# HOSTS 列に hello.local ＝ 振り分けルールはある
# ADDRESS が 192.168.49.2 などでも、Mac からその IP 直打ちはだいたい届かない（Docker 内の住所）
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube addons enable ingress` | Ingress Controller を有効化 | Ingress オブジェクトを処理する部品を入れる（Minikube 専用） |
| `kubectl apply -f .../05-ingress.yaml` | Ingress ルール作成 | `hello.local` → `hello-svc` の対応を宣言 |
| `kubectl get ingress` | Ingress 一覧 | `HOSTS=hello.local` と ADDRESS を確認 |

### なぜ `http://hello.local/` が開けないか（よくある・想定内）

ブラウザで何もせず `http://hello.local/` を開いても、だいたい失敗します。理由は次の **どちらか（両方のことも多い）** です。

| 原因 | 意味 | よくある症状 |
|------|------|--------------|
| 名前解決がない | Mac が `hello.local` → どの IP？ を知らない | サイトに接続できない / DNS エラー |
| 届く道がない | ADDRESS（例: `192.168.49.2`）は Docker の中の IP。Mac から直接届かない | hosts を書いてもタイムアウト |

Service 演習のときと同じく、macOS + Docker ドライバでは **トンネル（橋渡し）** が必要です。  
addon 有効化時のメッセージどおり:

> run `minikube tunnel` … available at `127.0.0.1`

### （任意）ブラウザ / curl で届ける手順

**ターミナル A**（開いたまま。sudo パスワードを聞かれることがある）:

```bash
minikube tunnel
```

**ターミナル B:**

```bash
# 名前解決（1 回書けばよい。管理者権限が必要）
# 127.0.0.1 は tunnel が Mac 側に出す入口
echo '127.0.0.1 hello.local' | sudo tee -a /etc/hosts

# 確認（Host ヘッダー付き）
curl -I -H "Host: hello.local" http://127.0.0.1/

# hosts を書いたあとはブラウザでも可
# http://hello.local/
```

| ステップ | 意味 | 目的 |
|----------|------|------|
| `minikube tunnel` | Mac ↔ クラスタ内 Ingress の橋 | `127.0.0.1:80` で Ingress に届くようにする |
| `/etc/hosts` に `127.0.0.1 hello.local` | 名前 → 127.0.0.1 | ブラウザが `hello.local` を tunnel 入口に向ける |
| `curl -H "Host: hello.local"` | Host 条件を明示 | Ingress の `host: hello.local` ルールに載せる |

トンネルを止めると再び開けなくなります（Service の `minikube service --url` と同じ考え方）。

**この節をパスしてよい条件:** `kubectl get ingress` で `HOSTS=hello.local` が見え、「Host で振り分ける層がある」と説明できれば OK。ブラウザ到達は必須ではありません。

片付け:

```bash
# tunnel を動かしているターミナルがあれば Ctrl+C で止める

kubectl delete -f ../manifests/05-ingress.yaml
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- **必須:** `kubectl get ingress` にリソースが出る（HOSTS に `hello.local`）
- **任意:** `minikube tunnel` + hosts（または curl の Host ヘッダー）で nginx の応答が返る

## 完了条件（DoD）

- [ ] `hello.local` は仮のホスト名で、Ingress の振り分け条件だと説明できる
- [ ] ホスト名（Host）と IP（ADDRESS）の違いを言える
- [ ] macOS でブラウザが開けない主因（名前解決 / tunnel）をざっくり言える
- [ ] Ingress と Service / Deployment の関係を説明できる
- [ ] 「OpenShift では Route」と覚えている
