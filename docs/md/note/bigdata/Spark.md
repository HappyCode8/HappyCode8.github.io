

# 安装与启动

docker pull singularities/spark

```yml
version: "2"
 
services:
  master:
    image: singularities/spark
    command: start-spark master
    hostname: master
    ports:
      - "6066:6066"
      - "7070:7070"
      - "8080:8080"
      - "50070:50070"
  worker:
    image: singularities/spark
    command: start-spark worker master
    environment:
      SPARK_WORKER_CORES: 1
      SPARK_WORKER_MEMORY: 2g
    links:
      - master
    volumes:
      # 设置本地目录和镜像目录的映射关系（格式：本地机器目录:镜像中对应路径）
      - /Users/wyj/Documents/spark/data:/input_files
```

