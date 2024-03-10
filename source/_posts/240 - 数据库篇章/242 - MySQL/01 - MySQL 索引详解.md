---
title: MySQL 索引详解
date: 2024-03-10
categories:
  - 数据库篇章
tags:
  - 数据库
  - MySQL
published: true
---
## 第4讲 深入浅出索引

### 常见索引模型
索引的作用：提高数据查询效率
下边介绍几种数据结构：
- **哈希表**
   存放 键-值 （key - value）对。
1. 哈希思路：把值放在数组里，用一个哈希函数把key换算成一个确定的位置，然后把value放在数组的这个位置
2. 哈希冲突的处理办法：链表
3. 哈希表适用场景：只有等值查询的场景，查询范围会很慢

- **有序数组**
  按顺序存储。查询用二分法就可以快速查询，时间复杂度是：O(log(N))
1. 有序数组查询效率高，更新效率低
2. 有序数组的适用场景：静态存储引擎。

- 二叉搜索树
每个节点的左儿子小于父节点，父节点又小于右儿子。查询时间复杂度O(log(N))，更新时间复杂度O(log(N))
数据库存储大多不适用二叉树，因为树高过高，会适用N叉树

> InnoDB中的索引模型：B+Tree（具体介绍看《数据结构与算法笔记》）

### InnoDB的索引模型
在InnoDB中，表都是根据主键顺序以索引的形式存放的，这种存储方式的表称为索引组织表。又因为前面我们提到的，InnoDB使用了B+树索引模型，所以数据都是存储在B+树中的。
每一个索引在InnoDB里面对应一棵B+树。
假设，我们有一个主键列为ID的表，表中有字段k，并且在k上有索引。
这个表的建表语句是：
```sql
mysql> create table T(
id int primary key, 
k int not null, 
name varchar(16),
index (k))engine=InnoDB;
```

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/703A4416-B9D0-4DCE-B917-6DB94A2FAD09.png)

### 索引类型
- 主键索引、非主键索引
主键索引的叶子节点存的是整行的数据(聚簇索引)，非主键索引的叶子节点内容是主键的值(二级索引)

- 主键索引和普通索引的区别
1. 如果语句是`select * from T where ID=500`，主键索引只要搜索ID这个B+Tree即可拿到数据。
2. 如果语句是`select * from T where k=5`，即普通索引查询方式，普通索引先搜索索引拿到主键值，再到主键索引树搜索一次(回表)
> 也就是说，基于非主键索引的查询需要多扫描一棵索引树。因此，我们在应用中应该尽量使用主键查询。

### 索引的设计原则
- 索引并非越多越好。
- 避免对经常更新的表进行过多的索引，并且索引中的列尽可能少。
- 数据量小的表最好不要使用索引。
- 在条件表达式中经常用到的不同值较多的列上建立检索，在不同值少的列上不要建立索引。
- 当唯一性是某种数据本身的特征时，指定唯一索引。
- 在频繁进行排序或分组（即进行group by或order by操作）的列上建立索引

#### 索引维护
为了维护索引结构，比如B+树，在新数据插入的时候树节点会发生变动，发生数据页变动。
一个数据页满了，按照B+Tree算法，新增加一个数据页，叫做页分裂，会导致性能下降。空间利用率降低大概50%。当相邻的两个数据页利用率很低的时候会做数据页合并，合并的过程是分裂过程的逆过程。

自增主键在插入数据时，一般都是追加操作，不涉及挪动其他记录，也不会触发叶子节点的分裂。
> 从性能和存储空间方面考量，自增主键往往是更合理的选择。

从存储空间上看，假设你的表中确实有一个唯一字段，比如字符串类型的身份证号，那应该用身份证号做主键，还是用自增字段做主键呢？
由于每个非主键索引的叶子节点上都是主键的值。如果用身份证号做主键，那么每个二级索引的叶子节点占用约20个字节，而如果用整型做主键，则只要4个字节，如果是长整型（bigint）则是8个字节。

显然，**主键长度越小，普通索引的叶子节点就越小，普通索引占用的空间也就越小。**

