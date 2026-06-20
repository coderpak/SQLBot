# db 目录教程

## 作用

对多种业务数据库做统一适配。这里连接的是用户配置的业务数据源，不是 SQLBot 自身的 PostgreSQL 系统库。

## 文件

- `constant.py`：`DB` 枚举、数据库类型、模板名、连接方式等元数据映射。
- `engine.py`：SQLAlchemy engine/URL 构建和不同方言驱动适配。
- `db.py`：连接测试、元数据读取、样例查询、`exec_sql`、值转换和只读 SQL 安全检查。
- `db_sql.py`：不同数据库需要的元数据 SQL/辅助 SQL。
- `es_engine.py`：Elasticsearch SQL HTTP 适配。
- `__init__.py`：包声明。

`exec_sql` 在执行前调用 `check_sql_read`，随后按 SQLAlchemy、达梦、Doris/StarRocks、Redshift、Kingbase、ES、Hive 等分支执行并统一返回 `{fields, data, sql}`。

## 与 `common/core/db.py` 的区别

`common/core/db.py` 只连接 SQLBot 自身系统库；本目录连接用户配置的业务数据源。前者由 FastAPI SessionDep 管理，后者根据每个 `CoreDatasource` 动态创建连接。看到 `get_session` 时必须先确认它来自哪个模块。

## 只读安全链

`check_sql_read` 先检查首关键字，再匹配危险模式，之后按数据源方言用 sqlglot 解析 AST，拒绝 Insert/Update/Delete/Create/Drop/Alter/Merge/Copy 和危险函数。默认 SHOW/DESCRIBE/EXPLAIN 也禁止。即使 chat 已校验过允许表，真正执行前仍会再过这里。

## 结果标准化

不同驱动返回 Row、tuple、Decimal、datetime、LOB 等不同对象；`convert_value` 把它们转为 JSON 可序列化值，字段名默认转小写，以匹配图表 JSON。返回的 SQL 使用 base64 保存，避免普通响应直接暴露或转义混乱。

## 连接生命周期与性能

SQLAlchemy 类型使用 engine/session；专用驱动必须用上下文管理器关闭 connection/cursor。连接超时、读取超时和池配置需要区分。大结果集应通过 SQL LIMIT 控制，不能依赖 Python 取回后再截断。

## 新数据库适配清单

类型枚举 → 连接表单与加密配置 → 驱动依赖 → 连接测试 → schema/table/field 查询 → 执行分支 → sqlglot dialect/危险函数 → 方言 prompt → Docker 系统依赖 → 集成测试。

