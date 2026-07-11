# 05. Ingress（任意）

Ingress はクラスタ外からの HTTP(S) 入り口です。OpenShift では **Route** が担うことが多いです。

無料の Minikube で概念だけ掴む章です。飛ばして Step 4 に進んでも L4 は達成可能です。

## ハンズオン

```bash
minikube addons enable ingress

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml
kubectl apply -f ../manifests/05-ingress.yaml

kubectl get ingress
```

hosts 解決は環境差あり。詰まったら公式 Minikube Ingress 手順へ。

片付け:

```bash
kubectl delete -f ../manifests/05-ingress.yaml
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- `kubectl get ingress` にリソースが出る
- （環境が許せば）Ingress 経由で到達できる

## 完了条件（DoD）

- [ ] Ingress と Service / Deployment の関係を説明できる
- [ ] 「OpenShift では Route」と覚えている
