####upsert##
upsert是一个选项，它是update的第三个参数，并不是一个方法。它是一种特殊的更新，要是没有文档符合匹配，那么它就会根据条件和更新文档为基础，创建新的文档，如有匹配，则正常更新。咱们之前见到的所有update操作，都是建立在有文档的基础之上的。upsert非常方便，不必预制集合，同一套代码既可以创建又可以更新。

超市需要修改商品的价格，比如将苹果的价格上调0.5元，但是店主不确定自己是否有购进苹果（偶尔会犯糊涂），那么他可以这样
> db.supermarket.update({"name":"apple"},{$set:{"price":5.5}},True)

如果MongoDB之前有苹果的记录，那么就会update苹果的价格，如果没有就会创建新的文档
> { "_id" : ObjectId("55083db0720f2a27156f66ed"), "name" : "apple", "price" : 5.5 }

***

###save###

save是一个shell函数，调用它，可以在文档不存在时插入，存在时更新，它只有一个参数：文档。如果文档有 _id 这个 键，那么save会调用upsert，否则会调用insert，非常方便。

***

###多文档更新###

当一次更新一个文档无法满足我们的脚步时，我们可以选择一次更新多个文档，及在update的第四个参数的位置添上true，及做多文档更新，建议就算不做多文档更新也显式的在第四个参数上置false，这样明确易懂，也可以在默认参数变化时从容应对。

运行getLastError命令可以帮助我们获取反馈信息
> db.count.update({"x":3},{$inc:{"x":2}},false,true)

> db.runCommand({getLastError:1})

{
	"connectionId" : 199,
	"updatedExisting" : true,
	"n" : 1,
	"syncMillis" : 0,
	"writtenTo" : null,
	"err" : null,
	"ok" : 1
}

***

###安全VS性能###
insert、remove、update这些操作都是瞬间的，他们不需要等待数据库响应，可以理解为子弹从枪膛里射出但不关心射没射中！并不是说这些操作是异步的，而是说后面的事你不用担心了，说白了，子弹射出去后怎么飞、怎么击中目标那都是子弹的事了，咱也管不了了。

这个特点是速度快，快通常挺好，但是万一遇到点什么岔子，小到踢断网线，中到停电淹水，大到火山爆发。借用周星驰电影里的一句话：“世事难预料”，可能有些情况还能接受，但是如果是安全性要求很高的应用场景，那就不能接受了。

所以，安全性要求高的解决方案是：执行完操作，立刻返回getLastError，如果出错，一般都会抛出一个可捕获的异常，好及时处理，如果成功了，会给出额外的信息作为反馈。

但这样一来，性能方面肯定就不如之前了，因为用户会额外等待数据库做出响应。

至于具体是对安全性要求高，还是对性能要求高，要视具体情况而定。