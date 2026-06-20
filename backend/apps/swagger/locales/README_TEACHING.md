# swagger/locales 目录教程

- `zh.json`：Swagger 中文接口、字段和 tag 文案。
- `en.json`：对应英文文案。

新增 API 的 summary/description 若使用 `PLACEHOLDER_PREFIX`，必须在两份 JSON 中增加同名 key。

两份文件的 key 集应保持一致。翻译值既用于 route summary，也可能用于 Pydantic Field description；修改后需重启或清 OpenAPI 语言缓存。这里不放运行期错误消息，运行期文案属于 backend/locales。

## 校验建议

CI 可比较 zh/en 的 key 集并检查 JSON 格式。占位符未翻译时会直接显示内部 key，是最直观的漏项信号。

## 文案原则

summary 简短描述动作，description 说明参数/权限/返回；不要在两种语言中改变接口语义。枚举值和 JSON 字段名保持代码原样，不翻译。

