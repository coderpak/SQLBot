# system/models 目录教程

| 文件 | 作用 |
|---|---|
| `system_model.py` | AI 模型、模型-工作空间映射、工作空间、用户-工作空间、助手、认证、API Key 等表模型。 |
| `user.py` | 用户主体、用户状态和认证相关持久化字段。 |
| `system_variable_model.py` | 系统变量及用户绑定值模型。 |
| `__init__.py` | 包声明。 |

多数主键继承 `SnowflakeBase`，返回前端时要注意 JS 安全整数问题。

## 关系模型

模型与工作空间、用户与工作空间使用独立 mapping 表，便于多对多授权；助手带 oid 表示归属；ApiKey 带 uid/status；Authentication 保存外部认证类型与配置。不要把 mapping 误当成对象所有权：同一模型可映射多个空间。

## 基类与序列化

`SnowflakeBase` 用 default_factory 生成 BigInteger ID。部分 response DTO 用 field_serializer 把 ID 转字符串；新增 DTO 时也要遵循同一策略。配置 JSON/Text 字段需要在 service 层解析与验证，model 只保证数据库类型。

## 表结构修改

修改 nullable/default/长度不仅改 Python Field，还要写 Alembic。新增敏感字段要设计加密、掩码、迁移旧值和审计脱敏。

