# Docker

## 获取镜像

`docker pull ubuntu`下载最新的ubuntu操作系统

`docker pull ubuntu：14.04 `下载ubuntu14.04操作系统

`docker pull registry.hub.docker.com/ubuntu：14.04`从指定仓库下载

`docker pull hub.c.163.com/public/ubuntu:14.04`网易蜂巢仓库下载

## 查看镜像

`docker images`查看所有镜像

![image-20201225142250550](./images/image-20201225142250550.png)

`docker tag ubuntu:15.10 myubuntu:x`添加一个新的镜像标签

![image-20201225142536039](./images/image-20201225142536039.png)

## 查看详细信息

`docker inspect ubuntu:15.10`查看详细信息，包括制作者、适应架构、各层数字摘要等

```json
[
    {
        "Id": "sha256:9b9cb95443b5f846cd3c8cfa3f64e63b6ba68de2618a08875a119c81a8f96698",
        "RepoTags": [
            "myubuntu:x",
            "ubuntu:15.10"
        ],
        "RepoDigests": [
            "ubuntu@sha256:02521a2d079595241c6793b2044f02eecf294034f31d6e235ac4b2b54ffc41f3"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2016-07-22T15:19:13.510288415Z",
        "Container": "9b5da12722b13c386447977fffcd067a0658f44e4b0838626b2e7b2da531801c",
        "ContainerConfig": {
            "Hostname": "ca9015dc6bb1",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) CMD [\"/bin/bash\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:8677cced8174b061771aa11eb3874d7eaaf26efceec04dac02c9d1a788fd3064",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "1.10.3",
        "Author": "",
        "Config": {
            "Hostname": "ca9015dc6bb1",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:8677cced8174b061771aa11eb3874d7eaaf26efceec04dac02c9d1a788fd3064",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 137205221,
        "VirtualSize": 137205221,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/693ccc37b2f6feb6b4828e6a8ec49966b2e44c8cdbfc63868f0e6364cbaa771e/diff:/var/lib/docker/overlay2/106031ccc632386fbb43678a1fdaf74e05c3db19c2b9e1f7a378a723b69f8c20/diff:/var/lib/docker/overlay2/95a0488893bcf2c5455b793f118c97a439eff064912f5225399b8931392fdd3f/diff",
                "MergedDir": "/var/lib/docker/overlay2/b3d0220e706b5c87b5d7b2641ff7d487247562e14fe830387e98210d95fec0c8/merged",
                "UpperDir": "/var/lib/docker/overlay2/b3d0220e706b5c87b5d7b2641ff7d487247562e14fe830387e98210d95fec0c8/diff",
                "WorkDir": "/var/lib/docker/overlay2/b3d0220e706b5c87b5d7b2641ff7d487247562e14fe830387e98210d95fec0c8/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:f121afdbbd5dd49d4a88c402b1a1a4dca39c9ae75ed7f80a29ffd9739fc680a7",
                "sha256:4b955941a4d0bfa4d3eed9ab5cf8e03479ece47a3f0c154439e93230b08a8048",
                "sha256:af288f00b8a7386eadb431dddd75e0f75f129994e89cfa424a00cfc9c8a89c95",
                "sha256:98d59071f692a7a8b02acfea340f4e63b8801d8914812df05334e4b264de2fdb"
            ]
        },
        "Metadata": {
            "LastTagTime": "2020-12-25T06:21:06.0150948Z"
        }
    }
]
```

`docker inspect -f {{".Architecture"}} ubuntu:15.10`查看其中一项

![image-20201225143517410](./images/image-20201225143517410.png)

## 搜寻镜像

`docker search --filter=is-automated=true --filter=stars=3 nginx`搜寻自动创建评价为3+的带nginx的关键字的镜像

## 删除镜像

`docker rmi myubuntu:x`一个镜像拥有多个标签的时候，只是删除指定标签而已，并不影响镜像文件，当只剩一个镜像的时候，会彻底删除镜像，使用镜像ID删除镜像时，会先尝试删除所有标签，最后删除镜像本身。当一个镜像创建的容器存在时，镜像文件是默认是无法删除的，可以使用`docker rmi -f myubuntu:x`强制删除。

## 创建镜像

### 基于已有的镜像的容器创建

`docker commit -m "add a new file" -a "Docker Newbee" df2b7a7478d8 test:0.1`

<img src="./images/image-20201225161252397.png" alt="image-20201225161252397" style="zoom:50%;" />

### 基于 本地模板的导入

`cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:14.04 `

## 存出和载入镜像

`docker save -o test.tar test:0.1`存出镜像

`docker load --input test.tar`导入镜像

