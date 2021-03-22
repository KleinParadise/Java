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

