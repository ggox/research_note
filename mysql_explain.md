# MySql_Explain详解

* 执行示例

  | id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
  | ---- | ----------- | ----- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |
  |  1 | SIMPLE      | test  | index | NULL          | PRIMARY | 4       | NULL |    1 | Using index |

* 字段详解

  1. id

     * 如果行记录的id值都一样则表示查询顺序是之上而下的
     * 如果id值一样则可以认为是同一组查询
     * id值不一样则表示可能存在子查询，如果输出null可能存在union查询，id值不一样的情况下id值越大则表示执行的顺序越靠前

  2. select_type

     * 查询类型，一般有12种返回结果

     | value                | comment                                                      |
     | -------------------- | ------------------------------------------------------------ |
     | SIMPLE               | 简单查询，标识查询不包含任何子查询或者UNION语句              |
     | PRIMARY              | 复杂查询的外层查询，一般都在第一行，代表这是一个复杂查询的最外层查询 |
     | UNION                | UNION中的第二个或后面的SELECT语句                            |
     | DEPENDENT UNION      | 复杂查询中，依赖外部查询的UNION子句查询                      |
     | UNION RESULT         | 复杂查询中，UNION的结果，这是一个从匿名临时表检索最终结果的查询 |
     | SUBQUERY             | 子查询中的第一个                                             |
     | DEPENDENT SUBQUERY   | 子查询的第一个，依赖外部查询的                               |
     | DERIVED              | SELECT, FROM子句的子查询                                     |
     | DEPENDENT DERIVED    | 依赖于另一张表的派生表查询                                   |
     | MATERIALIZED         | 复杂查询中，具体化的视图子查询                               |
     | UNCACHEABLE SUBQUERY | 不可缓存的子查询                                             |
     | UNCACHEABLE UNION    | 不可缓存的UNION子查询                                        |

  3. table

     对应语句访问哪张表

  4. type 性能优化中需要格外关注

     | Value           | Comment                                                      |
     | --------------- | ------------------------------------------------------------ |
     | system          | 特殊类型的const，表只有一行记录是会出现                      |
     | const           | 只有一行记录被匹配是会出现，例如使用的唯一索引和主键索引     |
     | eq_ref          | 性能良好的连接查询，对于前面结果的每一行都只有一条匹配记录   |
     | ref             | 比eq_ref性能要差，表示对于结果，每一行都有多条记录匹配       |
     | fulltext        | 全文索引                                                     |
     | ref_or_null     | 和ref差不多，只不过当索引可以为null时，条件中出现了IS NULL或IS NOT NULL时就会出现这种情况 |
     | index_merge     | 多个索引合并                                                 |
     | unique_subquery | 子查询返回了多行结果，但是每行结果只匹配一条记录             |
     | index_subquery  | 子查询返回了多行结果，但是每行结果都匹配了多个值             |
     | range           | 范围类型的查询                                               |
     | index           | 和all类型的全表扫描差不多                                    |
     | all             | 全表扫描                                                     |

  5. possible_keys

     可能会用到的索引

  6. key

     实际用到的索引

  7. key_len

     所使用的索引的最大长度，比如一个int字段len就是4字节长度，当然如果把它理解为索引字段的数据类型字节长度也不太对，这个因为这个长度还和是否可以为null有关，可以为null的话则key_len等于数据类型的字节长度+1.

  8. ref

     显示哪些列或者常量被用来跟key中显示的索引进行比较，从而获取结果，如果显示func，则说明是某些function的结果被用来进行比较

  9. rows

     表示mysql需要得到结果需要扫描的预计行数，这个值不是一个精准值

  10. filtered

      使用EXPLAIN EXTENDED才会显示的列，用百分比表示有多少行会被过滤出来去与之前的表关联rows * filtered / 100 就能得到行数

  11. extra

      额外的信息

