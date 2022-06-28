### jdbc连接数据库步骤
	1. 注册驱动和数据库信息
	2. 操作Connection,打开Statement对象
	3. 通过Statement对象执行sql,返回结果到ResultSet对象
	4. 从ResultSet中读取数据,通过代码转化为具体的pojo对象
	5. 释放数据库连接资源

### 使用jdbc的弊端
	1. 需要自己处理连接,然后处理jdbc底层事务,处理数据类型,
	2. 需要对jdbc产生的异常进行处理,
	3. 对象,资源需要管理和正确的释放

### ORM(对象关系映射)
	- 解决数据库数据和pojo对象的相互映射,通过映射关系,将数据库数据转化为pojo

	- 优点
		1. 消除了代码映射规则,将其分离到xml或者注解里面配置
		2. 无需管理数据库连接,也放在xml里面配置
		3. 一个会话中不需要操作多个对象,仅操作session对象即可
		4. 关闭资源也只需要处理session即可


### Hibernate 
	- pojo通过xml映射文件提供的映射规则映射到数据库表上的。即通过pojo直接操作数据库数据。是一种全表映射的模型。

	- 缺点
		1. 全表映射,更新时需要发送所有的字段
		2. 无法根据不同的条件,组装不同的sql
		3. 对多表关联和复杂sql查询支持较差,需要自己写sql,自己将返回数据组装为pojo
		4. 不能有效的支持存储过程
		5. hql性能较差,没法对sql进行优化

### Mybatis与jdbc比较
	- jdbc数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能.mybatis在mybatis-config.xml中配置数据链接池，使用连接池管理数据库连接
	- jdbc将Sql语句写在代码中造成代码不易维护.mybatis将Sql语句配置在XXXXmapper.xml文件中与java代码分离
	- jdbc向sql语句传参数麻烦,因为sql语句的where条件不一定,可能多也可能少,占位符需要和参数一一对应.Mybatis自动将java对象映射至sql语句
	- jdbc对结果集解析麻烦,sql变化导致解析代码变化,且解析前需要遍历.Mybatis自动将sql执行结果映射至java对象

### Mybatis
	- 半自动映射框架,需要手动配置pojo,sql和映射关系

	- 优点
		1. 可动态配置sql
		2. 可优化sql,通过配置优化sql的映射规则
		3. 支持存储过程
		4. 支持复杂和优化性能sql的查询

	- Mybatis的核心组件
		- Configuration对象
			- 存在于整个MyBatis的生命周期中,解析的xml文件配置信息都保存在这个对象中

		- SqlSessionFactoryBuilder
			- 根据配置信息生成SqlSessionFactory

		- SqlSessionFactory
			- 是一个工厂接口而不是现实类,任务是创建SqlSession

		- SqlSession
			- 类似于jdbc的Connection对象

		- SqlMapper
			- 由java接口和xml映射文件构成,负责发送sql去执行,并返回结果

### Mybatis三层功能架构
	- API接口层：提供给外部使用的接口API,开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理
	- 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作
	- 基础支撑层：负责最基础的功能支撑,包括连接管理、事务管理、配置加载和缓存处理,这些都是共用的东西,将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑.

### Mybatis查询流程
	- 加载配置
		- 配置来源于两个地方,一处是配置文件,一处是Java代码的注解,将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置)
		  存储在内存中
	- SQL解析
		- 当API接口层接收到调用请求时,会接收到传入SQL的ID和传入对象(可以是Map、JavaBean或者基本数据类型),Mybatis会根据SQL的ID找到对应的MappedStatement,
		  然后根据传入参数对象对MappedStatement进行解析,解析后可以得到最终要执行的SQL语句和参数
	- SQL执行
		- 将最终得到的SQL和参数拿到数据库进行执行,得到操作数据库的结果
	- 结果映射
		- 将操作数据库的结果按照映射的配置进行转换,可以转换成HashMap、JavaBean或者基本数据类型,并将最终结果返回


### Mybatis配置
	- properties 属性
	- settings 设置
	- typeAliases 别名
	- typeHandlers 类型处理器 java类型与数据库类型转换
	- objectFactory 对象工厂

