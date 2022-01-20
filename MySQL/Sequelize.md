# Sequelize
基于Promise的ORM(Object Relation Mapping)，是一种数据库中间件 支持多种数据库、事务、关联等  
## 配置
### 安装
这里结合mysql使用  
`` npm i sequelize mysql2 -S``
### 连接到数据库
```
const { Sequelize } = require('sequelize');

// 方法 1: 传递一个连接 URI
const sequelize = new Sequelize('sqlite::memory:') // Sqlite 示例
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // Postgres 示例

// 方法 2: 分别传递参数 (sqlite)
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});

// 方法 3: 分别传递参数 (其它数据库)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* 选择 'mysql' | 'mariadb' | 'postgres' | 'mssql' 其一 */
});
```
### 关闭连接
默认情况下,Sequelize 将保持连接打开状态,并对所有查询使用相同的连接. 如果你需要关闭连接,请调用 ``sequelize.close()``(这是异步的并返回一个 Promise).
## 模型基础
### 模型定义
- 调用 ``sequelize.define(modelName, attributes, options)``
```
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
});

// `sequelize.define` 会返回模型
console.log(User === sequelize.models.User); // true
```
- 扩展 ``Model`` 并调用 ``init(attributes, options)``
```
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory');

class User extends Model {}

User.init({
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
  sequelize, // 我们需要传递连接实例
  modelName: 'User' // 我们需要选择模型名称
});

// 定义的模型是类本身
console.log(User === sequelize.models.User); // true
```
### 表名推断
**在以上两种方法中,都从未明确定义表名(Users). 但是,给出了模型名称(User).默认情况下,当未提供表名时,Sequelize 会自动将模型名复数并将其用作表名. **
### 强制表名称等于模型名称
使用 ``freezeTableName: true`` 参数停止 Sequelize 执行自动复数化. 这样,Sequelize 将推断表名称等于模型名称,而无需进行任何修改:
```
创建一个名为 User 的模型,该模型指向一个也名为 User 的表
sequelize.define('User', {
  // ... (属性)
}, {
  freezeTableName: true
});
```
也可以为 sequelize 实例全局定义此行为：
```
const sequelize = new Sequelize('sqlite::memory:', {
  define: {
    freezeTableName: true
  }
});
```
### 直接提供表名
```
sequelize.define('User', {
  // ... (属性)
}, {
  tableName: 'Employees'
});
```
### 模型同步
- ``User.sync()`` - 如果表不存在,则创建该表(如果已经存在,则不执行任何操作)
- ``User.sync({ force: true })`` - 将创建表,如果表已经存在,则将其首先删除
- ``User.sync({ alter: true })`` - 这将检查数据库中表的当前状态(它具有哪些列,它们的数据类型等),然后在表中进行必要的更改以使其与模型匹配.
### 一次同步所有模型
```
await sequelize.sync({ force: true });
console.log("所有模型均已成功同步.");
```
### 删除表
```
// 删除与模型相关的表：
await User.drop();
console.log("用户表已删除!");

删除所有表：
await sequelize.drop();
console.log("所有表已删除!");
```
### 数据库安全检查
如上所示,sync和drop操作是破坏性的. Sequelize 使用 match 参数作为附加的安全检查,该检查将接受 RegExp：
```
// 仅当数据库名称以 '_test' 结尾时,它才会运行.sync()
sequelize.sync({ force: true, match: /_test$/ });
```
### 生产环境同步

### 时间戳
默认情况下,Sequelize 使用数据类型 DataTypes.DATE 自动向每个模型添加 createdAt 和 updatedAt 字段. 这些字段会自动进行管理 - 每当你使用Sequelize 创建或更新内容时,这些字段都会被自动设置. createdAt 字段将包含代表创建时刻的时间戳,而 updatedAt 字段将包含最新更新的时间戳.

