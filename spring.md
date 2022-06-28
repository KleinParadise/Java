### FactoryBean与BeanFactory的区别
	FactoryBean是一个Bean
		- Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean,通过FactoryBean的工厂类接口,用户可以通过实现该接口定制实例化Bean的逻辑,不需要复杂的配置
		- 即通过实现该结构,可通过编码的方式得到一个增强Bean
	BeanFactory是一个Factory
		- 是Spring里面最底层的接口,包含了各种Bean的定义,读取bean配置文档,管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。
		- ApplicationContext是BeanFactory的子接口

### ApplicationContext与BeanFactory的区别
	- ApplicationContext比BeanFactory多的功能
		1. 继承MessageSource,支持国际化
		2. 实现了ResourceLoader接口,使用getResource()方法实现统一的资源文件访问方式
		3. 同时加载多个配置文件
		4. 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次
	- 加载Bean方式
		- BeanFactory延迟加载,ApplicationContext启动后预载入所有的单实例Bean
	- 创建方式
		- BeanFactory编程方式创建,ApplicationContext还可通过声明方式创建
	-  对BeanPostProcessor、BeanFactoryPostProcessor的支持
		- BeanFactory需要手动注册才能使用,ApplicationContext配置对应的实现类,自动注册即可使用

### ApplicationContext接口的实现类
	- FileSystemXmlApplicationContext 此容器从一个XML文件中加载beans的定义
	- ClassPathXmlApplicationContext 此容器也从一个XML文件中加载beans的定义,该容器将在classpath里找bean配置
	- WebXmlApplicationContext：此容器加载一个XML文件，此文件定义了一个WEB应用的所有bean

### BeanPostProcessor、BeanFactoryPostProcessor的理解
	-Spring初始化bean时对外暴露的扩展点,即SpringBean后置处理器接口
	- 区别
		- BeanFactoryPostProcessor作用于bean实例化之前,读取配置元数据,并且可以修改
		- BeanPostProcessor作用于bean的实例化过程中进行bean实例的代理封装


### SpringBean的理解
	- Spring IoC容器管理的对象称为bean
	- bean是一个由Spring IoC容器实例化、组装和管理的对象
	- bean规范
		1. 所有属性为private
		2. 提供默认构造方法
		3. 提供getter和setter
		4. 实现serializable接口

### 容器启动阶段,对SpringBean的处理
	- 配置元信息
		- xml配置需要spring管理bean的信息
	- BeanDefination
		- xml配置信息转为BeanDefination对象
	- BeanDefinationReader
		- BeanDefinationReader的作用就是加载配置元信息，并将其转化为内存形式的BeanDefination
	- BeanDefinationRegistry
		- Beanid与BeanDefination的Map缓存
	- BeanFactoryPostProcessor
		- 负责对注册到BeanDefinationRegistry中的一个个的BeanDefination进行一定程度上的修改与替换

### SpringBean的生命周期
	- 实例化
		- 获取Bean时或容器启动时对Bean进行实例化
	- ioc 依赖注入
		- 根据spring上下文对实例化的Bean进行配置,即ioc注入
	- setBeanName实现
		- 如果该Bean实现了BeanNameAware接口,传递的就是Spring配置文件中Bean的id值,即给Bean设置的别名
	- BeanFactoryAware实现
		- 如果该Bean实现了BeanFactoryAware接口,使该Bean可引用Spring工厂(BeanFactory)自身,通过BeanFactory获取其他Bean
	- ApplicationContextAware实现
		- 如果该Bean实现了ApplicationContextAware接口,使该Bean可引用Spring上下文(ApplicationContext),使该bean能使用spring容器的方法
	- postProcessBeforeInitialization接口实现
		- Bean初始化之前调用
	- init-method
		- 如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法
	- postProcessAfterInitialization
		- Bean初始化之后调用
	- 销毁
		- 如果Bean实现了DisposableBean这个接口,会调用那个其实现的destroy()方法
	- destroy-method自配置清理
		- 如果Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法

### SpringBean的作用域
	- singleton 单例
		- springioc容器中只会存在一个共享的Bean实例,无论多少个bean引用它,始终指向同一对象
		- 多线程下是不安全的,spring默认作用域为单例
	- prototype 原型
		- 每次通过Spring容器获取prototype定义的bean时,容器都将创建一个新的Bean实例,每个Bean实例都有自己的属性和状态
	- request 
		- 一次request一个实例
		- 在一次Http请求中,容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效,当前Http请求结束,该bean实例也将会被销毁
	- session
		- 在一次Http Session中,容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例,该bean实例仅在当前Session内有效。同Http请求相同,每一次session 请求创建新的实例,而不同的实例之间不共享属性,且实例仅在自己的session请求内有效,请求结束,则实例将被销
	- global session
		- 在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用 portlet context时有效

### Spring线程的理解
	- 每个单例的无状态对象都是线程安全的
		- 无状态的对象即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题
	- Spring根本就没有对bean的多线程安全问题做出任何保证与措施,对于每个bean的线程安全问题，根本原因是每个bean自身的设计
	- 不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了

### SpringBean注入方式
	- set方法注入
		- 通过调用bean的set方法为属性注入值
		- <property name="car" ref="myCar" />
	- 构造器注入
		- 通过调用bean所属类的带参构造器为bean的属性注入值即为类提供包含参数的构造方法
		- <constructor-arg name="car" ref="myCar" /> constructor-arg的name是构造器参数的名称。
	- 静态工厂注入
		- 编写一个静态的工厂方法,这个工厂方法会返回一个我们需要的值,然后在配置文件中,我们指定使用这个工厂方法创建bean
		- <bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>
	- 实例工厂注入
		- 声明实例工厂bean，Spring容器需要先创建一个SimpleFactory对象，才能调用工厂方法
		- <bean id="factory" class="cn.tewuyiang.factory.SimpleFactory" />
		- 通过实例工厂的工厂方法，创建三个bean，通过factory-bean指定工厂对象,通过factory-method指定需要调用的工厂方法
		- <bean id="name" factory-bean="factory" factory-method="getName" />
	- 注解注入
		- 用@Autowired或者@Resource这两个注解进行依赖注入
		- 基本数据类型或者是Java的封装类型,使用@Value注解

