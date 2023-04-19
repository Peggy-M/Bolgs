# Nacos服务端服务注册源码分析 - 篇四

## 服务端调用接口

嗨 ~~~ 上班除了无聊的摸鱼，我还学了一个新技能，偷偷写博客。。。。

------

我们先回忆一下之前的三篇文章

### 🕐[Nacos 客户端服务注册源码分析-篇一](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi-pian-yi)

### 🕑[Nacos 客户端服务注册源码分析-篇二](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-pian-er)

### 🕒[Nacos 客户端服务注册源码分析-篇三](http://124.221.240.218:8080/archives/nacoske-hu-duan-shi-li-zhu-ce-yuan-ma-fen-xi-pian-san)

我们之前的三篇内容都是分析关于客户端也就是 Spring 端的注册的整个流程，三篇内容其实总结起来都是围绕 _NacosNamingService_ 所展开的。

第一、二篇主要说的是 **Nacos** 源码中提供的客户端 **NamingTest** 测试 Dome ，如何一步步构建 **NamingService** 服务对象，以及调用实例注册方法完成客户端的注册。而第三篇主要将关于**SpringBoot** 在自动装配加载的整个过程当中，是如何自动构建 **NamingService** 对象，将本地的 **yaml** 文件配置扫描到注册实例当中完成实例注册的。

好，说到这里其实客户端的注册其以及基本完成，这里的客户端就是指 **SpringBoot** 的客户端向 **Nacos** 仅仅注册的这一步目前完成，但是对于 **Nacos** 的服务端，如何接受客户端的注入，并将其注入的信息进行存储，我们还需要接着往下看。。。这就是今天要说的。

------

先丢一张图，看一下我们之前三篇分析的整体流程叭，然后引出一个今天要完成的内容，，嗯  我觉得这样效果可能会比较好一些。。哈哈，这里的效果当然是指对我自己的理解咯。。

我们就直接从 **SpringBoot** 的启动自己装配 **discovery** 说起 

![image-20230413184217838](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/Nacos_%E6%B3%A8%E5%86%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.drawio.png)



