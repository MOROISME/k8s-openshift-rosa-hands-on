# 無料枠の方針

この学習リポジトリでは、**課金が発生しうる操作を手順に含めません。**

## 使ってよい環境

| 環境 | 料金感 | 用途 |
|------|--------|------|
| ローカル PC / WSL | $0 | Linux・Docker・Minikube |
| Docker Desktop（個人）または Colima / Podman | $0※ | コンテナ |
| Minikube | $0 | Kubernetes 実習・個人運用 |
| [Developer Sandbox](https://developers.redhat.com/developer-sandbox) | $0（時間・利用上限あり） | OpenShift / `oc` / Route |
| AWS アカウント（無料利用枠） | 条件付き $0 | **閲覧中心**。作成は「料金 $0 が確認できる操作」のみ |
| 公式ドキュメント | $0 | ROSA・AWS・OpenShift の概念 |

※ Docker Desktop の商用利用条件は Docker の規約を確認。個人学習なら Colima でも可。

## 禁止（このリポジトリの手順ではやらない）

| 操作 | 理由 |
|------|------|
| ROSA クラスタ作成 | マネージド OpenShift は有料 |
| EKS / AKS / GKE 作成 | 有料になりやすい |
| NAT Gateway / ALB / NLB の作成 | 時間課金 |
| 常時起動の EC2 / RDS | 無料枠超過・消し忘れリスク |
| 有料の OpenShift クラウドクラスタ | 有料 |
| 「よく分からないが作ってみる」クラウドのもの | 請求事故の元 |

## ROSA をどう学ぶか（無料の代替）

```text
やりたいこと              無料での代替
─────────────────────────────────────────
OpenShift 操作     →  Developer Sandbox
責任分界・構成理解 →  AWS/Red Hat Docs + 08 練習問題
個人の定常運用     →  Minikube + Sandbox の毎週の手順書（09）
実 ROSA 操作       →  会社の非本番（閲覧権限）があれば観察のみ
```

## AWS でやってよいこと / だめなこと

### やってよい（原則 $0）

- 管理画面（Console）で **既存の Default VPC / Subnet を見る**
- IAM のドキュメントを読む、**権限シミュレータ**を使う
- Billing の「無料利用枠」画面で請求 $0 を確認する習慣
- Cost Explorer / Bills で **請求が $0 であること**を定期確認

### やらない

- ROSA / ロードバランサー / NAT / Elastic IP の新規作成
- 「無料枠があるから」と EC2 を起動したまま放置
- 手順に無いサービスの作成

不安なら **AWS アカウントを作らず Docs のみ**でも Step 6 は完了できます。

## Developer Sandbox の注意

- 無料だが **有効期限・ものの利用上限** あり
- 期限切れ後は再申請や待機が必要な場合あり
- クラスタ管理者権限はほぼ無い → SCC 変更などは「読む」まで

## 請求事故を防ぐ習慣

1. クラウドで何かを作る前に「この手順に書いてあるか？」を確認
2. 書いていなければ作らない
3. AWS を触ったら Billing を見る（週 1 で十分）
4. ROSA 作成ボタンは押さない
