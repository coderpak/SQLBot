# system/api 目录教程

| 文件 | 作用 |
|---|---|
| `login.py` | 登录、token、当前用户等认证入口。 |
| `user.py` | 用户分页、创建、编辑、启停、密码和工作空间关系。 |
| `workspace.py` | 工作空间 CRUD、成员与资源范围。 |
| `aimodel.py` | 模型配置 CRUD、连通性、默认模型、工作空间模型映射。 |
| `assistant.py` | 高级助手/嵌入应用配置、令牌、可用数据源等接口。 |
| `apikey.py` | 对外 API access key/secret key 管理。 |
| `parameter.py` | 可在 UI 配置的系统参数。 |
| `variable_api.py` | 系统变量与用户绑定变量接口。 |
| `__init__.py` | 包声明。 |

模型管理保存的是调用配置，真正创建 LLM 在 `apps/ai_model/model_factory.py`。

## API 共同模式

接口通过 SessionDep 操作系统库，通过 CurrentUser/CurrentAssistant 取得身份，通过 Trans 生成本地化错误；管理动作叠加 require_permissions 与 system_log。Snowflake ID 在 path/body 中可能以字符串传输，Pydantic/SQLModel 边界要统一转换。

## 关键业务关系

login 签发身份；user/workspace 建立租户上下文；aimodel 提供工作空间可用模型；assistant 提供嵌入/动态数据源上下文；parameter 改变 chat 默认行为；variable 为行权限提供动态值；apikey 服务外部 API。它们最终都被 chat/datasource 消费。

## 删除与更新

管理 API 不应只做单表 delete。用户、工作空间、模型和助手都有映射/引用；删除前需检查或级联清理。敏感配置更新要保留未修改值并清缓存。