### 反射与动态代理
	- 反射
		- 通过类类型知道一个类的属性和方法,并且可以调用该类的属性和方法,这是反射的基础
	- 动态代理
		- 代理类在程序运行时创建的代理方式被称为动态代理。
		- jdk动态代理
			- 由java内部的反射机制来实例化代理对象,并代理的调用委托类方法(被代理的对象是接口)
		- CGLIB动态代理
			- 基于继承,被代理类生成代理子类，不用实现接口。(被代理类是非final类)


### Mybatis的运行流程
	1. 根据配置的xml文件,读出配置参数,将读取的数据存入Configuration类中	
	2. Configuration初始化基础配置和重要的类对象(别名,插件,映射器,objectFactory,typeHandler)
	3. 构建映射器信息
		- mappedStatement(根据xml配置的sql生成的对象信息)
		- sqlSource 获取BoundSql对象(设置参数)
		- BoundSql 解析后的sql语句
	4. 根据配置信息通过SqlSessionFactoryBuilder构建SqlSessionFactory
	5. 通过SqlSessionFactory获取SqlSession
	6. openSession
		- openSessionFromDataSource 从数据源获取sqlsession对象
		- openSessionFromConnection	从数据库连接池获取sqlsession对象
	7. SqlSession通过selectOne或者mapper对象执行sql
		- selectOne
			- 从configuration的成员变量mappedStatements中获取MappedStatement对象。mappedStatements是Map<String, MappedStatement>类型的缓存结构，其中key就是mapper接口全类名+方法名，MappedStatement就是对标签中配置的sql一个包装
			- 使用executor成员变量来执行查询并且指定结果处理器，并且返回结果。Executor也是mybatis的一个重要的组件。sql的执行都是由Executor对象来操作的。
		- mapper对象
			- 调用代理对象的invoke方法,进而调用MapperMethod的execute方法
			- MapperMethod(调用sql的方法+sql)
	8. Executor执行器
		- Executor作为接口,包含更新,查询,事务等方法,每个sqlsession都有一个Executor对象,sqlsession的操作都由Executor执行
		- 三大执行器类
			- SimpleExecutor：默认的执行器,每次执行update或者select操作,都会创建一个Statement对象,执行结束后关闭Statement对象。
				1. 创建StatementHandler
				2. 创建Statement
				3. 执行sql操作
				4. 关闭Statement

			- ReuseExecutor：可重用执行器,重用的是Statement对象,第一次执行一条sql,会将这条sql的Statement对象缓存在key-value结构的map缓存中。下一次执行,就可以从缓存中取出Statement对象,减少了重复编译的次数,从而提高了性能。每个SqlSession对象都有一个Executor对象,因此这个缓存是SqlSession级别的,当SqlSession销毁时,缓存也会销毁。
				1. Sql是否命中缓存
				2. 命中缓存直接从缓存中获取到Statement对象
				3.如果缓存中没有，则创建新的Statement对象
				4.接第3步，以sql为key, Statement为value放到缓存中

			- BatchExecutor：批量执行器,默认情况是每次执行一条sql,MyBatis都会发送一条sql。而批量执行器的操作是,每次执行一条sql,不会立马发送到数据库,而是批量一次性发送sql。
				1. doUpdate()返回的值是固定的，不是影响的行数
				2. 如果连续提交相同的sql，则只会执行一次
				3. 提交sql不会立马执行，而是等到commit时候统一执行
				4. 底层使用的是JDBC的批处理操作，addBatch()和executeBatch()操作。

		- 缓存CachingExecutor
			- 如果开启了缓存,会使用装饰模式,将上面三执行器包装成CachingExecutor
				1. 执行更新操作的时候，先清空缓存，再去执行实际执行器的update方法
				2. 在执行查询的时候，先从缓存中获取，如果缓存中没有再去调用实际执行器的query方法查询数据库，并放到缓存中返回

	9. Statement和StatementHandler
		
		- StatementHandler管理Statement对象
			- 执行数据库操作时,都会创建StatementHandler对象
			- StatementHandler对象实际由Configuration对象根据MappedStatement对象的StatementType属性,创建出具体的StatementHandler对象。
			- BaseStatementHandler
				- SimpleStatementHandler
				- PreparedStatementHandler
				- CallableStatementHandler
			- RoutingStatementHandler
				- 包装具体的StatementHandler,作为代理类进行操作

		- Statement 
			- 三种实现
				- Statement 可发送字符串类型sql,不支持传递参数,静态sql
				- PreparedStatement 预编译的sql语句,接受参数传入,可防止sql注入,安全性高
				- CallableStatement 执行存储过程使用,接受参数传入

			- Statement对象在jdbc操作中就是向数据库发送sql语句,并获取执行结果
			- 通过StatementHandler创建Statement 
				1. 初始化Statement对象(jdbc创建Statement对象)
				2. 设置超时时间
				3. 设置查询大小 
				4. 出现异常关闭Statement对象

		- 预编译的理解
			- JDBC中使用对象PreparedStatement来抽象预编译语句,使用预编译。预编译阶段可以优化SQL的执行。预编译之后的SQL多数情况下可以直接执行. 
			  预编译语句对象可以重复利用。把一个SQL预编译后产生的PreparedStatement对象缓存下来,下次对于同一个SQL,可以直接使用这个缓存的
			  PreparedState对象。Mybatis默认情况下,将对所有的SQL进行预编译。


	10. MappedStatement对象
		- 对sql标签的描述

	11. BoundSql
		- 对动态标签的解析,并且将 #{} 解析为占位符?,还包含参数的描述信息。即对解析后的sql描述 
		- BoundSql从MappedStatement的成员变量sqlSource中获取
		- SqlSource
			- 如果sql中只包含#{}参数,不包含${}或者其它动态标签,那么创建SqlSource对象时则会创建RawSqlSource,否则创建DynamicSqlSource对象。
			- DynamicSqlSource会优先解析{}标签，然后解析#{}标签。其中{}会被解析为参数内容，不会加上双引号，而#{}会被解析为?，并且参数会加上双引号