### @Autowired与@Resource两注解的区别
	- @Autowired注解是按类型装配依赖对象,默认情况下它要求依赖对象必须存在,如果允许null值,可以设置它required属性为false。
	- @Resource注解和@Autowired一样,也可以标注在字段或属性的setter方法上,但它默认按名称装配。名称可以通过@Resource的name属性指定,如果没有指定name属性,当注解标注在字段上,即默认取字段的名称作为bean名称寻找依赖对象,当注解标注在属性的setter方法上,即默认取属性名作为bean名称寻找依赖对象

### @Component和@Bean的区别
	- @Component作用于类,通过类路径扫描来自动侦测和装配对象到Spring容器中
	- Spring的@Bean注解用于告诉方法,产生一个Bean对象,然后这个Bean对象交给Spring管理,产生这个Bean对象的方法Spring只会调用一次。

### @Autowired注解自动装配的过程
	- 开启注解,使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />
	- 扫描查询：在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean

### Spring如何处理Bean的循环依赖
	- 完整的对象包含两部分:当前对象实例化和对象属性实例化
	- 在Spring中，对象的实例化是通过反射实现的，而对象的属性则是在对象实例化之后通过一定的方式设置的
	- Spring实例化bean是通过ApplicationContext.getBean()方法来进行的
	- 假如A B两个对象互相依赖
		- 首先Spring尝试通过ApplicationContext.getBean()方法获取A对象的实例，由于Spring容器中还没有A对象实例，因而其会创建一个A对象
		- 发现其依赖了B对象，因而会尝试递归的通过ApplicationContext.getBean()方法获取B对象的实例
		- Spring容器中此时也没有B对象的实例，因而其还是会先创建一个B对象的实例
		- 此时A对象和B对象都已经创建了，并且保存在Spring容器中了，只不过A对象的属性b和B对象的属性a都还没有设置进去。
		- Spring创建B对象之后，Spring发现B对象依赖了属性A，因而还是会尝试递归的调用ApplicationContext.getBean()方法获取A对象的实例
		- Spring中已经有一个A对象的实例，虽然只是半成品（其属性b还未初始化），但其也还是目标bean，因而会将该A对象的实例返回。
		- B对象的属性a就设置进去了，然后还是ApplicationContext.getBean()方法递归的返回，也就是将B对象的实例返回，此时就会将该实例设置到A对象的属性b中

### SpringIoc的理解
	- 对于spring框架来说,就是由spring来负责控制对象的生命周期和对象间的关系
	- 实现原理
		- 工厂模式 + 反射机制(在工厂方法中通过反射创建类实例)
			- 使用反射技术获取对象的信息包括：类信息、成员、方法等等
			- 再通过 xml 配置 或者 注解 的方式，说明依赖关系
			- 在调用类需要使用其他类的时候，不再通过调用类自己实现，而是通过 IoC 容器进行注入

### SpringIoc的流程
	- 容器构建启动入口
		- 在web.xml中配置ContextLoaderListener监听器，当Tomcat启动时，会触发ContextLoaderListener的contextInitialized方法，从而开始 IoC 的构建流程
		- 配置contextConfigLocation，于指定Spring配置文件(appcontext-*.xml)的路径

	- ApplicationContext刷新前配置
		- 确认要使用的容器(XmlWebApplicationContext or AnnotationConfigApplicationContext)都继承AbstractApplicationContext，核心逻辑也都是在
		  AbstractApplicationContext中实现。
		- 如果定义ApplicationContextInitializer接口的实现类,在initialize方法中进行自己的逻辑操作，例如：添加监听器、添加BeanFactoryPostProcessor。
			- 定义ApplicationContextInitializer接口的实现类
			- 在web.xml中,配置改实现类

	- 初始化 BeanFactory、加载 Bean 定义
		- 创建一个新的 BeanFactory，默认为 DefaultListableBeanFactory
		- 根据 web.xml 中 contextConfigLocation 配置的路径，读取 Spring 配置文件，并封装成Resource
		- 根据 Resource 加载 XML 配置文件，并解析成Document对象
		- 从根节点开始，遍历解析Document中的节点
			- 对于默认命名空间的节点：先将bean节点内容解析封装成BeanDefinition，然后将beanName、BeanDefinition 放到BeanFactory的缓存中，用于后续创建bean实例时使用
			- 对于自定义命名空间的节点：会拿到自定义命名空间对应的解析器，对节点进行解析处理。
				- 如：<context:component-scan base-package="com.joonwhee" /> ，该节点对应的解析器会扫描 base-package 指定路径下的所有类，将使用了 @Component（@Controller、@Service、@Repository）注解的类封装成 BeanDefinition，然后将 beanName、BeanDefinition 放到 BeanFactory 的缓存中，用于后续创建 Bean 实例时使用

	- 触发 BeanFactoryPostProcessor
		- 实例化和调用所有 BeanFactoryPostProcessor，包括其子类 BeanDefinitionRegistryPostProcessor。
		- Spring IoC 容器允许 BeanFactoryPostProcessor 在容器实例化任何 bean 之前读取 bean 的定义，并可以修改它

	- 注册 BeanPostProcessor
		- 注册所有的 BeanPostProcessor，将所有实现了BeanPostProcessor接口的类加载到BeanFactory中
		- Spring IoC 容器允许 BeanPostProcessor 在容器初始化 bean 的前后，添加自己的逻辑处理。在这边只是注册到 BeanFactory 中，具体调用是在 bean 初始化的时候

	- 实例化所有剩余的非懒加载单例 bean
		- 遍历所有被加载到缓存中的 beanName，触发所有剩余的非懒加载单例 bean 的实例化。
		- 首先通过 beanName 尝试从缓存中获取，如果存在则跳过实例化过程；否则，进行 bean 的实例化。
		- 根据 BeanDefinition，使用构造函数创建 bean 实例。
		- 根据 BeanDefinition，进行bean实例属性填充。
		- 执行 bean 实例的初始化
			- 触发 Aware 方法
			- 触发 BeanPostProcessor 的 postProcessBeforeInitialization 方法
			- 如果 bean 实现了 InitializingBean 接口，则触发 afterPropertiesSet() 方法
			- 如果 bean 设置了 init-method 属性，则触发 init-method 指定的方法
			- 触发 BeanPostProcessor 的 postProcessAfterInitialization 方法
		- 将创建好的 bean 实例放到缓存中，用于之后使用

	- 完成上下文的刷新
		- 使用应用事件广播器推送上下文刷新完毕事件（ContextRefreshedEvent ）到相应的监听器。
			- 如何监听
				- 创建一个自定义监听器，实现 ApplicationListener 接口，监听ContextRefreshedEven
				- 将该监听器注册到Spring IoC容器即可

