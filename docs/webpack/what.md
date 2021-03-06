# webpack 是什么？

需要牢牢记住并强调的一点：webpack 只是一个静态模块打包器（static module bundler）

它所承担的核心指责就是生成一张依赖图（dependency graph）并最终合成一个或多个包（bundle）

_这意味着：当你发现你的 eslint 报错时，请查阅 eslint 文档，而不是搜索为什么 webpack 报错_

## static module bundler

打包器，顾名思义，将多个文件合成为一个文件。

它是根据模块的引用关系来生成依赖图，因此 webpack 最关键的就是如何索引，处理这些模块语法和引用关系。

webpack 只能处理静态资源(static assets)

```javascript
const staticImage = require('static/xxx.png') // 静态资源
const imageName = 'xxx.png'
const dynamicImage = require(`static/${imageName}`) // 动态资源
```

## webpack 因何诞生

要回答 why webpack, 实际上就是要回答 why static module bundler。

没有 webpack 时我们遇到了什么问题

1. 代码太多了（工程规模大）
   JS 早期的设计只是为了能在页面里实现一些简单的交互，随着网速的提升，Ajax 的出现和浏览器的升级，越来越多的功能从服务端转移到前端，且交互越来越复杂，UI 对动效等的要求也在提升，需要实现的功能多了，工程规模自然庞大了，对于模块化的需求也应运而生，而 JS 早期的设计中并没有考虑到模块化的问题（不同域的脚本甚至共享一个命名空间）。
   因此当时出现了 CMD（Commonjs，同步，服务于 nodejs 服务端），AMD（Requirejs，异步，服务于浏览器端），后来制定了标准 ES Modules。
   很可惜，早期由于标准不统一，浏览器没有实现，即使现代浏览器普遍支持 ES Modules（支持到的 stage 各有不同），但是用户浏览器（IE8,11）更新换代需要时间，因此至今仍然无法直接在生产环境中直接使用模块化语法。
2. HTTP1.1（并行限制和包头浪费）
   在 chrome 中，HTTP1.1 的同域名并行请求限制为 6 个，即如果同时有 10 个 HTTP1.1 的请求，那么只能同时请求 6 个，剩余 4 个得排队等待前面的请求完成。
   这意味着即使客户的浏览器支持模块化语法，但是因为同域名的并行限制，也会极大程度上影响用户网页的加载速度。
   是不是浏览器放开限制，就不存在这个问题呢？很不幸，不是，每一次 HTTP 传输都需要附带 HTTP 包头，包头中的绝大多数字段都是重复的，对业务无意义的，因此这一部分的网络传输时间实质上也是算到了用户头上，影响了网页的加载速度。

_说明：通常 webpack 用于打包面向服务器的代码，因为 nodejs 自身支持 CJS 规范，且 nodejsv13 起支持 ES Modules(mjs)_

_说明：CJS 是随着 Nodejs 的流行而流行的，因此，CommonJS 的生态的繁荣要远胜于 AMD，尽管当时有一些项目同时支持了两种规范，还有 UMD 的通用包装_

### webpack 解决的核心问题

1. 将模块化的源文件打包为一个或多个文件
2. 支持尽可能多的模块化规范

### 不止于此

webpack 如果只是止步于模块打包器，那么它也不会像如今一般无处不在。

在这之前，还是先看看需要解决的问题

1. ES6
   ES6 的标准一经发布就广受欢迎（块级作用域、解构、箭头表达式、类、Promise）实实在在的解决了开发者的痛点，开发者们已经迫不及待要上马 ES6 了。
   但是老大难的问题又来了，用户的浏览器的升级换代需要时间，浏览器对最新标准的支持也需要时间。
   幸运的是，JS 语言的灵活性使得可以用 ES5 的语法实现浏览器所不支持的 ES6 语法，这被称为 polyfill 和 transform，实现这一特性的是 core-js，集成工具是 babel。
2. 其他非 JS 资源的打包问题
   有些资源虽然不属于 JS Module 的一部分，但是对于其实也是有对它按模块进行处理的需求的，
   例如图片，需要对它做引用路径的转换，小的图片直接转为 base64，
   例如 CSS，需要对它做引用路径的转换或嵌入到页面的 style 标签中，甚至更进一步可以做预处理
   例如 TS，需要将其转换为 JS
