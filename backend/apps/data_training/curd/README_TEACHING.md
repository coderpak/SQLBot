# data_training/curd 目录教程

- `data_training.py`：SQL 样例 CRUD、批量导入、embedding 更新、`select_training_by_question` 相似检索、`get_training_template` 生成 LLM 上下文。
- `__init__.py`：包声明。

它是“越问越准”的关键模块之一：高质量少量样例通常比堆叠模糊提示词更有效。

## 函数组

query builder/execute 统一分页与关联数据源名称；create/update/batch/delete/enable 管生命周期；save_embeddings 补向量；`select_training_by_question` 使用 pgvector 距离并按 oid、datasource 或 advanced_application 过滤；`get_training_template` 返回 prompt 文本和命中明细供 ChatLog。

相似度阈值和 Top-N 来自 settings。阈值过高会无样例，过低会引入不相关 SQL。命中明细被记录后，可以从执行详情判断“错误 SQL 是否由坏样例诱导”。