### 列声明简写语法
```
如果关于列的唯一指定内容是其数据类型,则可以缩短语法：
// 例如:
sequelize.define('User', {
  name: {
    type: DataTypes.STRING
  }
});

// 可以简写为:
sequelize.define('User', { name: DataTypes.STRING });
```
### 默认值
默认情况下,Sequelize 假定列的默认值为 NULL. 可以通过将特定的 defaultValue 传递给列定义来更改此行为：
```
sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    defaultValue: "John Doe"
  }
});
```
一些特殊的值,例如 Sequelize.NOW,也能被接受：
```
sequelize.define('Foo', {
  bar: {
    type: DataTypes.DATETIME,
    defaultValue: Sequelize.NOW
    // 这样,当前日期/时间将用于填充此列(在插入时)
  }
});
```
### 数据类型
你在模型中定义的每一列都必须具有数据类型. Sequelize 提供很多内置数据类型. 要访问内置数据类型,必须导入 DataTypes：
```
const { DataTypes } = require("sequelize"); // 导入内置数据类型
```
### 字符串
```
DataTypes.STRING             // VARCHAR(255)
DataTypes.STRING(1234)       // VARCHAR(1234)
DataTypes.STRING.BINARY      // VARCHAR BINARY
DataTypes.TEXT               // TEXT
DataTypes.TEXT('tiny')       // TINYTEXT
DataTypes.CITEXT             // CITEXT          仅 PostgreSQL 和 SQLite.
DataTypes.TSVECTOR           // TSVECTOR        仅 PostgreSQL.
```
### 布尔
```
DataTypes.BOOLEAN            // TINYINT(1)
```
### 数字
```
DataTypes.INTEGER            // INTEGER
DataTypes.BIGINT             // BIGINT
DataTypes.BIGINT(11)         // BIGINT(11)

DataTypes.FLOAT              // FLOAT
DataTypes.FLOAT(11)          // FLOAT(11)
DataTypes.FLOAT(11, 10)      // FLOAT(11,10)

DataTypes.REAL               // REAL            仅 PostgreSQL.
DataTypes.REAL(11)           // REAL(11)        仅 PostgreSQL.
DataTypes.REAL(11, 12)       // REAL(11,12)     仅 PostgreSQL.

DataTypes.DOUBLE             // DOUBLE
DataTypes.DOUBLE(11)         // DOUBLE(11)
DataTypes.DOUBLE(11, 10)     // DOUBLE(11,10)

DataTypes.DECIMAL            // DECIMAL
DataTypes.DECIMAL(10, 2)     // DECIMAL(10,2)
```
在 MySQL 和 MariaDB 中,可以将数据类型INTEGER, BIGINT, FLOAT 和 DOUBLE 设置为无符号或零填充(或两者),如下所示：
```
DataTypes.INTEGER.UNSIGNED
DataTypes.INTEGER.ZEROFILL
DataTypes.INTEGER.UNSIGNED.ZEROFILL
// 你还可以指定大小,即INTEGER(10)而不是简单的INTEGER
// 同样适用于 BIGINT, FLOAT 和 DOUBLE
```
### 日期
```
DataTypes.DATE       // DATETIME 适用于 mysql / sqlite, 带时区的TIMESTAMP 适用于 postgres
DataTypes.DATE(6)    // DATETIME(6) 适用于 mysql 5.6.4+. 支持6位精度的小数秒
DataTypes.DATEONLY   // 不带时间的 DATE
```
### UUID
对于 UUID,使用 DataTypes.UUID. 对于 PostgreSQL 和 SQLite,它会是 UUID 数据类型;对于 MySQL,它则变成CHAR(36). Sequelize 可以自动为这些字段生成 UUID,只需使用 Sequelize.UUIDV1 或 Sequelize.UUIDV4 作为默认值即可：
```
{
  type: DataTypes.UUID,
  defaultValue: Sequelize.UUIDV4 // 或 Sequelize.UUIDV1
}
```
### 利用模型作为类
Sequelize 模型是 ES6 类. 你可以非常轻松地添加自定义实例或类级别的方法.
```
class User extends Model {
  static classLevelMethod() {
    return 'foo';
  }
  instanceLevelMethod() {
    return 'bar';
  }
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({
  firstname: Sequelize.TEXT,
  lastname: Sequelize.TEXT
}, { sequelize });

console.log(User.classLevelMethod()); // 'foo'
const user = User.build({ firstname: 'Jane', lastname: 'Doe' });
console.log(user.instanceLevelMethod()); // 'bar'
console.log(user.getFullname()); // 'Jane Doe'
```
## 模型实例
### 创建实例
尽管模型是一个类,但是你不应直接使用 new 运算符来创建实例. 相反,应该使用 build 方法：
```
const jane = User.build({ name: "Jane" });
console.log(jane instanceof User); // true
console.log(jane.name); // "Jane"
```
但是,以上代码根本无法与数据库通信(请注意,它甚至不是异步的)！ 这是因为 build 方法仅创建一个对象,该对象 表示 可以 映射到数据库的数据. 为了将这个实例真正保存(即持久保存)在数据库中,应使用 save 方法：
```
await jane.save();
console.log('Jane 已保存到数据库!');
```
**非常有用的捷径: create 方法 create**  
```
const jane = await User.create({ name: "Jane" });
// Jane 现在存在于数据库中！
console.log(jane instanceof User); // true
console.log(jane.name); // "Jane"
```
### 注意: 实例记录
```
const jane = await User.create({ name: "Jane" });
// console.log(jane); // 不要这样!
console.log(jane.toJSON()); // 这样最好!
console.log(JSON.stringify(jane, null, 4)); // 这样也不错!
```
### 更新实例
如果你更改实例的某个字段的值,则再次调用 save 将相应地对其进行更新：
```
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// 数据库中的名称仍然是 "Jane"
await jane.save();
// 现在该名称已在数据库中更新为 "Ada"！
```
### 删除实例
你可以通过调用 destroy 来删除实例:
```
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
await jane.destroy();
// 现在该条目已从数据库中删除
```
### 重载实例
你可以通过调用 reload 从数据库中重新加载实例:
```
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// 数据库中的名称依然是 "Jane"
await jane.reload();
console.log(jane.name); // "Jane"
```
reload 调用生成一个 SELECT 查询,以从数据库中获取最新数据.
### 仅保存部分字段
通过传递一个列名数组,可以定义在调用 save 时应该保存哪些属性.
```
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
console.log(jane.favoriteColor); // "green"
jane.name = "Jane II";
jane.favoriteColor = "blue";
await jane.save({ fields: ['name'] });
console.log(jane.name); // "Jane II"
console.log(jane.favoriteColor); // "blue"
// 上面显示为 "blue",因为本地对象将其设置为 "blue",
// 但是在数据库中它仍然是 "green"：
await jane.reload();
console.log(jane.name); // "Jane II"
console.log(jane.favoriteColor); // "green"
```
### 递增和递减整数值
为了递增/递减实例的值而不会遇到并发问题,Sequelize提供了 increment 和 decrement 实例方法.
```
const jane = await User.create({ name: "Jane", age: 100 });
const incrementResult = await jane.increment('age', { by: 2 });
// 注意: 如只增加 1, 你可以省略 'by' 参数, 只需执行 `user.increment('age')`

// 在 PostgreSQL 中, 除非设置了 `{returning：false}` 参数(不然它将是 undefined),
// 否则 `incrementResult` 将是更新后的 user.

// 在其它数据库方言中, `incrementResult` 将会是 undefined. 如果你需要更新的实例, 你需要调用 `user.reload()`.
```
你也可以一次递增多个字段：
```
const jane = await User.create({ name: "Jane", age: 100, cash: 5000 });
await jane.increment({
  'age': 2,
  'cash': 100
});

// 如果值增加相同的数量,则也可以使用以下其他语法：
await jane.increment(['age', 'cash'], { by: 2 });
```
### 递减的工作原理完全相同.
## 模型查询(基础)
### 简单 INSERT 查询
```
// 创建一个新用户
const jane = await User.create({ firstName: "Jane", lastName: "Doe" });
console.log("Jane's auto-generated ID:", jane.id);

const user = await User.create({
  username: 'alice123',
  isAdmin: true
}, { fields: ['username'] });
// 假设 isAdmin 的默认值为 false
console.log(user.username); // 'alice123'
console.log(user.isAdmin); // false
```
### 简单 SELECT 查询
你可以使用 findAll 方法从数据库中读取整个表
```
// 查询所有用户
const users = await User.findAll();
console.log(users.every(user => user instanceof User)); // true
console.log("All users:", JSON.stringify(users, null, 2));

SELECT * FROM ...
```
### SELECT 查询特定属性
选择某些特定属性,可以使用 ``attributes`` 参数：
```
Model.findAll({
  attributes: ['foo', 'bar']
});

SELECT foo, bar FROM ...
```
可以使用嵌套数组来重命名属性：
```
Model.findAll({
  attributes: ['foo', ['bar', 'baz'], 'qux']
});

SELECT foo, bar AS baz, qux FROM ...
```
你可以使用 sequelize.fn 进行聚合：
```
Model.findAll({
  attributes: [
    'foo',
    [sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats'],
    'bar'
  ]
});

SELECT foo, COUNT(hats) AS n_hats, bar FROM ...
```
使用聚合函数时,必须为它提供一个别名,以便能够从模型中访问它.  
如果只想添加聚合,那么列出模型的所有属性可能会很麻烦：
```
// 这是获取帽子数量的烦人方法(每列都有)
Model.findAll({
  attributes: [
    'id', 'foo', 'bar', 'baz', 'qux', 'hats', // 我们必须列出所有属性...
    [sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats'] // 添加聚合...
  ]
});

// 这个更短,并且更不易出错. 如果以后在模型中添加/删除属性,它仍然可以正常工作
Model.findAll({
  attributes: {
    include: [
      [sequelize.fn('COUNT', sequelize.col('hats')), 'n_hats']
    ]
  }
});

SELECT id, foo, bar, baz, qux, hats, COUNT(hats) AS n_hats FROM ...
```
同样,也可以排除某些属性：
```
Model.findAll({
  attributes: { exclude: ['baz'] }
});

-- Assuming all columns are 'id', 'foo', 'bar', 'baz' and 'qux'
SELECT id, foo, bar, qux FROM ...
```
### 应用 WHERE 子句
``where`` 参数用于过滤查询.``where`` 子句有很多运算符,可以从 ``Op`` 中以 Symbols 的形式使用.
#### 基础
```
Post.findAll({
  where: {
    authorId: 2
  }
});
// SELECT * FROM post WHERE authorId = 2
```
可以看到没有显式传递任何运算符(来自Op),因为默认情况下 Sequelize 假定进行相等比较. 上面的代码等效于：
```
const { Op } = require("sequelize");
Post.findAll({
  where: {
    authorId: {
      [Op.eq]: 2
    }
  }
});
// SELECT * FROM post WHERE authorId = 2
```
可以传递多个校验:
```
Post.findAll({
  where: {
    authorId: 12
    status: 'active'
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';
```
就像在第一个示例中 Sequelize 推断出 ``Op.eq`` 运算符一样,在这里 Sequelize 推断出调用者希望对两个检查使用 ``AND``. 上面的代码等效于：
```
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [
      { authorId: 12 },
      { status: 'active' }
    ]
  }
});
// SELECT * FROM post WHERE authorId = 12 AND status = 'active';
```
``OR`` 可以通过类似的方式轻松执行：
```
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.or]: [
      { authorId: 12 },
      { authorId: 13 }
    ]
  }
});
// SELECT * FROM post WHERE authorId = 12 OR authorId = 13;
```
由于以上的 OR 涉及相同字段 ,因此 Sequelize 允许你使用稍有不同的结构,该结构更易读并且作用相同：
```
const { Op } = require("sequelize");
Post.destroy({
  where: {
    authorId: {
      [Op.or]: [12, 13]
    }
  }
});
// DELETE FROM post WHERE authorId = 12 OR authorId = 13;
```
### 操作符
```
const { Op } = require("sequelize");
Post.findAll({
  where: {
    [Op.and]: [{ a: 5 }, { b: 6 }],            // (a = 5) AND (b = 6)
    [Op.or]: [{ a: 5 }, { b: 6 }],             // (a = 5) OR (b = 6)
    someAttribute: {
      // 基本
      [Op.eq]: 3,                              // = 3
      [Op.ne]: 20,                             // != 20
      [Op.is]: null,                           // IS NULL
      [Op.not]: true,                          // IS NOT TRUE
      [Op.or]: [5, 6],                         // (someAttribute = 5) OR (someAttribute = 6)

      // 使用方言特定的列标识符 (以下示例中使用 PG):
      [Op.col]: 'user.organization_id',        // = "user"."organization_id"

      // 数字比较
      [Op.gt]: 6,                              // > 6
      [Op.gte]: 6,                             // >= 6
      [Op.lt]: 10,                             // < 10
      [Op.lte]: 10,                            // <= 10
      [Op.between]: [6, 10],                   // BETWEEN 6 AND 10
      [Op.notBetween]: [11, 15],               // NOT BETWEEN 11 AND 15

      // 其它操作符

      [Op.all]: sequelize.literal('SELECT 1'), // > ALL (SELECT 1)

      [Op.in]: [1, 2],                         // IN [1, 2]
      [Op.notIn]: [1, 2],                      // NOT IN [1, 2]

      [Op.like]: '%hat',                       // LIKE '%hat'
      [Op.notLike]: '%hat',                    // NOT LIKE '%hat'
      [Op.startsWith]: 'hat',                  // LIKE 'hat%'
      [Op.endsWith]: 'hat',                    // LIKE '%hat'
      [Op.substring]: 'hat',                   // LIKE '%hat%'
      [Op.iLike]: '%hat',                      // ILIKE '%hat' (不区分大小写) (仅 PG)
      [Op.notILike]: '%hat',                   // NOT ILIKE '%hat'  (仅 PG)
      [Op.regexp]: '^[h|a|t]',                 // REGEXP/~ '^[h|a|t]' (仅 MySQL/PG)
      [Op.notRegexp]: '^[h|a|t]',              // NOT REGEXP/!~ '^[h|a|t]' (仅 MySQL/PG)
      [Op.iRegexp]: '^[h|a|t]',                // ~* '^[h|a|t]' (仅 PG)
      [Op.notIRegexp]: '^[h|a|t]',             // !~* '^[h|a|t]' (仅 PG)

      [Op.any]: [2, 3],                        // ANY ARRAY[2, 3]::INTEGER (仅 PG)
      [Op.match]: Sequelize.fn('to_tsquery', 'fat & rat') // 匹配文本搜索字符串 'fat' 和 'rat' (仅 PG)

      // 在 Postgres 中, Op.like/Op.iLike/Op.notLike 可以结合 Op.any 使用:
      [Op.like]: { [Op.any]: ['cat', 'hat'] }  // LIKE ANY ARRAY['cat', 'hat']

      // 还有更多的仅限 postgres 的范围运算符,请参见下文
    }
  }
});
```
**Op.in 的简写语法**  
直接将数组参数传递给 where 将隐式使用 IN 运算符：
```
Post.findAll({
  where: {
    id: [1,2,3] // 等同使用 `id: { [Op.in]: [1,2,3] }`
  }
});
// SELECT ... FROM "posts" AS "post" WHERE "post"."id" IN (1, 2, 3);
```
### 高级查询(不仅限于列)
如果你想得到类似 WHERE char_length("content") = 7 的结果怎么办？
```
Post.findAll({
  where: sequelize.where(sequelize.fn('char_length', sequelize.col('content')), 7)
});
// SELECT ... FROM "posts" AS "post" WHERE char_length("content") = 7
```
请注意方法 sequelize.fn 和 sequelize.col 的用法,应分别用于指定 SQL 函数调用和列. 应该使用这些方法,而不是传递纯字符串(例如 char_length(content)),因为 Sequelize 需要以不同的方式对待这种情况(例如,使用其他符号转义方法).
```
Post.findAll({
  where: {
    [Op.or]: [
      sequelize.where(sequelize.fn('char_length', sequelize.col('content')), 7),
      {
        content: {
          [Op.like]: 'Hello%'
        }
      },
      {
        [Op.and]: [
          { status: 'draft' },
          sequelize.where(sequelize.fn('char_length', sequelize.col('content')), {
            [Op.gt]: 10
          })
        ]
      }
    ]
  }
});
```
上面生成了以下SQL：
```
SELECT
  ...
FROM "posts" AS "post"
WHERE (
  char_length("content") = 7
  OR
  "post"."content" LIKE 'Hello%'
  OR (
    "post"."status" = 'draft'
    AND
    char_length("content") > 10
  )
)
```
### 简单 UPDATE 查询
Update 查询也接受 where 参数,就像上面的读取查询一样.
```
// 将所有没有姓氏的人更改为 "Doe"
await User.update({ lastName: "Doe" }, {
  where: {
    lastName: null
  }
});
```
### 简单 DELETE 查询
Delete 查询也接受 where 参数,就像上面的读取查询一样.
```
// 删除所有名为 "Jane" 的人 
await User.destroy({
  where: {
    firstName: "Jane"
  }
});
```
### 要销毁所有内容,可以使用 TRUNCATE SQL
```
// 截断表格
await User.destroy({
  truncate: true
});
```
### 批量创建
Sequelize 提供了 Model.bulkCreate 方法,以允许仅一次查询即可一次创建多个记录.  
通过接收数组对象而不是单个对象,Model.bulkCreate 的用法与 Model.create 非常相似.
```
const captains = await Captain.bulkCreate([
  { name: 'Jack Sparrow' },
  { name: 'Davy Jones' }
]);
console.log(captains.length); // 2
console.log(captains[0] instanceof Captain); // true
console.log(captains[0].name); // 'Jack Sparrow'
console.log(captains[0].id); // 1 // (或另一个自动生成的值)
```
但是,默认情况下,bulkCreate 不会在要创建的每个对象上运行验证(而 create 可以做到). 为了使 bulkCreate 也运行这些验证,必须通过validate: true 参数. 但这会降低性能. 用法示例：
```
const Foo = sequelize.define('foo', {
  bar: {
    type: DataTypes.TEXT,
    validate: {
      len: [4, 6]
    }
  }
});

// 这不会引发错误,两个实例都将被创建
await Foo.bulkCreate([
  { name: 'abc123' },
  { name: 'name too long' }
]);

// 这将引发错误,不会创建任何内容
await Foo.bulkCreate([
  { name: 'abc123' },
  { name: 'name too long' }
], { validate: true });
```
**限制实际插入的列**
```
await User.bulkCreate([
  { username: 'foo' },
  { username: 'bar', admin: true }
], { fields: ['username'] });
// foo 和 bar 都不会是管理员.
```
### 排序和分组
Sequelize 提供了 ``order`` and ``group`` 参数,来与 ``ORDER BY`` 和 ``GROUP BY`` 一起使用.
#### 排序
order 参数采用一系列 项 来让 sequelize 方法对查询进行排序. 这些 项 本身是 [column, direction] 形式的数组. 该列将被正确转义,并且将在有效方向列表中进行验证(例如 ASC, DESC, NULLS FIRST 等).
```
Subtask.findAll({
  order: [
    // 将转义 title 并针对有效方向列表进行降序排列
    ['title', 'DESC'],

    // 将按最大年龄进行升序排序
    sequelize.fn('max', sequelize.col('age')),

    // 将按最大年龄进行降序排序
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],

    // 将按 otherfunction(`col1`, 12, 'lalala') 进行降序排序
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],

    // 将使用模型名称作为关联名称按关联模型的 createdAt 排序.
    [Task, 'createdAt', 'DESC'],

    // 将使用模型名称作为关联名称通过关联模型的 createdAt 排序.
    [Task, Project, 'createdAt', 'DESC'],

    // 将使用关联名称按关联模型的 createdAt 排序.
    ['Task', 'createdAt', 'DESC'],

    // 将使用关联的名称按嵌套的关联模型的 createdAt 排序.
    ['Task', 'Project', 'createdAt', 'DESC'],

    // 将使用关联对象按关联模型的 createdAt 排序. (首选方法)
    [Subtask.associations.Task, 'createdAt', 'DESC'],

    // 将使用关联对象按嵌套关联模型的 createdAt 排序. (首选方法)
    [Subtask.associations.Task, Task.associations.Project, 'createdAt', 'DESC'],

    // 将使用简单的关联对象按关联模型的 createdAt 排序.
    [{model: Task, as: 'Task'}, 'createdAt', 'DESC'],

    // 将由嵌套关联模型的 createdAt 简单关联对象排序.
    [{model: Task, as: 'Task'}, {model: Project, as: 'Project'}, 'createdAt', 'DESC']
  ],

  // 将按最大年龄降序排列
  order: sequelize.literal('max(age) DESC'),

  // 如果忽略方向,则默认升序,将按最大年龄升序排序
  order: sequelize.fn('max', sequelize.col('age')),

  // 如果省略方向,则默认升序, 将按年龄升序排列
  order: sequelize.col('age'),

  // 将根据方言随机排序(但不是 fn('RAND') 或 fn('RANDOM'))
  order: sequelize.random()
});

Foo.findOne({
  order: [
    // 将返回 `name`
    ['name'],
    // 将返回 `username` DESC
    ['username', 'DESC'],
    // 将返回 max(`age`)
    sequelize.fn('max', sequelize.col('age')),
    // 将返回 max(`age`) DESC
    [sequelize.fn('max', sequelize.col('age')), 'DESC'],
    // 将返回 otherfunction(`col1`, 12, 'lalala') DESC
    [sequelize.fn('otherfunction', sequelize.col('col1'), 12, 'lalala'), 'DESC'],
    // 将返回 otherfunction(awesomefunction(`col`)) DESC, 这种嵌套可能是无限的!
    [sequelize.fn('otherfunction', sequelize.fn('awesomefunction', sequelize.col('col'))), 'DESC']
  ]
});
```
- 一个字符串 (它将被自动引用)
- 一个数组, 其第一个元素将被引用,第二个将被逐字追加
- 一个具有 raw 字段的对象:
  - raw 内容将不加引用地逐字添加
  - 其他所有内容都将被忽略,如果未设置 raw,查询将失败
