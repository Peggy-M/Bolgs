# Nacos 客户端服务发现源码分析-篇六

### 🕐[Nacos 客户端服务注册源码分析-篇一](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi-pian-yi)

### 🕑[Nacos 客户端服务注册源码分析-篇二](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-pian-er)

### 🕒[Nacos 客户端服务注册源码分析-篇三](http://124.221.240.218:8080/archives/nacoske-hu-duan-shi-li-zhu-ce-yuan-ma-fen-xi-pian-san)

### 🕓[Nacos 服务端服务注册源码分析-篇四](http://124.221.240.218:8080/archives/nacosfu-wu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi---pian-si)

### 🕔[Nacos 服务端健康检查-篇五](http://124.221.240.218:8080/archives/nacosfu-wu-duan-jian-kang-jian-cha-pian-wu)

------

## 注册发现流程

前面的几篇探究了客户端服务注册，服务端发现，以及服务端的健康检测方面的源码。

如果通过一个图去连接他们之间的关系其实，Nacos 客户端的服务发现，其实就是客户端实体参数封装，调用服务端接口请求，获得返回实例结果列表的过程。

![image-20230418211222559](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418211222559.png)

对于 NacosService 获取服务列表，在获取服务列表的过程中还涉及到通讯流程协议（Http / gPRC），订阅流程、故障转移等。

那么对于这些到底是怎么实现的呢？我们接着往下一点点的剖析关于 NamingTest 测试类中提供的源码流程。

~~~ java
@Test
public void testServiceList() throws Exception {
    ......
    NamingService namingService = NacosFactory.createNamingService(properties);
namingService.registerInstance("nacos.test.1", instance);
	ThreadUtils.sleep(5000L);
	List<Instance> list = namingService.getAllInstances("nacos.test.1");
    ......
}
~~~

前面的关于 registerInstance 方法我们前几篇已经分析完，我们接下来关注 getAllInstance 方法内部是怎么实现的，这里返回的 List 其实就是客户端在注册完毕后返回的实例列表。

我们先打断点看一下，官方提供的 MamingTest 中返回的 list 中都有些什么信息。

![image-20230418212323531](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418212323531.png) 

而这些数据就是在客户端返回的数据信息，运行之前的 9002 项目我们看观察一下。

![image-20230418212719865](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418212719865.png)

我们进入 getAllInstances 方法一探究竟。。。

~~~ java
@Override
public List<Instance> getAllInstances(String serviceName) throws NacosException {
    return getAllInstances(serviceName, new ArrayList<String>());
}
~~~

~~~ java
@Override
public List<Instance> getAllInstances(String serviceName, List<String> clusters) throws NacosException {
    return getAllInstances(serviceName, clusters, true);
}
~~~

~~~ java
@Override
public List<Instance> getAllInstances(String serviceName, List<String> clusters, boolean subscribe)
        throws NacosException {
    return getAllInstances(serviceName, Constants.DEFAULT_GROUP, clusters, subscribe);
}
~~~

~~~ java
@Override
public List<Instance> getAllInstances(String serviceName, String groupName, List<String> clusters,
        boolean subscribe) throws NacosException {
    ServiceInfo serviceInfo;
    String clusterString = StringUtils.join(clusters, ",");
    if (subscribe) {
        serviceInfo = serviceInfoHolder.getServiceInfo(serviceName, groupName, clusterString);
        if (null == serviceInfo) {
            serviceInfo = clientProxy.subscribe(serviceName, groupName, clusterString);
        }
    } else {
        serviceInfo = clientProxy.queryInstancesOfService(serviceName, groupName, clusterString, 0, false);
    }
    List<Instance> list;
    if (serviceInfo == null || CollectionUtils.isEmpty(list = serviceInfo.getHosts())) {
        return new ArrayList<Instance>();
    }
    return list;
}
~~~

可以看到在 NacosNamingService 服务当中有大量的 getAllInstances 的重载，提供一些默认的参数。

打个断点玩玩看，内部这些参数都是怎么进行调用，并且完成处理的。

![image-20230418214156751](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418214156751.png)

