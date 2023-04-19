# Nacos 客户端服务注册源码分析-篇一

> 版本说明：
>
> 源码版本 [**nacos-1.4.2**](https://github.com/alibaba/nacos/releases?page=2) 

![image-20230406165130271](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230406165130271.png)

### Nacos 的核心功能点

**服务注册：** Nacos Client 会通过发送 REST 请求的方式向 Nacos Server 注册自己的服务，提供自身的元数据，比如 ip 地址以及端口等信息。Nacos Server 在接受到注册请求后，就会把元数据信息存储在一个双层的内存 Map 当中

**服务心跳**：在服务注册后，Nacos Client会维护一个定时心跳来持续通知Nacos Server，说明服务一直处于可用状态，防止被剔除。默认5s发送一次心跳。

**服务健康检查**：Nacos Server会开启一个定时任务用来检查注册服务实例的健康情况，对于超过15s没有收到客户端心跳的实例会将它的healthy属性置为false(客户端服务发现时不会发现)，如果某个实例超过30秒没有收到心跳，直接剔除该实例(被剔除的实例如果恢复发送心跳则会重新注册)

**服务发现**：服务消费者（Nacos Client）在调用服务提供者的服务时，会发送一个REST请求给Nacos Server，获取上面注册的服务清单，并且缓存在Nacos Client本地，同时会在Nacos Client本地开启一个定时任务定时拉取服务端最新的注册表信息更新到本地缓存

**服务同步**：Nacos Server集群之间会互相同步服务实例，用来保证服务信息的一致性。 

### Nacos 服务端原理

![image-20230408123438352](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230408123438352.png)

### nacos 客户端原理

![20190703005828565](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/20190703005828565.png)

### 源码分析

#### 服务注册信息

我们在使用 nacos 的时候，其实只是在我们的项目当中引入了一个 nacos 的客户端的依赖，我们在使用的时候直接配置注入即可，然后通过 nacos 的 PC 客户端进行登录就可看到具体的实例信息，但这中间发生了什么我们是不知道的。

既然研究源码我们就从流程分析，一步一步的按照我们实际的操作去剖析，所以我们就从最基础的客户端的注册开始。



在这个目录包 _package com.alibaba.nacos.client_ 下 nacos 提供了一套客户端注册的测试类 _NamingTest_ 我们就从该类开始

##### nacos 的连接信息

~~~ java
//nacos连接信息
Properties properties = new Properties();
properties.put(PropertyKeyConst.SERVER_ADDR, "127.0.0.1:8848"); //服务器地址
properties.put(PropertyKeyConst.USERNAME, "nacos"); //主机名
properties.put(PropertyKeyConst.PASSWORD, "nacos"); //主机密码
~~~

- Service地址：Nacos 服务器的地址，属性 key 为了 *SERVER_ADDR* = "serverAddr" 
- 用户名：连接 Nacos 服务的用户名，属性 key 为 *USERNAME* = "username"
- 密码：连接 Nacos 服务的密码，属性 key 为 *PASSWORD* = "password"

##### 实例信息

~~~ java
//注册实例信息
Instance instance = new Instance();
instance.setIp("1.1.1.1");
instance.setPort(800);
instance.setWeight(2);
Map<String, String> map = new HashMap<String, String>();
map.put("netType", "external");
map.put("version", "2.0");
instance.setMetadata(map);
~~~

注册的实例信息用 _Instance_ 去作为基础的承载的，在 _Instance_ 对象中可以到由两部分组成，分别是实例信息（*user extended attributes.*）和元数据（*add meta data.*）

**基础实例信息 - user extended attributes**

~~~ java
private String instanceId; //unique id of this instance.实例的唯一ID；

private String ip; //instance ip.实例IP，提供给消费者进行通信的地址；

private int port; //instance port.端口，提供给消费者访问的端口；

private double weight = 1.0D; //instance weight.权重，当前实例的权限，浮点类型（默认1.0D）；

private boolean healthy = true; //instance health status.健康状况，默认true；

private boolean enabled = true; //If instance is enabled to accept request.实例是否准备好接收请求，默认true；

private boolean ephemeral = true; //If instance is ephemeral.实例是否为瞬时的，默认为true；

private String clusterName; //cluster information of instance.实例所属的集群名称；

private String serviceName; //Service information of instance.实例的服务信息；

~~~

**存储元数据 - add meta data**

~~~ java
private Map<String, String> metadata = new HashMap<String, String>();

public Map<String, String> getMetadata() {
    return this.metadata;
}

public void setMetadata(final Map<String, String> metadata) {
    this.metadata = metadata;
}

/**
 * add meta data.
 *
 * @param key   meta data key
 * @param value meta data value
 */
public void addMetadata(final String key, final String value) {
    if (metadata == null) {
        metadata = new HashMap<String, String>(4);
    }
    metadata.put(key, value);
}
    
~~~

可以看到的 _meta data_ 元数据是通过一个 Map 即可的 key-value 键值对进行维护的。

关于这两个键值具体是什么我们可以在 _NamingTest_ 的该 Dome 中找到具体指向。

~~~ java
Map<String, String> map = new HashMap<String, String>();
map.put("netType", "external");
map.put("version", "2.0");
~~~

- netType：顾名思义，网络类型，这里的值为external，也就是外网的意思；
- version：版本，Nacos的版本，这里是2.0这个大版本。

当然在 _Instance_ 对象当中对上述的基本信息都提供了具体的 get 方法例如：

~~~ java
//获取实例的心跳间隔
public long getInstanceHeartBeatInterval() {
    return getMetaDataByKeyWithDefault(PreservedMetadataKeys.HEART_BEAT_INTERVAL,
            Constants.DEFAULT_HEART_BEAT_INTERVAL);
}
//获取实例的心跳超时
public long getInstanceHeartBeatTimeOut() {
    return getMetaDataByKeyWithDefault(PreservedMetadataKeys.HEART_BEAT_TIMEOUT,
            Constants.DEFAULT_HEART_BEAT_TIMEOUT);
}
//获取 ip 的删除超时
public long getIpDeleteTimeout() {
    return getMetaDataByKeyWithDefault(PreservedMetadataKeys.IP_DELETE_TIMEOUT,
            Constants.DEFAULT_IP_DELETE_TIMEOUT);
}
//获取实例 ID 生成器
public String getInstanceIdGenerator() {
    return getMetaDataByKeyWithDefault(PreservedMetadataKeys.INSTANCE_ID_GENERATOR,
            Constants.DEFAULT_INSTANCE_ID_GENERATOR);
}
~~~

上面的get方法在需要元数据默认值时会被用到：

- preserved.heart.beat.interval：心跳间隙的key，默认为5s，也就是默认5秒进行一次心跳；
- preserved.heart.beat.timeout：心跳超时的key，默认为15s，也就是默认15秒收不到心跳，实例将会标记为不健康；
- preserved.ip.delete.timeout：实例IP被删除的key，默认为30s，也就是30秒收不到心跳，实例将会被移除；
- preserved.instance.id.generator：实例ID生成器key，默认为simple；

我们可以 _getInstanceHeartBeatInterval_ 该方法为例看一下整个过程

1. 当我们执行 _getInstanceHeartBeatInterval()_ 方法的时候会默认的调用该方法下的 _getMetaDataByKeyWithDefault(final String key**,** final long defaultValue)_ 方法

   ~~~ java
   private long getMetaDataByKeyWithDefault(final String key, final long defaultValue) {
       //返回为默认的元数据值 metadata
       if (getMetadata() == null || getMetadata().isEmpty()) {
           return defaultValue;
       }
       //根据具体的 key 获取预设的 matedata
       final String value = getMetadata().get(key);
       if (!StringUtils.isEmpty(value) && value.matches(NUMBER_PATTERN)) {
           return Long.parseLong(value);
       }
       return defaultValue;
   }
   ~~~

2. 如果当前对象元数据 _metadata_ 为空则返回默认值，以心跳检测为例，默认值为 *DEFAULT_HEART_BEAT_INTERVAL* = TimeUnit.*SECONDS*.toMillis(5) 也就是 5 秒，如果说不为空则会通过具体的 key （心跳间隙、心跳超时、实例 IP 被删除、实例 ID 生成器）等进行获取预设值

#### NamingService接口

~~~ java
NamingService namingService = NacosFactory.createNamingService(properties);
namingService.registerInstance("nacos.test.1", instance);
~~~

其实 _namingService_ 是通过工厂类 _NacosFactory_ 下的 _createNamingService_ 创建的

~~~ java
/**
 * Create naming service.
 *
 * @param properties init param
 * @return Naming
 * @throws NacosException Exception
 */
public static NamingService createNamingService(Properties properties) throws NacosException {
    return NamingFactory.createNamingService(properties);
}
~~~

这里通过反射获取 **com.alibaba.nacos.client.naming.NacosNamingService** 包下的实体对象

~~~ java
/**
 * Create a new naming service.
 *
 * @param properties naming service properties
 * @return new naming service
 * @throws NacosException nacos exception
 */
public static NamingService createNamingService(Properties properties) throws NacosException {
    try {
        Class<?> driverImplClass = Class.forName("com.alibaba.nacos.client.naming.NacosNamingService");
        Constructor constructor = driverImplClass.getConstructor(Properties.class);
        NamingService vendorImpl = (NamingService) constructor.newInstance(properties);
        return vendorImpl;
    } catch (Throwable e) {
        throw new NacosException(NacosException.CLIENT_INVALID_PARAM, e);
    }
}
~~~

可以看到的 _NamingService_ 接口提供了大量的接口

![image-20230411202432897](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230411202432897.png)

例如以下几个重要的接口：

~~~ java
/**
 * register a instance to service. 将实例注册到服务
 *
 * @param serviceName name of service
 * @param ip          instance ip
 * @param port        instance port
 * @throws NacosException nacos exception
 */
void registerInstance(String serviceName, String ip, int port) throws NacosException;

/**
 * deregister instance from a service. 从服务取消一个注册实例
 *
 * @param serviceName name of service
 * @param ip          instance ip
 * @param port        instance port
 * @throws NacosException nacos exception
 */
void deregisterInstance(String serviceName, String ip, int port) throws NacosException;

/**
 * get all instances of a service. 获取服务的所有实例
 *
 * @param serviceName name of service
 * @return A list of instance
 * @throws NacosException nacos exception
 */
List<Instance> getAllInstances(String serviceName) throws NacosException;

/**
 * Get qualified instances of service. 获取合格的服务实例
 *
 * @param serviceName name of service.
 * @param healthy     a flag to indicate returning healthy or unhealthy instances
 * @return A qualified list of instance
 * @throws NacosException nacos exception
 */
List<Instance> selectInstances(String serviceName, boolean healthy) throws NacosException;

/**
 * Select one healthy instance of service using predefined load balance strategy.
 * 使用预定义的负载平衡策略选择一个正常运行的服务实例。
 * @param serviceName name of service
 * @return qualified instance
 * @throws NacosException nacos exception
 */
Instance selectOneHealthyInstance(String serviceName) throws NacosException;

/**
 * Subscribe service to receive events of instances alteration.
 * 订阅服务以接收实例变更事件。
 * @param serviceName name of service
 * @param listener    event listener
 * @throws NacosException nacos exception
 */
void subscribe(String serviceName, EventListener listener) throws NacosException;

/**
 * Unsubscribe event listener of service.
 * 取消订阅服务的事件侦听器。
 * @param serviceName name of service
 * @param listener    event listener
 * @throws NacosException nacos exception
 */
void unsubscribe(String serviceName, EventListener listener) throws NacosException;

/**
 * Get all service names from server.
 * 从服务器获取所有服务名称。
 * @param pageNo   page index
 * @param pageSize page size
 * @return list of service names
 * @throws NacosException nacos exception
 */
ListView<String> getServicesOfServer(int pageNo, int pageSize) throws NacosException;

/**
 * Shutdown the resource service.
 * 关闭资源服务。
 * @throws NacosException exception.
 */
void shutDown() throws NacosException;
~~~

可以看到在 _NamingService_ 当中其实提供了大量的功能性的接口，我们可以在不用的场景去选择适配的功能进行调用。

既然说到这里了，不能只说一半，我们就实例服务注册方法 **registerInstance** 进一步，探究一下其具体的调用叭！

~~~ java
   //用户调用服务进行注册
   NamingService namingService = NacosFactory.createNamingService(properties);
   namingService.registerInstance("nacos.test.1", instance);
~~~

~~~ java
   //被调用的方法
   void registerInstance(String serviceName, String ip, int port) throws NacosException;
~~~

~~~ java
   public void registerInstance(String serviceName, String ip, int port) throws NacosException {
       //这里进行默认的初始化
       registerInstance(serviceName, ip, port, Constants.DEFAULT_CLUSTER_NAME);
   }
~~~

~~~ java
   public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
       //心跳检测-检查有关保持活动状态的实例参数。
       NamingUtils.checkInstanceIsLegal(instance);
       //获取分组名
       String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
       if (instance.isEphemeral()) {
           BeatInfo beatInfo = beatReactor.buildBeatInfo(groupedServiceName, instance);
           beatReactor.addBeatInfo(groupedServiceName, beatInfo);
       }
       //注册服务
       serverProxy.registerService(groupedServiceName, groupName, instance);
   }
