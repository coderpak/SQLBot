# `chat.py` API 文件教程

## 路由分组

- 会话：list/get/start/assistant-start/rename/delete。
- 问数：`POST /chat/question`。
- 结果：record data、predict_data、log、usage、Excel 导出。
- 衍生任务：analysis、predict、recommend_questions。

## 问数入口

`question_answer` 受 Chat 资源权限装饰器保护，只负责把 `ChatQuestionBase` 转成完整 `ChatQuestion`。`question_answer_inner` 是 Web 与 MCP 共享的业务入口：先用 `parse_quick_command` 判断普通问句还是重新生成/分析/预测，再调用对应流程。

`stream_sql` 做四件事：创建 `LLMService`、持久化新记录、提交后台任务、把 `await_result` 作为 SSE body。同步 MCP 调用也能用 `stream=false` 收集最终 JSON。

## 错误处理

流式请求发生错误时不能再改 HTTP status，因此函数发送 `type=error` 的 SSE；非流式请求才返回 500 JSON。前端必须同时处理 HTTP 错误和流内错误。

## 修改检查单

- 新接口是否需要 `require_permissions` 与 `system_log`。
- 新 SSE type 是否同步前端 switch。
- 是否使用 `asyncio.to_thread` 隔离同步数据库工作。
- 下载/StreamingResponse 是否会被统一响应中间件错误包装。

