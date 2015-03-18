终于到了CRUD的大哥：find方法了。

####第一个参数####
MongoDB使用find来进行查询，查询呢，就是返回一个集合中文档的子集，子集合的范围从0个文档到整个集合。

find的第一个参数决定了要返回那些文档，其形式也就一个文档，说明了要执行的查询细节。

通常呢，大家看到的find查询基本都长这个样子
> db.user.find({})

> db.user.find({"name":"qianjiahao"})

> db.user.find({"name":"qianjiahao","email":"example@example.com"})

####第二个参数####
但是，有的时候，我们并不希望将文档中的所有键/值对都返回，这时，我们可以在find方法的第二个参数上指明我们希望返回的信息。
> db.user.find({},{"name":1,"email":1})

上面的语句意思是：我们只想得到name和email，其他的不关心。像这样指明返回信息的做法肯定是有好处的，它可以帮助我们节省传输的数据量，又能节省客户端解码文档的时间和内存消耗。
比如，现在有这两条数据

{ "_id" : ObjectId("5509087e08fa61313b5a8230"), "name" : "william", "email" : "example@example.com" }

{ "_id" : ObjectId("5509088b08fa61313b5a8231"), "name" : "jack", "email" : "example@example1.com" }

我们只想得到name，连 _id  都不想要，那么可以这样
> db.user.find({},{"name":1,"_id":0})

{ "name" : "william" }

{ "name" : "jack" }

####注意####
**数据库关心的查询文档的值，必须是常量（在你自己的代码里可以是正常的变量），换句话说，不可以引用文档中其他键的值！**

###查询条件###

####$lt $lte $gt $gte####
以上四个分别表示为：< 、 <= 、 > 、 >= 。
通常的做法是将他们组合起来，以便查找一个范围。
比如，查询年龄在18到25岁（含）的人，我们可以这样
> db.user.find({"age":{"$gte":18,"$lte":25}})

这样的范围查询对查询日期特别有用
比如，查询在2015年1月1日后注册的用户
> start = new Date("01/01/2015")
> db.user.find({"register":{"$gte":start}})

####注意####
**不要去匹配精确的日期，而是用范围来对日期进行查询**

####$ne####
$ne表示不相等
> db.user.find({"name":{"$ne":"william"}})

###OR查询###
####$in####
$in可以查询一个键的多个值
举例，每个人有很多不同的爱好，
> db.user.find()

{ "_id" : ObjectId("55090f8008fa61313b5a8232"), "name" : "william", "hobby" : [ "swimming", "running" ] }

{ "_id" : ObjectId("55090f9408fa61313b5a8233"), "name" : "jack", "hobby" : [ "swimming", "singing" ] }

{ "_id" : ObjectId("55090fac08fa61313b5a8234"), "name" : "tom", "hobby" : [ "swimming", "running", "dancing" ] }

数据太多，咱们用第二个参数来过滤一下
> db.user.find({},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "running" ] }
{ "hobby" : [ "swimming", "singing" ] }
{ "hobby" : [ "swimming", "running", "dancing" ] }

这下看的清楚了一些，我们想查询喜欢swimming和dancing的人，可以得到如下结果
> db.user.find({"hobby":{"$in":["swimming","dancing"]}},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "running" ] }
{ "hobby" : [ "swimming", "singing" ] }
{ "hobby" : [ "swimming", "running", "dancing" ] }
我们会发现满足其一就可以被查询到，也就是说这是个并集
现在我们只要查询dancing的人
> db.user.find({"hobby":{"$in":["dancing"]}},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "running", "dancing" ] }

既然$in,那么与之相对的就$nin,可以查询到不包括指明信息的文档

###$or####
$in 是对单个键进行的查询，那么，对多个键呢，这时就要用到$or查询了，改写刚刚的user，为其添加gender键
> db.user.find({},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "running" ], "gender" : "male" }
{ "hobby" : [ "swimming", "singing" ], "gender" : "female" }
{ "hobby" : [ "swimming", "running", "dancing" ], "gender" : "female" }

使用$or查询
> db.user.find({"$or":[{"hobby":"dancing"},{"gender":"female"}]},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "singing" ], "gender" : "female" }
{ "hobby" : [ "swimming", "running", "dancing" ], "gender" : "female" }

两条数据，第一条满足女性，第二条满足会跳舞和女性

现在，我们把查询条件的female改成male
> db.user.find({"$or":[{"hobby":{"$in":["dancing"]}},{"gender":"male"}]},{"_id":0,"name":0})

{ "hobby" : [ "swimming", "running" ], "gender" : "male" }
{ "hobby" : [ "swimming", "running", "dancing" ], "gender" : "female" }

可以看到第一条满足男性，第二条满足会跳舞和女性

现在我们可以得出结论，OR查询（$in 和 $or）是尽可能的获取更多的匹配项。
举个夸张的例子，方便记忆：就是沾边的都算亲戚，就算是三舅的七姑的妹妹的女儿的哥哥，都算亲戚！

####$not####
not 是元条件句，可以用于任何条件之上，意为取反

####注意####
一个键不能对应多个更新修改器
但是可以对应多个条件查询句
比如，可以这样
> db.user.find({"age":{"$gt":18,"$lt":30}})

但是，不可以这样
> {"$inc":{"age":1},"$set":{"age":20}}

因为他们修改了两次age

