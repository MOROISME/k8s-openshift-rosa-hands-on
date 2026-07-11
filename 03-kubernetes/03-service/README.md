# 03. Service

Service は、入れ替わる Pod への**安定したアクセス口**です。  
**この節の目的:** Pod IP が変わっても届く仕組みを、実際に curl で確認する。

## ハンズオン

前提: Deployment があること（なければ再適用）。

```bash
kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml

kubectl get svc hello-svc
minikube service hello-svc --url
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f .../03-service.yaml` | Service 作成 | Pod 群への固定入口を作る |
| `kubectl get svc` | Service 一覧 | ClusterIP / ポート等を確認 |
| `minikube service NAME --url` | Minikube 用のアクセス URL を表示 | ローカルから届くアドレスを知る（Minikube 専用の便利コマンド） |

```bash
curl "$(minikube service hello-svc --url)"
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `curl "$(...)"` | 表示された URL に HTTP アクセス | Service 経由でアプリに届くことを確認 |

任意の確認:

```bash
kubectl get endpoints hello-svc
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl get endpoints` | Service が指す実体（Pod IP） | selector が合っていれば IP が並ぶ。空ならラベル不一致 |

片付け:

```bash
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

## 期待結果

- `hello-svc` がある
- `curl` が HTML 等を返す
- Endpoints に Pod IP がある

## ポイント

- Pod IP は作り直すと変わる → Service がその差分を吸収
- 種類: ClusterIP / NodePort / LoadBalancer など

## トラブル時

| 症状 | 対処 |
|------|------|
| curl 失敗 | Endpoints が空か。ラベル不一致を疑う |
| URL が出ない | `kubectl get svc` と `minikube status` |

## 完了条件（DoD）

- [ ] Service 経由で到達できた
- [ ] 「なぜ Service が必要か」「Endpoints を見る目的」を説明できる
