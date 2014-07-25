###JSONSerializer 序列化器
* 序列化器基类
* 序列化和反序列化模型记录的值
* 规范化数据
* 数据格式解析和转换
* 序列化模型关系
取到数据后:extract+normalize
有数据更新需要保存:serialize
####为了获得最大性能,Ember-Data建议使用`RESTSerializer`
当数据从后端返回时,这是的动作叫`extract`,根据不同的情况会有如下的不同数据解析
* `extractFindAll`
* `extractFindQuery`
* `extractFindMany`
* `extractFindHasMany`
* 

####alias
```javascript
DS.Store#findAll--->extractFindAll:extractArray
DS.Store#findQuery--->extractFindQuery:extractArray
DS.Store#findMany--->extractFindMany:extractArray
DS.Store#findHasMany--->extractFinHasMany:extractArray
DS.Store#createRecord--->extractCreateRecord:extractSave
DS.Store#update--->extractUpdateRecord:extractSave
DS.Store#deleteRecord--->extractDeleteRecord:extractSave
DS.Store#find--->extractFind:extractSingle
DS.Store#findBelongsTo--->extractFindBelongsTo:extractSingle
DS.Model#save--->extractSave:extractSingle
```

####数据格式
在返回的数据中都可以添加一个`links`字段,用于动态指定属性的资源路径
#####one-to-one
#####Model
```javascript
App.Person = DS.Model.extend({
    firstName: attr(),
    lastName: attr(),
    verified: attr("boolean"),
    profile:DS.belongsTo("profile")
});
App.Profile=DS.Model.extend({
    license:attr(),
    person:DS.belongsTo("person")
});
```
######Data
```javascript
//Person
{
    "id": 1,
    "firstName": "jwy",
    "lastName": "Scoot",
    "profile": 1
}
//Profile
{
    "id": 1,
    "license": "copy@emberjs.com",
    "person": 1
}
//当在`person`模型中使用`profile` 属性时,在一堆一关系下,Ember-Data会自动向后台请求profile的数据
//请求之前,本地是没有`profile`数据的
```
belongsTo async:true
```javascript
App.Person = DS.Model.extend({
    firstName: attr(),
    lastName: attr(),
    verified: attr("boolean"),
    profile:DS.belongsTo("profile",{async:true})
});
```

#####one-to-many
#####Model
```javascript
App.Person = DS.Model.extend({
    firstName: attr(),
    lastName: attr(),
    verified: attr("boolean"),
    profile:DS.hasMany("profile")
});
App.Profile=DS.Model.extend({
    license:attr(),
    person:DS.belongsTo("person")
});
```
######Data
```javascript
//Person
{
    "id": 1,
    "firstName": "jwy",
    "lastName": "Scoot",
    "profile": [1,2,3]
}
//Profile
{
    "id": 1,
    "license": "copy@emberjs.com",
    "person": 1
}
```
当在`person`模型中使用`profile` 属性时,在一堆多关系下,Ember-Data不会自动向后台请求`profile`的数据
会默认认为本地已有`profile`的数据
如果要在一堆多关系下,`profile`在需要时获取,需要设置为异步,如下:
```javascript
profile:DS.hasMany("profile",{async:true})
```


####属性钩子
#####primaryKey
用户可自定以主键,Ember-Data会统一处理如下:
```javascript
var primaryKey = get(this, 'primaryKey');
    if (primaryKey === 'id') { return; }
    hash.id = hash[primaryKey];
    delete hash[primaryKey];
    //统一存放在hash.id上
```

#####attrs
在normalize时,normalizeUsingDeclaredMapping,这个步骤会用到attrs
```javascript
JSONSerializer.extend({
    attrs:{
        firstName:"first_name"
    },
    //or
    attrs:{
        firstName:{key:"first_name"}
    }

});
```

####方法钩子