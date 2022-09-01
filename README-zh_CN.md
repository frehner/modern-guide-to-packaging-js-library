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
