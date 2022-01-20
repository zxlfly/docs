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
```
INSERT INTO websites(name, url,alexa,sal,country ) VALUES ('腾讯', 'https://www.qq.com',
18, 1000,'CN' );
```
### 删除语句
```
delete from websites where id = 5;
```
### 更新语句
```
update websites set sal = null where id = 3
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
is null is not null (因为在sql 语句中null 和任何东西比较都是假，包括它本身)  
like in  
``select * from websites where sal >500``

## 逻辑条件: and 、or
注意： null 的条件判断用 is null 或 is not null  
```
select * from websites where sal >= 0 and sal <=2000 ; -- 收入在 0 到 2000 之间
select * from websites where sal between 0 and 2000； -- 和上面一样的，没事找事
select * from websites where sal < 5 or sal is null ; -- 收入小于5 或者没收入
```
## order by
排序: 默认情况下是升序，asc 可以省略 。
``select * from websites order by sal asc,alexa desc ;--先根据sal 升序排序，再根据 alexa 降序``

## like 和 通配符
- like 模糊查询
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