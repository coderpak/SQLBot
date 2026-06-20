# system/schemas 目录教程

| 文件 | 作用 |
|---|---|
| `ai_model_schema.py` | 模型创建、编辑、列表和动态配置项 DTO。 |
| `auth.py` | 登录/token 等认证 DTO。 |
| `logout_schema.py` | 登出请求/响应结构。 |
| `permission.py` | `SqlbotPermission`、请求上下文中间件和权限装饰器；通过表达式从参数提取资源 ID。 |
| `system_schema.py` | 用户、工作空间、助手、API Key 等综合 DTO。 |
| `__init__.py` | 包声明。 |

`permission.py` 直接影响数据源和 Chat 接口的资源授权，修改表达式求值或 CurrentUser 结构时要做越权测试。

## DTO 作用

Schema 把“数据库可存字段”缩小为“客户端允许提交/查看字段”。Creator 不应接受 id/create_by 等服务端字段；Editor 才带 ID；Brief/Grid DTO 避免列表泄露 api_key/config；AssistantUiSchema 控制嵌入 UI。

## 权限表达式

`SqlbotPermission(type, keyExpression)` 描述资源类型与从函数参数中取 ID 的路径。装饰器绑定参数后求表达式，调用工作空间权限检查。嵌套 DTO、Path 参数和批量 ID 的提取方式不同，新增接口要用实际调用测试。

## 校验

语言、密码、模型 config_list、助手域名等可在 Pydantic validator 早期拒绝。校验错误应是用户可理解的业务信息，敏感字段不回显原值。

