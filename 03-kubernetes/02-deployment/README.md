# 02. Deployment

Deployment は「何台動かすか」「更新どうするか」を管理します。  
**この節の目的:** Pod を直接ではなく Deployment 経由で扱い、スケールを体験する。

## ハンズオン

```bash
kubectl apply -f ../manifests/02-deployment.yaml

kubectl get deploy
kubectl get pods -l app=hello

kubectl scale deploy hello-deploy --replicas=3
kubectl get pods -l app=hello
kubectl rollout status deploy/hello-deploy
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | Deployment を作成/更新 | 「望ましい状態」を宣言する |
| `kubectl get deploy` | Deployment 一覧 | READY（何台 Ready か）を見る |
| `kubectl get pods -l app=hello` | ラベルで Pod を絞る | Deployment が作った Pod だけ見る。`-l` は label selector |
| `kubectl scale ... --replicas=3` | 希望台数を 3 に変更 | スケールアウトを体験 |
| `kubectl rollout status ...` | 更新/スケールの完了待ち | 「終わるまで待つ」運用の型 |

片付けは Service 演習のあとでも可。先に消す場合:

```bash
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

```text
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   2/2     2            2           ...

# scale 後
hello-deploy   3/3     3            3           ...
```

## ポイント

- 普段触るのは Pod ではなく Deployment
- `replicas` = 「何個ほしいか」の宣言。K8s が実際の Pod 数を合わせる

## 完了条件（DoD）

- [ ] Deployment から複数 Pod が作られることを確認した
- [ ] `scale` と `-l` の目的を説明できる
