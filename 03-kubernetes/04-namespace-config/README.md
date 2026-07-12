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
| `command` | 起動時に実行するコマンド | 渡された値を **echo（印刷）** して確認する |
| `env[].valueFrom.configMapKeyRef` | ConfigMap から環境変数へ | イメージに設定を焼き込まない |
| `env[].valueFrom.secretKeyRef` | Secret から環境変数へ | パスワードを別管理する |

### 「ログで見る」とは何か（ここが本題）

ConfigMap は「設定の保管庫」です。渡ったかどうかは、保管庫を見るだけでは不十分で、**Pod の中に環境変数として入ったか**を確認します。

この演習のコンテナは、起動すると次のようなシェルを実行します（YAML の `command`）:

```sh
echo MESSAGE=$APP_MESSAGE
echo SECRET_SET=$( [ -n "$APP_PASSWORD" ] && echo yes || echo no )
sleep 3600
```

| 行 | やっていること | なぜそうするか |
|----|----------------|----------------|
| `echo MESSAGE=$APP_MESSAGE` | 環境変数 `APP_MESSAGE` の中身を印刷 | ConfigMap から渡った文字列が本当に入っているか見る |
| `echo SECRET_SET=yes/no` | パスワードが **空でないか** だけ印刷 | パスワード本文はログに出さない（漏洩防止の習慣） |
| `sleep 3600` | 1 時間待つ | すぐ終了すると Pod が落ちるので居座る |

`echo` の出力はコンテナの **標準出力** に出ます。  
Kubernetes ではそれを **`kubectl logs` で読む**のが定番です（＝「ログで見る」）。

流れ（番号どおりに腹落ちさせる）:

```text
① ConfigMap に APP_MESSAGE = "hello from ConfigMap" を置く
② Deployment の env が configMapKeyRef でそれを参照
③ Pod 起動時、コンテナの環境変数 APP_MESSAGE にコピーされる
④ command の echo が MESSAGE=hello from ConfigMap と印刷する
⑤ kubectl logs でその印刷結果を読む  ← 「Config が渡った結果をログで見る」
```

成功時に見える例:

```text
MESSAGE=hello from ConfigMap
SECRET_SET=yes
```

| 行 | 読み方 |
|----|--------|
| `MESSAGE=hello from ConfigMap` | ConfigMap の値が環境変数経由で届いた |
| `SECRET_SET=yes` | Secret も環境変数に入った（中身の文字列は出していない） |

もし ConfigMap 参照が壊れていると、だいたい `MESSAGE=`（空）になります。

## ハンズオン

**作業ディレクトリ:** `03-kubernetes/04-namespace-config`

```bash
kubectl apply -f ../manifests/04-namespace-config.yaml

kubectl get ns learn
kubectl get configmap,secret -n learn
kubectl get pods -n learn

# ① の保管庫の中身（設定そのもの）
kubectl get configmap hello-config -n learn -o yaml

# ④⑤ Pod が echo した内容（「渡った結果」）
kubectl logs -n learn deploy/hello-config

kubectl describe pod -n learn -l app=hello-config
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | NS / CM / Secret / Deployment 等を一括適用 | 仕切り付きアプリを一度に作る |
| `kubectl get ns learn` | Namespace の存在確認 | 仕切りができたか見る |
| `kubectl get configmap,secret -n learn` | 指定 NS の設定類を一覧 | `-n` = Namespace 指定 |
| `kubectl get pods -n learn` | その NS の Pod | アプリが learn にいるか確認 |
| `kubectl get configmap ... -o yaml` | ConfigMap の中身を表示 | 「保管庫に何が入っているか」を見る |
| `kubectl logs -n learn deploy/...` | Pod が印刷した標準出力を読む | 「環境変数として渡ったか」を echo 結果で確認する |
| `kubectl describe ... -n learn` | 詳細 | env の参照設定を確認 |

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
