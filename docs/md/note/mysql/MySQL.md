# MySQL

## 语句

**show variables like 'character%'**;

![image-20201025161517787](./images/image-20201025161517787.png)

**show engines**;

![image-20201025162725134](./images/image-20201025162725134.png)

**show create table student \G;**查看建表语句

```sql
Create Table: CREATE TABLE `student` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `number` int DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `insert_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `sex` tinyint DEFAULT NULL,
  `is_visible` tinyint DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

```sql
 CREATE TABLE `class` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `number` int DEFAULT NULL,
  `classname` varchar(20) DEFAULT NULL,
  `insert_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `is_visible` tinyint DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

**show index in student;**查看索引

![image-20201025191016247](./images/image-20201025191016247.png)

**duplicate key**语句一般应用在格式化多条更新语句，使用**insert into student(id,number,name,sex) values(10,1234567,'sss',0) on duplicate key update number=1234567;**一定要有唯一索引才能实现

**replace into**如果表中存在会先删除存在的记录，然后插入新的数据，否咋直接插入新的数据。



## 知识点

- MyISAM与InnoDB的差别

|        |   MyISAM   |                 InnoDB                 |
| :----: | :--------: | :------------------------------------: |
| 主外键 |   不支持   |                  支持                  |
|  事务  |   不支持   |                  支持                  |
| 行表锁 |    表锁    |                  行锁                  |
|  缓存  | 只缓存索引 | 不仅缓存索引而且缓存数据，对内存要求高 |
| 表空间 |     小     |                   大                   |
| 关注点 |    性能    |                  事务                  |

- 一条SQL语句的执行顺序：

**from->on->where->group by->having->select->order by->limit**

Left join:以左边为准，保留左边的全部

right join:以右边为准，保留右边的全部

inner join:以共有的为准，保留二者的共有部分

outer join:mysql不支持全外连接，左连接是左外连接，右连接是右外连接

- 添加**is_visible**字段，一方面为了数据浏览的完整性，另一方面为了索引
- 频繁删改的数据不适合建立索引

## 索引

- 单值索引，一个索引只包含单个列，一个表可以有多个单列索引，建议一张表索引不要超过5个

- 唯一索引，索引列的值必须唯一，但允许有空值，主键是一种唯一索引，主键自动建立唯一索引

- 复合索引，一个索引包含多个列

  ### 什么情况下建立索引

  1. 主键自动建立唯一索引
  2. 频繁作为查询条件的字段应该创建索引
  3. 查询中与其他表关联的字段
  4. 高并发倾向建立组合索引
  5. 查询中排序、组合、统计

  ### 什么情况下不建立索引

  1. 表记录太少，mysql官方5到8百万，但是不建议这么大
  2. 经常增删改
  3. 数据字段分布平均而且重复，比如性别、国籍，一般建议选择性（不重复值/所有）大的字段

  ### Explain

  ![image-20201025195509483](./images/image-20201025195509483.png)

  Explain+sql语句

  - 表的读取顺序
  - 数据读取操作的操作类型
  - 哪些索引可以使用
  - 哪些索引被实际使用
  - 表之间的引用
  - 每张表有多少行被优化器查询

- Id相同的情况下table列的顺序就是加载顺序；id不同时，id越大，优先级越高，越先被执行

![image-20201025195819981](./images/image-20201025195819981.png)

![image-20201025200254201](./images/image-20201025200254201.png)

- select_type
  - Simple简单查询
  - Primary最外层的查询
  - Subquery子查询
  - Derived衍生表，临时得
  - Union结果组装
  - Union Result两个查询合并

- type
  - All全表扫描，最差
  - index只遍历索引树
  - range只检索给定范围的行，使用一个索引来选择行，between，>，<等
  - ref非唯一索引扫描，根据索引查出来一部分，可能有多行，查找与扫描的混合体，一般优化到此
  - eq_ref唯一性索引扫描，根据索引查出来只有一条记录与之匹配,常见于主键或唯一索引扫描
  - const通过索引一次就找到了
  - system记录只有一行，const的特例，最好
- possible_keys,key
  - possible_keys索引可能被用到，但是不一定被查询实际使用
  - key实际使用

- Key_len
  - 表示索引使用的字节数，最大长度，并非实际使用长度

- ref
- rows
  - 实际扫描了多少行
- filtered
- Extras

## 表设计相关指南

- int(1)和int(11)在计算和存储上有什么区别？

  无区别，这只是展示长度，比如设为int(11)存为12，前边会有9个0，但是平时看不到，其实是对应的设置没开。

- big int与decimal有什么不同？

  Big int定长小数时，空间小，性能好；deciaml不定长的高精度运算

- varchar与char的区别
  varchar可变长度，支持到65535B，用在字符串最大长度比平均长度大很多，易产生碎片，char用于存储定长数据，支持到255B，比如MD5值，身份证号码会好一些，不易产生碎片。
- 分库分表
- 缓存数据异构，做缓存表，汇总表
  比如连锁店查询多门店订单数据，散表后无法进行聚合查询，使用in导致性能下降，order by查询速度慢

## 索引设计相关指南

![image-20201031231247148](./images/image-20201031231247148.png)

![image-20210618161706711](./images/image-20210618161706711.png)

平衡m树

- 只存关键字，增大业内数据量提升预读有效性，减少磁盘IO，（预读-》局部性原理，加载数据时加载前后。）

- 极大减少树的度，减少磁盘IO，单页16KB，两int的联合索引8B，4层可存80亿数据。

  16KB/8B=2000,2000X2000X2000=80亿

- 叶子节点直接绑定数据，减少磁盘IO

- 有序，叶子节点间双向链表，减少磁盘IO

- 聚簇索引叶子节点保存完整数据，如果定义了主键，主键就是聚簇索引，如果没有第一个费控唯一列作为聚簇索引，否则使用隐藏的

- 辅助索引叶子节点存聚簇索引的值，先通过普通索引查主键，再由主键查数据

  ## 主键设计规范

- 要有主键，且是顺序增长的，强烈建议使用int/bigint自增id作为主键

- 联合索引最左前缀匹配，一直向右匹配遇到范围查询

- 区分度高的尽量前置，范围查询的尽量后置，不要追求全覆盖，定位到千以内就好

- 不要让索引列的默认值为null

- Update/delete操作的where子句必须命中索引，否则相当于锁表

- 索引数量不要超过5个，长度不要超过3个字段

  ![image-20201031234018612](./images/image-20201031234018612.png)

  ## 

