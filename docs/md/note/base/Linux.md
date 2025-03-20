# 操作系统

https://mp.weixin.qq.com/s/5S0swlCe-Gx66aO3Lrpylg

https://mp.weixin.qq.com/s?__biz=MzkwODE5ODM0Ng==&mid=2247488406&idx=1&sn=93e2435b319c42497a4efa966ddc9237&chksm=c0ccfb33f7bb7225e458b84a8347f19a6dffaf9f38ad76c4b849e645b9543ee93e89b750eea6&scene=178&cur_album_id=2041709347461709827#rd

# linux

## 文件系统

- 什么是文件块？
  
  把硬盘按照扇区几个组合在一起就屏蔽了底层逻辑，操作系统以块为单位进行操作就可以了

- 为什么要有块位图？
  
  存取时要知道哪些块被占了，哪些没被占，弄个位图与块的映射查这个位图就好，文件存入时顺带把块对应的块位图位置设置为1

- 怎么读取存入的文件？
  
  要记录文件名与块的映射关系，输入文件名然后查找块的位置，最后去块里边拿，既然要记录名称那就顺带把文件大小、创建时间、文件权限等等都记录上，这些文件元信息的组合叫做inode。存储时要记录inode、改变块位图以及实际存入块

- 为什么要有inode位图？
  
  与块位图的作用一样，记录哪些inode被使用了

- 什么是inode表？
  
  纯粹用来存放inode信息的块

- 文件占用多个块怎么办？
  
  两种办法，一种是连续存储，inode里记录块的起始位置以及长度，但是容易引起文件空洞；另一种是inode里记录所有存储的块

- 大文件有很多个块怎么办,inode里记录不了那么多块怎么办？
  
  把记录块的最后一个信息存储一个二级索引指向一个块，指向的块继续存储与块的对应关系，不行就再存储一个三级索引

- inode、块的数量不够时怎么办？
  
  需要有个超级块记录已用的数量、剩余的数量

- 块位图、inode位图、inode表要动态扩展怎么办？
  
  在超级块后边加一个块描述符，可以记录块位图、inode位图、inode表分别都可以占用哪几个块

- 怎么区分目录文件与普通文件？
  
  在inode里边加一个字段区分文件类型，如果是目录文件，inode指向的数据块里存的都是指向inode表里的映射关系

- 如果想用ls查看目录下的所有文件怎么办？
  
  如果一个个遍历存取目录下的inode指向的inode表很慢，不如直接把这些常用的直接放到inode指向的数据块里存储的映射里，少了很多次再查询，既然文件名、文件类型放到了映射里那么inode再存储文件名也没什么用了，直接删了就行。

- 最上层的目录即根目录，怎么读文件？
  
  为了避免最上层的目录还要遍历，可以规定inode表的0号表示根目录

- ls ls-i ls -l区别？
  
  ls只是定位一次，然后遍历目录对应的块中的映射K
  
  ls -i也是只定位一次，但是拿到的是映射的KV
  
  ls -l要定位多次,首先定位到映射的块，然后根据映射一次拿inode的信息

- 目录的权限
  
  目录的权限rwx，如果只有r那就到一次定位的映射那里，要进一步寻找inode信息就要有x权限

- 由于inode一些特性，会有如下一些特性
  
  - find . -inum 57244606 -exec rm {} \;可以直接删除文件
  - mv没有真的移动文件，只是改变文件名，不影响inode号码，跨目录移动时只是将映射移到另一个目录下并在本目录映射删除

- 什么是硬链接？
  
  - 多个文件名指向同一个inode号码，inode信息中有一项叫做链接数记录指向该inode的文件名总数，当这个数据为0时系统就回收这个inode号码以及对应的block区域
  
  - 和..其实也是两个目录，一个的inode指向当前目录的inode号码，一个指向其父目录的inode号码，所以一个目录的硬链接等于2加上它的子目录的总数

