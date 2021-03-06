---
title: 《高性能MySQL》阅读笔记 第五章 创建高性能索引
date: 2019-04-30 23:50:17
updated: 2019-04-30 23:50:17
categories: tech
tags:
- DB
- MySQL
description: 这个笔记主要是写一些自己原来理解不足、可能在以后可能会遇到的一些点，做下记录，日后自己也能回顾重温。
toc: true
---

## 5.1 索引基础

索引类型：B-Tree，哈希索引，空间数据索引，全文索引....

1. 同样是使用B-Tree索引，MyISAM使用前缀压缩技术使得索引更小，但InnoDB按原数据进行存储。MyISAM索引通过数据的物理位置引用被索引的行，InnoDB根据主键引用被索引的行。

2. 对于多个索引，必须严格按从左到右顺序使用，缺一不可，并且， 如果中间某个列使用的是范围查询，其索引右边的列都无法使用到索引。

3. 哈希索引主要基于哈希表的实现，只有精确匹配索引所有列的查询才有效。

4. 哈希索引数据不是按照索引值顺序存储的，无法用于排序。

5. 哈希索引不支持部分索引列匹配查找，哈希索引始终是使用索引列的全部内容来计算哈希值，例如（A,B）上的索引，如果查询条件只有A则无法使用索引。

6. 哈希索引同样也只支持等值比较，范围查找无法使用哈希索引。

## 5.2 索引的优点

略

## 5.3 高性能索引策略（重点阅读）

### 5.3.1 独立的列：

查询中索引不能是表达式的一部分，不能是函数的参数，应该始终将索引单独放在比较符号一侧。

### 5.3.2 前缀索引：

索引的选择性 = 不重复的值（distinct后的值） / 数据表的总记录数，在选择前缀索引时这个比值越大说明性能越接近全字段索引，效果最优。

在处理BLOB、TEXT或者VARCHAR类型的列时，如果他们本身的长度很长，MySQL是不允许索引这些列的完整长度的，这里必须使用前缀索引，所以我们只选取这些值的前缀建立索引，具体选择多长的长度需要考察这个前缀的选择性，让其尽量接近完整列的选择性。

前缀索引可以使索引更小、更快，但是使用前缀索引后就不能对该列做ORDER BY和GROUP BY了。

不同的情况下也可以考虑后缀索引，依靠MySQL内的字符串转向实现。

总结来说使用前缀索引的情况有： ①字符串的列 ②字符串本身就比较长，并且字符串开始的几个字符差异比较大，能够进行很好的区分 ③只是前一半的字符的选择性就已经接近了全字段索引的选择性了。

### 5.3.3 多列索引

在多个列上分别建立索引，然后一起使用实际都可能会是全盘扫描。例如：SELECT xxx FROM xxx WHERE a = 1 OR b = 1; 但在新版本的MySQL中会自动处理好这两个单列索引，OR条件转化为union、AND查询转化为intersection的两表查询(这种合并的优化策略并不可取，只是说明索引建的很糟糕，MySQL会稍作修正)。

查询之前explain看看是否有索引合并，看看自己的索引是否建的足够好。

### 5.3.4 索引的组合顺序

对于一个单个查询，查询条件是A=1和B=2，创建索引应该是(A,B)还是(B,A)呢？单单对这条SQL，可以考察WHERE条件中列的选择性，选择性越高、数据基数越少越靠前。

这里定的顺序只是针对这单条SQL，并且是特定的查询值，这样对其他查询不公平，可能会使得某些查询不如预期，甚至影响服务器的整体性能。

具体在实际中如何选择顺序我们主要依靠经验，往往选择性更高的列放在前面，如果对于WHERE子句中的数据固定就考察数据基数，当然这些方法也不是一概而论，都需要具体分析。

### 5.3.5 聚簇索引

这里谈论InnoDB中的聚簇索引，简单理解就是数据行实际上存在索引的叶子页上，所以一个表只能有一个聚簇索引,也可以理解为聚簇索引就是“表”。InnoDB通过主键建立聚簇索引，没有定义主键则选择一个唯一的为空索引替代，如果没有唯一非空索引就会隐式定义一个主键。InnoDB的每一个叶子页占的空间是指定的。

聚簇索引优点：①数据查询时只需要一次的磁盘I/O。 ②索引和数据保存在一起，因此获取数据的速度会更快。 ③如果使用覆盖索引，就可以直接在取到的页中值，不需要再次回表查询。

