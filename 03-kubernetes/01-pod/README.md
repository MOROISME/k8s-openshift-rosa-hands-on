# 01. Pod

Pod は Kubernetes でコンテナを動かす**最小単位**です。  
**この節の目的:** 「クラスタの上でコンテナが 1 個 Running になる」感覚と、観察コマンドを身につける。

## 前提

先に [../README.md](../README.md) の **Minikube インストール〜 `kubectl get nodes` が Ready** まで完了していること。

```bash
kubectl get nodes
# minikube   Ready   ...
```

## ハンズオン

**作業ディレクトリ:** 先に `03-kubernetes/01-pod` へ移動する（`02-docker/exercises` など別フォルダでは相対パスが解決しない）。

```bash
cd 03-kubernetes/01-pod
# リポジトリ直下からの例:
# cd /Users/あなた/study/k8s-openshift-rosa-hands-on/03-kubernetes/01-pod

kubectl apply -f ../manifests/01-pod.yaml
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod
```

リポジトリのルートから打つ場合:

```bash
kubectl apply -f 03-kubernetes/manifests/01-pod.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl apply -f ...` | YAML の内容をクラスタに反映 | Pod を作成する |
| `kubectl get pods` | Pod 一覧 | STATUS / READY を一次確認 |
| `kubectl describe pod NAME` | 詳細と Events | 「なぜその状態か」を読む（障害の入口） |
| `kubectl logs NAME` | コンテナ標準出力 | アプリが出したログを見る |

任意（コンテナの中に入る）:

```bash
kubectl exec -it hello-pod -- /bin/sh
# exit で抜ける
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl exec -it POD -- /bin/sh` | Pod 内でシェル起動 | 中のファイル・プロセスを Linux 同様に確認（`-it` は対話用） |

片付け:

```bash
kubectl delete -f ../manifests/01-pod.yaml
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl delete -f ...` | YAML で定義したリソースを削除 | 学習用リソースを残さない |

## 期待結果

```text
NAME        READY   STATUS    RESTARTS   AGE
hello-pod   1/1     Running   0          ...
```

- `describe` の Events に失敗が並んでいない
- `logs` が取れる（nginx はアクセスが無いと静かでもよい）

## ポイント

- 本番で Pod を直接作り続けることは少ない（Deployment 経由が基本）
- まずは「1 個動いている」を掴む

## トラブル時

| 症状 | 対処 |
|------|------|
| `ImagePullBackOff` | ネット確認。`describe` の Events を読む |
| `Pending` | `minikube status`。ノード Ready か |

## 完了条件（DoD）

- [ ] `Running` / `1/1` を確認した
- [ ] `get` / `describe` / `logs` それぞれの目的を説明できる
