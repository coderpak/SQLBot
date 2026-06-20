# dashboard/api 目录教程

- `dashboard_api.py`：仪表板列表、详情、创建、编辑、删除以及视图数据相关路由；调用 crud 服务并做权限/审计包装。
- `__init__.py`：包声明。

前端 `views/dashboard` 与这里对应。调试“加入仪表板”时从前端 `ChartBlock.vue::addToDashboard` 追到本接口。

## 接口语义

`list_resource_api/load_resource_api` 服务资源树与详情；create/update/delete_resource 管资源元数据；create/update_canvas 管画布 JSON；check_name 在创建/重命名前验证同级重名。API 通过 CurrentUser 把资源限制在当前工作空间。

创建与更新操作带 system_log，资源 ID 可能来自请求或返回结果。删除时除数据库记录外还要考虑子资源/画布引用的级联策略。

