---
title: Dubbo学习笔记
date: 2021-08-13 10:00:41
comments: true
tags: 
    - Dubbo
categories: 后端
thumbnail: logo.jpeg
banner: logo.jpeg
cover: logo.jpeg
---

# 一、基本知识

## 1 分布式基础理论

### 1.1什么是分布式

分布式系统是若干个独立计算机的集合，，计算机对于用户来说就像个单个系统，建立在网络上的软件系统。

随着互联网的发展，规模不断扩大，垂直应用架构无法应付，需要一个治理系统确保架构有条不紊的演进。

dubbo作用：治理和维护各个分系统。

 ### 1.2 发展演变：

![Untitled.png](Untitled.png)

**单一应用架构：**

流量小时，所有功能放到一起，减少部署节点和成本。

![Untitled%201.png](Untitled%201.png)

缺点：

扩展不容易，需要重新打包重新部署

协同开发不容易

扩大性能提升不容易

**垂直应用架构：**

小功能独立为单个应用，部署在多个服务器上

![Untitled%202.png](Untitled%202.png)

优点：

分工合作容易，分别负责开发

性能扩展容易，多部署到几个服务器

缺点：

单个应用从头到尾都有的，包括页面逻辑和数据库。但是市场上界面改变快，改了需要重新部署，没法做到页面和业务逻辑的分离

垂直应用越来越多，不可能完全应用于应用完全独立。用户调用订单和商品，物流调用订单等。应用之间需要交互

**分布式服务架构：**

核心业务单独抽取出来，用户应用抽取为页面和业务逻辑。当业务逻辑不变时只改界面，界面重新部署就行了。

![Untitled%203.png](Untitled%203.png)

问题：

用户web可能再A服务器上，业务再B服务器。如果A服务器调用B服务器功能时，因为不在一个进程内，分隔异地了，这2个代码调用需要RPC远程过程调用。

难点：

如何远程过程调用

如何拆分业务，增加业务的复用程度

好的分布式服务框架（RPC），应该有调度中心，来实时的监控业务中心实现动态调度，增加利用率，当某一个业务调用多了，可以让更多的服务器来跑业务量更大的业务，这是用流动计算架构

**流动计算架构：**

![Untitled%204.png](Untitled%204.png)

它负责维护业务之间复杂的关系，以及实时管理服务集群，当A访问量大了，可以多来几台服务器。而且多个A服务器中按照访问量动态调用，以此来提高集群的利用率。

### 1.3 RPC

**什么叫RPC**

远程过程调用，A服务器调用B服务器上功能。是一种技术思想，不是规范。它允许程序调用另一个共享空间（通常是共享网络的另一台服务器）上的过程或函数。而不是程序员显示编码这个远程调用的细节。即程序员无论调用本地的还是远程的函数，本质上编写的调用代码基本相同。

**RPC基本原理**

![Untitled%205.png](Untitled%205.png)

clent functions想调用server functions时，通过小助手请求，通过sockets创建和server的sockets连接，将想调用的信息传递个server服务器，server小助手收到信息后确定调用函数，server用传递过来的信息调用自己方法后把返回值再通过网络sockets返回给client

A和B架起网络连接后调用

**例子：**

![Untitled%206.png](Untitled%206.png)

A调用B时传递对象需要序列化，B先反序列化为对象，B服务器调用一下，拿到返回值，通过网络传输，序列化后传输给A服务器，A服务器需要反序列化后拿到返回值。

**调用核心：**

AB两个需要建立连接

传递数据需要序列化和反序列化

**决定性能有两点：**

看RPC能否快速建立连接

序列化于反序列化是否快

**RPC框架：**

dubbo、gRPC、Thrift、HSF

思想相同，用法不同

## 2 dubbo核心概念

### 2.1 简介

Apache Dubbo 是一款高性能、轻量级的开源 Java 服务框架

注：rpc面向服务、cloud的是restful面向资源

14.10停止更新，18.1将dubbo和dubbox合并后发布为2.6版本，最终将dubbo开放到apache。

netty功能负责网络传输，dubbo使用netty作为网络传输框架。说到网络传输自然离不开Socket，Socket是端到端的连接。dubbo是无中心化，每个client端都能与server端连接，每个client端同时又是server端。

**特性：**

1.**面向接口代理的高性能RPC调用**

提供高性能的基于代理的远程调用能力，服务以接口为粒度，为开发者屏蔽远程调用底层细节。

A调用B只需要将接口拿来调用，dubbo会自动的找B服务器的这段代码，屏蔽了整个调用细节。

类似Mybatis，调用数据库是只需要写do-mapping接口，调用接口方法就行了。

**2.智能负载均衡**

内置多种负载均衡策略，智能感知下游节点健康状况，显著减少调用延迟，提高系统吞吐量。

一个服务1台服务器不够时部署多台后，会自动找空闲服务器运行。

不会压垮某一台服务器，也不会让某服务器太闲浪费资源。

**3.服务自动注册与发现**

支持多种注册中心服务，服务实例上下线实时感知。

业务非常多，订单调用支付业务时如何知道业务服务器是否知道有问题切在哪，为了动态感知，将所有程序注册到注册中心，维护注册清单。调用时会问一下调用的服务在哪个服务器，选择请求量最小的。建立通信，进行远程调用。

**4.高度可扩展能力**

遵循微内核+插件的设计原则，所有核心能力如Protocol、Transport、Serialization被设计为扩展点，平等对待内置实现和第三方实现。

所有东西都可扩展。

**5.运行期流量调度**

内置条件、脚本等路由策略，通过配置不同的路由规则，轻松实现灰度发布，同机房优先等功能。

灰度发布：比如有个用户服务，现在在100台上服务器在跑，用户服务做了升级，害怕升级不稳定，可以先选定20台服务器，让他们用新版本的用户服务，剩下80台用旧版本的用户服务。等20台用的没问题了，再选20台，一点一点增加，知道100台都过度到新版本用户服务。可以配置不同路由规则，请求进来后一部分用新版本服务。慢慢从旧服务转变到新服务的过程。

**6.可视化的服务治理与运维**

提供丰富服务治理、运维工具：随时查询服务元数据、服务健康状态及调用统计，实时下发路由策略、调整配置参数。

可以随时查询服务的信息、健康状况、调用统计记录等等。通过可视化界面。

### 2.2 dubbo设计架构

![Untitled%207.png](Untitled%207.png)

