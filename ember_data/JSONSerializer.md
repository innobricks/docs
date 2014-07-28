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
//当在`person`模型中使用`profile` 属性时,在一对一关系下,Ember-Data会自动向后台请求profile的数据
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
当在`person`模型中使用`profile` 属性时,在一对多关系下,Ember-Data不会自动向后台请求`profile`的数据
会默认认为本地已有`profile`的数据
如果要在一对多关系下,`profile`在需要时获取,需要设置为异步,如下:
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
####反序列化钩子，反序列化指的是从服务端返回数据后，客户端对返回的JSON进行解析，生成对应的Record.
+ extract (store, type, payload, id, requestType) 
    + extract用来将从服务端返回的数据进行反序列化，默认情况下JSON serializer并不会在该方法内，将模型推入Store中。JSON Serializer的某些子类实现可能会将模型推入Store，如REST Serializer
                
    ```javascript
    //如下例子,展示了通过socket的方式进行模型的添加
    var get = Ember.get;
    socket.on('message', function(message) {
      var modelName = message.model;
      var data = message.data;
      var type = store.modelFor(modelName);
      var serializer = store.serializerFor(type.typeKey);
      var record = serializer.extract(store, type, data, get(data, 'id'), 'single');
      store.push(modelName, record);
    });
    ```
+ extractArray (store, type, payload) 
    + extractArray 在后端返回值是数组的情况下进行数组的序列化为对象的操作
        
        
        ```javascript
          App.PostSerializer = DS.JSONSerializer.extend({
              extractArray: function(store, type, payload) {
                return payload.map(function(json) {
                  return this.extractSingle(store, type, json);
                }, this);
              }
            });  
        ```
+ extractCreateRecord(store, type, payload)
    + extractCreateRecord，在调用Store.createRecord时会触发该钩子方法，默认情况下，该钩子会调用extractSave钩子
    
+ extractDeleteRecord(store, type, payload)
    + extractDeleteRecord，在调用Store.deleteRecord时会触发该钩子方法，默认情况下，该钩子会调用extractSave钩子
    
+ extractFind(store, type, payload)
    + extractFind，在调用Store.find时会触发该钩子方法，默认情况下，该钩子会调用extractSingle钩子
    
+ extractFindAll(store, type, payload)
    + extractFindAll，在调用Store.findAll时会触发该钩子方法，默认情况下，该钩子会调用extractArray钩
    
+ extractFindBelongsTo(store, type, payload)
    + extractFindBelongsTo，在调用Store.findBelongsTo时会触发该钩子方法，默认情况下，该钩子会调用extractSingle钩子

+ extractFindHasMany(store, type, payload)
    + extractFindHasMany，在调用Store.findHasMany时会触发该钩子方法，默认情况下，该钩子会调用extractArray钩子
        
+ extractFindMany(store, type, payload)
    + extractFindMany，在调用Store.findMany时会触发该钩子方法，默认情况下，该钩子会调用extractSingle钩子

+ extractFindQuery(store, type, payload)
    + extractFindQuery，在调用Store.findQuery时会触发该钩子方法，默认情况下，该钩子会调用extractSingle钩子
    
+ extractMeta(store, type, payload)
    + extractMeta，用来反序列化payload中的元数据信息，默认情况下，元数据信息存储于payload中的meta属性
        
        ```javascript
        App.PostSerializer = DS.JSONSerializer.extend({
          extractMeta: function(store, type, payload) {
            if (payload && payload._pagination) {
              store.metaForType(type, payload._pagination);
              delete payload._pagination;
            }
          }
        });
        ```
    + extractSave(store, type, payload) 
        + extractSave在调用Model.save时会触发该钩子方法，默认情况下，该钩子会调用extractSingle钩子
    + extractSave(store, type, payload) 
        + extractSingle用来反序列化从adapter返回的单条数据记录
        
        ```javascript
            App.PostSerializer = DS.JSONSerializer.extend({
              extractSingle: function(store, type, payload) {
                payload.comments = payload._embedded.comment;
                delete payload._embedded;
            
                return this._super(store, type, payload);
              },
            });
        ```
+ extractUpdateRecord (store, type, payload)
    + extractUpdateRecord在调用Store.update时会触发该钩子方法，默认情况下，该钩子会调用 extractSave钩子
+ keyForAttribute(key) 
    + keyForAttribute 该方法定义了JSON对象与Model对象的映射规则
        
        ```javascript
            App.ApplicationSerializer = DS.RESTSerializer.extend({
              keyForAttribute: function(attr) {
                return Ember.String.underscore(attr).toUpperCase();
              }
            });
        ```
+ keyForRelationship (key, relationship) 
    + keyForRelationship定义了解析模型间关系时，JSON对象与Model对象的映射规则
        
        ```javascirpt
            App.PostSerializer = DS.JSONSerializer.extend({
               keyForRelationship: function(key, relationship) {
                  return 'rel_' + Ember.String.underscore(key);
               }
             });
        ```
+ normalize
    + normalize钩子可以将从服务端返回的数据进行一定的修改，比如将下划线风格的属性，转换为骆驼峰风格的属性等
        
        ```javascript
        App.ApplicationSerializer = DS.JSONSerializer.extend({
          normalize: function(type, hash) {
            var fields = Ember.get(type, 'fields');
            fields.forEach(function(field) {
              var payloadField = Ember.String.underscore(field);
              if (field === payloadField) { return; }
        
              hash[field] = hash[payloadField];
              delete hash[payloadField];
            });
            return this._super.apply(this, arguments);
          }
        });
        ```

