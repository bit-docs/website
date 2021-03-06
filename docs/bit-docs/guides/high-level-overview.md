@parent guides 0
@page guides/high-level-overview High Level Overview
@outline 2

@description A high-level overview of bit-docs.

@body

<div class="on-this-page-container"></div>

bit-docs is an evolving set of tools that allow you to:

- Write documentation inline, or in markdown files.
- Specify your code's behavior precisely with JSDoc
   and [Google Closure Compiler](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler)
   annotations.
- Generate a ready-to-publish website from that documentation.

The `bit-docs` tool (this repo) orchestrates "finder" plugins that slurp in raw data with "processor" and "generator" plugins that spit out structured documents from that data. The `bit-docs` tool sits happily between such plugins, automating their install and cooperation.

Depending on the plugins used, input may be in the form of inline code comments, markdown files, or anything else you could imagine; output may be in the form of HTML, or any other format.

You could write "finder" and "generator" plugins for the `bit-docs` tool geared towards static site generation for a blog or generic website, but `bit-docs` is particularly useful for generating documention websites from and for code projects.

## Projects currently using bit-docs

 - [CanJS](https://github.com/canjs/canjs) — [Generated website](http://canjs.com)
 - [StealJS](https://github.com/stealjs/stealjs) — [Generated website](http://stealjs.com)
 - [DoneJS](https://github.com/donejs/donejs-next) — [Generated website](https://donejs.github.io/donejs-next)

## Usage 

It is possible to add the bit-docs package as a dependency to the actual project you wish to document, but we have found creating an entirely new dedicated repository that will pull in the codebase(s) that you wish to document is a better paradigm. So, for `your-project` you might create a new repository called `your-project-site`.

This paradigm of creating a repository dedicated to generating documentation with bit-docs is especially useful when pulling in multiple codebases and wishing to generate a unified website. This is the strategy used by StealJS and DoneJS to document their core functionality and the functionality of their various modules that are broken out into separate repos and packages!

To use bit-docs, add it to the `package.json` of the project you want to use it in:

```shell
npm install bit-docs --save-dev
```

Next, in your project's `package.json`, add a section called `bit-docs`, like:

```json
  "bit-docs": {
    "dependencies": {
      "bit-docs-glob-finder": "*",
      "bit-docs-dev": "*",
      "bit-docs-js": "*",
      "bit-docs-generate-html": "*"
    },
    "glob": {
      "pattern": "docs/**/*.{js,md}"
    },
    "parent": "indexfile",
    "minifyBuild": false
  }
```

If you created a new repo specifically to hold this bit-docs stuff, you may wish to add the codebases you will be documenting as normal `package.json` devDependencies at this time. You will need to update the `bit-docs` glob pattern to be similar to what the StealJS website repo does (to tell the glob finder to look in `node_modules` for files to process):

```json
    "glob": {
      "pattern": "{node_modules,doc}/{steal,grunt-steal,steal-*}/**/*.{js,md}",
      "ignore": [
        "node_modules/steal/test/**/*",
        "node_modules/steal-tools/test/**/*",
        "node_modules/steal-conditional/{demo,test,node_modules}/**/*"
      ]
	},
```

Under the hood, bit-docs uses `npm` to install the `dependencies` defined under the `bit-docs` configuration in `package.json`. However, instead of installing packages into your project's top-level `node_modules`, bit-docs utilizes npm to install plugin packages to it's own directory:

```
./your-project/node_modules/bit-docs/lib/configure/node_modules
```

Maintaining this nested `node_modules` directory gives `bit-docs` complete control over this subset of plugin packages, enabling things like the `-f` "force" flag to completely delete and reinstall the plugin packages. The force flag is particularly useful after updating the dependency list to remove a plugin package that has previously been installed (otherwise, that old dependency won't be removed from the nested `node_modules` and will still be included by bit-docs).

Look at the `.gitignore` of this repo; you'll notice an entry for `lib/configure/node_modules/`, and entries for other directories that will be generated on the fly when `bit-docs` is run.

Utilization of npm under the hood means things like the `file://` syntax for `dependencies` in the `bit-docs` configuration is fair game, which can be useful for local debugging of bit-docs plugins.

For more information on developing locally, see [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Plugins

There are four handlers that any given bit-docs plugin can hook into using a standardized `bit-docs.js` file in the root of the plugin's directory. A plugin could hook into all four of these actions at once but, following the unix philosophy, bit-docs plugins should strive to do one thing only, and do it well. Therefore, most plugins will only hook into one (or two) of these handlers to accomplish their intended task, but probably never all four.

### Finder

Plugins that hook into the `finder` handler affect how bit-docs searches for source-files.

The default finder supports glob syntax, and should be sufficient for most use-cases:

- <https://github.com/bit-docs/bit-docs-glob-finder>

You might need to create a plugin that hooks into the `finder` handler if you're pulling source from a database, or some other location that's not the current working filesystem.

### Processor

Plugins that hook into the `processor` handler may augment how found files are processed.

The following plugin is always included by default in the core of bit-docs:

- <https://github.com/bit-docs/bit-docs-process-tags>

This is because that plugin provides the extremely common task of processing "tags". A tag is an identifier, usually embedded in source code comments, that provides documentation or information about some functionality, inline in the source code itself. Some of the many default tags are `@function @parent @description @body`, used in a source-file like:

```js
/**
 * @function yourproject.hellofunc hellofunc
 * @parent YourProject.apis
 * @description This documents something in a sub-page of YourProject.
 *
 * @body
 *
 * ## Usage
 *
 * Explain how to use it in _markdown_!
 */
```

For an example of a processor plugin that's not included by default, see:

- <https://github.com/bit-docs/bit-docs-process-mustache>

### Generator

Plugins that hook into the `generator` handler output something from the processed data.

For example, see these bit-docs plugins that output HTML files:

- <https://github.com/bit-docs/bit-docs-generate-html>
- <https://github.com/bit-docs/bit-docs-generate-readme>

### Tag

Plugins that hook into the `tag` handler add new tags like `@yourtag` to the default processing that bit-docs already does to source-file comments.

For example, see these bit-docs plugins for prettifying source-code snippets:

- <https://github.com/bit-docs/bit-docs-prettify>
- <https://github.com/bit-docs/bit-docs-html-highlight-line>

## Plugins that Hook into Other Plugins

The previously mentioned `bit-docs-generate-html` is a "generator" plugin but also accepts hooks into itself, which allows the following plugin to add functionality to that plugin:

- <https://github.com/bit-docs/bit-docs-html-toc>

Take a look at `bit-docs-html-toc`'s [`bit-docs.js`](https://github.com/bit-docs/bit-docs-html-toc/blob/master/bit-docs.js) file to see how a plugin registers itself with another plugin.

Find more plugins at the [bit-docs organization on GitHub](https://github.com/bit-docs).

## Contributing

Want to help make `bit-docs` even better? See [`CONTRIBUTING.md`](CONTRIBUTING.md).

Looking for a changelog? Try the [releases page on GitHub](https://github.com/bit-docs/bit-docs/releases).
