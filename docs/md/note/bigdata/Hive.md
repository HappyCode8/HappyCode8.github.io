## 常用函数

- 提取json数组中的所有相同字段
  
  > instr(get_json_object(call_back,'$.history[*].nluIntent'),'ruma'),但是这个要用Spark引擎(stable.level=5)

- 聚类时非聚类字段处理方式
  
  > 查询非聚类字段，可以使用`first()`或者`collect_set(template_name)[0]`来处理

- 提取某个字符串出现两次
  
  > `get_json_object(call_back, '$.history[*].source') REGEXP 'user.*user'`

- 取第一个非空字段
  
  > nvl

## 开窗函数

开窗函数的调用格式为：函数名(列名) OVER(partition by 列名 order by 列名 范围) 

范围

> - current row：当前行
> - n preceding：往前n行数据
> - n following：往后n行数据
> - unbounded：起点：unbounded preceding 表示从前面的起点开始，unbounded following表示到后面的终点结束

函数名

> - row_number()：对相等的值不进行区分，相等的值对应的排名相同，序号从1到n连续。
> - rank()：相等的值排名相同，但若有相等的值，则序号从1到n不连续。如果有两个人都排在第3名，则没有第4名。
> - dense_rank() ：对相等的值排名相同，但序号从1到n连续。如果有两个人都排在第一名，则排在第2名（假设仅有1个第二名）的人是第3个人。
> - count() over(partition by … order by …) 计数
> - max() over(partition by … order by …) 最大
> - min() over(partition by … order by …) 最小
> - sum() over(partition by … order by …) 求和
> - avg() over(partition by … order by …) 求平均
> - first_value() over(partition by … order by …) 第一行
> - last_value() over(partition by … order by …) 最后一行
> - lag(col,n) over(partition by … order by …) 向前第n行
> - lead(col,n) over(partition by … order by …) 向后第n行
> - ntile(n)：可以看作是把有序的数据集合平均分配到指定的数量n的桶中,将桶号分配给每一行，排序对应的数字为桶号。如果不能平均分配，则较小桶号的桶分配额外的行，并且各个桶中能放的数据条数最多相差1，这个函数一半用来获得前百分之几十的数据，比如切3部分获取第一份就是33%

## 安装

> [安装hadoop](https://blog.csdn.net/qq_20042935/article/details/123007927)
> 
> [安装hive](https://blog.csdn.net/qq_20042935/article/details/123046044)
> 
> 启停注意点：
> 
> 1. 要格式化文件系统
> 
> 2. cd /usr/local/Cellar/hadoop/3.3.1/libexec/sbin ./start-all.sh
