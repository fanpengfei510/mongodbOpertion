> 将json格式导入到mongodb库  
``` javascript
cat data.json | mongoimport -c collectionName  
``` 
> MongoDB数据库备份
``` javascript
mongodump -h dbhost -d dbname -o dbdirectory
参数说明：
    -h： MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
    -d： 需要备份的数据库实例，例如：test
    -o： 备份的数据存放位置，例如：/home/mongodump/，当然该目录需要提前建立，这个目录里面存放该数据库实例的备份数据。
```  
> MongoDB数据库恢复
``` javascript
mongorestore -h dbhost -d dbname --dir dbdirectory
参数或名：
    -h： MongoDB所在服务器地址
    -d： 需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2
    --dir： 备份数据所在位置，例如：/home/mongodump/itcast/
    --drop： 恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用！
```

> ---
## 基本查询与操作
``` javascript
// 只返回_id与age与name字段
db.numbers.find({},{age:1,name:1})     

// 设置索引
db.numbers.createIndex({num:1}) 

// 查看索引
db.numbers.getIndexes();    

// 显示db的统计数据
db.stats()    

// 显示集合的统计数据
db.numbers.stats()    

// 查看集合的方法列表
db.numbers.help()     

// .get为方法关键词，然后按两下tab键，提示所有含有get方法的列表
db.number.get       

//创建集合
db.createCollection("users")      

//创建集合，并分配存储空间
db.createCollection("users",{size:20000})     

// 修改集合名称
db.numbers.renameCollection("number")     

// 固定显示条数，返回的是对象
db.numbers.find().limit(1)        

// sort对象，1代表从小到大排序，-1相反排序
db.numbers.find().sort({'age':1}).limit(5)    

// skip代表跳过多少条
db.numbers.find().skip(5).limit(5)    

// 可使用正则来查询
db.numbers.find({name:/^fan/})   

// pretty() 查询全部数据并格式化
db.demo.find().pretty()  

// explain是一个调试和优化查询的工具
db.numbers.find({ num : {$gt:995} }).explain('executionStats');   
```
 

## 逻辑查询操作符
> $and 两者必须匹配查询
``` javascript
db.demo.find({
  $and : [
    {name : "zhangsan"},
    {age : 25}
  ]
})
```
> $or 或者匹配
``` javascript
db.demo.find({
  $or : [
    {name : 'zhangsan'},
    {name : '李四'}
  ]
})

// $not，不匹配结果，查询age不匹配小于等于28的值，返回的是大于28的值
db.tableName.find({age : {$not : {$lte:28}}})                           
```

> 更新或者添加字段，第一个对象为原始数据，第二个对象为要修改成的数据  
> $set：如果name存在则重写name内容为zhangsan111，如果name不存在，则创建name
``` javascript
db.demo.update(    
  { name : 'zhangsan' },
  { $set : { name : 'zhangsan111' }}
)
```
> 替换，匹配第一个对象的数据，全部抹掉替换成第二个对象的内容
``` javascript
db.demo.update(
  { name : 'zhangsan' },
  { name: 'zhangsan111' }
)
```
> $unset，删除age字段
``` javascript
db.demo.update( 
  { name : 'zhangsan' },
  { $unset : { age :1 } }
)
```
  
> 添加复杂数据
``` javascript
db.demo.update(
  { name : 'zhangsan' },
  {
    $set : {
      favorites : {
        cities : ['A','B','C'],
        movies : ['e','f','g']
      }
    }
  }
)
```
> 高级更新，$addToSet添加一条数据，是唯一的，防止重复的数据
``` javascript
db.demo.update(
  { name : 'zhangsan' },
  {
    $addToSet : {
      "favorites.cities" : 'a'
    }
  },
  false,   // 控制是否允许upsert
  true     // 是否多个更新
)
```
> 删除数据
``` javascript
db.demo.remove({name:'zhangsan'})
```
> 删除数据库
``` javascript
db.dropDatabase();
``` 
> 删除集合及其附带的索引数据
``` javascript
db.demo.drop()
```
> shell的帮助信息
``` javascript
help
```
> 提示如何使用函数
``` javascript
db.help()
```

