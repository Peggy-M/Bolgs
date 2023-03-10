#  【SpringBoot 自动配置】- Enable*注解

## 自定义 jar 的容器注入

我先看看 SpringBoot 启动类的 `SpringBootApplicaiton` 注解

~~~ java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
~~~

前面三个注解为元注解， `SpringBootConfiguration`  注解是一个 Configurantion 的配置类注解

~~~ java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
~~~

而这里的 `EnableAutoConfiguration` 注解就是自动配置的关键所在，进入到该注解。

~~~ java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
~~~

✨ 所以对于 Enable* 注解来说，这些注解都是动态的启动某个功能的。而它的底层就是使用 @import 注解导入一些配置类，实现 Bean 的动态加载。

下面我们通过一个简单的案例，进一步思考

🎄 在 SpringBoot 工程当中，我们是否可以直接获取 jar 包中定义的 Bean 呢？

1. 创建一个单独的用于准备注入对象的模块，并添加实体类于实体类的配置类

   ```java
   package com.peggy.enableother.dome;
   
   /**
    * @author peggy
    * @date 2023/2/21 11:33
    */
   public class User {
   }
   
   ```

   ```java
   package com.peggy.enableother.config;
   
   import com.peggy.enableother.dome.User;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   /**
    * @author peggy
    * @date 2023/2/21 11:33
    */
   @Configuration
   public class UserConfig {
       @Bean
       User user() {
           return new User();
       }
   }
   
   ```

   2.在另一个模块中导入我们创建好的实体类的 jar 包坐标

   ```xml
   <!--获取自定义的 jar 包 -->
   <dependency>
       <groupId>com.peggy</groupId>
       <artifactId>spring-boot-enable-other</artifactId>
       <version>0.0.1-SNAPSHOT</version>
   </dependency>
   ```

   3. 在启动类从容器中获取我们的指定的需要注入的 Bean 名称

   ```java
   
   /**
    * @author peggy
    * @date 2023/2/21 11:35
    */
   @SpringBootApplication
   public class SpringBootEnableApplication {
   
       public static void main(String[] args) {
           ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
   
           //获取自定义的 jar 包中的 User 对象
           Object user = run.getBean("user");
           System.out.println(user);
       }
   
   }
   ```

   然后我在另一个模块中，指定我们的需要从容器中获取的 bean 对象名，在启动类中运行可以发现抛出异常，提醒找不到指定的 bean 

   ![image-20230221113728217](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221113728217.png)

🎁 所以到这里就很明显，我们导入的自己的第三方模块是无法在 SpringBoot 工程启动的时候，将我们自己的定义的 Bean 对象加载到 容器当中的。

我们的有没有什么方式处理，可以让我们启动类的所在的模块，可以加载到我们自己定义的 jar 包中所在的对象呢？

1. 通过包扫描将我们的其他模块中的对象加载并注入到容器当中

   - 我们进入 `SpringBootApplication` 注解当中,可以看到一个 `CompoentScan` 这样的一个注解，该注解就是用于 SpringBoot 在启动的时候进行包扫描并实现容器注入的，而该注解扫描的范围是当前引导类所在包，以及他的子包。

   ![image-20230221140118136](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221140118136.png)

   - 由于我自定义的包对象所在的位置并不是子包的位置，要想将我们的自定义模块的包扫描到则必须自己指定

   ```java
   /**
    * @author peggy
    * @date 2023/2/21 11:35
    */
   @SpringBootApplication
   @ComponentScan("com.peggy.enableother")
   public class SpringBootEnableApplication {
   
       public static void main(String[] args) {
           ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
   
           //获取自定义的 jar 包中的 User 对象
           Object user = run.getBean("user");
           System.out.println(user);
       }
   
   }
   ```

   可以看到，当我们的在启动类上加入我们的包扫描并指定我们自定义的而模块中的类路径后，我们的启动类可以正常的加载，并获取容器中的对象。

   但是这种的方式是非常的繁琐的，毕竟我们不能将所以的包都指定，然后写一大堆包路径的。而且在正常的情况下我们的是不知道我们加载对象的包路径的。

2. 我们通过 `@Import` 注解， 完成类的加载

   ![image-20230221141605062](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221141605062.png)

   可以看到 Impot 注解中有一个 Class 属性数组，我们只需要指定该类的数组即可。

   ~~~ java
   /**
    * @author peggy
    * @date 2023/2/21 11:35
    */
   @SpringBootApplication
   @Import(UserConfig.class)
   public class SpringBootEnableApplication {
   
       public static void main(String[] args) {
           ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
   
           //获取自定义的 jar 包中的 User 对象
           Object user = run.getBean("user");
           System.out.println(user);
       }
   
   }
   ~~~

   - 但这样有一些麻烦，而且需要手动的进行指定，并要记该类的名称

   那有没有更好的方式的呢？我们可以想到，如果自己内部定义一个该类对象的 `@Import` 注解， 那是不是可以简化我们上面所出现的这种问题呢？

3. 自定义 Import 注解，进行对象注入加载

   ~~~ java
   /**
    * @author peggy
    * @date 2023/2/21 14:28
    */
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Import(UserConfig.class)
   public @interface EnableUser {
   }
   ~~~

   ![image-20230221143032716](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221143032716.png)

   

那么问题就是，我们一般在使用第三方的依赖时，只需要导入指定的 Maven 坐标，然后加载 jar 包后，就可以直接通过容器取出我们需要的对象。

