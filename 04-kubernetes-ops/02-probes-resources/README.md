# 02. Probe と resources

**この節の目的:** 「健康診断」と「CPU/メモリの枠」がなぜ必要かを、describe で実物を見て理解する。

| 種類 | 意味 | 目的 |
|------|------|------|
| readinessProbe | 準備チェック | まだ準備中の Pod を Service から外す |
| livenessProbe | 生存チェック | 固まったコンテナを再起動させる |
| requests | 予約量 | スケジューラがノードに載せられるか判断 |
| limits | 上限 | 食い過ぎを防ぐ（暴走抑制） |

## ハンズオン

```bash
kubectl apply -f ../manifests/probes-resources.yaml
kubectl get pods -l app=healthy-demo
kubectl describe pod -l app=healthy-demo
```

| コマンド | 目的 |
|----------|------|
| `apply` | Probe / resources 付き Deployment を作る |
| `get pods` | Running / Ready か確認 |
| `describe` | Readiness / Liveness / Limits / Requests の記載を自分の目で見る |

```bash
kubectl delete -f ../manifests/probes-resources.yaml
```

## 期待結果

- Pod `Running` / Ready
- describe に Probe と resources がある

## 完了条件（DoD）

- [ ] readiness と liveness の違い（目的の違い）を説明できる
- [ ] requests / limits の目的を説明できる