### 依赖注入(DI)的理解
	- 在系统运行中,动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的

### SpringAOP的理解
	- 在运行时,动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程
	- AOP两种生成代理对象的方式
		- 默认的策略是如果目标类是接口,则使用JDK动态代理技术,否则使用Cglib来生成代理。
		- JDKProxy
			- 只能为接口创建代理实例
		- Cglib
			- 通过继承,子类成为其代理对象
	- 核心概念
		- joinpoint 需要对那些类,那些方法增加
		- pointcut 提供一组规则来匹配joinpoint,给满足规则的joinpoint添加advice.
		- Aspect 切面 是Pointcut和Advice的合集
		- advice 执行增强的代码(前置、后置、异常、最终、环绕)
	- 应用场景
		- 拦截器(在方法执行前后加入权限判断)
		- 事务(方法开始之前开启事务,方法执行完毕关闭事务)
		- 日志(在一些方法开始之前或者开始之后加入日志)
		- 缓存(在查询数据库方法之前织入查询缓存的代码)

### SpringAOP的流程
	- AnnotationAwareAspectJAutoProxyCreator 是AOP核心处理类,实现了BeanProcessor
	- postProcessAfterInitialization是AnnotationAwareAspectJAutoProxyCreator的核心方法
		- getAdvicesAndAdvisorsForBean 获取当前bean匹配的增强器
			a. 找所有增强器，也就是所有 @Aspect 注解的 Bean
			b. 找匹配的增强器，也就是根据 @Before，@After 等注解上的表达式，与当前 bean 进行匹配，暴露匹配上的。
			c. 对匹配的增强器进行扩展和排序，就是按照 @Order 或者 PriorityOrdered 的 getOrder 的数据值进行排序，越小的越靠前
		- createProxy 为当前bean创建代理
			a. 如果设置了 proxyTargetClass=true，一定是 CGLIB 代理
			b. 如果 proxyTargetClass=false，目标对象实现了接口，走 JDK 代理
			c. 如果没有实现接口，走 CGLIB 代理

	- 代理如何调用增强
		- Spring 把Target object(目标对象)和Advice(通知/增强)
		  编织在一起后，再调用代理对象的方法，代理对象就会把调用再次处理，匹配的通知和方法按顺序执行；不匹配的方法，则不会执行通知，而是只执行方法本身
		- 因为advisors中的拦截器，只能说一定是匹配当前方法所属的类以及类中的某些方法，但并不一定匹配当前的方法

### Spring框架中用到了哪些设计模式
	- 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例
	- 单例模式：Bean默认为单例模式
	- 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术
	- 模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplat
	- 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现–ApplicationListener

