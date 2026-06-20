# common/audit/schemas 目录教程

| 文件 | 作用 |
|---|---|
| `logger_decorator.py` | `@system_log` 与 `LogConfig`，从函数参数/结果提取资源信息并写审计日志。 |
| `log_utils.py` | 日志字段、客户端信息和表达式解析辅助。 |
| `request_context.py` | 请求级审计上下文中间件，保存 request/user/ip 等信息。 |
| `__init__.py` | 包声明。 |

`logger_decorator` 负责拦截函数生命周期；`request_context` 用 ContextVar/中间件保存当前 Request，避免层层传参；`log_utils` 负责安全提取参数和客户端信息。异步任务脱离请求后 ContextVar 可能不存在，应允许显式传用户/资源信息。

表达式提取支持 `chat.id`、返回对象 id 等路径，但不能执行任意 Python。新增表达式能力时要限制属性访问，避免审计装饰器成为代码执行入口。

