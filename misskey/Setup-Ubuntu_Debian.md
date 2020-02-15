
# Misskey v10 セットアップ Ubuntu 18.04 編

## 前提

- 以下の環境用
  - Ubuntu 18.04
- サーバーはファイアウォールなどで保護されていて、22,80,443など以外は開いてない  
  (これ前提で認証設定は端折ってます)

## 手順

### Ubuntu 18.04 が入ってるサーバーを用意する

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

これ以降は [Setup-Ubuntu1804-Quick#misskeyユーザーで以下を実行](Setup-Ubuntu1804-Quick.md#misskeyユーザーで以下を実行) とその先を実施