### 品spring
	- BeanDefinition bean定义的接口,是一种数据结构,记录了一个bean的全部信息
	- 两种方法指定一个bean定义
		- 类名称
		- 工厂方法
	- spring提供了BeanDefinitionRegistryPostProcessor接口为其他类注册bean定义接口
		- ConfigurationClassPostProcessor 包含了所有Spring支持的注册bean定义方式的实现
		- spring统一了编程模型,所以ConfigurationClassPostProcessor通过手动方式注册到spring框架中
	- BeanFactoryPostProcessor为BeanDefinitionRegistryPostProcessor接口的父接口
		- 这个接口的目的就是再给一次修改（或完善）bean定义的机会。当这个接口执行完后，所有的bean定义就真的确定下来了，不会再变了

	- BeanFactoryPostProcessor与BeanDefinitionRegistryPostProcessor
		- 这两个接口用来为其它类注册bean定义或修改已注册好的bean定义的
		- 这两个接口的实现类也会被注册bean定义，只不过在时间轴上会非常的靠前
		- 这两个接口可以有多个实现类，可以通过实现排序接口进行排序，以保证逻辑上先后顺序的正确性

	- Spring支持的bean定义的注册方式
		- 注解注册bean定义
			- 标有@Component注解的类
			- 标有@Configuration注解的类
			- 标有@Configuration注解的类里标有@Bean注解的方法返回的类
		- 使用XML配置注册bean定义
			- 在标有@Configuration注解的类上，使用@ImportResource注解引入XML配置文件
		- 使用@Import注解引入
			- 在标有@Configuration注解的类上，使用@Import注解引入其它标有@Configuration注解的类
		- 使用ImportSelector接口注册bean定义
			- 接口方法返回的是类名称的数组，这些类将会被注册bean定义。在实现这个接口时，可以根据不同的情况返回不同的类名称，这就非常的灵活，实现了动态注册bean定义的需求。
		- 使用ImportBeanDefinitionRegistrar接口注册bean定义
			- 这个接口方法没有返回值，所以需要自己写代码直接注册bean定义

	- spring没有通过反射获取类(bean)信息的原因？
		- 性能损耗,使用反射需要加载类,生成class对象,然后缓存这些对象,耗费时间和内存
		- 信息完整度的问题(反射提供的信息并不完整)

	- spring直接通过asm框架从字节码文件获取类信息
		- 通过字节码访问不到注解的默认属性值
		- 字节码只能访问到类型自己定义的信息，从父类型继承的信息也是访问不到的
		- 这两个问题需要spring框架解决

	- Spring获取所有注解信息的方法
		- 反射 + 递归

	— Spring对注解的增强
		- 注解就是一个接口（多了个@而已），注解的属性就是接口方法，而且方法必须是无参的，返回值必须不能为void,这些方法就相当于“getter”方法
		- @AliasFor,建立属性映射关系,使注解从下向上传递数据
			- 注解其实是一个接口,正好采用JDK动态代理,在代理类内部,拦截住正在调用的方法,插入处理别名的逻辑即可。
			- 从下向上查找属性,从上往下赋值属性,递归
			- 把双方都列出来,再定义一个类,建立映射关系

	- Spring与SpringBoot
		- 传统Spring构建的web应用,会打成一个war包,放入tomcat下面。先启动tomcat，然后tomcat再去加载它下面的web应用（即war包）
		- SpringBoot构建的web应用,会打成一个jar包,采用内嵌的tomcat,先启动jar包，会进入SpringBoot中，然后再去启动tomcat,要自己实现一个webApplicationContext所使用的类
		- SpringBoot和Spring重叠的部分,其实本质没啥区别
		- SpringBoot新增的内容
			- SpringBoot的启动方式是把自身提前把web服务器移后
			- SpringBoot采用根据条件（condition）自动配置的方式（AutoConfiguration）

	- Bean定义注册的两个实现类
		- AnnotatedBeanDefinitionReader
			- 字段
				- BeanDefinitionRegistry，这是容器（或bean factory）会实现的接口,用于把一个BeanDefinition（bean定义）注册到容器中
				- BeanNameGenerator,注册bean定义时需要一个bean名称（即beanName）
				- ScopeMetadataResolver，是来决定bean实例的范围（即生命周期）的
				- ConditionEvaluator，条件计算器，根据“条件”判断一个bean定义该不该被注册

			- 方法(doRegisterBean)
				- 参数
					- annotatedClass，是Class<?>，表示要被注册的类
					- instanceSupplier,可以提供这个bean的实例对象，这样就不再需要通过反射调用构造函数了。
					- name,bean名称，如果传的话就不用再生成了
					- qualifiers，是一组用作限定修饰符的注解
					- definitionCustomizers，是一组可以自定义bean定义的接口，BeanDefinitionCustomizer
				- 流程
					- 先把类转变为bean定义，即把Class<?>转变为BeanDefinition
					- 使用条件计算器来确定是否要注册这个bean定义
					- 确定这个bean的生命周期
					- 确定这个bean的名称
					- 处理定义的公共注解信息
					- 处理限定修饰符，就是@Primary、@Lazy、@Qualifier这三个注解
					- 应用bean定义自定义器，对bean定义进行一些自定义
					- 根据bean的生命周期，使用AOP技术为该bean定义生成代理
					- 把这个bean定义注册到容器中

		- ClassPathBeanDefinitionScanner
			- 与AnnotatedBeanDefinitionReader整体逻辑一样,只是获取bean定义的方式不同
			- 收集bean定义的过程
				- 拼接资源路径(如classpath*:org/cnt/ts/**/*.class表示搜索类路径下所有的jar包里，以org/cnt/ts开头的包及其子包里的所有.class文件)
				- 找出上一步中的那些.class文件，并把它们转化为资源，即Resource类
				- 使用ASM框架逐个读取这些资源（其实就是字节码文件啦）
				- 应用过滤器和条件计算器，来确定这个bean定义是否要被注册
				- 使用从字节码中读出的内容来构建BeanDefinition，使用的是ScannedGenericBeanDefinition这个类
				- 确认下这个类是否符合要求,独立的、具体的、如果类是抽象的，它必须包含一个标有@Lookup注解的方法，来指定一个具体的bean
				- 收集好这个bean定义
		- 以上两个类并不处理@Bean这个注解注册的bean定义，也不处理由@Import注解引入的bean定义

	- SpringBoot bean如何通过两个阶段来注册所有bean定义的
		- 在Spring容器启动前,只有应用的主类自己被注册了bean定义
		- 通过调用ConfigurationClassPostProcessor这个一个bean工厂后处理器(专门用于注册bean定义的后处理器),对已经注册的bean定义即主类,判断它是否需要被处理
			- 判断条件
				- 标有@Configuration、@Component、@ComponentScan,@Import,@ImportResource,@Bean注解
				- 当标有这些注解的bean定义被注册成功之后,又会触发bean工厂后处理器,对该bean上符合条件的注解递归调用注册(再来一瓶)
			- 注册步骤(依附于@Configuration类的)
				- 如果类上标有@Component注解，就去处理它的静态内部（嵌套）类
				- 处理@PropertySource注解，它可以引入.properties文件，会把文件中的属性值放入到Environment中
				- 处理@ComponentScan注解，它会扫描指定的jar包，并从中获取bean定义
				- 处理@Import注解,可引入三类内容
					- 另一个普通类，但是当作@Configuration类
					- ImportSelector接口的实现类
					- ImportBeanDefinitionRegistrar接口的实现类
				- 处理@ImportResource注解，它用于引入.xml文件，可以使xml和注解两种方式混合使用
				- 处理类中的@Bean方法
				- 处理接口里面的默认方法，且方法上有@Bean注解
				- 处理父类
	- 注解递归
		- @Configuration类引入了@EnableXXX注解或@Import注解，@EnableXXX注解又引入了@Import注解,@Import注解可以再次引入@Configuration类,实现闭环
		- @Import注解(用于Spring框架自身的开发，以及第三方框架与Spring的整合开发)
			- @Import注解引入的类是实现了ImportSelector、ImportBeanDefinitionRegistrar接口
			- @Import注解引入被@Configuration注解标注的类或没有被注解标注的普通类
		- bean定义通过注解注册的过程
			- 从容器中拿出尚未处理过的所有@Configuration类
			- 从这一批@Configuration类中解析出所有需要被注册的类
			- 将这一批@Configuration类标记为已处理过
			- 将刚刚解析出来的类进行bean定义注册，这些类里面可能还含有@Configuration类
			- 循环执行
		- @ComponentScan注解是按字节码文件扫描的,所以此时内部类都会被注册,即使外部类上没有注解
		- @Import引入一个类时,不管该类上是否有注解,该类本身一定会被注册bean定义,所以外部类总是被注册.

	- 两个bean定义“后处理器”接口，
		- BeanDefinitionRegistryPostProcessor
			- 用来向容器中注册bean定义的，造成的结果就是bean定义的数目变多
		- BeanFactoryPostProcessor
			- 用来修改容器中的bean定义的，造成的结果就是bean定义的数目不变

	- bean的创建过程中五个后处理器
		- 指定bean初始化的三种方式
			- 实现Spring提供的一个接口，InitializingBean，它只有一个方法，就是afterPropertiesSet
			- 使用@Bean注解注册bean定义时，设置注解的initMethod属性为bean的一个方法名
			- 使用java的注解@PostConstruct，把它标在bean的一个方法上

		- 指定bean销毁的三种方式
			- 实现Spring提供的一个接口，DisposableBean，它只有一个方法destroy
			- 使用@Bean注解注册bean定义时，设置注解的destroyMethod属性为bean的一个方法名
			- 使用java的注解@PreDestroy，把它标在bean的一个方法上


		- BeanPostProcessor(接口)
			- BeforeInitialization bean初始化之前
			- AfterInitialization  bean初始化之后
			- bean的实例化-> bean的依赖装配 -> BeforeInitialization（初始化前） -> bean的初始化方法 -> AfterInitialization（初始化后） -> ...

		- InstantiationAwareBeanPostProcessor(参与到bean的实例化过程中)
			- BeforeInstantiation
			- AfterInstantiation
			- Properties
			- bean的实例化准备阶段 -> BeforeInstantiation（实例化前）-> bean的实例化 -> AfterInstantiation（实例化后） -> Properties（定制bean所需的属性值） -> bean的属性设置 -> ...

		- SmartInstantiationAwareBeanPostProcessor
			- predictBeanType 用来预测最终的bean类型,这是给我们提供一个修改bean类型的机会,方法的参数是原始的bean类型
			- determineConstructors 用来确定候选的构造方法，给我们一个定制构造方法的机会，方法的参数是原始的bean类型
			- getEarlyBeanReference 用来获取一个早期bean实例的引用,bean实例的初始化方法还没有执行(用来解决循环引用)

		- DestructionAwareBeanPostProcessor
			- postProcessBeforeDestruction 在bean实例销毁前会被调用，来执行一些定制的销毁代码
			- requiresDestruction 就是决定是否要为bean实例调用postProcessBeforeDestruction方法来执行一些销毁代码

		- MergedBeanDefinitionPostProcessor
			- 用来进行一些自省操作，如一些检测，或在处理bean实例之前缓存一些相关的元数据

	- @PostConstruct与@PreDestroy
		- @PostConstruct注解在方法上,表示此方法是在Spring实例化该Bean之后马上执行此方法,之后才会去实例化其他Bean,并且一个Bean中@PostConstruct注解的方法可以有多个
		- @PreDestroy即bean销毁时指定的调用方法
		- 原理步骤
			- 使用反射在一个类里面找出所有标有注解(@PostConstruct和@PreDestroy)的方法，就是Method类型的一个对象
			- 将找出的方法与类关联起来
				- 传进来的初始化或销毁方法存在重复或冗余,需要进行一些处理,然后变为Set类型,把冗余的过滤掉
		    - 将标有注解的方法进行处理
		    	- 标有初始化注解的方法是父类的方法在前面,子类的方法在后面。标有销毁注解的方法正好倒过来,子类的在前,父类的在后
		    	- 对于私有方法,identifier字段的值就是方法全名,因为私有方法不能被继续和重写,子类里和父类里定义的同名私有方法,也是不同的两个方法
		    	- 非私有方法,identifier字段的值就是方法的简单名，因为非私有方法可以被继续和重写
		    	- 子类里和父类里定义的同名非私有方法，虽然也是不同的两个方法，但是它们以反射的方式在子类对象上调用时产生的结果是一样的，都等同于调用子类上的方法,
		    	  此时只需保留一个就可以了,使用方法的简单名,通过Set时就可以过滤掉一个,实际保留的是父类的,过滤掉的是子类的
		    - 对找出来的注解方法进行检查
		    	- 私有同名方法都会被保留,非私有同名方法只会保留一个,由于顺序的原因,初始化方法保留的是父类中的,销毁方法保留的是子类中的
		    - 通过反射来调用这些被注解的方法
		    - 使用bean后处理器决定调用时机
		    	- postProcessMergedBeanDefinition
		    		- 所有的初始化和销毁方法都找出来，整理好并缓存起来备用。其实这些就相当于元数据的处理
		    	- postProcessBeforeInitialization
		    		- 完成对所有初始化方法的按顺序调用
		    	- postProcessBeforeDestruction
		    		- 完成对所有销毁方法的按顺序调用

	- @Resource注解
		- 它用在字段和方法上,对应于Spring的注入
		- 依赖注入三步
			- 找出需要被注入的元素,即标了注解的字段或方法
			- 根据注解的描述,在容器中找出依赖的bean
			- 完成注入,即设置字段的值或进行方法调用

		- 被注入的元素(字段和方法)在spring中的表示
			- InjectedElement类,来表示被注入的元素
			- InjectionMetadata类,来表示一个类的注入元数据
			- 读取注解@Resource的属性，为注入元素类的字段赋值

		- 如何找出依赖的名称和类型
			- 首先获取到这个注解，然后读出它的name和type属性值
			- 如果是字段就用字段名，如果是普通方法就用方法名，如果是setter方法就用属性名
			- 如果type的值是Object.class，则表明没有显式指定类型，那就读取被注入元素的类型
			- 如果是字段就用字段类型，如果是普通方法就用第一个参数的类型，如果是setter方法就用属性类型

		- 对依赖的描述
			- InjectionPoint类,注入点类
			- DependencyDescriptor类，继承了注入点类,

		- 获取依赖的逻辑
			- 如果使用的是默认名称，且容器中不包含这个名称的bean，则按照类型去解析依赖
			- 显式指定了名称或容器中包含这个名称的bean，则按照名称去解析依赖 

		- 与bean后处理器结合
			- postProcessMergedBeanDefinition这个方法里完成注入元数据的获取与缓存
			- postProcessProperties这个方法里完成依赖的注入


