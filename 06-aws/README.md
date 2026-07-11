# 06. AWS 基礎（無料・観察のみ）

ROSA の土台理解用です。**課金リソースは作成しません。**

必須ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## この Step でやること / やらないこと

| やる（$0） | やらない（課金リスク） |
|------------|------------------------|
| Docs を読む | ROSA クラスタ作成 |
| Console で Default VPC を眺める | NAT / ALB / NLB 作成 |
| Billing が $0 か確認 | 常時 EC2 起動 |
| IAM の概念を読む | 手順外のサービス作成 |

AWS アカウントが無い場合は **Docs のみ**で DoD 達成可。

## 1. Docs で全体像（必須）

読む:

- https://aws.amazon.com/jp/rosa/
- https://docs.aws.amazon.com/rosa/latest/userguide/what-is-rosa.html
- （任意）VPC / IAM の Getting Started 概要ページ

メモ表（自分用）:

| AWS 部品 | 自分の言葉 | OpenShift/ROSA との接点 |
|----------|------------|-------------------------|
| VPC | | ノードが住むネットワーク |
| Subnet | | 配置場所 |
| IAM / STS | | 権限・一時認証 |
| Load Balancer | | API / Route の入口 |
| Route 53 | | 名前解決 |
| CloudWatch | | 監視 |

## 2. Console 観察（任意・作成しない）

アカウントがある場合のみ。

1. Billing → 請求が **$0**（または学習と無関係な既存のみ）か確認
2. VPC → Your VPCs / Subnets を**見るだけ**
3. EC2 → Load Balancers を**見るだけ**（作らない）
4. IAM → Roles を**見るだけ**

### 禁止クリック

- Create cluster（ROSA）
- Create load balancer
- Create NAT gateway
- Launch instance（この教材では不要）

## 3. 期待結果

- 上記メモ表が埋まっている
- 「ROSA を作らなくても部品の役割は説明できる」
- （アカウントあり）今週の請求に学習由来の新規課金が無い

## トラブル時

| 症状 | 対処 |
|------|------|
| 請求が怖い | アカウントを触らず Docs のみで完了にする |
| 画面に Create ボタンが多い | この README の「やらない」以外は押さない |

## 完了条件（DoD）

- [ ] VPC / IAM / LB / DNS / CloudWatch を OpenShift と対応づけて一言で書ける
- [ ] ROSA・LB・NAT・常時 EC2 を**作っていない**
- [ ] 無料ポリシーを説明できる

次: [07-rosa](../07-rosa/)
