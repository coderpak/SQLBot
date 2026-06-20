# frontend/src/stores 模块教程

## Store 职责

- `user.ts`：token、用户、当前工作空间、权限、语言和菜单所需状态。
- `assistant.ts`：助手/嵌入 token、证书、类型、来源、在线状态和请求协调。
- `chatConfig.ts`：聊天界面开关，例如是否显示 SQL。
- `appearance.ts`：主题和品牌外观状态。
- `dashboard/`：画布状态与撤销快照。
- `index.ts`：Pinia 初始化/导出。

## 状态边界

跨路由仍需保留、多个远距离组件共享、需要统一变更的状态放 Pinia；输入框、弹窗开关、单组件 loading 留在局部 ref。Store action 可以组合 API，但不要直接依赖具体组件 ref。

## 用户与助手两套身份

完整站点通常使用 user token；嵌入页面使用 assistant token/证书。`request.ts` 根据 assistant store 决定 Header，并可能移除普通 token。调试 401 时要同时检查两个 store，而不是只看 localStorage。

## 持久化

部分状态通过 `useCache` 写 local/session storage。退出登录和切工作空间时必须清理或刷新相关缓存，防止旧工作空间资源短暂显示。

