# frontend 学习指南

## 目录职责

Vue 3 + TypeScript + Vite 单页应用。Element Plus 提供界面组件，Pinia 管理状态，Axios 处理普通接口，原生 fetch 读取 SSE，AntV G2/S2/X6 分别用于图表、表格和表关系画布。

## 顶层文件

| 文件 | 作用 |
|---|---|
| `package.json` | 依赖与 dev/build/lint/preview 命令。 |
| `vite.config.ts` | Vue、自动导入、Element Plus、SVG、`@` 别名和构建分包。 |
| `.env.development` / `.env.production` | API 根地址和应用标题。 |
| `index.html` | 主应用 HTML 入口。 |
| `embedded.html` | 嵌入式场景 HTML 入口。 |
| `tsconfig*.json` | 浏览器、Node/Vite 和工程引用的 TS 配置。 |
| `eslint.config.cjs`、`.prettierrc` | 代码质量与格式。 |
| `auto-imports.d.ts`、`.eslintrc-auto-import.json` | 自动导入生成的类型/规则。 |

## src 子目录

| 目录 | 作用 |
|---|---|
| `api/` | 与后端 `/api/v1` 对应的请求封装和前端实体转换。 |
| `views/` | 页面；`chat/` 是 AI 问数核心，另有数据源、仪表板和系统管理。 |
| `components/` | 跨页面复用组件。 |
| `stores/` | Pinia 状态，包括用户、助手、聊天配置、仪表板。 |
| `router/` | 路由与导航守卫。 |
| `utils/` | HTTP、缓存、加密、事件总线等基础工具。 |
| `entity/` | 通用前端实体。 |
| `i18n/` | 国际化配置和文本。 |
| `assets/` | 图片、SVG、主题样式。 |

## AI 问数前端主线

`views/chat/index.vue::sendMessage` 创建临时 ChatRecord → `answer/ChartAnswer.vue::sendMessage` 调用 `api/chat.ts::questionApi.add` → `utils/request.ts::fetchStream` 发起 POST → 循环解析 SSE → 收到 `sql-data` 拉取 rows → 收到 `chart` 保存配置 → `ChartBlock`、`DisplayChartBlock`、`ChartComponent` 渲染。

详见 `src/views/chat/CHAT_UI_TUTORIAL.md`。

## 模块导航

| 模块 | 教程 | 学习重点 |
|---|---|---|
| 启动与分层 | `src/README_TEACHING.md` | Vue app、分层和数据流 |
| API | `src/api/README_TEACHING.md` | 普通请求、SSE、大整数 |
| 路由 | `src/router/README_TEACHING.md` | hash 路由、动态菜单、守卫 |
| 状态 | `src/stores/README_TEACHING.md` | 用户/助手双身份、工作空间 |
| 工具 | `src/utils/README_TEACHING.md` | request、缓存、XSS、事件 |
| Chat | `src/views/chat/README_TEACHING.md` | 一轮问数的 UI 编排 |
| 数据源 | `src/views/ds/README_TEACHING.md` | 元数据治理如何影响 AI |
| Dashboard | `src/views/dashboard/README_TEACHING.md` | 画布、store、持久化 |
| 系统管理 | `src/views/system/README_TEACHING.md` | 模型/空间/权限/知识配置 |
| 嵌入 | `src/views/embedded/README_TEACHING.md` | Assistant token 与证书 |

## 前端调试顺序

先看浏览器 Network 的 URL/Header/Response；普通接口再看 Axios interceptor，流式接口看 fetchStream 与 SSE 缓冲；数据正确但 UI 不变时看 record 是否保持同一个响应式引用；图表问题最后看 chart JSON 的 value 是否等于 data key。

