# Redis 字符串数据类型
- 查看所有的key: ``keys *``
- 普通设置： ``set key value``
- 设置并加过期时间： ``set key value EX 30``
  - 表示 30 秒后过期
- 获取数据： ``get key``
- 删除指定数据： ``del key``
- 删除全部数据: ``flushall`` 
- 查看类型： ``type key ``
- 设置过期时间: ``expire key 20``
```
var redis = require('redis'); 
var client = redis.createClient(6379, 'localhost'); client.set(age, '20'); 
//设置时间 
client.set('username', '李四','EX','5'); //5 秒 client.get('username', function(err, val){ 
    console.log(val); 
});
```

# Redis 列表
Redis 列表是简单的字符串列表
- 列表右侧增加值： ``rpush key value``
- 列表左侧增加值： ``lpush key value`` 
- 右侧删除值： ``rpop key`` 
- 左侧删除值： ``lpop key`` 
- 获取数据： ``lrange key`` 
- 删除指定数据： ``del key`` 
- 删除全部数据: ``flushall`` 
- 查看类型： ``type key``
```
var redis = require('redis'); 
var client = redis.createClient(6379, 'localhost'); client.rpush('testLists', 'a'); 
client.rpush('testLists', 'b'); 
client.rpush('testLists', 'c'); 
client.lpush('testLists', 2); 
client.lpush('testLists', 1); 
client.lrange('testLists',0, -1,(err,lists)=>{
    if(err){ 
        console.log(err); return; 
    }
    console.log(lists) 
})
```

# Redis 集合
Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数 据。**它和列表的最主要区别就是没法增加重复值**
- 给集合增数据： ``sadd key value`` 
- 删除集合中的一个值： ``srem key value`` 
- 获取数据： ``smembers key`` 
- 删除指定数据： ``del key`` 
- 删除全部数据: ``flushall``
```
var redis = require('redis'); 
var client = redis.createClient(6379, 'localhost'); client.sadd('testSet', 1); 
client.sadd('testSet', 'a'); 
client.sadd('testSet', 'bb'); 
client.smembers('testSet', function(err, v){ console.log(v); })
```

# Redis 哈希
Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。
- 设置值 hmset ： ``hmset zhangsan name "张三" age 20 sex “男”`` 
- 设置值 hset ： ``hset zhangsan name "张三"``
- 获取数据： ``hgetall key``
- 删除指定数据： ``del key`` 
- 删除全部数据: ``flushall``
```
var redis = require('redis'); 
var client = redis.createClient(6379, 'localhost'); client.hset('userinfo',"username", "zhangsan"); client.hmset('userinfo',"username","张三","age", "20","sex","男"); 
client.hgetall('userinfo',function(err,val){ console.log(val); })
```
# Redis 订阅发布
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。Redis 客户端可以订阅任意数量的频道。
## 发布:
``client.publish('testPublish', 'message from publish.js');``
## 订阅
```
client.subscribe('testPublish'); 
client.on('message', function(channel, msg){
     console.log('client.on message, channel:', channel, ' message:', msg); 
});
```