# 03. Service

**この節の目的:** 「なぜ Service が必要か」を体感し、Service 経由でアプリに届くことを curl で確認する。

---

## Service を噛み砕くと

### 困りごと（Service が無いと）

Deployment で動いている Pod には、それぞれ一時的な IP があります。  
でも Pod は次の理由で **入れ替わります**。

- 落ちて作り直された
- `scale` で台数が変わった
- イメージ更新で新しい Pod に差し替わった

入れ替わると **IP も変わります**。  
「さっきの IP に curl」では、もう届かないことがあります。

```text
（悪い例）
あなた → Pod の IP（例: 10.244.0.7）に直接アクセス
              ↓ Pod が作り直される
         新しい IP（例: 10.244.0.12）
              ↓
         古い IP はもう使えない → 接続失敗
```

### Service は何をしてくれるか

**Service = 入れ替わる Pod 群への「固定の受付窓口」** です。

```text
（良い例）
あなた → Service（名前も入口も比較的安定）
              ↓ 自動で振り分け
         今生きている Pod たち
```

| たとえ | Kubernetes |
|--------|------------|
| お店の代表電話 | Service |
| その日いる店員さん | Pod（入れ替わる） |
| 「今いる店員」の名簿 | Endpoints |

店員（Pod）が交代しても、代表電話（Service）は同じ。  
電話をかければ、今いる店員につながる、というイメージです。

### どうやって「どの Pod？」を決めるか

Service は **ラベル** で対象を選びます（電話帳の条件検索に近い）。

この教材では:

| リソース | ラベル |
|----------|--------|
| Deployment が作る Pod | `app: hello` |
| Service の selector | `app: hello` |

両方の `app: hello` が一致しているので、Service は「hello の Pod」に届けます。  
ここがずれると、窓口はあるのに後ろに誰もいない（Endpoints が空）になります。

### ポートの数字（ざっくり）

マニフェスト例（`03-service.yaml`）:

```yaml
ports:
  - port: 80          # Service の受付ポート
    targetPort: 80    # 実際の Pod（コンテナ）側のポート
    nodePort: 30080   # （NodePort のとき）外からノード経由で触るポート
```

| 名前 | 噛み砕くと |
|------|------------|
| `port` | 受付窓口の番号 |
| `targetPort` | 奥の店員（コンテナ）が実際に聞いている番号 |
| `nodePort` | Minikube などから外から入りやすくするための番号（種類による） |

この演習の `type: NodePort` は、「学習用に外から届きやすくする」ための種類です。  
種類の詳細は後でで十分。まずは **「固定窓口がある」** ことが大事です。

### Endpoints とは

**Service が今つなごうとしている Pod IP の一覧**です。

- 載っている → 窓口の後ろに実体がある（届く見込み）
- 空 → ラベル不一致などで、誰にもつながっていない

障害のときは「Service があるか」だけでなく **Endpoints を見る**のが定石です。

---

## ハンズオン

**作業ディレクトリ:** `03-kubernetes/03-service`（またはリポジトリルートからパス指定）。

前提: Deployment があること（なければ再適用）。

```bash
cd 03-kubernetes/03-service

kubectl apply -f ../manifests/02-deployment.yaml
kubectl apply -f ../manifests/03-service.yaml

kubectl get svc hello-svc
minikube service hello-svc --url
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `apply` Deployment | アプリ（Pod 群）を用意 | Service の後ろに実体を置く |
| `apply` Service | 固定窓口を作る | Pod が入れ替わっても同じ入口で届くようにする |
| `kubectl get svc` | Service 一覧 | 窓口ができたか確認 |
| `minikube service NAME --url` | Minikube 用のアクセス URL | ローカルから叩く住所を知る（Minikube 専用の便利コマンド） |

```bash
curl "$(minikube service hello-svc --url)"
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `curl "$(...)"` | 表示 URL に HTTP アクセス | **Service 経由**でアプリに届くことを確認 |

任意（とても大事）:

```bash
kubectl get endpoints hello-svc
```

| コマンド | 意味 | 目的 |
|----------|------|------|
| `kubectl get endpoints` | 窓口の後ろの実体一覧 | Pod IP が並んでいれば接続先がある。空ならラベルを疑う |

片付け:

```bash
kubectl delete -f ../manifests/03-service.yaml
kubectl delete -f ../manifests/02-deployment.yaml
```

---

## 期待結果

- `hello-svc` がある
- `curl` で HTML などが返る
- `kubectl get endpoints hello-svc` に Pod IP が並ぶ

## ポイント（暗記用）

1. Pod IP は変わる → 直接 IP 頼みは危ない  
2. Service は固定の受付窓口  
3. selector（ラベル）で「どの Pod か」を決める  
4. 届かないときは Endpoints を見る  

## トラブル時

| 症状 | よくある原因 | 対処 |
|------|--------------|------|
| curl 失敗 | Endpoints が空（ラベル不一致） | `kubectl get endpoints` / Pod の label と Service の selector を照合 |
| URL が出ない | Service 未作成 / Minikube 未起動 | `kubectl get svc` / `minikube status` |

## 完了条件（DoD）

- [ ] Service を「入れ替わる Pod への固定窓口」と説明できる
- [ ] なぜ Pod IP 直打ちがまずいか言える
- [ ] Service 経由で curl できた
- [ ] Endpoints を見る目的を説明できる