> 显示数据总计
``` javascript
for(var i=0; i<1000; i++){
  db.numbers.save({num:i})
}
db.numbers.find().count();
db.numbers.count(); // 同上

db.numbers.find() // 数据过大时，只显示前20条
it                // 查询后20条数据，再次输入依然查询后20条数据，以此类推
```
## 比较查询操作符
``` javascript
// $gt，查询大于995的数据
db.numbers.find({ num : {$gt:995}});  

// $lt，查询小于10的数据
db.numbers.find({ num : {$lt:10}})

// 查询大于20，小于30的数据
db.numbers.find({ num : {$gt:20,$lt:30}})

// $lte，查询小于并等于21
db.numbers.find({ num : {$lte:21}})

// $gte，查询大于并等于21
db.numbers.find({ num : {$gte:21}})

// $ne，查询不等于1的数据
db.numbers.find({ num : {$ne:1}})

// $ne，查询不匹配参数条件的数据
db.numbers.find({tag : {$ne : 'mac'}})

// $in，传入数组，匹配user_id包含数组里的值
db.numbers.find({ 
  user_id : {$in : [ObjectId('123'),ObjectId('123')]}
})

// $nin，传入数组，匹配datails.color不包含数组里的值
db.numbers.find({ 'datails.color' :              
  {$nin : ['红色','黑色']}
})
     
```

> ---
## 数组查询操作符
``` javascript
// $all 匹配数组里所有元素
db.numbers.find({tag : {$all : ['tools','mac']}})

// $elemMatch，查询数组里匹配的条件，ancestors是一个数组
db.categories.find({ancestors : {$elemMatch : {name:'Home01'}}})

// $size，匹配tags数组大小等于1的数据
db.categories.find({tags : {$size : 1}})

// $push：在数组后插入数据
db.products.update({sku:9094},{$push : {tag : 'push'}})

// $push与$each：同时使用，添加多个值到tag里
db.products.update({sku:9094},{$push : { tag : {$each : ['a','b','c']}}})   

db.products.update({sku:9094},{     
  $push : {
    tag : {
      $each : ['d','e'],
      // $slice：如果传入-4，则保留数组后四位，如果是4整数，保留前4位，如果是0，则数组为空
      $slice : -4   
    }
  }
})
db.products.update({sku:9094},{
  $push : {
    tagArr : {
      $each : [{id : 101112}],    // 插入数组数据
      $slice : 2,                 // 截取前2个数据
      $sort : {                   // 在对数据进行排序
        id : -1
      }
    }
  }
})

db.products.update({sku:9094},{
  // $addToSet：指挥添加数组里没有的值，只有f会存入
  $addToSet : {                   
    tag : { 
      // $each：只能和$addToSet、$push操作一起使用
      $each : ['c','d','e','f']   
    }
  }
})

db.products.update({sku:9094},{
  // $pop：删除数组元素，-1代表第一个元素，1代表最后一个元素
  $pop : {tag : 1}                
})

db.products.update({sku:9094},{
  // $pull：删除数组里的e元素
  $pull : {tag : 'e'}             
})

db.products.update({sku:9094},{
  // $pull：删除数组大于100的元素
  $pull : {tag : {$gt : 100}}      
})

db.products.update({sku:9094},{
  // $pullAll：删除指定数组元素
  $pullAll : {tag : ['a','b','c']}  
})

// $addToSet 加一个值到数组内，而且只有当这个值不在数组内才增加。
db.numbers.update({},{
  $addToSet:{
    tags : "addtoset"
  }
},{multi:true})  // multi：所有tags数据增加addtoset

db.numbers.update({sku:9092},{
  $addToSet : {
    tags : "addtoset"
  }
},{upsert : true})  // upsert：查找文档，然后更新，如果没有查找到文档，则创建一个新的，只插入tags数据

// $ 对数组里的值进行匹配和更新
db.products.update(           // 更新数组指定的对象数据
  {"dataArr.key" : 'home'},   // 通过数组.对象里的key进行匹配
  {$set : {
    "dataArr.$.updateKey" : 4}  // 通过数组.$.要更新的key，要更改的值
  }
)
```

