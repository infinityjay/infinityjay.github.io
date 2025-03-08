---
title:  MySQL - learning notes
date:   2020-08-01
categories:
  - Database
tags:
  - MySQL
  - Learning notes
---
Content

{% include toc %}

## Learn SQL

> Related terms

* Database: a container for storing organized data
* Table: a structured list of a specific type of data
* Schema: information about the layout and characteristics of databases and tables
* Column: a field in a table. All tables are composed of one or more columns.
* Data type: the type of data allowed. Each table column has a corresponding data type, which limits (or allows) the data stored in the column.
* Row: a record in a table.
* Primary key: a column (or a group of columns) whose value can uniquely distinguish each row in the table.

## MySQL query statement

### Retrieve columns

```mysql
# Use database
use [database name]

# Find database
show databases;

# Get tables in database
show tables;

# Show table columns
show columns from customers;

# Retrieve columns
select prod_name, prod_price from products;

# Retrieve all values ​​with different values ​​in the column
select distinct id from products;

# Specify starting row (5) and number of rows (10)
select prod_name from products limit 5,10;

# Use limited table name or database name
select products.prod_name from crashcourse.products;
```

### Sorting retrieval

```mysql
# Sort by order by (default ascending order)
select prod_name from products order by prod_name;

# Sort by multiple keywords
select prod_name from products order by prod_price, prod_name;

# Specify sort direction, DESC - descending order
select prod_id, prod_name from products order by prod_price DESC, prod_name; # DESC is only valid for price

# Return the maximum or minimum value in a column, here return the most expensive item
select prod_price from products order by prod_price DESC limit 1;
```

### Filter data

```mysql
# Use where clause
select prod_name, prod_price from products where prod_price = 2.5;
select prod_name, prod_price from products where prod_price < 10;
select prod_name, prod_price from products where prod_price <= 10;

# Mismatch check, find out products not produced in 1003
select prod_name, vend_id from products where vend_id <> 1003;
select prod_name, vend_id from products where vend_id != 1003;

# Use where . . between . . and . . operators for a closed interval
select prod_name, prod_price from products where prod_price between 5 and 10;

# Use IS NULL for null value check
select prod_name, prod_price from products where prod_price IS NULL;

# Use where . . AND combination query (you can use multiple and )
select prod_id, prod_name, prod_price from products where vend_id = 1003 and prod_price <= 10;

# Use where . . or . . Combined query
select prod_id, prod_name, prod_price from products where vend_id = 1003 or vend_id = 1002;

# Use brackets to specify priority
select prod_name, prod_price from products where (vend_id = 1003 or vend_id = 1002) and prod_price >= 10;

# Use where. . in. . to specify the filter list, which is equivalent to or and is more suitable for more options
select prod_name, prod_price from products where vend_id in (1002,1003);

# Use where. . not in. . to exclude options
select prod_name, prod_price from products where vend_id not in (1002,1003);

# Use LIKE + % wildcards to filter, % means any number of occurrences of any character (including 0 characters)
select prod_name, prod_price from products where prod_name like 'jet%';

# Use LIKE + _ wildcards to filter, _ means matching a single character
select prod_name, prod_price from products where prod_name like '_ ton';

# Use regular expressions, operator REGEXP
# . in regular expressions means matching any character
select prod_name, prod_price from products where prod_name regexp '.000';
# ｜ means or operator in regular expressions
select prod_name, prod_price from products where prod_name regexp '1000｜2000';
# [123] in regular expression means to match one of a set of characters, which is equivalent to 1｜2｜3
select prod_name, prod_price from products where prod_name regexp '[123] Ton';
# [1-5] matches any character from 1 to 5, .5 Ton will also match and return
select prod_name, prod_price from products where prod_name regexp '[1-5] Ton';
# Match special characters. [] | _ needs to use the escape character \\, and to match \ itself, you need to use \\
select prod_name, prod_price from products where prod_name regexp '\\.';
```

### Regular expression

> Match a single character in regular expression

