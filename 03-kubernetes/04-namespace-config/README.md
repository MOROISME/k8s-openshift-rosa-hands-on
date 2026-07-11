# 04. Namespace / ConfigMap / Secret

## Namespace

クラスタ内の論理的な仕切り。チームや環境（dev / stg）の分離に使う。

## ConfigMap / Secret

- ConfigMap … 設定値（非機密寄り）
- Secret … パスワード等（Base64。暗号化とは限らない点に注意）

## ハンズオン

```bash
kubectl apply -f ../manifests/04-namespace-config.yaml

kubectl get ns learn
kubectl get configmap,secret -n learn
kubectl get pods -n learn

kubectl logs -n learn deploy/hello-config
kubectl describe pod -n learn -l app=hello-config

# 片付け
kubectl delete -f ../manifests/04-namespace-config.yaml
```

## チェックリスト

- [ ] `-n learn` で Namespace を指定して操作できる
- [ ] ConfigMap の値が Pod に渡っていることを確認した
