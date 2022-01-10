# mongoose
- 通过关系型数据库的思想来设计非关系型数据库
- 基于 mongodb 驱动，简化操作

## 基本使用方法
### 引入 mongoose 并连接数据库
```
const mongoose = require('mongoose');
mongoose.connect('mongodb://127.0.0.1/test'); 
// 如果有账户密码需要采用下面的连接方式：
mongoose.connect('mongodb://eggadmin:123456@127.0.0.1:27017/eggcms')
```
### 定义 Schema
数据库中的 Schema，为数据库对象的集合。schema 是 mongoose 里会用到的一种数据模式， 可以理解为表结构的定义；每个 schema 会映射到 mongodb 中的一个 collection，它不具备 操作数据库的能力
```
var UserSchema=mongoose.Schema({ 
    name: String, 
    age:Number, 
    status:'number' 
})
```
### 创建数据模型
定义好了 Schema，接下就是生成 Model。model 是由 schema 生成的模型，可以对数据库的 操作。
```
var User=mongoose.model('User', UserSchema);
```
mongoose.model 里面可以传入两个参数也可以传入三个参数  
``mongoose.model（参数 1:模型名称（首字母大写），参数 2:Schema，参数 3:数据库集合名 称）``  
如果传入 2 个参数的话:这个模型会和模型名称相同的复数的数据库建立连接：如通过下面 方法创建模型，那么这个模型将会操作 users 这个集合。  
如果传入 3 个参数的话:模型默认操作第三个参数定义的集合名称  
### 查找数据
```
User.find({},function(err,docs){ 
    if(err){
        console.log(err); return; 
    }
    console.log(docs); 
})
```
### 增加数据
var u=new User({ 
    //实例化模型 传入增加的数据 
    name:'lisi2222333', 
    age:20, 
    status:true 
})
u.save();
### 修改数据
```
User.updateOne({ name: 'lisi2222' }, { name: '哈哈哈' }, function(err, res) {
    if(err){
        console.log(err); 
        return; 
    }
    console.log('成功')
});
```
### 删除数据
```
User.deleteOne({ _id: '5b72ada84e284f0acc8d318a' }, function (err) {
    if (err) { 
        console.log(err); 
        return; 
    }
    // deleted at most one tank document 
    console.log('成功'); 
});
```
### 保存成功查找
```
var u=new User({ 
    name:'lisi2222333',
    age:20,
    status:true //类型转换 
})
u.save(function(err,docs){
    if(err){
        console.log(err);
        return;
    }
    //console.log(docs);
    User.find({},function(err,docs){
        if(err){
            console.log(err);
            return;
        }
        console.log(docs);
    )
});
```

## 预定义模式修饰符
mongoose 提供的预定义模式修饰符，可以对我们增加的数据进行一些格式化。
```
var UserSchema=mongoose.Schema({
    name:{ type:String, trim:true },
    age:Number,
    status:{ type:Number, default:1 }
})
```
## Mongoose Getters 与 Setters 自定义修饰符
除了 mongoose 内置的修饰符以外，我们还可以通过 set（建议使用） 修饰符在增加数据的 时候对数据进行格式化。也可以通过 get（不建议使用）在实例获取数据的时候对数据进行格式化。
```
var NewsSchema=mongoose.Schema({
    title:"string",
    author:String,
    pic:String,
    redirect:{
        type:String,
        set(url){
            if(!url) return url;
            if(url.indexOf('http://')!=0 && url.indexOf('https://')!=0){
                url = 'http://' + url;
            }
            return url;
        },
        get: function(url){
            if(!url) return url;
            if(url.indexOf('http://')!=0 && url.indexOf('https://')!=0){
                url = 'http://' + url;
            }
            return url;
        }
    },
    ...
```

## [Mongoose 内置 CURD](https://mongoosejs.com/docs/queries.html)
- Model.deleteMany()
- Model.deleteOne()
- Model.find()
- Model.findById()
- Model.findByIdAndDelete()
- Model.findByIdAndRemove()
- Model.findByIdAndUpdate()
- Model.findOne()
- Model.findOneAndDelete()
- Model.findOneAndRemove()
- Model.findOneAndUpdate()
- Model.replaceOne()
- Model.updateMany()
- Model.updateOne()

### 扩展 Mongoose CURD 方法
```
const UserSchema=mongoose.Schema({ 
    name:{ type:String },
    age:Number,
    status:{ type:Number, default:1 }
})
// 静态方法
UserSchema.statics.findByUid=function(uid,cb){
    this.find({"_id":uid},
    function(err,docs){
        cb(err,docs)
    })
}
// 实例方法 
UserSchema.methods.print = function(){
    console.log('这是一个实例方法');
    console.log(this);
};
module.exports=mongoose.model('User',UserSchema,'user');
```

## Mongoose 校验参数
- required : 表示这个数据必须传入 
- max: 用于 Number 类型数据，最大值 
- min: 用于 Number 类型数据，最小值 
- enum:枚举类型，要求数据必须满足枚举值 enum: ['0', '1', '2']
- match:增加的数据必须符合 match（正则）的规则 
- maxlength：最大值 
- minlength：最小值
```
const UserSchema = new mongoose.Schema({ 
    name:{ type:String, required: true, },
    age: { type: Number, // 是否必须的校验器 
    required: true, // 数字类型的最大值校验器 
    max: 120, // 数字类型的最小值校验器 
    min: 0 },
})
```

## Mongoose 自定义的验证器
```
const UserSchema = new mongoose.Schema({ 
    name:{ type:String, required: true, },
    desc: { 
        type: String, // 自定义的验证器，如果通过验证返回 true，没有通过则返回 false 
        validate: function(desc) { return desc.length >= 10; } 
    }
})
```
## 聚合管道 用法一样

## Mongoose populate 关联查询
```
const ArticleSchema = new Schema({
    title:{ 
        
        type: String, unique: true     
    },
    cid : { 
        
        type: Schema.Types.ObjectId,   
        ref:"ArticleCate"    
        //cid和 文章分类建立关系。   model    
    },   /*分类 id*/

    author_id:{        
        type: Schema.Types.ObjectId ,
        ref:"User"    //author_id和 用户表建立关系。   model
    },   /*用户的id*/
    author_name:{        
        type:String      
    },
    descripton:String,   
    content   : String
});
const ArticleCateSchema = new mongoose.Schema({
    title  : { type: String, unique: true },
    descripton:String,
    addtime:{
        type:Date       
    }
});
const UserSchema = new mongoose.Schema({
    username  : { type: String, unique: true },
    password:String,
    name:String,
    age:Number,
    sex:String,
    tel:Number,
    status:{
        type:Number,
        default:1
    }
});
// 关联查询
ArticleModel.find({}).populate('cid').populate('author_id').exec(function(err,docs){ 
    console.log(docs) 
})
```