[节点角色说明](https://www.notion.so/6c6f03ae75794978a31ec967f82fec02)

**运行流程**

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。当某一个提供者下线了，基于长连接方式，将这个变更推送给消费者，可以实时知道有一个提供者不能调用了。当消费者拿到所有可以调的服务后，可以调用提供者提供的服务，调用时根据负载均衡算法选择一个提供者调用。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

0、1、2是初始时启动完成的，第3、5布是异步的。第4步是同步调用。

应先创建提供者，注册到注册中心，再编写消费者，消费者从注册中心调用提供者。然后再编写消费者如何消费提供者提供的功能。

**连通性[](https://dubbo.apache.org/zh/docs/v2.7/user/preface/architecture/#%E8%BF%9E%E9%80%9A%E6%80%A7)**

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
- 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
- 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
- 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
- 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
- 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
- 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
- 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

**健壮性**

- 监控中心宕掉不影响使用，只是丢失部分采样数据
- 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
- 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
- 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

**伸缩性**

- 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
- 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

**升级性[](https://dubbo.apache.org/zh/docs/v2.7/user/preface/architecture/#%E5%8D%87%E7%BA%A7%E6%80%A7)**

当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。

## 3 dubbo环境搭建

### 3.1 搭建Zookeeper注册中心

**3.1.1 安装jdk**

上传jdk包到/opt/software

tar -zxvf jdk-8u291-linux-x64.tar.gz

```jsx
export JAVA_HOME=/opt/software/jdk1.8.0_291/  #jdk安装目录
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

**3.1.2安装zookeeper**

拷贝到/opt/software

解压

```jsx
tar -zxvf apache-zookeeper-3.7.0.tar.gz
```

**3.1.3修改zookeeper配置**

将"/opt/software/apache-zookeeper-3.7.0/conf/zoo_sample.cfg"拷贝份zoo.cfg

打开zoo.cfg，修改vim zoo.cfg

```jsx
dataDir=/opt/software/apache-zookeeper-3.7.0/zkData/
```

**3.1.4操作zookeeper**

(1) 启动zookeeper

```jsx
cd /opt/software/apache-zookeeper-3.7.0/bin
./zkServer.sh start
```

(2) 查看进程

```jsx
jps
./zkServer.sh status
```

(3)启动客户端

```jsx
./zkCli.sh
quit
```

(4) 关闭zookeeper

```jsx
./zkServer.sh stop
```

### 3.2. 搭建监控中心

[https://github.com/apache/dubbo-admin](https://github.com/apache/dubbo-admin)

root

root

3.2.1 **服务治理**

服务治理的部分，按照Dubbo 2.7的格式进行配置，同时兼容Dubbo 2.6，详见[这里](https://github.com/apache/dubbo-admin/wiki/%E6%9C%8D%E5%8A%A1%E6%B2%BB%E7%90%86%E5%85%BC%E5%AE%B9%E6%80%A7%E8%AF%B4%E6%98%8E)

3.2.2 **前端部分**

- 使用[Vue.js](https://vuejs.org/)作为javascript框架
- [dubbo-admin-ui/README.md](https://github.com/apache/dubbo-admin/blob/develop/dubbo-admin-ui/README.md)中有更详细的介绍
- 设置 npm **代理镜像** : 如果遇到了网络问题，可以设置npm代理镜像来加速npm install的过程：在~/.npmrc中增加 `registry =https://registry.npm.taobao.org`

3.2.3 **后端部分**

- 标准spring boot工程
- [application.properties配置说明](https://github.com/apache/dubbo-admin/wiki/Dubbo-Admin%E9%85%8D%E7%BD%AE%E8%AF%B4%E6%98%8E)

3.2.4 **生产环境配置**

1. 下载代码: `git clone https://github.com/apache/dubbo-admin.git`
2. 在 `dubbo-admin-server/src/main/resources/application.properties`中指定注册中心地址
3. 构建

    > mvn clean package -Dmaven.test.skip=true

4. 启动
    - `mvn --projects dubbo-admin-server spring-boot:run`或者
    - `cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.1.jar`
5. 访问 `http://localhost:8080`

---

3.2.5 **开发环境配置**

- 运行`dubbo admin server, dubbo admin server`是一个标准的spring boot项目, 可以在任何java IDE中运行它
- 运行`dubbo admin ui` `dubbo admin ui`由npm管理和构建，在开发环境中，可以单独运行: `npm run dev`
- 页面访问 访问 `http://localhost:8081`, 由于前后端分开部署，前端支持热加载，任何页面的修改都可以实时反馈，不需要重启应用。

3.2.6 **Swagger 支持**

部署完成后，可以访问 [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html) 来查看所有的restful api

### 4 dubbo-helloworld

**4.1 需求**

![Untitled%208.png](Untitled%208.png)

**4.2 工程架构** 

![Untitled%209.png](Untitled%209.png)

服务化最佳实践

[https://dubbo.apache.org/zh/docs/v2.7/user/best-practice/](https://dubbo.apache.org/zh/docs/v2.7/user/best-practice/)

分包：

建议将服务接口、服务模型、服务异常等均放在 API 包中，因为服务模型和异常也是 API 的一部分，这样做也符合分包原则：重用发布等价原则(REP)，共同重用原则(CRP)。

如果需要，也可以考虑在 API 包中放置一份 Spring 的引用配置，这样使用方只需在 Spring 加载过程中引用此配置即可。配置建议放在模块的包目录下，以免冲突，如：`com/alibaba/china/xxx/dubbo-reference.xml`。

粒度：

服务接口尽可能大粒度，每个服务方法应代表一个功能，而不是某功能的一个步骤，否则将面临分布式事务问题，Dubbo 暂未提供分布式事务支持。

服务接口建议以业务场景为单位划分，并对相近业务做抽象，防止接口数量爆炸。

不建议使用过于抽象的通用接口，如：`Map query(Map)`，这样的接口没有明确语义，会给后期维护带来不便。

```jsx
<!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
<dubbo:application name="user-service-provider"></dubbo:application>

<!-- 2、指定注册中心的位置 -->
<!-- <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry> -->
<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

<!-- 3、指定通信规则（通信协议？通信端口） -->
<dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>

<!-- 4、暴露服务   ref：指向服务的真正的实现对象 -->
<dubbo:service interface="gmall.service.OrderService"
	ref="userServiceImpl01" timeout="1000" version="1.0.0">
	<dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
</dubbo:service>

<!--统一设置服务提供方的规则  -->
<dubbo:provider timeout="1000"></dubbo:provider>
```

**4.3 创建模块**

1、gmall-interface：公共接口层（model，service，exception…）

作用：定义公共接口，也可以导入公共依赖

```jsx
1、Bean模型
package gmall.bean;

import java.io.Serializable;

/**
 * 用户地址
 *
 * @author lfy
 */
public class UserAddress implements Serializable {

    private Integer id;
    private String userAddress; //用户地址
    private String userId; //用户id
    private String consignee; //收货人
    private String phoneNum; //电话号码
    private String isDefault; //是否为默认地址    Y-是     N-否

    public UserAddress() {
        super();
        // TODO Auto-generated constructor stub
    }

    public UserAddress(Integer id, String userAddress, String userId, String consignee, String phoneNum,
                       String isDefault) {
        super();
        this.id = id;
        this.userAddress = userAddress;
        this.userId = userId;
        this.consignee = consignee;
        this.phoneNum = phoneNum;
        this.isDefault = isDefault;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getConsignee() {
        return consignee;
    }

    public void setConsignee(String consignee) {
        this.consignee = consignee;
    }

    public String getPhoneNum() {
        return phoneNum;
    }

    public void setPhoneNum(String phoneNum) {
        this.phoneNum = phoneNum;
    }

    public String getIsDefault() {
        return isDefault;
    }

    public void setIsDefault(String isDefault) {
        this.isDefault = isDefault;
    }

}

2、OrderService 接口
package gmall.service;

import gmall.bean.UserAddress;
import java.util.List;

/**
 * 订单服务
 * @author guoyh
 * @date 2021/7/5
 */
public interface OrderService {

    public List<UserAddress> initOrder(String userId);
}

3、UserService接口
package gmall.service;

import gmall.bean.UserAddress;
import java.util.List;

/**
 * 用户服务
 * @author guoyh
 * @date 2021/7/5
 */
public interface UserService {

    /**
     * 按照用户id返回所有收货地址
     *
     * @param userId
     * @return List<UserAddress>
     */
    public List<UserAddress> getUserAddressList(String userId);

}
```

2、user-service-provider：用户模块（对用户接口的实现）

```jsx
1、pom.xml
	<!-- 引入公共接口 -->
	<dependency>
	  <groupId>org.apache.dubbo</groupId>
	  <artifactId>gmall-interface</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	</dependency>	

2、UserServiceImpl服务
package com.atguigu.gmall.service.impl;

import gmall.bean.UserAddress;
import gmall.service.UserService;
import java.util.Arrays;
import java.util.List;

/**
 * 1、将服务提供者将服务注册到注册中心（暴漏服务）
 * 1) 导入dubbo依赖
 * 2、让服务消费者去注册中心订阅服务的服务地址
 *
 * @author guoyh
 * @date 2021/7/5
 */
public class UserServiceImpl implements UserService {

    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        UserAddress userAddressOne = new UserAddress();
        userAddressOne.setUserAddress("北京市区昌平");
        UserAddress userAddressTwo = new UserAddress();
        userAddressTwo.setUserAddress("天安门");
        return Arrays.asList(userAddressOne, userAddressTwo);
    }
}
```

3、user-service-consumer：订单模块（调用用户模块）

```jsx
1、pom.xml
	<dependency>
	  <groupId>org.apache.dubbo</groupId>
	  <artifactId>gmall-interface</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	</dependency>
	
2、OrderServiceImpl服务
package com.atguigu.gmall.service.impl;

import gmall.bean.UserAddress;
import gmall.service.OrderService;
import gmall.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

/**
 * @author guoyh
 * @date 2021/7/5
 */
@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    UserService userService;

    @Override
    public List<UserAddress> initOrder(String userId) {
        System.out.println("用户id:" + userId);
        // 1、查询用户收获地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        for (UserAddress address : userAddressList) {
            System.out.println(address.getUserAddress());
        }
        return userAddressList;
    }
}
```

现在这样是无法进行调用的。我们gmall-order-web引入了gmall-interface，但是interface的实现是gmall-user，我们并没有引入，而且实际他可能还在别的服务器中。

**4.4 使用dubbo改造**

1、改造user-service-provider作为服务提供者

```jsx
1、pom.xml
	<!-- 引入dubbo -->
	<!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
	<dependency>
	  <groupId>com.alibaba</groupId>
	  <artifactId>dubbo</artifactId>
	  <version>2.6.2</version>
	</dependency>
	<!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端端 -->
	<dependency>
	  <groupId>org.apache.curator</groupId>
	  <artifactId>curator-framework</artifactId>
	  <version>2.12.0</version>
	</dependency>

2、配置提供者provider.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
    <dubbo:application name="user-service-provider"></dubbo:application>

    <!-- 2、指定注册中心的位置 -->
    <!-- <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry> -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

    <!-- 3、指定通信规则（通信协议？通信端口） -->
    <dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>

    <!-- 4、暴露服务   ref：指向服务的真正的实现对象 -->
    <dubbo:service interface="gmall.service.UserService"
                   ref="userServiceImpl" timeout="1000" version="1.0.0">
        <!--<dubbo:method name="getUserAddressList" timeout="1000"/>-->
    </dubbo:service>

    <!-- 服务的真正实现 -->
    <bean id="userServiceImpl" class="com.atguigu.gmall.service.impl.UserServiceImpl"></bean>
</beans>

3.启动服务
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApplication {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"provider.xml"});
        context.start();
        //为了不让服务终止，在这阻塞读取一个字符
        System.in.read(); // 按任意键退出
    }
}
```

2、改造user-service-consumer作为服务消费者

```jsx
1、pom.xml
	<!-- 引入dubbo -->
	<!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
	<dependency>
	  <groupId>com.alibaba</groupId>
	  <artifactId>dubbo</artifactId>
	  <version>2.6.2</version>
	</dependency>
	<!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端端 -->
	<dependency>
	  <groupId>org.apache.curator</groupId>
	  <artifactId>curator-framework</artifactId>
	  <version>2.12.0</version>
	</dependency>

2、配置消费者consumer.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 让注解能够生效 -->
    <context:component-scan base-package="com.atguigu.gmall.service.impl"/>

    <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
    <dubbo:application name="order-service-consumer"></dubbo:application>

    <!-- 2、指定注册中心的位置 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>
		
		<!-- 3、生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference interface="gmall.service.UserService"
                     id="userService">
        <dubbo:method name="getUserAddressList"/>
    </dubbo:reference>

    <!-- 6、连接监控中心 -->
    <dubbo:monitor protocol="registry"></dubbo:monitor>
</beans>
```

3、调用测试

```jsx
package com.atguigu.gmall.service.impl;

import gmall.bean.UserAddress;
import gmall.service.OrderService;
import gmall.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author guoyh
 * @date 2021/7/5
 */
@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    UserService userService;

    @Override
    public List<UserAddress> initOrder(String userId) {
        System.out.println("用户id:" + userId);
        // 1、查询用户收获地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        for (UserAddress address : userAddressList) {
            System.out.println(address.getUserAddress());
        }
        return userAddressList;
    }
}
```

4、注解版本

```jsx
1、服务提供方
<dubbo:application name="gmall-user"></dubbo:application>
  <dubbo:registry address="zookeeper://118.24.44.169:2181" />
  <dubbo:protocol name="dubbo" port="20880" />
<dubbo:annotation package="com.atguigu.gmall.user.impl"/>

import com.alibaba.dubbo.config.annotation.Service;
import com.atguigu.gmall.bean.UserAddress;
import com.atguigu.gmall.service.UserService;
import com.atguigu.gmall.user.mapper.UserAddressMapper;

@Service //使用dubbo提供的service注解，注册暴露服务
public class UserServiceImpl implements UserService {

	@Autowired
	UserAddressMapper userAddressMapper;

2、服务消费方
<dubbo:application name="gmall-order-web"></dubbo:application>
<dubbo:registry address="zookeeper://118.24.44.169:2181" />
<dubbo:annotation package="com.atguigu.gmall.order.controller"/>

import com.alibaba.dubbo.config.annotation.Reference;
import gmall.bean.UserAddress;
import gmall.service.OrderService;
import gmall.service.UserService;
@Controller
public class OrderController {
	
	@Reference  //使用dubbo提供的reference注解引用远程服务
	UserService userService;
	
  @Override
    public List<UserAddress> initOrder(String userId) {
        System.out.println("用户id:" + userId);
        // 1、查询用户收获地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        return userAddressList;
    }
}
```

### 5、监控中心

5.1、dubbo-admin

图形化的服务管理页面；安装时需要指定注册中心地址，即可从注册中心中获取到所有的提供者/消费者进行配置管理

5.2、dubbo-monitor-simple

简单的监控中心；

1)、安装

```jsx
1、下载 dubbo-ops
https://github.com/apache/incubator-dubbo-ops
2、修改配置指定注册中心地址
进入 dubbo-monitor-simple\src\main\resources\conf
修改 dubbo.properties文件
3、打包dubbo-monitor-simple
mvn clean package -Dmaven.test.skip=true
4、解压 tar.gz 文件，并运行start.bat
如果缺少servlet-api，自行导入servlet-api再访问监控中心
5、启动访问8080
```

2)、监控中心配置

```jsx
所有服务配置连接监控中心，进行监控统计
<!-- 监控中心协议，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心 -->
<dubbo:monitor protocol="registry"></dubbo:monitor>
```

Simple Monitor 挂掉不会影响到 Consumer 和 Provider 之间的调用，所以用于生产环境不会有风险。

Simple Monitor 采用磁盘存储统计信息，请注意安装机器的磁盘限制，如果要集群，建议用mount共享磁盘。

### 6、SpringBoot 整合 Dubbo

```jsx
<!-- SpringBoot版本 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<!-- JDK 1.8-->
<properties>
    <java.version>1.8</java.version>
</properties>

<!-- 引入公共接口项目-->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>gmall-interface</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </dependency>
```

6.1、引入spring-boot-starter以及dubbo和curator的依赖

注意starter版本适配：

![Untitled%2010.png](Untitled%2010.png)

```jsx
<!-- Dubbo Spring Boot Starter -->
<dependency>
  <groupId>com.alibaba.boot</groupId>
  <artifactId>dubbo-spring-boot-starter</artifactId>
  <version>0.2.0</version>
</dependency>
```

6.2、配置application.properties

```jsx
1、提供者配置
#dubbo服务名称
dubbo.application.name=boot-order-service-provider
#注册中心地址
dubbo.registry.address=127.0.0.1:2181
#注册中心协议
dubbo.registry.protocol=zookeeper
#通信协议
dubbo.protocol.name=dubbo
dubbo.protocol.prot=20880
#监控中心地址:registry从注册中心自动发现
dubbo.monitor.protocol=registry
#application.name就是服务名，不能跟别的dubbo提供端重复
#registry.protocol 是指定注册中心协议
#registry.address 是注册中心的地址加端口号
#protocol.name 是分布式固定是dubbo,不要改。
#base-package  注解方式要扫描的包

2、消费者配置
dubbo.application.name=gmall-order-web
dubbo.registry.protocol=zookeeper
dubbo.registry.address=192.168.67.159:2181
dubbo.scan.base-package=com.atguigu.gmall
dubbo.protocol.name=dubbo
```

3、dubbo注解

```jsx
#提供者:实现类开放服务
import com.alibaba.dubbo.config.annotation.Service;
@Service

#消费者:从注册中心自动获取可用的实现服务
import com.alibaba.dubbo.config.annotation.Reference;
@Reference

如果没有在配置中写dubbo.scan.base-package,还需要使用@EnableDubbo注解
import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
@EnableDubbo
```

# 二、dubbo配置

## 1、配置原则

![Untitled%2011.png](Untitled%2011.png)

JVM 启动 -D 参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。

XML /applicatoin.properties次之，如果在 XML 中有配置，则 dubbo.properties 中的相应配置项无效。(自己测试结果为dubbo.properties 比XML/application.properties高)

Properties 最后，相当于缺省值，只有 XML 没有配置时，dubbo.properties 的相应配置项才会生效，通常用于共享公共配置，比如应用名。

[https://dubbo.apache.org/zh/docsv2.7/user/examples/preflight-check/](https://dubbo.apache.org/zh/docsv2.7/user/examples/preflight-check/)

**启动时检查：**

Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 `check="true"`。

可以通过 `check="false"` 关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

```jsx
-D
java -Ddubbo.reference.com.foo.BarService.check=false
java -Ddubbo.reference.check=false
java -Ddubbo.consumer.check=false 
java -Ddubbo.registry.check=false

XML
<dubbo:reference interface="com.foo.BarService" check="false" />
<dubbo:consumer check="false" />
<dubbo:registry check="false" />

dubbo.properties
dubbo.reference.com.foo.BarService.check=false
dubbo.reference.check=false
dubbo.consumer.check=false
dubbo.registry.check=false
```

## 2、重试次数

失败自动切换，当出现失败，重试其它服务器，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。

```jsx
重试次数配置如下：
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

## 3、超时时间

由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间.

3.1、消费端

```jsx
全局超时配置
<dubbo:consumer timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:reference interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:reference>
```

3.2、提供方

```jsx
全局超时配置
<dubbo:provider timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:provider interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:provider>
```

3.3、配置规则

Dubbo推荐在Provider上尽量多配置Consumer端属性

```jsx
1、作服务的提供者，比服务使用方更清楚服务性能参数，如调用的超时时间，合理的重试次数，等等
2、在Provider配置后，Consumer不配置则会使用Provider的配置值，即Provider配置可以作为Consumer的缺省值。否则，Consumer会使用Consumer端的全局设置，这对于Provider不可控的，并且往往是不合理的
```

配置的覆盖规则：

1) 方法级配置别优于接口级别，即小Scope优先

2) Consumer端配置 优于 Provider配置 优于 全局配置，

3) 最后是缺省值（见配置文档）

## 4、版本号

[https://dubbo.apache.org/zh/docsv2.7/user/examples/multi-versions/](https://dubbo.apache.org/zh/docsv2.7/user/examples/multi-versions/)

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。可以按照以下的步骤进行版本迁移：

在低压力时间段，先升级一半提供者为新版本

再将所有消费者升级为新版本

然后将剩下的一半提供者升级为新版本

```jsx
服务老版本提供者配置：
<dubbo:service interface="com.foo.BarService" version="1.0.0" />

服务新版本提供者配置：
<dubbo:service interface="com.foo.BarService" version="2.0.0" />

服务老版本消费者配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />

新版本服务消费者配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />

如果不需要区分版本，可以按照以下的方式配置：
<dubbo:reference id="barService" interface="com.foo.BarService" version="*" />
```

## 5、本地存根

[https://dubbo.apache.org/zh/docsv2.7/user/examples/local-stub/](https://dubbo.apache.org/zh/docsv2.7/user/examples/local-stub/)

远程服务后，客户端通常只剩下接口，而实现全在服务器端，但提供方有些时候想在客户端也执行部分逻辑，比如：做 ThreadLocal 缓存，提前验证参数，调用失败后伪造容错数据等等，此时就需要在 API 中带上 Stub，客户端生成 Proxy 实例，会把 Proxy 通过构造函数传给 Stub [1](https://dubbo.apache.org/zh/docsv2.7/user/examples/local-stub/#fn:1)，然后把 Stub 暴露给用户，Stub 可以决定要不要去调 Proxy。

![stub.jpg](stub.jpg)

在消费方，写一个远程接口本地的实现，提供 Stub 的实现 ：

```jsx
package com.foo;
public class BarServiceStub implements BarService {
    private final BarService barService;
    
    // 必须有一个有参构造，并且使构造函数传入真正的远程代理对象
    public BarServiceStub(BarService barService){
        this.barService = barService;
    }
 
    public String sayHello(String name) {
        // 此代码在客户端执行, 你可以在客户端做ThreadLocal本地缓存，或预先验证参数是否合法，等等
        // ...自己的代码
				// 当自己代码执行结束后，再调真正的远程服务
				try {
            return barService.sayHello(name);
        } catch (Exception e) {
            // 你可以容错，可以做任何AOP拦截事项
            return "容错数据";
        }
    }
}
```

在 spring 配置文件中按以下方式配置：

```jsx
提供者方配置
<dubbo:service interface="com.foo.BarService" stub="true" />

消费者方配置
<dubbo:reference interface="gmall.service.UserService"
  id="userService" timeout="5000" retries="3" version="*" 
	stub="com.atguigu.gmall.service.impl.UserServiceStub">
<dubbo:method name="getUserAddressList" timeout="1000"/>
</dubbo:reference>
```

---

1. Stub 必须有可传入 Proxy 的构造函数。 
2. 在 interface 旁边放一个 Stub 实现，它实现 BarService 接口，并有一个传入远程 BarService 实例的构造函数 

## 6、SpringBoot方式配置

### 6.1、超时属性

```jsx
import com.alibaba.dubbo.config.annotation.Service;
@Service(timeout)
import com.alibaba.dubbo.config.annotation.Reference;
@Reference(timeout = 0)
```

### 6.2、SpringBoot与Dubbo整合的三种方式

**6.2.1、第一种**

```jsx
1.导入dobuuo-starter
<dependency>
	<groupId>com.alibaba.boot</groupId>
	<artifactId>dubbo-spring-boot-starter</artifactId>
	<version>0.2.0</version>
</dependency>

2.配置application.properties
#dubbo服务名称
dubbo.application.name=boot-order-service-provider
#注册中心地址
dubbo.registry.address=127.0.0.1:2181
#注册中心协议
dubbo.registry.protocol=zookeeper
#通信协议
dubbo.protocol.name=dubbo
dubbo.protocol.prot=20880
#监控中心地址:registry从注册中心自动发现
dubbo.monitor.protocol=registry

3.暴漏服务注解@Service

4.使用远程服务@Reference

5.开启Dubbo注解功能@EnableDubbo

#老版本会指定包扫描规则
dubbo.scan.base-packages=com
```

**6.2.2、第二种：保留XML**

想要做到方法级别的配置，保留dubbo的xml配置文件，之后可以在xml中做到method方法级别的配置。

```jsx
# 拷贝xml文件到resource目录,删除application.properties中dubbo的相关配置
# 所有配置都在xml中
1.导入dobuuo-starter
<dependency>
	<groupId>com.alibaba.boot</groupId>
	<artifactId>dubbo-spring-boot-starter</artifactId>
	<version>0.2.0</version>
</dependency>

2.使用@ImportResource导入dubbo的xml配置文件
//@EnableDubbo
@ImportResource(location="classpath:provider.xml")
```

**6.2.3、第三种:注解API配置类**

将每一个组件手动创建到容器中

```jsx
1.创建配置类
package com.atguigu.gmall.config;

import com.alibaba.dubbo.config.*;
import gmall.service.UserService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;

@Configuration
public class MyDubboConfig {

    /**
     * 提供者名
     * @return
     */
    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("boot-order-service-provider-config");
        return applicationConfig;
    }

    /**
     * 注册中心地址
     * @return
     */
    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setProtocol("zookeeper");
        registryConfig.setAddress("127.0.0.1:2181");
        return registryConfig;
    }

    /**
     * 通信规则
     * @return
     */
    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20882);
        return protocolConfig;
    }

    /**
     * 配置暴漏服务
     * @return
     */
    @Bean
    public ServiceConfig<UserService> userServiceConfig(UserService userService) {
        ServiceConfig<UserService> userServiceConfig = new ServiceConfig<>();
        //配置接口级别
        userServiceConfig.setInterface(UserService.class);
        userServiceConfig.setRef(userService);
        userServiceConfig.setVersion("1.0.0");

        //配置方法级别
        MethodConfig methodConfig = new MethodConfig();
        methodConfig.setName("getUserAddressList");
        methodConfig.setTimeout(1000);

        userServiceConfig.setMethods(Arrays.asList(methodConfig));
        return userServiceConfig;
    }

    /**
     * 设置监控中心
     * @return
     */
    @Bean
    public MonitorConfig monitorConfig() {
        MonitorConfig monitorConfig = new MonitorConfig();
        monitorConfig.setProtocol("registry");
        return monitorConfig;
    }
}

