# 02. Probe と resources

**この節の目的:** 「健康診断」と「CPU/メモリの枠」がなぜ必要かを、describe で実物を見て理解する。

| 種類 | 意味 | 目的 |
|------|------|------|
| readinessProbe | 準備チェック | まだ準備中の Pod を Service から外す |
| livenessProbe | 生存チェック | 固まったコンテナを再起動させる |
| requests | 予約量 | スケジューラがノードに載せられるか判断 |
| limits | 上限 | 食い過ぎを防ぐ（暴走抑制） |

## YAML設定の解説（`../manifests/probes-resources.yaml`）

```yaml
containers:
  - name: web
    image: nginx:alpine
    ports:
      - containerPort: 80
    resources:
      requests:
        cpu: "50m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 10
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `resources.requests` | 最低このくらいは確保したい | ノード選び・混雑時の目安 |
| `resources.limits` | これ以上は使わせない | 暴走・隣への影響を抑える |
| `readinessProbe.httpGet` | 「準備できたか」を HTTP で聞く | 失敗中は Service の名簿から外す |
| `livenessProbe.httpGet` | 「生きているか」を HTTP で聞く | 失敗が続くとコンテナ再起動 |
| `initialDelaySeconds` | 最初のチェックまで待つ秒 | 起動直後の誤判定を減らす |
| `periodSeconds` | チェック間隔 | 何秒ごとに見るか |

`50m` = CPU の 0.05 コア相当。単位の細かい暗記より、「予約」と「上限」の違いが大事です。

## 実習

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
- [ ] YAML の `requests` / `limits` / Probe の意味を説明できる