> 所以，从性能和存储空间方面考量，自增主键往往都是更合理的选择。

### 索引的分类
1. 普通索引和唯一索引
2. 单列索引和组合索引
3. 全文索引 （MyISAM引擎）
4. 空间索引（NOT NULL）MyISAM引擎

### 创建表时创建索引
```sql
CREATE  TABLE  
     table_name [col_name data_type]   
     [UNIQUE|FULLTEXT|SPATIAL]  [INDEX|KEY]     
     [index_name]  (col_name [length])
     [ASC | DESC]
```

#### 普通索引
```sql
    CREATE TABLE book
    (
    bookid            	INT NOT NULL,
    bookname          	VARCHAR(255) NOT NULL,
    authors            	VARCHAR(255) NOT NULL,
    info               	VARCHAR(255) NULL,
    comment           	VARCHAR(255) NULL,
    year_publication   	YEAR NOT NULL,
    INDEX(year_publication)
    );
```
没有指定索引名称，则默认使用列名：
```sql
mysql> show create table book\G;
*************************** 1. row ***************************
       Table: book
Create Table: CREATE TABLE `book` (
  `bookid` int(11) NOT NULL,
  `bookname` varchar(255) NOT NULL,
  `authors` varchar(255) NOT NULL,
  `info` varchar(255) DEFAULT NULL,
  `comment` varchar(255) DEFAULT NULL,
  `year_publication` year(4) NOT NULL,
  KEY `year_publication` (`year_publication`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```
#### 唯一索引
```sql
    CREATE Table: CREATE TABLE `t1` (
      `id` int(11) NOT NULL,
      `name` char(30) NOT NULL,
      UNIQUE KEY `UniqIdx` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
```
- 查看表信息
```sql
     SHOW CREATE table t1 \G
    *************************** 1. row ***************************
           Table: t1
    CREATE Table: CREATE TABLE `t1` (
      `id` int(11) NOT NULL,
      `name` char(30) NOT NULL,
      UNIQUE KEY `UniqIdx` (`id`)  //标识索引
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    1 row in set (0.00 sec)
```

#### 单列索引
```sql
    CREATE TABLE t2
    (
    id   INT NOT NULL,
    name CHAR(50) NULL,
    INDEX SingleIdx(name(20))
    );
```

#### 组合索引
组合索引可起几个索引的作用，但是使用时并不是随便查询哪个字段都可以使用索引，而是遵从“最左前缀”：利用索引中最左边的列集来匹配行，这样的列集称为最左前缀。例如这里由id、name和age 3个字段构成的索引，索引行中按id/name/age的顺序存放，索引可以搜索下面字段组合：（id, name, age）、（id, name）或者id。如果列不构成索引最左面的前缀，MySQL不能使用局部索引，如（age）或者（name,age）组合则不能使用索引查询。
```sql
    CREATE TABLE t3
    (
    id    INT NOT NULL,
    name CHAR(30) 　NOT NULL,
    age  INT NOT　 NULL,
    info VARCHAR(255),
    INDEX MultiIdx(id, name, age(100))
    );
```

在t3表中，查询id和name字段，使用EXPLAIN语句查看索引的使用情况：
```sql
     explain select * from t3 where id=1 AND name='joe' \G
    *************************** 1. row ***************************
               id: 1
      select_type: SIMPLE
            table: t3
             type: ref
    possible_keys: MultiIdx   //索引类型
              key: MultiIdx
          key_len: 94
              ref: const,const
             rows: 1
            Extra: Using where
    1 row in set (0.00 sec)
```
> 可以看到，查询id和name字段时，使用了名称MultiIdx的索引

如果查询（name,age）组合或者单独查询name和age字段，结果如下：
```sql
explain select * from t3 where name='joe'  AND age = 5 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t3
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
        Extra: Using where
```
> 此时，possible_keys和key值为NULL，并没有使用在t3表中创建的索引进行查询。

#### 全文索引 

