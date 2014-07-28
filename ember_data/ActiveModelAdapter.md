# ActiveModelAdapter
继承自[RESTAdapter](https://github.com/innobricks/docs/tree/master/ember_data/RESTAdapter.md)
* 与`RESTAdapter`使用骆驼峰(`famousPeople`)不同,使用下划线(`famous_people`)
* 与`active_model_serializer`配套使用


### 属性钩子
无

### 方法钩子
1.pathForType 返回模型类型的字符串路径信息
```javascript
//camelize+puralize
App.FamousPerson-->famous_people
```