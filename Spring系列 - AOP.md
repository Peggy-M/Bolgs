# Spring 系列 - AOP

Spring 框架从使用到现在已经有相当的长的一段时间了，但总是在使用的时候，感觉一直停留在表面，对框架的底层了解的并不多，最近一段时间，打算好好折腾一下 Spring 的底层，想对 Spring 有一个更深层次的理解。

说来对于 Spring 的两大核心， IoC 与 AOP 这两个名词的表面概念已经与案例已经看过很多了，但似乎并没有自己深入的思考过，又或者说是思考过，但并未有相关的记录，所以在相当一段时间后就渐渐的忘记了，对于一些经典的案例已经变得模糊了。

好了，回归正题，Spring 的 AOP，其实通过下面的两幅案例图（感谢黑马程序员分享的案例），其实 AOP 的思想，已经表现的相当清晰了。AOP 就是将公共重复的部分，从业务中抽取出来，我们只关注那些改变的内部业务逻辑部分，不变的部分在后期通过反射拼接到一起即可。

这样可以大幅度的提高业务逻辑的编写效率，在减少代码冗余的同时，减少后期维护的工作量，提高代码的可靠性。

🌈举个栗子来理解 AOP ：

我是这样理解的：**<font color='orange'>*其实 Spring 就是一个工厂，什么工厂呢？代码自动化装配的工厂*</font>。**

- ⛄ 回归现实当中工厂的目的是什么？

  我这里就拿传统的流水线举例，传统流水线，无非是一个老板把有所的工作人员聚集到一条产线上加工产品，比如康师傅要生成泡面，那对于泡面来说，不同的泡面，将配料包与面饼，放到泡面盒里并且完成包装，对于这一连续的动作都是相同的。

  那么对于这件事，工厂老板相当为了节约人工成本，就干脆购进一台设备，这台设备需要做的就一件事，那就是将面饼与调料包放到泡面桶里，并且完成包装。

  机器的速度肯定比人工更快，效率更高，而对于泡面的调料包的分配，以及面饼的运输，这件事就由人工去完成。

  不难看出，这个流程中是机器与人工相互配合的，机器穿插在人工当中，总览所有的人工将配料包与面饼放到泡面桶中，并完成包装的这一连续动作。

  其实，这个就是一个面向切面的织入工作。

- ☃️ 编程是生活万物与规律的另一种抽象的表现形式

  所以通过上面的栗子其实可以发现，编程中的很多思想好像都是来源于生活中的，从生活中抽象出一种思想，然后将这种思想通过另一种方式展现出现，让程序去模式，现实中的生活中的某一些具体的事务，遵循其某种规律去运行。嘿嘿 扯远了，写到这里突然想到一点点，关于自己的浅显的看法。

下面的一个最原始的通过 JDBC ， 从数据库中获取数据的案例，可以看到的是，该方法中出现了大量重复的代码，因此，我们可以将这些共性的代码，从方法中抽取出来，然后使其成为一个通知，通过 Spring 的代理机制，切入到切点中，完成相应的功能。

![image-20230216140440580](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216140440580.png)

![image-20230216140543380](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216140543380.png)

🔔关于 AOP 的相关的概念点：

- Joinpoint (连接点) :就是方法
- Pointcut (切入点) :就是挖掉共性功能的方法
- Advice (通知) :就是共性功能，最终以一个方法的形式呈现Aspect(切面):就是共性功能与挖的位置的对应关系
- Target (目标对象) :就是挖掉功能的方法对应的类产生的对象，这种对象是无法直接完成最终工作
- Weaving (织入) :就是将挖掉的功能回填的动态过程
- Proxy (代理) :目标对象无法直接完成工作，需要对其进行功能回填，通过创建原始对象的代理对象实现
- lntroduction (引入/引介)︰就是对原始对象无中生有的添加成员变量或成员方法

![image-20230216165225694](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216165225694.png)

![image-20230216165238946](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216165238946.png)

![image-20230216165410819](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216165410819.png)

![image-20230216170608919](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230216170608919.png)

📚 为了使对 AOP 的设计理念更加的深入，通过下面的该案例进一步体会

- 定义切入点（务必绑定到接口上，而不是实现类上）
- 制作 AOP 怀绕通知，完成测试功能
- 注解配置 AOP
- 开启注解驱动支持

案例要求，编写一个通知完成对业务层的性能的监控案例

~~~ java

/**
 * @author peggy
 * @data 2023/2/16 21:23
 */
@Component
@Aspect
public class RunTimMonitorAdvice {

    //切入点，监控业务层接口
    @Pointcut("execution(* com.peggy.service.*Service.find*(..))")
    public void pt() {
    }

    @Around("pt()")
    public Object runtimeAround(ProceedingJoinPoint pjp) throws Throwable {
        long sum = 0L, startTime = 0L, endTime = 0L;
        Object proceed = null;
//        执行获取签名的信息
        Signature signature = pjp.getSignature();
//        获取当前的切入对象的类名
        String className = signature.getDeclaringTypeName();
//        获取当当前切入点对象的切入点的方法名
        String methodName = signature.getName();
        for (int i = 0; i < 10000; i++) {
//           获取执行前的当前时间
            startTime = System.currentTimeMillis();
            //切入点
            proceed = pjp.proceed(pjp.getArgs());
//          获取执行后的时间
            endTime = System.currentTimeMillis();
//          运行一次的时间 
            sum = sum + (endTime - startTime);
        }

        System.out.println(className + " " + methodName + " (万次)run time " + sum + "ms");
        return proceed;
    }

}
~~~

>  将通知对象织入到，服务层指定的切入点的方法当中，执行数次得到其执行的时间

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 4天***