
```sh
# 一新
yarn install && ./data/newver && yarn build && yarn start

yarn install --prod=false && ./data/newver && yarn clean && yarn build && yarn start 

# server-sideのみ
yarn gulp && yarn start


```

### To use Font Awesome icons

Find the icon you want to add  
https://fontawesome.com/icons?d=gallery&m=free

For example, if you want to add a `filter` ...

Search IconDefinition
```sh
$ find node_modules/@fortawesome -type f -name index.d.ts | xargs grep IconDefinition | grep -i filter
node_modules/@fortawesome/free-solid-svg-icons/index.d.ts:export const faFilter: IconDefinition;
```

in .vue ...
```ts
// import it
import { faFilter } from '@fortawesome/free-solid-svg-icons';

  data() {
		return {
			faFilter,
		};
	},

  // use it
  <fa :icon="faFilter"/>
```
