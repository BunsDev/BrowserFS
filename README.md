# DEPRECATED

_22 March 2024_  
_Active and updated fork: [ZenFS](https://github.com/zen-fs)_

Hey there, this is the current maintainer of BrowserFS.

I've been working on BrowserFS for over a year now, and have made some big improvements.

A few months ago, I began seperating some backends from BrowserFS to be placed into different packages. Along with this, I created the browser-fs organization on Github and the browserfs organization on NPM. I made new organizations to keep the repositories and packages organized. Another reason I did so was because John Vilk, the original author of the project, no longer had the time to administer the project. This means at the time of writing this, I still do not have access to the NPM package.

I feel that now, the project has changed so significantly it is no longer the same BrowserFS. Even the name, *Browser*FS, implies it is meant for browsers, which it has outgrown. For that reason, I think that the project should be something new, but still carry the legacy of BrowserFS. I've decided to call it **ZenFS**, since a core goal of mine is ease of use and peace of mind.

In a letter to Robert Hooke in 1675, Isaac Newton famously wrote "If I have seen further it is by standing on the shoulders of giants". This is most certainly true in the case of ZenFS. Without the creation of BrowserFS, I would not have found it and been amazed at a complete file system in Typescript/Javascript.

I would like to extend my deepest thanks to Dr. Emery Berger. Shortly after submitting my first pull request to BrowserFS, I reached out about becoming a maintainer of the project. Dr. Berger welcomed my maintainership of the project, and greatly helped in some other matters.

Thank you very much to the community for helping me with this project by submitting issues and pull requests.

The NPM organization `browserfs` has had all of its packages deprecated, with a message pointing to the new package as well as this notice. All of the versions published under `@browserfs` have also been published under `@zenfs`.

I hope that ZenFS can continue the legacy of BrowserFS, and can reach the same popularity and reliability.

Until next time,  
**James P.**  
a.k.a. Dr. Vortex  
BrowserFS maintainer  
[ZenFS](https://github.com/zen-fs) creator

# BrowserFS

BrowserFS is an in-browser file system that emulates the [Node JS file system API](http://nodejs.org/api/fs.html) and supports storing and retrieving files from various backends. BrowserFS also integrates nicely into the Emscripten file system.

### Backends

BrowserFS is highly extensible, and ships with many filesystem backends:

-   `HTTPRequest`: Downloads files on-demand from a webserver using `fetch`.
-   `LocalStorage`: Stores files in the browser's `localStorage`.
-   `IndexedDB`: Stores files into the browser's `IndexedDB` object database.
-   `Dropbox`: Stores files into the user's Dropbox account.
    -   Note: You provide this filesystem with an authenticated [DropboxJS V2 JS SDK client](https://github.com/dropbox/dropbox-sdk-js).
-   `InMemory`: Stores files in-memory. Thus, it is a temporary file store that clears when the user navigates away.
-   `ZipFS`: Read-only zip file-backed FS. Lazily decompresses files as you access them.
    -   Supports DEFLATE out-of-the-box.
    -   Have super old zip files? [The `browserfs-zipfs-extras` package](https://github.com/jvilk/browserfs-zipfs-extras) adds support for EXPLODE, UNREDUCE, and UNSHRINK.
-   `IsoFS`: Mount an .iso file into the file system.
    -   Supports Microsoft Joliet and Rock Ridge extensions to the ISO9660 standard.
-   `WorkerFS`: Lets you mount the BrowserFS file system configured in the main thread in a WebWorker, or the other way around!
-   `MountableFileSystem`: Lets you mount multiple file systems into a single directory hierarchy, as in -nix-based OSes.
-   `OverlayFS`: Mount a read-only file system as read-write by overlaying a writable file system on top of it. Like Docker's overlayfs, it will only write changed files to the writable file system.
-   `AsyncMirror`: Use an asynchronous backend synchronously. Invaluable for Emscripten; let your Emscripten applications write to larger file stores with no additional effort!
    -   Note: Loads the entire contents of the file system into a synchronous backend during construction. Performs synchronous operations in-memory, and enqueues them to be mirrored onto the asynchronous backend.
-   `FolderAdapter`: Wraps a file system, and scopes all interactions to a subfolder of that file system.
-   `Emscripten`: Lets you mount Emscripten file systems inside BrowserFS.

More backends can be defined by separate libraries, so long as they extend they implement `BrowserFS.FileSystem`. Multiple backends can be active at once at different locations in the directory hierarchy.

For more information, see the [API documentation for BrowserFS](https://jvilk.com/browserfs/2.0.0-beta/index.html).

### Building

Prerequisites:

-   Node and NPM
-   Run `npm install` (or the equivilent command using your package manager) to install local dependencies.

After running `npm run build`, you can find built versions in the `dist` directory.

Custom builds (not recommended):

If you want to build BrowserFS with a subset of the available backends,
change `src/core/backends.ts` to include only the backends you require,
and re-build.

### Usage

> 🛈 The examples are written in ESM. If you aren't using ESM, you can add `<script src="browserfs.min.js"></script>` to your HTML and use BrowserFS via the global `BrowserFS` object.

BrowserFS provides a convient `configure` function which you can use to easily configure BrowserFS to use a variety of file system types.

Here's a simple usage example using the LocalStorage-backed file system:

```js
import { configure, BFSRequire } from 'browserfs';

// you can also add a callback as the last parameter instead of using promises
await configure({ fs: 'LocalStorage' });

const fs = BFSRequire('fs');

// Now, you can write code like this:

fs.writeFile('/test.txt', 'Cool, I can do this in the browser!', function (err) {
	fs.readFile('/test.txt', function (err, contents) {
		console.log(contents.toString());
	});
});
```

The following code mounts a zip file to `/zip`, in-memory storage to `/tmp`, and IndexedDB browser-local storage to `/home`:

```js
import { configure, BFSRequire } from 'browserfs';
import Buffer from 'buffer';

const zipData = await (await fetch('mydata.zip')).arrayBuffer();

await configure({
	fs: 'MountableFileSystem',
	options: {
		'/mnt/zip': {
			fs: 'ZipFS',
			options: {
				zipData: Buffer.from(zipData)
			}
		},
		'/tmp': { fs: 'InMemory' },
		'/home': { fs: 'IndexedDB' }
	}
};
```

### Using with Browserify and Webpack

BrowserFS is published as a UMD module, so you can either include it on your webpage in a `script` tag or bundle it with your favorite
JavaScript module bundler.

You can also use BrowserFS to supply your application with `fs`, `path`, and `buffer` modules, as well as the `Buffer` and `process`
globals. BrowserFS contains shim modules for `fs`, `buffer`, `path`, and `process` that you can use with Webpack and Browserify.

Webpack:

```javascript
module.exports = {
	resolve: {
		// Use our versions of Node modules.
		alias: {
			fs: 'browserfs/dist/shims/fs.js',
			buffer: 'browserfs/dist/shims/buffer.js',
			path: 'browserfs/dist/shims/path.js',
			processGlobal: 'browserfs/dist/shims/process.js',
			bufferGlobal: 'browserfs/dist/shims/bufferGlobal.js',
			bfsGlobal: require.resolve('browserfs'),
		},
	},
	// REQUIRED to avoid issue "Uncaught TypeError: BrowserFS.BFSRequire is not a function"
	// See: https://github.com/jvilk/BrowserFS/issues/201
	module: {
		noParse: /browserfs\.js/,
	},
	plugins: [
		// Expose BrowserFS, process, and Buffer globals.
		// NOTE: If you intend to use BrowserFS in a script tag, you do not need
		// to expose a BrowserFS global.
		new webpack.ProvidePlugin({ BrowserFS: 'bfsGlobal', process: 'processGlobal', Buffer: 'bufferGlobal' }),
	],
	// DISABLE Webpack's built-in process and Buffer polyfills!
	node: {
		process: false,
		Buffer: false,
	},
};
```

Browserify:

```javascript
var browserfsPath = require.resolve('browserfs');
var browserifyConfig = {
	// Override Browserify's builtins for buffer/fs/path.
	builtins: Object.assign({}, require('browserify/lib/builtins'), {
		buffer: require.resolve('browserfs/dist/shims/buffer.js'),
		fs: require.resolve('browserfs/dist/shims/fs.js'),
		path: require.resolve('browserfs/dist/shims/path.js'),
	}),
	insertGlobalVars: {
		// process, Buffer, and BrowserFS globals.
		// BrowserFS global is not required if you include browserfs.js
		// in a script tag.
		process: function () {
			return "require('browserfs/dist/shims/process.js')";
		},
		Buffer: function () {
			return "require('buffer').Buffer";
		},
		BrowserFS: function () {
			return "require('" + browserfsPath + "')";
		},
	},
};
```

### Using with Node

You can use BrowserFS with Node. Simply add `browserfs` as an NPM dependency, and `require('browserfs')`.
The object returned from this action is the same `BrowserFS` global described above.

If you need BrowserFS to return Node Buffer objects (instead of objects that implement the same interface),
simply `require('browserfs/dist/node/index')` instead.

### Using with Emscripten

You can use any _synchronous_ BrowserFS file systems with Emscripten!
Persist particular folders in the Emscripten file system to `localStorage`, or enable Emscripten to synchronously download files from another folder as they are requested.

Include `browserfs.min.js` into the page, and configure BrowserFS prior to running your Emscripten code. Then, add code similar to the following to your `Module`'s `preRun` array:

```javascript
/**
 * Mounts a localStorage-backed file system into the /data folder of Emscripten's file system.
 */
function setupBFS() {
	// Grab the BrowserFS Emscripten FS plugin.
	var BFS = new BrowserFS.EmscriptenFS();
	// Create the folder that we'll turn into a mount point.
	FS.createFolder(FS.root, 'data', true, true);
	// Mount BFS's root folder into the '/data' folder.
	FS.mount(BFS, { root: '/' }, '/data');
}
```

Note: Do **NOT** use `BrowserFS.install(window)` on a page with an Emscripten application! Emscripten will be tricked into thinking that it is running in Node JS.

If you wish to use an asynchronous BrowserFS backend with Emscripten (e.g. Dropbox), you'll need to wrap it into an `AsyncMirror` file system first:

```javascript
/**
 * Run this prior to starting your Emscripten module.
 * @param dropboxClient An authenticated DropboxJS client.
 */
function asyncSetup(dropboxClient, cb) {
	// This wraps Dropbox in the AsyncMirror file system.
	// BrowserFS will download all of Dropbox into an
	// InMemory file system, and mirror operations to
	// the two to keep them in sync.
	BrowserFS.configure(
		{
			fs: 'AsyncMirror',
			options: {
				sync: {
					fs: 'InMemory',
				},
				async: {
					fs: 'Dropbox',
					options: {
						client: dropboxClient,
					},
				},
			},
		},
		cb
	);
}
function setupBFS() {
	// Grab the BrowserFS Emscripten FS plugin.
	var BFS = new BrowserFS.EmscriptenFS();
	// Create the folder that we'll turn into a mount point.
	FS.createFolder(FS.root, 'data', true, true);
	// Mount BFS's root folder into the '/data' folder.
	FS.mount(BFS, { root: '/' }, '/data');
}
```

### Testing

To run unit tests, simply run `npm test`.

### Citing

BrowserFS is a component of the [Doppio](http://doppiojvm.org/) and [Browsix](https://browsix.org/) research projects from the PLASMA lab at the University of Massachusetts Amherst. If you decide to use BrowserFS in a project that leads to a publication, please cite the academic papers on [Doppio](https://dl.acm.org/citation.cfm?doid=2594291.2594293) and [Browsix](https://dl.acm.org/citation.cfm?id=3037727):

> John Vilk and Emery D. Berger. Doppio: Breaking the Browser Language Barrier. In
> _Proceedings of the 35th ACM SIGPLAN Conference on Programming Language Design and Implementation_
> (2014), pp. 508–518.

```bibtex
@inproceedings{VilkDoppio,
	author		= {John Vilk and
							 Emery D. Berger},
	title		 = {{Doppio: Breaking the Browser Language Barrier}},
	booktitle = {Proceedings of the 35th {ACM} {SIGPLAN} Conference on Programming Language Design and Implementation},
	pages		 = {508--518},
	year			= {2014},
	url			 = {http://doi.acm.org/10.1145/2594291.2594293},
	doi			 = {10.1145/2594291.2594293}
}
```

> Bobby Powers, John Vilk, and Emery D. Berger. Browsix: Bridging the Gap Between Unix and the Browser. In _Proceedings of the Twenty-Second International Conference on Architectural Support for Programming Languages and Operating Systems_ (2017), pp. 253–266.

```bibtex
@inproceedings{PowersBrowsix,
	author		= {Bobby Powers and
							 John Vilk and
							 Emery D. Berger},
	title		 = {{Browsix: Bridging the Gap Between Unix and the Browser}},
	booktitle = {Proceedings of the Twenty-Second International Conference on Architectural
							 Support for Programming Languages and Operating Systems},
	pages		 = {253--266},
	year			= {2017},
	url			 = {http://doi.acm.org/10.1145/3037697.3037727},
	doi			 = {10.1145/3037697.3037727}
}
```

### License

BrowserFS is licensed under the MIT License. See `LICENSE` for details.
