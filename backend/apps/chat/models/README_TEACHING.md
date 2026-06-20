# chat/models 目录教程

## 文件

- `chat_model.py`：定义 `Chat`、`ChatRecord`、`ChatLog`、请求/响应 DTO、操作枚举、`ChatFinishStep`，以及 `AiModelQuestion` 的 SQL/图表/分析/预测 prompt 构造方法。
- `__init__.py`：包声明。

## 关键认识

数据库模型和 prompt 上下文集中在同一文件。`AiModelQuestion.sql_sys_question` 会把方言规则、Schema、样例、术语、训练 SQL、自定义提示词组成多段消息；`sql_user_question` 才放当前问题。`chart_user_question` 传入问题、已确认 SQL、建议图表类型和使用到的 Schema。

## 重要枚举

`OperationEnum` 定义日志步骤，驱动后端 ChatLog 与前端 ExecutionDetails；`ChatFinishStep` 允许流程停在生成 SQL、查询数据或生成图表；`QuickCommand`/相关 DTO 支持重新生成、分析和预测。枚举值一旦被持久化或发到前端，就要考虑兼容旧记录。

## ChatRecord 关系

普通记录可通过 `regenerate_record_id` 指向被重新生成的记录；分析和预测记录通过 analysis/predict 关联父记录；`first_chat` 等标志区分会话初始化。理解这些关系后，才能看懂快捷命令为何先查上一条有效记录。

## Prompt 数据来源

engine 来自数据源类型/版本；db_schema/sample_data 来自 datasource；terminologies/data_training/custom_prompt 来自 RAG；error_msg 来自上一轮执行错误；history 来自 ChatLog messages。AiModelQuestion 是上下文汇合点。

