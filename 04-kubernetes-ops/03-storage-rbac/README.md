# 03. Storage（PVC）と RBAC

**この節の目的:** 「消えないディスク要求」と「誰が何をできるか」の最小体験。

**作業ディレクトリ（重要）:**

```bash
cd 04-kubernetes-ops/03-storage-rbac
```

ここから見た相対パスが `../manifests/...` です。  
`04-kubernetes-ops` 直下やリポジトリ直下だと `../manifests/pvc-demo.yaml` は見つかりません。

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

流れ:

```text
① PVC「1Gi 欲しい」と宣言
② クラスタが実ボリュームを用意して Bound（結び付き）
③ Deployment が volumes + volumeMounts で /data にマウント
④ exec で /data に書く → ボリュームが効いている証拠
```

### ハンズオン（PVC）

#### 手順 0: 場所を確認

```bash
pwd
# .../04-kubernetes-ops/03-storage-rbac であること

ls ../manifests/pvc-demo.yaml
# ファイルが見えればパス OK
```

#### 手順 1: 適用する

```bash
kubectl apply -f ../manifests/pvc-demo.yaml
```

| 意味 | 目的 |
|------|------|
| YAML をクラスタに反映 | Namespace `ops-learn` + PVC + Deployment を一度に作る |

期待する出力例:

```text
namespace/ops-learn created
persistentvolumeclaim/demo-pvc created
deployment.apps/pvc-demo created
```

（再実行すると `unchanged` / `configured` になることもあります）

#### 手順 2: PVC が Bound か見る

```bash
kubectl get pvc -n ops-learn
```

| オプション / 列 | 意味 | 目的 |
|-----------------|------|------|
| `-n ops-learn` | Namespace 指定 | 仕切りの中だけ見る |
| `STATUS` | PVC の状態 | **`Bound`** = 実ボリュームと結び付いた（成功） |
| `VOLUME` | 実際に割り当てられたボリューム名 | 要求（PVC）に実体が付いた印 |
| `CAPACITY` | 容量 | だいたい `1Gi` |
| `STORAGECLASS` | どの種類のストレージか | Minikube では `standard` が多い |

期待例:

```text
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
demo-pvc   Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO            standard
```

`Pending` のまま長いときは `kubectl describe pvc demo-pvc -n ops-learn` で Events を見る。

#### 手順 3: Pod が Running か見る

```bash
kubectl get pods -n ops-learn
```

注意: `kubectl pods` は誤りです。必ず **`kubectl get pods`**。

| 見る列 | 意味 | 目的 |
|--------|------|------|
| `READY` | 準備できたか（例: `1/1`） | コンテナが動いている |
| `STATUS` | 状態 | **`Running`** が成功 |

期待例:

```text
NAME                        READY   STATUS    RESTARTS   AGE
pvc-demo-xxxxxxxxxx-xxxxx   1/1     Running   0          ...
```

#### 手順 4: `/data` に書いて読む（ボリューム確認）

```bash
kubectl exec -n ops-learn deploy/pvc-demo -- sh -c 'echo hello > /data/hello.txt && cat /data/hello.txt'
```

この 1 行を分解すると:

| 部分 | 意味 | 目的 |
|------|------|------|
| `kubectl exec` | Pod の中でコマンドを実行 | コンテナ内に入らずに 1 発実行 |
| `-n ops-learn` | Namespace | `ops-learn` の中の Pod を指定 |
| `deploy/pvc-demo` | Deployment 名で Pod を指定 | Pod 名が長くてもよい書き方 |
| `--` | ここから先はコンテナ内コマンド | kubectl の引数と分ける |
| `sh -c '...'` | シェルで複数処理 | 書く → 読むを連続実行 |
| `echo hello > /data/hello.txt` | `/data` にファイル作成 | **マウント先に書けるか** |
| `cat /data/hello.txt` | 中身表示 | 書けた証拠をターミナルに出す |

期待する出力:

```text
hello
```

`hello` と出れば、「PVC → ボリューム → `/data` マウント」までつながっています。

任意（中に入って自分で触る）:

```bash
kubectl exec -it -n ops-learn deploy/pvc-demo -- sh
# 中で:
#   ls /data
#   cat /data/hello.txt
#   exit
```

