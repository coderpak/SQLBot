# swagger 目录教程

- `i18n.py`：Swagger tag 元数据、占位符、语言列表和翻译加载；`main.py` 生成 OpenAPI 时替换占位符。
- `locales/`：Swagger 中英文 JSON 文案。
- `__init__.py`：包声明。

项目关闭 FastAPI 默认 docs，自己提供按请求语言生成并缓存的 `/docs` 和 `/openapi.json`。

## 工作过程

API 的 summary/description 使用 `PLACEHOLDER_PREFIX + key`；`main.py` 根据 query `lang` 或 Accept-Language 选择翻译，生成 OpenAPI 后递归替换占位符，并按语言缓存 schema。tags_metadata 同样会本地化。

## 新接口检查

1. 路由有清晰 tag、summary 和字段 description。
2. 新 key 同时加入 `locales/zh.json` 与 `en.json`。
3. 修改文案后清理/重启 OpenAPI 缓存。
4. 不在文档示例中泄露真实 token、数据库密码或模型 Key。

