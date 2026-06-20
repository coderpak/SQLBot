# chat 目录教程

## 作用

AI 问数业务核心。目录内形成完整闭环：HTTP/SSE 入口、Chat/ChatRecord 模型、持久化、LLM 状态机。

## 子目录

- `api/`：聊天 CRUD、提问、数据、分析、预测、导出等接口。
- `curd/`：历史拼写保留；Chat/Record/Log 的数据库操作和结果格式化。
- `models/`：会话模型、事件/步骤枚举、prompt 上下文 DTO。
- `task/`：LLM 问数编排。

先读 `AI_ASKING_FLOW.md`，再按 api → task → models → curd 阅读。`__init__.py` 仅声明包。

## 模块输入与输出

输入包括当前用户/助手、chat_id、问题、数据源、模型配置和 finish_step。输出不是单一 JSON，而是一系列状态：持久化的 ChatRecord/ChatLog、SSE 事件、查询数据、图表配置，以及 MCP 场景的 Markdown/图片。

## 内部状态迁移

`question saved` → `context retrieved` → `sql reasoning` → `sql validated` → `data queried` → `chart reasoning` → `chart validated` → `finished`。任何阶段失败都会把错误挂到同一 record，并尽量保留已生成的 SQL、日志和耗时，便于重新生成或定位问题。

## 四个子层如何协作

- api 维护 HTTP/SSE 边界，不直接拼 prompt。
- models 同时描述持久化对象、接口 DTO 和 prompt 上下文。
- task 实现有状态编排和并发。
- curd 负责每个阶段落库、恢复历史和数据格式化。

## 关键设计决定

查询数据不直接塞进 SSE，而在 `sql-data` 后由前端 GET；这样可以控制事件体大小、复用历史记录和统一大数格式。SQL 与 chart 原始回答也会保存，既能显示思考过程，也能在解析失败时审计模型原文。

## 扩展点

- 新增步骤：扩展 OperationEnum、日志、run_task、SSE type 和前端 switch。
- 新增衍生能力：复用 ChatRecord，像 analysis/predict 一样建立父子 record。
- 新增输出终点：利用 `ChatFinishStep`，不要复制 run_task 主流程。
- 修改 chart schema：同步 YAML、check_save_chart、前端类型和渲染工厂。

