# Vue 的原理和机制

## 1. Vue 简介

Vue 是一款渐进式 JavaScript 框架，专为构建用户界面而设计。其核心理念在于提供一种非侵入式的响应式系统，使状态管理变得简单直观。后面主要针对Vue3展开介绍。

## 2. 响应式系统

Vue3 的响应式系统核心基于 JavaScript 的 Proxy 对象，该对象在 ES6 中引入。Proxy 对象能够包装另一个对象（称为目标对象），并允许拦截和自定义对该目标对象的基本操作，例如属性查找（`get`）、赋值（`set`）、枚举以及函数调用。Vue 利用 Proxy 的 `get` 和 `set` 处理器来拦截属性的访问和修改。当属性被读取时，Vue会将其追踪为依赖项；当属性被写入时，Vue 会触发更新。通常，`Reflect` API 会与 Proxy 处理器结合使用，以将操作转发到原始目标对象，从而在允许拦截的同时确保默认行为。

参考:  [JS的proxy和reflect](JS的proxy和reflect.md)

Vue提供了两种主要的API来声明响应式状态：`ref` 和 `reactive`

- **`ref`**：用于创建原始值（字符串、数字、布尔值）的响应式引用，也可以包装对象。一个 `ref` 对象具有一个 `.value` 属性，该属性持有实际值，并且对该 `.value` 属性的访问或修改会触发响应式更新。
- **`reactive`**：用于创建响应式对象、数组和集合类型（如 Map 和 Set）。它直接包装对象，使其所有属性都实现深度响应式。然而，`reactive` 存在一些局限性，例如无法直接持有原始类型（如字符串、数字或布尔值）。

参考：

- <https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#ref>
- <https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#reactive>

> 所有可以使用 reactive() 的对象都可以使用 ref()，但反之则不然。目前多数文章和开发者认为 ref 更符合“一把梭”的实践，减少心智负担。

---

## 3. 虚拟DOM

虚拟 DOM（VDOM）是一种编程概念。它是真实 DOM 的抽象，是内存中的一个轻量级副本，允许高效更新。虚拟节点（vnode）是一个描述真实 DOM 元素的普通 JavaScript 对象，包括其类型、属性和子节点。这些 vnode 构成了虚拟 DOM 树。直接操作真实 DOM 通常速度较慢且资源密集，容易导致性能瓶颈。VDOM 提供了一个内存层，其操作速度远快于实际 DOM。Vue 等框架利用这种抽象来高效计算和优化更改。

## 4. 模版渲染

渲染过程从模板编译开始，最终呈现到真实 DOM。Vue 模板（例如 SFC 中的 `<template>` 块）不会被直接渲染。相反，它们被编译成 JavaScript **渲染函数**。这种编译可以在构建时提前完成（例如，在 Vite 中使用 SFCs）或在运行时即时完成（例如，CDN 使用）。渲染函数返回虚拟 DOM 树。开发者还可以直接使用 JSX 编写渲染函数。

当 Vue 组件首次挂载时，运行时渲染器会调用其渲染函数。然后，它遍历生成的虚拟 DOM 树，并根据该树创建实际的 DOM 节点。这个挂载过程是作为响应式效应执行的，这意味着它会追踪所有使用的响应式依赖项。

当组件的响应式状态发生变化时，相关的响应式效应会重新运行，生成一个新的虚拟 DOM 树。然后，Vue 会使用一种 **diffing 算法** 来比较这个新的 VDOM 树和旧的 VDOM 树。该算法的目标是高效地识别两个树之间的最小差异。在 diffing 算法识别出变化后，Vue 会将必要的更新应用到真实 DOM 上。这个过程被称为 **patching**（打补丁）。这最大限度地减少了重新渲染并提高了性能，避免了不必要的 DOM 操作。

Vue 的独特优势在于其对编译器和运行时的双重控制，这使得 **编译器知情虚拟 DOM** 成为可能。编译器静态分析模板并在生成的代码中嵌入提示，使运行时能够采取捷径。

