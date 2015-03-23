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

####$out####
创建指定副本集合

1.例子

       { "_id" : 8751, "title" : "The Banquet", "author" : "Dante", "copies" : 2 }

       { "_id" : 8752, "title" : "Divine Comedy", "author" : "Dante", "copies" : 1 }

       { "_id" : 8645, "title" : "Eclogues", "author" : "Dante", "copies" : 2 }

       { "_id" : 7000, "title" : "The Odyssey", "author" : "Homer", "copies" : 10 }

       { "_id" : 7020, "title" : "Iliad", "author" : "Homer", "copies" : 10 }

对其按author分组，然后out一个新集合authors

      db.books.aggregate( [
                      { $group : { _id : "$author", books: { $push: "$title" } } },
                      { $out : "authors" }
      ] )

结果

     { "_id" : "Homer", "books" : [ "The Odyssey", "Iliad" ] }
     { "_id" : "Dante", "books" : [ "The Banquet", "Divine Comedy", "Eclogues" ] }

我们现在看到的只是数据映射，不是实体文档，

但是在authors里，有映射副本存成的文档。

也就是说 $out  可以创建新的集合，存储聚合后的文档映射。