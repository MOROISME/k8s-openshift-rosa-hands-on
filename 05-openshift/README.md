# 05. OpenShift の基本（Developer Sandbox・無料）

**この Step の目的:** Kubernetes との差分（Project / Route / SCC / Operator）を、無料 Sandbox で体験する。  
有料の OpenShift / ROSA クラスタは使いません。

ポリシー: [docs/free-tier.md](../docs/free-tier.md)

## 前提

1. https://developers.redhat.com/developer-sandbox を開く  
2. Red Hat アカウントでログイン（無料）  
3. Sandbox を起動し Web Console を開く  
4. （推奨）Console の **Copy login command** でログインする  

```bash
oc whoami
oc project
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc whoami` | 今のユーザー | ログインできているか確認（`kubectl` の OpenShift 版クライアントが `oc`） |
| `oc project` | 現在の Project | どの仕切りで作業しているか確認（Namespace + 権限の寄せ集め） |

`oc` は公式無料バイナリで可。Console だけでも一部 DoD は達成可だが、YAML は `oc` 推奨。

## A. 概念チェック

- [ ] Project ≈ Namespace + 権限
- [ ] Route ≈ Ingress
- [ ] DeploymentConfig はレガシー寄り
- [ ] SCC が Pod 権限を制限
- [ ] Operator が運用機能を担う

## B. Web Console ハンズオン

**目的:** GUI でも同じオブジェクト（Pod / Route / ログ）を追えるようにする。

1. Project を確認  
2. カタログまたは YAML でデプロイ  
3. Route URL を開く  
4. Pod ログを見る  

## C. YAML ハンズオン

**目的:** 制限付き SCC でも動く非特権アプリを、Deployment → Service → Route で公開する。

```bash
oc project

oc apply -f manifests/app-deployment.yaml
oc apply -f manifests/app-service.yaml
oc apply -f manifests/app-route.yaml

oc get pods
oc get svc
oc get route
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `oc project` | 現在 Project 表示/切替 | 間違った Project にデプロイしない |
| `oc apply -f ...deployment` | アプリ本体 | Pod を Deployment で管理 |
| `oc apply -f ...service` | クラスタ内入口 | Pod への安定アクセス |
| `oc apply -f ...route` | 外部 URL | OpenShift 流の公開（Ingress 相当） |
| `oc get pods/svc/route` | 各リソース確認 | Running と HOST を見る |

```bash
curl -I https://<route-host>
```

| コマンド | 目的 |
|----------|------|
| `curl -I https://...` | Route 経由で外から届くか確認 |

片付け:

```bash
oc delete -f manifests/app-route.yaml
oc delete -f manifests/app-service.yaml
oc delete -f manifests/app-deployment.yaml
```

| コマンド | 目的 |
|----------|------|
| `oc delete -f ...` | 依存の逆順でも可。学習用を残さない |

### 期待結果

- Pod `1/1 Running`
- Route に HOST が出る
- curl / ブラウザで 200 相当

## D. `oc` 基本（観察セット）

```bash
oc get project
oc get pods
oc get svc
oc get route
oc logs deploy/sandbox-web
oc describe route/sandbox-web
```

| コマンド | 目的 |
|----------|------|
| `oc get project` | 触れる Project 一覧 |
| `oc logs deploy/...` | アプリログ（K8s と同じ型） |
| `oc describe route/...` | Route の詳細・イベント |

## E. SCC / RBAC（観察）

**目的:** 権限不足の切り分け入口を知る（Sandbox では変更できないことが多い）。

```bash
oc auth can-i create deployment
oc auth can-i get scc --all-namespaces
oc get scc 2>/dev/null || echo "SCC list not permitted (expected on Sandbox)"
```

| コマンド | 目的 |
|----------|------|
| `oc auth can-i ...` | 自分にその API 操作が許されるか |
| `oc get scc` | SCC 一覧（権限があれば）。拒否されても「制限がある」と分かれば OK |
| `2>/dev/null \|\| echo ...` | エラーを握りつぶして学習用メッセージ | 権限不足を失敗扱いにしない |

- RBAC = API を叩けるか  
- SCC = Pod がホストに対してどこまでできるか  

## F. Operator（観察）

```bash
oc get csv -A 2>/dev/null || true
```

| コマンド | 目的 |
|----------|------|
| `oc get csv -A` | ClusterServiceVersion＝Operator の導入状態を見る（環境差あり） |
| `\|\| true` | 失敗してもシェルを落とさない | Sandbox 制限への備え |

**Operator = 運用手順のソフトウェア化。**

## トラブル時

| 症状 | 対処 |
|------|------|
| 期限切れ | 再申請。その間は Minikube |
| 権限系で Pod 失敗 | このリポジトリの非特権 YAML を使う |
| Route 無し | apply 漏れ・Service 名不一致 |
| login 失敗 | Console から login command 再コピー |

## 完了条件（DoD）

- [ ] アプリを 1 つ動かして Route で到達した
- [ ] 主要 `oc` コマンドの目的を説明できる
- [ ] 有料クラスタを作っていない

次: [06-aws](../06-aws/)
