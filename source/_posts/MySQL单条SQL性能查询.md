---
title: MySQL单SQL查询剖析
date: 2019-03-28 16:32:36
updated: 2019-03-28 16:32:60
categories: tech
tags: 
    - MySQL
    - Database
description: 在定位了具体需要优化的单条SQL之后，我们可以有针对的对这条查询详细探究，获悉这条SQL为什么慢，这里主要介绍MySQL自带的相关方法，帮助我们很方便的测量各个部分花费的时间，这里简要介绍几个获取方法SHOW STATUS、SHOW PROFILE、慢查询日志、Performance Schema，具体的如何优化优化方法将在其他文章中阐述。

---
# MySQL单SQL查询剖析

## SHOW PROFILE

SHOW PROFILE命令是在MySQL 5.1之后引入的，相对于通常来说我们使用的EXPLAIN、slow query log指令来说，SHOW PROFILE可以给出SQL语句执行中各个资源的消耗情况，比如CPU、IO等，如果打开会对服务器上执行的所有语句在他们执行的过程中记录服务器的具体工作时间。这个内部工具默认是关闭的，如下指令查询打开状态：

```sql
mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set, 1 warning (0.00 sec)
```

需要如下指令打开：

```sql
mysql> SET profiling = 1;
```

开启之后这个工具会将所有的运行信息记录到一张临时表，并且给接下里的查询赋予一个编号，记录的信息包括运行的具体语句，每个语句在各个部分消耗的时间信息。例如：

```sql
mysql> SHOW PROFILES;
+----------+------------+---------------------------------------------------+
| Query_ID | Duration   | Query                                             |
+----------+------------+---------------------------------------------------+
|        1 | 0.02267950 | show tables                                       |
|        2 | 0.00135350 | select * from student where studentId = 1 limit 1 |
|        3 | 0.00134375 | select * from student where studentId = 2 limit 1 |
|        4 | 0.00008225 | show profils                                      |
+----------+------------+---------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

可以看到，这个工具给每个运行过的查询语句都给了一个编号，并且可以看到精度很高的响应时间，可这样的时间对于分析一个查询的效率来说还是不够的，进一步的话我们有SHOW PROFILE指令，可以进一步确认更加细节的时间消耗：

```sql
mysql> show profile for query 3;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000092 |
| checking permissions | 0.000008 |
| Opening tables       | 0.000021 |
| init                 | 0.000039 |
| System lock          | 0.000010 |
| optimizing           | 0.000011 |
| statistics           | 0.114235 |
| preparing            | 0.000033 |
| executing            | 0.000004 |
| Sending data         | 0.019880 |
| end                  | 0.000014 |
| query end            | 0.000016 |
| closing tables       | 0.000015 |
| freeing items        | 0.000036 |
| cleaning up          | 0.000031 |
+----------------------+----------+
15 rows in set, 1 warning (0.00 sec)
```

这里仅仅只是运行时间的展示，要获取更加细节的信息展示可以在SHOW PROFILE指令中加入更多的细节部分：

```sql
mysql> show profile cpu,block io,memory,swaps for query 3;
+----------------------+----------+----------+------------+--------------+---------------+-------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out | Swaps |
+----------------------+----------+----------+------------+--------------+---------------+-------+
| starting             | 0.000092 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| checking permissions | 0.000008 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| Opening tables       | 0.000021 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| init                 | 0.000039 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| System lock          | 0.000010 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| optimizing           | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| statistics           | 0.114235 | 0.000000 |   0.000000 |          856 |             0 |     0 |
| preparing            | 0.000033 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| executing            | 0.000004 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| Sending data         | 0.019880 | 0.004000 |   0.000000 |         1248 |             0 |     0 |
| end                  | 0.000014 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| query end            | 0.000016 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| closing tables       | 0.000015 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| freeing items        | 0.000036 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| cleaning up          | 0.000031 | 0.000000 |   0.000000 |            0 |             0 |     0 |
+----------------------+----------+----------+------------+--------------+---------------+-------+
15 rows in set, 1 warning (0.00 sec)
```

这里仅仅只是我的一个示例，详细展示了这条SQL在哪部分花费时间比较多，让我们可以有针对的去优化我们的数据库或者SQL。

但SHOW PROFILE只告诉了我们哪些活动花费了最多的时间，但并没有告诉我们为什么导致这样，至于怎么优化就要继续进一步研究这个子任务。

## SHOW STATUS

SHOW STATUS命令的作用是计数器，可以从服务器级别看到MySQL从启动开始计算的各类操作的运行次数。

```sql
mysql> SHOW STATUS;
```

这么听起来SHOW STATUS并不是一个SQL运行的剖析工具，他无法给出一个操作的具体消耗时间，他的作用在于运行完某条SQL之后，可以观察各个计数器的数量变化，也就是说在运行完一条指令后，可以知道它各类活动的频繁程度，例如读索引、创建临时表等，缺点也就在于无法给出每个的时间消耗。

重置大多数状态变量到0：

```sql
mysql> FLUSH STATUS;
```

EXPLAIN指令也可以获取很多类似的信息，但是EXPLAIN只是一个估计的结果，SHOW STATUS是实际结果的统计。

进一步的用法中，SHOW STATUS还可以对结果进行模糊搜索，例如：

```sql
mysql> SHOW STATUS LIKE 'qcache%';
```

## 慢查询日志

慢查询日志是比较常见的剖析单条SQL效率的方法，开启之后在数据库运行过程中，会根据相关的慢查询设置将查询较慢的SQL语句进行记录。

可以通过查看MySQL变量的方法，查看慢查询的打开关闭情况：

```sql
mysql> SHOW VARIABLE LIKE '%slow_query_log&';
```

其中slow_query_log表示的慢查询的开关状态，slow_query_log_file表示慢查询日志的记录位置。

开启慢查询：

```sql
set global slow_query_log=1;
```

日志中记录的慢查询是时间低于某个配置的时间来的，配置的时间也是个MySQL的变量值：long_query_time，查看和设置这个配置值。

```sql
mysql> show variables like '%long_query_time%'

mysql> set global long_query_time = 4;
```

## Performance Schema

Performance Schema是一个功能强大、较为底层的性能监控工具，可以自定义信息收集粒度，对于查找MySQL的性能瓶颈，查找long-SQL原因都有相关帮助。

这个功能是MySQL自带的，5.6.6版本中默认打开，可以使用如下指令查看打开情况。

```sql
mysql> show variables like 'performance_schema'
```

关于精细化控制主要通过performance_schema库下的各个setup表来进行配置，具体不在此描述。

日常可能使用的比较多的功能可能有：

* 获取执行最多的SQL语句
* 单条执行时间最常的SQL语句
* 操作最频繁的表
* 从未被使用过的索引
* 文件IO消耗

performance schema的统计数据非常强大，具体在此不展开描述，如有想法进一步了解可以Google相关的文章进一步了解。

ref:
1. http://keithlan.github.io/2015/07/17/22_performance_schema/
2. https://cloud.tencent.com/developer/article/1072598
3. https://allenhu0320.iteye.com/blog/2186156