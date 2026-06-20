# chat/api 目录教程

## 文件

- `chat.py`：聊天领域全部 HTTP 接口。核心是 `POST /chat/question`、`question_answer_inner` 和 `stream_sql`；还提供会话增删改查、记录数据、预测数据、日志、导出、推荐问题等。
- `__init__.py`：包声明。

## 核心流程

`question_answer` 把请求 DTO 补成 `ChatQuestion`；`question_answer_inner` 识别快捷命令；`stream_sql` 创建 `LLMService`、先保存记录、启动任务，并把 `await_result()` 包成 `StreamingResponse`。

API 层不要塞入生成 SQL 的细节。它的职责是鉴权、参数、协议和错误边界。

## 路由分组与返回类型

- 会话路由返回 ChatInfo/Chat 列表，内部使用 `asyncio.to_thread` 执行同步查询。
- record data/predict/log/usage 是按需加载接口，避免首屏把大数据一次性返回。
- question/analysis/predict/recommend_questions 返回 SSE。
- Excel export 返回二进制流，不能经过普通 JSON 包装。

## 权限与审计

提问按 chat_id 检查 Chat 权限；创建会话按 datasource 检查资源；重命名/删除使用 system_log 记录资源 ID 和名称。新增路由时要明确“权限 key 从哪个参数表达式取得”，否则装饰器可能取不到嵌套 DTO 的 ID。

## SSE 错误边界

响应开始后 HTTP 200 已发送，后续错误只能作为 `type=error` 事件。客户端中止不会自动停止所有后台工作，修改任务模型时要关注 future 生命周期和数据库连接释放。

