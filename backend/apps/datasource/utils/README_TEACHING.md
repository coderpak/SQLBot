# datasource/utils 目录教程

- `excel.py`：Excel 上传、读取、类型推断与落地为可查询数据源的辅助逻辑。
- `utils.py`：数据源配置 AES 解密、连接参数转换等通用函数。
- `__init__.py`：包声明。

排查某数据库连接参数时，还要结合 `apps/db/engine.py` 与 `apps/db/db.py`。

`utils.py` 的 AES 解密把 CoreDatasource.configuration 恢复为 DatasourceConf，任何异常都不应把明文配置写日志。`excel.py` 负责 Sheet、列名、类型推断和空值处理；导入后通常落为 PostgreSQL 可查询表并创建对应数据源元数据。

Excel 列名要处理重复、空白、特殊字符和超长名称；类型推断应允许用户修正，因为混合文本/数字列很容易误判。大文件要流式/分块，避免 DataFrame 一次性耗尽内存。

