# 00. 全体像

## 関係性（まずこれだけ覚える）

```text
Docker
  ↓  アプリをコンテナ化する技術

Kubernetes
  ↓  コンテナを複数サーバー上で管理する技術

OpenShift
  ↓  Kubernetes を企業向けに強化したプラットフォーム

ROSA
  ↓  AWS 上で使えるマネージド OpenShift
```

## 料理屋さんアナロジー

| 用語 | 例え |
|------|------|
| コンテナ化したアプリ | 完成済みの料理キット |
| サーバー | 店舗の厨房 |
| Kubernetes | 店舗全体を管理する店長・エリアマネージャー |
| OpenShift | 大手チェーン店向けの本部システム付き運営パッケージ |
| ROSA | その本部パッケージを AWS という土地で借りて運用する形態 |

## OpenShift の種類（ざっくり）

| 名前 | 意味 |
|------|------|
| Red Hat OpenShift | OpenShift 製品群全体の総称 |
| OpenShift Container Platform (OCP) | 自社 / クラウドに構築して使う代表的な OpenShift |
| OpenShift Dedicated | Red Hat が管理する専用 OpenShift |
| **ROSA** | AWS 上のマネージド OpenShift（現場で使うもの） |
| ARO | Azure 上のマネージド OpenShift |
| OpenShift Local | ローカル学習・開発用 |
| Developer Sandbox | ブラウザで試せる学習用環境 |

## この Step のゴール

- [ ] Docker / K8s / OpenShift / ROSA の上下関係を説明できる
- [ ] 「なぜ Minikube → Sandbox → ROSA Docs の順か」を説明できる

次: [01-linux](../01-linux/)
