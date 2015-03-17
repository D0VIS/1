###update###

update方法很强大，它有两个参数，一是查询文档，用来找出需要更新的文档，另一个是修改器（modifier）文档，描述对找到的文档做哪些修改。

####亮点####
更新操作是原子的，若两个更新同时发生，先到达服务器的现执行，接着执行另一个。所以，互相冲突的更新可以火速传递，并不会互相干扰，虽然这是一个拼速度的年代，但是后更新的会取得“胜利”（后发制人！）

因为使用原子的 **更新修改器** 进行更新操作极为高效，所以，就决定用它了！

***

###$inc###
例如：你需要存储一个网站的访问次数
> db.web.insert({"url":"www.example","count":2})

> { "_id" : ObjectId("55082435591555a6c35dd697"), "url" : "www.example", "count" : 2 }

这时，可以用$inc来原子的增加（减少）count的值
> db.web.update({"url":"www.example"},{$inc:{"count":1}})

> { "_id" : ObjectId("55082435591555a6c35dd697"), "url" : "www.example", "count" : 3 }

$inc指定是对数字的增减操作，count指定操作对象，1指定步长

***

###$set & $unset###

用$set指定一个键的值，如果不存在，就创建它。这对更新模式或者增加用户定义很有帮助。
> db.user.insert({"name":"qianjiahao"})

此用户现在只有姓名信息，现在需要给他添加email
> db.user.update({"name":"qianjiahao"},{"$set":{"email":"example@example.com"}})

> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com" }

比如现在他又要添加他的个人爱好
> db.user.update({"name":"qianjiahao"},{"$set":{"hobby":["swimming","running","reading"]}})

> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com", "hobby" : [ "swimming", "running", "reading" ] }

假如他现在又没有爱好了...
> db.user.update({"name":"qianjiahao"},{"$unset":{"hobby":1}})

> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com" }

爱好就消失了...

***

###$push$$$
数组修改器，既然名字都这样叫了，那么这个修改器就只能对数组进行操作啦。
> db.user.update({"name":"qianjiahao"},{"$push":{"hobby":"sleeping"}})

> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com", "hobby" : [ "swimming", "running", "reading", "sleeping" ] }

但是这里有个问题，万一你不确定hobby里面是否有一个值，如“singing”，那么也许下面这个方法更适合

***

###$addToSet & $each###
用$addToSet更新可以避免重复，将它与$each组合起来，可以一次性添加多条（就算后添加的值已存在也没有关系）
> db.user.update({"name":"qianjiahao"},{"$addToSet":{"hobby":{"$each":["singing","eating","dancing"]}}})
> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com", "hobby" : [ "swimming", "running", "reading", "sleeping", "singing", "eating", "dancing" ] }

***

###$pop###
如果将数组看做队列，可以用$pop方法删除第一个或者最后一个元素
{$pop:{"key":-1}}，{$pop:{"key":1}}

***

###$pull###
它可以删除所以匹配的值，如果[1,1,2,1] 执行pull后，只剩下[2]

***

###定位修改器###
如果要操作数组中的值，可以用值在数组中的位置当做参数来删除
> db.user.update({"name":"qianjiahao"},{"$set":{"hobby.0":"crying"}})
{ "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com", "hobby" : [ "crying", "running", "reading", "sleeping", "singing", "eating", "dancing" ] }

现在哭成了主要爱好了，这不行，咱得改了，但是如果不事先去查，并不知道哭是这个hobby数组的第几个值，MongoDB为我们考虑到了这点，使用$来代替位置。
> db.user.update({"hobby":"crying"},{"$set":{"hobby.$":"smiling"}})
> { "_id" : ObjectId("55082691591555a6c35dd698"), "name" : "qianjiahao", "email" : "example@example.com", "hobby" : [ "smiling", "running", "reading", "sleeping", "singing", "eating", "dancing" ] }

现在破涕为笑了，笑一笑，挺好~