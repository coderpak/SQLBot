# system/middleware 目录教程

- `auth.py`：`TokenMiddleware` 解析普通用户 token、助手 token、嵌入证书/API Key 等认证头，把身份写入请求上下文，并处理公开路径。
- `__init__.py`：包声明。

前端普通请求使用 `X-SQLBOT-TOKEN`；助手/嵌入场景使用 `X-SQLBOT-ASSISTANT-TOKEN` 等头。新增公开接口时不要随意绕过中间件，应明确认证模型。

## 请求身份分支

普通站点解析 Bearer 用户 token；助手请求解析 Assistant/Embedded 前缀、证书、来源域名和在线标志；白名单路径允许登录、静态资源或必要回调。解析结果写入 request，后续 deps 和权限装饰器消费。

## 安全重点

- token 必须校验签名、过期和用户/助手状态。
- Embedded 来源与动态 CORS 不能只信 Header，应和助手配置匹配。
- 白名单使用精确规则，避免前缀误匹配放开管理接口。
- 错误响应不要泄露“token 存在但属于哪个用户”等信息。

## 调试顺序

检查浏览器实际 Header → 中间件选择了哪种身份分支 → request 中是否写入 user/assistant → deps 是否正确读取 → require_permissions 是否通过。

