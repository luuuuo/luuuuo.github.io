# 介绍

## 演进历程

传统BI设计初衷在于分析类视角，但实际应用效果却不能令人满意。主要原因是

* 传统BI系统对企业的信息化水平要求较高
* 狭小的受众制约了传统BI的发展生命力
* 冗长的研发过程滞后了需求的响应时效

传统BI体系中，背后是传统的关系型数据库技术OLTP，基本操作思路是

* 对数据进行分层，层层递进形成数据集市，减少最终的查询体量
* 数据立方体，预处理数据，以空间换时间

从2003年后，Hadoop横空出世，虽然从非关系型数据库技术上实现BI，但在海量数据下实现多维分析（OLAP）的实时应答仍旧困难重重。常用的OLAP的架构大致为

* ROLAP（Relation OLAP），关系型OLAP，常用星型、雪花型数据模型。初期主要以MySQL、Oracle为代表，后期转借Hive和SparkSQL。
* MOLAP（Multidimensional OLAP），多维型OLAP，多维数组保存数据，以空间换时间。初期借助物化视图形式实现数据立方体，后期也转借Hadoop生态，如Kylin等。
* HOLAP（Hybrid OLAP），以上2种的混合。

| 发展历程    | OLAP架构 | Yandex.Metrica产品形态 | 备注                                  |
| ----------- | -------- | ---------------------- | ------------------------------------- |
| MySQL时期   | ROLAP    | 固定报告               | 无法做到顺序存储                      |
| Metrage时期 | MOLAP    | 固定报告               | 使用KV数据模型和LSM树索引，预处理数据 |
| OLAPServer  | HOLAP    | 自助报告               | 引入列式存储思想                      |
| ClickHouse  | ROLAP    | 自助报告               |                                       |

## 核心特性

* 完备的DBMS功能

数据库管理系统，支持DDL、DML、权限控制、数据备份与恢复、分布式管理等。

* 列式存储与数据压缩
* 向量化执行引擎
* 关系模型与SQL查询
* 多样化的表引擎
* 多线程与分布式
* 多主架构

每个节点角色对等，客户端访问任意节点都能得到相同效果。

* 在线查询
  复杂场景下，也可以做到快速响应，无需预处理数据。
* 数据分片与分布式查询

## 性能测试

