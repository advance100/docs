# internal webpack plugins

These is a list of plugins, which are internally used by webpack. You should only care about them if you are building a own compiler based on webpack, or introspec the internals.

categories of internal plugins:

* environment
* compiler
* entry
* output
* source
* optimize

## environment

Plugins affecting the environment of the compiler.

### `node/NodeEnvironmentPlugin`

Applies node.js style filesystem to the compiler.

## compiler

Plugins affecting the compiler

### `CachePlugin([cache])`

Adds a cache to the compiler, where modules are cached.

You can pass an `cache` object, where the modules are cached. Elsewise one is created per plugin instance.

### `ProgressPlugin(handler)`

Hook into the compiler to extract progress information. The `handler` must have the signature `function(percentage, message)`. It's called with `0 <= percentage <= 1`. `percentage == 0` indicates the start. `percentage == 1` indicates the end.

## entry

Plugins, which add entry chunks to the compilation.

### `SingleEntryPlugin(context, request, chunkName)`

Adds a entry chunk on compilation. The chunk is named `chunkName` and contains only one module (plus dependencies). The module is resolved from `request` in `context` (absolute path).

### `MultiEntryPlugin(context, requests, chunkName)`

Adds a entry chunk on compilation. The chunk is named `chunkName` and contains a module for each item in the `requests` array (plus dependencies). Each item in `requests` is resolved in `context` (absolute path).

## output

### `FunctionModulePlugin(context, options)`

Each emitted module is wrapped in a function.

`options` are the output options.

If `options.pathinfo` is set, each module function is annotated with a comment containing the module identifier shortened to `context` (absolute path).

### `JsonpTemplatePlugin(options)`

Chunks are wrapped into JSONP-calls. A loading algorithm is included in entry chunks. It loads chunks by adding a `<script>` tag.

`options` are the output options.

`options.jsonpFunction` is the JSONP function.

`options.publicPath` is uses as path for loading the chunks.

`options.chunkFilename` is the filename under that chunks are expected.

### `LibraryTemplatePlugin(name, target)`

The entries chunks are decorated to form a library `name` of type `type`.

### `webworker/WebWorkerTemplatePlugin(options)`

Chunks are loaded by `importScripts`. Else it's similar to `JsonpTemplatePlugin`.

`options` are the output options.

### `EvalDevToolModulePlugin`

Decorates the module template by wrapping each module in a `eval` annotated with `// @sourceURL`.

## source

Plugins affecting the source code of modules.

### `APIPlugin`

Make `__webpack_public_path__`, `__webpack_require__`, `__webpack_modules__`, `__webpack_chunk_load__` accessable. Ensures that `require.valueOf` and `require.onError` are not processed by other plugins.

### `CompatibilityPlugin`

Currently useless. Ensures compatiblitly with other module loaders.

### `ConsolePlugin`

Offers a pseudo `console` if it is not availible.

### `ConstPlugin`

Try to evaluate expressions in `if(...)` and replace it with `true`/`false`.

### `ProvidePlugin(name, request)`

If `name` is used in a module it is filled by a module loaded by `require(<request>)`.

### `NodeStuffPlugin`

Provide stuff that is normally avalible in node.js modules.

`__filename` is constant `"/index.js"`. `__dirname` is constant `"/"`. `require.main` is the `module` of your entry module.

It also ensures that `module` is filled with some node.js stuff if you use it.

### `NodeStuffRealPathsPlugin`

The `NodeStuffPlugin` only provides pseudo `__filename` and `__dirname`. This plugin provide the real `__filename` and `__dirname` of the modules.

Include this plugins before `NodeStuffPlugin`.

### `node/NodeSourcePlugin`

This module adds stuff from node.js that is not avalible in non-node.js environments.

It adds polyfills for `process` and `global` if used. It also binds the buildin node.js replacement modules.

### `node/NodeTargetPlugin`

The plugins should be used if you run the bundle in a node.js environment.

If ensures that native modules are loaded correctly even if bundled.

### `dependencies/AMDPlugin(options)`

Provides AMD-style `define` and `require` to modules. Also bind `require.amd`, `define.amd` and `__webpack_amd_options__` to the `options` passed as parameter.

### `dependencies/CommonJsPlugin`

Provides CommonJs-style `require` to modules.

### `dependencies/RequireContextPlugin(modulesDirectories, extensions)`

Provides `require.context`. The parameter `modulesDirectories` and `extensions` are used to find alternative requests for files. It's useful to provide the same arrays as you provide to the resolver.

### `dependencies/RequireEnsurePlugin`

Provides `require.ensure`.

### `dependencies/RequireIncludePlugin`

Provides `require.include`.

## optimize

### `optimize/LimitChunkCountPlugin(options)`

Merge chunks limit chunk count is lower than `options.maxChunks`.

The overhead for each chunks is provided by `options.chunkOverhead` or defaults to 10000. Entry chunks sizes are multiplied by `options.entryChunkMultiplicator` (or 10).

Chunks that reduce the total size the most are merged first. If multiple combinations are equal the minimal merged size wins.

### `optimize/MergeDuplicateChunksPlugin`

Chunks with the same modules are merged.

### `optimize/RemoveEmptyChunksPlugin`

Modules that are included in every parent chunk are removed from the chunk.

### `optimize/MinChunkSizePlugin(minChunkSize)`

Merges chunks until each chunk has the minimum size of `minChunkSize`.

### `optimize/FlagIncludedChunksPlugin`

Adds chunk ids of chunks which are included in the chunk. This eliminates unnessary chunk loads.

### `optimize/UglifyJsPlugin(options)`

Minimizes the chunks with `uglify.js`.

`options` are uglifyjs options.