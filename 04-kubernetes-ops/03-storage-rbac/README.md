# 03. Storage（PVC）と RBAC

## PVC

```bash
kubectl apply -f ../manifests/pvc-demo.yaml
kubectl get pvc -n ops-learn
kubectl get pods -n ops-learn
kubectl exec -n ops-learn deploy/pvc-demo -- sh -c 'echo hello > /data/hello.txt && cat /data/hello.txt'
kubectl delete -f ../manifests/pvc-demo.yaml
```

### 期待結果

- PVC が `Bound`
- Pod `Running`
- `/data/hello.txt` に `hello` が書ける

## RBAC

```bash
kubectl apply -f ../manifests/rbac-demo.yaml
kubectl auth can-i get pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
kubectl auth can-i delete pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
kubectl delete -f ../manifests/rbac-demo.yaml
```

### 期待結果

- `get pods` → `yes`
- `delete pods` → `no`

## 完了条件（DoD）

- [ ] PVC がディスク要求だと説明できる
- [ ] `can-i` で権限差を確認した
