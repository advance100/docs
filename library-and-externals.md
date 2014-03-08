You developed a library and want to distribute it in a compiled/bundled versions (in addition to the modulized version). You want to allow the user to use it in a `<script>`-tag or with a [[amd]] loader (i. e. `require.js`). Or you depend on various precompilations and want to take the pain for the user and distribute it as simple compiled [[commonjs]] module.

## configuration options

webpack has three [[configuration]] options that are relevant for that use cases: `output.library`, output.libraryTarget` and `externals`.

`output.libraryTarget` allows you to specify the kind to the output. I. e. CommonJs, AMD, for useage in a script tag or as UMD module.

`output.library` allows you to optionally specify a name of your library.

`externals` allows you to specify dependencies for your library that are not resolved by webpack, but become dependencies of the output. This means they are imported from the enviroment while runtime.

## Examples

#### compile library for usage in a `<script>`-tag

* depends on `"jquery"`, but jquery should not be included in the bundle.
* library should be available at `Foo` in the global context.

``` javascript
var jQuery = require("jquery");
var math = require("math-library");

function Foo() {}

// ...

module.exports = Foo;
```

Recommended configuration (only relevant stuff):

``` javascript
{
	output: {
		// export itself to a global var
		libraryTarget: "var"
		// name of the global var: "Foo"
		library: "Foo",
	},
	externals: {
		// require("jquery") is external and available
		//  on the global var jQuery
		"jquery": "jQuery"
	}
}
```

Resulting bundle:

``` javascript
var Foo = (/* ... webpack bootstrap ... */
({
	0: function(...) {
		var jQuery = require(1);
		/* ... */
	},
	1: function(...) {
		module.exports = jQuery;
	},
	/* ... */
});
```