### Spring启动流程

### spring面试补漏
	


### SpringMVC框架原理 
	- 客户端请求提交到DispatcherServlet
	- 由DispatcherServlet控制器查询一个或多个HandlerMapping,找到处理请求的Controller
	- DispatcherServlet将请求提交到Controller
	- Controller调用业务逻辑处理后,返回ModelAndView
	- DispatcherServlet查询一个或多个ViewResoler视图解析器,找到ModelAndView指定的视图
	- Http响应:视图负责将结果显示到客户端

### Dispatcher的流程
	1.从HandlerMapping中获取HandlerExecutionChain对象
	HandlerExecutionChain handlerExecutionChain = HandlerMapping.getHandler(request);
	2.请求前处理
	handlerExecutionChain.applyPreHandle(request,response);
	3.从handlerExecutionChain获取实际处理器
	Object handler = handlerExecutionChain.getHandler();
	4.获取处理器的适配器
	HandlerAdapter handlerAdapter = getHandlerAdapter(handler);
	5.调用适配器处理请求,返回一个'ModeAndView'对象
	ModeAndView mv = handlerAdapter.handle(request,response,handler)
	6.请求后处理
	handlerExecutionChain.applyPostHandle(request,response,mv);



### DispatcherServlet的理解
	- DispatcherServlet实现了Servlet这个接口,来自前端的请求会先到达这里,它负责到后台去匹配合适的handler

