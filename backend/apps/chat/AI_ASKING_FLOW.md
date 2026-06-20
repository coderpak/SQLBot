# AI 问数后端完整链路

## 入口与协议

`POST /api/v1/chat/question` 接收 `ChatQuestionBase`，API 层补成 `ChatQuestion` 后进入 `question_answer_inner`。普通提问走 `stream_sql`；带快捷命令的问题可走重新生成、分析或预测。默认返回 `text/event-stream`。

## 一轮问数的状态机

1. `LLMService.create` 选择工作空间可用的默认/自定义模型，由 `LLMFactory` 创建 LangChain chat model。
2. `init_record` 调用 `save_question`，先把问题变成 ChatRecord，确保后续每一步都有可追踪 ID。
3. `run_task_async` 把 `run_task` 放入线程执行，API 线程通过 `await_result` 消费队列并向客户端输出。
4. 有数据源时召回术语、SQL 训练示例、自定义 SQL 提示词；无数据源时模型先选择数据源。
5. `choose_table_schema` 调用 `get_table_schema`，根据问题的 embedding 从大量表中选相关表，并补上相关外键表、字段和样例数据。
6. `init_messages` 构造 SQL prompt：系统身份、SQL 规则、数据库方言、Schema、样例数据、术语、训练 SQL、自定义提示词与有限轮历史。
7. `generate_sql` 调用 `self.llm.stream`，一边产生 chunk 一边记录 token、思考内容和完整消息。
8. `check_sql`/`check_save_sql` 提取结构化 SQL；用 sqlglot 取得真实表名，与 `table_name_list` 比较，拒绝越权表。
9. 普通用户按行列权限改写 SQL；动态助手可生成中间 SQL。
10. `execute_sql` → `apps/db/db.py::exec_sql`，最终只读检查后按数据源驱动执行，返回 fields/data。
11. 结果做大整数保护和字段名规范化，保存后发送 `sql-data`。
12. `generate_chart` 用问题、SQL、用到的表结构和建议类型再次请求模型，得到图表 JSON；校验保存后发送 `chart`。
13. 记录耗时、token、日志和 finish；异常被转换成 `error` SSE。

## RAG 在哪里

- 表召回：`apps/datasource/embedding/table_embedding.py`。
- 数据源召回：`apps/datasource/embedding/ds_embedding.py`。
- 术语召回：`apps/terminology/curd/terminology.py::get_terminology_template`。
- SQL 示例召回：`apps/data_training/curd/data_training.py::get_training_template`。
- 自定义提示词：`LLMService.filter_custom_prompts`，部分高级能力受授权模块控制。

RAG 的目标不是让模型“查数据库”，而是在生成 SQL 之前，把最相关且被授权的元数据塞入 prompt。

## 安全检查层次

1. 数据源必须属于当前工作空间/助手可用范围。
2. Schema 只包含用户可见表字段。
3. sqlglot 从真实 SQL 提取表，不能只信模型附带的 tables。
4. 行列权限可能改写 SQL。
5. `check_sql_read` 在真正执行前再拒绝写操作和默认禁止的元数据查询。
6. 查询结果行数默认受 prompt 与系统参数限制；生产仍建议数据源账号只读和数据库侧限权。

## 读代码时关注的对象

- `Chat`：一次会话，绑定数据源、标题和助手上下文。
- `ChatRecord`：一次问答的持久状态，SQL、数据、图表、错误和衍生分析都围绕它。
- `ChatLog`：步骤级审计、prompt、推理、耗时与 token。
- `ChatQuestion/AiModelQuestion`：prompt 所需的可变上下文容器。
- `LLMService`：有状态的单轮执行器；它不是简单的“调一次模型”。

## 从 HTTP 请求开始逐步追踪

### 第 0 步：先有 Chat

前端通常先调用 `/chat/start` 或 `/chat/assistant/start`，得到 chat_id。Chat 保存 datasource、engine_type、会话来源和标题；用户后续每个问题都带相同 chat_id。若会话尚未绑定数据源，问题也可带 datasource_id，或者由自动选数据源流程决定。

### 第 1 步：问题进入 API

典型请求体：

```json
{
  "chat_id": "1912345678901234567",
  "question": "今年各地区销售额是多少？"
}
```

`question_answer` 的权限表达式从请求取 chat_id，确认当前用户可访问会话，然后构造 ChatQuestion。`question_answer_inner` 先解析快捷命令；没有命令时调用 `stream_sql`。

### 第 2 步：创建服务和记录

`LLMService.create` 从 Chat 取得数据源，从当前工作空间取得模型配置；助手可覆盖模型或提供外部数据源。`init_record` 立即保存 question 并生成 Snowflake record ID。之后第一批 SSE 就能返回 `id` 和 `question`。

这一设计非常重要：即使模型调用超时，也有 record 和日志可查；前端临时 record 可以被真实 ID 替换。

### 第 3 步：建立 RAG 上下文

对已选数据源，服务先召回：

1. 相关术语，例如“销售额”的业务定义。
2. 相似问题的正确 SQL 样例。
3. 生成 SQL 类自定义提示词。
4. 与问题最相似的表。
5. 关系图中连接这些表所必需的邻表。
6. 每张表字段、类型、人工注释和少量样例数据。

每一步通过 `start_log/end_log` 保存命中明细。执行详情看到的“选择数据表、术语、SQL 样例”就是这些日志，而不是前端猜出来的。

