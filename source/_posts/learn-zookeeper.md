---
title: Zookeeper学习笔记
date: 2020-11-13 10:00:41
comments: true
tags: 
    - zookeeper
categories: 中间件
thumbnail: Untitled.png
banner: Untitled.png
cover: Untitled.png
---


# 第一章 Zookeeper入门-了解

## 1.1 概述

开源的分布式的，为分布式应用提供**协调服务**的Apache项目

![Untitled.png](Untitled.png)

## 1.2 特点

![Untitled%201.png](Untitled%201.png)

## 1.3 数据结构

![Untitled%202.png](Untitled%202.png)

## 1.4 应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集权管理、服务器节点动态上下线、软负载均衡等。

统一命名服务：

![Untitled%203.png](Untitled%203.png)

统一配置管理：

![Untitled%204.png](Untitled%204.png)

统一集权管理：

![Untitled%205.png](Untitled%205.png)

服务器节点动态上下线：

![Untitled%206.png](Untitled%206.png)

4.4中会实现该案例.

软负载均衡：

![Untitled%207.png](Untitled%207.png)

## 1.5 下载地址

[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)

# 第二章 Zookeeper安装

## 2.1 本地模式安装部署

### 2.1.1 安装jdk

上传jdk包到/opt/software

tar -zxvf jdk-8u291-linux-x64.tar.gz

```jsx
export JAVA_HOME=/opt/software/jdk1.8.0_291/  #jdk安装目录
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

### 2.1.2安装zookeeper

拷贝到/opt/software

解压

```jsx
tar -zxvf apache-zookeeper-3.7.0.tar.gz
```

### 2.1.3修改zookeeper配置

将"/opt/software/apache-zookeeper-3.7.0/conf/zoo_sample.cfg"拷贝份zoo.cfg

打开zoo.cfg，修改vim zoo.cfg

```jsx
dataDir=/opt/software/apache-zookeeper-3.7.0/zkData/
```

### 2.1.4操作zookeeper

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
./zkServer.sh stop
```

```jsx
netstat -apn | grep 2181
```

## 2.2 配置参数解读

```docker
# 心跳ms
tickTime=2000
# 10个2s 启动时，leader和felower之间最大延迟时间，超过挂了
initLimit=10
# 5个2s 集群启动之后，leader和felower之间最大延迟时间，超过挂了
syncLimit=5
# 客户端访问服务器的端口号
clientPort=2181
```

# 第三章 Zookeeper内部原理

## 3.1 选举机制（面试重点）

1) 半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。

2) Zookeeper在配置文件中没有配置Leader和Follower，但是Zookeeper在工作时，是有一个Leader的，其他的则为Follower，Leader通过内部的选举机制选举产生的。

3) 以一个简单的例子说明选举过程

假设有5台服务器组成的Zookeeper集群，它们的id从1-5，同时他们都是最新启动的。也就没有历史数据。在存放数据量上都是一样的。假设服务器依次启动，会发生什么：

![Untitled%208.png](Untitled%208.png)

(1) 服务器1启动，此时只有它一个服务器启动了，它发出去的报文没有任何响应，所以它选举状态一直是LOOKING状态。

(2) 服务器2启动，它也投自己，它与最开始启动的服务器1进行通信，互相交换选举结果，由于两者都没有历史数据。所以id值较大的2号胜出，但由于没有达到半数以上。

(3) 服务器3启动，它投自己，前边的服务器1、2投自己后发现选不出leader，都选择id大的3号服务器。

(4) 服务器4启动，它投自己，但是3号服务器已经担任leader责任。4号服务器当不了了。

(5) 服务器5启动，只要leader选出来之后。改变不了leader了。

## 3.2 节点类型

### 2大类

持久：客户端和服务端断开连接后，创建的目录节点不删除

短暂：客户端和服务端断开连接后，创建的目录节点自己删除

![Untitled%209.png](Untitled%209.png)

### 持久分类：

(1) 持久化目录节点

客户端与Zookeeper断开连接后，该节点依旧存在

(2) 持久化顺序编号目录节点

客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号

**说明：**

创建znode时设置顺序编号，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护

**注意：**

在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

### 短暂分类：

(1) 临时目录节点

客户端和Zookeeper断开连接后，创建的目录节点被立刻删除，适合动态上下线

(2) 临时顺序编号目录节点

客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行了顺序编号

![Untitled%2010.png](Untitled%2010.png)

## 3.3 Stat结构体

