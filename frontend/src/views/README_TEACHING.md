# frontend/src/views 页面模块总览

- `chat/`：AI 问数、SSE、执行日志、图表和衍生分析。
- `ds/`：数据源、元数据治理、表关系和推荐问题。
- `dashboard/`：资源树、可视化画布、编辑与预览。
- `system/`：模型、用户、工作空间、权限、术语、训练、变量、助手和认证管理。
- `embedded/`：助手/页面嵌入运行模式。
- `login/`：本地和扩展认证登录。
- `work/`：工作台入口和数据源快捷卡片。
- `error/`：401/404 等错误页。
- `WelcomeView.vue`：默认欢迎/入口页面。

业务页面可以组合 components、stores 和 api，但应避免页面之间直接 import 对方内部组件；需要复用时上移到 components 或明确的业务公共目录。

