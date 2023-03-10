# Spring 底层原理与解析 - 容器接口

## BeanFactory 能做哪些事

- BeanFactory 与 ApplicaiotnContext 到底是谁提前做完了对象的加载

在之前的一篇关于 Spring 的文章[Spring IoC 与容器的初始化](https://juejin.cn/post/7199533150250172473)中提到过，BeanFactory 接口与 ApplicationContext 接口之间的关系 image-20230214191907914

![image-20230214193235253](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214193235253.png) 

![image-20230214191907914](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214191907914.png) 

可以看到的是 **BeanFactory** 是 **Spring** 容器的一个最原始的接口，同时也定义了获取 Bean 对象的方法 **getBean()**。

但 **BeanFactory**  接口所提供的 **getBEean()** 方法获取 **Bean**  的加载时机与 **ApplicationContext ** 对象获取的 **Bean** 对象的 **GetBean**  是不同的。

我们在上一篇 [Spring IoC 与容器的初始化](https://juejin.cn/post/7199533150250172473) 中已经进行过验证，**BeanFactory**  初始化 Bean 对象是在执行调用 **getBean()** 的时候，而 **ApplcationContext** 是在第一次加载 Spring 的配置文件的时候，就已经将 **Bean** 对象初始化到了容器当中的。

其实在 **ApplicationContext** 对象中调用的 **getBean()** 方式是来自于 **BeanFactory**  对象的，我们可以从代码中证明这一点。

```java
public class App {
    public static void main( String[] args ) {
        ApplicationContext applicationContext=new ClassPathXmlApplicationContext("applicationContext.xml");
        System.out.println("开始执行 getBean 方法");
        applicationContext.getBean("userService");
    }
}
```

```java
public Object getBean(String name) throws BeansException {
    this.assertBeanFactoryActive();
    return this.getBeanFactory().getBean(name);
}
```

我们进入 **applicationContext** 对象的 **getBean()** 方法中可以看到，**getBean()** 方法其实就是调用了 **getBeanFactory()** 方法首先获取 **BeanFactory** 对象，然后调用 **BeanFactory** 对象的 **getBean()** 方法获取容器中的 Bean 。

- BeanFactory 能干的事多吗？

BeanFactory 我们前面看到过，该接口所提供的功能似乎很少，唯一用的最多的只要一个 **getBean** 。那么，我们这么强大的轻量级框架的 Spring 真的就这么”轻“吗？

哎~ 实时不然，我们所用到的绝大多是方法其实都是由 BeanFactory 实现类所提供的，不信你看。

**BeanFactory**  的一个子接口 **ConfiguraableBeanFactory** 下的一个实现类 **DefaultListableBeanFactory** 

通过名字就可以看出 **ConfiguraableBeanFactory** 就是一个可以自己配置的 **BeanFactory** ，而 **DefaultListableBeanFactory** 提供了一些默认的 **BeanFactory** 的方法

可以看到其抽象父类 **AbstractAutowireCapableBeanFactory** 的继承关系图

![image-20230214200046517](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214200046517.png)

![image-20230214200549228](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214200549228.png)

我们 F4 进入 **SingletonBeanRegistry** 的实现类 **DefaultSingletonBeanRegistry** 中，该实现类中管理了我们在 Spring 容器中创建的所有的对象。

![image-20230214200904478](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214200904478.png)

可以看到有一个叫做 **singletonObjects ** 的对象，该对象是一个 HashMap 而这个 Map 集合中就存储这我们 Spring 中创建的所有的 Bean 对象。我们可以通过 DeBug 的方式进行查看，也可以通过反射的方式获取该 singletonObject 中的属性值

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService1" class="com.peggy.service.impl.UserServiceImpl1"/>
    <bean id="userService2" class="com.peggy.service.impl.UserServiceImpl2"/>

</beans>
```



```java
public class App {
    public static void main( String[] args ) {

        ConfigurableApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");

        try {

            Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
            singletonObjects.setAccessible(true);
            ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
            
            Map<String,Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
            /* 通过过滤器只保留一部分 */
            map.entrySet().stream().filter(e->e.getKey().startsWith("userService")).forEach((e)->{
                System.out.println(e.getKey()+"="+e.getValue());
            });
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}

```

![image-20230214204824710](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214204824710.png)

 可以看到 singletonObjects 属性中有我们注入的 UserService 对象

## ApplicationContext 有哪些扩展的功能

appllicationContext 的主要功能的扩展来自于 **MessageSource** （国际化处理）、**ResourcePatternResolver** （资源匹配，来自用磁盘本地的文件资源匹配）、**ApplicationEventPublisher** （发布事件对象、新用户短信验证注册）、**EnvironmentCapable** (处理环境信息，环境变量)

![image-20230214211123221](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214211123221.png)

- 国际化的处理

```java
@SpringBootApplication
public class SpringDomeApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringDomeApplication.class, args);

        System.out.println(context.getMessage("hi", null, Locale.CHINA));
        System.out.println(context.getMessage("hi", null, Locale.ENGLISH));

    }

}
```

![image-20230214212431657](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214212431657.png) 

 ![image-20230214212814688](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214212814688.png) 

- 获取资源

~~~ java
@SpringBootApplication
public class SpringDomeApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringDomeApplication.class, args);
        /*
         * classpath 到类路径下找到对应的资源
         * file 到磁盘目录下获取资源
         * */
        try {
            Resource[] resource = context.getResources("application.properties");
            for (Resource re : resource) {
                System.out.println(re);
            }
            
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
~~~

![image-20230214212844180](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214212844180.png)

> **classpath*:META-INF/spring.factories**  获取 jar 包下的类路径的所有同名文件

```java
@SpringBootApplication
public class SpringDomeApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringDomeApplication.class, args);
        /*
         * classpath*:META-INF/spring.factories  获取 jar 包下的类路径的所有同名文件
         * */
        try {
            Resource[] resource = context.getResources("classpath*:META-INF/spring.factories");
            for (Resource re : resource) {
                System.out.println(re);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

}
```

![image-20230214213648969](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214213648969.png)

- 获取环境变量与信息

```java
@SpringBootApplication
public class SpringDomeApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringDomeApplication.class, args);
        System.out.println(context.getEnvironment().getProperty("JAVA_HOME"));
        System.out.println(context.getEnvironment().getProperty("server.port"));    
    }
}
```

![image-20230214214138568](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214214138568.png)

## 事件的解耦

事件的解耦指的是什么呢？

可以实现这样的一个场景，比如说用户需要进行邮件的发送，用户发送出去一个邮件不需要一直等待，直到接受者接受到邮件后才能做其他的操作。

而我们的事件对象就是就可以达到者一个 `发送者` 与 `接受者` 之间相互解耦的目的。

那么，对于我们的发送者来说

可以利用 **ApplicationEventPublisher** 中提供的 **publishEvent**  来发送信息，而创建一个监听接受者用户信息的接收。

- 创建一个监听者

```java
/**
 * @author peggy
 * @data 2023/2/14 21:47
 * 事件的接受者
 */
