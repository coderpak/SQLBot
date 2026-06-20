# templates/sql_examples 目录教程

## 每个文件的作用

| 文件 | 数据库方言重点 |
|---|---|
| `PostgreSQL.yaml` | PostgreSQL 引号、LIMIT、日期与聚合示例，也是常见回退模板。 |
| `MySQL.yaml` | 反引号、LIMIT、日期函数等 MySQL 规则。 |
| `Microsoft_SQL_Server.yaml` | 方括号/TOP/OFFSET FETCH 与 T-SQL 示例。 |
| `Oracle.yaml` | 双引号、FETCH FIRST/ROWNUM 和 Oracle 函数。 |
| `ClickHouse.yaml` | ClickHouse 聚合、函数和 LIMIT 规则。 |
| `Doris.yaml` | Doris SQL 方言示例。 |
| `StarRocks.yaml` | StarRocks SQL 方言示例。 |
| `Hive.yaml` | Hive 反引号与函数约束。 |
| `Elasticsearch.yaml` | Elasticsearch SQL 支持范围与示例。 |
| `AWS_Redshift.yaml` | Redshift 方言与日期/分析函数示例。 |
| `Kingbase.yaml` | Kingbase 方言规则。 |
| `DM.yaml` | 达梦数据库方言规则。 |

新增数据库类型时，需要同步 `apps/db/constant.py::DB`、连接执行分支和这里的模板文件。

