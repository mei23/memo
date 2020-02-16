特に明記がない限り Misskey v10-v12, Dolphin全てにあてはまると思う  
個人の感想だわ

### configのportとかの設定
Misskey app自体でhttpsを喋る機能があったりするけど普通に`port: 3000`とかにして  
nginxとかでリバースプロキシするのがおすすめ。

### 動く環境
Windowsでも動くくらいだから条件満たせばどこでも動くはず  
悩んだらUbuntu 18.04でいいんじゃないかしら  
Arch Linuxぽいのも稀に見る  

### バージョンアップ時のclean
バージョンアップ時に`yarn clean`を流すと  
code splittingで分割されたassetsのjsが404 (キャッシュの状況による) になる問題があるので  
広く公開する環境では非Dockerで`yarn clean`を回さないでバージョンアップしていったほうがいいかも  
`clean`しないデメリットは、APIが廃止された時にいつまでも残っちゃうとか

### コードを改修したら
コードを改修したら (特にクライアント側コード) バージョンを変えないとうまく反映されない  
`x.y.z-なんたら`で`なんたら`の部分をユニークなのに変えていくといいかも  

リポジトリURLの設定は、v11にはあったけどv12やDolphinにはなさそうだからソースコード検索して変えるしかないかも。

### チューニング
基本不要

#### UV_THREADPOOL_SIZE
ただ、AP deliver以外の名前解決がデフォルト2スレッドなので場合によっては増やしたほうがいいかも。

Environment Variable `UV_THREADPOOL_SIZE=size`で設定できて、設定値の半分が名前解決に使われる。  
https://nodejs.org/api/cli.html#cli_uv_threadpool_size_size

値は例えば16とか32とか。

#### configのパフォーマンス系の設定項目

##### worker process数
```
clusterLimit: 1
```
worker processの数 デフォルトの1で大丈夫  
1だとNodeの処理にCPU1スレッド分しか使えないらしいけど、たいていDBとか別プロセスの画像処理の方が重いので大丈夫  
v11-, Dolphinでガンガン増やすとPostgreSQLの接続が足りなくなったりするので注意  


##### ジョブの並列度
```
deliverJobConcurrency: 128
inboxJobConcurrency: 16
```
それぞれリモート配信/受信のジョブの並列度  
`ここで設定した値 x clusterLimit x サーバー数`になる  
変える必要ないと思う、流量を制御したいなら下の値をいじったほうがいい  


##### ジョブの流量
```
deliverJobPerSec: 128
inboxJobPerSec: 16
```
それぞれリモート配信/受信のジョブを1秒あたりに処理する数 (インスタンス全体での値)  
あまり連合の処理で負荷を上げたくなければ少なくするといいかも

### サーバーの発信IPアドレスを隠したい

Proxy設定できるけどNATとかVPNしたほうが無難  
AWSでNATセグメントの下に入れるとか
