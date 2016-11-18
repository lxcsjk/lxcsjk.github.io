---
layout: post
title:  "zookeeper和solr搭建集群分片查询"
date:   2015-11-15 19:02:03 +0800
categories: jekyll update
tags: zookeeper Solr 集群 分片
---

这几天双十一弄得不要不要的。各种困。出差有一些时间。晚上回头摆弄摆弄。白天不忙就是找个地方想想写写。就这样一周多过去了。好了。不扯了入正题。

1 .环境搭建

    MacBook pro 15款840 
    OS X 10.10.5
    solr-5.2.1.tgz
    zookeeper-3.4.6.tar.gz
    VMWare Fusion8 
    Centos 6.7
    
 2 .搭建solr集群
 
	 在之前说过zookeeper集群的搭建，所以在这就别啰嗦了。基本是一样的。不过因为之前搭建过rabbitmq集群，改了一些配置。而且这里网络环境总是在变，所以IP地址不稳定，虚拟出的主机ip搞得甚是头疼。
	 安装solr 
	 

```
[root@rabbitmq-node2 bin]# cp /usr/programmer/solr-5.2.1.tgz /usr/local/

[root@rabbitmq-node2 bin]# cd /usr/local/

[root@rabbitmq-node2 local]# ls

bin    include      lib64    nginx    share           tomcat

etc    jdk1.7.0_80  libexec  openssl  solr-5.2.1.tgz  usr

games  lib          mongodb  sbin     src             zookeeper

[root@rabbitmq-node2 local]# tar xzf solr-5.2.1.tgz  solr-5.2.1/bin/install_solr_service.sh --strip-components=2

[root@rabbitmq-node2 local]#  ./install_solr_service.sh  solr-5.2.1.tgz -i /usr/local  -u solr -s solr -p 8983

Extracting solr-5.2.1.tgz to /usr/local

Creating /etc/init.d/solr script ...

Waiting to see Solr listening on port 8983 [/]  

Started Solr server on port 8983 (pid=2859). Happy searching!

Found 1 Solr nodes: 

Solr process 2859 running on port 8983

{

  "solr_home":"/var/solr/data/",

  "version":"5.2.1 1684708 - shalin - 2015-06-10 23:20:13",

  "startTime":"2015-11-15T01:59:53.628Z",

  "uptime":"0 days, 0 hours, 0 minutes, 11 seconds",

  "memory":"25.5 MB (%5.2) of 490.7 MB"}

Service solr installed.
```

```
-i安装目录 指定solr的安装目录  (默认为/opt)
-d指定写文件的目录，包括索引/日志/初始环境变量的配置等，(默认为/var/solr)
-u 指定solr文件和运行的所属用户，默认solr账号
-s solr服务的名称  默认为solr
-p solr服务的监听端口 默认为8983 默认为8983```
```

修改solr配置，与zookeeper进行通信

```
[root@rabbitmq-node2 local]# vim /var/solr/solr.in.sh 
```
这里需要注意 空格。 
```
ZK_HOST="192.168.1.167:2181,192.168.1.166:2181,192.168.1.168:2181"
```

在启动zookeeper时候要查看zookeeper.out这个启动日志文件，这个文件在你当前启动的目录。启动zookeeper集群要快一点。不然有报错。但是没事，看最后一个zookeeper启动日志 如果没有报错 基本上没有问题的。

同样的，solr启动日志 也是需要看的 但是会生成很多。我习惯把logs全部清楚只看新生成的solr.log。

修改之后重新启动一下solr服务

```
[root@rabbitmq-node2 local]# service solr restart

Sending stop command to Solr running on port 8983 ... waiting 5 seconds to allow Jetty process 2859 to stop gracefully.

Waiting to see Solr listening on port 8983 [/]  

Started Solr server on port 8983 (pid=3201). Happy searching!
```

之前用的是tomcat和solr整合，但是我觉得solr自带jetty容器。可以不用tomcat了。同样也可以用nginx进行负载均衡。

这时候通过浏览器访问以下 ip:8983.

![这里写图片描述](http://img.blog.csdn.net/20151115161225934)

会出现这个 如果报错。看log日志。


创建collection

```
[root@rabbitmq-node1 logs]# cd /usr/local/solr

[root@rabbitmq-node1 solr]# ls

CHANGES.txt  LUCENE_CHANGES.txt  README.txt  contrib  docs     licenses

LICENSE.txt  NOTICE.txt          bin         dist     example  server

