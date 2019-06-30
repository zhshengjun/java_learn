
[TOC]

# <font size = 4>EHR-Nacos 生产服务部署</font>

![Nacos集群示意](https://i.loli.net/2019/06/27/5d145f4104b2426890.png)

- 测试环境<br>
    ```
    Nacos 管理地址：xxxx
    用户名/密码：xxx/xx
    server-addr:
    ```
- 生产环境<br>
    ```
    Nacos 管理地址：xxxx
    用户名/密码：xxx/xx
    server-addr:
    ```
	

# <font size = 4>应用接入 Nacos </font>
## <font size = 3>Nacos 的服务注册与发现</font>
### <font size = 2>1. 编辑pom.xml，加入必要的依赖配置</font>

```
 <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>0.9.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
<!-- 注册和发现服务依赖-->
<dependencies>
    <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```
-  <font color=#e96900>dependencyManagement</font>：spring cloud alibaba 的版本，由于spring cloud alibaba 还未纳入spring cloud 的主版本管理中，所以需要自己加入。
- <font color=#e96900>dependencies</font>：当前应用要使用的依赖内容。这里主要新加入了 Nacos 的服务注册与发现模块：<font color=#e96900>spring-cloud-starter-alibaba-nacos-discovery </font>。由于 dependencyManagement 中已经引入了版本,这里就不用指定具体版本了。


### <font size = 2>2. 配置服务名称和 Nacos 地址</font>

```
spring.application.name= xxx
#使用 Nacos 作为服务注册中心
spring.cloud.nacos.discovery.server-addr= xxx
```
<font color=#e96900>说明</font>：
- <font color=#e96900> server-addr </font>：填写 Nacos 集群的全部服务地址,逗号分隔。


### <font size = 2>3. 添加注解</font>
在 SpringBoot 启动类上添加 <font color=#e96900> @EnableDiscoveryClient</font>；但不是必须的。只要添加了<font color=#e96900> spring-cloud-starter-alibaba-nacos-discovery </font> 依赖，就会自动注册到 Nacos 上。

```
@EnableDiscoveryClient
@SpringBootApplication
public class NacosClientApplication {
     ...
}
```

### <font size = 2>启动应用</font>
我们可以访问 Nacos 的管理页面：地址 xxxx

![IMG20190627_115044.png](https://i.loli.net/2019/06/27/5d143d1c809d225210.png)

这里会显示当前注册的所有服务，以及每个服务的集群数目、实例数、健康实例数。
点击详情，可以看到每个服务具体的实例信息，如下图所示：

![IMG20190627_115319.png](https://i.loli.net/2019/06/27/5d143dbbd398269410.png)



## <font size = 2>Nacos 作为配置中心</font>
### <font size = 2>1. 编辑pom.xml，加入必要的依赖配置</font>

```
<!-- spring-cloud-alibaba-dependencies 依赖同注册中心 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
```
### <font size = 2>2. 修改配置</font>

```
spring.cloud.nacos.config.server-addr= xxx
```
<font color=red size = 4>注意：</font>
这里必须使用 <font color=#e96900>bootstrap.properties</font>。
同时，<font color=#e96900>spring.application.name</font> 值必须 Nacos 中创建的配置<font color=#e96900> Data Id </font>匹配

### <font size = 2>3. 添加注解</font>
在 引入配置文件的类上增加 <font color=#e96900>@RefreshScope</font> 注解，表示这个类下的配置内容支持动态刷新
```
@RefreshScope
public class NacosClientController {
    @Value("${value}")
    private String value;
}
```


# <font size = 4>Nacos 管理配置</font>
## <font size = 3>创建配置项</font>
1.  进入Nacos控制台，点击导航栏的“配置列表”，然后点击右侧的“+” 按钮，如下图所示：
    ![IMG20190627_125511.png](https://i.loli.net/2019/06/27/5d144c3f5f7e751065.png)

2.  新建配置项
	![IMG20190627_125658.png](https://i.loli.net/2019/06/27/5d144cababcc313208.png)


## <font size = 3>多环境管理</font>
在 Nacos中，本身有多个不同管理级别的概念，包括：Data ID、Group、Namespace。

### <font size = 2>使用 Data ID 与 profiles 实现</font>
<font color=#e96900>Data Id </font>完整格式为：<font color=red>${prefix}-${spring.profile.active}.${file-extension}</font> 。

- <font color=#e96900>prefix</font> 默认为 <font color=#e96900>spring.application.name</font> 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
- <font color=#e96900>spring.profile.active</font> 即为当前环境对应的 <font color=red>profile</font>，详情可以参考 Spring Boot文档。
</br>**注意：当 spring.profile.active</font> 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成**<font color=red>${prefix}.${file-extension}</font>
- <font color=#e96900>file-exetension</font> 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

### <font size = 2>使用Group实现<br></font>
通过不同的 Group ，可以根据不同的环境使用不同的配置，但是它们的 Data ID 是完全相同的。
如图：
![IMG20190627_131356.png](https://i.loli.net/2019/06/27/5d1450a647e2916540.png)
此时项目的配置文件需要根据添加 group 来区分

```
spring.cloud.nacos.config.group=DEV_GROUP
```


### <font size = 2>使用 Namespace 实现</font>

    Namespace 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。
    常用于不同环境的配置的区分隔离，例如：开发测试环境和生产环境的资源（如配置、服务）隔离等。

1. 命名空间配置页面
    ![IMG20190627_132546.png](https://i.loli.net/2019/06/27/5d145369a9f7724817.png)

2. 修改应用的配置文件，增加Namespace的指定配置。
    ```
    spring.cloud.nacos.config.namespace= xxx
    ```
    **注意：若使用 namespace ，服务注册与发现也应加上 namespace 相关配置**

    ```
    spring.cloud.nacos.discovery.namespace= xxx
    ```

- <font color=#e96900> namespace </font>：用来区分不同环境的服务注册中心，等同于 Apollo 的 DEV,PRO。可以通过登陆 Nacos 管理页面获取，注意使用的是 <font color=red> **ID** </font>,不能使用环境的名称，默认的是 defalut 空间。

### <font size = 2>三种方式的区别</font>

1. 第一种：通过Data ID与profile实现。

    > 优点：这种方式与Spring Cloud Config的实现非常像，用过 Spring Cloud Config 的用户，可以毫无违和感的过渡过来，由于命名规则类似，所以要从 Spring Cloud Config 中做迁移也非常简单。<br>
    缺点：这种方式在项目与环境多的时候，配置内容就会显得非常混乱。配置列表中会看到各种不同应用，不同环境的配置交织在一起，非常不利于管理。<br>
    建议：项目不多时使用，或者可以结合 Group 对项目根据业务或者组织架构做一些拆分规划。

2. 第二种：通过Group实现。

    > 优点：通过Group按环境讲各个应用的配置隔离开。可以非常方便的利用 Data ID 和 Group 的搜索功能，分别从应用纬度和环境纬度来查看配置。<br>
    缺点：由于会占用Group纬度，所以需要对 Group 的使用做好规划，毕竟与业务上的一些配置分组起冲突等问题。<br>
    建议：这种方式虽然结构上比上一种更好一些，但是依然可能会有一些混乱，主要是在 Group 的管理上要做好规划和控制。

3. 第三种：通过Namespace实现。

    > 优点：官方建议的方式，通过 Namespace 来区分不同的环境，释放了 Group 的自由度，这样可以让 Group 的使用专注于做业务层面的分组管理。同时，Nacos 控制页面上对于 Namespace 也做了分组展示，不需要搜索，就可以隔离开不同的环境配置，非常易用。<br>
    缺点：没有啥缺点，可能就是多引入一个概念，需要用户去理解吧。<br>
    建议：直接用这种方式长远上来说会比较省心。

## <font size = 3>多文件加载与共享配置</font>
很多系统根据模块来定义配置文件，比如DB配置，日志配置等，可以用下面方法加载

### <font size = 2>加载多个配置</font>

```
spring.cloud.nacos.config.ext-config[0].data-id=db.properties
spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.ext-config[0].refresh=true
...
```
### <font size = 2>共享配置</font>

```
spring.cloud.nacos.config.shared-dataids=actuator.properties,log.properties
spring.cloud.nacos.config.refreshable-dataids=actuator.properties,log.properties
```
- <font color=#e96900>spring.cloud.nacos.config.shared-dataids</font> 参数用来配置多个共享配置的Data Id，多个的时候用逗号分隔。
- <font color=#e96900>spring.cloud.nacos.config.refreshable-dataids</font> 参数用来定义哪些共享配置的 Data Id 在配置变化时，应用中可以动态刷新，多个 Data Id 之间用逗号隔开。如果没有明确配置，默认情况下所有共享配置都不支持动态刷新。


## <font size = 3>配置加载的优先级</font>

在使用Nacos配置的时候，主要有以下三类配置：

- A: 通过<font color=#e96900>spring.cloud.nacos.config.shared-dataids</font> 定义的共享配置
- B: 通过<font color=#e96900>spring.cloud.nacos.config.ext-config[n]</font> 定义的加载配置
- C: 通过内部规则(<font color=#e96900>spring.cloud.nacos.config.prefix</font>、<font color=#e96900>spring.cloud.nacos.config.file-extension</font>、<font color=#e96900>spring.cloud.nacos.config.group</font>)这几个参数拼接出来的配置

<font color=red size=3>优先级关系是：A < B < C </font>
# <font size = 4>Nacos 替换 Eureka</font>
1. 添加 Nacos 的依赖，同时去掉 Eureka依赖<br>
    ```
	 <dependencies>
		    <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
	            <version>0.2.1.RELEASE</version>
	        </dependency>
	 </dependencies>
    ```

2. 修改application.properties
    ```
    spring.cloud.nacos.discovery.server-addr = xxx
    ```

3. 更换EnableEurekaClient 注解(可选)




