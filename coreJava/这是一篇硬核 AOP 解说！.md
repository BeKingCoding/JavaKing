> 本文首发于公众号【 **程序员大帝** 】，关注第一时间获取

> 一起码动人生，成为Coding King！！！

> GitHub上已经开源 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708132205796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

## 前言
如果你是个 Java 程序员，除了 JVM、并发编程等基础知识，Spring 必然是另一个绕不开的主题。Spring 框架事实上已经成为了各大公司使用 Java 进行开发时的首选，市面上各种技术层出不穷，但 Spring 全家桶却越来越全，历久弥新。



微服务架构目前大行其道，使用 SpringBoot、Spring Cloud 进行构建也更加流行。可究其本质，Spring 框架还是全家桶所有新奇技术的基础，其中 IOC 和 AOP 又是它的两大灵魂。


本文将对 AOP 的思想和实现从以下几个方面来讲述，相信大家耐心看了之后肯定有收获，码字不易，别忘了「在看」，「转发」哦。

- AOP 的前生今世
- 代理模式
- JDK 动态代理

- CGLIB 动态代理

- 自定义注解实现 AOP

## 正文 

## 01 AOP 的前生今世

**AOP 是什么？**



传统的 OOP 开发过程中，代码的逻辑是自上而下的。在这些自上而下的过程中会产生一些横切性的问题，而这些横切性的问题往往与业务逻辑关系并不大，散落在代码的各个地方，造成难以维护。



举个例子，对于日志功能，它的代码往往水平地散布在所有对象的层次中，而往往与它所散布到的对象核心功能毫无关系。对于其他类型的代码，如安全性、异常处理和透明的持续性也是如此。



原始代码：

```java
public void foo() {
    
    //do something...
  
  }
```



当需要在完成业务逻辑的同时记录日志，传统的做法：
```java
  public void foo() {
    
    //do something...
  
    writeLog(); //执行日志记录
  }   
```

这种散布在各处的无关的代码被称为横切（cross-cutting）代码，在 OOP 设计中，它导致了大量代码的重复，导致不利于各个模块重用。



AOP 编程思想就是把业务和横切问题进行分离，从而达到解耦的目的，使代码的重用性和开发效率更高。



AOP 的实现主要基于代理思想，对原来的目标对象，创建代理对象。在不修改原对象代码情况下，通过代理对象调用增强功能的代码，从而对原有业务方法进行增强。



AOP 的应用场景非常多，比如：

- 日志记录

- 权限校验、控制

- 效率检查（记录执行时间...）

- 事务管理（调用方法前开启事务，调用方法后提交关闭事务）

- 错误、异常处理

- 内容传递、增强

## 02 代理模式

代理模式的基本思想是给目标对象提供一个代理对象，并由代理对象控制对目前对象的引用，这样的好处有两个：



（1）通过代理对象来间接访问目标，防止了直接访问给系统带来的复杂性。



（2）实现了对原有业务的增强。



为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别。



通过代理类这中间一层，能有效控制对委托类对象的直接访问，也可以很好地隐藏和保护委托类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。



更通俗的说，代理解决的问题当两个类需要通信时，引入第三方代理类，将两个类的关系解耦，让我们只了解代理类即可。



而且代理的出现还可以让我们完成与另一个类之间的关系的统一管理，但是切记，代理类和委托类要实现相同的接口，因为代理真正调用的还是委托类的方法。



在介绍动态代理前，我们先来看一下静态代理的方式。



静态代理在使用时，需要定义接口或者父类，被代理对象与代理对象一起实现相同的接口或者继承的类。



静态代理是由程序员创建代理类或特定工具自动生成源代码再对其编译，在程序运行前代理类的.class文件就已经存在了。



举个例子，添加打印日志的功能，即每个方法调用之前和调用之后写入日志。



用户管理实现类.java
```java
public class UserManagerImpl implements UserManager {
    ...
    @Override
    public String findUser(String userId) {
        return "张三";
    }
    ...
}
```



用户管理实现代理类.java
```java
public class UserManagerImplProxy implements UserManager {
    // 目标对象
    private UserManager userManager;

    // 通过构造方法传入目标对象
    public UserManagerImplProxy(UserManager userManager){
        this.userManager=userManager;
    }

    @Override
    public void findUser(String userId) {
    // 添加打印日志的功能
    System.out.println("start-->findUser()");
    // 开始查询用户
    return userManager.findUser(userId);
    }
}
```

显而易见，静态代理存在以下几个缺点：



1、代理类和委托类必须实现相同接口，并且代理类通过委托类实现了相同的方法，这样就出现了大量的代码重复。



2、如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。



3、代理对象只服务于一种类型的对象，如果要服务多类型的对象。势必要为每一种对象都进行代理，静态代理在程序规模稍大时就无法胜任了。比如上面的代码只为 UserManager 类的访问提供了代理，但如果还要为如 DepartmentManager 类提供代理的话，就需要我们再次添加代理 DepartmentManager 的代理类。

