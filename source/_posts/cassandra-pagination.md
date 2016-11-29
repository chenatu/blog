title: cassandra分页
date: 2016/7/3
categories:
- coding
tags:
- cassandra
---
在cassandra的协议中，没有具体规定查询结果的行数限制。但是对于大的数据集，依然有结果分页的必要。过大的结果集会爆掉服务端或者客户端的内存。

传统的分页方法采用了一点trick，采用了token函数

```
SELECT * FROM images LIMIT 100;
SELECT * FROM images WHERE token(image_id) > token([Last image ID received]) LIMIT 100;
```

这种方式会造成一点编程上的麻烦，一般开发中会重新再封装一次分页的方法。在cassandra2.0的java api中，添加了对于分页的支持，如下所示：

```
Statement stmt = new SimpleStatement("select * FROM raw_weather_data WHERE wsid= '725474:99999' AND year = 2005 AND month = 6");
stmt.setFetchSize(24);
ResultSet rs = session.execute(stmt);
Iterator<Row> iter = rs.iterator();
while (!rs.isFullyFetched()) {
   rs.fetchMoreResults();
   Row row = iter.next();
   System.out.println(row);
}
```

**参考资料**
1. [Things You Should Be Doing When Using Cassandra Drivers](https://ahappyknockoutmouse.wordpress.com/2014/11/12/246/)
2. [Improvements on the driver side with Cassandra 2.0](http://www.datastax.com/dev/blog/client-side-improvements-in-cassandra-2-0)