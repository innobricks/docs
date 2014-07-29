# Model
继承自 Ember.Object

### 职能概述
* 定义了所有模型的基础
* 定义了状态树,清晰展示模型的所有可能状态
* 可以定义与其他模型之间的关系,如`belongsTo`和`hasMany`

### 状态树
```text
* root
  * deleted
    * saved
    * uncommitted
    * inFlight
  * empty
  * loaded
    * created
      * uncommitted
      * inFlight
    * saved
    * updated
      * uncommitted
      * inFlight
  * loading
```
`currentState`用来显式跟踪当前模型的状态
`currentState.stateName`可以得到当前状态名称
```javascript
record.get('currentState');
record.get('currentState.stateName');//root.loaded.created.uncommitted
```
#### 状态说明
```text
isEmpty     模型的第一状态,在模型`created`完成之后的状态
isLoading   `record`正在请求数据中
isLoaded    模型加载完成后所处的状态,数据已经请求回来
isDirty     `record`被修改后,处于脏数据状态
isSaving    正在保存中,请求已经发起,还没返回来
isDeleted   `record`b已经被删除
isNew       在`client`新建完成后,所出的状态,此时`adapter`还没报告已经保存完成,由`isEmpty`转移过来的
isValid     `record` 是合法的
```
#### record前世今生
```text
//store.find
一个模型刚开始创建时,处于`root.empty`;{isEmpty:true}
此时调用record.loadingData(),状态变为`root.loading`;{isLoading:true}
跟着发起请求,请求完成回来之后,数据设置完成后,转变为`root.loaded.saved`;{isLoaded:true}
此时,从`store`发起请求,到数据响应回来之后,状态转变已经完成,
这时,用户如果对模型发起修改操作,如`record.set('firstName','newName')`
状态转变为`root.loaded.updated.uncommitted`;({isDirty:true}
模型做了修改,需要持久化到后端,此时调用`record.save()`,状态转变为`root.loaded.updated.inFlight`
这时的状态为正在保存中,{isSaving:true}
```
### 属性定义
通过`DS.attr`方法,可以传入不同的数据格式,默认为`string`;
不同数据格式详情可以参见[Transform](https://github.com/innobricks/docs/tree/master/ember_data/Transform.md)
* boolean
* date
* number
* string
```javascript
var attr = DS.attr;
App.Person = DS.Model.extend({
    firstName:attr(),
    age:attr("number"),
    birthday:attr("date"),
    verified:attr("boolean")
});
```
### 关系定义
模型的关系定义可以建立起当前模型与其他模型的关系,主要有
* 一对一 One-to-One
* 一对多 One-to-Many
* 多对多 Many-to-Many

#### 一对一 One-to-One
```javascript
var attr = DS.attr,
    belongsTo = DS.belongsTo,
    hasMany = DS.hasMany
;
App.Person = DS.Model.extend({
    firstName:attr(),
    age:attr("number"),
    birthday:attr("date"),
    verified:attr("boolean"),
    profile:belongsTo('profile')
});

App.Profile = DS.Model.extend({
    license:attr(),
    person:belongsTo("person")
});
```
#### 一对多 One-to-Many
```javascript
var attr = DS.attr,
    belongsTo = DS.belongsTo,
    hasMany = DS.hasMany
;
App.Post = DS.Model.extend({
  content: DS.attr('string'),
  comments: DS.hasMany('comment',)
});

App.Comment = DS.Model.extend({
  message: DS.attr('string'),
  post: DS.belongsTo('post')
});
```
#### 多对多 Many-to-Many
```javascript
var attr = DS.attr,
    belongsTo = DS.belongsTo,
    hasMany = DS.hasMany
;
App.Author = DS.Model.extend({
  name:  DS.attr('string'),
  posts: DS.hasMany('post')
});

App.Post = DS.Model.extend({
  content: DS.attr('string'),
  authors: DS.hasMany('author')
});
```
#### 模型关系定义中可用的配置项
* async 异步
* polymorphic 多态
* defaultValue 默认值
* inverse 显式反转
##### async
`boolean`值,显式指定为异步关系,异步,意味着数据异步获取
```javascript
;
App.Post = DS.Model.extend({
  content: DS.attr('string'),
  //如果这里没有指定异步,则Ember-Data默认comments数据会在本地获取,而不会发起请求
  comments: DS.hasMany('comment',{async:true})
});

App.Comment = DS.Model.extend({
  message: DS.attr('string'),
  post: DS.belongsTo('post')
});
```
##### polymorphic
假设有一个订单系统,有供应商,供应商可以是供应不同货品的
```javascript
App.Provider = DS.Model.extend({
    link:   DS.attr('string'),
    admin:  DS.belongsTo('user',          {async: true}),
    orders: DS.hasMany('providerOrder', {async: true,polymorphic:true}),
    items:  DS.hasMany('items',           {async: true})
});
App.ProviderOrder = DS.Model.extend({
    // One of ['processed', 'in_delivery', 'delivered']
    status:   DS.attr('string'),	
    provider: DS.belongsTo('provider', {async: true,polymorphic:true}),
    order:    DS.belongsTo('order',    {async: true}),
    items:    DS.hasMany('item',       {async: true})
});
App.Shop = App.Provider.extend({
  status: DS.attr('string')
});

App.BookStore = App.Provider.extend({
  name: DS.attr('string')
});
GET /providerOrders
{
  "providerOrders": [{
    "status": "in_delivery",
    "provider": 1,
    "providerType": "shop"
  }]
}
//在数据当中指定providerType,可以动态指定是什么类型的provider
```
##### defaultValue
`boolean`值,显式指定为异步关系,异步,意味着数据异步获取
```javascript
App.Person = DS.Model.extend({
    verified:attr("boolean",{defaultValue:true}),
    isAdmin:attr("boolean",{defaultValue:function(){
        return !this.get('verified');
    }})
});
```
##### inverse
当一个模型定义了对同一个模型的多种关系时,需要显式指定,才能够在模型改变时,准确通知对应的关系
```javascript
//因为只存在comment和post之间唯一的一个关联关系
//所以当comments改变时,comments知道去通知他所关联的post模型

var belongsTo = DS.belongsTo,
    hasMany = DS.hasMany;
App.Comment = DS.Model.extend({
  post: belongsTo('post')
});


App.Post = DS.Model.extend({
  comments: hasMany('comment')
});

```
```javascript
//但是
//当comment和post存在多种映射关系时
//当comments发生改变时,不知道要通知哪一个post,所以需要通过inverse来显式指定
//指定当post下的comments属性改变时,要通知comment上的redPost模型
var belongsTo = DS.belongsTo,
    hasMany = DS.hasMany;
var belongsTo = DS.belongsTo,
    hasMany = DS.hasMany;

App.Comment = DS.Model.extend({
  onePost: belongsTo('post'),
  twoPost: belongsTo('post'),
  redPost: belongsTo('post'),
  bluePost: belongsTo('post')
});


App.Post = DS.Model.extend({
  comments: hasMany('comment', {
    inverse: 'redPost'
  })
});
```
### 可用属性
#### 类属性
##### attributes
该属性可以获取定义在当前模型上的属性信息,不包含关系
```javascript
App.Person = DS.Model.extend({
  firstName: attr('string'),
  lastName: attr('string'),
  birthday: attr('date')
});

var attributes = Ember.get(App.Person, 'attributes')

attributes.forEach(function(name, meta) {
  console.log(name, meta);
});

// prints:
// firstName {type: "string", isAttribute: true, options: Object, parentType: function, name: "firstName"}
// lastName {type: "string", isAttribute: true, options: Object, parentType: function, name: "lastName"}
// birthday {type: "date", isAttribute: true, options: Object, parentType: function, name: "birthday"}
```
##### fields
返回当前模型类型的字段信息,包含属性和关系
```javascript
App.Blog = DS.Model.extend({
  users: DS.hasMany('user'),
  owner: DS.belongsTo('user'),

  posts: DS.hasMany('post'),

  title: DS.attr('string')
});

var fields = Ember.get(App.Blog, 'fields');
fields.forEach(function(field, kind) {
  console.log(field, kind);
});

// prints:
// users, hasMany
// owner, belongsTo
// posts, hasMany
// title, attribute
```
##### relatedTypes
返回当前模型的相关联模型
```javascript
App.Blog = DS.Model.extend({
  users: DS.hasMany('user'),
  owner: DS.belongsTo('user'),

  posts: DS.hasMany('post')
});
var relatedTypes = Ember.get(App.Blog, 'relatedTypes');
//=> [ App.User, App.Post ]
```
##### relationshipNames
返回关系信息
```javascript
App.Blog = DS.Model.extend({
  users: DS.hasMany('user'),
  owner: DS.belongsTo('user'),

  posts: DS.hasMany('post')
});
var relationshipNames = Ember.get(App.Blog, 'relationshipNames');
relationshipNames.hasMany;
//=> ['users', 'posts']
relationshipNames.belongsTo;
//=> ['owner']
```
##### relationships
```javascript
App.Blog = DS.Model.extend({
  users: DS.hasMany('user'),
  owner: DS.belongsTo('user'),
  posts: DS.hasMany('post')
});
var relationships = Ember.get(App.Blog, 'relationships');
relationships.get(App.User);
//=> [ { name: 'users', kind: 'hasMany' },
//     { name: 'owner', kind: 'belongsTo' } ]
relationships.get(App.Post);
//=> [ { name: 'posts', kind: 'hasMany' } ]
```
##### relationshipsByName
```javascript
App.Blog = DS.Model.extend({
  users: DS.hasMany('user'),
  owner: DS.belongsTo('user'),

  posts: DS.hasMany('post')
});
var relationshipsByName = Ember.get(App.Blog, 'relationshipsByName');
relationshipsByName.get('users');
//=> { key: 'users', kind: 'hasMany', type: App.User }
relationshipsByName.get('owner');
//=> { key: 'owner', kind: 'belongsTo', type: App.User }
```
##### transformedAttributes
```javascript
App.Person = DS.Model.extend({
  firstName: attr(),
  lastName: attr('string'),
  birthday: attr('date')
});

var transformedAttributes = Ember.get(App.Person, 'transformedAttributes')

transformedAttributes.forEach(function(field, type) {
  console.log(field, type);
});

// prints:
// lastName string
// birthday date
```
#### 实例属性
##### dirtyType
如果当前模型处于脏状态,该属性可以返回是什么操作使模型变为脏状态
* created The record has been created by the client and not yet saved to the adapter.
* updated The record has been updated by the client and not yet saved to the adapter.
* deleted The record has been deleted by the client and not yet saved to the adapter.

```javascript
var record = store.createRecord('model');
record.get('dirtyType'); // 'created'
```
##### errors
当模型处于`invalid`状态时,会包含相应的错误信息
```javascript
record.get('errors.length'); // 0
record.set('foo', 'invalid value');
record.save().then(null, function() {
  record.get('errors').get('foo'); // ['foo should be a number.']
});
```
### API
* adapterDidCommit
* changedAttributes
* deleteRecord
* destroyRecord
* didDefineProperty
* eachAttribute
* eachRelatedType
* eachRelationship
* eachTransformedAttribute
* reload
* rollback
* save
* serialize
* toJSON
* typeForRelationship 

### 事件钩子
* becameError
* becameInvalid
* didCreate
* didDelete
* didLoad
* didUpdate
