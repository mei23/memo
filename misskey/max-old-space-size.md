# max-old-space-size
いわゆるNodeのヒープサイズ

実際のNodeのヒープサイズは以下で取得できる
```
node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
```

デフォルト値は実質環境依存値で、主に物理メモリサイズが影響。  
https://zenn.dev/legalscape/articles/a0715a699eacb5#node.js%E3%81%AE%E3%83%A1%E3%83%A2%E3%83%AA%E5%88%B6%E9%99%90

以下はUbuntu 22.04, Node v20の値の例
```
物理メモリ512MB: 259
物理メモリ1GB: 464
物理メモリ2GB: 974
物理メモリ4GB: 1962
物理メモリ8GB: 2096
```

値変えていてビルドするとどのくらいでビルド出来るかだいたいわかりそう
```
# 物理メモリ512MBでビルドできる？
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=259 pnpm run build
# 物理メモリ1GBでビルドできる？
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=464 pnpm run build
# 物理メモリ2GBでビルドできる？
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=974 pnpm run build
# 物理メモリ4GBでビルドできる？
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=1962 pnpm run build
```

実際普通にビルドするのに必要と思われるメモリはこのくらい
```
Misskey 2024.3.1: 物理メモリ4GB ※develop 2024/4/9でも同様
めいすきー 10.102.697-m544: 物理メモリ1GB
めいどるふぃん (Dolphin): 物理メモリ1GB ※512MBではダメ
```

スワップを増やしても解決しない。

値はオプションで変えられる  
実際のサイズはちょっと違う、以下はUbuntu 22.04, Node v20の値の例
```
NODE_OPTIONS=--max-old-space-size=500 node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
→ 524
NODE_OPTIONS=--max-old-space-size=1000 node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
→ 1024
NODE_OPTIONS=--max-old-space-size=1500 node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
→ 1524
NODE_OPTIONS=--max-old-space-size=2000 node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
→ 2024
```