缺点：①聚簇索引的分页设计，对于I/O密集的有性能提升，但是数据存在内存中就没有优势了。 ②数据插入如果不是按照主键顺序插入会慢。 ③更新聚簇索引的代价大。 ④插入一条数据到一个满页时，会导致页分裂，占用更多空间。 ⑤页分裂多了数据不连续带来查询慢的问题。 ⑥非聚簇索引访问需要两次索引查找，因为这类索引叶子中保存的不是指向物理地址的指针，而是行的主键值。

和MyISAM引擎的区别主要在于：MyISAM的索引和普通索引没有太大区别，叶子节点上就是索引值；InnoDB的叶子节点包含主键值、事务ID、事务和MVCC的回滚指针、以及剩余的列，相当于包含了表，还有一点不同上面也提到过，InnoDB中非聚簇索引叶子指向的是主键值，不是一般索引的“行指针”（设计目的是减少行移动、数据页分裂的维护工作）。

从下面这张图很容易可以看出区别：
<div align = "center"><img src = "https://i.loli.net/2019/04/29/5cc69b670e276.png"/></div>

最好避免随机的聚簇索引，特别是I/O密集的应用，最好是索引数据和应用无关，最简单的方式是使用AUTO_INCREMENT的列。实际中也推荐按照主键顺序插入数据，并且尽可能的使用单条增加的聚簇值来插入新行。

如下两个图：
<div align = "center"><img src = "https://i.loli.net/2019/04/29/5cc69f835877e.png"/></div>
<div align = "center"><img src = "https://i.loli.net/2019/04/29/5cc69f98422c3.png"/></div>

索引值是否是顺序的决定了聚簇索引插入的效率，顺序的话就直接插入到后面，达到页大小时就往后放；非顺序的情况的话就会出现新行随机插入已有的位置之间，这会导致频繁的页分裂，页也会变得稀疏不规则，如果插入过程中目标页已刷到磁盘不在缓存中的话就还需要从磁盘读取到内存中，会有大量的随机I/O。

### 5.3.6 覆盖索引

简单来说就是突破单纯考虑WHERE条件来创建索引，考虑到整个查询，**让MySQL可以使用索引就直接获取到数据**。如果一个索引包含（覆盖）所有需要查询的字段，就可以称为覆盖索引。查询只需要扫描索引不需要回表。

优势：①I/O数据量减少，数据更容易放入内存 ②聚簇索引是按值顺序存储的，范围查询会有更少的I/O ③如MyISAM引擎，内存只缓存索引，数据要依赖操作系统来缓存，一次数据访问就需要一次系统调用 ④对于聚簇索引如InnoDB中，通过非聚簇索引查询时得到主键，进一步要对主键索引进行查询，如果将二级主键做到覆盖查询，可以减少查询次数。

### 5.3.7 使用索引扫描来排序

EXPLAIN语句中type为index才使用索引，不要被Extra中的“Using index”搞混淆了。

MySQL有两种方式生成有序的结果，通过排序操作，或者按索引顺序扫描。只有当索引的顺序和ORDER BY子句的顺序完全一致时，MySQL才能够使用索引来对结果排序。

### 5.3.8 压缩（前缀压缩）索引

稍作了解，MyISAM压缩索引的机制是，先保存索引块的第一个值，然后将其他值和第一个值进行比较得到相同前缀的字节数和剩余的不同后缀部分。

压缩块使用更少的空间，但是每个索引都依赖于前面的值，所以MyISAM查找时只能从头开始扫描，这也是一个CPU内存资源和磁盘之间的一个资源权衡。

### 5.3.9 冗余和重复索引

MySQL允许创建冗余或者重复索引，但是这是很不推荐的，因为优化器都会单独进行处理，严重影响性能，因此在表创建时要避免这种情况。

例如已有索引(A,B)再创建(A)就是冗余索引，只要索引效果不同，或者是相同覆盖、不同类型也不算是冗余索引。

有种特殊情况也需要冗余索引，比如已有索引已经很大了，扩展会影响其他的使用该索引的查询性能。

### 5.3.10 未使用的索引

服务器有一些永远都用不到的索引，一些常见的服务器工具可以帮我们定位，例如Percona Server或者MariaDB中的userstates变量的开启，可以查询每个索引的使用频率，Percona Toolkit中的pt-index-usage了解每天查询的报告。

## 其他

1. 大多数情况还是使用B-Tree索引，其他索引都还是要依据特殊情况来考察。

2. 总体来看，查询时尽可能选择合适的索引避免单行查询，尽量使用数据的原始顺序避免排序，尽可能使用索引覆盖查询消除回表查询。

## Ref

BaronSchwartz, PeterZaitsev, VadimTkachenko, et al. 高性能MySQL[M]. 电子工业出版社, 2013.