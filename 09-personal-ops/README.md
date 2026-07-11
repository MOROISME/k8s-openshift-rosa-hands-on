# 09. 個人運用ランブック（無料・週次）

L4 ゴール: **Minikube + Developer Sandbox を、自分の手で毎週回せる。**

ROSA 本番構築は含みません（有料）。代わりに「自分がオーナーの検証基盤」を無料で持ちます。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## 個人環境の構成（すべて無料）

```text
[毎週使う]
  Minikube          … 障害演習・デプロイ練習のホーム
  Developer Sandbox … OpenShift / Route / oc の感覚維持（期限に注意）

[読んだり確認するだけ]
  AWS/Red Hat Docs  … ROSA 知識のメンテ
  AWS Billing       … 口座がある人だけ。常に $0 確認
```

## 初回セットアップ（1 回だけ）

### Minikube ホームアプリ

```bash
minikube start
kubectl apply -f ../03-kubernetes/manifests/02-deployment.yaml
kubectl apply -f ../03-kubernetes/manifests/03-service.yaml
kubectl get deploy,pods,svc
curl "$(minikube service hello-svc --url)"
```

期待: Deployment Ready、curl 成功。

### Sandbox ホームアプリ

Step 5 の YAML を再適用:

```bash
oc apply -f ../05-openshift/manifests/app-deployment.yaml
oc apply -f ../05-openshift/manifests/app-service.yaml
oc apply -f ../05-openshift/manifests/app-route.yaml
oc get route sandbox-web
```

期待: Route に到達できる。

### 運用ログ

```bash
cp templates/ops-log.md ./ops-log-local.md
# ops-log-local.md は個人メモ（gitignore 済み想定でも、秘密は書かない）
```

---

## 週次ランブック（30〜60 分）

毎週同じ順で実施。結果を `ops-log-local.md` に 5 行で残す。

### 1. 健全性（10 分）

```bash
minikube status
kubectl get nodes
kubectl get deploy,pods,svc
curl -I "$(minikube service hello-svc --url)"
```

Sandbox が生きていれば:

```bash
oc whoami
oc get pods
oc get route
```

期待: すべて Ready / 到達可。Sandbox 期限切れなら「期限切れ」とログに書き、Minikube のみ継続。

### 2. 障害ドリル（15 分）

次から **1 つだけ** 選んで実施（Step 4 の手順）:

- ImagePullBackOff
- CrashLoopBackOff
- Service ラベル不一致

必ず: 壊す → 観察 → 直す → 片付け。

### 3. OpenShift 感覚（10 分・Sandbox がある週）

- Route URL を開く
- Pod ログを見る
- `oc auth can-i get pods` を打つ

### 4. 知識メンテ（10 分）

- ROSA Docs を 1 セクション読む、または
- Step 8 シナリオを 1 つ、何も見ずに手順を書き出す

### 5. 請求ガード（5 分・AWS アカウントがある人のみ）

- Billing を開き、学習由来の課金が **$0** か確認
- ROSA / LB / NAT を作っていないか思い出す

---

## 月次（任意・30 分）

- [ ] Minikube を `minikube delete && minikube start` で作り直し、ホームアプリを再デプロイ（再現性確認）
- [ ] Sandbox の有効期限を確認
- [ ] L4 自己評価（下記）を見直す

---

## L4 自己評価（完了条件）

次をすべて満たしたら「個人で運用できる（無料検証基盤）」到達です。

- [ ] 週次ランブックを **連続 3 週** 実行し、ログが残っている
- [ ] 障害ドリル 3 種を、手順を見なくても大枠できる
- [ ] Sandbox で Route 公開を一人でやり直せる（または期限切れ時の代替方針がある）
- [ ] ROSA を「作らずに」説明・切り分けできる
- [ ] クラウドで有料リソースを学習目的に作っていない

---

## トラブル時

| 症状 | 対処 |
|------|------|
| Minikube 不調 | `minikube delete` → `minikube start` → ホームアプリ再適用 |
| Sandbox 期限切れ | 再申請。その間は Minikube + Docs で週次を継続 |
| 時間がない週 | 「健全性 10 分 + ドリル 1 つ」だけでもログに残す |

戻る: [リポジトリ README](../README.md)
