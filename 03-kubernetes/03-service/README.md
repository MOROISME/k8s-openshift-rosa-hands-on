# 03. Service

Service は、入れ替わる Pod に対して**安定した名前とアクセス口**を提供します。

## ハンズオン

前提: Deployment が残っていること（なければ `02-deployment` を再適用）。

```bash
kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml

kubectl get svc hello-svc

# Minikube で簡単にブラウザ/curl 用 URL を出す
minikube service hello-svc --url
```

別ターミナルで表示された URL に `curl` してみてください。

```bash
# 例
curl "$(minikube service hello-svc --url)"
```

片付け:

```bash
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## ポイント

- Pod IP は作り直すと変わる → Service がその差分を吸収する
- `ClusterIP` / `NodePort` / `LoadBalancer` などの種類がある（ここでは NodePort 寄りに Minikube で見る）

## チェックリスト

- [ ] Service 経由でアプリに到達できた
- [ ] 「なぜ Service が必要か」を説明できる
