# json-schema-visualizer

[See example](https://json-schema-visualizer.netlify.com).

## Project setup
```
npm install
```

### Compiles and hot-reloads for development the example
```
npm run serve
```


### Usage
```
import JsonSchema as 'src/components/JsonSchema'

<json-schema :value="schema"/>
```

#### Update Github Pages
While on the `master` branch:
```sh
git subtree push --prefix dist origin gh-pages
```