###聚合管道###
> 注明：本篇的所有例子来自MongoDB官网示例，本篇底部可连接官网。

####功能####

聚合管道的功能简单来说就分两种：
* 对文档进行“过滤”，也就是筛选出符合条件的文档;
* 对文档进行“变换”，也就是改变文档的输出形式。

####$project####
1.我们有这样的数据

      {
         "_id" : 1,
         title: "abc123",
         isbn: "0001122223334",>
         author: { last: "zzz", first: "aaa" },
         copies: 5
      }

现在使用project来变换输出
    
     db.books.aggregate( 
         [
              { $project : { title : 1 , author : 1 } } 
         ]
     )

可以得

     { 
        "_id" : 1,
        "title" : "abc123", 
        "author" : { "last" : "zzz", "first" : "aaa" } 
     }

在$project里，我们指明（筛选）了要显示的数据，title和author，_id是自带的，可以用 _id：0 来将其过滤掉

2.我们现在有基础数据
 
     {
         "_id" : 1,
         title: "abc123",
         isbn: "0001122223334",
         author: { last: "zzz", first: "aaa" },
         copies: 5
     }

但是我们需要变换他的输出形式，我们就可以这样
 
    db.books.aggregate(
     [
       {
          $project: {
             title: 1,
             isbn: {
                prefix: { $substr: [ "$isbn", 0, 3 ] },
                group: { $substr: [ "$isbn", 3, 2 ] },
                publisher: { $substr: [ "$isbn", 5, 4 ] },
                title: { $substr: [ "$isbn", 9, 3 ] },
                checkDigit: { $substr: [ "$isbn", 12, 1] }
             },
             lastName: "$author.last",
             copiesSold: "$copies"
          }
       }
     ]
     )

在isbn内部的键 prefix,group,publisher,title，checkDigit,外部的lastName,copiesSold都是我们自己定义的。
$substr取字串，$isbn是字串键名，第二参数是字串起始位置，第三参数是取几个。

最后结果

     {
      "_id" : 1,
      "title" : "abc123",
      "isbn" : {
          "prefix" : "000",
          "group" : "11",
          "publisher" : "2222",
          "title" : "333",
          "checkDigit" : "4"
      },
      "lastName" : "zzz",
      "copiesSold" : 5
     }

数据源没有变，但是我们改变的数据显示的方式。

####$match####
过滤数据，过滤完的数据，接下来用作其他用。
（水龙头上的过滤器，过滤干净的水，接下来淘米，煮饭都可以，貌似扯远了。。。回！）

1.例子

      db.articles.aggregate(
         [
              { $match : { author : "dave" } }  
         ]
      );

过滤条件为键 author 值为 dave

结果为

     {
      "result" : [
                   {
                     "_id" : ObjectId("512bc95fe835e68f199c8686"),
                     "author": "dave",
                     "score" : 80
                   },
                   { "_id" : ObjectId("512bc962e835e68f199c8687"),
                      "author" : "dave",
                      "score" : 85
                   }
                ],
      "ok" : 1
     }

2.再看一例

     db.articles.aggregate(    
     [                   
                   { $match : { score : { $gt : 70, $lte : 90 } } },      
                   { $group: { _id: null, count: { $sum: 1 } } }
     ] 
     );

这次有两步：
* 第一步，过滤 键 score 值 大于70 且 小于等于90 的文档，
* 再用group 对文档用 count 统计，统计方式 $sum 求和，步长为1。

> 因为group操作必须有个_id,所以给其置null。

结果为

     {
      "result" : [
                   {
                     "_id" : null,
                     "count" : 3
                   }
                 ],
      "ok" : 1 
     }

####$cond####
判断用的，可以跟if then 语句

1.举例

    { "_id" : 1, "item" : "abc1", qty: 300 }
    
    { "_id" : 2, "item" : "abc2", qty: 200 }
    
    { "_id" : 3, "item" : "xyz1", qty: 250 }


