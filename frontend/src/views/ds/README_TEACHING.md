# views/ds 数据源模块教程

## 核心作用

这是 AI 问数的数据准备台，不只是连接管理。用户在这里决定哪些表参与问数、表字段如何解释、表之间如何关联，以及数据预览和推荐问题。

## 组件分工

- `index.vue`、`DatasourceList*`、`Card`：数据源列表与选择。
- `AddDrawer`、`DatasourceForm`、`ParamsForm`：不同数据库连接参数、测试与保存。
- `ExcelDetailDialog`、`SheetTabs`：Excel 多 Sheet 导入。
- `TableList`、`DataTable`：选择表、同步/编辑字段和业务注释。
- `TableRelationship`：使用 X6 编辑表关系图，后端会把边转换成 Foreign keys prompt。
- `RecommendedProblemConfigDialog`：配置入口问题/推荐图表。
- `js/ds-type.ts`：数据源类型与表单元数据。
- `js/aes.ts`：前端必要的配置加密辅助；最终安全仍依赖 HTTPS 与后端加密。

## 典型操作链

新建并 check → 获取 schema/table → chooseTables → 同步 fields → 补充 comment → 保存关系 → 后端计算 embedding → ChatCreator 列出可用数据源。

## 调试重点

连接成功但问数无表：检查是否 chooseTables；表已选但模型不理解：检查 custom_comment；多表 SQL 不会连接：检查关系图字段端口；字段更新后权限失效：重新核对权限规则引用 ID。

