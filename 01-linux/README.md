# 01. Linux の基本

Kubernetes / OpenShift は Linux 上でコンテナを動かす前提です。  
ここでは **パス操作 + 疎通確認** までを無料のローカル環境で身につけます。

## 前提

- macOS ターミナル、または WSL / Linux（すべて無料）
- 追加クラウド不要

## ハンズオン

```bash
pwd
ls -la

echo "hello k8s" > /tmp/k8s-learn.txt
cat /tmp/k8s-learn.txt
grep k8s /tmp/k8s-learn.txt

ps aux | head
whoami
id
```

### 疎通

```bash
curl -I https://example.com
```

任意（入っていれば）:

```bash
# dig example.com +short
# nslookup example.com
```

### よく使うコマンド

| コマンド | 用途 |
|----------|------|
| `pwd` / `cd` / `ls` | 移動・一覧 |
| `cat` / `less` / `grep` | 確認・検索 |
| `ps` / `top` | プロセス |
| `curl` | HTTP |
| `dig` / `nslookup` | DNS |
| `ss` / `lsof` | ポート |

## 期待結果

| 操作 | 成功の目安 |
|------|------------|
| `pwd` | `/Users/...` や `/home/...` などパスが表示される |
| `cat /tmp/k8s-learn.txt` | `hello k8s` |
| `curl -I https://example.com` | `HTTP/2 200` または `HTTP/1.1 200` など |

## トラブル時

| 症状 | 対処 |
|------|------|
| `curl: command not found` | macOS は通常あり。WSL なら `sudo apt update && sudo apt install -y curl` |
| 社内プロキシで curl 失敗 | 別ネットワーク、または後続 Step で再試行 |

## 完了条件（DoD）

- [ ] ディレクトリ移動とファイル確認ができる
- [ ] `curl` で HTTP 成否を確認できる
- [ ] 「コンテナは Linux プロセスに近い」とイメージできる

次: [02-docker](../02-docker/)
