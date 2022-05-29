ES

## 安装

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.12.1
    container_name: elasticsearch
    environment:
      - cluster.name=elastic-pro
      - node.name=node_master_9200
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.type=single-node"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /Users/wyj/Documents/elasticsearch/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
  kibana:
    image: kibana:7.12.1
    container_name: kibana
    environment:
      - SERVER_NAME=kibana
      - ELASTICSEARCH_URL=elasticsearch:9200
      - XPACK_MONITORING_ENABLED=true
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
```

Localhost:9200,出现如图所示的信息时启动成功

```json
{
  "name" : "node_master_9200",
  "cluster_name" : "elastic-pro",
  "cluster_uuid" : "qoN8h3UhRyS6qW74LJlq1Q",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

localhost:5601使用kibana管理ES，在management的devtools可以运行ES命令

## 常用命令

> scan 'student',{LIMIT=>10} 查询数据表最多返回10条

