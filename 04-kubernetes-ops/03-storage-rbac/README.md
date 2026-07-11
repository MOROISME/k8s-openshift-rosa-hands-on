# 03. Storage（PVC）と RBAC

**この節の目的:** 「消えないディスク要求」と「誰が何をできるか」の最小体験。

---

## PVC

Pod は消えると中のファイルも消えがち。PVC は「ディスクが欲しい」という要求。

```bash
kubectl apply -f ../manifests/pvc-demo.yaml
kubectl get pvc -n ops-learn
kubectl get pods -n ops-learn
kubectl exec -n ops-learn deploy/pvc-demo -- sh -c 'echo hello > /data/hello.txt && cat /data/hello.txt'
kubectl delete -f ../manifests/pvc-demo.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `apply` | NS + PVC + Deployment | 永続ボリューム付き Pod を作る |
| `get pvc` | PVC 状態 | `Bound` = 実ボリュームに結び付いた |
| `get pods` | Pod 状態 | マウントできて Running か |
| `kubectl exec ... -- sh -c '...'` | Pod 内でコマンド実行 | `/data` に書ける＝ボリュームが効いているか確認 |
| `delete` | 片付け | 学習リソース削除 |

### 期待結果

- PVC `Bound`、Pod `Running`、`/data/hello.txt` に書ける

---

## RBAC

Role / RoleBinding で「誰が何をできるか」を制御。権限不足は現場頻出。

```bash
kubectl apply -f ../manifests/rbac-demo.yaml
kubectl auth can-i get pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
kubectl auth can-i delete pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
kubectl delete -f ../manifests/rbac-demo.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `apply` | SA / Role / RoleBinding | 読み取り専用アカウントを作る |
| `kubectl auth can-i VERB RESOURCE --as=...` | 「その主体はその操作を許されるか」を問い合わせ | 権限設計のテスト。実削除せずに yes/no が分かる |
| `delete` | 片付け | |

### 期待結果

- `get pods` → `yes`
- `delete pods` → `no`

## 完了条件（DoD）

- [ ] PVC と `exec` で書いたことの目的を説明できる
- [ ] `can-i` の目的を説明できる
