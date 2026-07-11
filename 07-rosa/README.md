# 07. ROSA の基本（Docs のみ・クラスタ作成禁止）

## 重要（無料枠）

**この Step では ROSA クラスタを作成しません。** 作成すると課金されます。

| 学びたいこと | 無料でのやり方 |
|--------------|----------------|
| OpenShift 操作 | Step 5 Developer Sandbox（済） |
| ROSA とは何か | 本 Step の Docs |
| 責任分界 | 本 Step + Step 8 |
| 実クラスタ操作 | 会社の非本番があれば**閲覧のみ**（任意） |

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## ROSA とは

**Red Hat OpenShift Service on AWS**。AWS 上のマネージド OpenShift。

```text
Kubernetes → OpenShift → ROSA（AWS 上マネージド）
```

## 1. 責任分界（必須）

| 層 | 見るもの（例） |
|----|----------------|
| アプリ | Pod、ログ、設定、イメージ、Route 応答 |
| OpenShift / ROSA | `oc get co`、Operator、ノード、SCC |
| AWS | VPC、IAM、LB、DNS、クォータ、リージョン障害 |

- [ ] マネージド＝コントロールプレーン運用の多くを Red Hat / サービス側が担う、と説明できる
- [ ] 顧客側に残る責任（アプリ、権限設計、ネットワーク接続など）を列挙できる

## 2. Classic と HCP（Docs）

| 種別 | ざっくり |
|------|-----------|
| ROSA Classic | 従来型 |
| ROSA HCP | コントロールプレーンをホストする構成 |

読む:

- https://aws.amazon.com/jp/rosa/
- https://docs.aws.amazon.com/rosa/latest/userguide/what-is-rosa.html
- https://www.redhat.com/en/technologies/cloud-computing/openshift/aws/learn

- [ ] 両者で「見え方・運用・課金」が違うことを一言で言える
- [ ] **どちらもこの教材では作らない**

## 3. Sandbox との対応づけ（無料の実践接続）

Sandbox でやったことを ROSA 脳に翻訳する。

| Sandbox で触ったこと | ROSA での位置づけ |
|----------------------|-------------------|
| Project / Deployment / Route | アプリ層。ROSA でも同様に触る |
| `oc` / Console | 同じ道具。接続先クラスタが ROSA になる |
| SCC / 権限の制限 | 本番でも同様。むしろ厳しい |
| Operator の存在 | ROSA ではクラスタ健全性（`co`）とセットで見る |

## 4. 任意: 会社クラスタの閲覧のみ

権限がある場合のみ（無料だが破壊禁止）:

```bash
oc whoami
oc get nodes
oc get co
oc get route -A | head
```

**delete / edit / スケール変更はしない。** 学習目的は観察。

## 期待結果

- 「ROSA = AWS 上マネージド OpenShift」を説明できる
- Classic / HCP の違いを一言で言える
- 3 層切り分けを説明できる
- 請求が発生する操作をしていない

## 完了条件（DoD）

- [ ] 上記「期待結果」を満たす
- [ ] ROSA 作成ウィザードを完了していない（開始も非推奨）

次: [08-field-ops](../08-field-ops/)
