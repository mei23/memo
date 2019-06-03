
### API emoji/list
管理者がリモート/ローカルカスタム絵文字を一覧できるAPI  
V10, v11 にあり
```
name: nameの部分
host: hostの部分 (v10:Unicode, v11:Punycode)
url: URL
id: DBのid (あまり使わない)
aliases: ローカルのMisskey用
```

### API emojis
実際にクライアント提示に適した形式
m544
```
name: 'name' or 'name@host' (hostは常にPunycode, hostが省略されている場合はローカル)
url: リモート分は'/files/name@host/time.png'のProxy用URLになる
```

### API emojis/recommendation
サジェスト用  
m544
Entityは上と同じ

### 添付用
Note, User に emojis として添付するための形式  
カスタム絵文字とアバター絵文字を扱う  
m544
```
name: 添付元のNote等に記述されている形式 host部分が省略されている場合は、常にローカルではなくNoteの所属hostを使用する
host: nameのhost部分
url: リモート分は'/files/name@host/time.png'のProxy用URLになる
resolvable: 'name' or 'name@host' (hostは常にPunycode, hostが省略されている場合はローカル)
```
