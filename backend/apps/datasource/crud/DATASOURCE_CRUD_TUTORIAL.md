# `datasource.py` 核心文件教程

这个文件同时承担数据源管理与 AI 元数据供应。阅读时分为四块：

1. 数据源 CRUD 与配置加解密。
2. 连接测试、预览、执行 SQL。
3. 表字段/样例数据读取。
4. `get_table_schema`：问数用 Schema 构造。

`get_table_schema` 先从系统库读取当前数据源已同步且对用户可见的表字段，生成类似 `# Table` 与 `(field:type, comment)` 的文本；表多时用问题 embedding 选 Top-N；若选中表参与关系图，则补齐关系另一端表与 Foreign keys。它同时返回允许表名列表，后续 `run_task` 用此列表做 SQL 越权检查。

表注释、字段注释和关系不是 UI 装饰，而是大模型理解业务含义的直接输入。数据源同步后应人工维护高质量注释。

