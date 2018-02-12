<div align="center">
  <a href="https://github.com/webpack/webpack">
    <img width="200" height="200" src="https://webpack.js.org/assets/icon-square-big.svg">
  </a>
</div>

[![npm][npm]][npm-url]
[![node][node]][node-url]
[![deps][deps]][deps-url]
[![tests][tests]][tests-url]
[![coverage][cover]][cover-url]
[![chat][chat]][chat-url]

# webpack-serve

A lean, modern, and flexible webpack development server

## Getting Started

To begin, you'll need to install `webpack-serve`:

```console
$ npm install webpack-serve --save-dev
```

## CLI

```console
$ webpack-serve --help

  Options
    --config            The webpack config to serve. Alias for <config>.
    --content           The path from which content will be served
    --dev               An object containing options for webpack-dev-middleware
    --host              The host the app should bind to
    --http2             Instruct the server to use HTTP2
    --https-cert        Specify a cert to enable https. Must be paired with a key
    --https-key         Specify a key to enable https. Must be paired with a cert
    --https-pass        Specify a passphrase to enable https. Must be paired with a pfx file
    --https-pfx         Specify a pfx file to enable https. Must be paired with a passphrase
    --log-level         Limit all process console messages to a specific level and above
                        {dim Levels: trace, debug, info, warn, error, silent}
    --log-time          Instruct the logger for webpack-serve and dependencies to display a timestamp
    --no-hot            Instruct the client not to apply Hot Module Replacement patches
    --no-reload         Instruct middleware {italic not} to reload the page for build errors
    --open              Instruct the app to open in the default browser
    --open-app          The name of the app to open the app within
    --open-path         The path with the app a browser should open to
    --port              The port the app should listen on
    --version           Display the webpack-serve version

  Examples
    $ webpack-serve ./webpack.config.js --no-reload
```

_Note: The CLI will use your local install of webpack-serve when available,
even when run globally._

## API

When using the API directly, the main entry point  is the `serve` function, which
is the default export of the module.

```js
const serve = require('webpack-serve');
const config = require('./webpack.config.js');

serve({ config });
```

### serve([options])

Returns an `Object` containing:

