# template/generate_sql 目录教程

- `generator.py`：`get_sql_template` 读取通用 Text-to-SQL 模板；`get_sql_example_template` 按数据库类型读取方言规则和示例。
- `__init__.py`：包声明。

最终拼接发生在 `chat_model.py::sql_sys_question`。数据库模板缺失时 `DB.get_db(..., default_if_none=True)` 会回退到 PostgreSQL 语义。

通用模板定义输出结构和安全规则，方言模板提供 quote/limit/function/example。新增规则时优先放正确层级：所有库适用放 template.yaml，仅单一方言适用放对应 SQL YAML，避免其他数据库被错误约束。

## 运行时拼装

`AiModelQuestion.sql_sys_question` 读取两类模板，组合 system/rules/schema/terminology/training/custom_prompt；`sql_user_question` 再加入当前时间、问题、错误反馈和重新生成提示。

## 输出消费者

`generate_sql` 保存完整模型文本；`check_sql` 提取 SQL/表信息；sqlglot 再解析真实表；权限模块可能改写；最终 exec_sql 执行。模板声称“只读”不是安全保证，只是第一层引导。

## 回归问题集

单表聚合、多表关联、日期范围、同比环比、Top-N、空值、别名、不同数据库分页、恶意写请求和上一轮 SQL 报错重试。

