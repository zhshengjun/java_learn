---
title: Nacos 集群服务搭建遇到的问题
tags: nacos,集群，注册中心
grammar_cjkRuby: true
---
前言: 因为了解到Eureka2.0.x及其部分组件闭源的消息, 再加上springcloud alibaba套件的孵化, 想必springcloud alibaba会在以后的一段时间中火热起来, 所以小生开始学习使用alibaba套件.本文所使用的是替换Eureka的Nacos,至于nacos是什么,想必看到本文的朋友应该都了解,不了解的可以看看: <https://blog.csdn.net/xstardust/article/details/81481478>
言归正传,我也是刚入坑的新人,在开始使用Nacos的时候的确有一定坎坷,在以下我会把我的问题列举出来,大家共同学习,如有错误的地方,望批评指教,谢谢.

# 安装 Nacos

nacos 的基本安装使用也很简单,这里推荐查看 @程序猿DD 的文章, 对部署等操作列举的清晰明了:
[使用Nacos实现服务注册与发现](http://blog.didispace.com/spring-cloud-alibaba-1/)

# 单机启动问题

在进行 Nacos 部署的时候后,如果是启动存在问题的可以看看以下方案:
[Nacos部署中的一些常见问题汇总](http://blog.didispace.com/nacos-faqs/)

# Nacos集群问题

这里我主要提一下我在集群中所遇到的主要问题:

```
code:503 msg: server is STARTING now, please try again later!
```

该问题分为两个类别:

## 同一网段

**Nacos集群时在 nacos/conf/cluster.conf 文件中所有节点都是使用同一网段的内网ip (注意是同一网段)**

```
	服务器: 三台同一内网的服务器
```
>
> 
> [root@dpl ~]# vim /usr/local/nacos/conf/cluster.conf
> #it is ip
> #example
> 172.27.0.6:8848
> 172.27.0.7:8848
> 172.27.0.8.37:8848
> 
> 

以上使用内网的三台Nacos服务进行集群, 注意是内网,内网,内网 !

**问题描述:**
按照官网文档修改了cluster.conf,添加了三台服务的IP(172.XX.XX.6:8848, 172.XX.XX.7:8848,
172.XX.XX.8:8848)，启动nacos服务时报错。
这种情况此时启动服务应用进行服务注册，发现报如下错误：

```java
java.lang.IllegalStateException: failed to req API:/nacos/v1/ns/instance after all servers([172.XX.XX.2:80]) tried
        at com.alibaba.nacos.client.naming.net.NamingProxy.reqAPI(NamingProxy.java:335)
        at com.alibaba.nacos.client.naming.net.NamingProxy.reqAPI(NamingProxy.java:267)
        at com.alibaba.nacos.client.naming.net.NamingProxy.registerService(NamingProxy.java:167)
        at com.alibaba.nacos.client.naming.NacosNamingService.registerInstance(NacosNamingService.java:170)
        at org.springframework.cloud.alibaba.nacos.registry.NacosServiceRegistry.register(NacosServiceRegistry.java:56)
        at org.springframework.cloud.alibaba.nacos.registry.NacosServiceRegistry.register(NacosServiceRegistry.java:29)
```
就算不报该错误的话, 这时nacos服务的控制台能够访问, 在进行注册服务的时候也无法注册, 会出现标题出现的问题

```
code:503 msg: server is STARTING now, please try again later!

```

然而这个时候我们查看 naming-raft.log 文件查看日志, 发现
```java
2019-05-15 00:02:00,000 WARN [IS LEADER] no leader is available now!

```

从 [Nacos集群模式下服务无法注册](https://www.wandouip.com/t5i92492/) 这篇文章的到一定的帮助, 并解决该问题.
深层的原因： 在大多数Linux操作系统中，都是以/etc/hosts中的配置查找主机名的，而Java 的InetAddress.java 调用 InetAddressImpl.java 的 public native String getLocalHostName() throws UnknownHostException; 来获取本地主机名，Java 的这个方法是native的，是本地系统的一个实现，此时根据本地/etc/hostname文件中的机器名来获取本机IP，然而这个IP并不是这台机器的内网IP，所以这篇文章是通过修改本机名称和本机IP来解决 .

## 不同网段

**Nacos集群时在 nacos/conf/cluster.conf 文件中所有节点是使用的不同网段的ip**

```
	服务器: 三台不同网段的服务器
	这里我使用一台腾讯云,两台阿里云, 都不在同一内网中
```

> 
> 
> [root@dpl1 ~]# vim /usr/local/nacos/conf/cluster.conf
> #it is ip
> #example
> 59.110.243.242:8848
> 120.79.4.129:8848
> 148.70.212.37:8848
> 
> 

像阿里云、腾讯云这种云服务器, 会提供一个外网ip和内网ip, 访问外网ip时会指向对应的内网ip来访问到该服务器, 由于nacos集群内部是指定的使用网卡ip地址来进行通信,但是由于三台服务器各自的内网ip不在同一网段, 所以造成无法通信, 也会造成以下问题

注册服务:

```
code:503 msg: server is STARTING now, please try again later!

```

naming-raft.log 日志:

```
2019-05-15 00:02:00,000 WARN [IS LEADER] no leader is available now!

```

这个问题找了很久, 最后源码发现可以通过设置ip地址的参数来自己指定使用的ip地址

``` java
private static String getHostAddress() {
        String address = System.getProperty("nacos.server.ip");
        if (StringUtils.isNotEmpty(address)) {
            return address;
        } else {
            address = "127.0.0.1";
        }
		...
}
```
这时只要修改启动参数, 设置本机ip地址就可以了
修改 nacos/bin/startup.sh 文件
找到 JVM Configuration 这部分, 在集群参数里增加 -Dnacos.server.ip=xx

```
#===========================================================================================
# JVM Configuration
#===========================================================================================

# 单机模式对应的启动参数
if [[ "${MODE}" == "standalone" ]]; then
    JAVA_OPT="${JAVA_OPT} -Xms512m -Xmx512m -Xmn256m"
    JAVA_OPT="${JAVA_OPT} -Dnacos.standalone=true"
else
# 集群模式对应的启动参数
    JAVA_OPT="${JAVA_OPT} -server -Xms2g -Xmx2g -Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
    JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${BASE_DIR}/logs/java_heapdump.hprof"
    JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages"

# 新增以下参数设置本机ip地址
    JAVA_OPT="${JAVA_OPT} -Dnacos.server.ip= 服务器的ip"

fi

```

# 服务器的 JDK 版本低于1.8

``` 
#===========================================================================================
# JVM Configuration
#===========================================================================================

# 系统默认的 java 路径
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
#[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/opt/taobao/java
#[ ! -e "$JAVA_HOME/bin/java" ] && unset JAVA_HOME

# 修改为自己指定的jdk
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME= 修改的jdk 路径

```
