# 01. トラブルシュート（壊して直す）

## シナリオ A: ImagePullBackOff

```bash
kubectl apply -f ../manifests/broken-imagepull.yaml
kubectl get pods -l app=broken-pull
kubectl describe pod -l app=broken-pull
```

### 期待結果（壊れた状態）

- `STATUS` が `ImagePullBackOff` または `ErrImagePull`
- Events に `Failed to pull image` 系

### 修復

```bash
kubectl apply -f ../manifests/fixed-imagepull.yaml
kubectl get pods -l app=broken-pull
```

期待: `Running` / `1/1`

```bash
kubectl delete -f ../manifests/broken-imagepull.yaml --ignore-not-found
kubectl delete -f ../manifests/fixed-imagepull.yaml
```

## シナリオ B: CrashLoopBackOff

```bash
kubectl apply -f ../manifests/broken-crashloop.yaml
kubectl get pods -l app=broken-crash
kubectl describe pod -l app=broken-crash
kubectl logs -l app=broken-crash --tail=50
```

### 期待結果（壊れた状態）

- `CrashLoopBackOff`
- logs に `boom: intentional crash` など

### 修復

```bash
kubectl apply -f ../manifests/fixed-crashloop.yaml
kubectl get pods -l app=broken-crash
```

期待: `Running`

```bash
kubectl delete -f ../manifests/broken-crashloop.yaml --ignore-not-found
kubectl delete -f ../manifests/fixed-crashloop.yaml
```

## シナリオ C: Service に届かない

```bash
kubectl apply -f ../manifests/broken-service.yaml
kubectl get pods,svc,endpoints -l exercise=svc-miss
kubectl describe svc svc-miss
```

### 期待結果（壊れた状態）

- Pod は Running でも `Endpoints` が空
- selector と Pod label の不一致

### 修復

```bash
kubectl apply -f ../manifests/fixed-service.yaml
kubectl get endpoints svc-miss
curl -I "$(minikube service svc-miss --url)"
```

```bash
kubectl delete -f ../manifests/fixed-service.yaml
kubectl delete -f ../manifests/broken-service.yaml --ignore-not-found
```

## 完了条件（DoD）

- [ ] 3 シナリオとも「壊れた状態」を観察してから直した
- [ ] describe の Events を最初に見るクセがついた
