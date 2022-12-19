---
title: Vite和Webpack对比
tags:
---

Vite和Webpack对比
<!--more-->
### Vite

- 一个开发服务器，它基于 [原生 ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 提供了 [丰富的内建功能](https://cn.vitejs.dev/guide/features.html)，如速度快到惊人的 [模块热更新（HMR）](https://cn.vitejs.dev/guide/features.html#hot-module-replacement)。
- 一套构建指令，它使用 [Rollup](https://rollupjs.org/) 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

### 为什么Vite

- 启动快

  Vite区分依赖和源码,依赖采用esbuild预构建.源码采用esm方式提供,由浏览器做处理

- 热更新快

  Vite采用 源码协商缓存,依赖强缓存

  改动一个文件时，Vite 只需要精确地使已编辑的模块与其最近的 HMR 边界之间的链失活

- 缺点

  Vite支持es2015语法.不提供es2015之前版本代码转换(`@vitejs/plugin-legacy`和`rollupOption`均不可)

  Vite社区不太完善,目前无官方`postcss8`支持

### Webpack

各个阶段执行的操作如下：

1. 初始化参数：从配置文件(默认webpack.config.js)和shell语句中读取与合并参数，得出最终的参数
2. 开始编译(compile)：用上一步得到的参数初始化Comiler对象，加载所有配置的插件，通过执行对象的run方法开始执行编译
3. 确定入口：根据配置中的entry找出所有的入口文件
4. 编译模块：从入口文件出发，调用所有配置的Loader对模块进行翻译,再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过处理
5. 完成编译模块：经过第四步之后，得到了每个模块被翻译之后的最终内容以及他们之间的依赖关系
6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk，再将每个chunk转换成一个单独的文件加入输出列表中，这是可以修改输出内容的最后机会
7. 输出完成：在确定好输出内容后，根据配置(webpack.config.js && shell)确定输出的路径和文件名，将文件的内容写入文件系统中(fs)

在以上过程中，webpack会在特定的时间点广播特定的事件，插件监听事件并执行相应的逻辑，并且插件可以调用webpack提供的api改变webpack的运行结果

- Entry: 入口，webpack构建第一步从entry开始
- module:模块，在webpack中一个模块对应一个文件。webpack会从entry开始，递归找出所有依赖的模块
- Chunk：代码块，一个chunk由多个模块组合而成，用于代码合并与分割
- Loader: 模块转换器，用于将模块的原内容按照需求转换成新内容
- Plugin:拓展插件，在webpack构建流程中的特定时机会广播对应的事件，插件可以监听这些事件的发生，在特定的时机做对应的事情

#### SourceMap

一项将编译、打包、压缩后的代码映射回源代码的技术.

文件指纹

文件指纹是打包后输出的文件名的后缀。

`Hash`：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改

`Chunkhash`：和 Webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash

`Contenthash`：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变

##### 使用过的loader和plugin

`Loader` 在 module.rules 中配置，作为模块的解析规则，类型为数组。每一项都是一个 Object，内部包含了 test(类型文件)、loader、options (参数)等属性。

`Plugin` 在 plugins 中单独配置，类型为数组，每一项是一个 Plugin 的实例，参数都通过构造函数传入。

Loader

babel-loader

sass-loader

postcss-loader

image-loader

plugin

speed-measure-webpack-plugin 可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)

webpack-bundle-analyzer  可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)

DllPlugin 为了极大减少构建时间，进行分离打包

#### 构建速度

- 使用高版本的node和webpack

- 多线程 

  thread-loader

- 压缩代码 

  通过 mini-css-extract-plugin 提取 Chunk 中的 CSS 代码到单独文件，通过 css-loader 的 minimize 选项开启 cssnano 压缩 CSS

- 图片压缩

  Image-webpack-loader

- 缩小打包作用域

  exclude/include(确定loader规则范围)

  resolve.modules 指明第三方模块的绝对路径 (减少不必要的查找)

- 提取页面公共资源

  使用 html-webpack-externals-plugin，将基础包通过 CDN 引入，不打入 bundle 中

- 充分利用缓存提升二次构建速度

  babel-loader 开启缓存

  terser-webpack-plugin 开启缓存

  使用 cache-loader 或者 hard-source-webpack-plugin

#### webpack3和4对比

 在 `webpack4` 中不再支持 `CommonsChunkPlugin`，而是使用 `splitChunks` 替代，那么这两者有什么区别？为什么要废弃之前的，使用 `splitChunks` 的呢？

根据我查到的资料显示，`CommonsChunkPlugin` 存在以下的问题：

- `CommonsChunkPlugin` 会提取一些我们不需要的代码
- 它在异步模块上效率低下
- 很难使用，配置也很难理解

相比，`splitChunks` 具有以下特点：

- 它不会打包不需要的模块
- 对异步模块有效（默认情况下是打开的）
- 更加容易使用和更加自动化
