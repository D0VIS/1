MongoDB从零起步走之insert、remove
----------------
#这里需要先启动mongodb服务器：.\mongod.exe --dbpath E:\mongodb\data\db 最后一个参数是指定数据库文件所在的位置，不指定的话会启动失败
#启动之后会等待连接
#这时候新开一个terminal，输入mongo 就可以连接服务器，对应服务器也会打印相应的内容

连接MongoDB（bin目录下）
>./mongo

如果觉得shell里空空的可以输入help，在刷屏的同时大致了解下有哪些方法
> help

现在咱们还没有数据库，咱们创建一个，任性起名：template
>use template

咱们确认下，数据库有没有创建成功
>show dbs

> template  0.078GB

如果存在template，就进入，如果没有，在最后保存的时候就会创建template

***

###insert###

发现已经创建成功，继续走，现在咱们创建一个集合,任性起名：room,再插入点数据
> db.room.insert({"desk":1,"bed":1,"window":2})

> db.room.find()

> { "_id" : ObjectId("55081e42591555a6c35dd695"), "desk" : 1, "bed" : 1, "window" : 2 }


####知识点####
* insert会给文档增加一个_id（如果原来没有的话），这个_id是文档的唯一标示，然后保存到数据库中

###remove###

现在咱们不想要这个room文档了，那么来删除它
> db.room.remove({})

注意，在remove() 里传入了空对象{},意为删除全部数据，除了这样删除，我们还可以这样删除
> db.room.remove({"desk":1})

####Tips####
> db.dropDatabase()

删除数据库

> db.collection.drop()

删除集合
