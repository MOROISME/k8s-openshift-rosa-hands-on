# 04. Namespace / ConfigMap / Secret

**この節の目的:** 論理仕切りと「設定をイメージに焼き込まない」やり方を体験する。

## 概要

| リソース | 意味 | 目的 |
|----------|------|------|
| Namespace | クラスタ内の論理仕切り | チーム・環境（dev 等）の分離 |
| ConfigMap | 非機密寄りの設定 | 環境変数や設定ファイルとして Pod に渡す |
| Secret | 機密情報 | パスワード等（中身は Base64。暗号化とは限らない） |

## マニフェスト解説（`../manifests/04-namespace-config.yaml`）

1 ファイルに複数リソースが `---` で並んでいます（まとめて apply できる）。

### 1) Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: learn
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `kind: Namespace` | 仕切りを作る | 以降の資源を `learn` に置く箱 |

### 2) ConfigMap

```yaml
kind: ConfigMap
metadata:
  name: hello-config
  namespace: learn
data:
  APP_MESSAGE: "hello from ConfigMap"
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `namespace: learn` | 所属 Namespace | 仕切りの中に置く |
| `data.APP_MESSAGE` | 設定キーと値 | あとで環境変数として Pod に渡す |

### 3) Secret

```yaml
kind: Secret
metadata:
  name: hello-secret
  namespace: learn
type: Opaque
stringData:
  APP_PASSWORD: "changeme"
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `type: Opaque` | 汎用 Secret | 任意の鍵・値 |
| `stringData` | 平文で書ける欄 | apply 時に Base64 化される（暗号化ではない） |

### 4) Deployment（設定を受け取る側）

```yaml
kind: Deployment
metadata:
  name: hello-config
  namespace: learn
spec:
  # ...
  template:
    spec:
      containers:
        - name: demo
          image: busybox:1.36
          command: ["sh", "-c", "echo MESSAGE=$APP_MESSAGE; echo SECRET_SET=...; sleep 3600"]
          env:
            - name: APP_MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: hello-config
                  key: APP_MESSAGE
            - name: APP_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: hello-secret
                  key: APP_PASSWORD
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `command` | 起動時に実行するコマンド | 渡された値をログに出して確認する |
| `env[].valueFrom.configMapKeyRef` | ConfigMap から環境変数へ | イメージに設定を焼き込まない |
| `env[].valueFrom.secretKeyRef` | Secret から環境変数へ | パスワードを別管理する |

流れ:

```text
ConfigMap / Secret（learn NS）
        ↓ valueFrom で参照
Deployment → Pod の環境変数 APP_MESSAGE / APP_PASSWORD
        ↓ echo
kubectl logs に MESSAGE=... / SECRET_SET=yes
```

## ハンズオン

**作業ディレクトリ:** `03-kubernetes/04-namespace-config`

```bash
kubectl apply -f ../manifests/04-namespace-config.yaml

kubectl get ns learn
kubectl get configmap,secret -n learn
kubectl get pods -n learn

kubectl logs -n learn deploy/hello-config
kubectl describe pod -n learn -l app=hello-config
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | NS / CM / Secret / Deployment 等を一括適用 | 仕切り付きアプリを一度に作る |
| `kubectl get ns learn` | Namespace の存在確認 | 仕切りができたか見る |
| `kubectl get configmap,secret -n learn` | 指定 NS の設定類を一覧 | `-n` = Namespace 指定 |
| `kubectl get pods -n learn` | その NS の Pod | アプリが learn にいるか確認 |
| `kubectl logs -n learn deploy/...` | Deployment 配下 Pod のログ | Config が渡った結果をログで見る |
| `kubectl describe ... -n learn` | 詳細 | 環境変数の実態を確認 |

片付け:

```bash
kubectl delete -f ../manifests/04-namespace-config.yaml
```

## 期待結果

- `learn` Namespace がある
- Pod が `Running`
- logs に `MESSAGE=hello from ConfigMap` と `SECRET_SET=yes`

## 完了条件（DoD）

- [ ] `-n` の目的を説明できる
- [ ] YAML で ConfigMap / Secret が env に渡る流れを説明できる