## 03 JDK动态代理

由于静态代理存在的诸多不便，自然我们就会想到引入动态代理。



动态代理是生成一个包装类对象，由于代理的对象是动态的，所以叫动态代理。代理的主要目的是为了进行增强操作，这个增强是需要留给开发人员开发代码的。



因此代理类不能直接包含被代理对象，而是一个 InvocationHandler，该 InvocationHandler 包含被代理对象，并负责分发请求给被代理对象，分发前后均可以做增强。从原理可以看出，JDK 动态代理是“对象”的代理。



在上面的静态代理示例中，一个代理只能代理一种类型，而且是在编译器就已经确定被代理的对象。而动态代理是在运行时，通过反射机制实现动态代理，并且能够代理各种类型的对象。



在 Java 中要想实现动态代理机制，需要 java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy 类的支持



java.lang.reflect.InvocationHandler接口的定义如下：


```java
//  Object proxy:被代理的对象
//  Method method:要调用的方法
//  Object[] args:方法调用时所需要参数

public interface InvocationHandler {

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;

}
```

java.lang.reflect.Proxy 类的定义如下：
```java
//  CLassLoader loader:类的加载器
//  Class<?> interfaces:得到全部的接口
//  InvocationHandler h:得到InvocationHandler接口的子类的实例

public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
```
下面举例采用动态代理的方式，对用户管理实现类进行日志功能代理：

//动态代理类只能代理接口（不支持抽象类），代理类都需要实现InvocationHandler类，实现invoke方法。该invoke方法就是调用被代理接口的所有方法时需要调用的，该invoke方法返回的值是被代理接口的一个实现类
```java
public class LogHandler implements InvocationHandler {

    // 目标对象

    private Object targetObject;
    //绑定关系，也就是关联到哪个接口（与具体的实现类绑定）的哪些方法将被调用时，执行invoke方法。            
    public Object newProxyInstance(Object targetObject){

        this.targetObject=targetObject;

        //该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例  
        //第一个参数指定产生代理对象的类加载器，需要将其指定为和目标对象同一个类加载器
        //第二个参数要实现和目标对象一样的接口，所以只需要拿到目标对象的实现接口
        //第三个参数表明这些被拦截的方法在被拦截时需要执行哪个InvocationHandler的invoke方法
        //根据传入的目标返回一个代理对象
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),
                targetObject.getClass().getInterfaces(),this);
    }



    @Override
    //关联的这个实现类的方法被调用时将被执行
    /*InvocationHandler接口的方法，proxy表示代理，method表示原对象被调用的方法，args表示方法的参数*/
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {

        Object ret=null;
        try{
            /*原对象方法调用前处理日志信息*/
            System.out.println("satrt-->>");
            //调用目标方法
            ret=method.invoke(targetObject, args);
            /*原对象方法调用后处理日志信息*/
            System.out.println("success-->>");
        }catch(Exception e){
            e.printStackTrace();
            System.out.println("error-->>");
            throw e;
        }
        return ret;
    }
}
```

客户端代码：
```java
public class Client {
    public static void main(String[] args){
        LogHandler logHandler=new LogHandler();
        UserManager userManager=(UserManager)logHandler.newProxyInstance(new UserManagerImpl());
        userManager.findUser("1111");
    }
}
```

由以上例子可以看到，我们可以通过 LogHandler 代理不同类型的对象，如果我们把对外的接口都通过动态代理来实现，那么所有的函数调用最终都会经过invoke 函数的转发。



因此我们就可以在这里做一些自己想做的操作，比如日志系统、事务、拦截器、权限控制等。这也就是 AOP 的基本原理。

## 04 CGLIB动态代理

CGLIB（Code Generator Library）是一个强大的、高性能的代码生成库，可以在运行期间扩展 Java 类与实现 Java 接口。



其被广泛应用于 AOP 框架中，用以提供方法拦截操作。Hibernate 作为一个受欢迎的 ORM 框架，同样使用CGLIB 来代理单端（多对一和一对一）关联（延迟提取集合使用的另一种机制）。


**为什么使用CGLIB**

CGLIB 代理主要通过对字节码的操作，为对象引入间接级别，以控制对象的访问。我们知道 Java 中的动态代理也是做这个事情的，那我们为什么不直接使用Java 动态代理，而要使用 CGLIB 呢？



答案是 CGLIB 相比于 JDK 动态代理更加强大，JDK 动态代理虽然简单易用，但是其有一个致命缺陷是，只能对接口进行代理。如果要代理的类为一个普通类、没有接口，那么 Java 动态代理就没法使用了。



而 CGLIB 不仅可以接管接口类的方法，也可以接管普通类的方法，为 JDK 的动态代理提供了很好的补充。



CGLIB 底层使用了 Java 字节码操作框架 ASM。它是一个短小精悍的字节码操作框架，用于操作字节码生成新的类。除了 CGLIB 库外，脚本语言如 Groovy也使用 ASM 生成字节码。ASM 使用类似 SAX 的解析器来实现高性能。我们不鼓励直接使用 ASM，因为它需要对 Java 字节码的格式足够的了解。


