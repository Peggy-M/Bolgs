# Linux 离线安装 MySql (图文版)

## 前言说明

> Linux 环境：
>
> [**Linux version 3.10.0-1160.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Mon Oct 19 16:18:59 UTC 2020**](http://isoredirect.centos.org/centos/7/isos/x86_64/)
>
> MySql 环境：
>
> [**Linux - Generic (glibc 2.5) (x64, 32-bit), Compressed TAR Archive**](https://downloads.mysql.com/archives/community/)
>
> 远程终端工具：
>
> [**electerm  v1.25.30**](https://github.com/electerm/electerm/releases)

## MySql 版本选择下载

[ MySql 官网 ](https://dev.mysql.com/)

- 点击这里的 <font color='orange'>**DOWNLOADS **</font>进入下载界面

![image-20230403213152354](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403213152354.png)

-  选择社区服务器版

  ![image-20230403213855090](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403213855090.png)

- 选择对应的版本下载即可

  ![image-20230403214119909](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403214119909.png)

## 远程登录 Linux 上载安装包

安装 <font color='orange'>**electerm**</font> 打开，添加新的终端

![image-20230403214331801](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403214331801.png)

选择指定的文件目录

![image-20230403214532272](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403214532272.png)

![image-20230403214554619](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403214554619.png)

## 安装 MySql

### 分配用户组

-  解压 

  ~~~ 
  tar -xvzf mysql-5.7.14-linux-glibc2.5-x86_64.tar.gz 
  ~~~

![image-20230403214637311](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403214637311.png)

- 检查创建 MySQL 用户组

  ~~~ 
  groups mysql 
  ~~~

  ![image-20230403215243839](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403215243839-16805300548191.png)

  

  如果没有则添加新组 mysql与 组用户 mysql

  ~~~ 
  groupadd mysql && useradd -r -g mysql mysql
  ~~~

  ![image-20230403215532492](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403215532492.png)

- 创建数据目录并赋予权限

  ~~~
  mkdir -p /data/mysql
  chown mysql:mysql -R /data/mysql
  ~~~

  ![image-20230403220105597](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403220105597.png)

### MySql 配置修改

- 修改目录 <font color='orange'>**/etc/my.cnf**</font> 配置文件 （没有就新建）

~~~ 
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
~~~

![image-20230403220142708](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403220142708.png)

- 将在家目录解压的 mysql 安装文件，移动到 <font color='orange'>**/usr/local/mysql **</font>目录下

  ![image-20230403220429942](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403220429942.png)

- 初始化 mysql 文件

  ~~~
  cd /usr/local/mysql/bin
  ~~~

  ~~~
  ./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
  ~~~

  ![image-20230403221343691](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403221343691.png)

  <font color='red'>**注意:**</font>

  如果在初始化的时候出现错误 【error while loading shared libraries: <font color='red'>**libaio.so.**</font>1: cannot open shared object file: No such file or directory】

  这是缺少相应的组件 <font color='red'>**libaio.so.**</font> 安装即可

  ~~~ java
  yum install -y libaio
  ~~~

  初始化成功可以在指定的数据目录 /data/mysql 中可以看到初始化的文件

  ![image-20230403221525631](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403221525631.png)

- 查看初始化的密码（密码：I?)uH*s8F+)/）

  ~~~ 
  cat /data/mysql/mysql.err
  ~~~

  ![image-20230403221039984](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403221039984.png)

- 启动 MySql

  - 指定连接

  ~~~ 
  cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
  ~~~

  ![image-20230403221735750](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403221735750.png)

  - 启动服务

  ~~~ 
  service mysql start
  ~~~

  ![image-20230403221816241](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403221816241.png)

  可以看到 mysql 服务已经启动 <font color='orange'>**3306 **</font>端口被占用

  ![image-20230403222202038](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403222202038.png)

### 修改 MySql 的原始密码

- 开启免密码登陆 

  修改 <font color='orange'>**/etc/my.cnf**</font> 目录下的文件 my.cnf

  在【mysqld】模块下面添加 skip-grant-tables 并保存

  ![image-20230403222424224](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403222424224.png)

- 重新启动服务

  ~~~
  service mysql restart
  ~~~

- 登录（不需要输入密码）

  ~~~ 
  /usr/local/mysql/bin/mysql -u root -p
  ~~~

  ![image-20230403225540020](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403225540020.png)

- 刷新规则允许外部访问

  ~~~ 
  use mysql 　 #选择访问mysql库
  ~~~

  ~~~ 
  update user set host = '%' where user = 'root'; 　#使root能再任何host访问
  ~~~

  ~~~ 
  FLUSH PRIVILEGES;  #刷新 
  ~~~

  ![image-20230403225718468](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403225718468.png)

- 修改密码（密码 root ）

  ~~~
  ALTER USER "root"@"%" IDENTIFIED  BY "root";
  ~~~

  ~~~
  FLUSH PRIVILEGES; 　　 　　 #刷新 
  ~~~

  ![image-20230403225832428](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403225832428.png)

- 退出 并且 /etc/my.cnf 文件删除

  ![image-20230403225954035](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403225954035.png)

  重新启动 mysql 服务 service mysql restart

-  登陆  /usr/local/mysql/bin/mysql -u root -p  这个时候输入 刚才设置的密码 root 

![image-20230403230130459](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403230130459.png)

### 远程桌面连接测试

![image-20230403230333748](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230403230333748.png)