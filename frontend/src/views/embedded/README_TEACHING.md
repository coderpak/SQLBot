# views/embedded 嵌入模块教程

## 核心作用

允许外部系统以助手 token、证书和来源域名加载 SQLBot 的完整问数、弹窗或单页模式，同时复用 chat 组件并按配置隐藏管理能力。

- `common.vue`：嵌入公共初始化与身份上下文。
- `index.vue`：主要嵌入入口。
- `page.vue`：页面式嵌入。
- `AssistantPreview.vue`：管理端预览助手效果。

请求层会根据 assistant type 设置 `X-SQLBOT-ASSISTANT-TOKEN`、certificate、host-origin 和 online Header，并移除普通用户 token。来源校验、动态 CORS 和证书仍由后端 system/assistant 负责。

