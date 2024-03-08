# 常用插件

- MybatisX: 提供自动代码生成以及xml与interface之间的接口跳转
  
  > 相关的使用方式可以参考 http://t.zoukankan.com/mingforyou-p-14662566.html

- Docker: 提供对Docker的支持

- Vue.sj: 提供对vue的支持

- 其他语言支持：Python提供对Python的支持，Scala提供对Scala的支持，Go提供对Go的支持

- 代码质量的：Alibaba Java Coding Guidelines、SonarLint
  
  > 这两个插件都可以项目右键上打开或关闭实时监测

- 代码行数检测：Statistic

- json与bean互转：GsonFormat-plus、java Bean to Json

- Grep console:控制台输出不同日志级别的颜色

- sequenceDiagram：时序图

- redis simple: 提供redis读取

- Maven helper: 提供maven的可视化操作

# 激活

- 使用插件无限使用，但是注意使用2021.2.2以下版本，以上的需要登录才能使用

- 过期以后使用下面方法
  
  ```
  IntelliJIdea
  
  1.打开目录
  /Users/**/Library/Application Support/JetBrains/IntelliJIdea2021.2/eval
  删除一下文件就行
  rm -f idea212.evaluation.key
  
  同理：
  PyCharm
  /Users/**/Library/Application Support/JetBrains/PyCharm2021.2/eval
  rm -f PyCharm212.evaluation.key
  ```

# 快捷键

- 格式化所有代码：option+command+L
- 注销单行：command+/
- 注销多行：option+command+/+
- 查看接口实现
  - control+h，会跳出所有的
  - 点接口左边的按钮
- option+shift,然后鼠标左键可以多行编辑
- 选择代码环绕方式：option+command+T
- 回到上次的位置，command+[，回到下次的位置command+]

# 常用设置

- 模版输入
  
  搜索Live Templates，然后可以在其中新建组或者新建模板，可以在Abbreviation设置快捷键，可以设置默认的触发键在上边的default expand，也可以在下边为expand with为模版设定独立的触发键。以设置自动填写文件作者信息为例：
  
  ```properties
  1. 新建一个组为Personal，在下边建一个qqq
  2. 填上如下内容，设置快捷键qqq、触发键tab
  /**
  @author wyj
  */
  3. 在代码页可以直接通过qqq+tab加入如上代码
  ```

# 断点调试

- step over 下一步，不进入方法

- step into 下一步，如果是方法进入方法

- force step into 能够进入所有的方法，包括jdk的方法

- step out跳出

- resume program 恢复程序运行，但是如果接下来还有断点，停在接下来的断点上

- stop 直接停止程序

- mute breakpoints 所有断点失效

- view breakpoints 查看所有断点

- Run to Cursor 运行到光标处，你可以将光标定位到你需要查看的那一行，然后使用这个功能，代码会运行至光标行，而不需要打断点

- Evaluate Expression 计算表达式
  
  > 比如：对于代码中的一个数组x，想计算下数组的长度+3是多少，就可以在上边输入len(x)+3，好处是不用再打印值了，可以针对变量做一些简单操作。更广泛的应用是：
  > 
  > - 当你的一行代码中调用了几个方法时，就可以通过这种方式查看查看某个方法的返回值
  > 
  > - 在计算表达式的框里，可以改变变量的值，这样有时候就能很方便我们去调试各种值的情况

- smart step into 一行代码里有好几个方法自动定位到当前断点行，并列出需要进入的方法，选择其中一个进入

- 断点条件设置 在断点上右键加入条件
  
  > 在遍历一个比较大的集合或数组时常用
  > 
  > 比如：对于如下代码，在第4行代码设置断点，可以设置v==3为condition，只有执行到v等于3才会执行
  > 
  > ```go
  > func main() {
  >     var x = []int64{1, 2, 3, 4, 5}
  >     for _, v := range x {
  >         fmt.Println(v)
  >     }
  > }
  > ```
  
  回退断点、多线程调试等更多用法[参考这里](https://www.pdai.tech/md/java/jvm/java-jvm-debug-idea.html)

# 重构

- extract方法
- introduce抽取常量、变量等
- rename重命名类、方法、子段

# 可视化的Git

- copy revision number
  
  > 当前的提交id

- create patch
  
  > 记录下当前的提交影响的文件

- cherry pick
  
  > 将某几个提交应用到分支，比如将f2的分支上的某个提交应用到master

- show repository at revision
  
  > 当时工程的状态

- compare with local
  
  > 和本地目前状态对比

- reset current branch to here
  
  > 回退

- Revert commit
  
  > 重做某一次提交，相当于把本次提交的都删了做一次提交

- undo commit
  
  > 重做上一次提交

- rebase current onto selected
  
  > 当前分支f1, 选中分支f2
  > 
  > 该操作会把f2的内容放到f1的前面形成一个新的f1，可以强制push新的分f1

- merge selected into current
  
  > 当前分支f1, 选中分支f2
  > 
  > 该操作会把f2的内容merge到f1上同时产生一次合并记录
