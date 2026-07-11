# 公式 nginx（Alpine 版）を土台にする
# → Web サーバ一式が入った状態からスタートできる
FROM nginx:alpine

# 同じフォルダの index.html を、nginx が公開する場所へコピーする
# → ブラウザ / curl でアクセスしたときに、自分の HTML が返る
# コンテナ内の既定ドキュメントルート（nginx 公式イメージの慣習）:
#   /usr/share/nginx/html/
COPY index.html /usr/share/nginx/html/index.html
