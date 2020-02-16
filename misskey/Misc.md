

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

### コードを改修したら
コードを改修したら (特にクライアント側コード) バージョンを変えないとうまく反映されない  
`x.y.z-なんたら`で`なんたら`の部分をユニークなのに変えていくといいかも  

リポジトリURLの設定はv11にはあったけど、v12やDolphinにはなさそうだからソースコード検索して変えるしかないかも。

### チューニング
基本不要

#### UV_THREADPOOL_SIZE
ただ、AP deliver以外の名前解決がデフォルト2スレッドなので場合によっては増やしたほうがいいかも。

Environment Variable `UV_THREADPOOL_SIZE=size`で設定できて、設定値の半分が名前解決に使われる。  
https://nodejs.org/api/cli.html#cli_uv_threadpool_size_size

値は例えば16とか32とか。







