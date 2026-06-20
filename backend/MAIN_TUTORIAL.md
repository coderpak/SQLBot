# `main.py` 启动入口教程

## 导入阶段

模块创建两个应用：`app` 是 8000 端口的 Web/API 应用；`mcp_app` 是 8001 端口的 MCP 与图片静态服务。`apps.api.api_router` 被统一挂到 `settings.API_V1_STR`，默认 `/api/v1`。

## lifespan 启动顺序

1. `run_migrations`：Alembic 升到 head。
2. `init_sqlbot_cache`：选择 memory 或 Redis。
3. `init_dynamic_cors`：加入助手配置的动态来源。
4. 补齐术语、训练样例、表和数据源 embedding。
5. 清理/初始化授权扩展缓存，处理已有模型敏感配置。
6. 应用 ready；退出时记录关闭日志。

因此“服务端口已监听”不一定代表全部初始化已完成，首次运行要观察日志。

## 中间件

- `CORSMiddleware`：浏览器跨域。
- `TokenMiddleware`：解析用户/助手/嵌入身份。
- `ResponseMiddleware`：统一普通 JSON 响应。
- `RequestContextMiddleware`：权限上下文。
- `RequestContextMiddlewareCommon`：审计上下文。

SSE 是流式响应，修改统一响应中间件时要确认不会把 StreamingResponse 再包成 JSON。

## Swagger 与 MCP

项目按请求语言动态生成 OpenAPI，并把占位符替换为中英文文本。FastApiMCP 只公开 allowlist 中的 operation，不会自动暴露所有管理 API。MCP 图片通过 `MCP_IMAGE_PATH` 挂载到 `/images`。

