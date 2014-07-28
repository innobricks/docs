CTIVEMODELSERIALIZER
ACTIVEMODELSERIALIZER是RESTSerializer的子类实现，它与RESTSerializer最大的不同在于它使用了下划线风格的代码风格，而不是传统的骆驼峰风格的代码风格

####代码约定
对于给定的模型结构
```javascript
App.FamousPerson = DS.Model.extend({
  firstName: DS.attr('string'),
  lastName: DS.attr('string'),
  occupation: DS.attr('string')
});
```
与之匹配的后端返回数据结构应该为:
```javascript
{
  "famous_person": {
    "first_name": "Barack", //这里都是使用下划线的代码风格
    "last_name": "Obama", //这里都是使用下划线的代码风格
    "occupation": "President"
  }
}
```

-----------------------
对于存在关联关系的模型，如1对多，多对已等，对于给定模型

```javascript
  App.Person = DS.Model.extend({
    firstName: DS.attr('string'),
    lastName: DS.attr('string'),
    occupation: DS.belongsTo('occupation')  //一对多，保持单数形式
  });

  App.Occupation = DS.Model.extend({
    name: DS.attr('string'),
    salary: DS.attr('number'),
    people: DS.hasMany('person') //注意，这里是多对一，一那一端的复数形式
  });
```
所对应后端应该返回的JSON文件格式应为：
```javascript
{
    "people": [{
      "id": 1,
      "first_name": "Barack",
      "last_name": "Obama",
      "occupation_id": 1 //这里都是使用下划线的代码风格
    }],

    "occupations": [{
      "id": 1,
      "name": "President",
      "salary": 100000,
      "person_ids": [1] //这里都是使用下划线的代码风格
    }]
  }
```








