# dashboard/models 目录教程

- `dashboard_model.py`：仪表板、组件/视图、布局配置等持久化模型与传输结构。
- `__init__.py`：包声明。

阅读时把 JSON 配置字段与前端 `views/dashboard` 的画布结构对照。

模型通常包含资源 ID、名称、父 ID、资源类型、工作空间/创建人、画布内容和时间等字段。资源树记录“在哪里”，canvas 记录“里面是什么”。CreateDashboard/QueryDashboard 等 DTO 分别服务创建与查询更新。

画布内容虽以 JSON/Text 保存，但仍是版本化协议；前端组件 schema 变化时要提供默认值或迁移，避免旧看板无法打开。