~~~ java
@Override
public List<Instance> getAllInstances(String serviceName, String groupName, List<String> clusters,
        boolean subscribe) throws NacosException {
    ServiceInfo serviceInfo;
    //获取当前的集群
    String clusterString = StringUtils.join(clusters, ",");
    //是否订阅模式（默认 true ）
    if (subscribe) {
        //获取客户端缓存中获取服务信息
        serviceInfo = serviceInfoHolder.getServiceInfo(serviceName, groupName, clusterString);
        if (null == serviceInfo) {
            //如果本地的缓存不存在服务信息，则进行订阅
            serviceInfo = clientProxy.subscribe(serviceName, groupName, clusterString);
        }
    } else {
        //如果未订阅服务信息，则直接从服务器进行查询
        serviceInfo = clientProxy.queryInstancesOfService(serviceName, groupName, clusterString, 0, false);
    }
    List<Instance> list;
    //从服务信息当中获取实例列表
    if (serviceInfo == null || CollectionUtils.isEmpty(list = serviceInfo.getHosts())) {
        return new ArrayList<Instance>();
    }
    return list;
}
~~~

- 分组名称默认：DEFAULT_GROUOP
- 集群列表：默认为空数组
- 是否订阅：订阅

可以看到上面的几个关键方法所执行的功能基本就这么几个。。。

可以通过一个流程图进行分析。

![image-20230418215956134](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418215956134.png)

那么对于 缓存获取与本地缓存内部是怎么实现，以及从服务器端进行查询服务是怎么实现的呢？我们进入具体的实现方法中瞅瞅。。。

## 客户端服务信息缓存的实现

进入 <font color='orange'>**serviceInfoHolder.getServiceInfo()**</font> 方法当中看具体的实现

~~~ java
public ServiceInfo getServiceInfo(final String serviceName, final String groupName, final String clusters) {
    NAMING_LOGGER.debug("failover-mode: " + failoverReactor.isFailoverSwitch());
    String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
    String key = ServiceInfo.getKey(groupedServiceName, clusters);
    //当前的客户端连接是否故障，如果故障 isFailoverSwitch 返回的结果是 false
    if (failoverReactor.isFailoverSwitch()) {
        return failoverReactor.getService(key);
    }
    //返回当前本地缓存中的数据
    return serviceInfoMap.get(key);
}
~~~

### serviceInfoMap 的实现

~~~ java
//线程安全的
private final ConcurrentMap<String, ServiceInfo> serviceInfoMap;
~~~

~~~ java
public ServiceInfoHolder(String namespace, Properties properties) {
    initCacheDir(namespace, properties);
    if (isLoadCacheAtStart(properties)) {
        //DiskCache.read() 从磁盘中读取缓存信息
        this.serviceInfoMap = new ConcurrentHashMap<String, ServiceInfo>(DiskCache.read(this.cacheDir));
    } else {
        //如果是第一次未有缓存则初始化一个默认大小的集合
        this.serviceInfoMap = new ConcurrentHashMap<String, ServiceInfo>(16);
    }
    this.failoverReactor = new FailoverReactor(this, cacheDir);
    this.pushEmptyProtection = isPushEmptyProtect(properties);
}
~~~

~~~ java
private void initCacheDir(String namespace, Properties properties) {
    String jmSnapshotPath = System.getProperty(JM_SNAPSHOT_PATH_PROPERTY);
    String namingCacheRegistryDir = "";
    if (properties.getProperty(PropertyKeyConst.NAMING_CACHE_REGISTRY_DIR) != null) {
        namingCacheRegistryDir = File.separator + properties.getProperty(PropertyKeyConst.NAMING_CACHE_REGISTRY_DIR);
    }
    
    if (!StringUtils.isBlank(jmSnapshotPath)) {
        //初始化获取磁盘中缓存信息的 key
        cacheDir = jmSnapshotPath + File.separator + FILE_PATH_NACOS + namingCacheRegistryDir
                + File.separator + FILE_PATH_NAMING + File.separator + namespace;
    } else {
        cacheDir = System.getProperty(USER_HOME_PROPERTY) + File.separator + FILE_PATH_NACOS + namingCacheRegistryDir
                + File.separator + FILE_PATH_NAMING + File.separator + namespace;
    }
~~~

通过上面的实现可以发现哈，其实在 Nacos 的内部是通过一个线程安全的 ConcurrentHashMap 进行缓存信息的维护的，其缓冲的具体 key 分两种情况一种是没有其缓存快照的一种是存在缓存快照的。

<font color='orange'>**initCacheDir() 初始化缓存信息**</font>

![image-20230418231102010](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418231102010.png)

![image-20230418231115233](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230418231115233.png)

## 未完待续