### 在mapper中如何传递多个参数
	- 顺序传参法 select * from user where user_name = #{0} and dept_id = #{1} #{}里面的数字代表传入参数的顺序
	- @Param注解传参法 public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);  
	  select * from user where user_name = #{userName} and dept_id = #{deptId}
	  #{}里面的名称对应的是注解@Param括号里面修饰的名称
	- Map传参法 public User selectUser(Map<String, Object> params);
	  select * from user where user_name = #{userName} and dept_id = #{deptId}
	  #{}里面的名称对应的是Map里面的key名称
	- Java Bean传参法 public User selectUser(User user);
	  select * from user where user_name = #{userName} and dept_id = #{deptId}
	  #{}里面的名称对应的是User类里面的成员属性

### 实体类中的属性名和表中的字段名不一致处理
	- 通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致
	- 通过<resultMap>来映射字段名和实体类属性名的一一对应的关系

### MyBatis的接口绑定的理解
	- 在MyBatis中任意定义接口,然后把接口里面的方法和SQL语句绑定,直接调用接口方法.
	- 绑定方式
		- 通过注解绑定,就是在接口的方法上面加上 @Select、@Update等注解，里面包含Sql语句来绑定
		- 通过xml里面写SQL来绑定,指定xml映射文件里面的namespace必须为接口的全路径名

### Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗
	- Dao接口即Mapper接口,接口的全限名,就是映射文件中的namespace的值,接口的方法名,映射文件中MappedStatement的id值,接口方法内的参数,就是传递给sql的参数。
	  Mapper接口是没有实现类的,当调用接口方法时,接口全限名+方法名拼接字符串作为key值,可唯一定位一个MappedStatement。
	- Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略
	- Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，
	  然后将sql执行结果返回

### Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？
	- 不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复
	- namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖

### 动态SQL
	- Mybatis动态sql实现Xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能