~~~

~~~ java
      /**
       * <p>Check instance param about keep alive.</p>
       *
       * <pre>
       * heart beat timeout must > heart beat interval
       * ip delete timeout must  > heart beat interval
       * </pre>
       *
       * @param instance need checked instance
       * @throws NacosException if check failed, throw exception
       */
      public static void checkInstanceIsLegal(Instance instance) throws NacosException {
          if (instance.getInstanceHeartBeatTimeOut() < instance.getInstanceHeartBeatInterval()
                  || instance.getIpDeleteTimeout() < instance.getInstanceHeartBeatInterval()) {
              //实例“心跳间隔”必须小于“心跳超时”和“IP 删除超时”。
              throw new NacosException(NacosException.INVALID_PARAM,
                      "Instance 'heart beat interval' must less than 'heart beat timeout' and 'ip delete timeout'.");
          }
      }
~~~

~~~ java
      /**
       * Returns a combined string with serviceName and groupName. serviceName can not be nil.
       *
       * <p>In most cases, serviceName can not be nil. In other cases, for search or anything, See {@link
       * com.alibaba.nacos.api.naming.utils.NamingUtils#getGroupedNameOptional(String, String)}
       *
       * <p>etc:
       * <p>serviceName | groupName | result</p>
       * <p>serviceA    | groupA    | groupA@@serviceA</p>
       * <p>nil         | groupA    | threw IllegalArgumentException</p>
       *
       * @return 'groupName@@serviceName'
       */
      public static String getGroupedName(final String serviceName, final String groupName) {
          if (StringUtils.isBlank(serviceName)) {
              //参数“服务名称”是非法的，服务名称为空
              throw new IllegalArgumentException("Param 'serviceName' is illegal, serviceName is blank");
          }
          if (StringUtils.isBlank(groupName)) {
              //参数“组名称”是非法的，组名称为空
              throw new IllegalArgumentException("Param 'groupName' is illegal, groupName is blank");
          }
          final String resultGroupedName = groupName + Constants.SERVICE_INFO_SPLITER + serviceName;
          return resultGroupedName.intern();
      }
