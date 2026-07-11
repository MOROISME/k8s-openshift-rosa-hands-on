# 01. トラブルシュート（壊して直す）

**この節の目的:** わざと壊した状態を観察し、現場と同じ順（get → describe → logs）で直す。

共通オプション:

| オプション | 意味 | 目的 |
|------------|------|------|
| `-l app=...` | ラベルで絞る | 関係ない Pod を見ない |
| `-f FILE` | マニフェスト指定 | 作成・削除の対象を YAML で揃える |
| `--ignore-not-found` | 無くてもエラーにしない | 片付けを安全に繰り返す |

---

## シナリオ A: ImagePullBackOff

**目的:** 「イメージが取れない」障害の見た目と直し方を覚える。

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
| `apply` fixed | 正しいイメージ名に直して再宣言 |
| `get pods` | Running に戻ったか確認 |

```bash
kubectl delete -f ../manifests/broken-imagepull.yaml --ignore-not-found
kubectl delete -f ../manifests/fixed-imagepull.yaml
```

---

## シナリオ B: CrashLoopBackOff

**目的:** 「起動してもすぐ落ちる」障害をログで特定する。

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

## シナリオ C: Service に届かない

**目的:** Pod は生きているのに届かない＝経路（ラベル）問題を Endpoints で見抜く。

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

- [ ] 3 シナリオとも「壊れた状態」を観察してから直した
- [ ] 各コマンドを打つ目的を説明できる
