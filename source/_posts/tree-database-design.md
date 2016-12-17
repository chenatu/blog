---
title: 数据库友好的树结构设计
date: 2016-12-17 10:39:12
categories:
- coding
---
在最近的系统设计中，涉及到对组织对象或者目录对象组成的树进行重构。我们第一版本的设计参考了这篇文章的设计[Managing Hierarchical Data in MySQL](http://mikehillyer.com/articles/managing-hierarchical-data-in-mysql/)。

最简单朴素的树结构设计是这样的：

````
CREATE TABLE category(
        category_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(20) NOT NULL,
        parent INT DEFAULT NULL
);
````

但是这种设计对于你想拿一个节点下的子树这种操作不太友好。虽然这种对子树操作比如移动，删除这种还是比较友好的。但是考虑到，组织树这种很少修改，而获取子树这种操作比较多。所以就采取了这篇文章推荐的方案。

````
CREATE TABLE nested_category (
        category_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(20) NOT NULL,
        lft INT NOT NULL,
        rgt INT NOT NULL
);

INSERT INTO nested_category VALUES(1,'ELECTRONICS',1,20),(2,'TELEVISIONS',2,9),(3,'TUBE',3,4),
 (4,'LCD',5,6),(5,'PLASMA',7,8),(6,'PORTABLE ELECTRONICS',10,19),(7,'MP3 PLAYERS',11,14),(8,'FLASH',12,13),
 (9,'CD PLAYERS',15,16),(10,'2 WAY RADIOS',17,18);

SELECT * FROM nested_category ORDER BY category_id;

+-------------+----------------------+-----+-----+
| category_id | name                 | lft | rgt |
+-------------+----------------------+-----+-----+
|           1 | ELECTRONICS          |   1 |  20 |
|           2 | TELEVISIONS          |   2 |   9 |
|           3 | TUBE                 |   3 |   4 |
|           4 | LCD                  |   5 |   6 |
|           5 | PLASMA               |   7 |   8 |
|           6 | PORTABLE ELECTRONICS |  10 |  19 |
|           7 | MP3 PLAYERS          |  11 |  14 |
|           8 | FLASH                |  12 |  13 |
|           9 | CD PLAYERS           |  15 |  16 |
|          10 | 2 WAY RADIOS         |  17 |  18 |
+-------------+----------------------+-----+-----+

````

具体模型如图所示

![Nested Set Model][1]

这种你要想获取一个子树，只要限定lft和rgt的范围就好
````
SELECT node.name
FROM nested_category AS node,
        nested_category AS parent
WHERE node.lft BETWEEN parent.lft AND parent.rgt
        AND parent.name = 'ELECTRONICS'
ORDER BY node.lft;
````

从代码上，简练了很多。但是考虑数据库性能方面就有一个问题。比如，为了数据库加速，是不是要给lft和rgt做索引？虽然我们可以做索引，但是是非聚合索引，也就是说在磁盘上不是连续的。获取多行的时候，还是要涉及随机读写。

为了加强连续读写的性能，我们利用前缀树这种方式建立了树
````
CREATE TABLE `pe_organ` (
  `parent` varchar(255) NOT NULL,
  `current` varchar(255) NOT NULL,
  `level` int(11) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `site_id` varchar(10) NOT NULL,
  PRIMARY KEY (`parent`,`current`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

parent	current	level
NULL	/1-1	1
NULL	/1-2	1
NULL	/1-3	1
/1-1	/1-1/2-1	2
/1-1	/1-1/2-2	2
/1-1/2-1	/1-1/2-1/3-1	3

````

parent为父亲节点的路径，current表示当前节点的路径。如果我们想拿`/1-1`的所有子节点的话只要

````
SELECT * FROM pe_organ WHERE parent LIKE '/1-1%'
````
这个查询是走索引的。

同时由于我们建立了联合主键，获取的子节点在硬盘的安排上是连续的

至于每个级别子路径的设计 我们采取了 `level-number`的模式

需要注意，这种树的设计在移动树的方面还是比较费劲的，这种设计的主要目的是加速子树查询。

[1]: http://mikehillyer.com/media//nested_categories.png  "Nested Set Model"