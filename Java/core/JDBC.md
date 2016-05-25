### JDBC与事务 ###

#####JDBC分页#####

```
Class.forName("com.mysql.jdbc.Driver");
String url = "";
String user = "root";
String password = "000000";

String sql = "select * from user";
Connection conn = DriverManager.getConnection(url, user, password);
PreparedStatement stmt = conn.preparedStatement(sql);
//(1) stmt.setFetchSize(5); // 每次从数据库取5条
//(2) stmt.setMaxRows(3);   // ResultSet最多返回3条（即使从数据库查询到10条）

ResultSet rs = stmt.executeQuery();
//(3) rs.absolute(2);  游标移到从头数第2位(绝对位置)
while(rs.next()) {
  //(4) rs.relative(2);  游标移到当前位置后第2位(相对位置)，超过有效位置置为first或last
  if(!rs.isAfterLast()) {
    System.out.println(rs.getString(1));
  }
}
```

1. 不放开注释， 输出：1\2\3\4\5\6\7。(7条数据)
2. 放开注释(1)，影响数据库查询性能(和数据库交互次数)，不影响返回的数据。
3. 放开注释(2)，输出：1\2\3。
4. 放开注释(3)，输出：3\4\5\6\7。
5. 放开注释(4)，输出：3\6。

> the JDBC fetch size gives the JDBC driver a hint as to the number of rows that should be fetched from the database when more rows are needed. For large queries that return a large number of objects you can configure the row fetch size used in the query to improve performance by reducing the number database hits required to satisfy the selection criteria. Most JDBC drivers defalt to a fetch size of 10, so if you are reading 1000 objects, increasingthe fetch size to 256 can significantly reduce the time required to fetch the query's results. Usually, a fetch size of one half or one quarter of the total expected result size is optimal. 


##### ACID
数据库事务正确执行的四要素 
 
原子性Atomicity  
	整个事务中的操作，要么全部完成，要么全部不完成。事务执行过程中发生错误时，会回滚到事务开始前的状态  

一致性Consistency  
	事务必须保持系统处于一致状态。如系统中有5个账户，每个账户100元，多个事务并发操作这5个账户间的转账，操作完成后账户总额应仍为500元  

隔离性Isolation  
	两个事务同时操作同一行数据时，相互不能影响  

持久性Durability  
	事务完成后，事务对数据库的操作能持久化到数据库中，不会被回滚

##### 并发事务可能带来的问题
1、更新丢失  
   两个事务同时更新同一行数据，后提交的事务覆盖先提交的事务  

2、脏读  
   事务A修改了数据未提交；事务B读取到修改后的数据；事务A回滚。事务B读取的是脏数据  

3、不可重复读  
   一个事务两次读取同一行数据，但两次读取的内容不一致。事务B读取修改前的数据；事务A修改数据；事务B再次读取同一行数据（虽同在事务B内，但数据已被修改，两次读取的内容不一致）  

4、幻读(虚读)  
   一个事务两次读取一个范围的数据，两次读取的内容数量不一致。事务B查询数据；事务A操作数据（插入或删除）；事务B再次查询数据（相同sql，也在同一个事务A内，但两次读取的内容不一致）  

##### 事务隔离级别
1、读取未提交  
   读事务不会阻塞度事务和写事务    
   写事务不会阻塞读事务  
   写事务会阻塞写事务  

2、读取已提交(解决脏读)  
   读事务不会阻塞读事务和写事务  
   写事务会阻塞读事务和写事务  

3、可重复读(解决脏读 + 不可重复读)  
   读事务不会阻塞读事务  
   读事务会阻塞写事务  
   写事务会阻塞读事务和写事务  

4、序列化(解决脏读 + 不可重复读 + 幻读)  
   读写事务相互阻塞（相当于串行事务了）  