```sql
    CREATE TABLE t4
    (
    id    INT NOT NULL,
    name CHAR(30) NOT NULL,
    age  INT NOT NULL,
    info VARCHAR(255),
    FULLTEXT INDEX FullTxtIdx(info)
    ) ENGINE=MyISAM;
```
> 因为MySQL5.6中默认存储引擎为InnoDB，在这里创建表时需要修改表的存储引擎为MyISAM，不然创建索引会出错。
> 在MySQL5.7上默认使用InnoDB创建全文索引成功。

语句执行完毕之后，使用SHOW CREATE TABLE查看表结构：
```sql
     SHOW CREATE table t4 \G
    *************************** 1. row ***************************
           Table: t4
    CREATE Table: CREATE TABLE `t4` (
      `id` int(11) NOT NULL,
      `name` char(30) NOT NULL,
      `age` int(11) NOT NULL,
      `info` varchar(255) DEFAULT NULL,
      FULLTEXT KEY `FullTxtIdx` (`info`)
    ) ENGINE=MyISAM DEFAULT CHARSET=utf8
```
> 由结果可以看到，info字段上已经成功建立了一个名为FullTxtIdx的FULLTEXT索引。全文索引非常适合于大型数据集，对于小的数据集，它的用处可能比较小。

#### 空间索引 
空间索引必须在MyISAM类型的表中创建，且空间类型的字段必须为非空。

```sql
    CREATE TABLE t5( 
        g GEOMETRY NOT NULL, 
        SPATIAL  INDEX spatIdx(g) 
    )ENGINE=MyISAM;
```


show index from book \G;

### 在已经存在的表上创建索引

#### 使用ALTER 创建索引
```sql
ALTER TABLE book ADD INDEX BkNameIdx ( bookname(30) ); //添加普通索引
ALTER TABLE book ADD UNIQUE INDEX UniqidIdx ( bookId ); //建立唯一索引
ALTER TABLE book ADD INDEX BkCommontIdx(comment(50)); //建立单列索引
ALTER TABLE book ADD INDEX BkAuAndInfoIdx(authors(20), info(50)); //建立组合索引
ALTER TABLE t6 ADD FULLTEXT INDEX infoFTIdx ( info ); //建立全文索引
```

#### 使用CREATE INDEX创建索引
```sql
CREATE INDEX BkNameIdx ON book(bookname); //普通索引
CREATE UNIQUE INDEX UniqidIdx  ON book ( bookId ); //唯一索引
CREATE INDEX BkcmtIdx ON book(comment(50) );  //单列索引
CREATE INDEX BkAuAndInfoIdx ON book ( authors(20),info(50) ); //组合索引
CREATE FULLTEXT INDEX ON t6(info); //全文索引
```

### 删除索引
1. 使用ALTER TABLE删除索引
```sql
ALTER TABLE table_name DROP INDEX index_name;
```

2. 使用DROP INDEX语句删除索引
```sql
DROP INDEX index_name ON table_name;
```

### 索引优化
**索引示例：**

```bash
mysql> create table T (
ID int primary key,
k int NOT NULL DEFAULT 0, 
s varchar(16) NOT NULL DEFAULT '',
index k(k))
engine=InnoDB;

insert into T values(100,1, 'aa'),(200,2,'bb'),(300,3,'cc'),(500,5,'ee'),(600,6,'ff'),(700,7,'gg');
```