- **静态提升（Static Hoisting）**：模板中不包含任何动态绑定的部分被认为是静态的。在初始渲染期间，它们的 vnode 会被创建并缓存，然后在后续的重新渲染中重复使用，从而跳过重新创建和 diffing。连续的静态元素可以被压缩成一个“静态 vnode”，并通过直接设置 `innerHTML` 进行挂载，以进一步优化。
- **补丁标志（Patch Flags）**：对于具有动态绑定的元素，编译器会在编译时推断出所需的特定更新类型（例如，类、样式、文本），并将此信息直接编码到 vnode 创建调用中作为“补丁标志”。运行时渲染器使用位运算快速检查这些标志，确定更新所需的最小工作量。这避免了对每个 vnode 进行完整的属性 diffing。
- **树扁平化（Tree Flattening）**：Vue 识别“块”（模板中具有稳定内部结构的部分）。块的根节点是使用特殊的 `createElementBlock()` 调用创建的。每个块只追踪其具有补丁标志的**动态后代节点**（而不仅仅是直接子节点）。这导致了一个动态节点的扁平化数组。当组件重新渲染时，渲染器只需遍历这个扁平化数组，大大减少了需要协调的节点数量，并有效地跳过了静态部分。像 `v-if` 和 `v-for` 这样的指令会创建新的块节点，从而为其父级维护一个稳定的结构。

---

## 5. 组件生命周期

每个 Vue 组件实例在创建时都会经历一系列初始化步骤：创建、数据观察、模板编译、挂载到 DOM、数据变化时更新 DOM，以及最终销毁。生命周期钩子是预定义的方法，在组件存在的特定时间点执行，允许开发者在这些阶段运行自定义代码。提供的钩子包括：

| 钩子函数         | 触发时机                                                                 |
|------------------|--------------------------------------------------------------------------|
| `onBeforeMount`  | 组件挂载到DOM之前                                                        |
| `onMounted`      | 组件挂载到DOM之后（可以安全操作DOM）                                      |
| `onBeforeUpdate` | 响应式数据变化导致组件更新之前                                            |
| `onUpdated`      | 响应式数据变化导致组件更新之后                                            |
| `onBeforeUnmount`| 组件卸载之前（适合清理工作）                                              |
| `onUnmounted`    | 组件卸载之后                                                             |

---

**`setup()` 替代了 VUE2的 `beforeCreate` 和 `created`** .  在组合式API中，这两个钩子被`setup()`函数取代，所有初始化代码都应放在这里：

```javascript
setup() {
   // 这里相当于原来的 created 阶段
   const count = ref(0)
   
   // 注册生命周期钩子
   onMounted(() => {
      console.log('组件已挂载')
   })
   
   return { count }
}
```

所有钩子必须在 setup() 中同步注册​（不能在异步回调中使用），钩子函数接受一个回调函数作为参数。在组合式API中，this 不可用，所有数据都应通过 ref/reactive 声明并在 setup() 中返回。

## 5. 构建工具Vite

Vite 是一款旨在提供更快、更精简的现代 Web 项目开发体验的构建工具。

### Vite功能

- **开发阶段**：Vite 通过 **原生 ES 模块 (ESM)** 直接向浏览器提供源代码。这意味着在开发过程中，它避免了预先捆绑整个应用程序。浏览器负责模块解析和加载。Vite 利用 `esbuild` 对依赖项和 TypeScript 进行极速转译。`index.html` 在 Vite 项目中居于核心地位，被视为源代码和应用程序的入口点。
- **部署阶段**：对于生产环境，Vite 使用 **Rollup** 捆绑代码。Rollup 预配置为输出高度优化的静态资产。

### Vite原理

当浏览器请求你的 index.html 文件时，Vite 的开发服务器（基于 Node.js）不会直接将原始的 index.html 文件发送给浏览器。在发送之前，Vite 会对其进行一些预处理。这些预处理的目的是为了让浏览器能够正确地加载和运行你的基于 ES 模块的应用程序。

- 注入客户端入口脚本： Vite 需要在浏览器中运行一些客户端代码来处理模块加载、热模块替换 (HMR) 等功能。Vite 会在你的 index.html 中自动注入一个 `<script type="module" src="/@vite/client"></script>` 类似的标签。这个脚本是 Vite 客户端的核心，它负责与开发服务器建立 WebSocket 连接，监听文件变化，并执行 HMR 更新。
- 处理环境变量和模式： 如果你在 Vite 配置文件中定义了环境变量或使用了不同的构建模式（例如 development, production），Vite 可能会在 index.html 中注入一些全局变量，以便你的客户端代码可以访问这些信息。
- 转换 `<base>` 标签： 如果你的应用配置了 base 选项（用于指定应用的部署路径），Vite 可能会调整 index.html 中的 `<base>` 标签。
- 处理 CSS 注入： 当你的组件或模块中引入 CSS 文件时，Vite 会将这些 CSS 提取出来，并在 index.html 的 `<head>` 标签中动态注入 `<link>` 标签，以便浏览器加载这些样式。在开发阶段，为了实现 HMR，CSS 的处理方式可能有所不同，可能会通过 JavaScript 动态注入样式。
- 其他内部逻辑所需的代码： Vite 可能会根据其内部的运行逻辑，注入一些其他的辅助性代码或属性。

