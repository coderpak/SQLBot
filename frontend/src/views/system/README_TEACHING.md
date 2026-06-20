# views/system 系统管理模块教程

## 核心作用

这些页面配置 AI 问数的控制面：模型、租户、用户、助手、权限、业务知识和运行参数。配置改变后，真正效果体现在 chat/datasource 后端流程。

## 页面映射

- `model/`：模型供应商、API、默认模型与参数。
- `workspace/`、`member/`、`user/`：工作空间、成员和用户生命周期。
- `permission/`：数据源、表、字段和行过滤树。
- `prompt/`：术语/自定义提示词运营。
- `training/`：问题-SQL 样例运营。
- `embedded/`：助手/嵌入应用、数据源和 UI。
- `variables/`：系统变量与用户绑定，用于行权限。
- `parameter/`：聊天行数、上下文轮数等动态参数。
- `authentication/`：LDAP/OIDC/OAuth2/SAML/CAS 等认证配置。
- `audit/`：系统操作审计。
- `appearance/`、`platform/`、`professional/`：外观、平台和扩展能力。

## 表单开发原则

前端类型、后端 Pydantic DTO、数据库 model 和 Alembic 必须一致。API Key/密码编辑时不要把掩码值重新提交；测试连接与保存应是两个动作；工作空间授权使用 Snowflake string ID，避免 Number 精度丢失。

