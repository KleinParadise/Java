### Spring Aop
- AOP = 面向切面编程
  - 通过代理模式,为实例对象创建了代理对象,调用某一个实例对象的方法时，都会经过这个实例对象相对应的代理对象,即执行的控制权先交给代理对象。
  - spring Aop负责控制着整个容器内部的代理对象,Spring知晓每一次对实例对象的方法调用,在这个动态代理的过程中插入Spring的自己的业务代码(代理实现)
  - 通过实现MethodBeforeAdvice,AfterReturningAdvice,ThrowsAdvice接口添加在调用实例方法之前,之后,异常时需要插入的代理实现

- 动态代理
  - (why) 使用静态代理，会使我们系统内的类的规模增大,并且不易维护和扩展业务
  - (what) 动态代理的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以在运行前并不存在代理类的字节码文件
  - (how) 在运行状态中，需要代理的地方，根据Subject 和RealSubject，动态地创建一个Proxy，用完之后，就会销毁，避免Proxy 角色的class在系统中冗杂的问题

- InvocationHandler触发管理器
  - (why)使用如Javassist开源框架,在代码中动态地生成Proxy的字节码,手动地创建了太多的业务代码,并且封装性也不够,完全不具有可拓展性和通用性
  - (what)
  - (how)外界对Proxy角色中的每一个方法的调用，Proxy角色都会交给InvocationHandler来处理，而InvocationHandler则调用具体对象角色的方法

- 通过JDK和CGLIB来实现动态代理
  -(why)使用InvocationHandle,代理Proxy和RealSubject应该实现相同的功能(在Proxy功能调用转入到InvocationHandle的invoke方法,实现Advice的加入和真实实例函数的调用)
  - JDK动态代理的实现
    - 定义一个功能接口，然后让Proxy 和RealSubject来实现这个接口
      1.  获取 RealSubject上的所有接口列表
      2.  确定要生成的代理类的类名，默认为：com.sun.proxy.$ProxyXXXX
      3.  根据需要实现的接口信息，在代码中动态创建 该Proxy类的字节码
      4.  将对应的字节码转换为对应的class 对象
      5.  创建InvocationHandler 实例handler，用来处理Proxy所有方法调用
      6.  Proxy的class对象以创建的handler对象为参数，实例化一个proxy
      
  - CGLIB动态代理的实现 
    - 通过继承,Proxy 继承自RealSubject，Proxy则拥有了RealSubject的功能  

      1. 查找A上的所有非final 的public类型的方法定义
      2. 将这些方法的定义转换成字节码
      3. 将组成的字节码转换成相应的代理的class对象
      4. 现 MethodInterceptor接口，用来处理 对代理类上所有方法的请求（这个接口和JDK动态代理InvocationHandler的功能和角色是一样的）


- ProxyFactoryBean
  - 从ProxyFactoryBean获得代理对象需要提供的信息
    1.Proxy应该感兴趣的Adivce列表
    2.真正的实例对象引用
    3.告诉ProxyFactoryBean使用基于接口实现的JDK动态代理机制实现proxy
    4.Proxy应该具备的Interface接口


- Advice的执行顺序实现
  - ReflectiveMethodInvocation 维护一个Advice的List
    - 对该List递归调用实现对其顺序控制

- Adivce的条件执行(加入过滤器)
  -  MethodMatcher 


- FactoryBean与BeanFactory的区别
  - FactoryBean最为典型的一个应用就是用来创建AOP的代理对象ProxyFactoryBean
  - BeanFactory是个Factory，也就是 IOC 容器或对象工厂，所有的 Bean 都是由 BeanFactory( 也就是 IOC 容器 ) 来进行管理 


- BeanFactory与BeanFactory的区别
  - BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为               BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：
      - 1. 继承MessageSource,支持国际化
      - 2. 实现了ResourceLoader接口,使用getResource()方法实现统一的资源文件访问方式
      - 3. 
  - BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没       有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。  
    ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预         载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。  
    相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。
       
  - BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader
  - BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。




