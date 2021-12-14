##  前端构建工具vite介绍
本文主要介绍vite，同时也简单引出vite与当下最热门的构建工具webpack之间的差别。

## 主流构建工具对比
构建工具指能自动对代码执行检验、转换、压缩等功能的工具。常见功能包括：代码转换、代码打包、代码压缩、HMR、代码检验。构建工具也随着前端技术的发展，从Browserify、Gulp到Parcel，从Webpack到Rollup，一直到最近比较火的面向非打包的Snowpack和Vite。

- Gulp
  - 基于流的自动化构建工具（工程化）
  - 配置复杂度高，偏向编程式，需要定义task处理构建
  - 支持监听读写文件
  - 可搭配Browserify等模块化工具来使用

- Webpack
  - 预编译模块化方案（工程化：大而全）
  - 通过配置文件达到一站式配置
  - loader进行资源转换，功能全面（css+js+icon+front）
  - 插件丰富，灵活扩展
  - 社群庞大
  - 大型项目构建慢

- Rollup
  - 基于ES6打包（模块打包工具）
  - Tree-shaking
  - 打包文件小且干净，执行效率更高
  - 更专注于JS打包

- Snowpack
  - 基于ESM运行时编译（工程化：ESM运行时）
  - 无需递归循环依赖组装依赖树
  - 默认输出单独的构建模块（未打包），可选择不同打包器（webpack、rollup- 等）

- Vite
  - 基于ESM运行时打包
  - 借鉴了Snowpack
  - 生产环境使用Rollup，集成度更高，相比Snowpack支持多页面、库模式、动态导入自动polyfill等

## 一、vite简介
Vite是一种新型前端构建工具，能够显著提升前端开发体验，主要由两部分组成：
  - 一个开发服务器，它基于 原生 ES 模块 提供了 丰富的内建功能，如速度快到惊人的 模块热更新（HMR）。
  - 一套构建指令，它使用 Rollup 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

### 1.1 为什么需要vite？<br/>
在浏览器支持 ES 模块之前，JavaScript 并没有提供原生机制让开发者以模块化的方式进行开发。因此开发者使用工具抓取、处理并将源码模块串联成可以在浏览器中运行的文件。这也产生了如webpack、Rollup、Parcel等前端构建工具

然而当现在我们的项目代码会越来越大，使用 JavaScript 开发的工具通常需要很长时间才能启动开发服务器，即使使用 HMR，文件修改后的效果也需要几秒钟才能在浏览器中反映出来。

随着浏览器开始支持原生的ES模块，Vite 旨在利用生态系统中的新进展解决上述问题。

### 1.2 vite特点
- 快速的冷启动，构建一个基于浏览器原生 ES imports 的开发服务器。通过劫持浏览器的这些请求，并在后端进行相应的处理将项目中使用的文件通过简单的分解与整合，然后再返回给浏览器，利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用
- 即时的热模块更新，基于ESM，编辑一个文件后，Vite 只需要精确地使已编辑的模块与其最近的 HMR 边界之间的链失效（大多数时候只需要模块本身），使 HMR 更新始终快速，无论应用的大小
- 真正的按需编译，Vite 以 原生 ESM 方式提供源码。这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

### 1.3 开发环境中vite与webpack对比
vite
![avatar](./esm.3070012d.png)
webpack
![avatar](./bundler.37740380.png)
Vite开发环境冷启动无需打包，无需分析模块之间的依赖，同时也无需在启动开发服务器前进行编译，启动时还会使用 **esbuild 预构建依赖** 。Esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。

- 预构建
    - Vite 将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM。
    - 性能： Vite 将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能。例如，import { debounce } from 'lodash-es' 时，浏览器同时会发出 600 多个 HTTP 请求！而通过预构建 lodash-es 成为一个模块，我们就只需要一个 HTTP 请求。

以webpack来说，启动后会做一堆事情，经历一条很长的编译打包链条，从入口开始需要逐步经历语法解析、依赖收集、代码转译、打包合并、代码优化，最终将高版本的、离散的源码编译打包成低版本、高兼容性的产物代码，这可满满都是 CPU、IO 操作，可以看得出来，仅仅是兼容性处理这一操作，就让webpack做多了许多事情。