2.暴露服务接口
@Service //暴露服务

3.启动dubbo并指定扫描路径
@EnableDubbo(scanBasePackages = "com.atguigu.gmall")
```

# 三、高可用

## 3.1、zookeeper宕机与dubbo直连

![Untitled%2012.png](Untitled%2012.png)

注册中心的作用就是保存服务提供者位置信息，我们完全可以绕过注册中心使用dubbo直连.

宕机了有缓存，依旧可以调用。没有注册中心使用直连

```jsx
@Reference(timeout = 0,url = "127.0.0.1:20880")
UserService userService;
```

## 3.2、集群下dubbo负载均衡机制

[https://dubbo.apache.org/zh/docsv2.7/user/examples/loadbalance/](https://dubbo.apache.org/zh/docsv2.7/user/examples/loadbalance/)

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用,

```jsx
**Random LoadBalance**
随机，按权重设置随机概率。
在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。
**RoundRobin LoadBalance**
轮循，按公约后的权重设置轮循比率。
存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。
**LeastActive LoadBalance**
最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。
使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。
**ConsistentHash LoadBalance**
一致性 Hash，相同参数的请求总是发到同一提供者。
当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。算法参见：http://en.wikipedia.org/wiki/Consistent_hashing
缺省只对第一个参数 Hash，如果要修改，请配置 <dubbo:parameter key="hash.arguments" value="0,1" />
缺省用 160 份虚拟节点，如果要修改，请配置 <dubbo:parameter key="hash.nodes" value="320" />
```

### 3.2.1、Random LoadBalance

![Untitled%2013.png](Untitled%2013.png)

### 3.2.2、RoundRobin LoadBalance

第一调完，必须去第二个,根据权重1只能占2次，2占4次，3占1次。

![Untitled%2014.png](Untitled%2014.png)

### 3.2.3、LeastActive LoadBalance

![Untitled%2015.png](Untitled%2015.png)

每次都统计上次请求时间

调哪个前会查那个上次最快

### 3.2.4、ConsistentHash LoadBalance

比如getUser,id都等于1的，根据hash分布都会来到1号机器。

![Untitled%2016.png](Untitled%2016.png)

### 3.2.5、配置

默认为随机负载均衡机制

![Untitled%2017.png](Untitled%2017.png)

![Untitled%2018.png](Untitled%2018.png)

XML：

```jsx
### **服务端服务级别**

