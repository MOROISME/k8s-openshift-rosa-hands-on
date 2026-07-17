# 08. 現場の練習問題（無料環境で演習）

**この Step の目的:** 障害を見たとき「最初の 15 分で何をするか」を自分の言葉で言えるようにする。  
実機は Minikube / 練習用の無料環境（Sandbox）のみ。ROSA は作らない。

## ゴール

- [ ] 3 層（アプリ / OpenShift / AWS）で仮説立てできる
- [ ] 最初に打つコマンドの**目的**を言える
- [ ] 自分で追う / 渡す判断ができる

## 使い方

1. 練習問題を読む  
2. 何も見ずに手順を書く（コマンド名だけでなく「何のため」も）  
3. 模範と照合  
4. 可能なら無料環境で近い操作  

---

## 練習問題 1: URL が 503 / タイムアウト

**目的:** 公開 URL 障害を「経路を辿る」型で切る。

| 環境 | やること | 目的 |
|------|----------|------|
| Minikube | Step 4 C（探す条件 selector 壊し） | 届け先一覧（Endpoints）空＝経路切れを体感 |
| Sandbox | Route → Service → Pod を辿る | OpenShift 公開経路の確認順を体に入れる |

模範の見方:

| 層 | 何をするか | 目的 |
|----|------------|------|
| アプリ | Route/Ingress → Service → Endpoints → Pod Ready | 自分たちの公開設定か |
| 基盤 | Router 全体・他アプリ | クラスタ共通部品か |
| AWS（概念） | LB / DNS | クラウド入口か（作成はしない） |

---

## 練習問題 2: CrashLoopBackOff

**目的:** 単一 Pod 障害の定石順を固定する。

```text
describe → Events → logs (--previous) → 設定/イメージ/Probe/SCC
```

| 手順 | 目的 |
|------|------|
| describe / Events | クラスタ側の診断メッセージ |
| logs / `--previous` | 今・直前コンテナのアプリ出力（再起動後も理由を残す） |
| Probe / SCC | 「アプリバグ」以外の落とし穴 |

無料実機: Step 4 練習問題 B。

---

## 練習問題 3: 多数 NS で ImagePullBackOff

**目的:** 「1 アプリ」か「基盤・レジストリ横断」かを分ける。

無料実機: Step 4 A。ECR は作らず仮説として書く。

---

## 練習問題 4: Forbidden

**目的:** 権限エラーを can-i で再現・切り分ける。

```text
誰の資格情報か → can-i → RoleBinding →（OpenShift）SCC
```

無料実機: Step 4 の `kubectl auth can-i`、Sandbox の `oc auth can-i`。

---

## 練習問題 5: API が重い / クラスタ異常

**目的:** アプリ個別ではなく制御面の異常を疑う入口。

```bash
# Minikube
kubectl get nodes
kubectl get --raw='/readyz?verbose' 2>/dev/null || true

# Sandbox（権限があれば）
oc get co 2>/dev/null || echo "co may be restricted on Sandbox"
```

| コマンド | 目的 |
|----------|------|
| `kubectl get nodes` | ノード NotReady がないか |
| `kubectl get --raw='/readyz?verbose'` | API サーバ自身のヘルス（取れる環境のみ） |
| `oc get co` | OpenShift クラスタ Operator の劣化確認 |
| `2>/dev/null \|\| ...` | 権限・非対応環境でも学習を止めない |

AWS Status は**読むだけ**。

---

## 責任分界チートシート

| 観察 | 寄せやすい層 |
|------|----------------|
| 単一アプリの例外ログ | アプリ |
| Endpoints 空 | アプリ（YAML設定） |
| 多数 NS で ImagePull | レジストリ / 権限 / ネットワーク |
| Router 全体死 | OpenShift |
| `co` Degradation | OpenShift / ROSA |
| LB・DNS・VPC | AWS（観察・Docs） |

## 完了条件（DoD）

- [ ] 5 つの練習問題で「最初の 3 手」とその目的を言える
- [ ] 有料のものを作っていない

次: [09-personal-ops](../09-personal-ops/)
