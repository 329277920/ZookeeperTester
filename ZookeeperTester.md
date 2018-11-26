# Zookeeper #

参考:<br>
[https://zookeeper.apache.org/doc/current/index.html](https://zookeeper.apache.org/doc/current/index.html)<br>
[https://github.com/apache/zookeeper](https://github.com/apache/zookeeper)<br>
[https://www.cnblogs.com/yjmyzz/tag/zookeeper/](https://www.cnblogs.com/yjmyzz/tag/zookeeper/)<br>
[https://www.cnblogs.com/leesf456/p/6239578.html](https://www.cnblogs.com/leesf456/p/6239578.html)

>目录<br>
>[一、概述](1#)<br>
>[二、安装](2#)<br>
>[三、开发指南](3#)<br>
>[四、客户端api](4#)<br>
>[五、.Net集成](5#)<br>
>[附录](100#)<br>
>&nbsp;&nbsp;[Maven](100.1)

<h2 id="1#">一、概述</h2>

*定义*

分布式协调服务，提供了配置中心、服务注册中心、分布式同步等相关服务。它本质是一个基于树的文件系统，每个节点称为 **znode** 与传统文件系统不同的是，它的数据常驻内存，以实现比较高的吞吐量，同时它也支持集群部署，以支持高可用。如下图:

![](https://i.imgur.com/QnbJ4yz.jpg)

Zookeeper的数据存储在多个服务器实例中，自身会做数据同步。同一时间，多个服务器节点会选举出一个Leader节点，提供数据的写入操作，并负责将操作日志同步到其他的Follower节点。客户端可以连接到任意一个服务器节点，以访问提供的服务。<br>

*数据结构*

znode是Zookeeper中存储数据的节点，对节点的操作都是原子性的，节点路径是以'/'分隔的命名空间，每个znode都有属于自己的数据和子节点，如下图:<br>

![](https://i.imgur.com/iufJf01.jpg)

znode分为普通节点和临时节点，普通节点只有在显式删除时，才会失效，临时节点当创建该节点的回话关闭后，会自动删除。同时Zookeeper也提供监控机制，允许客户端注册监控事件，当znode发生变更（更新、增加节点、删除节点）时会触发客户端事件。<br>

*注册中心*

利用Zookeeper的临时节点很容易完成服务注册功能，某个业务服务在启动时，会向Zookeeper写入一个普通节点，记录服务名称和一个临时节点，记录服务服务IP的端口。服务消费者通过服务名称路径获取服务节点列表（IP、端口）。

*配置中心*

增加普通节点即可。

*分布式锁*

Zookeeper为新增加的子节点设置编号，从小到大，比如A、B两个服务同时访问资源R，这时，A和B同时向Zookeeper的某个节点下新增一个临时（防止该服务宕机后，节点未删除）子节点，并获取到自身添加的子节点的排序号是不是最小，如果是，则认为获取到了锁，并在执行完资源访问后，删除该节点。如果获取不到，可以采取循环等待，或者向Zookeeper注册一个监控事件来等待获取锁。

<h2 id="2#">二、安装</h2>

*系统依赖*

Zookeeper包含多个组件。<br>
>Client 是一个java的客户端库，用于连接Zookeeper服务端。<br>
>Server 运行Zookeeper服务器节点，基于java。<br>
>Native Client 是一个c语言写的客户端。<br>
>Contrib 其他组件。<br>

![](https://i.imgur.com/jXz1qxv.png)

*软件依赖*

Zookeeper运行在java平台，依赖的jdk版本在1.6或以上。官方建议3台或以上节点运行Zookeeper服务器实例。

*下载*

下载地址:[http://mirrors.hust.edu.cn/apache/zookeeper/](http://mirrors.hust.edu.cn/apache/zookeeper/)

*安装实例*

1、环境<br>
>系统:centos7<br>
>jdk:11.0.1<br>
>Zookeeper:3.4.13<br>

2、安装jdk(有安装的跳过)

>下载地址:https://www.ore.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html<br>
>本事例下载的一个rpm包。例如:jdk-11.0.1_linux-x64_bin.rpm<br>
>下载后，cd到下载目录，使用yum安装: 
	
	yum localinstall jdk-11.0.1_linux-x64_bin.rpm
	java -version

3、安装Zookeeper
    
	wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
	tar -zvxf zookeeper-3.4.13.tar.gz 

4、配置

解压后，在"conf"目录下可以看到几个配置文件，其中"zoo_sample.cfg"为zookeeper的配置文件事例，zookeeper默认读取zoo.cfg。参数解释:
>tickTime: 心跳时间，最小会话超时时间（tickTime*2），单位毫秒，最大会话超时时间（tickTime * 20）<br>
>dataDir: 存储内存数据库快照的位置。<br>
>clientPort: 绑定端口<br>

在"conf"目录下创建zoo.cfg，这里使用"zoo_sample.cfg"的配置，端口号为：2181

	cp zoo_sample.cfg zoo.cfg

5、运行

解压后，在"bin"目录下可以看到一些脚本文件。
>zkServer.sh :用于管理服务"start|stop|status"<br>
>zkCli.sh: zk自带得客户端<br>

*开启服务*

	bin/zkServer.sh start #启动服务
    firewall-cmd --permanent --zone=public --add-port=2181/tcp #开放端口
    firewall-cmd --reload

6、安装ui

这里使用 [zkui](https://github.com/DeemOpen/zkui) 最为管理界面，提供了对zookeeper的CRUD操作。运行在java7或以上版本。以及依赖Maven环境。Maven的安装查看[附录](#100.1)。

	git clone https://github.com/DeemOpen/zkui.git  #下载源文件
    mvn clean install
    #上一步成功后，会在target生成jar包。
    cp config.cfg target/config.cfg #复制配置文件
    cd target
    vi config.cfg# 编辑配置，管理界面地址、zookeeper服务端口等。这里使用默认的配置。
    nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar & #执行
    firewall-cmd --permanent --zone=public --add-port=9090/tcp #开放端口
    firewall-cmd --reload

7、replicated（复制）模式

在生产环境下，zookeeper建议运行在replicated模式下，并且至少3台服务器，同时建议为奇数（因leader的选举需要得到集群中超过半数的节点(follower)的认可）。如果只有2台服务器，当其中一台出现故障，剩余的一台不足以发起选举，所以本质上2台服务器还不如单台服务器稳定。<br>
在数据同步的处理，zookeeper使用**Zab协议**保证最终一致性。可以参考"[https://www.jianshu.com/p/2bceacd60b8a](https://www.jianshu.com/p/2bceacd60b8a)"。

每台节点都使用相同的配置文件"zoo.cfg"，例如：

	tickTime=2000
	dataDir=/apps/zookeeper/data
	clientPort=2181
	initLimit=5
	syncLimit=2
	server.1=zoo1:2888:3888
	server.2=zoo2:2888:3888
	server.3=zoo3:2888:3888

*tickTime*: 最小时间单位（毫秒）。<br>
*initLimit*: 初始化连接到leader的超时时间。<br>
*syncLimit*: 接收数据应答的超时时间<br>
*dataDir*: 存放数据（数据操作日志）的目录<br>
*server.n=zoo1:2888:3888*: 解释为: server.[服务器编号,需要与myid文件中的内容一致]=[主机名或IP]:[集群通信端口]:[集群选举端口]。

myid: 需要在 "dataDir" 目录中写入编号。

**部署实例**

**1、环境**<br>

>服务器1: hostname :centos_128(192.168.100.128),集群端口:2888,3888,客户端端口:2181。<br>
>服务器2: hostname :centos_129(192.168.100.129),集群端口:2888,3888,客户端端口:2181。<br>
>服务器3: hostname :centos_130(192.168.100.130),集群端口:2888,3888,客户端端口:2181。<br>

提示：确保各个服务器可以通过主机名访问(因本次实例，使用的是主机名而不是IP，可以在/etc/hosts中配置)，以及端口互通，这里为了测试，直接关闭了防火墙。

**2、安装zookeeper**<br>

参考单节点的安装步骤。

**3、配置**<br>
修改conf/zoo.cfg文件（在单节点安装中有说明，默认没有这个文件，可以从"zoo_sample.cfg"中复制一份）。
	 
	tickTime=2000 # 最小时间单位（毫秒）
	initLimit=10 # 对等节点建立连接N倍最小时间单位	
	syncLimit=5	# 对等节点数据传输N倍最小时间单位	
	dataDir=/tmp/zookeeper # 数据目录和日志目录，也可以单独配置日志目录"dataLogDir"
	clientPort=2181 # 客户端连接端口
	server.128=centos_128:2888:3888 #集群配置，节点1，"myid"为:128
	server.129=centos_129:2888:3888 #集群配置，节点2，"myid"为:129
	server.130=centos_130:2888:3888 #集群配置，节点3，"myid"为:130

分别在数据目录"dataDir"下，建立myid文件，写入内容与配置文件中一致。

    echo 128 > /tmp/zookeeper/myid #128节点
	echo 129 > /tmp/zookeeper/myid #129节点
	echo 130 > /tmp/zookeeper/myid #130节点

	
**4、启动**<br>

*场景一、2台服务器集群*

分别在128、和129节点执行。

	./bin/zkServer.sh start # 启动zookeeper

在128上查看状态:

![](https://i.imgur.com/vmPgYyN.png)

在129上查看状态:

![](https://i.imgur.com/z8zoUky.png)
	
可以看到其中129被选举为"leader"，128为"follower"。在客户端可以连接上任意一台服务器，即可访问zk中的数据。

停止其中的某个节点的zk实例，会发现客户端无法连接上zk集群。所以在建立zk集群时，必须大于2台，建议为奇数。

*场景二、3台服务器集群*

分别在128、129、130 节点执行。

	./bin/zkServer.sh start # 启动zookeeper

使用"status"命令查看节点状态，其中1台服务器被选中为"leader"，其余2台为"follower"。

停止被选中为"leader"的节点实例，在"leader"节点上执行，在测试时,"129"被选中。

	./bin/zkServer.sh stop

再次使用"status"命令查看节点状态，发现进群运行正常，且"130"节点由"follower"转变为"leader"。"129"节点"恢复"后，可以再次使用"start"命令，加入到进群中。


<h2 id="3#">三、开发指南</h2>


<h3 id="3.1">1、数据模型</h3>
zookeeper整体表现为一个树结构，它的每一个节点都可以拥有子节点和数据。节点的路径始终为斜杠分隔的，绝对的路径。

*znode*

>1> 树中的每个节点称之为 **znode**，它维护一个"**stat**"的结构，其中包括数据的版本号和时间戳。客户端可以提供版本号来查询、更新和删除数据。 **state** 数据结构参考: [https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_zkDataModel](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_zkDataModel) <br>
>
>2> 客户端可以为每一个 **znode** 设置监控，当"znode"中的数据或子节点发生变化时，会向客户端发出一个通知，**并清除该监控**。任何一个读操作，包括"getData"、"getChildren"、"exists"，都可以添加一个监控设置。该设置将在 **znode** 的第一次改变并发出通知后清除，客户端需再次调用一个读操作，并重新设置监控事件。更多监控相关参考:[https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_zkWatches](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_zkWatches)<br>
>
>3> 对 znode 中的数据的操作都是原子性的。每个节点都有一个访问控制列表，用于控制当前谁可以操作其中的数据。 zookeeper 不适合存储大型数据对象， **znode** 中存储的数据应远小于1M。<br>
>
>4> 临时节点，允许在创建某个节点的会话结束后，自动删除该节点。临时节点不允许有子节点。

>5> 带序号的节点，允许某个父节点下的所有子节点拥有一个自增的序列号（4字节）。

<h3 id="3.2">2、会话</h3>

zookeeper 提供一个客户端类库，用于建立与服务器的连接。客户端需提供一个连接串，该串包括1个或多个服务实例地址，客户端类库将尝试其中的任意一个，直到建立连接。<br>
当客户端获取一个zookeeper的句柄时，客户端会维护一个64位数字的会话id，在会话超时时间段内，不管客户端连接断开、重连，会话id始终有效。除非该回话超时，会话超时时间需要客户端在建立会话时指定，范围在服务端配置的tickTime的2倍到20倍之间。<br>
当客户端建立一个会话时，状态为"CONNECTING",直到与一个服务器建立连接，会话状态转化为"CONNECTED"，断开连接后，状态转化为"DISCONNECTED"，这个状态下，客户端会重新发起对服务器的连接。如果超过了会话超时时间，还未能建立连接，该状态变为"EXPIRED"。该状态下客户端可以认为该会话已经无效，客户端需要重新建立会话。<br>
zk集群将维护每个客户端的会话，在会话超时时间内，未收到来自客户端的心跳，将此会话置为过期，并删除该回话拥有的所有临时节点，并通知其余会话更新。已过期的会话不会收到过期通知，**直到该会话重新建立连接**。<br>

![](https://i.imgur.com/bn9bMhr.png)

具体参考博客园的[https://www.cnblogs.com/leesf456/p/6103870.html](https://www.cnblogs.com/leesf456/p/6103870.html)

<h3 id="3.3">3、访问控制</h3>

Zookeeper使用ACLs来控制客户端访问ZNode，通俗来说，就是用一个列表来说明某个节点（znode）能被哪些对象（用户名、IP），执行哪些操作（READ|CREATE|WRITE|DELETE|ADMIN）。

zookeeper的每个znode节点都包含一个Acls（访问控制列表），每个acl由三部分组成:验证模式(scheme)、对象id、权限。

*验证模式(scheme)*: 指示对象id的类型，包括下面几个:
>digest：用户名和密码，格式:user:password，digest的密码生成方式是Sha1摘要的base64形式<br>
>auth: 任何用户，客户端访问时需指定用户，但用户名不限制。<br>
>ip: ip地址，或ip段<br>
>world: 公开访问<br>

*对象id*: 按验证模式指定对象id。如模式为digest，id为"user:password"。

*权限*: 

>read: 读取本节点内容 -- 1<br>
>write: 编辑节点内容  -- 2<br> 
>create: 允许创建子节点 -- 4<br>
>delete: 删除子节点 -- 8<br>
>admin: 允许设置本节点的acl -- 16<br>

备注：授权的操作，在.net集成部分介绍。

参考:[https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#sc_ZooKeeperAccessControl](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#sc_ZooKeeperAccessControl)


<h2 id="4#">四、客户端api</h2>

执行安装目录的"bin"目录下的"zkCli.sh"，可以连接上zookeeper服务器，默认为localhost:2181。

	./zkCli.sh

<h3 id="4.1">1、help</h3>

用户显示帮助


![](https://i.imgur.com/L00Ujde.png)


<h3 id="4.2">2、基础命令</h3>

	ls / # 列出根目录下的所有子节点
	create -e /test "This is data" # 创建一个临时节点（永久-p）
    get /test # 查看节点信息
    getAcl /test # 查看节点授权信息
    delete /test # 删除节点
    quit # 退出


<h2 id="5#">五、.net集成</h2>

<h2 id="100">附录</h2>

<h3 id="100.1">1、Maven</h3>

参考地址:[http://maven.apache.org/](http://maven.apache.org/)

*下载*

	wget http://mirrors.shu.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz   

*安装*

	 tar -zvxf apache-maven-3.5.4-bin.tar.gz 
     # 这一步将解压后的bin目录添加到环境变量"/etc/profile"中
     source /etc/profile #加载环境变量
     mvn -v #验证

