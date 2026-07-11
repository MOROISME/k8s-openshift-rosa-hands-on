# 01. Pod

Pod は Kubernetes でコンテナを動かす**最小単位**です。

## ハンズオン

```bash
# マニフェスト適用
kubectl apply -f ../manifests/01-pod.yaml

# 確認
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod

# 中に入る（任意）
kubectl exec -it hello-pod -- /bin/sh
# exit で抜ける

# 削除
kubectl delete -f ../manifests/01-pod.yaml
```

## ポイント

- Pod を直接運用し続けることは少ない（本番では Deployment 経由が基本）
- まず「クラスタ上でコンテナが 1 個動いている」感覚を掴む

## チェックリスト

- [ ] Pod が Running になるまで待てる
- [ ] `describe` / `logs` で状態とログを見られる