[root@rabbitmq-node1 solr]# bin/solr create -c szss-solr -d data_driven_schema_configs -s 3 -rf 3 -n myconf

Connecting to ZooKeeper at 192.168.1.167:2181,192.168.1.166:2181,192.168.1.168:2181

Uploading /usr/local/solr/server/solr/configsets/data_driven_schema_configs/conf for config myconf to ZooKeeper at 192.168.1.167:2181,192.168.1.166:2181,192.168.1.168:2181



Creating new collection 'szss-solr' using command:

http://192.168.1.167:8983/solr/admin/collections?action=CREATE&name=szss-solr&numShards=3&replicationFactor=3&maxShardsPerNode=3&collection.configName=myconf



{

  "responseHeader":{

    "status":0,

    "QTime":19520},

  "success":{"":{

      "responseHeader":{

        "status":0,

        "QTime":18974},

      "core":"szss-solr_shard1_replica2"}}}

```

-s分片个数
-rf  节点数

这时候可能会报错。我之前想把分两片 改为 分三片  需要在zookeeper里面修改

查看zookeeper的客户端命令：

```
查看节点列表：ls /path
获取节点数据：get /path
删除所有节点：rmr path
关闭节点:quit
查看节点状态：stat path
create -s /source sss  创建永久节点
create -e /temp sss  创建临时节点
集群状态的查看：./zkServer.sh status
```
打开浏览器。
![这里写图片描述](http://img.blog.csdn.net/20151115162540354)

一些参数信息
![这里写图片描述](http://img.blog.csdn.net/20151115162620511)

这里是虚拟机运行的参数
![这里写图片描述](http://img.blog.csdn.net/20151115162712016)

log日志 以及打印的等级
![这里写图片描述](http://img.blog.csdn.net/20151115162809408)

分片的信息
![这里写图片描述](http://img.blog.csdn.net/20151115162902672)


之后的分词，还有数据连接和之前写的都是一样的。在solr_home里面操作。加入3个jar，加入/dataimport data-config.xml 还有managed-schema。

下载sqljdbc4.jar 包 放在 
/usr/local/solr-5.2.1/server/solr-webapp/webapp/WEB-INF/lib/
solr-5.2.1/dist/solr-dataimporthandler-5.x.jar    到 /usr/local/solr-5.2.1/server/solr-webapp/webapp/WEB-INF/lib/下

动态加载配置文件到zookeeper中并生效

```
[root@rabbitmq-node1 solr]# /usr/local/solr-5.2.1/server/scripts/cloud-scripts/zkcli.sh -zkhost 192.168.1.168:2181 -cmd upconfig -collections szss-solr -confdir /usr/local/solr-5.2.1/server/solr/configsets/data_driven_schema_configs/conf -confname myconf
```

进入：
/usr/local/solr/server/solr/configsets/data_driven_schema_configs/conf/
vim solrconfig.xml
在requestHandler处新建：

```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
     <lst name="defaults">
          <str name="config">data-config.xml</str>
     </lst>
</requestHandler>
```
![这里写图片描述](http://img.blog.csdn.net/20151115163159992)
3、新建data-config.xml
新建一个data-config.xml文件，与solrconfig.xml同一个目录下,内容如下，数据库驱动/链接地址/sql语句请修改。

```
<?xml version="1.0" ?>
<dataConfig>
    <dataSource type="JdbcDataSource"
              driver="com.microsoft.sqlserver.jdbc.SQLServerDriver"
              url="jdbc:sqlserver://127.0.0.1;databaseName=szss"
              user="sa"
              password="szss" />
    <document>
        <entity name="solr_test" transformer="DateFormatTransformer"
            query="select id,product_full_name,product_short_name,product_content,specification,taste_type,date_created,last_updated from product_b">
            <field column='date_created' dateTimeFormat='yyyy-MM-dd HH:mm:ss' />
            <field column='last_updated' dateTimeFormat='yyyy-MM-dd HH:mm:ss' />
        </entity>
    </document>
</dataConfig>
```

4、在managed-schema中增加域

```
    <field name="product_full_name" type="string" indexed="true" stored="true" />
    <field name="product_short_name" type="string" indexed="true" stored="true" />
    <field name="product_content" type="string" indexed="true" stored="true" />
    <field name="specification" type="string" indexed="true" stored="true" />
    <field name="taste_type" type="string" indexed="true" stored="true" />
    <field name="date_created" type="date" indexed="true" stored="true" />
    <field name="last_updated" type="date" indexed="true" stored="true" />
```
![这里写图片描述](http://img.blog.csdn.net/20151115165729329)