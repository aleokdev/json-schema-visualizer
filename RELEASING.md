## For updating the Github Pages version
1. Run `npm run build`
2. Create a new commit with a message akin to "Update dist"
3. Run `git subtree push --prefix dist origin gh-pages`

## For updating on other host
1. On `vue.config.js`, change `/json-schema-visualizer/` on `publicPath` to the path where the visualizer will be located
2. Run `npm run build`
3. Copy the contents of `dist` to your target path