# apps 目录教程

## 作用

后端业务模块根目录。`api.py` 创建总路由，并 include 登录、用户、工作空间、助手、模型、术语、训练、数据源、聊天、仪表板、MCP、参数、API Key 和变量等子路由。

## 文件

- `api.py`：唯一的业务路由聚合器，被 `main.py` 挂到 `/api/v1`。
- `__init__.py`：声明 Python 包。
- 其余子目录：按业务域隔离；多数遵循 api → crud/curd → models/schemas 的方向。

## 业务域分组

- 问数主链：chat、ai_model、datasource、db、template、terminology、data_training。
- 控制面：system、settings、swagger。
- 输出与集成：dashboard、mcp。

## 一次请求怎样穿过 apps

`main.py` 挂载 `api_router` → 具体 `api/*.py` 用 FastAPI 解析和鉴权 → crud/service 操作系统库或组织领域逻辑 → models/schemas 约束数据 → 必要时调用其他领域公开函数。API 不应直接 import 另一个模块的内部前端概念。

## 依赖规则

chat 可以编排 datasource/ai_model/template/RAG/db；这些底层模块不应反向依赖 chat 的 HTTP API。MCP 可以复用 chat 的公共入口；dashboard 接收 chat 产物但不参与 Text-to-SQL。保持单向依赖能减少循环导入。

## 新增业务模块

创建包和 router → 定义 schema/model → 实现 crud/service → 在 `apps/api.py` 注册 → 加权限与审计 → 补 migration/i18n/API 文档/测试 → 在对应目录写教学文档。

