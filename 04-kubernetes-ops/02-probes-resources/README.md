# 02. Probe と resources

| 種類 | 意味 |
|------|------|
| readinessProbe | 準備できてから Service に載せる |
| livenessProbe | 固まったら再起動 |
| requests | 予約 |
| limits | 上限 |

## ハンズオン

```bash
kubectl apply -f ../manifests/probes-resources.yaml
kubectl get pods -l app=healthy-demo
kubectl describe pod -l app=healthy-demo
```

`Readiness` / `Liveness` / `Limits` / `Requests` が describe に出ることを確認。

```bash
kubectl delete -f ../manifests/probes-resources.yaml
```

## 期待結果

- Pod `Running` / Ready
- describe に Probe と resources が記載される

## 完了条件（DoD）

- [ ] readiness と liveness の違いを説明できる
- [ ] requests / limits の目的を説明できる
