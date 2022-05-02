# Hbase

## 逻辑结构

![image-20210912212744733](./images/image-20210912212744733.png)

- 行键：字典序
- 根据rowkey水平切分，根据column-family垂直切分
- Region是表的切片

## 物理存储结构

![image-20210912214138790](./images/image-20210912214138790.png)

- 删除时当时不会立刻删除,只是通过比较时间戳+类型字段确定

## 数据模型

1. Name Space

   ![image-20210912215327048](./images/image-20210912215327048.png)

2. Region

   ![image-20210912215349208](./images/image-20210912215349208.png)

3. Row

   ![image-20210912214737224](./images/image-20210912214737224.png)

   范围查询也可以

4. Column

   ![image-20210912214834798](./images/image-20210912214834798.png)

5. Time Stamp

   ![image-20210912214923197](./images/image-20210912214923197.png)

6. Cell

   ![image-20210912214955629](./images/image-20210912214955629.png)

## 架构

![image-20210912220516100](./images/image-20210912220516100.png)

## 安装

```shell
docker search hbase
docker pull harisekhon/hbase
docker run -d --name hbase001 -p 16010:16010 harisekhon/hbase:latest
docker exec -it hbase001  bash
hbase shell
界面访问：localhost:16010
docker stop hbase001
```

