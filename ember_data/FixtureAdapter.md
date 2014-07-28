# FixtureAdapter
继承自[Adapter](https://github.com/innobricks/docs/tree/master/ember_data/Adapter.md)
* 实现了`Adapter`的所有抽象方法
* 用于处理本地数据的数据请求(模拟)
* 默认情况下,fixtures数据已经是规范化的,不再需要序列化和反序列化

### 数据规则
FixtureAdapter用于模拟远程请求,所以数据在本地构造
```javascript
//大写
type.FIXTURES
//要求是一个数组,可以重写`fixturesForType`,自定以规则
App.Person.FIXTURES=[];
```
```javascript
//在本地也模拟分页
App.Person = DS.Model.extend({
    firstName:attr(),
    lastName:attr()
})；
App.Person.FIXTURES={
    data:[{
            id:1,
            firstName:"fixture1",
            lastName:"lastFix"
        }],
    meta:{
        total:100
    }
};
FixtureAdapter.extend({
    fixturesForType:function(type){
        if(type.FIXTURES){
            var meta=type.FIXTURES.meta;
            //doSomething
            var fixtures = Ember.A(type.FIXTURES.data);
              return fixtures.map(function(fixture){
                var fixtureIdType = typeof fixture.id;
                if(fixtureIdType !== "number" && fixtureIdType !== "string"){
                  throw new Error(fmt('the id property must be defined as a number or string for fixture %@', [fixture]));
                }
                fixture.id = fixture.id + '';
                return fixture;
              });
        }
        return this._super(type)；
    }
});
```

### 属性钩子
simulateRemoteResponse:true 模拟远程请求
latency:50 模拟远程请求所花费的时间
### 方法钩子
queryFixtures 查询
generateIdForRecord 客户端自定以`id`生成规则
### API
* createRecord
* updateRecord
* deleteRecord