### HandlerMapping的理解
	- Map<Url,Handler>,存储了URL和具体哪一个Handler来处理这个URL的请求
	- 根据请求信息,匹配出处理该请求Controller或者Method对象,并将其包装成HandlerExecutionChain对象

### HandlerAdapter的理解
	- HandlerAdapter的职责是让固定的servlet处理方法调用handler来进行处理,在springmvc项目中，可以把Controller中的一个处理请求的方法，看成是一个Handler，而这个时候Controller类就可以理解为一个HandlerAdapter

	- SimpleServletHandlerAdapter
		- 对handler的处理是调用Servlet的service方法处理请求
	- SimpleControllerHandlerAdapter
		- 对handler的处理是调用Controller的handleRequest()方法
	- HttpRequestHandlerAdapter
		- 对handler的处理是调用HttpRequestHandler的handleRequest()方法
	- RequestMappingHandlerAdapter
		- 对handler的处理是调用通过java反射调用HandlerMethod的方法
	- 将HandlerExecutionChain中handler与上面四种类型做匹配并调用对应的方法

### ModelAndView的理解
	- 封装处理的数据和指定显示的界面

### ViewResolver的理解
	- 对ModeAndView指定的视图进行解析获取对应的视图最终呈现给浏览器进行渲染

### ContextLoaderListener的理解
	- ContextLoaderListener继承自ContextLoader,实现的是ServletContextListener接口。
	- 启动Web容器时,自动装配ApplicationContext的配置信息，并产生applicationcontext对象，然后将这个对象放置在ServletContext的域里
	  通过ServletContext就可以得到共享的applicationcontext对象,并利用这个对象访问spring容器管理的bean

