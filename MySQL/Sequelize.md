# Sequelize
基于Promise的ORM(Object Relation Mapping)，是一种数据库中间件 支持多种数据库、事务、关联等  
`` npm i sequelize mysql2 -S``

```
// 以操作对象的形式操作数据库
(async () => {
    const Sequelize = require('sequelize')
    // 建立连接
    const sequelize = new Sequelize('mydb', 'root', '999999', {
        host: 'localhost',
        dialect: 'mysql',
        // 兼容性处理
        // 仍可通过传入 operators map 至 operatorsAliases 的方式来使用字符串运算符，但会返回弃用警告
        operatorsAliases: false
    })
    // 定义模型
    const Fruit = sequelize.define('Ftuit', {
        // 默认id可以不配置会有默认的自增id
        // 但是很多时候我们需要uuid
        id: { 
            type: Sequelize.DataTypes.UUID, 
            defaultValue: Sequelize.DataTypes.UUIDV1,
            primaryKey: true 
        },
        name: {
            type: Sequelize.STRING(20), allowNull: false
        },
        price: {
            type: Sequelize.FLOAT,
            allowNull: false
        },
        stock: {
            type: Sequelize.INTEGER,
            defaultValue: 0
        }
    }, {
        // 不要时间戳
        timestamps: false,
        // 设置表名,不设置会有默认值Ftuits
        tableName:'TBL_FRUIT'
    })
    // 同步数据库，{force: true}则会删除已存在表
    let ret = await Fruit.sync()
    // 插入
    ret = await Fruit.create({
        name: '香蕉',
        price: 2.5
    })
    // 更新 
    await Fruit.update(
        { price: 4 },
        { where: { name: '香蕉' } }
    )
    // 操作符
    const Op = Sequelize.Op
    // 查询
    ret = await Fruit.findAll({
        where: {
            price: { [Op.lt]: 4, [Op.gt]: 2 }
        }
    })
    console.log('findAll', JSON.stringify(ret, '', '\t'))
})()
```

### 强制同步：创建表之前先删除已存在的表
``Fruit.sync({force: true})``
### 避免自动生成时间戳字段
``const Fruit = sequelize.define("Fruit", {}, { timestamps: false });``
### 指定表名
``freezeTableName: true 或 tableName:'xxx'``
设置前者则以modelName作为表名；设置后者则按其值作为表名。蛇形命名 underscored: true,默认驼峰命名
### UUID-主键
```
id: { 
    type: Sequelize.DataTypes.UUID, 
    defaultValue: Sequelize.DataTypes.UUIDV1, 
    primaryKey: true
}
```
### Getters & Setters：可用于定义伪属性或映射到数据库字段的保护属性
```
// 
定义为属性的一部分 
name: {
    type: Sequelize.STRING,
    allowNull: false, 
    get() { 
        const fname = this.getDataValue("name"); 
        const price = this.getDataValue("price"); 
        const stock = this.getDataValue("stock"); 
        return `${fname}(价格：￥${price} 库存：${stock}kg)`; 
    } 
}
// 定义为模型选项 
// options中 
{ 
    getterMethods:{ 
        amount(){ 
            return this.getDataValue("stock") + "kg"; 
        } 
    },
    setterMethods:{ 
        amount(val){ 
            const idx = val.indexOf('kg'); 
            const v = val.slice(0, idx); 
            this.setDataValue('stock', v); 
        } 
    } 
}
// 通过模型实例触发setterMethods 
Fruit.findAll().then(fruits => { 
    console.log(JSON.stringify(fruits)); 
    // 修改amount，触发setterMethods 
    fruits[0].amount = '150kg'; 
    fruits[0].save(); 
});
```
### 校验：可以通过校验功能验证模型字段格式、内容，校验会在 create 、 update 和 save 时自动运行
```
price: { 
    validate: { 
        isFloat: { msg: "价格字段请输入数字" }, 
        min: { args: [0], msg: "价格字段必须大于0" } 
    } 
},
stock: { 
    validate: { 
        isNumeric: { msg: "库存字段请输入数字" } 
    } 
}
```

### 模型扩展：可添加模型实例方法或类方法扩展模型
```
// 添加类级别方法 
Fruit.classify = function(name) { 
    const tropicFruits = ['香蕉', '芒果', '椰子']; // 热带水果 return tropicFruits.includes(name) ? '热带水果':'其他水果'; 
};
// 添加实例级别方法 
Fruit.prototype.totalPrice = function(count) { 
    return (this.price * count).toFixed(2); 
};
// 使用类方法 
['香蕉','草莓'].forEach(f => console.log(f+'是'+Fruit.classify(f))); 
// 使用实例方法 
Fruit.findAll().then(fruits => { 
    const [f1] = fruits; 
    console.log(`买5kg${f1.name}需要￥${f1.totalPrice(5)}`); 
});
```
### 数据查询
```
// 通过id查询(不支持了)
Fruit.findById(1).then(fruit => { 
    // fruit是一个Fruit实例，若没有则为null 
    console.log(fruit.get()); 
});
// 通过属性查询 
Fruit.findOne({ where: { name: "香蕉" } }).then(fruit => { 
    // fruit是首个匹配项，若没有则为null 
    console.log(fruit.get()); 
});
// 指定查询字段
Fruit.findOne({ attributes: ['name'] }).then(fruit => {
    // fruit是首个匹配项，若没有则为null
    console.log(fruit.get());
});
// 获取数据和总条数 
Fruit.findAndCountAll().then(result => {
    console.log(result.count); 
    console.log(result.rows.length); 
});
// 查询操作符
const Op = Sequelize.Op; 
Fruit.findAll({ 
    // where: { price: { [Op.lt]:4 }, stock: { [Op.gte]: 100 } } 
    where: { price: { [Op.lt]:4,[Op.gt]:2 }} 
}).then(fruits => { 
    console.log(fruits.length); 
});
// 或语句 
Fruit.findAll({
    // where: { [Op.or]:[{price: { [Op.lt]:4 }}, {stock: { [Op.gte]: 100 }}]}
    where: { price: { [Op.or]:[{[Op.gt]:3 }, {[Op.lt]:2 }]}}
}).then(fruits => {
    console.log(fruits[0].get()); 
});
// 分页
Fruit.findAll({ 
    offset: 0, 
    limit: 2, 
})
// 排序 
Fruit.findAll({ 
    order: [['price', 'DESC']], 
})
// 聚合 
Fruit.max("price").then(max => { 
    console.log("max", max); 
}); 
Fruit.sum("price").then(sum => { 
    console.log("sum", sum); 
});
```
### 更新
```
Fruit.findById(1).then(fruit => { 
    // 方式1
    fruit.price = 4;
    fruit.save().then(()=>console.log('update!!!!'));
}); 
// 方式2
Fruit.update({price:4}, {where:{id:1}}).then(r => {
    console.log(r);
    console.log('update!!!!')
})
```
### 删除
```
// 方式1
Fruit.findOne({ where: { id: 1 } }).then(r => r.destroy());
// 方式2 
Fruit.destroy({ where: { id: 1 } }).then(r => console.log(r));
```