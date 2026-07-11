# 02. Deployment

Deployment は「何台動かすか」「更新どうするか」を管理します。

## ハンズオン

```bash
kubectl apply -f ../manifests/02-deployment.yaml

kubectl get deploy
kubectl get pods -l app=hello

kubectl scale deploy hello-deploy --replicas=3
kubectl get pods -l app=hello
kubectl rollout status deploy/hello-deploy
```

片付けは Service 演習のあとでも可。先に消す場合:

```bash
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

```text
kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   2/2     2            2           ...

# scale 後
hello-deploy   3/3     3            3           ...
```

## ポイント

- 普段触るのは Pod ではなく Deployment
- `replicas` が「何個ほしいか」の宣言

## 完了条件（DoD）

- [ ] Deployment から複数 Pod が作られることを確認した
- [ ] `scale` で台数が変わることを確認した