针对HMR慢，即使只有很小的改动，Webpack依然需要将该模块的相关依赖模块全部编译一次。而Vite仅需让浏览器重新请求该模块即可，更新速度与项目复杂度无关。

| webpack      | vite |
| ----------- | ----------- |
| 先打包生成bundle，再启动开发服务器      | 先启动开发服务器，利用新一代浏览器的ESM能力，无需打包，直接请求所需模块并实时编译       |
| HMR时需要把改动模块及相关依赖全部编译	   | HMR时只需让浏览器重新请求该模块，同时利用浏览器的缓存（源码模块协商缓存，依赖模块强缓存）来优化请求        |

### 1.4 vite的生产环境
- 仍需要打包，尽管原生 ESM 现在得到了广泛支持，但由于**嵌套导入会导致额外的网络往返，在生产环境中发布未打包的 ESM 仍然效率低下**（即使使用 HTTP/2）。为了在生产环境中获得最佳的加载性能，最好还是将代码进行 tree-shaking、懒加载和 chunk 分割（以获得更好的缓存）。

- Rollup替换esbuild打包，针对构建 应用 的重要功能仍然还在持续开发中 —— 特别是代码分割和 CSS 处理方面。就目前来说，Rollup 在应用打包方面更加成熟和灵活。尽管如此，当未来这些功能稳定后，不排除使用 esbuild 作为生产构建器的可能。

## 二、vite原理
### 2.1 ES Module
浏览器解析ES module的过程：
1. 识别带有熟悉 type="module"的script标签
2. 获取并解析该标签内的js内容。
3. 识别 import语法，生成请求url,向服务器请求该地址的模块

下面是一张caniuse中说明的浏览器对于 ES Module的静态import语法的支持情况。可以看出除了IE外的主流浏览器基本上都支持了 ES Module的import语法。
![avatar](./caniuseesm.png)

### 2.2 请求拦截
当我们访问localhost:3000/时，这个请求发送到 Vite 后端之后经过静态资源服务器的处理，会进而请求到 /index.html，此时 Vite 就开始对这个请求做拦截和处理了。

```
// index.html
<div id="app"></div>
<script type="module">
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
</script>
```
到了浏览器中，代码会变成<br/>
<img src="./微信截图_20211211180924.png" width="300">

浏览器只能解析以'/', './', 或 '…/'开头的模块路径，对于像引用node_modules中的模块，如图中的vue，vite会对这样直接引用 node_modules 的模块都做了路径的替换，换成了 /@modules/ 并返回回去。而后浏览器收到后，会发起对 /@modules/:id 的请求，然后被 Vite 客户端做一层拦截来解析模块的真实位置。

首先正则匹配请求路径，如果是/@modules开头就进行后续处理，否则就跳过。若是，会设置响应类型为js，读取真实模块路径内容，返回给客户端。

客户端注入本质上是创建一个script标签（type='module'），然后将其插入到head中，这样客户端在解析html是就可以执行代码了

```
export const moduleRE = /^\/@modules\//
// plugin for resolving /@modules/:id requests.
app.use(async (ctx, next) => {
  if (!moduleRE.test(ctx.path)) {
    return next()
  }
  // path maybe contain encode chars
  const id = decodeURIComponent(ctx.path.replace(moduleRE, ''))
  ctx.type = 'js'
  const serve = async (id: string, file: string, type: string) => {
    // 在代码中做一个缓存，下次访问相同路径直接从 map 中获取 304 返回
    moduleIdToFileMap.set(id, file)
    moduleFileToIdMap.set(file, ctx.path)
    debug(`(${type}) ${id} -> ${getDebugPath(root, file)}`)
    await ctx.read(file)
    return next()
  }
 }
  // 兼容 alias 情况
  const importerFilePath = importer ? resolver.requestToFile(importer) : root
  const nodeModulePath = resolveNodeModuleFile(importerFilePath, id)
  // 如果是个 node_modules 的模块，读取文件。
  if (nodeModulePath) {
   return serve(id, nodeModulePath, 'node_modules')
  }
})
```

对于vue文件，代码会被拆分成 template, css, script 模块三个模块进行分别处理。最后会对 script, template, css 发送多个请求获取

### 2.3 热更新
Vite 的热加载原理，其实就是在客户端与服务端建立了一个 websocket 连接，当代码被修改时，服务端发送消息通知客户端去请求修改模块的代码，完成热更新。