| Predefined character class | Description |
| :----------: | :---------------------------------------------- |
| [:alnum:] | Any letter and digit (same as [a-zA-Z0-9]) |
| [:alpha:] | Any character (same as [a-zA-Z]) |
| [:blank:] | Space and tab (same as [\\t]) |
| [:cntrl:] | ASCII control characters (ASCII 0 to 31 and 127) |
| [:digit:] | Any digit (same as [0-9]) |
| [:graph:] | Same as [:print:], but without spaces |
| [:lower:] | Any lowercase letter (same as [a-z]) |
| [:print:] | Any printable character |
| [:punct:] | Any character not in either [:alnum:] or [:cntrl:] |
| [:space:] | Any whitespace character including spaces (same as [\\f\\n\\r\\t\\v]) |
| [:upper:] | Any uppercase letter (same as [A-Z]) |
| [:xdigit:] | Any hexadecimal digit (same as [a-fA-F0-9]) |

> Matching multiple characters in regular expressions

| Metacharacters | Description |
| :----: | :------------------------- |
| * | 0 or more matches |
| + | 1 or more matches (equal to {1,}) |
| ? | 0 or 1 matches (equal to {0,1}) |
| {n} | Specified number of matches |
| {n,} | No less than the specified number of matches |
| {n,m} | Range of matches (m does not exceed 255) |

```mysql
# The regular expression \\([0-9] sticks?\\) needs some explanation. \\(matches), [0-9] matches any digit (1 and 5 in this case), and sticks? matches sticks and sticks (the ? after s makes s optional, since ? matches 0 or 1 occurrence of any character preceding it), and \\) matches). Without ?, matching sticks and sticks would be very difficult.
select prod_name from products where prod_name REGEXP '\\([0-9] sticks?\\)';
# Output
TNT (1 stick)
TNT (5 sticks)

# As mentioned before, [:digit:] matches any digit, so it is a set of digits. {4} requires exactly 4 occurrences of the preceding character (any digit), so [[:digit:]]{4} matches any 4 digits in a row
select prod_name from products where prod_name REGEXP '[[:digit:]]{4}';
# Output
JetPack 1000
JetPack 2000
```

> Locator in regular expression

| Locator metacharacter | Description |
| :--------: | :--------- |
| ^ | Beginning of text |
| $ | End of text |
| [[:<:]] | Beginning of word |
| [[:>:]] | End of word |

```mysql
# ^ matches the beginning of a string. Therefore, ^[0-9\\.] matches . or any digit only when it is the first character in the string. Without ^, 4 more rows (those with numbers in the middle) would be retrieved.
select prod_name from products where prod_name REGEXP '^[0-9\\.]';
# Output
.5 ton anvil
1 ton anvil
2 ton anvil
```

### Common functions

```mysql
# In selectect
select prod_id, quantity, item_price, quantity*item_price AS expanded_price from orderitems where order_num = 2005;

# Use Concat() to concatenate strings, that is, connect multiple strings to form a longer string, and separate each string with a comma.
select Concat(vend_name, '(', vend_contry, ')') from vendors order by vend_name;
# Use AS to set an alias for easy reference
select Concat(vend_name, '(', vend_contry, ')') AS vend_title from vendors order by vend_name;

# Upper(), convert text to uppercase
select vend_name, Upper(vend_name) AS vend_name_upper from vendors;

# SOUNDEX is an algorithm that converts any text string into an alphanumeric pattern that describes its phonetic representation. SOUNDEX takes into account similarly pronounced characters and syllables, allowing strings to be compared phonetically rather than alphabetically.
select cust_name, cust_contact from customers where Soundex(cust_contact) = Soundex('Y. Lie');
# Result
Coyote Inc. Y Lee
```

> Date and time processing functions

| Function | Description |
| :-----------: | :----------------------------- |
| AddDate() | Add a date (day, week, etc.) |
| AddTime() | Add a time (hour, minute, etc.) |
| CurDate() | Returns the current date |
| CurTime() | Returns the current time |
| Date() | Returns the date part of a date and time |
| DateDiff() | Calculates the difference between two dates |
| Date_Add() | Highly flexible date operation functions |
| Date_Format() | Returns a formatted date or time string |
| Day() | Returns the day part of a date |
| DayOfWeek() | For a date, returns the corresponding day of the week |
| Hour() | Returns the hour portion of a time |
| Minute() | Returns the minute portion of a time |
| Month() | Returns the month portion of a date |
| Now() | Returns the current date and time |
| Second() | Returns the second portion of a time |
| Time() | Returns the time portion of a date and time |
| Year() | Returns the year portion of a date |

