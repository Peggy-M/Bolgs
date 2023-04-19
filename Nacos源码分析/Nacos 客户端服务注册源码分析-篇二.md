# Nacos 客户端服务注册源码分析-篇二

继续接上回，上回分析到 _NacosNamingService_ 的整个注册的流程，其实是通过 _NacosFactory.createNamingService_ 方法，反射获取 _NacosNamingService_ 接口的实现类 _NacosNamingService_ ，而 _NacosNamingService_ 对象的构造方法中有一个初始化的 _init_ 方法，该方法就初始化了客户端的代理委托。

![image-20230412191036102](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230412191036102.png)

#### NamingGrpcClientProxy中实现

~~~ java
public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
    NamingUtils.checkInstanceIsLegal(instance);
    clientProxy.registerService(serviceName, groupName, instance);
}
~~~

可以看到在 _NacosNamingService_ 对象的 _registerInstance_ 注册服务中调用了客户端代理 _clientProxy_ 下的 _registerService_ 注册服务方法，而该方法对应的接口 _NamigClinetProxy_ 下的实现类有五个。

![image-20230412192201427](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230412192201427.png)

_Nacos_ 基于gRPC进行服务的调用和结果的处理的所以，_registerService_指向的是实现类 _NamingGrpcClientProxy_ 

~~~ java
    public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {
        NAMING_LOGGER.info("[REGISTER-SERVICE] {} registering service {} with instance {}", namespaceId, serviceName,
                instance);
        redoService.cacheInstanceForRedo(serviceName, groupName, instance); //缓存数据
        doRegisterService(serviceName, groupName, instance); //基于 gRPC进行服务的调用
    }
~~~

~~~ java
/**
 * Cache registered instance for redo. 缓存已注册的实例以进行重做。
 * 
 * @param serviceName service name
 * @param groupName   group name
 * @param instance    registered instance
 */
public void cacheInstanceForRedo(String serviceName, String groupName, Instance instance) {
    String key = NamingUtils.getGroupedName(serviceName, groupName);
    InstanceRedoData redoData = InstanceRedoData.build(serviceName, groupName, instance);
    synchronized (registeredInstances) {
        registeredInstances.put(key, redoData);
    }
}
~~~

~~~ java
/**
 * Execute register operation. 执行寄存器操作。
 *
 * @param serviceName name of service
 * @param groupName   group of service
 * @param instance    instance to register
 * @throws NacosException nacos exception
 */
public void doRegisterService(String serviceName, String groupName, Instance instance) throws NacosException {
    InstanceRequest request = new InstanceRequest(namespaceId, serviceName, groupName,
            NamingRemoteConstants.REGISTER_INSTANCE, instance);
    requestToServer(request, Response.class);
    redoService.instanceRegistered(serviceName, groupName);
~~~

![image-20230412193048982](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230412193048982.png)