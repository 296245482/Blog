---
title: SQL语句写法优化
date: 2018-07-04 17:26:36
updated: 2018-07-04 17:30:00
categories: tech
tags: DataBase
description: 本文从SQL语句的层面上介绍了SQL语句的优化写法，当数据量达到一定程度时将会提高查询效率。

---

## EXPLAIN用法

在不清楚索引等情况下，对于自己效率不太确定的SELECT语句，在SQL语句前加入 `EXPLAIN` 关键字，该语句会返回你这条SQL语句的执行计划，比如有没有用索引，有没有全盘扫描，可供效率参考，`EXPLAIN`运行返回的结果如下(例如：`EXPLAIN SELECT id,name,score FROM student_score`)：

`id`: SELECT编号，出现顺序排序
`select_type`: 查询的类型，简单或复杂
`table`: 表名，或者其他查询语句ID
`partition`: 该表的分区信息
`type`: 访问类型
`possible_keys`: 查询可能会用到的索引
`key`: 实际使用的索引
`key_len`: 使用的索引长度
`ref`: 哪些列或常量被用于查找索引列上的值
`rows`: 估计读取的行数
`filtered`: 返回结果的行数占需要度的航的百分比
`Extra`: MySQL解决查询的额外信息

其中，访问类型：ALL, index,  range, ref, eq_ref, const, system, NULL，从左到右性能从差到好，当自己的语句出现靠左侧的type时要考虑优化自己的索引或者查询语句

注：MYSQL 5.6.3以前只能`EXPLAIN SELECT`； MYSQL5.6.3以后就可以`EXPLAIN SELECT、UPDATE、DELETE`

**有了`EXPLAIN`语句，通过SQL语句的执行计划，我们可以很快定位到查询语句过慢的问题。**

## NULL与空值

MySQL中的“空值”和“NULL”是有不同的，并且在对NULL进行 **'<'、'>'、'='** 操作时肯定很多人都遇到过坑，在对一些NULL值的判断若理解不到位可能会导致得到的查询结果和你所设想的SQL结果不一致的情况。

'' - 空值在MySQL中不占空间，而NULL是占用空间的，好比一个容器，空值代表容器内是真空的，NULL则代表容器内装满了空气。B树索引时不会存储NULL值，如果索引的字段可以为NULL，索引的效率会下降很多。

`IS NOT NULL`和`!=NULL`是不一样的操作，对于 **"=、<、>..."** 等这些判断来说，`NULL`表示什么都不是，任何运算符的操作结果都是false，对于`NULL`的计算只能使用`IS NULL`来判断，一般情况下推荐使用`IS NOT NULL`，对于`!= NULL`来说返回的结果永远都是0行，并且不会有任何语法错误。

但是如果一定要使用`!= NULL`的话，可以通过set ANSI_NULLS off让`IS NOT NULL`和`!= NULL`等价。

## 避免查询的全盘扫描，保证索引正确使用

1. 在所有的SQL中尽量避免type为ALL的全盘扫描，在`WHERE`和`ORDER BY`涉及的字段上建立索引，否则数据量过大时经常会导致超时。


2. `WHERE`子句中尽量减少`!=`和`<>`操作符，这种情况下索引可能不会被使用进而进行全盘扫描。


3. 尽量避免将会在`WHERE`子句中涉及的字段设置可以为null，索引中有NULL值时索引效率会下降很多，例如还有如下情况：

    ```sql
    SELECT id,name,score FROM STUDENT_SCORE WHERE score is null;
    ```

    `is null`语句的使用会放弃使用索引，进而进行全盘扫描，在设计中可以将null值替换为默认值"0"，上述语句`WHERE`后的语句可以改为`score = 0`。


4. 将`WHERE`子句中的`OR`条件改写，拆成两条语句的`UNION`操作，例如下列语句将会进行全表扫描：

    ```sql
    SELECT id FROM student_score WHERE score = 100 OR socre = 0
    ```

    可以改成这样的查询提升查询效率：
    ```sql
    SELECT id FROM student_score WHERE score = 100
    UNION ALL
    SELECT id FROM student_score WHERE score = 20
    ```


5. 若查询中经常涉及`LIKE`与通配符`%`的查询可以考虑在原表中加入全文索引，如下：
    ```sql
    SELECT id FROM student_score WHERE name LIKE '%long%'
    ```
    这句SQL语句若要考虑优化可以将`name`字段设置成全文索引，提高效率。全文索引的使用方法参考[MySQL使用全文索引](https://blog.csdn.net/u011734144/article/details/52817766/)


6. `IN`和`NOT IN`的使用大多也会放弃使用索引，进行全表扫描，对于连续值的情况，使用`BETWEEN`代替`IN`,例如：
    ```sql
    SELECT id FROM student_score WHERE score IN (98,99,100)
    ```
    改写为：
    ```sql
    SELECT id FROM student_score WHERE score BETWEEN 98 and 100
    ```


7. 如果在`WHERE`子句中有参数，也会导致全表扫描，例如：下述SQL语句：
    ```sql
    SELECT id FROM student_score WHERE name=@name
    ```
    因为SQL只有在运行时才会解析局部变量，SQL执行计划在编译时生成，可这个时候变量的值还是未知的，因而无法作为索引的输入项，实际执行时只能进行全表扫描。可以改为强制查询使用索引：
    ```sql
    SELECT id FROM student_score WITH(INDEX("索引名")) WHERE name=@name
    ```


8. 避免在`WHERE`子句中进行表达式操作，例如：
    ```sql
    SELECT id FROM student_score WHERE score/2=50
    ```
    为了避免放弃索引而全表扫描，应改为
    ```sql
    SELECT id FROM student_score WHERE score=2*50
    ```


9. 避免在`WHERE`子句中的函数操作，如下：
    ```sql
    SELECT id FROM student_score WHERE substring(name,1,3)='cheng' --name以cheng开头
    ```
    为了避免放弃索引，而全表扫描改写为name字段添加全文索引的通配符匹配，或者使用`WHERE name LIKE 'cheng%'`。


10. 除了上述的8和9外，我们尽量避免在`WHERE`子句中的`=`左边进行函数、算数运算或其他表达式的运算，保证索引的正确使用。

## 索引的使用

1. `WHERE`子句中的条件是`OR`关系的话，加索引将不会有任何作用。考虑改写为两条SQL语句的`UNION ALL`操作。


2. 数据重复并且分布均匀的表字段建了索引一般不会对查询效率的提升有很大影响。


3. 符合索引从左到右使用索引中的字段，可以只是用一部分，但必须是最左侧的部分。所以复合索引把最常用的字段放在最左边，重要程度一次递减。


4. 复合索引中任何字段含有NULL值，那么该字段对复合索引是无效的。


5. `LIKE '%abc%'`不会使用索引而`LIKE 'abc%'`会使用索引。


6. 使用`EXISTS`和`NOT EXISTS`代替`IN`和`NOT IN`，后者不会使用索引而进行全表扫描。

## 其他Tips

1. 能设计成数字型的字段就不设计成字符型。因为在处理查询等操作时字符型会挨个比较没一个字符，而数字型比较一次就够了。


2. 使用`varchar/nvarchar`代替`char/nchar`，前者是边长字段，可以节省空间，并且字段越小搜索效率越高。


### REF
1. https://blog.csdn.net/wendy432/article/details/52319908
2. https://blog.csdn.net/u011734144/article/details/52817766/
3. https://www.cnblogs.com/softidea/p/5977860.html