# terminology/curd 目录教程

- `terminology.py`：术语 CRUD、向量化、相似检索、`get_terminology_template` 组装 prompt；也承载部分自定义提示词检索逻辑。
- `__init__.py`：包声明。

召回时会考虑工作空间、数据源、助手范围，不能只按文本相似度跨租户检索。

## 关键查询路径

管理查询使用 build/execute 分层组合分页、名称、数据源过滤；召回查询有普通工作空间、指定数据源、高级应用三套 SQL；`select_terminology_by_word` 返回命中实体；`get_terminology_template` 转 XML 文本并同时返回命中列表，供 prompt 和执行日志使用。

`pid` 可表达术语组/父子关系。更新父子或适用范围时要防循环和跨工作空间引用。保存 embedding 可能在线程中批量执行，需要独立 Session。

