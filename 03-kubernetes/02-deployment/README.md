# 02. Deployment

Deployment は「このアプリを何台動かすか」「更新どうするか」を管理します。Pod が落ちても作り直してくれます。

## ハンズオン

```bash
kubectl apply -f ../manifests/02-deployment.yaml

kubectl get deploy
kubectl get pods -l app=hello

# レプリカを増やす
kubectl scale deploy hello-deploy --replicas=3
kubectl get pods -l app=hello

# ロールアウト状況
kubectl rollout status deploy/hello-deploy

# 片付け（Service をまだ作っていない場合）
# kubectl delete -f ../manifests/02-deployment.yaml
```

## ポイント

- 普段触るのは Pod ではなく Deployment（やそれに相当するもの）
- `replicas` が「何個ほしいか」の宣言

## チェックリスト

- [ ] Deployment から複数 Pod が作られることを確認した
- [ ] scale で台数が変わることを確認した
