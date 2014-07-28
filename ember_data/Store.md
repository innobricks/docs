# Store
### 职能概述
* `store`包含了所有从`server`回来的模型
* 创建`Model`实例
* 调度`adapter`,以便与后端进行通讯
* 提供各种api接口,以及管理所有模型记录

### Store内部要点
#### typeMaps
该属性为`Store`内部的类型映射,一个同一片`type`对应一个`typeMap`,格式如下
```javascript
var guid = guidFor[type]
typeMaps:{
    [guid]:{
        idToRecord:{
            [id]:[record]
        },
        records:[],
        metadata:{},
        type:type
    }
}
```
#### 默认配置项
* recordArrayManager: 记录管理器,默认创建
* adapter:-rest:默认值为`-rest`

#### adapter查找规则
* adapter:type.typeKey
    * adapter:application
        * defaultAdapter
        
adapterFor 私有,不可直接调用
```javascript
adapter = container.lookup('adapter:' + type.typeKey) || container.lookup('adapter:application');
return adapter || get(this, 'defaultAdapter');
```
defaultAdapter 私有,不可直接调用
```javascript
//adapter是上面`adapter`属性的值:`-rest`
//该属性为默认适配器,查找规则如下代码所示
this.container.lookup('adapter:' + adapter) || this.container.lookup('adapter:application') || this.container.lookup('adapter:-rest');
```

#### serializer查找规则
* serializer:type.typeKey
    * serializer:application
        * serializer:defaultSerializer
            * serializer:-default 
            
### API
* serialize
* createRecord
* deleteRecord
* unloadRecord
```javascript
store.find('post', 1).then(function(post) {
  store.unloadRecord(post);
});
```
* find 查询数据
```javascript
store.find('person', 1);
```
```javascript
//查找所有findAll
store.find('person');
```

```javascript
//查询findQuery
store.find('person', { page: 1 });
```
* getById 通过类型和id查找对应的`record`
* hasRecordForId 查询对应id的模型是否已经加载过
* all 与findAll不同,返回本地已经加载的所有指定类型的记录
* unloadAll
* filter
* recordIsLoaded
* metadataFor 返回指定模型的`metadata`
* metaForType 设置指定类型的`metadata`
* modelFor 根据字符串返回指定的模型类型
* push 将数据根据对应的`type`放入`store`
* pushPayload
```javascript
App.ApplicationSerializer = DS.ActiveModelSerializer;

var pushData = {
  posts: [
    {id: 1, post_title: "Great post", comment_ids: [2]}
  ],
  comments: [
    {id: 2, comment_body: "Insightful comment"}
  ]
}

store.pushPayload(pushData);
```
### 说明
Store用于查找数据的api,如`find`,`findAll`等返回的对象全部都是Promise对象
* 单个:PromiseObject
* 多个:PromiseArray