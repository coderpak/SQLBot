# terminology/models 目录教程

- `terminology_model.py`：术语、自定义提示词、归属范围、embedding 等 SQLModel/DTO。
- `__init__.py`：包声明。

模型保存 word、description、pid、工作空间、是否限定数据源、datasource/advanced_application、enabled 和 VECTOR embedding。word/description 是模型真正看到的业务知识；适用范围决定是否能被本轮问数召回。

自定义提示词与术语可能共用部分结构/迁移，但语义不同：术语描述业务口径，自定义提示词直接约束某类生成任务。