### Interceptor拦截器的理解
	- HandlerInterceptor
		- preHandle()方法：该方法在执行控制器方法之前执行。
		- postHandle()方法：该方法在执行控制器方法调用之后，且在返回ModelAndView之前执行。
		- afterCompletion()方法：该方法在执行完控制器之后执行，由于是在Controller方法执行完毕后执行该方法，所以该方法适合进行一些资源清理，记录日志信息等处理操作。
	- WebRequestInterceptor
		- preHandle(WebRequest request) ：该方法不能用于请求阻断，一般用于资源准备。
		- postHandle(WebRequest request, ModelMap model)：preHandle 中准备的数据都可以通过参数WebRequest访问。ModelMap 是Controller 处理之后返回的Model 对象，可以通过改变它的属性来改变Model 对象模型，达到改变视图渲染效果的目的
		- afterCompletion(WebRequest request, Exception ex) ：。Exception 参数表示的是当前请求的异常对象，如果Controller 抛出的异常已经被处理过，则Exception对象为null 
	- 两接口的区别
		- HandlerInterceptor通过在preHandle方法中返回false来禁止执行处理程序
		- WebRequestInterceptor一般用于准备上下文资源,

### Spring、SpringMVC及web容器的Context上下文
 	- ServletContext web容器上下文环境,为后面的spring IoC容器提供宿主环境
 	- contextLoaderListener会监听web容器启动事件spring会初始化一个启动上下文,这个上下文被称为根上下文WebApplicationContext即spring的IoC容器
 	- 在contextLoaderListener监听器初始化完毕后，开始初始化web.xml中配置的Servlet，这个servlet可以配置多个，
 	  以DispatcherServlet为例，这个servlet实际上是一个标准的前端控制器，用以转发、处理每个servlet请求。DispatcherServlet上下文在初始化的时候会建立自己的IoC上下文，用以持有spring mvc相关的bean。每个servlet就持有自己的上下文，即拥有自己独立的bean空间，同时各个servlet共享相同的bean，即根上下文(WebApplicationContext)。

 	- ContextLoaderListener中创建ApplicationContext主要用于整个Web应用程序需要共享的一些组件，比如DAO，数据库的ConnectionFactory等。
 	  而由DispatcherServlet创建的ApplicationContext主要用于和该Servlet相关的一些组件，比如Controller、ViewResovler等

### SpringMvc中Filter的使用


### SpringMvcInterceptor的理解与Filter的区别(拦截器与过滤器的区别)
	- 拦截器是基于java的反射机制的，而过滤器是基于函数回调
	- 拦截器不依赖与servlet容器，过滤器依赖与servlet容器
	- 拦截器是一个Spring的组件,归Spring管理,配置在Spring文件中,因此能使用Spring里的任何资源.过滤器不能够使用Spring容器资源,且定义在web.xml中

### Servlet的理解,生命周期
	- Servlet是一个接口,定义的是一套处理网络请求的规范
	- 实现Servlet接口的类(即处理网络请求的类),要处这三个方法
		- init 初始化要做啥
		- destroy 销毁的时候做啥
		- service() 处理请求的逻辑
	- 生命周期
		- 实例化——new：服务器第一次被访问时，加载一个Servlet容器，而且只会被加载一次。
		- 初始化——init：创建完Servlet容器后，会调用仅执行一次的init()初始化方法，用于初始化Servlet对象，无论多少台客户端在服务器运行期间访问都不会再执行init()方法
		- 执行处理——service()方法
		- 销毁——destroy：在服务器关闭或重启时，Servlet会调用destroy方法来销毁，将Servlet容器标记为垃圾文件，让GC做回收处理。
	- 如何处理请求
		1. 客户端发送请求给服务器
		2. 容器根据请求及web.xml判断对应的Servlet是否存在，如果不存在则返回404
		3. 容器根据请求及web.xml判断对应的Servlet是否已经被实例化，若是相应的Servlet没有被实例化，则容器将会加载相应的Servlet到Java虚拟机并实例化
		4. 调用实例对象的service()方法，并开启一个新的线程去执行相关处理。调用servce方法，判断是调用doGet方法还是doPost方法
		5. 业务完成后响应相关的页面发送给客户端

	- doGet与doPost的区别
		- 1. 通过get方式提交的数据有大小的限制,通常在1024字节左右。post方式没有数据大小的限制
		- 2. get传递数据明文显示,post传递数据是通过http请求的附件进行的，在URL中并没有明文显示。
		- 3. Get方式提交的数据安全性不高，而Post方式的更加安全

	- HttpServlet与Servlet的区别
		- HttpServlet指能够处理HTTP请求的 servlet，它在原有Servlet接口上添加了一些与HTTP协议处理方法
		- HttpServlet在实现Servlet接口时,覆写了service方法,该方法体内的代码会自动判断用户的请求方式，如为GET请求，则调用 HttpServlet的doGet方法,如为Post请求,则调用 doPost方法

	- ServletConfig
		- servlet配置文件的对象,由服务器创建,通过Servlet中init方法传递到Servlet中
	- ServletRequest 
		- 代表一个HTTP请求，请求在内存中是一个对象，存放请求参数和属性。
	- ServletResponse
		- 由容器自动创建的，代表Servlet对客户端请求的响应

### Servlet容器的理解
	- 提供Servlet代码的运行环境,为Servlet的运行提供底层支持,还需要管理由用户编写的Servlet类，比如实例化类（创建对象）、调用方法、销毁类等
	- 为何Servlet需要Servlet容器管理
		-  Servlet类没有main()函数，不能独立运行，只能作为一个模块被载入到Servlet容器，然后由Servlet容器来实例化，并调用其中的方法

### servletContext的理解
	- 为servlet的上下文,当项目部署到服务器的时候,服务器会为这个项目创建一个对象,而这个对象就是ServletContext对象,并且这个对象是全局唯一的,而且所有的servlet都共享这个对象
	- 作用
		- Web缓存,把不经常更改的内容读入内存，所以服务器响应请求的时候就不需要进行慢速的磁盘I/O了
		- Web应用范围内存取共享数据
		- 访问web应用的静态资源
		- Servlet对象之间通过ServletContext对象来实现通讯

