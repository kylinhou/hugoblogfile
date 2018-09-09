---
title: Elasticsearch的安装与配置
categories: ["Elasticsearch"]
tags: ["Elasticsearch"]
date: 2018-07-23
author: "kylin"
grammar_cjkRuby: true
---

# Elasticsearch的安装与配置

环境信息：

centos、Elasticsearch2.1.1

## 安装

### 下载

> curl -L -O https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-2.1.1.zip

### 解压 

> yum install -y unzip zip
>
> unzip elasticsearch-2.1.1.zip

### 启动

>  cd elasticsearch-2.1.1
>
>  ./bin/elasticsearch -d

至此，安装完成

## 配置

主要修改的配置：

* cluster.name：集群名称
* node.name：节点名称
* network.host：节点的ip
* transport.tcp.port：tcp端口

* http.port：http端口
* discovery.zen.ping.unicast.hosts：整个集群的ip

附完整配置

```xml
################################### Cluster ###################################
cluster.name: es2.1.1-appsearch-bj

#################################### Node #####################################
node.name: "es-81-9300"

#################################### Index ####################################
index.number_of_shards: 5
index.number_of_replicas: 1
#################################### Paths ####################################

################################### Thread_Pool ###############################
threadpool:
    search:
        type: fixed
        min: 96
        max: 96
        queue_size: 2k

################################### Memory ####################################
network.host: 10.101.22.81
transport.tcp.port: 9300
transport.tcp.compress: false
http.port: 9200

discovery.zen.ping.unicast.hosts: ["10.101.22.81", "10.101.22.78","10.101.22.84","10.101.22.85"]
################################## Slow Log ##################################
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms
index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
index.indexing.slowlog.threshold.index.warn: 10s
index.indexing.slowlog.threshold.index.info: 5s
index.indexing.slowlog.threshold.index.debug: 2s
index.indexing.slowlog.threshold.index.trace: 500ms
################################## GC Logging ################################

monitor.jvm.gc.young.warn: 1000ms
monitor.jvm.gc.young.info: 700ms
monitor.jvm.gc.young.debug: 400ms
monitor.jvm.gc.old.warn: 10s
monitor.jvm.gc.old.info: 5s
monitor.jvm.gc.old.debug: 2s

################################## Security ################################
################################# analyzer #################################

index:
  analysis:
    filter:
      my_synonym:
         type: synonym
         synonyms_path : analysis/synonym.txt
    analyzer:
      ik:
          alias: [ik_analyzer]
          type: org.elasticsearch.index.analysis.IkAnalyzerProvider
      ik_max_word:
          type: ik
          use_smart: false
      ik_smart:
          type: ik
          use_smart: true

      ik_syno:
          type: custom
          tokenizer: ik
          filter: [my_synonym]
          use_smart : true
      ik_max_word_syno:
          type: custom
          tokenizer: ik
          filter: [my_synonym]
          use_smart: false

#end
```

