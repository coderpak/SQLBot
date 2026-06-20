# views/dashboard 模块教程

## 核心作用

把 Chat 中有价值的图表组织成可保存、可编辑、可预览的画布。它包含资源树、拖拽缩放、组件配置、Tab/文本/问数视图、快照和全屏预览。

## 子模块

- `common/ResourceTree`：目录/资源树 CRUD、拖拽和选择。
- `editor/`：编辑器容器、工具栏、图表选择和保存。
- `canvas/`：组件定位、缩放、拖动、碰撞与选择框。
- `components/`：sq-view、文本、Tab、按钮等画布组件。
- `preview/`：按保存配置只读渲染。
- `utils/`：树排序、坐标和拖拽算法。
- `common/AddViewDashboard`：从聊天图表进入看板的桥梁。

## 数据模型

画布配置必须是纯 JSON：组件 id/type、x/y/width/height、样式和业务 data。运行期 Chart 实例、DOM 和 Vue proxy 不应持久化。保存前后要保证 ID、字段映射和数据源引用稳定。

