布尔操作符
> 注明：本篇所有示例均来自MongoDB官方文档提供示例

####$and####
判断依据：
* true：判断一个或多个表达式，如果所有表达式全为真 或 没有表达式，返回 true
* false：其他情况返回 false

1.例子

        Example	 	                                   Result
        { $and: [ 1, "green" ] }	 	               true                   
        { $and: [ ] }	 	                           true                   
        { $and: [ [ null ], [ false ], [ 0 ] ] }       true                   
        { $and: [ null, true ] }	 	               false                  
        { $and: [ 0, true ] }	 	                   false                  


**注意**
> 如果发现有 null、0、undefined，$and就会判断为false，但如果他们在数组中，则反之。

解释：
* 有多个表达式
* 无表达式
* 有多个表达式（虽然有0和null，但是0和null在数组中，视为表达式）
* 有多个表达式，但其中有null
* 有多个表达式，但其中还有0


2.例子

      { "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 }

      { "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 }

      { "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 }

      { "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 }

      { "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }


按要求对 键qty值大于100 和 键qty值小于250 进行$and 操作，再赋给新键result

     db.inventory.aggregate([{
       $project:
          {
            item: 1,
            qty: 1,
            result: { $and: [ { $gt: [ "$qty", 100 ] }, { $lt: [ "$qty", 250 ] } ] }
          }
       }])

结果

     { "_id" : 1, "item" : "abc1", "result" : false }
     { "_id" : 2, "item" : "abc2", "result" : true }
     { "_id" : 3, "item" : "xyz1", "result" : false }
     { "_id" : 4, "item" : "VWZ1", "result" : false }
     { "_id" : 5, "item" : "VWZ2", "result" : true }

因为200 和 180 是介于100和250之间的，所以他们的键result的值为true

####$or####
判断依据
* true：对于一个或多个表达式，只要任意表达式的值为true，返回true
* false：其他情况，返回false

1.例子

      Example	 	                     Result
      { $or: [ true, false ] }	 	      true
      { $or: [ [ false ], false ] }	 	  true
      { $or: [ null, 0, undefined ] }	  false
      { $or: [ ] }	 	                  false

**注意**
> 如果发现有 null、0、undefined，$and就会判断为false，但如果他们在数组中，则反之。

解释
* 有多个表达式
* 有多个表达式
* 有三个表达式，但其中有 null、0、undefined
* 没有表达式

2.例子

      { "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 }

      { "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 }

      { "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 }

      { "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 }

      { "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }

按要求对 键qty值大于250 或 键qty值小于200 进行$or 操作，再赋给新键result

       db.inventory.aggregate([{
         $project:
            {
               item: 1,
               result: { $or: [ { $gt: [ "$qty", 250 ] }, { $lt: [ "$qty", 200 ] } ] }
            }
       }])

结果

     { "_id" : 1, "item" : "abc1", "result" : true }
     { "_id" : 2, "item" : "abc2", "result" : false }
     { "_id" : 3, "item" : "xyz1", "result" : false }
     { "_id" : 4, "item" : "VWZ1", "result" : true }
     { "_id" : 5, "item" : "VWZ2", "result" : true }

因为200 和 250 是不再范围之外的，所以他们的键result的值为false

####$not####
判断布尔值，然后置反

     Example	 	                Result
     { $not: [ true ] }	 	        false
     { $not: [ [ false ] ] }	 	false
     { $not: [ false ] }	      	true
     { $not: [ null ] }	 	        true
     { $not: [ 0 ] }	 	        true

**注意**
> 如果发现有 null、0、undefined，$and就会判断为false，但如果他们在数组中，则反之。

解释
* 一个表达式，值为true，置反为false
* 一个表达式，[false]布尔值为true，置反为false
* 一个表达式，值为false，置反为true
* 一个表达式，值为false，置反为true
* 一个表达式，值为false，置反为true

2.例子

     { "_id" : 1, "item" : "abc1", description: "product 1", qty: 300 }

     { "_id" : 2, "item" : "abc2", description: "product 2", qty: 200 }

     { "_id" : 3, "item" : "xyz1", description: "product 3", qty: 250 }

     { "_id" : 4, "item" : "VWZ1", description: "product 4", qty: 300 }

     { "_id" : 5, "item" : "VWZ2", description: "product 5", qty: 180 }


对大于250的值置反

    db.inventory.aggregate([{
       $project:
          {
            item: 1,
            result: { $not: [ { $gt: [ "$qty", 250 ] } ] }
          }
     }])

结果

    { "_id" : 1, "item" : "abc1", "result" : false }
    { "_id" : 2, "item" : "abc2", "result" : true }
    { "_id" : 3, "item" : "xyz1", "result" : true }
    { "_id" : 4, "item" : "VWZ1", "result" : false }
    { "_id" : 5, "item" : "VWZ2", "result" : true }

300大于250，置反后result为false