### 第 4 步：组装 SQL 消息

`init_messages` 不是简单地拼一个字符串，而是多条 message：SystemPrompt 定义身份，HumanPrompt 给规则，AI Prompt 确认已理解，再给 Schema、术语、训练样例与自定义提示词；最后追加有限轮历史。系统参数 `chat.context_record_count` 控制历史轮数，避免 prompt 无限增长。

### 第 5 步：流式生成 SQL

`generate_sql` 把当前用户问题追加到 sql_message，调用 `self.llm.stream`。`process_stream` 统一供应商 chunk，区分 content、reasoning_content 和 token usage。每个 chunk 通过 `sql-result` SSE 发到前端，同时累积完整回答并保存 ChatLog/ChatRecord。

### 第 6 步：解析和验证模型输出

完整文本先被解析为 SQL 和模型声称使用的 tables。但安全检查不信任 tables 字段，而是用 sqlglot 从真实 SQL AST 提取实际表名，再与 `choose_table_schema` 返回的 `table_name_list` 比较。访问任何未授权表都会拒绝。

如果当前用户需要行列权限，原 SQL 还会进入权限处理；动态助手可能生成包含外部子查询占位符的 SQL。最终保存的是校验/改写后的 SQL，并通过 `type=sql` 发给前端。

### 第 7 步：数据库执行

`execute_sql` 调用多数据库适配器 `exec_sql`。执行前 `check_sql_read` 再检查：首关键字、危险模式、sqlglot 写 AST 和危险函数。通过后才执行，输出：

```json
{
  "fields": ["region", "sales"],
  "data": [
    {"region": "华东", "sales": 123456.78}
  ]
}
```

数据进行大整数、Decimal、字段名规范化后保存。后端只发送 `sql-data` 完成信号；前端再按 record ID 调 data 接口，这避免大结果阻塞 SSE。

### 第 8 步：生成图表配置

服务根据 SQL 回答中的建议类型、实际使用表 Schema、问题和已确认 SQL构造 chart_message，再次调用同一 LLM。模型输出 chart JSON；`check_save_chart` 提取第一个合法 JSON、处理 error、将字段 value 统一小写并保存。

典型输出：

```json
{
  "type": "column",
  "title": "今年各地区销售额",
  "columns": [{"name": "地区", "value": "region"}],
  "axis": {
    "x": {"name": "地区", "value": "region"},
    "y": {"name": "销售额", "value": "sales"}
  }
}
```

`chart-result` 是生成中的推理片段，`chart` 才是最终配置；最后 `finish` 表示主流程完成。

## 一轮正常 SSE 的示例顺序

```text
id
question
sql-result       # 可能出现很多次
brief            # 首次问题可能更新会话标题
sql
sql-data
chart-result     # 可能出现很多次
chart
finish
```

若自动选择数据源，前面还会有 `datasource-result` 和 `datasource`。异常时可能在任意位置出现 `error`，因此前端不能假设每轮都有 chart/finish。

## `finish_step` 的意义

| 终点 | 已完成内容 | 典型调用方 |
|---|---|---|
| GENERATE_SQL | 生成并校验 SQL | 只需要 SQL 的 MCP 工具 |
| QUERY_DATA | 再执行并返回数据 | 数据分析/自动化流程 |
| GENERATE_CHART | 再生成图表 | 完整 Web 问数和图片输出 |

复用 finish_step 可以避免为了 MCP 复制三套状态机。

## 快捷命令如何复用历史

重新生成、分析、预测可以指定 record_id；未指定时寻找最后一条普通问答记录。重新生成会沿 regenerate_record_id 找到基础问题并带入错误反馈；分析/预测创建关联子记录，不会覆盖原记录。首条初始化记录和已有分析/预测子记录不能被当作普通目标。

## 错误定位表

| 停在哪一步 | 常见原因 | 看哪里 |
|---|---|---|
| 创建服务前 | 无默认模型、模型未映射工作空间 | system/aimodel、LLMService.create |
| 选择数据源 | 没有候选、外部助手返回格式错误 | ds_embedding、AssistantOutDs |
| 选表 | embedding 模型失败、元数据未同步 | choose_table_schema 日志 |
| SQL 生成 | 模型超时/输出格式错误 | GENERATE_SQL ChatLog |
| SQL 校验 | 访问未授权表、解析失败 | check_sql、actual_tables |
| 权限改写 | 行权限树/系统变量错误 | permission、row_permission |
| 执行 | 连接、方言、字段变化、只读拒绝 | exec_sql、SQL_DEBUG 日志 |
| 图表 | JSON 无法解析、字段名不匹配 | check_save_chart、record data |

## 推荐断点顺序

1. `chat/api/chat.py::question_answer_inner`
2. `chat/api/chat.py::stream_sql`
3. `chat/task/llm.py::create` 和 `init_record`
4. `run_task` 的 1220、1267、1305、1377、1428 附近
5. `datasource/crud/datasource.py::get_table_schema`
6. `chat/models/chat_model.py::sql_sys_question`
7. `db/db.py::check_sql_read` 与 `exec_sql`
8. `chat/task/llm.py::check_save_chart`

用同一个 record ID 对照 ChatRecord、ChatLog、服务日志和浏览器 SSE，能把完整因果链串起来。

