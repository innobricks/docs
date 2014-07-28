# Adapter
是一个抽象类,必须至少实现如下方法
* `find()`
* `createRecord()`
* `updateRecord()`
* `deleteRecord()`
* `findAll()`
* `findQuery()`

### 职能概述
* 从`store`接口请求,并发起实际的请求动作
* 基类,定义适配器子类必须实现的方法

### 方法钩子
* generateIdForRecord
```javascript
//用于在客户端创建`record`时,生成一个`id`
generateIdForRecord: function(store, record) {
  var uuid = App.generateUUIDWithStatisticallyLowOddsOfCollision();
  return uuid;
}
```