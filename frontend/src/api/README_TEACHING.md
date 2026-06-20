# frontend/src/api 模块教程

## 核心作用

把后端 URL、HTTP 方法和参数结构封装成有业务名称的函数。API 层应保持薄：不操作 DOM、不弹复杂 UI、不保存全局状态。

## 文件分组

- `chat.ts`：Chat/ChatRecord 类型转换、会话、提问、数据、分析、预测和导出。
- `datasource.ts`：连接测试、数据源 CRUD、表字段、关系、预览、同步和 Schema 导入导出。
- `system.ts`：AI 模型管理；其他 system 域拆在 user/workspace/assistant/permissions/variables 等文件。
- `dashboard.ts`：资源树和画布 API。
- `training.ts`、`prompt.ts`：SQL 样例、术语/自定义提示词运营。
- `auth.ts`、`login.ts`、`embedded.ts`：普通登录与助手/嵌入认证。
- `audit.ts`：系统审计查询。
- `recommendedApi.ts`：推荐问题配置。
- `setting.ts`、`professional.ts`、`license.ts`：系统设置及扩展能力。

## 普通请求与流式请求

普通 API 使用 Axios 单例，成功时通常由 interceptor 解包 `code=0` 的 data。问数 `questionApi.add` 使用 fetchStream，返回原始 Response；调用者必须读取 body 并解析 SSE。两条路径的 token/助手证书/语言头要保持一致。

## 类型和大整数

Snowflake ID 可能超过 JS 安全整数。普通响应由 Axios JSONBig transform 解析，SSE 在 ChartAnswer 中使用 JSONBig。新增实体时优先把 ID 类型设计为 string 或能容纳 JSONBig 的类型，避免 `Number(id)`。

## 新增 API 检查

确认后端前缀、方法、参数放 query/path/body 的位置；确认统一响应是否已解包；下载需 blob；流式需 AbortController；敏感请求不要记录完整 body。

