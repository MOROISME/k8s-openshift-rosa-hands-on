# 03. Service

Service は、入れ替わる Pod への**安定したアクセス口**です。

## ハンズオン

前提: Deployment があること（なければ再適用）。

```bash
kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml

kubectl get svc hello-svc
minikube service hello-svc --url
```

表示された URL に:

```bash
curl "$(minikube service hello-svc --url)"
```

片付け:

```bash
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- `kubectl get svc` で `hello-svc` がある
- `curl` が HTML（nginx 既定ページ等）を返す / 接続できる
- `kubectl get endpoints hello-svc` で Pod IP が並ぶ

## ポイント

- Pod IP は作り直すと変わる → Service が吸収する
- 種類: ClusterIP / NodePort / LoadBalancer など

## トラブル時

| 症状 | 対処 |
|------|------|
| curl 失敗 | Endpoints が空でないか。ラベル不一致を疑う |
| URL が出ない | `kubectl get svc` と `minikube status` |

## 完了条件（DoD）

- [ ] Service 経由で到達できた
- [ ] 「なぜ Service が必要か」を説明できる
