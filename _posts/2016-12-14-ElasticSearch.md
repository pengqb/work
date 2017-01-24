# 一、从以下位置下载对应版本的elasticsearch #
https://www.elastic.co/downloads/elasticsearch


# 二、elasticsearch-5.1.1.zip解压 #

将当前目录下的elasticsearch-5.1.1.zip解压到本地磁盘某个目录下.如伪集群E:\green\elasticsearch-5.1.1。

# 三、设置环境变量 #

## 修改环境变量 ##

    Path = Path + E:\green\elasticsearch-5.1.1\bin;

## 修改配置,让外网可以访问 ##
E:\green\elasticsearch-5.1.1\config\elasticsearch.yml

    network.host: 192.168.1.106

# 四、启动服务器 #
在控制台执行 bin/elasticsearch (or bin\elasticsearch.bat on Windows)

    C:\Users\pengqb>elasticsearch

# 五、验证服务器启动成功 #

    E:\green\elasticsearch-5.1.1>curl http://localhost:9200/
    {
      "name" : "fwkZChW",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "L09iCqKQTN6zE36eW3Uw7g",
      "version" : {
        "number" : "5.1.1",
        "build_hash" : "5395e21",
        "build_date" : "2016-12-06T12:36:15.409Z",
        "build_snapshot" : false,
        "lucene_version" : "6.3.0"
      },
      "tagline" : "You Know, for Search"
    }

# 六、集群配置 #
节点(node)是一个运行着的Elasticsearch实例。集群(cluster)是一组具有相同cluster.name的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。
你最好找一个合适的名字来替代cluster.name的默认值，比如你自己的名字，这样可以防止一个新启动的节点加入到相同网络中的另一个同名的集群中。
你可以通过修改config/目录下的elasticsearch.yml文件，然后重启ELasticsearch来做到这一点。当Elasticsearch在前台运行，可以使用Ctrl-C快捷键终止。

如果在同一台机器启动伪集群，只需要拷贝第一个节点。

    cp E:\green\elasticsearch-5.1.1 E:\green\elasticsearch2-5.1.1

启动第二个节点
    E:\green\elasticsearch2-5.1.1\bin\elasticsearch

# 七、概念 #
在Elasticsearch中存储数据的行为就叫做索引(indexing)，不过在索引之前，我们需要明确数据应该存储在哪里。在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库：

    Relational DB -> Databases -> Tables -> Rows -> Columns
    Elasticsearch -> Indices   -> Types  -> Documents -> Fields

Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。

**倒排索引** 

传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。Elasticsearch和Lucene使用一种叫做倒排索引(inverted index)的数据结构来达到相同目的。
默认情况下，文档中的所有字段都会被索引（拥有一个倒排索引），只有这样他们才是可被搜索的。


# 八、Java API #
Elasticsearch为Java用户提供了两种内置客户端：
## 节点客户端(node client)： ##
节点客户端以无数据节点(none data node)身份加入集群，换言之，它自己不存储任何数据，但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上。
## 传输客户端(Transport client ##
这个更轻量的传输客户端能够发送请求到远程集群。它自己不加入集群，只是简单转发请求给集群中的节点。

# 九、数据交互格式 #
基于HTTP协议，以JSON为数据交互格式的RESTful API
向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：

- curl -X\<VERB\> '\<PROTOCOL\>://\<HOST\>/\<PATH\>?\<QUERY_STRING\>' -d '\<BODY\>'
- VERB HTTP方法： GET , POST , PUT , HEAD , DELETE
- PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
- HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
- PORT Elasticsearch HTTP服务所在的端口，默认为9200
- QUERY_STRING 一些可选的查询请求参数，例如 ?pretty 参数将使请求返回更加美观易读的JSON数据
- BODY 一个JSON格式的请求主体（如果请求需要的话）

举例说明，为了计算集群中的文档数量，我们可以这样做：

    curl -XGET 'http://localhost:9200/_count?pretty' -d '
    {
    	"query": {
    		"match_all": {}
    	}
    }'

我们看不到HTTP头是因为我们没有让 curl 显示它们，如果要显示，使用 curl 命令后跟 -i 参数:

    curl -i -XGET 'localhost:9200/'


# 十、性能 #
    http rest for 100 took:8398801411
    sync  transport for 10000 took:8390398718
    async transport for 10000 took:1561183912
    bulk  transport for 10000 took:565804559
    bulk  transport for 10000 took:475822746
    
    sync  transport for 100000 took:55923396897
    async transport for 100000 took:10566496908
    bulk  transport for 100000 took:1788979040
    bulk  transport for 100000 took:1577938836

# 命令集 #

    curl -XGET http://localhost:9200/iot/gw/1?pretty
    
    curl -XGET http://localhost:9200/twitter/_mapping
    
    //在任意的查询字符串中增加 pretty 参数，类似于下面的例子。会让Elasticsearch美化输出(pretty-print)JSON响应以
    //便更加容易阅读。 _source 字段不会被美化，它的样子与我们输入的一致。
    curl -i -XGET 'http://localhost:9200/_count?pretty' -d '
    {
    	"query": {
    		"match_all": {}
    	}
    }'
    
    curl http://192.168.1.106:9200/_cluster/health
    
    curl -XGET http://192.168.1.106:9200/blogs
    
    curl -XDELETE http://192.168.1.106:9200/blogs
    
    curl -XPUT 'http://192.168.1.106:9200/blogs' -d '
    {
    	"settings" : {
    		"number_of_shards" : 3,
    		"number_of_replicas" : 1
    	}
    }'