```docker
[zk: localhost:2181(CONNECTED) 15] get -s /sanguo
jinlian
cZxid = 0xd
# 创建节点的事务id，每次修改节点状态都会收到一个zxid形势的时间戳，也就是zookeeper事务id。
# 事务id是zookeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，
# 那么zxid1在zxid2之前发生

ctime = Wed Jun 16 13:44:06 CST 2021
# znode被创建的毫秒数(从1970年开始)

mZxid = 0xe
#最后更新的事务zxid

mtime = Wed Jun 16 13:44:19 CST 2021
# znode最后被修改的毫秒数(从1970年开始)

pZxid = 0xf
# 最后更新的子节点zxid

cversion = 1
# znode子节点变化号，znode子节点修改次数

dataVersion = 1
# znode数据变化号

aclVersion = 0
# znode访问控制列表的变化

ephemeralOwner = 0x0
# 如果是临时节点，这个znode拥有者的session的id。如果不是临时节点则是0

dataLength = 7
# znode的数据长度

numChildren = 1
# znode子节点数量
```

## 3.4 监听原理（面试重点）

### 3.4.1监听原理详解：

1) 首先有一个main线程

2) 在main线程中创建zookeeper客户端，这时创建两个线程，一个负责网络通信(connect)，一个负责监听(listener)

3) 通过connect将注册的监听事件发送给zookeeper

4) 在zookeeper的zhu'c监听器列表中将注册的监听事件添加到列表中

5) zookeeper监听到有数据或路劲的变化后，就会将这条消息发送给listener线程

6) listener线程内部钓用process方法(在程序中自己写)

![Untitled%2011.png](Untitled%2011.png)

### 3.4.2常见的监听

1) 监听节点数据的变化

get path [watch]

2) 监听子节点增减的变化

ls path [watch]

## 3.5 写数据流程

### 问题

客户端提交数据，是怎么写的？

怎么保持副本中都一样？

leader干什么事、server干什么事？

### 步骤

![Untitled%2012.png](Untitled%2012.png)

# 第四章 Zookeeper实战（开发重点）

## 4.1 分布式安装部署

### 4.1.1集群规划

在服务器102、服务器103、服务器104三个节点上部署zookeeper

### 4.1.2解压安装

(1) 解压Zookeeper安装包到/opt/modult/目录下

```docker
tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/
```

(2) 同步/opt/module/zookeeper-3.4.10目录内容到服务器103、服务器104

```docker
xsync zookeeper-3.4.10
```

### 4.1.3 配置服务器编号

(1) 在/opt/module/zookeeper-3.4.10/下创建zkData

```docker
mkdir -p zkData
```

(2) 在/opt/module/zookeeper-3.4.10/zkData/目录下创建一个myid文件

```docker
touch myid
```

(3) 编辑myid文件

```docker
vim myid
#添加与server对应编号
2
```

(4) 拷贝配置好的zookeeper到其他服务器上

```docker
xsync myid
```

分别在103、104服务器上修改myid文件内容为3、4

### 4.1.4 配置zoo.cfg文件

(1) 添加如下配置

```docker
###############cluster###################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888 
```

(2) 同步zoo.cfg

```docker
xsync zoo.cfg
```

(3) 配置解读

```docker
server.A=B:C:D
```

A是一个数字，myid些什么A就写什么。

集群模式下配置了一个myid文件，zookeeper启动时读取此文件，拿到数据后与zoo.cfg中的配置信息进行对比从而确定是哪个server

B是这个服务器的ip地址

C是这个服务器与集群中的Leader服务器与Follower交换信息的端口

D是万一集群中的Leader服务器挂了，需要一个端口来进行重新选举，选出一个新的Leader

### 4.1.5 集群操作

(1) 分别启动zookeeper

(2) 查看状态

## 4.2 客户端命令行操作

