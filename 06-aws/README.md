# 06. AWS 基礎（無料・観察のみ）

**この Step の目的:** ROSA / クラウド OpenShift の土台になる AWS 部品の**役割**を理解する。  
**課金対象の資源は作成しない。** コマンドより「何を見るか・なぜ見るか」が中心。

方針: [docs/free-tier.md](../docs/free-tier.md)

## やること / やらないこと

| やる（$0） | 目的 | やらない |
|------------|------|----------|
| Docs を読む | 部品の定義を公式で押さえる | ROSA 作成 |
| Default VPC を眺める | 「計算機のまとまり（クラスタ）が住む箱」の実物イメージ | NAT / ALB 作成 |
| Billing 確認 | 学習で課金していないか守る | 常時 EC2 |
| IAM 概念を読む | 権限まわりの問題の土台 | 手順外の作成 |

アカウントが無い場合は **Docs のみ**で DoD 達成可。

## 1. Docs（必須）— なぜ読むか

| URL | 読む目的 |
|-----|----------|
| [AWS ROSA](https://aws.amazon.com/jp/rosa/) | 製品が何か・何と一体で動くかを掴む |
| [ROSA User Guide](https://docs.aws.amazon.com/rosa/latest/userguide/what-is-rosa.html) | 責任範囲・構成の公式説明 |
| （任意）VPC / IAM 概要 | ネットワークと権限の語彙を入れる |

メモ表（自分用に埋める）:

| AWS 部品 | 自分の言葉 | なぜ OpenShift と関係するか |
|----------|------------|------------------------------|
| VPC | | ノードが住むネットワーク |
| Subnet | | 配置場所 |
| IAM / STS | | 誰が何をできるか・一時認証 |
| Load Balancer | | API / Route の入口の裏 |
| Route 53 | | 名前解決 |
| CloudWatch | | 監視・ログ |

## 2. 管理画面での観察（任意）— 画面操作の目的

| 見る場所 | 目的 | やってはいけないこと |
|----------|------|----------------------|
| Billing | 請求 $0（学習由来）か確認 | — |
| VPC → VPCs / Subnets | 「箱」と区画の実物を見る | 新規 VPC 作成（不要） |
| EC2 → Load Balancers | LB という入口の存在を知る | Create load balancer |
| IAM → Roles | ロール＝権限の束、とイメージ | 不要なロール乱造 |

### 禁止（目的: 請求事故防止）

- Create cluster（ROSA）
- Create load balancer / NAT gateway
- Launch instance（この教材では不要）

## 3. 期待結果

- メモ表が埋まっている
- 部品の役割を説明できる
- 学習由来の新規課金が無い

## 完了条件（DoD）

- [ ] 各 AWS 部品を「なぜ見るか」付きで説明できる
- [ ] 有料のものを作っていない

次: [07-rosa](../07-rosa/)
