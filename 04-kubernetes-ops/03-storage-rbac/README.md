# 03. Storage（PVC）と RBAC

**この節の目的:** 「消えないディスク要求」と「誰が何をできるか」の最小体験。

---

## PVC

Pod は消えると中のファイルも消えがち。PVC は「ディスクが欲しい」という要求。

### マニフェスト解説（`../manifests/pvc-demo.yaml`）

1 ファイルに Namespace + PVC + Deployment（`---` 区切り）。

**PVC:**

```yaml
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
  namespace: ops-learn
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `kind: PersistentVolumeClaim` | ディスク要求 | 「1Gi 欲しい」と宣言（実体はクラスタ側が用意） |
| `accessModes: ReadWriteOnce` | 同時に書き込めるノード数の目安 | 1 ノードから読み書き（学習用の定番） |
| `storage: 1Gi` | 容量 | 要求サイズ |

**Deployment 側のマウント:**

```yaml
containers:
  - name: app
    volumeMounts:
      - name: data
        mountPath: /data
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: demo-pvc
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `volumes[].persistentVolumeClaim` | PVC をボリュームとして使う | `demo-pvc` を Pod に紐づける |
| `volumeMounts.mountPath` | コンテナ内のパス | `/data` に見えるようにする |

流れ: PVC（要求）→ Bound（実ボリュームと結び付き）→ Pod の `/data` にマウント。

### ハンズオン

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

### マニフェスト解説（`../manifests/rbac-demo.yaml`）

**ServiceAccount（誰）:**

```yaml
kind: ServiceAccount
metadata:
  name: readonly-sa
  namespace: ops-rbac
```

**Role（何ができるか）:**

```yaml
kind: Role
metadata:
  name: pod-reader
  namespace: ops-rbac
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

| フィールド | 意味 | 目的 |
|------------|------|------|
| `resources: ["pods"]` | 対象リソース | Pod だけ |
| `verbs: get,list,watch` | 許可する操作 | 読む系のみ（delete は無し） |

**RoleBinding（誰にその Role を付けるか）:**

```yaml
kind: RoleBinding
subjects:
  - kind: ServiceAccount
    name: readonly-sa
roleRef:
  kind: Role
  name: pod-reader
```

流れ: SA（主体）← RoleBinding ← Role（権限の束）。

### ハンズオン

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

- [ ] PVC の YAML（claim / mount）の流れを説明できる
- [ ] Role の `verbs` と can-i の結果の関係を説明できる
