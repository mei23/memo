# Dolphin セットアップ Ubuntu 18.04

## 前提

- Ubuntu 18.04
- サーバーはファイアウォールなどで保護されていて、22,80,443など以外は開いてない  

## 手順

### Ubuntu 18.04 を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- メモリ512MB+スワップ(1-2GB)で余裕で動くけど、メモリ1GBあった方がいい。

### 管理者ユーザーで以下を実行

#### dolphin用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login dolphin
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
create role dolphin LOGIN CREATEDB PASSWORD 'password';
\du

# DB (dolphin) の作成と確認
create database dolphin owner dolphin;
\l

# 接続終了
\q
```

接続を試したい場合は以下コマンド後に
```sh
psql --host localhost --username dolphin --password
# パスワードを入力
# 接続したら \q または Ctrl+D で抜けられます
```

#### dolphinユーザーでdolphinのインストールをする
```sh
# dolphinユーザーに変更
sudo su - dolphin

# リポジトリクローン
git clone https://github.com/mei23/dolphin.git

# ディレクトリ移動
cd ~/dolphin

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
  db: dolphin

  # Auth
  user: dolphin
  pass: 設定したパスワード
```

### 編集が終わったら、引き続きdolphinユーザーでインストールとビルドを行う
```sh
cd ~/dolphin

# パッケージインストール
yarn install

# ビルド
NODE_ENV=production yarn build
# メモリが512MBくらいの環境とかで、heapがなんたらと言われて失敗するときは以下のコマンドでを使用
NODE_OPTIONS="--max-old-space-size=2048" NODE_ENV=production yarn build

# DBスキーマ作成
yarn migrate

# 起動してみる
yarn start

# Now listening on port 3000 on https://example.com などと出ればOK

正常に起動することを確認して Ctrl+Cで 終了する

# 管理者ユーザに戻る
exit
```

### nginxの設定

管理者ユーザーで実施
```sh
sudo cp ~dolphin/dolphin/docs/examples/dolphin.nginx /etc/nginx/sites-enabled/
# 本当は sites-available にコピーして sites-enabled からシンボリックリンクが流儀

# example.tldをドメインで置き換える
sudo vim /etc/nginx/sites-enabled/dolphin.nginx
```

引き続き
```sh
# Nginxをリロード
sudo service nginx reload
```

### systemdで起動するように設定する

```sh
# serviceファイルをコピー
sudo cp ~dolphin/dolphin/docs/examples/dolphin.service /etc/systemd/system/

# 再読込して自動起動するように設定
sudo systemctl daemon-reload
sudo systemctl enable dolphin

# サービスの起動
sudo systemctl start dolphin

# 状態の確認
sudo systemctl status dolphin
```

CloudFlareとの間でSSL/TLS設定をする  
https://github.com/mei23/memo/blob/master/dolphin/Setup-CloudFlareNginx.md の方法1などを参照

起動後Webブラウザで実際にアクセスしてみて動作が確認できれば完了

### バージョンアップ方法

```sh
# dolphinユーザーでなければ変更する
sudo su - dolphin

cd ~/dolphin

# 変更を取得＆マージ
git pull

# 依存関係の更新とビルド
yarn install && NODE_ENV=production yarn build

# 管理者ユーザーに戻る
exit

# dolphinを再起動する
sudo systemctl restart dolphin
```
