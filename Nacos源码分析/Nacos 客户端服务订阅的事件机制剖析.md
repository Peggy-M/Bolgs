# Nacos 客户端服务订阅的事件机制剖析 - 篇八

## 历史篇章

### 🕐[Nacos 客户端服务注册源码分析-篇一](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi-pian-yi)

### 🕑[Nacos 客户端服务注册源码分析-篇二](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-pian-er)

### 🕒[Nacos 客户端服务注册源码分析-篇三](http://124.221.240.218:8080/archives/nacoske-hu-duan-shi-li-zhu-ce-yuan-ma-fen-xi-pian-san)

### 🕓[Nacos 服务端服务注册源码分析-篇四](http://124.221.240.218:8080/archives/nacosfu-wu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi---pian-si)

### 🕔[Nacos 服务端健康检查-篇五](http://124.221.240.218:8080/archives/nacosfu-wu-duan-jian-kang-jian-cha-pian-wu)

 ### 🕖[Nacos 客户端的服务发现与服务订阅机制的纠缠 - 篇七](https://peggy-m.top/archives/nacos-ke-hu-duan-de-fu-wu-fa-xian-yu-fu-wu-ding-yue-ji-zhi-de-jiu-chan---pian-qi)

------

上一篇我们分析了 Naocs 客户端的无法发现与订阅之间的的联系，发现其实在源码中。无论是服务的发现还是服务的注册，最终都是调用的 NacosClienProxyDelegate 类下的 subscribe 方法完成服务的发现或订阅的。

那么接下来该篇章当中我们的会详细的剖析服务进行订阅的整个事件机制是如何进行处理的。

在我们最后一次调用 sunscribe 的方法的时候，会订阅一个 EventListener 监听事件。

而定时任务 UbdateTask 定时获取实例列表之后，会调用 ServiceInfoHolder.rpocessServiceInfo 方法对 ServiceInfo 进行本地处理，这其中就包含和事件处理。

~~~ java
@Override
public ServiceInfo subscribe(String serviceName, String groupName, String clusters) throws NacosException {
    String serviceNameWithGroup = NamingUtils.getGroupedName(serviceName, groupName);
    String serviceKey = ServiceInfo.getKey(serviceNameWithGroup, clusters);
    //开启定时任务调度 UpdateTask
    serviceInfoUpdateService.scheduleUpdateIfAbsent(serviceName, groupName, clusters);
    //获取缓存中的 ServiceInfo
    ServiceInfo result = serviceInfoHolder.getServiceInfoMap().get(serviceKey);
    if (null == result) {
        //如果缓存中没有数据,则进行订阅逻辑处理,基于 gRPC 协议
        result = grpcClientProxy.subscribe(serviceName, groupName, clusters);
    }
    //serviceInfo 本地缓存处理
    serviceInfoHolder.processServiceInfo(result);
    return result;
}
~~~

在进行多次的 subscribe 重载之后，在最终的 subscribe 方法当中，可以看到有一个监听器的注册。

