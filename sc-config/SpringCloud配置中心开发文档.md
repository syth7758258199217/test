

###所需要准备的知识

git的相关操作

git和gitHub的同步


这里简单做一个最常规的如何操作

详情请查看Git相关文档

其实最重要的就是会add、comit、拉取、push同步

springCloud分为服务端和客户端


###现在开始项目

#首先创建一个父项目

例如：sc-config

pom形式的

pom文件中加入：
```aidl
<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<!--springCloud的版本-->
		<spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
	</properties>

	<dependencies>
	<!--spring的web支持-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
<!--spring的Test支持-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
		<!--子模块使用的springCloud的版本-->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

###创建Config的服务端（主要是用来更新gitHub上面的文件内容的）


项目为：config-server

1.先创一个普通的springBoot的项目
2.pom文件中加入：
```aidl
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--表示是springCloudConfig的服务端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    </dependencies>
```

3.启动类加入
@EnableConfigServer

注解：表示是ConfigServer



4.加入配置文件信息
```aidl
#设置当前服务的名称
spring:
  application:
    name: config-server
    #配置config所在的gitHub地址
  cloud:
    config:
      server:
        git:
          uri:  https://github.com/syth7758258/test/
          search-paths: SpringCloudConfigTest
          username:
          password:
      label: master
#当前服务的端口
server:
  port: 8888

```

# ~~备注~~
 在使用配置文件的时候
 spring.cloud.config.server.git.uri：配置git仓库地址
 spring.cloud.config.server.git.searchPaths：配置仓库路径
 spring.cloud.config.label：配置仓库的分支
 spring.cloud.config.server.git.username：访问git仓库的用户名
 spring.cloud.config.server.git.password：访问git仓库的用户密码
 
 
 所以首先我们这里需要先在自己的GitHub上面先做一个普通文件的上传
 
 我们这里使用的git地址是
 https://github.com/syth7758258/test/tree/master/SpringCloudConfigTest
 
 其中里面的文件都是自己写的格式，一般的文件命名方式是：aaaa-bbb.yml或者aaaa-bbb.properties
 
 
 保证github上面有的时候，我们就可以启动项目。测试一下
 
 http://localhost:8888/abc/ad
 
 abc就是你的aaaa；ad表示-后面的值
 
 得到结果
 ```aidl
{"name":"abc","profiles":["ad"],"label":null,"version":"56b427a3a858806de984eb112af5a82123e492bc","state":null,"propertySources":[{"name":"https://github.com/syth7758258/test//SpringCloudConfigTest/abc-ad.properties","source":{"session_time":"86400","redis_cache_flag":"true","redis_cache_time":"86400","updateActionFlag":"true","isLdap":"false","redis_url":"10.20.1.191","mserver_url":"http://10.20.5.146:8080/itfer/rest/","push_app_key":"ca0b1e7f2d9356eebad65ae3","push_master_secret":"d796da93fdf1f3f4a0171f81","push_flags":"false","SKregistUrl":"22222","SKuploadUrl":"http://10.20.1.192:80/upload/UploadAvatarServlet","SKCreateGroup":"http://10.20.1.192:82/room/addRoomAndIcon"}}]}
```

其中需要记住：

name
profiles

在后面client里面会用到



###创建Config的客户端（主要是用来使用configServer获取到的文件内容）


首先这边肯定还是需要创建一个普通的springboot模块的项目

pom引用

```aidl
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--表示客户端-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies>
```


+ [x] 加入配置文件信息
```
spring.application.name=config-client
#这里配置的的是服务端给的
spring.cloud.config.label=master
spring.cloud.config.profile=ad
spring.cloud.config.name=abc
spring.cloud.config.uri= http://localhost:8888/

server.port=8881
```

+ [x] 如何使用呢

在需要使用的类中，加入配置即可

```aidl
@RestController
public class ConfigClientController {

    #这里就是配置文件中的值
    @Value("${SKregistUrl}")
    String SKregistUrl;

    @RequestMapping(value = "/get")
    public String getValue(){
        return SKregistUrl;
    }
}

