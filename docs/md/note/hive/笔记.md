- 提取json数组中的所有相同字段
> instr(get_json_object(call_back,'$.history[*].nluIntent'),'ruma'),但是这个要用Spark引擎(stable.level=5)

- 聚类时非聚类字段处理方式
> 查询非聚类字段，可以使用`first()`或者`collect_set(template_name)[0]`来处理