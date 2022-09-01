# 打包 JavaScript 库的现代化指南

[English](./README.md) | 简体中文

## 简介

本指南旨在提供一些大多数库都应该遵循的一目了然的建议。以及一些额外的信息，用来帮助你了解这些建议被提出的原因，或帮助你判断是否不需要遵循某些建议。这个指南仅适用于 **库（libraries）**，不适用于应用（app）。

要强调的是，这只是一些 **建议**，并不是所有库都必须要遵循的。每个库都是独特的，它们可能有充足的理由不采用本文中的任何建议。

最后，这个指南不针对某一个特定的打包工具 —— 已经有许多指南来说明如何在配置特定的打包工具。相反我们聚焦于每个库和打包工具（或不用打包工具）都适用的事项。

---

## 输出 `esm`、`cjs` 和 `umd` 格式

<details>
<summary>支持全部的生态</summary>

`esm` 是“EcmaScript module”的缩写。

`cjs` 是“CommonJS module”的缩写。

`umd` 是“Universal Module Definition”的缩写，它可以在 `<script>` 标签中执行、被 `CommonJS` 模块加载器加载、被 `AMD` 模块加载器加载。

`esm` 被认为是“未来”，但 `cjs` 仍然在社区和生态系统中占有重要地位。`esm` 对打包工具来说更容易正确地进行 treeshaking，因此对于库来说，拥有这种格式很重要。或许在将来的某一天，你的库只需要输出 `esm`。

你可能已经注意到，`umd` 已经兼容 CommonJS 模块加载器 —— 是否要同时指定 `cjs` 和 `umd` 完全取决于你。在某些情况下，这并无必要。但另外一些情况下，最好有一个纯 `cjs` 输出，它保留源代码的文件和目录结构，和一个输出到单个文件的 `umd`，这样就可以轻松地将其用于 `<script>` 标签。