~~~

~~~ java
      package com.alibaba.nacos.client.naming.net;
      /**
       * register a instance to service with specified instance properties.
       * 使用指定的实例属性将实例注册到服务。
       * @param serviceName name of service
       * @param groupName   group of service
       * @param instance    instance to register
       * @throws NacosException nacos exception
       */
      //代理类 NamingProxy 下的注册服务方法
      public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {
          
          NAMING_LOGGER.info("[REGISTER-SERVICE] {} registering service {} with instance: {}", namespaceId, serviceName,
                  instance);
          
          final Map<String, String> params = new HashMap<String, String>(16);
          params.put(CommonParams.NAMESPACE_ID, namespaceId);
          params.put(CommonParams.SERVICE_NAME, serviceName);
          params.put(CommonParams.GROUP_NAME, groupName);
          params.put(CommonParams.CLUSTER_NAME, instance.getClusterName());
          params.put("ip", instance.getIp());
          params.put("port", String.valueOf(instance.getPort()));
          params.put("weight", String.valueOf(instance.getWeight()));
          params.put("enable", String.valueOf(instance.isEnabled()));
          params.put("healthy", String.valueOf(instance.isHealthy()));
          params.put("ephemeral", String.valueOf(instance.isEphemeral()));
          params.put("metadata", JacksonUtils.toJson(instance.getMetadata()));
          
          reqApi(UtilAndComs.nacosUrlInstance, params, HttpMethod.POST);
          
      }
