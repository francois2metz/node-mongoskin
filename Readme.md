<a name='index'>

* [Nodejs mongodb drivers comparation](#comparation)
* [Install](#install)
* [Quick Start](#quickstart)
    * [Connect easier](#quickstart-1)
    * [Server options and BSON options](#quickstart-2)
    * [Similar API with node-mongodb-native](#quickstart-3)
    * [Cursor easier](#quickstart-4)
    * [MVC helper](#quickstart-5)
* [Documentation](#documentation)
    * [Module](#module)
    * [SkinServer](#skinserver)
    * [SkinDb](#skindb)
    * [SkinCollection](#skincollection)
      * [Additional methods](#additional-collection-op)
      * [Collection operation](#inherit-collection-op)
      * [Indexes](#inherit-indexes)
      * [Querying](#inherit-query)
      * [Aggregation](#inherit-aggregation)
      * [Inserting](#inherit-inserting)
      * [Updating](#inherit-updating)
      * [Removing](#inherit-removing)
    * [SkinCursor](#skincursor)

<a name='comparation'>

Nodejs mongodb drivers comparation
========

node-mongodb-native
--------

Powerful, most of drivers include mongoskin build upon it,
  but node-mongodb-native has awkward syntax, too many callbacks,
  and we need a way to hold Collection instance as Model for MVC.

mongoose
--------

It provide an ORM way to hold Collection instance as Model,
  you should define schema first. But why mongodb need schema?
  Some guys like me, want to write code from application layer but not database layer,
  and we can use any fields without define it before.

  Mongoose provide a DAL that you can do validation, and write your middlewares.
  But some guys like me would like to validate manually, I think it is the tao of mongodb.

  If you don't thinks so, [Mongoose-ORM](https://github.com/LearnBoost/mongoose) is probably your choice.

mongoskin
--------

Mongoskin is an easy to use driver of mongodb for nodejs,
  it is similar with mongo shell, powerful like node-mongodb-native,
  and support additional javascript method binding, which make it can act as a Model(in document way).

It will provide full features of [node-mongodb-native](https://github.com/christkv/node-mongodb-native),
  and make it [future](http://en.wikipedia.org/wiki/Future_%28programming%29).

[Back to index](#index)

<a name='install'></a>

Install
========

    npm install mongoskin

[Back to index](#index)


<a name='quickstart'></a>

Quick Start
========

 **Is mongoskin synchronized?**

Nope! It is asynchronized, it use the [future pattern](http://en.wikipedia.org/wiki/Future_%28programming%29).
**Mongoskin** is the future layer above [node-mongodb-native](https://github.com/christkv/node-mongodb-native)

<a name='quickstart-1'></a>

Connect easier
--------
You can connect to mongodb easier now.

    var mongo = require('mongoskin');
    mongo.db('localhost:27017/testdb').collection('blog').find().toArray(function(err, items){
        console.dir(items);
    })

<a name='quickstart-2'></a>

Server options and BSON options
--------
You can also set `auto_reconnect` options querystring.
And native_parser options will automatically set if native_parser is avariable.

    var mongo = require('mongoskin'),
        db = mongo.db('localhost:27017/test?auto_reconnect');

<a name='quickstart-3'></a>

Similar API with node-mongodb-native
--------
You can do everything that node-mongodb-native can do.

    db.createCollection(...);
    db.collection('user').ensureIndex([['username', 1]], true, function(err, replies){});
    db.collection('posts').hint = 'slug';
    db.collection('posts').findOne({slug: 'whats-up'}, function(err, post){
        // do something
    });

<a name='quickstart-4'></a>

Cursor easier
--------

    db.collection('posts').find().toArray(function(err, posts){
        // do something
    });

<a name='quickstart-5'></a>

MVC helper
--------

You can bind **additional methods** for collection.
It is very useful if you want to use MVC patterns with nodejs and mongodb.
You can also invoke collection by properties after bind,
it could simplfy your `require`.

    db.bind('posts', {
       findTop10 : function(fn){
         this.find({}, {limit:10, sort:[['views', -1]]}).toArray(fn);
       },
       removeTagWith : function(tag, fn){
         this.remove({tags:tag},fn);
       }
    });

    db.bind('comments');

    db.collection('posts').removeTagWith('delete', function(err, replies){
      //do something
    });

    db.posts.findTop10(function(err, topPosts){
      //do something
    });

    db.comments.find().toArray(function(err, comments){
      //do something
    });

[Back to index](#index)


<a name='documentation'>

Documentation
========

for more information, see the source.

[Back to index](#index)


<a name='module'>

Module
--------

### MongoSkin Url format

    [*://][username:password@]host[:port][/database][?auto_reconnect[=true|false]]`

e.g.

    localhost/blog
    mongo://admin:pass@127.0.0.1:27017/blog?auto_reconnect
    127.0.0.1?auto_reconnect=false


### bind(collectionName)

### bind(collectionName, SkinCollection)

### bind(collectionName, extendObject1, extendObject2 ...)

Bind [SkinCollection](#skincollection) to db properties. see [SkinDb.bind](#skindb-bind) for more information.

### db(databaseUrl)

Get or create instance of [SkinDb](#skindb).

### cluster(serverUrl1, serverUrl2, ...)

Create [SkinServer](#skinserver) of native [ServerCluster](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/connection.js#L221). e.g.

    var mongo = require('mongoskin');
    var cluster = mongo.cluster('192.168.0.1:27017', '192.168.0.2:27017', '192.168.0.3:27017')
    var db = cluster.db('dbname', 'admin', 'pass');

### pair(leftServerUrl, rightServerUrl)

Create [SkinServer](#skinserver) of native [ServerPair](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/connection.js#L190)

### router(select)

select is function(collectionName) returns a database instance, means router collectionName to that database.

    var db = mongo.router(function(coll_name){
        switch(coll_name) {
        case 'user':
        case 'message':
          return mongo.db('192.168.1.3/auth_db');
        default:
          return mongo.db('192.168.1.2/app_db');
        }
    });
    db.bind('user', require('./shared-user-methods'));
    var users = db.user; //auth_db.user
    var messages = db.collection('message'); // auth_db.message
    var products = db.collection('product'); //app_db.product



[Back to index](#index)

<a name='skinserver'>

SkinServer
--------

### SkinServer(server)

Construct SkinServer from native Server instance.

### db(dbname, username=null, password=null)

Construct [SkinDb](#skindb) from SkinServer.

[Back to index](#index)

<a name='skindb'>

SkinDb
--------

### SkinDb(db, username=null, password=null)

Construct SkinDb.

### open(callback)

Connect to database, retrieval native
[Db](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/db.js#L17)
instance, callback is function(err, db).

### collection(collectionName)

Retrieval [SkinCollection](#skincollection) instance of specified collection name.

<a name='skindb-bind'>

### bind(collectionName)

### bind(collectionName, SkinCollection)

### bind(collectionName, extendObject1, extendObject2 ...)

Bind [SkinCollection](#skincollection) to db properties as a shortcut to db.collection(name).
You can also bind additional methods to the SkinCollection, it is useful when
you want to reuse a complex operation. This will also affect
db.collection(name) method.

e.g.

    db.bind('book', {
        firstBook: function(fn){
            this.findOne(fn);
        }
    });
    db.book.firstBook(function(err, book){});

### all the methods from Db.prototype

See [Db](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/db.js#L17) of node-mongodb-native for more information.

[Back to index](#index)

<a name='skincollection'>

SkinCollection
--------

See [Collection](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L45) of node-mongodb-native for more information.

<a name='additional-collection-op'>
### open(callback)

Retrieval native
[Collection](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L45)
instance, callback is function(err, collection).

### id(hex)

Equivalent to

    db.bson_serilizer.ObjectID.createFromHexString(hex);

See [ObjectID.createFromHexString](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/bson/bson.js#L548)


<a name='inherit-collection-op'>

### Collection operation

    checkCollectionName(collectionName)
    options(callback)
    rename(collectionName, callback)
    drop(callback)

<a name='inherit-indexes'>

### Indexes

    createIndex (fieldOrSpec, unique, callback)
    ensureIndex (fieldOrSpec, unique, callback)
    indexInformation (callback)
    dropIndex (indexName, callback)
    dropIndexes (callback)

<a name='inherit-query'>

### Query

#### findItems(..., callback)

Equivalent to

    collection.find(..., function(err, cursor){
        cursor.toArray(callback);
    });

See [Collection.find](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L348)

#### findEach(..., callback)

Equivalent to

    collection.find(..., function(err, cursor){
        cursor.each(callback);
    });

See [Collection.find](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L348)

#### findById(id, ..., callback)

Equivalent to

    collection.findOne({_id, ObjectID.createFromHexString(id)}, ..., callback);

See [Collection.findOne](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L417)

#### find(...)

If the last parameter is function, it is equivalent to native
[Collection.find](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L348)
method, else it will return a future [SkinCursor](#skincursor).

e.g.

    // callback
    db.book.find({}, function(err, cursor){/* do something */});
    // future SkinCursor
    db.book.find().toArray(function(err, books){/* do something */});



#### normalizeHintField(hint)

#### find

    /**
     * Various argument possibilities
     * 1 callback
     * 2 selector, callback,
     * 2 callback, options  // really?!
     * 3 selector, fields, callback
     * 3 selector, options, callback
     * 4,selector, fields, options, callback
     * 5 selector, fields, skip, limit, callback
     * 6 selector, fields, skip, limit, timeout, callback
     *
     * Available options:
     * limit, sort, fields, skip, hint, explain, snapshot, timeout, tailable, batchSize
     */

#### findAndModify(query, sort, update, options, callback) 

    /**
      Fetch and update a collection
      query:        a filter for the query
      sort:         if multiple docs match, choose the first one in the specified sort order as the object to manipulate
      update:       an object describing the modifications to the documents selected by the query
      options:
        remove:   set to a true to remove the object before returning
        new:      set to true if you want to return the modified object rather than the original. Ignored for remove.
        upsert:       true/false (perform upsert operation)
    **/

#### findOne(queryObject, options, callback)

<a name='inherit-aggregation'>

### Aggregation

#### mapReduce(map, reduce, options, callback)

    e.g.  ```
      var map = function(){
          emit(test(this.timestamp.getYear()), 1);
      }
      
      var reduce = function(k, v){
          count = 0;
          for(i = 0; i < v.length; i++) {
              count += v[i];
          }
          return count;
      }
      collection.mapReduce(map, reduce, {scope:{test:new client.bson_serializer.Code(t.toString())}}, function(err, collection) {
        collection.find(function(err, cursor) {
              cursor.toArray(function(err, results) {
              test.equal(2, results[0].value)
              finished_test({test_map_reduce_functions_scope:'ok'});            
          })
        })
          ```

#### group(keys, condition, initial, reduce, command, callback)

    e.g.  `collection.group([], {}, {"count":0}, "function (obj, prev) { prev.count++; }", true, function(err, results) {`

    count(query, callback)
    distinct(key, query, callback)

<a name='inherit-inserting'>

### Inserting

#### insert(docs, options, callback)

#### insertAll(docs, options, callback)

<a name='inherit-updating'>

### Updating

#### save(doc, options, callback)

    /**
      Update a single document in this collection.
        spec - a associcated array containing the fields that need to be present in
          the document for the update to succeed

        document - an associated array with the fields to be updated or in the case of
          a upsert operation the fields to be inserted.

      Options:
        upsert - true/false (perform upsert operation)
        multi - true/false (update all documents matching spec)
        safe - true/false (perform check if the operation failed, required extra call to db)
    **/

#### update(spec, document, options, callback)

#### updateById(_id, ..., callback)

Equivalent to

    collection.update({_id, ObjectID.createFromHexString(id)}, ..., callback);

See [Collection.update](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/collection.js#L198)


<a name='inherit-removing'>

### Removing

#### remove(selector, options, callback)

#### removeById(_id, options, callback)

[Back to index](#index)

<a name='skincursor'>

SkinCursor
---------

See [Cursor](https://github.com/christkv/node-mongodb-native/blob/master/lib/mongodb/cursor.js#L1)
of node-mongodb-native for more information.

All these methods will return the SkinCursor itself.

    sort(keyOrList, [direction], [callback])
    limit(limit, [callback])
    skip(skip, [callback])
    batchSize(skip, [callback])

    toArray(callback)
    each(callback)
    count(callback)
    nextObject(callback)
    getMore(callback)
    explain(callback)


[Back to index](#index)
