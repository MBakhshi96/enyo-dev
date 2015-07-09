# Changes in this release

#### INSTALLATION / UPGRADE INSTRUCTIONS

Before you do anything else, ensure that you follow these directions when upgrading from your current version of _enyo-dev_ to the latest release. The tools still require a local copy of the source. Note that this version requires node version 0.12.2 or higher. To check your version of node use the following command:

```bash
node --version
```

##### Clean Install

If you follow these instructions for a clean install on a system that has never had _enyo-dev_ then it will be easy to upgrade or test patch-fixes.

```bash
git clone https://github.com/enyojs/enyo-dev.git
cd enyo-dev
npm install
npm link
```

##### Upgrade

If you already have a local clone of _enyo-dev_ and need to upgrade follow these instructions from the root of the local repository.

```bash
git pull
npm install
npm prune
npm link
```


## Overview

* [Removed dependency on browserify](#1)
* [Removed dependency on bower](#2)
* [Added the ability to generate packaged library output](#3)
* [Added the ability to access modules from the web console](#4)
* [Added new command `enyo` with accessible sub-commands](#5)
	* [`enyo init`](#enyo-init)
	* [`enyo link`](#enyo-link)
	* [`enyo unlink`](#enyo-unlink)
	* [`enyo find-links`](#enyo-find-links)
	* [`enyo config`](#enyo-config)
	* [`enyo serve`](#enyo-serve)
	* [`enyo pack`](#enyo-pack)
* [Added dynamic asset path expansion in JavaScript source](#6)
* [Added the ability to use the `--watch` (auto-rebuild) feature without using `enyo-serve`](#7)
* [Added the ability to create on-demand loaded _bundles_ of modules via the `request` function (EXPERIMENTAL)](#8)
* [Added more useful debugging information](#9)
* [Added better error handling and reporting](#9)
* [Added new logging utility capable of filtering output and outputing logs to filterable files](#9)
* Fixed `enyo-serve`
* [Example on accessing private repositories](#10)
 
 
### Notes

* Currently 32-bit systems can't build one of the dependencies and therefore can't use the tools
* The `request` feature is still considered experimental and should be used with caution
* The utility used to derive Git uri's is limited to basic git-ssh/https protocol formats at this time (e.g. `git@github.com:enyojs/enyo.git#master` and `https://github.com/enyojs/enyo.git#2.6.0-dev` type uri's)

## Details

###### <a name="1"></a>Removed dependency on browserify

While the browserify project is very useful and generally extensible, it is slow and did not expose some of the features we needed most. Now we control our own dependency bundler that manages our source and packaging needs.

###### <a name="2"></a>Removed dependency on bower

We no longer use the bower project underneath our `egen init` and `enyo init` commands. This means you can safely remove the `bower.json` and `.bowerrc` files from your project. There is a new way to specify libraries used by your projects and is discussed below with the addition of the `enyo` command and sub-commands (namely `enyo init` and `enyo config`).

###### <a name="3"></a>Added the ability to generate packaged library output

You can now generate the complete, includable output from any Enyo 2.6+ library. You do this with the `--library` flag to `epack` or `enyo pack`. Most builds produce an `index.html` file and only include the modules as _required_ by any modules and an _entry file_ (e.g. index.js). When building a library, it scans and includes __all__ modules contained in the library source. For example, to build the complete, includable output from the Enyo library, navigate to the project root.

```bash
cd enyo
enyo pack --library
# check the ./dist directory for the output
```

###### <a name="4"></a>Added the ability to access modules from the web console

Previously you would need to provide your own code to expose module content for debugging in the web console. Now this ability exists out of the box. For any library module you access (e.g. `enyo/kind`) you can access it the same way in the console via the `require` function directly. For your project modules just remember the path is relative to the project root, without the leading './'. So, for example, in the [enyo-strawman](https://github.com/enyojs/enyo-strawman/) project there is a module located at `./src/svg-samples` that exports an object with a few properties. To retrieve this same object at runtime in the web console use `require('src/svg-samples')`. It's that simple! Make sure that the module is actually included in the final source by another module or it won't be available in the output.

###### <a name="5"></a>Added new command `enyo` with accessible sub-commands

There is a new command, `enyo`, available once you've installed _enyo-dev_. It requires that you specify one of its child commands which can be listed using `enyo -h`. These helpers were designed specifically for the Enyo developers' workflow and to assist when working with modularized code bases.

> The `enyo-gen` and `egen` commands are now a direct alias to the _preferred_ `enyo` command but could be removed in a future release. You should now use `enyo` instead.

> The `enyo-serve`, `eserve`, `enyo-pack` and `epack` commands are now aliases for `enyo serve` and `enyo pack` respectively. It is still safe to use the aliases, however.

### Commands
##### <a name="enyo-init"></a>`enyo init`

> Previously `enyo-gen init` or `egen init`

This command is used to initialize a new Enyo project or re-initialize an existing project. Enyo now configures projects _two different ways_. For general configuration of projects and to disambiguate the use of the `package.json` file the tools use a new file `.enyoconfig`. The `enyo init` command will generate a global reference for defaults at `~/.enyoconfig` that can be modified. It will also create one in the root of your projects. The `.enyoconfig` file is JSON and is easily manipulated by hand or via the `enyo config` command. It supports a special set of keys which can be listed using `enyo config --list` or by reviewing any `.enyoconfig` generated with the defaults.

Of particular interest are the `"libraries"` and `"sources"` properties. The `"libraries"` array is a list of library names that should be installed for a particular project. The defaults list the most common Enyo related libraries. And there is no harm in having extra libraries aside from used disk-space. So, in most cases the defaults should be used. You can add/remove entries to the `"libraries"` array using `enyo config` as discussed below. The `"sources"` object maps library names to their respective remote repository.

##### <a name="enyo-link"></a>`enyo link`

If used without arguments will make the current library/project available to be linked as a library in other projects. Then, in another project you can use it again with the name of the library and it will be installed without copying the files. This is most handy for framework development but is also useful for developers in some debugging scenarios. This can be handled automatically by `enyo init` if the libraries are named in the `.enyoconfig` `"link"` array or the `"linkAllLibs"` property or command-line flag are used. Also see the `enyo find-links` command for convenient way to make multiple libraries linkable all at once. Note that using `enyo link` assumes you will control the current _target_ of the repository you are linking. You will need to ensure you have checked out the correct _branch_ or _tag_.

```bash
cd Devel/enyo
enyo link
cd ../projects/myproject
enyo link enyo
# you will now have lib/enyo setup and linked to Devel/enyo
```

##### <a name="enyo-unlink"></a>`enyo unlink`

This command is the counterpart to the `enyo link` command and conveniently unlinks the named library.

##### <a name="enyo-find-links"></a>`enyo find-links`

This is a helper command used to aid in finding and preparing libraries to be linked. For example, if you have a local clone of many Enyo libraries in a single directory simply `enyo find-links` from within that directory or pass it a relative path to the directory and it will walk you through the projects and ask if you want them to be linked. If you use `-i false` it won't ask and will assume you want all available libraries linkable.

```bash
cd Devel
enyo find-links -i false
cd projects/myproject
enyo link enyo
# you will now have lib/enyo setup and linked to enyo found in find-links
```

##### <a name="enyo-config"></a>`enyo config`

This is a convenience tool for modifying or accessing configuration settings for your global or local (project-level) configuration files (`~/.enyoconfig` and `.enyoconfig` respectively). When you run `enyo init` you will be prompted to create the file (first at the global level if you don't already have one) in your local project. Configurations are considered in a particular resolution order: local setting -> local default setting -> global setting -> global default setting. As you will see by reviewing any `.enyoconfig` generated with defaults it has an entry for each of the major known properties (use `enyo config --list` to see all of the available options).

In a project with the defaults in `.enyoconfig`:

```bash
enyo config --get libraries # -> undefined
enyo config --get defaults.libraries # -> 0=enyo, 1=layout, 2=canvas,...
enyo config --global --get libraries # -> undefined
enyo config --global --get defaults.libraries # -> 0=enyo, 1=layout, 2=canvas,...
```
The reason `--get libraries` was `undefined` is because there is no local entry for that property but there is a `defaults.libraries` entry. When running, the tools would find the defaults and use them but you can override the defaults by creating your own `"libraries"` array on the top level of the JSON manually or using the cli tool.

```bash
# add an entry to override defaults
enyo config libraries enyo
enyo config --get libraries # -> 0=enyo
enyo config --get defaults.libraries # -> 0=enyo, 1=layout, 2=canvas,...
enyo config -r libraries enyo
enyo config --get libraries # -> (nothing, no entries)
```
At this point if you run `enyo init` it would only attempt to install/copy/link `enyo` as that is the only _local entry_ for the property. You can add or remove in this manner or modify the `.enyoconfig` JSON directly. Make sure to read the `enyo config -h` help output for more details on the optional use-cases of `enyo config`.

Also note in the examples above we're modifying array entries and it handles that automatically. The same is true for other types such as booleans and strings:

```bash
enyo config linkAllLibs true
enyo config --get linkAllLibs # -> true
enyo config user.name "Cole Davis"
enyo config user.email cole.davis@lge.com
enyo config --get user.name # -> Cole Davis
enyo config --get user.email # -> cole.davis@lge.com
```

##### <a name="enyo-serve"></a>`enyo serve`

This is the preferred way to launch the test server. Still accessible through the aliases `enyo-serve` and `eserve`. Execute `enyo serve -h` to see the supported parameters.

##### <a name="enyo-pack"></a>`enyo pack`

This is the preferred replacement for the (still active) aliases `enyo-pack` and `epack`. Execute `enyo pack -h` to see the supported parameters.


###### <a name="6"></a>Added dynamic asset path expansion in JavaScript source

There are times when you need to reference an asset path directly in JavaScript. The build tools automatically correct asset paths in CSS/Less style code and copy assets referenced in the `package.json` `"assets"` array to the correct output location based on any relevant configuration/packaging options. Now, in JavaScript you can have the same dynamic benefit.

In order for the build tools to understand that you want to expand a path you have to follow these two rules:

1. Prefix the path with `@`
2. Use a relative path from the _current source file_ to the asset.

So, if you had a project structure like this:

```
assets/
  myfile.png
src/
  mymodule/
    assets/
      mysubfile.png
    index.js
    package.json
package.json
```

Assuming that both `package.json` files had `"assets": ["assets/**/*.png"]` in them (so all `.png` files would be copied into the build), the following code in `src/mymodule/index.js` would be correct:

```javascript
// for the correct final output path of assets/myfile.png
var myfilepath = '@../../assets/myfile.png';
// for the correct final output path of src/mymodule/assets/mysubfile.png
var mysubfilepath = '@./assets/mysubfile.png';
```

With standard configuration options, the final output file would contain:

```javascript
// for the correct final output path of assets/myfile.png
var myfilepath = 'assets/myfile.png';
// for the correct final output path of src/mymodule/assets/mysubfile.png
var mysubfilepath = 'src/mymodule/assets/mysubfile.png';
```

###### <a name="7"></a>Added the ability to use the `--watch` (auto-rebuild) feature without using `enyo-serve`

The ability to watch your source code for changes and automatically rebuilding the output has been moved to `enyo pack` when specifying the `--watch` flag. This means in an environment where you are already outputting your packaged applications to a web root you don't need to run `enyo serve` needlessly. Make sure to review `enyo pack -h` for all available options relating to the `--watch` command as different environments may require additional information.

###### <a name="8"></a>Added the ability to create on-demand loaded _bundles_ of modules via the `request` function (EXPERIMENTAL)

More documentation on this in an official release as this feature is experimental and under development.

###### <a name="9"></a>Added more useful debugging information

Packaging applications requires many complex steps and debugging possible errors can be painful. Sharing debugging output remotely can be even worse. This release added more useful debugging information that is highly versatile. By default, debugging is set to output fatal messages only, this way it can more efficiently do the work you expect it to. When it does produce logging information it is in JSON form. This makes it easier to store or transfer and it can be filtered.

> NOTE: For human-friendly form please `npm install -g bunyan`. This is __not required__ but is __extremely useful__.

Assuming you have `bunyan` installed globally you can do the following to see verbose output from the build tools in a human readable way.

```bash
epack -l info | bunyan -o short
```

And in _very tricky scenarios_ you could use

```bash
epack -l debug | bunyan -o short
```

This is only useful as long as you have the output in your terminal buffer. It is very easy to store the output to filter later or pass to another developer to help debug an issue.

```bash
# works in Windows, OS X and Linux
epack -l debug > build_log.txt
```

This can later be reviewed using the `bunyan` cli tool

```bash
bunyan -o short build_log.txt
```

The build tools separates its _steps_ into different _components_. This allows us to filter output by the _component_ of interest. For example, if you were debugging why it was not finding a particular module you could do the following:

```bash
epack -l debug > build_log.txt
bunyan -o short -c 'this.component == "process-source-stream"' build_log.txt
```

###### <a name="10"></a>Example on accessing private repositories

As stated above, adding libraries to your configuration is quite simple. Add the library name to the `"libraries"` configuration array and an entry mapping the name to its remote location on the `"sources"` configuration object.

__NOTE: There is currently a limitation in the tools that will not properly authenticate users for private repositories! Use the below method to get around this limitation while that implementation is being completed.__

1. Create a local clone of the private repository using standard `git` commands (e.g. `git clone https://github.com/organization/repository.git`)
2. From within that repository, run `enyo link`

Now whenever you need this library in a project, simply add it to the `"link"` and `"libraries"` arrays or manually link it yourself via `enyo link [repository]`.