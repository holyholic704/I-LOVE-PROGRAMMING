# MySQL

MySQL 默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭，重新恢复隐式提交

通过 set autocommit=0 可以取消自动提交，直到 set autocommit=1 才会提交；autocommit 标记是针对每个连接而不是针对服务器的。

## 数据库范式

## 语法相关

### count

对于 `COUNT(*)`、`COUNT(常数)`、`COUNT(主键)` 来说，优化器可以选择最小的索引执行查询，从而提升效率，它们的执行过程是一样的，只不过对于 NULL 值有不同的判断方式，这个判断为 `NULL` 的过程的代价可以忽略不计，所以可以认为 `COUNT(*)`、`COUNT(常数)`、`COUNT(主键)` 所需要的代价是相同的

对于 `COUNT(非主键列)` 来说，server 层必须要从 InnoDB 中读到包含非主键列的记录，所以优化器并不能随心所欲的选择最小的索引去执行

- `COUNT(*)`：包含 NULL 值
- `COUNT(常数)`：包含 NULL 值
- `COUNT(列名)`：不包含 NULL 值

## 主键

### 主键生成策略

如果没有用户自定义主键，MySQL 会自动生成主键

1. 优先使用用户自定义主键
2. 如果没有用户自定义主键，则选取一个 Unique 键
3. 如果没有 Unique 键，则 InnoDB 会为表默认添加一个名为 row_id 的隐藏列作为主键

建议使用自定义的有序的主键

- row_id 会占用 6 个字节，而自定义的主键使用 int 类型的话只占 4 个字节
- 插入数据需要生成 row_id，而生成的 row_id 是全局共享的，并发会导致锁竞争，影响性能

### 有序的主键

有序的主键性能会更好

- 写入的目标页很可能已经刷新到磁盘上并且从缓存上移除，或者还没有被加载到缓存中，InnoDB 在插入之前不得不先找到并从磁盘读取目标页到内存中，这将导致大量的随机 IO
- 数据是有序的存放在数据页上的，数据页满了会换一个新的页面进行插入。如果主键是乱序插入的，插入到一个已满的数据页，就会产生页分裂，造成不必要的性能损耗和碎片空间
- 建立索引时也就是一个排序的过程，如果数据原本就是有序的话，可以省略一些交换位置的操作

一般为了保证主键的有序性，会将主键设置为 AUTO_INCREMENT，但最好使用第三方的 ID 生成器

- 自增主键容易暴露一些业务信息
- 分库分表场景下可能出现主键冲突
- 使用自增主键可能会产生 AUTO-INC 锁的争夺，造成性能损耗

#### 自增主键一定是连续的吗

- auto_increment_offset（自增的初始值）或 auto_increment_increment（自增的步长）不为 1
- 唯一键冲突导致插入失败，此时主键已完成自增
- 事务回滚时，已发生的自增不会回滚
- 批量插入时，MySQL 会批量申请自增主键，如果数据插入完，没使用的自增主键也就被浪费了

### 自增主键用完了怎么办

- 修改字段类型，使用更大的数据类型，或将字段设置为 unsigned
- 如果表中数据量并没有达到主键的上限，可以考虑重新设置自增主键的起始值
- 使用分布式 ID 生成器
- 分库分表
- 如果是以 row_id 为主键的，达到最大值后会从 0 重新开始算；前面插入的数据就会被后插入的数据覆盖，且不会报错

## 查询语句的执行顺序

