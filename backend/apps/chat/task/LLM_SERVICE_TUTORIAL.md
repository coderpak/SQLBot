# `llm.py` 核心文件教程

这是项目最重要、也最值得分段阅读的文件。不要从第一行硬读到最后一行；按下面的功能块阅读。

## 1. 初始化（`__init__` / `create`）

读取 Chat 和数据源，验证工作空间归属，选择 AI 模型，创建 LangChain LLM，加载历史生成日志和系统参数。这里决定“本轮用谁回答、可访问哪个数据源、保留几轮上下文”。

## 2. 上下文准备

- `filter_terminology_template`：召回业务术语。
- `filter_training_template`：召回相似的问句-SQL 样例。
- `filter_custom_prompts`：召回场景提示词。
- `choose_table_schema`：召回相关表、字段、关系和样例数据。
- `init_messages`：把上述内容和历史对话变成 SQL/Chart 两套消息列表。

SQL 生成和图表生成是两套独立 prompt，不能混为一次模型调用。

## 3. 生成与校验

- `generate_sql`：流式调用模型并持久化原始回答。
- `check_sql`、`check_save_sql`：从模型文本提取 SQL、规范化并保存。
- `generate_filter`：为普通用户生成/应用数据权限过滤。
- `generate_chart`、`check_save_chart`：生成并校验图表 JSON。

## 4. 执行和持久化

`execute_sql` 只做适配与异常翻译，真正的多数据库执行在 `apps/db/db.py::exec_sql`。`save_sql_data` 等方法调用 chat CRUD，把中间态持续落库，因此刷新页面仍可恢复一轮问答。

## 5. 编排（`run_task`）

这是状态机主函数。建议按 SSE `yield` 搜索阅读：`id` → `question` → `datasource` → `sql-result` → `sql` → `sql-data` → `chart-result` → `chart` → `finish`。`finish_step` 允许 MCP 等调用方停在“只生成 SQL”或“只查询数据”。

## 6. 并发模型

FastAPI API 创建服务后调用 `run_task_async`。耗时的同步数据库与模型工作在后台线程中执行，chunk 进入队列；`await_result` 再把队列内容转换成 SSE。修改这里时要同时考虑线程独立 Session、客户端中止、异常如何入队和资源关闭。

## 7. 修改时的高风险点

- prompt 输出格式与 `check_*` 解析逻辑必须同步。
- 新增 SSE type 必须同步前端 `ChartAnswer.vue`。
- 不可绕过表白名单和 `exec_sql` 的只读检查。
- 不要跨线程复用 FastAPI 注入的 SQLModel Session；`run_task` 自己创建 session 正是为此。
- 大结果集会影响数据库、内存和前端，限制行数不是装饰规则。

## 8. 先认识实例字段

| 字段 | 含义 |
|---|---|
| `sql_message` | SQL 生成消息历史 |
| `chart_message` | 图表生成消息历史 |
| `generate_sql_logs/chart_logs` | 以前轮次的模型消息，用于有限上下文 |
| `current_logs` | 本轮各 OperationEnum 正在记录的日志 |
| `chunk_list` | 后台任务与 SSE 消费者之间的缓冲 |
| `current_user/current_assistant` | 权限和运行场景 |
| `ds/out_ds_instance` | 普通或动态数据源 |
| `chat_question` | 本轮 prompt 上下文容器 |
| `record` | 当前 ChatRecord |
| `llm` | LangChain BaseChatModel |
| `table_name_list` | 本轮允许访问的真实表名白名单 |

LLMService 是单轮、有状态、不可跨请求共享的对象。

## 9. `create` 与 `__init__` 为什么分开

Python `__init__` 不能 await，而默认模型配置读取/解密可能是异步，所以类方法 `create` 先 await `get_default_config`，再构造实例并读取聊天参数。调用方必须用 `await LLMService.create(...)`，不能直接 `LLMService(...)`，否则缺少 config。

构造阶段还会验证数据源工作空间归属、设置 engine 名称与版本、加载历史日志、判断是否需要生成会话标题、读取上一轮 SQL 执行错误，并创建 LLM。

## 10. `init_messages` 的消息顺序

SQL 消息通常是：

