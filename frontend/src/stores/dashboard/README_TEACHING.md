# stores/dashboard 模块教程

- `dashboard.ts`：画布组件列表、当前选中组件、画布尺寸、编辑状态、Tab 碰撞等共享状态与 action。
- `snapshot.ts`：把可序列化画布状态压入历史栈，提供撤销/重做。

编辑器、CanvasCore、Toolbar、预览和组件都依赖 dashboard store。更新组件时要维持响应式引用；做快照前应深拷贝纯数据，不能把 DOM、Chart 实例或 Vue proxy 当作持久化配置。

