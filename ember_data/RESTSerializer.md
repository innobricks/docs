### 概述
REST Adapter是[JSON Serializer](./JSONSerializer.md)的实现子类
通常情况下，应用程序将使用RESTSerializer的normalizeHash方法完全数据的序列化.

#### 方法钩子(重写的部分钩子方法，更多钩子方法见[JSON Serializer](./JSONSerializer.md))
+ extractArray (store, primaryType, payload)
>当服务端返回多条数据时，该钩子方法得到调用，如对于如下数据

    ```js
    {
      "_embedded": {
        "post": [{
          "id": 1,
          "title": "Rails is omakase"
        }, {
          "id": 2,
          "title": "The Parley Letter"
        }],
        "comment": [{
          "_id": 1,
          "comment_title": "Rails is unagi"
          "post_id": 1
        }, {
          "_id": 2,
          "comment_title": "Don't tread on me",
          "post_id": 2
        }]
      }
    }
    ```
我们可以实现以下钩子方法，完成对服务端JSON string的反序列化
    
    ```js
    App.PostSerializer = DS.RESTSerializer.extend({
  // First, restructure the top-level so it's organized by type
  // and the comments are listed under a post's `comments` key.
      extractArray: function(store, type, payload) {
        var posts = payload._embedded.post;
        var comments = [];
        var postCache = {};
    
        posts.forEach(function(post) {
          post.comments = [];
          postCache[post.id] = post;
        });
    
        payload._embedded.comment.forEach(function(comment) {
          comments.push(comment);
          postCache[comment.post_id].comments.push(comment);
          delete comment.post_id;
        }
    
        payload = { comments: comments, posts: payload };
    
        return this._super(store, type, payload);
      },
    
      normalizeHash: {
        // Next, normalize individual comments, which (after `extract`)
        // are now located under `comments`
        comments: function(hash) {
          hash.id = hash._id;
          hash.title = hash.comment_title;
          delete hash._id;
          delete hash.comment_title;
          return hash;
        }
      }
})
```
+ extractSingle(store, primaryType, payload, recordId)
>当服务端返回单条数据时，该方法得到调用，对于如下数据
    
    ```js
    {
      "id": 1,
      "title": "Rails is omakase",
    
      "_embedded": {
        "comment": [{
          "_id": 1,
          "comment_title": "FIRST"
        }, {
          "_id": 2,
          "comment_title": "Rails is unagi"
        }]
      }
    }
    ```
我们可以实现以下钩子方法，实现对数据的反序列化

    ```js
    App.PostSerializer = DS.RESTSerializer.extend({
      // First, restructure the top-level so it's organized by type
      extractSingle: function(store, type, payload, id) {
        var comments = payload._embedded.comment;
        delete payload._embedded;
    
        payload = { comments: comments, post: payload };
        return this._super(store, type, payload, id);
      },
    
      normalizeHash: {
        // Next, normalize individual comments, which (after `extract`)
        // are now located under `comments`
        comments: function(hash) {
          hash.id = hash._id;
          hash.title = hash.comment_title;
          delete hash._id;
          delete hash.comment_title;
          return hash;
        }
      }
    })
    ```
+ pushPayload (store, payload)

> 向Store内加入模型，该方法将在向Store推入模型前，进行数据的normalize

#### 属性钩子

+ normalizeHash
>该钩子将在数据进行normalize时，根据所匹配的key进行相应的序列化，如
    
    ```js
    //对于给定的以下模型，comment属性内的id是_id
    {
      "post": {
        "id": 1,
        "title": "Rails is omakase",
        "comments": [ 1, 2 ]
      },
      "comments": [{
        "_id": 1,
        "body": "FIRST"
      }, {
        "_id": 2,
        "body": "Rails is unagi"
      }]
    }
    ```
    可以特别指定对comment的反序列化
    ```js
    App.PostSerializer = DS.RESTSerializer.extend({
      normalizeHash: {
        comments: function(hash) {
          hash.id = hash._id;
          delete hash._id;
          return hash;
        }
      }
    });
    ```