# common/core 目录教程

| 文件 | 作用 |
|---|---|
| `config.py` | Pydantic Settings；读取根 `.env`，定义数据库、CORS、缓存、路径、embedding、SQL 安全和连接池配置。 |
| `db.py` | SQLBot 系统库 SQLModel engine 与 `get_session` 依赖。 |
| `deps.py` | `SessionDep`、`CurrentUser`、`CurrentAssistant`、语言翻译等 FastAPI 依赖别名。 |
| `security.py` | 密码 hash、JWT/token 等安全函数。 |
| `security_config.py` | 安全路径/认证相关配置。 |
| `response_middleware.py` | 统一 API 响应包装与全局异常处理。 |
| `sqlbot_cache.py` | memory/Redis 缓存初始化和访问封装。 |
| `models.py` | `SnowflakeBase` 等基础模型。 |
| `schemas.py` | 通用 DTO，如分页/创建审计字段。 |
| `pagination.py` | 分页请求与响应辅助。 |
| `file.py` | 上传文件、路径和静态资源辅助。 |
| `__init__.py` | 包声明。 |

最先读 `config.py`、`db.py`、`deps.py`。注意系统库 engine 与 `apps/db` 的业务数据源连接是两套概念。

## 配置生命周期

`settings = Settings()` 在模块导入时构造，因此 `.env` 修改后通常需要重启。computed field 生成 `/api/v1` 和完整 CORS；布尔 validator 兼容字符串 true/false。路径默认面向 Docker 的 `/opt/sqlbot`，Windows 本地开发必须覆盖上传、Excel、图片、模型和脚本目录。

## 请求依赖链

`TokenMiddleware` 先解析请求并写身份 → `deps.get_current_user/get_current_assistant` 从 request 获取 → API 参数中的 `CurrentUser/CurrentAssistant` 自动注入。语言依赖按 Accept-Language 创建 I18n；SessionDep 从 `get_session` 取得系统库事务。

## 事务边界

`get_session` 正常结束 commit，异常 rollback。业务函数如果中途手工 commit，就会缩短原本的请求事务；后台线程必须新建 Session，不可使用请求线程注入的实例。写复杂流程时应明确哪些状态必须原子提交，哪些中间态需要提前持久化。

## 缓存选择

memory 简单但仅对单进程有效；多 worker/多实例部署应配置 Redis，否则 token/动态配置/许可等缓存可能不一致。缓存 key 必须包含工作空间或用户维度，避免串租户。

## 统一响应例外

普通 JSON 可包装为统一 code/data/message；StreamingResponse、FileResponse 和 SSE 应保持原始 body/content-type。扩展 ResponseMiddleware 时需特别测试下载、Swagger、静态资源和聊天流。

