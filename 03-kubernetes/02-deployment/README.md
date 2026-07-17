# 02. Deployment

Deployment は「何台動かすか」「更新どうするか」を管理します。  
**この節の目的:** Pod を直接ではなく Deployment 経由で扱い、スケールを体験する。

## YAML設定の解説（`../manifests/02-deployment.yaml`）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: web
          image: nginx:alpine
          ports:
            - containerPort: 80
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `kind: Deployment` | Deployment を作る | Pod を直接ではなく管理者経由で動かす |
| `replicas: 2` | 複製の台数 | 「同じ Pod を 2 つ」と宣言 |
| `selector.matchLabels` | Deployment が管理する Pod の探す条件 | `app=hello` の Pod を自分の管轄にする |
| `template.metadata.labels` | 作られる Pod に付く付箋 | **探す条件と同じ付箋が必要**（ここが実体の印） |
| `template.spec.containers` | Pod の中身の設計図 | 実際に起動するコンテナ定義 |

関係のイメージ:

```text
Deployment (hello-deploy)
  replicas: 2
  └── template（設計図）→ Pod × 2（どちらも labels: app=hello）
```

## 実習

```bash
kubectl apply -f ../manifests/02-deployment.yaml

kubectl get deploy
kubectl get pods -l app=hello

kubectl scale deploy hello-deploy --replicas=3
kubectl get pods -l app=hello
kubectl rollout status deploy/hello-deploy
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | Deployment を作成/更新 | 「望ましい状態」を宣言する |
| `kubectl get deploy` | Deployment 一覧 | READY（何台 Ready か）を見る |
| `kubectl get pods -l app=hello` | 付箋で Pod を絞る | Deployment が作った Pod だけ見る。`-l` は label selector |
| `kubectl scale ... --replicas=3` | 希望台数を 3 に変更 | スケールアウトを体験 |
| `kubectl rollout status ...` | 更新/スケールの完了待ち | 「終わるまで待つ」運用の型 |

片付けは Service 演習のあとでも可。先に消す場合:

```bash
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

```text
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   2/2     2            2           ...

# scale 後
hello-deploy   3/3     3            3           ...
```

## ポイント

- 普段触るのは Pod ではなく Deployment
- `replicas` = 「何個ほしいか」の宣言。K8s が実際の Pod 数を合わせる

## 完了条件（DoD）

- [ ] Deployment から複数 Pod が作られることを確認した
- [ ] `scale` と `-l` の目的を説明できる