最后还需要注意是，开发人员可能会在其应用中同时使用 `cjs` 和 `esm` 格式的情况下，发生双包危险。[dual package hazard](https://nodejs.org/api/packages.html#dual-package-hazard) 一文介绍了一些缓解该问题的方法，利用 [`package.json#exports`](#定义你的-exports) 的条件导出也可以帮助防止这种情况的发生。

</details>

## 输出多文件

<details>
<summary>通过保留文件结构更好地支持 treeshaking</summary>

如果你对你的库使用了打包工具或编译器，可以对其进行配置以保留源文件目录结构。这样可以更容易地对特定文件进行 [side effects](#列出-sideeffects) 标记，有助于开发者的打包工具进行 threeshaking。更多信息，参考[这篇文章](https://levelup.gitconnected.com/code-splitting-for-libraries-bundling-for-npm-with-rollup-1-0-2522c7437697)了解更多信息。

一个例外是，如果你要创建一个无需任何打包工具可以直接在浏览器中使用的产出（通常，这些是 `umd` 格式，但也可能是现代的 `esm` 格式）。在这种情况下，最好让浏览器请求一个大文件，而不是请求多个小文件。此外，你应该 [minify](#不要压缩代码) 产出并为其创建 [sourcemap](#创建-sourcemap)。

</details>

## 不要压缩代码

<details>
<summary>让开发者自己处理代码压缩</summary>

如果你对你的库使用了打包工具或编译器，对其进行配置不要进行代码压缩。压缩后的代码难以被开发者的打包工具进行 threeshaking，而且开发者的打包工具将会对你的库进行压缩。参考[这篇文章](https://levelup.gitconnected.com/code-splitting-for-libraries-bundling-for-npm-with-rollup-1-0-2522c7437697)了解更多信息

一个例外是，如果你要创建一个无需任何打包工具可以直接在浏览器中使用的产出（通常，是 `umd` 格式的，但也可以是现代的 `esm` 格式）。在这种情况下，你应该对代码进行压缩，并创建 [sourcemap](#创建-sourcemap)，而且可能希望它是个[单文件](#输出多文件)。

</details>

## 创建 sourcemap

<details>
<summary>当使用打包工具或编译器时，生成 sourcemap</summary>

对源代码进行任何形式的编译，都将导致未来某个抛出的异常位置，无法与源码对应起来。为了帮助未来的自己，创建 sourcemap，即使只进行了很少的编译工作。

</details>

## 创建 TypeScript 类型

<details>
<summary>类型提升开发体验</summary>

随着使用 TypeScript 的开发者数量不断增长，将类型内置到你的库中将有助于改善开发体验 (DX)。此外，不使用 TypeScript 的开发者在使用支持类型的编辑器（例如 VSCode，它使用类型来支持其 [Intellisense 功能](https://code.visualstudio.com/docs/) 时也会获得更好的 DX。

但是，创建类型并不意味着你必须使用 TypeScript 来编写你的库。

一种选择是继续在源代码中使用 JavaScript，然后通过 [JSDoc](https://jsdoc.app/) 注释来支持类型。然后，你可以将 TypeScript 配置为仅从你的 JavaScript 源代码中[emit the declaration files](https://www.typescriptlang.org/tsconfig/#emitDeclarationOnly)。

另一种选择是直接在 `index.d.ts` 文件中编写 TypeScript 类型文件。

获得类型文件后，请确保设置了 [`package.json#exports`](#定义你的-exports) 和 [`package.json#types`](#设置-types) 字段.

</details>

## 外置框架

<details>
<summary>不要将 React、Vue 等框架打包到你的库中</summary>

当构建的库依赖某个框架（例如 React、Vue 等），或者是作为另一个库的插件，你可能需要将框架配置到“externals”中。这可以使你的库引用这个框架，但不会将其包含在最终的产出中。这会防止产生一些 bug，并减少库的体积。

你应该还需要将框架添加到库的 `package.json` 的 [peer dependencies](#列出-peerdependencies) 中，这将帮助开发者发现你依赖于某个框架。

</details>

## 面向现代浏览器

<details>
<summary>如果需要，使用新特性，让开发者支持旧浏览器</summary>

[这篇 web.dev 上的文章](https://web.dev/publish-modern-javascript/)提供了一个很好的案例，并提供了关于指导原则：

- 当使用你的库时，允许开发者支持老版本的浏览器。
- 输出多个产出来支持不同版本的浏览器。

举个例子，如果你使用 TypeScript，你应该在 `tsconfig.json` 中将 `"target"` 设置为 `ESNext`。

</details>

## 必要的编译

<details>
<summary>编译 TypeScript、将 JSX 转换为函数调用</summary>

如果库的源码是需要进行编译的形式，如 TypeScript、React 或 Vue 组件等，那么你库需要输出的是编译后的代码。

例如：

- 你的 TypeScript 代码应该输出为 JavaScript。
- 你的 React 组件，例如 `<Example />`，应该在输出中使用 `jsx()` 或 `createElement()` 来替换 JSX 语法。

进行这样的编译时，请确保同时也[创建 sourcemap](#创建-sourcemap)

</details>

## 维护 changelog

<details>
<summary>追踪更新和变更</summary>

只要能让开发者了解到有哪些变更和对他们的影响，至于是通过自动化工具还是通过亲自动手的方式来处理，这都无关紧要。理想情况下，库的每次[版本](#设置-version)变更都应该在 changelog 中进行相应的更新。

</details>

## 配置 `package.json`

`package.json` 中有许多重要的配置字段值得讨论；我在这里将着重讨论其中最为重要的一些，这还有很多 [additional fields](https://docs.npmjs.com/cli/v8/configuring-npm/package-json)，你同样可以进行配置。

### 设置 `name`

<details>
<summary>给你的库取一个名字</summary>

`name` 字段将决定你的包在 `npm` 上的名字，开发者可以通过这个名字去安装并使用你的库。

注意，库的命名是有限制的，如果你的代码库属于某个组织，你还可以创建一个命名空间。更多细节可以参考 [name docs on npm](https://docs.npmjs.com/cli/v8/configuring-npm/)。

`name` 和 [version](#设置-version) 的组合为库每次迭代创建一个唯一标识。

</details>

### 设置 `version`

<details>
<summary>通过更改 version 来对你的库发布更新</summary>

正如 [name](#设置-name) 部分所说，`name` 和 `version` 的组合为你的库在 npm 上创建一个唯一标识。当你更新库中的代码时，你可以更新 `version` 字段并发布以允许开发者获取该新代码。

推荐使用 [semver](https://semver.org/) 版本控制策略，但要注意的是有些库选择 [calver](https://calver.org/) 或使用他们自己特有的版本控制策略。无论你选择使用哪种策略，都应该记录下来，以便开发者了解你的库是如何进行版本控制的。

你还应该在 [changelog](#维护-changelog) 中记录你的更改。

</details>

### 定义你的 `exports`

<details>
<summary><code>exports</code> 为你的库定义公共 API</summary>

`package.json` 中的 `exports` 字段 - 有时被称为“导出映射” - 是一个非常有用的补充，尽管它确实引入了一些复杂性。它做的最重要的两件事是：

1. 定义哪些东西可以从你的库中导入，哪些则不可以，以及可导入的内容的名字。如果没有在 `exports` 中被列出，那么开发者就不可以 `import` 或 `require` 它们。换句话说，`exports` 的表现像是给你的库用户查看的公共 API，帮助定义哪些是外部的哪些是内部的。

2. 允许你根据不同的条件（你可以定义）去选择那个文件是被导入的，例如“文件是被 `import` 还是被 `require`？开发人员需要的是 `development` 版本的库还是 `production` 版本等等。

关于这部分的内容[NodeJS 团队](https://nodejs.org/api/packages.html#package-entry-points)和[Webpack 团队](https://webpack.js.org/guides/package-exports/)提供了一些很优秀的文档。在此我们就列出一个涵盖大部分常用场景的例子：

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

- `"."` 表示你的库的默认入口
- 解析过程是 **从上往下** 的，并在找到匹配的字段后立即停止；所以入口的顺序是非常重要的
- `types` 字段应该始终放在第一位，它用来帮助 TypeScript 找到类型文件
- `script` 字段应该填写可以直接在 `<script>` 标签中使用的 `umd` 产出
- `module` 是“非官方”字段，被例如 Webpack 和 Rollup 等打包工具所支持。它应该被放在 `import` 和 `require` 之前，并且指向 `esm` 格式的产出 -- 如果你的源代码是纯 `esm` 的，它也可以是你的源代码。 你可以从[这里](https://github.com/webpack/webpack/issues/11014#issuecomment-641550630)、[这里](https://github.com/webpack/webpack/issues/11014#issuecomment-643256943)、还有 [这里](https://github.com/rollup/plugins/pull/540#issuecomment-692078443)了解更多关于 `module` 的内容
- `import` 用于当有人通过 `import` 使用你的库时
- `require`用于当有人通过 `require` 使用你的库时
- `default` 应该始终放在最后，如果没有其他匹配项，作为兜底方案

当一个打包工具或者运行时支持 `exports` 字段的时候，那么 `package.json` 中的顶级字段 [main](#设置-main)、[types](#设置-types)、[module](#设置-module) 还有 [browser](#设置-browser) 将被忽略，被 `exports` 取代。但是，对于尚不支持 `exports` 字段的工具或运行时来说，设置这些字段仍然很重要。

如果你有一个 "development" 和一个 "production" 的产出（例如，你有一些 warning 在 development 产出中但在 production 产出中没有），那么你可以在 `exports` 中通过 `"development"` 和 `"production"` 来设置它们。 `webpack` 将会自动识别这些导出，Rollup 也可以通过[配置](https://github.com/rollup/plugins/tree/master/packages/node-resolve/#exportconditions)来识别它们。

</details>

### 列出要发布的 `files`

<details>
<summary><code>files</code> 定义你的 NPM 包中要包含哪些文件</summary>

[`files`](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#files) 决定 `npm` CLI 在打包库时哪些文件和目录包含到最终的 NPM 包中。

例如，如果你将代码从 TypeScript 编译为 JavaScript，你可能就不想在 NPM 包中包含 TypeScript 的源代码。（相反，你应该包含 [sourcemap](#创建-sourcemap)）。

`files` 可以接受一个字符串数组（如果需要，这些字符串可以包含类似 glob 的语法），例如：

```json
{
  "files": ["dist"]
}
```

注意，文件数组不接受相对路径表示；`"files": ["./dist"]` 将无法正常工作。

验证你已正确设置 `files` 的一种好方法是运行 [`npm publish --dry-run`](https://docs.npmjs.com/cli/v8/commands/npm-publish#dry-run），它将根据此设置列出将会包含的文件。

</details>

### 为你的 JS 文件设置默认的模块 `type`

<details>
<summary><code>type</code> 规定你的 <code>.js</code> 文件使用哪个模块系统</summary>

随着 CommonJS 和 ESM 模块系统的拆分，运行时和打包工具需要一种方式来决定对你的 `.js` 文件采用哪种模块系统。因为 CommonJS 首先出现，所以它是默认的 - 但你可以改变它，通过在你的 `package.json` 中添加 `"type": "module"`，这样你的 `.js` 文件将被当作 ESM 模块。

你的选项可以是 `module` 或 `commonjs`，强烈建议你设置其中的一个，显式地声明你正在使用哪一个。

请注意，你可以通过几个技巧在项目中混用模块类型：

- `.mjs` 文件总是 ESM 模块，即使你的 `package.json` 有 `"type": "commonjs"`（或者没有 `type`）
- `.cjs` 文件总是 CommonJS 模块，即使你的 `package.json` 有 `"type": "module"`
- 你可以在子目录下添加其他 `package.json` 文件；运行时和打包工具将向上遍历文件目录，直到寻找到最近的 `package.json`。这意味着你可以有两个不同的文件夹，都使用 `.js` 文件，但每个文件夹都有自己的 `package.json` 并设置为不同的 `type` 以获得基于 CommonJS 和 ESM 的文件夹。

参考优秀的 NodeJS 文档 [这里](https://nodejs.org/docs/latest-v18.x/api/packages.html#determining-module-system) 和 [这里](https://nodejs.org /docs/latest-v18.x/api/packages.html#packagejson-and-file-extensions）了解更多信息。

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

导入 `myVar` 时，你的模块自动设置 `window.example`。

例如：

```js
import { myVar } from "库";

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
import { myVar, setExample } from "库";

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

`main` 是一个当打包工具或运行时不支持 [`package.json#exports`](#定义你的-exports) 时的兜底方案；如果打包工具或运行时支持条件导出，则不会使用 `main`。

`main` 应该指向一个兼容 CommonJS 格式的产出；它应该与条件导出中的 `require` 保持一致。

</details>

### 设置 `module`

<details>
<summary><code>module</code> 定义 ESM 入口</summary>

`module` 是一个当打包工具或运行时不支持 [`package.json#exports`](#定义你的-exports) 时的兜底方案；如果打包工具或运行时支持条件导出，则不会使用 `module`。

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

`types` 是一个当打包工具或运行时不支持 [`package.json#exports`](#定义你的-exports) 时的兜底方案； 如果打包工具或运行时支持条件导出，则不会使用 `types`。

`types` 应该指向你的 TypeScript 入口文件，例如 `index.d.ts`；它应该与条件导出中的 `types` 文件保持一致。

</details>

### 列出 `peerDependencies`

<details>
<summary>如果你依赖别的框架或库，将它设置为 peer dependency</summary>

你应该[外置框架](#外置框架)。然而，这样做后，你的库只有在开发人员自行安装你需要的框架后才能工作。设置 `peerDependencies` 让他们知道他们需要安装的框架。- 例如，如果你在创建一个 React 库：

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

### 说明你的库使用哪个`许可证`

<details>
<summary>保护你自己和其他的贡献者</summary>

> 开源许可证用于保护贡献者和用户。没有这种保护，企业和有经验的开发者不会使用该项目。

上述引用自 [Choose a License](https://choosealicense.com/)，这也是一篇很好的文章，帮助你来决定哪个许可证适合你的项目。

当你决定了许可证，[关于许可证的 npm 文档](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#license)中描述了许可证字段的格式。例如：

```json
{
  "license": "MIT"
}
```

除此之外，你可以在项目的根目录下创建一个 `LICENSE.txt` 文件，并将许可证的文本复制到这里。

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
