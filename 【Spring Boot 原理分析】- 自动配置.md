# 【Spring Boot 原理分析】- 自动配置

## Condition 注解

> Condition 是 Spring 4.0 增加的条件判断功能，通过这个功能可以实现选择的创建 Bean 操作

👑 我们在使用 Spring 的时候，只需导入某个依赖的坐标，就可以直接通过 Autwired 注解，进行依赖注入，那我们有没有想过，我们的 Spring 是怎么知道，我们要将那个对象注入到容器中呢？

就比如 SpringBoot 是如何知道要创建的 RedisTemplate 的呢？

*好，那我们就带着这个问题向下继续看。*

- 导入坐标

~~~ xml
<!--导入 data-redis 坐标 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.2</version>
</dependency>
~~~

- 获取 redisTemplate 对象

~~~java
@SpringBootApplication
public class SpringBootDomeApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(SpringBootDomeApplication.class, args);
        //获取容器中的 redisTemplate 对象
        Object redisTemplate = run.getBean("redisTemplate");
        System.out.println(redisTemplate);
    }

}
~~~

![image-20230220222233254](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220222233254.png)

可以看到最终输出打印了容器中获取的 redisTemplate 对象。

而当我们将 spring-boot-start-data-redis 坐标从依赖中去除时，当我们再次执行，肯定会报出一个 **NoSuchBeanDefinitionException** 的异常，如下图所示。

![image-20230220222955471](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220222955471.png)

💡问题就是，我们的 Spring 是怎么知道，我们到底有没有导入 redisTemlate 这个的起步依赖的呢？

我们通过一个案例，对上面的这种情况进行一个分析

📒 在 Spring 的 IOC 容器中有一个 User 的 Bean，现要求如下：

1. 导入 Jedis 坐标后，加载该 Bean ，没导入，则不加载

首先我们的创建一个 User 实体对象，然后添加一个 UserConfig 的配置类，让 Spring 启动的时候，自动将我们的 User 对象注入到我们的容器当中。

~~~ java
public class User {
}
~~~

~~~ java
@Configuration
public class UserConfig {
    @Bean
    public User user(){
        return new User();
    }
}
~~~

然后在启动类中修改 getBean 的参数，获取我们注入的 User 对象

```java
@SpringBootApplication
public class SpringBootDomeApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(SpringBootDomeApplication.class, args);
        //获取容器中的 redisTemplate 对象
        Object redisTemplate = run.getBean("user");
        System.out.println(redisTemplate);
    }

}
```

![image-20230220224145104](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220224145104.png)

为了控制我们的 Spring 在创建注入 User 到我们的 Spring 容器中，我在 UserConfig 的配置类上加一个 Condition 的注解。

```java
@Configuration
public class UserConfig {
    @Bean
    @Conditional()
    public User user(){
        return new User();
    }
}
```

我们进入这个注解，然后看看这个注解都做了什么事呢？

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

可以看到改注解中有一个 <font color='red'>**Condition**</font> 的参数，进入 Condition 这个接口中，可以看到我们需要给该注解提供一个Condition 的类对象 ，还有一个返回值为 <font color='orange'>**Boolean **</font> 类型的方法<font color='orange'>**matches **</font>。

```java
    /**
     * 确定条件是否匹配。
     * 参数：
     * context – 条件上下文 metadata – 正在检查的 class OR method 的元数据
     * 返回：
     * true 如果条件匹配并且可以注册组件，或者 false 否决注释组件的注册
     */
@FunctionalInterface
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

其实看到这里大概就已经明白了，官方解释原来就是说，如果 <font color='orange'>**matches**</font> 返回的是一个 true 那么就注入到容器当中，负责就不注入。

所以我们得首先有一个用于实现 Condition 接口的实现类，并对该接口中的 matches 方法提供相应的实现，然后将该实现类文件对象，放置到我们需要加载的类上的 Condition 注解上。

~~~ java
public class ClassCondtion implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return false;
    }
}
~~~

```java
@Configuration
public class UserConfig {
    @Bean
    @Conditional(ClassCondtion.class)
    public User user(){
        return new User();
    }
}
```

默认情况下返回的是 <font color='cornflowerblue'>**false**</font> ,也就是说默认情况下是对象是在 Spring 初始化的时候，不注入到容器中的。

所以我们现在再次运行查看控制台输出。可以看到控制台报出一个 Bean 找不到的错误

![image-20230220230550243](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220230550243.png)

我们将上面的 ClassCondition 类做略微的修改，再看看运行的情况。

```java
public class ClassCondtion implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        try {
            //获取是否导入了该类
            Class<?> aClass = Class.forName("org.springframework.data.redis.core.RedisTemplate");
        } catch (ClassNotFoundException e) {
            return false;
        }
        return true;
    }
}
```

我们将 pom 文件中的 **spring-boot-starter-data-redis** 取消，我们的 User 对象则无法成功的注入到容器当中。

![image-20230220231452401](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220231452401.png)

所以通过上面的案例，其实我们就明白了，原来我们的 Spring 在进行启动的时候，会先通过 Condition 对象进行判断，如果注入的条件满足（坐标导入），则将对象注入到容器中，否则不进行注入。

## Condition 注解的动态控制

⚠️ 上面的案例中有一个明显的问题，我们在 Condition 的实现类中是采用硬编码的方式，将包路径写在入了 实现类中。那有没有办法通过注解方式，自己指定要扫描的包呢？

其实是可以，这就需要我自定义注解，然后组合我们的 Condition注解，在自定义的注解中，定义一个用于存储路径的 value 变量，具体的实现步骤我们接着往下看。

- 自定义 ConditionOnClass 注解

```java
@Target({ElementType.TYPE, ElementType.METHOD}) //注解可加的范围
@Retention(RetentionPolicy.RUNTIME) //注解生效时机
@Documented //生成 Java document 文档
@Conditional(ClassCondtion.class)
public @interface ConditionOnClass {
    //定义一个用于存储扫描的路径的变量
    String[] value();
}
```

- 通过参数获取注解中的属性值

```java
package com.peggy.springbootdome.condtion;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.context.annotation.Conditional;
import org.springframework.core.type.AnnotatedTypeMetadata;

import java.util.Map;

public class ClassCondtion implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        try {
            //获取注解注解中的所有的属性值
            Map<String, Object> attributes = metadata.getAnnotationAttributes(ConditionOnClass.class.getName());
            String[] value = (String[]) attributes.get("value");

            for (String s : value) {
                //获取是否导入了该类
                Class<?> aClass = Class.forName(s);
            }

        } catch (ClassNotFoundException e) {
            return false;
        }
        return true;
    }
}
```

- 在自定义的类中加入，自己定义的注解，并指定类路径测试

```java
@Configuration
public class UserConfig {
    @Bean
    @ConditionOnClass("org.springframework.data.redis.core.RedisTemplate")
    public User user(){
        return new User();
    }
}
```

![image-20230220235828156](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230220235828156.png)

通过上面两个案例其实就解释了我们的 SpringBoot 是如何自动进行对象的选择装配的。

🪧 <font color='purple'>**其实在我们的 SpringBoot 当中是提供了很多的 Condition 这样类似的注解的**</font>

我们可以到 <font color='green'>**spring-boot-autconfig**</font> 该包查看。

![image-20230221000400680](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221000400680.png)

![image-20230221000553533](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221000553533.png)

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 7 天***