**CGLIB的原理**

CGLIB 底层采用 ASM 字节码生成框架，使用字节码技术生成代理类。



CGLIB 是动态生成被代理类的子类，子类重写委托类的所有非 private、非 final 的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。



因此如果委托类被 final 修饰，那么它就不可以被继承，导致不可以被代理。同理如果委托类的一个方法被 final 修饰后，那么此方法也不可以被代理。



下面举例使用 CGLIB 完成日志记录：
```java
public class LogCGlibProxy implements MethodInterceptor {

    public Object newProxyInstance(Class clazz) {
        // 创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
        Enhancer enhancer = new Enhancer();
        // 设置目标类的字节码文件
        enhancer.setSuperclass(clazz);
        // 设置回调函数
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override

    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("调用代理对象前");
        Object result = methodProxy.invokeSuper(proxy, args);
        System.out.println("调用代理对象后");
        return result;
    }

}
```

在 Spring 中 AOP 的实现方式遵循以下原则：



（1）如果目标对象实现了接口，默认采用 JDK 动态代理进行实现。



（2）如果目标对象实现了接口，也可以强制用 CGLIB 进行实现。



（3）如果目前对象没有接口，则必须采用 CGLIB 实现动态代理。

## 05 自定义注解实现AOP

AOP 是一种概念，Spring AOP 与 AspectJ 都是AOP的实现方式。Spring AOP 有自己的语法，但是较为复杂。因此 Spring AOP 借鉴了 AspectJ 的语法格式（注解），但是底层还有由自己本身实现，也就是 JDK 动态代理和 CGLIB 动态代理。



@Aspect	利用AspectJ注解语法
xml aop:config	利用Spring命名空间


Java 注解是 JDK5.0 版本开始支持加入源代码的特殊语法元数据。



Java 语言中的类、方法、变量、参数和包等都可以被标注。和 Javadoc 不同，Java 标注可以通过反射获取标注内容。



在编译器生成类文件时，标注可以被嵌入到字节码中。Java 虚拟机可以保留标注内容，在运行时可以获取到标注内容。当然它也支持自定义 Java 标注。

**元注解**

Target：描述了注解修饰的对象范围，取值在java.lang.annotation.ElementType 定义，常用的包括：

METHOD：用于描述方法

PACKAGE：用于描述包

PARAMETER：用于描述方法变量

TYPE：用于描述类、接口或enum类型



Retention: 表示注解保留时间长短。取值在 java.lang.annotation.RetentionPolicy 中，取值为：

SOURCE：在源文件中有效，编译过程中会被忽略

CLASS：随源文件一起编译在class文件中，运行时忽略

RUNTIME：在运行时有效



只有定义为 RetentionPolicy.RUNTIME 时，我们才能通过注解反射获取到注解。

**自定义注解**

以权限校验的业务场景为例，在对资源进行操作时，需要先判断此用户是否有相对应的权限。





（1）自定义注解 @PermissionAuth，它有一个属性 role ，代表只有拥有声明的指定权限才可以进行资源操作。
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PermissionAuth {
    String role();
}
```

（2）声明切面，对自定义注解 @PermissionAuth 拦截，定义前置权限校验业务。
```java
@Component
@Aspect
public class PermissionAuthAspect {

    @Pointcut(value = "@annotation(com.xuwuji.spring.aop.PermissionAuth)")
    public void pointCut() {
    }
    /**
     * Validate User Permission
     *
     * @param jwtAuth
     * @throws QmtException
     */
    @Before(value = "pointCut()&&@annotation(permissionAuth)")
    public void validateRole(PermissionAuth permissionAuth) {
        // perimission check
    }

}
```
（3）在用户访问资源时，如果资源需要权限校验，则在对应方法上添加自定义注解 @PermissionAuth
```java
    @PermissionAuth(role = "admin”) //代表拥有admin权限的用户才能进行findUser的操作
    public User findUser(String userId) {
        return new User(userId, map.get(userId));
```


## 日常求赞
好了各位，以上就是这篇文章的全部内容了，能看到这里的人呀，都是人才。

我后面会持续更新《Offer收割机》系列和互联网常用技术栈相关的文章，非常感谢各位老板能看到这里，如果这个文章写得还不错，觉得我有点东西的话，求点赞👍 求关注❤️ 求分享👥 对我来说真的非常有用！！！

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！

Craig无忌 | 文 【原创】【转载请联系本人】 如果本篇博客有任何错误，请批评指教，不胜感激 ！

------

>《Offer收割机》系列会持续更新，可以关注我的公众号「 程序员大帝 」第一时间阅读和催更（公众号比博客早一到两篇哟），本文GitHub上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ，会对一线大厂面试要求的核心知识点持续讲解、并有对标阿里P7级别的成长体系脑图，欢迎加入技术交流群，我们一起有点东西。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715124857432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70#pic_center)
