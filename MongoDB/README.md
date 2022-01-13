# MongoDB
- MongoDB结构
  - 数据库
  - 集合
  - 文件

## 基本命令
| 命令                              | 描述                                                                                                     |
| :-------------------------------- | :------------------------------------------------------------------------------------------------------- |
| ``show dbs``                      | 显示已有数据库                                                                                           |
| ``use admin``                     | 进入数据，也可以理解成为使用数据库。如果库不存在会自动新建，默认为空。此时show不会显示                   |
| ``show collections``              | 显示数据库中的集合                                                                                       |
| ``db``                            | 显示当前位置                                                                                             |
| ``db.集合.insert( )``             | 新建数据集合和插入文件（数据），当集合没有时，这时候就可以新建一个集合，并向里边插入数据。               |
| ``db.集合.find( )``               | 查询所有数据                                                                                             |
| ``db.集合.findOne( )``            | 查询第一个文件数据                                                                                       |
| ``db.集合.update({查询},{修改})`` | 修改文件数据，第一个是查询条件，第二个是要修改成的值。这里注意的是可以多加文件数据项的([详情](##update)) |
| ``db.集合.remove(条件)``          | 删除文件数据，注意的是要跟一个条件                                                                       |
| ``db.集合.drop( )``               | 删除整个集合                                                                                             |
| ``db.dropDatabase( )``            | 删除整个数据库，在删除库时，一定要先进入数据库，然后再删除                                               |  |

## js文件写mongo命令
```
var jsonDdatabase={"loginUnser":"userName","loginTime":Date.parse(new Date())};
var db = connect('log');   //链接数据库
db.login.insert(jsonDdatabase);  //插入数据
print('[demo]log  print success');  //没有错误显示成功
```
### 批量插入
批量数据插入是以数组的方式进行的,注意一次插入不要超过48M
```
db.test.insert([
    {"_id":1},
    {"_id":2},
    {"_id":3}
])
```
### hasNext循环结果
```
var db = connect("log")
var result = db.workmate.find()
//利用游标的hasNext()进行循环输出结果。
while(result.hasNext()){
    printjson(result.next())
}
```
### forEach循环
```
var db = connect("log")
var result = db.workmate.find() 
//利用游标的hasNext()进行循环输出结果。
result.forEach(function(result){
    printjson(result)
})
```
## update
| 命令          | demo                                                                                                          | 描述                                                                                           |
| :------------ | :------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------- |
| ``$set``      | ``db.集合.update({查询},{"$set":数据})``                                                                      | 用来修改一个指定的键值(key)，嵌套内容使用``"a.b"``的形式                                       |
| ``$unset``    | ``db.集合.update({查询},{"$unset":{"key":''}})``                                                              | 用来删除一个key值和键                                                                          |
| ``$inc``      | ``db.集合.update({查询},{"$inc":{"age":-2}})``                                                                | 用来对value值的修改，但是修改的必须是数字                                                      |
| ``multi``     | ``db.集合.update({查询},{"$set":数据,{multi:true}})``                                                         | 用来修改每条数据                                                                               |
| ``upsert``    | ``db.集合.update({查询},{"$set":数据,{upsert:true}})``                                                        | 找不到值的情况下，直接插入这条数据                                                             |
| ``$push``     | ``db.集合.update({查询},{"$push":{数组对应的key:数据}})``or ``db.集合.update({查询},{"$push":{"a.b":数据}})`` | 追加数组中的值，也可以内嵌文档                                                                 |
| ``$ne``       | ``db.集合.update({key:{$ne:3},{"$set":{key:val}})``                                                           | 检查一个值是否存在，如果不存在再执行操作，存在就不执行                                         |
| ``$addToSet`` | ``db.集合.update({查询},{"$addToSet":{key:val}})``                                                            | 升级版的$ne                                                                                    |
| ``$each``     | ``db.集合.update({查询},{"$addToSet":{key:{$each:newArr}}})``                                                 | 可以传入一个数组，一次增加多个值进去，相当于批量操作                                           |
| ``$pop``      | ``db.集合.update({查询},{"$pop":{key:1}})``                                                                   | 删除数组值,只删除一次，并不是删除所有数组中的值。1：从数组末端进行删除 ;-1：从数组开端进行删除 |
| ``数组key.2`` | ``db.集合.update({查询},{"$set":{key.2:数据}})``                                                              | 数组定位修改                                                                                   |

## 状态返回
db.listCommands( )可以查看所有的Commad命令
### db.runCommand()数据库运行命令的执行器
```
db.workmate.update({sex:1},{$set:{money:1000}},false,true)
var resultMessage=db.runCommand({getLastError:1})
printjson(resultMessage);
```
- false：第一句末尾的false是upsert的简写，代表没有此条数据时不增加
- true：true是multi的简写，代表修改所有
- getLastError:1 :表示返回功能错误
- printjson：表示以json对象的格式输出到控制台。

### findAndModify查找并修改
配置它可以在修改后给我们返回修改的结果
```
var myModify={
    findAndModify:集合,
    query:{name:'name'},
    update:{$set:{age:18}},
    new:true    //更新完成，需要查看结果，如果为false不进行查看结果
}
var ResultMessage=db.runCommand(myModify);
printjson(ResultMessage)
```
- query：需要查询的条件/文档
- sort: 进行排序
- remove：[boolean]是否删除查找到的文档，值填写true，可以删除。
- new:[boolean]返回更新前的文档还是更新后的文档。
- fields：需要返回的字段
- upsert：没有这个值是否增加。

## find
参数
- query：这个就是查询条件，MongoDB默认的第一个参数。
- fields：（返回内容）查询出来后显示的结果样式，可以用true和false控制是否显示。
- limit：返回的数量，后边跟数字，控制每次查询返回的结果数量。
- skip:跳过多少个显示，和limit结合可以实现分页。
- sort：排序方式，从小到大排序使用1，从大到小排序使用-1。

### 简单查找
``db.集合.find({查询})``
### 筛选字段
```
db.集合.find(
    {查询},
    {key1:true,key2:false}
)
```
### 比较修饰符
- 小于($lt):英文全称less-than
- 小于等于($lte)：英文全称less-than-equal
- 大于($gt):英文全称greater-than
- 大于等于($gte):英文全称greater-than-equal
- 不等于($ne):英文全称not-equal
- 等于($eq)
- ...

### $in修饰符
用来查询多个键值的情况
```
db.集合.find(
    {age:{$in:[25,33]}},
    {key1:true,key2:false}
)
```
### $or修饰符
用来查找几个key值满足其中一个即可的情况
```
db.集合.find(
    {$or:[
        {age:{$gte:30}},
        {"sex":'男'}
    ]},
    {key1:true,key2:false}
)
```
### $and修饰符
用来查找几个key值全部满足的情况
```
db.集合.find(
    {$and:[
        {age:{$gte:30}},
        {"sex":'男'}
    ]},
    {key1:true,key2:false}
)
```
### $not修饰符
查询除条件之外的值
```
db.集合.find(
    {$not:[
        {age:{$gte:30}},
        {"sex":'男'}
    ]},
    {key1:true,key2:false}
)
```
### 数组查询
#### 在数组中查询一项
```
db.集合.find(
    {数组key:val},
    {name:1,数组key:1,age:1,_id:0} 
)
```
#### 数组多项查询
```
db.集合.find(
    {数组key:{$all:[val1,val2]}},
    {name:1,数组key:1,age:1,_id:0} 
)
```
#### 数组的或者查询
```
db.集合.find(
    {数组key:{$in:[val1,val2]}},
    {name:1,数组key:1,age:1,_id:0} 
)
```
#### 数组个数查询
```
db.集合.find(
    {数组key:{$size:5}},
    {name:1,数组key:1,age:1,_id:0} 
)
```
#### 显示选项
```
db.集合.find(
    {数组key:{$size:5}},
    {name:1,数组key:{$slice:-1},age:1,_id:0} 
)
```
### where修饰符
```
db.集合.find(
    {$where:"this.age>30"},
    {name:true,age:true}
)
```

## 索引
### 建立索引
``db.集合.ensureIndex({key:1})``
参数  
- background（Boolean默认false）：建索引过程会阻塞其他数据库操作，可以指定以后台方式创建
- unique（Boolean默认false）：建立的索引是否唯一
- name（String）：索引名称，如果未指定通过连接索引的字段名称和排序生成一个索引名称
- dropDups（Boolean默认false）：在建立唯一索引时是否删除重复记录
### 查看现有索引
``db.集合.getIndexes()``
### 复合索引
``db.集合.find({key1:val1,key2:val2});``
### 指定索引查询（hint）
``db.集合.find({{key1:val1,key2:val2}}).hint({key2:1});``
### 全文索引
#### 建立全文索引``db.集合.ensureIndex({key:'text'})``
text关键词来代表全文索引
- $text:表示要在全文索引中查东西。
- $search:后边跟查找的内容。
``db.集合.find({$text:{$search:"关键词"}})``
#### 查找多个词
``db.集合.find({$text:{$search:"关键词1 关键词2 关键词3"}})``
``-``表示希望不查找带有此关键词的数据
``db.集合.find({$text:{$search:"关键词1 关键词2 -关键词3"}})``
#### 转义符
如果关键词中文有空格需要使用
``db.集合.find({$text:{$search:"\"关键 词1\" 关键词2"}})``

## 聚合管道（Aggregation Pipeline）
使用聚合管道可以对集合中的文档进行变换和组合。表关联查询、数据的统计。
| 管道操作符   | 描述                                                  |
| :----------- | :---------------------------------------------------- |
| ``$project`` | 增加、删除、重命名字段                                |
| ``$match``   | 条件匹配。只满足条件的文档才能进入下一阶段            |
| ``$limit``   | 限制结果的数量                                        |
| ``$skip``    | 跳过文档的数量                                        |
| ``$sort``    | 条件排序                                              |
| ``$group``   | 条件组合结果  统计                                    |
| ``$lookup``  | $lookup 操作符 用以引入其它集合的数据  （表关联查询） |
例子：
```
// 将字段改为key1和key2两个，其他没有列出的都抛弃
db.order.aggregate([
    {
        $project:{ key1:1, key2:1 }
    }
])
// key2值大于等于90的筛选出啦
db.order.aggregate([
    {
        $project:{ key1:1, key2:1 }
    },
    {
        $match:{"key2":{$gte:90}}
    }
])
// 统计每个订单的订单数量，按照订单号分组
// $sum指定字段求和
db.order_item.aggregate([
    {
        $group: {_id: "$order_id", total: {$sum: "$num"}}
    }
])
// 排序
db.order.aggregate([
    {
        $project:{ key1:1, key2:1 }
    },
    {
        $match:{"key2":{$gte:90}}
    },
    {
        $sort:{"all_price":-1}
    }
])
// from 指定同一数据库中的集合以执行连接，要关联的集合（order_item）
// localField 指定从文档（order）输入到$lookup关联的字段，如果输入文档不包含则当null处理
// foreignField 指定from 集合（order_item）中需要关联的字段，如果集合中的文档不包含，则将值视为匹配目的
// as 为输出文档的新增值命名，如果输入文档中已经存在指定的名称，则覆盖现有字段
db.order.aggregate([
    {
      $lookup:
        {
          from: "order_item",
          localField: "order_id",
          foreignField: "order_id",
          as: "items"
        }
   }
])
```
## 管理
### 内置角色
- 数据库用户角色：read、readWrite；
- 数据库管理角色：dbAdmin、dbOwner、userAdmin;
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManage；
- 备份恢复角色：backup、restore；
- 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
- 超级用户角色：root
- 内部角色：__system

### 创建用户
首先需要进入admin库``use admin``
```
db.createUser({
    user:"user",
    pwd:"pwd",
    customData:{
        name:'name',
        email:'email',
        age:18,
    },
    roles:[
        {
            role:"readWrite",
            db:"company"
        },
        'read'
    ]
})
```
### 查找用户信息
``db.system.users.find()``

### 删除用户
``db.system.users.remove({user:"user"})``

### 建权
验证用户的用户名密码是否正确
``db.auth("user","pwd")``
如果正确返回1

### 启动建权
``mongod --auth``启动后，用户登录只能用用户名和密码进行登录，原来的mongo形式链接已经不起作

## 数据库备份恢复
如果相关[工具](https://www.mongodb.com/try/download/database-tools)不存在需要去下载并复制到bin目录下
### 备份
需要开启服务，打开命令提示符窗口，进入MongoDB安装目录的bin目录
``mongodump -h dbhost -d dbname -u username -p pwd -o dbdirectory``
- -h：MongoDB 所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
- -d：需要备份的数据库实例，例如：test
- -o：备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。

```
mongodump --host 127.0.0.1 --port 27017  --out E:/beifen/

```
### 恢复
``mongorestore -h <hostname><:port> -d dbname -u username -p pwd <path>``
- --host <:port>, -h <:port>：MongoDB所在服务器地址，默认为： localhost:27017
- --db , -d ：需要恢复的数据库实例，例如：test，当然这个名称也可以和备份时候的不一样，比如test2
- --drop：恢复的时候，先删除当前数据，然后恢复备份的数据。就是说，恢复后，备份后添加修改的数据都会被删除，慎用哦！
- <path>：mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。
你不能同时指定 <path> 和 --dir 选项，--dir也可以设置备份目录。
- --dir：指定备份的目录。你不能同时指定 <path> 和 --dir 选项。
```
mongorestore --host 127.0.0.1 --port 27017  E:/beifen/
```
