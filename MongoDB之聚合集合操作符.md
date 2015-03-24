####$setEquals####
检查是否有重复的值

判断条件
有重复值，返回true，否则，返回false

1.例子

      Example	 	                                            Result
      { $setEquals: [ [ "a", "b", "a" ], [ "b", "a" ] ] }	 	true
      { $setEquals: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }	 	false

2.例子

           { "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] }
           { "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] }
           { "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] }
           { "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] }
           { "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] }
           { "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] }
           { "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] }
           { "_id" : 8, "A" : [ ], "B" : [ ] }
           { "_id" : 9, "A" : [ ], "B" : [ "red" ] }

对键A和键B的值进行查重布尔判断

     db.experiments.aggregate(
        [
             { $project: { A: 1, B: 1, sameElements: { $setEquals: [ "$A", "$B" ] }, _id: 0 } }
        ]
     )

结果

        { "A" : [ "red", "blue" ], "B" : [ "red", "blue" ], "sameElements" : true }

        { "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ], "sameElements" : true }

        { "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ], "sameElements" : false }

        { "A" : [ "red", "blue" ], "B" : [ "green", "red" ], "sameElements" : false }

        { "A" : [ "red", "blue" ], "B" : [ ], "sameElements" : false }

        { "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ], "sameElements" : false }

        { "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ], "sameElements" : false }

        { "A" : [ ], "B" : [ ], "sameElements" : true }

        { "A" : [ ], "B" : [ "red" ], "sameElements" : false }

注意，空值和空值可以视为值相同

####$setIntersection####
获取交集，忽略重复，忽略顺序

判断条件：对一个或多个数组判断，返回依次出现在每个数组中的值，简单说就是取交集

1.例子

      Example	 	                                                Result
      { $setIntersection: [ [ "a", "b", "a" ], [ "b", "a" ] ] }	 	[ "b", "a" ]
      { $setIntersection: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }	 	[ ]

2.例子

      { "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] }
      { "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] }
      { "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] }
      { "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] }
      { "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] }
      { "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] }
      { "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] }
      { "_id" : 8, "A" : [ ], "B" : [ ] }
      { "_id" : 9, "A" : [ ], "B" : [ "red" ] }

对键A和键B 取交集

     db.experiments.aggregate(
       [
           { $project: { A: 1, B: 1, commonToBoth: { $setIntersection: [ "$A", "$B" ] }, _id: 0 } }
       ]
     )

结果

      { "A" : [ "red", "blue" ], "B" : [ "red", "blue" ], "commonToBoth" : [ "blue", "red" ] }
      { "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ], "commonToBoth" : [ "blue", "red" ] }
      { "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ], "commonToBoth" : [ "blue", "red" ] }
      { "A" : [ "red", "blue" ], "B" : [ "green", "red" ], "commonToBoth" : [ "red" ] }
      { "A" : [ "red", "blue" ], "B" : [ ], "commonToBoth" : [ ] }
      { "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ], "commonToBoth" : [ ] }
      { "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ], "commonToBoth" : [ ] }
      { "A" : [ ], "B" : [ ], "commonToBoth" : [ ] }
      { "A" : [ ], "B" : [ "red" ], "commonToBoth" : [ ] }

**注意**：
* $setIntersection操作忽略重复、忽略顺序
* 空集和任何相交为空集

####$setUnion####
取并集，忽略重复，忽略顺序

1.例子

      Example	 	                                                Result
      { $setUnion: [ [ "a", "b", "a" ], [ "b", "a" ] ] }	 	    [ "b", "a" ]
      { $setUnion: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }	 	        [ [ "a", "b" ], "b", "a" ]

2.例子

      { "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] }
      { "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] }
      { "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] }
      { "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] }
      { "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] }
      { "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] }
      { "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] }
      { "_id" : 8, "A" : [ ], "B" : [ ] }
      { "_id" : 9, "A" : [ ], "B" : [ "red" ] }

我们对键A和键B 取并集

      db.experiments.aggregate(
         [
            { $project: { A:1, B: 1, allValues: { $setUnion: [ "$A", "$B" ] }, _id: 0 } }
         ]
      )

结果

       { "A": [ "red", "blue" ], "B": [ "red", "blue" ], "allValues": [ "blue", "red" ] }
       { "A": [ "red", "blue" ], "B": [ "blue", "red", "blue" ], "allValues": [ "blue", "red" ] }
       { "A": [ "red", "blue" ], "B": [ "red", "blue", "green" ], "allValues": [ "blue", "red", "green" ] }
       { "A": [ "red", "blue" ], "B": [ "green", "red" ], "allValues": [ "blue", "red", "green" ] }
       { "A": [ "red", "blue" ], "B": [ ], "allValues": [ "blue", "red" ] }
       { "A": [ "red", "blue" ], "B": [ [ "red" ], [ "blue" ] ], "allValues": [ "blue", "red", [ "red" ], [ "blue" ] ] }
       { "A": [ "red", "blue" ], "B": [ [ "red", "blue" ] ], "allValues": [ "blue", "red", [ "red", "blue" ] ] }
       { "A": [ ], "B": [ ], "allValues": [ ] }
       { "A": [ ], "B": [ "red" ], "allValues": [ "red" ] }

**注意**：
* $setUnion忽略重复、忽略顺序
* 空集和其他集合去并集为其他集合

####$setDifference####
找不同，忽略重复，忽略顺序
判断依据：
* 只看后者比前者多了什么
* 忽略后者比前者少了什么

1.例子

       Example	 	                                                Result
       { $setDifference: [ [ "a", "b", "a" ], [ "b", "a" ] ] }	 	[ ]
       { $setDifference: [ [ "a", "b" ], [ [ "a", "b" ] ] ] }	 	[ "a", "b" ]

**

解释：
* 忽略重复、顺序后，后者跟前者一样，所以空集
* 忽略重复、顺序后，后者比前者多了一个子集合，所以是那个多出来的子集合

2.例子

      { "_id" : 1, "A" : [ "red", "blue" ], "B" : [ "red", "blue" ] }
      { "_id" : 2, "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ] }
      { "_id" : 3, "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ] }
      { "_id" : 4, "A" : [ "red", "blue" ], "B" : [ "green", "red" ] }
      { "_id" : 5, "A" : [ "red", "blue" ], "B" : [ ] }
      { "_id" : 6, "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ] }
      { "_id" : 7, "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ] }
      { "_id" : 8, "A" : [ ], "B" : [ ] }
      { "_id" : 9, "A" : [ ], "B" : [ "red" ] }

我们用键B比较键A，找出多出来的部分 

     db.experiments.aggregate(
        [
          { $project: { A: 1, B: 1, inBOnly: { $setDifference: [ "$B", "$A" ] }, _id: 0 } }
        ]
     )

结果
     { "A" : [ "red", "blue" ], "B" : [ "red", "blue" ], "inBOnly" : [ ] }
     { "A" : [ "red", "blue" ], "B" : [ "blue", "red", "blue" ], "inBOnly" : [ ] }
     { "A" : [ "red", "blue" ], "B" : [ "red", "blue", "green" ], "inBOnly" : [ "green" ] }
     { "A" : [ "red", "blue" ], "B" : [ "green", "red" ], "inBOnly" : [ "green" ] }
     { "A" : [ "red", "blue" ], "B" : [ ], "inBOnly" : [ ] }
     { "A" : [ "red", "blue" ], "B" : [ [ "red" ], [ "blue" ] ], "inBOnly" : [ [ "red" ], [ "blue" ] ] }
     { "A" : [ "red", "blue" ], "B" : [ [ "red", "blue" ] ], "inBOnly" : [ [ "red", "blue" ] ] }
     { "A" : [ ], "B" : [ ], "inBOnly" : [ ] }
     { "A" : [ ], "B" : [ "red" ], "inBOnly" : [ "red" ] }

**注意**
* 忽略顺序、重复
* 我们只关心多了什么，不关心少了什么