**根据以上的建表语句，会产生主键索引和非主键索引，索引结构如下:**
![](https://i.loli.net/2019/08/26/rzQ5eTlf3Ps9Npd.png)
下边我们执行`select * from T where k between 3 and 5;`语句的执行步骤如下：

1. 在k索引树上找到k=3的记录，取得 ID = 300；
2. 再到ID索引树查到ID=300对应的R3；
3. 在k索引树取下一个值k=5，取得ID=500；
4. 再回到ID索引树查到ID=500对应的R4；
5. 在k索引树取下一个值k=6，不满足条件，循环结束。

在这个过程中，回到主键索引树搜索的过程，我们称为回表。可以看到，这个查询过程读了k索引树的3条记录（步骤1、3和5），回表了两次（步骤2和4）。

在这个例子中，由于查询结果所需要的数据只在主键索引上有，所以不得不回表。那么，有没有可能经过索引优化，避免回表过程呢？

#### 覆盖索引
如上如果执行select ID from T where k between 3 and 5语句，ID的值已经在k索引树上了，因此可以直接提供查询结果，不需要回表。也就是说，在这个查询中索引k已经“覆盖了”我们查询需求，称为覆盖索引。

覆盖索引是根据索引字段的结构，为了尽量减少查询次数，提高查询效率。设计一个合适的查询语句。

#### 最左前缀原则 
 前缀索引是符合最左前缀原则，在建立联合索引时，安排好索引内字段的顺序，通过合理安排索引顺序，可以少维护一个或多个索引，这种设计方案应该优先考虑。
**B+树这种索引结构，可以利用索引的“最左前缀”，来定位记录。**

如下是使用(name,age)创建的联合索引：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/CDD382A1-54CC-412B-8F4A-FC033E2F2F80.png)
当你的逻辑需求是查到所有名字是“张三”的人时，可以快速定位到ID4，然后向后遍历得到所有需要的结果。

如果你要查的是所有名字第一个字是“张”的人，你的SQL语句的条件是"where name like ‘张%’"。这时，你也能够用上这个索引，查找到第一个符合条件的记录是ID3，然后向后遍历，直到不满足条件为止。
这里我们的评估标准是，索引的复用能力。因为可以支持最左前缀，所以当已经有了(a,b)这个联合索引后，一般就不需要单独在a上建立索引了。因此，第一原则是，如果通过调整顺序，可以少维护一个索引，那么这个顺序往往就是需要优先考虑采用的。

#### 索引下推

MySQL 5.6引入索引下推优化，可以在索引遍历中，对索引中包含的字段先做判断，直接过滤掉不满足条件记录，减少回表次数。执行如下语句示例：

```bash
mysql> select * from tuser where name like '张%' and age=10 and ismale=1;
```

- **无索引下推流程：**