## 元查询操作符
``` javascript
// $exists，检测details.color字段是或否存在
db.tableName.find({'details.color' : {$exists : true}})
```

> ---   

## 评价查询操作符
> $where，js表达式查询
``` javascript
db.categories.find({
  '$where' : 'this.details.weight > 100'
})
```
> ---
## 字段更新操作符
``` javascript
// $inc：total_reviews增加值1，-1代表减去值
db.products.update({sku:9092},{$inc : {total_reviews:1}})               

// $rename：修改key名，将tag修改为tags，或者{'obj.arr':'obj.arrs'}
db.products.update({sku:9094},{$rename : {'tag':'tags'}})   

// 未解
db.products.update({sku:9094},{
  $inc : {
    total : 1
  },
  $setOnInsert : {
    state : "abcdef"
  }
},{upsert : true})
```
            
> ---  
# 聚合管道
> $project：输出指定的文档字段，不显示其他字段  
> $group：根据指定的key进行分组  
``` javascript
  $addToSet：为组里唯一的值创建一个数组  
  $first：组里的第一个值，只有前缀$sort才有意义  
  $last：组里最后一个值，只有前缀$sort才有意义  
  $max：组里某个字段最大是  
  $min：组里某个字段最小值  
  $avg：某个字段的平均值  
  $push：返回组内所有值得数据，不去除重复值  
  $sum：求组内所有值得和
```
> $match：选择要查询的数据，类似于find({name:"abc"})  
> $limit：限制显示多少条数据  
> $skip：跳过多少条数据  
> $unwind：扩展数据，为每个数据入口生成一个输出文档  
> $sort：数据排序  
> $geoNear：选择某个地理位置附近的文档  
> $out：把管道的结果写入某个集合  
> $redact：控制特定数据的访问
``` javascript
await Products.aggregate([
  {
    $group : {
      _id : '$primary_category',    // 将_id相同的进行数据分组
      count : { $sum : 1 }          // $sum：总共有几条数据
    }
  }
],(err,data)=>{
  ctx.body = data       // [{"_id":"6a5b1476238d3b4dd5000048","count":2}]
})

// ---------------------------------------

await Products.aggregate([
  {
    $group : {
      _id : '$main_cat_id',
      average_revies : {$avg : '$average_revies'},  // $avg：字段平均值
      count : {$sum : 1}
    }
  }
],function(err,data){
  ctx.body = data;      //[{"_id":null,"average_revies":29.5,"count":2}]
})

// ---------------------------------------

await Products.aggregate([
  {
    $group : {
      _id : '$main_cat_id',
      average_revies : {$avg : '$average_revies'},
      count : {$sum : 1}
    }
  },
  {
    $out : 'subcate'        // $out：将结果插入到一个新的集合里，subcate为集合名
  }
],function(err,data){
  // ctx.body = data;
})

// ---------------------------------------

db.cateType.aggregate([
  {
    $project : { poiId : 1 }
  },{
    $unwind : '$poiId'        // 将_id设置成$poiId的值，而不是自动生成ObjectId
  },{
    $group : { _id : '$poiId',count : {$sum : 1} }
  },{
    $out : 'subcate2'         // 然后创建一个新的集合，只有创建新集合$unwind才生效
  }
])
```

# 重塑文档
> 字符串函数
 * $concat：连接多个字符串并合并成一个字符串
 * $strcasecmp：大小写敏感的比较，返回数字
 * $substr：获取字符串的子串
 * $toLower：转换为小写字符串
 * $toUpper：转换为大写字符串
``` javascript
// 修改返回key的命名，并将name的值全部修改为大写
db.demo.aggregate([
  {
    $match : { name : 'zhangsan' }
  },
  {
    $project : {
      userName : {
        $concat : [ '$name',' ','$sex' ]
      },
      firstInitial : {
        $substr : [ '$name',0,1 ]
      },
      usernameUpperCase : {
        $toUpper : '$name'
      }
    }
  }
])

/*
  { 
    "_id" : ObjectId("5c5fe94f640c81830d5ed078"), 
    "userName" : "zhangsan 男", 
    "firstInitial" : "z", 
    "usernameUpperCase" : "ZHANGSAN" 
  }
*/
```
 
