查看某个包是怎么引入的

```shell
#查看某个包是怎么引入的
dependency:tree -Dverbose -Dincludes=org.mybatis:mybatis
```

- **dependenciesManagement**写在父pom里，只会标记版本号，子pom不会继承，子pom引入的时候不需要version，会循着父pom逐级往上找寻找到version。

- Maven的snapshot版本，主要是为了两方同时开发，不同于常规的版本，Maven 每次构建都会在远程仓库中检查新的快照。 版本如果下载到了本地，如果不更新版本号，是不会去远程仓库寻找新的jar包的，只会在本地找。

要想发布，需要在代码中加上：

```xml
<distributionManagement>
        <repository>
            <id>meituan-nexus-releases</id>
            <name>Meituan Nexus Releases Repository</name>
            <url>http://XXX/content/repositories/releases/</url>
        </repository>

        <snapshotRepository>
            <id>meituan-nexus-snapshots</id>
            <name>Meituan Nexus Snapshots Repository</name>
            <url>http://XXX/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
</distributionManagement>
```

- 版本仲裁

  > - 最短路径优先原则
  > - 相同路径优先声明原则
  > - 显示指定优先原则

- Scope

  > | 周期         | 阶段             | 是否打包       | 举例        |
  > | ------------ | ---------------- | -------------- | ----------- |
  > | **compile**  | 编译、测试、运行 | 不参与打包     | 常见        |
  > | **test**     | 测试             | 不参与项目打包 | JUnit       |
  > | **provided** | 编译、测试       | 不参与项目打包 | servlet-api |
  > | **runtime**  | 测试、运行       | 参与打包       | JDBC驱动    |