* type 字段详解(性能由高到低)

  1. system

     当表里面只有一行数据的时候就会这样，而且不管有没有索引都一样，这是const连接类型的特殊情况

  2. const

     当查询只有唯一的一条记录被匹配，并且使用主键等于常数或者唯一索引等于某常数作为查询条件时， MySQL 会视查询出来的值为常数，这种类型非常快。和system不同的地方是system是表里面只有一行数据，而const有多行数据，const是只有一行数据被匹配

  3. eq_ref

     对于前一个表中的每个行组合，只从该表中读取一行。除了system和const类型之外，这是最好的连接类型。只有当联接使用索引的部分都是主键或惟一非空索引时，就会出现这种类型

  4. ref

     如果理解了eq_ref类型的话那么ref类型就也很好理解了，当联接使用索引的部分不是主键或惟一非空索引或者使用了最左前缀索引（包括唯一索引和主键的），就会出现这种类型，表示连接的键使用了多行

  5. fulltext

     使用了全文索引的情况下就会出现这种情况

  6. ref_or_null

     和ref类似，但是如果当被索引的字段可以为NULL时，就又可能出现这种情况

  7. index_merge

     表示使用了索引合并优化，每个索引单独使用，再通过某种算法来合并结果

  8. unique_subquery

     子查询类型的eq_ref，表示子查询返回了多列，但是每一列的值都只匹配一条记录

  9. index_subquery

     子查询类型的ref，和unique_subquery的关系类似于eq_ref和ref，表示子查询返回了多列，但是每一列都会匹配多个记录

  10. range

      表示使用了范围类型的查询，一般出现在使用了=, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, 货 IN() 之类的字句和常量值作比较时

  11. index

      和后面要介绍的all类型差不多，也是全表扫描，不过index有两种情况：

      * 如果是覆盖索引的情况下，只扫描整个索引树，这种情况要比all类型的全表扫描要快

      * 按索引顺序扫描全表，几乎和all类型没有区别

  12. all

      全表扫描

