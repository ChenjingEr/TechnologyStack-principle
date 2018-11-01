## mysql索引(二):索引
>创建索引是为了提高查询速度，MySQL的索引是基于B+树数据结构实现。InnoDB是基于索引组织表，其中表的主键作为键，行记录保存在叶子节点。通过索引能够找到行所在的页，然后将这个页加载到内存找到所在的行。并不是直接看可以定位到所在的行。

**索引分类:**  <br>

* 普通索引：在一个列上创建的索引,B-Tree索引
* 复合索引：多个列组成的索引
* 主键索引：主键上默认创建一个索引，唯一，不为空
* 唯一索引：唯一，可以为空(更多用于列约束)
* 全文索引：特殊类型的索引，用于查找全文中的关键词

**索引创建&删除语法：** <br>

* 创建语法：
    - ALTER TABLE tbl_name ADD INDEX|KEY index_name INDEX_TYPE (index_col_name,...)
    - CREATE [UNIQUE] INDEX index_name [INDEX_TYPE] ON tbl_name (index_col_name,...)
* 删除语法
    - ALTER TABLE tbl_name DROP INDEX|KEY index_name
    - DROP INDEX index_name ON tbl_name
* 显示索引
    - SHOW INDEX FOM tbl_name;

**高效索引：** <br>

1. 索引列的选择考虑因素
   
    * Cardinality:列的选择性。也就是选择作为索引列的唯一值与总行数的比值，这个比值越接近于1选择性越高。这个值的计算在数据库是个预估值。计算方法是通过随机8个页的采样。所以每次SHOW INDEX FROM出来的Cardinality可能是一样的，也可能是不一样的。
    
    * 排序 与 分组：选择复合索引时，查询需要排序或分组时考虑索引排序避免在内存里创建临时表再filesort带来的性能提升。

2. B+树的使用
    
    * 建立B+树之后，对这个索引的使用应该只是取少量的数据，这时建立的B+树才有用。(如果通过索引取的是大部分数据，那么这样在读取行的时候会有大量的随机IO,这时的性能并不如全表扫描)
    * 联合索引：

        ①联合索引的查询性能,联合索引的使用需要满足最左前缀。如果有联合索引(a, b)，对于查询 where a=xxx / where a =xxx and b = xxx / where a = xxx and b range (c,d) / where a range(c,d)可以使用到联合索引，对于 where b = xxx / where b range(c,d)是使用不到这个索引的。

        ②联合索引的排序性能。联合索引对第二个键已经排好序。例如订单记录中的(userId,buy_date)，当需要查找userId=1的最近3比订单，where userId=1 limit 3，这时直接取出数据，无需对结果再进行一次排序操作。
   
    * 覆盖索引：从辅助索引中就可以查找到所需的结果,不需要再从聚集索引中扫描得到记录。覆盖索引记录的行内容少所以一个页可以容纳的数据更多，需要的IO操作更少。辅助索引叶子节点保存的是主键的信息，select key from tbl where a=xxx,这样的查询语句会利用覆盖索引。select count(\*) from tbl wehre a=xxx，该查询不需要查询行的具体信息，也可用于覆盖索引。
    
    * <font color="#CD6155">  不使用索引的情况：</font>
    
        ① 查询整行信息，辅助索引中只能找到主键的信息，此时如果满足的行超过20%,就不会使用辅助索引(通过辅助索引还需要通过主键书签扫描聚集索引，此时是随机IO)

    * 索引提示：
        
        USE INDEX() <br>
        FOURCE INDEX()
    


**索引副作用：** <br>
索引对数据的插入性能会有一定的影响，维护索引结构也会有开销。所以并不是一个表建立的索引越多越好。

**题外话:**  <br>
<font color="#CD6155">Cardinality何时更新？</font> 在发生Insert | Update时会触发Cardinality的更新操作，但是并不是每一次的Insert | Update都会更新重新计算一个这个值。当以下条件中的任何一个发生：

    1. 表中1/6的数据发生变化
    2. stat_modified_counter > 2000000000:表示对一行的频繁更新操作

<font color="#CD6155">Alter Table如何创建索引？</font> 
1. 根据alter table 结构创建临时表
2. 将表中的数据临时表
3. 修改临时表为正式表
4. 删除正式表
