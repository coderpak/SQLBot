# datasource/api 目录教程

## 文件

- `datasource.py`：数据源 CRUD、连接测试、同步表字段、预览、Excel 上传等路由。
- `table_relation.py`：维护表之间的关系图；这些关系会补充到 Text-to-SQL Schema 的 Foreign keys。
- `recommended_problem.py`：每个数据源的推荐问题/推荐图表配置接口。
- `__init__.py`：包声明。

接口层通过权限装饰器限制工作空间资源。连接密码等敏感配置在持久化前会加密。

## 接口阶段

连接阶段：check/check_by_id/add/update/delete；元数据阶段：get_schema/get_tables/get_fields/choose_tables/sync_fields；治理阶段：edit_table/edit_field、table_list/field_list、关系与推荐问题；验证阶段：preview_data/field_enum；Excel 阶段：upload/parse/import；迁移阶段：export/upload_ds_schema。

耗时的连接、同步、预览和文件解析常通过 `asyncio.to_thread` 执行，避免阻塞事件循环。上传接口需限制扩展名、大小、Sheet/表名，并使用服务端生成路径，不能信任客户端文件名。

## 与 CRUD 的边界

API 解析 Path/Query/UploadFile、执行权限和审计，再调用 crud；数据库连接细节、加密、元数据 SQL 和持久化应留在 crud/db/utils。若一个 route 内出现大量方言判断，通常说明逻辑应下沉。

