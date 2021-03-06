## Minimize

To minimize your scripts (and your css, if you use the css-loader) webpack supports a simple option:

`--optimize-minimize` resp. [`new webpack.optimize.UglifyJsPlugin()`](http://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin)

That's a simple but effective way to optimize your web app.

As you already know (if you've read the remaining docs) webpack gives your modules and chunks ids to identify them. Webpack can vary the distribution of the ids to get the smallest id length for often used ids with a simple option:

`--optimize-occurence-order` resp. `new webpack.optimize.OccurenceOrderPlugin()`

The entry chunks have higher priority for file size.



## Deduplication

If you use some libraries with cool dependency trees, it may occur that some files are identical. Webpack can find these files and deduplicate them. This prevents the inclusion of duplicate code into your bundle and instead applies a copy of the function at runtime. It doesn't affect semantics. You can enable it with:

`--optimize-dedupe` resp. `new webpack.optimize.DedupePlugin()`

This feature adds some overhead to the entry chunk.



## Chunks

While writing your code, you may have already added many code split points to load stuff on demand. After compiling you might notice that there are too many chunks that are too small - creating larger HTTP overhead. Luckily, Webpack can post-process your chunks by merging them. You can provide two options:

* Limit the maximum chunk count with `--optimize-max-chunks 15` `new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})`
* Limit the minimum chunk size with `--optimize-min-chunk-size 10000` `new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})`

Webpack will take care of it by merging chunks (it will prefer merging chunk that have duplicate modules). Nothing will be merged into the entry chunk, so as not to impact initial page loading time.



## Single-Page-App

A Single-Page-App is the type of web app webpack is designed and optimized for.

You may have split the app into multiple chunks, which are loaded at your router. The entry chunk only contains the router and some libraries, but no content. This works great while your user is navigating through your app, but for initial page load you need 2 round trips: One for the router and one for the current content page.

If you use the HTML5 History API to reflect the current content page in the URL, your server can know which content page will be requested by the client code. To save round trips the server can include the content chunk in the response: This is possible by just adding it as script tag. The browser will load both chunks parallel.

``` html
<script src="entry-chunk.js" type="text/javascript" charset="utf-8"></script>
<script src="3.chunk.js" type="text/javascript" charset="utf-8"></script>
```

You can extract the chunk filename from the stats. ([stats-webpack-plugin](https://www.npmjs.com/package/stats-webpack-plugin) could be used for exports the build stats)



## Multi-Page-App

When you compile a (real) multi page app, you want to share common code between the pages. In fact this is really easy with webpack: Just compile with multiple entry points:

`webpack p1=./page1 p2=./page2 p3=./page3 [name].entry-chunk.js`

``` javascript
module.exports = {
	entry: {
		p1: "./page1",
		p2: "./page2",
		p3: "./page3"
	},
	output: {
		filename: "[name].entry.chunk.js"
	}
}
```

This will generate multiple entry chunks: `p1.entry.chunk.js`, `p2.entry.chunk.js` and `p3.entry.chunk.js`. But additional chunks can be shared by them.

If your entry chunks have some modules in common, there is a cool plugin for this. The `CommonsChunkPlugin` identifies common modules and put them into a commons chunk. You need to add two script tags to your page, one for the commons chunk and one for the entry chunk.

``` javascript
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
	entry: {
		p1: "./page1",
		p2: "./page2",
		p3: "./page3"
	},
	output: {
		filename: "[name].entry.chunk.js"
	},
	plugins: [
		new CommonsChunkPlugin("commons.chunk.js")
	]
}
```

This will generate multiple entry chunks: `p1.entry.chunk.js`, `p2.entry.chunk.js` and `p3.entry.chunk.js`, plus one `commons.chunk.js`. First load `commons.chunk.js` and then one of the `xx.entry.chunk.js`.

You can generate multiple commons chunks, by selecting the entry chunks. And you can nest commons chunks.

``` javascript
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
	entry: {
		p1: "./page1",
		p2: "./page2",
		p3: "./page3",
		ap1: "./admin/page1",
		ap2: "./admin/page2"
	},
	output: {
		filename: "[name].js"
	},
	plugins: [
		new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
		new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
	]
};
// <script>s required:
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js
```

Advanced hint: You can run code inside the commons chunk:

``` javascript
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
	entry: {
		p1: "./page1",
		p2: "./page2",
		commons: "./entry-for-the-commons-chunk"
	},
	plugins: [
		new CommonsChunkPlugin("commons", "commons.js")
	]
};
```

See also [multiple-entry-points example](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points) and [advanced multiple-commons-chunks example](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks).