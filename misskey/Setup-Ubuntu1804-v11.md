# Misskey v11/v12 セットアップ Ubuntu 18.04

## 手順

### Ubuntu 18.04 を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- `物理メモリ2GB` or `物理メモリ1GB + スワップ1GB` くらいあるとよい
- 20.04でも大丈夫

### 管理者ユーザーで以下を実行

#### Misskey用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login misskey
```

#### Node.jsインストール  
```sh
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```
引用元: https://github.com/nodesource/distributions/blob/master/README.md


### yarnインストール
```sh
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn
```
引用元: https://classic.yarnpkg.com/en/docs/install#debian-stable

#### その他の必要パッケージをインストール
```sh
sudo apt -y install redis git build-essential nginx ssl-cert letsencrypt ffmpeg postgresql
```

#### DBロールとユーザーの作成

```sh
sudo -u postgres psql
```

```sql
# ロールの作成と確認 (passwordは適切に置き換える)
create role misskey LOGIN CREATEDB PASSWORD 'password';
\du

# DB (misskey) の作成と確認
create database misskey owner misskey;
\l

# 接続終了
\q
```

接続を試したい場合は以下コマンド後に
```sh
psql --host localhost --username misskey --password
# パスワードを入力
# 接続したら \q または Ctrl+D で抜けられます
```

#### misskeyユーザーでMisskeyのインストールをする
```sh
# misskeyユーザーに変更
sudo su - misskey

# リポジトリクローン
git clone -b master https://github.com/syuilo/misskey.git

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

# db設定は少なくとも pass は置き換える必用があるはず
db:
  host: localhost
  port: 5432

  # Database name
  db: misskey

  # Auth
  user: misskey
  pass: 設定したパスワード
```

### 編集が終わったら、引き続きmisskeyユーザーでインストールとビルドを行う
```sh
cd ~/misskey

# パッケージインストール
yarn install

# ビルド
NODE_ENV=production yarn build

# DBスキーマ作成
NODE_ENV=production yarn migrate

# 起動してみる
yarn start
# Now listening on port 3000 on https://example.com などと出ればOK
# 正常に起動することを確認して Ctrl+Cで 終了する

# 管理者ユーザに戻る
exit
```

### nginxの設定

管理者ユーザーで実施
```sh
sudo cp ~misskey/misskey/docs/examples/misskey.nginx /etc/nginx/sites-enabled/
# 本当は sites-available にコピーして sites-enabled からシンボリックリンクが流儀

# example.tldをドメインで置き換える
# letsencryptの設定をコメントアウトして、snakeoilの設定のコメントをはずす。
sudo vim /etc/nginx/sites-enabled/misskey.nginx

# nginxをリロード
sudo service nginx reload
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

CloudFlareとの間でSSL/TLS設定をする  
https://github.com/mei23/memo/blob/master/misskey/Setup-CloudFlareNginx.md の方法1などを参照

起動後Webブラウザで実際にアクセスしてみて動作が確認できれば完了

### バージョンアップ方法

```sh
# Misskeyユーザーでなければ変更する
sudo su - misskey

cd ~/misskey

# 変更を取得＆マージ
git pull

# 依存関係の更新
yarn install

# ビルド
NODE_ENV=production yarn build

# DBマイグレーション
yarn migrate

# 管理者ユーザーに戻る
exit

# Misskeyを再起動する
sudo systemctl restart misskey
```
