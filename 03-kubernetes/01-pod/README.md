# 01. Pod

Pod は Kubernetes でコンテナを動かす**最小単位**です。

## ハンズオン

```bash
kubectl apply -f ../manifests/01-pod.yaml
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod
```

任意:

```bash
kubectl exec -it hello-pod -- /bin/sh
# exit で抜ける
```

片付け:

```bash
kubectl delete -f ../manifests/01-pod.yaml
```

## 期待結果

```text
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
hello-pod   1/1     Running   0          ...
```

- `describe` の末尾 `Events` に失敗が並んでいない
- `logs` が取れる（nginx はアクセスが無いと静かでもよい）

## ポイント

- 本番運用で Pod を直接作り続けることは少ない（Deployment 経由が基本）
- まずは「クラスタ上で 1 個動いている」感覚を掴む

## トラブル時

| 症状 | 対処 |
|------|------|
| `ImagePullBackOff` | ネット確認。しばらく待つ。`describe` の Events を読む |
| `Pending` のまま | `minikube status`。ノードが Ready か確認 |

## 完了条件（DoD）

- [ ] `Running` / `1/1` を確認した
- [ ] `describe` / `logs` を使った
