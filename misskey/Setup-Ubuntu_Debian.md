
# Misskey セットアップ Ubuntu/Debian 編

## 前提

- 以下の環境用
  - Ubuntu 16.04
  - Ubuntu 18.04 で最新のNode.jsとMongoDBを使用したい場合  
    Ubuntu 18.04 でディストリビューション付属のバージョンで良い場合は [Ubuntu 18.04 簡単編](Setup-Ubuntu1804-Quick.md) を参照してください。
  - Debian
- サーバーはファイアウォールなどで保護されていて、22,80,443など以外は開いてない  
  (これ前提で認証設定は端折ってます)

## 手順

### Ubuntu / Debian 等を用意する

- VPSやクラウドで、FW付きで(Allow 22,80,443)、サーバーインストールを想定
- `物理メモリ2GB` or `物理メモリ1GB + スワップ1GB` くらいあるとよい

### 管理者ユーザーで以下を実行

#### Misskey用ユーザー作成
```sh
sudo adduser --disabled-password --disabled-login misskey

```

#### Node.jsインストール  
参考: https://nodejs.org/ja/download/package-manager/#debian-and-ubuntu-based-linux-distributions-debian-ubuntu-linux
```sh
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs

```

#### MongoDBインストール
参考: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
```sh
# 鍵インポート
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

# sources.list作成
# ディストリ/バージョンにより違うため、以下のあたりから拾ってください。  
# https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/#create-a-list-file-for-mongodb  
# https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/#create-a-etc-apt-sources-list-d-mongodb-org-4-0-list-file-for-mongodb  
#
# 例えば Ubuntu 18.04の場合
# echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list

# インストール
sudo apt-get update
sudo apt-get install -y mongodb-org

# MongoDBを自動起動するようにする
sudo systemctl enable mongod

# その他の必要パッケージをインストール
sudo apt -y install redis git build-essential nginx ssl-cert letsencrypt

```

#### misskeyユーザーに変更
```sh
sudo su - misskey

```

これ以降は [Setup-Ubuntu1804-Quick#misskeyユーザーで以下を実行](Setup-Ubuntu1804-Quick.md#misskeyユーザーで以下を実行) とその先を実施
