# The Modern Guide to Packaging your JavaScript library

## Some Context

This article is written to provide at-a-glance suggestions that most libraries should follow, while also giving additional nuance and context if you want to understand why a suggestion is made or if you feel like you need to deviate from these suggestions.

To emphasize, this is a list of **suggestions** and is not meant to be definitive or to provide flame bait for libraries that do not follow these suggestions. Each library is unique and there are probably good reasons a library may chose to not implement any one of these suggestions.

This guide is not meant to be specific to any particular bundler - there already exist many guides on how to set configs in specific bundlers. Instead, we want to focus on things that apply to every library and bundler (or lack of bundler).

---

## Output to `esm`, `cjs`, and `umd` formats

<details>
<summary>Supporting the whole ecosystem</summary>

`esm` is short for "EcmaScript module."

`cjs` is short for "CommonJS module."

`umd` is short for "Universal Module Definition," and can be run by a raw `<script>` tag, or in CommonJS module loaders, or by `AMD` module loaders.

Without getting into the flame wars that generally happen around `esm` and `cjs` formats, you should know that generally `esm` is considered "the future" but `cjs` still has a strong hold on the community and ecosystem. `esm` is generally easier for bundlers to correctly treeshake, so it's especially important for libraries to have this format. It's also possible that some day in the future your library only needs to output to `esm`.

If you output to `umd` format, you'll note that `umd` is already compatible with CommonJS. It's up to you if you want to have both a specific `cjs` _and_ `umd` output; in some cases, there's no need to. In other cases, it can be nice to have a pure `cjs` output that keeps the file and folder structure of your source code, and a `umd` output to a single file so it can be easily `<script>`-tagged.

