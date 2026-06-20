# `db.py` 多数据库执行教程

## 统一输入输出

`exec_sql(ds, sql, origin_column=False)` 接收数据源配置与 SQL，统一返回：

```python
{"fields": ["column"], "data": [{"column": "value"}], "sql": "base64..."}
```

字段默认小写，方便与模型图表配置对齐；不同驱动值经 `convert_value` 统一为可 JSON 序列化类型。

## 安全门

`check_sql_read` 先检查首关键字与危险正则，再用 sqlglot 按数据库方言解析 AST，拒绝写语句和危险函数。默认仅允许 SELECT/WITH；只有显式开启 `SQLBOT_ALLOW_METADATA_QUERIES` 才允许 SHOW/DESCRIBE/DESC/EXPLAIN。

## 执行分支

常规数据库走 SQLAlchemy session；达梦、Doris/StarRocks、Redshift、Kingbase、Hive 等使用专用驱动；Elasticsearch 走 SQL HTTP。所有分支应使用上下文管理器关闭连接，并把结果解析异常转换为 `ParseSQLResultError`。

## 新增数据库步骤

更新 `constant.py::DB`、连接 URL/engine、元数据 SQL、`exec_sql` 分支或 SQLAlchemy 方言、sqlglot dialect、危险函数列表、依赖、`templates/sql_examples` 方言模板，并补连接/只读/类型转换测试。