哇哦，这个图画的我差点离开这个世界。。。可以点 [***这里***](https://viewer.diagrams.net/index.html?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Nacos_%E6%B3%A8%E5%86%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.drawio#R7V3bk6K4Gv9rqJp50EqAhPCI3fbubM1MTe08nJ2nUyhR2bXBRezL%2BetPwkUlBKWbS0S7p6ZbPgIC%2BX33Lx%2Bacff48lvkblbfQo%2BuNR14L5pxr%2Bm6DoDB%2FnDKa0ohVkZYRr6XkuCB8NP%2FH82IIKPufI9uCwPjMFzH%2FqZInIdBQOdxgeZGUfhcHLYI18Vv3bhLWiL8nLvrMvU%2FvhevsrvQrQP9d%2BovV%2Fk3Q2ynex7dfHB2J9uV64XPRyRjqhl3URjG6afHlzu65g8vfy7pcQ8Ve%2FcXFtEgrnPAXz9%2F%2F%2Fblv3%2FaP152P1Acvu6IBUfZZGzj1%2FyGqcfuP9sMo3gVLsPAXU8P1EkU7gKP8rMCtnUY8zUMN4wIGfFvGsev2WS6uzhkpFX8uM720hc%2F%2FosfPkbZ1q%2BjPfcv2ZmTjdd8I4ij16OD%2BOav432Hw5Kt%2FLj0%2FvhNVT62jLQNd9GcnnhWOfzcaEnjE%2BP0%2FeQyrqDhI2XXw46L6NqN%2FafidbgZPJf7cdmhThS5r0cDNqEfxNujM%2F%2FgBDYg4zRMMphlfGYLYBCG6xidGM4%2BpN%2Bfbx3dyIGU4OsNWDPTb3xy17vsIWg6XrMnOVmEybUdUIj%2F3YX5jtE2wZHDBkBz85LMab6ffVpmf5MTzXLCdhP5wXK8cOdxGPlMgDDmmGLNxhqxtKmpTSbaBGtTwjcdR5sijTiaQzjFMTSCtKnNf5P7%2FMzsjmfitzFaeuU5WcJKX90Zk4gF%2BLtrfxmwz3OGQBoxwhONYp%2BJHCfb8eh7XspplN27O0vOx8GcgYCdHE00dL%2F%2FTn4C%2BiKTh9nBByl0DPxqiVBGb3b2ERgzGVfETi5E3gbwEiShKZwWWsVThIvFljFeEaetIBM2Ryapi8zv7jxM4Yg0Z6o5OsclIdqE4ZJhcaI5D00wdxDOHGrPKz%2BmPzduIteemYYuInHhr9d34TqMkmONxWKhz%2Bece%2BIo%2FIce7fHwDCPM9iwj1%2FMZgApHeTbGp7BYEraV8DKtohTLrYDng%2Bo1QUZbHaldA1SjrQCWtyLDQCoVZP75V0FZXqyC1GsqSNRQQTaaUb2S14ss%2BpNGT%2F6c%2FkmXPmOHV4dN0l0YLPzlLmJXGQanmHQWiRTPf6oWI88ZkrkgmSUG6oQ%2FRvoSjzJdwfes6SI%2BJ2Ik3zBz5%2F8sE0SO5inT8pP5gR%2F77lp%2BPmtyRgIldzPgGywK9%2Flelh0GGoD9LBaSY00woW5QKY7f%2BtxKw95zUQlgOUKLoBVxWvdKO4FvlA7qBb%2FnrLbCPDRRoC2oPAgElaeXdZ5hSHTe%2FsDWRSTEQ1F6LSovVFN5QaXaC9odOlHcQZpqxEycH6A5ucs0sfmHiaHZ6KpdnxT3p3wfSHSzyK3NXJ%2FuXRtDKS8PLMJTVwZYDUXAuyI8yCgqCp30ELPBRlnecH%2FV0Sb3PFgygZoDuePKPkyuRAwY58UAAGYzxs9jHGbxiO7kADrl9sjMrjDyaHRscj250afR6Jj%2B%2Bd3W9WJBADdfDqRWL6E1e1ixbWiKLC%2BxDXVZPAR1ZRruRektmYZWXdOQqDQN9erIxodp2FQnnGFUMNaRjou82kxB5MrGlp20B7PR%2BjAb25cPTTODjeQDluTfmOFmTzQb8mSEzdj6LjHuppqDkpTYA0%2BGXQX%2FGhUm%2B8GmQwg2ZNmMR3FfPGpdlEmnY%2F6vM5POmXEDbh5foVVngJpWndlZwO9kUuTqoLTZrJlg4rD5ygBEAxoNCj5CjtSwyvDZB5H7gc9lOZcdw2e729Do0%2BfeICMm6YFrADd7BEd0kPxokuT95N5K97QAPiJkK%2FaZ9yPwWX0m6FOf61aw557WgreKSksvlg4ZAJU1qkwkdlc3Yl%2BO%2F7R3mU77T7DgPR2cqR7qRoyaDpSuNPeW609Fc6rVj30NaU4VB80k%2BY3jmocmkr%2Bi0CNe%2BdvxtlhdxE6anA%2BI9LMlJRVSvqpCp6v72UThhvv7aaVtcitHpFbuok1t5XqWKy10nJkYAbMdrQQBFDN21liimGyJYsrLs9sHPJYAvn%2Bp1qakGUblhl7tJLXHhlEhJJIxYoH4RlYcHNPZooOi47FeZjooYzpjz57t24NEpe3wvjpidbaDXZOjm9ZhNIunqzUHx9Z7ciSXP6eNkyTvKq4xTCGolhX%2BV62fKo3vYwFV%2FgxvI9YRBkeB2ukTQ5y68AacuZDqWim8sUh%2BZDoNADx1HtrRaToQEwYShYb7DG%2BYSozGRuENIT2894y7F31m7YWjtkp9hhTrs0Gl%2FGvPqVmxnrKnkFX1ysYr1BgzP%2FBuNQSuC6tnDVLWEb0mZpAMeoMONJh1Q5qm2jB1dUjzCnl%2By2ZDoWF4WUxvGoqZ3vwIc7xBntQNXJqtrDcp%2B7C46FYgcUFhemHZUQIy2liSICtILK8tKdcZSxA2uIJEsyLtddRnwyTF4t%2BLX2tm3lQdUBpWv91SoJLyUW1xYhn8hm1x1q0sR0qT6Fip1h9a5KDunOKKZYg9eRE3VV0uq8kY7wU8m6dild3nonS%2FIaFvCqFomdA3YK9Sf3grldRVZSEyCOFjKW1L%2BS43cgBzaoGGc%2FquzKpty73MqsyqOB5nMYVOM6sQXFAj1PcltPpsk1EXcRDoKsUIuqm1AbKudb0ZKS2YFhYS%2FUldUi0pz3N3VbZlKfEgB%2BrZYNyXImo2p7KwY9KLldxlvVjtk91Z%2BaJpJ%2BmFkEQkeddgpBHMD7%2BK0KRVIbT3vQ6gldd%2F7Fv1ahcemiTDW7sxAEYmStU7lpWyX616Vx9uns8pkhe7GdiwDa8dMwAJ3cVRXpd0ZASQPm2A%2FIUO1wkzULcF0Hf30Q%2BWmX2pCoIUQbMiyKVn4a%2BqCs02LFQh%2BGVJ2nFJV953Z5%2Bq6e7YuYo6H59qWiopDz5YQkocW0LYMtWdnaXELUnUnSfA73jTLd5lccKNUZ4bBxoxEkvUTAzQI5PUJskrBIDAsFdhmZ7tzKgTA7ZqmuanlrZ97aHHD7hm5SNeQnAM2X3%2B5Uuwjd1gTj9lCZovnqazpwl4V%2FBN%2BtHPhly7bSSKJ0n3DaNX48g6GWKTAqtJF8Kzr0w5al1f9s4ZxQHaJKFMHrgAbYAW3t%2B%2BCIni1AdhQAWcZKT6MlaGwaL2bsOqEZpnWJJ%2BQlJMdZXRI%2FISr96MGk1VJofofVk%2FzeZHv2adVNshKigrdV75DHreQuoSQWAZNm1HSIh5f%2FWuj602ojes0DypHdFTuqTaHkx36UHNqdLiQCaFKtVF%2B%2B995IVccpF7xc2p7dOym7vFCAjJVe3SEzY3FdxPcvdqIqwtGAdEKNmxJG5pv3l7CAaoSdT5HXVLktVqEnJTJcml6JcyD4NCD1FL5mHY2DLcll79agt5PwKl5T%2B91hbb%2BoePUV%2BK1K0KJGqjFzdVFMitCRr9iMKXQdUCQiC8zp7AmqWAemfxBvPDoqgvC%2Bp2%2B7KVtkfJL%2FM2ZEFuUSgPWdZ4z3wbMgQKPi%2BxZO1Xe7YoPl7HWV%2BK2HUbrthq38kLLigUXXPxiLrlSrUnVXXYslo3fIQtWwlbZnxzotCc%2FdcLInyUbV5u3NK%2Bqe5LachyeO6FLhTSXIRp8NE76Q1apG7vJLub3knQFBoA2yYaAwvbDB%2FJb%2BGMHbdSsm%2By845yR8ZDlHimzJEh%2BszAbTkyorSyZfmVfmXVx%2BuN3iCr6qZX1AZD9gjqpZZTsHjfzLaDtHjtisj3YWUlydesX66FC%2FO3hV4C819%2BDKP2y6FhhRbotj0HNJGgXMjp%2FhwYG6fGd9Sf48QrRjcRFSXHPPQYDXhu7I62m8eRG8xXYTTKviIXRWDM%2F%2FlgbGHsmu7Ce%2Flj%2BfdkWjBb5mt3uy0ctXaD5c5d0tFm7frBWfvovIQklUGCh4CXhbC%2FT5Bv8E%2F5QofzojN9BiVy4Wldg0yFVS8MOQhVQ4Bsw%2BKnPBZhFo%2FoMKZwY4UP%2FzobX2F2gsypPDsxI4hLy3aMelMXjXpZvUOvKU4IlOY4B6bZYR6bPB%2FJbtpM7X2aHQohCnxGs9vWyfFddd6SFHqWez5LlrwSjUyTJWBIIw98PF8d9pAclfeOvg4Fl3LlqfJeAAEpTF1Li16h3ZeK2zPTbei4S1k75M6JZ8hUnW6YJmpp1WqpmAchSay919dzM9n4oenqazpYty0DhIpDWPCWpMj31D0EeYuI%2B0QXOslbE4R10coi48vI9XxaaMliWx6wpPXEbUbNDaGemGmYctQc54mTflYlwJtKDp%2BCJV%2FAP9UclFlvTqOl%2BH2njzHYZ4tzcO1rgDpIybDNKORP9mBwsbtafQs9bshO%2Fw8%3D) 看到这个流程图的，对咯。推荐读者一个灰常不错的免费作图软件 ***[draw.io](https://app.diagrams.net/)*** 

其实说了这么多就是为了给下面的 Nacos 服务端做铺垫的，下面开始我们服务端的研究咯。。。

------

先看一眼 Nacos 官方提供的架构图咯

[***Nacos 架构官方文档***](https://nacos.io/zh-cn/docs/architecture.html)

![官方图](https://cdn.nlark.com/yuque/0/2019/jpeg/338441/1561217892717-1418fb9b-7faa-4324-87b9-f1740329f564.jpeg)

![nacos-stru.drawio2](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/nacos-stru.drawio2.png)

从第一张图中就可以看到，消费者(Consumer) 通过 NacosServer（服务）调用提供者(Provider)，而他们之间的调用其实是通过中间 OpenAPI 的 URL 请求进行的。

而对于 ***Consumer*** 与 ***NacosServer*** 对应的是客户端也就是平常生产环境中对应的 Spring 项目，对于 OpenAPI 是在 Nacos 的服务层，是 Nacos 对外开放的请求接口，客户端的 discovery 只需要向服务端发起请求，将客户端封装实体信息 instance 相应到服务端即可。

所以如果要研究其服务端正确的打开方式就先从 OpenAPI 入手，那就继续接下来的探究咯。

[***官方Open API 指南***](https://nacos.io/zh-cn/docs/open-api.html)

![image-20230413185733188](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413185733188.png) 

其实，通过第三篇以及上面的这个图中可以看到，其实我们的客户端最终是向服务端发起了一个 POST 请求，请求的接口为 **/nacos/v1/ns/instance** ,那么这个请求到底指向的是 Nacos 服务端源码中的那一块呢？

哎~~ 我们看下面这个 Nacos 源码模块的细分图：

![image-20230413192550481](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413192550481.png)

既然我们探究是 <font color='orange'>***服务注册***</font> 那么我们就到 <font color='green'>***naming***</font> 这个模块下看看有没有我们想要的呢？

![image-20230413192900073](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413192900073.png) 

在 naming 模块下我们看到有很多的 Controller ，进入实例注册的请求瞅瞅咯。

![image-20230413193420833](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413193420833.png) 

![image-20230413193431387](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413193431387.png) 

![image-20230413193618946](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413193618946.png) 

没错，看到这熟悉的请求路径，就是我们要找的客户端注册请求接口咯，为了验证这个猜想，我们可以直接运行通过 Debug 赖。

先启动，服务端的 Nacos 端，随后启动 Spring 客户端 9002 的实例注册。

<font color='orange'>***客户端***</font>  发起注册请求

![image-20230413194050632](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413194050632.png)

<font color='red'>***服务端***</font> 相应对象的请求

![image-20230413194248504](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413194248504.png) 

可以在服务端中的 instance 实例中，看到客户端注册的信息。

![image-20230413194349504](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413194349504.png) 

![image-20230413194531616](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230413194531616.png) 

至此。。。从客户端请求到服务端注册这个流程完成。

但怎么感觉好像却点什么呢？

对，缺少~......  服务端详细的注册过程，只是看到服务端响应，但从客户端响应到的 instance 实例到底是什么注册返回到前端，这个过程却只字未提，对咯。我猜应该是作者不会 哈哈~~~

既然标题都说是服务端源码分析，那必须分析到位捏，，我们接着往下看。。。

前端看到，在服务端接受到响应后，会调用 <font color='orange'>**getInstanceOperator().registerInstance(namespaceId, serviceName, instance)**</font>

所以我们就必须从该方法进行剖析，进入该方法我们瞅瞅到底做了什么捏。

~~~ java
 getInstanceOperator().registerInstance(namespaceId, serviceName, instance);
~~~

~~~ java
private InstanceOperator getInstanceOperator() {
    //判断实例是否走的 Grpc 协议
    return upgradeJudgement.isUseGrpcFeatures() ? instanceServiceV2 : instanceServiceV1;
}
~~~

~~~ java
private InstanceOperatorClientImpl instanceServiceV2;
~~~

可以看到是对当前 Nacos 网络传输是否采用的是 Grpc 协议，很明显返回的这里的实例 instanceServiceV2 而该实例是一个 <font color='orange'>*** InstanceOperatorClientImpl***</font> 对象。

那就进入 **InstanceOperatorClientImpl** 里面瞅瞅咯，看看具体是怎么实现在该接口中调用的 <font color='orange'>***registerInstance***</font> 方法的。

~~~ java
@Override
public void registerInstance(Service service, Instance instance, String clientId) {
    final ClientOperationService operationService = chooseClientOperationService(instance);
    operationService.registerInstance(service, instance, clientId);
}
~~~

~~~ java
/**
 * This method creates {@code IpPortBasedClient} if it don't exist.
 * 这里需要说明一点就是 nacos 在 2.0 版本之后，对将原来的 Http 协议改为了 Grap 协议。
 */
@Override
public void registerInstance(String namespaceId, String serviceName, Instance instance) {
    //判断是否为瞬时对象（临时客户端）
    /*
    	这里的瞬时对象其实就是没有被初始化的一个对象，该对象是一个临时的客户端
    */
    boolean ephemeral = instance.isEphemeral();
    //获取客户端的 ID
    String clientId = IpPortBasedClient.getClientId(instance.toInetAddr(), ephemeral);
    //通过客户端的 ID 创建客户端的连接
    createIpPortClientIfAbsent(clientId);
    //获取服务
    Service service = getService(namespaceId, serviceName, ephemeral);
    //完成服务的具体注册
    clientOperationService.registerInstance(service, instance, clientId);
}
~~~
<font color='orange'>**EphemeralClientOperationServiceImpl**</font> 类下的 <font color='green'>**registerInstance**</font> 实例负责的就是服务端的实例注册

~~~ java
@Override
public void registerInstance(Service service, Instance instance, String clientId) {
    //确保 service 是单利存在的
    Service singleton = ServiceManager.getInstance().getSingleton(service);
    //根据注册的 id 获取当前客户端的实体
    Client client = clientManager.getClient(clientId);
    if (!clientIsLegal(client, clientId)) {
        return;
    }
    //将客户端的实例信息转化为服务端的实例信息
    InstancePublishInfo instanceInfo = getPublishInfo(instance);
    //添加实例信息到转存入 Client 实例当中
    client.addServiceInstance(singleton, instanceInfo);
    //设置客户端最后一次更新时间为当前的时间
    client.setLastUpdatedTime();
    //建立 Service 与 ClinetId 的关系
    NotifyCenter.publishEvent(new ClientOperationEvent.ClientRegisterServiceEvent(singleton, clientId));
    NotifyCenter
            .publishEvent(new MetadataEvent.InstanceMetadataEvent(singleton, instanceInfo.getMetadataId(), false));
}
~~~

服务器的客户的实体 Client 模型

~~~ java

/**
 * Nacos naming client.
 *  客户端的抽象概念存储在服务器端的naco命名模块上。它用于存储客户端已发布和订阅的服务
 * <p>The abstract concept of the client stored by on the server of Nacos naming module. It is used to store which
 * services the client has published and subscribed.
 * 
 * @author xiweng.yy
 */
public interface Client {
    
    /**
     * Get the unique id of current client.
     */
    //获取当前客户端的唯一 Id
    String getClientId();
    
    /**
     * Whether is ephemeral of current client.
     */
    //当前客户的是否短暂的
    boolean isEphemeral();
    
    /**
     * Set the last time for updating current client as current time.
     */
    //设置该客户端的最后更新时间为当前的时间
    void setLastUpdatedTime();
    
    /**
     * Get the last time for updating current client.
     */
    //获取最近一次客户更新的时间
    long getLastUpdatedTime();
    
    /**
     * Add a new instance for service for current client.
     */
    //为当前客户端添加服务的新实例
    boolean addServiceInstance(Service service, InstancePublishInfo instancePublishInfo);
    
    /**
     * Remove service instance from client.
     */
    //从客户端删除服务实例
    InstancePublishInfo removeServiceInstance(Service service);
    
    /**
     * Get instance info of service from client.
     */
    //从客户端获取实例的服务信息
    InstancePublishInfo getInstancePublishInfo(Service service);
    
    /**
     * Get all published service of current client.
     */
    //获取当前客户端已经发布的所有信息
    Collection<Service> getAllPublishedService();
    
    /**
     * Add a new subscriber for target service.
     */
    //为目标服务添加新的订阅者
    boolean addServiceSubscriber(Service service, Subscriber subscriber);
    
    /**
     * Remove subscriber for service.
     */
    //删除服务的订阅者
    boolean removeServiceSubscriber(Service service);
    
    /**
     * Get subscriber of service from client.
     */
    //从客户端获取服务的订阅者
    Subscriber getSubscriber(Service service);
    
    /**
     * Get all subscribe service of current client.
     */
    //获取当前客户的所有订阅服务
    Collection<Service> getAllSubscribeService();
    
    /**
     * Generate sync data.
     */
    ClientSyncData generateSyncData();
    
    /**
     * Whether current client is expired.
     */
    boolean isExpire(long currentTime);
    
    /**
     * Release current client and release resources if neccessary.
     */
    void release();
}
~~~

可以看到在注册实例当中有一个获取客户的 ID 在 nacos 的 2.0 版本以后，添加了一个新的 <font color='orange'>**Client 模型**</font> ，对于每一个 Client 都有自己的唯一 id 标识 clientId 。

Client 负责管理一个客户端的服务实例注册 Publish 和 订阅服务 Subscribe 。我们可以看一下这个模型其实就是一个接口。

## ServiceManager 类的具体实现

这里有一个有意思一点，通过 <font color='orange'>**ServiceManager**</font> 调用其内部的静态方法 <font color='green'>**getInstance**</font> ，通过内部的 <font color='orange'>**ServiceManager **</font> 对象调 <font color='green'>**getSingleton**</font> 获取单利服务 。

那么这个单利是怎么实现的呢？我们接着探究。。。

~~~ java
/**
 * Nacos service manager for v2.
 *
 * @author xiweng.yy
 */
public class ServiceManager {
    
    private static final ServiceManager INSTANCE = new ServiceManager();
    
    //单例Service，可以查看Service的equals和hasCode方法
	private final ConcurrentHashMap<Service, Service> singletonRepository;
	//namespace 的所有 Service
	private final ConcurrentHashMap<String, Set<Service>> namespaceSingletonMaps;
    
    private ServiceManager() {
        singletonRepository = new ConcurrentHashMap<>(1 << 10);
        namespaceSingletonMaps = new ConcurrentHashMap<>(1 << 2);
    }
    
}
~~~

获取 Service 单利

~~~ java
/**
 * Get singleton service. Put to manager if no singleton.
 *
 * @param service new service
 * @return if service is exist, return exist service, otherwise return new service
 */
public Service getSingleton(Service service) {
    singletonRepository.putIfAbsent(service, service);
    Service result = singletonRepository.get(service);
    namespaceSingletonMaps.computeIfAbsent(result.getNamespace(), (namespace) -> new ConcurrentHashSet<>());
    namespaceSingletonMaps.get(result.getNamespace()).add(result);
    return result;
}
~~~

## clientManager 获取 Client 模型

~~~ java
 @Override
 public Client getClient(String clientId) {
     return getClientManagerById(clientId).getClient(clientId);
 }
~~~

这是一个接口这里我们要看它对应的一个实现类 **ConnectionBasedClientManager**，这个实现类负责管理长连接 clientId 与 Client 模型的映射关系

~~~ java
@Override
public Client getClient(String clientId) {
    return clients.get(clientId);
}
~~~

## Clinet 实例 AbstractClient 

负责存储当前客户端的服务注册表，既 Service 与 instance 的关系。

注意 <font color='orange'>**对于单个客户端来说，同一个服务只能注册处一个实例**</font>

~~~ java
@Override
public boolean addServiceInstance(Service service, InstancePublishInfo instancePublishInfo) {
    if (null == publishers.put(service, instancePublishInfo)) {
        MetricsMonitor.incrementInstanceCount();
    }
    NotifyCenter.publishEvent(new ClientEvent.ClientChangedEvent(this));
    Loggers.SRV_LOG.info("Client change for service {}, {}", service, getClientId());
    return true;
}
~~~

## ClientOperationEvent.ClientRegisterServiceEvent

```java
private final ConcurrentMap<Service, Set<String>> publisherIndexes = new ConcurrentHashMap<>();
    
private final ConcurrentMap<Service, Set<String>> subscriberIndexes = new ConcurrentHashMap<>();

private void handleClientOperation(ClientOperationEvent event) {
    Service service = event.getService();
    String clientId = event.getClientId();
    if (event instanceof ClientOperationEvent.ClientRegisterServiceEvent) {
        addPublisherIndexes(service, clientId);
    } else if (event instanceof ClientOperationEvent.ClientDeregisterServiceEvent) {
        removePublisherIndexes(service, clientId);
    } else if (event instanceof ClientOperationEvent.ClientSubscribeServiceEvent) {
        addSubscriberIndexes(service, clientId);
    } else if (event instanceof ClientOperationEvent.ClientUnsubscribeServiceEvent) {
        removeSubscriberIndexes(service, clientId);
    }
}

//建立Service与发布Client的关系
private void addPublisherIndexes(Service service, String clientId) {
    publisherIndexes.computeIfAbsent(service, (key) -> new ConcurrentHashSet<>());
    publisherIndexes.get(service).add(clientId);
    NotifyCenter.publishEvent(new ServiceEvent.ServiceChangedEvent(service, true));
}
```
