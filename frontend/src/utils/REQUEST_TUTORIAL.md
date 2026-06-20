# `request.ts` 请求层教程

普通 REST 请求使用 Axios：统一 baseURL、token、助手/嵌入证书、语言头、JSONBig 解析、502 重试与错误提示。

问数流式请求使用 `fetchStream`，因为需要直接访问 `Response.body.getReader()`。它必须手工重复认证头逻辑；以后新增认证头时，Axios interceptor 和 `fetchStream` 两处都要同步，否则普通接口正常而问数接口会 401。

`fetchStream` 不解析 SSE，只返回原生 Response；事件缓冲与 JSONBig 解析在 `ChartAnswer.vue`。AbortController 用于用户点击停止和组件卸载时取消请求。

