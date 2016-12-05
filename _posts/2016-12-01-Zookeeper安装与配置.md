# 伪集群模式 #
所谓伪集群, 是指在单台机器中启动多个zookeeper进程, 并组成一个集群. 以启动3个zookeeper进程为例.
下载zookeeper-3.4.9.tar.gz安装包之后, 解压到合适目录，如E:\green\zookeeper349_0。将zookeeper的目录拷贝2份:
目录结构如下：  

    |-- zookeeper349_0  
    |-- zookeeper349_1  
    |-- zookeeper349_2  

进入zookeeper目录下的conf子目录, 创建zoo.cfg:
更改zookeeper0/conf/zoo.cfg文件为:

    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/green/zookeeper349_0/data 
    dataLogDir=/green/zookeeper349_0/logs 
    clientPort=2081  
    server.0=127.0.0.1:8880:7770
    server.1=127.0.0.1:8881:7771
    server.2=127.0.0.1:8882:7772  

参数说明:

- tickTime: zookeeper中使用的基本时间单位, 毫秒值.
- dataDir: 数据目录. 可以是任意目录.
- dataLogDir: log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和dataDir相同的设置.
- clientPort: 监听client连接的端口号.
- initLimit: zookeeper集群中的包含多台server, 其中一台为leader, 集群中其余的server为follower. initLimit参数配置初始化连接时, follower和leader之间的最长心跳时间. 此时该参数设置为10, 说明时间限制为10倍tickTime, 即10*2000=20000ms=20s.
- syncLimit: 该参数配置leader和follower之间发送消息, 请求和应答的最大时间长度. 此时该参数设置为5, 说明时间限制为5倍tickTime, 即10000ms.
- server.X=A:B:C 其中X是一个数字, 表示这是第几号server. A是该server所在的IP地址. B配置该server和集群中的leader交换消息所使用的端口. C配置选举leader时所使用的端口. 由于配置的是伪集群模式, 所以各个server的B, C参数必须不同.

参照zookeeper0/conf/zoo.cfg, 配置zookeeper1/conf/zoo.cfg, 和zookeeper2/conf/zoo.cfg文件. 只需更改dataDir, dataLogDir, clientPort参数即可.
更改zookeeper1/conf/zoo.cfg文件为:
 
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/green/zookeeper349_1/data 
    dataLogDir=/green/zookeeper349_1/logs 
    clientPort=2181  
    server.0=127.0.0.1:8880:7770
    server.1=127.0.0.1:8881:7771
    server.2=127.0.0.1:8882:7772  

更改zookeeper2/conf/zoo.cfg文件为:
  
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/green/zookeeper349_2/data 
    dataLogDir=/green/zookeeper349_2/logs 
    clientPort=2281  
    server.0=127.0.0.1:8880:7770
    server.1=127.0.0.1:8881:7771
    server.2=127.0.0.1:8882:7772  

在之前设置的dataDir中新建myid文件, 写入一个数字, 该数字表示这是第几号server. 该数字必须和zoo.cfg文件中的server.X中的X一一对应.

    /green/zookeeper349_0/data/myid文件中写入0, 
    /green/zookeeper349_1/data/myid文件中写入1, 
    /green/zookeeper349_2/data/myid文件中写入2.

分别进入/green/zookeeper349_r0/bin, /green/zookeeper349_1/bin, /green/zookeeper349_r2/bin三个目录, 启动server.
至此, zookeeper的伪集群模式已经配置好了. 启动server只需运行脚本:
 
    bin/zkServer.sh start

 Server启动之后, 就可以启动client连接server了, 任意选择一个server目录, 启动客户端:
执行脚本:
  
    bin/zkCli.sh -server localhost:2081

# 集群模式 #
集群模式的配置和伪集群基本一致.
由于集群模式下, 各server部署在不同的机器上, 因此各server的conf/zoo.cfg文件可以完全一样.
下面是一个示例:

    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/home/zookeeper/data
    dataLogDir=/home/zookeeper/logs
    clientPort=4180  
    server.43=10.1.39.43:2888:3888  
    server.47=10.1.39.47:2888:3888
    server.48=10.1.39.48:2888:3888  

示例中部署了3台zookeeper server, 分别部署在10.1.39.43, 10.1.39.47, 10.1.39.48上. 需要注意的是, 各server的dataDir目录下的myid文件中的数字必须不同.

    10.1.39.43 server的myid为43, 
    10.1.39.47 server的myid为47, 
    10.1.39.48 server的myid为48.