~~~

~~~ java
      public String reqApi(String api, Map<String, String> params, String method) throws NacosException {
              return reqApi(api, params, Collections.EMPTY_MAP, method);
      }
~~~

~~~ java
      public String reqApi(String api, Map<String, String> params, Map<String, String> body, String method)
              throws NacosException {
          return reqApi(api, params, body, getServerList(), method);
      }
~~~

~~~ java
      /**
       * Request api.
       *
       * @param api     api
       * @param params  parameters
       * @param body    body
       * @param servers servers
       * @param method  http method
       * @return result
       * @throws NacosException nacos exception
       */
      public String reqApi(String api, Map<String, String> params, Map<String, String> body, List<String> servers,
              String method) throws NacosException {
          
          params.put(CommonParams.NAMESPACE_ID, getNamespaceId());
          
          if (CollectionUtils.isEmpty(servers) && StringUtils.isBlank(nacosDomain)) {
              throw new NacosException(NacosException.INVALID_PARAM, "no server available");
          }
          
          NacosException exception = new NacosException();
          
          if (StringUtils.isNotBlank(nacosDomain)) {
              for (int i = 0; i < maxRetry; i++) {
                  try {
                      return callServer(api, params, body, nacosDomain, method);
                  } catch (NacosException e) {
                      exception = e;
                      if (NAMING_LOGGER.isDebugEnabled()) {
                          NAMING_LOGGER.debug("request {} failed.", nacosDomain, e);
                      }
                  }
              }
          } else {
              Random random = new Random(System.currentTimeMillis());
              int index = random.nextInt(servers.size());
              
              for (int i = 0; i < servers.size(); i++) {
                  String server = servers.get(index);
                  try {
                      return callServer(api, params, body, server, method);
                  } catch (NacosException e) {
                      exception = e;
                      if (NAMING_LOGGER.isDebugEnabled()) {
                          NAMING_LOGGER.debug("request {} failed.", server, e);
                      }
                  }
                  index = (index + 1) % servers.size();
              }
          }
          
          NAMING_LOGGER.error("request: {} failed, servers: {}, code: {}, msg: {}", api, servers, exception.getErrCode(),
                  exception.getErrMsg());
          
          throw new NacosException(exception.getErrCode(),
                  "failed to req API:" + api + " after all servers(" + servers + ") tried: " + exception.getMessage());
~~~

#### NacosNamingService的实现

前面提到过在 _NacosService_ 中是通过 _registerInstance_ 方法进行服务层的注册的，而对于该方法只需要提过两个参数服务名称 _serviceName_ 与实例信息对象 _instance_ 。

~~~ java
@Override
public void registerInstance(String serviceName, Instance instance) throws NacosException {
    registerInstance(serviceName, Constants.DEFAULT_GROUP, instance);
}
~~~



对于这两个参数其实最大的作用就是设置了当前实例的分组信息。在 Nacos 中是通过Namespace、group、Service、Cluster 进行一级一级的隔离的，在不指定的前提下一般都是默认调用  **String DEFAULT_GROUP = "DEFAULT_GROUP"** 来实现的。

![image-20230411214042881](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230411214042881.png)

而在 _registerInstance_ 方法中有重要的两个方法

- 检查心跳时间设置的对不对（心跳默认为5秒）
- 通过 _NamingClientProxy_ 这个代理来执行服务注册操作

~~~ java
public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
    //检查有关保持活动状态的实例参数。
    NamingUtils.checkInstanceIsLegal(instance);
    String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
    if (instance.isEphemeral()) {
        BeatInfo beatInfo = beatReactor.buildBeatInfo(groupedServiceName, instance);
        beatReactor.addBeatInfo(groupedServiceName, beatInfo);
    }
    //接口代理
    serverProxy.registerService(groupedServiceName, groupName, instance);
}
~~~

其实在第一步通过反射获取 **com.alibaba.nacos.client.naming.NacosNamingService** 包，进行创建 _NamingService_ 的实现类 _NacosNamingService_ 的时候就已经通过默认的构造方法完成了初始化。

~~~ java
//反射获取对象
NamingService namingService = NacosFactory.createNamingService(properties);

/**
 * Create a new naming service.
 *
 * @param properties naming service properties
 * @return new naming service
 * @throws NacosException nacos exception
 */
public static NamingService createNamingService(Properties properties) throws NacosException {
    try {
        Class<?> driverImplClass = Class.forName("com.alibaba.nacos.client.naming.NacosNamingService");
        //调用 Properties 参数的构造方法
        Constructor constructor = driverImplClass.getConstructor(Properties.class);
        NamingService vendorImpl = (NamingService) constructor.newInstance(properties);
        return vendorImpl;
    } catch (Throwable e) {
        throw new NacosException(NacosException.CLIENT_INVALID_PARAM, e);
    }
}

public NacosNamingService(Properties properties) throws NacosException {
    //在默认的 NacosNamingService(Properties properties) 构造方法中调用初始化方法 init 
    init(properties);
}
~~~

