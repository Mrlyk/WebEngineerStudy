## 前端工程化项目构建-笔记  
### 一、工程目录下安装webpack 与 webpack-cli
>[webpack官方文档](https://webpack.js.org/concepts/)
- webpack 打包工具,工程化基础工具
- webpack-cli用来在命令行中执行webpack命令

#### 1.安装与初始化
```
// 初始化一个由node管理模块的项目
npm init

// 存储到本地开发依赖,只需要在开发时使用,不需要在生产时使用-D或--save-dev
npm install webpack webpack-cli -D
```
tips:如果未配置webpack入口与输出目录,则默认入口目录为src/index.js,输出为dist/main.js

#### 2.初始化目录
```
webpack-demo
├── package.json
├── dist
├── index.html
└── src
    └── index.js
```
在index.html中将打包的输出文件(main.js)引入.

#### 3.初始化webpack配置
>在根目录下添加webpack.config.js
>>path.resolve() 方法会把一个路径或路径片段的序列解析为一个绝对路径。
  path.resolve([…paths])里的每个参数都类似在当前目录执行一个cd操作，从左到右执行，返回的是最后的当前目录.
  ```
  // exp:path.resolve('/foo/bar','./baz');相当于：
   cd /foo/bar //此时当前路径为 /foo/bar
   cd ./baz //此时路径为 /foo/bar/baz
  ```
#### 初始化配置的一些问题
1. 引入内部js文件放在index.html的body中,放在head中的是外部js文件.内部js文件如果放在head中当加载到的时候会直接执行,
这时document还没加载完会报错.
2. 

___
### 二、常用插件plugins
>插件用于扩展webpack自身的功能,使用插件让我们能够干预webpack的构建过程,达到自定义的目的
- HtmlWebpackPlugin 自动生成入口文件(html入口文件)，并将最新的资源注入到 HTML 中.
- CommonsChunkPlugin 用以创建独立文件，常用来提取多个模块中的公共模块
- DefinePlugin 用以定义在编译时使用的全局常量.
- DllPlugin 拆分 bundle 减少不必要的构建.
- ExtractTextWebpackPlugin 将文本从 bundle 中提取到单独的文件中。常见的场景是从 bundle 中将 CSS 提取到独立的 css 文件中.
- HotModuleReplacementPlugin 在运行过程中替换、添加或删除模块，而无需重新加载整个页面.
- UglifyjsWebpackPlugin 对 js 进行压缩，减小文件的体积.
- CopyWebpackPlugin 将单个文件或整个目录复制到构建目录.一个常用的场景是将项目中的静态图片不经构建直接复制到构建后的目录.
#### 1.HtmlWebpackPlugin
>介绍:上面每次我们更改输出文件的名字, 也要手动去修改index.html中引入js的名字,很不方便.
本插件可以帮助我们自动生成入口文件(生成入口index.html,所以原来手动创建的可以删除),自动将生成的的资源注入到index.html中
```
//安装:
npm install html-webpack-plugin -D
```

##### 使用:
1.在webpack配置文件中先实例化
```
const HtmlWebpackPlugin = require("html-webpack-plugin")
```
2.在配置插件的数组中new一个该实例
```
plugins:[
    new HtmlWebpackPlugin()
    ]
```
3.配置参数  
[配置参数详见](https://segmentfault.com/a/1190000007294861)  
配置参数如果是title等可以传递的参数,可以在入口文件中使用脚本语言获取<%= htmlWebpackPlugin.options.title%>
```
// 传入一个对象来配置参数,常用的有template,filename
plugins:[
    new HtmlWebpackPlugin({
        template:'index.html', //指定一个模板来创建入口文件.会处理webpack.config.js中的参数,同时复制这个文件上的配置来生成新的入口文件
        filename:'../index.html'//指定文件名(默认index.html)和路径,默认在output的路径中
    })
    ]
```
---
### 三、loader及常用的一些loader
>webpack本身只能对js和json文件进行处理,为了对css,静态资源等文件进行处理,需要使用loader
对非js文件进行预处理.loader与plugins有些相似,但是loader只专注于对文件的transform,plugin的则不止于此.
- babel-loader 将 ES2015+ 代码转译为 ES5.
- ts-loader 将 TypeScript 代码转译为 ES5.
- css-loader 解析 @import 和 url()，并对引用的依赖进行解析.
- style-loader 在 HTML 中注入\<style>标签将 css 添加到 DOM 中。通常与 css-loader 结合使用.
- sass-loader 加载 sass/scss 文件并编译成 css.
- postcss-loader 使用 PostCSS 加载和转译 css 文件.
- html-loader 将 HTML 导出为字符串.
- vue-loader 加载和转译 Vue 组件.
- url-loader 和 file-loader 一样，但如果文件小于配置的限制值，可以返回 data URL.
- file-loader 将文件提取到输出目录，并返回相对路径
#### 1.style-loader/css-loader
>介绍:css-loader将css代码抽出import或require的模块代码交给下一个loader,这里即style-loader处理.
style-loader则将css代码直接使用\<style>标签插入入口文件的head中
```
//安装
npm i style-loader css-loader -D
```
##### 使用:
1.在webpack.config.js中新增module对象,表示要对模块进行配置处理  
2.在module中配置处理规则rules,规则中包含不同类来处理不同的模块  
3.一类模块配置多个loader时是从右往左,单个loader时可以直接用
```
loader: "css-loader"
```
4.include和exclude,一个是包含一个是不包含且exclude的优先级最高
```
module: {
        rules: [
            {
                test: /\.css/,   //test匹配:匹配所有含有.css的文件
                use:["style-loader","css-loader"] //使用这两个loader进行处理,从右到左
                include:[        //和test相同,但使用绝对路径直接进行匹配
                    path.resolve(__dirname, "app")
                ],
                exclude:[       //表示绝对不匹配的选项
                    path.resolve(__dirname, "app/demo")
                ]
            }
        ]
    }
```
### 四、搭建一个基于vue框架的脚手架

>搭建一个单文件的vue项目,需要使用vue-loader和vue-template-compiler

[vue-loader官方文档](https://vue-loader.vuejs.org/zh/)
- vue-loader 会解析 .vue 文件，提取每个语言块，如有必要会通过其它 loader 处理，
最后将他们组装成一个 ES Module，它默认导出一个 Vue.js 组件选项的对象。
- vue-template-compiler 会接解析 template 标签中的内容，预处理为 JS 渲染函数，并最终注入到从 \<script> 导出的组件中。
```
// 安装
npm i vue-loader vue-template-compiler -D
```
__注意__:在vue-loader v15版本更新后,不仅需要配置loader还需要引入一个VueLoaderPlugin,他在vue-loader的lib文件夹中.
这个插件是必须的！ 它的职责是将你定义过的其它规则复制并应用到 .vue 文件里相应语言的块。例如，如果你有一条匹配 /\.js$/ 的规则，那么它会应用到 .vue 文件里的 \<script> 块。

>创建根目录文件

1. 创建一个src/app.vue作为项目的根目录
2. 创建一个src/app.js作为应用的入口，它会把 src/app.vue 作为模板进行渲染(使用vue渲染函数)。同时 webpack 会将 src/app.js 作为入口进行打包。

>css使用scss语法,安装sass-loader node-sass
```
npm i sass-loader node-sass -D
```
node-sass是一个提供将node.js绑定到