![](https://i.loli.net/2019/08/26/km8esS5J7tuCIWl.png)
无索引下推，在搜索时不考虑第二个索引字段，导致多次回表。

- **索引下推执行流程**：

![](https://i.loli.net/2019/08/26/3jgyaxPBskwOib6.png)

通过索引下推，可以少了多次回表次数


## 第9讲 普通索引和唯一索引怎么选择
唯一索引和普通索引的区别：
唯一索引保证了数据列不包含重复的值，如果创建的是唯一索引，在插入数据是，索引列的数据相同，则会插入失败。创建唯一索引往往不是为了提升访问速度，而是为了避免数据重复。

那么在考虑性能时，普通索引和唯一索引应该怎么选择呢。

![055433c4882707c6e7a949328f1bc9bc.png](evernotecid://B723A9F1-46B3-45CC-8F83-76830D257C8D/appyinxiangcom/19257560/ENResource/p279)
下边分析如上图中的索引情况，假设字段 k 上的值都不重复。

#### 查询过程

假设，执行查询的语句是 select id from T where k=5。这个查询语句在索引树上查找的过程，先是通过B+树从树根开始，按层搜索到叶子节点，也就是图中右下角的这个数据页，然后可以认为数据页内部通过二分法来定位记录。

* 对于普通索引来说，查找到满足条件的第一个记录(5,500)后，需要查找下一个记录，直到碰到第一个不满足k=5条件的记录。
* 对于唯一索引来说，由于索引定义了唯一性，查找到第一个满足条件的记录后，就会停止继续检索。

以上查询过程中不同索引之间造成的性能差异可以忽略不计。
因为对于InnoDB，数据页在内存中读，普通索引就多了一次查找下一个元素，性能影响微乎其微。

#### 更新过程
在说更新过程前，先说一个change buffer的概念。
当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InooDB会将这些更新操作缓存在change buffer中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行change buffer中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。
将change buffer中的操作应用到原数据页，得到最新结果的过程称为merge。除了访问这个数据页会触发merge外，系统有后台线程会定期merge。在数据库正常关闭（shutdown）的过程中，也会执行merge操作。
根据以上的说法，change buffer的使用可以加快更新数据的速度。
但对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束。比如，要插入(4,400)这个记录，就要先判断现在表中是否已经存在k=4的记录，而这必须要将数据页读入内存才能判断。如果都已经读入到内存了，那直接更新内存会更快，就没必要使用change buffer了。

因此，唯一索引的更新就不能使用change buffer，实际上也只有普通索引可以使用。
change buffer用的是buffer pool里的内存，因此不能无限增大。change buffer的大小，可以通过参数innodb_change_buffer_max_size来动态设置。这个参数设置为50的时候，表示change buffer的大小最多只能占用buffer pool的50%。

那么我们再一起来看看如果要在这张表中插入一个新记录(4,400)的话，InnoDB的处理流程是怎样的。

第一种情况是，这个记录要更新的目标页在内存中。这时，InnoDB的处理流程如下：

* 对于唯一索引来说，找到3和5之间的位置，判断到没有冲突，插入这个值，语句执行结束；
* 对于普通索引来说，找到3和5之间的位置，插入这个值，语句执行结束。

这样看来，普通索引和唯一索引对更新语句性能影响的差别，只是一个判断，只会耗费微小的CPU时间。
但，这不是我们关注的重点。
第二种情况是，这个记录要更新的目标页不在内存中。这时，InnoDB的处理流程如下：

* 对于唯一索引来说，需要将数据页读入内存，判断到没有冲突，插入这个值，语句执行结束；
* 对于普通索引来说，则是将更新记录在change buffer，语句执行就结束了。

将数据从磁盘读入内存涉及随机IO的访问，是数据库里面成本最高的操作之一。change buffer因为减少了随机磁盘访问，所以对更新性能的提升是会很明显的。
> 所以在使用唯一索引时，为了保证唯一索引列数据的唯一性，在更新数据时需要判断唯一性，这样无法使用change buffer，相对来说影响了性能。

> 关于索引的选择，更新数据频繁，但不急着查询时，可以通过change buffer加速，但是如果每次更新后立即需要查询，这时候不适合开启change buffer，因为更新后，立即查询又要出发change buffer的merge操作，反而影响查询的性能。不过对于大部分情况，还是建议使用普通索引配合change buffer使用，这样对数据更新的性能还是有明显提升的。

#### change buffer的使用场景
change buffer是在更新操作时起到了加速作用，并且只适用于在普通索引场景，不适用于唯一索引。
change buffer在merge的时候是真正进行数据更新的时刻，而change buffer的主要目的就是将记录的变更动作缓存下来，所以在一个数据页做merge之前，change buffer记录的变更越多（也就是这个页面上要更新的次数越多），收益就越大。
因此，对于写多读少的业务来说，页面在写完以后马上被访问到的概率比较小，此时change buffer的使用效果最好。这种业务模型常见的就是账单类、日志类的系统。

反过来，假设一个业务的更新模式是写入之后马上会做查询，那么即使满足了条件，将更新先记录在change buffer，但之后由于马上要访问这个数据页，会立即触发merge过程。这样随机访问IO的次数不会减少，反而增加了change buffer的维护代价。所以，对于这种业务模式来说，change buffer反而起到了副作用。

#### 普通索引和唯一索引选择
两类索引在查询能力上没有差别，主要考虑的是对更新性能的影响，一般建议尽量选择普通索引。
如果所有的更新后面，都马上伴随着对这个记录的查询，那么你应该关闭change buffer。而在其他情况下，change buffer都能提升更新性能。

在实际使用中，你会发现，普通索引和change buffer的配合使用，对于数据量大的表的更新优化还是很明显的。

#### change buffer 和 redo log

redo log 主要节省的是随机写磁盘的IO消耗（转成顺序写），而change buffer主要节省的则是随机读磁盘的IO消耗。

## 第10讲 MySQL为什么会选错索引
在表中可能存在多个索引，如下：
```sql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `b` (`b`)
) ENGINE=InnoDB；
```
通过执行存储过程来插入10万条数据：
```sql
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
```
通过执行如下语句
```sql
mysql> explain select * from t where (a between 1 and 1000) and (b between 50000 and 100000) order by b limit 1;
```

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/ED3EB71A-7ABD-4B4D-A59D-E97ADD14F2BA.png)
会发现选择的索引是b，但是我们分析一下的话其实如果使用a索引查询时更快的
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/0723CDE2-8504-4DA2-BC54-B671B84317A2.png)
可以看到，原本语句需要执行2.23秒，而当你使用force index(a)的时候，只用了0.05秒，比优化器的选择快了40多倍。
通过我们指定使用索引a查询，发现确实会快很多。
但为什么MySQL会默认选择了索引b呢。这是因为在MySQL的索引选择逻辑中，会综合判断那种查询方式最优，但其实判断的最终结果不一定是最好的。

为了避免以上的情况，有如下几个处理思路：
1. 一种方法是，像我们第一个例子一样，采用force index强行选择一个索引。（这种不推荐，应为如果在语句中指定了，就相当于不使用MySQL的自动选择，在以后对数据库有改动，这些语句也需要改动，而且这种方式也不一定在所有引擎中都支持）
2. 第二种方法就是，我们可以考虑修改语句，引导MySQL使用我们期望的索引。通过修改成如下的查找方式，发现MySQL就选择了a索引
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/89C98C1F-6750-4E83-886A-0DAF62305A3B.png)
之前优化器选择使用索引b，是因为它认为使用索引b可以避免排序（b本身是索引，已经是有序的了，如果选择索引b的话，不需要再做排序，只需要遍历），所以即使扫描行数多，也判定为代价更小。
现在order by b,a 这种写法，要求按照b,a排序，就意味着使用这两个索引都需要排序。因此，扫描行数成了影响决策的主要条件，于是此时优化器选了只需要扫描1000行的索引a。
3. 第三种方法是，在有些场景下，我们可以新建一个更合适的索引，来提供给优化器做选择，或删掉误用的索引。（这个需要考虑业务，出现索引选择错误，可能表索引的设计有问题）

