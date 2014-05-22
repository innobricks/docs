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










     
    




