## 概念

工作区（代码）---->暂存区（add以后）---->版本库（本地仓库commit）

## 常用命令

- `git add .`  将文件交给git管理
- `git commit -m "提交注释"`   提交本地
- `git push origin  分支名称 `  推送远程

## 已push的多次提交合并为一次

```properties
git rebase -i HEAD~3将最近的三次合并为一次，然后得到如下信息：

pick 25ac405 test3
pick 9fa83dd first
pick b58e272 third
....

将9fa, b58的pick字段改为f或者s，然后执行
git rebase --continue
git push -f
最后再合并到主分支上
```

也可以使用IDEA的图形化界面,在IDEA上

```properties
- 选中分支的提交记录右键
- Interactively rebase from here（开始交互式变基）
- 把第一次以后的都改为fixup（抛弃这次的提交记录）或者squash（与以前的提交记录可以合并，在idea中可以重写以前的提交记录），也可以reword提交记录，然后push
```

- ## 打tag

`git tag v1.0 dd18e8`将v1.0标签打到dd18e8分支上

`git push origin v1.0`推送标签

`git tag -d v1.0`删除本地标签

`git push origin --delete v1.0`删除远程标签

## 全局设置账号与邮箱

`git config --global user.name "git_ai_ivr"`

`git config --global user.email "git_ai_ivr@meituan.com"`