```text
System: 身份和总体要求
Human: 规则/方言/示例
AI: 已理解规则
Human: Schema/样例数据
AI: 已确认 Schema
Human + AI: 可选自定义提示词
Human + AI: 可选术语
Human + AI: 可选 SQL 训练样例
历史 human/ai 若干轮
当前 Human 问题（generate_sql 时追加）
```

Chart 使用独立消息列表，防止 SQL 规则和 Chart JSON 规则互相污染。历史日志中过去标记为 `sqlbot_system` 的系统消息会被排除，再由当前版本模板重新生成，避免系统规则重复堆叠。

## 11. RAG 方法的共同结构

每个 filter 方法都会 start_log → 计算 oid/ds/assistant 范围 → 调领域模块相似检索 → 把格式化文本写入 chat_question → end_log 保存命中明细。这样 prompt 使用文本，ExecutionDetails 使用结构化命中列表。

普通数据源和动态助手的 oid/ds 计算不同；修改范围逻辑时最危险的是跨工作空间召回。

## 12. SQL 生成后的多层处理

1. `generate_sql` 得到原始文本和 reasoning。
2. `get_chart_type_from_sql_answer`/`get_brief` 读取附加信息。
3. `check_sql` 提取 SQL 和模型 tables。
4. sqlglot `extract_tables_from_sql` 取得真实表。
5. 与 `table_name_list` 比较，拒绝 unauthorized tables。
6. 普通用户调用 `generate_filter` 处理权限。
7. 动态数据源生成/替换子查询。
8. `check_save_sql` 保存最终 SQL。
9. `exec_sql` 再做只读 AST 检查。

任何“模型输出已经写了 tables，所以可直接执行”的改法都会破坏安全边界。

## 13. 数据结果处理

执行返回 fields/data 后，先把大整数转字符串，再规范化带表前缀的 column key，然后保存。SSE 只发 `sql-data` 信号。`finish_step=QUERY_DATA` 时 MCP 可把数据转为 Pandas/Markdown；空数据要输出明确提示。

## 14. Chart 处理

只对 SQL 真正使用的 tables 再获取一次 Schema，减少 chart prompt。`generate_chart` 流式发送 reasoning；`check_save_chart` 用 extract_nested_json 取 JSON，处理 y 单对象/数组和 multi-quota，统一 value 小写，保存并返回。

当前校验更偏格式解析；二次开发可增加 Pydantic ChartSchema、type 枚举、字段必须存在于 result.fields 等严格验证。

## 15. 日志和 token

每个 AI 步骤记录发送的完整 message、reasoning、token usage、开始结束时间；本地步骤如选表/执行 SQL记录 local_operation 和输入摘要。ChatLog 既用于执行详情，也用于下轮上下文，因此删除/改写日志会影响后续生成。

敏感数据治理要权衡可观测性：完整 prompt 可能含 Schema、样例和业务数据，生产应控制日志查看权限、保留周期和脱敏。

## 16. 调试一次 `run_task`

用固定问题，在以下位置记录/断点：

1. filter 后的 terminologies/data_training/custom_prompt。
2. init_messages 最终消息条数和内容片段。
3. full_sql_text。
4. check_sql 返回 sql/tables 与 actual_tables。
5. 权限改写后的 SQL。
6. execute_sql fields/row count。
7. full_chart_text 与解析后 chart。
8. finish_record 的 duration/token/error。

始终用 record.id 串联，不要混看其他并发请求日志。

## 17. 典型改造场景

### 增加 SQL 自动修复重试

捕获数据库错误 → 保存错误 → 把错误以受控标签加入下一次 user prompt → 限制重试次数 → 每次重新过表白名单/权限/只读检查。不能直接执行模型第二次输出。

### 增加新 RAG 知识源

定义归属和权限 → 建 embedding/检索 → 在 LLMService 新建 filter 方法和 OperationEnum → 加入 sql/analysis 消息 → 记录命中明细 → 前端 ExecutionDetails 增加展示。

### 支持新输出格式

先定义 Pydantic schema → 修改 prompt → 严格解析/兼容旧格式 → 持久化字段和 migration → SSE/非流式/MCP 输出 → 前端消费与历史恢复。

## 18. 代码拆分建议

若继续演进，可把超大 llm.py 拆为 ContextRetriever、PromptBuilder、SqlGenerationPipeline、ChartPipeline、TaskStreamer 和 ResultRepository，但要保持当前状态机顺序和安全门。拆分目标是明确职责，不是把共享可变状态散落到更多全局对象。

