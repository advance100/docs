*Note that Hot Module Replacement (HMR) is still an experimental feature.*

# Introduction

Hot Module Replacement (HMR) is a way of exchanging modules in a running application (and adding/removing modules). You basically can update changed modules without a full page reload.

## Prerequirements

* Using Plugins: http://webpack.github.io/docs/using-plugins.html
* Code Splitting: http://webpack.github.io/docs/code-splitting.html
* webpack-dev-server: http://webpack.github.io/docs/webpack-dev-server.html

## How does it work?

### From the app view

The app code asks the HMR runtime to check of updates. The HMR runtime downloads the updates (async) and tell the app code that an update is available. The app code asks the HMR runtime to apply updates.  The HMR runtime applies the update (sync). The app code may or may not require user interaction in this process (you decide).

### From the compiler (webpack) view

In addition to the normal assets the compiler need to emit the "Update" to allow updating from previous version to this version. The "Update" contains two parts:

1. the update manifest (json)
2. one or multiple update chunks (js)

The manifest contains the new compilation hash and a list of all update chunks (2.).

The update chunks contains code for all updated modules in this chunk (or a flag if a module was removed).

The compiler addtionally makes sure that module and chunk ids as consistent between these builds. It uses a "records" json file to store them between builds (on it store them in memory).

### From the module view

HMR is a opt-in feature, so it only affects modules that contains HMR code. The documentation describes the API that is available in modules. In general the module developer writes handlers that are called when a dependency of this module is updated. He can also write a handler that are called when this module is updated.

In most cases it's not mandatory to write HMR code in every module. If a module has no HMR handlers the update bubbles up. This means a single handler can handle a update to a complete module tree. If a single module in this tree is updated, the complete module tree is reloaded (only reloaded not transferred).

### From the HMR runtime view (technical)

For the module system runtime is additional code emitted to track module `parents` and `children`.

On the management side the runtime supports two methods: `check` and `apply`.

A `check` does a HTTP request to the update manifest. When this request fails, there is no update available. Otherwise the list of updated chunks is compared to the list of currently loaded chunks. For each loaded chunk the corresponding update chunk is downloaded. All module updates as stored in the runtime as update. The runtime switches into the `ready` state, meaning an update has been downloaded and is ready to be applied.

For each new chunk request in the ready state the update chunk is also downloaded.

The `apply` method flags all updated modules as invalid. For each invalid module there need to be a update handler in the module or update handlers in every parent. Else the invalid bundles up and mark all parents as invalid too. This process continues until no more "bubble up" occurs. If it bubbles up from an entry point the process fails.

Now all invalid modules are disposed (dispose handler) and unloaded. Then the current hash is updated and all "accept" handlers are called. The runtime switches back to the `idle` state and everything continues as normal.

### Generated files (technical)

The left side represents the initial compiler pass. The right side represents an additional pass with module 4 and 9 updated.

![generated update chunks](http://webpack.github.io/assets/HMR.svg)

## What can I do with it?

You can use it in development as LiveReload replacement. Actually the webpack-dev-server supports a hot mode which try to update with HMR before trying to reload the whole page. You only need to add the `webpack/hot/dev-server` entry point and call the dev-server with `--hot`.

You can also use it in production as update mechanisms. Here you need to write you own management code that integrates HMR with your app.

Some loaders already generate modules that are hot-updateable. I. e. the `style-loader` can exchange the stylesheet. You don't need to do something special.


## What is needed to use it?

A module can only be updated if you "accept" it. So you need to `module.hot.accept` the module in the parents or the parents of the parents... I. e. a Router is a good place or a subview.

If you only want to use it with the webpack-dev-server, just add `webpack/hot/dev-server` as entry point. Else you need some HMR management code that calls `check` and `apply`.

You need to enable records in the Compiler to track module id between processes. (watch mode and the webpack-dev-server keep records in memory, so you don't need it for development)

You need to enable HMR in the Compiler to let it add the HMR runtime.


## What makes it so cool?

* It's LiveReload but for every module kind.
* You can use it in production.
* The updates respect your Code Splitting and only download updates for the used parts of your app.
* You can use in for a part of your application and it doesn't affect other modules
* If HMR is disabled all HMR code is removed by the compiler (wrap it in `if(module.hot)`


## Caveats

* It's experimental and not tested so well.
* Expect some bugs
* Theoretically usable in production, but it maybe to early to use it for something serious
* The module ids need to be tracked between compilations so you need to store them (`records`)
* Optimizer cannot optimize module ids anymore after the first compilation. A bit impact on bundle size.
* HMR runtime code increase bundle size.
* For production usage additional testing is required to test the HMR handlers. This could be pretty difficult.


# Tutorial

To use hot code replacement with webpack you need to four things:

* records (`--records-path`, `recordsPath: ...`)
* global enable hot code replacement (`HotModuleReplacementPlugin`)
* hot replacement code in your code `module.hot.accept`
* hot replacement management code in your code `module.hot.check`, `module.hot.apply`

A small testcase:

``` css
/* style.css */
body {
	background: red;
}
```

``` javascript
/* entry.js */
require("./style.css");
document.write("<input type='text' />");
```

That's enough to use hot code replacement with the dev-server.

``` sh
npm install webpack webpack-dev-server -g
npm install webpack css-loader style-loader
webpack-dev-server webpack/hot/dev-server ./entry --hot --module-bind "css=style!css"
```

The dev server provides in memory records, which is good for development.

There is special management code for the dev-server at `webpack/hot/dev-server`.

The `style-loader` already includes hot replacement code.

If you visit [http://localhost:8080](http://localhost:8080) you should see the page with a red background and a input box. Type some text into the input box and edit `style.css` to have another background color. 

Voilà... The background updates but without full page refresh. Text and selection in the input box should stay.

Read more about how to write you own hot replacement (management) code: [[hot module replacement]]

Check the [example-app](http://webpack.github.io/example-app/) for a demo without coding