现在我们想根据qty的值来生成新的数据（值）

    db.inventory.aggregate( 
    [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
               }
           }
      }
     ]
     )

结果为

     { "_id" : 1, "item" : "abc1", "discount" : 30 }
     { "_id" : 2, "item" : "abc2", "discount" : 20 }
     { "_id" : 3, "item" : "xyz1", "discount" : 30 }

可以发现，discount是我们新的键，它根据cond的if判断后，分别被赋上了相应的值（then和else可以省略）

####$limit####
限制个数

1.例子

    db.article.aggregate(
        { $limit : 5 }
    );

值得注意的是，当聚合操作中同时出现sort和limit，
sort只会对通过limit的数据排序，内存中也仅会存储通过limit的数据。

####$skip####
略过N个

1.例子

    db.article.aggregate(
       { $skip : 5 }
    );

####$unwind####
拆解数组集合

1.例子

     { 
          "_id" : 1, 
          "item" : "ABC1", 
          sizes: [ "S", "M", "L"] 
     }

现在对sizes进行拆解

     db.inventory.aggregate( 
        [ 
            { $unwind : "$sizes" }
        ] 
     )

结果

     { "_id" : 1, "item" : "ABC1", "sizes" : "S" }
     { "_id" : 1, "item" : "ABC1", "sizes" : "M" }
     { "_id" : 1, "item" : "ABC1", "sizes" : "L" }

我们可以看到sizes里每一个数据被拆解到每一个文档里了，除了sizes 的值不同外，其他相同。

$unwind与$group组合可以实现distinct

####$group####
先分组，再合并

1.例子

    { "_id" : { "month" : 3, "day" : 15, "year" : 2014 }, 
          "totalPrice" : 50, "averageQuantity" : 10, "count" : 1 }

    { "_id" : { "month" : 4, "day" : 4, "year" : 2014 }, 
          "totalPrice" : 200, "averageQuantity" : 15, "count" : 2 }

    { "_id" : { "month" : 3, "day" : 1, "year" : 2014 }, 
          "totalPrice" : 40, "averageQuantity" : 1.5, "count" : 2 }

_id 为分组依据，_id 为null，及不分组，直接合并。

合并依据：
*  键 totalPrice 保存 键 price 和 键 quantity 值 的乘积 的和
*  键averageQuantity 保存 键 quantity 的值的平均值
*  键 count 作统计

    db.sales.aggregate(
     [
         {
           $group : {
             _id : null,
             totalPrice: { $sum: { $multiply: [ "$price", "$quantity" ] } },
             averageQuantity: { $avg: "$quantity" },
             count: { $sum: 1 }
           }
         }
     ]
    )

结果
    
     { "_id" : null, "totalPrice" : 290, "averageQuantity" : 8.6, "count" : 5 }

2.再看一例

     { "_id" : 1, "item" : "abc", "price" : 10, "quantity" : 2, 
                "date" : ISODate("2014-03-01T08:00:00Z") }
    
     { "_id" : 2, "item" : "jkl", "price" : 20, "quantity" : 1, 
                "date" : ISODate("2014-03-01T09:00:00Z") }
 
     { "_id" : 3, "item" : "xyz", "price" : 5, "quantity" : 10, 
                "date" : ISODate("2014-03-15T09:00:00Z") }
    
     { "_id" : 4, "item" : "xyz", "price" : 5, "quantity" : 20, 
                "date" : ISODate("2014-04-04T11:21:39.736Z") }

     { "_id" : 5, "item" : "abc", "price" : 10, "quantity" : 10, 
                "date" : ISODate("2014-04-04T21:23:13.331Z") }

咱们依据 键 item 分组

      db.sales.aggregate( [ { $group : { _id : "$item" } } ] )

结果

     { "_id" : "xyz" }
     { "_id" : "jkl" }
     { "_id" : "abc" }


####$sort####
排序

1.例子

    db.users.aggregate(
      [
         { $sort : { age : -1, posts: 1 } }
      ]
    )

对键age  顺序排序，对键 posts 逆序排序

