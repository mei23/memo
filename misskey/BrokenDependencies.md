# 壊れるdependencies

Misskey, Dolphin 各バージョンでUpdateすると不具合の出るdependencies

バージョン, ロックファイルの状態, コードの状態によって出る出ないがあるのでここにまとめて記述

### commander v5.x

マイグレーションが作成できない https://github.com/syuilo/misskey/issues/6337

他の依存関係も 2.x あたり指定してたりするので、影響なくても上げるのは微妙。

### gulp-terser v1.2.1

terserのバージョン指定が`^4.0.0` => `>=4` になってるが、`5.0.0`に解決されるとエラーになる。

### css-loader v4.x
`esModule`オプション
https://github.com/syuilo/misskey/commit/60736bab2ac974c2d2c2c106d297fa67fdaff87a#diff-b0ba29b245f5327bca2d51ac8bbb0821

https://github.com/syuilo/misskey/issues/6613

なんかまだ他のバグ多そう

### koa-router

v9.1.0 AP/Web両対応パスが壊れる https://github.com/syuilo/misskey/issues/6533

v9.x の時点でパスマッチにBreaking changes あり

そもそも、他のプラグインと組み合わせるとtype errorになりがち