```



#利用服务中心来做负载均衡

首先创建一个普通的springboot项目

1.加入spring-cloud-starter-netflix-eureka-server引用

2.配置启动类@EnableEurekaServer

3.配置服务类的标记信息：
spring.application.name=eureka-config
server.port=8889
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.instance.hostname=localhost


4.修改config-server（配置服务端）
    4.1：引用：spring-cloud-starter-netflix-eureka-client
         启动类注入：@EnableEurekaClient表示服务提供商
         配置信息中配置服务注册中心地址：
         eureka:
           client:
             service-url:
               defaultZone: http://localhost:8889/eureka
               
5.修改config-client(配置客户端)

    1.注册到服务中心去：
    
            加入引用：spring-cloud-starter-netflix-eureka-client
            启动类注入：@EnableEurekaClient表示服务提供商
            修改配置文件信息
            
                这句话注释
                #spring.cloud.config.uri= http://localhost:8888/
                
                下面是新增
                eureka.client.service-url.defaultZone=http://localhost:8889/eureka/这一句表示服务中心地址
                这一句表示true
                spring.cloud.config.discovery.enabled=true
                这一句表示服务id
                spring.cloud.config.discovery.service-id=config-server
                
6.验证方式：

        
    访问使用的客户端的值是否注入进去即可
    
    


#加入消息总线（BUS）（主要是用来，通过发送消息来，自动同步配置上面的文件信息）

这里面涉及到技术是

**RabbitMq消息**

***
准备工作

安装工作准备文档详情：https://www.cnblogs.com/junrong624/p/4121656.html


1.首先从官网上面先下载RabbitMq消息的安装包
    我们这里采用的是windwos版本的
    http://www.rabbitmq.com/
    下载的文件为：rabbitmq-server-3.7.7.exe
    安装方法：一直下一步即可
    
    
    
2.安装rabbitMq需要erlang的支持
    官网地址：http://www.erlang.org/download.html
    下载的文件为：otp_win64_21.0.1.exe
    安装方法：一直下一步即可


3.安装完了以后，windows在启动栏目里面，点击启动栏里面的RabbitMq service - start进行启动

4.启动完成以后，浏览器输入http://127.0.0.1:15672

        登录账号密码默认都是guest
        
        
        在admin选项卡里面，可以创建自己的用户和密码。
        
        然后记得分配权限，不然无法使用，set permission按钮
5.到现在我们所有的安装工作都已经完成了


***



这个东西一般是在配置中心的客户端进行实现的

以后我们这个配置中心的客户端如何使用呢。

我们可以搭建一个配置中心的服务端来获取git上面的文件信息。

然后在不同的微服务里面简历配置客户端，让他们用服务端得到的东西。

这样子就会对于配置中心用起来了。

那么更新了配置文件，如何让他们同时进行更新呢

就是我们下面要用的BUS消息总线。


*[改良config-client]
    1.pom文件加入引用信息
        ```
        <!--加入消息队列-->
                <dependency>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
                </dependency>
                <!--监控-->
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-actuator</artifactId>
                </dependency>
        ```
        
    2.启动类上面加入@EnableDiscoveryClient注解
    
    3.配置文件中加入条件信息
    
        这里是加入的rabbitmq相关的配置信息
        spring.rabbitmq.addresses=127.0.0.1
        spring.rabbitmq.port=5672
        spring.rabbitmq.username=springcloud
        spring.rabbitmq.password=springcloud
        
        #刷新时候，开启安全验证
        spring.cloud.bus.enabled=true
        #开启消息跟踪
        spring.cloud.bus.trace.enabled=true
        #
        management.endpoints.web.exposure.include=
     
    4.启动即可
  
  如何测试呢
  
  
    1.先访问使用客户端，获取git上面的文件进行展示
    
            例如：http://desktop-se7uhdf:8881/get
            
     2.用post方式访问：http://localhost:8881/actuator/bus-refresh
     
            进行文件刷新
     前提是git文件已经更改
     
     
     3.在访问  http://desktop-se7uhdf:8881/get查看效果
      




