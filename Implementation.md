## pub-relay-prototype

https://source.joinmastodon.org/mastodon/pub-relay-prototype

- Ruby実装、動作環境はたぶんほぼMastodonと同じ。
- メンテはされていない。
- 挙動はたぶんREADMEの通り。
- HTTP-SignatureにalgorithmフィールドがないのでMisskeyでは動かない。

## pub-relay-prototype for taruntarun

https://source.joinmastodon.org/mayaeh/pub-relay-prototype/tree/for-taruntarun
- pub-relay-prototypeのfork
- https://relay.taruntarun.net/ で使われている。
- メンテされている。
- algorithmの問題が修正されている。
- Move ActivityをForwardする

## pub-relay

https://source.joinmastodon.org/mastodon/pub-relay

- たぶんリファレンス実装
- pub-relay-prototypeのCrystal移植？
- HTTP-SignatureにalgorithmフィールドがないのでMisskeyでは動かない
- READMEの`Only payloads that contain a linked-data signature will be re-broadcast`は多分間違いでなんでもforwardする
- ちょっと動かすのに苦労する
- 実装はシンプルでわかりやすいが、キャッシュとかキューとか輻輳制御とかはほぼないので注意。

## pub-relay (fork by noellabo)

https://github.com/noellabo/pub-relay

- 主な挙動的にはLitePub (Pleroma) サポートと対象Activityの追加など
  - 2021/11現在、LitePubは最近のPleromaでは動かない模様。
- 2021/11現在 relay.fedibird.com で稼働しているものは `Sep 21, 2020` あたりのコミットのものと思われるので注意。
  - `0.2.0`を名乗ってUser-Agentにdeliver先のホストを指定してしまっている場合おそらくこのバージョン
  - このバージョンの挙動検証用のDocker https://github.com/mei23/pub-relay/tree/n202009 (Crystalのバージョンとかはちょっと違うけど)
- publishedを付けないとActivityがforwardされない バグ？
- LitePub fallbackの挙動がちょっとあやしい

## pub-relay (mei23)

https://github.com/mei23/pub-relay

- pub-relay (fork by noellabo) のfork
- noellabo版からの主な挙動変更はLitePubの廃止とバグ修正
  - 結果的にはpub-relayに対象のActivity typeが追加されてバグが修正された感じ。