- `close()` *(Function)* - Closes the server and its dependencies.
- `on(eventName, fn)` *(Function)* - Binds a serve event to a function. See
[Events](#events).

#### options

Type: `Object`

Options for initializing and controlling the server provided.

##### compiler

Type: `webpack`  
Default: `null`

An instance of a `webpack` compiler. A passed compiler's config will take
precedence over `config` passed in options.

##### config

Type: `Object`  
Default: `{}`

An object containing the configuration for creating a new `webpack` compiler
instance.

##### content

Type: `String|[String]`  
Default: `[]`

The path, or array of paths, from which content will be served.

##### dev

Type: `Object`  
Default: `{ publicPath: '/' }`

An object containing options for [webpack-dev-middleware][dev-ware].

##### host

Type: `Object`  
Default: `'localhost'`

Sets the host that the `WebSocket` server will listen on. If this doesn't match
the host of the server the module is used with, the module will not function
properly.

##### hot

Type: `Object`  
Default: `{}`

An object containing options for [webpack-hot-client][hot-client].

##### http2

Type: `Boolean`  
Default: `false`

If using Node v9 or greater, setting this option to `true` will enable HTTP2
support.

##### https

Type: `Object`  
Default: `null`

Passing this option will instruct `webpack-serve` to create and serve the webpack
bundle and accompanying content through a secure server. The object should
contain properties matching:

```js
{
  key: fs.readFileSync('...key'),   // Private keys in PEM format.
  cert: fs.readFileSync('...cert'), // Cert chains in PEM format.
  pfx: <String>,                    // PFX or PKCS12 encoded private key and certificate chain.
  passphrase: <String>              // A shared passphrase used for a single private key and/or a PFX.
}
```

See the [Node documentation][https-opts] for more information.

##### logLevel

Type: `String`  
Default: `info`

Instructs `webpack-serve` to output information to the console/terminal at levels
higher than the specified level. Valid levels:

```js
[
  'trace',
  'debug',
  'info',
  'warn',
  'error'
]
```

##### logTime

Type: `Boolean`  
Default: `false`

Instruct `webpack-serve` to prepend each line of log output with a `[HH:mm:ss]`
timestamp.

##### open

Type: `Boolean|Object`  
Default: `false`

Instruct the module to open the served bundle in a browser. Accepts an `Object`
that matches:

```js
{
  app: <String>, // The proper name of the browser app to open.
  path: <String> // The url path on the server to open.
}
```

##### port

Type: `Number`  
Default: `8080`

The port the server should listen on.

## Events

`webpack-serve` emits select events which can be subscribed to. For example;

```js
const serve = require('webpack-serve');
const config = require('./webpack.config.js');

serve({ config });

serve.on('listening', () => {
  console.log('happy fun time');
});
```

#### listening

Arguments: _None_

## Add-on Features

A core tenant of `webpack-serve` is to stay lean in terms of feature set, and to
empower users with familiar and easily portable patterns to implement the same
features that those familiar with `webpack-dev-server` have come to rely on. This
makes the module far easier to maintain, which ultimately benefits the user.

Luckily, flexibility baked into `webpack-serve` makes it a snap to add-on features.
Listed below are some of the add-on patterns that can be found in
[docs/addons](docs/addons):

- [bonjour](docs/addons/bonjour.config.js)
- [compress](docs/addons/compress)
- [historyApiFallback](docs/addons/history-fallback.config.js)
- [proxy](docs/addons/proxy.config.js)
- [staticOptions](docs/addons/static-content-options.config.js)
- [useLocalIp](docs/addons/local-ip.config.js)

## Node v6 Support

This module will continue to support Node v6 through the use of
[Babel](https://babeljs.io) until Node v10 enters [Active LTS](https://github.com/nodejs/Release).

In order to run `webpack-serve` under Node v6 with the CLI, you must install the
[optionalDependencies](/webpack-serve/blob/master/package.json#L30) that are
required to automatically wire up in-process Babel transpilation via a require
hook.

When running  `webpack-serve` using the API, you must still install the
`optionalDependencies` for Babel, and can make use of a utility
function for registering the proper Babel settings. Add the following to your
top-level file:

```
  const { babel } = require('webpack-serve/lib/global');
  babel();
```

If you have an existing babel setup, you should add an equivalent Babel config
to your implementation as can be found in
[`global.js`](/webpack-contrib/webpack-serve/blob/master/lib/server.js).

_Note: Performance will be impacted, though the amount of that impact will
vary on the implementation. This is a deliberate decision and will remain the
only way to run `webpack-serve` under Node v6. We have hopes of removing this
requirement in the near future._

## Contributing

We welcome your contributions! Please have a read of
[CONTRIBUTING.md](CONTRIBUTING.md) for more information on how to get involved.

## License

#### [MIT](./LICENSE)

[npm]: https://img.shields.io/npm/v/webpack-serve.svg
[npm-url]: https://npmjs.com/package/webpack-serve

[node]: https://img.shields.io/node/v/webpack-serve.svg
[node-url]: https://nodejs.org

[deps]: https://david-dm.org/webpack-contrib/webpack-serve.svg
[deps-url]: https://david-dm.org/webpack-contrib/webpack-serve

[tests]: http://img.shields.io/travis/webpack-contrib/webpack-serve.svg
[tests-url]: https://travis-ci.org/webpack-contrib/webpack-serve

[cover]: https://codecov.io/gh/webpack-contrib/webpack-serve/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/webpack-serve

[chat]: https://badges.gitter.im/webpack/webpack.svg
[chat-url]: https://gitter.im/webpack/webpack

[dev-ware]: /webpack/webpack-dev-middleware
[hot-client]: /webpack-contrib/webpack-hot-client
[https-opts]: https://nodejs.org/api/tls.html#tls_tls_createsecurecontext_options
