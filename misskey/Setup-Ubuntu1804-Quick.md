
# Misskey セットアップ Ubuntu 18.04 簡単編

## 前提

- Ubuntu 18.04 (16.04 でも CentOSでもない)  
  (これ前提で外部レポジトリは使用してません)  
  この手順ではディストリビューション付属のNode.jsとMongoDBを使用します。  
  最新のバージョンを使用したい場合は [Ubuntu/Debian 編](Setup-Ubuntu_Debian.md) から参照してください。
- サーバーはファイアウォールなどで保護されていて、22,80,443など以外は開いてない  
  (これ前提で認証設定は端折ってます)

## 手順

### Ubuntu 18.04 を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- `物理メモリ2GB` or `物理メモリ1GB + スワップ1GB` くらいあるとよい

### 管理者ユーザーで以下を実行
```sh
# Misskey用ユーザー作成
sudo adduser --disabled-password --disabled-login misskey

# 必要パッケージインストール
sudo apt -y install nodejs npm redis mongodb git build-essential nginx ssl-cert letsencrypt

# misskeyユーザーに変更
sudo su - misskey

```

### misskeyユーザーで以下を実行
```sh
# リポジトリクローン
git clone -b master git://github.com/syuilo/misskey.git

# ディレクトリ移動
cd ~/misskey

# 最新リリースをチェックアウト
git checkout $(git tag -l | grep -v 'rc[0-9]*$' | sort -V | tail -n 1)

# コンフィグコピー
cp .config/example.yml .config/default.yml

# コンフィグ編集(vimじゃなくてnanoとかでもいい)
vim .config/default.yml

```

コンフィグ編集はとくに以下を変更する

```yml
# urlはクライアントから見えるURLに変更する
#   example.com は 公開するホスト名に置き換える
#   httpsです
url: https://example.com/

# ポートは1024以上の任意のポートに置き換える
port: 3000

# mongodb 今回は認証をしないのでuser,passはコメントアウトか削除する
mongodb:
  host: localhost
  port: 27017
  db: misskey

# redis 今回は認証をしないので(というか対応してない)passはコメントアウトか削除する
redis:
  host: localhost
  port: 6379
```

### 編集が終わったら、引き続きmisskeyユーザーでインストールとビルドを行う
```sh
cd ~/misskey

# パッケージインストール
npm install

# ビルド
NODE_ENV=production npm run build

# ログアウトして管理者ユーザに戻る
exit

```

### 管理者ユーザでNginxとSSL証明書を設定

`/etc/nginx/sites-enabled/misskey.nginx` に以下の内容を保存する  
`misskey.example.com` の部分は使用するホスト名に置き換える  
ここでは一旦自己署名証明書を設定している
```nginx
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache1:16m max_size=1g inactive=720m use_temp_path=off;

server {
    listen 80;
    listen [::]:80;
    server_name misskey.example.com;

    # For SSL domain validation
    root /var/www/html;
    location /.well-known/acme-challenge/ { allow all; }
    location /.well-known/pki-validation/ { allow all; }
    location / { return 301 https://$server_name$request_uri; }
}

server {
    listen 443 http2;
    listen [::]:443 http2;
    server_name misskey.example.com;
    ssl on;
    ssl_session_timeout 5m;

    # self-signed certificate
    ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;

    # letsencrypt certificate
    #ssl_certificate           /etc/letsencrypt/live/misskey.example.com/fullchain.pem;
    #ssl_certificate_key       /etc/letsencrypt/live/misskey.example.com/privkey.pem;

    # SSL protocol
    ssl_protocols TLSv1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:AES128-SHA;
    ssl_prefer_server_ciphers on;

    # これが実質アップロード時の1ファイルあたりのサイズ制限になる
    client_max_body_size 16m;

    # Proxy to Node
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Proxy "";
        proxy_buffering off;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }

    # Proxy to Node with caching
    location ~ /(assets/|files/|url) {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Proxy "";
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        # Cache settings
        proxy_cache cache1;
        proxy_cache_lock on;
        proxy_cache_use_stale updating;
        add_header X-Cache $upstream_cache_status;
    }
}
```

Nginxコンフィグを編集したら、管理者ユーザで以下を実行する
```sh
# Nginxをリロード
sudo service nginx reload

# Let's Encryptで証明書取得(misskey.example.comの部分は使用するホスト名に置き換える)
#   メールアドレス入力したりAgreeしたりする
#   Congratulations! とかでたらOK
sudo letsencrypt certonly --webroot -d misskey.example.com -w /var/www/html/

```

再度 `/etc/nginx/sites-enabled/misskey.nginx` を編集して  
`# self-signed certificate` 下の2行をコメントアウトして、  
`# letsencrypt certificate` 下の2行のコメントアウトを解除する

引き続き
```sh
# Nginxをリロード
sudo service nginx reload
```

```sh
# 管理者 から misskeyユーザーに変更する
sudo su - misskey
```

### misskeyユーザーででMisskeyを起動してみる

```
NODE_ENV=production npm start
```

Webブラウザでアクセスして確認する

### この後

デーモンで起動するように設定する  
参考: https://github.com/syuilo/misskey/blob/master/docs/setup.ja.md#systemd%E3%82%92%E7%94%A8%E3%81%84%E3%81%9F%E8%B5%B7%E5%8B%95 

Let's Encryptの自動更新を設定する

管理者ユーザーを設定する
```sh
# misskeyユーザーで
cd ~/misskey
node cli/mark-admin.js @ユーザーID
```
