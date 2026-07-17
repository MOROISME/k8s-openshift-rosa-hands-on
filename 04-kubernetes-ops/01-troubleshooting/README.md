# 01. 障害の切り分け（壊して直す）

**この節の目的:** わざと壊した状態を観察し、現場と同じ順（get → describe → logs）で直す。

共通の追加指定:

| 追加の指定 | 意味 | 目的 |
|------------|------|------|
| `-l app=...` | 付箋で絞る | 関係ない Pod を見ない |
| `-f FILE` | YAML設定指定 | 作成・削除の対象を YAML で揃える |
| `--ignore-not-found` | 無くてもエラーにしない | 片付けを安全に繰り返す |

---

## 練習問題 A: ImagePullBackOff

**目的:** 「コンテナのひな形が取れない」障害の見た目と直し方を覚える。

### YAML設定の解説

**壊れた版**（`../manifests/broken-imagepull.yaml`）:

```yaml
containers:
  - name: web
    # わざと存在しないコンテナのひな形
    image: nginx:this-tag-does-not-exist-12345
```

| フィールド | 意味 | 目的（この演習） |
|------------|------|------------------|
| `image: ...does-not-exist...` | 存在しないタグ | pull 失敗 → `ImagePullBackOff` を意図的に起こす |

**直した版**（`../manifests/fixed-imagepull.yaml`）:

```yaml
image: nginx:alpine
```

| 差分 | 意味 | 目的 |
|------|------|------|
| 正しいコンテナのひな形名 | レジストリから取れる | Running に戻す |

現場でも「YAML の `image` が間違っていないか」は最初に疑うポイントです。

### 実習

```bash
kubectl apply -f ../manifests/broken-imagepull.yaml
kubectl get pods -l app=broken-pull
kubectl describe pod -l app=broken-pull
```

| コマンド | 目的 |
|----------|------|
| `apply` broken | 存在しないタグの Deployment を意図的に作る |
| `get pods -l ...` | STATUS が ImagePullBackOff 等か確認 |
| `describe` | Events に Failed to pull image があるか読む |

### 期待結果（壊れた状態）

- `ImagePullBackOff` / `ErrImagePull`
- Events に pull 失敗

### 修復

```bash
kubectl apply -f ../manifests/fixed-imagepull.yaml
kubectl get pods -l app=broken-pull
```

| コマンド | 目的 |
|----------|------|
| `apply` fixed | 正しいコンテナのひな形名に直して再宣言 |
| `get pods` | Running に戻ったか確認 |

```bash
kubectl delete -f ../manifests/broken-imagepull.yaml --ignore-not-found
kubectl delete -f ../manifests/fixed-imagepull.yaml
```

---

## 練習問題 B: CrashLoopBackOff

**目的:** 「起動してもすぐ落ちる」障害をログで特定する。

### YAML設定の解説

**壊れた版**（`../manifests/broken-crashloop.yaml`）:

```yaml
containers:
  - name: boom
    image: busybox:1.36
    # わざとすぐ終了する
    command: ["sh", "-c", "echo boom: intentional crash; exit 1"]
```

| フィールド | 意味 | 目的（この演習） |
|------------|------|------------------|
| `command` | コンテナ起動時のコマンド | コンテナのひな形自体は正しい |
| `exit 1` | 異常終了 | すぐ落ちて `CrashLoopBackOff` になる |

**直した版**（`../manifests/fixed-crashloop.yaml`）:

```yaml
command: ["sh", "-c", "echo ok: staying up; sleep 3600"]
```

| 差分 | 意味 | 目的 |
|------|------|------|
| `sleep 3600` | 落ちずに居座る | Running を維持する |

コンテナのひな形 pull は成功するのに CrashLoop なら、**ログ（アプリの落ち方）** を見ます。

### 実習

```bash
kubectl apply -f ../manifests/broken-crashloop.yaml
kubectl get pods -l app=broken-crash
kubectl describe pod -l app=broken-crash
kubectl logs -l app=broken-crash --tail=50
```

| コマンド | 目的 |
|----------|------|
| `apply` broken | すぐ exit 1 するコンテナを起動 |
| `get` / `describe` | CrashLoop と Events を確認 |
| `logs ... --tail=50` | 直近 50 行のアプリログ。落ちる理由はここに出ることが多い |

### 期待結果（壊れた状態）

- `CrashLoopBackOff`
- logs に `boom: intentional crash` など

### 修復

```bash
kubectl apply -f ../manifests/fixed-crashloop.yaml
kubectl get pods -l app=broken-crash
kubectl delete -f ../manifests/broken-crashloop.yaml --ignore-not-found
kubectl delete -f ../manifests/fixed-crashloop.yaml
```

---

## 練習問題 C: Service に届かない

**目的:** Pod は生きているのに届かない＝経路（付箋）問題を Endpoints で見抜く。

### YAML設定の解説

1 ファイルに Deployment + Service（`---` 区切り）。

**壊れた版**（`../manifests/broken-service.yaml`）の要点:

```yaml
# Pod 側
template:
  metadata:
    labels:
      app: svc-miss-pod

# Service 側（わざと不一致）
spec:
  selector:
    app: wrong-label
```

| フィールド | 意味 | 目的（この演習） |
|------------|------|------------------|
| Pod の `labels.app` | 実体の付箋 | `svc-miss-pod` |
| Service の `selector` | 探す条件 | `wrong-label` → 誰にもヒットしない |
| 結果 | Endpoints が空 | 窓口はあるが後ろに誰もいない |

**直した版**（`../manifests/fixed-service.yaml`）:

```yaml
selector:
  app: svc-miss-pod
```

| 差分 | 意味 | 目的 |
|------|------|------|
| selector を Pod 付箋に合わせる | 届け先一覧に載る | curl が届く |

### 実習

```bash
kubectl apply -f ../manifests/broken-service.yaml
kubectl get pods,svc,endpoints -l exercise=svc-miss
kubectl describe svc svc-miss
```

| コマンド | 目的 |
|----------|------|
| `apply` broken | selector がずれた Service を作る |
| `get pods,svc,endpoints` | 一度に経路の部品を見る。Endpoints 空が典型 |
| `describe svc` | selector の内容を確認 |

### 修復

```bash
kubectl apply -f ../manifests/fixed-service.yaml
kubectl get endpoints svc-miss
curl -I "$(minikube service svc-miss --url)"
```

| コマンド | 目的 |
|----------|------|
| `apply` fixed | 正しい selector に直す |
| `get endpoints` | Pod IP が載ったか確認 |
| `curl -I` | 実際に HTTP で届くか確認 |

```bash
kubectl delete -f ../manifests/fixed-service.yaml
kubectl delete -f ../manifests/broken-service.yaml --ignore-not-found
```

## 完了条件（DoD）

- [ ] 3 つの練習問題とも「壊れた状態」を観察してから直した
- [ ] 各練習問題で「YAML のどこが原因か」を説明できる
- [ ] 各コマンドを打つ目的を説明できる