* extra 字段详解

  1. Child of 'table' pushed join@1

     这个表被引用为可以下推到NDB内核的连接中的表的子表。 只适用于MySQL NDB群集7.2及更高版本，当启用了下推连接时

  2. const row not found

     对于SELECT … FROM tbl_name之类的查询，该表为空

  3. Deleting all rows

     对于DELETE，一些存储引擎(如MyISAM)支持一个处理程序方法，该方法以一种简单而快速的方式删除所有表行。如果引擎使用此优化，则会显示此额外值

  4. Distinct

     MySQL正在寻找不同的值，所以当它找到第一个匹配的行后，它停止为当前行组合搜索更多的行

  5. FirstMatch

     对tbl_name进行了semi-join firstmatch优化，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个

  6. Full scan on NULL key

     子查询优化，当优化器不能使用索引查找访问的时候，采用回退策略

  7. Impossible HAVING

     HAVING子句始终为false，无法查询任何行

  8. Impossible WHERE

     WHERE子句始终为false，无法查询任何行

  9. Impossible WHERE noticed after reading const tables

     MySQL已经读取了所有const（和system）表，并发现where子句总是返回false

  10. LooseScan(m..n)

      Semi-join 松散策略被使用，m 和 n 是key 的一部分

  11. No matching min/max row

      没有满足查询条件的行，例如SELECT MIN（…）FROM … WHERE之类的条件结果行

  12. no matching row in const table

      使用了join，有空表，或者在唯一索引条件下没有匹配上的行

  13. No matching rows after partition pruning

      发生分区清理之后发现没有东西能够被delete或者update，和impossible where意思一样

  14. No tables used

      查询中没有FROM子句，或者FROM DUAL子句

  15. Not exists

      MySQL能够对查询执行LEFT JOIN优化，并且在找到与LEFT JOIN条件匹配的行之后，不会检查此表中针对上一行组合的更多行

      * 示例：

        ```mysql
        SELECT * FROM t1 LEFT JOIN t2 ON t1.id=t2.id WHERE t2.id IS NULL;
        ```

        假设t2.id被定义为NOT NULL。在本例中，MySQL扫描t1并使用t1.id的值查找t2中的行。如果MySQL在t2中找到匹配的行，它就知道是t2.id永远不能为空，并且不会扫描t2中具有相同id值的其余行。换句话说，对于t1中的每一行，MySQL只需要在t2中执行一次查询，而不管t2中实际匹配多少行

  16. Plan isn't ready yet

      当优化器尚未为在指定连接中执行的语句创建完执行计划时，EXPLAIN FOR CONNECTION将出现此值。如果执行计划输出包含多行，那么其中任何一行或所有一行都可能有这个额外的值，这取决于优化器在确定完整执行计划方面的进度

  17. Range checked for each record (index map: N)

      MySQL发现没有好的索引可以使用，但发现在前面的表的列值已知后可能会使用某些索引。 对于上表中的每个行组合，MySQL检查是否可以使用range或index_merge访问方法来检索行。 这不是很快，但比执行没有索引的连接更快

  18. Recursive

      这表明该行应用于递归公共表表达式的递归SELECT部分

  19. Rematerialize

      Rematerialize(X, …)显示在表t的执行计划中，其中X是在表t读取新一行时触发其Rematerialize的任何具体化派生表

  20. Scanned N databases

      这表示在处理INFORMATION_SCHEMA表的查询时服务器执行的目录扫描数。 N的值可以是0,1或all

  21. Select tables optimized away

      使用某些聚合函数来访问存在索引的某个字段时，优化器会通过索引直接一次定位到所需要的数据行完成整个查询，例如MIN()\MAX()，这种也是比较好的结果之一

  22. Skip_open_table, Open_frm_only, Open_full_table

      这些值表示用于INFORMATION_SCHEMA表查询打开文件的优化。

      1. Skip_open_table 不需要打开表文件。通过扫描数据库目录，该信息已在查询中可用
      2. Open_frm_only 只需要打开表的.frm文件
      3. Open_full_table 未经优化的信息查找，必须打开.frm，.MYD和.MYI文件

  23. Start temporary, End temporary

      这表示临时表用于半连接Duplicate Weedout策略

  24. unique row not found

      对于类似于SELECT … FROM tbl_name之类的查询，没有行满足表唯一索引或主键的条件

  25. Using filesort

      表示MySQL会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。 MySQL中无法利用索引完成的排序操作称为“文件排序”

  26. Using index

      仅使用索引树中的信息从表中检索列信息，而不必另外寻找读取实际行。当查询仅使用属于单个索引的列时，可能使用这个策略。
      对于具有用户定义的聚簇索引的InnoDB表，即使Extra列中不存在使用索引，也可以使用该索引。 如果type是index并且key是PRIMARY，则会出现这种情况

  27. Using index condition

      会先条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行

  28. Using index for group-by

      与Using index table访问方法类似，Using index for group-by表示MySQL找到了一个索引，可用于检索GROUP BY或DISTINCT查询的所有列，而无需对实际表进行任何额外的磁盘访问。 此外，索引以最有效的方式使用，因此对于每个组，只读取少数索引条目

  29. Using index for skip scan

      指示使用跳过扫描访问方法

  30. Using join buffer

      将联接中的表分成几部分读入连接缓冲区，然后从缓冲区中使用它们的行来与当前表执行连接。 （Block Nested Loop）表示使用块嵌套循环算法，（Batched Key Access）表示使用批量密钥访问算法。 也就是说，来自EXPLAIN输出前一行的表中的键将被缓冲，匹配的行将从连接缓冲区的行的表中批量提取

  31. Using MRR

      读取表时，使用了多范围读取优化策略

  32. Using sort_union(…), Using union(…), Using intersect(…)

      这些表示特定的算法，该算法显示如何合并index_merge连接类型的索引扫描

  33. Using temporary

      在查询时，MySQL需要创建一个临时表来保存结果。经常出现在查询包含以不同方式列出列的GROUP BY和ORDER BY子句的情况

  34. Using where

      在查找使用索引的情况下，需要回表去查询所需的数据，后再用WHERE子句完成结果过滤，通常出现这种情况的话就需要添加合适的索引来做优化了

  35. Using where with pushed condition

      仅适用于NDB表。这意味着NDB集群使用条件下推优化来提高非索引列和常量之间直接比较的效率。在这种情况下，条件被“下推”到集群的数据节点，并在所有数据节点上同时进行评估。这消除了通过网络发送不匹配行的需要，并且可以将这种查询的速度提高5到10倍，这是在可以使用条件下推送但没有使用的情况下

  36. Zero limit

      查询中存在LIMIT 0子句，无法选择任何行

  >  tips: 通常情况下当Extra中出现了Using filesort、Using temporary或Using where，则说明你写的SQL需要优化啦

  