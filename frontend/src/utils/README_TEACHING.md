# frontend/src/utils 模块教程

## 文件职责

- `request.ts`：Axios + fetchStream 请求基础设施。
- `useCache.ts`：local/session storage 包装。
- `useEmitt.ts`：mitt 事件订阅与组件卸载清理。
- `markdown.ts`：Markdown 渲染与代码高亮配置。
- `xss.ts`：输出清洗，降低富文本/Markdown XSS 风险。
- `canvas.ts`：仪表板画布计算辅助。
- `date.ts`：日期格式化。
- `url.ts`：URL/上下文路径处理。
- `RemoteJs.ts`：动态脚本加载封装。
- `propTypes.ts`、`utils.ts`：通用类型和小工具。

## 安全与边界

模型返回的 Markdown/HTML 不可信，渲染前应清洗；动态脚本仅加载可信来源；缓存中的 token 仍可能被 XSS 读取，所以前端清洗和 CSP 很重要。通用工具不应偷偷依赖某个页面 store，否则复用时容易循环依赖。

问数请求细节见 `REQUEST_TUTORIAL.md`。