那么 SpringBoot 怎么这么智能 ，它怎么知道我们需要那个 jar 包的中的容器依赖呢？它为什么可以这么精准的将我们需要的对象注入的容器当中呢？

其实啊，这就依赖于我们前面看到的一个叫做 Enable * 类型的一系列注解啦。

我们看第二个重要的注解 `import`

## Import 注解的实现

🎖️ 你知道的 Import 的用法有几种呢？

1. 导入 Bean
2. 导入配置类
3. 导入 ImportSelector 实现类，一般用于加载配置文件的类
4. 导入 ImproBeanDefintionRegistrar 实现类

- 导入 User 直接 Bean 获取

  ~~~java
  /**
   * @author peggy
   * @date 2023/2/21 11:35
   */
  @SpringBootApplication
  @Import(User.class)
  public class SpringBootEnableApplication {
  
      public static void main(String[] args) {
          ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
          //将容器中对象为 User 的 Bean 取出
          Map<String, User> beansOfType = run.getBeansOfType(User.class);
          System.out.println(beansOfType);
  
          //这里需要进行修改一下，让我们的 getBean 通过获取的是 User 类型获取
          Object user = run.getBean(User.class);
          System.out.println(user);
      }
  
  }
  ~~~

  ![image-20230221144811579](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221144811579.png)

- 导入 UserConfig 配置类获取

  ~~~ java
  
  /**
   * @author peggy
   * @date 2023/2/21 11:35
   */
  @SpringBootApplication
  @Import(UserConfig.class)
  public class SpringBootEnableApplication {
  
      public static void main(String[] args) {
          ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
  
          //由于是通过配置类进行注入的 Bean ，所以在默认情况下 User 对象 Bean 的名字与配置类中注入的时候 Bean 对象名与方法名相同
          Object user = run.getBean("user");
          System.out.println(user);
      }
  
  }
  ~~~

- 导入 ImportSelector 实现类，获取 Bean 对象

  ~~~ java
  public interface ImportSelector {
  	String[] selectImports(AnnotationMetadata importingClassMetadata);
  }
  ~~~

  可以看到 ImportSelector 接口中提供了一个方法 `selectImports`， 该方法的返回值是一个字符串类型的数组，而这里最终返回的就是我们的当前要注入的对象路径。

  1. 创建 ImportSelector 的实现类，并指定要注入的对象所在的包路径

     ~~~ java
     /**
      * @author peggy
      * @date 2023/2/21 14:53
      */
     public class MyImportSelector implements ImportSelector {
         @Override
         public String[] selectImports(AnnotationMetadata importingClassMetadata) {
             return new String[]{"com.peggy.dome.User"};
         }
     }
     ~~~

  2. 在启动类的 Import 注解中指定我们的实现类

     ~~~ java
     /**
      * @author peggy
      * @date 2023/2/21 11:35
      */
     @SpringBootApplication
     @Import(MyImportSelector.class)
     public class SpringBootEnableApplication {
     
         public static void main(String[] args) {
             ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
             //将容器中对象为 User 的 Bean 取出
             Map<String, User> beansOfType = run.getBeansOfType(User.class);
             System.out.println(beansOfType);
     
             //这里需要进行修改一下，让我们的 getBean 通过获取的是 User 类型获取
             Object user = run.getBean(User.class);
             System.out.println(user);
         }
     }
     ~~~

- 导入 ImproBeanDefintionRegistrar 的实现类完成注入

  ![image-20230221150214804](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221150214804.png)

  我们需要用的方法就是 registerBeanfinition 该方法，并且要用到的参数就是 registry

  ```java
  /**
   * @author peggy
   * @date 2023/2/21 14:58
   */
  public class MyImproBeanDefintionRegistrar implements ImportBeanDefinitionRegistrar {
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          //创建一个 BeanDefinitionRegistry 对象
          AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class).getBeanDefinition();
          
          registry.registerBeanDefinition("user",beanDefinition);
      }
  }
  ```

  ```java
  /**
   * @author peggy
   * @date 2023/2/21 11:35
   */
  @SpringBootApplication
  @Import(MyImproBeanDefintionRegistrar.class)
  public class SpringBootEnableApplication {
  
      public static void main(String[] args) {
          ApplicationContext run = SpringApplication.run(SpringBootEnableApplication.class, args);
          //将容器中对象为 User 的 Bean 取出
          Map<String, User> beansOfType = run.getBeansOfType(User.class);
          System.out.println(beansOfType);
  
          //这里需要进行修改一下，让我们的 getBean 通过获取的是 User 类型获取
          Object user = run.getBean(User.class);
          System.out.println(user);
      }
  }
  ```

  ![image-20230221150831838](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221150831838-16769633125481.png)

  可以看到容器中的对象的名称就是我们刚在设置的 user 

有了上面的理论，接下来我们就细细的说说，SpringBoot 是如何进行自动的装配的。

![image-20230221151327968](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221151327968.png)

![image-20230221151452215](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221151452215.png)

![image-20230221151526965](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221151526965.png)

![image-20230221151628144](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221151628144.png)

![image-20230221151647169](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230221151647169.png)

可以看到原来我们的 SpringBoot 的 SpringBootApplication 注解中是通过 Import 导入预先定义好的一些类，然后实现自动的装配的，而与我们上面的第三种的导入方式相同，都是实现了 importSelect 接口。

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 8 天***