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

| 文件/文件夹 | 作用 |
|----------|
|app/|项目主目录，包含样式文件,模板文件,EmberJs文件，在开发过程中/构建后，项目JS文件将被合并至app.js文件|
|dist/|发布版本目录，根据Ember Appkit的设置，JS文件与css文件将被合并，压缩，打包至dist目录
|public/|在开发过程/构建后，该文件将被合并到项目根目录下,建议将图片放置在public/images目录下
|task/|本目录包含构建脚本文件
|tasks/options/|用来放置用户自定义的构建脚本
|tests/|放置Ember项目的测试文件
|tmp/|编译过程使用到的临时目录（可以无视它）
|vendor/|项目依赖的文件，包括Ember依赖的JS文件，和通过Bower下载的项目依赖
|.jshintrc|语法检查配置文件
|.travis.yml|github构建集成文件
|Gruntfile.js|构建脚本配置|
|bower.json| bower依赖配置文件|
|package.json| nodejs配置文件|









     
    