### Servlet的Listener监听器理解
	- Listener是Servlet的监听器,它可以监听客户端的请求、服务端的操作等。通过监听器,可以自动激发一些操作,比如监听在线的用户的数量
	- 原理
		- Listener监听器就是一个实现特定接口的普通Java程序，这个程序专门用于监听一个java对象的方法调用或属性改变，当被监听对象发生上述事件后，监听器某个方法将立即被执行。
	- 监听器Listener就是在application,session,request三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件
	- 分类
		- ServletContextListener：用于对Servlet整个上下文进行监听（创建、销毁）
		- ServletContextAttributeListener：对Servlet上下文属性的监听（增删改属性）
		- HttpSessionListener接口：对Session的整体状态的监听。
		- HttpSessionAttributeListener接口：对session的属性监听
		- ServletRequestListener：用于对Request请求进行监（创建、销毁）
		- ServletRequestAttributeListener：对Request属性的监听（增删改属性）

### Servlet的Filter过滤器理解
	- 概念
		- Filter并不是一个标准的Servlet,它不能处理用户请求,也不能对客户端生成响应。主要用于对HttpServletRequest进行预处理,也可以对HttpServletResponse进行后处理,是个典型的处理链
	- 作用
		1.在HttpServletRequest到达Servlet之前,拦截客户的HttpServletRequest 
		2.根据需要检查HttpServletRequest,也可以修改HttpServletRequest头和数据
		3.在HttpServletResponse到达客户端之前,拦截HttpServletResponse 
		4.根据需要检查HttpServletResponse,可以修改HttpServletResponse头和数据

	- Filter接口三个方法
		- init(FilterConfig)
			- Servlet过滤器的初始化方法，Servlet容器创建Servlet过滤器实例后就会调用这个方法。在这个方法中可以通过FilterConfig来读取web.xml文件中Servlet过滤器的初始化参数
		- doFilter(ServletRequest, ServletResponse, FilterChain)
			- 完成实际的过滤操作的方法，当客户请求访问与过滤器关联的URL时，Servlet容器先调用该方法。FilterChain参数用来访问后续的过滤器的doFilter()方法
		- destroy()
			- Servlet容器在销毁过滤器实例前调用该方法，在这个方法中，可以释放过滤器占用的资源

	- Filter建立步骤
		1.建立一个实现Filter接口的类
		2.在doFilter方法中放入过滤行为
		3.调用FilterChain对象的doFilter方法
		4.对相应的servlet和JSP页面注册过滤器
		5.禁用激活器servlet。防止用户利用默认servletURL绕过过滤器设置


### Servlet,Listener,Filter的区别
	- web.xml的加载顺序是：ServletContext -> context-param -> listener -> filter -> servlet
	- servlet url传来之后，就对其进行处理，之后返回或转向到某一自己指定的页面。它主要用来在业务处理之前进行控制
	- Filter url传来之后，检查之后，可保持原来的流程继续向下执行，被下一个filter,  servlet接收等，而servlet  处理之后，不会继续向下传递
	- servlet,filter都是针对url之类的，而listener是针对对象的操作的，如session的创建，session.setAttribute的发生，在这样的事件发生时做一些事情

### DispatcherServlet Servlet HttpServlet的区别

### javaWeb三大组件
	- listener
	- filter
	- servlet
### Web服务器的作用
	- 将某个主机上的资源映射为一个URL供外界访问


### Spring事务
	- 事务管理器——PlatformTransactionManager
		- 事务管理器的顶层接口,只规定了事务的基本操作:创建事务，提交事物和回滚事务。

	- 事务状态——TransactionStatus
		- 包含了事务对象，并且存储了事务的状态,
		- PlatformTransactionManager.getTransaction() 时创建的也正是这个对象。这个对象的方法都和事务状态相关。

	- 事务属性的定义——TransactionDefinition
		- 表示一个事务的定义，将根据它规定的特性去开启事务。事务的传播等级和隔离级别的常量同样定义在这个接口中。

	- 抽象类 - AbstractPlatformTransactionManager实现了PlatformTransactionManager接口
		- 以模板方法的形式封装了固定的事务处理逻辑，而只将与事务资源相关的操作以protected或者abstract方法的形式留给子类来实现
		- 子类
			- DataSourceTransactionManager
			- HibernateTransactionManager
			- JtaTransactionManager

	- 编程式事务
		- TransactionTemplate
			- 使用模板方法设计模式对spring原始事务管理方式的封装。
			- 依赖于execute(TransactionCallback<T> action)方法执行事务管理

	- 注解式事务
		- @EnableTransactionManagement
			- 引入一些属性配置
			- TransactionManagentConfigurationSelector

		- TransactionManagentConfigurationSelector
			- AutoProxyRegistra
				- AOP代理
			- ProxyTransactionManagementConfiguration
				- BeanFactoryTransactionAttributeSourceAdvisor
					- 包含了切入点和增强点
				- TransactionAttributeSource
					- 可以用来帮助Pointcut匹配
				- TransactionInterceptor
					- 增强逻辑织入
	
	- 编程式事务与注解事务区别
		- 编程式事务侵入到了业务代码里面，但是提供了更加详细的事务管理
		- 而声明式事务由于基于AOP，所以既能起到事务管理的作用，又可以不影响业务代码的具体实现,最细粒度只能作用到方法级别

	- 只要是以代理方式实现的声明式事务，无论是JDK动态代理，还是CGLIB直接写字节码生成代理，都只有public方法上的事务注解才起作用。而且必须在代理类外部调用才行，如果直接在目标类里面调用，事务照样不起作用
