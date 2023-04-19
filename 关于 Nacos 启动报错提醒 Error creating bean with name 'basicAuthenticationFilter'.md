# 关于 Nacos 启动报错提醒 Error creating bean with name 'basicAuthenticationFilter'

今天下载最新版本的 [2.2.1（2023 年 3 月 17 日）](https://github.com/alibaba/nacos/releases/tag/2.2.1)的 nacos 启动项目发现提醒一下报错

<font color='red'>**Error creating bean with name 'basicAuthenticationFilter' defined in class path resource **</font>

![image-20230330190409702](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230330190409702.png)

提醒在创建 bean 的时候报错，当时在网上搜索说是 token.secret.key 的问题，之所以出现这个问题就是，由于我们没有指定默认的  token.secret.key ，官网在文中指出说是为了避免通过碰撞，绕过身份验证的安全漏洞的问题。

![image-20230330191242774](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230330191242774.png)

>  解决方案：
>
> 官方给出的建议就是 ***关闭鉴权*** 或者 ***开启鉴权*** 后自定义一个 key   [<font color='red'>**参考文档**</font>](https://nacos.io/zh-cn/blog/announcement-token-secret-key.html)



![image-20230330192252096](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230330192252096.png)

![image-20230330192236472](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230330192236472.png)



修改配置文件 <font color='orange'>**Nacos/conf/application.properties**</font>

- 开启鉴权

~~~ java
### If turn on auth system:
nacos.core.auth.caching.enabled=true
### The default token (Base64 String):
nacos.core.auth.plugin.nacos.token.secret.key= "填充32位以上字符即可"
~~~

- 关闭鉴权

~~~ java
nacos.core.auth.caching.enabled=false   
~~~

![image-20230330195534729](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230330195534729.png)

此时就可以正常启动了