### Mybatis的缓存
	- 一级缓存
		- 默认开启,作用范围是sqlsession级别的
		-关闭方式
			- mybatis-config.xml中的settings标签中将localCacheScope设置成Statement类型,关闭一级缓存
			- 在select标签加上flushCache="true"关闭该sql的一级缓存
		- 创建
			- createCacheKey会给每个sql都生成一个key，如果两个生成的key一致,表明不管是sql还是参数都是一致的,通过该key进行缓存查询
		- 清除
			- 执行update，commit，或者rollback操作的时候都会对一级缓存进行清除缓存操作。

	- 二级缓存
		- 是Mapper级别的缓存，因此不同sqlSession间可以共享缓存
		- 开启
			- setting配置cacheEnabled为true，这个属性默认为true。<setting name="cacheEnabled" value="true"/>
			- 需要进行开启二级缓存的mapper中新增cache配置
			- 对单个Statement标签进行关闭和开启操作，通过配置useCache="true"来开启缓存
		- 实现
			- 在Configuration创建Executor的时候，如果开启了二级缓存，就使用到了CachingExecutor进行了包装,通过TransactionalCacheManager用来管理缓存。
			- 二级缓存开启时，优先从二级缓存中查找，再去从一级缓存中查找，最后从数据库查找。
		- 清除
			- 在执行更新操作的时候进行缓存删除操作

### Mybatis的事务
	- Transaction
		- JdbcTransaction 使用Jdbc中的java.sql.Connection来管理事务,包括提交回滚
		- ManagedTransaction MyBatis不会管理事务，而是交由程序的运行容器(weblogic，tomcat)来进行管理。
	- 创建
		- 在mybatis-config.xml中配置事务管理的类型
		- XMLConfigBuilder中的environmentsElement会解析transactionManager类型,并且会创建一个TransactionFactory类型的事务工厂,这个工厂的作用就是来创建Transaction事务对象。
			- 从已有的连接中创建Transaction对象
			- 根据数据源,数据库隔离级别,自动提交创建Transaction对象
		- 获取SqlSession的时候,会根据事务工厂创建一个Transaction事务对象,根据这个对象,再创建具体的Executor,最终创建出创建SqlSession对象。

### mybatis关联查询
	- 两种方式
		- 查询嵌套,返回结果赋值给主对象 分多个sql查询,再把查询结果汇总
		- 嵌套查询,一个sql关联多张表

### mybatis延迟加载
	- 按需查询,在需要的时候进行查询
	- 使用CGLIB创建目标的代理对象，当调用目标方法时，进入拦截器方法
	- 支持association关联对象和collection关联集合对象的延迟加载，association指的是一对一collection 值的就是一对多查询

### Mybatis的插件
	- 插件可拦截的四种对象类型
		- ParameterHandler : 对sql参数进行处理
		- ResultSetHandler : 对结果集对象进行处理
		- StatementHandler : 对sql语句进行处理
		- Executor : 执行器，执行增删改查

	- 实现原理
		责任链模式 + 动态代理

	- 加载流程
		- 编写插件类,指定要拦截的类型和方法
		- xml配置插件
		- XMLConfigBuilder加载插件
			- 读取plugins标签下的插件,并加载Configuration中的InterceptorChain中。
				InterceptorChain 是一个interceptor集合,相当于是一层层包装,后一个插件就是对前一个插件的包装,并返回一个代理对象(责任链)
		- 根据插件配置要拦截的对象和方法创建插件对象
			1. 获取拦截器拦截的方法，以拦截对象为key,拦截方法集合为value
			2. 获取目标对象的class，比如Executor对象
			3. 如果拦截器中拦截的对象包含目标对象实现的接口，则返回拦截的接口
			4. 创建代理类Plugin对象，Plugin实现了InvocationHandler接口，最终对目标对象的调用都会调用Plugin的invoke方法。




### #{}和${}
	- #{}是预编译处理,${}是字符串替换。
	- MyBatis在处理#{}时，会将SQL中的#{}替换为?号，使用PreparedStatement的set方法来赋值,MyBatis在处理 ${}时，就是把${}替换成变量的值。
	- 使用#{}可以有效的防止SQL注入,提高系统安全性
	- 占位符
		- 占位符就是在sql语句中用？代替变量

### openSessionFromDataSource 执行流程
	1. 从获取configuration中获取Environment对象，Environment包含了数据库配置
	2. 从Environment获取DataSource数据源
	3. 从DataSource数据源中获取Connection连接对象
	4. 从DataSource数据源中获取TransactionFactory事物工厂
	5. 从TransactionFactory中创建事物Transaction对象
	6. 创建Executor对象
	7. 包装configuration和Executor对象成DefaultSqlSession对象

### Mybatis面试题


### Mybatis源码分析
http://www.songshuiyang.com/categories/Mybatis/
	