`<dubbo:service interface="..." loadbalance="roundrobin" />`

### **客户端服务级别**

`<dubbo:reference interface="..." loadbalance="roundrobin" />`

### **服务端方法级别**

`<dubbo:service interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:service>`

### **客户端方法级别**

`<dubbo:reference interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:reference>`
```

注解：

```jsx
@Reference(timeout = 0,loadbalance = "roundrobin") 

```

![Untitled%2019.png](Untitled%2019.png)

## 3.3、整合hystrix，服务熔断与降级处理

### 3.3.1、服务降级

什么是服务降级？

当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作

可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。

支持2种：

- mock=force:return+null 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。（直接在客户端返回null，不发起远程调用）
- 还可以改为 mock=fail:return+null 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。（到达一定超时时间，超时后调用在失败后，再返回 null 值，这个是服务已经调用了）

通过服务降级手段，牺牲非核心业务的占用的资源，达到让其他核心业务能占用服务器更多资源

**配置，在消费者端**

第一种：屏蔽（不进行远程调用，返回null）

![Untitled%2020.png](Untitled%2020.png)

第二种：容错（当远程调用失败后，返回null）

![Untitled%2021.png](Untitled%2021.png)

### 3.3.2、集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

```jsx
1.**Failover Cluster**
失败自动切换，当出现失败，重试其它服务器。通常**用于读操作**，但重试会带来更长延迟。
可通过 retries="2" 来设置重试次数(不含第一次)。

