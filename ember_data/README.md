# Ember-Data使用指南
### 更多教程
+ [Ember Data: A Comprehensive Tutorial for the ember-data Library](http://www.toptal.com/emberjs/a-thorough-guide-to-ember-data)
+ [EMBER 1.0 – RISE FROM THE ASHES](http://dev.mygrid.org.uk/blog/tag/ember-data/)

>>>>>>> e2538df32632b4fa9b8afc2e1ab64453fd25daaf

### 要素
* Store
* Adapter
* Serializer
* Transform
* Model
* Inflection

### 职能
* `Store`负责缓存所有的模型,以及提供统一的接口,调度`adapter`向后端发起请求
* `Adapter`负责与后端进行通讯,负责数据的传输,目前支持HTTP,由`Store`调度
* `Serializer`负责数据的序列化和反序列化,由`Adapter`调度
* `Transform`负责数据格式的转换,由`Serializer`调度
* `Model`定义模型的状态,属性,接口等等,是所有模型都应该继承的基类
* `Inflection`负责`RESTful`的url资源映射

### 推荐
* 官方推荐使用`RESTAdapter`

### 说明
* 整个Ember-Data已经完全采用`Promise`模型进行编码
* `Store`获取模型,模型获取属性,都需要用`Promise`方式进行编码
```javascript
this.store.find('person',1).then(function(person){
    //doSomething
});
person.get('profile').then(function(profile){
    //doSomething
});
```
## 大纲
* [Store](https://github.com/innobricks/docs/tree/master/ember_data/Store.md)
* [Adapter](https://github.com/innobricks/docs/tree/master/ember_data/Adapter.md)
    * [RESTAdapter](https://github.com/innobricks/docs/tree/master/ember_data/RESTAdapter.md)
        * [ActiveModelAdapter](https://github.com/innobricks/docs/tree/master/ember_data/ActiveModelAdapter.md)
    * [FixtureAdapter](https://github.com/innobricks/docs/tree/master/ember_data/FixtureAdapter.md)
* [JSONSerializer](https://github.com/innobricks/docs/tree/master/ember_data/JSONSerializer.md)
    * [RESTSerializer](https://github.com/innobricks/docs/tree/master/ember_data/RESTSerializer.md)
        * [ActiveModelSerializer](https://github.com/innobricks/docs/tree/master/ember_data/ActiveModelSerializer.md)
* [Transform](https://github.com/innobricks/docs/tree/master/ember_data/Transform.md)
* [Model](https://github.com/innobricks/docs/tree/master/ember_data/Model.md)
* [Inflection](https://github.com/innobricks/docs/tree/master/ember_data/Inflection.md)
* [EmbeddedRecordsMixin](https://github.com/innobricks/docs/tree/master/ember_data/EmbeddedRecordsMixin.md)