Finally, if your library is stateful, be aware that this does open the possibility of your library running into the [dual package hazard](https://nodejs.org/api/packages.html#dual-package-hazard), which can occur in situations where your library maintains some internal state and a developer happens to be using both a `cjs` and `esm` version of your library. The linked article describes some ways to mitigate this issue; another way to help prevent it is to use the `module` condition in [`package.json#exports`](#set-the-exports-field).

</details>

## Output to multiple files

<details>
<summary>Better treeshaking by maintaining the file structure</summary>

If you use a bundler or transpilier in your library, set it up so that it outputs files in the same way that they were authored. This makes it easier to mark specific files as having [side effects](#set-the-sideeffects-field), so that the rest of your modules aren't included in the side effects list. See [this article](https://levelup.gitconnected.com/code-splitting-for-libraries-bundling-for-npm-with-rollup-1-0-2522c7437697) for more details.

The exception to this is if you are making a bundle meant to be consumed directly in the browser without _any_ bundler (commonly, these are `umd` bundles but could also be modern `esm` bundles as well). In this case, it is better to have the browser request a single large file than need to request multiple smaller ones.

</details>

## Don't minify

<details>
<summary>Let developers minify your library on their own</summary>

If you use a bundler or transpilier in your library, configure it so that your output is not minified. Minification of your library makes it harder on the developer's bundler to do things like tree shaking. See [this article](https://levelup.gitconnected.com/code-splitting-for-libraries-bundling-for-npm-with-rollup-1-0-2522c7437697) for more details.

The exception to this is if you are making a bundle meant to be consumed directly in the browser without _any_ bundler (commonly, these are `umd` bundles but could also be modern `esm` bundles as well). You may want to consider making a "development" and "production" version of these bundles, or at the very least ensuring that you created [sourcemaps](#create-sourcemaps) for them.

</details>

## Create sourcemaps

<details>
<summary>When using a bundler or transpiler, generate sourcemaps</summary>

Any sort of transformation of your source code to a bundle will produce errors that point at the wrong location in your code. Help your future self out and create sourcemaps, even if your transformations are small!

</details>

## Create TypeScript types

<details>
<summary>Types improve the developer experience</summary>

The number of developers using TypeScript continues to grow each year, and having types built-in to your library help improve the developer experience (DX) for those devs. Additionally, devs who are not using TypeScript still get a better DX when they use an editor that understands types (such as VSCode, which uses the types to power its Intellisense feature).

However, creating types does NOT mean you must author your library in TypeScript. One option is to continue using normal JavaScript as your source code, but to also supplement it with [JSDoc](https://jsdoc.app/) comments. You can then configure TypeScript to only [emit declaration files](https://www.typescriptlang.org/tsconfig/#emitDeclarationOnly) from your JavaScript source code. TypeScript documentation even has a guide on [which JSDoc features are supported](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html).

Another option is to write the TypeScript type files directly, not even as part of your code. You can create an `index.d.ts` file and write the types there.

Once you have the types file, make sure you set your [`package.json#exports`](#set-the-exports-field) and [`package.json#types`](#set-the-types-field) fields.

</details>

## Externalize frameworks

<details>
<summary>Don't include a copy of React, Vue, etc. in your bundle</summary>

When building a library that relies on a framework (such as React, Vue, etc.), you'll want to add the framework to your bundler's "externals" (or equivalent) configuration, which makes it so that your library will reference the framework but will not include it in the bundle. This will prevent bugs and also reduce the size of your library's package.

You should also add the framework to your library's `package.json`'s [peer dependencies](#set-your-peerdependencies) to indicate to developers that you rely on that framework.

</details>

## Target modern browsers

<details>
<summary>Use modern features and let devs choose to support older browsers if they need to</summary>

[This article on web.dev](https://web.dev/publish-modern-javascript/) makes a great case for your library to target modern features, and offers guidelines on how to:

- Enable developers to support older browsers when using your library
- Output multiple bundles that support various levels of browser support

As one example, if you're transpiling from TypeScript, you should set `"target"` in your `tsconfig.json` to `ESNext`.

</details>

## Transpile if necessary

TODO

## `package.json` settings

There's a lot of important things to talk about in `package.json`, so let's break it down further:

### Set the `exports` field

<details>
<summary><code>exports</code> define the <i>how</i> and <i>what</i> of what can be imported</summary>

The `exports` field on `package.json`, sometimes called "export maps", is an incredibly useful addition, though it does add some complexity. The two most important things that it does is:

1. Defines what can and cannot be imported from your library, and what the name of it is. If it's not listed in `exports`, then developers cannot `import`/`require` it. In other words, it acts like a public API for users of your library and helps define what is public and what is internal.
2. Allows you to change which file is imported based on conditions you define, such as "Was the file `import`ed or `require`d? Do they want a `development` or `production` version of my library?" etc.

There are some good docs from the [NodeJS team](https://nodejs.org/api/packages.html#package-entry-points) and the [Webpack team](https://webpack.js.org/guides/package-exports/) on the possibilities here. Here is an example that covers some common use-cases:

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

Important things to know about the `exports` field:

- `"."` indicates the default entry for your package.
- The resolution happens from **top to bottom** and stops as soon as a matching field is found; the order of entries is very important.
- `types` should always come first, and helps TypeScript find the types file
- `default` should always be last, and is meant as a fallback
- The `module` field is an "unofficial" field that is supported by bundlers like Webpack and Rollup. It should come before `import` and `require`, and point to an `esm`-only bundle - which can be the same as your original `esm` bundle if it's purely `esm`. For a deeper dive, read more [here](https://github.com/webpack/webpack/issues/11014#issuecomment-641550630), [here](https://github.com/webpack/webpack/issues/11014#issuecomment-643256943), and [here](https://github.com/rollup/plugins/pull/540#issuecomment-692078443).

If a bundler or environment understands the `exports` field, then the `package.json`'s top-level [main](#set-the-main-field), [types](#set-the-types-field), [module](#set-the-module-field), and [browser](#set-the-browser-field) fields are ignored, as `exports` supersedes those fields. However, it's still importantant to set those fields, for tools or environments that do not yet understand the `exports` field.

If you have a "development" and a "production" bundle (e.g. you have warnings in the development bundle that don't exist in the production bundle), then you can also set that up here in the `exports` field. `webpack` will recognize these conditions automatically, and Rollup [can be configured](https://github.com/rollup/plugins/tree/master/packages/node-resolve/#exportconditions) to recognize them as well.

</details>

### Set the `files` field

<details>
<summary><code>files</code> defines which files are included in your NPM package</summary>

The [`files`](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#files) field tells `npm` which files and folders to include when you bundle your library up and put it on NPM's package registry.

For example, if you transform your code from TypeScript into Javscript, you probably don't want to include the TypeScript source code in your NPM package, as it's not useful and just increases the disk size of your package.

Files can take an array of strings (and those strings can include glob-like syntax if needed), so generally you would want something like:

```json
{
  "files": ["dist"]
}
```

Be aware that the files array doesn't accept a relative specifier; writing `"files": ["./dist"]` will not work as expected.

One great way to verify you have set the files field up correctly is by running [`npm publish --dry-run`](https://docs.npmjs.com/cli/v8/commands/npm-publish#dry-run), which should list off the files that would be included based on your settings.

</details>

### Set the `sideEffects` field

<details>
<summary><code>sideEffects</code> enables tree shaking </summary>

Much a like creating a [pure function](https://en.wikipedia.org/wiki/Pure_function) can bring benefits, creating a "pure module" enables certain benefits as well; bundlers can do a much better job of tree shaking your library.

The way to communicate to bundlers which of your modules are "pure" or not is by setting the `sideEffects` field in `package.json` - without this field, bundlers have to assume that <strong>all</strong> of your modules are impure. `sideEffects` can either be set to `false` to indicate that none of your modules have side effects, or an array of strings to list which files have side effects. For example:

```json
{
  // all modules are "pure"
  "sideEffects": false
}
```

or

```json
{
  // all modules are "pure" except "module.js"
  "sideEffects": ["module.js"]
}
```

What make a module "inpure?" Some examples are modifying a global variable, sending an API request, or importing CSS, without the developer doing anything to invoke that action. For example:

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

In some cases, like polyfills, that's hard to avoid (or even intentional). However, if we wanted to make this module "pure", we could move the assignment to `window.example` into a function. For example:

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

See [this article](https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free) for more details.

</details>

### Set the `main` field

<details>
<summary><code>main</code> defines the CommonJS entry </summary>

`main` is a fallback for bundlers or environments that don't yet understand [`package.json#exports`](#set-the-exports-field); if a bundler/environment does understand package exports, then `main` is not used.

`main` should point to a CommonJS-compatible bundle; it should probably match the same file as your package export's `require` field.

</details>

### Set the `module` field

<details>
<summary><code>module</code> defines the ESM entry </summary>

`module` is a fallback for bundlers or environments that don't yet understand [`package.json#exports`](#set-the-exports-field); if a bundler/environment does understand package exports, then `module` is not used.

`module` should point to a ESM-compatible bundle; it should probably match the same file as your package export's `import` field.

</details>

### Set the `browser` field

<details>
<summary><code>browser</code> defines the script-taggable bundle </summary>

`browser` is a fallback for bundlers or environments that don't yet understand [`package.json#exports`](#set-the-exports-field); if a bundler/environment does understand package exports, then `browser` is not used.

`browser` should point to the `umd` bundle; it should probably match the same file as your package export's `script` field.

</details>

### Set the `types` field

<details>
<summary><code>types</code> defines the TypeScript types </summary>

`types` is a fallback for bundlers or environments that don't yet understand [`package.json#exports`](#set-the-exports-field); if a bundler/environment does understand package exports, then `types` is not used.

`types` should point to your TypeScript entry file, such as `index.d.ts`; it should probably match the same file as your package export's `types` field.

</details>

### Set your `peerDependencies`

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

You should also include your reliance on those dependencies and have them in your `npm install <packages>` documentation for your library; `npm v3-v6` does not auto install peerDependencies, while `npm v7+` does auto install peer dependencies. It's also good to be explicit about your package's needs in your documentation.

</details>

---

## Contributing

TODO: add notes about how to contribute here, such as "if you disagree with a suggestion, please link to a source for your disagreement."

## Special Thanks

-
