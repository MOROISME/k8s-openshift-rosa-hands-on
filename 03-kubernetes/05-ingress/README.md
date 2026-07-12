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

流れ:

```text
外からの HTTP（host: hello.local）
  → Ingress
    → Service (hello-svc)
      → Pod（app=hello）
```

前提として、先に Deployment と Service（`02` / `03`）があること。Ingress だけではアプリは動きません。

## ハンズオン

```bash
minikube addons enable ingress

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml
kubectl apply -f ../manifests/05-ingress.yaml

kubectl get ingress
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `minikube addons enable ingress` | Ingress Controller を有効化 | Ingress オブジェクトを処理する部品を入れる（Minikube 専用） |
| `kubectl apply -f .../05-ingress.yaml` | Ingress ルール作成 | ホスト/パス → Service の対応を宣言 |
| `kubectl get ingress` | Ingress 一覧 | リソースができたか・ADDRESS 等を確認 |

hosts 解決は環境差あり。詰まったら公式 Minikube Ingress 手順へ。

片付け:

```bash
kubectl delete -f ../manifests/05-ingress.yaml
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- `kubectl get ingress` にリソースが出る
- （環境が許せば）Ingress 経由で到達できる

## 完了条件（DoD）

- [ ] Ingress と Service / Deployment の関係を説明できる
- [ ] YAML の `host` / `backend.service` の意味を説明できる
- [ ] 「OpenShift では Route」と覚えている
