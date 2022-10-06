# The Modern Guide to Packaging your JavaScript library

English | [简体中文](./README-zh_CN.md)

## Context about this guide

This guide is written to provide at-a-glance suggestions that most libraries should follow, while also giving additional information if you want to understand why a suggestion is made or if you feel like you need to deviate from these suggestions. This guide is meant for **libraries** and not for applications (apps).

To emphasize, this is a list of **suggestions** and is not meant to be definitive list that all libraries must follow, or to provide flame bait for libraries that do not follow these suggestions. Each library is unique and there are probably good reasons a library may chose to not implement any given suggestion.

Finally, this guide is not meant to be specific to any particular bundler - there already exist many guides on how to set configs in specific bundlers. Instead, we want to focus on things that apply to every library and bundler (or lack of bundler).

---

## Output to `esm`, `cjs`, and `umd` formats

<details>
<summary>Supporting the whole ecosystem</summary>

`esm` is short for "EcmaScript module."

`cjs` is short for "CommonJS module."

`umd` is short for "Universal Module Definition," and can be run by a raw `<script>` tag, or in CommonJS module loaders, or by `AMD` module loaders.

Without getting into the flame wars that generally happen around `esm` and `cjs` formats, `esm` is considered "the future" but `cjs` still has a strong hold on the community and ecosystem. `esm` is easier for bundlers to correctly treeshake, so it is especially important for libraries to have this format. It's also possible that some day in the future your library only needs to output to `esm`.

You may have noticed that `umd` is already compatible with CommonJS module loaders - so why would you want to have both `cjs` _and_ `umd` output? One reason is that CommonJS files generally perform better when _conditionally_ depended on compared to `umd` files; for example:

```js
if (process.env.NODE_ENV === "production") {
  module.exports = require("my-lib.production.js");
} else {
  module.exports = require("my-lib.development.js");
}
```

