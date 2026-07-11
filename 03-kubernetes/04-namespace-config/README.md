# 04. Namespace / ConfigMap / Secret

**この節の目的:** 論理仕切りと「設定をイメージに焼き込まない」やり方を体験する。

## 概要

| リソース | 意味 | 目的 |
|----------|------|------|
| Namespace | クラスタ内の論理仕切り | チーム・環境（dev 等）の分離 |
| ConfigMap | 非機密寄りの設定 | 環境変数や設定ファイルとして Pod に渡す |
| Secret | 機密情報 | パスワード等（中身は Base64。暗号化とは限らない） |

## ハンズオン

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
| `kubectl get configmap,secret -n learn` | 指定 NS の設定類を一覧 | `-n` = Namespace 指定。他 NS と混ざらないようにする |
| `kubectl get pods -n learn` | その NS の Pod | アプリが learn にいるか確認 |
| `kubectl logs -n learn deploy/...` | Deployment 配下 Pod のログ | Config が渡った結果をログで見る |
| `kubectl describe ... -n learn` | 詳細 | 環境変数やマウントの実態を確認 |

片付け:

```bash
kubectl delete -f ../manifests/04-namespace-config.yaml
```

## 期待結果

- `learn` Namespace がある
- Pod が `Running`
- logs / describe で ConfigMap 由来の値が確認できる

## 完了条件（DoD）

- [ ] `-n` の目的を説明できる
- [ ] ConfigMap が Pod に渡る流れを説明できる