[shell](https://www.notion.so/3c402338899946a68de5eea00d2ea386)

## 4.3 API应用

### 4.3.1 环境搭建

1.创建工程

2.pom文件添加依赖

```docker
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.1</version>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.9</version>
</dependency>
```

3.resources目录创建log4j.properties

```docker
### 设置###
log4j.rootLogger = info,stdout

### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n
```

### 4.3.2 创建Zookeeper客户端

```docker
@Before
public void init() throws IOException {
    String connectString = "39.106.63.189:2181";
    int sessionTimeout = 200000;
    ZooKeeper zkCli = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
        //监听回调
        @Override
        public void process(WatchedEvent watchedEvent) {
        }
    });
}
```

### 4.3.3 创建子节点

```docker
@Test
public void createNode() throws KeeperException, InterruptedException {
    //参数1：节点路径
    //参数2：节点数据
    //参数3：节点权限
    //参数4：节点类型
    String path = zkCli.create("/guoyh", "guoyh".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    System.out.println(path);
}
```

### 4.3.4 获取子节点并监听节点变化

```docker
@Test
public void getDataAndWatch() throws KeeperException, InterruptedException {
    //参数1：节点路径
    //参数2：是否监听
    List<String> datas = zkCli.getChildren("/",true);
    for (String data : datas) {
        System.out.println(data);
    }

    //睡一会，当有节点变化时响应,在process中处理
    Thread.sleep(Long.MAX_VALUE);
}

@Before
public void init() throws IOException {
    String connectString = "39.106.63.189:2181";
    int sessionTimeout = 200000;
    ZooKeeper zkCli = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
        //监听回调
        @Override
        public void process(WatchedEvent watchedEvent) {
            //监听节点变化
            List<String> datas = null;
            try {
                datas = zkCli.getChildren("/", true);
                for (String data : datas) {
                    System.out.println(data);
                }
            } catch (KeeperException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
}
```

### 4.3.5 判断Znode是否存在

```docker
@Test
public void exist() throws KeeperException, InterruptedException {
    //参数1：节点路径
    //参数2：是否监听
    Stat stat = zkCli.exists("/guoyha", false);
    System.out.println(stat == null ? "not exist":"exist");
}
```

## 4.4 监听服务器节点动态上下线案例

### 4.4.1 需求

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知主节点服务器的上下线。

### 4.4.2 需求分析

![Untitled%2013.png](Untitled%2013.png)

相对于在Zookeeper集群，3个服务器和3个客户端都是连接zookeeper的客户端，3个服务器在zookeeper的/servers目录下创建对应的节点并存储连接数。3个客户端连接zookeeper集群并监听/servers目录下节点列表，当有服务器下线后，能够监听到节点列表的变化。

### 4.4.3 代码实现

**服务端：**

```docker
package com.example.zookeeper.server;

import org.apache.zookeeper.*;

import java.io.IOException;

/**
 * @author guoyh
 */
public class DistributeServer {
    private ZooKeeper zkCli;
    String connectString = "127.0.0.1:2181";
    int sessionTimeout = 200000;

    public static void main(String[] args) {
        DistributeServer server = new DistributeServer();

        // 1、连接Zookeeper集群
        server.getConnect();

        // 2、注册创建临时节点
        // 设置服务器名称
        String application = "hadoop101";
        server.regist(application);

        // 3、自己的业务逻辑代码处理
        server.business();
    }

    /**
     * 服务自己的业务处理
     */
    private void business() {
        try {
            //保证进程不结束
            Thread.sleep(Long.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 注册服务
     * @param hostname
     */
    private void regist(String hostname) {
        try {
            //创建短暂带序号的子节点,存放主机名
            String path = zkCli.create("/servers/server", hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
            System.out.println(hostname + "is online");
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取Zookeeper连接
     */
    private void getConnect() {
        try {
            zkCli = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
                //监听回调
                @Override
                public void process(WatchedEvent watchedEvent) {

                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**客户端：**

```docker
package com.example.zookeeper.client;

import org.apache.zookeeper.*;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author guoyh
 */
public class DistributeClient {
    private ZooKeeper zkCli;
    String connectString = "0.0.0.0:2181";
    int sessionTimeout = 200000;

    public static void main(String[] args) {
        DistributeClient client = new DistributeClient();

        // 1、获取zookeeper连接
        client.getConnect();

        // 2、注册监听
        String application = "hadoop101";
        client.getChildren();

        // 3、业务逻辑处理
    }

    /**
     * 注册服务
     */
    private void getChildren() {
        try {
            List<String> children = zkCli.getChildren("/servers/server", true);
            // 存储服务器节点主机名称集合
            List<String> hostnames = new ArrayList<String>();

            System.out.println("------start------");
            for (String child : children) {
                byte[] data = zkCli.getData("/servers/" + child, false, null);
                hostnames.add(new String(data));
            }
            // 将所有在线主机名打印
            System.out.println(hostnames);
            System.out.println("------end------");
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取Zookeeper连接
     */
    private void getConnect() {
        try {
            zkCli = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
                //监听回调
                @Override
                public void process(WatchedEvent watchedEvent) {

                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

# 第五章 企业面试真题（面试重点）

## 5.1 简述Zookeeper的选择机制

详见3.1

半数机制

## 5.2  Zookeeper的监听原理是什么

详见3.4

## 5.3  Zookeeper的部署方式有哪几种？

(1) 单机模式、集群模式

(2) 角色：Leader和Follower

(3) 集群最少需要机器数3个

## 5.4 Zookeeper的常见命令

ls、create、get、delete、set