The above example, when used with CommonJS modules, will only end up with either the `production` or `development` bundle. However, with a UMD module, it may be the case that a developer would end up with _both_ bundles. Refer to [this discussion](https://github.com/frehner/modern-guide-to-packaging-js-library/issues/9) for more information.

Finally, if your library is stateful, be aware that this does open the possibility of your library running into the [dual package hazard](https://nodejs.org/api/packages.html#dual-package-hazard), which can occur in situations where a developer ends up with both a `cjs` and `esm` version of your library in their application. The "dual package hazard" article linked above describes some ways to mitigate this issue, and the `module` condition in [`package.json#exports`](#define-your-exports) can also help prevent this from happening.

</details>

## Output to multiple files

<details>
<summary>Better treeshaking by maintaining the file structure</summary>

If you use a bundler or transpilier in your library, it can be configured to output files in the same way that they were authored. This makes it easier to mark specific files as having [side effects](#set-the-sideeffects-field), which helps the developer's bundler with treeshaking. Refer to [this article](https://levelup.gitconnected.com/code-splitting-for-libraries-bundling-for-npm-with-rollup-1-0-2522c7437697) for more details.

An exception is if you are making a bundle meant to be used directly in the browser without _any_ bundler (commonly, these are `umd` bundles but could also be modern `esm` bundles as well). In this case, it is better to have the browser request a single large file than need to request multiple smaller ones. Additionally, you should [minify](#to-minify-or-not-to-minify) the bundle and create [sourcemaps](#create-sourcemaps) for it.

</details>

## To Minify or Not to Minify

<details>
<summary>Determine your preferred level of minification</summary>

There are certain levels of minification you can apply to your library, and depending on how aggressive you want to be will determine how small your code will be once it's finally through a developer's bundler.

For example, most bundlers are already configured to remove whitespace and other easy optimizations, even from an NPM module (in this case, your library). According to [Terser](https://github.com/terser/terser#terser-fast-minify-mode) - a popular JavaScript mangler/compressor - that type of compression can reduce your bundle's final size by up to 95%. In some cases, you may be happy with those savings with no effort on your part.

However, there are additional savings that can occur if you were to run a minifier on your library before publishing, but doing so requires deeply understanding the settings and side-effects of your minifer. These type of compressions are generally _not_ run by minifiers on NPM modules, and therefore you will miss out on those savings unless you do it yourself. Refer to [this issue](https://github.com/frehner/modern-guide-to-packaging-js-library/issues/9) for additional information.

Finally, if you are creating a bundle intended to be used directly in the browser without a bundler (commonly, these are `umd` bundles but could also be modern `esm` bundles as well), you should always minify your bundle, create [sourcemaps](#create-sourcemaps) for it, and output to a [single file](#output-to-multiple-files).

</details>

## Create sourcemaps

<details>
<summary>When using a bundler or transpiler, generate sourcemaps</summary>

Any sort of transformation of your source code to a bundle will produce errors that point at the wrong location in your code. Help your future self out and create sourcemaps, even if your transformations are small.

</details>

## Create TypeScript types

<details>
<summary>Types improve the developer experience</summary>

As the number of developers using TypeScript continues to grow, having types built-in to your library will help improve the developer experience (DX). Additionally, devs who are not using TypeScript also get a better DX when they use an editor that understands types (such as VSCode, which uses the types to power its [Intellisense feature](https://code.visualstudio.com/docs/editor/intellisense)).

However, creating types does NOT mean you must author your library in TypeScript.

One option is to continue using JavaScript in your source code and then also supplement it with [JSDoc](https://jsdoc.app/) comments. You would then configure TypeScript to only [emit the declaration files](https://www.typescriptlang.org/tsconfig/#emitDeclarationOnly) from your JavaScript source code.

Another option is to write the TypeScript type files directly, in an `index.d.ts` file.

Once you have the types file, make sure you set your [`package.json#exports`](#define-your-exports) and [`package.json#types`](#set-the-types-field) fields.

</details>

## Externalize frameworks

<details>
<summary>Don't include a copy of React, Vue, etc. in your bundle</summary>

When building a library that relies on a framework (such as React, Vue, etc.) or is a plugin for another library, you'll want to add that framework to your bundler's "externals" (or equivalent) configuration. This will make it so that your library will reference the framework but will not include it in the bundle. This will prevent bugs and also reduce the size of your library's package.

You should also add that framework to your library's `package.json`'s [peer dependencies](#set-your-peerdependencies), which will help developers discover that you rely on that framework.

</details>

## Target modern browsers

<details>
<summary>Use modern features and let devs support older browsers if needed</summary>

[This article on web.dev](https://web.dev/publish-modern-javascript/) makes a great case for your library to target modern features, and offers guidelines on how to:

- Enable developers to support older browsers when using your library
- Output multiple bundles that support various levels of browser support

As one example, if you're transpiling from TypeScript, you could create two versions of your package's code:

1. An `esm` version with modern JavaScript generated by setting `"target"="esnext"` in your `tsconfig.json`
2. A `umd` version with more broadly-compatible JavaScript generated by setting `"target"="es5"` in your `tsconfig.json`

With these settings, most users will get the modern code, but those using older bundler configurations or loading the code using a `<script>` tag will get the version with additional transpilation for older browser support.

</details>

## Transpile if necessary

<details>
<summary>Turn TypeScript or JSX into function calls</summary>
If your library's source code in in a format that requires transpilation, such as TypeScript, React or Vue components, etc., then your output should be transpiled. For example:

- Your TypeScript library should output JavaScript bundles
- Your React components like `<Example />` should output bundles that use `jsx()` or `createElement()` instead of JSX syntax.

When transpiling this way, make sure you [create sourcemaps](#create-sourcemaps) as well.

</details>

## Keep a changelog

<details>
<summary>Track updates and changes</summary>

It doesn't matter whether it's through automatic tooling or through manual process, as long as developers have a way to see what has changed and how it affects them. Ideally, every change to your library's [version](#set-the-version-field) has a corresponding update in your changelog.

</details>

## Split out your CSS files

<details>
<summary>Enable devs to only include the CSS they need</summary>

If you are creating a CSS library (like Bootstrap, Tailwind, etc.), it may be easier to provide a single CSS bundle that includes all the functionality that your library provides. However, even in that situation, your CSS bundle may end up becoming large enough that it affects the performance of the devs' sites. To help prevent that, libraries generally provide methods of generating a CSS bundle that only includes the necessary CSS for what the developer is using (for example, see how [Bootstrap](https://getbootstrap.com/docs/5.2/customize/optimize/) and [Tailwind](https://tailwindcss.com/docs/optimizing-for-production) do it).

If CSS is only a part of what your library exposes (for example, a component library that has default styles), then it is ideal if you separate out your CSS into individual bundles per component that are imported when the corresponding component is used. One example of this is [react-component](https://github.com/react-component/slider#usage).

</details>

## `package.json` settings

There are a lot of important settings and fields to talk about in `package.json`; I will highlight the most important ones here, but be aware that there are [additional fields](https://docs.npmjs.com/cli/v8/configuring-npm/package-json) that you can set as well.

### Set the `name` field

<details>
<summary>Give a name to your library</summary>

The `name` field will determine the name of your package on `npm`, and therefore the name that developers will use to install your library.

Note that there are restrictions on what you can name your library, and additionally you can add a "scope" if your library is part of an organization. Refer to the [name docs on npm](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#name) for more details.

The `name` and the [version](#set-the-version-field) fields combine to create a unique identifier for each iteration of your library.

</details>

### Set the `version` field

<details>
<summary>Publish updates to your library by changing the version</summary>

As noted in the [name](#set-the-name-field) section, the name and the version combine to create a unique identifier for your library on npm. When you make updates to the code in your library, you can then update the `version` field and publish to allow developers to get that new code.

Libraries are encouraged to use a versioning strategy called [semver](https://semver.org/), but note that some libraries choose to [calver](https://calver.org/) or their own unique versioning strategy. Whichever strategy you choose to use, you should document it so that developers understand how your library's versioning works.

You should also keep track of your changes in a [changelog](#keep-a-changelog).

</details>

### Define your `exports`

<details>
<summary><code>exports</code> define the public API for your library</summary>

The `exports` field on `package.json` - sometimes called "package exports" - is an incredibly useful addition, though it does add some complexity. The two most important things that it does is:

1. Defines what can and cannot be imported from your library, and what the name of it is. If it's not listed in `exports`, then developers cannot `import`/`require` it. In other words, it acts like a public API for users of your library and helps define what is public and what is internal.
2. Allows you to change which file is imported based on conditions (that you can define), such as "Was the file `import`ed or `require`d? Does the developer want a `development` or `production` version of my library?" etc.

There are some good docs from the [NodeJS team](https://nodejs.org/api/packages.html#package-entry-points) and the [Webpack team](https://webpack.js.org/guides/package-exports/) on the possibilities here. I'll provide one example that covers the most common use-cases:

```json
{
  "exports": {
    ".": {
      "types": "index.d.ts",
      "module": "index.js",
      "import": "index.js",
      "require": "index.cjs",
      "default": "index.js"
    },
    "./package.json": "./package.json"
  }
}
```

Let us dive into the meaning of these fields and why I chose this specific shape:

- `"."` indicates the default entry for your package
- The resolution happens from **top to bottom** and stops as soon as a matching field is found; the order of entries is very important
- The `types` field should always [come first](https://devblogs.microsoft.com/typescript/announcing-typescript-4-7/#package-json-exports-imports-and-self-referencing), and helps TypeScript find the types file
- The `module` field is an "unofficial" field that is supported by bundlers like Webpack and Rollup. It should come before `import` and `require`, and point to an `esm`-only bundle -- which can be the same as your original `esm` bundle if it's purely `esm`. As noted in the [formats section](#output-to-esm-cjs-and-umd-formats), it is meant to help bundlers only include one copy of your library, no matter if it was `import`ed or `require`ed. For a deeper dive and the reasoning behind this decision, you can read more [here](https://github.com/webpack/webpack/issues/11014#issuecomment-641550630), [here](https://github.com/webpack/webpack/issues/11014#issuecomment-643256943), and [here](https://github.com/rollup/plugins/pull/540#issuecomment-692078443).
- The `import` field is for when someone `import`s your library
- The `require` field is for when someone `require`s your library
- The `default` field is used as a fallback for if none of the conditions match. While it may not be used at the moment, it's good to have it for ["unknown future situations"](https://webpack.js.org/guides/package-exports/#common-patterns)

If a bundler or environment understands the `exports` field, then the `package.json`'s top-level [main](#set-the-main-field), [types](#set-the-types-field), [module](#set-the-module-field), and [browser](#set-the-browser-field) fields are ignored, as `exports` supersedes those fields. However, it's still importantant to set those fields, for tools or runtimes that do not yet understand the `exports` field.

If you have a "development" and a "production" bundle (for example, you have warnings in the development bundle that don't exist in the production bundle), then you can also set them in the `exports` field with `"development"` and `"production"`. Note that some bundlers like `webpack` and `vite` will recognize these conditions automatically; however, while Rollup [can be configured](https://github.com/rollup/plugins/tree/master/packages/node-resolve/#exportconditions) to recognize them, that is something that you would have to instruct developers to do in their own bundler config.

</details>

### List the `files` to be published

<details>
<summary><code>files</code> defines which files are included in your NPM package</summary>

The [`files`](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#files) field indicates to the `npm` CLI which files and folders to include when you package your library to be put on NPM's package registry.

For example, if you transform your code from TypeScript into JavaScript, you probably don't want to include the TypeScript source code in your NPM package. (Instead, you should include [sourcemaps](#create-sourcemaps))

Files can take an array of strings (and those strings can include glob-like syntax if needed), so generally it will look like:

```json
{
  "files": ["dist"]
}
```

Be aware that the files array doesn't accept a relative specifier; writing `"files": ["./dist"]` will not work as expected.

One great way to verify you have set the files field up correctly is by running [`npm publish --dry-run`](https://docs.npmjs.com/cli/v8/commands/npm-publish#dry-run), which will list off the files that will be included based on this setting.

</details>

### Set the default module `type` for your JS files

<details>
<summary><code>type</code> dictates which module system your <code>.js</code> files use</summary>

Runtimes and bundlers need a way to determine what type of module system your `.js` files are using - ESM or CommonJS. Because CommonJS came first, that is the what bundlers will assume by default, but you can control it by adding `"type"` your `package.json`.

Your options are either `"type":"module"` or `"type":"commonjs"`, and though you can leave it blank (to default to CommonJS) it's highly recommended that you set it to one or the other to explicity declare which one you're using.

Note that you can have a mix of module types in the project, through a couple of tricks:

- `.mjs` files will _always_ be ESM modules, even if your `package.json` has `"type": "commonjs"` (or nothing for `type`)
- `.cjs` files will _always_ be CommonJS modules, even if your `package.json` has `"type": "module"`
- You can add additional `package.json` files that are nested inside of folders; runtimes and bundlers look for the _nearest_ `package.json` and will traverse the folder path upwards until they find it. This means you could have two different folders, both using `.js` files, but each with their own `package.json` set to a different `type` to get both a CommonJS- and ESM-based folder.

Refer to the excellent NodeJS documentation [here](https://nodejs.org/docs/latest-v18.x/api/packages.html#determining-module-system) and [here](https://nodejs.org/docs/latest-v18.x/api/packages.html#packagejson-and-file-extensions) for more information.

</details>

### List which modules have `sideEffects`

<details>
<summary>Setting the <code>sideEffects</code> field enables tree shaking </summary>

Much a like creating a [pure function](https://en.wikipedia.org/wiki/Pure_function) can bring benefits, creating a "pure module" enables certain benefits as well; bundlers can do a much better job of tree shaking your library.

The way to communicate to bundlers which of your modules are "pure" or not is by setting the `sideEffects` field in `package.json` - without this field, bundlers have to assume that **all** of your modules are impure.

`sideEffects` can either be set to `false` to indicate that none of your modules have side effects, or an array of strings to list which files have side effects. For example:

```jsonc
{
  // all modules are "pure"
  "sideEffects": false
}
```

or

```jsonc
{
  // all modules are "pure" except "module.js"
  "sideEffects": ["module.js"]
}
```

So, what make a module "impure?" Some examples are modifying a global variable, sending an API request, or importing CSS, without the developer doing anything to invoke that action. For example:

```js
// a module with side effects

export const myVar = "hello";

window.example = "testing";
```

By importing `myVar`, your module sets `window.example` automatically! For example:

```js
import { myVar } from "library";

console.log(window.example);
// logs "testing"
```

In some cases, like polyfills, that behavior is intentional. However, if we wanted to make this module "pure", we could move the assignment to `window.example` into a function. For example:

```js
// a "pure module"

export const myVar = "hello";

export function setExample() {
  window.example = "testing";
}
```

This is now a "pure module." Also note the difference in how things look on the developer's side of things:

```js
import { myVar, setExample } from "library";

console.log(window.example);
// logs "undefined"

setExample();

console.log(window.example);
// logs "testing"
```

Refer to [this article](https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free) for more details.

</details>

### Set the `main` field

<details>
<summary><code>main</code> defines the CommonJS entry </summary>

`main` is a fallback for bundlers or runtimes that don't yet understand [`package.json#exports`](#define-your-exports); if a bundler/environment does understand package exports, then `main` is not used.

`main` should point to a CommonJS-compatible bundle; it should probably match the same file as your package export's `require` field.

</details>

### Set the `module` field

<details>
<summary><code>module</code> defines the ESM entry</summary>

`module` is a fallback for bundlers or runtimes that don't yet understand [`package.json#exports`](#define-your-exports); if a bundler/environment does understand package exports, then `module` is not used.

`module` should point to a ESM-compatible bundle; it should probably match the same file as your package export's `module` and/or `import` field.

</details>

### Set additional fields for CDNs

<details>
<summary>Support CDNs like <code>unpkg</code> and <code>jsdelivr</code></summary>

To enable your library to "work by default" on CDNs like [unpkg](https://unpkg.com) and [jsdelivr](https://www.jsdelivr.com), you can set their specific fields to point to your `umd` bundle. For example:

```json
{
  "unpkg": "./dist/index.umd.js",
  "jsdelivr": "./dist/index.umd.js"
}
```

</details>

### Consider setting the `browser` field

<details>
<summary><code>browser</code> points to a bundle that works in the browser</summary>

`browser` is a fallback for bundlers or runtimes that don't yet understand [`package.json#exports`](#define-your-exports); if a bundler/environment does understand package exports, then `browser` is not used.

`browser` should point to an `esm` bundle that works in the browser. However, you'll only need to set this field if you are creating different bundles for browsers and servers (and/or other non-browser environments). If you're not creating multiple bundles for multiple environments, or if your bundles are "pure JavaScript" / "universal" and can be run in any JavaScript environment, then you don't need to to set the `browser` field.

If you do need to set this field, here's an [excellent guide](https://github.com/defunctzombie/package-browser-field-spec) on the different ways you can configure it.

Note that the `browser` field shouldn't point to a `umd` bundle, as that would make it so that your library isn't treeshaked by bundlers (like Webpack) that prioritize this field over the others such as [module](#set-the-module-field) and [main](#set-the-main-field).

</details>

### Set the `types` field

<details>
<summary><code>types</code> defines the TypeScript types </summary>

`types` is a fallback for bundlers or runtimes that don't yet understand [`package.json#exports`](#define-your-exports); if a bundler/environment does understand package exports, then `types` is not used.

`types` should point to your TypeScript entry file, such as `index.d.ts`; it should probably match the same file as your package export's `types` field.

</details>

### List your `peerDependencies`

<details>
<summary>If you rely on another framework or library, set it as a peer dependency</summary>

You should [externalize any frameworks](#externalize-frameworks) you rely on. However, in doing so, your library will only work if the developer installs the framework you need on their own. One way to help them know that they need to install the framework is by setting `peerDependencies` - for example, if you were building a React library, it would potentially look like this:

```json
{
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

Refer to [this article](https://nodejs.org/en/blog/npm/peer-dependencies/) for more details.

You should also document your reliance on these dependencies; for example, `npm v3-v6` does not install peer dependencies, while `npm v7+` will automatically install peer dependencies.

</details>

### State which `license` your library falls under

<details>
<summary>Protect yourself and other contributors</summary>

> An open source license protects contributors and users. Businesses and savvy developers won’t touch a project without this protection.

That quote comes from [Choose a License](https://choosealicense.com/), which is also a great resource for deciding which license is right for your project.

Once you have decided on a license, the [npm Docs for the license](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#license) describe the format that the license field takes. An example:

```json
{
  "license": "MIT"
}
```

Additionally, you can create a `LICENSE.txt` file in the root of your project and copy the license text there.

</details>

---

## Special Thanks

A big thank you to the people who took the time out of their busy schedules to review and suggest improvements to the first draft of this document (ordered by last name):

- Joel Denning @joeldenning
- Fran Dios @frandiox
- Kent C. Dodds @kentcdodds
- Carlos Filoteo @filoxo
- Jason Miller @developit
- Konnor Rogers @paramagicdev
- Matt Seccafien @cartogram
- Nate Silva @natessilva
- Cong-Cong Pan @SyMind
