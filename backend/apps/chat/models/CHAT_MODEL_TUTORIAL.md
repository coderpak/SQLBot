# `chat_model.py` 文件教程

## 三类内容

1. **表模型**：`Chat`、`ChatRecord`、`ChatLog`，保存会话、每轮结果和步骤日志。
2. **API DTO**：创建/重命名会话、问题、坐标轴、历史与响应结构。
3. **Prompt 上下文**：`AiModelQuestion` 保存 question、engine、schema、sample_data、术语、训练样例、错误反馈等，并提供 prompt 构造方法。

## Prompt 为什么拆成多段消息

`sql_sys_question` 返回 system/rules/schema/custom_prompt/terminologies/data_training，多段消息之间插入“已理解规则”的 AI 消息。这样既能明确不同信息的角色，也方便日志排查究竟是哪一段上下文影响了 SQL。

`sql_user_question` 会带当前时间、数据库类型、错误反馈和是否生成标题；重新生成时附加 regenerate hint。图表阶段的 `chart_user_question` 使用已经确认的 SQL 和相关 Schema，不直接依赖完整查询结果。

## 修改风险

模型字段改动要配 Alembic；DTO 字段改动要同步前端实体；prompt 变量改动要同步 YAML format 占位符，否则运行时会抛 KeyError。