- 此节内容参考《MySQL实战45讲》9、10讲


## 第11讲 关于给字符串字段建索引
#### 给字符串字段加前缀索引
如下表定义：
```
mysql> create table SUser(
ID bigint unsigned primary key,
email varchar(64), 
... 
)engine=innodb; 
```
由于要使用邮箱登录，所以业务代码中一定会出现类似于这样的语句：
```
mysql> select f1, f2 from SUser where email='xxx';
```
如果以上表中没有加索引，则只能进行全表扫描。
如果创建索引，比如以下两个方式，看看有什么区别呢：
```
mysql> alter table SUser add index index1(email);
或
mysql> alter table SUser add index index2(email(6));
```
第一个语句创建的index1索引里面，包含了每个记录的整个字符串；而第二个语句创建的index2索引里面，对于每个记录都是只取前6个字节。
那么，这两种不同的定义在数据结构和存储上有什么区别呢？如图2和3所示，就是这两个索引的示意图。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/AE7309BC-1958-4057-9712-32124C92DA50.png)
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/5BCD3E71-7A0E-4A5E-8985-C864E26E2F1E.png)
从图中你可以看到，由于email(6)这个索引结构中每个邮箱字段都只取前6个字节（即：zhangs），所以占用的空间会更小，这就是使用前缀索引的优势。
虽然第二种方式使用的空间更小，但是也可能有其他损失，如果执行如下查询：
```
select id,name,email from SUser where email='zhangssxyz@xxx.com';
```
- 如果使用的是index1（即email整个字符串的索引结构），执行顺序是这样的：

1. 从index1索引树找到满足索引值是’zhangssxyz@xxx.com’的这条记录，取得ID2的值；
2. 到主键上查到主键值是ID2的行，判断email的值是正确的，将这行记录加入结果集；
3. 取index1索引树上刚刚查到的位置的下一条记录，发现已经不满足email='zhangssxyz@xxx.com’的条件了，循环结束。
> 这个过程中，只需要回主键索引取一次数据，所以系统认为只扫描了一行。

