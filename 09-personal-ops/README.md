# 09. 個人運用ランブック（無料・週次）

**この Step の目的:** 学習を「一度やって終わり」にせず、無料検証環境を毎週自分で回す習慣にする（L4）。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## 個人環境

| 環境 | 役割（目的） |
|------|----------------|
| Minikube | 障害演習・デプロイ練習のホーム |
| Developer Sandbox | OpenShift / Route / `oc` の感覚維持 |
| Docs / Billing | ROSA 知識メンテ・請求 $0 確認（口座がある人） |

## 初回セットアップ（1 回）

### Minikube ホームアプリ

**目的:** 毎週の健全性チェック対象となる「いつも同じアプリ」を用意する。

```bash
minikube start
kubectl apply -f ../03-kubernetes/manifests/02-deployment.yaml
kubectl apply -f ../03-kubernetes/manifests/03-service.yaml
kubectl get deploy,pods,svc
curl "$(minikube service hello-svc --url)"
```

| コマンド | 目的 |
|----------|------|
| `minikube start` | クラスタ起動 |
| `apply` Deployment/Service | ホームアプリを宣言 |
| `get deploy,pods,svc` | 一式が Ready か確認 |
| `curl "$(minikube service ...)"` | 外から届くか確認 |

### Sandbox ホームアプリ

**目的:** OpenShift 側にも同様の定点観測対象を置く。

```bash
oc apply -f ../05-openshift/manifests/app-deployment.yaml
oc apply -f ../05-openshift/manifests/app-service.yaml
oc apply -f ../05-openshift/manifests/app-route.yaml
oc get route sandbox-web
```

| コマンド | 目的 |
|----------|------|
| `oc apply` 一式 | Deployment / Service / Route を揃える |
| `oc get route` | 公開 URL（HOST）を控える |

### 運用ログ

```bash
cp templates/ops-log.md ./ops-log-local.md
```

| コマンド | 目的 |
|----------|------|
| `cp templates/... ./ops-log-local.md` | 週次記録用ファイルを作る（秘密は書かない。gitignore 対象） |

---

## 週次ランブック（30〜60 分）

毎週同じ順。結果を `ops-log-local.md` に残す。

### 1. 健全性（10 分）— なぜやるか

「壊れていないか」を定点観測し、異常の早期発見と手順の定着。

```bash
minikube status
kubectl get nodes
kubectl get deploy,pods,svc
curl -I "$(minikube service hello-svc --url)"
```

| コマンド | 目的 |
|----------|------|
| `minikube status` | Minikube 自体が生きているか |
| `kubectl get nodes` | ノード Ready か |
| `kubectl get deploy,pods,svc` | ホームアプリ一式 |
| `curl -I` | HTTP 到達性 |

Sandbox がある週:

```bash
oc whoami
oc get pods
oc get route
```

| コマンド | 目的 |
|----------|------|
| `oc whoami` | ログイン有効か |
| `oc get pods` / `route` | アプリと公開面 |

### 2. 障害ドリル（15 分）— なぜやるか

本番前に切り分け筋記憶を維持する。週 1 種でよい。

- ImagePullBackOff / CrashLoop / Service 不一致（Step 4）

必ず: 壊す → 観察 → 直す → 片付け。

### 3. OpenShift 感覚（10 分）— なぜやるか

Sandbox 期限までに Route / 権限感覚を忘れない。

- Route を開く / ログ / `oc auth can-i get pods`

### 4. 知識メンテ（10 分）— なぜやるか

ROSA は作れなくても説明力を落とさない。

- Docs 1 セクション、または Step 8 を白紙で書く

### 5. 請求ガード（5 分）— なぜやるか

消し忘れ課金を防ぐ。AWS 口座がある人のみ。

---

## 月次（任意）

| 作業 | 目的 |
|------|------|
| `minikube delete && minikube start` → 再デプロイ | 再現性・手順の鮮度確認 |
| Sandbox 期限確認 | 突然使えなくなる前に把握 |
| L4 自己評価 | 到達の見直し |

## L4 自己評価

- [ ] 週次を連続 3 週実施しログがある
- [ ] 障害ドリル 3 種の大枠を一人でできる
- [ ] Sandbox で Route 公開をやり直せる（または代替方針）
- [ ] ROSA を作らず説明・切り分けできる
- [ ] 有料リソースを学習目的に作っていない

## トラブル時

| 症状 | 対処 |
|------|------|
| Minikube 不調 | delete → start → ホームアプリ再適用 |
| Sandbox 期限切れ | 再申請。Minikube + Docs で継続 |
| 時間なし | 健全性 + ドリル 1 つだけでもログに残す |

戻る: [リポジトリ README](../README.md)
