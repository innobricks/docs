# EmbeddedRecordMixin
### 职能概述
* 解析内嵌数据
* 体现在`attrs`的配置上
* serialize
* deserialize

### 用法
用于扩充序列化器的功能,所以应该由序列化器继承
```javascript
App.PostSerializer = DS.ActiveModelSerializer.extend(DS.EmbeddedRecordsMixin,{
    attrs:{
        author:{embedded:"always"},
        comments:{serialize:"ids"}
    }
});
```
### attrs
```javascript
//可以配置的值
attrs:{
    firstName:"first_name",
    lastName:{key:"last_name"},
    author:{embedded:"always"},
    comments:{deserialize:"records",serialize:"records"}
}
```
### 可能值
serialize和deserialize可以配置的值如下所示
* id    
* ids
* records
* no

> serialize 为序列化,即发送到后端前进行数据序列化
> deserialize 为反序列化,即数据请求到前端后根据规则进行反序列化

```javascript
BelongsTo:{serialize:'id',deserialize:'id'}
HasMany:{serialize:'no',deserialize:'id'}
```
如果`serialize`配置为`id`,则意味着发送到后端会带这个`id`的值
```javascript
App.Person = DS.Model.extend({
    profile:DS.belongsT('profile')
});
App.Profile = DS.Model.extend({
    person:DS.belongsT('person')
});
App.PersonSerializer = DS.RESTSerializer.extend(DS.EmbeddedRecordMixin,{
    attrs:{
        profile:{
            deserialize:"records",
            serialize:"id"
        }
    }
});
//请求返回的数据
{
    "person":{
        "id",1
        "profile":{//deserialize 配置的结果,可以反序列化模型
            "id":"profile_1"
        }
    }
}
//保存更新到后端
{
    "person":{
        "id":1,
        "profile":1 //serialize 配置的结果,可以序列化id
    }
}
```
### 简写
```javascript
{
    embedded:"always"
}
//相当于
{
    deserialize:"records",
    serialize:"records"
}
```