~~~ java
@Override
public void subscribe(String serviceName, String groupName, List<String> clusters, EventListener lis
        throws NacosException {
    //如果事件监听器为空 则返回
    if (null == listener) {
        return;
    }
    String clusterString = StringUtils.join(clusters, ",");
    //注册监听器
    changeNotifier.registerListener(groupName, serviceName, clusterString, listener);
    //对于订阅的本质就是服务的发现的一种方式,也就是服务在发现的时候执行订阅方法,同时触发定时任务去服务端拉去数据
    clientProxy.subscribe(serviceName, groupName, clusterString);
}
~~~

这里的关键代码就是 changNotifier.registerListener 方法，该方法主要就是完成，监听对象 listener 的注册。

我们进入该方法当中去看看其中的一些具体的实现是怎么完成的。。。

~~~ java
/**
 * register listener.
 *
 * @param groupName   group name
 * @param serviceName serviceName
 * @param clusters    clusters, concat by ','. such as 'xxx,yyy'
 * @param listener    custom listener
 */
public void registerListener(String groupName, String serviceName, String clusters, EventListener listener) {
    String key = ServiceInfo.getKey(NamingUtils.getGroupedName(serviceName, groupName), clusters);
    //从缓存当中获取时间监听器对象
    ConcurrentHashSet<EventListener> eventListeners = listenerMap.get(key);
    //如果缓存当中没有监听的对象
    if (eventListeners == null) {
        synchronized (lock) {
            eventListeners = listenerMap.get(key);
            if (eventListeners == null) {
                // 创建一个事件监听器
                eventListeners = new ConcurrentHashSet<EventListener>();
                //将事件监听器添加到缓存当中
                listenerMap.put(key, eventListeners);
            }
        }
    }
    //并将监听事件添加到当前对象的监听器当中
    eventListeners.add(listener);
}
~~~

## ServiceInfo 处理

上面的代码主要完成的就是监听事件的注册，一个事件需要被触发才会执行具体的操作，那么我们接下来就就去追溯一下触发事件的具体来源。

在 UpdateTask 当中获取的实例会进行本地化处理，部分源码如下所示：

~~~ java
//获取缓存中的信息
ServiceInfo serviceObj = serviceInfoHolder.getServiceInfoMap().get(serviceKey);
//缓存为空
if (serviceObj == null) {
    //生成一个服务实例对象
    serviceObj = namingClientProxy.queryInstancesOfService(serviceName, groupName, clusters, 0, false);
    //处理更新或添加到本地的缓存当中
    serviceInfoHolder.processServiceInfo(serviceObj);
    //更新最后一次的时间
    lastRefTime = serviceObj.getLastRefTime();
    return;
}
~~~

可以在 ServiceInfoUpdateService 下的内部类 UpdateTask 当中可以看到其 run 方法当中，在获取最新的实例之后进行本地缓存缓存处理。

~~~ java
//处理更新或添加到本地的缓存当中
    serviceInfoHolder.processServiceInfo(serviceObj);
~~~

我们进入该方法当中看一下其具体实现。[**原图点击这里**](https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Nacos_%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%9C%8D%E5%8A%A1%E8%AE%A2%E9%98%85%E7%9A%84%E6%97%B6%E9%97%B4%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90.drawio#R7Vxbc9o4FP41mmkfwshX7EfMZXdn252dTWfaPgpbgFuDWNkksL9%2BJVnCF2xwwGCSJi%2BxjyRblr7znU9HSoAxXG5%2Fo2i9%2BEwCHAEdBltgjICu6xAa7Be37FKLozupYU7DIDVpmeEx%2FA9LI5TWTRjguFAxISRKwnXR6JPVCvtJwYYoJc%2FFajMSFd%2B6RnN8YHj0UXRo%2FRoGyUJ9RT%2Bz%2F47D%2BUK9WbPdtGSJVGX5JfECBeQ5ZzLGwBhSQpL0arkd4ogPnhqXtN2kpnTfMYpXSZMGXjzRvtIvm3lIksWXH1P36z%2F0QX7GE4o28oNlZ5OdGgEcsAGRt4QmCzInKxSNM6tHyWYVYP4ayO6yOp8IWTOjxow%2FcJLs5OyiTUKYaZEsI1mavpO%2FqPbbpCkmG%2BrjIx8k4ZYgOsfJkXrmfgYYdDFZ4oTuWDuKI5SET8V%2BIImh%2Bb5eNszsQo70C0bdqBh1O2Ld9absYp6IIUkNM8JGID8f9r8bogoeYjGiA1ZBM9fbtJksVw9aU%2BLjOH7E9Cn08R%2BrGfmQu%2BaPzu4%2Bqreyr0pfXOwMM%2Bc6WMJJhgI%2Bpc%2BLMMGPayQm65lxQ3HGZ2EUDUlEqGhrzGZ%2BwIjB8OYUBSGb9EIZsiAHVpxQ8hPnSoK%2BOxUlcjgxTfD2OIIOZ1wxVV%2B6qeIpefucOb0B7dS2yDm8Da%2BEEbsLR8TbMPnGm%2Fcsefc9VzLayieLm526WbHPzTXit9%2FzZVkzcafatej0ZkOn12ow0NjrZdO%2FSSi8UmLH1I0CdnS3BIq0Y7JVCRf7bpwPFU2v5ZOL6QOMLcA%2BZ2DXUkMFSj%2BhKZMBBWShKJyv2LXPJhcz%2F%2FW4t4Yszg5kwTIMghTEmHULTcXzOE7WfNjEQFoesEbH3F2KANk4C715TNU7Wy03PMCeiotyhiXgLsSNwq1qQWazGF8FIe5bC%2FNNPd7pMswrDXsdv7SB64DB5E37pXvaL6FV8MyHdlxT0yqfen1PNW8oDTm3OwxDpgCTDQajTA7%2BiflAZyjbh4GxCTwHeAMw7oOByy860owBws7Mr9KFtu%2Fg6awdXWgYxdiuVQjDve0mwtCpRUi8RqvLAFGGGsXJhq6qp3JvS197jVWB42O%2FcoanjmVaLSl%2F24ClGbYOZxhWzLB1rRnWqkjgVYdrRaenFbrRacC2f92Btzod%2BKo81PsK5gVKSTu9hOnbRZ5rRSg9FIWSfiudpIB9I6HkAhbjXSZ6mPSZAIfBFf6UCskBzhh4fSGmJlxGFVJq3Mzkk%2BeKphoYaHEh%2Byb01ISZudRytJygn9IjOq2JLuMPdHh91nvXBK7oAmtyZM1wZcmmTZGGdTnwhRQf%2F6kK9BDaY9bfq0g5A3Yt5bRbJoIldhkSOGY84I4F%2FBqghdVh0HfMQt7YRks%2B2atpvM4rR1oBptumkm8iGq0Smd4Blt5cjkezmmqXbrM81su1S4OlF%2FdSDwLHFlwvrrmLjsBgLOMBjzkiHvCi1JJesGqWiDsjHiT4c9iFISMEX%2BTXL%2BcO%2Fbf9RWWZO3IB8TNa99ab5MPHY308WHI2pp42aQYiA6LDUAbFTxX9eKM%2BbGu3qhzKTK1r%2BtGr9Njrph%2BnIf3kdvO7oJ%2F6fNBl9MN0psFVL%2BcYJgMscTHkDMQ5ZiAla1mFWpyWPJ1rBs%2FjgpjX8YBnZlzVMf2E8XCBVnMc5LfE759wELR0PXiJdrZH2uRahGPpZq8iTXZbyrG6YJj9FjnINsiz7fJTW%2BQgv0Ge7Zdff4tcJRJOs5neJZvp10wElZkgoewdx%2F3%2B7SWK9JpEn3w87GmW4RR8vZ1EkVv1zOvnifRuztGc6fDnkEsnJNHp8Tn9DMlzNknMUBT%2FgixxPJ0Me4ZmmcUlSCssoY4yFR96A5o446hGs6VuO7CcEhJhIUFLaTfop2L2Rem2LvQrNnxHqNHmC2boWm3p1%2F79yVejCnKvesWsPuB0%2BKg513Oj09edjLMK7er6tawbms6pcenZ2sskgf7O32%2BZvw2zeEjHNjtPeNaf1DsDce3r1nM6c7hDWJHzs%2FgOBC8ygTcGjtdoB7saoOp4mdowf5HM7iTrf5PNxTLY%2B1bXYFfZv1cUMvfL8NuHTLepDLp0FS2aDihFu1wFuTysT8TAEpWW%2F27wRP2%2BA0t4SnvQ7nqsajf7ntiVcZ0ntjgkMUJ1%2FsfJ7XgcUOUBaf1FknC2G4ocQG%2B9mUZhvBg%2FsduTOxLdk2EAfSxO8xyQoWVbWh%2B3Q4YKnvulm3tIhrp5UzI038mw%2Bd%2FjNj2Aeh9k2L9HMlRjeL9kuD%2BqknKgUImeAxxjf3ZMcOBAyUVGj7Y8qsIPmln8wArfYe6LPWdNtPKOHl7JqG4Uxj%2BHyF%2Fg3jNl9PVOnDXEaV%2BRONlt9o8MUthn%2Fw7CGP8P)

![image-20230422220809850](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230422220809850.png)

~~~ java
/**
 * Process service info.
 *
 * @param serviceInfo new service info
 * @return service info
 */
public ServiceInfo processServiceInfo(ServiceInfo serviceInfo) {
    //判断 ServiceInfo 的 key 是否为 null
    String serviceKey = serviceInfo.getKey();
    if (serviceKey == null) {
        return null;
    }
    //判断 ServiceInfo 信息是否为空或者错误
    ServiceInfo oldService = serviceInfoMap.get(serviceInfo.getKey());
    if (isEmptyOrErrorPush(serviceInfo)) {
        //empty or error push, just ignore
        return oldService;
    }
    //将数据更新到内存当中
    serviceInfoMap.put(serviceInfo.getKey(), serviceInfo);
    //检查是否是已经更新的服务
    boolean changed = isChangedServiceInfo(oldService, serviceInfo);
    //判断 ServiceInfo 当中是否存在来自服务端的 JSON 串
    if (StringUtils.isBlank(serviceInfo.getJsonFromServer())) {
        serviceInfo.setJsonFromServer(JacksonUtils.toJson(serviceInfo));
    }
    MetricsMonitor.getServiceInfoMapSizeMonitor().set(serviceInfoMap.size());
    //如果服务信息已经发生改变
    if (changed) {
        NAMING_LOGGER.info("current ips:(" + serviceInfo.ipCount() + ") service: " + serviceInfo.getKey() + " -> "
                + JacksonUtils.toJson(serviceInfo.getHosts()));
        //添加实例变更事件，会被订阅者执行
        NotifyCenter.publishEvent(new InstancesChangeEvent(serviceInfo.getName(), serviceInfo.getGroupName(),
                serviceInfo.getClusters(), serviceInfo.getHosts()));
        //将发布的事件写入本地的磁盘中
        DiskCache.write(serviceInfo, cacheDir);
    }
    //返回已经改变最新或者为改变的服务信息
    return serviceInfo;
}
~~~

可以通过上面的整个源码过程可以发现，其实当执行 isChangedServiceInfo 方法检查服务是否更新以后，如果服务的信息发生了变化，则会调用 NatofyCenter 类下的 publishEvent 方法添加发布变更的事件。

所以服务发生变更后的事件是由 NoatofyCenter 进行发布的，我们继续探究其源码的具体执行过程。。。



### 事件追踪

~~~ java
//添加实例变更事件，会被订阅者执行
NotifyCenter.publishEvent(new InstancesChangeEvent(serviceInfo.getName(), serviceInfo.getGroupName(),
        serviceInfo.getClusters(), serviceInfo.getHosts()));
~~~

NaotifyCenter 通知中心的整体流程大致如下：

![image-20230422221633220](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230422221633220.png) 

~~~ java
/**
 * Request publisher publish event Publishers load lazily, calling publisher.
 *
 * @param eventType class Instances type of the event type.
 * @param event     event instance.
 */
private static boolean publishEvent(final Class<? extends Event> eventType, final Event event) {
    if (ClassUtils.isAssignableFrom(SlowEvent.class, eventType)) {
        return INSTANCE.sharePublisher.publish(event);
    }
    // 根据 InstancesChangeEvent 事件类型，获得对应的 CanonicalName
    final String topic = ClassUtils.getCanonicalName(eventType);
    // 将CanonicalName作为Key，从NotifyCenter#publisherMap中获取对应的事件发布者（EventPublisher）
    // INSTANCE 只有一个是单利关于 publisherMap 中的 key val 关系的建立时机,其实是在 NacosNamingService 实例化时调用 init
    // 初始化方法中进行绑定的
    EventPublisher publisher = INSTANCE.publisherMap.get(topic);
    if (publisher != null) {
        // 事件发布者publisher发布事件（InstancesChangeEvent）
        return publisher.publish(event);
    }
    LOGGER.warn("There are no [{}] publishers for this event, please register", topic);
    return false;
}
~~~

## 未完待续。。。

