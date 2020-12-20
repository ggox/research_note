## elasticsearch 学习笔记

### 一、基本概念

1. 文档

   * es是面向文档的，是所有可搜索数据的最小单位，文档是json格式，每个文档有一个Unique ID,可以自己自动或者es自动生产
   * 元数据：_index _type(默认_doc,未来可能被废除) _id _source _version _score

2. 索引

   * 文档的容器，文档的一个集合
   * 包含：Mapping  和 Setting
   
3. 节点
   
   * master/master-eligible/data/coordinating/hot/warm/machine learning/tribe(用于集群间交互，不建议使用)
   
   * | 节点类型         | 配置参数    | 默认值                |
       | ---------------- | ----------- | --------------------- |
       | maste eligible   | node.master | true                  |
       | data             | node.data   | true                  |
       | ingest           | Node.ingest | true                  |
       | coordinating     | 无          | 默认都是              |
       | machine learning | node.ml     | true(需enable x-pack) |
   
4. 分片

   * 主分片和副本分片，主分片数量创建索引时指定，后续不允许修改
   * 一个分片一个lucene实例
   * Green(主分片和副本分片都正常)、Yellow(主分片全部正常，有副分片未能正常分配)、Red(有主分片未能分配)

5. 倒排索引

   * 词典 （FST-确定无环优先状态转移器）
   * 倒排表