---

## 6. 开发建议

对于Vue3开发建议：

- 优先采用 **组合式 API**，充分利用 `ref`和`setup()` 来优化代码组织和可扩展性;
- 优先选择 Vite 作为构建工具，以充分利用其卓越的开发性能和优化的生产构建能力;
- 在大型项目中，应利用 TypeScript 来提高代码质量和可维护性;

## 7. 举例

使用下面命令创建工程：

```bash
$ pnpm create vite@latest vue-demo --template vue
$ cd vue-demo
$ tree .
.
├── README.md
├── index.html
├── package.json
├── public
│   └── vite.svg
├── src
│   ├── App.vue
│   ├── assets
│   │   └── vue.svg
│   ├── components
│   │   └── HelloWorld.vue
│   ├── main.js
│   └── style.css
└── vite.config.js
```

也可以使用 ts

```bash
pnpm create vite@latest vue-demo --template vue-ts
```

### 文件结构解析

1. **`index.html`**  
   - 这是项目的入口文件，浏览器加载的第一个文件。它包含了应用的根节点（如 `<div id="app"></div>`）以及对 `main.js` 的引用。
   - Vite 会自动处理 `index.html` 中的资源引用，将其解析为构建后的路径。

2. **`vite.config.js`**  
   - Vite 的配置文件，用于定义构建和开发服务器的行为。例如，可以配置别名、代理、插件等。

3. **`public/`**
   - 存放静态资源（如图片、图标等），这些资源不会被 Vite 处理，构建时会直接复制到输出目录。`vite.svg` 是一个示例文件。

4. **`src/`**  
   - 项目的核心代码目录，包含以下内容：
     - **`App.vue`**：主组件，定义了应用的整体结构和逻辑。
     - **`main.js`**：应用的入口文件，负责挂载 Vue 应用到 `index.html` 中的根节点。
     - **`components/`**：存放子组件，例如 `HelloWorld.vue`。
     - **`assets/`**：存放需要通过 JavaScript 或 CSS 引用的资源，例如 `vue.svg`。
     - **`style.css`**：全局样式文件。

### 构建与部署流程

1. **开发阶段**  
   - 运行 `pnpm dev` 启动 Vite 开发服务器。Vite会启动一个基于nodejs的开发服务器。
   - Vite 会解析 `index.html`，并通过原生 ES 模块加载 `main.js`.
   - `main.js` 引入了 `App.vue`，Vite 会动态编译 `.vue` 文件，将其转换为 JavaScript 模块。
   - 浏览器通过 HMR（热模块替换）实时更新页面，无需刷新。

2. **构建阶段**  
   - 运行 `pnpm build` 生成生产环境的构建文件。
   - Vite 使用 Rollup 将所有模块打包为优化后的静态资源，输出到 `dist/` 目录。
   - 构建过程包括以下步骤：
     - **HTML 处理**：`index.html` 中的资源路径会被替换为构建后的路径。
     - **CSS 提取**：所有样式会被提取到单独的 CSS 文件中。
     - **JavaScript 打包**：所有模块会被合并、压缩为一个或多个 JavaScript 文件。
     - **资源优化**：图片、字体等资源会被压缩或内联。

3. **部署阶段**  
   - 将 `dist/` 目录中的文件上传到静态服务器（如 Nginx、Apache）或部署到 CDN。
   - 浏览器加载 `index.html`，通过 `<script>` 标签加载打包后的 JavaScript 文件。
   - JavaScript 文件会初始化 Vue 应用，挂载到 `index.html` 中的根节点，最终呈现为一个完整的页面。

### 浏览器加载成单个页面的原理

- **模块化加载**：  
  浏览器通过 `<script>` 标签加载打包后的 JavaScript 文件。Vite 使用 Rollup 将所有模块（包括组件、样式、资源）打包为一个或多个文件，确保浏览器只需加载少量文件即可运行整个应用。

- **Vue 应用初始化**：  
  `main.js` 是应用的入口文件，它通过 `createApp` 创建 Vue 实例，并将 `App.vue` 挂载到 `index.html` 中的根节点（如 `<div id="app"></div>`）。`App.vue` 是应用的主组件，它可以通过子组件（如 `HelloWorld.vue`）构建页面结构。

- **资源优化**：  
  构建过程中，Vite 会将所有资源（如图片、字体）优化为浏览器友好的格式（如压缩或内联）。这些资源会通过 JavaScript 或 CSS 文件引用，确保页面加载时能够正确显示。

通过上述流程，多个文件（如组件、样式、资源）被打包为少量优化后的文件，浏览器加载这些文件后即可呈现为一个完整的页面。