## 上传镜像

` docker tag test:0.1 用户名/test:push`给镜像打标签，必须是自己的用户名

`docker push 用户名/test:push`上传

## 创建容器

`docker create -it myubuntu:x`创建容器

`docker ps -a  `查看正在运行的容器

![image-20201225163649863](./images/image-20201225163649863.png)

`docker start serene_bhaskara`启动一个容器

`docker run ubuntu:15.10 echo 'hello world'`新建并启动容器

### 守护态运行

`docker run -d  /bin/sh -c "while true; do echo hello world; sleep 1; done"`守护态运行

`docker logs confident_elion`查看容器运行日志

## 终止容器

`docker stop confident_elion`等待一段时间（默认10秒）后杀死

`docker restart confident_elion`重启一个容器

## 进入一个容器

` docker exec -it e2c41beb47d6 /bin/bash`进入一个容器

## 删除一个容器

`docker rm e2c41beb47d6`删除，默认只能删除终止或者退出状态的容器，并不能删除处于运行状态的容器，如果要删除需要添加的-f参数。

## 导出容器

`docker export -o /Users/wyj/Desktop/test.tar jovial_wiles`导出容器

`docker import test.tar - test/ubuntu:v1.0`导入容器编程一个镜像

实际上，既可以使用docker load命令来导入镜像存储文件到本地镜像库，也可以使用docker import命令来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也更大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。”

## Docker数据管理

### 挂载到宿主容器

`docker run -it -v /Users/wyj/Desktop/testx:/testx myubuntu:x /bin/bash`冒号前边的是本地的路径，冒号后边的是容器内的路径

`docker run -it -v /Users/wyj/Desktop/testx:/testx:ro myubuntu:x /bin/bash`挂载的数据卷默认是读写，用户可以通过ro指定为只读

### 数据卷容器

`docker run -it -v /dbdata --name dbdata ubuntu`创建一个数据卷容器，并在其中创建一个数据卷挂载到/dbdata

`$ docker run -it --volumes-from dbdata --name db1 ubuntu`db1,db2挂载到/dbdata上
`$ docker run -it --volumes-from dbdata --name db2 ubuntu`

## 端口映射与容器互联

### 从外部访问容器应用

`docker run -d --name nginx_1 -P nginx:latest `从外部访问容器应用,大P随机映射，小p可以指定要映射的端口

![image-20201225201132893](./images/image-20201225201132893.png)

使用localhost:32768访问

### 映射所有接口地址

使用HostPort:ContainerPort格式将本地的5000端口映射到容器的5000端口：

`docker run -itd -p 5000:5000 --name nginx_2 nginx:latest`

此时默认会绑定本地所有接口上的所有地址。多次使用-p参数可以绑定多个端口：

`docker run -itd -p 3000:2700 -p 2389:8863 --name nginx_3 nginx:latest`

### 映射到指定地址的指定端口

使用IP:HostPort:ContainerPort格式指定映射使用一个特定地址：

`docker run -itd -p 10.0.0.31:89:8081 --name nginx_4 nginx:latest `

### 映射到指定地址的任意端口

使用IP::ContainerPort格式绑定本机的任意端口到容器的指定端口：

`docker run -itd -p 10.0.0.31::8082 --name nginx_5 nginx:latest`

### 使用docker port命令来查看当前映射的端口配置，也可以查看绑定的地址

### 容器互联

使用--link参数可以让容器之间安全地进行交互。

创建一个数据库容器：`docker run -itd --name db --env MYSQL_ROOT_PASSWORD=example  mariadb`

创建一个web容器并将它连接到db容器：`docker run -itd -P --name web --link db:db nginx:latest `

## 使用DockerCompose配置启动Docker

```yml
version: "2.1"
services:
  jobmanager: #服务名称
    image: flink #镜像名称
    expose:
      - "6123" #指定暴露的端口，但是只是作为一种参考
    ports:
      - "8081:8081" #使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口，会绑定所有地址
    command: jobmanager #覆盖容器启动后默认执行的命令
    environment:  #设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: flink
    expose:
      - "6121"
      - "6122"
    depends_on: #启动顺序,要先启动jobmanager
      - jobmanager
    command: taskmanager
    links:  #连接到jobmanager，使用的别名将会自动在服务容器中的/etc/hosts里创建
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    links:  #连接到jobmanager，使用的别名将会自动在服务容器中的/etc/hosts里创建
      - "jobmanager:jobmanager"

  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    links:
      - "jobmanager:jobmanager"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://:9092
      KAFKA_LISTENERS: PLAINTEXT://:9092
```

使用docker-compose up`命令启动

