# 04. Namespace / ConfigMap / Secret

## 概要

- Namespace … 論理的な仕切り
- ConfigMap … 設定値（非機密寄り）
- Secret … 機密（Base64。暗号化とは限らない）

## ハンズオン

```bash
kubectl apply -f ../manifests/04-namespace-config.yaml

kubectl get ns learn
kubectl get configmap,secret -n learn
kubectl get pods -n learn

kubectl logs -n learn deploy/hello-config
kubectl describe pod -n learn -l app=hello-config
```

片付け:

```bash
kubectl delete -f ../manifests/04-namespace-config.yaml
```

## 期待結果

- `learn` Namespace がある
- Pod が `Running`
- logs / describe で ConfigMap 由来の値が確認できる（マニフェスト設計どおり）

## 完了条件（DoD）

- [ ] `-n learn` で操作できる
- [ ] ConfigMap が Pod に渡る流れを説明できる
