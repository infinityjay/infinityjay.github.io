---
categories:
  - Cloud Computing
title:  MySQL notes
tags:   [MySQL]
mathjax: false
---
## description
MySQL必知必会笔记
## 目录
- [目录](#目录)
- [了解SQL](#了解sql)
- [MySQL查询语句](#mysql查询语句)
  - [检索列](#检索列)
  - [排序检索](#排序检索)
  - [过滤数据](#过滤数据)
  - [正则表达式](#正则表达式)
  - [常用函数](#常用函数)
  - [分组数据](#分组数据)
  - [子查询](#子查询)
  - [联结表](#联结表)
- [MySQL插入语句](#mysql插入语句)
- [MySQL更新和删除语句](#mysql更新和删除语句)
- [创建和操纵表](#创建和操纵表)
- [存储程序](#存储程序)
- [管理事务处理](#管理事务处理)
- [安全管理](#安全管理)

## 了解SQL

> 相关名词解释

* 数据库（database）：保存有组织的数据的容器
* 表（table）：某种特定类型数据的结构化清单
* 模式（schema）：关于数据库和表的布局及特性的信息
* 列(column) ：表中的一个字段，所有表都是由一个或多个列组成的。
* 数据类型(datatype) ：所容许的数据的类型。每个表列都有相 应的数据类型，它限制(或容许)该列中存储的数据。
* 行(row) ：表中的一个记录。
* 主键(primary key)：一列(或一组列)，其值能够唯一区分表 中每个行。

## MySQL查询语句

### 检索列

```mysql
# 使用数据库
use 【数据库名】

# 查找数据库
show databases;

# 获取数据库内表
show tables;

# 显示表列
show colums from customers;

# 检索列
select prod_name, prod_price from products;

# 检索列中所有数值不同的值
select distinct id from products;

# 指定开始行(5)和行数(10)
select prod_name from products limit 5,10;

# 使用限定的表名或者数据库名
select products.prod_name from crashcourse.products;
```



### 排序检索

```mysql
# 使用 order by 进行排序（默认升序排序）
select prod_name from products order by prod_name;

# 多个关键字排序
select prod_name from products order by prod_price, prod_name;

# 指定排序方向，DESC - 降序排序
select prod_id, prod_name from products order by prod_price DESC, prod_name; # 此处DESC只对价格有效

# 返回一列中最大值或者最小值，此处返回最贵的物品
select prod_price from products order by prod_price DESC limit 1;
```



### 过滤数据

```mysql
# 使用 where 子句
select prod_name, prod_price from products where prod_price = 2.5;
select prod_name, prod_price from products where prod_price < 10;
select prod_name, prod_price from products where prod_price <= 10;

# 不匹配检查，找出不是1003生产的产品
select prod_name, vend_id from products where vend_id <> 1003;
select prod_name, vend_id from products where vend_id != 1003;

# 使用 where 。。between。。and。。 操作符，为闭区间
select prod_name, prod_price from products where prod_price between 5 and 10;

# 使用 IS NULL 进行空值检查
select prod_name, prod_price from products where prod_price IS NULL;

# 使用 where 。。AND 组合查询(可以使用多个 and )
select prod_id, prod_name, prod_price from products where vend_id = 1003 and prod_price <= 10;

# 使用 where。。or。。组合查询
select prod_id, prod_name, prod_price from products where vend_id = 1003 or vend_id = 1002;

# 使用括号来指定优先级
select prod_name, prod_price from products where (vend_id = 1003 or vend_id = 1002) and prod_price >= 10;

# 使用 where。。in。。来指明过滤清单，等同于 or ，更适用于较多选项
select prod_name, prod_price from products where vend_id in (1002,1003);

# 使用 where。。not in。。来排除选项
select prod_name, prod_price from products where vend_id not in (1002,1003);

# 使用 LIKE + % 通配符进行过滤, % 表示任意字符出现任意次数（包括0个字符）
select prod_name, prod_price from products where prod_name like 'jet%';

# 使用 LIKE + _ 通配符进行过滤，_ 表示匹配单个字符
select prod_name, prod_price from products where prod_name like '_ ton';

# 使用正则表达式，操作符 REGEXP 
# . 在正则表达式中表示匹配任意一个字符
select prod_name, prod_price from products where prod_name regexp '.000';
# ｜ 在正则表达式中表示 or 操作符
select prod_name, prod_price from products where prod_name regexp '1000｜2000';
# [123] 在正则表达式中表示在一组字符中匹配其中一个,等同于 1｜2｜3
select prod_name, prod_price from products where prod_name regexp '[123] Ton';
# [1-5] 匹配1-5中任意一个字符，.5 Ton 也会匹配并返回
select prod_name, prod_price from products where prod_name regexp '[1-5] Ton';
# 匹配特殊字符. [] | _ 需要使用转义符 \\, 为了匹配 \ 本身，需要使用 \\ 
select prod_name, prod_price from products where prod_name regexp '\\.';
```

### 正则表达式

> 正则表达式中匹配单个字符

| 预定义字符类 | 说明                                            |
| :----------: | :---------------------------------------------- |
|  [:alnum:]   | 任意字母和数字(同[a-zA-Z0-9])                   |
|  [:alpha:]   | 任意字符(同[a-zA-Z])                            |
|  [:blank:]   | 空格和制表(同[\\t])                             |
|  [:cntrl:]   | ASCII控制字符(ASCII 0到31和127)                 |
|  [:digit:]   | 任意数字(同[0-9])                               |
|  [:graph:]   | 与[:print:]相同，但不包括空格                   |
|  [:lower:]   | 任意小写字母(同[a-z])                           |
|  [:print:]   | 任意可打印字符                                  |
|  [:punct:]   | 既不在[:alnum:]又不在[:cntrl:]中的任意字符      |
|  [:space:]   | 包括空格在内的任意空白字符(同[\\f\\n\\r\\t\\v]) |
|  [:upper:]   | 任意大写字母(同[A-Z])                           |
|  [:xdigit:]  | 任意十六进制数字(同[a-fA-F0-9])                 |

>  正则表达式中匹配多个字符

| 元字符 | 说明                       |
| :----: | :------------------------- |
|   *    | 0个或多个匹配              |
|   +    | 1个或多个匹配(等于{1,})    |
|   ?    | 0个或1个匹配(等于{0,1})    |
|  {n}   | 指定数目的匹配             |
|  {n,}  | 不少于指定数目的匹配       |
| {n,m}  | 匹配数目的范围(m不超过255) |

```mysql
# 正则表达式\\([0-9] sticks?\\)需要解说一下。\\(匹配)，[0-9]匹配任意数字(这个例子中为1和5)，sticks?匹配stick 和sticks(s后的?使s可选，因为?匹配它前面的任何字符的0次或1次出现)，\\)匹配)。没有?，匹配stick和sticks会非常困难。
select prod_name from products where prod_name REGEXP '\\([0-9] sticks?\\)';
# 输出
TNT (1 stick)
TNT (5 sticks)

# 如前所述，[:digit:]匹配任意数字，因而它为数字的一个集合。{4}确切地要求它前面的字符(任意数字)出现4次，所以 [[:digit:]]{4}匹配连在一起的任意4位数字
select prod_name from products where prod_name REGEXP '[[:digit:]]{4}';
# 输出
JetPack 1000
JetPack 2000
```

> 正则表达式中定位符

| 定位元字符 | 说明       |
| :--------: | :--------- |
|     ^      | 文本的开始 |
|     $      | 文本的结尾 |
|  [[:<:]]   | 词的开始   |
|  [[:>:]]   | 词的结尾   |

```mysql
# ^匹配串的开始。因此，^[0-9\\.]只在.或任意数字为串中第一个字符时才匹配它们。没有^，则还要多检索出4个别的行(那些中间有数字的行)。
select prod_name from products where prod_name REGEXP '^[0-9\\.]';
# 输出
.5 ton anvil
1 ton anvil
2 ton anvil
```

### 常用函数

```mysql
# 在select的时候执行计算语句
select prod_id, quantity, item_price, quantity*item_price AS expanded_price from orderitems where order_num = 2005;

# 使用Concat（）拼接串，即把多个串连接起来形成一个较长的串,各个串之间用逗号分隔。
 select Concat(vend_name, '(', vend_contry, ')') from vendors order by vend_name;
# 使用 AS 设置别名，以方便引用
select Concat(vend_name, '(', vend_contry, ')') AS vend_title from vendors order by vend_name;

# Upper()，将文本转化为大写
select vend_name, Upper(vend_name) AS vend_name_upper from vendors;

# SOUNDEX是一个将任何文 本串转换为描述其语音表示的字母数字模式的算法。SOUNDEX考虑了类似 的发音字符和音节，使得能对串进行发音比较而不是字母比较。
select cust_name, cust_contact from customers where Soundex(cust_contact) = Soundex('Y. Lie');
# 结果
Coyote Inc.  Y Lee
```

> 日期和时间处理函数

|     函数      | 说明                           |
| :-----------: | :----------------------------- |
|   AddDate()   | 增加一个日期(天、周等)         |
|   AddTime()   | 增加一个时间(时、分等)         |
|   CurDate()   | 返回当前日期                   |
|   CurTime()   | 返回当前时间                   |
|    Date()     | 返回日期时间的日期部分         |
|  DateDiff()   | 计算两个日期之差               |
|  Date_Add()   | 高度灵活的日期运算函数         |
| Date_Format() | 返回一个格式化的日期或时间串   |
|     Day()     | 返回一个日期的天数部分         |
|  DayOfWeek()  | 对于一个日期，返回对应的星期几 |
|    Hour()     | 返回一个时间的小时部分         |
|   Minute()    | 返回一个时间的分钟部分         |
|    Month()    | 返回一个日期的月份部分         |
|     Now()     | 返回当前日期和时间             |
|   Second()    | 返回一个时间的秒部分           |
|    Time()     | 返回一个日期时间的时间部分     |
|    Year()     | 返回一个日期的年份部分         |

```mysql
# 举例
select cust_id, order_num from orders where Data(order_date) = '2005-09-01';
# 查找所有9月份的订单
select cust_id, order_num from orders where Data(order_date) between '2005-09-01' and '2005-09-30';
select cust_id, order_num from orders where year(order_date)=2005 and Month(order_date)=9;
```



> 数值处理函数

|  函数  | 说明               |
| :----: | :----------------- |
| Abs()  | 返回一个数的绝对值 |
| Cos()  | 返回一个角度的余弦 |
| Exp()  | 返回一个数的指数值 |
| Mod()  | 返回除操作的余数   |
|  Pi()  | 返回圆周率         |
| Rand() | 返回一个随机数     |
| Sin()  | 返回一个角度的正弦 |
| Sqrt() | 返回一个数的平方根 |
| Tan()  | 返回一个角度的正切 |

> 聚集函数

|  函数   | 说明             |
| :-----: | :--------------- |
|  AVG()  | 返回某列的平均值 |
| COUNT() | 返回某列的行数   |
|  MAX()  | 返回某列的最大值 |
|  MIN()  | 返回某列的最小值 |
|  SUM()  | 返回某列值之和   |

```mysql
# 举例
select AVG(prod_price) AS avg_price from products;

select SUM(item_price*quantity) AS total_price from orderitems where order_num = 20005;
```

### 分组数据

```mysql
# group by 进行分组
select vend_id, COUNT(*) AS num_prods from products group by vend_id;
```

上面的SELECT语句指定了两个列，vend_id包含产品供应商的ID，num_prods为计算字段(用COUNT(*)函数建立)。GROUP BY子句指 示MySQL按vend_id排序并分组数据。这导致对每个vend_id而不是整个表 计算num_prods一次。从输出中可以看到，供应商1001有3个产品，供应商 1002有2个产品，供应商1003有7个产品，而供应商1005有2个产品。

因为使用了GROUP BY，就不必指定要计算和估值的每个组了。系统 会自动完成。GROUP BY子句指示MySQL分组数据，然后对每个组而不是 整个结果集进行聚集。

*  GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套， 为数据分组提供更细致的控制。

*  如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上 进行汇总。换句话说，在建立分组时，指定的所有列都一起计算(所以不能从个别的列取回数据)。

* GROUP BY子句中列出的每个列都必须是检索列或有效的表达式(但不能是聚集函数)。如果在SELECT中使用表达式，则必须在 GROUP BY子句中指定相同的表达式。不能使用别名。

* 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子 句中给出。

* 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列 中有多行NULL值，它们将分为一组。

* <font color = red>GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。</font>

```mysql
# 使用 having 过滤分组，所有类型的where子句都可以用 having 来代替
select cust_id, COUNT(*) as orders from orders group by cust_id having count(*) >= 2;

select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having count(*) >= 2;

select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having count(*) >= 2 order by vend_id;
```

<font color = red>可以理解为 where 在数据分组之前进行过滤，因此 group by 需要在 where 之后，而having是对分组进行过滤。</font>

|   子句   | 说明               | 是否必须使用           |
| :------: | ------------------ | ---------------------- |
|  SELECT  | 要返回的列或表达式 | 是                     |
|   FROM   | 从中检索数据的表   | 仅在从表选择数据时使用 |
|  WHERE   | 行级过滤           | 否                     |
| GROUP BY | 分组说明           | 仅在按组计算聚集时使用 |
|  HAVING  | 组级过滤           | 否                     |
| ORDER BY | 输出排序顺序       | 否                     |
|  LIMIT   | 要检索的行数       | 否                     |

### 子查询

```mysql
select cust_id from orders where order_num in (select order_num from orderitems where prod_id = 'TNT2');

# 两张表查询需要完全限定列名
select cust_name,
			 cust_state,
			 (select count(*)
        from orders
        where orders.cust_id = customers.cust_id)
as orders
from customers
order by cust_name;
```

### 联结表

> 自联结

```mysql
select p1.prod_id, p1.prod_name 
from products as p1, products as p2
where p1.vend_id = p2.vend_id
and p2.prod_id = 'DTNTR';
```

> 自然联结

```mysql
# 在这个例子中，通配符只对第一个表使用。所有其他列明确列出，所以没有重复的列被检索出来.
select c.*, o.order_num, o.order_date, oi.prod_id, oi_quantity, OI.item_price
from customers as c, orders as o, orderitems as oi
where c.cust_id = o.cust_id
and oi.order_num = o.order_num
and prod_id = 'FB';
```

> 外部联结

```mysql
# 左联结，主表为 outer join 左边的表，结果中包括主表 customers 中的所有行
select customers.cust_id, orders.order_num
from customers left outer join orders
on customers.cust_id = order.cust_id;

# 右联结，主表为 outer join 右边的表，结果中包括主表 orders 中的所有行
select customers.cust_id, orders.order_num
from customers right outer join orders
on customers.cust_id = order.cust_id;
```

## MySQL插入语句

```mysql
# insert 插入语句(依赖于插入顺序)
insert into customers
values(NULL, 'Pep E. LaPew', '100 Main Street');

# 明确给出列名，方便对应
insert into customers(cust_contact, cust_name, cust_address)
values(NULL, 'Pep E. LaPew', '100 Main Street');

# 插入多行数据，可以写多个命令或者如下
insert into customers(cust_contact, cust_name, cust_address)
values(NULL, 'Pep E. LaPew', '100 Main Street'),
			(NULL, 'M. Martin', '42 Galaxy Way');
			
# 插入检索出来的数据
insert into customers(cust_contact, cust_name, cust_address)
select cust_contact, cust_name, cust_address
from custnew;
```

## MySQL更新和删除语句

```mysql
# update 语句更新数据
update customers
set cust_email = 'elmer@fudd.com'
where cust_id = 10005;
```

<font color = red>注意：确定 where 后面的条件，如果不指定 where 会更新整个表</font>

```mysql
# delete 语句删除数据
delete from customers
where cust_id = 10006;
```

## 创建和操纵表

```mysql
# 创建表 create
create table customers
(
  cust_id			int				NOT NULL Auto_INCREMENT,
  cust_name		char(50)	NOT NULL ,
  cust_email	char(255)	NULL ,
  PRIMARY KEY (cust_id)
)ENGINE = InnoDB;
```

* `Auto_INCREMENT` 

  AUTO_INCREMENT告诉MySQL，本列每当增加一行时自动增量。每次 执行一个INSERT操作时，MySQL自动对该列增量(从而才有这个关键字 AUTO_INCREMENT)，给该列赋予下一个可用的值。这样给每个行分配一个 唯一的cust_id，从而可以用作主键值。每个表只允许一个AUTO_INCREMENT列，而且它必须被索引(如，通过使它成为主键)。

* 引擎类型

  * I n n o D B 是 一 个 可 靠 的 事 务 处 理 引 擎 ( 参 见 第 2 6 章 )， 它 不 支 持 全 文 本搜索;

  * MEMORY在功能等同于MyISAM，但由于数据存储在内存(不是磁盘) 中，速度很快(特别适合于临时表);
  * MyISAM是一个性能极高的引擎，它支持全文本搜索(参见第18章)， 但不支持事务处理。

```mysql
# 更新表 alter
# 增加一个列
alter table vendors
add vend_phone		CHAR(20);

# 删除一个列
alter table vendors
DROP column vend_phone;
```

```mysql
# 删除表
drop table customers2;

# 重命名表
rename table customers2 to customers;
```

## 存储程序

存储过程简单来说，就是为以后的使用而保存 的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用 不仅限于批处理。

使用存储过程有3个主要的好处，即简单、安全、高性能。

```mysql
# 示例 创建存储程序
create procedure productpricing()
BEGIN
		select avg(prod_price) as priceaverage
		from products;
end;
# 调用存储程序
call productpricing();
# 输出
priceaverage
16.133517

# 删除存储程序
drop procedure productpricing;
```

> 使用参数存储

```mysql
# 例子
# 此存储过程接受3个参数:pl存储产品最低价格，ph存储产品最高价格，pa存储产品平均价格。每个参数必须具有指定的类 型，这里使用十进制值。关键字OUT指出相应的参数用来从存储过程传出 一个值(返回给调用者)。MySQL支持IN(传递给存储过程)、OUT(从存 储过程传出，如这里所用)和INOUT(对存储过程传入和传出)类型的参数。存储过程的代码位于BEGIN和END语句内，如前所见，它们是一系列 SELECT语句，用来检索值，然后保存到相应的变量(通过指定INTO关键字)。
create procedure productpricing(
  OUT pl DECIMAL(8,2),
  OUT ph DECIMAL(8,2),
  OUT pa DECIMAL(8,2)
)
begin
	select Min(prod_price)
	INTO pl
	from products;
	select Max(prod_price)
	INTO ph
	from products;
	select Avg(prod_price)
	INTO pa
	from products;
end;	

# 变量传递参数
call productpricing(
  @pricelow,
  @pricehigh,
  @priceaverage);
  
# 调用存储程序
select @pricehigh, @pricelow, @priceaverage;


# 例子2
# onumber定义为IN，因为订单号被传入存储过程。ototal定义为OUT，因为要从存储过程返回合计。SELECT语句使用这两个 参数，WHERE子句使用onumber选择正确的行，INTO使用ototal存储计算 出来的合计。
create procedure ordertotal(
  IN onumber INT,
  OUT ototal DECIMAL(8,2)
)
begin
	select Sum(item_price*quantity)
	from orderitems
	where order_num = onumber
	INTO ototal;
end;
# 必须给ordertotal传递两个参数;第一个参数为订单号，第二 个参数为包含计算出来的合计的变量名。
call ordertotal(20005, @total);
# 调用存储程序
select @total;

```

## 管理事务处理

并非所有引擎都 支持明确的事务处理管理。MyISAM和InnoDB是两种最常使用 的引擎。前者不支持明确的事务处理管理，而后者支持。这 就是为什么本书中使用的样例表被创建来使用InnoDB而不是 更经常使用的MyISAM的原因。如果你的应用中需要事务处理 功能，则一定要使用正确的引擎类型。

事务处理(transaction processing)可以用来维护数据库的完整性，它 保证成批的MySQL操作要么完全执行，要么完全不执行。

* 事务(transaction)指一组SQL语句;
* 回退(rollback)指撤销指定SQL语句的过程;
* 提交(commit)指将未存储的SQL语句结果写入数据库表;
* 保留点(savepoint)指事务处理中设置的临时占位符(place-holder)，你可以对它发布回退(与回退整个事务处理不同)

```mysql
# 控制事务处理
# 管理事务处理的关键在于将SQL语句组分解为逻辑块，并明确规定数 据何时应该回退，何时不应该回退。
# 开始事务
START TRANSACTION
# 使用ROLLBACK
# 这个例子从显示ordertotals表(此表在第24章中填充)的内容开始。首先执行一条SELECT以显示该表不为空。然后开始一 个事务处理，用一条DELETE语句删除ordertotals中的所有行。另一条 SELECT语句验证ordertotals确实为空。这时用一条ROLLBACK语句回退 START TRANSACTION之后的所有语句，最后一条SELECT语句显示该表不为空。显然，ROLLBACK只能在一个事务处理内使用(在执行一条START TRANSACTION命令之后)
select * from ordertotals;
START TRANSACTION;
DELETE from ordertotals;
select * from ordertotals;
rollback;
select * from ordertotals;

# commit提交
# 在事务处理块中，提交不会隐含地进行。为进行明确的提交， 使用COMMIT语句
# 在这个例子中，从系统中完全删除订单20010。因为涉及更新两个数据库表orders和orderItems，所以使用事务处理块来 保证订单不被部分删除。最后的COMMIT语句仅在不出错时写出更改。如 果第一条DELETE起作用，但第二条失败，则DELETE不会提交(实际上， 它是被自动撤销的)。
start transaction;
delete from orderitems where order_num = 20010;
delete from orders where order_num = 20010;
commit;

# 使用保留点
# 每个保留点都取标识它的唯一名字，以便在回退时，MySQL知道要 回退到何处。保留点在事务处理完成(执行一条ROLLBACK或 COMMIT)后自动释放。自MySQL 5以来，也可以用RELEASE SAVEPOINT明确地释放保留点。
savepoint delete1;
rollback to delete1;
```



## 安全管理

> 用户管理

```mysql
# 获取所有用户账号列表
use mysql;
select user from user;

# 创建账号
create user ben IDENTIFIED BY '123456';

# 删除用户账号
drop user ben;
```

> 访问权限

```mysql
# 显示用户的权限
show GRANTS for ben;

# 设置访问权限
# 此GRANT允许用户在crashcourse.*(crashcourse数据库的所有表)上使用SELECT。通过只授予SELECT访问权限，用户ben 对crashcourse数据库中的所有数据具有只读访问权限.
GRANTS select on crashcourse.* to ben;

# 撤销权限
REVOKE select on on crashcourse.* from ben;
```

> 更改密码

```mysql
# set password
set password for ben = password('ben123');

# 不指定用户名时，默认更新当前用户口令
set password = password('ben123');
```
