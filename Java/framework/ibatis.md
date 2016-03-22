##读源码理解ibatis

`jdbc.poperties`
<pre><code>
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/manager
jdbc.user=root
jdbc.password=123456
</code></pre>

`sqlMapConfig.xml`


	<sqlMapConfig>
		<!-- 本配置文件中定义的变量(如${jdbc.url})会替换成properties中配置的值 -->
		<properties resource="jdbc.properties" />

		<settings cacheModelsEnabled="true"     // true的情况下会把sqlMap文件中的cacheModel节点配置到delegate中
			enhancementEnabled="true"
			classInfoCacheEnabled="true"
			lazyLoadingEnabled="true"
			statementCachingEnabled="true"      // 缓存preparedStatement
			maxRequests="512"
			maxSessions="128"
			maxTransactions="32"
			defaultStatementTimeout="11"		// 设置preparedStatement的查询超时时间, preparedStatement.setQueryTimeout()
			useStatementNamespaces="true" />	// 如，sqlMapParser构建statement的ID时，使用sqlmap文件的namespace + statement的id值构成全局唯一的ID

		<transactionManager type="JDBC">		// JDBC, JTA, EXTERNAL
			<dataSource type="SIMPLE">			// SIMPLE, DBCP, JNDI
				<property name="JDBC.Driver" value="${jdbc.driver}" />
				<property name="JDBC.ConnectionURL" value="${jdbc.url}" />
				<property name="JDBC.Username" value="${jdbc.user}" />
				<property name="JDBC.Password" value="${jdbc.password}" />
			</dataSource>
		</transactionManager>
	
		<typeAlias alias="Address" type="com.long.account.vo.Address" />

		<typeHandler javaType="" callback=""/>

		<sqlMap resource="com/long/account/sqlmap/sql_map_account.xml" />
	</sqlMapConfig>


`sql_map_account.xml`



	<sqlMap namespace="account">
		<typeAlias alias="Account" type="com.long.account.vo.Account" />
		
		<resultMap id="account.result" class="map">
			<result column="user_name" property="userName" jdbcType="VARCHAR"></result>
		</resultMap>

		<sql id="page"></sql>
		<select id="queryAccount" cacheModel="lruCache"></select>
		<update id="updateAccount"></update>

		<cacheModel id="lruCache" 
			type="LRU" 
			serialize="true"		// 序列化缓存的对象，从cache里读取的是反序列化的对象，操作时不影响缓存中的对象(readonly=false时才有效)
			readonly="false">		// 缓存对象是否只读，不应该被修改

			<flushOnExecute statement="updateAccount" />
		</cacheModel>
	</sqlMap>



##### SqlMapClientImpl实例化
使用ibatis的第一步是实例化一个SqlMapClientImpl实例化对象。

1. 使用SqlMapClientBuilder.buildSqlMapClient(Reader reader, Properties props)构建SqlMapClientImpl实例
2. 使用`SqlMapConfigParser`.parse(Reader reader, Properties props)解析`sqlMapConfig.xml`
3. 注册基本的type alias(`JDBC`, `JTA`, `SIMPLE`, `OSCACHE`, `dom`, `...`)，并放入`TypeHandlerFactory实例`中，TypeHandlerFactory实例为delegate的一个属性。
4. 注册缓存的监听器。sqlMap文件的cacheModel节点定义了那些statement会引发cache的flush。*所有的sqlmap文件都解析完后才会注册监听器*。
5. 读取`properties节点`定义的资源文件。
6. 读取`settings节点`定义，并注入`delegate实例`。
7. 读取自定义`typeAlias节点`定义并注册到`TypeHandlerFactory实例`中。
8. 读取`typeHandler节点`定义，并注册(javaType + callback `typeHandler`)到`TypeHandlerFactory实例`中。javaType可能是alias。
9. 读取`transactionManager节点`定义。主要是数据源定义信息和事务定义信息。`delegate`对象持有一个TransactionManager实例，TransactionManager实例持有一个TransactionConfig对象，包含一些事务相关信息；TransactionConfig对象持有一个数据源对象DataSource，DataSource对象持有数据库连接（池）相关信息。
10. 读取`sqlMap节点`定义。定义了`SqlMapParser`.parse方法需要解析的数据。
11. 读取`resultObjectFactory节点`定义。