[开源OLAP引擎测评报告(SparkSql、Presto、Impala、HAWQ、ClickHouse、GreenPlum)](http://www.clickhouse.com.cn/topic/5c453371389ad55f127768ea)
[Elasticsearch和Clickhouse基本查询对比](https://zhuanlan.zhihu.com/p/353296392)

### 与分析型数据库对比

![](https://uploader.shimo.im/f/y9gXUqNrQK0YsRIJ.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

官方测试图来源：[Performance comparison of analytical DBMS](https://clickhouse.tech/benchmark.html#[%221000000000%22,[%22ClickHouse%22,%22Vertica%20(x6)%22,%22Greenplum(x2)%22,%22Greenplum%22],[%222%22]])

### 与presto对比

ClickHouse集群：32核128G内存机器60台，ReplicatedMergeTree引擎，1分片2副本。
Presto集群的：32核128G内存机器100台。

|                                                  | ClickHouse集群   | presto集群         |
| ------------------------------------------------ | ---------------- | ------------------ |
| count() 亿级别数据                               | 2s，计算机器数30 | 80s，计算机器数100 |
| 亿级别指标 count() + group by + order by + limit | 5s               | 100s               |

测试来源：[趣头条基于 ClickHouse 玩转每天 1000 亿数据量](https://www.infoq.cn/article/g94gMf26m6ONtNeoe0r5)

### 为什么快

ClickHouse是一种列式数据库管理系统，它之所以快速，是因为它采用了多种技术来提高查询性能。以下是一些可能导致ClickHouse快速的原因：

* 列式存储：ClickHouse使用列式存储，这意味着它只加载需要的列，而不是整个行。这使得ClickHouse能够更快地访问和处理大量数据。
  * 对于列的聚合，计数，求和等统计操作原因优于行式存储。
  * 由于某一列的数据类型都是相同的，针对于数据存储更容易进行数据压缩，每一列选择更优的数据压缩算法，大大提高了数据的压缩比重。
  * 由于数据压缩比更好，一方面节省了磁盘空间，另一方面对于 cache 也有了更大的发挥空间。
* 数据压缩：ClickHouse使用多种压缩算法来压缩数据，从而减少磁盘空间和网络带宽的使用。这也使得ClickHouse能够更快地读取和写入数据。
* 数据分区：ClickHouse将数据分成多个分区，每个分区都是一个独立的数据块。这使得ClickHouse能够更快地过滤和查询数据。
* 并行处理：ClickHouse可以在多个CPU核心上并行处理查询，从而提高查询性能，单条 Query 就能利用整机所有 CPU。

这些技术的结合使得ClickHouse成为一种非常快速的数据库管理系统，特别适用于需要处理大量数据的场景。

## 版本规则

LTS，即 Long Term Support 。大家应该对这个并不陌生，很多开源软件都支持 LTS 版本，比如 NodeJS。但是每家软件对于 LTS 发布的规则是不一样的。
ClickHouse LTS 版本的发布规则是: 每半年发布一次 LTS 大版本;在上一个 LTS 半年后，选择当时至少被一个大客户使用过的 stable 版本作为新的 LTS 版本。
针对不同的用户群体，ClickHouse 现在提供了不同的发布版本供选择。
如果你是稳定性优先的用户，可以选择 LTS 版本；
如果你是新特性优先的用户，使用普通的 stable 版本即可。
[ClickHouse 的 LTS 版本是什么?](https://mp.weixin.qq.com/s/N_vvZoUDXUCszTFiW6Oqpw)

# 数据库引擎

拥有ClickHouse和MySQL引擎，可以操作ClickHouse，也可以通过ClickHouse来操作MySQL。使用ClickHouse的MySQL引擎操作MySQL，可以获得比MySQL更好的性能。

> ClickHouse是一个快速聚合的数据库，使用GROUP BY的查询是使用ClickHouse端的MySQL数据聚合的，而不是通常在MySQL上聚合的。[MySQLのデータをClickHouseで高速に集计する](https://qiita.com/mikage/items/fba19b3b8697fc6f73f0#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)

![](https://uploader.shimo.im/f/g3uOblmZzoA9Wzhr.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

从左到右分别是MySQL中的时间，ClickHouse从MySQL读取和处理数据的时间以及处理在ClickHouse上复制的数据的时间。

# 表引擎

## 介绍

表引擎（即表的类型）决定了：

* 数据的存储方式和位置，写到哪里以及从哪里读取数据
* 支持哪些查询以及如何支持。
* 并发数据访问。
* 索引的使用（如果存在）。
* 是否可以执行多线程请求。
* 数据复制参数。

## MergeTree Family

[ClickHouse各种MergeTree的关系与作用](https://mp.weixin.qq.com/s/0bXEfAN3xRXOWgGKr2GQZg)

### MergeTree

> Engines in the MergeTree family are designed for inserting a very large amount of data into a table. The data is quickly written to the table part by part, then rules are applied for merging the parts in the background. This method is much more efficient than continually rewriting the data in storage during insert.

MergeTree用于插入巨量数据。通过块状插入并在后台进行重排合并的方式并远比插入时就排序高效的多。

[一分钟视频解读ClickHouse MergeTree](https://mp.weixin.qq.com/s/6v-d5daDMC3AM8p5hJiZvA)

#### 写数据

表ddl：

```sql
CREATE TABLE ClickHouse_test.clickhouse (
 sum_date Date,
  x String,
  y String,
  z String
) ENGINE = MergeTree PARTITION BY toYYYYMMDD(sum_date) ORDER BY (x, y) SETTINGS index_granularity = 3
```

插入数据

```sql
insert into clickhouse values('2020-02-02', '1', 'a','1');
insert into clickhouse values('2020-02-02', '1', 'a','1');
insert into clickhouse values('2020-02-02', '1', 'b','1');
insert into clickhouse values('2020-02-02', '1', 'b','1');
insert into clickhouse values('2020-02-02', '2', 'c','1');
insert into clickhouse values('2020-02-02', '3', 'b','1');
insert into clickhouse values('2020-02-02', '3', 'b','1');
insert into clickhouse values('2020-02-02', '3', 'c','1');
insert into clickhouse values('2020-02-02', '4', 'e','1');
insert into clickhouse values('2020-02-02', '5', 'a','1');
insert into clickhouse values('2020-02-02', '5', 'b','1');
insert into clickhouse values('2020-02-02', '5', 'b','1');
insert into clickhouse values('2020-02-02', '6', 'b','1');
insert into clickhouse values('2020-02-02', '6', 'c','1');
insert into clickhouse values('2020-02-02', '6', 'd','1');
insert into clickhouse values('2020-02-02', '7', 'a','1');
```

![](https://uploader.shimo.im/f/YiYwajgNUlkDd23i.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

数据组成：

![](https://uploader.shimo.im/f/PAJ5PfCB5ms2WFQv.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

过段时间内部merge后

![](https://uploader.shimo.im/f/ugk8n7L6Y6gUYpZg.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

对记录的删除也是类似情况，虽然客户端select结果为空，但是数据暂时还存在于库中。
进入目录/home/grp/ClickHouse/data/ClickHouse_test/ClickHouse/20200202_1_15_3

```shell
总用量 56
-rw-r-----. 1 ClickHouse 395 2月  10 17:56 checksums.txt
-rw-r-----. 1 ClickHouse  86 2月  10 17:56 columns.txt
-rw-r-----. 1 ClickHouse   2 2月  10 17:56 count.txt
-rw-r-----. 1 ClickHouse   4 2月  10 17:56 minmax_sum_date.idx
-rw-r-----. 1 ClickHouse   4 2月  10 17:56 partition.dat
-rw-r-----. 1 ClickHouse  20 2月  10 17:56 primary.idx
-rw-r-----. 1 ClickHouse  37 2月  10 17:56 sum_date.bin
-rw-r-----. 1 ClickHouse  80 2月  10 17:56 sum_date.mrk
-rw-r-----. 1 ClickHouse  53 2月  10 17:56 x.bin
-rw-r-----. 1 ClickHouse  80 2月  10 17:56 x.mrk
-rw-r-----. 1 ClickHouse  52 2月  10 17:56 y.bin
-rw-r-----. 1 ClickHouse  80 2月  10 17:56 y.mrk
-rw-r-----. 1 ClickHouse  37 2月  10 17:56 z.bin
-rw-r-----. 1 ClickHouse ClickHouse  80 2月  10 17:56 z.mrk
```

ClickHouse_test为库名，ClickHouse为表名，20200202_1_15_3为其中一个part（分区），每次插入数据就会生成一个part，part会不定时的merge成更大的一个part，每个part里的数据都是按照主键排序存储的。Data belonging to different partitions are separated into different parts.
checksums.txt：校验值文件
columns.txt：列名文件，记录了表中的所有列名
count.txt 记录数
minmax_sum_date.idx
partition.dat
primary.idx：主键文件，存储了主键值，数据结构类似于一系列marks组成的数组，这里的marks就是每隔index_granularity行取的主键值，一般默认index_granularity=8192。

```shell
cat primary.idx 
1a1b3b5a6b7a
```

column_name.bin：每个列都有一个bin文件，里边存储了压缩后的真实数据
column_name.mrk：每个列都有一个mrk文件，每隔 index_granularity行就会记录一次offset。
primark.idx和column_name.mrk文件做了逻辑行的映射关系

![](https://uploader.shimo.im/f/AxIDwcaqMMwZOq8B.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

分区目录的命名规则

图来源：[ClickHouse MergeTree原理解析-朱凯](https://github.com/ClickHouse-China/ClickhouseMeetup/blob/master/20191020%20Clickhouse%20Meetup%20(Shenzhen)_Git/2.%20ClickHouse%20MergeTree%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90-%E6%9C%B1%E5%87%AF.pdf)

#### 读数据

![](https://uploader.shimo.im/f/CG9ghY1t0k0OxXtu.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

MergeTree存储结构

![](https://uploader.shimo.im/f/xSnofVxkJYYYzvWW.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

MergeTree携带不同查询条件查询过程

图来源：[MySQL DBA解锁数据分析的新姿势-ClickHouse-新浪-高鹏-2017年12月08日](http://www.clickhouse.com.cn/topic/5a6c27509d28dfde2ddc5e76)

新浪高鹏：[http://m.zm518.cn/zhangmen/livenumber/share/entry/?liveId=1460023&amp;sharerId=6fd3bac16125e71d69-899&amp;circleId=b0b78915b2edbfe6c-78f7&amp;followerId=&amp;timestamp=1517022274560&amp;from=groupmessage&amp;isappinstalled=0#/](http://m.zm518.cn/zhangmen/livenumber/share/entry/?liveId=1460023&sharerId=6fd3bac16125e71d69-899&circleId=b0b78915b2edbfe6c-78f7&followerId=&timestamp=1517022274560&from=groupmessage&isappinstalled=0#/) 2h40m处

* 携带分区和主键的查询

1. 定位具体的part分区数据
2. 根据primary.idx主键定位数据block_id
3. 根据block_id查询.rmk文件定位真正数据的offset
4. 通过向量扫描此段offset的bin内容。

* 未携带分区和主键的查询
  全表扫描。
  查询时务必使用分区，且尽量传递第一个主键，最好能使用所有主键进行查询。

### ReplicatedMergeTree

```sql
CREATE TABLE duoxue.test_center_base_dictionary_replica (
`id` Int64,
 `tenant_id` Int64,
 `name` Nullable(String),
 `mnemonic_code` Nullable(String),
 `dictionary_type_name` Nullable(String),
 `description` Nullable(String),
 `create_time` Nullable(DateTime),
 `create_by` Nullable(Int64),
 `update_time` DateTime,
 `update_by` Nullable(Int64),
 `is_deleted` Nullable(Int8),
 `code` Nullable(String),
 `order_by` Nullable(Int32)
) ENGINE = ReplicatedMergeTree('/data/clickhouse/tables/duoxue/{shard}/test_center_base_dictionary_replica', '{replica}') PARTITION BY tenant_id ORDER BY (id, update_time) SETTINGS index_granularity = 8192
```

#### Zookeeper内节点结构

在zk_path为根路径，在每张表下创建一组监听节点

```shell
[zk: localhost:2181(CONNECTED) 22] ls /data/clickhouse/egypt_mdm/01/ft_app_used_d_202003_replica  
[metadata, temp, mutations, log, leader_election, columns, blocks, nonincrement_block_numbers, replicas, quorum, block_numbers]
```

* metadata
* temp
* mutations

保存alter delete和alter update指令

* log

保存副本中需要执行的指令

* leader_election
* columns
* blocks
* nonincrement_blck_numbers
* replicas
* quorum
* block_numbers

#### 副本协同流程

以写入数据同步为例

| 操作                      | Zookeeper动作                                                                     |
| ------------------------- | --------------------------------------------------------------------------------- |
| 创建第一个副本replica实例 | 在${zk_path}/下创建副本实例启动监听任务，监听/log日志节点参与副本选举，选举主副本 |
| 创建第二个副本replica实例 |                                                                                   |
| 向第一个副本写入数据      | 在本地分区写入数据向/blocks节点写入数据分区block_id向/log节点推送操作日志         |
| 其余副本实例开始同步数据  | 拉取log日志向远端副本请求下载本地副本下载数据并完成本地写入                       |

#### Q&A

Q： 更改、删除某一台的本地表结构，集群其他表是否同步改变。
A： 表结构更改可以同步，表删除无法同步。
Q： 更改、删除某一台本地表记录，集群其他表是否同步改变。
A： 是。更改和删除记录都会同步。

在jdbc连接中，指定ClickhouseSource的ClickhouseProperties配置

```java
@Configuration
@Service
public class DbConfig {
    JdbcTemplate jdbcTemplate;
    @Value("${spring.datasource.url}")
    private String datasource;
    JdbcTemplate getJdbcTemplate(){
        if(jdbcTemplate == null){
            jdbcTemplate = clickHouseJdbcTemplate(clickHouseDataSource());
        }
        return jdbcTemplate ;
    }
    ClickHouseProperties properties;
    public ClickHouseProperties getProperties(){
        if(properties == null){
            properties = new ClickHouseProperties();
            properties.setInsertDeduplicate(false);
        }
        return properties;
    }
    @Bean
    public DataSource clickHouseDataSource() {
        BalancedClickhouseDataSource balancedClickhouseDataSource = new BalancedClickhouseDataSource(datasource, getProperties());
        balancedClickhouseDataSource.scheduleActualization(1, TimeUnit.MINUTES);
        return balancedClickhouseDataSource;
    }
    @Bean
    public JdbcTemplate clickHouseJdbcTemplate(DataSource clickHouseDataSource) {
        final JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(clickHouseDataSource);
        jdbcTemplate.setQueryTimeout(300000);
        return jdbcTemplate;
    }
}
```

* 本地复制表无法自同步
  Q： node1和node2组成ck集群，往ck1写入数据，ck2未同步到数据。
  A： 检查内部通信端口，如果不是真正物理隔离的机器，在config.xml文件中需要配置interserver_http_host和interserver_http_port配置，避免冲突。

### SummingMergeTree

用户只需要查询数据的汇总结果，不关系明细数据，并且待会数据的汇总条件是预先明确的。

### AggregatingMergeTree

在数据合并分区时，按照预先定义的条件聚合数据，通常作为无话视图的表引擎，与普通MergeTree搭配使用。

[利用Null引擎、AggregateFunction、AggregatingMergeTree和物化视图构建数据管道](https://mp.weixin.qq.com/s/qM1kD3TH9Hft4zxUUeWE_Q)

### CollapsingMergeTree

以增代删，支持行级数据修改和删除。

### VersionCollapsingMergeTree

对数据的写入顺序没有要求，在同一个分区内，任意顺序的数据都能够完成折叠操作。

## database engine

### MaterializeMySQL

[MySQL到ClickHouse 实时复制与实现](https://www.modb.pro/db/28316)
[clickhouse高级功能之MaterializeMySQL详解](https://www.jianshu.com/p/d0d4306411b3)
[ClickHouse和他的朋友们（9）MySQL实时复制与实现](https://bohutang.me/2020/07/26/clickhouse-and-friends-mysql-replication/#%E4%BB%A3%E7%A0%81%E8%8E%B7%E5%8F%96)

1.支持mysql 库级别的数据同步，暂不支持表级别的。
2.MySQL 库映射到clickhouse中自动创建为ReplacingMergeTree引擎的表
3.支持全量和增量同步，首次创建数据库引擎时进行一次全量复制，之后通过监控binlog变化进行增量数据同步
4.支持的MySQL版本：5.6 5.7 8.0
5.支持的操作:insert，update，delete，alter，create，drop，truncate等大部分DDL操作
6.支持的MySQL复制为GTID复制
确保MySQL以下配置开启

```sql
mysql>  show variables like 'ENFORCE_GTID_CONSISTENCY';
| enforce_gtid_consistency | ON    |
mysql> show variables like '%binlog_format%';
| binlog_format | ROW   |
mysql> show variables like 'server_id';
| server_id     | 1     |
mysql> show variables like 'gtid_mode';
| gtid_mode     | ON    |
mysql> show variables like 'ENFORCE_GTID_CONSISTENCY';
| enforce_gtid_consistency | ON    |
```

确保Clickhouse以下配置开启

```sql
select * from system.settings where name ='allow_experimental_database_materialize_mysql';
set allow_experimental_database_materialize_mysql=1;
```

MySQL创建库表，

```sql
CREATE TABLE `t1` (
  `a` int(11) NOT NULL AUTO_INCREMENT,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`a`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=latin1;
```

对应进行MySQL的DDL，查询Clickhouse得到实时同步。

## 外部存储类型

### MySQL

```sql
CREATE TABLE duoxue.eln_posts ENGINE=MergeTree order by id AS  SELECT * FROM mysql('192.168.182.151:3306','eln_1','post','big_data','big_data');
```

目前无法读取MySQL的bit和json类型，[MaterializeMySQL don&#39;t support the json,bit,time data type. #14735](https://github.com/ClickHouse/ClickHouse/issues/14735)

创建MySQL的外表

![](https://uploader.shimo.im/f/ifjQ0vgG7cSSUbAU.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

使用MySQL表引擎带来的速度：[AGGREGATE MYSQL DATA AT HIGH SPEED WITH CLICKHOUSE](https://altinity.com/blog/2018/2/12/aggregate-mysql-data-at-high-speed-with-clickhouse)

### HDFS

> Usage
> ENGINE = HDFS(URI, format)
> The URI parameter is the whole file URI in HDFS. The format parameter specifies one of the available file formats. To perform SELECT queries, the format must be supported for input, and to perform INSERT queries -- for output. The available formats are listed in the [Formats](https://clickhouse.tech/docs/en/interfaces/formats/#formats) section. The path part of URI may contain globs. In this case the table would be readonly.

从19.14版本开始支持hdfs，截止版本20.3都不支持kerberos验证，[https://github.com/ClickHouse/ClickHouse/issues/5747](https://github.com/ClickHouse/ClickHouse/issues/5747)

### JDBC

不仅可以对接MySQL，还可以与PostgreSQL、SQLite和H2数据库对接。

### Kafka

[Clickhouse Engine kafka 将kafka数据同步clickhouse](https://blog.csdn.net/weixin_41461992/article/details/106790507)

![](https://uploader.shimo.im/f/ym7N2z8RWT0FSW95.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

Kafka Engine表作为数据通道，负责拉取Kafka中的数据。
任意类型的用户表，如MergeTree，作为最终的面向终端用户表。
Materialized物化视图表，负责将数据通道Kafka表同步到终端用户表中。

### File

读取本地文件数据，File表引擎的数据文件只能保存在config.xml配置的路径下。

## 内存类型

### Memory

作为集群间分发数据的存储载体使用。

### Set

### Join

将Join查询进行的封装。

### Buffer

## 日志类型

Merge

如果有分表的情况，查询跨年的场景适用，代理查询多张表，异步执行最终返回一个结果集。

### Distribute

分布式引擎接受参数有：集群名称，数据库的名称，表名称以及分片表达式（路由算法）。例：
Distributed(logs, default, hits[, sharding_key])
参数说明：
logs : 服务器配置文件中的群集名称
default: 库名，也可以使用常量表达式来代替数据库名称，如currentDatabase()
hits:表名
sharding_key：分片表达式（路由算法）

#### 写数据

* 分片数据写入集群方式

有两种方法将数据写入集群

* 指定写入哪些服务器，并直接在每个分片上执行写入。可以使用任何分片方案，对于复杂业务特性的需求，这可能是非常重要的。 这也是最佳解决方案，因为数据可以完全独立地写入不同的分片。但实现主要依赖外部系统，不在ClickHouse本身。
* 在分布式表上执行 INSERT，有本节点负责切分数据，并向其他所有分片节点发送数据。在这种情况下，分布式表会跨服务器分发插入数据。 为了写入分布式表，必须要配置分片键（最后一个参数）。

2. 在第一个分片节点写入本地分片数据（原本在其他分片的数据也暂时在此节点）
3. 想远端分片发送临时保存在本地的数据
4. 远端节点写入完毕
5. 本地节点确认

* 分片表达式

引擎将通过sharding key（分片表达式）算法分散数据落到不同的服务器上。例如，可以使用表达式“rand()”来随机分配数据，或者使用“UserID”来分配剩余的用户ID。如果需要某列均匀分布，可以用一个哈希函数来做sharding key：intHash64（用户名）。
划分的一个简单的余数(rand算法)是分片的有限解决方案，并不总是合适的。它适用于大中型数据（数十台服务器），但不适用于大量数据（数百台服务器或更多）。

* 分片权重
  每个分片都可以在配置文件中定义一个权重来配置数据具体落盘。
  假设有3个分片，权重分别是9，10，11。权重空间分割的计算方式：[prev_weight,pre_weights+weight) … 其中，prev_weights是最左分片权重的总和，weight表示当前shard权重，示例如上。3个shard分割出如下3个权重空间[0,9);[9,19);[19,30]。

现在来了一条待写入的row

2. sharding key表达式先被解析，假定sharding key是rand()函数，函数返回值是43。
3. 函数返回值/shards权重总和=43/(9+10+11) = 13
4. 查找13属于范围[9,19)，属于第二个shard，于是，数据将落到第二个shard上。
5. #### 分片数据同步

分片数据同步方式可在config.xml配置文件中 定义 'internal_replication' 参数 。

* 使用Distributed复制数据

同时负责分片和副本的写入和同步，不会检查副本的一致性，并且随着时间的推移，副本数据可能会有些不一样。可能成为系统的瓶颈。本地表类型不是ReplicatedMergeTree也能做到同步。

* 使用ReplicatedMergeTree同步数据

此参数设置为“ true ”时，写操作只选一个正常的副本写入数据。如果分布式表的子表是复制表(*ReplicaMergeTree)。 数据的复制工作交给实际需要写入数据的表本身而不是分布式表 。

#### 读数据

分布式引擎本身不存储数据, 但可以在多个服务器上进行分布式查询。 读是自动并行的。

> all表的概念可以理解为一个视图。在all表上读数据时CH数据流程如下：
> 1.分发SQL到对应多个shard上执行SQL
> 2.执行SQL后的数据的中间结果发送到主server上
> 3.数据再次汇总过滤。
> [ClickHouse Distribute 引擎深度解读](http://www.clickhouse.com.cn/topic/5a3e768d2141c2917483557e)

![](https://uploader.shimo.im/f/8dqxJYhEZkw6wMxX.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)![](https://uploader.shimo.im/f/oYjVDJLhbbwqqFhS.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

如果，数据在不同的分片，分布式查询又是需要多个分片节点查询完成，比如最后having sum(XXX)>2，此时在本地表汇聚后再UNION出的结果会不正确，需要使用Global优化分布式子查询。

2. 开发问题
3. ### Dictionary

字典数据常驻内存，非常适合保存常量或经常使用的维表数据，避免不必要的JOIN查询。其中ClickHouse内置的GEO字典只提供定义和取数函数，没有现成数据。扩展字典支持7种类型的内存布局和4类数据来源。
7种内存布局，4种数据来源

* Local file 本地文件
* Executable file 可执行文件
* HTTP 远程文件
* DBMS 数据库类型

| 内存布局           | 存储结构         | 字典键类型              | 数据来源                           |
| ------------------ | ---------------- | ----------------------- | ---------------------------------- |
| flat               | 数组             | UInt64                  | Local fileExecutable fileHTTPDBMS  |
| hashed             | 散列             | UInt64                  |                                    |
| range_hashed       | 散列并按时间排序 | UInt64和时间            |                                    |
| complex_key_hashed | 散列             | 复合型key               |                                    |
| ip_trie            | 层次结构         | 复合型key（单个String） |                                    |
| cache              | 固定大小数组     | UInt64                  | Executable fileHTTPClickHouseMySQL |
| complex_key_cache  | 固定大小数组     | 复合型key               |                                    |

## 其他类型

### Live View

通过HTTP/HTTPS协议访问REST服务。

### Materialized

> Materialized views in ClickHouse are implemented more like insert triggers. If there’s some aggregation in the view query, it’s applied only to the batch of freshly inserted data. Any changes to existing data of source table (like update, delete, drop partition, etc.) doesn’t change the materialized view.

当库表有数据插入时，物化视图就像会被触发，将新入的数据插入到物化视图结果表中。在[ClickHouse MATERIALIZED VIEW](https://www.jianshu.com/p/94b3cdab4afd)这篇文章中，时效性在5s内。

物化视图是查询结果集的一份持久化存储，所以它与普通视图完全不同，而非常趋近于表。“查询结果集”的范围很宽泛，可以是基础表中部分数据的一份简单拷贝，也可以是多表join之后产生的结果或其子集，或者原始数据的聚合指标等等。所以，物化视图不会随着基础表的变化而变化，所以它也称为快照（snapshot）。如果要更新数据的话，需要用户手动进行，如周期性执行SQL，或利用触发器等机制。
产生物化视图的过程就叫做“物化”（materialization）。广义地讲，物化视图是数据库中的预计算逻辑+显式缓存，典型的空间换时间思路。所以用得好的话，它可以避免对基础表的频繁查询并复用结果，从而显著提升查询的性能。它当然也可以利用一些表的特性，如索引。
问题一: 每次视图中的数据总数不一致
本地通过insert....remote的语句，模拟主表的插入，就是触发物化视图的计算功能；但是当基础信息表数据更新之后，发现每次聚合的数据条目总数都是不一致

* 通过查看clickhouse的[官方issue](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FClickHouse%2F)

查询时，可以再做聚合，筛选出唯一记录。
问题2：如果物化视图是由两个或者多个表Join生成，那么仅当左表插入数据时才更新，右表插入数据不更新
[ClickHouse物化视图](https://blog.csdn.net/whiteBearClimb/article/details/111284176)

# 数据定义

## 分区操作

### DETACH&ATACH

分区被卸载后，它的物理数据并没有删除，而是转移到了当前数据表目录的detached子目录下。detach分区后，集群对此分区无感知，可以对数据目录做任意处理。

Moves all data for the specified partition to the `detached` directory. The server forgets about the detached data partition as if it does not exist. The server will not know about this data until you make the [ATTACH](https://clickhouse.tech/docs/en/sql-reference/statements/alter/#alter_attach-partition) query.

Example:

```sql
ALTER TABLE visits DETACH PARTITION 201901
```

After the query is executed, you can do whatever you want with the data in the `detached` directory — delete it from the file system, or just leave it.

This query is replicated – it moves the data to the `detached` directory on all replicas. Note that you can execute this query only on a leader replica. To find out if a replica is a leader, perform the `SELECT` query to the [system.replicas](https://clickhouse.tech/docs/en/operations/system-tables/#system_tables-replicas) table. Alternatively, it is easier to make a `DETACH` query on all replicas - all the replicas throw an exception, except the leader replica.

* 问题：在drop表之后，提示not empty directory

> If your table partitioned by date you need to detach partition first and then you're able to remove the data.

> Look at DETACH section of [https://clickhouse.yandex/docs/en/query_language/misc/](https://clickhouse.yandex/docs/en/query_language/misc/)

对无法删除的表先detach。

```sql
DETACH TABLE IF EXISTS egypt_mdm.ft_app_run_info_replica;
```

## 数据类型

### 基础类型

* 数值类型
  * Int
  * Float
  * 定点数Decimal
* 字符串类型
* String

字符串拼接函数：concat(s1, s2, ...)，将参数中的多个字符串拼接，不带分隔符。

同步MySQL表数据到ClickHouse

* FixedString
* UUID
* 时间类型

  * DateTime
  * DateTime64
  * Date

| DateTime   | 2019-01-01 00:00:00     | select toDateTime('2019-01-01 00:00:00') |
| ---------- | ----------------------- | ---------------------------------------- |
| DateTime64 | 2019-01-01 00:00:00.000 |                                          |
| Date       | 2019-01-01              |                                          |

[时间日期函数](https://clickhouse.tech/docs/zh/sql-reference/functions/date-time-functions/#tosecond)

### 复合类型

* Array
* Tuple
* Enum
* Nested

### 特殊类型

* Nullable
* Domain

# 数据查询

## 客户端登录

clickhouse client -u admin --password bd_admin

## WITH

通过WITH表达式支持CTE（Common Table Expression），以增强查询语句的表达，提高语句的可读性和可维护性。

2. FROM
3. SAMPLE
4. ARRAY JOIN

与数组或嵌套类型的字段进行JOIN操作，从而将一行数组展开为多行。

## JOIN

ASOF模糊连接，在连接键之后追加一个模糊连接的匹配条件。

* 应该遵循左大右小原则，JOIN查询会将右表全部加载进内存与左表进行比较。
* 在大量维度属性补全的场景中，使用字典代替JOIN查询。

## WHERE与PREWHERE

首先读取PREWHERE指定的列字段数据，用于数据过滤的条件判断，最后再读取SELECT声明的列字段以补全其余属性。

## GROUP BY

WITH CUBE：基于聚合键之间所有的组合生成小计信息，如果聚合键的个数为n，最终小计组合的个数为2^n。

2. HAVING
3. ORDER BY
4. LIMIT BY
5. LIMIT
6. SELECT
7. DISTINCT
8. UNION ALL
9. 查看SQL执行计划

[如何在ClickHouse中查看SQL执行计划](https://mp.weixin.qq.com/s/cO_cAnXAviHMMnPHga71vw)

```shell
clickhouse-client --send_logs_level=trace <<< 'select * from egypt_mdm.ft_app_used_d_202007_replica' > /dev/null
```

![img](https://uploader.shimo.im/f/SMIJubQ5MKaMnPXD.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

## DDL

新增

```sql
ALTER TABLE ft_stat_d ADD COLUMN container String AFTER market
```

重命名

```sql
RENAME TABLE [db11.]name11 TO [db12.]name12
```

无法重命名字段，需要新建表后再把旧表数据导入，重命名新表。

修改字段类型

```sql
ALTER TABLE dws_grp_detail_day MODIFY COLUMN sum_year Int64
```

```sql
ALTER TABLE center_study_staff_plan_state_replica MODIFY  COLUMN staff_study_status Nullable(Enum8('exception' = 0, 'unread' = 1, 'inread' = 2, 'passed' = 3, 'failed' = 4, 'invalid' = 5, '' = 99))


ALTER TABLE center_study_staff_plan_state_replica MODIFY COLUMN status Nullable(Enum8('exception' = 0, 'unread' = 1, 'inread' = 2, 'passed' = 3, 'failed' = 4, 'invalid' = 5, '' = 99))
```

## DML

[ClickHouse之DBA运维宝典](https://mp.weixin.qq.com/s/ENdKSSmnIzVP2Ve7wb6wlQ)

新增记录（从MySQL）

```sql
INSERT INTO [db].table [(c1, c2, c3)] select * from mysql('host:port', 'db', 'table_name', 'user', 'password')
```

删除

删除字段

```sql
ALTER TABLE ft_grp_resource_stat_d DROP COLUMN market_id
```

修改

修改记录

```sql
ALTER TABLE city UPDATE area='South' WHERE city='wuhan';
```

[无法修改主键的记录](https://stackoverflow.com/questions/55976700/is-it-possible-to-update-primary-key-in-clickhouse-using-mergetree-engine)

查询

* union查询
* 开窗函数

[正宗的ClickHouse开窗函数来袭](https://mp.weixin.qq.com/s/TInLoWYhEkUj08-kUXkRlQ)

* 查看分区数

SELECT * FROM `system`.parts WHERE database IN ('zz_test') AND `table` IN ('ods_yuntu_101ppt_custom_event_day_test_add_month');

* 查看数据表

```sql
select
    database,
    table,
    formatReadableSize(size) as size,
    formatReadableSize(bytes_on_disk) as bytes_on_disk,
    formatReadableSize(data_uncompressed_bytes) as data_uncompressed_bytes,
    formatReadableSize(data_compressed_bytes) as data_compressed_bytes,
    compress_rate,
    rows,
    days,
    formatReadableSize(avgDaySize) as avgDaySize
from
(
    select
        database,
        table,
        sum(bytes) as size,
        sum(rows) as rows,
        min(min_date) as min_date,
        max(max_date) as max_date,
        sum(bytes_on_disk) as bytes_on_disk,
        sum(data_uncompressed_bytes) as data_uncompressed_bytes,
        sum(data_compressed_bytes) as data_compressed_bytes,
        (data_compressed_bytes / data_uncompressed_bytes) * 100 as compress_rate,
        max_date - min_date as days,
        size / (max_date - min_date) as avgDaySize
    from system.parts
    where active 
   and database = 'zz_test'
   and table = 'ods_yuntu_101ppt_custom_event_day_new'
    group by
        database,
        table
)
```

* 查询各个库数据大小

```sql
with sum(data_uncompressed_bytes) as bytes select database, formatReadableSize(bytes) as format from system.columns group by database order by bytes desc;
```

* 当前连接数

众所周知，CH 对外暴露的原生接口分为 TCP 和 HTTP 两类，通过 system.metrics 即可查询当前的 TCP、HTTP 与内部副本的连接数。

```sql
ch7.nauu.com :) SELECT * FROM system.metrics WHERE metric LIKE '%Connection';


SELECT *
FROM system.metrics
WHERE metric LIKE '%Connection'


┌─metric────────────────┬─value─┬─description─────────────────────────────────────────────────────────┐
│ TCPConnection │ 2 │ Number of connections to TCP server (clients with native interface) │
│ HTTPConnection        │     1 │ Number of connections to HTTP server                                │
│ InterserverConnection │ 0 │ Number of connections from other replicas to fetch parts │
└───────────────────────┴───────┴─────────────────────────────────────────────────────────────────────┘
```

* 当前正在执行的查询

通过 system.processes 可以查询目前正在执行的查询，例如：

```sql
ch7.nauu.com :) SELECT query_id, user, address, query FROM system.processes ORDER BY query_id;


SELECT
query_id,
user,
address,
query
FROM system.processes
ORDER BY query_id ASC


┌─query_id─────────────────────────────┬─user────┬─address────────────┬─query─────────────────────────────────────────────────────────────────────────────┐
│ 203f1d0e-944e-472d-8d8f-bae548ff9899 │ default │ ::ffff:10.37.129.4 │ SELECT query_id, user, address, query FROM system.processes ORDER BY query_id ASC │
│ fb7fba85-b2a0-4271-87ff-22da97ae511b │ default │ ::ffff:10.37.129.4 │ INSERT INTO hits_v1 FORMAT TSV │
└──────────────────────────────────────┴─────────┴────────────────────┴───────────────────────────────────────────────────────────────────────────────────┘

```

可以看到，CH 目前正在执行两条语句，其中第 2 条是 INSERT 查询正在写入数据。

* 终止查询

通过 KILL QUERY 语句，可以终止正在执行的查询:

```sql
KILL QUERY WHERE query_id = 'query_id'
```

例如，终止刚才的 INSERT 查询 :

```sql
ch7.nauu.com :) KILL QUERY WHERE query_id='ff695827-dbf5-45ad-9858-a853946ea140';


KILL QUERY WHERE query_id = 'ff695827-dbf5-45ad-9858-a853946ea140' ASYNC


Ok.


0 rows in set. Elapsed: 0.024 sec.
```

众所周知，除了常规的 SELECT 和 INSERT 之外，在 ClickHouse 中还存在一类被称作 Mutation 的操作，也就是 ALTER DELETE 和 ALTER UPDATE。

对于 Mutation 操作， ClickHouse 专门提供了 system.mutations 用于查询，例如：

```sql
ch7.nauu.com :) SELECT database, table, mutation_id, command, create_time, is_done FROM system.mutations;


SELECT
database,
table,
mutation_id,
command,
create_time,
is_done
FROM system.mutations


┌─database─┬─table──────┬─mutation_id────┬─command──────────────────┬─────────create_time─┬─is_done─┐
│ default  │ testcol_v9 │ mutation_2.txt │ DELETE WHERE ID = 'A003' │ 2020-06-29 01:15:04 │       1 │
└──────────┴────────────┴────────────────┴──────────────────────────┴─────────────────────┴─────────┘


1 rows in set. Elapsed: 0.002 sec.
```

同样的，可以使用 KILL MUTATION 终止正在执行的 Mutation 操作:

```sql
KILL MUTATION WHERE mutation_id = 'mutation_id';
```

* 存储空间统计

查询 CH 各个存储路径的空间:

```sql
SELECT name,path,formatReadableSize(free_space) AS free,formatReadableSize(total_space) AS total,formatReadableSize(keep_free_space) AS reserved FROM system.disks;


┌─name──────┬─path──────────────┬─free──────┬─total─────┬─reserved─┐
│ default │ /chbase/data/ │ 36.35 GiB │ 49.09 GiB │ 0.00 B │
│ disk_cold │ /chbase/cloddata/ │ 35.35 GiB │ 48.09 GiB │ 1.00 GiB │
│ disk_hot1 │ /chbase/data/ │ 36.35 GiB │ 49.09 GiB │ 0.00 B │
│ disk_hot2 │ /chbase/hotdata1/ │ 36.35 GiB │ 49.09 GiB │ 0.00 B │
└───────────┴───────────────────┴───────────┴───────────┴──────────┘


4 rows in set. Elapsed: 0.001 sec.
```

* 各数据库占用空间统计

```sql
SELECT database, formatReadableSize(sum(bytes_on_disk)) on_disk FROM system.parts GROUP BY database;


┌─database─┬─on_disk──┐
│ system │ 1.59 MiB │
│ default │ 3.60 GiB │
└──────────┴──────────┘
```

* 个列字段占用空间统计

每个列字段的压缩大小、压缩比率以及该列的每行数据大小的占比

```sql
SELECT
database,
table,
column,
any(type),
sum(column_data_compressed_bytes) AS compressed,
sum(column_data_uncompressed_bytes) AS uncompressed,
round(uncompressed / compressed, 2) AS ratio,
compressed / sum(rows) AS bpr,
sum(rows)
FROM system.parts_columns
WHERE active AND database != 'system'
GROUP BY
database,
table,
column
ORDER BY
database ASC,
table ASC,
column ASC


┌─database─┬─table────────┬─column─────────────────────┬─any(type)──────────────────────────────┬─compressed─┬─uncompressed─┬──ratio─┬───────────────────bpr─┬─sum(rows)─┐
│ default │ hits_v1 │ AdvEngineID │ UInt8 │ 351534 │ 26621706 │ 75.73 │ 0.013204788603705563 │ 26621706 │
│ default │ hits_v1 │ Age │ UInt8 │ 7543552 │ 26621706 │ 3.53 │ 0.2833609536518809 │ 26621706 │
│ default │ hits_v1 │ BrowserCountry │ FixedString(2) │ 6549379 │ 53243412 │ 8.13 │ 0.24601650247358303 │ 26621706 │
│ default │ hits_v1 │ BrowserLanguage │ FixedString(2) │ 2819085 │ 53243412 │ 18.89 │ 0.10589422781545255 │ 26621706 │
│ default │ hits_v1 │ CLID │ UInt32 │ 2311006 │ 106486824 │ 46.08 │ 0.08680908729140048 │ 26621706 │
│ default │ hits_v1 │ ClientEventTime │ DateTime │ 98518704 │ 106486824 │ 1.08 │ 3.7006908573026838 │ 26621706 │
│ default │ hits_v1 │ ClientIP │ UInt32 │ 25120766 │ 106486824 │ 4.24 │ 0.9436196913901761 │ 26621706 │
│ default │ hits_v1 │ ClientIP6 │ FixedString(16) │ 25088558 │ 425947296 │ 16.98 │ 0.9424098515699934 │ 26621706 │
│ default │ hits_v1 │ ClientTimeZone │ Int16 │ 8487148 │ 53243412 │ 6.27 │ 0.3188055641512982 │ 26621706 │
│ default │ hits_v1 │ CodeVersion │ UInt32 │ 11976952 │ 106486824 │ 8.89 │ 0.4498942329240658 │ 26621706 │
│ default │ hits_v1 │ ConnectTiming │ Int32 │ 27937373 │ 106486824 │ 3.81 │ 1.0494208372671534 │ 26621706 │
│ default │ hits_v1 │ CookieEnable │ UInt8 │ 202718 │ 26621706 │ 131.32 │ 0.007614763681936838 │ 26621706 │
│ default │ hits_v1 │ CounterClass │ Int8 │ 425492 │ 26621706 │ 62.57 │ 0.015982897564866805 │ 26621706 │
...
```

* 慢查询

```sql
SELECT
user,
client_hostname AS host,
client_name AS client,
formatDateTime(query_start_time, '%T') AS started,
query_duration_ms / 1000 AS sec,
round(memory_usage / 1048576) AS MEM_MB,
result_rows AS RES_CNT,
result_bytes / 1048576 AS RES_MB,
read_rows AS R_CNT,
round(read_bytes / 1048576) AS R_MB,
written_rows AS W_CNT,
round(written_bytes / 1048576) AS W_MB,
query
FROM system.query_log
WHERE type = 2
ORDER BY query_duration_ms DESC
LIMIT 10


┌─user────┬─host─────────┬─client────────────┬─started──┬────sec─┬─MEM_MB─┬──RES_CNT─┬────────────────RES_MB─┬────R_CNT─┬─R_MB─┬───W_CNT─┬─W_MB─┬─query───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ default │ ch7.nauu.com │ ClickHouse client │ 01:05:03 │ 51.434 │   1031 │  8873898 │      8706.51146697998 │        0 │    0 │ 8873898 │ 8707 │ INSERT INTO hits_v1 FORMAT TSV                                                                                                                                          │
│ default │ ch7.nauu.com │ ClickHouse client │ 01:01:48 │ 43.511 │   1031 │  8873898 │      8706.51146697998 │        0 │    0 │ 8873898 │ 8707 │ INSERT INTO hits_v1 FORMAT TSV                                                                                                                                          │
│ default │ ch7.nauu.com │ ClickHouse client │ 17:12:04 │ 11.12 │ 1801 │ 18874398 │ 446.8216323852539 │ 6291466 │ 351 │ 0 │ 0 │ SELECT id, arrayJoin(arrayConcat(groupArray(a), groupArray(b), groupArray(c))) AS v FROM test_y GROUP BY id ORDER BY v ASC │
│ default │ ch7.nauu.com │ ClickHouse client │ 17:13:28 │ 3.992 │ 1549 │ 18874398 │ 446.8216323852539 │ 6291466 │ 351 │ 0 │ 0 │ SELECT id, arrayJoin(arrayConcat(groupArray(a), groupArray(b), groupArray(c))) AS v FROM test_y GROUP BY id │
│ default │ ch7.nauu.com │ ClickHouse client │ 17:13:12 │ 3.976 │ 1549 │ 18874398 │ 446.8216323852539 │ 6291466 │ 351 │ 0 │ 0 │ SELECT id, arrayJoin(arrayConcat(groupArray(a), groupArray(b), groupArray(c))) AS v FROM test_y GROUP BY id │
│ default │ ch7.nauu.com │ ClickHouse client │ 01:25:39 │ 3.962 │ 1549 │ 18874398 │ 446.8216323852539 │ 6291466 │ 351 │ 0 │ 0 │ SELECT id, arrayJoin(arrayConcat(groupArray(a), groupArray(b), groupArray(c))) AS v FROM test_y GROUP BY id │
│ default │ ch7.nauu.com │ ClickHouse client │ 04:32:29 │ 3.114 │ 1542 │ 10000000 │ 219.82192993164062 │ 10500000 │ 231 │ 0 │ 0 │ SELECT user_id, argMax(score, create_time) AS score, argMax(deleted, create_time) AS deleted, max(create_time) AS ctime FROM test_a GROUP BY user_id HAVING deleted = 0 │
│ default │ ch7.nauu.com │ ClickHouse client │ 02:59:56 │ 3.03 │ 1544 │ 10000000 │ 219.75380992889404 │ 10500000 │ 231 │ 0 │ 0 │ SELECT user_id, argMax(score, create_time) AS score, argMax(is_update, create_time) AS is_update, max(create_time) AS ctime FROM test_a GROUP BY user_id │
│ default │ ch7.nauu.com │ ClickHouse client │ 02:54:01 │ 3.019 │ 1543 │ 10000000 │ 219.3450927734375 │ 10500000 │ 230 │ 0 │ 0 │ SELECT user_id, argMax(score, create_time) AS score, argMax(delete, create_time) AS delete, max(create_time) AS ctime FROM test_a GROUP BY user_id │
│ default │              │                   │ 03:03:12 │  2.857 │   1543 │       10 │ 0.0002269744873046875 │ 10500000 │  231 │       0 │    0 │ SELECT * FROM view_test_a limit 10                                                                                                 │
└─────────┴──────────────┴───────────────────┴──────────┴────────┴────────┴──────────┴───────────────────────┴──────────┴──────┴─────────┴──────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘


10 rows in set. Elapsed: 0.017 sec. Processed 1.44 thousand rows, 200.81 KB (83.78 thousand rows/s., 11.68 MB/s.)
```

* 副本预警监控

通过下面的 SQL 语句对副本进行预警监控，其中各个预警的变量可以根据自身情况调整。

```sql
SELECT database, table, is_leader, total_replicas, active_replicas
FROM system.replicas
WHERE is_readonly
OR is_session_expired
OR future_parts > 30
OR parts_to_check > 20
OR queue_size > 30
OR inserts_in_queue > 20
OR log_max_index - log_pointer > 20
OR total_replicas < 2
    OR active_replicas < total_replicas


┌─database─┬─table───────────────────────┬─is_leader─┬─total_replicas─┬─active_replicas─┐
│ default │ replicated_sales_12 │ 0 │ 0 │ 0 │
│ default │ test_fetch │ 0 │ 0 │ 0 │
│ default │ test_sharding_simple2_local │ 0 │ 0 │ 0 │
└──────────┴─────────────────────────────┴───────────┴────────────────┴─────────────────┘
```

## HTTP interface

```shell
echo 'SELECT 1' | curl 'http://n1.bigdata.kafka.egypt.sdp:8123/?user=mdm&password=mdm' -d @-
```

```shell
echo 'SELECT create_table_query FROM `system`.tables' | curl 'http://localhost:8123/?user=admin&password=bd_admin' -d @- > result.log
```

```shell
echo 'SELECT create_table_query FROM `system`.tables' | curl 'http://localhost:8123' -d @- > result.log
```

### sql形式

导出

```shell
echo 'select * from table_name' | curl ip:8123?database=database_name -uuser:password -d @- > table_name.sql
```

导入

```shell
cat ft_app_run_info.sql | clickhouse-client -u mdm --password mdm --query="INSERT INTO egypt_mdm_v12.ft_app_run_info_old FORMAT TabSeparated"
```

### csv形式

导出

clickhouse-client -h 127.0.0.1 --database="db" --query="select * from db_name.table_name FORMAT CSV" > table_name.csv

导入

clickhouse-client -h 127.0.0.1 --database="db" --query="insert into db_name.table_name FORMAT CSV" < ./table_name.csv

## 函数

### [argMax](https://clickhouse.tech/docs/en/sql-reference/aggregate-functions/reference/argmax/)

在使用分布式集群表的时候，常常会使用ReplacingMergeTree表引擎，但是后台在去重时是不定时执行的去重，会出现短暂数据不一致的情况。在对一致性非常敏感的场景，有什么办法可以确保取到最新数据呢？

一种可以使用Final触发合并分区动作，但是这个操作很重。20.5.2.7版本后，使用Final性能有所提升。

一种是使用anyLast函数，通过groupby之后取最新值。但性能较差。

还有一种可以使用argMax()函数，按照field2的最大值取field1的值，可以保存相同主键的多条记录，再根据某字段取最新值，有点像hbase的版本号概念。

[ClickHouse准实时数据更新的新思路](https://mp.weixin.qq.com/s/a8OfsBn9VFnj7oxp0IIVGg)

SELECT argMax(status,tuple(status, binlog_create_time)) FROM duoxue.center_study_plan_replica WHERE id = 79274;

根据status和binlog_create_time共同作为排序依据。

### ngramSimHash

[巧用ClickHouse快速判断两个集合的相似度](https://mp.weixin.qq.com/s/nMkf4pdFB19FADU9F5TJRA)

在业务中我们经常会遇到查重的需求，例如给定一个文本字符串，判断在已有的文档中，是否存在与其相似的。想要实现这类功能的方式有很多种，一种高效的方式是先利用SinHash 将数据降维压缩成一串哈希值，再利用海明距离(Hamming Distance) 来比较两者之间的相似度。SinHash 是一种局部敏感性哈希算法，特别适合在海量数据下的场景使用。在ClickHouse 中现在已经内置了 MinHash 和 海明距离的相关函数，使用 bitHammingDistance 函数计算哈希值之间的差异距离可以判断文本的相似度。

### 时间或日期截取函数

| 函数           | 用途                                             | 举例                                                                                 | 结果                           |
| -------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------ | ------------------------------ |
| toYear()       | 取日期或时间日期的年份                           | toYear(toDateTime(‘2018-12-11 11:12:13’)) toYear(toDate(‘2018-12-11’))           | 返回 2018 返回 2018            |
| toMonth()      | 取日期或时间日期的月份                           | toMonth(toDateTime(‘2018-12-11 11:12:13’)) toMonth(toDate(‘2018-12-11’))         | 返回 12返回 12                 |
| toDayOfMonth() | 取日期或时间日期的天（1-31）                     | toMonth(toDayOfMonth(‘2018-12-11 11:12:13’)) toMonth(toDayOfMonth(‘2018-12-11’)) | 返回 11返回 11                 |
| toDayOfWeek()  | 取日期或时间日期的星期（星期一为1，星期日为7）。 | toDayOfWeek(toDateTime(‘2018-12-11 11:12:13’)) toDayOfWeek(toDate(‘2018-12-11’)) | 返回 2返回 2                   |
| toHour()       | 取时间日期的小时                                 | toHour(toDateTime(‘2018-12-11 11:12:13’))                                          | 返回 11                        |
| toMinute()     | 取时间日期的分钟                                 | toMinute(toDateTime(‘2018-12-11 11:12:13’))                                        | 返回 12                        |
| toSecond()     | 取时间日期的秒                                   | toSecond(toDateTime(‘2018-12-11 11:12:13’))                                        | 返回 13                        |
| toMonday()     | 取时间日期最近的周一（返回日期）                 | toMonday(toDate(‘2018-12-11’)) toMonday(toDateTime(‘2018-12-11 11:12:13’))       | 返回 2018-12-10返回 2018-12-10 |
| toTime()       | 将时间日期的日期固定到某一天，保留原始时间       | toTime(toDateTime(‘2018-12-11 11:12:13’))                                          | 返回 1970-01-02 11:12:13       |

# 运维

## 部署安装

从官网[https://packagecloud.io/Altinity/clickhouse/](https://packagecloud.io/Altinity/clickhouse/)下载如下几个文件

```
wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-server-common-19.16.12.49-1.el7.x86_64.rpm/download.rpm


wget --content-disposition https://packagecloud.io/Altinity/clickhouse/packages/el/7/clickhouse-common-static-19.16.12.49-1.el7.x86_64.rpm/download.rpm


wget --content-disposition https://packagecloud.io/Altinity/cli
7.x86_64.rpm/download.rpm
```

### yum安装

```
rpm -ivh clickhouse-server-common-19.16.12.49-1.el7.x86_64.rpm
rpm -ivh clickhouse-common-static-19.16.12.49-1.el7.x86_64.rpm
rpm -ivh clickhouse-server-19.16.12.49-1.el7.x86_64.rpm
rpm -ivh clickhouse-client-19.16.12.49-1.el7.x86_64.rpm
或者执行以下一个命令即可
yum -y localinstall clickhouse-*.rpm
```

安装过程遇到glibc升级：[https://cloud.tencent.com/developer/article/1463094](https://cloud.tencent.com/developer/article/1463094)

遇到无libicu问题，下载如下，并执行

```
rpm -ivh libicu-50.2-3.el7.x86_64.rpm
```

[ libicu-50.2-3.el7.x86_64.rpm ](https://uploader.shimo.im/f/7vCevgL2iF4qRyFf.rpm)

通过clickhouse client进入cli。

### tgz安装

[https://github.com/ClickHouse/ClickHouse/releases/tag/v20.8.12.2-lts](https://github.com/ClickHouse/ClickHouse/releases/tag/v20.8.12.2-lts)
[https://repo.clickhouse.tech/tgz/lts/](https://repo.clickhouse.tech/tgz/lts/)
curl -O https://repo.clickhouse.tech/tgz/clickhouse-common-static-20.8.12.2.tgz
curl -O https://repo.clickhouse.tech/tgz/clickhouse-common-static-dbg-20.8.12.2.tgz
curl -O https://repo.clickhouse.tech/tgz/clickhouse-server-20.8.12.2.tgz
curl -O https://repo.clickhouse.tech/tgz/clickhouse-client-20.8.12.2.tgz
tar -xzvf clickhouse-common-static-20.8.12.2.tgz
tar -xzvf clickhouse-common-static-dbg-20.8.12.2.tgz
tar -xzvf clickhouse-server-20.8.12.2.tgz
tar -xzvf clickhouse-client-20.8.12.2.tgz
sudo clickhouse-common-static-20.8.12.2/install/doinst.sh
sudo clickhouse-common-static-dbg-20.8.12.2/install/doinst.sh
sudo clickhouse-server-20.8.12.2/install/doinst.sh
sudo clickhouse-client-20.8.12.2/install/doinst.sh
sudo clickhouse-common-static-21.3.2.5/install/doinst.sh
sudo clickhouse-common-static-dbg-21.3.2.5/install/doinst.sh
sudo clickhouse-server-21.3.2.5/install/doinst.sh
sudo clickhouse-client-21.3.2.5/install/doinst.sh
systemctl restart clickhouse-server
systemctl status clickhouse-server

## config.xml配置

```shell
<?xml version="1.0"?>
<yandex>
    <logger>
        <level>information</level>
        <log>/var/log/clickhouse-server/clickhouse-server.log</log>
        <errorlog>/var/log/clickhouse-server/clickhouse-server.err.log</errorlog>
        <size>1000M</size>
        <count>10</count>
    </logger>
    <http_port>8123</http_port>
    <tcp_port>9000</tcp_port>
    <openSSL>
        <server> 
            <certificateFile>/etc/clickhouse-server/server.crt</certificateFile>
            <privateKeyFile>/etc/clickhouse-server/server.key</privateKeyFile>
            <dhParamsFile>/etc/clickhouse-server/dhparam.pem</dhParamsFile>
            <verificationMode>none</verificationMode>
            <loadDefaultCAFile>true</loadDefaultCAFile>
            <cacheSessions>true</cacheSessions>
            <disableProtocols>sslv2,sslv3</disableProtocols>
            <preferServerCiphers>true</preferServerCiphers>
        </server>
        <client> 
            <loadDefaultCAFile>true</loadDefaultCAFile>
            <cacheSessions>true</cacheSessions>
            <disableProtocols>sslv2,sslv3</disableProtocols>
            <preferServerCiphers>true</preferServerCiphers>
            <invalidCertificateHandler>
                <name>RejectCertificateHandler</name>
            </invalidCertificateHandler>
        </client>
    </openSSL>
    <interserver_http_port>9009</interserver_http_port>
    <listen_host>::</listen_host>
    <max_connections>4096</max_connections>
    <keep_alive_timeout>3</keep_alive_timeout>
    <max_concurrent_queries>150</max_concurrent_queries>
    <uncompressed_cache_size>8589934592</uncompressed_cache_size>
    <mark_cache_size>5368709120</mark_cache_size>
    <path>/data/clickhouse/</path>
    <tmp_path>/data/clickhouse/tmp/</tmp_path>
    <user_files_path>/var/lib/clickhouse/user_files/</user_files_path>
    <users_config>users.xml</users_config>
    <default_profile>default</default_profile>
    <default_database>default</default_database>
   <remote_servers incl="clickhouse_remote_servers">
        <ck_1shards_2replicas>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>localhost</host>
                    <port>9000</port>
                </replica>
            </shard>
        </ck_1shards_2replicas>
    </remote_servers>
    <zookeeper>
        <node index="1">
            <host>localhost</host>
            <port>2181</port>
        </node>
    </zookeeper>
    <macros>
        <shard>01</shard>
        <replica>01</replica>
    </macros>
    <zookeeper incl="zookeeper-servers" optional="true" />
    <macros incl="macros" optional="true" />
    <builtin_dictionaries_reload_interval>3600</builtin_dictionaries_reload_interval>
    <max_session_timeout>3600</max_session_timeout>
    <default_session_timeout>60</default_session_timeout>
    <query_log>
        <database>system</database>
        <table>query_log</table>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </query_log>
    <dictionaries_config>*_dictionary.xml</dictionaries_config>
    <compression incl="clickhouse_compression">
    </compression>
    <distributed_ddl>
        <path>/clickhouse/task_queue/ddl</path>
    </distributed_ddl>
    <merge_tree>
        <replicated_deduplication_window_seconds>1</replicated_deduplication_window_seconds>
        <replicated_deduplication_window>10</replicated_deduplication_window>
    </merge_tree>
    <graphite_rollup_example>
        <pattern>
            <regexp>click_cost</regexp>
            <function>any</function>
            <retention>
                <age>0</age>
                <precision>3600</precision>
            </retention>
            <retention>
                <age>86400</age>
                <precision>60</precision>
            </retention>
        </pattern>
        <default>
            <function>max</function>
            <retention>
                <age>0</age>
                <precision>60</precision>
            </retention>
            <retention>
                <age>3600</age>
                <precision>300</precision>
            </retention>
            <retention>
                <age>86400</age>
                <precision>3600</precision>
            </retention>
        </default>
    </graphite_rollup_example>
    <format_schema_path>/var/lib/clickhouse/format_schemas/</format_schema_path>
</yandex>

```

## user.xml配置

```shell
<?xml version="1.0"?>
<yandex>
    <profiles>
        <default>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <max_partitions_per_insert_block>1000</max_partitions_per_insert_block>
            <load_balancing>random</load_balancing>
            <log_queries>1</log_queries>
        </default>


        <readonly>
            <readonly>1</readonly>
        </readonly>
    </profiles>


    <users>
        <admin>
            <password>bd_admin</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </admin>
        <duoxue>
            <password>duoxue_test</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
            <allow_databases>
                <database>duoxue</database>
            </allow_databases>
        </duoxue>
        <mdm>
            <password>mdm_test</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
            <allow_databases>
                <database>egypt_mdm_v12</database>
            </allow_databases>
        </mdm>
    </users>


    <quotas>
        <default>
            <interval>
                <duration>3600</duration>
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>
</yandex>

```

## 版本升级

3. 查看数据地址 `<path>`/data/clickhouse/`</path>`，备份数据与配置文件
4. 卸载删除安装文件

```
yum list installed | grep clickhouse
yum remove -y clickhouse-common-static
yum remove -y clickhouse-server-common
rm -rf /var/lib/clickhouse
rm -rf /etc/clickhouse-*
rm -rf /var/log/clickhouse-server
```

2. 按6.1部署安装步骤重新安装
3. 还原配置文件
4. 重启即可
5. 服务监控

[clickhouse监控--grafana安装](https://shimo.im/docs/xdgP3rh9CTGhVTCq)

[Prometheus+Grafana+ClickHouse](https://shimo.im/docs/3Dw3GWk3Y8vgdxdd)

[ clickhouse-performance-monitor_rev3.json ](https://uploader.shimo.im/f/FhIemZN0d6mvVN9o.json)

## 可视化工具

[ dbeaver-ce-6.2.0-win32.win32.x86_64.zip ](https://uploader.shimo.im/f/AED6EKYKWuQmXgIx.zip)

服务需要将    <listen_host>::</listen_host>  的注释打开，开放端口。

[ClickHouse可视化管理工具ckman](https://mp.weixin.qq.com/s/v_-3zOHabt1DETvuruWbfg)

## 系统表

memory_usage 单位为bytes

merges表：正在合并的信息，如果有依赖操作的顺序，可以先查询上一步是否操作完成

mutation表：update/delete操作对分区的操作进度

ClickHouse Mutation：[https://livedig.com/986](https://livedig.com/986)

[https://clickhouse.tech/docs/en/sql_reference/statements/misc/#kill-mutation](https://clickhouse.tech/docs/en/sql_reference/statements/misc/#kill-mutation)

Updates and Deletes in ClickHouse：[https://www.altinity.com/blog/2018/10/16/updates-in-clickhouse](https://www.altinity.com/blog/2018/10/16/updates-in-clickhouse)

mutation执行完会留下记录，is_done=0表示未完成。

replicas表：记录分区详情，如是否是leader partition

query_log表：包含一次查询的所用信息。

## 问题记录

* 集群本地表数据未同步

查看创建的本地表语句，是否在zookeeper中带有空格。

![](https://uploader.shimo.im/f/mUKNwDGq00dTLC5W.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

如果在zk中一致，再尝试一下，把所有节点表全删除后，确认data和metadata路径下没有数据后，再重新建表。

* 创建本地表和分布式表失败

Code: 1000. DB::Exception: Received from localhost:9000. DB::Exception: I/O error: 19.。

A：未知原因，重装ck、删除ck数据和znode都不起作用。最终通过克隆别的没问题的虚拟机加入集群解决。

* 删除表失败

Can't drop readonly replicated table (need to drop data in ZooKeeper as well).'

[How to force dropping readonly replicated table #13836](https://github.com/ClickHouse/ClickHouse/issues/13836)

先detach然后删除数据库目录

sql: "detach table hits_replica"

rm -rf /var/lib/clickhouse/data/tutorial/hits_replica ;

rm -rf /var/lib/clickhouse/metadata/tutorial/hits_replica.sql

* 升级后启动失败

> `<Error>` Application: DB::Exception: More than one field of 'password', 'password_sha256_hex', 'password_double_sha1_hex' is used to specify password for user default. Must be only one of them.

[Conflicting user configuration files created during update #8316](https://github.com/ClickHouse/ClickHouse/issues/8316)

Deleted "users.d" folder with conflicting settings that was created after an update.

## 日志维护

```shell
<logger>
    <level>trace</level>  --日志记录级别。可接受的值： trace, debug, information, warning, error
    <log>/var/log/clickhouse-server/clickhouse-server.log</log> --日志文件，根据级别包含所有条目
    <errorlog>/var/log/clickhouse-server/clickhouse-server.err.log</errorlog> -- 错误日志文件
    <size>1000M</size> -- 文件的大小。适用于loganderrorlog，文件达到大小后，ClickHouse将对其进行存档并重命名，并在其位置创建一个新的日志文件
    <count>10</count>  --  ClickHouse存储的已归档日志文件的数量
</logger>
```

## 数据备份与还原

```shell
clickhouse-client --query="SELECT * FROM egypt_mdm.ft_device_used_info_replica" > /data/device_used.tsv
cat /data/device_used.tsv | clickhouse-client --query "insert into egypt_mdm.ft_device_used_info_replica format TSV"
```

## 权限用户配置

[拒绝裸奔--ClickHouse用户名密码设置](https://www.jianshu.com/p/40d8e00cb1b5)

```shell
<?xml version="1.0"?>
<yandex>
    <profiles>
        <default>
            <max_memory_usage>10000000000</max_memory_usage>
            <use_uncompressed_cache>0</use_uncompressed_cache>
            <max_partitions_per_insert_block>1000</max_partitions_per_insert_block>
            <load_balancing>random</load_balancing>
            <log_queries>1</log_queries>
        </default>


        <readonly>
            <readonly>1</readonly>
        </readonly>
    </profiles>


    <users>
        <admin>
            <password>bd_admin</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
        </admin>
        <duoxue>
            <password>duoxue_dev</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
            <allow_databases>
                <database>duoxue</database>
            </allow_databases>
        </duoxue>
        <mdm>
            <password>mdm_dev</password>
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>
            <profile>default</profile>
            <quota>default</quota>
            <allow_databases>
                <database>egypt_mdm_v12</database>
            </allow_databases>
        </mdm>
    </users>


    <quotas>
        <default>
            <interval>
                <duration>3600</duration>
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>
</yandex>
```

## 熔断机制

单次insert限制创建的最大分区个数，默认值为100，在user.xml中进行配置

![](https://uploader.shimo.im/f/57eOE9N2xMyFCpSn.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

    <max_partitions_per_insert_block>1000</max_partitions_per_insert_block>

并发插入数据过于频繁，设置合并分区时延，在config.xml中配置

```shell
<merge_tree>
<parts_to_delay_insert>300</parts_to_delay_insert>
<parts_to_throw_insert>600</parts_to_throw_insert>
<max_delay_to_insert>2</max_delay_to_insert>
</merge_tree>
```

# 最佳实践

## 最佳实践配置

![](https://uploader.shimo.im/f/scprMLY5zHIWGHmB.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

最佳实践配置

图来源：[ClickHouse万亿数据双中心的设计与实践 ](https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup24/4.%20ClickHouse%E4%B8%87%E4%BA%BF%E6%95%B0%E6%8D%AE%E5%8F%8C%E4%B8%AD%E5%BF%83%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5%20.pdf)

## 客户端

### JDBC驱动

* [官方JDBC 的驱动](https://github.com/ClickHouse/clickhouse-jdbc)集群配置：使用BalancedClickhouseDataSource类，通过解析url多个地址，随机获取某个server获取connection，如果server宕机，无法实现故障转移。
* [ClickHouse-Native-JDBC](https://github.com/housepower/ClickHouse-Native-JDBC)
* [clickhouse4j](https://github.com/blynkkk/clickhouse4j)沿用官方驱动，使用BalancedClickhouseDataSource类

### ChProxy

* [chproxy](https://github.com/Vertamedia/chproxy)官网 通过http或者https的协议对clickhouse集群做负载均衡操作
* 根据用户路由集群。May proxy requests to multiple distinct `ClickHouse` clusters depending on the input user. For instance, requests from `appserver` user may go to `stats-raw` cluster, while requests from `reportserver` user may go to `stats-aggregate` cluster.
* May map input users to per-cluster users. This prevents from exposing real usernames and passwords used in `ClickHouse` clusters. Additionally this allows mapping multiple distinct input users to a single `ClickHouse` user.
* May accept incoming requests via HTTP and HTTPS.
* 提供IP权限访问。May limit HTTP and HTTPS access by IP/IP-mask lists.
* May limit per-user access by IP/IP-mask lists.
* 用户降级访问。May limit per-user query duration. Timed out or canceled queries are forcibly killed via [KILL QUERY](http://clickhouse-docs.readthedocs.io/en/latest/query_language/queries.html#kill-query).
* May limit per-user requests rate.
* May limit per-user number of concurrent requests.
* All the limits may be independently set for each input user and for each per-cluster user.
* May delay request execution until it fits per-user limits.
* Per-user [response caching](https://github.com/Vertamedia/chproxy#caching) may be configured.
* Response caches have built-in protection against [thundering herd](https://en.wikipedia.org/wiki/Cache_stampede) problem aka `dogpile effect`.
* Evenly spreads requests among replicas and nodes using `least loaded` + `round robin` technique.
* 监控节点，选择健康路由。Monitors node health and prevents from sending requests to unhealthy nodes.
* Supports automatic HTTPS certificate issuing and renewal via [Let’s Encrypt](https://letsencrypt.org/).
* May proxy requests to each configured cluster via either HTTP or [HTTPS](https://github.com/yandex/ClickHouse/blob/96d1ab89da451911eb54eccf1017eb5f94068a34/dbms/src/Server/config.xml#L15).
* Prepends User-Agent request header with remote/local address and in/out usernames before proxying it to `ClickHouse`, so this info may be queried from [system.query_log.http_user_agent](https://github.com/yandex/ClickHouse/issues/847).
* Exposes various useful [metrics](https://github.com/Vertamedia/chproxy#metrics) in [prometheus text format](https://prometheus.io/docs/instrumenting/exposition_formats/).
* Configuration may be updated without restart - just send `SIGHUP` signal to `chproxy` process.
* Easy to manage and run - just pass config file path to a single `chproxy` binary.
* Easy to [configure](https://github.com/Vertamedia/chproxy/blob/master/config/examples/simple.yml):

 目前只支持insert和select操作。Yes, we successfully use it in production for both `INSERT` and `SELECT` requests.

* [chproxy部署](https://shimo.im/docs/wPYpJWxkK9VP9rJ9)

```shell
nohup /opt/module/clickhouse/chproxy/chproxy -config=/opt/module/clickhouse/chproxy/config/config.yml  >> /opt/module/clickhouse/chproxy/logs/chproxy.out 2>&1 &
/opt/module/clickhouse/chproxy/restart.sh
```

## 最佳架构

![](https://uploader.shimo.im/f/mVHZYaqMc1YiD5LD.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

最佳架构图

图来源：[MySQL DBA解锁数据分析的新姿势-ClickHouse-新浪-高鹏-2017年12月08日](http://www.clickhouse.com.cn/topic/5a6c27509d28dfde2ddc5e76)

A、B、C是数据的3个分片，各自承担1/3的查询负载，查询性能A+B+C。3个IDC使用复制机制互备，只要还有1台不宕机，就可以提供服务。

![](https://uploader.shimo.im/f/at9p1pwhHHkp15zr.png!thumbnail?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2ODAzMzU1MzgsImZpbGVHVUlEIjoiUlR5eENRY3JWZFI2cnl5SiIsImlhdCI6MTY4MDMzNTIzOCwiaXNzIjoidXBsb2FkZXJfYWNjZXNzX3Jlc291cmNlIiwidXNlcklkIjoxMzUwNzkzOX0.rjqy0lE-J_Jo3YLTZqHeCKkiwcOgeAaVUoCEuchLCtw)

本方案采用1分片3副本，通过chproxy对集群进行负载均衡查询、插入。使用的SLB是ClickHouse自带的BalancedClickhouseDataSource。ClickHouse自身没有对失效的server进行剔除，但是提供了一个定时方法定时清除宕机server。

[ClickHouse 在有赞的实践之路](https://mp.weixin.qq.com/s/ctYvyGYCKZYqWD8ieuHNew)

# 生产实践

![1679913292743](image/Clickhouse/Clickhouse生产架构.png)

### Chproxy

由于各种原因，ClickHouse 的最大执行时间、最大并发语句可能会超过max_execution_time 和max_concurrent_queries 的限制 :
max_execution_time 可能会因为当前实现的缺陷而被超过
max_concurrent_queries 只针对每个节点的限制。如果是在集群节点上，是没法限制集群整体的并发查询数量。
这种“泄漏”的限制可能会导致所有集群节点的高资源使用率。遇到这个问题后，我们不得不在我们的 ClickHouse 集群前维护 2 个不同的 http 代理 -- 一个用于在集群节点间分散INSERT 操作，另一个用于发送 SELECT 到一个专用节点，在该节点再通过某种方式进行限制。这样很不健壮，管理起来也很不方便，所以我们开发了 Chproxy。 : )

chproxy是针对clickhouse集群的一种代理，它可以提供clickhouse的权限控制，支持通过http和https的协议方式与集群进行交互，详见[https://github.com/Vertamedia/chproxy](https://github.com/Vertamedia/chproxy)
