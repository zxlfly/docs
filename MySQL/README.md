# MySQL
## MySQL的数据类型
- MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。
- char 和varchar 一定要指定长度，float 会自动提升为double，timestamp 是时间的混合类型，理论上可以存储 时间格式和时间戳。

| 类型          | 描述                               |
| :------------ | :--------------------------------- |
| ``int``       | 整型                               |
| ``bigint``    | 整型（大数）                       |
| ``float``     | 浮点型                             |
| ``double``    | 浮点型                             |
| ``datetime``  | 日期类型                           |
| ``timestamp`` | 日期类型（可存储时间戳）           |
| ``char``      | 定长字符                           |
| ``varchar``   | 不定长字符                         |
| ``text``      | 大文本，用于存储很长的字符内容     |
| ``blob``      | 字节数据类型，存储图片、音频等文件 |
| ...           | ...                                |
## 创建数据库操作
```
// 创建
CREATE DATABASE 数据库名;
// 删除
drop database <数据库名>;
// 选择数据库
use RUNOOB;
```
## 建表操作
```
-- 删除表
DROP TABLE IF EXISTS 表名;
-- 新建表
create table 表名(
字段名 类型 约束（主键，非空，唯一，默认值），
字段名 类型 约束（主键，非空，唯一，默认值），
)编码，存储引擎；
```
### 在 SQL 中，我们有如下约束：
- ``NOT NULL`` - 指示某列不能存储 NULL 值。
- ``UNIQUE`` - 保证某列的每行必须有唯一的值。
- ``PRIMARY KEY`` - ``NOT NULL`` 和 ``UNIQUE`` 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
- ``FOREIGN KEY`` - 保证一个表中的数据匹配另一个表中的值的参照完整性。
- ``CHECK`` - 保证列中的值符合指定的条件。
- ``DEFAULT`` - 规定没有给列赋值时的默认值。

```
DROP TABLE IF EXISTS `websites`;
CREATE TABLE `websites` (
  id int(11) NOT NULL AUTO_INCREMENT,
  name char(20) NOT NULL DEFAULT '' COMMENT '站点名称',
  url varchar(255) NOT NULL DEFAULT '',
  alexa int(11) NOT NULL DEFAULT '0' COMMENT 'Alexa 排名',
  sal double COMMENT '广告收入',
  country char(10) NOT NULL DEFAULT '' COMMENT '国家',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 插入、删除、更新
### 插入语句
如果数据是字符型，必须使用单引号或者双引号，如："value"。
```
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( value1, value2,...valueN );
```
### 删除语句
- 如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。
- 你可以在 WHERE 子句中指定任何条件
- 您可以在单个表中一次性删除记录。
```
delete from table_name where id = 5;

DELETE FROM table_name [WHERE Clause]
```
### 更新语句
- 你可以同时更新一个或多个字段。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以在一个单独表中同时更新数据。
```
update table_name set sal = null where id = 3

UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]
```

## 基本 select 查询语句
### 初始化数据
```
DROP TABLE IF EXISTS `websites`;
CREATE TABLE `websites` (
  id int(11) NOT NULL AUTO_INCREMENT,
  name char(20) NOT NULL DEFAULT '' COMMENT '站点名称',
  url varchar(255) NOT NULL DEFAULT '',
  alexa int(11) NOT NULL DEFAULT '0' COMMENT 'Alexa 排名',
  sal double COMMENT '广告收入',
  country char(10) NOT NULL DEFAULT '' COMMENT '国家',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO `websites` VALUES
  (1, 'Google', 'https://www.google.cm/', '1', 2000,'USA'),
  (2, '淘宝', 'https://www.taobao.com/', '13',2050, 'CN'),
  (3, '菜鸟教程', 'http://www.runoob.com/', '4689',0.0001, 'CN'),
  (4, '微博', 'http://weibo.com/', '20',50, 'CN'),
  (5, 'Facebook', 'https://www.facebook.com/', '3', 500,'USA');
CREATE TABLE IF NOT EXISTS `access_log` (
  `aid` int(11) NOT NULL AUTO_INCREMENT,
  `site_id` int(11) NOT NULL DEFAULT '0' COMMENT '网站id',
  `count` int(11) NOT NULL DEFAULT '0' COMMENT '访问次数',
  `date` date NOT NULL,
  PRIMARY KEY (`aid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO `access_log` (`aid`, `site_id`, `count`, `date`) VALUES
  (1, 1, 45, '2016-05-10'),
  (2, 3, 100, '2016-05-13'),
  (3, 1, 230, '2016-05-14'),
  (4, 2, 10, '2016-05-14'),
  (5, 5, 205, '2016-05-14'),
  (6, 4, 13, '2016-05-15'),
  (7, 3, 220, '2016-05-15'),
  (8, 5, 545, '2016-05-16'),
  (9, 3, 201, '2016-05-17'),
  (10, 88, 9999, '2016-09-09');

```
### 查询语句
```
select * from websites
select id, name, url, alexa, sal, country from websites （推荐使用的方式）
```

### 分页查询
```
select * from websites limit 2,3 ; --从第2条(下标从0开始)开始查，查3条数据
select * from websites limit 3 ; --从第0条(下标从0开始)开始查，查3条数据
```

### distinct 关键字
DISTINCT 关键词用于返回唯一不同的值。``select distinct country from websites``

## where 语句
作为条件筛选, 运算符: > < >= <= <> != =  
IS NULL: 当列的值是 NULL,此运算符返回 true。  
IS NOT NULL: 当列的值不为 NULL, 运算符返回 true。  
<=>: 比较操作符（不同于 = 运算符），当比较的的两个值相等或者都为 NULL 时返回 true  
关于 NULL 的条件比较运算是比较特殊的。你不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。  
在 MySQL 中，NULL 值与任何其它值的比较（即使是 NULL）永远返回 NULL，即 NULL = NULL 返回 NULL 。  
MySQL 中处理 NULL 使用 IS NULL 和 IS NOT NULL 运算符。  
like in  
``select * from websites where sal >500``
- 查询语句中你可以使用一个或者多个表，表之间使用逗号, 分割，并使用WHERE语句来设定查询条件。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以使用 AND 或者 OR 指定一个或多个条件。
- WHERE 子句也可以运用于 SQL 的 DELETE 或者 UPDATE 命令。
- WHERE 子句类似于程序语言中的 if 条件，根据 MySQL 表中的字段值来读取指定的数据。
```
SELECT field1, field2,...fieldN FROM table_name1, table_name2...
[WHERE condition1 [AND [OR]] condition2.....
```
## 逻辑条件: and 、or
注意： null 的条件判断用 is null 或 is not null  
```
select * from websites where sal >= 0 and sal <=2000 ; -- 收入在 0 到 2000 之间
select * from websites where sal between 0 and 2000； -- 和上面一样的，没事找事
select * from websites where sal < 5 or sal is null ; -- 收入小于5 或者没收入
```
## order by
- 你可以使用任何字段来作为排序的条件，从而返回排序后的查询结果。
- 你可以设定多个字段来排序。
- 你可以使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。
- 你可以添加 WHERE...LIKE 子句来设置条件。
```
select * from websites order by sal asc,alexa desc ;--先根据sal 升序排序，再根据 alexa 降序


SELECT field1, field2,...fieldN FROM table_name1, table_name2...
ORDER BY field1 [ASC [DESC][默认 ASC]], [field2...] [ASC [DESC][默认 ASC]]
```

## like 和 通配符
- like 模糊查询
  - LIKE 通常与 % 一同使用，类似于一个元字符的搜索。
  - 可以使用 AND 或者 OR 指定一个或多个条件。
  - 可以在 DELETE 或 UPDATE 命令中使用 WHERE...LIKE 子句来指定条件。
  -  LIKE 子句中使用百分号 %字符来表示任意字符，类似于UNIX或正则表达式中的星号 *。
  -  没有使用百分号 %, LIKE 子句与等号 = 的效果是一样的。
- 通配符
  - ``%`` ： 0个或多个字符
  - ``_`` : 1 个字符

## in
匹配多个条件``select * from websites where country in ('USA','鸟国','CN');``  
等价于``select * from websites where country = 'USA' or country = '鸟国' or country = 'CN'``

## 别名
``select tt.name '网站名字' from websites tt``

## Group by 分组查询
注意：分组时候的筛选用 having  
常见的几个组函数： max() min() avg() count() sum()
```
select avg(sal) aa from websites where sal is not null group by country having aa > 1500
```

## 子查询
把查询的结果当作一个表来使用

## 连接查询
```
select name,count,date from websites w , access_log a ; --著名的笛卡尔积，没什么意义的
select name,count,date from websites w , access_log a where w.id = a.site_id；-- 这是 1992
的语法
select name,count,date from websites w inner join access_log a on w.id = a.site_id；-- 这是
1999 年的语法，推荐使用
select name,count,date from websites w left outer join access_log a on w.id = a.site_id；--
把没有访问的网站也显示出来
-- 注意: inner 和 outer 是可以默认省略的
```

## Null 处理l 函数
```
select name,ifnull(count,0),ifnull(date,'') from websites w left outer join access_log a on
w.id = a.site_id
```