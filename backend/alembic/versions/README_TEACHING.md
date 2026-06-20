# alembic/versions 迁移导读

这些文件按 revision/down_revision 串成升级链。文件名概括了版本意图：

| 文件 | 主要作用 |
|---|---|
| `001_ddl.py` | 初始系统表。 |
| `002_ddl_autogenerate.py` | 初始结构自动生成补充。 |
| `003_add_datasource.py` | 新增数据源结构。 |
| `004_add_aimodel_auto.py` | 新增 AI 模型配置。 |
| `005_table_and_field.py` | 新增表、字段元数据。 |
| `006_add_ds_field.py` | 扩展数据源字段。 |
| `007_add_chat.py` | 新增聊天与记录。 |
| `008_modify_field_type.py` | 调整字段类型。 |
| `009_0_modify_chat.py` | 调整聊天结构。 |
| `009_1_add_core_dashboard.py` | 新增仪表板。 |
| `010_upgrade_user_language.py` | 用户语言字段升级。 |
| `011_update_dashboard.py` | 仪表板结构升级。 |
| `012_license_ddl.py` | 授权相关结构。 |
| `013_modify_chat.py` | 聊天字段升级。 |
| `014_modify_chat_record.py` | ChatRecord 升级。 |
| `015_modify_chat.py` | 聊天字段升级。 |
| `016_modify_chat.py` | 聊天字段升级。 |
| `017_rsa_ddl.py` | RSA/安全相关结构。 |
| `018_modify_chat.py` | 聊天字段升级。 |
| `019_upgrade_model.py` | AI 模型配置升级。 |
| `020_workspace_ddl.py` | 新增工作空间。 |
| `021_user_ws_ddl.py` | 新增用户-工作空间映射。 |
| `022_assistant_ddl.py` | 新增助手配置。 |
| `023_modify_chat_record.py` | ChatRecord 扩展。 |
| `024_modify_chat_record.py` | ChatRecord 扩展。 |
| `025_ds_num.py` | 数据源编号/数量相关调整。 |
| `026_row_column_permission.py` | 新增行列权限。 |
| `027_modify_permission.py` | 权限结构调整。 |
| `028_ds_oid.py` | 数据源增加工作空间归属。 |
| `029_modify_chat.py` | 聊天结构调整。 |
| `030_permission_oid.py` | 权限增加工作空间归属。 |
| `031_modify_chat_record.py` | ChatRecord 扩展。 |
| `032_modify_assistant_ddl.py` | 助手结构调整。 |
| `033_chat_origin_ddl.py` | 记录聊天来源。 |
| `034_field_sort.py` | 字段排序。 |
| `035_sys_arg_ddl.py` | 新增系统参数。 |
| `036_modify_assistant.py` | 助手字段升级。 |
| `037_create_chat_log.py` | 新增 AI 步骤日志。 |
| `038_remove_chat_record_cloumns.py` | 清理 ChatRecord 旧列。 |
| `039_create_terminology.py` | 新增术语库。 |
| `040_modify_ai_model.py` | AI 模型字段升级。 |
| `041_add_terminology_oid.py` | 术语增加归属。 |
| `042_data_training.py` | 新增 SQL 训练样例。 |
| `043_modify_ds_id_type.py` | 调整数据源 ID 类型。 |
| `044_table_relation.py` | 新增表关系。 |
| `045_modify_terminolog.py` | 术语结构升级。 |
| `046_add_custom_prompt.py` | 新增自定义提示词。 |
| `047_table_embedding.py` | 新增表 embedding。 |
| `048_authentication_ddl.py` | 新增认证配置。 |
| `049_user_platform_ddl.py` | 用户平台信息。 |
| `050_modify_ddl_py.py` | 综合结构修订。 |
| `051_modify_data_training_ddl.py` | 训练样例升级。 |
| `052_add_recommended_problem.py` | 新增推荐问题。 |
| `053_update_chat.py` | 聊天升级。 |
| `054_update_chat_record_dll.py` | ChatRecord 升级。 |
| `055_add_system_logs.py` | 新增系统审计日志。 |
| `056_api_key_ddl.py` | 新增 API Key。 |
| `057_update_sys_log.py` | 审计日志升级。 |
| `058_update_chat.py` | 聊天升级。 |
| `059_chat_log_resource.py` | ChatLog 增加资源信息。 |
| `060_platform_token_ddl.py` | 平台 token。 |
| `061_assistant_oid_ddl.py` | 助手增加工作空间归属。 |
| `062_update_chat_log_dll.py` | ChatLog 升级。 |
| `063_update_chat_log_dll.py` | ChatLog 再升级。 |
| `064_system_variable.py` | 新增系统变量。 |
| `065_user_bind_var.py` | 新增用户变量绑定。 |
| `066_update_assistant_model.py` | 助手模型配置升级。 |
| `067_ai_model_workspace_mapping.py` | 新增模型-工作空间映射。 |
| `068_alter_sys_logs_user_agent_length.py` | 扩大审计日志 User-Agent 长度。 |
| `069_term_custom_prompt.py` | 术语与自定义提示词结构升级。 |

每个迁移文件都包含 `upgrade`/`downgrade`（或等价迁移定义）。理解当前表结构应以 models + 全部迁移链为准；不要只看 `001_ddl.py`。

