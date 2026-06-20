# frontend/src 源码总览

## 启动骨架

`main.ts` 创建 Vue app，安装 Pinia、router、i18n 与 UI 插件，再挂载 `App.vue`。`App.vue` 是根容器，`style.less` 是全局主题；`vite-env.d.ts` 提供 Vite 环境变量类型。

## 分层

- `api/`：无状态 HTTP 函数和部分响应实体转换。
- `stores/`：跨页面共享的用户、助手、聊天和仪表板状态。
- `router/`：页面地址、动态菜单和导航生命周期。
- `views/`：按业务域组织的页面。
- `components/`：跨业务复用 UI。
- `utils/`：HTTP、缓存、Markdown、事件、日期和安全转换。
- `entity/`：跨模块 TypeScript 实体/枚举。
- `i18n/`：浏览器 UI 多语言。
- `assets/`：主题、SVG 和静态图片。

## 数据流约定

页面触发 action → 调用 api 函数 → request 层加身份/语言并解析响应 → 页面更新局部 ref 或 Pinia 全局状态 → 组件由 props/computed 渲染。不要让 api 层操作组件状态，也不要把纯页面临时状态全部塞进 Pinia。

## 最重要的两条链路

问数：`views/chat` → `api/chat.ts` → `utils/request.fetchStream`。

资源配置：`views/system` 或 `views/ds` → 对应 api → 后端 system/datasource → 配置反过来影响 chat。

