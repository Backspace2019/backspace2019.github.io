#                              ------------Elastic Search------------





## 概述ElasticSearch

### 零、全文搜索、倒排索引和Luncene

[全文检索](https://baike.baidu.com/item/全文检索/8028630)是指计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。

lucene，就是一个jar包，里面包含了封装好的各种建立倒排索引，以及进行搜索的代码，包括各种算法。我们就用java开发的时候，引入lucene jar，然后基于lucene的api进行去进行开发就可以了

### 一、为什么用ES而不用MySQL？

1. MySQL来实现搜索，是不太靠谱的。通常来说，性能会很差的。
2. %效率低，全表扫描效率低。
3. 不支持纠错。

### 二、ES是什么？

`Elasticsearch是一个实时分布式搜索和分析引擎。它用于全文搜索、结构化搜索、分析。`Elasticsearch，基于Lucene，隐藏复杂性，提供简单易用的RestfulAPI接口get post put等、JavaAPI接口（还有其他语言的API接口）。

### 三、ES的核心概念

- `Index[索引]`相当于MySQL中的`Database[数据库]`
- `Type[类型]`相当于MySQL中的`Table[表]`,3.6版本以前允许有多个,3.6版本只允许有一个，3.7版本以后取消了
- `Document[文档]`相当于MySQL中的`Row[行]`
- `Field[字段]`相当于MySQL中的`Column[列]`
- `Mapping[映射]`相当于MySQL中的`Schema[约束]`

## 安装 ElasticSearch

### Elasticsearch安装配置集群

1）解压elasticsearch-6.6.0.tar.gz到/opt/module目录下

`[atguigu@hadoop102 software]$ tar -zxvf elasticsearch-6.6.0.tar.gz -C /opt/module/`

------------------------

2）在/opt/module/elasticsearch-6.6.0路径下创建data文件夹

`[atguigu@hadoop102 elasticsearch-6.6.0]$ mkdir data`

------------------------

3）修改配置文件/opt/module/elasticsearch-6.6.0/config/elasticsearch.yml

`[atguigu@hadoop102 config]$ pwd`

​	`/opt/module/elasticsearch-6.6.0/config`

`[atguigu@hadoop102 config]$ vi elasticsearch.yml`

```yml
#-----------------------Cluster-----------------------
cluster.name: my-application
#-----------------------Node-----------------------
node.name: node-102
#-----------------------Paths-----------------------
path.data: /opt/module/elasticsearch-6.6.0/data
path.logs: /opt/module/elasticsearch-6.6.0/logs
#-----------------------Memory-----------------------
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
#-----------------------Network-----------------------
network.host: 192.168.9.102 
#-----------------------Discovery报道中心-----------------------
discovery.zen.ping.unicast.hosts: ["192.168.9.102"]
```

（A）cluster.name

如果要配置集群需要两个节点上的elasticsearch配置的cluster.name相同，都启动可以自动组成集群，这里如果不改cluster.name则默认是cluster.name=my-application，

（B）nodename随意取但是集群内的各节点不能相同

（C）修改后的每行前面不能有空格，修改后的“：”后面必须有一个空格。分发至hadoop103以及hadoop104，分发之后修改：

`[atguigu@hadoop102 module]$ xsync elasticsearch-6.6.0/`

```shell
node.name: node-103
network.host: 192.168.9.103
node.name: node-104
network.host: 192.168.9.104
```

------------------------

4）配置linux系统环境（参考：http://blog.csdn.net/satiling/article/details/59697916）

- 问题1，max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536] elasticsearch

  （1）切换到root用户，编辑limits.conf 添加类似如下内容

  ​	[root@hadoop102s elasticsearch-6.6.0]# vi /etc/security/limits.conf

  ​	添加如下内容:

  ```shell
  * soft nofile 65536
  * hard nofile 131072
  * soft nproc 2048
  * hard nproc 4096
  ```

- 问题2，max number of threads [1024] for user [judy2] likely too low, increase to at least [4096]** （CentOS7.x 不用改）

  （2）切换到root用户，进入limits.d目录下修改配置文件。

  ​	[root@hadoop102 elasticsearch-6.6.0]# vi /etc/security/limits.d/90-nproc.conf

  ​	修改如下内容：

  ```shell
  * soft nproc 1024
  #修改为
  * soft nproc 4096
  ```