服务端：服务端做的就是监听代码文件的改变，在合适的时机向客户端发送 websocket 信息通知客户端去请求新的模块代码。

客户端：Vite 中客户端的 websocket 相关代码在处理 html 中时被写入代码中。可以看到在处理 html 时，vite/client 的相关代码已经被插入。

```
export const clientPublicPath = `/vite/client`
const devInjectionCode = `\n<script type="module">import "${clientPublicPath}"</script>\n`
async function rewriteHtml(importer: string, html: string) {
  return injectScriptToHtml(html, devInjectionCode)
}
```
当request.path 路径是 /vite/client 时，请求获取已经提前写好的关于 websocket 的代码。因此在客户端中我们创建了一个 websocket 服务并与服务端建立了连接。

Vite 的 WS 客户端目前监听这几种消息：
   - connected: WebSocket 连接成功
   - vue-reload: Vue 组件重新加载（当你修改了 script 里的内容时）
   - vue-rerender: Vue 组件重新渲染（当你修改了 template 里的内容时）
   - style-update: 样式更新
   - style-remove: 样式移除
   - js-update: js 文件更新
   - full-reload: fallback 机制，网页重刷新

```
import { HMRRuntime } from 'vue' // 来自 Vue3.0 的 HMRRuntime

console.log('[vite] connecting...')

declare var __VUE_HMR_RUNTIME__: HMRRuntime

const socket = new WebSocket(`ws://${location.host}`)

// Listen for messages
socket.addEventListener('message', ({ data }) => {
  const { type, path, id, index, timestamp, customData } = JSON.parse(data)
  switch (type) {
    case 'connected':
      console.log(`[vite] connected.`)
      break
    case 'vue-reload':
      import(`${path}?t=${timestamp}`).then((m) => {
        __VUE_HMR_RUNTIME__.reload(path, m.default)
        console.log(`[vite] ${path} reloaded.`) // 调用 HMRRUNTIME 的方法更新
      })
      break
    case 'vue-rerender':
      import(`${path}?type=template&t=${timestamp}`).then((m) => {
        __VUE_HMR_RUNTIME__.rerender(path, m.render)
        console.log(`[vite] ${path} template updated.`) // 调用 HMRRUNTIME 的方法更新
      })
      break
    case 'style-update':
      updateStyle(id, `${path}?type=style&index=${index}&t=${timestamp}`) // 重新加载 style 的 URL
      console.log(
        `[vite] ${path} style${index > 0 ? `#${index}` : ``} updated.`
      )
      break
    case 'style-remove':
      const link = document.getElementById(`vite-css-${id}`)
      if (link) {
        document.head.removeChild(link) // 删除 style
      }
      break
    case 'js-update':
      const update = jsUpdateMap.get(path)
      if (update) {
        update(timestamp) // 用新的时间戳加载并执行 js，达到更新的目的
        console.log(`[vite]: js module reloaded: `, path)
      } else {
        console.error(
          `[vite] got js update notification but no client callback was registered. Something is wrong.`
        )
      }
      break
    case 'custom':
      const cbs = customUpdateMap.get(id)
      if (cbs) {
        cbs.forEach((cb) => cb(customData))
      }
      break
    case 'full-reload':
      location.reload()
  }
})
```

## vite缺点
1. 目前 Vite 还是使用的 es module 模块不能直接使用生产环境（兼容性问题）。默认情况下，无论是 dev 还是 build 都会直接打出 ESM 版本的代码包，这就要求客户浏览器需要有一个比较新的版本，这放在现在的国情下还是有点难度的。不过 Vite 同时提供了一些弥补的方法，使用 build.polyfillDynamicImport 配置项配合 @vitejs/plugin-legacy 打包出一个看起来兼容性比较好的版本。
2. 生产环境使用 rollup 打包会造成开发环境与生产环境的不一致。
3. 很多 第三方 sdk 没有产出 ems 格式的的代码，这个需要自己去做一些兼容。


## 参考
- [Vite介绍及实现原理](https://zhuanlan.zhihu.com/p/424842555)
- [Vite 原理浅析](https://juejin.cn/post/6844904146915573773#heading-9)
- [下一代前端构建工具Vite](https://juejin.cn/post/6914570873580027917#heading-7)