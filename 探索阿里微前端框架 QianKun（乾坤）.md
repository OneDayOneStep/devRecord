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

  1. 安装 
     
     ``` Terminal
     npm init                   # 创建一个空的项目
     yarn add qiankun # 或者 npm i qiankun -S 
     ```
     
  2. 主应用注册微应用（根目录创建 index.js ）
  
     ```Javascript
     import { registerMicroApps, start } from "qiankun";
     
     registerMicroApps([
         {
             name: "app-react", // app name registered
             entry: "//localhost:3000",
             container: "#react",
             activeRule: "/react"
         },
         {
             name: "app-vue",
             entry: { 
                 scripts: [
                     "//localhost:4000/main.js"
                 ] 
             },
             container: "#vue",
             activeRule: "/vue"
         }
     ]);
     
     start();
     ```

  3. 创建两个微应用 React & Vue （对应脚手架创建项目）放至 apps 目录；

     通过 Vite 创建 app-vue 项目(esm真快)，create-react-app 创建 app-react 项目；

     此时项目结构如下：
  
     Root

     ├─ apps

     ​			├─ app-react
  
     ​			└─ app-vue
  
     ├─ node_modules
  
     ├─ index.js
  
     ├─ package.json
  
     └─ public-path.js
  
     
  
  4. 微应用（根目录创建public-path.js）
  
     

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

