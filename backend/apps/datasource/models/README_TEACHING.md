# datasource/models 目录教程

- `datasource.py`：定义数据源、表、字段、关系/权限相关 SQLModel，以及 `DatasourceConf` 等连接配置 DTO。连接配置通常以加密 JSON 存储。
- `__init__.py`：包声明。

重点区分：系统库模型描述“SQLBot 管理了哪些数据源”，它不会把业务表复制进系统 PostgreSQL；问数时仍直接连接外部业务库。

## 模型层次

`CoreDatasource` 是连接与治理根对象；`CoreTable`、`CoreField` 是同步后的元数据快照；`DatasourceConf` 是解密后的连接 DTO；CreateDatasource 等 DTO 控制 API 输入；权限/关系字段可能以 JSON 保存。

## 敏感字段

configuration 中包含账号密码，数据库只保存加密文本，API 详情应返回掩码或可编辑占位。更新时要区分“用户未改密码”和“把密码改为空”。模型的 `type`/type_name 会影响 DB 枚举、prompt 方言与前端表单。

## ID 与关系

表关系图的 cell/port 常引用 table/field Snowflake ID，前端必须当字符串处理。重新同步若重建 ID，会破坏关系和权限，因此同步逻辑应尽量稳定更新现有实体。

