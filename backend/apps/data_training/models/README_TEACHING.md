# data_training/models 目录教程

- `data_training_model.py`：问题、SQL、数据源/工作空间归属、embedding 等训练样例字段模型。
- `__init__.py`：包声明。

模型字段围绕 question、sql、datasource/advanced_application、oid、enabled、embedding 和审计时间。question 是向量检索文本，SQL 是 few-shot 答案，两者的修改都应触发重新计算或重新验证。

VECTOR 字段依赖系统 PostgreSQL 的 pgvector 扩展；普通业务数据库不需要安装 pgvector。

## 查询视角

列表响应通常还需要关联数据源/高级应用名称，因此 ORM 表模型与 `DataTrainingInfoResult` 等展示 DTO 分离。不要把 VECTOR 直接序列化到普通 API。

## 生命周期

创建生成 id/审计字段和 embedding；更新 question 时重算向量，更新 SQL 时至少重新验证；enabled 控制召回；删除可按业务选择硬删或保留审计记录。

