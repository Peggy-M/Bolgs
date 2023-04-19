# Nacos服务端健康检查-篇五

### 🕐[Nacos 客户端服务注册源码分析-篇一](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi-pian-yi)

### 🕑[Nacos 客户端服务注册源码分析-篇二](http://124.221.240.218:8080/archives/nacos-ke-hu-duan-fu-wu-zhu-ce-yuan-ma-pian-er)

### 🕒[Nacos 客户端服务注册源码分析-篇三](http://124.221.240.218:8080/archives/nacoske-hu-duan-shi-li-zhu-ce-yuan-ma-fen-xi-pian-san)

### 🕓[Nacos 服务端服务注册源码分析-篇四](http://124.221.240.218:8080/archives/nacosfu-wu-duan-fu-wu-zhu-ce-yuan-ma-fen-xi---pian-si)

------

上篇分析l服务端的注册服务的整个流程，探究了如何将客户端的实例信息注册变为 Client 模型实体，完成服务端 Service 与 客户端模型 Client 以及实例信息 instance 三个之间的关联的，[**原图点这里**](https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Nacos_%E6%9C%8D%E5%8A%A1%E7%AB%AF_%E6%B3%A8%E5%86%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.drawio#R7V1tl6I4Fv41nNPzoeoQwutHteyd3u3u7TN1zszOR1RK2UFwkeqqml%2B%2FCSQIIWhUSHyhes%2BORoKY3Dy5ee6TGw1O1u%2F%2FSP3N6luyCCLN0BfvGnzSDANano3%2Bg0s%2BihIDGGZRskzDRVEGdgXP4d8BKdRJ6Wu4CLa1C7MkibJwUy%2BcJ3EczLNamZ%2BmyVv9spckqn%2Frxl8GjYLnuR81S%2F8IF9mqKHUNZ1f%2BaxAuV%2FSbge0Vn6x9ejH5JduVv0jeKkVwqsFJmiRZ8Wr9Pgki3Hq0XYp6n1s%2BLR8sDeJMpEKwspf%2FtmFofwTxdwP8mP3zy%2FSB3GWbfdAfHCzQ7ydvkzRbJcsk9qPprnScJq%2FxIsB3Bejd7pqvSbIhhf8NsuyDdKb%2FmiWoaJWtI%2FJp8B5m%2F0Gv9UeLvPuz8skTNh6dvvmgb%2BIs%2FahUwm%2F%2FrH62q5a%2Fo%2FWK34d%2FVGuz0TZIXtN5sKetDGJ%2BfroMsj3XwbJz0bAIknWAngfVS4PIz8Kf9efwiXkuy%2Bt2PYhekE48okPJQ%2F70o1fyTZphR%2Bhxx7NaP9v%2Fe03y0iRdBOnDPImSVIOjvGr66eGhWv5L3o60Bnq1JP%2FN7%2FuSoIbEAzC%2Fw1P1QmjY%2BF%2B1qNNH%2BO7Pky26XpvamjfR3CdtamnuSBsBbepoo7E2%2BkyfEjVm8aD1h0fFs10ZMxTqhv62CrPgeePnNvKG4K5u1C9hFE3KNoALK3AXJipfpv4iRLZW%2BcxzFrrjoM%2B2WZr8FVQ%2BcY0ZtG38CQULUD7YzyDNgvf9Vty0OlLhAViuW1T6oCXAJMj0tgM227OKslUF1KDVk7FaStGngj07JOKjT4coAgVRxFSJIvCeUORLvM38eI7GIZo5kigKUvSJv8bjO55tN3W06BdEOhnpUNeZka47zZEOqF9SG%2Bl6TwYFnFt3NDiI3QlemIJ4AWyVgGHeN2B8%2BkUZZDB%2Bx4vuQ93n%2BRbjJ0fXddJylXI9%2F%2BvF5zBtBok8lwNEtkwgMi5oxVNiT98%2Bhy2KIZ7SpYvHARH5fdUXkIt2gqHU87PvCchDAuTPQfoznAe%2FG8pQ28L%2FeKht53%2F8deQs%2FycHuV2oGrkpI3id8HBw2J%2FrvpGqP5IwH0671YDL9KRjMT1UIBepyHRS%2BSRnrCV5lJT8fusQyV3BLoUto0zSGp6ziMc82VRzTW3qacguRgVzZmtjD78YQw3TPwTu0xLpMKnmal7%2BYjzWxiP8YjTFt8IffdZcdGdTG0%2FyG6IS9Cmf2vnqz4Ko3i9%2BFC5j9HqOugGtvOEYI1c496MR%2BWAdLhaFCQTb8G9%2Flt8PO0gbbLF5q1ljzXpqGEkrEJJYAbmVVjL0VVvYY8rtsKk%2FGlZtsJEvPHsQ127q1usnLy%2FboJeR696TL0AXdf%2FeBKmfJekkwrPtl%2FUmkuYU9DCNO3V7fHB4TJAndQHG8zCvejoAooEhtZ49fcxKw6fBMtwi1KW2r00%2FYxRHYE9eeI2u2b6F68iPAzJYaS9hPJ6vwmjx1f9IXnEjoDvO%2F6LvxqskDf9G1%2FulraPWykjvQL12xTOuSe6ZY37wg%2FYKYIq%2B%2Be%2B1C7%2F624w%2BTRJF%2FmYbFiCPK65R%2F4TxOMmyZE0uOs5L13UP%2FYnPM8eMVNtgqRLeSDU5IxWA3jzu9jBAEvU0A0Rhv1PLObfW8T0etrnR4uuAvnnf%2B51dT5Oxvw622Fq%2FLPizz6GJCrfu0N7Cv2Vb8ALfUbMP7S2hvSkf01Vjo0IMVC2%2BGYLrvJNrlEucFHNbBflJEV2oRMFLtm%2BZggdoGC%2B%2F5pc9mbuS3whm46IEVX%2BJcmdvhSoGcWM9M0aoPsF8MVrZGBP0Huze54udDfJ6JglqsdQP81kmQJPfW4AnQDRBZX5WWSodmOdOmM%2F2uBjNWe6j7lMemtHc3nxP3oR2NRTSYUf0Onh%2BcFccM%2FWxhxChYIhQOdGsBhRoiJC%2BFpEltWkVgCZNFAk8QcxRqooEvNDizUIO6giWzRrQRzDMpVygYJoq0ec0pVQVe3IOXJ4mW1QepTYWY9yVPoqlFQfw4VN%2BlF%2FbG2OnfLkU8IG8GPvN2mkR4immyDCJiQrkR5q8f1xzzMd16mZVUssVsypjnnLmtAsS3V2X%2BheKykegzjcLSfJfpfLuY5ZMKvrGtFT2DX3M%2B8D0ORfTlfkbF6%2Frs5UveGC7b9wa02ACFZ7kOEWHMZfKMBAcLl3KVTjx%2FpbKgwygHJ5PNv7XmMA6GZ8GVWmUawLOJk1g8WQArKyToyv7Dave4iVqispXAlYg6lqcr3QFIMGP0KIv9rNgjBtm20CGLsSkVitY3JAkQRDeirnumx%2F7y3zPIqvbvIrwtOBvRYbjL%2FzMZ37tTfy2U30W%2Fk%2Fs9tmIBOIZTTE4U8YtNTs1qaLhA6bZb3QcoYk7m6%2BekrUfxjf%2BUzev25XYWOI2wCAnUS0ngS162T1yEq5nZPS1dLFue8eiaQkSHJajlHxq9wlvkOAYgitiCyk2CQaX6JAaXLGMi0CLLgFCdCecWgbUbG5kGnY%2BXMzOBx0wI9XhxKsk73ww2%2FVCN0QzXJEyfNvw42ecee2C1yxX1NZdq%2FCHxj4Yr%2BpuP8%2BwRpW7Ri18i8vd8mBBFT7mKTkWS%2BlFqV6u1JKjJoSiC96WTj9un%2F8oTf2PygVkMOzu3EgD0IgOOZbFWEhx005jLhYnQEsSrnq7BBCctBGNZBM8S7zylBBWi659lxICOLTXzk0D4dRr9Jf4wborRmW6WQXrIPUjvh7wylNAWDqDGZbOkQPyeNPeiBBbrcT9lDmJkbg7EiclS3RX39kS9xNnJctiFvCWWZuVRKrYBYW8p0pTuVSv0s%2FcZ7cjoWh4ChiHlxRFQbYKt0etA25vOrVbQj6VDEsOFTSeOp1SAzKYtOb9zae2qxTwjkg6euqOwr5CUOLgp1T%2FbDfp4kHg3m0HW0pDCI7SHQx1gbsjTeEu3DmO0tFn3VVCgWD%2FckVZ9PfipO6uybilnIxaBj2YoLrw2aekPc9Q1fgBLcTawX26p8wvXaKPIzo1KD36gD7mfaDPfJUk24APPYMUhS9FMdkNmcBsApHUDZnO9fmr6qICdNo47AUplcG5ak%2FTuq6z%2FIT71FWat8bleLZTV3On2tghoRd8wiYvY%2Ff4SfMgSdQ9ygM2rpvXogfj7XJ4m%2BTiG%2BST3JZVI7m9%2FggdunY%2FlU6SkIf7guijGxrZStHa4%2BzKxqNW19CY3zdqHc0b5UHV6vCdau44f%2FE5P%2FsS1QKVsy%2BrwODisT6a5OAxweCBa7l5Tv%2F2MzSP%2B9IbRJGWeH%2BJIo5JDwQ%2Bk5RmFY39gQodJ%2FexbOGq5oelSUMlbwGOYEmqSt7hRdzkz35dzkiG6PpBKYtKH3NQyV%2BFSt7i7dKXq5J3hvMBLktMPKjk5bX1oJKX2NiDSv66VfKFb3G5KnmXlyPtup1O0eCZe67TeaJqEDp23aGB5BiydgkgZE8zY6r0IwF0m35xzowATIUU9IeHCvUy%2Fl4cqugSjtV9wocqFkcojvhhl2tnRw74rvojTilX67kzFXwS2JG7CuoS0xXOrdQRM9KD5sNkUiqbvPRpdJuPFD7DvSuerXL0wKdfHtG75xDnm8uSeGDcCgu1IXtI7APkHfXnWI8yzdS7DNqtL5mpCwXdIU%2FpGZ1uO6Nyg2BRy1%2BIwaKQFQ1IkSOFYZiPDYf3ApDCvSvXCFnlj9dZFG5XX%2BKXZDBNbIWe54hZJpA8h3FWardrmQgsY7z4uVpn3dDNR8aKDJ4VSXXXvcs4nrwvP8gTPS%2FJa2HsJKlj1CqeNCXnSYj3jVrlEic31tTGIqXxmGqH9FPlSTlNNtZrOqXRE2bT8EfoNcICvXATc6Kt%2BS2l8jHXNY2euIZ05Yybd1CP5AAGWc9k3OoPXgrZ6%2FX7o%2BPo0JA%2Fs3dz6%2BPOdpjLndkvfSMXoArd0tfkJAikOSWkOAnAUHPekfrjXj3RsBI4%2B4zpliiQyaaOKCOF9CbFs5F6u54%2BIUTFquMORagsVqQjIz7l3dV63F8saF6eQcpZM1bXaaymaGZshWwRoOmHrjaSDigKXLhfXj7nfThJXQPBIFwVX7gclX%2FMZOZELsdjcNy33tQ%2BQG8%2FEvH6hKv8sS2q2BORMW43fsx%2FeGTPy9xmqj8gjMMsxOMBj%2Bnzfx96ouIBTjqzZZBVCmhYcWhjnwEM0srLlVaSaf8obSUPbXtLelLuP7iE3EmnpXuUmGwA6KJKBaArlSoA0LKDdZxvJs23jroY5Cq6SA7hm7O07pSL6rVUIkW4C0vA9dp%2BVBd9JcQl6P%2FHTcC5AbaXDKA9dK9hA4aZ7YbufaDBL%2Ba2%2FRO%2BQE3Y7bw973XQkJkjVhw1zs6V1EaWWQbj55us%2F94ZLwcgk1CHHlh9RMLZE6pY0GBMvQc2r%2BzJ%2B6DzvidZ%2BPIxKVD4tqQMnCCFXCkD0JVG0U9yvNRleSLulJDjpTTfHNB5AfgyEE7PY8B7UPJ8IrfoEemHAuAmqMcMO%2FKH6vMOlBb%2FBqZ97e7QZQ5l2MlJLk1DsT3WG9J78oZcyCo9rAOeTXkS5uk1LEtCYLPsxvtwhTaFzHj6c5DAU9P2mN1cXJ9KssiYzP5cq2wnysU54co16XL2ycB5z9GT6tUX5FJMpj6QeRtfX07dqlnr4yJ5cfDGIXuq%2BWXzMfFYFP5GctIQWqkYLlu6y6xoIpouQGAU1UMLJ7LV6MdN8j%2Ftpinr2gDvI8dNI1ToNcc7hJ7c8W40Tw8%2BOJqYIWKYLYGdeUlqip3wchSydwLBbJeUx%2FDU4gmcXA12bx0CBST6OCq5OWvhcaBhbLZheBtgykaoNozB6tU6pOR5xxafY0IMmAlZ1L5Oa7anwtaCPKatS1fKXzj%2BnOdKzUzb0s1%2BGpTcxwAMIcUZtwZweVAK%2Bhu6dKAd4zux7oYrepwW3SawN669d%2F4%2Fp%2Fd9Q9fzExZa3eWGvPoz%2BaQ%2Fq2BAC%2FCyPVICumoSPULW%2BbNrq0U006F1N7k2FFSuz1scveR%2FvN7WdcvpubfL4Ua7m7tHkwO6%2FXW3ofTggdO0AEcelJV39B%2BkhaEYs7Xf0TlMeNGOFpDln0t4ndf%2FZlOsPilzu8knqs%2FyEPdZ%2BB5qGgFsfWbuiJuuKwCMev0eqWnYPqcLI3jb%2Bmg%2F3befQ9s38Yver8bDHZbFidAHmEnTPauYLOqTSADnbl5%2BwxxCmzfU3yzo1CdBHoXIW7%2F2RyjAvbS2IF8nOoJyck3n8mlNzdUehu0sQpoMhCqVdKQA9DDY8%2Bxu5zj0a2IMoPPcLE8qZwV5XrVsIuBYBsW%2BAAYFds2gtK0K21d%2FbSuITtp8t%2BIvW513HHwLMdCfRtjs2mBZLuYlsOdcLmbheDPxGehQ%2B%2BqN9uWdOgk8fvv2lqm93CPbxzL7X%2BjDB3yH21luH%2BxmNh7OSTPpSXUqzPYtTWd38O%2Bkg7809wr11sNg5oPAOK6H7enoc0c9bBlM6NniwCSXKuuxi5uQOHAnR6iCTGGSxGyxDlkkSbsE%2BOyxfOwK%2BMA222FNrHpNfNDjY71szrkx5TFQkoCsfQHcJWN0RBRIxGSvYfV6aFZzzUbyYYvm86pObJZUe7A6wDtxe0ji%2BWuKm%2BxXf7v6hlp6MI3cNETWLXLtgnMi2Q3pP1kdXatIqIFgUgWho1bGYmwcYCw6F4SWib9KNRKPzbW8R6%2F65zRt1nYf3cqf15sFq8kTdkIey1Nl%2B02XfYFgldEFduO%2FO8I7o9uOG%2Br5TJuGedIUJq3CemifXQOSM8L7leLvOervnqD4%2BQDF1TMWl7RXO72lDouh7j2apg4d%2Br%2FLRmalG6pk7I2UiczC%2B60UITNgUZNmatxz2NjZNSQh801vkhJG5oPcdM%2FQXPLV7by0Omi2IBdVZUMzepsmuNd2AwA16%2BpbssAar%2Bn%2FAQ%3D%3D) 。

![Nacos_服务端_注册流程图.drawio](../../思维导图/nacos/Nacos_服务端_注册流程图.drawio.png)

## 长连接

在之前的第四篇以及第三篇，探究其客户端的注册实现的时候，曾分析 **NamingClientProxyDelegate** 代理类中 **getExecuteClientProxy** 关于当前通讯所实现的具体协议。

~~~ JAVA
private NamingClientProxy getExecuteClientProxy(Instance instance) {
    // 临时节点，走grpc长连接；持久节点，走http短连接
            return instance.isEphemeral() ? grpcClientProxy : httpClientProxy;
        }
~~~

那么长连接与短连接之间有何异同呢？

长连接，是指一个连接只要建立，就可以发送多个数据包进行响应，如果没有数据包发送，则需要双方发送链路检测包，实时的检测当前链路的状态。

Nacos 在 2.0 之后，用 gPRC 长连接代替了原来的 Http 短连接请求。

**NamingClientProxy** 接口负责底层通讯，调用服务端接口。有三个实现类：

- **NamingClientProxyDelegate**：代理类，对 NacosNamingService 中的方法进行**代理** ，根据实际的请求情况选择 http 或 gRPC 协议请求服务端。
- **NamingGrpcClientProxy**：底层通讯基于 gRPC 长连接
- **NamingHttpClientProxy**: 底层通讯基于http短连接

NamingClientProxyDelegate会**根据instance实例是否是临时节点而选择不同的协议**。

​	临时instance：gRPC

​	持久instance：http

## 健康检查

​	在之前的1.x版本中临时实例走Distro协议内存存储，客户端向注册中心发送心跳来维持自身healthy状态，持久实例走Raft协议持久化存储，服务端定时与客户端建立tcp连接做健康检查。

​	但是2.0版本以后持久化实例没有什么变化，但是2.0临时实例不在使用心跳，而是通过长连接是否存活来判断实例是否健康。

**ConnectionManager**负责管理所有客户端的长连接。

**每3s**检测所有超过**20s没发生过通讯**的客户端，向客户端发起**ClientDetectionRequest探测请求**，如果客户端在**1s内成功响应，则检测通过**，否则执行unregister方法移除Connection。

**如果客户端持续与服务端通讯，服务端是不需要主动探活的**

```java
Map<String, Connection> connections = new ConcurrentHashMap<String, Connection>();
@PostConstruct
public void start() {

    // 启动不健康连接排除功能.
    RpcScheduledExecutor.COMMON_SERVER_EXECUTOR.scheduleWithFixedDelay(new Runnable() {
        @Override
        public void run() {
            try {

                int totalCount = connections.size();
                Loggers.REMOTE_DIGEST.info("Connection check task start");
                MetricsMonitor.getLongConnectionMonitor().set(totalCount);
                //统计过时（20s）连接
                Set<Map.Entry<String, Connection>> entries = connections.entrySet();
                int currentSdkClientCount = currentSdkClientCount();
                boolean isLoaderClient = loadClient >= 0;
                int currentMaxClient = isLoaderClient ? loadClient : connectionLimitRule.countLimit;
                int expelCount = currentMaxClient < 0 ? 0 : Math.max(currentSdkClientCount - currentMaxClient, 0);

                Loggers.REMOTE_DIGEST
                    .info("Total count ={}, sdkCount={},clusterCount={}, currentLimit={}, toExpelCount={}",
                          totalCount, currentSdkClientCount, (totalCount - currentSdkClientCount),
                          currentMaxClient + (isLoaderClient ? "(loaderCount)" : ""), expelCount);

                List<String> expelClient = new LinkedList<>();

                Map<String, AtomicInteger> expelForIp = new HashMap<>(16);

                //1. calculate expel count  of ip.
                for (Map.Entry<String, Connection> entry : entries) {

                    Connection client = entry.getValue();
                    String appName = client.getMetaInfo().getAppName();
                    String clientIp = client.getMetaInfo().getClientIp();
                    if (client.getMetaInfo().isSdkSource() && !expelForIp.containsKey(clientIp)) {
                        //get limit for current ip.
                        int countLimitOfIp = connectionLimitRule.getCountLimitOfIp(clientIp);
                        if (countLimitOfIp < 0) {
                            int countLimitOfApp = connectionLimitRule.getCountLimitOfApp(appName);
                            countLimitOfIp = countLimitOfApp < 0 ? countLimitOfIp : countLimitOfApp;
                        }
                        if (countLimitOfIp < 0) {
                            countLimitOfIp = connectionLimitRule.getCountLimitPerClientIpDefault();
                        }

                        if (countLimitOfIp >= 0 && connectionForClientIp.containsKey(clientIp)) {
                            AtomicInteger currentCountIp = connectionForClientIp.get(clientIp);
                            if (currentCountIp != null && currentCountIp.get() > countLimitOfIp) {
                                expelForIp.put(clientIp, new AtomicInteger(currentCountIp.get() - countLimitOfIp));
                            }
                        }
                    }
                }

                Loggers.REMOTE_DIGEST
                    .info("Check over limit for ip limit rule, over limit ip count={}", expelForIp.size());

                if (expelForIp.size() > 0) {
                    Loggers.REMOTE_DIGEST.info("Over limit ip expel info, {}", expelForIp);
                }

                Set<String> outDatedConnections = new HashSet<>();
                long now = System.currentTimeMillis();
                //2.get expel connection for ip limit.
                for (Map.Entry<String, Connection> entry : entries) {
                    Connection client = entry.getValue();
                    String clientIp = client.getMetaInfo().getClientIp();
                    AtomicInteger integer = expelForIp.get(clientIp);
                    if (integer != null && integer.intValue() > 0) {
                        integer.decrementAndGet();
                        expelClient.add(client.getMetaInfo().getConnectionId());
                        expelCount--;
                    } else if (now - client.getMetaInfo().getLastActiveTime() >= KEEP_ALIVE_TIME) {
                        outDatedConnections.add(client.getMetaInfo().getConnectionId());
                    }

                }

                //3. if total count is still over limit.
                if (expelCount > 0) {
                    for (Map.Entry<String, Connection> entry : entries) {
                        Connection client = entry.getValue();
                        if (!expelForIp.containsKey(client.getMetaInfo().clientIp) && client.getMetaInfo()
                            .isSdkSource() && expelCount > 0) {
                            expelClient.add(client.getMetaInfo().getConnectionId());
                            expelCount--;
                            outDatedConnections.remove(client.getMetaInfo().getConnectionId());
                        }
                    }
                }

                String serverIp = null;
                String serverPort = null;
                if (StringUtils.isNotBlank(redirectAddress) && redirectAddress.contains(Constants.COLON)) {
                    String[] split = redirectAddress.split(Constants.COLON);
                    serverIp = split[0];
                    serverPort = split[1];
                }

                for (String expelledClientId : expelClient) {
                    try {
                        Connection connection = getConnection(expelledClientId);
                        if (connection != null) {
                            ConnectResetRequest connectResetRequest = new ConnectResetRequest();
                            connectResetRequest.setServerIp(serverIp);
                            connectResetRequest.setServerPort(serverPort);
                            connection.asyncRequest(connectResetRequest, null);
                            Loggers.REMOTE_DIGEST
                                .info("Send connection reset request , connection id = {},recommendServerIp={}, recommendServerPort={}",
                                      expelledClientId, connectResetRequest.getServerIp(),
                                      connectResetRequest.getServerPort());
                        }

                    } catch (ConnectionAlreadyClosedException e) {
                        unregister(expelledClientId);
                    } catch (Exception e) {
                        Loggers.REMOTE_DIGEST.error("Error occurs when expel connection, expelledClientId:{}", expelledClientId, e);
                    }
                }

                //4.client active detection.
                Loggers.REMOTE_DIGEST.info("Out dated connection ,size={}", outDatedConnections.size());
                //异步请求所有需要检测的连接
                if (CollectionUtils.isNotEmpty(outDatedConnections)) {
                    Set<String> successConnections = new HashSet<>();
                    final CountDownLatch latch = new CountDownLatch(outDatedConnections.size());
                    for (String outDateConnectionId : outDatedConnections) {
                        try {
                            Connection connection = getConnection(outDateConnectionId);
                            if (connection != null) {
                                ClientDetectionRequest clientDetectionRequest = new ClientDetectionRequest();
                                connection.asyncRequest(clientDetectionRequest, new RequestCallBack() {
                                    @Override
                                    public Executor getExecutor() {
                                        return null;
                                    }

                                    @Override
                                    public long getTimeout() {
                                        return 1000L;
                                    }

                                    @Override
                                    public void onResponse(Response response) {
                                        latch.countDown();
                                        if (response != null && response.isSuccess()) {
                                            connection.freshActiveTime();
                                            successConnections.add(outDateConnectionId);
                                        }
                                    }

                                    @Override
                                    public void onException(Throwable e) {
                                        latch.countDown();
                                    }
                                });

                                Loggers.REMOTE_DIGEST
                                    .info("[{}]send connection active request ", outDateConnectionId);
                            } else {
                                latch.countDown();
                            }

                        } catch (ConnectionAlreadyClosedException e) {
                            latch.countDown();
                        } catch (Exception e) {
                            Loggers.REMOTE_DIGEST
                                .error("[{}]Error occurs when check client active detection ,error={}",
                                       outDateConnectionId, e);
                            latch.countDown();
                        }
                    }

                    latch.await(3000L, TimeUnit.MILLISECONDS);
                    Loggers.REMOTE_DIGEST
                        .info("Out dated connection check successCount={}", successConnections.size());
					// 对于没有成功响应的客户端，执行unregister移出
                    for (String outDateConnectionId : outDatedConnections) {
                        if (!successConnections.contains(outDateConnectionId)) {
                            Loggers.REMOTE_DIGEST
                                .info("[{}]Unregister Out dated connection....", outDateConnectionId);
                            unregister(outDateConnectionId);
                        }
                    }
                }

                //reset loader client

                if (isLoaderClient) {
                    loadClient = -1;
                    redirectAddress = null;
                }

                Loggers.REMOTE_DIGEST.info("Connection check task end");

            } catch (Throwable e) {
                Loggers.REMOTE.error("Error occurs during connection check... ", e);
            }
        }
    }, 1000L, 3000L, TimeUnit.MILLISECONDS);

}

//注销（移出）连接方法
public synchronized void unregister(String connectionId) {
    Connection remove = this.connections.remove(connectionId);
    if (remove != null) {
        String clientIp = remove.getMetaInfo().clientIp;
        AtomicInteger atomicInteger = connectionForClientIp.get(clientIp);
        if (atomicInteger != null) {
            int count = atomicInteger.decrementAndGet();
            if (count <= 0) {
                connectionForClientIp.remove(clientIp);
            }
        }
        remove.close();
        Loggers.REMOTE_DIGEST.info("[{}]Connection unregistered successfully. ", connectionId);
        clientConnectionEventListenerRegistry.notifyClientDisConnected(remove);
    }
}
```

移除connection后，继承ClientConnectionEventListener的**ConnectionBasedClientManager**会移除Client，发布**ClientDisconnectEvent事件**。

```java
@Override
public boolean clientDisconnected(String clientId) {
    Loggers.SRV_LOG.info("Client connection {} disconnect, remove instances and subscribers", clientId);
    ConnectionBasedClient client = clients.remove(clientId);
    if (null == client) {
        return true;
    }
    client.release();
    NotifyCenter.publishEvent(new ClientEvent.ClientDisconnectEvent(client));
    return true;
}
```

ClientDisconnectEvent会触发几个事件：

**1）Distro协议**：同步移除的client数据

**2）清除两个索引缓存**：ClientServiceIndexesManager中Service与发布Client的关系；ServiceStorage中Service与Instance的关系

**3）服务订阅**：ClientDisconnectEvent会**间接触发ServiceChangedEvent事件**，将服务变更通知客户端。