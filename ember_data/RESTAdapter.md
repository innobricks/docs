# RESTAdapter
继承自[Adapter](https://github.com/innobricks/docs/tree/master/ember_data/Adapter.md)
* 实现了`Adapter`的所有抽象方法
* 用于处理`REST`方式的数据请求
* 默认的序列化器为`RESTSerializer`

### url拼接规则
* host
* namespace
* pluralize(camelize(type))
* id

根据`/`合并字符串

### 属性钩子
* defaultSerializer:'-rest'
适配器默认的序列化器,具体序列化器查找规则详见[Store](https://github.com/innobricks/docs/tree/master/ember_data/Store.md)
* namespace
* host
* headers
```javascript
App.ApplicationAdapter = DS.RESTAdapter.extend({
  headers: {
    "API_KEY": "secret key",
    "ANOTHER_HEADER": "Some header value"
  }
});
```
```javascript
//作为计算属性
App.ApplicationAdapter = DS.RESTAdapter.extend({
headers: function() {
  return {
    "API_KEY": this.get("session.authToken"),
    "ANOTHER_HEADER": "Some header value"
  };
}.property("session.authToken")
});
```
```javascript
//计算属性,不缓存,动态变化和获取
App.ApplicationAdapter = DS.RESTAdapter.extend({
  headers: function() {
    return {
      "API_KEY": Ember.get(document.cookie.match(/apiKey\=([^;]*)/), "1"),
      "ANOTHER_HEADER": "Some header value"
    };
  }.property().volatile();
});
```
* coalesceFindRequests:false 合并请求
```javascript
post: {
    id:1,
    comments: [1,2]
}
```

```
默认会发出如下请求
GET /comments/1
GET /comments/2
```
```
设为true后
GET /comments?ids[]=1&ids[]=2
```
        
### 方法钩子
1.pathForType 返回模型类型的字符串路径信息
```javascript
//camelize+puralize
App.FamousPerson-->famousPersons
```
```javascript
//重写pathForType
App.ApplicationAdapter = DS.RESTAdapter.extend({
  pathForType: function(type) {
    var decamelized = Ember.String.decamelize(type);
    return Ember.String.pluralize(decamelized);
  }
});
```

### API
* buildURL
* createRecord
* updateRecord
* deleteRecord
* findHasMany
* findMany
* find