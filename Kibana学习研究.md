# Kibana 学习研究
------------

## 1. Kibana 介绍

Kibana 是一款开源的数据分析和可视化平台，它是`ELK`成员之一，设计用于和 Elasticsearch 协作。

你可以使用 Kibana 对 Elasticsearch 索引中的数据进行搜索、查看。

用户可以在大量数据之上很方便的创建条形图，折线图和散点图，或饼图和数据地图，从而对数据进行多元化的分析和呈现。

## 2. Kibana 安装和配置

### 2.1 安装

选择的Kibana版本最好与Elasticsearch版本相同，我安装的ES版本是5.5.3，找到相应的[Kibana版本](https://www.elastic.co/guide/en/kibana/current/index.html)，跟着[官方安装文档](https://www.elastic.co/guide/en/kibana/5.5/targz.html)安装即可。

```sh
# cd /usr/local/software/
# wget https://artifacts.elastic.co/downloads/kibana/kibana-5.5.3-linux-x86_64.tar.gz
# tar -zxvf  kibana-5.5.3-linux-x86_64.tar.gz -C /usr/local/
```

运行`Kibana`

```
[root@localhost software]# cd /usr/local/kibana-5.5.3-linux-x86_64/
[root@localhost kibana-5.5.3-linux-x86_64]# ./bin/kibana
```

默认`Kinaba`是运行在**5601**端口上，我们可以检查该端口有没有在监听服务：

```sh
[root@localhost ~]# netstat -tlunp | grep 5601
tcp        0      0 127.0.0.1:5601          0.0.0.0:*               LISTEN      19272/./bin/../node
```

我们一般把`Kibana`放到后台运行：

```sh
[root@localhost kibana-5.5.3-linux-x86_64]# nohup /usr/local/kibana-5.5.3-linux-x86_64/bin/kibana &
```

如果需要关闭或者修改了配置文件要重启`Kinaba`，只要找到`Kinaba`进程，然后`kill`掉就行：

```sh
[root@localhost kibana-5.5.3-linux-x86_64]# ps -ef | grep kibana
root     19597  2570 17 19:04 pts/1    00:00:05 /usr/local/kibana-5.5.3-linux-x86_64/bin/../node/bin/node --no-warnings /usr/local/kibana-5.5.3-linux-x86_64/bin/../src/cli
```

### 2.2 配置Kibana

`Kibana`的配置文件是`$KIBANA_HOME/config/kibana.yml`，默认`Kinaba`在`localhost:5601`上运行，及后端连接的`ES`机器是在`http://localhost:9200`上运行，我们可以按自己的情况来配置

```sh
server.host: "192.168.100.144"
server.port: 5601
elasticsearch.url: "http://192.168.100.144:9200"
pid.file: /var/run/kibana-dev.pid
```

上面的`pid.file: /var/run/kibana-dev.pid`意思是为`Kinaba`分配的`进程ID`写到 /var/run/kibana-dev.pid 文件中，这样方便杀死`Kinaba`进程。

```sh
# kill $(cat /var/run/kibana-dev.pid)
```

更多的配置项请参考[官方配置说明](https://www.elastic.co/guide/en/kibana/5.5/settings.html)


## 3. Kinaba 的使用

成功安装好以后就可以通过浏览器来访问`Kinaba`主机了，如下图：

![kibana_start_up](https://github.com/dendi875/images/blob/master/Linux/kibana_start_up.png)

仔细观察上图会发现有一个红色的报错“Unable to fetch mapping. Do you have indices matching the pattern?”，这是因为我们首次访问的时候，会提示你需要定义一个 `Index pattern`(索引模式) 匹配一个或多个索引。默认情况下，Kibana 会认为数据是通过 Logstash 解析送进 Elasticsearch 的。这种情况可以使用默认的 logstash-* 作为索引模式。星号 (*) 匹配0或多个索引名称中的字符
，但我们还没安装 Logstash ，所以才有这个报错。


### 3.1 使用 Index Pattern 配置数据

我们按照官方文档来[创建索引模式](https://www.elastic.co/guide/en/kibana/5.5/index-patterns.html)来配置数据

如下图如示我们我们配置的索引模式为`userdoor*`，因为索引中没有`timestamp `字段，暂时不使用时间过滤器。

![kibana_config_index_pattern](https://github.com/dendi875/images/blob/master/Linux/kibana_config_index_pattern.png)

单击`Create`按钮添加索引模式，创建完成就变成下面这样子：

![kibana_index_pattern_created](https://github.com/dendi875/images/blob/master/Linux/kibana_index_pattern_created.png)


### 3.2 使用 Discover 来搜索数据

索引模式配置好以后，我们就可以使用 Discover 来搜索数据。Discover 也是我们平时使用最多的一个功能。

点击 Discover 就会展示数据，如下面这样：

![kibana_search](https://github.com/dendi875/images/blob/master/Linux/kibana_search.png)


搜索可以包含一个或多个字或者短语。

* 单个字符串搜索

![kibana_search_single_str_query](https://github.com/dendi875/images/blob/master/Linux/kibana_search_single_str_query.png)

* 多个字符串搜索

![kibana_search_multi_str_query](https://github.com/dendi875/images/blob/master/Linux/kibana_search_multi_str_query.png)

**要搜索一个确切的字符串，需要使用双引号引起来。如果不加双引号，则会匹配每个单词。等于使用OR连接查询**

![kibana_search_str_double_quotes](https://github.com/dendi875/images/blob/master/Linux/kibana_search_str_double_quotes.png)


* 基于字段的查询

例如：搜索 `name`字段包含"王五"关键字的记录

![kibana_search_field_query](https://github.com/dendi875/images/blob/master/Linux/kibana_search_field_query.png)

* 支持字段的区间查询

例如：搜索 `age`在 27-28之间的记录

![kibana_search_field_between_and](https://github.com/dendi875/images/blob/master/Linux/kibana_search_field_between_and.png)

* 支持布尔运算符（AND，OR，NOT）

**允许通过逻辑运算符组合多个子查询。运算符AND/OR/NOT必须大写**

![kibana_search_bool](https://github.com/dendi875/images/blob/master/Linux/kibana_search_bool.png)


### 3.3 最后

我们只是对`Kibana`的安装和使用做了简单的介绍，其实`Kinaba`还有其它许多玩法，包括**控制台的使用**、**构建 Dashboard**、**Visualize 可视化视图**、**插件的使用**等等，更多的玩法请参考官方文档。

## 4. 参考资料

- [Kibana 官方文档](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Kibana 官方中文文档](https://www.elastic.co/guide/cn/kibana/current/index.html)



