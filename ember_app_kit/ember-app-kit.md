build-tools
===========
###Ember App kit(EAK) 可以简化Ember项目的开发，构建和测试工作。
Ember Appkit以下特性

- 资源编译
- ES模块化
- 测试
- 依赖管理
- 命名约定
- 代理/数据模拟

------------------------

###资源编译
Ember Appkit支持对以下类型的资源进行预编译

- CoffeeScript
- Emblem
- LESS
- SASS
- Compass
- Stylus

Ember Appkit还支持CSS压缩，JS文件压缩，图片压缩等特性

---------------------------
###ES6模块化
Ember App kit使用ES6 Module Transipiler将ES6 JS语法转换为AMD(RequireJs)模块，使用模块转换器，你今天就可以用上明天的语法。

---------------------------
###测试
Ember Appkit 默认使用Qunit，Ember Testing包，Testem工具进行单元测试

----------------------------
###依赖管理
Ember App kit 使用[Bower](http://bower.io/)进行包依赖管理，让包依赖，包更新变得更简单高效

----------------------------
###命名约定
Ember Appkit 使用ES6 Module Transipiler将ES6模块转换为AMD模块，根据文件名称定义AMD模块名称

      appkit
        |
        +--APP
        |   |
        |   +--Controllers
            |       |
            |       +--Posts
                         |
                         +--index.js
                               

对于一个在Root/App/Controllers/Posts/index.js的ES6文件，

```javascript    
export default Ember.ArrayController.extend({
    model:function(){
        return [1,2,3,4,5,6,7];
    }
});
```

将被打包为

```javascript
define("appkit/controllers/posts/index", 
  ["exports"],
  function(__exports__) {
    "use strict";
    __exports__["default"] = Ember.ArrayController.extend({
      model:function(){
        return [1,2,3,4,5,6,7];
      }
    });
  });
  
```

---------------------------------------

###代理/数据模拟
Ember Appkit 允许使用数据Mock或Proxy的方式进行数据测试

#####数据模拟
如果希望使用该功能，可以在 api-stub/router.js文件下添加数据模拟,并在项目的package.js文件中制定APIMEthod属性为 "stub"
在下一次grunt server 中Ember Appkit 将开启一个express服务器进行数据模拟工作

```javascript
module.exports = function(server) {
  
  /**
  *如果浏览器发起一个目的地为api/posts的get请求，服务端将返回 hello mocking world.
  */
  server.namespace("/api", function() {
    server.get('/posts', function(req, res) {
      res.send({ post: { body: "hello mocking world" } });
    });
  });
};

```

#####代理
Ember appkit同样允许使用代理模式进行数据代理，如果需要使用代理模式，请在package.js文件中添加如下属性
```javascript
  APIMethod: "proxy",
  proxyURL: "http://apiserver.dev:3000",
  proxyPath: "/api"
```
对于浏览器一个 api/posts的请求，Ember app kit 将会把请求转发至http://apiserver.dev:3000/api/posts

---------------------------------------


###项目结构说明

<table>
<thead>
<tr>
  <th>文件/文件夹</th>
  <th>作用</th>
</tr>
</thead>
<tbody><tr>
  <td>app/</td>
  <td>项目主目录，包含样式文件,模板文件,EmberJs文件，在开发过程中/构建后，项目JS文件将被合并至app.js文件</td>
</tr>
<tr>
  <td>dist/</td>
  <td>发布版本目录，根据Ember Appkit的设置，JS文件与css文件将被合并，压缩，打包至dist目录</td>
</tr>
<tr>
  <td>public/</td>
  <td>在开发过程/构建后，该文件将被合并到项目根目录下,建议将图片放置在public/images目录下</td>
</tr>
<tr>
  <td>task/</td>
  <td>本目录包含构建脚本文件</td>
</tr>
<tr>
  <td>tasks/options/</td>
  <td>用来放置用户自定义的构建脚本</td>
</tr>
<tr>
  <td>tests/</td>
  <td>放置Ember项目的测试文件</td>
</tr>
<tr>
  <td>tmp/</td>
  <td>编译过程使用到的临时目录（可以无视它）</td>
</tr>
<tr>
  <td>vendor/</td>
  <td>项目依赖的文件，包括Ember依赖的JS文件，和通过Bower下载的项目依赖</td>
</tr>
<tr>
  <td>.jshintrc</td>
  <td>语法检查配置文件</td>
</tr>
<tr>
  <td>.travis.yml</td>
  <td>github构建集成文件</td>
</tr>
<tr>
  <td>Gruntfile.js</td>
  <td>构建脚本配置</td>
</tr>
<tr>
  <td>bower.json</td>
  <td>bower依赖配置文件</td>
</tr>
<tr>
  <td>package.json</td>
  <td>nodejs配置文件</td>
</tr>
</tbody></table>

###app结构说明

<table>
<thead>
<tr>
  <th>文件/文件夹</th>
  <th>作用</th>
</tr>
</thead>
<tbody><tr>
  <td>app/app.js</td>
  <td>应用入口，该文件是最先执行的模块</td>
</tr>
<tr>
  <td>app/index.html</td>
  <td>应用界面，其中包含js,css依赖</td>
</tr>
<tr>
  <td>app/router.js</td>
  <td>Ember Router配置文件</td>
</tr>
<tr>
  <td>app/styles/</td>
  <td>项目样式文件</td>
</tr>
<tr>
  <td>app/templates/</td>
  <td>应用模板文件</td>
</tr>
<tr>
  <td>app/controllers/, app/models/, …</td>
  <td>应用相关的Controller,Model…</td>
</tr>
</tbody></table>

---------------------------------------

###Grunt文件说明

Ember Appkit基于[Grunt](http://gruntjs.com/)构建，如果需要了解Grunt的每个任务，可以查看Grunt.js文件，如果需要添加自定义任务，可以将自定义任务放置在tasks/options目录下
####Grunt命令说明
+ grunt 运行grunt default任务
+ grunt server编译源原件，并启动一个Http服务,同时监听文件的变化，做到浏览器热刷新。用于开发环境。
+ grunt test:server与grunt server一致，不过在进入应用后会同时执行测试文件
+ grunt dist 构建可发布版本，项目打包至/dist目录
+ grunt server:dist与grunt dist一致，不过在构建前执行server任务


-----------------------------------------------
更多详情可查看[Ember App Kit Getting Start](http://iamstef.net/ember-app-kit/)



     
    




