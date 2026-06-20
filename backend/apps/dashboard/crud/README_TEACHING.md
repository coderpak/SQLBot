# dashboard/crud 目录教程

- `dashboard_service.py`：仪表板与视图的数据库读写、结构转换和业务校验，是 API 与 models 之间的服务层。
- `__init__.py`：包声明。

## 服务分组

`list_resource` 构建当前工作空间资源树；`load_resource` 读取单个资源和配置；`get_create_base_info` 统一创建人/工作空间/时间字段；create/update_resource 修改名称、父子层级等元数据；create/update_canvas 保存大块画布内容；`validate_name` 防同级重名；delete_resource 执行删除。

该层应保持工作空间过滤，即使 API 已鉴权也不能只按 resource_id 查询。画布 JSON 是前端协议，后端至少应验证资源归属、大小和基本可序列化性。

