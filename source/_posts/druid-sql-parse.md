---
title: 利用druid sql parser搞一些事情
date: 2017-01-19 14:16:00
categories:
- coding
---
在最近的项目开发中，有这样一个需求，就是给定一个查询的sql，在where语句中添加几个条件语句。刚开始想的是，是否能用正则去做这个事情呢？其实不用语法树还是有一点困难的。

经过一系列google，看到了我们国产的druid里面sql parse的稳当还是比较详尽。具体参考这个文档[SQL Parser](https://github.com/alibaba/druid/wiki/SQL-Parser)

还是回到之前的需求：

````
public List<Map<String, Object>> search(String sql, Map<String, Object> conditions) {
    List<Map<String, Object>> result = new ArrayList<>();
    // SQLParserUtils.createSQLStatementParser可以将sql装载到Parser里面
    SQLStatementParser parser = SQLParserUtils.createSQLStatementParser(sql, JdbcUtils.MYSQL);
	// parseStatementList的返回值SQLStatement本身就是druid里面的语法树对象
    List<SQLStatement> stmtList = parser.parseStatementList();


    SQLStatement stmt = stmtList.get(0);
    if (stmt instanceof SQLSelectStatement) {
        // convert conditions to 'and' statement
        StringBuffer constraintsBuffer = new StringBuffer();
        Set<String> keys = conditions.keySet();
        Iterator<String> keyIter = keys.iterator();
        if (keyIter.hasNext()) {
            constraintsBuffer.append(keyIter.next()).append(" = ?");
        }
        while (keyIter.hasNext()) {
            constraintsBuffer.append(" AND ").append(keyIter.next()).append(" = ?");
        }
        SQLExprParser constraintsParser = SQLParserUtils.createExprParser(constraintsBuffer.toString(), JdbcUtils.MYSQL);
        SQLExpr constraintsExpr = constraintsParser.expr();

        SQLSelectStatement selectStmt = (SQLSelectStatement) stmt;
        // 拿到SQLSelect 通过在这里打断点看对象我们可以看出这是一个树的结构
        SQLSelect sqlselect = selectStmt.getSelect();
        SQLSelectQueryBlock query = (SQLSelectQueryBlock) sqlselect.getQuery();
        SQLExpr whereExpr = query.getWhere();
        // 修改where表达式
        if (whereExpr == null) {
            query.setWhere(constraintsExpr);
        } else {
            SQLBinaryOpExpr newWhereExpr = new SQLBinaryOpExpr(whereExpr, SQLBinaryOperator.BooleanAnd, constraintsExpr);
            query.setWhere(newWhereExpr);
        }
        sqlselect.setQuery(query);
        sql = sqlselect.toString();
        Session session = sessionFactory.openSession();
        SQLQuery sqlQuery = session.createSQLQuery(sql);
        Collection values = conditions.values();
        int index = 1;
        for (Object value : values) {
            sqlQuery.setParameter(index, value);
            index++;
        }
        result = sqlQuery.list();
        session.close();
    } else {
        throw new Exception("not select statement");
    }
    return result;
}
````