title: cassandra查询效率探讨
date: 2016/7/3
categories:
- coding
tags:
- cassandra
---
cassandra目前提倡的建表与查询方式为CQL方式，传统的cassandra-cli相关的api由于性能问题将逐步淘汰，而cassandra-cli也将在2.2版本之后被淘汰。

在CQL中，可以利用类似SQL的方式建立数据表，例如：

```
CREATE TABLE monitor (
    id bigint,
    value text,
    num int,
    timestamp timestamp,
    PRIMARY KEY (id, timestamp ));
```
其中id与timestamp共同构成了primary key。primary key可以不止一个字段，大于一个字段的可以构成clustering key。其中在primary key中第一个字段为partition key，用来决定row在整个ring中的分布。后面的字段为clustering key，对于同一个partition key所代表的行，是根据clustering key以一定顺序在物理上相邻存储的。所以根据partition key以及clustering key进行联合查询速度会比较快。cassandra对于如下查询效率比较高

```
select * from monitor WHERE id = 1;
select * from monitor WHERE id = 2 AND timestamp = '2015-12-01 12:00:00+0800';
select * from monitor WHERE id = 2 AND timestamp > '2015-12-01 12:00:00+0800' AND timestamp < '2015-12-01 23:00:00+0800';
```
但是对于下面的查询，cassandra会返回`InvalidRequest: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"`
```
select * from monitor WHERE timestamp = '2015-12-01 12:00:00+0800';
```
其原因为是cassandra认为这查询效率比较低下，需要用户显式地增加`ALLOW FILTERING`修饰。这种查询过程是先获取所有行，然后在根据`timestamp = '2015-12-01 12:00:00+0800'`进行过滤，效率自然比较低。

解决的办法通常有在`timestamp`字段上建立所以。但不能简单地将cassandra建立索引的机制与普通的关系型数据库如mysql划等号。通过primary key查询，可以通过ring的信息很快的定位到具体的节点。但是通过index查询字段的话，cassandra会每个节点进行查询。虽然节点内部也会对本地数据进行索引，但是效率还是远不如直接查询primary key快。此外cassandra并不能够对于`timestamp >'2015-12-01 12:00:00+0800'`这种范围条件进行查询。所以更好的方式是另外建立一个表，将需要查询的字段作为主键，并存储对应关系。

**参考资料**

1. [ALLOW FILTERING explained](http://www.datastax.com/dev/blog/allow-filtering-explained-2)
2. [A deep look at the CQL WHERE clause](http://www.datastax.com/dev/blog/a-deep-look-to-the-cql-where-clause)
3. [When to use an index](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_build_index_c.html)