- 调用 Sequelize.fn (这将在 SQL 中生成一个函数调用,创建一个表示数据库函数的对象)
- 调用 Sequelize.col (这将引用列名,创建一个代表数据库中的列的对象)
#### 分组
分组和排序的语法相同,只是分组不接受方向作为数组的最后一个参数(不存在 ASC, DESC, NULLS FIRST 等).  
你还可以将字符串直接传递给 group,该字符串将直接(普通)包含在生成的 SQL 中. **请谨慎使用,请勿与用户生成的内容一起使用.**
```
Project.findAll({ group: 'name' });
// 生成 'GROUP BY name'
```
### 限制和分页
使用 ``limit`` 和 ``offset`` 参数可以进行 限制/分页：
```
// 提取10个实例/行
Project.findAll({ limit: 10 });

// 跳过8个实例/行
Project.findAll({ offset: 8 });

// 跳过5个实例,然后获取5个实例
Project.findAll({ offset: 5, limit: 5 });
```
通常这些与 ``order`` 参数一起使用
### 实用方法
#### count
```
console.log(`这有 ${await Project.count()} 个项目`);

const amount = await Project.count({
  where: {
    id: {
      [Op.gt]: 25
    }
  }
});
console.log(`这有 ${amount} 个项目 id 大于 25`);
```
#### max, min 和 sum
Sequelize 还提供了 max,min 和 sum 便捷方法.假设我们有三个用户,分别是10、5和40岁.
```
await User.max('age'); // 40
await User.max('age', { where: { age: { [Op.lt]: 20 } } }); // 10
await User.min('age'); // 5
await User.min('age', { where: { age: { [Op.gt]: 5 } } }); // 10
await User.sum('age'); // 55
await User.sum('age', { where: { age: { [Op.gt]: 5 } } }); // 50
```
## 模型查询(查找器)
### findAll
在上一教程中已经知道 findAll 方法. 它生成一个标准的 SELECT 查询,该查询将从表中检索所有条目(除非受到 where 子句的限制).
### findByPk
使用提供的主键从表中仅获得一个条目.
```
const project = await Project.findByPk(123);
if (project === null) {
  console.log('Not found!');
} else {
  console.log(project instanceof Project); // true
  // 它的主键是 123
}
```
### findOne
获得它找到的第一个条目(它可以满足提供的可选查询参数).
```
const project = await Project.findOne({ where: { title: 'My Title' } });
if (project === null) {
  console.log('Not found!');
} else {
  console.log(project instanceof Project); // true
  console.log(project.title); // 'My Title'
}
```
### findOrCreate
除非找到一个满足查询参数的结果,否则方法 findOrCreate 将在表中创建一个条目. 在这两种情况下,它将返回一个实例(找到的实例或创建的实例)和一个布尔值,指示该实例是已创建还是已经存在.  
使用 where 参数来查找条目,而使用 defaults 参数来定义必须创建的内容. 如果 defaults 不包含每一列的值,则 Sequelize 将采用 where 的值(如果存在).  
假设我们有一个空的数据库,该数据库具有一个 User 模型,该模型具有一个 username 和一个 job.
```
const [user, created] = await User.findOrCreate({
  where: { username: 'sdepold' },
  defaults: {
    job: 'Technical Lead JavaScript'
  }
});
console.log(user.username); // 'sdepold'
console.log(user.job); // 这可能是也可能不是 'Technical Lead JavaScript'
console.log(created); // 指示此实例是否刚刚创建的布尔值
if (created) {
  console.log(user.job); // 这里肯定是 'Technical Lead JavaScript'
}
```
### findAndCountAll
``findAndCountAll`` 方法是结合了 ``findAll`` 和 ``count`` 的便捷方法. 在处理与分页有关的查询时非常有用,在分页中,你想检索带有 limit 和 offset 的数据,但又需要知道与查询匹配的记录总数.  
``findAndCountAll`` 方法返回一个具有两个属性的对象：
- ``count`` - 一个整数 - 符合查询条件的记录总数
- ``rows`` - 一个数组对象 - 获得的记录
```
const { count, rows } = await Project.findAndCountAll({
  where: {
    title: {
      [Op.like]: 'foo%'
    }
  },
  offset: 10,
  limit: 2
});
console.log(count);
console.log(rows);
```
## 获取器, 设置器 & 虚拟字段
### 获取器
获取器是为模型定义中的一列定义的 get() 函数：
```
const User = sequelize.define('user', {
  // 假设我们想要以大写形式查看每个用户名,
  // 即使它们在数据库本身中不一定是大写的
  username: {
    type: DataTypes.STRING,
    get() {
      const rawValue = this.getDataValue('username');
      return rawValue ? rawValue.toUpperCase() : null;
    }
  }
});
```
### 设置器
设置器是为模型定义中的一列定义的 set() 函数. 它接收要设置的值：
```
const User = sequelize.define('user', {
  username: DataTypes.STRING,
  password: {
    type: DataTypes.STRING,
    set(value) {
      // 在数据库中以明文形式存储密码是很糟糕的.
      // 使用适当的哈希函数来加密哈希值更好.
      this.setDataValue('password', hash(value));
    }
  }
});
```
### 组合获取器和设置器
假设我们正在建一个 Post 模型,其 content 是无限长度的文本. 假设要提高内存使用率,我们要存储内容的压缩版本.
```
const { gzipSync, gunzipSync } = require('zlib');

const Post = sequelize.define('post', {
  content: {
    type: DataTypes.TEXT,
    get() {
      const storedValue = this.getDataValue('content');
      const gzippedBuffer = Buffer.from(storedValue, 'base64');
      const unzippedBuffer = gunzipSync(gzippedBuffer);
      return unzippedBuffer.toString();
    },
    set(value) {
      const gzippedBuffer = gzipSync(value);
      this.setDataValue('content', gzippedBuffer.toString('base64'));
    }
  }
});
```
### 虚拟字段
虚拟字段是 Sequelize 在后台填充的字段,但实际上它们不存在于数据库中.
```
const { DataTypes } = require("sequelize");

const User = sequelize.define('user', {
  firstName: DataTypes.TEXT,
  lastName: DataTypes.TEXT,
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      throw new Error('不要尝试设置 `fullName` 的值!');
    }
  }
});
```
``VIRTUAL`` 字段不会导致数据表也存在此列. 换句话说,上面的模型虽然没有 fullName 列. 但是它似乎存在着！
## 验证 & 约束
验证是在纯 JavaScript 中在 Sequelize 级别执行的检查. 如果你提供自定义验证器功能,它们可能会非常复杂,也可能是 Sequelize 提供的内置验证器之一. 如果验证失败,则根本不会将 SQL 查询发送到数据库.  
另一方面,约束是在 SQL 级别定义的规则. 约束的最基本示例是唯一约束. 如果约束检查失败,则数据库将引发错误,并且 Sequelize 会将错误转发给 JavaScript(在此示例中,抛出 SequelizeUniqueConstraintError). 请注意,在这种情况下,与验证不同,它执行了 SQL 查询.
### 唯一约束
下面的代码示例在 username 字段上定义了唯一约束
```
/* ... */ {
  username: {
    type: DataTypes.TEXT,
    allowNull: false,
    unique: true
  },
} /* ... */
```
同步此模型后(例如,通过调用sequelize.sync),在表中将 username 字段创建为 `username` TEXT UNIQUE,如果尝试插入已存在的用户名将抛出 SequelizeUniqueConstraintError.