+ normalizePayload(payload)
    + normalizePayload，将在normalize方法调用之前得到调用，可以在该钩子内进行payload的调整，如
    
    ```js
    App.ApplicationSerializer = DS.JSONSerializer.extend({
      normalizePayload: function(payload) {
        delete payload.version;
        delete payload.status;
        return payload;
      }
    });
    ```



+ 定制化
    + 有些时候，服务端所需要的数据格式与内建序列化器产生的格式不同，这时候则可以进行数据的定制化
    
    ```javascript
    App.PostSerializer = DS.JSONSerializer.extend({
      //实现serialize钩子，在钩子内构造满足后端数据的数据格式
      serialize: function(post, options) {
        var json = {
          POST_TTL: post.get('title'),
          POST_BDY: post.get('body'),
          POST_CMS: post.get('comments').mapProperty('id')
        }
    
        if (options.includeId) {
          json.POST_ID_ = post.get('id');
        }
    
        return json;
      }
    });
    ```
+ 全局化定制
    + 在需要进行全局性序列化定制时，可以
    
    ```javascript
    App.ApplicationSerializer = DS.JSONSerializer.extend({
      serialize: function(record, options) {
        var json = {};
    
        record.eachAttribute(function(name) {
          json[serverAttributeName(name)] = record.get(name);
        })
    
        record.eachRelationship(function(name, relationship) {
          if (relationship.kind === 'hasMany') {
            json[serverHasManyName(name)] = record.get(name).mapBy('id');
          }
        });
    
        if (options.includeId) {
          json.ID_ = record.get('id');
        }
    
        return json;
      }
    });
    
    function serverAttributeName(attribute) {
      return attribute.underscore().toUpperCase();
    }
    
    function serverHasManyName(name) {
      return serverAttributeName(name.singularize()) + "_IDS";
    }
    
    //序列化的结果为
    {
      "TITLE": "Rails is omakase",
      "BODY": "Yep. Omakase.",
      "COMMENT_IDS": [ 1, 2, 3 ]
    }
    ```

####序列化钩子（将客户短的Record对象转换为JSON String）

+ 序列化微调
    + 某些情况下，我们可能只需要对返回后端的数据进行轻微的调整，这种情况下，可以先调用父类方法
    
    ```javascript
    App.PostSerializer = DS.JSONSerializer.extend({
      serialize: function(record, options) {
        var json = this._super.apply(this, arguments);
    
        json.subject = json.title;
        delete json.title;
    
        return json;
      }
    });
    ```

+ serialize (record, options)
    + serialize钩子，当需要将前端的数据提交到后端时，通过该钩子将record序列化为json数据，默认情况下形如
        
        ```javascript
        App.Comment = DS.Model.extend({
          title: DS.attr(),
          body: DS.attr(),
        
          author: DS.belongsTo('user')
        });
        
        //将会被序列化为下列格式
        {
          "title": "Rails is unagi",
          "body": "Rails? Omakase? O_O",
          "author": 12
        }
        ```
        
+ serializeAttribute(record, json, key, attribute)
    + serializeAttribute用来序列化使用DS.attr方法定义出来的属性
        
        ```javascript
        App.ApplicationSerializer = DS.JSONSerializer.extend({
          serializeAttribute: function(record, json, key, attributes) {
            json.attributes = json.attributes || {};
            this._super(record, json.attributes, key, attributes);
          }
        });
        ```
+ serializeBelongsTo(record, json, relationship)
    + serializeBelongsTo用来序列化使用DS.belongsTo方法定义出来的属性
        
        ```javascript
        App.PostSerializer = DS.JSONSerializer.extend({
          serializeBelongsTo: function(record, json, relationship) {
            var key = relationship.key;
        
            var belongsTo = get(record, key);
        
            key = this.keyForRelationship ? this.keyForRelationship(key, "belongsTo") : key;
        
            json[key] = Ember.isNone(belongsTo) ? belongsTo : belongsTo.toJSON();
          }
        });
        ```
+ serializeHasMany (record, json, relationship)
    + serializeHasMany用来序列化使用DS.hasMany方法定义出来的属性
        
        ```javascript
        App.PostSerializer = DS.JSONSerializer.extend({
          serializeHasMany: function(record, json, relationship) {
            var key = relationship.key;
            if (key === 'comments') {
              return;
            } else {
              this._super.apply(this, arguments);
            }
          }
        });
        ```

+ serializePolymorphicType (record, json, relationship)
    + serializePolymorphicType用来序列化‘多态’属性，当使用DS.belongsTo方法定义，并将{polymorphic: true}作为方法的第二个参数，则该属性为多态属性。该属性序列化时将调用serializePolymorphicType方法进行序列化
        
        ```javascript
        App.CommentSerializer = DS.JSONSerializer.extend({
          serializePolymorphicType: function(record, json, relationship) {
            var key = relationship.key,
                belongsTo = get(record, key);
            key = this.keyForAttribute ? this.keyForAttribute(key) : key;
            json[key + "_type"] = belongsTo.constructor.typeKey;
          }
        });
        ```
