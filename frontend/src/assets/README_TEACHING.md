# frontend/src/assets 静态资源教程

包含主题样式、数据源/模型/权限/工作空间图片、嵌入资源和大量 SVG 图标。`assets/svg/chart` 对应问数图表类型，`svg/ds` 对应数据源类型，其他目录服务菜单和操作按钮。

SVG 由 Vite loader 作为组件导入。新增图标要保持 viewBox 和主题颜色策略，避免写死不适配深色模式的 fill；大位图应压缩，避免进入首屏 bundle。

