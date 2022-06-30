- 安装与启动：

  ```bash
  安装：brew install apache-flink
  找到安装目录：brew info apache-flink
  进入安装目录执行：  ./libexec/bin/start-cluster.sh
  web页面(http://localhost:8081/)
  例子：https://www.cnblogs.com/duniqb/p/14070809.html
  ```

  无法关闭：https://blog.csdn.net/qq_37135484/article/details/102474087

  使用docker安装

  docker-compose.yml

  ```yml
  version: "2.1"
  services:
    jobmanager:
      image: flink
      expose:
        - "6123"
      ports:
        - "8081:8081"
      command: jobmanager
      environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager
  
    taskmanager:
      image: flink
      expose:
        - "6121"
        - "6122"
      depends_on:
        - jobmanager
      command: taskmanager
      links:
        - "jobmanager:jobmanager"
      environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager
  ```

  运行

  ```
  docker-compose up -d
  ```

  浏览器打开 http://127.0.0.1:8081 可以看到dashboard