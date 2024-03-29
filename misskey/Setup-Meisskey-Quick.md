
# Meisskey セットアップ Ubuntu 簡単編

## 手順

### Ubuntu 22.04を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- 少なくとも `物理メモリ1GB + スワップ2GB` くらいあるとよい
- amd64/arm64いずれでも動きます

### 管理者ユーザーで以下を実行

#### Misskey用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login misskey
```

#### Node.jsインストール  

https://github.com/nodesource/distributions/blob/master/README.md#debian-and-ubuntu-based-distributions  
※ `NODE_MAJOR=18` にしてインストール

#### pnpmインストール
```sh
sudo npm i -g pnpm
```

#### MongoDBインストール
https://www.mongodb.com/docs/v6.0/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition


```sh
# 開始する
sudo systemctl start mongod

# MongoDBを自動起動するようにする
sudo systemctl enable mongod
```

#### その他の必要パッケージをインストール
```sh
sudo apt -y install redis git build-essential nginx ssl-cert letsencrypt ffmpeg
```

#### misskeyユーザーに変更
```sh
sudo su - misskey
```

### misskeyユーザーで以下を実行
```sh
# リポジトリクローン (めいすきーforkをcloneしてます)
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
NODE_ENV=production pnpm i

# ビルド
NODE_ENV=production pnpm build
# heap out of memory (exit code 134) でビルドできない場合は以下のコマンドを使用 (2048は必要に応じて増やしてください)
# NODE_OPTIONS="--max-old-space-size=2048" NODE_ENV=production pnpm build

# ログアウトして管理者ユーザに戻る
exit
```

### nginxの設定

管理者ユーザーで実施
```sh
sudo cp ~misskey/misskey/docs/examples/misskey.nginx /etc/nginx/sites-enabled/
# 本当は sites-available にコピーして sites-enabled からシンボリックリンクが流儀

# example.tldをドメインで置き換える
sudo vim /etc/nginx/sites-enabled/misskey.nginx
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

### misskeyユーザーでMisskeyを起動してみる

```sh
cd ~/misskey

# 起動してみる
pnpm start
# Now listening on port 3000 on https://example.com などと出ればOK
# 正常に起動することを確認して Ctrl+Cで 終了する

# 管理者ユーザに戻る
exit
```

### systemdで起動するように設定する

```sh
# serviceファイルをコピー
sudo cp ~misskey/misskey/docs/examples/misskey.service /etc/systemd/system/

# 再読込して自動起動するように設定
sudo systemctl daemon-reload
sudo systemctl enable misskey

# サービスの起動
sudo systemctl start misskey

# 状態の確認
sudo systemctl status misskey
```

### CloudFlareとの間でSSL/TLS設定をする  
https://github.com/mei23/memo/blob/master/misskey/Setup-CloudFlareNginx.md の方法1などを参照

設定後Webブラウザで実際にアクセスしてみて動作が確認できれば完了

### 管理者ユーザーを設定する
```sh
# Misskeyユーザーでなければ変更する
sudo su - misskey

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

# 依存関係の更新
NODE_ENV=production pnpm i

# ビルド
NODE_ENV=production pnpm build

# 管理者ユーザーに戻る
exit

# Misskeyを再起動する
sudo systemctl restart misskey
```
