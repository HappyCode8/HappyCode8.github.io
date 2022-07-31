# 常用插件

- MybatisX: 提供自动代码生成以及xml与interface之间的接口跳转

  > 相关的使用方式可以参考 http://t.zoukankan.com/mingforyou-p-14662566.html

- Docker: 提供对Docker的支持

- Vue.sj: 提供对vue的支持

- 其他语言支持：Python提供对Python的支持，Scala提供对Scala的支持，Go提供对Go的支持

- 代码质量的：Alibaba Java Coding Guidelines、SonarLint

- json与bean互转：GsonFormat-plus、java Bean to Json

- Grep console:控制台输出不同日志级别的颜色

- sequenceDiagram：时序图

- redis simple: 提供redis读取

- Maven helper: 提供maven的可视化操作

# 激活

- 使用插件无限使用，但是注意使用2021.2.2以下版本，以上的需要登录才能使用

# 快捷键

- 格式化所有代码：option+command+L
- 注销单行：command+/
- 注销多行：option+command+/+
- 查看接口实现
  - control+h，会跳出所有的
  - 点接口左边的按钮
- option+shift,然后鼠标左键可以多行编辑
- 选择代码环绕方式：option+command+T

![images](https://s2.loli.net/2022/06/12/Ez8tAfLhbWQV7Nm.png)

# 断点调试

- step over下一步，不进入方法

- step into下一步，如果是方法进入方法

- force step into能够进入所有的方法，包括jdk的方法

- step out跳出

- resume program恢复程序运行，但是如果接下来还有断点，停在接下来的断点上

- stop直接停止程序

- mute breakpoints所有断点失效

- view breakpoints查看所有断点

  更多用法[参考这里](https://www.pdai.tech/md/java/jvm/java-jvm-debug-idea.html)

# 重构

- extract方法
- introduce抽取常量、变量等
- rename重命名类、方法、子段

# 可视化的Git

- copy revision number

  >当前的提交id

- create patch

  >记录下当前的提交影响的文件

- cherry pick

  >将某几个提交应用到分支，比如将f2的分支上的某个提交应用到master

- show repository at revision

  >当时工程的状态

- compare with local

  >和本地目前状态对比

- reset current branch to here

  >回退

- Revert commit

  >重做某一次提交，相当于把本次提交的都删了做一次提交

- undo commit

  >重做上一次提交

- rebase current onto selected

  >当前分支f1, 选中分支f2
  >
  >该操作会把f2的内容放到f1的前面形成一个新的f1，可以强制push新的分f1

- merge selected into current

  >当前分支f1, 选中分支f2
  >
  >该操作会把f2的内容merge到f1上同时产生一次合并记录

# 

