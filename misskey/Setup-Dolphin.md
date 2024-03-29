# Dolphin セットアップ Ubuntu

## 手順

### Ubuntu 22.04を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- メモリ512MB+スワップ(1-2GB)で余裕で動くけど、メモリ1GBあった方がいい。
- 20.04でも大丈夫

### 管理者ユーザーで以下を実行

#### dolphin用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login dolphin
```

#### Node.jsインストール  

https://github.com/nodesource/distributions/blob/master/README.md#debian-and-ubuntu-based-distributions  
※ `NODE_MAJOR=18` にしてインストール

### yarnインストール
```sh
sudo npm install -g yarn

```

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

# リポジトリクローン (ここではfork版をcloneしてます)
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
# メモリが512MBくらいの環境とかで、heapがなんたらと言われて失敗するときは以下のコマンドを使用
NODE_OPTIONS="--max-old-space-size=2048" NODE_ENV=production yarn build

# DBスキーマ作成
yarn migrate

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

# 依存関係の更新
yarn install

# ビルド
NODE_ENV=production yarn build

# DBマイグレーション
yarn migrate

# 管理者ユーザーに戻る
exit

# dolphinを再起動する
sudo systemctl restart dolphin
```
