### JDBC与事务 ###

**JDBC分页**

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


**ACID**
数据库事务正确执行的四要素 
 
原子性Atomicity  
	整个事务中的操作，要么全部完成，要么全部不完成。事务执行过程中发生错误时，会回滚到事务开始前的状态  

一致性Consistency  
	事务必须保持系统处于一致状态。如系统中有5个账户，每个账户100元，多个事务并发操作这5个账户间的转账，操作完成后账户总额应仍为500元  

隔离性Isolation  
	两个事务同时操作同一行数据时，相互不能影响  

持久性Durability  
	事务完成后，事务对数据库的操作能持久化到数据库中，不会被回滚

**并发事务可能带来的问题**
1、更新丢失  
   两个事务同时更新同一行数据，后提交的事务覆盖先提交的事务  

2、脏读  
   事务A修改了数据未提交；事务B读取到修改后的数据；事务A回滚。事务B读取的是脏数据  

3、不可重复读  
   一个事务两次读取同一行数据，但两次读取的内容不一致。事务B读取修改前的数据；事务A修改数据；事务B再次读取同一行数据（虽同在事务B内，但数据已被修改，两次读取的内容不一致）  

4、幻读(虚读)  
   一个事务两次读取一个范围的数据，两次读取的内容数量不一致。事务B查询数据；事务A操作数据（插入或删除）；事务B再次查询数据（相同sql，也在同一个事务A内，但两次读取的内容不一致）  


* 结合spring了解事务的隔离性和传播性：

**事务传播(Propagation)**

`REQUIRED`  如果当前存在事务，则使用当前事务；否则新建事务。

`SUPPORTS`  如果当前存在事务，则使用当前事务；否则不使用事务。

`MANDATORY` 支持当前事务，如果当前没有事务，则抛出异常

`REQUIRES_NEW`  如果当前存在事务，则挂起当前事务。新建事务执行。

`NOT_SUPPORTED` 如果当前存在事务，则挂起当前事务。以非事务的方式执行。

`NEVER`   如果当前存在事务，抛出异常。以非事务的方式执行。

`NESTED`  如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则操作类似 `REQUIRED`

**事务隔离**

`DEFAULT`

`READ_COMMITTED`  读取已提交数据（会出现不可重复读、幻读）
   读事务不会阻塞读事务和写事务  
   写事务会阻塞读事务和写事务  

`READ_UNCOMMITTED`    读取未提交数据（会出现脏读，不可重复读），基本不会使用
   读事务不会阻塞度事务和写事务    
   写事务不会阻塞读事务  
   写事务会阻塞写事务  

`REPEATABLE_READ` 可重复读（会出现幻读）
   读事务不会阻塞读事务  
   读事务会阻塞写事务  
   写事务会阻塞读事务和写事务 

`SERIALIZABLE`  串行化
   读写事务相互阻塞（相当于串行事务了）  


**事务回滚**

> 默认抛出 RuntimeException时回滚事务

`rollbackFor` 指定事务内抛出哪些异常时执行回滚操作

`noRollbackFor` 指定事务内抛出哪些异常不执行回滚操作

> rollbackFor=Exception.class, noRollbackFor=RuntimeException.class  

> 抛出除了 RuntimeException 之外的异常时，回滚事务

**只读事务**

> 默认情况是，事务是 read/write模式。当我们设置事务为 readonly=true时，表明该事务为只读事务，在只读事务内不会出现更改数据的操作(修改或删除)，对只读事务做更新操作会抛出异常。
> 
> 一般数据库都会对只读事务进行一些优化操作(例如Oracle对只读事务，不启动回滚段，不记录回滚log)。但是单挑查询没有必要使用事务，当我们的业务操作需要多条sql查询时，可以考虑是否需要使用只读事务。

> 当我们做分页查询时，一般会有两条查询一句，一条查询分页的数据，一条查询总数。这时，为保证读一致性，我们可以把分页查询定义为一个 readonly 事务，配合 串行化 事务隔离级别(防止出现幻读)。