- 如果使用的是index2（即email(6)索引结构），执行顺序是这样的：
1. 从index2索引树找到满足索引值是’zhangs’的记录，找到的第一个是ID1；
2. 到主键上查到主键值是ID1的行，判断出email的值不是’zhangssxyz@xxx.com’，这行记录丢弃；
3. 取index2上刚刚查到的位置的下一条记录，发现仍然是’zhangs’，取出ID2，再到ID索引上取整行然后判断，这次值对了，将这行记录加入结果集；
4. 重复上一步，直到在idxe2上取到的值不是’zhangs’时，循环结束。
> 在这个过程中，要回主键索引取4次数据，也就是扫描了4行。

根据以上的不同情况，对于前缀索引，确定好长度是关键，如果定义好长度，就可以做到既节省空间，有不用额外增加太多的查询成本。

#### 怎么确定前缀索引长度
实际上，我们在建立索引时关注的是区分度，区分度越高越好。因为区分度越高，意味着重复的键值越少。因此，我们可以通过统计索引上有多少个不同的值来判断要使用多长的前缀。

首先，你可以使用下面这个语句，算出这个列上有多少个不同的值：
```sql
mysql> select count(distinct email) as L from SUser;
```
然后，依次选取不同长度的前缀来看这个值，比如我们要看一下4~7个字节的前缀索引，可以用这个语句：
```sql
mysql> select 
  count(distinct left(email,4)）as L4,
  count(distinct left(email,5)）as L5,
  count(distinct left(email,6)）as L6,
  count(distinct left(email,7)）as L7,
from SUser;
```
当然，使用前缀索引很可能会损失区分度，所以你需要预先设定一个可以接受的损失比例，比如5%。然后，在返回的L4~L7中，找出不小于 L * 95%的值，假设这里L6、L7都满足，你就可以选择前缀长度为6。

#### 前缀索引对覆盖索引的影响
前面我们说了使用前缀索引可能会增加扫描行数，这会影响到性能。其实，前缀索引的影响不止如此，我们再看一下另外一个场景。
你先来看看这个SQL语句：
```sql
select id,email from SUser where email='zhangssxyz@xxx.com';
```
与前面例子中的SQL语句
```sql
select id,name,email from SUser where email='zhangssxyz@xxx.com';
```
相比，这个语句只要求返回id和email字段。

所以，如果使用index1（即email整个字符串的索引结构）的话，可以利用覆盖索引，从index1查到结果后直接就返回了，不需要回到ID索引再去查一次。而如果使用index2（即email(6)索引结构）的话，就不得不回到ID索引再去判断email字段的值。

即使你将index2的定义修改为email(18)的前缀索引，这时候虽然index2已经包含了所有的信息，但InnoDB还是要回到id索引再查一下，因为系统并不确定前缀索引的定义是否截断了完整信息。

也就是说，使用前缀索引就用不上覆盖索引对查询性能的优化了，这也是你在选择是否使用前缀索引时需要考虑的一个因素。

#### 其他方式

1. 第一种方式是使用倒序存储：如果把字符串倒过来可以由更好的区分度的话，可以考虑把字符串倒过来，然后再建立前缀索引
> 比如：身份证号对于一个省份的人来说，前面的几位都是一样的，不一样的基本都在后几位，这样可以倒过来
```sql
mysql> select field_list from t where id_card = reverse('input_id_card_string');
```
2. 第二种方式是使用hash字段。
你可以在表上再创建一个整数字段，来保存身份证的校验码，同时在这个字段上创建索引。
```
mysql> alter table t add id_card_crc int unsigned, add index(id_card_crc);
```
然后每次插入新记录的时候，都同时用crc32()这个函数得到校验码填到这个新字段。由于校验码可能存在冲突，也就是说两个不同的身份证号通过crc32()函数得到的结果可能是相同的，所以你的查询语句where部分要判断id_card的值是否精确相同。
```sql
mysql> select field_list from t where id_card_crc=crc32('input_id_card_string') and id_card='input_id_card_string'
```
这样，索引的长度变成了4个字节，比原来小了很多。

