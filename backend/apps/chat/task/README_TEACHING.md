# chat/task 目录教程

## 文件

- `llm.py`：`LLMService` 核心状态机；召回上下文、生成 SQL、校验/权限处理、执行、生成图表、分析、预测、推荐问题、SSE 队列和任务线程都在这里。
- `LLM_SERVICE_TUTORIAL.md`：按功能块拆解 `llm.py` 的专门教程。
- `__init__.py`：包声明。

这个目录是后端第一优先级阅读区，但要先理解 `api/chat.py` 的协议入口和 `models/chat_model.py` 的 prompt 数据。

## 方法分组

- 生命周期：`create`、`__init__`、`init_record`、`run_task_async`、`await_result`。
- RAG：filter_terminology/training/custom_prompt、choose_table_schema。
- LLM：init_messages、generate_sql/chart/analysis/predict/guess。
- 安全：check_sql、check_save_sql、validate_history_ds、权限过滤。
- 执行：execute_sql、save_sql_data。
- 协议：run_task 中各 SSE yield 与 finish_step 分支。

## 线程和 Session

全局 executor 提交 `run_task_cache`，它消费 `run_task` generator 并缓存 chunk；API 的 `await_result` 一边判断 future，一边弹出 chunk。后台方法使用 `session_maker()` 新建 Session，不能复用 FastAPI 注入 Session。

## 失败恢复

数据库错误会保存并进入下次 prompt 的 error_msg，帮助重新生成；模型输出解析错误转换为 SingleMessageError；日志的 trigger/error 标记让前端定位失败步骤。异常处理要保证 record 最终状态和 Session close。