@Component
public class ListeningOrage {
    /**
     * 注入一个事件的监听器 当接受的监听的对象 执行该方法 
     * 该方法的参数与方法名返回值都随意
     * @param registeredEvder
     */
    @EventListener
    void take(UserRegisteredEvder registeredEvder){
        System.out.println("接受的事件: "+registeredEvder);
    }
}
```

- 创建一个时间对象

```java
/**
 * 事件对象
 * @author peggy
 * @data 2023/2/14 21:43
 */
public class UserRegisteredEvder extends ApplicationEvent {
    /**
     * 事件源
     * @param source
     */
    public UserRegisteredEvder(Object source) {
        super(source);
    }
}
```

- 事件的发送者

```java
/**
 * 时间的发送者
 * @author peggy
 * @data 2023/2/14 22:00
 */
@Component
public class UserSendEvder {
    @Autowired
    ApplicationEventPublisher eventPublisher;

    /**
     * 实现的需要调用的时间发送方法
     * 方法名与方法的参数以及返回值随意
     */
    void send(){
        System.out.println("发送信息");
        eventPublisher.publishEvent(new UserRegisteredEvder(this));
    }
}
```

- 开始发送事件

```java
@SpringBootApplication
public class SpringDomeApplication {

    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(SpringDomeApplication.class, args);
        /*
        * 通过类信息获容器中的发送者的对象，并执行发送者的发送方法
        * */
        context.getBean(UserSendEvder.class).send();
    }

}
```

> 这里可以发现我们的接受到的对象就是我们创建的事件发送者的 this 对象本身

![image-20230214221417782](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230214221417782.png)

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 2 天***