重试次数配置如下：
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>

2.**Failfast Cluster**
快速失败，只发起一次调用，失败立即报错。通常用于**非幂等性**的写操作，比如新增记录。

3.**Failsafe Cluster**
失败安全，出现异常时，直接**忽略**。通常用于写入审计日志等操作。

4.**Failback Cluster**
**失败自动恢复**，后台记录失败请求，定时重发。通常**用于消息通知**操作。

5.**Forking Cluster**
**并行调用**多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，
但需要浪费更多服务资源。可通过 forks="2" 来设置最大并行数。太浪费资源。

6.**Broadcast Cluster**
广播**调用所有提供者，逐个调用，任意一台报错则报错** [2]。
通常**用于通知所有提供者更新缓存**或日志等本地资源信息。

7.**集群模式配置**
按照以下示例在服务提供方和消费方配置集群模式
<dubbo:service cluster="failsafe" />
或
<dubbo:reference cluster="failsafe" />
```

### 3.3.3、实际开发整合hystrix提供容错（SpringCloud默认整合的框架）

Hystrix 旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。

**1、配置spring-cloud-starter-netflix-hystrix**

spring boot官方提供了对hystrix的集成，直接在消费者、提供者的pom.xml里加入依赖：

```jsx
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
	<version>1.4.4.RELEASE</version>
