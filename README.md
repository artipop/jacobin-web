# Quasar App (quasar-project)

A Quasar Project

## Install the dependencies

- `yarn add assert`
- `yarn add process`
- `yarn add url`
- `yarn add util`
- `yarn add path-browserify` // or just `path`, idk

Then set up webpack, kinda:
```js
extendWebpack(cfg, {isServer, isClient}) {
  cfg.resolve.fallback = {
    ...cfg.resolve.fallback,

    assert: require.resolve('assert'),
    buffer: require.resolve('buffer'),
    path: require.resolve('path-browserify'),
    process: require.resolve('process/browser'),
    stream: require.resolve('readable-stream'), // seems like we don't need it
    url: require.resolve('url'),
    util: require.resolve('util'),
  }
  cfg.plugins.push(
    new webpack.ProvidePlugin({
      process: 'process/browser',
      Buffer: ["buffer", "Buffer"],
    }),
  )
}
```
And finally:
- `yarn add memfs`

```bash
yarn
# or
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)
```bash
quasar dev
```


### Lint the files
```bash
yarn lint
# or
npm run lint
```


### Format the files
```bash
yarn format
# or
npm run format
```



### Build the app for production
```bash
quasar build
```

### Customize the configuration
See [Configuring quasar.config.js](https://v2.quasar.dev/quasar-cli-webpack/quasar-config-js).
