
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
name: 'name' or 'name@host' (hostは常にPunycode)
url: リモート分は'/files/name@host/time.png'のProxy用URLになる
```

### API emojis/recommendation
サジェスト用  
m544
Entityは上と同じ

