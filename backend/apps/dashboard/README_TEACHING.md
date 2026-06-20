# dashboard 目录教程

## 作用

把问数结果保存成可复用仪表板视图，并提供仪表板编辑/预览数据。

## 文件与子目录

- `api/`：仪表板 HTTP 接口。
- `crud/`：仪表板查询、保存和更新服务。
- `models/`：仪表板、视图及布局相关 SQLModel/DTO。
- `__init__.py`：包声明。

## 核心作用

Chat 回答面向一次会话，Dashboard 面向持续消费。它把问数产生的 SQL、数据源、图表字段映射、标题和样式转成可保存的 view，再把多个 view 与文本/Tab 等组件组织到画布。

## 后端结构

API 提供资源树列表、加载、创建/更新/删除、canvas 保存和重名检查；service 负责工作空间/用户归属、资源基本信息和持久化；models 保存资源层级与画布 JSON。目录树元数据与大块画布配置分开，便于独立更新。

## 与 Chat 的连接

前端 `ChartBlock.addToDashboard` 从 ChatRecord 提取 data、sql、datasource、chart.type/axis/columns，转换成 dashboard view。后端并不会重新理解自然语言问题，它保存的是已经生成的可视化定义。

## 失效风险

数据源被删除、字段改名、行列权限变化或 SQL 不再合法时，旧看板会受到影响。保存时要保留稳定资源 ID；复制组件时要生成新组件 ID；编辑器快照要避免把不可序列化的 Vue 对象写入 JSON。