- 问题3，max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]** （CentOS7.x 不用改）

  ​	（3）切换到root用户修改配置sysctl.conf

  ​		`[root@hadoop102 elasticsearch-6.6.0]# vi /etc/sysctl.conf` 

  ​		添加下面配置：

  ​		`vm.max_map_count=655360`

  ​		并执行命令：

  ​		`[root@hadoop102 elasticsearch-6.6.0]# sysctl -p`

**以上修改的Linux配置需要分发至其他节点。然后，重新启动Linux，必须重启！！！**

------------------------

5）启动Elasticsearch

`[atguigu@hadoop102 elasticsearch-6.6.0]$ bin/elasticsearch`

------------------------

6）测试elasticsearch

`[atguigu@hadoop102 elasticsearch-6.6.0]$ curl http://hadoop102:9200`

```json
{
 "name" : "node-102",
 "cluster_name" : "my-application",
 "cluster_uuid" : "KOpuhMgVRzW_9OTjMsHf2Q",
 "version" : {
  "number" : "6.6.0",
  "build_flavor" : "default",
  "build_type" : "tar",
  "build_hash" : "eb782d0",
  "build_date" : "2018-06-29T21:59:26.107521Z",
  "build_snapshot" : false,
  "lucene_version" : "7.3.1",
  "minimum_wire_compatibility_version" : "5.6.0",
  "minimum_index_compatibility_version" : "5.0.0"
 },
 "tagline" : "You Know, for Search"
}
```

------------------------

7）停止集群

​    `kill -9 进程号`

------------------------

8）群起脚本

`[atguigu@hadoop102 bin]$ vim es.sh`

```shell
#!/bin/bash
es_home=/opt/module/elasticsearch
case $1 in
 "start") {
 for i in hadoop102 hadoop103 hadoop104
 do
  echo "==============$i=============="
  ssh $i "source /etc/profile;${es_home}/bin/elasticsearch >/dev/null 2>&1 &"
 done
};;
"stop") {
 for i in hadoop102 hadoop103 hadoop104
 do
  echo "==============$i=============="
  ssh $i "ps -ef|grep $es_home |grep -v grep|awk '{print \$2}'|xargs kill" >/dev/null 2>&1
 done
};;
esac
```



## 使用ElasticSearch

### 数据类型

- 核心数据类型

  字符串：text(分词) keyword不分词

  数值：long，integer，short，byte，double，float，half_float，scaled_float

  日期：date

  布尔：boolean

  二进制：binary

  范围：integer_range、float_range、long_range、double_range、date_range

- 复杂数据类型

  数组类型：array

  对象类型：object

  嵌套类型：nested object

- 地理位置数据类型

  geo_point(点)、geo_shape(形状)

- 专用类型

  记录IP地址ip

  实现自动补全completion

  记录分词数：token_count

  记录字符串hash值murmur3

  多字段特性multi-fields

### Mapping

- 手动创建

  1）创建操作

  ```
  PUT my_index1
  {
   "mappings": {
    "_doc":{
     "properties":{
  ​    "username":{
  ​     "type": "text", 
  ​     "fields": {
  ​      "pinyin":{
  ​       "type": "text"
  ​      }
  ​     }
  ​    }
     }
    }
   }
  }
  ```

  2）创建文档

  ```
  PUT my_index1/_doc/1
  {
   "username":"haha heihei"
  }
  ```

  3）查询

  ```
  GET my_index1/_search
  {
   "query": {
    "match": {
     "username.pinyin": "haha"
    }
   }
  }
  ```

- 自动创建

  ES可以自动识别文档字段类型，从而降低用户使用成本

  1）直接插入文档

  ```
  PUT /test_index/_doc/1
  {
   "username":"alfred",
   "age":1,
  "birth":"1991-12-15"
  }
  ```

  2）查看mapping

  ```
  GET /test_index/doc/_mapping
  {
   "test_index": {
    "mappings": {
     "doc": {
  ​    "properties": {
  ​     "age": {
  ​      "type": "long"
  ​     },
  ​     "birth": {
  ​      "type": "date"
  ​     },
  ​      "username": {
  ​      "type": "text",
  ​       "fields": {
  ​       "keyword": {
  ​        "type": "keyword",
  ​        "ignore_above": 256
  ​       }
  ​      }
  ​     }
  ​    }
     }
    }
   }
  }
  ```

  age自动识别为long类型，username识别为text类型

  3）日期类型的自动识别

  日期的自动识别可以自行配置日期格式，以满足各种需求。

  （1）自定义日期识别格式

  ```
  PUT my_index
  {
   "mappings":{
    "_doc":{
     "dynamic_date_formats": ["yyyy-MM-dd","yyyy/MM/dd"]
    }
   }
  }
  ```

  （2）关闭日期自动识别

  ```
  PUT my_index
  {
   "mappings": {
    "_doc": {
     "date_detection": false
    }
   }
  }
  ```

  4）字符串是数字时，默认不会自动识别为整形，因为字符串中出现数字时完全合理的

  Numeric_datection可以开启字符串中数字的自动识别

  ```
  PUT my_index
  {
   "mappings":{
    "doc":{
     "numeric_datection": true
    }
   }
  }
  ```

