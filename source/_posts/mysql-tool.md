---
title: MySQL问题排查工具介绍
date: 2017-03-24 08:08:58
tags: mysql
categories: 开发
---
本总结来自美团内部分享，屏蔽了内部数据与工具

## 知识准备
### 索引
- 索引是存储引擎用于快速找到记录的一种数据结构
- B-Tree，适用于全键值，键值范围或键最左前缀：(A,B,C): A, AB, ABC,B,C,BC
- 哪些列建议创建索引：WHERE, JOIN , GROUP BY, ORDER BY等语句使用的列
- 如何选择索引列的顺序：
  1. 经常被使用到的列优先
  2. 选择性高的列优先：选择性=distinct(col)/count(col) 
  3. 宽度小的列优先：宽度 = 列的数据类型

### 慢查询
#### 原因
1. 未使用索引
2. 索引不优
3. 服务器配置不佳
4. 死锁
5. ...

#### 命令
##### 看版本
mysql -V 客户端版本 select version 服务器版本
##### explain 执行计划，慢查询分析神器
- type
  - const,system: 最多匹配一个行，使用主键或者unique进行索引
  - eq_ref: 返回一行数据，通常在联接时出现，使用主键或者unique索引（内表索引连接类型）
  - ref: 使用key的最左前缀，且key不是主键或unique键
  - range: 索引范围扫描，对索引的扫面开始于某一点，返回匹配的行
  - index:以索引的顺序进行全表扫描，优点是不用排序，缺点是还要全表扫描
  - all: 全表扫描 no no no
  
- extra
  - using index : 索引覆盖，只用到索引，可以避免访问表
  - using where: 在存储引擎检索行后再做过滤
  - using temporary:使用临时表，通常在使用GROUP BY，ORDER BY 时出现（严禁）
  - using filesort:  到非索引顺序的额外排序，当order by col未使到索引时发生（严禁）
- possible_keys: 显示查询可能使用的索引
- key:优化器决定采用哪个索引来优化对该表的访问
- rows:MySQL估算的为了找到所需行要检索的数，优化选择key的参考 （不是结果集的行数）
- key_len: 使用的索引左前缀的长度(字节数)，亦可理解为使用了索引中哪些字段
  - 定长字段，int占4个字节、date占3个字节、timestamp占4个字节，char(n)占n个字节
  - NULL的字段:需要加1个字节，因此建议尽亮设计为NOT NULL
  - 变长字段varchar(n)，则需要 (n * 编码字符所占字节数 + 2 、)个字节，如utf8编码的， 个字符
占 3个字节，则 度为 n * 3 + 2
- 强制使用索引: USE INDEX （建议）或 FORCE_INDEX （强制）

#### show 命令
- show status
  - 查看select语句的执行数 show global status like 'Com_select';
  - 查看慢查询的个数  show global status like 'Slow_queries';
  - 表扫描情况 show global status like 'Handler_read%';  Handler_read_rnd_next / com_select > 4000 需要考虑优化索引
- show variables
  - 查看慢查询相关的配置 show variables like 'long_query_time';
  - 将慢查询时间线设置为2s set global long_query_time=2;
  - 查看InnoDB缓存  show variables like 'innodb_buffer_pool_size';
  - 查看InnoDB缓存的使用状态 show status like 'Innodb_buffer_pool_%'; 缓存命中率=(1-Innodb_buffer_pool_reads/ Innodb_buffer_pool_read_requests) * 100%；缓存率=（Innodb_buffer_pool_pages_data/ Innodb_buffer_pool_pages_total) * 100%
  - SHOW PROFILES;该命令可以trace在整个执行过程中各资源消耗情况(会话级)
  - SHOW PROCESSLIST; 查看当前有哪些线程正在运行，并且处在何种状态
  - SHOW ENGINE INNODB STATUS; 可用于分析死锁，但需要super权限
