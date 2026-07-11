# 04. OpenShift の基本（Developer Sandbox）

## OpenShift とは

企業向けに強化された Kubernetes。  
Web コンソール、認証・認可、CI/CD 連携、イメージ管理、運用機能などがまとまっている。

| 用語 | 例え |
|------|------|
| Kubernetes | 店舗全体を管理する店長・エリアマネージャー |
| OpenShift | 大手チェーン店向けの本部システム付き運営パッケージ |

## 学習環境: Developer Sandbox

ローカルに OpenShift を建てなくても、ブラウザから無料で触れる学習用環境。

1. https://developers.redhat.com/developer-sandbox にアクセス
2. Red Hat アカウントでログイン
3. Sandbox を起動し、Web Console を開く

CLI（任意）:

```bash
# oc をインストールしたうえで、Console の「Copy login command」からログイン
oc whoami
oc project
```

## 学ぶこと（チェックしながら進める）

### A. Kubernetes との違い（概念）

- [ ] Project ≈ Namespace（＋権限などの寄せ集め）
- [ ] Route ≈ Ingress（OpenShift 流の外部公開）
- [ ] ImageStream / BuildConfig など「イメージとビルド」の仕組みがある
- [ ] DeploymentConfig はレガシー寄り。新しい環境では Deployment もよく使う

### B. Web Console ハンズオン

Console 上で次を体験する:

1. 新しい Project（または既存）を確認
2. カタログまたは YAML からサンプルアプリをデプロイ
3. Route の URL を開いてアクセス
4. Pod のログを Console から見る

### C. `oc` ハンズオン（任意）

```bash
oc get project
oc get pods
oc get svc
oc get route
oc logs deploy/<name>
oc describe route/<name>
```

## OpenShift で追加で覚えたい用語

| 用語 | ざっくり |
|------|-----------|
| Project | 作業単位（Namespace + 権限など） |
| Route | 外部公開（ホスト名付き） |
| BuildConfig | ソースからイメージを作る定義 |
| ImageStream | イメージのタグ・参照を管理 |
| Operator | 運用知識を自動化する拡張 |
| Web Console | GUI |

## 参考

- [Red Hat OpenShift on AWS Learn](https://www.redhat.com/en/technologies/cloud-computing/openshift/aws/learn)
- Developer Sandbox: https://developers.redhat.com/developer-sandbox

## 章のゴール

- [ ] OpenShift = 「企業向け Kubernetes プラットフォーム」と説明できる
- [ ] Console でアプリを 1 つ動かせた
- [ ] Route で外部 URL を確認できた

次: [05-rosa](../05-rosa/)
