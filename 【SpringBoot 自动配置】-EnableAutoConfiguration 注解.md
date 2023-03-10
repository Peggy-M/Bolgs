# 【SpringBoot 自动配置】-EnableAutoConfiguration 注解

续接上回 [【Spring Boot 原理分析】- 自动配置](https://juejin.cn/post/7202238261641068605) 与[【SpringBoot 自动配置】- Enable*注解 ](https://juejin.cn/post/7202502142866538557),在前面笔者分析了在 SpringBoot 自动装配中的最重要的两个注解类， `@Condition` 与 `@EnableAutoConfiguration`

哎~说到这里，那首先我们跟随笔者一起先回忆一下这两个注解的在 SpringBoot 当中充当什么角色呢？

## 梦回 Condition

> ✈️ Condition 注解：是在 Spring 4.0 增加的条件判断的一个功能，通过该功能可以实现选择性的创建 Bean

官方解释如下所示：

![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3064952aea64208a2c30a5f73a8c599~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图1]

具体的使用方式就是，将该注解作用于我们要注入的 Bean 的 Config 配置类返回 Bean 对象的方法上例如下面的案例

```
@Configuration
public class UserConfig {
    @Bean
    @Conditional(".....")
    public User user(){
        return new User();
    }
}
```

                                         [SpringBoot 自动配置-code-1]

而 `@Conditional` 注解的内部需要我们的传入具体的实参，通过上面的 **[SpringBoot 自动配置-图1]** 可以看到，我们的 `@Conditional` 注解中解释的参数是一个 ***Condition*** 的 Class 类型的数组，所以我们的如果说，要让我们的对象是通过条件触发注入到容器当中的，那么我们必须自己定义一个 ***Condition*** 的实现类，并将该定义的实现类通过参数的形式添加到 `@Conditional` 的注解上。

```
// 自定义注解实现类
public class ClassCondtion implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        boolean result=false;
        //条件不满足
         **** ** 
        //条件满足 
        result=true;
        return result;
    }
}
```

                                         [SpringBoot 自动配置-code-2]

我们通过对于实现类返回值得控制 **（ true[加载] / false[不加载] ）** 便可以实现在 SpringBoot 容器创建的时候更加条件决定是否加载该对象到容器当中。

🛸 当然对于 `@Conditional` 注解在此基础上，官方延伸出了一些功能性更强的注解，比如：

> @ConditionalOnClass :判断是否存在这个类文件，如果有这个文件就相当于满足条件，然后可以注入到容器当中

> @ConditionalOnBean: bean存在的时候注入，不存在的时候不注入

> @ConditionalOnProperty: 主要可用于通过和 SpringBoot 当中 application 配置文件来使用

**所以 `Condition` 注解就一句话总结就是，该注解作用在对象的配置类的对象方法上，通过条件触发创建 Bean**

## 糟糕的梦魇 @EnableAutoConfiguration

紧接上篇的余尾，大致回忆一下 `@EnableAutoConfiguration` 注解的作用，看该注解的名字就知道该注解的作用就是启动自动配置。

上回主要说的就是关于 `@Import` 注解的详细使用，以及对 `ImportSelector` 接口的分析，那么这两个为什么会这么重要呢？

🛫 我们继续从 SpringBoot 的启动类上的 `SpringBootApplication` 注解说起，搬好小板凳！！！！！起飞

![image-20230221194625207](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/644842dec6e2419a9b57a80df1d4b8f6~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图2]

![image-20230221194655982](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ea3c70378104889951d1b69bcd609af~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图3]

![image-20230221194720158](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e5ec0288b1d4ccb982a64a383acda81~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图4]

![image-20230221194823600](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a2cdf181f7d4248b6f6d7506b819ebb~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图5]

![image-20230221194906945](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50d867195d3e4f99a677bfa046eccf51~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图6]

![image-20230221195023692](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5c0f116d5b54184a40aaecd755118d5~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图7]

============================================================================================================================

🪂 这几幅图的连环炮，大家都还和我一样，都挺的住吧。。。如果没挺住，哎~~先别骂，笔者我第一次，不，第 N 次看的时候其实和你们一样，准备放弃自己的职业生涯，到我们公司楼下去应聘保安的。

🚠 好从现在下面的分割线开始，大家先学会一点，就是忘记，没错先忘记之前看的并用你聪明的小脑袋瓜理解的所以的点，就像太极拳一样，当你全部忘记的时候，就是你学会的时候。跟着笔者一步一步来，我们从后向前看。。。

===========================================================================================================================

最后一张图，也就是 **[SpringBoot 自动配置-图7]** 我们看到一个 `ImportSelector` 字面意思就是导入选择器，那么这个选择器怎么使用呢？

既然是接口，那必须要相应的实现了，所以首先必须有其具体的实现类，继续向上看 **[SpringBoot 自动配置-图6]** 中的 **DeferredImportSelector** 接口基础了 `ImportSelector` ， 再继续 **[SpringBoot 自动配置-图5]** 可以看到 **AutoConfigurationImportSelector** 实现了 **DeferredImportSelector** 接口，所以也就是说 **AutoConfigurationImportSelector** 其实就是 `ImportSelector` 的实现类。

接着继续看 **[SpringBoot 自动配置-图4]** 在 `@Import` 注解中将 `ImportSelector` 接口的实现类 **AutoConfigurationImportSelector** 的类对象加载其中。

还记得我们前面提到过的 `@Import` 注解的四种用法吗？

1.  > 导入 Bean
1.  > 导入配置类
1.  > 导入 ImportSelector 实现类，一般用于加载配置文件的类
1.  > 导入 ImproBeanDefintionRegistrar 实现类

而这里 SpringBoot 官网的启动类的自动加载就是使用的第三种，通过导入 `ImportSelector` 的实现类 ***AutoConfigurationImportSelector*** 来实现指定类的加载的。

🚀 所以大家懂的了吧，原来啊，是这样的 `@EnableAutoConfiguration` 注解是一个复合注解，主要由 `@Import` 注解加实现类 ***AutoConfigurationImportSelector*** 控制，来完成对 Bean 的初始化注入啊。

## 自动加载的心脏 - AutoConfigurationImportSelector

![image-20230221201012868](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04b5defe08b842aa81f3918d2f38a1d3~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图8]

可以看到的是 `ImportSelector` 接口中的 **selectImports** 方法返回的是一个字符串类型的数组，注释中指明到返回的是指定的类的全限定名。

所以我们返回到 ***AutoConfigurationImportSelector*** 来看看他是具体怎么实现 **selectImports** 方法的呢？

![image-20230221201858469](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58524251ecc44083a1fcf28c3342a1e8~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图9]

通过 **[SpringBoot 自动配置-图9]** 可以看到在 `selectImports` 的方法中调用了该类中的 **getAutoConfigurationEntry** 方法获取了自动加载的条目，然后转化成 Stirng 类型的数组并返回。

![image-20230221202216178](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d762e6dcbaaa4c4baec289b25d881cee~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图10]

通过 **[SpringBoot 自动配置-图10]** 可以看到 **getAutoConfigurationEntry** 方法中在调用 **getCandidateConfigurations** 方法获取了所有需要加载的配置集合。

```
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```
                                         [SpringBoot 自动配置-code-3]

![image-20230221202705672](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/721b0f4613a345e4837fe0318c1ab726~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图11]

通过 **[SpringBoot 自动配置-图11]** 可以看到返回的是自动需要装配的类的全限定名，并且断言是否找到 META-INF/spring.factories 文件。

🎷 哎~~ 说到这里，那么这里 spring.factories 文件到底是什么呢？为什么通过该文件，就可以找到需要加载类的全限定名，提取作为字符串集合返回呢？

我们接着看。。。。。。

随便找一个自动加载的依赖我进行查看，其目录的结构以及目录下的 **spring.favtories** 文件

![image-20230221203326680](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3cc1c1787744053870fa1c4281d556d~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图12]

![image-20230221203446067](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2b0dd90d7d544f68c2d027b8333ed02~tplv-k3u1fbpfcp-zoom-1.image)

                                         [SpringBoot 自动配置-图13]

可以看到 spring-boot-autoconfig 包下的 **spring.favtories** 文件中有大量的自动加载的类路径资源

看到这里，聪明的你是不是以及明白了呢，其实这些就是 SpringBoot 通过 `@import` 能实现自动加载的原因了，原来是 SpringBoot 的启动类上有一个子注解叫做 `@import` 而该注解呢，通过对 `ImportSelect` 接口的实现类`AutoConfigurationImportSelector` 类下的 `selectImports` 方法，该方法通过其他方法的调用实现对 **spring.favtories** 配置文件中的指定类路径获取从而实现了自动注入。

其实看到这里大家有没有想到一点呢？

就单单一个 spring-boot-autoconfig 的包下就有这么多的配置，那么对于其他的包导入后是不是也都是 **spring.favtories** 文件下的全部导入呢？

其实并不是这样啦，如果将 **spring.favtories** 配置文下所有的类路径都导入，那样就会让我们的 SpringBoot 变得很看起来 "繁重" ，不知大家还记得之前提到过的 `@ImportSelector` 注解的衍生注解吗？比如 `@ConditionalOnClass` `@ConditionalOnBean` `@ConditionalOnProperty` 而其实在我们导入的 jar 包中都是有 `@ImportSelector` 注解的衍生注解去约束的，从而保证导入到容器中的对象都是高可用的。

看到这里，相比大家其实对 SpringBoot 的自己装载都有了一个清晰的认识啦，关于如何自定义一个类似于第三方提供的依赖的那种自动更具配置文件进行自动导入，我们下一篇再解释咯。

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 9 天***