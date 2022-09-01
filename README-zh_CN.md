# 如何打包 JavaScript 库的现代化指南

## 简介

这个指南的写作目的在于提出一些大多数库都应该遵循的建议，同时也提供额外的信息，帮助你了解这些建议被提出的原因，或帮助你判断是否不需要遵循某些建议。这个指南仅适用于 **库（libraries）**，不适用于应用。

要强调的是，这只是一些 **建议**，并不是所有库都必须要遵循的。每个库都是独特的，可能有明确的理由选择让库不实现任何给出的建议。

最后，这个指南包含任何打包工具的特性 —— 已经有许多指南来说明如何在配置特定的打包工具。相反我们聚焦于每个库和打包工具都适用的事项。

## 输出 `esm`、`cjs` 和 `umd` 格式

<details>
<summary>支持全部的生态</summary>

`esm` 是“EcmaScript module”的缩写。

`cjs` 是“CommonJS module”的缩写。

`umd` 是“Universal Module Definition”的缩写，它可以在 `<script>` 标签中执行、被 `CommonJS` 模块加载器加载、被 `AMD` 模块加载器加载。

`esm` 被认为是“未来”，但 `cjs` 仍然在社区和生态系统中占有重要地位。`esm` 对于打包工具来说更容易正确地进行 treeshaking，因此对于库来说，具有这种格式很重要。或许在将来的某一天，你的库只需要输出 `esm`。

你可能已经注意到，`umd` 兼容 CommonJS 模块加载器 —— 是否要同时指定 `cjs` 和 `umd` 完全取决于你。在某些情况下，这没有必要。但有些情况下，最好有一个保持源代码的文件和目录结构的纯 `cjs` 输出，和一个输出到单个文件的 `umd`，这样就可以轻松地将其用于 `<script>` 标签。

