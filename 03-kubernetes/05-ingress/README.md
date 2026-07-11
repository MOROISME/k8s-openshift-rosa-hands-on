# 05. Ingress（任意）

Ingress はクラスタ外からの HTTP(S) 入り口です。  
OpenShift では似た役割を **Route** が担うことが多いです。

## ハンズオン（Minikube）

```bash
# Ingress アドオン
minikube addons enable ingress

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml
kubectl apply -f ../manifests/05-ingress.yaml

kubectl get ingress
```

hosts の解決は環境によって異なります。うまくいかない場合は公式の Minikube Ingress 手順を参照してください。

```bash
# 片付け
kubectl delete -f ../manifests/05-ingress.yaml
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## チェックリスト

- [ ] Ingress と Service / Deployment の関係を説明できる
- [ ] 「OpenShift では Route」と覚えている