# 算数运算函数
> $add：求和  
> $divide：除法  
> $mod：求余  
> $multiply：乘积  
> $subtract：减法

# 日期函数
> $dayOfYear：一年365天中的某一天  
> $dayOfMonth：一月中的某一天  
> $dayOfWeek：一周中的某一天，1代表周日  
> $month：日期的月份，1~12  
> $year：日期的月份  
> $week：一年中的某一周，0~53  
> $hour：日期中的小事，0~23  
> $minute：日期中的分钟，0~59  
> $second：日期中的秒，0~59  
> $millisecond：日期中的毫秒，0~999

# 逻辑函数
> $and true：与操作，如果数组里所有的值为true，则返回true  
> $cmp：如果两个数相等就返回0  
> $cond if... then... else：条件逻辑  
> $eq：两个值是否相等  
> $gt：大于  
> $gte：大于并等于  
> $ifNull：把null值/表达式转换为特定的值  
> $lt：小于  
> $lte：小于并等于  
> $ne.：取反操作  
> $or：或，如果数组中有一个true，就返回true

# 集合操作符
> 允许我们比较两个数组的内容。  
> 使用集合操作符，可以比较两个数组、判断他们是否完全一样、是否有公共元素、是否有差别元素  
> $setEquals：如果两个集合的元素完全相等，返回true  
> $setIntersection：返回两个集合的公共元素  
> $setDifference：返回第一个结合中与第二个集合中不同的元素  
> $setUnion：合并集合  
> $setIsSubset：如果第二个集合为第一个集合的子集，返回true  
> $anyElementTrue：如果某个集合元素为true，返回true  
> $allElementsTrue true：如果所有集合元素都为true，返回true

``` javascript
db.products.aggregate([
  {
    $project : {
      productName : '$name',
      tags : 1,
      // 将abcd合并到所有tags数组里
      setUnion : { $setUnion : ['$tags',['abcd']] }
    }
  }
]).pretty()
```

# 其他函数
> $meta：  
> $size：返回数组大小  
> $map：对数组的每个成员应用表达式  
> $let：定义表达式内使用的变量  
> $literal：返回表达式的值，而不评估它

# 聚合管道选项
> 我们可以为aggregate()传递第二个参数来指定集合调用
> explain()：运行管道并且只返回管道处理详细信息
> allowDiskUse：使用磁盘存储数据，处理超大数据
> cursor：指定初始化处理的大小
```javascript
db.collectioin.aggregate(pipeline,
  {
    explain:true,
    allowDiskUse:true,
    cursor:{ batchSize:n }
  }
)

db.products.aggregate([
  {
    $project : {
      skus : '$sku'
    }
  }
],{explain : true})
```

> cursor.hasNext()：确定结果集是否包含下一个元素  
> cursor.next()：返回结果集的下一个文档  
> cursor.toArray()：已数组返回结果  
> cursor.forEach()：遍历结果集的每一行  
> cursor.map()：遍历结果集的每一行，返回一个结果数组  
> cursor.itcount()：返回结果数据(仅做测试)  
> cursor.pretty()：显示格式化结果的数组
``` javascript
// 返回布尔值
db.products.aggregate([
  {
    $project : {
      skus : '$sku'
    }
  }
],{cursor : {}}).hasNext()

// 查找满足total_reviews值为4的数量
db.products.count({total_reviews : 4})    

// State为key，返回不重复的值[ "STATE", "ERROR" ]
db.orders.distinct('State')     
```

# findAndModify 命令
> query：查询选择器，默认为{}  
> update：指定更新的文档，默认为{}  
> remove：布尔值，如为true，则犯规删除的对象，默认为flase  
> new：布尔值，如为true，则返回修改后的文档，默认为false，返回修改前的文档  
> sort：指定排序的方向，  
> fields：如果只要返回部分字段，就使用这个参数来指定，这在处理大文档是非常有用  
> upsert：布尔值，为true时，findAndModify作为upsert操作。如果文档不存在就创建，如果要返回新创建的文档，就需要指定{new : true}