```mysql
# Example
select cust_id, order_num from orders where Data(order_date) = '2005-09-01';
# Find all orders in September
select cust_id, order_num from orders where Data(order_date) between '2005-09-01' and '2005-09-30';
select cust_id, order_num from orders where year(order_date)=2005 and Month(order_date)=9;
```

> Numerical processing functions

| Function | Description |
| :----: | :----------------- |
| Abs() | Returns the absolute value of a number |
| Cos() | Returns the cosine of an angle |
| Exp() | Returns the exponential value of a number |
| Mod() | Returns the remainder of a division operation |
| Pi() | Returns the ratio of pi |
| Rand() | Returns a random number |
| Sin() | Returns the sine of an angle |
| Sqrt() | Returns the square root of a number |
| Tan() | Returns the tangent of an angle |

> Aggregate Functions

| Function | Description |
| :-----: | :--------------- |
| AVG() | Returns the average value of a column |
| COUNT() | Returns the number of rows in a column |
| MAX() | Returns the maximum value of a column |
| MIN() | Returns the minimum value of a column |
| SUM() | Returns the sum of the values ​​in a column |

```mysql
# Example
select AVG(prod_price) AS avg_price from products;

select SUM(item_price*quantity) AS total_price from orderitems where order_num = 20005;
```

### Grouping data

```mysql
# group by Grouping
select vend_id, COUNT(*) AS num_prods from products group by vend_id;
```

The above SELECT statement specifies two columns, vend_id contains the ID of the product vendor, and num_prods is a calculated field (created using the COUNT(*) function). The GROUP BY clause instructs MySQL to sort and group the data by vend_id. This results in num_prods being calculated once for each vend_id rather than for the entire table. As you can see from the output, vendor 1001 has 3 products, vendor 1002 has 2 products, vendor 1003 has 7 products, and vendor 1005 has 2 products.

Because you use GROUP BY, you do not have to specify each group to be calculated and evaluated. This is done automatically. The GROUP BY clause instructs MySQL to group the data and then aggregate on each group rather than on the entire result set.

* The GROUP BY clause can contain any number of columns. This allows nested groupings, providing more fine-grained control over data grouping.

* If you nest groupings in the GROUP BY clause, the data is aggregated on the last specified group. In other words, all the specified columns are calculated together when creating the grouping (so you cannot retrieve data from individual columns).

* Each column listed in the GROUP BY clause must be a search column or a valid expression (but not an aggregate function). If you use an expression in the SELECT, you must specify the same expression in the GROUP BY clause. You cannot use aliases.

* Except for aggregate calculation statements, every column in the SELECT statement must be given in the GROUP BY clause.

* If there is a NULL value in the grouping column, NULL is returned as a group. If there are multiple rows with NULL values ​​in the column, they will be grouped together.

* <font color = red>The GROUP BY clause must appear after the WHERE clause and before the ORDER BY clause. </font>

```mysql
# Use having to filter groups. All types of where clauses can be replaced by having
select cust_id, COUNT(*) as orders from orders group by cust_id having count(*) >= 2;

select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having count(*) >= 2;

select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having count(*) >= 2 order by vend_id;
```

<font color = red>It can be understood that where filters before data grouping, so group by needs to be after where, and having filters the groups. </font>

| Clause | Description | Must be used |
| :------: | ------------------ | ---------------------- |
| SELECT | Column or expression to return | Yes |
| FROM | Table to retrieve data from | Only used when selecting data from the table |
| WHERE | Row-level filtering | No |
| GROUP BY | Grouping description | Only used when calculating aggregations by group |
| HAVING | Group-level filtering | No |
| ORDER BY | Output sort order | No |
| LIMIT | Number of rows to retrieve | No |

### Subquery

```mysql
select cust_id from orders where order_num in (select order_num from orderitems where prod_id = 'TNT2');

# Two-table query requires fully qualified column name
select cust_name,
cust_state,
(select count(*)
from orders
where orders.cust_id = customers.cust_id)
as orders
from customers
order by cust_name;
```

### Joining tables

> Self-join

