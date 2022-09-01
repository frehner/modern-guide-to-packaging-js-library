# 如何打包 JavaScript 库的现代化指南

## 简介

这个指南的写作目的在于提出一些大多数库都应该遵循的建议，同时也提供额外的信息，帮助你了解这些建议被提出的原因，或帮助你判断是否不需要遵循某些建议。这个指南仅适用于 **库（libraries）**，不适用于应用。

要强调的是，这只是一些 **建议**，并不是所有库都必须要遵循的。每个库都是独特的，可能有充足的理由选择让库不采用任何给出的建议。

最后，这个指南包含任何打包工具的特性 —— 已经有许多指南来说明如何在配置特定的打包工具。相反我们聚焦于每个库和打包工具都适用的事项。

## 输出 `esm`、`cjs` 和 `umd` 格式

<details>
<summary>支持全部的生态</summary>

`esm` 是“EcmaScript module”的缩写。

`cjs` 是“CommonJS module”的缩写。

`umd` 是“Universal Module Definition”的缩写，它可以在 `<script>` 标签中执行、被 `CommonJS` 模块加载器加载、被 `AMD` 模块加载器加载。

`esm` 被认为是“未来”，但 `cjs` 仍然在社区和生态系统中占有重要地位。`esm` 对于打包工具来说更容易正确地进行 treeshaking，因此对于库来说，具有这种格式很重要。或许在将来的某一天，你的库只需要输出 `esm`。

你可能已经注意到，`umd` 兼容 CommonJS 模块加载器 —— 是否要同时指定 `cjs` 和 `umd` 完全取决于你。在某些情况下，这没有必要。但有些情况下，最好有一个保持源代码的文件和目录结构的纯 `cjs` 输出，和一个输出到单个文件的 `umd`，这样就可以轻松地将其用于 `<script>` 标签。

最后还需要注意是，在开发人员在其应用中同时使用 `cjs` 和 `esm` 格式的情况下，可能会发生双包危险。[dual package hazard](https://nodejs.org/api/packages.html#dual-package-hazard) 一文介绍了一些缓解该问题的方法，[`package.json#exports`](#define-your-exports) 中的条件导出也可以帮助防止这种情况的发生。

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

## 外置框架

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

### 列出 `sideEffects`

<details>
<summary>设置 <code>sideEffects</code> 来允许 treeshaking </summary>

创建一个“纯模块”带来的优点与创建一个[纯函数](https://en.wikipedia.org/wiki/Pure_function)十分类似；打包工具能够对你的库更好的进行 treeshaking。

通过设置 `sideEffects` 让打包工具知道你的模块是否是“纯”的。不设置这个字段，打包工具将不得不假设你**所有**的模块都是有副作用。

`sideEffects` 可以设为 `false`，表示没有任何模块具有副作用，也可以设置为字符串数组来列出哪些文件具有副作用。例如：


```jsonc
{
  // 所有模块都是“纯”的
  "sideEffects": false
}
```

or

```jsonc
{
  // 除了 "module.js"，所有模块都是“纯”的
  "sideEffects": ["module.js"]
}
```

所以，什么让一个模块具有副作用？例如修改一个全局变量，发送 API 请求，或导出 CSS，而且开发人员不需要做任何事情这些动作就会被执行。例如：

```js
// 具有副作用的模块

export const myVar = "hello";

window.example = "testing";
```

导入 `myVar` 时，你的模块自动设置  `window.example`。

例如：

```js
import { myVar } from "library";

console.log(window.example);
// 打印 "testing"
```

在某些情况下，如 polyfill，这种行为是有意的。然而，如果我们想让这个模块是“纯”的，我们可以将对 `window.example` 的赋值移动到一个函数中。例如：

```js
// 一个“纯”模块

export const myVar = "hello";

export function setExample() {
  window.example = "testing";
}
```

现在这是一个“纯”模块。注意，从开发者的角度来看会有不同：

```js
import { myVar, setExample } from "library";

console.log(window.example);
// 打印 "undefined"

setExample();

console.log(window.example);
// 打印 "testing"
```

通过[这篇文章](https://webpack.js.org/guides/tree-shaking/#mark-the-file-as-side-effect-free)来了解更多。

</details>

### 设置 `main`

<details>
<summary><code>main</code> 定义 CommonJS 入口 </summary>

`main` 是一个当打包工具或运行时不支持 [`package.json#exports`](#define-your-exports) 时的兜底方案；如果打包工具或运行时支持条件导出，则不会使用 `main`。

`main` 应该指向一个兼容 CommonJS 格式的产出；它应该与条件导出中的 `require` 保持一致。

</details>

### 设置 `module`

<details>
<summary><code>module</code> 定义 ESM 入口</summary>

`module` 是一个当打包工具或运行时不支持 [`package.json#exports`](#define-your-exports) 时的兜底方案；如果打包工具或运行时支持条件导出，则不会使用 `module`。

`module` 应该指向一个兼容 ESM 格式的产出；它应该与条件导出中的 `module` 或 `import` 保持一致。

</details>

### 设置 `browser`

<details>
<summary><code>browser</code> 定义用于 script 标签的产出 </summary>

`browser` 是一个当打包工具或运行时不支持 [`package.json#exports`](#define-your-exports) 时的兜底方案；如果打包工具或运行时支持条件导出， 则不会使用 `browser`。

`browser` 应该指向 `umd` 格式的产出；它应该与条件导出中的 `script` 文件保持一致。

</details>

### 设置 `types`

<details>
<summary><code>types</code> 定义 TypeScript 类型 </summary>

`types` 是一个当打包工具或运行时不支持 [`package.json#exports`](#define-your-exports) 时的兜底方案； 如果打包工具或运行时支持条件导出，则不会使用 `types`。

`types` 应该指向你的 TypeScript 入口文件，例如 `index.d.ts`；它应该与条件导出中的 `types` 文件保持一致。

</details>

### 列出 `peerDependencies`

<details>
<summary>如果你依赖其他的框架或库，将它设置为 peer dependency</summary>

你应该[外置框架](#externalize-frameworks)。然而，这样做后，开发者为了让你的库正常工作，需要自行安装你需要的框架。通过设置 `peerDependencies` 让他们知道他们所需要安装的框架。
- 例如，如果你创建了一个 React 库：

```json
{
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

通过[这篇文章](https://nodejs.org/en/blog/npm/peer-dependencies/)来了解更多。

你应该以书面形式来体现这些依赖；例如，`npm v3-v6` 不安装 peer dependencies，而 `npm v7+` 将自动安装 peer dependencies。

</details>

### 明确你的库使用的`许可证`

<details>
<summary>保护你自己和其他的贡献者</summary>

> 开源许可证用于保护贡献者和用户。没有这种保护，企业和有经验的开发人员不会使用该项目。

上述引用来自[选择许可证](https://choosealicense.com/)，这也是一篇很好的文章，来帮助你决定哪个许可证适合你的项目。

当你决定了许可证，[关于许可证的 npm 文档](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#license)中描述了许可证字段的格式。例如：

```json
{
  "license": "MIT"
}
```

除此之外，你可以在你的项目根目录下创建一个 `LICENSE.txt` 文件，并将许可证的文本复制到这里。

</details>

---

## 致谢

十分感谢抽出时间对这个文档的初稿进行审查并给出修改意见的人（按姓氏排序)：

- Joel Denning @joeldenning
- Fran Dios @frandiox
- Kent C. Dodds @kentcdodds
- Carlos Filoteo @filoxo
- Jason Miller @developit
- Konnor Rogers @paramagicdev
- Matt Seccafien @cartogram
- Nate Silva @natessilva