```sql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT
DISTINCT
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

1. 首先对 `FROM` 子句中的前两个表执行一个笛卡尔乘积，此时生成虚拟表 vt1
    - `FROM` 子句中写在最后的表将被最先处理，即最后的表为驱动表，当 `FROM` 子句中包含多个表的情况下，选择数据最少的表作为基础表
2. 对虚拟表 vt1 应用 `ON` 筛选器，`ON` 中的逻辑表达式将应用到虚拟表 vt1 中的各个行，筛选出满足 `ON` 逻辑表达式的行，生成虚拟表 vt2
3. 如果使用的是 `OUTER JOIN`，就将第二步中过滤掉的数据重新添加过来，生成虚拟表 vt3
4. 如果 `FROM` 子句包含两个以上的表，就对虚拟表 vt3 和下一个表重复 1 ~ 3 的步骤，最终得到一个新的虚拟表 vt3
5. 对虚拟表 vt3 应用 `WHERE` 筛选器。根据指定的条件对数据进行筛选，并把满足的数据插入虚拟表 vt4
6. 按 `GROUP BY` 子句中的列表将虚拟表 vt4 中的行唯一的值组合成为一组，生成虚拟表 vt5。如果应用了 `GROUP BY`，那么后面的所有步骤都只能得到的虚拟表 vt5 的列或者是聚合函数。原因在于最终的结果集中每一个组只用一行数据来表示
    - 从这一步开始，后面的语句中都可以使用 `SELECT` 中的别名
7. 对虚拟表 vt5 应用 `HAVING` 筛选器，根据指定的条件对数据进行筛选，并把满足的数据插入虚拟表 vt6
8. 将虚拟表 vt6 中的在 `SELECT` 中出现的列筛选出来，产生虚拟表 vt7
9. 将重复的行从虚拟表 vt7 中移除，产生虚拟表 vt8
10. 将虚拟表 vt8 中的行按 `ORDER BY` 子句中的列表进行排序，生成一个游标
11. 使用 `LIMIT` 指定需要返回的行数

使用内连接时，我们写的 `ON` 条件都会解析成 `WHERE` 条件，所以我们将条件写在 `ON` 或者 `WHERE` 里是没有区别的

## 架构

![](./md.assets/mysql_structure.png)

<small>[SQL语句在MySQL中的执行过程](https://javaguide.cn/database/mysql/how-sql-executed-in-mysql.html)</small>

### server 层

#### 连接器

主要负责用户登录数据库，进行用户的身份认证

#### 查询缓存

MySQL 8.0 版本后已删除该功能

主要用来缓存我们所执行的 `SELECT` 语句以及该语句的结果集，以查询语句为 key，结果集为 value，缓存在内存中

在执行查询语句时，会先查询缓存。如果缓存命中，就会直接返回，没有命中，才会执行查询语句，执行完成后也会将结果集缓存起来

两个查询在任何字符上的不同都会导致缓存不命中，且如果对表结构或数据进行了修改，那么所有与这个表相关的缓存都会失效

#### 分析器

1. 词法分析：一条 SQL 语句有多个字符串组成，首先要提取关键字，比如 `SELECT`，提出查询的表，提出字段名，提出查询条件等
2. 语法分析：判断输入的 SQL 是否正确，是否符合 MySQL 的语法

#### 优化器

选择最优的方案执行 SQL 语句

#### 执行器

执行 SQL 语句

### 语句分析

#### 查询语句是如何执行的

1. 先检查该语句是否有权限，如果没有权限，直接返回错误信息，如果有权限会先查询缓存（MySQL 8.0 版本之前）
2. 使用分析器进行词法分析和语法分析
3. 使用优化器确定执行方案
4. 按照生成的执行计划，执行并返回结果

#### 更新语句是如何执行的

1. 先找到要修改的数据
2. 对数据进行修改后，调用引擎 API 接口，写入这一行数据
3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 中，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务
4. 执行器收到通知后记录 binlog，并把 binlog 写入磁盘，然后调用引擎接口，提交 redo log 为提交状态，更新完成

## 存储引擎

![](./md.assets/all_engines.png)

![](./md.assets/engines.png)

<small>[MySQL进阶之存储引擎【InnoDB、MySAM、Memory】](https://blog.csdn.net/weixin_53041251/article/details/124241047)</small>

- InnoDB 支持的哈希索引是自适应的，InnoDB 会根据表的使用情况自动为表生成哈希索引，不能人为干预是否在一张表中生成哈希索引

### Memory

所有的数据都存储在内存中，数据的处理速度快，但是安全性不高，存储的数据量受到内存大小的限制

### InnoDB 与 MylSAM

- 事务：InnoDB 支持，MylSAM 不支持
- 外键：InnoDB 支持，MylSAM 不支持
  - 不建议在实际生产环境中使用外键约束
- 锁粒度：InnoDB 支持行级锁，MylSAM 只支持表级锁
- 表的具体行数：MyISAM 保存了表的总行数
- 索引：InnoDB 与 MyISAM 都使用 B+ 树作为索引结构，但具体实现的方式不一样
  - InnoDB 索引与数据是存放在一起的，索引叶子节点存储的就是数据记录，即聚簇索引
  - MyISAM 索引文件和数据文件是分离的，索引叶子节点存储的是数据记录的地址，需要根据地址读取相应的数据记录，即非聚簇索引

## 深度分页

在使用 `LIMIT` 进行分页查询时，当查找的偏移量越高时，查询的性能越低

`LIMIT` 在进行查找时，需要扫描 `offset + N` 行，并舍弃掉前 offset 行，并返回 N 行。当偏移量越大时，需要扫描的记录也就越多

如果没有覆盖索引，那么回表的次数也就越多，而通常情况下使用分页的业务场景，需要的基本上是一个表的全部或大部分字段，所以很难满足索引覆盖的条件

### 使用主键进行范围查询

需保证主键是连续的，但在实际环境中主键很难保证完全连续，所以此方法使用的不多

```sql
SELECT id FROM test WHERE id > 1000000 LIMIT 10;
```

### 子查询

先使用子查询查出 `LIMIT` 范围内的主键，再用主键去查询所需的列

```sql
SELECT * FROM test WHERE id IN (SELECT id FROM test LIMIT 1000000, 10);
```

子查询的结果会产生一张新表，会影响性能，应该尽量避免大量使用子查询

### 延迟关联

与子查询的优化思路类似，都是把条件转移到主键索引，减少回表的次数。不同点是，延迟关联使用了内连接包含子查询

```sql
SELECT * FROM test T1 INNER JOIN (SELECT id FROM test LIMIT 1000000, 10) T2 ON T1.id = T2.id;
```

### 覆盖索引

查询时只查出索引中包含的列

## 优化建议

- 尽可能的使用索引，最好是覆盖索引或主键索引
- 避免使用 `SELECT *`，只查询需要的字段
- 深度分页优化
- 避免多表 `JOIN`，最好不要超过 3 个表
  - 可以考虑增加冗余字段
  - 尽量使用小表驱动大表
- 避免隐式转换
- 使用合适的字段类型，尽量使用更小的字段类型
  - 对于一些不可能为负数的数据，可以将字段设置为 unsigned
  - 日期类型不要使用字符串
  - 金额使用 decimal
- 尽量使用自定义的有序主键
- 尽量为列设置默认值，最好不为 NULL
- 尽量用 `UNION ALL` 代替 `UNION`
  - `UNION ALL` 会有去重操作
- 尽量使用批量操作，但不要过大
- 使用 EXPLAIN 分析 SQL 语句
- 尽量不在数据库做运算，复杂运算需移到业务应用里完成
- 避免使用子查询，可以把子查询优化为连接查询
  - 子查询的结果集无法使用索引，通常子查询的结果集会被存储到临时表中，不论是内存临时表还是磁盘临时表都不会存在索引，所以查询性能会受到一定的影响
  - 由于子查询会产生大量的临时表也没有索引，所以会消耗过多的 CPU 和 IO 资源，产生大量的慢查询
- 拆分复杂的大 SQL 为多个小 SQL
- 避免产生大事务操作

## EXPLAIN

通过 `EXPLAIN` 命令可以获取 MySQL 的执行计划，帮助我们对 SQL 进行优化

### id

查询语句中每出现一个 `SELECT` 关键字，MySQL 就会为它分配一个唯一的 id 值

id 如果相同，从上往下依次执行。id 不同，id 值越大，执行优先级越高，如果行引用其他行的并集结果，则该值可以为 NULL

### select_type

查询的类型

- SIMPLE：简单查询，查询语句中不包含 `UNION` 或者子查询的查询
- PRIMARY：查询中如果包含 `UNION` 或者子查询，最左边或最外层的 `SELECT` 将被标记为 PRIMARY
- SUBQUERY：子查询中的第一个 `SELECT`
- UNION：在 `UNION` 查询中，除了第一个查询之后出现的 `SELECT` 都会被标记为 UNION
- DERIVED：在 `FROM` 中出现的子查询将被标记为 DERIVED
- UNION RESULT：`UNION` 查询的结果

### table

查询用到的表名，每行都有对应的表名

### partitions

一般情况下查询语句的执行计划的 partitions 列的值都是 NULL

### type

查询执行的类型，描述了查询是如何执行的，以性能从最优到最差排序

- system：当表中只有一条记录并且该表使用的存储引擎的统计数据是精确的，比如 MyISAM、Memory
- const：表中最多只有一行匹配的记录，一次查询就可以找到，常用于使用主键或唯一索引的所有字段作为查询条件
- eq_ref：当连表查询时，前一张表的行在当前这张表中只有一行与之对应
- ref：使用普通索引作为查询条件，查询结果可能找到多个符合条件的行
- fulltext：全文索引
- ref_or_null：使用普通索引作为查询条件，并需要查出 NULL 值
- index_merge：当查询条件使用了多个索引时，表示使用了索引合并
- range：对索引列进行范围查询
- index：扫描整个索引
- ALL：全表扫描

### possible_keys

执行查询时可能用到的索引

### key

实际使用到的索引

### key_len

索引的最大长度

### rows

预计需要扫描的行数

### filtered

在使用索引时，有多少条记录满足搜索条件，是一个估计的百分比

### Extra

额外信息

- Using filesort：无法使用索引进行排序，只能在内存中或者磁盘中进行排序
- Using temporary：需要创建临时表来存储查询的结果，常见于 `ORDER BY` 和 `GROUP BY`
- Using index：使用了索引覆盖
- Using index condition：使用了索引下推
- Using where：在存储引擎检索出数据后，返回给 server 层时再进行过滤
- Using join buffer (Block Nested Loop)：连表查询的方式，表示当被驱动表的没有使用索引的时候，MySQL 会先将驱动表读出来放到 join buffer 中，再遍历被驱动表与驱动表进行查询

## 参考

- [MySQL 是怎样运行的：从根儿上理解 MySQL](https://juejin.cn/book/6844733769996304392)
- [mysql中varchar能存多少汉字、数字，以及varchar(100)和varchar(10)的区别](https://blog.csdn.net/weixin_43431218/article/details/124734940)
- [mysql中的int(10)int(20)分别代表什么意思](https://blog.csdn.net/weixin_45707610/article/details/131439336)
- [SQL 查询语句的执行顺序解析](https://learnku.com/articles/35655)
- [Mysql关键字执行顺序-深入解析](https://developer.aliyun.com/article/1131899)
- [MySQL中SQL语句的执行顺序（详细）](https://www.cnblogs.com/antLaddie/p/17175396.html)
- [SQL语句在MySQL中的执行过程](https://javaguide.cn/database/mysql/how-sql-executed-in-mysql.html)
- [MySQL的表没有设置主键带来的问题](https://blog.csdn.net/user2025/article/details/115430396)
- [Mysql主键不要使用uuid或者不连续不重复雪花id](https://www.cnblogs.com/xiaobaicai12138/p/17833311.html)
- [MySql为什么不推荐使用UUID做主键](https://blog.csdn.net/chenwiehuang/article/details/123420278)
- [MySQL自增主键一定是连续的吗](https://javaguide.cn/database/mysql/mysql-auto-increment-primary-key-continuous.html)
- [面试官：数据库自增 ID 用完了会咋样？](https://juejin.cn/post/6984275678761844743)
- [MySQL 自增 ID 用完了怎么办？4 种解决方案！](https://xie.infoq.cn/article/2c7109ed4c4a82413ca43863e)
- [深度分页介绍及优化建议](https://javaguide.cn/high-performance/deep-pagination-optimization.html)
- [实战！聊聊如何解决MySQL深分页问题](https://juejin.cn/post/7012016858379321358)
- [MySQL高性能优化规范建议总结](https://javaguide.cn/database/mysql/mysql-high-performance-optimization-specification-recommendations.html)
- [MySQL执行计划分析](https://javaguide.cn/database/mysql/mysql-query-execution-plan.html)