```mysql
select p1.prod_id, p1.prod_name
from products as p1, products as p2
where p1.vend_id = p2.vend_id
and p2.prod_id = 'DTNTR';
```

> Natural join

```mysql
# In this example, the wildcard is only used for the first table. All other columns are explicitly listed, so no duplicate columns are retrieved.
select c.*, o.order_num, o.order_date, oi.prod_id, oi_quantity, OI.item_price
from customers as c, orders as o, orderitems as oi
where c.cust_id = o.cust_id
and oi.order_num = o.order_num
and prod_id = 'FB';
```

> Outer join

```mysql
# Left join, primary table is the table on the left side of outer join, the result includes all rows in primary table customers
select customers.cust_id, orders.order_num
from customers left outer join orders
on customers.cust_id = order.cust_id;

# Right join, the main table is the table on the right side of outer join, and the result includes all rows in the main table orders
select customers.cust_id, orders.order_num
from customers right outer join orders
on customers.cust_id = order.cust_id;
```

## MySQL insert statement

```mysql
# insert insert statement (depends on the insertion order)
insert into customers
values(NULL, 'Pep E. LaPew', '100 Main Street');

# Explicitly give the column name for easy correspondence
insert into customers(cust_contact, cust_name, cust_address)
values(NULL, 'Pep E. LaPew', '100 Main Street');

# To insert multiple rows of data, you can write multiple commands or as follows
insert into customers(cust_contact, cust_name, cust_address)
values(NULL, 'Pep E. LaPew', '100 Main Street'),
(NULL, 'M. Martin', '42 Galaxy Way');

# Insert the retrieved data
insert into customers(cust_contact, cust_name, cust_address)
select cust_contact, cust_name, cust_address
from custnew;
```

## MySQL update and delete statements

```mysql
# update statement updates data
update customers
set cust_email = 'elmer@fudd.com'
where cust_id = 10005;
```

<font color = red>Note: Make sure the conditions after where, if you do not specify where, the entire table will be updated</font>

```mysql
# delete statement deletes data
delete from customers
where cust_id = 10006;
```

## Creating and manipulating tables

```mysql
# Creating tables create
create table customers
(
cust_id int NOT NULL Auto_INCREMENT,
cust_name char(50) NOT NULL ,
cust_email char(255) NULL ,
PRIMARY KEY (cust_id)
)ENGINE = InnoDB;
```

* `Auto_INCREMENT`

AUTO_INCREMENT tells MySQL that this column should be incremented automatically whenever a row is added. Each time an INSERT is performed, MySQL automatically increments the column (hence the keyword AUTO_INCREMENT), assigning the next available value to the column. This gives each row a unique cust_id, which can be used as a primary key value. Only one AUTO_INCREMENT column is allowed per table, and it must be indexed (e.g., by making it the primary key).

* Engine Type

* InnoDB is a reliable transaction processing engine (see Chapter 26), which does not support full-text search;

* MEMORY is equivalent to MyISAM in function, but because the data is stored in memory (not disk), it is very fast (especially suitable for temporary tables);

* MyISAM is an extremely high-performance engine that supports full-text search (see Chapter 18), but does not support transaction processing.

```mysql
# Update table alter
# Add a column
alter table vendors
add vend_phone CHAR(20);

# Delete a column
alter table vendors
DROP column vend_phone;
```

```mysql
# Delete table
drop table customers2;

# Rename table
rename table customers2 to customers;
```

## Stored procedures

Stored procedures are simply a collection of one or more MySQL statements saved for later use. They can be thought of as batch files, although their use is not limited to batch processing.

There are three main benefits to using stored procedures, namely simplicity, security, and high performance.

```mysql
# Example Create a stored procedure
create procedure productpricing()
BEGIN
select avg(prod_price) as priceaverage
from products;
end;
# Call a stored procedure
call productpricing();
# Output
priceaverage
16.133517

# Delete a stored procedure
drop procedure productpricing;
```

> Using parameter storage

