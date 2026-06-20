# frontend/src/i18n 国际化教程

- `index.ts`：创建 vue-i18n、选择缓存/浏览器语言和导出实例。
- `zh-CN.json`、`zh-TW.json`、`en.json`、`ko-KR.json`：前端 UI 文案。

前端文案与后端 `locales`、Swagger locales 分开。新增 key 时应补所有语言或明确 fallback；组件脚本中使用 `useI18n`，路由等非组件代码使用全局 i18n 实例。

