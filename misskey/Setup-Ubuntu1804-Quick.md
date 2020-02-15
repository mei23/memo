
# Misskey v10 セットアップ Ubuntu 18.04 簡単編

## 前提

- Ubuntu 18.04
- サーバーはファイアウォールなどで保護されていて、22,80,443など以外は開いてない  
  (これ前提で認証設定は端折ってます)

## 手順

### Ubuntu 18.04 を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- `物理メモリ2GB` or `物理メモリ1GB + スワップ1GB` くらいあるとよい

### 管理者ユーザーで以下を実行

#### Misskey用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login misskey

```

#### Node.jsインストール  
参考: https://github.com/nodesource/distributions/blob/master/README.md
```sh
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs

```

### yarnインストール
参考: https://classic.yarnpkg.com/en/docs/install#debian-stable
```sh
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn

```

#### MongoDBインストール
参考: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
```sh
# 鍵インポート
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list

# インストール
sudo apt-get update
sudo apt-get install -y mongodb-org

# 開始する
sudo systemctl start mongod

# MongoDBを自動起動するようにする
sudo systemctl enable mongod

```

#### その他の必要パッケージをインストール
```sh
sudo apt -y install redis git build-essential nginx ssl-cert letsencrypt

```

#### misskeyユーザーに変更
```sh
sudo su - misskey

```

### misskeyユーザーで以下を実行
```sh
# リポジトリクローン
git clone -b mei-m544 https://github.com/mei23/misskey.git

# ディレクトリ移動
cd ~/misskey

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
yarn install

# ビルド
NODE_ENV=production yarn build

# ログアウトして管理者ユーザに戻る
exit

```

### nginx config
```sh
cp ~/misskey/docs/examples/misskey.nginx /etc/nginx/sites-enabled/
# 本当は sites-available にコピーして sites-enabled からシンボリックリンクが流儀

# example.tldをドメインで置き換える
vim /etc/nginx/sites-enabled/misskey.nginx

```

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
cd ~/misskey
NODE_ENV=production yarn start

# ※ デバッグログ等を参照したい場合は以下のコマンドで起動する
# yarn start
```

CloudFlareなんかでこのサーバーのアドレスを追加するか (デフォルトのFull SSLモードでOK)  
Let's Encryptで証明書取ったりする

Webブラウザでアクセスして確認する

### この後

デーモンで起動するように設定する  
参考: https://github.com/mei23/misskey/blob/mei-m544/docs/setup.ja.md#systemd%E3%82%92%E7%94%A8%E3%81%84%E3%81%9F%E8%B5%B7%E5%8B%95

管理者ユーザーを設定する
```sh
# misskeyユーザーで
cd ~/misskey
node cli/mark-admin.js @ユーザーID
```

### バージョンアップ方法

```sh
# Misskeyユーザーでなければ変更する
sudo su - misskey

cd ~/misskey

# 変更を取得＆マージ
git pull

# 依存関係の更新とビルド
yarn install && NODE_ENV=production yarn build

# 管理者ユーザーに戻る
exit

# Misskeyを再起動する
sudo systemctl restart misskey

```