</dependency>
```

然后在Application类上增加@EnableHystrix来启用hystrix starter：

```jsx
@SpringBootApplication
@EnableHystrix
public class ProviderApplication {
}
```

**2、配置Provider端**

在Dubbo的Provider上增加@HystrixCommand配置，这样子调用就会经过Hystrix代理。

```jsx
@Service(version = "1.0.0")
public class HelloServiceImpl implements HelloService {
		//使用Hystrix代理，进行处理容错异常
    @HystrixCommand(commandProperties = {
     @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
     @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public String sayHello(String name) {
        // System.out.println("async provider received: " + name);
        // return "annotation: hello, " + name;
        throw new RuntimeException("Exception to show hystrix enabled.");
    }
}
```

**3、配置Consumer端**

对于Consumer端，则可以增加一层method调用，并在method上配置@HystrixCommand。当调用出错时，会走到fallbackMethod = "reliable"的调用里。

```jsx
@Reference(version = "1.0.0")
    private HelloService demoService;
		//fallbackMethod 回调方法,一旦调用失败出错后，调用reliable方法
		//就不用在调用代码中trycatch了
    @HystrixCommand(fallbackMethod = "reliable")
    public String doSayHello(String name) {
        return demoService.sayHello(name);
    }
    public String reliable(String name) {
        return "hystrix fallback value";
    }
```

# 四、Dubbo原理

## 4.1、RPC原理

rpc就是想完成一次远程调用

![Untitled%2022.png](Untitled%2022.png)

一次完整的RPC调用流程（同步调用，异步另说）如下：

1）Computer01（client）发起远程调用请求,接下来有一个代理对象client stub

**2）client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；**

**3）client stub找到服务地址，并将消息发送到服务端server stub；**

**4）server stub收到消息后进行解码(可能传输的是序列化对象，需要反序列化)；**

**5）server stub根据解码结果调用本地的方法服务；**

**6）本地方法服务执行完成后，将数据结果返回给server stub；**

**7）server stub将返回结果打包(序列化)成消息通过网络发送至消费方client stub；**

**8）client stub接收到消息，并进行解码(反序列化)；**

9）服务消费方Computer01得到最终结果。

RPC框架的目标就是要2~8这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。

## 4.2、netty通信原理

Dubbo底层通信时是使用netty，Netty时基于Java的NIO(Non-Blocking IO)非阻塞,BIO(Blocking IO)阻塞IO。

**BIO：**

![Untitled%2023.png](Untitled%2023.png)

每一个请求进来开一个线程Socket，读取Socket的输入流、进行业务逻辑等等。同时在这操作，并且此时在我们业务逻辑没完成之前，我们线程都不能得到释放的，这样我们的服务器就不能同时处理大量的请求。因为有大量的线程在等待业务逻辑的完成。

**NIO：**

![Untitled%2024.png](Untitled%2024.png)

Channel：通道，通道里还有Buffer，利用Buffer进行数据传输。

Selector ：一般称 为**选择器** ，也可以翻译为 **多路复用器。**

通道状态：Connect（连接就绪）、Accept（接受就绪）、Read（读就绪）、Write（写就绪）。

**基本原理：**

1个Selector会有很多通道注册了进来，Selector通过监听多个通道，监听会有很多事件，如当Connect了发生什么，Accept了会做什么，Read或Write发生了时要做什么，当发现某一个通道数据准备好了。所以我们通过1个Selector监听多个通道的方式，当某个通道任何一个状态准备好了，我们可以额外开一个线程进行处理。这叫多路复用模型。

![Untitled%2025.png](Untitled%2025.png)

1.启动

2.监听某一个端口

3.初始化一个通道，注册到Selector中

4.轮询监听通道的accept事件

5.当accept事件发生后，扔到任务队列，处理通道信息，就与客户端建立连接来生成SocketChannel

6.然后把刚生成的SocketChannel通道再注册到Selector里面，进行监听read、wirte事件

7.read、wirte事件准备就绪了就来处理，抛给任务队列。

注意：有2个Selector，一个是boss来进行监听准备就绪事件，另一个woker是当准备就绪后要做什么工作，把这个工作抛给woker慢慢来做。

## 4.3、dubbo原理

### 4.3.1、dubbo原理	-   框架设计

![Untitled%2026.png](Untitled%2026.png)

**各层说明**

- **config 配置层**：对外配置接口，以 `ServiceConfig`, `ReferenceConfig` 为中心，可以直接初始化配置类，也可以通过 spring 解析**配置生成配置类**
- **proxy 服务代理层**：服务接口透明代理，**生成服务的客户端 Stub 和服务器端 Skeleton,** 以 `ServiceProxy` 为中心，扩展接口为 `ProxyFactory`
- **registry 注册中心层**：封装服务地址的**注册与发现**，以服务 URL 为中心，扩展接口为 `RegistryFactory`, `Registry`, `RegistryService`
- **cluster 路由层**：封装多个提供者的路由及**负载均衡**，并桥接注册中心，以 `Invoker` 为中心，扩展接口为 `Cluster`, `Directory`, `Router`, `LoadBalance`
- **monitor 监控层**：RPC **调用次数和调用时间监控**，以 `Statistics` 为中心，扩展接口为 `MonitorFactory`, `Monitor`, `MonitorService`
- **protocol 远程调用层**：**封装 RPC 调用**，以 `Invocation`, `Result` 为中心，扩展接口为 `Protocol`, `Invoker`, `Exporter`
- **exchange 信息交换层**：封装请求响应模式，同步转异步，以 `Request`, `Response` 为中心，扩展接口为 `Exchanger`, `ExchangeChannel`, `ExchangeClient`, `ExchangeServer`，创建客户端和服务端，两个架起管道进行数据互联互通。
- **transport 网络传输层**：抽象 mina 和 **netty (transport 底层)**为统一接口，以 `Message` 为中心，扩展接口为 `Channel`, `Transporter`, `Client`, `Server`, `Codec`
- **serialize 数据序列化层**：可复用的一些工具，扩展接口为 `Serialization`, `ObjectInput`, `ObjectOutput`, `ThreadPool`

**说明：**Consumer在左边，Provider在右边。Call为调用逻辑，黑颜色是依赖顺序。

### 4.3.2、dubbo原理	-   启动解析、加载配置信息

BeanDefinitionParser定义解析器

DubboBeanDefinitionParser中的parse方法来解析标签。挨个标签来解析。

DubboBeanDefinitionParser构造器打断点看怎么创建出来的。

![Untitled%2027.png](Untitled%2027.png)

init方法注册很多标签解析器。容器启动，解析每一个标签，每一个标签都有对应的***Config.class，保存到指定的对象中,Service标签牵扯到服务暴漏的功能。

![Untitled%2028.png](Untitled%2028.png)

### 4.3.3、dubbo原理	-   服务暴露

![dubbo-%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2.jpg](dubbo-%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2.jpg)

Service标签牵扯到服务暴漏的功能，ServiceBean它实现了InitalizingBean（它在创建完对象以后调用afterPropertisSet方法）、ApplicationListener(事件是ContextRefreshedEvent(当IOC容器完成后会回调onApplicationEvent方法))

afterPropertisSet方法和onApplicationEvent方法都做了什么：

afterPropertisSet方法保存配置的属性信息。

onApplicationEvent方法判断当没有爆露时调用export方法暴漏服务，调用doExport方法。调用doExportUrls方法，第一步加载注册信息，再调用doExportUrlsFor1Portocol方法，创建invoker，调用protocol(DubboProtocol).export(wrapperInvoker)

暴漏URL,OPENServer(url)，启动netty服务器，监听20880端口

创建服务器createServer(url)

Exchangers.bind(url,requestHandler)绑定url和处理器

Transporters.bind(url,new DecodeHandler(new HanderExchangesHandler()))这都是netty的底层，创建一个netty的服务器。

注册提供者。

总共2步：

启动netty服务器，监听20880端口。

注册中心进行注册服务，把注册好的地址，把注册信息保存在注册中心。

microkernel plugin模式

### 4.3.4、dubbo原理	-   服务引用

如何通过配置reference标签远程引用暴漏的服务。

![Untitled](Untitled%2029.png)

前置类似暴漏服务原理，对应的是ReferenceBean,特殊在implement FactoryBean，FactoryBean作用是当我们要获取userService，在引用userService的类中需要自动注入userService，需要从spring容器中找，通过调用FactoryBean的getObject方法获取userService对象，getObject调用ReferenceConfig的getT方法，如果ref引用是空的，就init（）初始化对象，ref = createProxy（map）。

![Untitled](Untitled%2030.png)

![Untitled](Untitled%2031.png)

然后远程引用接口，urls保存了注册中心的地址。

![Untitled](Untitled%2032.png)

refprotocal还是Protocal，Protocal有2种，1个是DubboProtocal，另一个是RegistryProtocol，这里调用RegistryProtocol的refer方法。

![Untitled](Untitled%2033.png)

先根据注册中心地址得到注册中心信息。然后调用doRefer,传入了注册中心地址，还有要引用的userService

![Untitled](Untitled%2034.png)

之后进行订阅服务

![Untitled](Untitled%2035.png)

订阅服务会进入到DubboProtocol，DubboProtocol会getClients获取客户端。

![Untitled](Untitled%2036.png)

拿到客户端，初始化客户端

![Untitled](Untitled%2037.png)

然偶会进行连接

![Untitled](Untitled%2038.png)

获取返回连接，调用Transporter.connect（），之后就到达了netty的底层。

![Untitled](Untitled%2039.png)

![Untitled](Untitled%2040.png)

![Untitled](Untitled%2041.png)

相当于创建一个netty客户端，根据url地址监听一个端口。创建完连接后返回DubboProtocol中，创建好invoker，返回invoker。

![Untitled](Untitled%2042.png)

获取创建好的invoker，里面有url地址，消费者地址。之后在消费者注册表里把刚创建的invoker注册进去。注册进入subscribeUrl订阅地址。记录消费者消费哪个服务。

![Untitled](Untitled%2043.png)

保存好了后返回到doRefer()

![Untitled](Untitled%2044.png)

但后一直返回到createProxy。ref代理对象就创建好了，远程信息等都在里面存好了。

之后使用代理对象来执行远程调用方法。

![Untitled](Untitled%2045.png)

### 4.3.5、dubbo原理	-   服务调用

整体流程：

![Untitled](Untitled%2046.png)

代理对象如何执行方法呢，在这里打个断点。

![Untitled](Untitled%2047.png)

会调用invoke，并且会传入method和args并封装成一个叫RpcInvocation远程调用的对象。

![Untitled](Untitled%2048.png)

之后再进一个Invoker，它里面封装了FailOverCluster，他是dubbo带集群容错功能的invoker

![Untitled](Untitled%2049.png)

进去后查询有多少个可执行版本的方法，再获取负载均衡的机制，之后继续doInvoke

![Untitled](Untitled%2050.png)

之后有个select能根据我们的负载均衡策略进行负载随机选择1个invoker

![Untitled](Untitled%2051.png)

接下来调用invoker.invoke执行，到了真正要执行时还封装了一些filter，这个filter在最开始proxy代理对象要用invoker时会在外面封装filter来做（local、mock、cache）

[https://dubbo.apache.org/zh/docsv2.7/user/examples/local-mock/](https://dubbo.apache.org/zh/docsv2.7/user/examples/local-mock/)

[https://dubbo.apache.org/zh/docsv2.7/user/examples/result-cache/](https://dubbo.apache.org/zh/docsv2.7/user/examples/result-cache/)

缓存功能时，然后invoker选中了负载均衡功能后，又进入filter，filter就是各个统计信息（监控中心等）ProtocolFilter，ProtocolFilter里封装到最终时DubboInvoker.

![Untitled](Untitled%2052.png)

![Untitled](Untitled%2053.png)

![Untitled](Untitled%2054.png)

在DubboInvoker中获取到在服务引用时建立的客户端，拿到引用的哪个远程服务，之后再request发起请求，返回结果。这里request就已经进入到底层了。之后拿客户端，拿通道。

![Untitled](Untitled%2055.png)

![Untitled](Untitled%2056.png)

![Untitled](Untitled%2057.png)

然后给通道发送出去。

![Untitled](Untitled%2058.png)

之后返回拿到返回结果对象，里面封装了结果数据

![Untitled](Untitled%2059.png)

返回返回， 之后打印到控制台。

![Untitled](Untitled%2060.png)

# 5.结束语

祝大家学业有成