### 允许/禁止 null 值
默认情况下,null 是模型每一列的允许值. 可以通过为列设置 allowNull: false 参数来禁用它,就像在我们的代码示例的 username 字段中所做的一样：
```
/* ... */ {
  username: {
    type: DataTypes.TEXT,
    allowNull: false,
    unique: true
  },
} /* ... */
```
如果没有 allowNull: false, 那么调用 User.create({}) 将会生效.
### 关于 allowNull 实现的说明
``allowNull`` 检查是 Sequelize 中唯一由 验证 和 约束 混合而成的检查. 
- 如果试图将 null 设置到不允许为 null 的字段,则将抛出ValidationError ,而且 不会执行任何 SQL 查询.
- 另外,在 ``sequelize.sync`` 之后,具有 ``allowNull: false`` 的列将使用 ``NOT NULL`` SQL 约束进行定义. 这样,尝试将值设置为 ``null`` 的直接 SQL 查询也将失败.

### 验证器
使用模型验证器,可以为模型的每个属性指定 格式/内容/继承 验证. 验证会自动在 create, update 和 save 时运行. 你还可以调用 validate() 来手动验证实例.

### 按属性验证
你可以定义你的自定义验证器,也可以使用由 validator.js (10.11.0) 实现的多个内置验证器,如下所示.
```
sequelize.define('foo', {
  bar: {
    type: DataTypes.STRING,
    validate: {
      is: /^[a-z]+$/i,          // 匹配这个 RegExp
      is: ["^[a-z]+$",'i'],     // 与上面相同,但是以字符串构造 RegExp
      not: /^[a-z]+$/i,         // 不匹配 RegExp
      not: ["^[a-z]+$",'i'],    // 与上面相同,但是以字符串构造 RegExp
      isEmail: true,            // 检查 email 格式 (foo@bar.com)
      isUrl: true,              // 检查 url 格式 (http://foo.com)
      isIP: true,               // 检查 IPv4 (129.89.23.1) 或 IPv6 格式
      isIPv4: true,             // 检查 IPv4 格式 (129.89.23.1)
      isIPv6: true,             // 检查 IPv6 格式
      isAlpha: true,            // 只允许字母
      isAlphanumeric: true,     // 将仅允许使用字母数字,因此 '_abc' 将失败
      isNumeric: true,          // 只允许数字
      isInt: true,              // 检查有效的整数
      isFloat: true,            // 检查有效的浮点数
      isDecimal: true,          // 检查任何数字
      isLowercase: true,        // 检查小写
      isUppercase: true,        // 检查大写
      notNull: true,            // 不允许为空
      isNull: true,             // 只允许为空
      notEmpty: true,           // 不允许空字符串
      equals: 'specific value', // 仅允许 'specific value'
      contains: 'foo',          // 强制特定子字符串
      notIn: [['foo', 'bar']],  // 检查值不是这些之一
      isIn: [['foo', 'bar']],   // 检查值是其中之一
      notContains: 'bar',       // 不允许特定的子字符串
      len: [2,10],              // 仅允许长度在2到10之间的值
      isUUID: 4,                // 只允许 uuid
      isDate: true,             // 只允许日期字符串
      isAfter: "2011-11-05",    // 仅允许特定日期之后的日期字符串
      isBefore: "2011-11-05",   // 仅允许特定日期之前的日期字符串
      max: 23,                  // 仅允许值 <= 23
      min: 23,                  // 仅允许值 >= 23
      isCreditCard: true,       // 检查有效的信用卡号

      // 自定义验证器的示例:
      isEven(value) {
        if (parseInt(value) % 2 !== 0) {
          throw new Error('Only even values are allowed!');
        }
      }
      isGreaterThanOtherField(value) {
        if (parseInt(value) <= parseInt(this.otherField)) {
          throw new Error('Bar must be greater than otherField.');
        }
      }
    }
  }
});
```
### allowNull 与其他验证器的交互
如果将模型的特定字段设置为不允许为 null(使用 allowNull: false),并且该值已设置为 null,则将跳过所有验证器,并抛出 ValidationError.  
另一方面,如果将其设置为允许 null(使用 allowNull: true),并且该值已设置为 null,则仅会跳过内置验证器,而自定义验证器仍将运行.  
举例来说,这意味着你可以拥有一个字符串字段,该字段用于验证其长度在5到10个字符之间,但也允许使用 null (因为当该值为 null 时,长度验证器将被自动跳过)：
```
class User extends Model {}
User.init({
  username: {
    type: DataTypes.STRING,
    allowNull: true,
    validate: {
      len: [5, 10]
    }
  }
}, { sequelize });
```
你也可以使用自定义验证器有条件地允许 null 值,因为不会跳过它：
```
class User extends Model {}
User.init({
  age: Sequelize.INTEGER,
  name: {
    type: DataTypes.STRING,
    allowNull: true,
    validate: {
      customValidator(value) {
        if (value === null && this.age !== 10) {
          throw new Error("除非年龄为10,否则名称不能为 null");
        }
      })
    }
  }
}, { sequelize });
```
你可以通过设置 notNull 验证器来自定义 allowNull 错误消息：
```
class User extends Model {}
User.init({
  name: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notNull: {
        msg: '请输入你的名字'
      }
    }
  }
}, { sequelize });
```
### 模型范围内的验证
还可以定义验证,来在特定于字段的验证器之后检查模型.  
所收集的任何错误消息都将与字段验证错误一起放入验证结果对象中,其关键字以 validate 选项对象中验证方法失败的键命名. 
```
class Place extends Model {}
Place.init({
  name: Sequelize.STRING,
  address: Sequelize.STRING,
  latitude: {
    type: DataTypes.INTEGER,
    validate: {
      min: -90,
      max: 90
    }
  },
  longitude: {
    type: DataTypes.INTEGER,
    validate: {
      min: -180,
      max: 180
    }
  },
}, {
  sequelize,
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Either both latitude and longitude, or neither!');
      }
    }
  }
})
```
## 关联
Sequelize 支持标准关联关系: 一对一, 一对多 和 多对多.为此,Sequelize 提供了 四种 关联类型,并将它们组合起来以创建关联：
- ``HasOne`` 关联类型
- ``BelongsTo`` 关联类型
- ``HasMany`` 关联类型
- ``BelongsToMany`` 关联类型
### 定义 Sequelize 关联
四种关联类型的定义非常相似. 假设我们有两个模型 A 和 B. 告诉 Sequelize 两者之间的关联仅需要调用一个函数：
```
const A = sequelize.define('A', /* ... */);
const B = sequelize.define('B', /* ... */);

A.hasOne(B); // A 有一个 B
A.belongsTo(B); // A 属于 B
A.hasMany(B); // A 有多个 B
A.belongsToMany(B, { through: 'C' }); // A 属于多个 B , 通过联结表 C
```
它们都接受一个对象作为第二个参数(前三个参数是可选的,而对于包含 through 属性的 belongsToMany 是必需的)： They all accept an options object as a second parameter
```
A.hasOne(B, { /* 参数 */ });
A.belongsTo(B, { /* 参数 */ });
A.hasMany(B, { /* 参数 */ });
A.belongsToMany(B, { through: 'C', /* 参数 */ });
```
关联的定义顺序是有关系的. 换句话说,对于这四种情况,定义顺序很重要. 在上述所有示例中,A 称为 **源** 模型,而 B 称为 **目标** 模型. 此术语很重要.
- ``A.hasOne(B)`` 关联意味着 A 和 B 之间存在一对一的关系,外键在目标模型(B)中定义.
- ``A.belongsTo(B)``关联意味着 A 和 B 之间存在一对一的关系,外键在源模型中定义(A).
- ``A.hasMany(B)`` 关联意味着 A 和 B 之间存在一对多关系,外键在目标模型(B)中定义.
这三个调用将导致 Sequelize 自动将外键添加到适当的模型中(除非它们已经存在).  
``A.belongsToMany(B, { through: 'C' })`` 关联意味着将表 C 用作联结表,在 A 和 B 之间存在多对多关系. 具有外键(例如,aId 和 bId). Sequelize 将自动创建此模型 C(除非已经存在),并在其上定义适当的外键.
### 创建标准关系
- 创建一个 一对一 关系, ``hasOne`` 和 ``belongsTo`` 关联一起使用;
- 创建一个 一对多 关系, ``hasMany`` 和 ``belongsTo`` 关联一起使用;
- 创建一个 多对多 关系, 两个 ``belongsToMany`` 调用一起使用.
### 一对一关系
```
//  hasOne 和 belongsTo 调用都会推断出要创建的外键应称为 fooId.
Foo.hasOne(Bar);
Bar.belongsTo(Foo);

// 自定义外键
// 方法 1
Foo.hasOne(Bar, {
  foreignKey: 'myFooId'
});
Bar.belongsTo(Foo);

// 方法 2
Foo.hasOne(Bar, {
  foreignKey: {
    name: 'myFooId'
  }
});
Bar.belongsTo(Foo);

// 方法 3
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: 'myFooId'
});

// 方法 4
Foo.hasOne(Bar);
Bar.belongsTo(Foo, {
  foreignKey: {
    name: 'myFooId'
  }
});
```
### 强制性与可选性关联
默认情况下,该关联被视为可选. 换句话说,在我们的示例中,fooId 允许为空,这意味着一个 Bar 可以不存在 Foo 而存在. 只需在外键选项中指定 allowNull: false 即可更改此设置：
```
Foo.hasOne(Bar, {
  foreignKey: {
    allowNull: false
  }
});
// "fooId" INTEGER NOT NULL REFERENCES "foos" ("id") ON DELETE RESTRICT ON UPDATE RESTRICT
```
### 一对多关系
一对多关联将一个源与多个目标连接,而所有这些目标仅与此单个源连接.
```
Team.hasMany(Player);
Player.belongsTo(Team);
```
### 多对多关系
不能像其他关系那样通过向其中一个表添加一个外键来表示这一点. 取而代之的是使用联结模型的概念. 这将是一个额外的模型(以及数据库中的额外表),它将具有两个外键列并跟踪关联. 联结表有时也称为 join table 或 through table.
```
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
Movie.belongsToMany(Actor, { through: 'ActorMovies' });
Actor.belongsToMany(Movie, { through: 'ActorMovies' });
```
因为在 belongsToMany 的 through 参数中给出了一个字符串,所以 Sequelize 将自动创建 ActorMovies 模型作为联结模型.  
除了字符串以外,还支持直接传递模型,在这种情况下,给定的模型将用作联结模型(并且不会自动创建任何模型). 例如：
```
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
const ActorMovies = sequelize.define('ActorMovies', {
  MovieId: {
    type: DataTypes.INTEGER,
    references: {
      model: Movie, // 'Movies' 也可以使用
      key: 'id'
    }
  },
  ActorId: {
    type: DataTypes.INTEGER,
    references: {
      model: Actor, // 'Actors' 也可以使用
      key: 'id'
    }
  }
});
Movie.belongsToMany(Actor, { through: ActorMovies });
Actor.belongsToMany(Movie, { through: ActorMovies });
```
### 基本的涉及关联的查询
例子,其中有船和船长,以及它们之间的一对一关系. 我们将在外键上允许 null(默认值),这意味着船可以在没有船长的情况下存在,反之亦然.
```
// 这是我们用于以下示例的模型的设置
const Ship = sequelize.define('ship', {
  name: DataTypes.STRING,
  crewCapacity: DataTypes.INTEGER,
  amountOfSails: DataTypes.INTEGER
}, { timestamps: false });
const Captain = sequelize.define('captain', {
  name: DataTypes.STRING,
  skillLevel: {
    type: DataTypes.INTEGER,
    validate: { min: 1, max: 10 }
  }
}, { timestamps: false });
Captain.hasOne(Ship);
Ship.belongsTo(Captain);
```
### 获取关联 - 预先加载 vs 延迟加载
延迟加载是指仅在确实需要时才获取关联数据的技术.  
预先加载是指从一开始就通过较大的查询一次获取所有内容的技术.  
### 延迟加载示例
```
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  }
});
// 用获取到的 captain 做点什么
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
// 现在我们需要有关他的 ship 的信息!
const hisShip = await awesomeCaptain.getShip();
// 用 ship 做点什么
console.log('Ship Name:', hisShip.name);
console.log('Amount of Sails:', hisShip.amountOfSails);
```
上面使用的 ``getShip()`` 实例方法是 Sequelize 自动添加到 Captain 实例的方法之一. 
### 预先加载示例
```
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  },
  include: Ship
});
// 现在 ship 跟着一起来了
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
console.log('Ship Name:', awesomeCaptain.ship.name);
console.log('Amount of Sails:', awesomeCaptain.ship.amountOfSails);
```
如上所示,通过使用 ``include`` 参数 在 Sequelize 中执行预先加载. 观察到这里只对数据库执行了一个查询(与实例一起带回关联的数据).
### 创建, 更新和删除
直接使用标准模型查询：
```
// 示例：使用标准方法创建关联的模型
Bar.create({
  name: 'My Bar',
  fooId: 5
});
// 这将创建一个属于 ID 5 的 Foo 的 Bar
// 这里没有什么特别的东西
```
save()实例方法 并不知道关联关系. 如果你修改了 父级 对象预先加载的 子级 的值,那么在父级上调用 save() 将会忽略子级上发生的修改.
### 关联别名 & 自定义外键
```
const Ship = sequelize.define('ship', { name: DataTypes.STRING }, { timestamps: false });
const Captain = sequelize.define('captain', { name: DataTypes.STRING }, { timestamps: false });
```
有三种方法可以为外键指定不同的名称：
- 通过直接提供外键名称
```
Ship.belongsTo(Captain, { foreignKey: 'bossId' }); // 这将在 Ship 中创建 `bossId` 外键.

// 通过将模型传递给 `include` 来完成预先加载:
console.log((await Ship.findAll({ include: Captain })).toJSON());
// 或通过提供关联的模型名称:
console.log((await Ship.findAll({ include: 'Captain' })).toJSON());

// 同样,实例获得用于延迟加载的 `getCaptain()` 方法:
const ship = Ship.findOne();
console.log((await ship.getCaptain()).toJSON());
```
- 通过定义别名
```
Ship.belongsTo(Captain, { as: 'leader' }); // 这将在 Ship 中创建 `leaderId` 外键.

// 通过将模型传递给 `include` 不能再触发预先加载:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // 引发错误
// 相反,你必须传递别名:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// 或者,你可以传递一个指定模型和别名的对象:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// 同样,实例获得用于延迟加载的 `getLeader()`方法:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```
- 通过两个方法同时进行
- 我们可以定义别名,也可以直接定义外键
```
Ship.belongsTo(Captain, { as: 'leader', foreignKey: 'bossId' }); // 这将在 Ship 中创建 `bossId` 外键.

// 由于定义了别名,因此仅通过将模型传递给 `include`,预先加载将不起作用:
console.log((await Ship.findAll({ include: Captain })).toJSON()); // 引发错误
// 相反,你必须传递别名:
console.log((await Ship.findAll({ include: 'leader' })).toJSON());
// 或者,你可以传递一个指定模型和别名的对象:
console.log((await Ship.findAll({
  include: {
    model: Captain,
    as: 'leader'
  }
})).toJSON());

// 同样,实例获得用于延迟加载的 `getLeader()` 方法:
const ship = Ship.findOne();
console.log((await ship.getLeader()).toJSON());
```
### 添加到实例的特殊方法
当两个模型之间定义了关联时,这些模型的实例将获得特殊的方法来与其关联的另一方进行交互.
#### ``Foo.hasOne(Bar)``
- ``fooInstance.getBar()``
- ``fooInstance.setBar()``
- ``fooInstance.createBar()``
```
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBar()); // null
await foo.setBar(bar1);
console.log((await foo.getBar()).name); // 'some-bar'
await foo.createBar({ name: 'yet-another-bar' });
const newlyAssociatedBar = await foo.getBar();
console.log(newlyAssociatedBar.name); // 'yet-another-bar'
await foo.setBar(null); // Un-associate
console.log(await foo.getBar()); // null
```
#### ``Foo.belongsTo(Bar)``
- ``fooInstance.getBar()``
- ``fooInstance.setBar()``
- ``fooInstance.createBar()``
```
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBar()); // null
await foo.setBar(bar1);
console.log((await foo.getBar()).name); // 'some-bar'
await foo.createBar({ name: 'yet-another-bar' });
const newlyAssociatedBar = await foo.getBar();
console.log(newlyAssociatedBar.name); // 'yet-another-bar'
await foo.setBar(null); // Un-associate
console.log(await foo.getBar()); // null
```
#### ``Foo.hasMany(Bar)``
- ``fooInstance.getBars()``
- ``fooInstance.countBars()``
- ``fooInstance.hasBar()``
- ``fooInstance.hasBars()``
- ``fooInstance.setBars()``
- ``fooInstance.addBar()``
- ``fooInstance.addBars()``
- ``fooInstance.removeBar()``
- ``fooInstance.removeBars()``
- ``fooInstance.createBar()``
```
const foo = await Foo.create({ name: 'the-foo' });
const bar1 = await Bar.create({ name: 'some-bar' });
const bar2 = await Bar.create({ name: 'another-bar' });
console.log(await foo.getBars()); // []
console.log(await foo.countBars()); // 0
console.log(await foo.hasBar(bar1)); // false
await foo.addBars([bar1, bar2]);
console.log(await foo.countBars()); // 2
await foo.addBar(bar1);
console.log(await foo.countBars()); // 2
console.log(await foo.hasBar(bar1)); // true
await foo.removeBar(bar2);
console.log(await foo.countBars()); // 1
await foo.createBar({ name: 'yet-another-bar' });
console.log(await foo.countBars()); // 2
await foo.setBars([]); // 取消关联所有先前关联的 Bars
console.log(await foo.countBars()); // 0
```
#### ``Foo.belongsToMany(Bar, { through: Baz })``
来自 Foo.hasMany(Bar) 的相同内容:
- ``fooInstance.getBars()``
- ``fooInstance.countBars()``
- ``fooInstance.hasBar()``
- ``fooInstance.hasBars()``
- ``fooInstance.setBars()``
- ``fooInstance.addBar()``
- ``fooInstance.addBars()``
- ``fooInstance.removeBar()``
- ``fooInstance.removeBars()``
- ``fooInstance.createBar()``
#### 注意: 方法名称
如上面的示例所示,Sequelize 赋予这些特殊方法的名称是由前缀(例如,get,add,set)和模型名称(首字母大写)组成的. 必要时,可以使用复数形式,例如在 ``fooInstance.setBars()`` 中. 同样,不规则复数也由 Sequelize 自动处理. 例如,``Person`` 变成 ``People`` 或者 ``Hypothesis`` 变成 ``Hypotheses``.  
如果定义了别名,则将使用别名代替模型名称来形成方法名称. 例如：
```
Task.hasOne(User, { as: 'Author' });
```
- ``taskInstance.getAuthor()``
- ``taskInstance.setAuthor()``
- ``taskInstance.createAuthor()``
### 涉及相同模型的多个关联
在 Sequelize 中,可以在同一模型之间定义多个关联. 你只需要为它们定义不同的别名：
```
Team.hasOne(Game, { as: 'HomeTeam', foreignKey: 'homeTeamId' });
Team.hasOne(Game, { as: 'AwayTeam', foreignKey: 'awayTeamId' });
Game.belongsTo(Team);
```
### 创建引用非主键字段的关联
此字段必须具有唯一的约束(否则,这将没有意义).
## 偏执表
Sequelize 支持 paranoid 表的概念. 一个 paranoid 表是一个被告知删除记录时不会真正删除它的表.反而一个名为 deletedAt 的特殊列会将其值设置为该删除请求的时间戳.  
这意味着偏执表会执行记录的 软删除,而不是 硬删除.
### 将模型定义为 paranoid
要定义 paranoid 模型,必须将 paranoid: true 参数传递给模型定义. Paranoid 需要时间戳才能起作用(即,如果你传递 timestamps: false 了,paranoid 将不起作用).  
你还可以将默认的列名(默认是 deletedAt)更改为其他名称.
```
class Post extends Model {}
Post.init({ /* 这是属性 */ }, {
  sequelize,
  paranoid: true,

  // 如果要为 deletedAt 列指定自定义名称
  deletedAt: 'destroyTime'
});
```
### 删除
当你调用 destroy 方法时,将发生软删除：
```
await Post.destroy({
  where: {
    id: 1
  }
});
// UPDATE "posts" SET "deletedAt"=[timestamp] WHERE "deletedAt" IS NULL AND "id" = 1
```
如果你确实想要硬删除,并且模型是 paranoid,则可以使用 force: true 参数强制执行：
```
await Post.destroy({
  where: {
    id: 1
  },
  force: true
});
// DELETE FROM "posts" WHERE "id" = 1
```
所有实例方法的工作方式相同：
```
const post = await Post.create({ title: 'test' });
console.log(post instanceof Post); // true
await post.destroy(); // 只设置 `deletedAt` 标志
await post.destroy({ force: true }); // 真的会删除记录
```
### 恢复
要恢复软删除的记录,可以使用 restore 方法,该方法在静态版本和实例版本中都提供：
```
// 展示实例 `restore` 方法的示例
// 我们创建一个帖子,对其进行软删除,然后将其还原
const post = await Post.create({ title: 'test' });
console.log(post instanceof Post); // true
await post.destroy();
console.log('soft-deleted!');
await post.restore();
console.log('restored!');

// 展示静态 `restore` 方法的示例.
// 恢复每个 likes 大于 100 的软删除的帖子
await Post.restore({
  where: {
    likes: {
      [Op.gt]: 100
    }
  }
});
```
### 其他查询行为
Sequelize 执行的每个查询将自动忽略软删除的记录(当然,原始查询除外).  
这意味着,例如,``findAll`` 方法将看不到软删除的记录,仅获取未删除的记录.  
即使你单纯的调用提供了软删除记录主键的findByPk,结果也将是 null,就好像该记录不存在一样.  
如果你真的想让查询看到被软删除的记录,可以将 paranoid: false 参数传递给查询方法. 例如：
```
await Post.findByPk(123); // 如果 ID 123 的记录被软删除,则将返回 `null`
await Post.findByPk(123, { paranoid: false }); // 这将检索记录

await Post.findAll({
  where: { foo: 'bar' }
}); // 这将不会检索软删除的记录

await Post.findAll({
  where: { foo: 'bar' },
  paranoid: false
}); // 这还将检索软删除的记录
```
## 原始查询
由于常常使用简单的方式来执行原始/已经准备好的SQL查询,因此可以使用 ``sequelize.query`` 方法.  
默认情况下,函数将返回两个参数 - 一个结果数组,以及一个包含元数据(例如受影响的行数等)的对象. 
```
const [results, metadata] = await sequelize.query("UPDATE users SET y = 42 WHERE x = 12");
// 结果将是一个空数组,元数据将包含受影响的行数.
```
在不需要访问元数据的情况下,你可以传递一个查询类型来告诉后续如何格式化结果. 例如,对于一个简单的选择查询你可以做:
```
const { QueryTypes } = require('sequelize');
const users = await sequelize.query("SELECT * FROM `users`", { type: QueryTypes.SELECT });
// 我们不需要在这里分解结果 - 结果会直接返回
```
**第二种选择是模型. 如果传递模型,返回的数据将是该模型的实例.**
```
// Callee 是模型定义. 这样你就可以轻松地将查询映射到预定义的模型
const projects = await sequelize.query('SELECT * FROM projects', {
  model: Projects,
  mapToModel: true // 如果你有任何映射字段,则在此处传递 true
});
// 现在,`projects` 的每个元素都是 Project 的一个实例
```
查看 ``Query API`` 参考中的更多参数. 以下是一些例子:
```
const { QueryTypes } = require('sequelize');
await sequelize.query('SELECT 1', {
  // 用于记录查询的函数(或false)
  // 将调用发送到服务器的每个SQL查询.
  logging: console.log,

  // 如果plain为true,则sequelize将仅返回结果集的第一条记录. 
  // 如果是false,它将返回所有记录.
  plain: false,

  // 如果你没有查询的模型定义,请将此项设置为true.
  raw: false,

  // 你正在执行的查询类型. 查询类型会影响结果在传回之前的格式.
  type: QueryTypes.SELECT
});

// 注意第二个参数为null！
// 即使我们在这里声明了一个被调用对象,
// raw: true 也会取代并返回一个原始对象.

console.log(await sequelize.query('SELECT * FROM projects', { raw: true }));
```
### 其他参考官网
## [进阶](https://www.sequelize.com.cn/advanced-association-concepts/eager-loading)
## demo
### 初始化数据库
```
const sequelize = require('./util/database'); 
const Product = require('./models/product'); 
const User = require('./models/user'); 
const Cart = require('./models/cart'); 
const CartItem = require('./models/cart-item'); 
const Order = require('./models/order'); 
const OrderItem = require('./models/order-item');
Product.belongsTo(User, {
    constraints: true,
    onDelete: 'CASCADE'
});
User.hasMany(Product);
User.hasOne(Cart);
Cart.belongsTo(User);
Cart.belongsToMany(Product, {
    through: CartItem
});
Product.belongsToMany(Cart, { 
    through: CartItem 
});
Order.belongsTo(User);
User.hasMany(Order);
Order.belongsToMany(Product, {
    through: OrderItem
});
Product.belongsToMany(Order, {
    through: OrderItem 
});
```
### 同步数据
```
sequelize.sync().then(
    async result => {
        let user = await User.findByPk(1)
        if (!user) {
            user = await User.create({
                name: 'Sourav',
                email: 'sourav.dey9@gmail.com'
            })
            await user.createCart();
        }
        app.listen(3000, () => console.log("Listening to port 3000"));
    })
```
### 中间件鉴权
```
app.use(async (ctx, next) => { 
    const user = await User.findByPk(1)
    ctx.user = user; 
    await next(); 
});
```
### 功能实现
```
const router = require('koa-router')()
// 查询产品
router.get('/admin/products', async (ctx, next) => {
    const products = await Product.findAll()
    ctx.body = { prods: products }
})
// 创建产品
router.post('/admin/product', async ctx => {
    const body = ctx.request.body
    const res = await ctx.user.createProduct(body)
    ctx.body = { success: true }
})
// 删除产品
router.delete('/admin/product/:id', async (ctx, next) => {
    const id = ctx.params.id
    const res = await Product.destroy({
        where: {
            id
        }
    })
    ctx.body = { success: true }
})
// 查询购物车
router.get('/cart', async ctx => {
    const cart = await ctx.user.getCart()
    const products = await cart.getProducts()
    ctx.body = { products }
})
// 添加购物车
router.post('/cart', async ctx => {
    const { body } = ctx.request
    const prodId = body.id
    let newQty = 1
    const cart = await ctx.user.getCart()
    const products = await cart.getProducts({
        where: {
            id: prodId
        }
    })
    let product
    if (products.length > 0) { 
        product = products[0] 
    }
    if (product) { 
        const oldQty = product.cartItem.quantity 
        newQty = oldQty + 1 
    } else { 
        product = await Product.findByPk(prodId) 
    }
    await cart.addProduct(product, {
        through: {
            quantity: newQty
        }
    })
    ctx.body = { success: true }
})
// 添加订单
router.post('/orders', async ctx => { 
    const cart = await ctx.user.getCart() 
    const products = await cart.getProducts() 
    const order = await ctx.user.createOrder()
    const result = await order.addProduct( 
        products.map(p => { 
            p.orderItem = { 
                quantity: p.cartItem.quantity 
            }
            return p 
        }) 
    )
    await cart.setProducts(null) 
    ctx.body = { success: true }
})
// 删除购物车
router.delete('/cartItem/:id', async ctx => { 
    const id = ctx.params.id 
    const cart = await ctx.user.getCart() 
    const products = await cart.getProducts({ 
        where: { id } 
    })
    const product = products[0] 
    await product.cartItem.destroy() 
    ctx.body = { success: true } 
})
// 查询订单
router.get('/orders', async ctx => { 
    const orders = await ctx.user.getOrders({
         include: ['products'], 
         order: [['id', 'DESC']] 
    }) 
    ctx.body = { orders } 
})
app.use(router.routes())
```