#### 总结
下边我们总结一下给字符串创建索引的几种方式和比较：

1. 直接创建完整索引，这样可能比较占用空间；
2. 创建前缀索引，节省空间，但会增加查询扫描次数，并且不能使用覆盖索引；
3. 倒序存储，再创建前缀索引，用于绕过字符串本身前缀的区分度不够的问题；
4. 创建hash字段索引，查询性能稳定，有额外的存储和计算消耗，跟第三种方式一样，都不支持范围扫描。

## 索引问题小结
#### 问题1
- **问题描述**

如下重建索引k和重建主键索引：
```sql
alter table T drop index k;
alter table T add index(k);
```
```bash
alter table T drop primary key;
alter table T add primary key(id);
```
以上两个重建索引的方式是否有问题。

- **问题解答**

重建索引k的做法是合理的，可以达到省空间的目的。但是，重建主键的过程不合理。不论是删除主键还是创建主键，都会将整个表重建。所以连着执行这两个语句的话，第一个语句就白做了。这两个语句，你可以用这个语句代替 ： alter table T engine=InnoDB。

#### 问题2

- **问题描述**
为什么要删除重建索引？

- **问题解答**
索引可能因为删除，或者页分裂等原因，导致数据页有空洞，重建索引的过程会创建一个新的索引，把数据按顺序插入，这样页面的利用率最高，也就是索引更紧凑、更省空间。

#### 问题3
- **问题描述**

实际上主键索引也是可以使用多个字段的。DBA小吕在入职新公司的时候，就发现自己接手维护的库里面，有这么一个表，表结构定义类似这样的：
```sql
CREATE TABLE `geek` (
  `a` int(11) NOT NULL,
  `b` int(11) NOT NULL,
  `c` int(11) NOT NULL,
  `d` int(11) NOT NULL,
  PRIMARY KEY (`a`,`b`),
  KEY `c` (`c`),
  KEY `ca` (`c`,`a`),
  KEY `cb` (`c`,`b`)
) ENGINE=InnoDB;
```
公司的同事告诉他说，由于历史原因，这个表需要a、b做联合主键，这个小吕理解了。
但是，学过本章内容的小吕又纳闷了，既然主键包含了a、b这两个字段，那意味着单独在字段c上创建一个索引，就已经包含了三个字段了呀，为什么要创建“ca”“cb”这两个索引？
同事告诉他，是因为他们的业务里面有这样的两种语句：
```sql
select * from geek where c=N order by a limit 1;
select * from geek where c=N order by b limit 1;
```
我给你的问题是，这位同事的解释对吗，为了这两个查询模式，这两个索引是否都是必须的？为什么呢？

- **问题解答**

表记录

```
–a--|–b--|–c--|–d--
1 2 3 d
1 3 2 d
1 4 3 d
2 1 3 d
2 2 2 d
2 3 4 d
```

主键 a，b的聚簇索引组织顺序相当于 order by a,b ，也就是先按a排序，再按b排序，c无序。

索引 ca 的组织是先按c排序，再按a排序，同时记录主键

```
–c--|–a--|–主键部分b-- （注意，这里不是ab，而是只有b）
2 1 3
2 2 2
3 1 2
3 1 4
3 2 1
4 2 3
```

这个跟索引c的数据是一模一样的。

索引 cb 的组织是先按c排序，在按b排序，同时记录主键

```
–c--|–b--|–主键部分a-- （同上）
2 2 2
2 3 1
3 1 2
3 2 1
3 4 1
4 3 2
```

所以，结论是ca可以去掉，cb需要保留。
#### 问题4
- 问题描述

如果某次写入使用了change buffer机制，之后主机异常重启，是否会丢失change buffer和数据。

- 答案

不会。虽然是只更新内存，但是在事务提交的时候，我们把change buffer的操作也记录到redo log里了，所以崩溃恢复的时候，change buffer也能找回来。



