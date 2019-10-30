---
title: SQL优化
date: 2019-10-23 23:12:48
tags:

---

#### 数据结构

1. 3层Btree可以存放上百万条数据
2. Btree一般指的是B+树，数据全部存放在叶子节点中。
3. B+树中查询任意的数据次数：n次（B+树的高度）

#### 分类：

1. 单值索引    单列的索引，一个表可以有多个单值索引
2. 唯一索引    不能重复    可以为null
3. 符合索引    多个列构成的索引
4. 主键索引    不能重复，不能为null

#### SQL性能问题

1. 分析sql的执行计划（explain）,可以模拟SQL优化器执行sql语句
2. Mysql查询优化会干扰我们的优化。

#### explain 参数解析：

1. id:编号  

   * id值相同，从上往下顺序执行;

   * id值越大越优先查询 (本质：在嵌套子查询时，先查内层 再查外层)

2. select_type 查询类型

   * PRIMARY:包含子查询SQL中的 主查询 （最外层）

   * SUBQUERY：包含子查询SQL中的 子查询 （非最外层）

   * simple:简单查询（不包含子查询、union）

   * derived:衍生查询(使用到了临时表)

     * 在from子查询中只有一张表

       > explain select  cr.cname 	from ( select * from course where tid in (1,2) ) cr ;

     * 在from子查询中， 如果有table1 union table2 ，则table1 就是derived,table2就是union

       > explain select  cr.cname 	from ( select * from course where tid = 1  union select * from course where tid = 2 ) cr ;

3. table

4. partitions  

5. type  类型（索引类型）

   * system>const>eq_ref>ref>range>index>all  其中：system,const只是理想情况；实际能达到 ref>range
   * system（忽略）: 只有一条数据的系统表 ；或 衍生表只有一条数据的主查询
   * const:仅仅能查到一条数据的SQL ,用于Primary key 或unique索引  （类型 与索引类型有关）
   * eq_ref:唯一性索引：对于每个索引键的查询，返回匹配唯一行数据（有且只有1个，不能多 、不能0）;常见于唯一索引 和主键索引
   * 非唯一性索引，对于每个索引键的查询，返回匹配的所有行（0，多）

6. possible_keys  预测用到的索引

7. key   实际使用的索引

8. key_len   实际使用的索引的长度

9. ref    表之间的引用

10. rows  通过索引查询到的数据量

11. filtered 

12. Extra  额外的信息

#### 索引分类

1. 聚集索引：页节点包含了完整的数据记录。（innoDB的主键索引）
2. 非聚集索引  （myISAM的主键索引）

#### Q&A

1. 为什么InnoDB表必须有主键，并且推荐使用整形的自增主键。

   InnoDB的索引和数据存在同一个`表名.ibd`文件中

   自增：可以减少B+树的分裂。