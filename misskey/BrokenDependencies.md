# 壊れるdependencies

Misskey, Dolphin 各バージョンでUpdateすると不具合の出るdependencies

バージョン, ロックファイルの状態, コードの状態によって出る出ないがあるのでここにまとめて記述

### commander v5.x

マイグレーションが作成できない https://github.com/syuilo/misskey/issues/6337

他の依存関係も 2.x あたり指定してたりするので、影響なくても上げるのは微妙。

### gulp-terser v1.2.1

terserのバージョン指定が`^4.0.0` => `>=4` になってるが、`5.0.0`に解決されるとエラーになる。
