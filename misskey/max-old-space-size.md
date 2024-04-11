# max-old-space-size

Nodeのヒープサイズは以下で取得できる
```
node -e 'console.log(Math.floor(v8.getHeapStatistics().heap_size_limit/1024/1024))'
```

オプションで変えられる  
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

デフォルト値は実質環境依存値で、主に物理メモリサイズが影響。  
https://zenn.dev/legalscape/articles/a0715a699eacb5#node.js%E3%81%AE%E3%83%A1%E3%83%A2%E3%83%AA%E5%88%B6%E9%99%90

スワップを増やしても解決しない。

値変えていてビルドするとどのくらいでビルド出来るかわかる
```
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=500 pnpm run build
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=1000 pnpm run build
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=1500 pnpm run build
NODE_ENV=production NODE_OPTIONS=--max-old-space-size=2000 pnpm run build
```
