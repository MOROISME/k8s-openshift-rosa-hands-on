# 04. Kubernetes 運用入門（壊して直す）

**この章の目的:** 正常系だけでなく、障害時に「何を・どの順で見るか」を体に入れる。  
無料の Minikube だけで完結します。

## 前提

- [03-kubernetes](../03-kubernetes/) 完了
- `minikube start` 済み、`kubectl get nodes` で Ready

## 学習順

1. [01-troubleshooting](./01-troubleshooting/) … 典型障害 3 種
2. [02-probes-resources](./02-probes-resources/) … 健康診断とリソース
3. [03-storage-rbac](./03-storage-rbac/) … ディスクと権限

マニフェスト: [manifests/](./manifests/)

## 切り分けの基本手順（暗記）

| 順 | コマンド | 目的 |
|----|----------|------|
| 1 | `kubectl get pods` | どの Pod が怪しいか一覧で掴む |
| 2 | `kubectl describe pod <name>` | Events で「クラスタが何と言っているか」を読む |
| 3 | `kubectl logs <name>` | アプリ自身の出力を見る |
| 4 | `kubectl get events --sort-by='.lastTimestamp'` | 時系列で周辺イベントを追う |
| 5 | Service / ラベル / ポート | ネットワーク経路の不一致を疑う |

## NetworkPolicy（概要のみ）

**NetworkPolicy = Pod 間のファイアウォール。** 概念理解で十分（DoD に実機必須なし）。

## 完了条件（DoD）

- [ ] 3 種の障害を区別し、各確認コマンドの目的を説明できる
- [ ] Probe / resources / PVC / RBAC の目的を説明できる

次: [05-openshift](../05-openshift/)
