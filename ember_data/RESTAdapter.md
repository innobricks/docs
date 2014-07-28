ta 基础用法

-------------------------------


####模型属性

使用DS.attr进行属性的定义，例如
```javascript
App.User = DS.Model.extend({
  firstName: DS.attr('string'),
  lastName:  DS.attr('string'),
  orders:DS.hasMany('order')
});

App.Order = DS.Model.extend({
  user:DS.belongsTo('user')
})

```
只有使用DS.attr定义的属性会被在 创建或者更新时加入到payload中。

DS.attr 默认可以接受包括 string , number , boolean ,date在内的四种数据类型

-------------------------

####RESTSerializer

Ember Data默认

+   Ember Data 默认使用RESTSerializer处理前端模型->json(序列化)和json->前端模型（反序列化）
+   RESTSerializer对于使用DS.belongsTo定义的模型，对于后端返回的数据格式，要求在响应中含有user字段，字段的value值为对应user的id
+   RESTSerializer将在传递给后端的数据中添加 user字段，并将其对应的id作为user字段的值

对于以上的例子，后端返回的数据应为

```javascript
//GET http://api.example.com/orders?ids[]=19&ids[]=28
{
  "orders": [
    {
      "id":        "19",
      "createdAt": "1401492647008",
      "user":      "1"
    },
    {
      "id":        "28",
      "createdAt": "1401492647008",
      "user":      "1"
    }
  ]
}
```
在数据保存时传递给后端的值应为
```javascript
//POST http://api.example.com/orders
{
  "order": {
    "createdAt": "1401492647008",
    "user":      "1"
  }
}
```

---------------------

####一对一关联关系
对于模型关系对象User和Profile而言，一个User只有一个Profile,一个Profile也只属于一个User,对于这样的关系模型，使用Ember Data可以表示为

```javascript
App.User = DS.Model.extend({
  profile: DS.belongsTo('profile', {async: true})
});

App.Profile = DS.Model.extend({
  user: DS.belongsTo('user', {async: true})
});
```
之后我们可以使用user.get('profile')获取到profile的值，也可以通过user.set('profile',profile)设置profile的值

RESTSerializer希望每个模型都包含有与之关联模型的id:

```javascript
//GET /users
{
  "users": [
    {
      "id": "14",
      "profile": "1"  /* ID of profile associated with this user */
    }
  ]
}

//GET /profiles
{
  "profiles": [
    {
      "id": "1",
      "user": "14"  /* ID of user associated with this profile */
    }
  ]
}
```

相似的，在创建的过程中也应传入对应的模型id

```javascript
//POST /profiles
{
  "profile": {
    "user": "17"  /* ID of user associated with this profile */
  }
}
```

-------------------------------------------
####一对多与多对一关联关系
对于模型关系对象Post和Comment而言，一个Post含有多个Comment,一个Comment只属于一个Post,对于这样的关系模型，使用Ember Data可以表示为

```javascript
App.Post = DS.Model.extend({
  content: DS.attr('string'),
  comments: DS.hasMany('comment', {async: true})
});

App.Comment = DS.Model.extend({
  message: DS.attr('string'),
  post: DS.belongsTo('post',      {async: true})
});
```
之后我们就可以通过post.get('comments', {async: true})获取到post对应的comment列表，或者是post.get('comments').then(function(comments){ return comments.pushObject(aComment);}). 为Post添加新的comment了

服务端需要返回的代码格式应该形如
```javascript
//GET /posts
{
  "posts": [
    {
      "id":       "12",
      "content":  "",
      "comments": ["56", "58"]
    }
  ]
}
```

对于comment返回的数据格式形如
```javascript
//GET /comments?ids[]=56&ids[]=58
{
  "comments": [
    {
      "id":      "56",
      "message": "",
      "post":    "12"
    },
    {
      "id":      "58",
      "message": "",
      "post":    "12"
    }
  ]
}
```

RESTSerializer会在数据提交时将post的id动态打入 payload中

```javascript
//POST /comments
{
  "comment": {
    "message": "",
    "post":    "12"    /* ID of post associated with this comment */
  }
}
```

注意，一对多一的一段而言，RESTSerializer并不会将其多的一段的ids放入请求后端的payload中，所以对于我们上面的例子来说

```javascript
//POST /posts
{
  "post": {
    "content": ""      /* no associated post IDs added here */
  }
}
```

如果强制需要在一的一段加入 多的一段的ids，可以使用[Embedded Records Mixin.](http://www.toptal.com/emberjs/a-thorough-guide-to-ember-data#embeddedRecordsMixin)

-------------------------------------
####多对多关联关系

对于学生与教师这样的关系，使用Ember Data可以表示为

```javascript
App.Teacher = DS.Model.extend({
  name:  DS.attr('string'),
  students: DS.hasMany('post', {async: true}) //注意，key值为student的复数
});

App.Student = DS.Model.extend({
  content: DS.attr('string'),
  teachers: DS.hasMany('author', {async: true}) //teacher的复数
});
```
之后我们可以用teacher.get('students')获取到student列表，也可以使用teacher.get('students').then(function(students){ return students.pushObject(student);})为教师添加新的学生

服务端返回的数据应该形如:
```javascript
//GET /teachers
{
  "teachers": [
    {
      "id":    "1",
      "name":  "",
      "students": ["12"]     /* IDs of students associated with this author */
    }
  ]
}

//GET /students
{
  "students": [
    {
      "id":      "12",
      "content": "",
      "teachers": ["1"]    /* IDs of teachers associated with this post */
    }
  ]
}
```