### IK分词器

为什么使用分词器

分词器主要应用在中文上，在ES中字符串类型有keyword和text两种。keyword默认不进行分词，而text是将每一个汉字拆开称为独立的词，这两种都是不适用于生产环境，所以我们需要有其他的分词器帮助我们完成这些事情，其中IK分词器是应用最为广泛的一个分词器。

1）keyword类型的分词

```
GET _analyze
{
 "keyword":"我是程序员"
}
```

结果展示（报错）

```
{
 "error": {
  "root_cause": [
   {
​    "type": "illegal_argument_exception",
​     "reason": "Unknown parameter [keyword] in request body or parameter is of the wrong type[VALUE_STRING] "
   }
  ],
  "type": "illegal_argument_exception",
  "reason": "Unknown parameter [keyword] in request body or parameter is of the wrong type[VALUE_STRING] "
 },
 "status": 400
}
```

2）text类型的分词

```
GET _analyze
{
 "text":"我是程序员"
}
```

结果展示：

```
{
 "tokens": [
  {
   "token": "我",
   "start_offset": 0,
   "end_offset": 1,
   "type": "<IDEOGRAPHIC>",
   "position": 0
  },
  {
   "token": "是",
   "start_offset": 1,
   "end_offset": 2,
   "type": "<IDEOGRAPHIC>",
   "position": 1
  },
  {
   "token": "程",
   "start_offset": 2,
   "end_offset": 3,
   "type": "<IDEOGRAPHIC>",
   "position": 2
  },
  {
   "token": "序",
   "start_offset": 3,
   "end_offset": 4,
   "type": "<IDEOGRAPHIC>",
   "position": 3
  },
  {
   "token": "员",
   "start_offset": 4,
   "end_offset": 5,
   "type": "<IDEOGRAPHIC>",
   "position": 4
  }
 ]
}
```

#### 3.3.2 IK分词器安装

1）下载与安装的ES相对应的版本

2）解压elasticsearch-analysis-ik-6.6.0.zip，将解压后的IK文件夹拷贝到ES安装目录下的plugins目录下，并重命名文件夹为ik（什么名称都OK）

`[atguigu@hadoop102 plugins]$ mkdir ik`

`[atguigu@hadoop102 software]$ unzip elasticsearch-analysis-ik-6.6.0.zip -d /opt/module/elasticsearch-6.6.0/plugins/ik/`

3）分发分词器目录

`[atguigu@hadoop102 elasticsearch-6.6.0]$ xsync plugins/`

4）重新启动Elasticsearch，即可加载IK分词器

#### 3.3.3 IK分词器测试

IK提供了两个分词算法ik_smart 和 ik_max_word，其中 ik_smart 为最少切分，ik_max_word为最细粒度划分。

1）最少划分ik_smart

```
get _analyze
{
 "analyzer": "ik_smart",
 "text":"我是程序员"
}
```

结果展示

```
{
  "tokens" : [
​     {
​      "token" : "我",
​      "start_offset" : 0,
​      "end_offset" : 1,
​      "type" : "CN_CHAR",
​      "position" : 0
​    },

​    {
​      "token" : "是",
​      "start_offset" : 1,
​       "end_offset" : 2,
​      "type" : "CN_CHAR",
​      "position" : 1
​    },

​    {
​      "token" : "程序员",
​      "start_offset" : 2,
​      "end_offset" : 5,
​      "type" : "CN_WORD",
​      "position" : 2
​    }
  ]
}
```

2）最细切分ik_max_word

```
get _analyze
{
 "analyzer": "ik_max_word",
 "text":"我是程序员"
}
```

