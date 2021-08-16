### VueRouter 区分 403 和 404
---

- [x] Vue@2.5.x
- [x] VueRouter@3.x
- [x] NodeJs@14.x

+ 前端路由除去一些如登录 / 注册等基本路由外，其他路由都是在登录后由后台返回；

  也就是当前登录用户拥有权限的页面；

  但如果我直接从地址栏跳转到：http://localhost:8080/unknownPage；

  这时候根据路由表匹配将会匹配到最后的规则：


```Javascript
{
        path: "*",
        redirect: "/404"
}
```

  上面这种情况是没有什么问题，的确没有 `unknownPage` 这个页面( 我项目中没有🙄 )。



+ 如果我是跳转到：http://localhost:8080/adminManage 也会跳转到 404 页面；

  但是我项目中是有 `adminManage` 这个页面的；

  所以不应该是 404 页面，而是因为没有权限跳转到 403 页面。
  
  

+ （我的做法） 解决方案：

1. 将路由匹配表的最后一项指向 `404` 改为指向 `statusPage`：

   ```Vue
   <template>
       <div>
           <statusPage_403 v-if="type === '403'" />
           <statusPage_404 v-else-if="type === '404'" />
           <!-- 这里可以多加一个错误页 -->
       </div>
   </template>
   
   <script>
   import viewsPages from "@/viewsPages"; // 存放了 views 下所有的 vue 页面路径
   import statusPage_403 from "./403.vue";
   import statusPage_404 from "./404.vue";
   export default {
       name: "statusPage",
       components: {
           statusPage_403,
           statusPage_404
       },
       data: () => {
           return {
               type: ""
           };
       },
       // 进入到这个组件说明当前路由表是找不到这个页面的
       // 但不知道是因为没有权限还是因为真的没有这个页面
       beforeRouteEnter(to, from, next) {
           next(vm => {
               if (viewsPages.includes(to.fullPath)) { // 判断项目中是否存在改页面
                   vm.type = "403"; // 存在但没路由说明没权限
               } else {
                   vm.type = "404"; // 真系冇
               }
           });
       }
   };
   </script>
   ```

   

2. 获取全部 views 下的页面数据，在项目根目录创建 `fetchPages.js` 内容：

    ```Javascript
    /**
     * @description 获取项目下 views 中所有的 vue 文件生成 js 内里由数组导出
     */
    const fs = require("fs"); // node 文件模块
    const pages = []; // 存储页面数据
    function files(path, prePath) {
        fs.readdirSync(path, {
            withFileTypes: true // 是否获取文件类型
        }).forEach((file) => {
            let fileName = file.name;
            if (file.isDirectory()) { // 如果是目录类型
                // 继续往下查找
                files(
                    path + "/" + fileName,      // 继续遍历该目录
                    prePath + fileName + "/" // 将路径向下传递
                );
            } else if (new RegExp(/\.vue$/).test(fileName)) { // 是否是 vue 文件
                pages.push(prePath + fileName);
            }
        })
    }
    files("./src/views", "/");
    
    // 将获取到的 pages 数据写入文件放到项目中
    fs.writeFileSync(
        "./src/viewsPages.js",
        "export default " // 便于 import
            .concat(JSON.stringify(pages))
            .replace(/\.vue/g, "") // 去除后缀名
            .replace(/",/g, '",\n')); // 略微格式化代码 - 非必要
    
    console.log("fetchPages done!"); // ide log 提示
    ```
    
    执行完 `fetchPages.js` 后会得到一份 views 所有 vue 文件的路径数据，但还是不稳妥不方便。
    
    
    
3. 打开 package.json，修改一下 `npm` 脚本

    ```JSON
    // before
    "scripts": {
      "serve": "vue-cli-service serve",
      "build": "vue-cli-service build",
      "lint": "vue-cli-service lint"
    }
    
    // after
    "scripts": {
      "serve": "vue-cli-service serve",
      "build": "node fetchPages .js && npm run build-project", // 在每次打包前先执行一次 fetchPages
      "build-project": "vue-cli-service build",
      "lint": "vue-cli-service lint"
    }
    ```

4. 这也只是其中一个办法，也可以按照这个思路做更多不同的需求，如：不通过 `statusPage` 判断，而是在路由拦截器进行判断后跳转到对应的状态页。




<div align=right>✒️Author: Lee</div>
<div align=right>⏰ RecordTime: 2021-08-13</div>
