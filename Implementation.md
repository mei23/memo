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