输出的结果为：

```
{
  "tokens" : [

​    {
​       "token" : "我",
​      "start_offset" : 0,
​      "end_offset" : 1,
​      "type" : "CN_CHAR",
​      "position" : 0
​    },
​    {
​      "token" : "是",
​      "start_offset" : 1,
​      "end_offset" : 2,
​      "type" : "CN_CHAR",
​      "position" : 1
​    },
​    {
​      "token" : "程序员",
​      "start_offset" : 2,
​      "end_offset" : 5,
​      "type" : "CN_WORD",
​      "position" : 2
​    },
​    {
​      "token" : "程序",
​      "start_offset" : 2,
​      "end_offset" : 4,
​      "type" : "CN_WORD",
​      "position" : 3
​    },

​    {
​      "token" : "员",
​      "start_offset" : 4,
​      "end_offset" : 5,
​       "type" : "CN_CHAR",
​      "position" : 4
​    }

  ]

}
```

### 检索文档

向Elasticsearch增加数据

```
PUT /atguigu/doc/1
{
  "first_name" : "John",
  "last_name" : "Smith",
  "age" :     25,
  "about" :   "I love to go rock climbing",
  "interests": ["sports", "music"]
}
```

如果在关系型数据库Mysql中主键查询数据一般会执行下面的SQL语句

`select * from atguigu where id = 1;`

但在Elasticsearch中需要采用特殊的方式

\# 协议方法 索引/类型/文档编号

`GET /atguigu/doc/1`

响应

```
{
 "_index": "atguigu",
 "_type": "doc",
 "_id": "1",
 "_version": 1,
 "found": true,
 "_source": { // 文档的原始数据JSON数据
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
   "sports",
   "music"
  ]
 }
}
```

我们通过HTTP方法GET来检索文档，同样的，我们可以使用DELETE方法删除文档，使用HEAD方法检查某文档是否存在。如果想更新已存在的文档，我们只需再PUT一次。

#### 3.4.1 元数据查询

`GET _cat/indices`

| health         | green(集群完整) yellow(单点正常、集群不完整) red(单点不正常) |
| -------------- | ------------------------------------------------------------ |
| status         | 是否能使用                                                   |
| index          | 索引名                                                       |
| uuid           | 索引统一编号                                                 |
| pri            | 主节点几个                                                   |
| rep            | 从节点几个                                                   |
| docs.count     | 文档数                                                       |
| docs.deleted   | 文档被删了多少                                               |
| store.size     | 整体占空间大小                                               |
| pri.store.size | 主节点占                                                     |

#### 3.4.2 全文档检索

如果在关系型数据库Mysql中查询所有数据一般会执行下面的SQL语句

`select * from user;`

但在Elasticsearch中需要采用特殊的方式

\# 协议方法 索引/类型/_search

`GET /atguigu/_doc/**_search**`

响应内容不仅会告诉我们哪些文档被匹配到，而且这些文档内容完整的被包含在其中—我们在给用户展示搜索结果时需要用到的所有信息都有了。

#### 3.4.3 字段全值匹配检索

如果在关系型数据库Mysql中查询多字段匹配数据（字段检索）

一般会执行下面的SQL语句

`select * from atguigu where name = 'haha';`

但在Elasticsearch中需要采用特殊的方式

```
GET atguigu/_search

{

 "query": {

  "bool": {

   "filter": {

​    "term": {

​     "about": "I love to go rock climbing"

​    }

   }

  }

 }

}
```

#### 3.4.4 字段分词匹配检索

```
GET atguigu/_search

{

 "query": {

  "match": {

   "about": "I"

  }

 }

}
```

#### 3.4.5 字段模糊匹配检索

如果在关系型数据库Mysql中模糊查询多字段数据

一般会执行下面的SQL语句

`select * from user where name like '%haha%'`

但在Elasticsearch中需要采用特殊的方式，查询出所有文档字段值分词后包含haha的文档

```
GET test/_search

{

 "query": {

  "fuzzy": {

   "aa": {

​    "value": "我是程序"

   }

  }

 }

}
```

#### 3.4.6 聚合检索

```
GET test/_search

{

 "aggs": {

  "groupby_aa": {

   "terms": {

​    "field": "aa",

​    "size": 10

   }

  }

 }

}
```

#### 3.4.7 分页检索