- 什么是软链接？
  
  - 文件 A 和文件 B 的 inode 号码虽然不一样，但是文件 A 的内容是文件 B 的路径。读取文件 A 时，系统会自动将访问者导向文件 B。因此，无论打开哪一个文件，最终读取的都是文件 B。这时，文件 A 就称为文件 B 的"软链接"。这意味着，文件 A 依赖于文件 B 而存在，如果删除了文件 B，打开文件 A 就会报错："No such file or directory"。这是软链接与硬链接最大的不同：文件 A 指向文件 B 的文件名，而不是文件 B 的 inode 号码，文件 B 的 inode "链接数"不会因此发生变化。
  
  > 使用zsh，在home文件下的.zshrc配置alias可以简化命令

## 常用命令

1. 查看占用了8080端口号的进程`lsof -i:8080`

2. 查看某个进程ps -aux | grep python

3. 查看当前在哪个目录下`pwd`

4. `chmod a+x `文件 赋予文件可执行权限

5. 文件追加`cat 1.txt>>2.tx`

6. 文件覆盖`cat 1.txt>2.txt`

7. 文件清空`cat /dev/null>2.txt `

8. 查看某个关键字前几行后几行cat filename|grep '关键字' -A4(后四行) -B4(前四行)

9. 查看已删除空间却没有释放的进程`lsof -n / |grep deleted `

10. 在标准unix/linux下的grep命令中，通过以下参数控制上下文的显示
    
    grep -C 10 keyword catalina.out 显示file文件中匹配keyword字串那行以及上下10行
    
    grep -B 10 keyword catalina.out 显示keyword及前10行
    
    grep -A 10 keyword catalina.out 显示keyword及后10行

11. sed
    
    ```shell
    sed -e "s/^/'/" -e "s/$/',/" temp.txt  # 在文件的每一行行首加上单引号，行尾加上单引号与逗号
    ```

12. awk
    
    ```shell
    awk '{print $1,$4}' temp.txt # 输出文件每行的第一项与第四项
    awk '/2023-07-25T17:26:15.805*/,/2023-07-25T17:26:15.807*/'log.txt #输出固定时间段的日志
    ```

13. tail与grep连用查看日志
    
    ```java
    tail -f catalina.out | grep --line-buffer "发送邮件" //只打印满足条件的，如果需要固定时间段的与awk使用管道符连接共用
    tail -f sss.txt | grep --line-buffer -E '12345|nbsx' //满足任意一个条件
    tail -f sss.txt | grep --line-buffer '12345' | grep --line-buffer 'nbsz'//同时满足两个条件
    ```

## Shell

统计文件夹下文件总共有多少行

```shell
#!/bin/bash 

filesCount=0
linesCount=0
function funCount()
{
    for file in ` ls $1 `
    do
        if [ -d $1"/"$file ];then
            funCount $1"/"$file
        else
            declare -i fileLines
            fileLines=`sed -n '$=' $1"/"$file`
            let linesCount=$linesCount+$fileLines
            let filesCount=$filesCount+1
        fi
    done
}

if [ $# -gt 0 ];then
    for m_dir in $@
    do
        funCount $m_dir
    done
else
    funCount "."
fi
echo "filesCount = $filesCount"
echo "linesCount = $linesCount"
```

一段初始化JVM的脚本

