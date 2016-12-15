# 一、从以下位置下载对应版本的elasticsearch #
https://www.elastic.co/downloads/elasticsearch


# 二、elasticsearch-5.1.1.zip解压 #

将当前目录下的elasticsearch-5.1.1.zip解压到本地磁盘某个目录下.如E:\green\elasticsearch-5.1.1。

# 三、设置环境变量 #

修改环境变量

    Path = Path + E:\green\elasticsearch-5.1.1\bin;

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

# 七、概念 #
在Elasticsearch中存储数据的行为就叫做索引(indexing)，不过在索引之前，我们需要明确数据应该存储在哪里。在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库：

    Relational DB -> Databases -> Tables -> Rows -> Columns
    Elasticsearch -> Indices   -> Types  -> Documents -> Fields

Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。

**倒排索引** 

传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。Elasticsearch和Lucene使用一种叫做倒排索引(inverted index)的数据结构来达到相同目的。
默认情况下，文档中的所有字段都会被索引（拥有一个倒排索引），只有这样他们才是可被搜索的。

