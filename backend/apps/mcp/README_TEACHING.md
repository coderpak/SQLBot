# mcp 目录教程

- `mcp.py`：把数据源列表、模型列表、开始会话、问数、助手问数等能力暴露给 MCP；复用 `question_answer_inner`，并通过 `finish_step` 控制返回 SQL、数据或图表。
- `__init__.py`：包声明。

`main.py` 使用 FastApiMCP 挑选这些 operation，并在 8001 的 `mcp_app` 上挂载 MCP 与图片静态目录。外部客户端不直接复制一套问数逻辑，而是复用 chat 核心。

## Operation 语义

- `mcp_start`：建立 Chat，返回后续提问所需 chat_id。
- `ws_list`：列出 token 用户可访问的工作空间。
- `datasource_list`：按工作空间/问题提供候选数据源。
- `mcp_question`：普通用户问数。
- `mcp_assistant`：按助手配置和 token 问数。

## 与 Web 的不同

MCP 使用同一 `question_answer_inner`，但 `in_chat=false` 时可输出 Markdown、record id、SQL、表格或图片 URL；`finish_step` 允许只生成 SQL、查询数据或继续生成图表。安全检查、RAG、权限和日志仍复用 Web 流程。

## 部署关键点

8001 必须能被 MCP 客户端访问；`SERVER_IMAGE_HOST` 必须是客户端能访问的公网/内网地址，而不是容器里的 localhost；图片目录需要持久化并定期清理。operation allowlist 在 `main.py`，新增普通 API 不会自动成为 MCP 工具。

