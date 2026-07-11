# 05. OpenShift の基本（Developer Sandbox・無料）

有料の OpenShift / ROSA クラスタは使いません。  
**[Developer Sandbox](https://developers.redhat.com/developer-sandbox) のみ**で完結します。

無料枠の注意: [docs/free-tier.md](../docs/free-tier.md)

## 前提

1. https://developers.redhat.com/developer-sandbox を開く
2. Red Hat アカウントでログイン（無料）
3. Sandbox を起動し Web Console を開く
4. （推奨）Console の **Copy login command** で `oc login` する

```bash
oc whoami
oc project
```

`oc` のインストールは公式の無料バイナリで可。Console だけでも DoD の一部は達成可能だが、YAML 適用は `oc` 推奨。

## A. 概念チェック

- [ ] Project ≈ Namespace + 権限
- [ ] Route ≈ Ingress
- [ ] DeploymentConfig はレガシー寄り。Deployment を使う
- [ ] SCC が Pod 権限を制限する
- [ ] Operator が運用機能を担う

## B. Web Console ハンズオン

1. 自分の Project を確認
2. カタログまたは YAML からアプリをデプロイ（次節の YAML 可）
3. Route の URL をブラウザで開く
4. Pod ログを Console で見る

## C. YAML ハンズオン（再現手順）

このリポジトリのマニフェストは **制限付き SCC 向け**（非特権・8080）です。

```bash
# 自分の Project にいることを確認
oc project

oc apply -f manifests/app-deployment.yaml
oc apply -f manifests/app-service.yaml
oc apply -f manifests/app-route.yaml

oc get pods
oc get svc
oc get route
```

Route の `HOST/PORT` をブラウザまたは curl で開く。

```bash
# HOST を控えて
curl -I https://<route-host>
```

片付け:

```bash
oc delete -f manifests/app-route.yaml
oc delete -f manifests/app-service.yaml
oc delete -f manifests/app-deployment.yaml
```

### 期待結果

```text
oc get pods
NAME                      READY   STATUS    RESTARTS   AGE
sandbox-web-...           1/1     Running   0          ...

oc get route
NAME           HOST/PORT                         ... 
sandbox-web    sandbox-web-....apps....openshiftapps.com
```

- curl / ブラウザで HTTP 200 相当
- 失敗時は `oc describe pod` / `oc get events` / `oc describe route`

## D. `oc` 基本

```bash
oc get project
oc get pods
oc get svc
oc get route
oc logs deploy/sandbox-web
oc describe route/sandbox-web
```

## E. SCC / RBAC（読む・調べる）

Sandbox では変更できないことが多い。**観察で十分（無料制約）。**

```bash
oc auth can-i create deployment
oc auth can-i get scc --all-namespaces
oc get scc 2>/dev/null || echo "SCC list not permitted (expected on Sandbox)"
```

覚えること:

- RBAC = API を叩けるか
- SCC = Pod がホストに対してどこまでできるか
- 動かない原因が SCC のこともある

## F. Operator（観察）

```bash
oc get csv -A 2>/dev/null || true
```

**Operator = 運用手順のソフトウェア化。** 一覧が見えなくても「そういう仕組みがある」と分かれば OK。

## トラブル時

| 症状 | 対処 |
|------|------|
| Sandbox 期限切れ | 再申請 / 待機。その間は Minikube で K8s 復習 |
| Pod が CreateContainerConfigError / 権限系 | このリポジトリの YAML（非特権）を使う。独自 YAML は SCC を疑う |
| Route が無い | `app-route.yaml` 適用漏れ。Service 名と一致しているか |
| `oc login` 失敗 | Console から login command を再コピー |

## 完了条件（DoD）

- [ ] Console または `oc` でアプリを 1 つ動かした
- [ ] Route URL で外部から到達できた
- [ ] Project / Route / SCC / Operator を一言で説明できる
- [ ] 有料クラスタを作っていない

次: [06-aws](../06-aws/)
