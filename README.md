# mkver

**Easy access to your version and build metadata from within
[Node.js](https://nodejs.org/)**

[![npm version](https://badge.fury.io/js/mkver.svg)](https://badge.fury.io/js/mkver)
[![Build Status](https://travis-ci.org/mceachen/mkver.svg?branch=master)](https://travis-ci.org/mceachen/mkver)
[![Build status](https://ci.appveyor.com/api/projects/status/6kw1acvoeuw02d4g/branch/master?svg=true)](https://ci.appveyor.com/project/mceachen/mkver/branch/master)
[![Language grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/mceachen/mkver.svg)](https://lgtm.com/projects/g/mceachen/mkver/context:javascript)

## Why

I need simple, reliable access to my version and build information from within
my node and Electron apps. Even if I push git SHAs into my `package.json`, after
minification, `asar`ification and installation into who-knows-where
platform-specific directory structures, I didn't want to (continue to) fight
`__dirname` bugs trying to find where my `package.json` went. I'm not getting
any younger here, people, I just want something that works. So what works?

In TypeScript and ES6 Module worlds, there's a super simple,
compatible-with-minification solution to importing information from outside your
current file, and it's great.

It's called `import`. Or for you old-skool kids, `require`.

If we can write build-specific information as constants in our codebase,
consumption of this metadata becomes trivial. Add it to your build pipeline,
import the thing, and then solve the Big Problems.

## What

`mkver` produces a `Version.ts` (by default), or a `version.js` (if you're writing
ECMAScript) with your git SHA and version information exported as constants.

For extra credit, it also creates a [SemVer-compatible `release`
tag](https://semver.org/#spec-item-10) that looks like
`${version}+${YYYYMMDDhhmmss of gitDate}`, and a `gitDate`, which is a `Date`
instance of when that last git commit happened.

## Installation

### Step 1: add `mkver` to your package.json

`npm i --save-dev mkver` or `yarn add -D mkver`

### Step 2: For TypeScript users

Add a `precompile` or `prebuild` npm script to your `package.json` that runs
`mkver`:

```json
  "scripts": {
    ...
    "precompile": "mkver",
    "compile": "tsc",
    ...
  }
```

### Step 2: For ECMAScript users

Add `mkver` as a `pre` script for your test script and/or your
webpack/gulp/grunt/browserify pipeline in your `package.json`.

```json
  "scripts": {
    ...
    "prebuild": "mkver ./lib/version.js",
    "build": "webpack",
    ...
  }
```

### Step 3: Add to .gitignore

I recommend adding your `Version.ts` or `version.js` file to your project's
`.gitignore`, but that isn't a requirement.

## How

`mkver` is a pretty simple three-step, one-trick pony:

1. `mkver` first looks for a `package.json` in `.`, then `..`, then `../..`,
   etc, and extracts the `version` value.
2. `mkver` then
   [execs](https://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback)
   `git rev-parse HEAD` to get the last commit SHA. Having `git` available to
   the calling shell is a prerequisite. Please don't file a bug report for this.
3. Finally, `mkver` writes the contents to the first argument given to `mkver`,
   which can include a subdirectory. The default output is `./Version.ts`.
   Existing files with that name will be overwritten. `mkver` uses the file
   extension to determine what format (TypeScript or es6) to render the output.

If anything goes wrong, expect output on `stderr`, and a non-zero exit code.

## Example output

Version files will have the following fields exported:

```ts
/** from your package.json: */
export const version: string = "1.0.0"

/** from `git rev-parse HEAD` */
export const gitSha: string = "bfed72637e3bb3b1f5d4c677909fce85e9258b3a"

/** Time of last commit, rendered as a Date */
export const gitDate: Date = new Date(1519003153587)

/** `version` + gitDate, rendered as YYYYMMDDhhmmss: */
export const release: string = "1.0.0+20180919202444"
```

### With TypeScript or MJS Modules

```ts
import { release } from "./Version"
```

### With <= ES6 javascript

```js
const { release } = require("./version") // < mind the case matches whatever you give mkver
```

Remember to `mkver version.js` in your npm script (see the Installation's "Step 2" above!)

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