#### 手順 5: PVC の片付け

RBAC に進む前でも、後でまとめてでも可。消すとき:

```bash
kubectl delete -f ../manifests/pvc-demo.yaml
```

| 意味 | 目的 |
|------|------|
| YAML と同じ資源を削除 | NS / PVC / Deployment を片付ける |

### 期待結果（PVC）

- PVC `STATUS=Bound`
- Pod `Running`
- `exec` で `hello` と表示される

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

流れ:

```text
ServiceAccount（readonly-sa）＝ 誰
        ↑ RoleBinding で結びつける
Role（pod-reader）＝ get/list/watch だけ可
        ↓
can-i get → yes / can-i delete → no
```

### ハンズオン（RBAC）

作業ディレクトリは引き続き:

```bash
cd 04-kubernetes-ops/03-storage-rbac
```

#### 手順 1: 適用する

```bash
kubectl apply -f ../manifests/rbac-demo.yaml
```

| 作られるもの | 意味 |
|--------------|------|
| Namespace `ops-rbac` | この演習用の仕切り |
| ServiceAccount `readonly-sa` | 読み取り専用の「誰」 |
| Role `pod-reader` | Pod の get/list/watch だけ |
| RoleBinding `read-pods` | その Role を SA に付ける |

期待する出力例:

```text
namespace/ops-rbac created
serviceaccount/readonly-sa created
role.rbac.authorization.k8s.io/pod-reader created
rolebinding.rbac.authorization.k8s.io/read-pods created
```

任意で中身確認:

```bash
kubectl get sa,role,rolebinding -n ops-rbac
```

#### 手順 2: 「読めるか？」を問い合わせる（期待: yes）

```bash
kubectl auth can-i get pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
```

| 部分 | 意味 | 目的 |
|------|------|------|
| `auth can-i` | 「この操作は許される？」と API に聞く | 実際に削除せずに権限テスト |
| `get pods` | 動詞 + リソース | Role の `verbs` / `resources` と対応 |
| `-n ops-rbac` | どの Namespace か | Role は Namespace 付きなので範囲を合わせる |
| `--as=system:serviceaccount:ops-rbac:readonly-sa` | **自分ではなく**その SA のつもりで聞く | 「readonly-sa ならできるか？」 |

`--as=` の形:

```text
system:serviceaccount:<namespace>:<sa名>
                 ops-rbac      readonly-sa
```

期待する出力:

```text
yes
```

Role に `get` があるので yes。

#### 手順 3: 「消せるか？」を問い合わせる（期待: no）

```bash
kubectl auth can-i delete pods -n ops-rbac --as=system:serviceaccount:ops-rbac:readonly-sa
```

| 意味 | 目的 |
|------|------|
| 同じ SA で `delete` を試す | Role に `delete` が無いことを確認 |

期待する出力:

```text
no
```

これが「読み取り専用」の実感です。権限不足の切り分けでも、まず `can-i` で yes/no を見ます。

#### 手順 4: RBAC の片付け

```bash
kubectl delete -f ../manifests/rbac-demo.yaml
```

PVC 側もまだ残っていれば:

```bash
kubectl delete -f ../manifests/pvc-demo.yaml
```

### 期待結果（RBAC）

| コマンド | 結果 |
|----------|------|
| `can-i get pods ...` | `yes` |
| `can-i delete pods ...` | `no` |

## よくあるつまずき

| 症状 | 原因 | 対処 |
|------|------|------|
| `path does not exist` | 作業ディレクトリが違う | `cd 04-kubernetes-ops/03-storage-rbac` |
| `unknown command "pods"` | `kubectl pods` と打った | `kubectl get pods` |
| PVC が `Pending` | ストレージ供給が遅い／失敗 | `kubectl describe pvc -n ops-learn` |
| `can-i` がどちらも no | apply 漏れ・`--as` の Namespace/名前違い | YAML 再 apply、`--as=` を見直す |

## 完了条件（DoD）

- [ ] 作業ディレクトリと `../manifests/` の関係を説明できる
- [ ] PVC の YAML（claim / mount）と `Bound` / `exec` の流れを説明できる
- [ ] Role の `verbs` と `can-i` の yes/no の関係を説明できる
