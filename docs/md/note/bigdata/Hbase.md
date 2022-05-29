# Hbase

## 逻辑结构

![image-20210912212744733.png](https://s2.loli.net/2022/05/28/1cfzlOHeAEUI8bF.png)

- 行键：字典序
- 根据rowkey水平切分，根据column-family垂直切分
- Region是表的切片

## 物理存储结构

![image-20210912214138790.png](https://s2.loli.net/2022/05/28/TBsaYCwxFc1Rq8n.png)

- 删除时当时不会立刻删除,只是通过比较时间戳+类型字段确定

## 数据模型

1. Name Space

   ![image-20210912215327048.png](https://s2.loli.net/2022/05/28/dXbvSrFtGlVqULw.png)

2. Region

   ![image-20210912215349208.png](https://s2.loli.net/2022/05/28/dJOCFLoU89n5uB7.png)

3. Row

   ![image-20210912214737224.png](https://s2.loli.net/2022/05/28/Xjqo7aY8r9WTeAR.png)

   范围查询也可以

4. Column

   ![image-20210912214834798.png](https://s2.loli.net/2022/05/28/7axVKr63LCI8qdg.png)

5. Time Stamp

   ![image-20210912214923197.png](https://s2.loli.net/2022/05/28/8KIxqb3NsrFHgUT.png)

6. Cell

   ![image-20210912214955629.png](https://s2.loli.net/2022/05/28/swOezPZMvAB5nHl.png)

## 架构

![image-20210912220516100.png](https://s2.loli.net/2022/05/28/yg7wfierCcT3Klm.png)

## 安装

```shell
docker search hbase

docker pull harisekhon/hbase
#启动zk
docker run -d --name zokeeper -p 2181:2181 wurstmeister/zookeeper
#2181口被zk占了，将2182映射到2181
docker run -d -p 2182:2181  -p 9090:9090 -p 9095:9095 -p 16000:16000 -p 16010:16010 -p 16201:16201 -p 16301:16301  -p 16030:16030 -p 16020:16020 --name hbase001 harisekhon/hbase
#进入zk以后，cd /habse/bin
./hbase shell
界面访问：localhost:16010
要想访问节点情况，需要把Region Servers下的ServerName对应的12位串码映射到/etc/hosts/ localhost
```

