###概述
DS.Transform在模型saved或者loaded的时候进行属性的序列化和反序列化工作，继承该DS.Transform可以实现自定义属性类型，
继承DS.Transform必须实现`deserialize`(服务端数据->客户端record)与`serialize`(客户端record->JSON string)方法.

---------------------------------------------
教程见: [EmberJS How To Create Custom Data Types Using Tranforms](http://www.lswebapps.com/code-snippet/emberjs-how-to-create-custom-data-types-using-tranforms/)

---------------------------------------------

```js
// Converts centigrade in the JSON to fahrenheit in the app
App.TemperatureTransform = DS.Transform.extend({
  deserialize: function(serialized) {
    return (serialized *  1.8) + 32;
  },
  serialize: function(deserialized) {
    return (deserialized - 32) / 1.8;
  }
});
App.register("transform:temperatureTransform", DS.TemperatureTransform);
```

之后便可以在在DS.attr中使用该属性

```js
var Post=DS.Model.extend({
    title:DS.attr('string'),
    something:DS.attr('temperatureTransform')
});
```