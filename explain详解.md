### explain执行计划包含的信息
[![Vz3bYn.md.png](https://s2.ax1x.com/2019/06/21/Vz3bYn.md.png)](https://imgchr.com/i/Vz3bYn)

**其中最重要的字段为：id、type、key、rows、Extra**

**各字段详解**

#### id
##### select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序 
1. id相同：执行顺序由上至下 
[![Vz851x.md.png](https://s2.ax1x.com/2019/06/21/Vz851x.md.png)](https://imgchr.com/i/Vz851x)
2. id不同：如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行 
[![Vz8ojK.md.png](https://s2.ax1x.com/2019/06/21/Vz8ojK.md.png)](https://imgchr.com/i/Vz8ojK)
3. id相同又不同（两种情况同时存在）：id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行 

[![Vz8ONd.md.png](https://s2.ax1x.com/2019/06/21/Vz8ONd.md.png)](https://imgchr.com/i/Vz8ONd)

#### select_type

##### 查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询

1. SIMPLE：简单的select查询，查询中不包含子查询或者union 
2. PRIMARY：查询中包含任何复杂的子部分，最外层查询则被标记为primary 
3. SUBQUERY：在select 或 where列表中包含了子查询 
4. DERIVED：在from列表中包含的子查询被标记为derived（衍生），mysql或递归执行这些子查询，把结果放在零时表里 
5. UNION：若第二个select出现在union之后，则被标记为union；若union包含在from子句的子查询中，外层select将被标记为derived 
6. UNION RESULT：从union表获取结果的select 

[![Vz8X4A.md.png](https://s2.ax1x.com/2019/06/21/Vz8X4A.md.png)](https://imgchr.com/i/Vz8X4A)

#### type

##### 访问类型，sql查询优化中一个很重要的指标，结果值从好到坏依次是：

> system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
1. system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略不计
2. const：表示通过索引一次就找到了，const用于比较primary key 或者 unique索引。因为只需匹配一行数据，所有很快。如果将主键置于where列表中，mysql就能将该查询转换为一个const 
[![Vz8v9I.md.png](https://s2.ax1x.com/2019/06/21/Vz8v9I.md.png)](https://imgchr.com/i/Vz8v9I)
3. eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键 或 唯一索引扫描。
[![Vz8zgP.md.png](https://s2.ax1x.com/2019/06/21/Vz8zgP.md.png)](https://imgchr.com/i/Vz8zgP)
注意：ALL全表扫描的表记录最少的表如t1表
4. ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质是也是一种索引访问，它返回所有匹配某个单独值的行，然而他可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体 
[![VzGSjf.md.png](https://s2.ax1x.com/2019/06/21/VzGSjf.md.png)](https://imgchr.com/i/VzGSjf)
5. range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了那个索引。一般就是在where语句中出现了bettween、<、>、in等的查询。这种索引列上的范围扫描比全索引扫描要好。只需要开始于某个点，结束于另一个点，不用扫描全部索引 
[![VzG9u8.md.png](https://s2.ax1x.com/2019/06/21/VzG9u8.md.png)](https://imgchr.com/i/VzG9u8)
6. index：Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常为ALL块，应为索引文件通常比数据文件小。（Index与ALL虽然都是读全表，但index是从索引中读取，而ALL是从硬盘读取）
[![VzGA4s.md.png](https://s2.ax1x.com/2019/06/21/VzGA4s.md.png)](https://imgchr.com/i/VzGA4s)
7. ALL：Full Table Scan，遍历全表以找到匹配的行 
[![VzGlE4.md.png](https://s2.ax1x.com/2019/06/21/VzGlE4.md.png)](https://imgchr.com/i/VzGlE4)

#### possible_keys

* 查询涉及到的字段上存在索引，则该索引将被列出，但不一定被查询实际使用

#### key

* 实际使用的索引，如果为NULL，则没有使用索引。

#### key_len

* 表示索引中使用的字节数，查询中使用的索引的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。key_len是根据表定义计算而得的，不是通过表内检索出的

#### ref

* 显示索引的那一列被使用了，如果可能，是一个常量const。

#### rows

* 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

#### Extra
1. Using filesort 
> mysql对数据使用一个外部的索引排序，而不是按照表内的索引进行排序读取。也就是说mysql无法利用索引完成的排序操作成为“文件排序” 

[![VzG0Ve.md.png](https://s2.ax1x.com/2019/06/21/VzG0Ve.md.png)](https://imgchr.com/i/VzG0Ve)
2. Using temporary：
> 使用临时表保存中间结果，也就是说mysql在对查询结果排序时使用了临时表，常见于order by 和 group by 

[![VzGsPA.md.png](https://s2.ax1x.com/2019/06/21/VzGsPA.md.png)](https://imgchr.com/i/VzGsPA)
3. Using index： 
> 表示相应的select操作中使用了覆盖索引（Covering Index），避免了访问表的数据行，效率高 
如果同时出现Using where，表明索引被用来执行索引键值的查找（参考上图） 
如果没用同时出现Using where，表明索引用来读取数据而非执行查找动作

![VzGcxP.png](https://s2.ax1x.com/2019/06/21/VzGcxP.png)

4. Using where
> 使用了where过滤

5. Using join buffrt
> 使用了链接缓存

6. Impossible WHERE
> where子句的值总是false，不能用来获取任何元祖 

7. select tables optimized away
> 在没有group by子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT（*）操作，不必等到执行阶段在进行计算，查询执行计划生成的阶段即可完成优化

8. distinct
> 优化distinct操作，在找到第一个匹配的元祖后即停止找同样值得动作


