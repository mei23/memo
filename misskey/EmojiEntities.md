
### API emoji/list で使われてるやつ
V10, v11 にあり
```
name: nameの部分
host: hostの部分 (v10:Unicode, v11:Punycode)
url: URL
id: DBのid (あまり使わない)
aliases: ローカルのMisskey用
```

### API emojis, emojis/recommendation で使われてるやつ
XEmojiと呼んでる  
実際のクライアント提示に適した形式  
m544
```
name: 'name' or 'name@host' (hostは常にPunycode, hostが省略されている場合はローカル)
url: リモート分は'/files/name@host/time.png'のProxy用URLになる
```

### 添付用 で使われてるやつ
- REmojiと呼んでる
- Note, User に emojis として添付するための形式
- カスタム絵文字とアバター絵文字を扱う
- クライアント側がNote中の絵文字を表示する時には、`:name:`を`url`にマップすればいい。
- m544
```
name: 添付元のNote等に記述されている形式 (host部分が省略されている場合は、Noteの所属hostを使用することを意味する, Punycode)
host: nameのhost部分 (Punycode)
url: リモート分は'/files/name@host/time.png'のProxy用URLになる
resolvable: 'name' or 'name@host' (hostは常にPunycode, hostが省略されている場合はローカル XEmojiのnameと同じ)
```
