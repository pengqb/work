# 一、错误现象 #
使用com.zaxxer.HikariCP 2.4.7 做为mysql连接池。在使用过程中，有两台机器使用一段时间后，开始报下面的错误。其他机器没有报这样的错误，其他机器可能的区别是其他机器和mysql交互比较多，这两天机器和mysql交互比较少。

    org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.exceptions.PersistenceException: 
    ### Error querying database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30056ms.
    ### The error may exist in com/vanelife/server/dao/InfoAppKeyMapper.xml
    ### The error may involve com.vanelife.server.dao.InfoAppKeyMapper.selectByAppKey
    ### The error occurred while executing a query
    ### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30056ms.
    	at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:75) ~[mybatis-spring-1.2.2.jar:1.2.2]
    	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:371) ~[mybatis-spring-1.2.2.jar:1.2.2]
    	at com.sun.proxy.$Proxy23.selectOne(Unknown Source) ~[?:?]

# 二、定位步骤 #
# 1、设置leakDetectionThreshold来检测连接泄漏。 #

	<!-- Hikari Datasource -->  
	<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource"  destroy-method="shutdown">  
		 <property name="driverClassName" value="${jdbc.driver}" />  <!-- 无需指定，除非系统无法自动识别 -->  
		 <property name="jdbcUrl" value="${jdbc.url}" />  
		 <property name="username" value="${jdbc.username}" />  
		 <property name="password" value="${jdbc.password}" />  
		  <!-- 连接只读数据库时配置为true， 保证安全 -->  
		 <property name="readOnly" value="false" />  
		 <!-- 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒 -->  
		 <property name="connectionTimeout" value="30000" />  
		 <!-- 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟 -->  
		 <property name="idleTimeout" value="600000" />  
		 <!-- 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒，参考MySQL wait_timeout参数（show variables like '%timeout%';） -->  
		 <property name="maxLifetime" value="1800000" />  
		 <!-- 连接池中允许的最大连接数。缺省值：10；推荐的公式：((core_count * 2) + effective_spindle_count) -->  
		 <property name="maximumPoolSize" value="10" />  
		 <property name="leakDetectionThreshold" value="30000" />  
	</bean>

2、log4j2里打印Hikari日志

    <Logger name="com.zaxxer.hikari" level="debug" />

看到如下日志

    2016-12-15 15:57:03.198|vanelife-App9|debug|HikariPool-1 connection closer|com.zaxxer.hikari.pool.PoolBase|quietlyCloseConnection|HikariPool-1 - Closing connection com.mysql.jdbc.JDBC4Connection@6662850c: (connection has passed maxLifetime)
    2016-12-15 15:57:07.880|vanelife-App9|debug|HikariPool-1 connection adder|com.zaxxer.hikari.pool.HikariPool|createPoolEntry|HikariPool-1 - Added connection com.mysql.jdbc.JDBC4Connection@542bd87
    2016-12-15 15:55:37.791|vanelife-App9|debug|HikariPool-1 housekeeper|com.zaxxer.hikari.pool.HikariPool|logPoolState|HikariPool-1 - Pool stats (total=10, active=0, idle=10, waiting=0)

最后发现是Hikari Datasource配置的 maxLifetime远远大于mysql数据库配置的wait_timeout造成的。<!-- 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒，参考MySQL wait_timeout参数（show variables like '%timeout%';） --> 
<property name="maxLifetime" value="1800000" />

参考文档
https://github.com/brettwooldridge/HikariCP/issues/109
http://stackoverflow.com/questions/32968530/hikaricp-connection-is-not-available