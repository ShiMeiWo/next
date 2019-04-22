---
layout: detail
title: Compiler
---

## Introduction

Custom tags need to be transformed to javascript before the browser can execute them.
The riot compiler is designed to transpile riot tags into javascript modules.
A compiled riot tag will look like this:

```js
export default {
  css: `my-tag { color: red; }`, // component css string
  template: function() {}, // internal riot template factory function
  exports: {}, // component events and lifecycle methods
  name: 'my-tag' // component id
}
```

**Each tag file must contain only one tag definition**.

## In-browser compilation

The `riot+compiler.js` bundle lets you compile and execute riot tags directly in your browser for quick prototypes and tests.
You can load riot tags into your browser by setting a `type="riot"` attribute on your script tags.
For example:

``` html
<!-- mount point -->
<my-tag></my-tag>

<!-- <my-tag/> is specified in an external file -->
<script src="path/to/javascript/my-tag.riot" type="riot"></script>

<!-- include riot.js and the compiler -->
<script src="https://unpkg.com/riot@{{ site.data.globals.version }}/riot+compiler.min.js"></script>

<!-- compile and mount -->
<script>
(async function main() {
  await riot.compile()

  riot.mount('my-tag')
}())
</script>
```

Notice that in this case riot will internally transform all the `export default` expressions to enable a better support for browsers that don't support javascript modules yet.

Riot can compile asynchronously all the external tags included via `<script>` into the DOM and via `riot.mount` it will render them.

You might prefer using `data-src` instead of `src` on your `<script>` tags stop your browser prefetching automatically any riot script tag in order to avoid to load the same resources twice. Riot will automatically fetch and compile your tags via ajax.

## Pre-compilation

The Compilation phase is asynchronous and it will not block your application rendering. However you should use the browser compilation only for prototyping or for quick experiments.

Pre-compilation on gives you following benefits:

- Ability to compile tags with your [favorite pre-processor](#pre-processors).
- Big performance benefit. No need to load and execute the compiler on browser.
- Sourcemaps support for debugging.

### Riot loaders

Tools like [`webpack`](https://webpack.js.org/) and [`rollup`](https://rollupjs.org/) are the perfect match to bundle your riot application tags.
For such tools we provide riot official loaders to let import natively riot components into your source code:
  - [webpack](https://github.com/riot/webpack-loader)
  - [rollup](https://github.com/riot/rollup-plugin-riot).
  - [parcel](https://github.com/riot/parcel-plugin-riot)
  - [riotify](https://github.com/riot/riotify)

With the riot loaders your application entry script might look like this:

```js
import { component } from 'riot'
import MyTag from './path/to/tags/my-tag.riot'

component(MyTag)(document.getElementById('root'))
```

### Compilation via Node

``` javascript
import {compile} from 'riotjs@compiler'

const { code, map } = compile('<p>{hello}</p>')
```

The compile function takes a string and returns an object containing the `code` and `map` keys.
You can handle the code generated however you like and use it into your build system.

Remember that the riot compiler outputs javascript modules and you might want to transpile them in your bundle.


### Compilation via Riot.js CLI

You can precompile Riot.js files also via the [`riot`](https://github.com/riot/cli) executable, which can be installed with NPM as follows:

``` sh
npm install @riotjs/cli -g
```

#### Using

Here is how `riot` command works:

``` sh
# compile a file to current folder
riot some.riot

# compile file to target folder
riot some.riot --output some-folder

# compile file to target path
riot some.riot --output some-folder/some.js

# compile all files from source folder to target folder
riot some/folder --output path/to/dist

```

The source files can contain only one custom element each!

For more information, type: `riot --help`


#### Watch mode

You can watch directories and automatically transform files when they are changed.

``` sh
# watch for
riot -w src -o dist
```


#### Custom extension

You're free to use any file extension for your tags (instead of default `.riot`):

``` sh
riot --extension html
```

#### ES6 Config file

You can use a config file to store and configure easily all your `@riotjs/cli` options and create your custom parsers

``` sh
riot --config riot.config src
```

The riot `riot.config.js` file:

```js
export default {
  output: 'tags/dist',
  // files extension
  extension: 'foo'
}
```

If you want to use custom preprocessor in your project you should install `@riotjs/cli` as `devDependency` runnint it via npm scripts as follows:


```json

{
  "scripts": {
    "build": "npx riot -c riot.config src"
  },
  "devDependencies": {
    "@riotjs/cli": "^4.0.0",
    "@riotjs/compiler": "^4.0.0",
    "pug": "^2.0.3"
  }
}
```

That's how your `riot.config.js` file might look like in case you want to use `pug` as components template engine

```js
import { registerPreprocessor } from '@riotjs/compiler'
import { render } from 'pug'

// register the pug preprocessor
registerPreprocessor('template', 'pug', (code, options) => {
  const { file } = options

  return {
    code: render(code, {
      filename: file,
      pretty: true,
      doctype: 'html'
    })
  }
})

export default {
  extension: 'pug',

  // assign the pug preprocessor to the riot compiler options
  riot: {
    template: 'pug'
  }
}
```



