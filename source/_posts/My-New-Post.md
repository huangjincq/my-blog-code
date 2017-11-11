---
title: 从0用Vue2.5和Elemnt-ui2搭建一个后台模版（1.搭建框架）
date: 2017-11-11 10:36:35
tags: Vue
---

> 前言：用vue做后台管理系统做了半年，最近element-ui升级到2了，所以决定搭建一个基于ele2的后台模版，在这我将手把手javascript:void(null)分享一下如何从0搭建一个后台系统模版。本模版将会长期更新，以后维护中会把常用结局方案常用组件封装在另外一个系统中，模版保障：方便快速开发，极易扩展的一个模版，基于此模版，您只需要修改少量的代码，就能进行开发。
分享开发过程，待续未完......

## 概述
[地址](www.baidu.com)
此模板内涵盖：
1. 登录权限控制
2. 路由跳转拦截
3. 网络请求拦截
4. 侧边导航动态生成
5. 动态换肤
6. 结合Element-ui 2.0版本以上

等等包含后台系统常用功能，方便快速开发，极易扩展的一个模版，基于此模版，您只需要修改少量的代码，就能进行开发。

## 项目主依赖总览
1. [Vue.js](https://cn.vuejs.org/v2/guide/index.html)
2. [Vue-router](https://router.vuejs.org/zh-cn/)
3. [Vuex](https://vuex.vuejs.org/zh-cn/)
4. [axios](https://github.com/axios/axios) 一个封装了网络请求的库
5. [element-ui2](http://element-cn.eleme.io/#/zh-CN) 2.0版本以上，最近饿了吗团队除了2.0，1.+年后也不再维护
6. [normalize.css](https://github.com/necolas/normalize.css) css样式重置
7. [nprogress](https://github.com/rstacruz/nprogress) 一个加载进度条插件
8. [screenfull](https://github.com/sindresorhus/screenfull.js) 一个全屏的插件
9. [stylus](https://github.com/stylus/stylus)、stylus-loader 本项目使用的Css预处理器,也可自行选择其他与处理器
10. [eslint-config-vue](https://github.com/vuejs/eslint-config-vue) Eslint规则采用Vue源码的官方规则，你可以用自定义或者其他规则，[Eslint规则查询字典](http://eslint.cn/docs/rules/)

## 搭建前准备

### 一、环境准备
1. 最新node.js环境安装、npm、yarn、vue-cli脚手架安装这些就不在此多说。

### 二、模拟数据
1. 由于是个人项目，本模板中的数据都为[Easy Mock](http://www.easy-mock.com)生成的在线模拟数据。有兴趣的可以了解下。
    模拟数据的地址接口配置在根目录下 /config/dev.env.js 的相关配置中，改成您的请求地址即可
```javascript
module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  BASE_API: '"http://www.easy-mock.com/mock/59dc61571de3d46fa94cebc7/lolapp"'  // 这里换成您的请求地址即可
})
```
2.接口约定。一般在项目中前后端联系的接口都会有固定的约定，虽然本项目是模拟数据，但是为了做到规范，先简单约定好接口格式，方便后面请求拦截的时候判断，接口约定规则如下
```
1、数据交互统一为JSON格式
2、出参通用格式类似
   {
    "code": "H0000",
    "data": "88888888"
   }

   其中：code指返回码，有具体业务含义，如：
  S0000  系统错误!
  B0000  未登录或登录失效!
  H0000  执行成功

  首字母
  S: 系统错误
  B: 业务异常
  H: 提示
  M: 消息

  data指返回的json数据。

3、登陆后返回session_key，后续每次请求时都须将此作为token加入在header的Authorization信息中。
```

### 三、跨域问题
1. 由于本模板项目是用的 Easy Mock，不用前端解决跨域问题，但是如果您的实际需求中需要解决跨域问题 可以 百度 cors 跨域 或者 webpack 配置反向代理：这里具体就不详细说明了，百度有很多答案，我这里简单贴个图：
webpack配置   'config/index.js'
    ![webpack配置.png](http://upload-images.jianshu.io/upload_images/6914113-40a1de6d20924bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
axios配置
![axios配置.png](http://upload-images.jianshu.io/upload_images/6914113-a501a8a668ec62d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这几个地方对应好了就行

### 四、字体图标
1.Element-ui里面自带的图标在实际项目中肯定不够用，我是使用的阿里[iconfont](http://www.iconfont.cn/)字体图标库，注册一个帐号，创建一个项目添加你想要的字体图标，会生成一个cdn的css链接，然后在你的head 里面 link这个样式即可。当你增加新图标的时候更新这个地址就好了。
![iconfont.png](http://upload-images.jianshu.io/upload_images/6914113-6735d398365a6790.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 五、关于vuex的使用
并不是每个项目都必须使用vuex，根据项目需要来使用。对于数据流不复杂的项目，完全可以讲数据保存在本地，也能达到类似的效果也不影响项目需求，vuex我个人感觉使用很繁琐，根据你的项目来进行使用。
还是借用官方文档的术语：
虽然 Vuex 可以帮助我们管理共享状态，但也附带了更多的概念和框架。这需要对短期和长期效益进行权衡。
如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用够简单，您最好不要使用 Vuex。一个简单的 [global event bus](https://cn.vuejs.org/v2/guide/components.html#非父子组件通信) 就足够您所需了

## 开始搭建
### 1. Vue-cli脚手架生成项目
使用vue官方脚手架生成项目，生成的时候最好选择webpack模版，单元测试，根据你的需求是否安装。
### 2. 目录结构
```shell
├── build                      // 构建相关  
├── config                     // 配置相关
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 图片 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── filtres                // 全局filter
│   ├── mock                   // mock数据
│   ├── router                 // 路由
│   ├── store                  // 全局store管理
│   ├── styles                 // 全局样式
│   ├── utils                  // 全局公用方法
│   ├── views                  // view
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限控制 初始用户数据等
├── static                     // 第三方不打包资源
│   ├── img                    // 第三方不打包图片
│   └── theme                  // 主题包
├── .babelrc                   // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── .fjpublish.config.js       // 自动化发布服务器 配置
├── index.html                 // html模板
└── package.json               // package.json

```
### 3.webpack配置改造
#### 3.1 在实际开发过程中，往往有多个环境，开发环境、测试环境、生产环境等等。我们需要进行多环境配置。
##### 步骤1
在./config文件夹下新建一个sit.env.js 的文件
```javascritp
// 测试仿真环境
module.exports = {
  NODE_ENV: '"production"',
  BASE_API: '"http://120.55.169.121:8888/index"',
  CRM_PATH: '"http://120.55.169.121:8083"',        //  其他配置
};
```
dev.env.js 、prode.env.js 也可进行类似的配置，把各个环境请求的不同端口配置在这

后面在你的程序中，如果要使用这些变量可参考下例
```javascript
let baseUrl  = process.env.BASE_API;
let crmPath = process.env.CRM_PATH;
// 创建axios实例
const service = axios.create({
  baseURL: BASE_API,      // api的base_url
})
```


##### 步骤2
修改./config/index.js文件，将 sit.env.js 在 index.js 的 build 对象中引入：如图

![引入sit.env.js](http://upload-images.jianshu.io/upload_images/6914113-1eac13bbd8c679e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 步骤3
修改./build/build.js 文件，将 process.env.NODE_ENV = 'production' 注释或者删除，因为我们在后面需要动态配置NODE_ENV，此步骤如图

![./build/build.js ](http://upload-images.jianshu.io/upload_images/6914113-78dd9623a309044f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 步骤4
修改./build/webpack.prod.conf.js 文件 修改 evn,不BB直接上图

![./build/webpack.prod.conf.js](http://upload-images.jianshu.io/upload_images/6914113-b4ec9c1bd85c5516.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样webpack 的配置就修改完了。
在这里可以顺便修改一下这个文件下 UglifyJsPlugin，打包构建的时候可以去除console.log，debugger。配置如下。（此步骤和多环境配置无关）
```javascript
new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false,
        drop_debugger: true,     // 去除构建的 debugger
        drop_console: true        //  去除构建的 console
      },
      sourceMap: true
    }),

```
##### 步骤5
我们需要修改 package.json 的script 语句来增加命令启动我们新增的服务
```javascript
"scripts": {
    "dev": "node build/dev-server.js",
    "start": "node build/dev-server.js",
    "build:sit": "NODE_ENV=sit node build/build.js",
    "build:prod": NODE_ENV=production node build/build.js"
  },
```
然后启动 run run build:sit，可是这个时候报错了，NODE_ENV不被识别，这是由于 windows 不支持 NODE_ENV=sit 设置的方式。我们只需要安装一个 cross-env 的插件即可
```javascript
  yarn add cross-env -D   // 或者你使用npm 也可
  npm install cross-env -dev--save
```
接下来 继续修改一个 script 语句启动即可
```javascript
"scripts": {
    "dev": "node build/dev-server.js",
    "start": "node build/dev-server.js",
    "build:sit": "cross-env NODE_ENV=sit node build/build.js",
    "build:prod": "cross-env NODE_ENV=production node build/build.js"
  },
```

这样 你启动 npm run build:sit  将构建打包测试环境的代码 生成在 dist 文件目录下
启动 npm run build:prod  将构建打包生产环境的代码  生成在 dist 文件目录下

#### 3.2 结合 [fjpublish](https://github.com/zczhangchao51/fjpublish)  自动化发布到服务器。

##### 步骤1
安装 [fjpublish](https://github.com/zczhangchao51/fjpublish)
```javascript
npm install fjpublish -g
```
在项目根目录下建立一个 fjpublish.config.js 文件（为fjpublish配置文件）
```javascript
module.exports = {
  modules: [{
    name: '测试环境',
    env: 'sit',
    ssh: {
      host: '11.11.111.11',     // 服务器地址
      port: 22,                        // 端口
      user: 'root',                   // 用户
      userName: 'root',         // 用户名
      password: 'XXXX'        // 密码
    },
    buildCommand: 'build:sit',    // 构建命令  === npm run build:sit
    localPath: 'dist',                   // 构建后上传文件
    remotePath: '/test/xx',       // 服务端路径
    postCommands: ['chmod -R 777 /test/xx']   // 远程项目文件替换后执行的linux命令
  }，｛
    name: '其他环境',
    env: 'other',
    ....
｝]
}
```
同样的为了方便我们需要修改 package.json 的script mode
```javascript
"scripts": {
    "dev": "node build/dev-server.js",
    "start": "node build/dev-server.js",
    "build:sit": "cross-env NODE_ENV=sit node build/build.js",
    "build:prod": "cross-env NODE_ENV=production node build/build.js"
    "public:sit": "fjpublish env sit"
  },
```
运行命令 npm run  public:sit　确认后就会自动打包build:sit的代码，并且压缩后发布带你指定的服务器上，并且执行你的相应配置，如果需要多环境同时发布，只需要在fjpublish.config.js里面的modules里面增加一个对象进行相关配置即可。

> 至此 Vue-cli 项目的前端自动化已经配置好了