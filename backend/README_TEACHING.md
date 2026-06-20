# backend 学习指南

## 目录职责

这是 SQLBot 的 FastAPI 后端。它同时承担 API、认证授权、系统库访问、业务数据源访问、RAG、LLM 编排、SSE、MCP 和数据库迁移。

## 顶层文件

| 文件 | 作用 |
|---|---|
| `main.py` | 应用入口；启动迁移/缓存/embedding 初始化，注册中间件和路由，创建 Web 与 MCP 两个 FastAPI 应用。 |
| `pyproject.toml` | Python 3.11 与依赖清单；uv 的 CPU/CUDA extra、ruff/mypy/pytest 配置。 |
| `alembic.ini` | Alembic 配置入口。 |
| `README.md` | 上游项目留下的简短后端说明。 |
| `.gitignore` | 后端忽略规则。 |

## 子目录

| 目录 | 作用 |
|---|---|
| `apps/` | 按业务域组织的 FastAPI API、CRUD、SQLModel 与任务编排。 |
| `common/` | 配置、依赖注入、认证、安全、缓存、审计和通用工具。 |
| `alembic/` | 系统 PostgreSQL 数据库结构迁移。 |
| `templates/` | LLM 提示词和各数据库 SQL 方言样例。 |
| `locales/` | 后端业务错误与消息的国际化文本。 |
| `scripts/` | 启动前迁移、测试、lint、格式化脚本。 |

## 启动过程

`uvicorn main:app` 导入模块后创建 FastAPI；进入 `lifespan` 时依次执行 Alembic upgrade、缓存初始化、动态 CORS、术语/SQL 示例/表与数据源向量补齐、模型配置处理。随后中间件按认证、统一响应、权限上下文、审计上下文工作，`apps/api.py` 汇总业务路由并挂到 `/api/v1`。

## 分层约定

- `api/`：HTTP 输入输出、依赖注入、状态码和 StreamingResponse。
- `crud/` 或历史拼写 `curd/`：查询、持久化、业务数据拼装。
- `models/`：SQLModel 表模型与核心 DTO。
- `schemas/`：Pydantic 请求/响应结构。
- `task/`：长流程、异步或 LLM 编排。

## 阅读顺序

`main.py` → `apps/api.py` → `apps/chat/api/chat.py` → `apps/chat/task/llm.py` → `apps/chat/models/chat_model.py` → `apps/datasource/crud/datasource.py` → `apps/db/db.py`。

## 模块之间如何分工

| 模块 | 它拥有的决定权 | 它不应该做的事 |
|---|---|---|
| `system` | 身份、工作空间、模型/助手可用范围 | 生成或执行业务 SQL |
| `ai_model` | 把模型配置变成统一 LLM/embedding | 决定用户能访问哪些表 |
| `datasource` | 连接元数据、Schema、关系、行列权限 | 管理系统用户登录 |
| `db` | 数据库方言连接、只读校验与执行 | 选择模型或拼业务 prompt |
| `terminology` | 业务词口径 RAG | 保存正确 SQL 样例 |
| `data_training` | 问题-SQL few-shot RAG | 修改模型参数 |
| `template` | LLM 输入/输出协议 | 直接持久化 ChatRecord |
| `chat` | 编排整轮问数和 SSE 状态 | 自己实现每种数据库驱动 |
| `dashboard` | 长期保存和组织可视化 | 重新实现 Text-to-SQL |
| `mcp` | 对外暴露可组合工具 | 复制一套问数核心 |
| `common` | 跨模块运行基础设施 | 堆放具体业务规则 |

更完整的输入输出与依赖关系见根目录 `MODULE_DEEP_DIVE.md`。

## 排错原则

先定位错误属于“控制面、RAG、生成、安全、执行、展示”哪一层，再进入相应模块。不要看到 SQL 错就立刻改 prompt：可能是表注释、错误训练样例、权限改写或方言适配导致。

