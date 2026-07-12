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

名前解決ができていないと、「Ingress リソースはあるのにブラウザで開けない」が起きます。学習では次のどちらかが多いです。

- `/etc/hosts` に `（Ingress の ADDRESS） hello.local` を書く  
- または `curl` で Host を明示する（IP 直打ち + ヘッダー）:

```bash
# ADDRESS は kubectl get ingress の ADDRESS 列（環境差あり）
curl -H "Host: hello.local" http://<INGRESSのADDRESS>/
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

```bash
minikube addons enable ingress

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml
kubectl apply -f ../manifests/05-ingress.yaml

kubectl get ingress
# HOSTS 列に hello.local が出る＝「この仮ホスト名用のルールがある」
# ADDRESS 列＝パケットの届け先 IP（環境によって空のまま／時間がかかることも）
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube addons enable ingress` | Ingress Controller を有効化 | Ingress オブジェクトを処理する部品を入れる（Minikube 専用） |
| `kubectl apply -f .../05-ingress.yaml` | Ingress ルール作成 | `hello.local` → `hello-svc` の対応を宣言 |
| `kubectl get ingress` | Ingress 一覧 | `HOSTS=hello.local` と ADDRESS を確認 |

`hello.local` でブラウザ到達までやる場合（任意・環境差あり）:

1. `kubectl get ingress` で ADDRESS（IP）を控える  
2. Mac の `/etc/hosts` に例えば `192.168.x.x  hello.local` を追加（IP は自分の ADDRESS）  
3. ブラウザで `http://hello.local/` を開く  

詰まったら「Ingress オブジェクトはあるか」と「名前解決（hosts）ができているか」を分けて疑う。公式 Minikube Ingress 手順も参照。

片付け:

```bash
kubectl delete -f ../manifests/05-ingress.yaml
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- `kubectl get ingress` にリソースが出る（HOSTS に `hello.local`）
- （環境が許せば）`http://hello.local/` または `curl -H "Host: hello.local" ...` で到達できる

## 完了条件（DoD）

- [ ] `hello.local` は仮のホスト名で、Ingress の振り分け条件だと説明できる
- [ ] ホスト名（Host）と IP（ADDRESS）の違いを言える
- [ ] Ingress と Service / Deployment の関係を説明できる
- [ ] 「OpenShift では Route」と覚えている