6. SearchAPI(具体请看官网api) [Documentationi链接](https://www.elastic.co/guide/index.html)

   * URI search
   * RquestBody search: term match match_all match_phrase query_string simple_query_string

7. Mapping/Dynamic Mapping

   * mapping类似数据库中的schema定义
   * 简单类型：text/keyword  date  integer/floating  boolean  ipv4/ipv6
   * 复杂类型：对象类型/嵌套类型
   * 特殊类型： geo_point/geo_shape  percolator
   * 修改字段定义的注意事项：
     1. 新增字段：
        * dynamic 设置为true时，一旦有新增字段写入，mapping也会同时更新
        * dynamic 设为false时，mapping不会被更新，新增字段的数据无法被索引，信息留在_source中
        * dynamic 设为strict时，文档写入失败
     2. 对已有字段，一旦数据写入，不再支持修改字段定义
   * 四种不同级别的Index Options
     1. docs - 记录 doc id
     2. freqs - 记录 doc id 和 term frequencies
     3. positions - 记录 doc id / term frequencies / term position
     4. offsets - doc id / term frequencies / term position /character offsets

8.  分词器

   * character filters:
     1. HTML strip - 去除html标签
     2. Mapping - 字符串替换
     3. Pattern replace - 正则匹配替换
   * tokenizer: whitespace / standard / uax_url_email / pattern / keyword / path hierarchy
   * token filters: lowercase / stop / synonym(添加近义词)

9. Index Template / Dynamic Template

   * Index Template - 帮助你设定 Mapping 和 Settings ，并按照一定规则，自动匹配到新创建的索引上
   * Dynamic Template - 根据es识别的数据类型，结合字段名称，动态设定字段类型

10. Aggregation (聚合)
	* 什么是聚合？
    * 统计分析功能，实时性高
    * 通过聚合，得到数据的概览，分析和总结圈套的数据
    * 高性能，只需要一条语句就可以得到分析结果
  * 聚合分类
    1. Bucket Aggregation - 一些列满足特定条件的文档的集合
    2. Metric Aggregation - 一些数学运算，可以对文档的字段进行统计分析
    3. Pipeline Aggregation - 对其他的聚合结果进行二次聚合
    4. Matrix Aggregation- 支持多个字段的操作并提供一个结果矩阵

### 二、深入查询

1. Term Query：不会分词，会返回score，有算分开销
2. Constant Score 转为 filter 有效利用缓存，避免算分开销
3. 相关性搜索 具体看文档吧

### 三、分布式特性

1. 分片：分为主分片和副本分片

2. 脑裂：master故障或者网络问题，如何避免：设置quorum = 节点数/2+1

   es7.0前可以设置master节点需要达到quorum的节点同意才行 7.0版本不需要再手动设置了

3. 路由算法：hash(_rounting) % primary_node_nums

   _rounting 默认为_id 也可以执行设置

4. 分片原理：

   * 倒排索引不可变

     优点：

     1. 无需考虑并发写文件问题，避免锁机制带来的性能问题
     2. 方便读入文件系统缓存，减少磁盘命中
     3. 缓存容易生成和维护、数据可以被压缩

     挑战：如果需要让一个新的文档被搜索到，需要重建整个索引

   * lucene index（一个es分片） ：

     1. 多个Segment的集合，segment是自包含的，<b>不可变的</b>
     2. 有新文档写入时，生成新的segment，查询时查询所有的segment并汇总结果。lucene中有一个文件，用来记录所有的segment信息，叫做commit point
     3. 删除的文档保存在“.del”文件中
     
   * es refresh
   
     1. Index Buffer 写入 segment的过程叫做refresh，不执行fsync,因为写入磁盘比较耗时，借助文件系统换出，先将segment写入缓存以开放查询
     2. refresh默认1秒一次，通过index.refresh_interval配置 refresh后文件才能被搜索到
     3. 系统有大量输入就会有大量segment
     4. Index Buffer 被占满时，会触发refresh，默认是jvm的10%
     5. 为了保证数据不丢失，在index文档时，同时写 transaction log,高版本开始transaction log 默认落盘。每个分片一个transaction log  因为是磁盘的顺序io性能较好
     6. refresh时Index Buffer被清空，transaction log不会被清空
     
   * es flush
   
     1. 调用refresh，index buffer 清空
     2. 调用fsync 将缓存中的segments 写入磁盘
     3. 清空transaction log
     4. 默认30分钟一次 或者 transaction log 满了（默认512M）
   
   * Merge
   
     1. segement很多，需要定期合并，减少sements/删除已经删除的文档
     2. es和lucene会自动进行merge，也可以手动触发：POST my_index/_forcemerge
   
   * 排序
   
     1. 排序子弹针对原始内容进行的，倒排索引无法发挥作用
   
     2. 需要用到正排索引，通过文档id和字段快速得到子弹的原始内容
   
     3. 两种实现：fileddata 和 doc values(列式存储)
   
        |          |           Doc Values           |                  Field Data                  |
        | -------- | :----------------------------: | :------------------------------------------: |
        | 何时创建 |   索引时，和倒排索引一起创建   |               搜索时候动态创建               |
        | 创建位置 |            磁盘文件            |                   JVM Heap                   |
        | 优点     |        避免大量内存占用        |       索引速度快，不占用额外的磁盘空间       |
        | 缺点     | 降低索引速度，占用额外磁盘空间 | 文档过多时，动态创建开销大，占用过多JVM Heap |
        | 缺省值   |           ES2.x之后            |                ES 1.x 及之前                 |
   
     4. 深度分页性能问题：需要从全部分片上获取所有可能的数据，在Coordinating节点上进行聚合返回
   
        * Search After：通过唯一排序值定位，将每次要处理的文档数控制在一定范围，不支持From参数
        * ScrollAPI：创建一个快照，有新数据写入后，无法没查询，每次查询需要输入上次的Scroll Id 可用于导出数据等业务场景 
   
     5. 并发读控制：乐观锁并发控制 
   
        * 内部版本控制：早期版本可以使用_version 新版本使用 if_seq_no + if_primary_term
        * 外部版本：version + version_type=external

### 四、深入聚合分析

* 分布式系统近似统计算法

  ![模型](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwsa3i0gaj30jg0c93zi.jpg)

* Min聚合分析的执行流程

  ![Min聚合分析](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwsdbbz3uj30jk09zwfe.jpg)

* Trems Aggregation

  ![terms聚合流程](https://tva1.sinaimg.cn/large/007S8ZIlly1ggwshcir6rj30ja09vwfd.jpg)

  1. doc_count_error_upper_bound：被遗漏的term分桶包含的文档有可能的最大值

  2. sum_other_doc_count：除了返回结果bucket的terms以外，其他terms的文档总数

  3. 解决精准度问题：
  
     * 数据量不大时，设置Primary Shard为1
     * 设置shard_size参数 提高精准度：每次从shar上额外多获取数据，提升准确率

### 五、数据建模

* 嵌套对象 vs 父子文档

  |          | Nested Object                            | Parent / Child                           |
  | -------- | ---------------------------------------- | ---------------------------------------- |
  | 优点     | 文档存储在一起，读取性能高               | 父子文档可以独立更新                     |
  | 缺点     | 更新嵌套对象的子文档时，需要更新整个文档 | 需要额外的内存维护关系；读取性能相对较差 |
  | 适用场景 | 子文档偶尔更新，已查询为主               | 子文档更新频繁                           |
  
* 重建索引：Update By Query  /  Reindex

* Ingest Node vs Logstash

|                | Logstash                               | Ingest Node                                            |
| -------------- | -------------------------------------- | ------------------------------------------------------ |
| 数据输入与输出 | 支持从不同数据源读取，并写入不同数据源 | 支持从es rest api获取数据，写入es                      |
| 数据缓冲       | 实现了简单的数据队列，支持重写         | 不支持缓冲                                             |
| 数据处理       | 支持大量的插件，也支持定制开发         | 内置插件，可以开发Plugin进行扩展（plugin更新需要重启） |
| 配置和使用     | 增加了一定的架构复杂度                 | 无需额外  部署                                         |