12. 使用SqlMapParser.parse()解析`sql_map_account.xml`
13. 记录namespace
14. 解析`typeAlias`节点，与SqlMapConfigParser中处理方式一致(6)
15. 解析`sql`节点，(namespace + id) 作为key存入sqlIncludes.
16. 解析`statement`, `insert`, `select`, `update`, `delete`节点，解析成InsertStatement等并注册到`delegate实例`
17. 解析`procedure`节点，解析成ProcedureStatement并注册到`delegate实例`
18. 如果`select`节点带`cacheModel`属性，则SelectStatement变成CachingStatement。statement实际执行时先查缓存，缓存未命中再走普通的jdbc查询方式
19. 解析`cacheModel`节点，如果settings中定义isCacheModelsEnabled=true才会把CacheModel注册到`delegate实例`中
20. 解析`parameterMap`节点。注册到`delegate实例`中
21. 解析`resultMap`节点。注册到`delegate实例`中

##### 执行数据库操作
    sqlMapClient.insert(id, param)

1. sqlMapClient调用sqlMapSessionImpl.insert(id, param)。`sqlMapSessionImpl`为sqlMapClient持有的一个ThreadLocal属性，线程独立。sqlMapSessionImpl持有一个`SqlMapExecutorDelegate`实例，使用的`SessionScop`由delegate实例维护的一个池提供。SessionScope包含sqlMapClient, sqlMapExecutor, sqlMapTxMgr属性，三个属性均为sqlMapClient.
2. sqlMapSessionImpl调用调用delegate.insert(sessionScope, id, param)
3. 通过id查到MappedStatement。
4. 从sessionScope中获取一个`Transaction对象`
5. 如果需要自动启动事务。session.getSqlMapTxMgr().startTransaction()，实际调用sqlMapSessionImpl.startTransaction(),再代理到delegate.startTransaction(session, transactionIsolation)，再代理到TransactionManager.begin(session, transactionIsolation)。TransactionManager实例是在实例化SqlMapClientImpl中，读取sqlmapConfig文件解析transactionManager节点实例化注入delegate的。该步操作其实是实例化一个 Transaction对象并注入session。
6. 从delegate实例维护的一个池中提取一个RequestScope,把sesion和statement关联到request
7. 调用statement.execute(request, transaction, param);  



	如果是CachingStatement，先从CacheModel中查询目标内容，如果没有命中，继续下面流程； 

	从transaction中获取一个Connection对象;  
	获得一个preparedStatement(通过connection.preparedStatement()获得,或者当statementCachingEnabled=true时，可能会从**session的缓存**中获取);  
	设置preparedStatement的timeout;  
	设置preparedStatement的parameters;  
	preparedStatement.execute();  
	获得返回数据；  
	preparedStatement.close();    


   
8.. 把request对象还回池中  
9.. 如果需要自动提交事务


##### ibatis与分页
1、调用jdbc处理ResultSet时实现分页。  
   rs.absolute(skipResults), 找到目标第一行数据；  
   一直读取到(最多)maxResults行；  
   不再处理剩余数据；  

2、缓存中实现分页  
   缓存中的key，与request(statement's id, sql), parameter, executeQueryForObject| executeQueryForList, skipResults, maxResults有关。任意一个不一致，都可能导致缓存不能命中，即使sql内容一致

3、sql语句中实现分页（不同数据库sql语句分页方式不一样）


http://www.ibm.com/developerworks/cn/java/j-lo-ibatis-principle/