3. 开发效率低
   随着 JS 工程体量的增大和前置步骤的增多（TS 转 JS，SASS 转 CSS，ejs 转 html，图片压缩等），每一次的修改都需要漫长的重新编译，并刷新页面来重新预览。
   因此热更新的需求也是与日俱增。
   热更新即 启动服务 -> 客户端监听 websocket -> 开发者修改源文件 -> 目标文件更新 -> websocket 通知客户端 -> 客户端拉取更新 -> 客户端热更新或自动刷新

_以上提到的这些问题如果是直接使用脚手架如 CRA 等，是不了解曾经的开发者们一路走来的幸苦的，在 15 年时，还是 Grunt 和 Gulp 的天下，任务都需要自己配置，需要预先规划代码结构，同时离不开的工具还有 livereload_

webpack 解决了以上问题：

1. **Loader**，webpack 将所有引用依赖关系都视为模块，默认可以处理的模块是 JS、JSON、WebAssembly，对于默认不支持的模块类型，webpack 提供了 Loader 来接入，
   例如 Babel，Babel 自身提供了命令行和多种集成方式，在 webpack 中，它的集成是通过`babel-loader`，简单的配置后，所有的 JS 模块在引用前都会被 babel 预处理。
   同理的还有各式资源文件，`raw-loader`,`url-loder`,`file-loader`是三种常见的 loader 用于对资源文件进行引用和处理。
   在 webpack 的设计里，Loader 可以串联执行，这就留下了很丰富的想象空间，例如 SCSS 文件，可以先使用`sass-loader`转换为 css，再使用`postcss-loader`进行`autoprefixer`,`cssnano`等操作，最后使用`css-loader`来正确解析 CSS 模块，最后使用`style-loader`使得该 CSS 文件在运行时可以作为`style`标签的内容被插入到页面中。
2. **HMR(webpack-dev-server)**，模块热更新，webpack 保存了模块之间的依赖关系，这意味这当我们修改某个文件时，webpack 可以仅替换、添加或删除对应模块，同时 webpack 还做到了保留应用状态，只更新变更内容，非常的强大。
   要做到这一点，需要编译器（compiler），模块 Loader 实现 HMR 接口，运行时（runtime）配合运作，webpack 的设计使得这一切有可能发生。

webpack 为了拓展自己的能力，还做了一些额外的工作

1. **Plugin**，loader 还是有许多能力无法实现，例如需要编译上下文的功能、额外生成文件的功能，这些能力通通交由 Plugin 来实现。
   webpack 直接将编译器上下文提供给了插件，并提供了大量非常实用的钩子函数，这意味着插件可以在 webpack 编译的任何环节做自定义的修改，以实现非常强大的功能，
   例如 SplitChunksPlugin，提供了对 webpack 输出 bundle 的任意裁剪组合。
2. **Tree Shaking**，webpack 不满足于仅仅生成依赖图并按依赖图拼接文件，它深入代码细节，剔除掉没有副作用的没有被引用的模块或文件，这为包大小的优化提供了最大的可能性。
3. **Sourcemap**，webpack 输出的通常是经过转换后的代码，Souremap 是调试中避不开的一环，webpack 不仅提供了默认支持模块的 Souremap 支持，也为第三方 Loader 提供了生成 souremap 的接口，只要正确配置，即使你的 CSS 文件经历了 5 道转换流程，最终依然可以通过 Souremap 找回源代码。

此外，webpack 为了提升大型项目的性能，采取了很多优化措施。

## webpack 的局限性

webpack 诞生的背景是 JS 生态模块化不规范，浏览器支持不到位的年代，
在如今 2020 年，ES Module 统一标准，旧的浏览器（Android4.4，IE8,11）即将淘汰，HTTP2 的部署环境已然成熟的年代，
我们写的源代码不需要打包也是有可能直接跑在浏览器上的，即 bundless 概念
vite,snowpack 等这些就是基于这样的理念而诞生的。
