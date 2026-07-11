# 08. 現場シナリオ（無料環境で演習）

思考手順を固めます。実機は **Minikube** と（あれば）**Developer Sandbox** のみ。ROSA は作りません。

## ゴール

- [ ] 障害を 3 層（アプリ / OpenShift / AWS）で仮説立てできる
- [ ] 最初の 15 分で見る順を言える
- [ ] 「自分で追う / 渡す」判断ができる

## 使い方

1. シナリオを読む
2. 何も見ずに手順を書く
3. 模範と照合
4. 可能なら無料環境で近い操作を行う

---

## シナリオ 1: URL が 503 / タイムアウト

### 無料での実機対応

| 環境 | やること |
|------|----------|
| Minikube | Service selector を壊す → Endpoints 空を確認 → 直す（Step 4 C） |
| Sandbox | Route → Service → Pod を Console / `oc` で辿る |

### 模範

```text
1. アプリ: Route/Ingress → Service → Endpoints → Pod Ready
2. 基盤: Router / Ingress Controller、他アプリは生きているか
3. AWS（概念）: LB / DNS / SG（実機作成はしない）
```

---

## シナリオ 2: CrashLoopBackOff

無料実機: Step 4 シナリオ B を再実行。

```text
describe → Events → logs (--previous) → 設定/イメージ/Probe/SCC
```

---

## シナリオ 3: 複数 NS で ImagePullBackOff

無料実機: Step 4 シナリオ A。  
「横断的ならレジストリ/権限/ネットワーク」は **Docs 上の仮説**として書く（ECR は作らない）。

---

## シナリオ 4: Forbidden

無料実機: Step 4 の `kubectl auth can-i`、Sandbox の `oc auth can-i`。

```text
誰の資格情報か → can-i → RoleBinding →（OpenShift）SCC
```

---

## シナリオ 5: API が重い / クラスタ異常

無料実機の代替:

```bash
# Minikube
kubectl get nodes
kubectl get --raw='/readyz?verbose' 2>/dev/null || true

# Sandbox（権限があれば）
oc get co 2>/dev/null || echo "co may be restricted on Sandbox"
```

AWS リージョン障害は Status ページを**読むだけ**。

---

## 責任分界チートシート

| 観察 | 寄せやすい層 |
|------|----------------|
| 単一アプリの例外ログ | アプリ |
| Endpoints 空 | アプリ（マニフェスト） |
| 多数 NS で ImagePull | レジストリ / 権限 / ネットワーク |
| Router 全体死 | OpenShift |
| `co` Degradation | OpenShift / ROSA |
| LB・DNS・VPC | AWS（観察・Docs） |

## 完了条件（DoD）

- [ ] 5 シナリオで「最初の 3 手」を言える
- [ ] 3 層切り分けを 1 分で説明できる
- [ ] 有料リソースを作っていない

次: [09-personal-ops](../09-personal-ops/)
