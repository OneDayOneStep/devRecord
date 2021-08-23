### 探索阿里微前端框架 QianKun（乾坤）
---

- [x] 微前端
- [x] SPA
- [x] QianKun@2.4.x

+ 最近经常在网上会看见 `微前端`  的字眼，今天发现阿里也有一套开源框架乾坤。

  > 主页：https://qiankun.umijs.org/zh
  >
  > github：https://github.com/umijs/qiankun

+ 官方特性说明：

  + 📦 **基于 [single-spa](https://github.com/CanopyTax/single-spa)** 封装，提供了更加开箱即用的 API。
  +  📱  **技术栈无关**，任意技术栈的应用均可 使用/接入，不论是 React/Vue/Angular/JQuery 还是其他等框架。
  + 💪 **HTML Entry 接入方式**，让你接入微应用像使用 iframe 一样简单。
  + 🎯 **样式隔离**，确保微应用之间样式互相不干扰。
  + 🧳  **JS 沙箱**，确保微应用之间 全局变量/事件 不冲突。
  + ⚡️ **资源预加载**，在浏览器空闲时间预加载未打开的微应用资源，加速微应用打开速度。
  + 🔌 **umi 插件**，提供了 [@umijs/plugin-qiankun](https://github.com/umijs/plugins/tree/master/packages/plugin-qiankun) 供 umi 应用一键切换成微前端架构系统。

  

+ 官方对于放弃 iframe 的解释：[Why Not Iframe](https://www.yuque.com/kuitos/gky7yw/gesexv)


+ 快速上手教程

  ​		该文章完整的项目 DEMO 地址：

  ​		https://github.com/OneDayOneStep/AliQK

  1. 安装 

  ``` Terminal
  npm init                   # 创建一个空的项目
  yarn add qiankun
  ```

  2. 根目录创建 `index.html` 和 `webpack.config.js` 来运行QianKun

  `index.html`：

    ```HTML
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>QK</title>
        <style>
            html, body {
                margin: 0;
            }
            .microAppList {
                box-shadow: 0 0 4px #888;
                margin: 0;
                padding: 10px;
                font-family: Arial, serif;
                font-size: 0;
            }
            .microAppList li {
                display: inline-block;
                line-height: 40px;
                padding: 0 10px;
                background: #555;
                color: #fff;
                cursor: pointer;
                margin-right: 10px;
                font-size: 13px;
            }
            .microAppList .active {
                background: #66cccc;
                color: #fff;
            }
        </style>
    </head>
  
    <body>
    <div class="mainApp">
        <!--微应用菜单-->
        <ul class="microAppList"></ul>
        <!--微应用容器-->
        <main id="microAppContainer"></main>
    </div>
  
    <script>
        // 微应用列表
        const apps = [{
            name: "React",         // 只是名称
            path: "/app-react", // 推送到地址栏的路径 跟 index.js 的 activeRule 对应
            effective: true          // 忽略
        }, {
            name: "Vue for cli",
            path: "/app-vue-cli",
            effective: true
        }, {
            name: "Vue for vite",
            path: "/app-vue-vite",
            effective: false
        }];
  
        // 加载微应用
        function push(t, app) {
            if (!app.effective) {
                alert("QianKun@2.4.x 暂不支持 module script");
                return;
            }
            Array.from(document.getElementsByClassName("active")).forEach(e =>{
                e.classList.remove("active");
            });
            t.classList.add("active");
            // pushState 改变浏览器的地址栏并且会生成前后记录 但不会造成页面跳转/刷新
            history.pushState(null, "", app.path);
        }
  
        const microAppList = document.querySelector(".microAppList");
        apps.forEach(e => {
            const li = document.createElement("li");
            li.innerText = e.name;
            li.onclick = function (ev) {
                push(ev.target, e);
            }
            if (location.pathname.includes(e.path)) {
                li.classList.add("active");
            }
            microAppList.appendChild(li);
        });
    </script>
    </body>
    </html>
    ```

    `webpack.config.js`：

    ```Javascript
    const HtmlWebpackPlugin = require("html-webpack-plugin");
    module.exports = {
        mode: "development",
        entry: "./index.js",
        devtool: "source-map",
        output: {
            publicPath: '/',
        },
        devServer: {
            open: true,
            port: '5500',
            historyApiFallback: true // 忽略地址栏 pathname 都返回 index.html 解决非根路径刷新 404 的问题
        },
        plugins: [
            new HtmlWebpackPlugin({
                filename: 'index.html',
                template: "./index.html"
            })
        ]
    };
    ```

    配置一下主应用 `package.json` 添加一条 script：

    ```JSON
    "start": "webpack-dev-server --open --config  webpack.config.js"
    ```

  

  3. 创建两个微应用 React & Vue & ~~Vite~~ 放至 apps(自建) 目录；

    create-react-app 创建 app-react 项目；

    @vue/cli 创建 vue3 项目；

    ~~Vite 创建 vue3 项目(esm真快)~~ ；

    > Vite 被划掉的原因是通过 Vite 创建的 Vue3 项目报出：
    >
    > <div style="color: red">Is it possible to load a sub app using `type="module"` script tag when the master app doesn't use `type="module"` script tag</div>
    >
    > 通过查阅 issues 发现当前版本(2.4.x)不支持 module 引入方式 ，后面应该会完善。

  4. 主应用注册微应用（根目录创建 `index.js` ）

    ```Javascript
    import { registerMicroApps, start } from "qiankun";
  
    registerMicroApps([
        {
            name: 'project-react',                       // 项目名称 (异常报错时有用)
            entry: '//localhost:5505',                 // 微应用运行的地址与端口
            container: '#microAppContainer', // 微应用挂载的 DOM 节点
            activeRule: '/app-react',                  // 重要, 根据 location.pathname 进行匹配
        },
        {
            name: "project-vue-cli",
            entry: "//localhost:5510",
            container: "#microAppContainer",
            activeRule: "/app-vue-cli"
        },
        {
            name: "project-vue-vite",
            entry: "//localhost:5515",
            container: "#microAppContainer",
            activeRule: "/app-vue-vite"
        }
    ]);
  
    console.log("QK Running");
  
    start();   // 运行
    ```

    此时项目结构如下：
  
    Root
  
    ├─ apps
  
    │	├─ app-react
  
    │	├─ app-vue-cli
  
    │	└─ ~~app-vue-vite~~
  
    ├─ index.js
  
    └─ package.json
  
  
  
  ​	主应用(乾坤)已经可以跑起来了，但只是个空壳容器，接下来配置微应用
  
  
  
  5. 微应用部分
  
     在各个 `apps/app-*/src` 中创建 `public-path.js` 内容一致为：
  
     ```Javascript
     if (window.__POWERED_BY_QIANKUN__) {
         __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
     }
     ```
  
     React 项目：
  
     根目录创建 `.rescriptsrc.js` ：
  
     ```Javascript
     const { name } = require('./package');
     
     module.exports = {
         // umd 打包
         webpack: config => {
             config.output.library = `${name}-[name]`;
             config.output.libraryTarget = 'umd';
             config.output.jsonpFunction = `webpackJsonp_${name}`;
             config.output.globalObject = 'window';
     
             return config;
         },
     
         devServer: e => {
             const config = e;
     
             config.headers = {
                 // 允许跨域 否则 Qiankun 加载该微应用时可能报跨域问题
                 'Access-Control-Allow-Origin': '*',
             };
             config.historyApiFallback = true;
             config.hot = false;
             config.watchContentBase = false;
             config.liveReload = false;
     
             return config;
         },
     };
     ```
  
     修改 React 项目的 package.json - script - start
  
     改变端口并使用 rescripts 配置启动
  
     ```JSON
      "start": "set PORT=5505 && rescripts start"
     ```
  
     修改 src/index.js：
  
     ```React
     import './public-path';
     import React from 'react';
     import ReactDOM from 'react-dom';
     import App from './App.js';
     
     function render(props) {
         const { container } = props;
         ReactDOM.render(
             <React.StrictMode>
                 <App/>
             </React.StrictMode>,
             // 根据是否传入的 props 是否为空寻找容器
             container ? container.querySelector('#root') : document.querySelector('#root'));
     }
     
     if (!window.__POWERED_BY_QIANKUN__) {
         // 独立运行
         render({});
     }
     
     export async function bootstrap() {
         // 暂时没发现作用
         console.log('app-react bootstraped');
     }
     
     export async function mount(props) {
         // 供乾坤调用进行应用装载(就是load项目进去)
         render(props);
         console.log('app-react mount', props);
     }
     
     export async function unmount(props) {
         // 供乾坤调用进行应用卸载(销毁实例)
         const { container } = props;
         ReactDOM.unmountComponentAtNode(
             container ? container.querySelector('#root') : document.querySelector('#root')
         );
         console.log('app-react unMount');
     }
     ```
  
     然后自行配置对应的 Router，详细例子见 [DEMO 地址](https://github.com/OneDayOneStep/AliQK)
  
     其中需要修改的是路由的根路径 (这里我用的是 `react-router-dom` )
  
     ```React
     // 是否独立运行决定根路径
     <BrowserRouter basename={ window.__POWERED_BY_QIANKUN__ ? '/app-react' : '/' }>
     	...
     </BrowserRouter>
     ```
  
     
  
     
  
     Vue3 项目：
  
     根目录创建 `vue.config.js` ：
  
     ```Javascript
     const path = require('path');
     const { name } = require('./package');
     
     function resolve(dir) {
         return path.join(__dirname, dir);
     }
     module.exports = {
         devServer: {
             headers: {
                 // 允许跨域 否则 Qiankun 加载该微应用时可能报跨域问题
                 'Access-Control-Allow-Origin': '*'
             }
         },
         configureWebpack: {
             resolve: {
                 alias: {
                     '@': resolve('src'),
                 },
             },
             output: {
                 // 把子应用打包成 umd 库格式
                 library: `${name}-[name]`,
                 libraryTarget: 'umd',
                 jsonpFunction: `webpackJsonp_${name}`,
             }
         }
     };
     ```
  
     修改 Vue3 项目的 package.json - script - serve
  
     改变端口
  
     ```JSON
       "serve": "set PORT=5510 && vue-cli-service serve"
     ```
  
     修改 main.js：
  
     ```Javascript
     import './public-path';
     import { createApp } from 'vue'
     import App from './App.vue'
     import routerGenerator from "./router";
     
     let app, router;
     function render(props = {}) {
     
         const { container } = props; // 从乾坤接收参数
     
         app = createApp(App); // 创建 Vue 实例
     
         // 是否独立运行决定根路径
         router = routerGenerator(
             window.__POWERED_BY_QIANKUN__ ? '/app-vue-cli' : '/'
         );
     
         app.use(router);
     
         // 根据是否传入的 props 是否为空寻找容器
         const mountDOM = container ? container.querySelector('#app') : '#app';
     
         app.mount(mountDOM);
     }
     
     if (!window.__POWERED_BY_QIANKUN__) {
         // 独立运行
         render();
     }
     
     export async function bootstrap() {
         // 暂时没发现作用
         console.log('app-vue-cli bootstraped');
     }
     
     export async function mount(props) {
         // 供乾坤调用进行应用装载(就是load项目进去)
         render(props);
         console.log('app-vue-cli mount', props);
     }
     
     export async function unmount() {
         // 供乾坤调用进行应用卸载(销毁实例)
         app.unmount();
         app = null;
         router = null;
         console.log('app-vue-cli unMount');
     }
     ```
  
     router.js：
  
     ```Javascript
     import { createRouter, createWebHistory } from "vue-router";
     export default (base = "/") => {
         return createRouter({
             history: createWebHistory(base),
             routes: [{
                 path: "/page_A",
                 component: () => import('@/components/page_A')
             }, {
                 path: "/page_B",
                 component: () => import('@/components/page_B')
             }]
         });
     }
     ```
  
  6. 运行项目
  
     到这里就已经完成了基本的配置 可以把主应用和微应用都跑起来看看成果了
  
     总体的运行情况图：
  
     
  
     
  
+ 中途碰到了图片找不到的情况，比如 Vue3 的地址是 localhost:2000 乾坤的地址是 localhost:1000

  然后 Vue3 项目的 img 路径为 "img/xxx.png"，就会跑到乾坤的地址上变成 "localhost:1000/img/xxx.png"

  最后是因为 `import './public-path'` 的位置不对的问题，放到第一行就正常了

  原理是在解析到对应的图片时 webpack 的 publicPath 是 "/"

  运行完 `public-path.js` 后就变成了对应微应用的地址了。



基本运行流程：

![](D:\Project\devRecord\QK.png)





附：官方DEMO指南

```Terminal
git clone https://github.com/umijs/qiankun.git   // 拉取项目
cd qiankun
yarn install                       // 安装乾坤依赖
yarn examples:install   // 安装示例依赖
yarn examples:start      // 启动查看 DEMO
```

<div align=right>✒️Author: Lee</div>
<div align=right>⏰ RecordTime: 2021-08-16</div>

