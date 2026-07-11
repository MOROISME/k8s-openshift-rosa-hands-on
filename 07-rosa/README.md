# 07. ROSA の基本（Docs のみ・クラスタ作成禁止）

**この Step の目的:** ROSA が何か・誰が何を責任持つかを説明できるようにする。  
**クラスタは作らない（有料）。** OpenShift 操作は Step 5 で済み。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

| 学びたいこと | 無料でのやり方 | 目的 |
|--------------|----------------|------|
| OpenShift 操作 | Step 5 Sandbox | 手を動かす部分は済んでいる |
| ROSA とは | 本 Step Docs | マネージドサービスの位置づけ |
| 責任分界 | 本 Step + Step 8 | 障害時にどこを疑うか |
| 実クラスタ | 会社非本番の閲覧のみ（任意） | 実物の雰囲気（破壊禁止） |

## ROSA とは

**Red Hat OpenShift Service on AWS** = AWS 上のマネージド OpenShift。

```text
Kubernetes → OpenShift → ROSA（AWS 上マネージド）
```

## 1. 責任分界 — なぜ学ぶか

障害対応で「自分で直す / ベンダーや基盤に渡す」判断の軸になる。

| 層 | 見るもの（例） | 見る目的 |
|----|----------------|----------|
| アプリ | Pod、ログ、設定、Route 応答 | 自分たちのデプロイ起因か |
| OpenShift / ROSA | `oc get co`、Operator、ノード、SCC | クラスタ基盤の劣化か |
| AWS | VPC、IAM、LB、DNS | クラウド土台の問題か |

## 2. Classic と HCP — なぜ区別するか

現場のクラスタ種別で「何が見えるか・課金モデル」が違うため。

| 種別 | ざっくり |
|------|-----------|
| Classic | 従来型 |
| HCP | コントロールプレーンをホストする構成 |

読む目的: 公式の定義を自分の言葉に落とす（作成はしない）。

- https://aws.amazon.com/jp/rosa/
- https://docs.aws.amazon.com/rosa/latest/userguide/what-is-rosa.html
- https://www.redhat.com/en/technologies/cloud-computing/openshift/aws/learn

## 3. Sandbox との対応 — なぜやるか

「ROSA を作れなくても、触った OpenShift 操作が本番 ROSA でも同じ道具」と接続するため。

| Sandbox | ROSA での意味 |
|---------|----------------|
| Project / Deploy / Route | アプリ層。同じ |
| `oc` / Console | 接続先が ROSA になるだけ |
| SCC | 本番でも（より）重要 |
| Operator | `co` とセットで健全性 |

## 4. 任意: 会社クラスタ閲覧

**目的:** 実クラスタの見え方を知る。変更はしない。

```bash
oc whoami
oc get nodes
oc get co
oc get route -A | head
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc whoami` | 誰で入っているか | 権限・監査の起点 |
| `oc get nodes` | ワーカー等の状態 | ノード障害の一次確認 |
| `oc get co` | Cluster Operator 一覧 | クラスタ機能の健全性（ROSA/OCP 運用の定番） |
| `oc get route -A \| head` | 全 NS の Route の先頭 | 公開面の全体感。`-A` は全 Namespace。`head` で出力抑制 |

**delete / edit / scale はしない。**

## 完了条件（DoD）

- [ ] ROSA と責任分界・Classic/HCP を説明できる
- [ ] （任意コマンドを打った場合）各コマンドの目的を説明できる
- [ ] クラスタを作っていない

次: [08-field-ops](../08-field-ops/)