```shell
#!/usr/bin/env bash

function init() {
    APP_KEY="com.sankuai.scoai.pallas"

    if [ -z "$LOG_PATH" ]; then  #如果LOG_PATH长度为0，拼接LOG_PATH
        LOG_PATH="/opt/logs/$APP_KEY"
    fi

    mkdir -p $LOG_PATH  #建立目录

    if [ -z "$WORK_PATH" ]; then  #如果WORK_PATH长度为0，拼接WORK_PATH
        WORK_PATH="/opt/meituan/$APP_KEY"
    fi

    JAVA_CMD="java"
    if ! command -v $JAVA_CMD >/dev/null 2>&1; then #如果不支持直接执行java命令，拼接JAVA_CMD
        JAVA_CMD="/usr/local/$JAVA_CMD/bin/java"
    fi

    JVM_ARGS="-server -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Djava.net.preferIPv6Addresses=false"
    #-server 服务器模式
    #-Dfile.encoding java文件编码
    #-Dsun.jnu.encoding 操作系统默认编码

    if [ -z "$JVM_GC" ]; then
        JVM_GC="-XX:+UseG1GC -XX:G1HeapRegionSize=4M -XX:InitiatingHeapOccupancyPercent=40 -XX:MaxGCPauseMillis=100 -XX:+TieredCompilation -XX:CICompilerCount=4 -XX:-UseBiasedLocking -Xlog:gc*:$LOG_PATH/gc.log:time,uptime:filecount=20,filesize=50M"
    #-XX:+UseG1GC 使用G1垃圾回收器
    #-XX:G1HeapRegionSize=4M 堆内存中一个Region的大小可以通过-XX:G1HeapRegionSize参数指定，大小区间只能是1M、2M、4M、8M、16M和32M
    #-XX:InitiatingHeapOccupancyPercent 当老年代大小占整个堆大小百分比达到该阈值时，会触发一次mixed gc(当越来越多的对象晋升到老年代old region时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即mixed gc，该算法并不是一个old gc，除了回收整个young region，还会回收一部分的old region，这里需要注意：是一部分老年代，而不是全部老年代，可以选择哪些old region进行收集，从而可以对垃圾回收的耗时时间进行控制
    #-XX:MaxGCPauseMillis=100     设置G1收集过程目标时间
    #-XX:+TieredCompilation  这个参数主要用于是否开启JVM的分层编译
    #-XX:CICompilerCount 最大并行编译数
    #-XX:-UseBiasedLocking 不使用偏向锁
    #--Xlog:gc*:$LOG_PATH/gc.log:time,uptime:filecount=20,filesize=50M 指明GC日志存放位置[格式采用时间：到现在系统运行时间]，保留20个文件，每50M就轮换

    fi

    if [ -z "$JVM_EXT_ARGS" ]; then
        JVM_EXT_ARGS=""
    fi

    if [ -z "$JVM_HEAP" ]; then
        JVM_HEAP=`getJVMMemSizeOpt`
    fi
}


function run() {
    EXEC="exec"
    CONTEXT=/
    EXEC_JAVA="$EXEC $JAVA_CMD $JVM_ARGS $JVM_EXT_ARGS $JVM_HEAP $JVM_GC \
    -XX:ErrorFile=$LOG_PATH/vmerr.log \
    -XX:HeapDumpPath=$LOG_PATH/HeapDump"
    # -XX:ErrorFile系统奔溃时的日志
    # -XX:HeapDumpPath 堆快照路径

    if [ "$UID" = "0" ]; then
        ulimit -n 1024000 # 修改最大连接数
        umask 000 # 去掉权限
    else
        echo $EXEC_JAVA 
    fi
    cd $WORK_PATH
    pwd
    targetPackage=`find . -maxdepth 1 -type f \( -name "*.jar" -o -name "*.war" \)`
    #找到当前文件夹中的深度为1的类型为文件的name为jar或war的

    env=`getEnv`
    SPRING_ENV=""
    if [ -n "$env" ]; then
        SPRING_ENV="--spring.profiles.active=$env"
    fi

    echo "this target jar will be executed: "$targetPackage
    $EXEC_JAVA -jar $targetPackage $SPRING_ENV 2>&1
}

function getTotalMemSizeMb() {
    memsizeKb=`cat /proc/meminfo|grep MemTotal|awk '{print $2}'`
    if [ -z "$memsizeKb" ]; then
        memsizeKb=8*1000*1000
    fi
    memsizeMb=$(( $memsizeKb/1024 ))
    echo $memsizeMb
}

function outputJvmArgs() {
    jvmSize=$1
    MaxMetaspaceSize=$2
    ReservedCodeCacheSize=$3
    echo "-Xss512k -Xmx"$jvmSize" -Xms"$jvmSize" -XX:MetaspaceSize="$MaxMetaspaceSize" -XX:MaxMetaspaceSize="$MaxMetaspaceSize" -XX:+AlwaysPreTouch -XX:ReservedCodeCacheSize="$ReservedCodeCacheSize" -XX:+HeapDumpOnOutOfMemoryError "
}
# -Xss设置每个线程的堆栈大小
# -Xmx JVM最大可用内存
# -Xms JVM初始内存
# -XX:MetaspaceSize  设置metaspace区域的最大值
# -XX:+AlwaysPreTouch 启动的时候真实的分配物理内存给jvm
# -XX:ReservedCodeCacheSize 设置Code Cache大小，JIT编译的代码都放在Code Cache中，若Code Cache空间不足则JIT无法继续编译，并且会去优化，比如编译执行改为解释执行，由此，性能会降低
# -XX:+HeapDumpOnOutOfMemoryError 当堆内存空间溢出时输出堆的内存快照


function getJVMMemSizeOpt() {
    memsizeMb=`getTotalMemSizeMb`

    #公司的机器内存比实际标的数字要小，比如8G实际是7900M左右，一般误差小于1G
    #内存分级，单位兆/M
    let maxSize_lvl1=63*1024
    let maxSize_lvl2=31*1024
    let maxSize_lvl3=21*1024
    let maxSize_lvl4=15*1024
    let maxSize_lvl5=7*1024
    let maxSize_lvl6=3*1024
    let maxSize_lvl7=1024
    let maxSize_lvl8=512

    if [[ $memsizeMb -gt $maxSize_lvl1 ]]
    then
        jvmSize="32g"
        MaxMetaspaceSize="2g"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl2 && $memsizeMb -le $maxSize_lvl1 ]]
    then
        jvmSize="24g"
        MaxMetaspaceSize="1g"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl3 && $memsizeMb -le $maxSize_lvl2 ]]
    then
        jvmSize="18g"
        MaxMetaspaceSize="512m"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl4 && $memsizeMb -le $maxSize_lvl3 ]]
    then
        jvmSize="12g"
        MaxMetaspaceSize="512m"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl5 && $memsizeMb -le $maxSize_lvl4 ]]
    then
        jvmSize="4g"
        MaxMetaspaceSize="512m"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl6 && $memsizeMb -le $maxSize_lvl5 ]]
    then
        jvmSize="2g"
        MaxMetaspaceSize="256m"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl7 && $memsizeMb -le $maxSize_lvl6 ]]
    then
        jvmSize="1g"
        MaxMetaspaceSize="256m"
        ReservedCodeCacheSize="240m"
    fi

    if [[ $memsizeMb -gt $maxSize_lvl8 && $memsizeMb -le $maxSize_lvl7 ]]
    then
        jvmSize="512m"
        MaxMetaspaceSize="256m"
        ReservedCodeCacheSize="240m"
    fi

    if [ $memsizeMb -le $maxSize_lvl8 ]; then
        echo "service start fail:not enough memory for MDP service"
        exit 1
    fi
    outputJvmArgs $jvmSize $MaxMetaspaceSize $ReservedCodeCacheSize
    exit 0
}

function getEnv(){
    FILE_NAME="/data/webapps/appenv"
    PROP_KEY="env"
    PROP_VALUE=""
    if [[ -f "$FILE_NAME" ]]; then
        PROP_VALUE=`cat ${FILE_NAME} | grep -w ${PROP_KEY} | cut -d'=' -f2`
    fi
    echo $PROP_VALUE
}

init
run
```

## 问题排查

磁盘空间不足时：

1. 使用df -h整体查看磁盘
2. 使用du -sh * 查看当前路径下的各个文件和目录的大小
3. 使用ls -lh命令查看文件占地大小

CPU与内存使用过高

1. top命令，使用q退出
2. 如果只看java使用jps找到PID，然后使用top -p 19063专门查看某个java进程的情况，如果要细化到线程可以加个top -p 19063 -H参数
3. 如果想单独分析内存，使用free

网络延迟

1. netstat -a查看所有连接中的socket, netstat -tnpa命令可以查看所有 tcp 连接的信息，包括进程号
2. 使用ps -ef命令查看进程相关信息
