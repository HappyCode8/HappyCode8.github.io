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

## kibana、es、ik

```shell
docker network create es-net
docker pull elasticsearch:7.12.1
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1

命令解释：
-e "cluster.name=es-docker-cluster"：设置集群名称
-e "http.host=0.0.0.0"：监听的地址，可以外网访问
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"：内存大小
-e "discovery.type=single-node"：非集群模式
-v es-data:/usr/share/elasticsearch/data：挂载逻辑卷，绑定es的数据目录
-v es-logs:/usr/share/elasticsearch/logs：挂载逻辑卷，绑定es的日志目录
-v es-plugins:/usr/share/elasticsearch/plugins：挂载逻辑卷，绑定es的插件目录
--privileged：授予逻辑卷访问权
--network es-net ：加入一个名为es-net的网络中
-p 9200:9200：端口映射配置
在浏览器中输入：http://localhost:9200 即可看到elasticsearch的响应结果
```

- 安装kibana

  ```shell
  docker pull kibana:7.12.1
  docker run -d \
  --name kibana \
  -e ELASTICSEARCH_HOSTS=http://es:9200 \
  --network=es-net \
  -p 5601:5601  \
  kibana:7.12.1
  
  --network es-net ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
  -e ELASTICSEARCH_HOSTS=http://es:9200"：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
  -p 5601:5601：端口映射配置
  ```

- 安装ik插件

  ```shell
  # 进入容器内部
  docker exec -it elasticsearch /bin/bash
  
  # 在线下载并安装
  ./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip
  
  #退出
  exit
  #重启容器
  docker restart elasticsearch
  ```

  