```
GET movie_index/movie/_search
{

 "query": { "match_all": {} },
 "from": 1,
 "size": 1
}
```

### 索引别名_aliases

索引*别名*就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何一个需要索引名的API来使用。别名带给我们极大的灵活性，允许我们做下面这些：

1）给多个索引分组 (例如， last_three_months)

2）给索引的一个子集创建视图

3）在运行的集群中可以无缝的从一个索引切换到另一个索引

#### 3.5.1 创建索引别名

建表时直接声明

```
PUT movie_chn_2020

{ "aliases": {

   "movie_chn_2020-query": {}

 }, 

 "mappings": {

  "movie":{

   "properties": {

​    "id":{

​     "type": "long"

​    },

​    "name":{

​     "type": "text"

​     , "analyzer": "ik_smart"

​    },

​    "doubanScore":{

​     "type": "double"

​    },

​    "actorList":{

​     "properties": {

​      "id":{

​       "type":"long"

​      },

​      "name":{

​       "type":"keyword"

​      }

​     }

​    }

   }

  }

 }

}
```

为已存在的索引增加别名

```
POST _aliases

{

  "actions": [

​    { "add":  { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }}

  ]

}

也可以通过加过滤条件缩小查询范围，建立一个子集视图

POST _aliases

{

  "actions": [

​    { "add":  

{ "index": "movie_chn_xxxx", 

"alias": "movie_chn0919-query-zhhy",

​       "filter": {

​         "term": {  "actorList.id": "3"

​         }

​        }

 }

}

  ]

}
```

#### 3.5.2 查询别名

与使用普通索引没有区别

`GET movie_chn_2020-query/_search`

#### 3.5.3 删除某个索引的别名

```
POST _aliases
{
  "actions": [
​    { "remove":  { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }}
  ]
}
```

#### 3.5.4 为某个别名进行无缝切换

```
POST /_aliases
{
  "actions": [
​    { "remove": { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }},
​    { "add":  { "index": "movie_chn_yyyy", "alias": "movie_chn_2020-query" }}
  ]
}
```

#### 3.5.5 查询别名列表

`GET _cat/aliases?v`

### 索引模板

Index Template 索引模板，顾名思义，就是创建索引的模具，其中可以定义一系列规则来帮助我们构建符合特定业务需求的索引的mappings和 settings，通过使用 Index Template 可以让我们的索引具备可预知的一致性。

#### 6.1 常见的场景: 分割索引

   分割索引就是根据时间间隔把一个业务索引切分成多个索引。比如把order_info 变成 order_info_20200101,order_info_20200102 …..

这样做的好处有两个：

结构变化的灵活性：因为elasticsearch不允许对数据结构进行修改。但是实际使用中索引的结构和配置难免变化，那么只要对下一个间隔的索引进行修改，原来的索引位置原状。这样就有了一定的灵活性。

查询范围优化：因为一般情况并不会查询全部时间周期的数据，那么通过切分索引，物理上减少了扫描数据的范围，也是对性能的优化。

#### 6.2 创建模板

```
PUT _template/template_movie2020
{
 "index_patterns": ["movie_test*"],          
 "settings": {                        
  "number_of_shards": 1
 },
 "aliases" : { 
  "{index}-query": {},
  "movie_test-query":{}
 },
 "mappings": {                      
"_doc": {
   "properties": {
​    "id": {
​     "type": "keyword"
​    },
​    "movie_name": {
​     "type": "text",
​     "analyzer": "ik_smart"
​    }
   }
  }
 }
}
```

其中 "index_patterns": ["movie_test*"], 的含义就是凡是往movie_test开头的索引写入数据时，如果索引不存在，那么es会根据此模板自动建立索引。

在 "aliases" 中用{index}表示，获得真正的创建的索引名。

测试

```
POST movie_test_2020xxxx/_doc
{
 "id":"333",
 "name":"zhang3"
}
```

#### 6.3 查看系统中已有的模板清单

`GET _cat/templates`

#### 6.4查看某个模板详情

```
GET _template/template_movie2020
或者
GET _template/template_movie*
```

### API操作

新建工程并导入依赖：

