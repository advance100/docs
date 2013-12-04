# Why should I prefer webpack over browserify?

* can automatically split you code into multiple chunks, which are loaded on demand
* takes care of other resources too (i. e. css, images, fonts)
* rich configuration (i. e. for custom module directory)
* can use AMD modules natively
* generate smaller files (It replaces all require strings with ids)
* minimizing and hashing integrated, plugins for gzipping and localization
* has a rich plugin system, that let you extend the compiler
* Hot Code Replacement

# Usage

Like browserify webpack analyse all the node-style `require()` calls in your app and build a bundle that you can serve up to the browser using a `<script>` tag.

Instead of doing

``` sh
$ browserify main.js > bundle.js
```

do

``` sh
$ webpack main.js bundle.js
```

> webpack doesn't write to stdout. You need to specify a filename. It cannot write to stdout because other than browserify is may generate multiple output files.

---

The best way to configure webpack is with a `webpack.config.js` file. It's loaded from current directory, when running the executable.

So

``` sh
$ browserify --entry main.js --outfile bundle.js
```

Maps to `webpack` with this config:

``` javascript
// webpack.config.js
module.exports = {
  entry: "./main.js",
  output: {
    filename: "bundle.js"
  }
};
```

---

## outfile

If you want to emit the output files to another directory: 

``` sh
$ browserify --outfile js/bundle.js
```

``` javascript
var path = require("path");
module.exports = {
  output: {
    path: path.join(__dirname, "js"),
    filename: "bundle.js"
  }
};
```

## entry

``` sh
$ browserify --entry a.js --entry b.js
```

``` javascript
module.exports = {
  entry: [
    "./a.js",
    "./b.js"
  ]
};
```

## transform

browserify uses *transforms* to preprocess files. webpack uses *loaders*. Loaders are functions that take source code as argument and return (modified) source code. Like transforms thay run in node.js, can be chained and can be async. Loader can take additional parameters by query strings. Loaders can be used from `require()` call. Transforms can be specified in the `package.json`. browserify apply configurated transforms for each module, in webpack config you select the modules by RegExp. In the common case you specify loaders in the `webpack.config.js`:

``` sh
$ browserify --transform coffeeify
```

``` javascript
module.exports = {
  module: {
    loaders: [
      { test: /\.coffee$/, loader: "coffee-loader" }
    ]
  }
};
```

## debug

``` sh
$ browserify -d
# Add inlined SourceMap
```

``` sh
$ webpack --devtool inline-source-map
# Add inlined SourceMaps

$ webpack --devtool source-map
# Emit SourceMaps as separate file

$ webpack --debug
# Add more debugging information to the source

$ webpack --output-pathinfo
# Add comments about paths to source code

$ webpack -d
# = webpack --devtool source-map --debug --output-pathinfo
```

## extension

``` sh
$ browserify --extension coffee
```

``` javascript
module.exports = {
  resolve: {
    extensions: ["", ".js", ".coffee"]
  }
};
```

## standalone

``` sh
browserify --standalone MyLibrary
```

``` javascript
module.exports = {
  output: {
    library: "MyLibrary",
    libraryTarget: "umd"
  }
};
// webpack --output-library MyLibrary --output-library-target umd
```

## ignore

``` sh
$ browserify --ignore file.js
```

``` javascript
var IgnorePlugin = require("webpack/lib/IgnorePlugin");
module.exports = {
  plugins: [
    new IgnorePlugin(/file\.js$/)
  ]
};
```

## node globals

``` sh
$ browserify --insert-globals
$ browserify --detect-globals
```

You can enable/disable these node globals individually:

``` javascript
module.exports = {
  node: {
    filename: true,
    dirname: "mock",
    process: false,
    global: true
  }
};
```

## ignore-missing

``` sh
$ browserify --ignore-missing
```

webpack prints errors for each missing dependency, but doesn't fail to build a bundle. You are free to ignore these errors. The `require` call will throw an error on runtime.

## noparse

``` sh
$ browserify --noparse=file.js
```

``` javascript
module.expors = {
  module: {
    noParse: [
      /file\.js$/
    ]
  }
};
```

## build info

``` sh
$ browserify --deps
$ browserify --list
```

``` sh
$ webpack --json
```

## external requires

webpack do not support external requires. You cannot expose the `require` function to other scripts. Just use webpack for all scripts on a page or do it like this:

``` javascript
module.exports = {
  output: {
    library: "require",
    libraryTarget: "var"
  }
};
```

``` javascript
// entry point
module.exports = function(module) {
  switch(module) {
  case "through": return require("through");
  case "duplexer": return require("duplexer");
  }
  throw new Error("Module '" + module + "' not found");
};
```

## multiple bundles

With browserify you can create a commons bundle that you can use in combination with bundles on multiple pages. To generate these bundles you exclude the common stuff with the `--exclude` option. Here is the example from the browserify README:

``` sh
$ browserify -r ./robot > static/common.js
$ browserify -x ./robot.js beep.js > static/beep.js
$ browserify -x ./robot.js boop.js > static/boop.js
```

webpack supports multi-page compilation and has a plugin for the automatic extraction of common modules:

``` javascript
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
  entry: {
    beep: "./beep.js",
    boop: "./boop.js",
  },
  output: {
    path: "static",
    filename: "[name].js"
  },
  plugins: [
    // ./robot is automatically detected as common module and extracted
    new CommonsChunkPlugin("common.js")
  ]
};
```

``` html
<script src="common.js"></script>
<script src="beep.js"></script>
```

# API

No need to learn much more. Just pass the config object to the `webpack` API:

``` javascript
var webpack = require("webpack");

webpack({
  entry: "./main.js",
  output: {
    filename: "bundle.js"
  }
}, function(err, stats) {
  err // => fatal compiler error (rar)
  var json = stats.toJson() // => webpack --json
  json.errors // => array of errors
  json.warnings // => array of warnings
});
```

# Third-party-tool mappings

| browserify | webpack |
|------------|---------|
| watchify   | `webpack --watch` |
| browserify-middleware | [[webpack-dev-middleware]] |
| beefy | [[webpack-dev-server]] |
| deAMDify | `webpack` |
| decomponentify | [component-webpack-plugin](https://github.com/webpack/component-webpack-plugin) |
| list of source transforms | [[loader list]], [transform-loader](https://github.com/webpack/transform-loader) |
| 