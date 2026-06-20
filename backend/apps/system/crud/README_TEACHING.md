# system/crud 目录教程

| 文件 | 作用 |
|---|---|
| `aimodel_manage.py` | 模型加密存储、查询、默认模型、工作空间映射和启动兼容处理。 |
| `apikey_manage.py` | API Key 生成、查询、状态与校验。 |
| `assistant.py` | 助手运行期对象、动态数据源工厂、CORS 等集成能力。 |
| `assistant_manage.py` | 助手配置的数据库 CRUD。 |
| `parameter_manage.py` | 系统参数读取、分组和更新。 |
| `system_variable.py` | 系统/用户变量管理与值解析。 |
| `user.py` | 用户认证、CRUD、密码、工作空间关系。 |
| `user_excel.py` | 用户 Excel 导入导出。 |
| `workspace.py` | 工作空间和成员关系 CRUD。 |
| `__init__.py` | 包声明。 |

问数时 `LLMService.create` 会从 `aimodel_manage` 获取当前工作空间的可用模型；身份依赖则会经过 `user`、`workspace` 和助手服务。

## 服务层边界

CRUD 层封装多表查询、加解密、映射维护和领域校验，API 层只处理 HTTP。`aimodel_manage` 和 `assistant` 还承担运行期适配：前者输出 LLMConfig，后者把外部数据源 API 适配成统一 AssistantOutDs。

## 缓存与一致性

模型、助手、动态 CORS、参数等可能被缓存。更新数据库后要清对应缓存，否则管理页显示新值而问数仍用旧值。工作空间映射更新应在事务内完成，避免模型短暂对所有空间不可见。

## 外部数据源助手

`AssistantOutDs.get_ds_from_api/get_db_schema/get_ds` 通过助手 domain 调外部端点，并把返回字典转换为 AssistantOutDsSchema。外部响应是不可信输入，要校验 URL、超时、字段、数据库类型和凭证，避免 SSRF 与配置注入。