```xml
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
   <version>4.5.5</version>
</dependency>
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpmime</artifactId>
  <version>4.3.6</version>
</dependency>
<dependency>
  <groupId>io.searchbox</groupId>
  <artifactId>jest</artifactId>
  <version>5.3.3</version>
</dependency>
<dependency>
  <groupId>net.java.dev.jna</groupId>
  <artifactId>jna</artifactId>
  <version>4.5.2</version>
</dependency>
<dependency>
  <groupId>org.codehaus.janino</groupId>
  <artifactId>commons-compiler</artifactId>
  <version>2.7.8</version>
</dependency>
 <dependency>
   <groupId>org.elasticsearch</groupId>
   <artifactId>elasticsearch</artifactId>
   <version>6.6.0</version>
 </dependency>
```

#### 3.7.1 写数据

```Java
//1.创建ES客户端连接池
​    JestClientFactory factory = new JestClientFactory();
​    //2.创建ES客户端连接地址
​    HttpClientConfig httpClientConfig = new HttpClientConfig.Builder("http://hadoop102:9200").build();
​    //3.设置ES连接地址
​    factory.setHttpClientConfig(httpClientConfig);
​    //4.获取ES客户端连接
​    JestClient jestClient = factory.getObject();
​    //5.构建ES插入数据对象
​    Index index = new Index.Builder("{\n" +
​        " \"name\":\"zhangsan\",\n" +
​        " \"age\":17\n" +
​        "}").index("test5").type("_doc").id("2").build();
​    //6.执行插入数据操作
​    jestClient.execute(index);
​    //7.关闭连接
​    jestClient.shutdownClient();
```

#### 3.7.2 读数据

```java
//1.创建ES客户端连接池
JestClientFactory factory = new JestClientFactory();

//2.创建ES客户端连接地址
HttpClientConfig httpClientConfig = new 
HttpClientConfig.Builder("http://hadoop102:9200").build();

//3.设置ES连接地址
factory.setHttpClientConfig(httpClientConfig);

 //4.获取ES客户端连接
JestClient jestClient = factory.getObject();

 //5.构建查询数据对象

Search search = new Search.Builder("{\n" +

" \"query\": {\n" +

"  \"match\": {\n" +

"   \"name\": \"zhangsan\"\n" +

"  }\n" +

" }\n" +

"}").addIndex("test5").addType("_doc").build();


//6.执行查询操作

 SearchResult searchResult = jestClient.execute(search);


//7.解析查询结果

System.out.println(searchResult.getTotal());

List<SearchResult.Hit<Map, Void>> hits = searchResult.getHits(Map.class);

for (SearchResult.Hit<Map, Void> hit : hits) {

System.out.println(hit.index + "--" + hit.id);

}

//8.关闭连接
jestClient.shutdownClient();
```



## Kibana安装

1.将kibana压缩包上传到虚拟机指定目录

`[atguigu@hadoop102 software]$ tar -zxvf kibana-6.6.0-linux-x86_64.tar.gz -C /opt/module/`

2.修改相关配置，连接Elasticsearch

`[atguigu@hadoop102 kibana]$ vi config/kibana.yml`

```yml
# Kibana is served by a back end server. This setting specifies the port to use.

server.port: 5601

\# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.

\# The default is 'localhost', which usually means remote machines will not be able to connect.

\# To allow connections from remote users, set this parameter to a non-loopback address.

server.host: "192.168.9.102"

... ...

... ...

\# The URL of the Elasticsearch instance to use for all your queries.

elasticsearch.url: "http://192.168.9.102:9200"
```

3.启动Kibana

`[atguigu@hadoop102 kibana]$ bin/kibana`

![img](G:\Temp\github\backspace2019.github.io\img\in-post\2020-05\Kibana.png)

4.修改之前的ES启动脚本为：

```shell
#!/bin/bash
es_home=/opt/module/elasticsearch
kibana_home=/opt/module/kibana
case $1 in
 "start") {
 for i in hadoop102 hadoop103 hadoop104
 do
  echo "==============$i=============="
  ssh $i "source /etc/profile;${es_home}/bin/elasticsearch >/dev/null 2>&1 &"
 done
 nohup ${kibana_home}/bin/kibana > kibana.log 2>&1 &
};;
"stop") {
 ps -ef | grep ${kibana_home} | grep -v grep | awk '{print $2}'| xargs kill
 for i in hadoop102 hadoop103 hadoop104
 do
  echo "==============$i=============="
  ssh $i "ps -ef|grep $es_home |grep -v grep|awk '{print \$2}'|xargs kill" >/dev/null 2>&1
 done
};;
esac
```