Finally, if your library is stateful, be aware that this does open the possibility of your library running into the [dual package hazard](https://nodejs.org/api/packages.html#dual-package-hazard), which can occur in situations where a developer ends up with both a `cjs` and `esm` version of your library in their application. The "dual package hazard" article describes some ways to mitigate this issue, and the `module` condition in [`package.json#exports`](#define-your-exports) can also help prevent this from happening.

</details>

## 输出多文件

<details>
<summary>通过保留文件目录结构更好地支持 treeshaking</summary>

如果你对你的库使用了打包工具或编译工具，对其进行配置保留源文件目录结构。这样可以更容易地标记特定文件具有副作用，这有助于开发者的打包工具进行 threeshaking。更多信息，请参阅这篇文章。

</details>

## 不要压缩代码

<details>
<summary>让开发者自己处理代码压缩</summary>

如果你对你的库使用了打包工具或编译工具，对其进行配置不要进行代码压缩。压缩后的代码难以被开发者的打包工具进行 threeshaking，而且开发者的打包工具将会对你的库进行压缩。更多信息，请参阅这篇文章。

一个例外是，如果你正在创建一个无需打包工具处理，能在直接在浏览器中使用的包（通常，这些包是 `umd` 格式的，但也可以是现代的 `esm` 格式）。在这种情况下，你应该对代码进行压缩，并创建 sourcemap，而且可能期望它是个单文件。

</details>

## 创建 sourcemap

<details>
<summary>当进行了打包或编译，生成 sourcemap</summary>

对源代码进行任何形式的编译，都将导致未来某个抛出的异常位置，无法与源码对应起来。
为了帮助未来的自己，创建 sourcemap，即使编译工作很少。

</details>

## 创建 TypeScript 类型

<details>
<summary>用类型来提升开发体验</summary>

</details>

## 处理框架依赖

<details>
<summary>不要将 React、Vue 等框架打包到你的库中</summary>

当构建的库依赖某个框架（例如 React、Vue 等），或者是作为另一个库的插件，你可能需要将框架配置到“externals”中。这可以使你的库引用这个框架，但不会将其包含在最终的产出中。这会防止产生一些 bug，并减少库的体积。

你应该也需要将框架添加到库的 `package.json` 的 peer dependencies 中，这将帮助开发者发现你依赖某个框架。

</details>

## 目标浏览器

<details>
<summary>使用新特性时，要对旧浏览器进行必要的支持</summary>

这篇 web.dev 上的文章提供了一个很好的案例，并提供了关于指导原则：

* 当使用你的库时，允许开发者支持老版本的浏览器。
* 输出多个产物来支持不同版本的浏览器。

一个例子是，如果你使用 TypeScript，你应该在 `tsconfig.json` 中将 `"target"` 设置为 `ESNext`。

</details>

## 必要的编译

<details>
<summary>编译 TypeScript、将 JSX 转换为函数调用</summary>

如果库的源码是需要进行编译的形式，如 TypeScript、React 或 Vue 组件等，那么你库需要输出的是编译后的代码。

例如：

* 你的 TypeScript 代码应该输出为 JavaScript。
* 你的 React 组件，例如 `<Example />`，应该在输出中使用 `jsx()` 或 `createElement()` 替换 JSX 语法。

</details>

## 维护 changelog

<details>
<summary>追踪更新和变更</summary>

要让开发者能了解到有哪些变更和对他们的影响，至于是通过自动化工具还是通过亲自动手的方式来处理，这无关紧要。理想情况下，库的每次版本变更都应该在 changelog 进行同步更新。

</details>

## `package.json` 设置

`package.json`中有很多重要的设置和字段值的讨论；这里将着重讨论一些比较重要的，但是还有很多你可以通过[additional fields](https://docs.npmjs.com/cli/v8/configuring-npm/package-json)去了解。

### 设置`name`字段

<details>
<summary>给你的library取一个名字</summary>

`name`字段是用来表明你的包在`npm`上的名字，开发者可以通过这个名字去安装并使用你的代码。

怎样给代码库命名是有一些规范限制的，如果你的代码库属于某个组织，你还可以创建一个scope。更多细节可以参考[name docs on npm](https://docs.npmjs.com/cli/v8/configuring-npm/)。

`name`和[version](#设置`version`字段)字段在代码库每次迭代的过程中共同形成一个独一无二的标志。

</details>

### 设置`version`字段

<details>
<summary>通过更改version去更新你的library</summary>

正如[name](#设置`name`字段)部分所说，`name`和`version`共同为您的library在npm上创建一个唯一标识。当您对库中的代码进行更新时，您可以更新`version`字段并发布以允许开发人员获取该新代码。


建议使用[semver](https://semver.org/)版本策略，但要注意的是有些library选择[calver](https://calver.org/)策略或者使用他们自己制定的版本策略。无论您选择使用哪种策略，都应该记录下来，以便开发人员了解您的库的版本控制是如何工作的。

您还应该在[changelog](#维护-changelog)中记录您的更改。

</details>

### 定义你的`exports`

<details>
<summary>`exports`为您的library定义公共API</summary>

`package.json`中的`exports`字段 - 有时被称为"export maps" - 是一个非常有用的补充，尽管它确实增加了一些复杂性。它做的最重要的两件事是：

1. 定义哪些东西可以从你的library中导入，哪些则不可以，以及可导入的内容的名字。如果没有在`exports`中被列出，那么开发者就不可以`import`/`require`它们。换句话说，`exports`的表现像是给你的library用户查看的公共API，帮助定义哪些是外部的哪些是内部的。

2. 允许您根据不同的条件（您可以定义）去选择那个文件是被导入的，例如“文件是被`import`还是被`require`”？开发人员需要的是`development`版本的library还是`production`版本等等。

关于这部分的内容 [NodeJS团队](https://nodejs.org/api/packages.html#package-entry-points) 和 [Webpack团队](https://webpack.js.org/guides/package-exports/) 提供了一些很优秀的文档。在此我们就列出一个涵盖大部分常用场景的例子：

```json
{
  "exports": {
    ".": {
      "types": "index.d.ts",
      "script": "index.umd.js",
      "module": "index.js",
      "import": "index.js",
      "require": "index.cjs",
      "default": "index.js"
    },
    "./package.json": "./package.json"
  }
}
```
</details>


让我们深入了解这些字段的含义以及我选择这个示例的原因：

- `"."` 表示你的library的默认入口。
- 解析过程是 **从上往下** 的并且一旦匹配就不会继续往下解析；所以入口的顺序是非常重要的
- `types` 字段应该放在第一位，它用来帮助TypeScript寻找类型文件
- `script` 字段应该填写可以直接在 `<script>` 标签中使用的 `umd` bundle 文件
- `module` 字段是一个被打包器（bundlers）例如 Webpack 和 Rollup 支持的 “非官方”字段。它应该被放在`import`和`require`之前，并且被指向只有 `esm` 规范的 bundle -- 如果你的源代码是纯`esm`的，它也可以是你的源代码。 您可以从[这里](https://github.com/webpack/webpack/issues/11014#issuecomment-641550630), [这里](https://github.com/webpack/webpack/issues/11014#issuecomment-643256943), 还有 [此处](https://github.com/rollup/plugins/pull/540#issuecomment-692078443)了解更多关于`module`的内容
- `import` 当有人通过 `import` 使用你的library的时候，会使用该字段指向的文件
- `require`当有人通过 `require` 使用你的library的时候，会使用该字段指向的文件
- `default` 应该始终排在最后，如果没有其他匹配项，则作为兜底方案


当一个bundler或者environment支持`exports`字段的时候，那么`package.json`中的顶级字段[main](#set-the-main-field)，[types](#set-the-types-field)，[module](#set-the-module-field)还有 [browser](#set-the-browser-field)将被忽略，被`exports`取代。但是，对于尚不支持`exports`字段的工具或运行时来说，设置这些字段仍然很重要。

如果你有一个 "development" 和 "production" 的 bundle（例如，您有一些warning在development bundle中但在production bundle中没有），那么你可以在 `exports` 字段中通过设置`"development"` 和 `"production"`实现。 `webpack` 将会自动识别并且 Rollup [can be configured](https://github.com/rollup/plugins/tree/master/packages/node-resolve/#exportconditions) 也可以通过配置去识别。

</details>

### 列出要发布的`files`

<details>
<summary><code>files</code> 定义 NPM 包中包含哪些文件</summary>

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

With the split the CommonJS and ESM module systems, runtimes and bundlers need a way to determine what type of module system your `.js` files are using. Because CommonJS came first, that is the default - but you can change it by adding `"type": "module"` to your `package.json`, which then means that your `.js` files will be viewed as ESM modules.

Your options are either `"module"` or `"commonjs"`, and it's highly recommended that you set it to one or the other to explicity declare which one you're using.

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

So, what make a module "inpure?" Some examples are modifying a global variable, sending an API request, or importing CSS, without the developer doing anything to invoke that action. For example:

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

### Set the `browser` field

<details>
<summary><code>browser</code> defines the script-taggable bundle </summary>

`browser` is a fallback for bundlers or runtimes that don't yet understand [`package.json#exports`](#define-your-exports); if a bundler/environment does understand package exports, then `browser` is not used.

`browser` should point to the `umd` bundle; it should probably match the same file as your package export's `script` field.

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