```mysql
# Example
# This stored procedure accepts 3 parameters: pl stores the lowest price of the product, ph stores the highest price of the product, and pa stores the average price of the product. Each parameter must have a specified type, and decimal values ​​are used here. The keyword OUT indicates that the corresponding parameter is used to pass a value out of the stored procedure (returned to the caller). MySQL supports parameters of type IN (passed to a stored procedure), OUT (passed out of a stored procedure, as used here), and INOUT (passed in and out of a stored procedure). The code of the stored procedure is located within the BEGIN and END statements. As you can see, they are a series of SELECT statements that retrieve values ​​and then store them in the corresponding variables (by specifying the INTO keyword).
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

# Variable passing parameters
call productpricing(
@pricelow,
@pricehigh,
@priceaverage);

# Calling a stored procedure
select @pricehigh, @pricelow, @priceaverage;

# Example 2
# onumber is defined as IN because the order number is passed to the stored procedure. ototal is defined as OUT because we want to return the total from the stored procedure. The SELECT statement uses these two parameters, the WHERE clause uses onumber to select the correct row, and INTO uses ototal to store the calculated total.
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
# Two parameters must be passed to ordertotal; the first is the order number, and the second is the name of the variable that contains the calculated total.
call ordertotal(20005, @total);
# Calling a stored procedure
select @total;

```

## Managing Transactions

Not all engines support explicit transaction management. MyISAM and InnoDB are the two most commonly used engines. The former does not support explicit transaction management, while the latter does. This is why the sample tables used in this book are created to use InnoDB rather than the more commonly used MyISAM. If you need transaction processing in your application, be sure to use the correct engine type.

Transaction processing can be used to maintain the integrity of the database by ensuring that batches of MySQL operations are either executed completely or not executed at all.

* A transaction is a group of SQL statements;
* A rollback is the process of undoing a specified SQL statement;
* A commit is the writing of unstored SQL statement results to a database table;
* A savepoint is a temporary placeholder set in a transaction that you can issue a rollback to (as opposed to rolling back the entire transaction)

```mysql
# Controlling transactions
# The key to managing transactions is to break groups of SQL statements into logical chunks and to specify when data should and should not be rolled back.
# Start a transaction
START TRANSACTION
# Using ROLLBACK
# This example starts by displaying the contents of the ordertotals table (which was populated in Chapter 24). First, a SELECT is executed to show that the table is not empty. Then a transaction is started, with a DELETE statement to delete all rows in ordertotals. Another SELECT statement verifies that ordertotals is indeed empty. A ROLLBACK statement is then used to roll back all statements after START TRANSACTION, and the last SELECT statement shows that the table is not empty. Obviously, ROLLBACK can only be used within a transaction (after a START TRANSACTION command)
select * from ordertotals;
START TRANSACTION;
DELETE from ordertotals;
select * from ordertotals;
rollback;
select * from ordertotals;
# commit
# Within a transaction block, a commit is not implicitly performed. To perform an explicit commit, use the COMMIT statement
# In this example, order 20010 is completely deleted from the system. Because it involves updating two database tables, orders and orderItems, a transaction block is used to ensure that orders are not partially deleted. The final COMMIT statement writes out the changes only if there are no errors. If the first DELETE works but the second fails, the DELETE is not committed (in fact, it is automatically undone).
start transaction;
delete from orderitems where order_num = 20010;
delete from orders where order_num = 20010;
commit;

# Using savepoints
# Each savepoint takes a unique name that identifies it so that MySQL knows where to roll back to when rolling back. Savepoints are automatically released after the transaction is completed (a ROLLBACK or COMMIT is executed). Since MySQL 5, savepoints can also be released explicitly with RELEASE SAVEPOINT.
savepoint delete1;
rollback to delete1;
```

## Security management

> User management

```mysql
# Get a list of all user accounts
use mysql;
select user from user;

# Create an account
create user ben IDENTIFIED BY '123456';

# Delete a user account
drop user ben;
```

> Access permissions

```mysql
# Show user permissions
show GRANTS for ben;

# Set access permissions
# This GRANT allows the user to use SELECT on crashcourse.* (all tables in the crashcourse database). By granting only SELECT access, user ben has read-only access to all data in the crashcourse database.
GRANTS select on crashcourse.* to ben;

# Revoke permissions
REVOKE select on on crashcourse.* from ben;
```

> Change password

```mysql
# set password
set password for ben = password('ben123');

# When the user name is not specified, the current user password is updated by default
set password = password('ben123');
```